# Gate 5.3F — Homologação runtime do Health Check — Fase 1 (Preflight)

## Status: FASE 1 CONCLUÍDA. Nenhuma chamada Anthropic real realizada.

Esta fase auditou e preparou (sem executar) o caminho para EXATAMENTE UMA
futura chamada Anthropic real, substituindo o MOCK apenas na fronteira de
inferência. Nenhum flow Node-RED foi alterado nesta fase — auditoria e
testes usaram exclusivamente o pipeline já existente e homologado
(Gates 5.3B–5.3E).

## 1. Baseline Git (antes e depois, idênticos)

`git status --short`: mesmos arquivos pré-existentes de outras frentes
(CSMR) modificados, nunca tocados por esta frente; `git diff --check`
limpo; `git diff --stat` idêntico (9 arquivos, 1696 inserções/69 deleções,
todos de outras frentes). Nenhuma alteração funcional nesta fase — apenas
este documento foi criado.

## 2. Snapshot de recuperação do Node-RED

`GET /flows` completo capturado via paginação (112 nós, 3 páginas),
reconstruído localmente e serializado canonicamente
(`json.dumps(..., sort_keys=True, separators=(',',':'))`).

- Arquivo: `snapshot_gate53f_fase1_pre.json` (scratchpad local, 69.873 bytes)
- **SHA-256:** `68c15776ea37ef1cfcdc5a66645bf5ea768422c3000129463aa6f3eeaf8d7bf5`
- Total de nós: 112 (confirma exatamente o total reportado pela própria
  API do Node-RED, `total_items:112`)

### Validação estrutural (automatizada)

| Checagem | Resultado |
|---|---|
| JSON bem formado | PASS (parse sem erro nas 3 páginas) |
| IDs duplicados | PASS — 0 duplicados |
| Tabs presentes | PASS — 5 tabs: Home Office (disabled), Reset buttons (disabled), Reseta botão Everthing, Gate 5.2A, Gate 5.3B — todas as esperadas, nenhuma nova/faltante |
| Subflows presentes | PASS — 1 subflow (`gate53b_subflow_executor`), o executor MOCK |
| Referências `z` órfãs (nó apontando para tab/subflow inexistente) | PASS — 0 órfãs |
| Alvos de `wires` inexistentes | PASS — 0 wires apontando para IDs que não existem |
| Segredo em claro (`sk-ant-...`) | PASS — 0 ocorrências em qualquer nó |
| Valor do env `anthropic_api_key` no template do subflow | PASS — `""` (vazio, confirmando que o valor real fica exclusivamente na instância, nunca no template — mecanismo já homologado no Gate 5.3B.1) |

**POST /flows não foi usado em nenhum momento desta fase** — nenhuma
escrita foi feita; a única necessidade era leitura para snapshot/auditoria.

## 3. Auditoria do pipeline atual (leitura, sem alteração)

Cadeia confirmada por leitura direta do flow real (`gate53b_tab` +
`gate52a_tab`, via `link call`/`link in`/`link out`):

```
trigger manual (input_button) / scheduled (tick 15/15min)
  -> gate53b_fn_lock_check (lock)
  -> gate53b_fn_preparing (FSM: preparing)
  -> gate53c_fn_router_coleta
  -> gate53c_link_call_coleta -> gate52a_tab (coleta real, 12 entidades)
  -> gate53c_fn_montar_payload_real (evidence-bundle real + payload_analitico_real)
  -> gate53b_fn_calling (FSM: calling)
  -> gate53b_delay_mock (simula duração)
  -> gate53b_fn_processing (FSM: processing)
  -> gate53b_subflow_instance  <-- FRONTEIRA MOCK
  -> gate53c_fn_validar_contrato (FSM: validating; 9 chaves)
  -> gate53b_fn_finalize (FSM: success/failed; libera lock; publica evento)
```

### Fronteira MOCK — identificação exata

- **Nó MOCK:** `gate53b_subflow_instance` (tipo `subflow:gate53b_subflow_executor`),
  que invoca internamente `gate53b_sf_fn_exec` ("Executor mock (le
  credencial, nunca expoe)").
- **Nó imediatamente anterior:** `gate53b_fn_processing` ("4. processing").
- **Nó imediatamente posterior:** `gate53c_fn_validar_contrato` ("GATE
  5.3C: validar contrato").
- **Entrada recebida pelo MOCK:** o `msg` completo da execução, incluindo
  `execution_id`, `origem`, `mock_mode`, `mock_duration_ms`, e — quando
  `origem` é `manual`/`scheduled` — `evidence_bundle_real`/
  `payload_analitico_real` (o payload real já montado). **Achado
  relevante:** o corpo atual do executor MOCK decide o resultado
  exclusivamente por `msg.mock_mode`; não lê nem envia
  `payload_analitico_real` a lugar nenhum — correto para um MOCK
  determinístico, e é exatamente o ponto onde a Fase 2 deve inserir o
  envio real desse payload como corpo da requisição HTTP.
- **Saída produzida pelo MOCK:** `msg.resposta_mock_texto` (string JSON
  simulando a resposta estruturada) OU `msg.executor_erro` (motivo de
  erro), mais `msg.credencial_presente` (booleano — nunca o valor da
  credencial).

Nenhuma alteração foi feita nesta fronteira ou em qualquer nó do pipeline.

## 4. Auditoria do payload real (sem envio à Anthropic)

Evidência obtida por leitura do contexto global do Node-RED
(`gate53b_trace`, últimas 50 entradas) — sem disparar nenhuma nova
execução, reaproveitando evidência real já produzida minutos antes desta
fase (Gate 5.3E, mesma sessão):

| Campo | Valor observado (execução `hc-mtevsvnr-xfj8h17s`) |
|---|---|
| `collected_at` | `2026-08-29T21:17:21.037Z` |
| `contract_version` | `evidence-bundle-0.2` |
| Quantidade de entidades | `12` |
| Tamanho do bundle | `9973 bytes` |
| Estrutura | idêntica à documentada no Gate 5.3C.1 (`collected_at`, `source`, `contract_version`, `entities[]`, `dominios_sem_evidencia[]`) |
| Correlação amostral com HA | já comprovada independentemente no Gate 5.3C.1 (3 entidades, match exato) — não repetida nesta fase por não ter havido nenhuma mudança no pipeline de coleta desde então |
| Segredo indevido no payload | Nenhum — o payload contém apenas estado/atributos whitelisted de entidades HA, sem credenciais |

Este payload é o mesmo que seria anexado ao corpo da futura chamada real
(campo `coleta` de `payload_analitico_real`).

## 5. Contrato Anthropic homologado (Gate 5.2B, reconfirmado por leitura)

| Item | Valor |
|---|---|
| Endpoint | `https://api.anthropic.com/v1/messages` |
| Modelo | `claude-sonnet-5` |
| Mecanismo de saída estruturada | `output_config.format` (JSON Schema nativo da Anthropic Messages API, confirmado documentalmente para `claude-sonnet-5`) + validação local defensiva (qualquer fence Markdown remanescente = violação de contrato, nunca corrigida automaticamente) |
| Chaves esperadas (9, nível superior) | `resumo`, `avaliacao_geral`, `evidencias_relevantes`, `anomalias`, `inferencias`, `riscos`, `recomendacoes`, `dados_insuficientes`, `possiveis_divergencias` |
| Regra evidência×inferência | disponibilidade/capacidade ≠ uso ativo; simultaneidade ≠ causalidade; ausência de evidência de uso → `dados_insuficientes`, nunca inferência de uso (regra 7 do system prompt, Gate 5.2B) |

Nenhuma modificação foi feita a este contrato. Nenhuma chamada de teste
foi feita contra `api.anthropic.com`.

**Risco residual explícito:** as 3 correções pós-POC (JSON Schema nativo,
regra evidência×inferência reforçada, deadline total) foram validadas
apenas OFFLINE (Gate 5.2B, 8/8 testes locais) — nunca contra o
comportamento real da Anthropic. Além disso, a arquitetura de execução
mudou desde então: o executor Python original (`urllib` + `SIGALRM`) foi
substituído, por decisão do Gate 5.3C.1, pelo nó nativo `http request` do
Node-RED com `msg.requestTimeout`. **A única chamada real autorizada na
Fase 2 será, portanto, a primeira vez que TODAS estas peças — schema
nativo, regra evidência×inferência, deadline via Node-RED, coleta real
deste pipeline — se encontram simultaneamente com a Anthropic real.**
Isso não é um bloqueio, mas deve ser lido com atenção redobrada ao
resultado dessa primeira chamada.

## 6. Credencial (auditoria, sem exposição)

Mecanismo já homologado no Gate 5.3B.1: env var `anthropic_api_key`
(tipo `cred`) no subflow, valor real fica exclusivamente na instância
(`gate53b_subflow_instance`), nunca no template. Confirmado por leitura de
`GET /flows`: o template mantém `value:""` (vazio) — consistente com o
mecanismo correto.

O executor MOCK reporta, a cada execução, `credencial_presente_no_mock`
(booleano, nunca o valor) — observado como `true` nas últimas execuções
reais desta sessão. **Isso confirma apenas que o mecanismo de leitura da
credencial está funcional**, usando o valor FAKE de teste estabelecido no
Gate 5.3B.1. Não foi feita, e não deve ser feita, nenhuma inspeção do
valor em si — impossível (e desnecessário) distinguir "fake" de "real"
sem violar a regra de nunca reproduzir/inspecionar o segredo.

**CREDENCIAL REAL AINDA NÃO CONFIGURADA** — nenhuma evidência neste
engajamento indica que o valor FAKE do Gate 5.3B.1 tenha sido substituído
por uma chave Anthropic real; nenhuma chave foi solicitada ao usuário
nesta fase, e nenhuma será, até autorização explícita da Fase 2.

## 7. Decisão arquitetural desta fase — caminho real NÃO implementado ainda

Avaliada a permissão da Seção 8 do Gate ("é permitido preparar o caminho
técnico... desde que não seja executado"), a decisão desta Fase 1 foi
**não inserir ainda o nó `http request` real no Node-RED**. Justificativa:

- A preparação é permitida, não obrigatória ("é permitido", não "deve").
- Construir um nó com endpoint/headers reais da Anthropic e mantê-lo
  parado no flow aumenta a superfície de risco (maior chance de uma
  futura edição o conectar acidentalmente) sem nenhum ganho de
  correção — o mecanismo (`msg.requestTimeout` + `senderr:true` + `catch`
  dedicado) já foi validado empiricamente com um endpoint MOCK local no
  Gate 5.3C.1, usando o mesmo tipo de nó.
- Preferência por construir e testar esse nó de forma atômica,
  imediatamente antes da única chamada real autorizada na Fase 2 — quando
  poderá ser exercitado ponta a ponta em um único passo controlado, em vez
  de ficar como infraestrutura inerte por tempo indeterminado.

A Seção 14 abaixo apresenta a especificação exata e completa desse nó,
pronta para implementação na Fase 2 — sem tê-la implementado agora.

## 8. Guardrails confirmados por leitura de código (sem execução real)

| Guardrail | Evidência |
|---|---|
| Máximo 1 chamada por execução | `gate53b_subflow_instance` tem saída única, sem laço, sem chamada condicional repetida |
| Zero retry | Nenhuma função em todo o pipeline (`gate53b_tab`/`gate52a_tab`) contém lógica de nova tentativa — confirmado por leitura de todas as 112 definições de nó |
| Lock ativo | `gate53b_fn_lock_check` — `flow.get('hc_em_andamento')`, testado e comprovado (2 rejeições reais nesta sessão) |
| Rejeição de concorrência | Comprovada 2x nesta sessão (Gate 5.3E: dois cliques reais no botão; trace confirma rejeição exatamente durante execução em andamento) |
| Deadline total | Decidido e testado com endpoint MOCK local (Gate 5.3C.1): `msg.requestTimeout`, sucesso e timeout ambos exercitados |
| Tratamento HTTP não-2xx / timeout | Mecanismo genérico (`senderr:true` + `catch` dedicado) comprovado com o mesmo tipo de nó (`http request`) contra um MOCK local — ainda não exercitado com o nó real da Anthropic (ver risco na Seção 5) |
| Tratamento JSON inválido | `gate53c_fn_validar_contrato` testado com `mock_mode: json_invalido` (Gate 5.3C) — PASS |
| Tratamento contrato inválido | Testado com `invalid_schema`/`tipo_invalido` (Gate 5.3C) — PASS |
| Finalize em sucesso ou falha | `gate53b_fn_finalize` libera o lock incondicionalmente na primeira linha, antes de qualquer outra lógica |
| Nenhuma ação física | Confirmado por leitura: nenhum nó `api-call-service` existe em `gate53b_tab`/`gate52a_tab` (apenas leituras `api-current-state` e `ha-fire-event`) |

## 9. Scheduler durante a homologação

`input_select.saude_sistema_health_check_frequencia` = **`Desativado`**,
confirmado por leitura direta do estado e pelo trace do próprio scheduler
(`scheduler_nao_devido`, `motivo:"desativado"`, última avaliação
`2026-08-29T21:24:22.339Z` — minutos antes desta fase). Nenhuma alteração
foi feita a este helper nesta fase.

## 10. Estimativa de custo (ordem de grandeza, não SLA)

Referência histórica (Gate 5.2B, única chamada real já feita):
`input_tokens≈5926`, `output_tokens≈2486`, custo `≈US$ 0.036712`, payload
enviado de `10.036 bytes`. O payload real atual (`9973 bytes`, 12
entidades) é praticamente do mesmo tamanho — não há indício de que o
próximo payload seria substancialmente maior ou menor. **Estimativa: a
próxima chamada real deve ficar na mesma ordem de grandeza (dezenas de
milhares de tokens de entrada, poucos milhares de saída, custo da ordem
de alguns centavos de dólar)** — não é uma previsão precisa nem um
compromisso de custo.

## 11. Testes executados nesta fase (todos sem chamada Anthropic)

| # | Teste | Resultado |
|---|---|---|
| 1 | Snapshot + validação estrutural do Node-RED | PASS (Seção 2) |
| 2 | Auditoria do pipeline/fronteira MOCK | PASS (Seção 3) |
| 3 | Auditoria do payload real (sem reexecutar) | PASS (Seção 4) — reaproveitou evidência real já produzida minutos antes |
| 4 | Contrato Anthropic (endpoint/modelo/schema/9 chaves) | PASS (Seção 5) |
| 5 | Credencial (mecanismo, sem inspecionar valor) | PASS (Seção 6) |
| 6 | Ausência de segredo em claro | PASS (Seção 2) |
| 7 | Zero retry (leitura de código) | PASS (Seção 8) |
| 8 | Lock / concorrência | PASS — evidência real já produzida nesta sessão (2 rejeições) |
| 9 | Deadline (endpoint local) | PASS — já validado no Gate 5.3C.1, não repetido (mecanismo inalterado) |
| 10 | JSON inválido / contrato inválido | PASS — já validado no Gate 5.3C (mecanismo inalterado) |
| 11 | Scheduler desativado | PASS (Seção 9) |
| 12 | Execução manual até a fronteira real, sem ultrapassá-la | PASS — reaproveitada a execução `hc-mtevsvnr-xfj8h17s` desta mesma sessão |
| 13 | Ausência de ação física | PASS (Seção 8) |
| 14 | Sensor determinístico preservado | PASS — `sensor.saude_sistema_status` no estado `degraded`, refletindo apenas a realidade determinística, sem qualquer influência da camada analítica |

Custo Anthropic de todos os testes acima: **US$ 0,00**.

## 12. Readiness — 24 critérios (Seção 13 do Gate)

| # | Critério | Veredito |
|---|---|---|
| 1 | Pipeline único íntegro | PASS |
| 2 | Coleta real | PASS |
| 3 | Evidence-bundle real | PASS |
| 4 | Payload real | PASS |
| 5 | Boundary MOCK→Anthropic identificado | PASS |
| 6 | Contrato Anthropic válido | PASS |
| 7 | Mecanismo de credencial seguro | PASS |
| 8 | Zero segredo em claro | PASS |
| 9 | Lock | PASS |
| 10 | Concorrência | PASS |
| 11 | Máximo uma chamada | PASS (por construção do nó único, não testado com chamada real) |
| 12 | Zero retry | PASS |
| 13 | Deadline | PASS (mecanismo validado; nó real ainda não instanciado — ver Seção 7) |
| 14 | Tratamento de erro | PASS (mecanismo genérico validado; não exercitado com o nó real — ver Seção 5) |
| 15 | Validação de contrato | PASS |
| 16 | Persistência | PASS |
| 17 | Telemetria | PASS |
| 18 | Finalize | PASS |
| 19 | Scheduler desativado | PASS |
| 20 | Ausência de ação física | PASS |
| 21 | Sensor determinístico preservado | PASS |
| 22 | Rollback definido | PASS — ver Seção 13 |
| 23 | Snapshot de recuperação válido | PASS — SHA-256 `68c15776ea37ef1cfcdc5a66645bf5ea768422c3000129463aa6f3eeaf8d7bf5` |
| 24 | Custo estimado informado | PASS — Seção 10 |

**Nenhum FAIL crítico.** Readiness para a Fase 2 (uma única chamada real,
mediante nova autorização humana explícita): **PASS**.

## 13. Rollback definido

Se a única chamada real da Fase 2 falhar, retornar erro inesperado, ou
produzir um resultado que viole o contrato: (a) o `catch` dedicado
absorve o erro e roteia para `finalize` como `failed`, liberando o lock
— sem retry; (b) a credencial real pode ser removida da instância do
subflow via `PUT /flow/gate53b_tab` cirúrgico (não `POST /flows`),
restaurando o boundary para MOCK puro; (c) o snapshot desta fase (Seção 2)
permite comparação byte-a-byte de qualquer alteração feita na Fase 2,
usando o procedimento de verificação (nunca retry automático) já
homologado no incidente do Gate 5.3B.1.

## 14. Plano exato da futura chamada real (Fase 2 — NÃO EXECUTADO)

```
BOTÃO MANUAL (input_button.saude_sistema_executar_health_check_manual)
  -> LOCK (gate53b_fn_lock_check)
  -> FSM preparing
  -> COLETA REAL (gate52a_tab, 12 entidades)
  -> EVIDENCE-BUNDLE REAL (~9973 bytes)
  -> PAYLOAD REAL (payload_analitico_real)
  -> FSM calling
  -> [NOVO] msg.requestTimeout = 300000; monta headers e body
  -> [NOVO] http request (POST https://api.anthropic.com/v1/messages,
             model: claude-sonnet-5, output_config.format = JSON Schema
             das 9 chaves, x-api-key da credencial real na instância,
             anthropic-version, content-type: application/json,
             senderr:true)
  -> [NOVO] catch dedicado (erro/timeout -> failed, zero retry)
  -> FSM processing / validating (gate53c_fn_validar_contrato,
     inalterado — já sabe validar as 9 chaves de qualquer fonte)
  -> PERSISTÊNCIA (sensor.saude_sistema_analitico_status, 9 chaves +
     telemetria real: input_tokens/output_tokens/custo_usd)
  -> FINALIZE (success ou failed; libera lock)
```

- **Modelo:** `claude-sonnet-5`.
- **Endpoint:** `https://api.anthropic.com/v1/messages`.
- **Deadline:** `300000ms` (300s), `msg.requestTimeout`, zero retry.
- **Estimativa de tokens/custo:** ordem de grandeza de `~5900` tokens de
  entrada / `~2500` de saída / `~US$ 0,04` (Seção 10) — não garantido.
- **Entidade de persistência:** `sensor.saude_sistema_analitico_status`
  (mesma de sempre; nenhuma entidade nova).
- **Rollback:** ver Seção 13.

Esta especificação não foi implementada nesta fase.

## 15. Pendência do recorder (não ampliada, não corrigida aqui)

A pendência registrada no Gate 5.3E (política de recorder dos sensores do
Health Check) permanece separada e não bloqueante. `recorder.yaml` não foi
tocado nesta fase.

## Confirmações finais

Zero chamadas Anthropic. Zero uso de `ANTHROPIC_API_KEY` real. Zero
solicitação de chave ao usuário. Zero ação física. Zero retry. Zero
alteração de flow Node-RED (`PUT`/`POST` nenhum usado — fase somente
leitura). Zero alteração de helper/sensor/automação/script. Zero restart
de HA ou Node-RED. Zero commit, zero push. Scheduler desativado durante
toda a fase.
