# Gate 5.3F — Fase 2 — Preparação da execução real (dark path)

## Status: PREPARAÇÃO VALIDADA. Nenhuma chamada Anthropic real realizada.

Esta fase implementou, de forma cirúrgica, o caminho técnico para uma
futura chamada Anthropic real — mantendo-o **dark/inalcançável** por
qualquer gatilho operacional ativo. Nenhuma chamada a `api.anthropic.com`
ocorreu em nenhum momento.

## Alterações realizadas (duas escritas cirúrgicas, sem `POST /flows`)

1. **`PUT /flow/global`** — adicionados 6 nós novos dentro do subflow
   `gate53b_subflow_executor` (nunca no template, sempre nós internos):
   `gate53b_sf_fn_router`, `gate53b_sf_fn_build_real_request`,
   `gate53b_sf_http_request_real`, `gate53b_sf_catch_real`,
   `gate53b_sf_fn_parse_real_response`, `gate53b_sf_fn_real_error`. O
   `in`/`out` do subflow foi religado para que o roteador novo decida entre
   o caminho MOCK original (inalterado) e o caminho REAL novo. Verificado
   por releitura: hash canônico `17ab5697aa2f2787e8ecf9c503eee9c605290be03c25142183d8db29ab73fcab`,
   idêntico ao pretendido.
2. **`PUT /flow/gate53b_tab`** — uma linha duplicada em dois branches de
   `gate53b_fn_finalize`: `modelo: 'mock'` → `modelo: msg.modelo_usado ||
   'mock'`. Verificado por releitura: hash canônico
   `b8f6ad3309013fdbe9536a4e6866e4aee1e4cadfe15426d906149aa3cd7d95a9`,
   idêntico ao pretendido. 58 nós, 0 IDs duplicados, 0 wires inválidos.

Nenhuma outra tab, helper, sensor, automação ou script foi tocado.

## Prova de dark path (fail-closed)

Busquei a string `real_anthropic` em todo o conteúdo vivo de
`gate53b_tab` e do subflow: aparece **0 vezes** em `gate53b_tab` (nenhum
inject de teste, nenhum prep do botão manual, nenhuma função do
scheduler jamais produz esse valor) e **exatamente 2 vezes** no subflow —
ambas dentro do próprio nó roteador novo (uma no nome/comentário, uma na
comparação `===`). É a única porta de entrada para o caminho real.

Todo ponto que atribui `mock_mode` em `gate53b_tab` foi enumerado e
inspecionado: 7 injects de teste e 4 funções (`gate53b1_fn_prep_payload`,
`gate53c_fn_prep_manual` — botão de produção —, `gate53d_fn_marcar_checkpoint`
— scheduler —, e `gate53b_fn_lock_check`, que apenas repassa o valor
recebido). Todos usam literais fixos (`'success'`, `'error'`, `'timeout'`,
`'invalid_schema'`, `'json_invalido'`, `'tipo_invalido'`) — nunca uma
variável, nunca entrada de usuário, nunca `'real_anthropic'`.

A comparação do roteador é `===` estrita contra a string exata — sem
coerção de tipo, sem `default`, sem `undefined`/`null` capaz de cair no
branch real. Qualquer valor diferente de `'real_anthropic'` cai
incondicionalmente no branch MOCK original.

**Mudança mínima futura que tornaria o caminho alcançável (identificada,
NÃO executada):** trocar o literal `'success'` por `'real_anthropic'` em
`gate53c_fn_prep_manual` (o botão manual de produção) — uma única linha,
um único ponto, deliberado e auditável.

## Auditoria dos 6 nós novos

| Nó | Tipo | Finalidade |
|---|---|---|
| `gate53b_sf_fn_router` | function | Decide MOCK (default) vs REAL (`mock_mode==='real_anthropic'`, exato) |
| `gate53b_sf_fn_build_real_request` | function | Lê a credencial (`env.get`, só aqui), monta headers/body/deadline |
| `gate53b_sf_http_request_real` | http request | POST `https://api.anthropic.com/v1/messages`, `senderr:true`, sem retry |
| `gate53b_sf_catch_real` | catch (scope dedicado) | Captura erro/timeout dessa chamada especificamente |
| `gate53b_sf_fn_parse_real_response` | function | Extrai texto (`content[].type==='text'`) e telemetria (tokens/modelo) |
| `gate53b_sf_fn_real_error` | function | Normaliza qualquer erro real para o mesmo formato do MOCK |

Requisição validada estruturalmente: endpoint correto, método POST,
`anthropic-version: 2023-06-01`, `content-type: application/json`, modelo
`claude-sonnet-5`, payload vindo de `payload_analitico_real` (coleta real),
`output_config.format` com JSON Schema das 9 chaves (`additionalProperties:
false`), `msg.requestTimeout = 300000`. Mecanismo de saída estruturada
confirmado contra a documentação oficial atual (não apenas memória
treinada) durante a Fase 1.

Zero retry confirmado por inspeção: nenhum loop, nenhum catch que
reenvia, nenhum segundo caminho para o mesmo nó HTTP. Validação de
contrato reaproveita `gate53c_fn_validar_contrato` sem duplicação — trata
MOCK e REAL identicamente. Lock: `gate53b_fn_finalize` libera
incondicionalmente (`flow.set('hc_em_andamento', false)` é a primeira
linha executável), para qualquer saída (sucesso, erro HTTP, timeout, erro
de parse, contrato inválido).

## Correção da linha do `finalize`

Antes: `modelo: 'mock'` hardcoded — mesmo numa eventual execução real,
sempre diria "mock" (bookkeeping incorreto). Depois: `modelo:
msg.modelo_usado || 'mock'`. Como `msg.modelo_usado` só é setado pelos 2
nós novos do caminho REAL, no caminho MOCK atual ele é sempre
`undefined`, preservando exatamente o comportamento anterior
(`undefined || 'mock'` → `'mock'`). Comportamento MOCK comprovadamente
inalterado por dois testes reais pós-escrita.

## Evidência de regressão zero (testes reais, não apenas inspeção)

Dois disparos reais via `input_boolean.gate53b1_teste_disparo_ha` (helper
de teste do Gate 5.3B.1, nunca o botão de produção), um antes e um depois
das duas escritas, ambos completaram normalmente em MOCK:
`hc-mtex223t-njtx0975` e `hc-mtfwelq5-xqzcnpfy` — `contrato_ok:true`,
`credencial_presente_no_mock:true` (confirmação booleana da credencial,
valor nunca lido/exposto). Lock confirmado livre (`false`) por leitura
direta do contexto de flow. Scheduler confirmado `Desativado` continuamente
(ticks a cada 15 min de 09:09 a 14:21 UTC, todos "não devido/desativado").
`sensor.saude_sistema_status` inalterado (`degraded`, realidade
determinística real, sem qualquer influência da camada analítica). Gate
5.2A (coleta) e as demais tabs não foram tocados por nenhuma das duas
escritas.

## Estimativa da futura chamada (não executada)

Payload estimado (bundle real 9973 bytes + system prompt ~2,3KB + schema
~0,7KB + overhead) ≈ 13.000–13.500 bytes, mesma ordem de grandeza do
payload homologado no Gate 5.2B (10.036 bytes). Usando a proporção
EMPÍRICA observada na única chamada real conhecida (10.036 bytes →
5.926 input tokens, ≈1,7 bytes/token — mais confiável que uma estimativa
genérica de 4 bytes/token), estima-se **≈7.800–7.900 input tokens**.
`max_tokens: 4096` é o teto configurado, não uma previsão do output real.
Preços de referência já documentados para `claude-sonnet-5` ($2/MTok
input, $10/MTok output):

- Custo de input estimado: ≈US$0,0157.
- Custo de output (cenário realista, usando o output histórico de 2.486
  tokens como referência): ≈US$0,0249.
- **Custo total estimado (realista): ≈US$0,041** — mesma ordem de
  grandeza do valor histórico (US$0,036712).
- Custo MÁXIMO teórico (se o output usasse todo o teto de 4096 tokens):
  ≈US$0,057.

Isso não é tarifa fixa nem SLA — é uma estimativa de ordem de grandeza
sobre uma única amostra histórica.

## Confirmações finais

Zero chamadas a `api.anthropic.com`. Zero chamada Anthropic real. Custo
Anthropic desta fase: US$ 0,00. Zero exposição do valor da credencial.
Zero retry automático. Scheduler Desativado durante toda a fase. Caminho
REAL deployado e estruturalmente validado, porém comprovadamente dark.
Nenhum commit, nenhum push, nenhum restart de HA/Node-RED.

## Riscos residuais

- Telemetria real (`msg.telemetria_real`: tokens/modelo/stop_reason) é
  capturada pelo novo nó de parse mas **ainda não é persistida** no
  sensor — `gate53b_fn_finalize` não a consome. Gap conhecido e
  deliberado (fora do escopo desta fase); a persistir em Gate futuro.
- As correções pós-POC do Gate 5.2B (JSON Schema nativo, regra
  evidência×inferência) seguem nunca testadas contra o comportamento real
  da Anthropic — apenas validadas offline/estruturalmente.
- Pendência do recorder (Gate 5.3E) permanece separada, não bloqueante.
