# Gate 5.3F — Fase 4: Segunda Execução Real Controlada Após Correção da Causa Raiz

**Data:** 2026-08-30
**Autorização:** exatamente 1 chamada Anthropic real nesta fase (chamada real nº 2; total histórico do Gate = 2), origem exclusivamente manual, zero retry, zero segunda tentativa.

## 1. Preflight (leitura, antes de qualquer habilitação)

| Item | Resultado |
|---|---|
| Scheduler = Desativado | PASS |
| Lock livre (`hc_em_andamento`) | PASS (`false`) |
| Nenhuma execução em andamento | PASS (`sensor.saude_sistema_analitico_status` = `failed`, estado terminal) |
| Pipeline operacional = MOCK | PASS (`gate53c_fn_prep_manual` com `mock_mode:'success'`) |
| Caminho Anthropic = DARK | PASS (inalcançável sem o toggle) |
| Credencial presente (booleano, sem ler valor) | Confirmado behavioralmente durante a própria execução (`credencial_presente_no_mock:true`) |
| Zero mecanismo de retry | PASS (revisão de código, `senderr:true`, sem loop) |
| Instrumentação da Fase 3.2 presente | PASS (`gate53b_sf_fn_parse_real_response` com captura de status/headers/erro) |
| Sensor determinístico preservado | PASS (`sensor.saude_sistema_status` com mesmo `last_changed` de antes da sessão) |
| Contador histórico de chamadas reais = 1 (antes desta fase) | PASS (confirmado via `/context/flow/gate53b_subflow_instance/anthropic_calls_this_gate` = `1`) |

## 2. Alteração mínima necessária identificada no preflight

O guard permanente de defesa em profundidade (`gate53b_sf_fn_build_real_request`) bloqueia com `contagemAtual >= 1` — desenhado na Fase 3 para permitir no máximo 1 chamada real *para sempre*. Esta Fase autoriza explicitamente uma segunda chamada (total histórico = 2), portanto o guard precisava ser ajustado para refletir esse novo teto autorizado.

**Alteração aplicada (menor alteração possível):** `contagemAtual >= 1` → `contagemAtual >= 2`, no subflow `gate53b_subflow_executor`, nó `gate53b_sf_fn_build_real_request`. Nenhuma outra linha do guard foi alterada — o mecanismo de incremento (`flow.set('anthropic_calls_this_gate', contagemAtual + 1)`) permanece intacto, agora bloqueando corretamente qualquer terceira chamada.

*Nota de transparência:* durante a primeira tentativa de deploy desse patch foi introduzido acidentalmente um `;` duplo inofensivo (`}; ;`) por erro de transcrição manual do JSON; detectado via verificação pós-deploy e corrigido imediatamente com um segundo patch, ambos com verificação de hash antes e depois. Nenhum impacto funcional (JavaScript trata `; ;` como um statement vazio subsequente).

## 3. Habilitação temporária

Único toggle: `gate53c_fn_prep_manual` (`gate53b_tab`) — `mock_mode: 'success'` → `mock_mode: 'real_anthropic'`. Nenhuma outra alteração: scheduler, coleta, helpers, dashboard, contrato, persistência determinística, frequência, lógica de retry e credencial permaneceram intocados.

Aguardados ≥15-20s após cada deploy (guard threshold + habilitação), confirmando reconciliação `startup_limpo` (não `interrupted`) antes de prosseguir, mitigação já validada na Fase 3.1/3.2.

## 4. Execução

Disparo único via `input_button.saude_sistema_executar_health_check_manual` (serviço `press`), origem `manual`. Cadeia observada no trace:

`lock_check_entry` → `accepted` (`execution_id: hc-mtfz4wx5-a0qgbbqh`) → `preparing` → `router_coleta` → `coleta_real_concluida` (12 entidades, 10290 bytes) → `calling` → `processing` → `real_http_response` → `real_http_error_body` → `validating` → `contract_skip_erro_executor` → `failed`.

Nenhum segundo trigger. Nenhum retry (uma única entrada de request no trace).

## 5. Evidência HTTP capturada (sanitizada)

| Campo | Valor |
|---|---|
| Timestamp início | 2026-08-30T15:38:26.344Z |
| Timestamp término | 2026-08-30T15:38:28.996Z |
| Duração | 2.65s |
| execution_id | `hc-mtfz4wx5-a0qgbbqh` |
| Status HTTP | **401** |
| request-id | `req_011CeZBZaafyTbgzzUvg5aXd` (formato real da Anthropic — confirma que a requisição efetivamente alcançou os servidores da API desta vez, diferente da Fase 3) |
| content-type | `application/json` |
| Corpo de erro (sanitizado) | `{"type":"authentication_error","message":"invalid x-api-key"}` |
| usage / tokens | não retornado (erro de autenticação, sem processamento) |
| modelo retornado | não aplicável (erro antes do processamento do modelo) |

Nunca registrados: valor da API key, header `x-api-key`, `Authorization`, ou qualquer segredo.

## 6. Classificação HTTP

401 = falha de requisição (autenticação). Não avança como diagnóstico analítico válido — `msg.executor_erro = 'http_error_real_status_401'` corretamente impediu qualquer avaliação de contrato sobre uma resposta de erro.

## 7. Contrato analítico

**NÃO APLICÁVEL** — HTTP não foi sucesso (2xx), portanto a validação de contrato de 9 chaves foi corretamente pulada (`contract_skip_erro_executor`), sem tentativa de interpretar o corpo de erro como diagnóstico.

## 8. Evidência × Inferência

Não aplicável — nenhuma resposta de diagnóstico foi produzida pela IA (falha de autenticação antes de qualquer geração de conteúdo). Nenhum overreach possível.

## 9. Custo real

**NÃO CALCULÁVEL** — nenhum `usage` foi retornado pela API (erro 401 ocorre antes do processamento de tokens). Não presumido como US$ 0,00, conforme exigido pela autorização desta fase.

## 10. Causa provável da falha (observação, não correção nesta fase)

O corpo de erro `"invalid x-api-key"` indica que a credencial atualmente configurada na variável de ambiente `anthropic_api_key` do subflow (colada manualmente pelo usuário em fase anterior deste Gate) não é aceita pela API Anthropic — possivelmente por estar incorreta, incompleta, expirada, revogada, ou por corresponder a uma organização/projeto sem acesso ao endpoint. Esta é uma **observação factual baseada na evidência capturada**, não uma inferência de causa definitiva. Conforme a Seção 13 desta Fase, nenhuma tentativa de diagnóstico adicional contra a Anthropic (curl de confirmação, teste de credencial, nova chamada) foi realizada — a correção da credencial, se necessária, é uma decisão e ação exclusivamente humana, fora do escopo desta fase.

## 11. Restauração pós-execução (imediata, independente do resultado)

- Caminho Anthropic real: revertido para **DARK** (`gate53c_fn_prep_manual` de volta a `mock_mode:'success'`, hash de reversão `689c808...791a9` idêntico byte-a-byte ao estado pré-Fase-4).
- Pipeline operacional: **MOCK**.
- Scheduler: mantido **Desativado** (nunca alterado).
- Guard de chamadas reais: mantido em `>= 2` (reflete corretamente o teto histórico autorizado; não revertido para `>= 1`, pois isso classificaria retroativamente a Fase 4 como "excedente" quando na verdade foi explicitamente autorizada).
- Lock: **livre** (`hc_em_andamento: false`).
- Contador histórico de chamadas reais: **2** (confirmado via leitura de contexto).

## 12. Validação pós-execução

| Item | Resultado |
|---|---|
| Chamadas autorizadas nesta Fase | 1 |
| Chamadas realizadas nesta Fase | 1 |
| Total histórico do Gate | 2 |
| Retry | Zero |
| Scheduler | Desativado |
| Caminho Anthropic | DARK |
| Pipeline | MOCK |
| Lock | Livre |
| `sensor.saude_sistema_status` | Preservado (intocado) |
| Ação física | Zero |
| Execução automática | Zero |
| Regressão estrutural | Nenhuma (68 nós em `gate53b_tab`, 7 nós no subflow, 5 tabs, 0 IDs duplicados) |

## 13. Riscos residuais

- A credencial Anthropic configurada não é válida (`invalid x-api-key`), portanto qualquer futura execução real (fora do escopo desta Fase, requer nova autorização) continuará falhando até que a credencial seja corrigida por ação humana direta na configuração do Node-RED.
- O guard agora permite até 2 chamadas reais históricas; qualquer futura autorização de uma terceira chamada precisará de um ajuste explícito e documentado do mesmo guard (`>= 2` → `>= 3`), seguindo o mesmo padrão de menor-alteração-possível usado nesta fase.
- A condição de corrida de reconciliação (Fase 3.1/3.2) permanece um limite de design aceito, mitigado pela espera de ≥15-20s pós-deploy — aplicada consistentemente nesta fase sem incidentes.

## 14. Princípio permanente reafirmado

A IA não é fonte da verdade operacional. `sensor.saude_sistema_status` permanece intocado. Nenhuma ação física foi disparada. Nenhuma credencial foi impressa, logada ou reproduzida em texto. Nenhuma terceira chamada Anthropic real foi realizada ou tentada.
