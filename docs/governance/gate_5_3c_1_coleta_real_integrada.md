# Gate 5.3C.1 — Fechamento da cadeia real de coleta

## Motivo

O Gate 5.3C encerrou como PASS PARCIAL: 21 de 22 critérios com evidência de
PASS, mas o payload que percorria a execução manual era um payload de
controle MOCK (`{origem, mock_mode, mock_duration_ms}`), não o
evidence-bundle real do pipeline Gate 5.2A — violando a exigência de que
"o MOCK deve substituir apenas a etapa externa de inferência".

## Objetivo A — Coleta real integrada (sem duplicar)

O pipeline de coleta do Gate 5.2A (`gate52a_tab`) não foi duplicado nem
reescrito. Foi exposto como sub-rotina reutilizável via o mecanismo nativo
`link in` / `link out` (modo `return`) do Node-RED core, chamado
sincronamente por um nó `link call` a partir da execução manual
(`gate53b_tab`). A mesma implementação de coleta agora serve tanto o
disparo de teste manual original (`gate52a_inject`) quanto a execução
manual real — nenhum código duplicado, nenhum evidence-bundle paralelo.

### Cadeia final (execução manual real)

```
input_button (press)
  → trigger Node-RED
  → validação (lock check)
  → lock adquirido, origem=manual
  → preparing
  → router (origem=manual, sem falha forçada)
  → link call → gate52a_tab (coleta REAL: 12 leituras HA, normalização,
    montagem do evidence-bundle, validação determinística P0)
  → link out (return) com o evidence-bundle REAL
  → monta payload analítico REAL (execution_id + origem + bundle)
  → calling → delay MOCK → processing
  → executor MOCK (SOMENTE a inferência é simulada)
  → validando → validação de contrato (9 chaves, pos-POC)
  → finalize (success/failed) + libera lock
  → persistência em sensor.saude_sistema_analitico_status
```

Execuções de origem `system_test` (os injects de teste do Gate 5.3B/5.3C)
continuam usando o caminho de bypass direto (sem coleta real), preservando
a velocidade e o isolamento desses testes de FSM/lock/contrato — decisão
deliberada para não introduzir latência/dependência de infraestrutura real
em testes que não a exercitam por design. Isso não duplica a coleta: ela
tem uma única implementação, apenas nem todo caminho de teste a invoca.

### Evidência de coleta real (não fixture/cache/MOCK)

Execução manual `hc-mteucm07-lkfl7m1a`: bundle real com 12 entidades,
9973 bytes, `contract_version: evidence-bundle-0.2`, `collected_at`
coincidente com o momento da execução. Correlação comprovada contra o
estado ao vivo do HA (consultado independentemente, após a execução):

| Entidade | Bundle (execução manual) | HA (consulta independente) | Match |
|---|---|---|---|
| `person.wmoura` | state=home, last_updated=17:48:27.559 -03:00 | state=home, last_updated=14:48:27.559070 -03:00* | Sim |
| `binary_sensor.backups_stale` | state=off, last_changed=01:05:16.949 -03:00 | state=off, last_changed=01:05:16.949700 -03:00 | Sim |
| `sensor.usw_lite_8_poe_uptime` | last_updated=11:51:17.419 -03:00 | last_updated=11:51:17.419873 -03:00 | Sim |

*\*bundle registra em UTC (17:48Z = 14:48 -03:00); os relógios de parede
batem exatamente após a conversão de fuso.*

### Falha de coleta controlada

Teste dedicado (`forcar_falha_coleta`) desvia a execução para um handler
de falha SEM tocar a coleta real nem qualquer infraestrutura — resultado
`estado: failed`, `motivo_falha: coleta_falhou_simulada`, lock liberado,
zero retry. Uma falha real de coleta (erro/timeout do `link call`) é
capturada por um nó `catch` dedicado e tratada de forma idêntica
(`motivo_falha: coleta_falhou`).

## Objetivo B — Decisão de deadline (sem chamar Anthropic)

Investigado o código-fonte real do nó core `http request`
(`@node-red/nodes` v5.0.4, `core/network/21-httprequest.js`): **não existe
campo de timeout no editor do nó** — apenas um default global de instância
(`RED.settings.httpRequestTimeout`, 120000ms se não configurado) e um
**override dinâmico por mensagem**: `msg.requestTimeout` (milissegundos),
aplicado como `opts.timeout.request` da biblioteca `got` — um deadline
**total** (DNS + conexão + envio + espera + leitura), não apenas de uma
operação de socket individual. Isso é uma melhoria genuína sobre o
mecanismo do executor pós-POC Python (que usava `signal.SIGALRM` porque o
`timeout` do `urllib` cobria só operações de bloqueio individuais).

**Decisão arquitetural**: a futura chamada real usará `msg.requestTimeout`
setado imediatamente antes do nó `http request`, com `senderr: true` (para
que qualquer erro — incluindo timeout — vá exclusivamente para um nó
`catch` dedicado, nunca prossiga silenciosamente pelo fio normal). Valor
planejado para produção: 300000ms (mesma ordem de grandeza do deadline
pós-POC). Runtime escolhido: o nó core `http request` do próprio Node-RED
— nenhum executor externo, nenhuma dependência nova.

### Teste empírico (100% localhost, zero Anthropic)

Endpoint MOCK local (`http in` + `delay` configurável + `http response`,
todos nós core) simulando uma API lenta/rápida. Cliente com
`msg.requestTimeout = 2000` (2s):

- **A (atraso 500ms < deadline)**: sucesso, `{ok:true, delayed_ms:500}`
  recebido via round-trip HTTP real. PASS.
- **B (atraso 5000ms > deadline)**: abortado pelo próprio nó
  (`"no response from server"`), capturado pelo `catch` dedicado, roteado
  a um handler de falha controlada. PASS. Zero retry, zero segunda
  chamada.

## Nota de segurança

Uma chamada de inspeção do add-on (`ha_get_app`) retornou incidentalmente
o `credential_secret` mestre do Node-RED no campo de opções do add-on.
Registrado aqui apenas que essa exposição incidental ocorreu no output de
uma ferramenta — o valor não foi reproduzido, copiado ou usado em nenhum
artefato, conforme a regra permanente estabelecida no Gate 5.3B.1.

## Alterações no Node-RED (sem POST /flows integral)

- `PUT /flow/gate52a_tab`: +2 nós (`link in`, `link out` em modo return),
  19→21 nós. Nenhuma outra tab tocada.
- `PUT /flow/gate53b_tab`: +18 nós (router de coleta, `link call`, catch,
  montagem do payload real, handler de falha, teste de falha simulada,
  harness completo de teste de deadline localhost), 27→45 nós. Nenhuma
  outra tab tocada.
- Total final: 99 nós, 5 tabs. Home Office (17), Reset buttons (5),
  Reseta Everthing (3) inalterados byte-a-byte.

## Estimativa de tokens/custo futuro (ESTIMATIVA, não proporcional garantida)

O evidence-bundle real desta execução tem 9973 bytes. Uma estimativa
grosseira (~4 bytes/token, ordem de grandeza apenas) sugere algo em torno
de 2500 tokens só de bundle; somando o system prompt do executor pós-POC
(reforço de disciplina evidência×inferência, extenso), o total de entrada
provavelmente fica na mesma ordem de grandeza da chamada histórica do Gate
5.2B (5926 input tokens). **Isso não é uma previsão confiável** — o
tamanho real do payload varia com o estado do sistema no momento da coleta
e não há relação estritamente proporcional entre bytes de JSON e tokens.
Custo Anthropic real deste Gate: **US$ 0,00**.

## Testes executados (todos com MOCK apenas na inferência)

| Teste | Resultado |
|---|---|
| Nominal (coleta real + MOCK) | PASS — cadeia completa, bundle real de 12 entidades, contrato válido, persistido |
| Correlação bundle × HA | PASS — 3 entidades comparadas, todas idênticas |
| Concorrência pós-integração | PASS — 2ª rejeitada durante a coleta real da 1ª; 1 única coleta, 1 única inferência |
| Falha de coleta simulada | PASS — `coleta_falhou_simulada`, lock liberado, zero retry, infra real não tocada |
| Recuperação após falha | PASS — nova execução real aceita e concluída imediatamente |
| Regressão testes automáticos (`system_test`) | PASS — caminho de bypass intacto, sem latência de coleta |
| Deadline A (resposta a tempo) | PASS |
| Deadline B (resposta atrasada) | PASS — abortado, capturado, falha controlada |
| Regressão Node-RED | PASS — 99 nós, todas as tabs pré-existentes intactas |
| Sensor determinístico | PASS — `sensor.saude_sistema_status` inalterado |

## Confirmações finais

Zero chamadas Anthropic. Zero uso de `ANTHROPIC_API_KEY` real. Zero ação
física. Zero retry automático em qualquer caminho (coleta, inferência,
deadline, validação, persistência). Nenhum commit, nenhum push.

## Riscos residuais

- Mesmos já registrados no Gate 5.3C (cosmético `contrato_ok` como string;
  reload de templates atualiza timestamp de sensores não relacionados).
- O `link call` tem timeout próprio de 30s para a chamada de coleta —
  valor não testado empiricamente neste Gate (só o timeout da futura
  chamada Anthropic foi testado); considerar revisão futura se a coleta
  real algum dia demorar mais que isso.
