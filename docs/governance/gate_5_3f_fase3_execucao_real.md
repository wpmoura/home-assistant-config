# Gate 5.3F — Fase 3 — Execução real única (autorizada)

## Status: EXECUTADO. Resultado: FALHA (`resposta_real_sem_texto`). Zero segunda chamada.

Autorização humana explícita concedida para exatamente uma chamada
Anthropic real manual. Executada exatamente uma vez. Não obteve sucesso.
Nenhuma segunda tentativa foi feita, conforme proibição absoluta da
autorização.

## Preflight (antes da chamada)

- Scheduler: `Desativado` (confirmado).
- `credencial_presente`: `true`, confirmado booleanamente minutos antes via
  `input_boolean.gate53b1_teste_disparo_ha` (execução `hc-mtfwelq5-xqzcnpfy`),
  valor nunca lido/exposto.
- Lock: livre (`false`), confirmado por leitura direta do contexto de flow.
- `anthropic_calls_this_gate`: garantido 0 por construção (guarda recém
  adicionada, nunca executada antes).
- Zero retry, deadline 300000ms, endpoint, modelo: reconfirmados por
  inspeção de código (inalterados desde a Fase 2).
- Ação física: impossível (nenhum nó de ação física em todo o pipeline).
- `sensor.saude_sistema_status`: somente leitura para a IA, confirmado por
  ausência de qualquer escrita a ele em todo o código do pipeline.

## Habilitação temporária (mínima, revertida imediatamente após)

Duas escritas cirúrgicas (`PUT /flow/global`, `PUT /flow/gate53b_tab`,
nunca `POST /flows`), cada uma com snapshot/hash/validação prévios e
releitura de verificação posterior:

1. **Guarda de chamada única** — prependida a
   `gate53b_sf_fn_build_real_request` (subflow): lê
   `flow.get('anthropic_calls_this_gate')`, recusa (`chamadas_reais_excedidas`)
   se ≥1, senão incrementa antes de prosseguir. Mantida permanentemente
   como camada extra de segurança (não é o mecanismo de dark path em si).
2. **Gatilho temporário** — `gate53c_fn_prep_manual` (botão de produção)
   teve `mock_mode: 'success'` trocado para `mock_mode: 'real_anthropic'`
   — a única mudança que tornou o caminho alcançável, e apenas por essa
   função específica (scheduler permaneceu com `'success'` hardcoded,
   nunca tocado).

## Execução

`input_button.saude_sistema_executar_health_check_manual` pressionado
exatamente uma vez, `wait:false`, em `2026-08-30T14:33:28.692Z`.

Cadeia observada (trace real do Node-RED):

| Passo | Timestamp (UTC) |
|---|---|
| `lock_check_entry` / `accepted` | 14:33:28.729 |
| `preparing` | 14:33:28.731 |
| `router_coleta` (origem=manual, coleta real acionada) | 14:33:28.737 |
| `coleta_real_concluida` (12 entidades, 10.306 bytes) | 14:33:29.716 |
| `calling` | 14:33:29.717 |
| `processing` | 14:33:30.709 |
| `validating` | 14:33:31.406 |
| `contract_skip_erro_executor` (motivo: `resposta_real_sem_texto`) | 14:33:31.406 |
| `failed` | 14:33:31.689 |

`execution_id`: `hc-mtfwtdi1-vw2cjpdf`. Duração total: `2.959s`.

## Achado anômalo durante a execução (registrado com transparência)

Entre `calling` e `processing`, um evento `reconcile_startup_read` /
`reconcile_decision` (`decisao: interrupted`, `estado_encontrado: calling`,
mesmo `execution_id`) apareceu no trace às 14:33:30.095Z — a rotina de
reconciliação pós-restart do próprio nó `gate53b_fn_reconcile` (que só
dispara via o inject `gate53b_startup_inject`, que só dispara após um
deploy/restart daquela tab). Isso indica que **um segundo ciclo de
redeploy/reinicialização da tab ocorreu durante a execução**, cuja causa
exata não foi determinada com certeza (não corresponde a nenhum `PUT`
adicional feito por mim — apenas os dois `PUT`s de habilitação, ambos
antes do clique). A execução, no entanto, **continuou e completou
normalmente** na sequência (`processing`→`validating`→`failed` para o
mesmo `execution_id`), indicando que o contexto de execução em memória
não foi de fato destruído.

**Hipótese mais provável (não confirmada com certeza):** a resposta HTTP
recebida foi um erro em nível de aplicação da própria Anthropic (ex.: HTTP
4xx — autenticação, validação de payload, etc.), que o nó `http request`
entrega normalmente pela saída de sucesso (não pelo `catch`, reservado a
erros de transporte/timeout) — um corpo de erro JSON não tem a estrutura
`content[].type==='text'`, produzindo exatamente `resposta_real_sem_texto`.
A duração total (`2.959s`) é ordens de grandeza menor que a latência real
histórica do Gate 5.2B (`259.717s`), consistente com uma resposta de erro
rápida em vez de uma geração completa.

**Limitação de instrumentação identificada:** `gate53b_sf_fn_parse_real_response`
não capturou `msg.statusCode` nem o corpo bruto da resposta em caso de
erro, e não chamou `tracelog(...)` — portanto não há evidência adicional
(código HTTP exato, mensagem de erro da Anthropic) disponível para
diagnóstico mais preciso. Isso é uma lacuna real da implementação da
Fase 2, registrada aqui para uma eventual Fase futura, e não foi
corrigida nem re-executada nesta etapa (proibido pela autorização).

## Persistência

`sensor.saude_sistema_analitico_status`: `state=failed`,
`execution_id=hc-mtfwtdi1-vw2cjpdf`, `motivo_falha=resposta_real_sem_texto`,
`contrato_ok=false`, **`modelo=claude-sonnet-5`** (confirma que a correção
de uma linha da Fase 2 em `gate53b_fn_finalize` funcionou exatamente como
projetado — a primeira vez que esse campo reflete algo além de `"mock"`).
`input_tokens`/`output_tokens`/`custo_usd` permanecem `null` — nenhum dado
de uso foi retornado (consistente com uma resposta de erro, que
tipicamente não carrega `usage`). Campos `resumo`/`avaliacao_geral`/etc.
mostram o conteúdo `[MOCK]` **residual da última execução MOCK bem
sucedida anterior** (preservado por design em caminhos de falha) — não é
um resultado desta chamada.

Contadores: `contagem_execucoes_total` 11→12, `contagem_execucoes_manual`
3→4 — exatamente uma execução adicional de origem `manual`, confirmando
que nenhuma segunda tentativa ocorreu.

## Custo real

**Não calculável com precisão** — nenhum `usage` (input/output tokens) foi
retornado, consistente com uma resposta de erro da API (que tipicamente
não é cobrada como geração, embora isso não tenha sido confirmado com uma
fonte oficial nesta etapa — nenhuma chamada adicional foi feita para
verificar). Não presumo US$ 0,00 com certeza absoluta, mas não há
evidência de tokens de geração consumidos. **Não usei o valor histórico
(US$ 0,036712) como custo desta chamada** — esse número permanece
exclusivamente como referência do Gate 5.2B.

## Reversão pós-execução (imediata)

`gate53c_fn_prep_manual` revertido para `mock_mode: 'success'` via novo
`PUT /flow/gate53b_tab` cirúrgico, verificado por releitura (`func`
idêntico ao original). Guarda de contagem mantida permanentemente (camada
extra de segurança, não afeta o comportamento MOCK). Scheduler
reconfirmado `Desativado`. Lock reconfirmado livre. `sensor.saude_sistema_status`
reconfirmado inalterado (`degraded`, mesmo valor de antes). Total de nós
no Node-RED: 118 (112 da Fase 1 + 6 da Fase 2), 5 tabs, todas presentes e
com a mesma configuração de `disabled` de sempre — zero regressão.

## Confirmações finais

Exatamente 1 chamada Anthropic autorizada, exatamente 1 realizada, 0
segunda tentativa, 0 retry automático. Caminho REAL novamente dark
(nenhum gatilho ativo produz `'real_anthropic'`). Pipeline operacional
observável: MOCK. `sensor.saude_sistema_status` preservado. Nenhum
commit, nenhum push, nenhum restart de HA/Node-RED, nenhuma ação física.

## Riscos residuais

- Causa exata do segundo ciclo de reconciliação durante a execução não
  determinada com certeza — registrada como achado, não como fato
  comprovado.
- Falta de captura de `msg.statusCode`/corpo de erro na Fase 2 impede
  diagnóstico mais preciso da causa raiz da falha — gap de instrumentação
  a corrigir em Fase futura, antes de qualquer nova autorização de chamada
  real.
- Telemetria real (tokens/custo) segue não persistida no sensor (gap já
  registrado na Fase 2).
- Pendência do recorder (Gate 5.3E) permanece separada, não bloqueante.
