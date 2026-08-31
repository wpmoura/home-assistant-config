# Gate 5.3C — Execução manual controlada do Health Check (MOCK)

## Objetivo

Implementar e validar a execução MANUAL do Health Check da Central
Operacional, reutilizando integralmente a fundação homologada nos Gates
5.3B/5.3B.1 (FSM, lock, credencial, publicação Node-RED↔HA). Toda a
validação deste Gate foi feita com o executor MOCK — nenhuma chamada
Anthropic real foi realizada.

## Arquitetura

```
input_button (press)
  → server-state-changed (Node-RED)
  → monta payload (origem=manual)
  → lock check (reaproveitado do Gate 5.3B)
  → preparing → calling → processing (reaproveitados)
  → executor MOCK (subflow, atualizado para o contrato pos-POC)
  → validando (NOVO estado de FSM)
  → validação de contrato (9 chaves do schema pos-POC homologado no Gate 5.2B)
  → finalize (success/failed) + libera lock
  → publica em sensor.saude_sistema_analitico_status
```

Nenhuma arquitetura paralela foi criada para o MOCK: a mesma cadeia de
lock/FSM/publicação usada nos testes automáticos do Gate 5.3B é reaproveitada
pelo trigger manual — apenas a origem muda (`origem: 'manual'`).

## Trigger manual

`input_button.saude_sistema_executar_health_check_manual` ("Saúde do Sistema
- Executar Health Check agora"). Escolhido `input_button` (não
`input_boolean`) porque cada `press` produz um timestamp de estado novo,
dispensando lógica de "reset" do helper — semântica nativa de botão
momentâneo, pronta para uso futuro no dashboard (Gate 5.3E, não
implementado). O helper de teste bidirecional do Gate 5.3B.1
(`input_boolean.gate53b1_teste_disparo_ha`) foi preservado sem alteração,
mantendo seu escopo original de validação de interface.

## FSM

Estados: `idle → preparing → calling → processing → validating → success`
ou `failed` (mais `interrupted` para reconciliação pós-restart). O novo
estado `validating` foi adicionado para tornar observável a etapa de
validação de contrato; a reconciliação pós-restart foi atualizada para
tratá-lo como transitório (interrompe corretamente se o Node-RED reiniciar
durante essa fase).

## Lock e concorrência

Reaproveitados sem alteração (`flow.hc_em_andamento`). Testado explicitamente
com uma execução MOCK de 15s mantida ativa e uma segunda solicitação
simultânea — rejeitada corretamente, execução original preservada.

## Contrato do executor MOCK

Atualizado para o schema pos-POC homologado no Gate 5.2B
(`executor_gate52b_pos_poc.py`, `JSON_SCHEMA`/`CHAVES_ESPERADAS`): 9 chaves
de nível superior — `resumo`, `avaliacao_geral`, `evidencias_relevantes`,
`anomalias`, `inferencias`, `riscos`, `recomendacoes`, `dados_insuficientes`,
`possiveis_divergencias`. Um novo nó de validação de contrato em Node-RED
reimplementa (em JS) a mesma lógica de validação estrutural do executor
Python pos-POC: JSON parseável, objeto (não array/primitivo), 9 chaves
exatas (sem faltantes/inesperadas), tipos corretos (string para
resumo/avaliacao_geral, lista de strings para as demais). Essa é a MESMA
validação que será usada quando uma resposta Anthropic real substituir o
MOCK — apenas a etapa de inferência externa é simulada.

## Persistência

`sensor.saude_sistema_analitico_status` (mesma entidade dos Gates
5.3B/5.3B.1) teve seu schema de atributos estendido (schema_version 2) para
refletir as 9 chaves do contrato, mais `contrato_ok` (booleano) e `modelo`
(populado com o literal `"mock"` sempre que uma execução é tentada — nunca
fabricado como chamada real). Apenas a ÚLTIMA execução é persistida (sem
histórico cumulativo). `input_tokens`/`output_tokens`/`custo_usd` permanecem
`null` — reservados para um Gate futuro que popule a chamada real.
`sensor.saude_sistema_status` (determinístico) nunca foi escrito por este
Gate.

## Telemetria MOCK

`modelo: "mock"` em toda execução tentada (sucesso ou falha) — deixa
inequívoco que nenhuma chamada Anthropic ocorreu. Tokens e custo
permanecem `null` (nunca fabricados como reais).

## Testes executados (todos com MOCK, zero Anthropic real)

| # | Teste | Resultado |
|---|---|---|
| 1 | Execução manual nominal (`input_button.press`) | PASS — FSM completa, contrato válido, persistido, lock liberado |
| 2 | Concorrência (reqA 15s ativa + reqB simultânea) | PASS — reqB rejeitada, reqA preservada, 1 execução |
| 3 | Contrato inválido — campo obrigatório ausente | PASS — `schema_nao_corresponde`, falha controlada |
| 4 | Erro interno do executor MOCK | PASS — `http_error_mock`, falha controlada |
| 5 | Deadline excedido | PASS — `deadline_total_excedido`, falha controlada |
| 5b | Contrato — JSON sintaticamente inválido | PASS — `json_invalido` |
| 5c | Contrato — tipo incompatível | PASS — `schema_nao_corresponde` (tipos_invalidos) |
| 6 | Recuperação após 5 falhas seguidas | PASS — nova execução manual aceita imediatamente, sem lock órfão |
| 7 | Persistência | PASS — resultado em `sensor.saude_sistema_analitico_status`; `sensor.saude_sistema_status` com conteúdo inalterado |
| 8 | Origem manual | PASS — `origem: "manual"` em todas as execuções via botão |
| 9 | Telemetria MOCK | PASS — `modelo: "mock"`, tokens/custo `null` |
| 10 | Regressão Node-RED | PASS — 79 nós totais; Home Office (17), Reset buttons (5), Reseta Everthing (3), Gate 5.2A (19) intactos; Gate 5.3B/5.3C (27) |

## Alterações no Node-RED (mecanismos de menor blast radius, sem POST /flows integral)

- `PUT /flow/global`: atualização apenas da definição do subflow executor
  (3 nós: server config + subflow + função interna) — nenhuma tab tocada.
- `PUT /flow/gate53b_tab`: adição de 5 nós (trigger manual, função de
  preparo, validador de contrato, 2 injects de teste extras) e rewiring
  mínimo dentro da própria tab — as outras 4 tabs (52 nós) permaneceram
  intocadas, confirmado por comparação de contagem antes/depois.

## Riscos residuais e observações

- O atributo `contrato_ok` é renderizado como string `"true"`/`"false"`
  pelo motor de template Jinja2 (em vez de booleano nativo) — cosmético,
  não afeta nenhum dos testes funcionais deste Gate; correção recomendada
  para um Gate futuro (`| bool` ou comparação direta).
- O `ha_reload_core(target="templates")` executado para carregar o schema
  novo do sensor analítico recarrega TODOS os template sensors do sistema,
  incluindo `sensor.saude_sistema_status` — isso atualizou seu
  `last_updated`/`last_changed`, mas **não alterou nenhum valor de estado
  ou atributo** (confirmado por comparação byte-a-byte antes/depois).
  Registrado para consciência de futuras operações de reload.
- Nenhuma outra regressão conhecida.

## Confirmações finais

- Zero chamadas Anthropic realizadas.
- Zero uso de `ANTHROPIC_API_KEY` real (ausente do ambiente, confirmado
  repetidamente).
- Zero ação física.
- Zero retry automático em qualquer caminho de código.
- Nenhum commit, nenhum push.

## LACUNA IDENTIFICADA (impede PASS integral)

A Seção 5 do Gate 5.3C exige que o MOCK substitua **apenas a etapa externa
de inferência** — a coleta/montagem do payload deveria continuar derivando
de evidências reais do Home Assistant (contrato já homologado no Gate 5.2A,
`gate52a_*`). A implementação atual **não conecta** essa coleta real à
cadeia de execução manual: o payload que percorre preparing/calling/
processing é um payload de controle (`{origem, mock_mode, mock_duration_ms}`),
não o evidence-bundle real produzido pelo pipeline `gate52a_*`.

Isso não invalida nenhum dos 10 testes executados (todos passaram
exatamente como especificado, usando MOCK também na coleta) — mas significa
que o critério 6 da Seção 25 ("coleta/payload funcional", no sentido de
coleta REAL integrada à cadeia manual) não está satisfeito. Correção
recomendada para antes da liberação da primeira execução manual real:
conectar a saída do pipeline `gate52a_*` (ou equivalente promovido a
produção) como entrada da etapa "preparing"/"calling" desta cadeia, no
lugar do payload de controle atual.
