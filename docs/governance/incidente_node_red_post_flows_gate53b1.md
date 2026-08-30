# Incidente Node-RED — POST /flows truncado (Gate 5.3B.1)

## Causa

Durante o Gate 5.3B.1, ao tentar comprovar que a credencial FAKE de teste do
subflow `HC Executor (mock)` sobrevive a um redeploy normal, um `POST /flows`
foi disparado com um array **truncado** (colado parcialmente, interrompido em
torno do nó `9c361e0f546f0a98`) em vez do array completo de 74 nós. Como
`POST /flows` no Node-RED é uma operação de **substituição integral** do
conjunto de flows (não faz merge), o array truncado substituiu tudo.

## Impacto

- Estado anterior: 74 nós, 5 tabs (`Home Office`=17 filhos, `Reset buttons`=5,
  `Reseta botão Everthing`=3, `Gate 5.2A - Health Check POC (coleta)`=19,
  `Gate 5.3B - Fundacao Operacional Health Check`=22, mais a definição do
  subflow `gate53b_subflow_executor` e seu nó interno).
- Estado danificado: 16 nós, 3 tabs. `Gate 5.2A` e `Gate 5.3B` (com o subflow)
  desapareceram por completo; `Reset buttons` e `Reseta botão Everthing`
  ficaram vazias; `Home Office` perdeu 5 dos 17 nós.
- Perda: 58 nós (74 → 16).
- Nenhuma ação física, nenhuma automação de produção fora do escopo deste
  Gate foi afetada (as tabs pré-existentes fora do Gate 5.3B/5.2A tiveram
  apenas nós internos removidos, sem impacto operacional observado durante
  o incidente).
- `sensor.saude_sistema_status` (determinístico) não foi afetado em nenhum
  momento.

## Detecção

Detecção imediata pelo próprio operador (Claude Code) ao tentar ler o estado
via `GET /flows` logo após o POST truncado, seguida de confirmação visual
humana na UI do Node-RED.

## Recuperação

1. Congelamento de qualquer nova escrita até autorização humana explícita.
2. Backup forense do estado danificado (16 nós) salvo localmente antes de
   qualquer ação corretiva.
3. Snapshot de recuperação (74 nós), capturado por leitura real momentos
   antes do incidente, validado estruturalmente (JSON válido, IDs únicos,
   wires e referências `z` íntegras, 5 tabs com contagens esperadas, subflow
   presente, zero exposição de credencial FAKE, zero `ANTHROPIC_API_KEY`).
4. Restauração executada com **um único** `POST /flows` contendo o conteúdo
   integral do snapshot, lido diretamente do arquivo (sem reconstrução
   manual). Resultado: HTTP 204. Zero retry.
5. Verificação pós-restauração via `GET /flows`: 74 nós, 5 tabs, conjunto de
   IDs idêntico ao snapshot, zero diferenças de conteúdo/wires/`z`, subflow e
   nó interno restaurados, zero credencial exposta.
6. Validação humana posterior na UI do Node-RED: PASS — as cinco flows e os
   nós de `Home Office` que haviam desaparecido reapareceram.

## Efeito colateral identificado e corrigido

O `POST /flows` truncado excluiu temporariamente o nó de instância do
subflow (`gate53b_subflow_instance`) do array enviado. O mecanismo de
credenciais do runtime do Node-RED (`credentials.clean()`) remove do cache
de credenciais qualquer id de nó que não esteja presente no array recebido
num deploy. Consequência: a credencial FAKE de teste, armazenada antes do
incidente, foi apagada do cache mesmo após a restauração completa (que
reintroduziu o nó, mas sem a credencial). Isso foi diagnosticado
objetivamente (via o indicador booleano interno `credencial_presente_no_mock`,
nunca pela exposição do valor) e corrigido reestabelecendo a credencial
através do mecanismo de **menor blast radius** descrito abaixo — nenhum novo
`POST /flows` integral foi necessário.

## Nova regra operacional adotada

`POST /flows` é tratado, a partir deste incidente, como operação de
substituição integral e de alto risco para esta frente. Regras adotadas:

- Antes de qualquer `POST /flows`: `GET /flows` prévio, backup local
  íntegro, SHA-256 registrado, validação de JSON/IDs/wires/`z`/tabs, geração
  do payload final em arquivo e validação desse arquivo antes do envio —
  nunca reconstruir ou colar manualmente um array grande dentro de uma
  chamada, nunca presumir merge.
- Depois de qualquer `POST /flows` autorizado: zero retry automático,
  `GET /flows` de verificação, comparação estrutural completa; qualquer
  divergência interrompe o fluxo para decisão humana.
- Para alterações localizadas, preferir mecanismos de menor blast radius
  quando disponíveis e tecnicamente validados — nesta frente, confirmou-se
  via leitura do código-fonte do runtime (`@node-red/runtime` v5.0.4) que
  `PUT /flow/<id>` atualiza uma única tab por vez, preservando
  automaticamente as demais no processo em memória. Este foi o mecanismo
  usado para reestabelecer a credencial FAKE pós-incidente, sem tocar nas
  outras 4 tabs.

## Estado final

Incidente recuperado e validado (técnica e humanamente). Nenhuma perda
residual conhecida. Nenhuma credencial real ou FAKE exposta em qualquer
artefato. Nenhuma chamada Anthropic realizada em nenhuma etapa. Gate 5.3B.1
permaneceu suspenso durante todo o incidente e sua recuperação.
