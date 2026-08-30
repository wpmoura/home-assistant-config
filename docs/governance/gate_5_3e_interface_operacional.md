# Gate 5.3E — Persistência e interface operacional do Health Check (MOCK only)

## Status final: PASS

Fechado em 2026-08-29 após validação visual humana direta na interface real
(`Sistema Casa → Saúde do Sistema`). A única pendência que mantinha o
veredito técnico em PASS PARCIAL era a impossibilidade de obter screenshot
pelo mecanismo automatizado (`dashboard screenshot mode is disabled`) — não
um bloqueio técnico ou funcional. Essa pendência foi resolvida por
inspeção humana, não por nova evidência técnica automatizada; nenhum teste
foi repetido e nenhuma nova execução do Health Check foi disparada só para
fechar esta documentação.

### Validação visual humana (registro do resultado)

Confirmado por inspeção direta do usuário na view real:

- a área determinística preexistente (5 seções originais) permanece
  presente e intacta;
- separação visual clara entre a saúde determinística e a seção "Health
  Check Analítico (IA)";
- motor analítico identificado explicitamente como "MOCK — Homologação
  (Gate 5.3E)", com a ausência de chamada real à Anthropic declarada na
  própria interface;
- controle "Frequência automática" presente e utilizável;
- botão "Executar Health Check agora" presente e utilizável;
- política de agendamento visível na interface;
- card "Estado da Execução" exibindo corretamente estado, horário, origem
  e duração;
- bloco "Resultado da Análise (mais recente)" presente, conteúdo MOCK
  claramente identificado;
- bloco "Telemetria e Uso" presente, identificando motor MOCK, contrato de
  dados, tokens N/A em MOCK, custo real US$ 0,00 em MOCK, contadores de
  execução;
- nenhum card quebrado, sobreposição de componentes, campo essencial
  ilegível, ou mistura visual que faça o resultado da IA parecer ser a
  fonte determinística da verdade operacional.

Com isso, todos os critérios do Gate 5.3E têm evidência (funcional +
visual): **GATE 5.3E — PASS.**

### Princípio preservado

A IA não é a fonte da verdade operacional. A cadeia conceitual
(evidências reais → indicadores determinísticos → payload analítico →
camada de IA → diagnóstico complementar) permanece intacta: o resultado
analítico não altera a verdade determinística, não comanda ações físicas,
não é tratado como evidência, e permanece visualmente distinguível da
camada determinística — confirmado tanto pela auditoria técnica quanto
pela inspeção visual humana acima.

## Objetivo

Expor, na interface já existente do Home Assistant, os controles e o
resultado do Health Check Analítico homologado nos Gates 5.3B–5.3D — sem
criar uma segunda cadeia de execução, sem duplicar helpers, e sem redesenhar
nada que já existia. Toda a camada analítica permanece 100% MOCK: zero
chamada Anthropic real, zero uso de `ANTHROPIC_API_KEY`, zero ação física.

## Escopo da edição (aditivo, não destrutivo)

Dashboard `sistema-casa` (storage mode), view `saude-do-sistema` ("Saúde do
Sistema"). A view já continha 5 seções cobrindo a saúde **determinística**
(AT-HC-01, `sensor.saude_sistema_status`) — farol/status, contadores,
domínios, "o que preciso fazer hoje", alertas e evidências — nenhuma delas
foi tocada. A outra view do dashboard (`central-operacional`, "Casa") não
foi tocada. Nenhum outro dashboard foi tocado.

Edição feita via `ha_config_set_dashboard(python_transform=...)` (não
`config` integral), endereçando `config['views'][1]['sections'].extend(...)`
— apêndice puro de 4 novas seções ao final da lista já existente, validado
por `config_hash` fresco imediatamente antes da escrita. Confirmado por
releitura pós-escrita: as 4 seções pré-existentes permanecem byte-idênticas;
`config_hash` mudou de `072d4b5115c6fb35` para `f725da72f441b18a` refletindo
apenas o apêndice.

## Novas seções (camada analítica, MOCK)

1. **"Health Check Analítico (IA)"** — banner MOCK explícito ("⚠️ Motor
   analítico: MOCK — Homologação (Gate 5.3E)... Custo real desta camada:
   US$ 0,00"); tile nativo do `input_select.saude_sistema_health_check_frequencia`
   (feature `select-options`, dropdown) — mesmo helper do Gate 5.3D, sem
   duplicação; tile nativo do `input_button.saude_sistema_executar_health_check_manual`
   com `tap_action` explícito chamando `input_button.press` no mesmo
   entity_id — mesmo botão do Gate 5.3C, sem segunda cadeia; texto estático
   de política de agendamento (dias/horário fixo já documentado no Gate
   5.3D), puramente informativo.
2. **"Estado da Execução"** — markdown lendo `sensor.saude_sistema_analitico_status`:
   estado FSM mapeado para rótulos amigáveis (idle→"Aguardando",
   preparing/calling/processing/validating→frases descritivas,
   success/failed/interrupted→rótulos finais), sem inventar nenhum estado
   novo (vocabulário idêntico ao `estados_fsm` do contrato do sensor);
   última execução concluída (`finalizado_em`), origem mapeada
   (manual/scheduled/system_test → rótulos amigáveis), duração,
   frequência/janela vigente quando `origem=scheduled`, motivo da falha
   quando `estado=failed`.
3. **"Resultado da Análise (mais recente)"** — as 9 chaves do contrato
   (`resumo`, `avaliacao_geral`, `anomalias`, `riscos`, `recomendacoes`,
   `dados_insuficientes`, `possiveis_divergencias`) exibidas diretamente dos
   atributos já persistidos — nenhuma análise nova, nenhuma chamada. Os dois
   campos de risco/transparência (`dados_insuficientes`,
   `possiveis_divergencias`) são exibidos sempre que não vazios, nunca
   ocultados por limpeza visual.
4. **"Telemetria e Uso"** — motor analítico rotulado explicitamente como
   MOCK; validade de contrato (`contrato_ok`, tratado tanto como booleano
   quanto como string "true"/"false" — risco cosmético já registrado nos
   Gates anteriores); tokens de entrada/saída exibidos como "N/A (MOCK)"
   enquanto `input_tokens`/`output_tokens` forem `none`; custo exibido como
   "US$ 0,00 (MOCK)" enquanto `custo_usd` for `none` — nunca fabricado;
   contadores `contagem_execucoes_total/manual/scheduled` exibidos,
   explicitamente rotulados "todas via motor MOCK até o momento".

## Decisões de escopo (avaliadas e conscientemente não implementadas)

- **Projeção de custo na UI**: avaliada e **não incluída**. A estimativa
  ilustrativa já documentada no Gate 5.3D (multiplicação simples sobre um
  valor histórico do Gate 5.2B) não é uma previsão confiável e, exibida na
  interface operacional, poderia ser lida como um número ao vivo. Permanece
  apenas na documentação (`gate_5_3d_scheduler_frequencia.md`).
- **"Próxima execução agendada"**: avaliada e **não incluída**. Calculválo
  corretamente exigiria replicar em Jinja a política de dias/horário que
  vive em `gate53d_fn_decidir` (Node-RED) — duplicação de lógica com risco
  de divergência silenciosa entre as duas implementações. Em vez disso, a
  seção de controles mostra a política fixa (texto estático, idêntico ao já
  documentado) como referência, sem tentar prever a próxima data.
- **Indicador de "diagnóstico desatualizado" para a camada analítica**:
  avaliado e **não incluído**. Não existe hoje um watchdog equivalente ao
  `sensor.saude_sistema_watchdog` (AT-HC-01) para a camada analítica, e
  criar um exigiria nova lógica de frescor — fora do escopo aditivo deste
  Gate. Fica registrado como candidato a um Gate futuro, quando a chamada
  real tornar a frescura operacionalmente relevante.

## PENDÊNCIA TÉCNICA SEPARADA — REVISÃO DA POLÍTICA DE RECORDER DOS SENSORES DO HEALTH CHECK

Este achado é independente do fechamento do Gate 5.3E acima e **não é
bloqueante** para o veredito PASS deste Gate — é uma divergência entre
intenção documentada e comportamento real do recorder, a ser tratada em
escopo próprio, com avaliação de impacto antes de qualquer alteração
global no `recorder.yaml`. Nenhuma alteração de recorder, nenhum reinício
de Home Assistant e nenhuma exclusão foram feitos nesta etapa.

### Achado de auditoria — recorder (não corrigido neste Gate)

O comentário original de `packages/saude_sistema_analitico.yaml` (e também
o de `packages/saude_sistema.yaml`, AT-HC-01) afirma que o sensor está
"deliberadamente NÃO incluído em `recorder.include`". **Essa premissa está
incorreta**: `recorder.yaml` não usa um modelo de include-list — usa
`exclude` (domínios `updater/camera/image/button` + uma lista de
`entity_globs`), sem nenhum `include:`. Sob esse modelo, tudo que não casa
com uma regra de exclusão **é gravado por padrão**. `sensor.saude_sistema_status`
e `sensor.saude_sistema_analitico_status` não casam com nenhuma regra de
exclusão existente — logo, ambos **estão sendo gravados no recorder** a
cada mudança de estado, contrariando a intenção documentada.

Impacto prático avaliado como baixo (atributos por linha somam poucos KB;
execuções são esparsas — no máximo 1x/dia mesmo na frequência mais alta), mas
o achado é real e deve ser tratado como uma decisão pendente, não uma
correção silenciosa: alterar `recorder.yaml` afeta todo o sistema, tem maior
raio de impacto que este Gate (escopo estritamente de dashboard) e merece
sua própria decisão explícita — não foi alterado aqui. Recomendação: avaliar
em um Gate dedicado se `sensor.saude_sistema_analitico_status` (e talvez
`sensor.saude_sistema_status`) devem entrar no `exclude.entity_globs` do
recorder, corrigindo também o comentário desatualizado nos dois arquivos.

## Tamanho de atributos (auditoria)

Os atributos de `sensor.saude_sistema_analitico_status`, no estado
observado durante os testes deste Gate, somam poucos KB (a resposta bruta
observada, incluindo o atributo `contrato` estático, ficou bem abaixo de
16KB). Nenhum campo acumula histórico — todas as 9 chaves do contrato
refletem apenas a última execução, e os únicos campos cumulativos são
contadores inteiros simples. Sem risco de crescimento ilimitado.

## Testes executados (evidência real, via entidades de produção)

| Teste | Como | Resultado |
|---|---|---|
| Alteração de frequência via UI | `input_select.select_option` no helper de produção, "Desativado" → "1x por semana" → "Desativado" | PASS — `verified_state` confirmou cada transição; valor restaurado ao original |
| Execução manual via UI | `input_button.press` no botão de produção | PASS — cadeia completa: `origem=manual`, coleta real, MOCK, `contrato_ok=true`, persistido em `sensor.saude_sistema_analitico_status` (`execution_id hc-mtevsvnr-xfj8h17s`) |
| Concorrência via UI | Dois `input_button.press` consecutivos (~536ms de intervalo) no mesmo botão | PASS — `ultima_rejeicao_motivo=busy_execution_rejected`, `ultima_rejeicao_origem=manual`, timestamp de rejeição (21:17:21.076Z) cai dentro da janela de execução aceita (21:17:19.819Z–21:17:22.035Z); contadores confirmam exatamente 1 execução aceita (`contagem_execucoes_total` 8→9, `contagem_execucoes_manual` 2→3) apesar de 2 cliques |
| Falha controlada | Não re-testada nesta Gate | Já validada nos Gates 5.3C.1/5.3D via caminho de teste do Node-RED (`forcar_falha_coleta`); o botão de produção exposto na UI só aciona o caminho de sucesso simulado por design — comportamento correto para produção, não uma lacuna |
| Ação física | Inspeção do transform e da cadeia | PASS — nenhuma chamada a domínio de atuação física em nenhum novo card; `tap_action` do botão chama exclusivamente `input_button.press` no próprio helper |
| Zero Anthropic / zero API key | Inspeção do transform e da resposta observada | PASS — `modelo="mock"`, `input_tokens`/`output_tokens`/`custo_usd` = `null` |
| Regressão da view determinística | Releitura pós-escrita | PASS — as 4 seções originais (Saúde do Sistema, Domínios, O que preciso fazer hoje, Alertas e evidências) permanecem byte-idênticas |
| Regressão da outra view/dashboard | `render_paths` da resposta de escrita | PASS — `central-operacional` presente e inalterada; nenhum outro dashboard tocado |
| Confirmação visual (renderizada) | `ha_config_get_dashboard(include_screenshot=true)` | **NÃO OBTIDA** — `"Screenshot requested but dashboard screenshot mode is disabled"`. Limitação de ferramenta, não de configuração: o recurso beta de screenshot não está habilitado nesta instância. |

## Limitação declarada (visual) — RESOLVIDA

Na execução original deste Gate, a única verificação disponível havia sido
estrutural (config JSON válido, `write_committed`/`post_write_verified` =
true, seções corretas, tipos de card corretos, entidades corretas) e
funcional (testes acima, via chamadas reais de serviço HA). Não foi
possível obter uma captura de tela pelo mecanismo automatizado — o recurso
beta correspondente está desabilitado nesta instância
(`"dashboard screenshot mode is disabled"`) — o que manteve o veredito em
PASS PARCIAL.

**Essa pendência foi resolvida por inspeção visual humana direta** na
interface real (`Sistema Casa → Saúde do Sistema`), registrada na seção
"Status final: PASS" no topo deste documento. Nenhuma nova evidência
técnica automatizada foi produzida para este fechamento, e nenhuma
execução do Health Check foi repetida.

## Confirmações finais

Zero chamadas Anthropic. Zero uso de `ANTHROPIC_API_KEY` real. Zero ação
física. Zero retry. `sensor.saude_sistema_status` inalterado. Nenhum flow
Node-RED tocado (Gate 5.3E foi 100% do lado do dashboard HA — a regra
permanente de `POST /flows` seguiu não sendo necessária). Nenhum restart de
HA ou Node-RED. Nenhum commit, nenhum push.

## Riscos residuais

- Pendência técnica separada do recorder acima (decisão pendente, fora do
  escopo deste Gate, não bloqueante).
- Mesmos riscos cosméticos já registrados nos Gates 5.3C/5.3C.1/5.3D
  (`contrato_ok` como string em alguns caminhos).
- Restart real do Node-RED, substituição por chave Anthropic real e
  homologação da chamada real seguem explicitamente adiados para o Gate
  5.3F, **não iniciado nesta etapa** — aguardando autorização humana.

## Próximo Gate

**GATE 5.3F — HOMOLOGAÇÃO RUNTIME DO HEALTH CHECK.** Não iniciado nesta
execução.
