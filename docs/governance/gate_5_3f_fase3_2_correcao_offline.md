# Gate 5.3F — Fase 3.2: Correção Offline do HTTP Request e da Instrumentação

**Data:** 2026-08-30
**Escopo:** correção offline (zero chamadas Anthropic reais) do nó `http request` real e da instrumentação de captura de evidências, seguida de validação via harness HTTP 100% local (localhost/127.0.0.1, zero internet).

## 1. Causa raiz confirmada (refinamento da Fase 3.1)

A Fase 3.1 (investigação forense read-only) havia hipotetizado que `msg.headers` (contendo `x-api-key`) era bloqueado pelo mecanismo de "no override" do Node-RED. Nesta fase, a leitura do código-fonte real do nó `http request` (`@node-red/nodes`, `core/network/21-httprequest.js`, via WebFetch) refinou o diagnóstico:

- **Mecanismo confirmado:** o campo `method` do nó estava configurado como `"POST"` (valor fixo) em vez de `"use"` (que ativa a leitura dinâmica de `msg.method`). Quando `msg.method` também está setado e o `method` configurado não é `"use"`, o nó emite o aviso `common.errors.nooverride` — exatamente o aviso encontrado nos logs do add-on na Fase 3.1.
- **Headers:** o mecanismo de merge de `msg.headers` com os headers configurados é baseado em hash de deduplicação, não um bloqueio rígido — a hipótese original da Fase 3.1 sobre bloqueio de `x-api-key` foi corrigida por esta descoberta mais precisa.
- **url:** `var url = nodeUrl || msg.url` — com `url` configurado vazio, `msg.url` é corretamente usado; nenhum aviso dispara nesse caso.

## 2. Correção aplicada (offline, zero chamadas Anthropic)

### 2.1 Nó real `gate53b_sf_http_request_real` (subflow `gate53b_subflow_executor`)
- `method`: `"POST"` → **`"use"`** (única mudança de configuração no nó).

### 2.2 Instrumentação adicionada
- `gate53b_sf_fn_parse_real_response`: captura explícita de `statusCode`, headers de diagnóstico (`content-type`, `request-id`/`x-request-id`, `retry-after`, todos os `anthropic-ratelimit-*`), gating explícito de sucesso (`status < 200 || status >= 300` → `executor_erro = 'http_error_real_status_' + status`), corpo de erro sanitizado (truncado a 500 caracteres, nunca inclui a credencial).
- `gate53b_sf_fn_real_error`: log de rastreamento (`tracelog`) do erro de transporte (timeout, DNS, conexão recusada) via `global.get('gate53b_trace')`.
- Nenhum dos logs de instrumentação grava a API key ou qualquer segredo — apenas metadados de diagnóstico e trechos sanitizados do corpo de erro.

## 3. Harness de teste local (zero internet, 100% localhost)

Construído inteiramente com nós core do Node-RED (`http in`, `function`, `delay`, `http response`, `http request`, `catch`), sem dependências de terceiros. O nó de teste (`gate53f32_http_request_test`) replica exatamente a configuração corrigida do nó real (`method:"use"`, `url` vazio, `senderr:true`, `ret:"obj"`).

**Bug descoberto e corrigido durante a construção do harness:** a função de disparo (`gate53f32_fn_build_test_request`), acionada por um `server-state-changed` no `input_text` de teste, inicialmente lia `msg.data.state` — mas o output `data` desse tipo de nó é `eventData`-tipado, não `entity`-tipado, e não carrega `.state`. Corrigido para ler `msg.payload` (que é `entityState`-tipado e contém a string do estado diretamente). Adicionalmente, a URL do harness inicialmente omitia o prefixo `/endpoint/` exigido pela configuração do `httpNodeRoot` do add-on — sem esse prefixo a requisição caía fora do espaço de rotas dos nós customizados e retornava 401 do proxy nginx do add-on (não do próprio `http in`). Corrigido para `http://localhost:1880/endpoint/gate53f32-mock`, no mesmo padrão já usado com sucesso no harness de deadline do Gate 5.3C.1.

## 4. Matriz de resultados dos 8 testes

| Teste | Cenário | Status HTTP | request-id capturado | retry-after capturado | Resultado |
|---|---|---|---|---|---|
| A | 200 válido | 200 | `req_local_gate53f32_A` | — | PASS — método/headers/URL dinâmicos confirmados corretamente enviados e recebidos pelo mock (echo); corpo parseável |
| B | 400 | 400 | `req_local_gate53f32_B` | — | PASS — corpo de erro capturado e classificável |
| C | 401 | 401 | `req_local_gate53f32_C` | — | PASS |
| D | 429 + retry-after | 429 | `req_local_gate53f32_D` | `30` | PASS — retry-after capturado corretamente |
| E | 500 | 500 | `req_local_gate53f32_E` | — | PASS |
| F | timeout (deadline 2000ms, mock atrasa 6000ms) | — (abortado) | — | — | PASS — `erro_transporte:"no response from server"`, abortou em ~2.0s, sem segunda tentativa |
| G | 200 com corpo JSON inválido | 200 | — | — | PASS — `parse_error:true` detectado via try/catch, `raw_excerpt` sanitizado (200 chars), sem crash do flow |
| H | 200 com contrato incompatível | 200 | — | — | PASS (transporte) — ver limitação de escopo na seção 5 |

Zero retries observados em todos os 8 casos (uma única entrada `mock_local_recebeu_request` por teste no trace).

## 5. Limitação de escopo do harness (documentada, não corrigida nesta fase)

O harness possui sua própria função de resultado (`gate53f32_fn_test_result`), separada da função real (`gate53b_sf_fn_parse_real_response`). O harness prova que a configuração do nó `http request` (method/headers/url dinâmicos, captura de status/headers/erro) funciona corretamente — mas não exercita diretamente a lógica completa de gating 2xx/3xx nem a validação de contrato de 9 chaves (`gate53c_fn_validar_contrato`) que só roda no pipeline real. Essa lógica foi revisada por leitura de código (seção 2.2) mas não testada端-to-end automaticamente nesta fase. Risco residual: baixo, pois a lógica de gating é simples (comparação numérica de status) e já foi lida linha a linha.

## 6. Investigação da condição de corrida de reconciliação (revisitada)

A condição de corrida identificada na Fase 3.1 (deploy do flow re-arma o timer de 10s do `gate53b_startup_inject`, podendo disparar reconciliação espúria se uma execução real começar dentro dessa janela) permanece um **limite de design aceito**, mitigado operacionalmente por uma espera de ≥15-20s após qualquer deploy antes de disparar testes ou execuções. Nesta fase, essa mitigação foi aplicada consistentemente (esperas de 15-20s após os dois deploys) e nenhuma reconciliação espúria ocorreu durante os testes locais — confirmando que a mitigação funciona na prática. Não foi implementada correção estrutural (ex.: desarmar o inject de startup após reconciliação bem-sucedida), pois está fora do escopo desta fase (que é restrita à correção do HTTP Request e à instrumentação).

## 7. Estado final verificado

- `sensor.saude_sistema_status`: **intocado** (mesmo `last_changed` de antes desta sessão).
- `sensor.saude_sistema_analitico_status`: mesma `execution_id` da Fase 3 (`hc-mtfwtdi1-vw2cjpdf`), mesma contagem de execuções (12) — confirma que os testes locais desta fase nunca tocaram o pipeline real, lock ou este sensor.
- Scheduler: `Desativado`.
- Dark path: `gate53c_fn_prep_manual` permanece com `mock_mode:'success'` (inalterado, nunca habilitado nesta fase).
- Contador `anthropic_calls_this_gate`: nenhuma nova chamada real ocorreu; mecanismo de guarda permanece ativo como defesa em profundidade.
- Regressão estrutural: 5 tabs, contagens de nós inalteradas nas tabs pré-existentes (Home Office=17, Reset buttons=5, Reseta botão Everthing=3, Gate 5.2A=21), `gate53b_tab`=68 nós, 1 subflow, 0 IDs duplicados.
- Git (`/Volumes/config`): nenhuma alteração nova nesta sessão — arquivos modificados/untracked são os mesmos pré-existentes de outras frentes (CSMR); nenhum commit, nenhum push.

## 8. Princípio permanente reafirmado

A IA não é fonte da verdade operacional. Nenhuma escrita em `sensor.saude_sistema_status` ocorreu. Nenhuma ação física foi disparada. Nenhuma credencial foi impressa, logada ou reproduzida em texto (apenas presença booleana e headers de diagnóstico sanitizados). Nenhuma segunda chamada Anthropic real foi realizada ou tentada.
