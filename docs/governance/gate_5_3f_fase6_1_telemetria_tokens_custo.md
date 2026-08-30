# Gate 5.3F — Fase 6.1: Correção Offline da Telemetria de Tokens e Custo

**Data:** 2026-08-30
**Autorização:** correção offline/estrutural apenas. **ZERO chamadas Anthropic reais autorizadas ou realizadas nesta fase** (nem mesmo teste de credencial). Toda a validação foi feita com fixtures locais claramente marcadas como teste, injetadas diretamente em `gate53b_fn_finalize`, sem lock/coleta/HTTP real.

## 1. Causa raiz (dois pontos independentes)

A Fase 6 relatou honestamente que `input_tokens`/`output_tokens`/`custo_usd` permaneciam `null` apesar da 4ª chamada real ter sido bem-sucedida (HTTP 200). Esta fase refinou esse diagnóstico para **dois pontos de perda independentes**, ambos necessários para a lacuna observada:

**Ponto A — Node-RED (`gate53b_fn_finalize`, tab `gate53b_tab`):** `msg.telemetria_real` (calculado corretamente em `gate53b_sf_fn_parse_real_response` a partir de `resp.usage`) nunca era referenciado na construção do objeto `out` persistido no evento `health_check_state_changed`. O dado existia transitoriamente em memória, mas nunca chegava ao evento — logo nunca chegava ao HA.

**Ponto B — HA YAML (`packages/saude_sistema_analitico.yaml`, linhas 242-244):** mesmo se o Ponto A fosse corrigido, os três atributos do sensor `sensor.saude_sistema_analitico_status` estavam hardcoded para **sempre** reaproveitar `this.attributes.*` (nunca liam `evt.*`), diferente do padrão usado por todos os outros atributos pós-execução do mesmo arquivo (ex.: `modelo`, `contrato_ok`). Isso bloquearia a telemetria mesmo com o Ponto A corrigido.

Ambos os pontos foram corrigidos nesta fase.

## 2. Baseline (snapshot pré-correção)

| Artefato | Hash SHA-256 (canônico) | Nós/linhas |
|---|---|---|
| Node-RED `gate53b_tab` | `689c808188ce0050a634a608cc5ea84eaa759ad740a5ad10752a2d35a6c791a9` | 68 nós |
| `packages/saude_sistema_analitico.yaml` | `c1045e0f63b1dbbb8d7ae35fdc2956c48b27f42cfc37e459afad2ec48b2a88c3` | 377 linhas |

Confirmado imediatamente antes de qualquer edição, byte-a-byte idêntico ao estado ao vivo.

## 3. Preços oficiais usados (Claude Sonnet 5)

Consultados via `platform.claude.com/docs/en/about-claude/pricing` nesta fase (não reutilizados de memória/treino):

| Categoria | Preço por MTok |
|---|---|
| Input | US$ 2.00 |
| Output | US$ 10.00 |
| Cache write (5 min) | US$ 2.50 |
| Cache read | US$ 0.20 |

O baseline histórico do Gate 5.2B (US$ 0.036712) **não foi reutilizado** — cálculo feito exclusivamente a partir dos tokens reais e dos preços acima.

## 4. Correção — Node-RED (`gate53b_fn_finalize`)

Reescrita local (drafted, hash-verificado) e deployada via `PUT /flow/gate53b_tab`. Adiciona:

- Extração de `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` de `msg.telemetria_real`, com guarda estrita `typeof === 'number'` (nunca fabrica valor — campo ausente ou não numérico permanece `null`).
- Função `custoPorTokens(qtd, precoPorMtok)` — retorna `null` para entrada não numérica; senão calcula e arredonda a 6 casas decimais.
- `custo_usd` total: soma os componentes conhecidos tratando os ausentes como zero, e só é `null` quando **nenhum** componente é numérico (comportamento documentado — ver Seção 6, Teste E).
- `telemetriaPersistida` (objeto com os 12 campos acima + `modelo_retornado`, `stop_reason`, `request_id_ultima_chamada`) mesclado via `Object.assign` no `out` de **ambos** os ramos (`success` e `failed`), reaproveitando 100% a estrutura `telemetria_real` já existente (nenhuma lógica de extração de usage duplicada).

Nenhuma outra função do flow foi tocada. Verificado por comparação de tamanho de `func` (bytes) de `gate53b_fn_lock_check` e `gate53c_fn_router_coleta` antes/depois — idênticos.

## 5. Correção — HA YAML (`packages/saude_sistema_analitico.yaml`)

Linhas 242-244 (hardcoded `this.attributes.*`) substituídas pelo mesmo padrão `{% if estado_valido and evt.estado in ['success','failed'] %} evt.X {% else %} this.attributes.X {% endif %}` usado pelos demais campos pós-execução do arquivo. Adicionados aditivamente (sem quebrar os três nomes de atributo já existentes, para compatibilidade com a UI): `cache_creation_input_tokens`, `cache_read_input_tokens`, `custo_input_usd`, `custo_output_usd`, `custo_cache_write_usd`, `custo_cache_read_usd`, `modelo_retornado`, `stop_reason`, `request_id_ultima_chamada`. Comentário de cabeçalho do arquivo atualizado para refletir que os campos de telemetria deixaram de ser reservados. Aplicado via `template.reload` (não requer restart do HA nem do Node-RED).

## 6. Harness de teste local (zero Anthropic)

Novo helper `input_text.gate_5_3f_6_1_disparo_de_teste_local_de_telemetria_nao_e_producao` + novo par de nós Node-RED (`gate53f61_ha_trigger` → `gate53f61_fn_build_test_msg`) que constrói um `msg` sintético com `telemetria_real` FIXTURE e injeta **diretamente** em `gate53b_fn_finalize`, contornando lock/coleta/HTTP real por completo. `origem:'telemetry_test'` e `execution_id` prefixado `test-telemetria-` deixam essas execuções inequivocamente distintas de produção/MOCK no sensor e no trace.

### Resultados dos 8 testes (todos via fixture local, zero rede Anthropic)

| Teste | Entrada (tokens) | Resultado no sensor | Veredito |
|---|---|---|---|
| A — usage completo | in=6000, out=2500, cache=0/0 | input=$0.012, output=$0.025, cache=$0/$0, total=**$0.037** | PASS |
| B — cache zero | in=3000, out=1200, cache=0/0 | input=$0.006, output=$0.012, total=**$0.018** | PASS |
| C — cache presente | in=1000, out=500, cache_w=4000, cache_r=2000 | input=$0.002, output=$0.005, cache_w=$0.01, cache_r=$0.0004, total=**$0.0174** | PASS |
| D — usage ausente | `telemetria_real={}` | todos os 9 campos de tokens/custo = `null`, sem crash, `modelo_retornado`/`stop_reason` populados normalmente | PASS |
| E — output_tokens ausente | in=4000, out ausente | input_tokens=4000, custo_input_usd=$0.008, output/cache=`null`, custo_usd total=**$0.008** (soma apenas dos componentes conhecidos — ver nota abaixo) | PASS |
| F — persistência | (via Teste A) | `sensor.saude_sistema_analitico_status` refletiu os 12 campos corretamente após o evento, confirmado por leitura subsequente | PASS |
| G — MOCK permanece sem custo real | disparo via `input_boolean.gate53b1_teste_disparo_ha` | `modelo='mock'`, todos os 9 campos de telemetria = `null` | PASS |
| H — regressão completa | (ver Seção 7) | FSM/lock/coleta/scheduler/dark path/guard/sensor determinístico — todos intactos | PASS |

**Nota sobre o Teste E (comportamento documentado, não um defeito):** quando parte dos componentes é numérica e parte é ausente, `custo_usd` soma apenas os componentes conhecidos (tratando os ausentes como contribuição zero), em vez de retornar `null` para o total inteiro. Isso é uma decisão de design explícita do código (`custoTotalUsd` só é `null` quando **nenhum** componente é numérico) — não fabrica o lado ausente (que permanece `null` individualmente), mas soma o que é conhecido. Registrado aqui para transparência caso um auditor futuro espere "não calculável" para o total nesse cenário.

Todos os cálculos conferem manualmente com os preços oficiais da Seção 3 (verificação: `tokens / 1_000_000 * preço_por_MTok`).

## 7. Regressão (Teste H, detalhado)

| Item | Resultado |
|---|---|
| Guard `gate53b_sf_fn_build_real_request` (>= 4) | Intacto — verificado via `/flow/global`, byte idêntico, não tocado nesta fase |
| Contador `anthropic_calls_this_gate` (contexto de nó) | **4** (inalterado — zero chamadas reais nesta fase) |
| Dark path (`gate53c_fn_prep_manual`) | Intacto — `mock_mode:'success'` (DARK) |
| Lock (`flow.hc_em_andamento`) | `false` (livre) |
| Scheduler (`input_select.saude_sistema_health_check_frequencia`) | `Desativado` |
| `sensor.saude_sistema_status` (AT-HC-01, determinístico) | Intocado — estado `degraded`, não escrito por este flow |
| Reconciliação pós-deploy | `startup_limpo` (sem `interrupted` espúrio) |
| Contagem de nós do tab | 68 → 70 (+2: novo trigger + nova função de teste) |
| IDs de nós pré-existentes | 100% preservados (conjunto completo comparado) |
| Tamanho de `func` de nós não relacionados (`lock_check`, `router_coleta`) | Idêntico antes/depois |

## 8. Persistência

Confirma-se: **apenas** os campos estruturados definidos (12 campos de telemetria + os já existentes do contrato de 9 chaves) são persistidos — nunca a resposta íntegra da Anthropic. Nenhuma credencial foi impressa, logada ou exposta em nenhum momento desta fase.

## 9. Compatibilidade de UI

Os três nomes de atributo já existentes (`input_tokens`, `output_tokens`, `custo_usd`) foram preservados sem alteração de nome/formato — qualquer card de dashboard pré-existente que os referencie continua funcionando sem alteração. Os 9 novos atributos são aditivos.

## 10. Git

`git status --short` confirma: `packages/saude_sistema_analitico.yaml` já era untracked (nunca commitado nesta frente) e permanece untracked com o conteúdo corrigido; este arquivo de governança é o único novo untracked criado por esta fase. Arquivos de outra frente (CSMR, `packages/csmr_dispatcher_integracao_v20_2c.yaml` etc.) permanecem intocados por qualquer ação desta fase. `git diff --check` sem apontamentos. Hash pós-correção do YAML: `7940c40df1795442714e9f337ff7bf5809c4fc7ba106b46229a8d70c978a29b4`.

## 11. Riscos residuais

- O comportamento de soma parcial de `custo_usd` quando só parte dos componentes é numérica (Seção 6, Teste E) é uma escolha de design que pode surpreender um leitor esperando "tudo ou nada"; documentado aqui para referência futura.
- O preço de cache write usado é o de 5 minutos (TTL mais comum); a integração ainda não usa `cache_control` nas chamadas reais, então os campos de cache permanecerão `0`/`null` na prática até/se `prompt caching` for adotado — o mecanismo já está pronto para essa mudança futura.
- Nenhuma mudança nesta fase afeta o guard de chamadas reais (`>= 4`); uma 5ª chamada real continua exigindo nova autorização humana explícita (bump `>=4→>=5`).

## 12. Princípio permanente reafirmado

A IA não é fonte da verdade operacional. Nenhuma chamada Anthropic real foi realizada, tentada ou sequer validada quanto à credencial nesta fase. `sensor.saude_sistema_status` permanece intocado. Toda a telemetria de custo é derivada de tokens numéricos reais quando disponíveis — nunca fabricada quando ausentes.
