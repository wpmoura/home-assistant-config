# AT-HC-02 — Homologação da Camada Analítica por IA (Health Check Operacional)

Data: 2026-08-29
Status: **HOMOLOGADA**
Classificação: governança de domínio (Saúde do Sistema / Health Check Operacional), frente independente
Autoridade: subordinada a `at_hc_01_persistencia_health_check.md` (camada de persistência, Gate 3) e aos Gates 5.1, 5.1B, 5.1C, 5.2A, 5.2A.1, 5.2A.2 e 5.2B do Health Check Operacional
Escopo deste documento: **exclusivamente a camada analítica por IA (chamada Anthropic)** — discovery de arquitetura de execução, POC de coleta determinística em Node-RED, e a primeira chamada real controlada à Anthropic Messages API, com suas correções pós-POC.

## 1. Objetivo

Encerrar formalmente, com homologação técnica, o ciclo de trabalho que levou da persistência do Health Check (AT-HC-01) até a primeira chamada real e controlada a um modelo Anthropic como camada complementar de interpretação, validando viabilidade técnica, segurança operacional da credencial, contrato de resposta e ausência de qualquer automação/retry/scheduler.

Esta homologação **não amplia escopo funcional** além do que foi efetivamente implementado e testado (uma coleta determinística em Node-RED, uma única chamada real à API, uma correção offline pós-POC validada apenas por testes locais).

## 2. Arquitetura homologada

```
evidências reais (Home Assistant)
      ↓
indicadores determinísticos (Node-RED, coleta + validação de domínio)
      ↓
payload homologado (contract_version evidence-bundle-0.2, 10.036 bytes)
      ↓
Anthropic API (camada complementar de interpretação)
      ↓
diagnóstico complementar
```

**Princípio arquitetural a preservar em todos os Gates futuros: a IA NÃO é fonte da verdade operacional.** A Anthropic não altera `sensor.saude_sistema_status`, não altera helpers/sensores/automações, não comanda ações físicas, não modifica arbitrariamente o estado operacional, e uma inferência do modelo nunca deve ser tratada como evidência. O payload de entrada é sempre determinístico e auditável antes de qualquer envio; a resposta é sempre um insumo complementar para decisão humana.

## 3. Escopo entregue e homologado

1. **Discovery de arquitetura de execução** (Gates 5.1, 5.1B) — comparação Mac+launchd vs. host HA sempre ligado; conclusão: Node-RED (já instalado, `state:"started"`, `boot:"auto"`) no host HA é arquiteturalmente superior.
2. **Validação read-only do Node-RED** (Gate 5.1C) — confirmado instalado, rodando, auto-start, acessível via `ha_manage_app`, tecnicamente capaz.
3. **POC de coleta determinística** (Gate 5.2A, 5.2A.1, 5.2A.2) — flow `gate52a_tab` no Node-RED: coleta real de 12 entidades HA → normalização por whitelist por entidade → validação de cobertura de domínio → payload `evidence-bundle-0.2`. Execução manual única, `valido:true, erros:[]`, 5 domínios declarados explicitamente como `indeterminate` (nunca omitidos).
4. **Primeira chamada real controlada à Anthropic Messages API** (Gate 5.2B) — auditoria pré-chamada, preparação de executor isolado, verificação de credencial, uma única chamada real, telemetria completa, cálculo de custo com preços oficiais.
5. **Identificação de 3 ressalvas da POC e correção offline pós-POC** — JSON estruturado nativo, disciplina evidência×inferência reforçada, deadline total real — validadas por 8 testes offline, sem nova chamada real.

## 4. Estado homologado — evidência quantitativa (chamada real)

| Campo | Valor |
|---|---|
| Timestamp início | `2026-08-28T20:08:31-0300` |
| Timestamp fim | `2026-08-28T20:12:51-0300` |
| Latência real | `259.717 s` |
| Endpoint | `https://api.anthropic.com/v1/messages` |
| Modelo solicitado / retornado | `claude-sonnet-5` / `claude-sonnet-5` |
| HTTP status | `200` |
| `request_id` | `req_011CeVzZc1wzHsEMEY5nT5Hx` |
| `input_tokens` | `5926` |
| `output_tokens` | `2486` |
| `total_tokens` | `8412` |
| `cache_creation_input_tokens` / `cache_read_input_tokens` | `0` / `0` |
| `stop_reason` | `end_turn` |
| Erro | `null` |
| Payload homologado (enviado) | `10.036 bytes` |
| Request / Response | `13.799 bytes` / `6.335 bytes` |

**Custo real desta chamada:** input `US$ 0.011852` + output `US$ 0.024860` = **`US$ 0.036712`**.

**Incidente histórico separado (lição aprendida, não é custo do Health Check):** um consumo acidental de aproximadamente `US$ 3,44` ocorreu anteriormente por `ANTHROPIC_API_KEY` (chave dedicada `health-check-poc`) ter sido exportada no ambiente antes de iniciar o Claude Code, fazendo o próprio Claude Code consumi-la sem relação com este Health Check. Lição registrada: nunca exportar uma chave de API dedicada a uma frente específica no ambiente geral onde o Claude Code roda — usar sempre entrada interativa isolada por processo, como implementado no executor deste Gate.

**Projeções econômicas — puramente matemáticas (lineares, baseadas em UMA amostra), não representam frequência adotada:**

| Cenário | Projeção mensal/total |
|---|---|
| 1 execução/dia (30 dias) | `US$ 1.10136/mês` |
| 1 execução/semana | `≈ US$ 0.158963/mês` |
| 2 execuções/semana | `≈ US$ 0.318293/mês` |
| 3 execuções/semana | `≈ US$ 0.477256/mês` |
| 52 execuções | `US$ 1.909024` |
| 365 execuções | `US$ 13.39988` |

O custo real por chamada pode variar conforme tamanho do payload, tamanho da resposta, modelo utilizado, mudanças de preço da Anthropic, uso de cache, e evoluções futuras do contrato de resposta — estas projeções não são um SLA nem uma previsão orçamentária comprometida.

## 5. Achados da POC real e correções pós-POC (offline, sem nova chamada)

### 5.1 JSON puro — FAIL de contrato
O `system prompt` pediu JSON puro textualmente, mas a resposta real veio envolvida em fence Markdown (` ```json ... ``` `). **Conclusão: instrução textual sozinha não é contrato suficientemente forte para automação.**
**Correção:** uso de `output_config.format` (JSON Schema nativo da Anthropic Messages API — mecanismo oficial, confirmado documentalmente para `claude-sonnet-5`), mais validação local defensiva que trata qualquer fence remanescente como violação de contrato (nunca corrigida automaticamente, nunca gera segunda chamada).

### 5.2 Disciplina evidência × inferência — PASS COM RESSALVA
A resposta real associou `binary_sensor.backup_4g_operacional = on` a `binary_sensor.casa_internet_degradada_v20 = on` e inferiu possível uso ativo de contingência — mais forte do que a evidência permitia. **Regra arquitetural registrada: disponibilidade/capacidade/operacionalidade não implica uso ativo.** `operacional` ≠ `em uso`; `disponível` ≠ `acionado`; simultaneidade ≠ causalidade; backup operacional ≠ failover ativo; ausência de evidência de uso deve gerar `dados_insuficientes`, nunca uma inferência de uso.
**Correção:** regra 7 explícita e genérica adicionada ao `system prompt` (não restrita ao caso do 4G).

### 5.3 Timeout — NÃO COMPROVADO como deadline total
O executor usava `urllib.request.urlopen(timeout=60)`, mas a chamada real levou `259.717s` sem erro de timeout. **Explicação técnica:** o parâmetro `timeout` do `urlopen()` limita cada operação de socket bloqueante individual (conexão, cada `recv()`), não a duração total da operação — comportamento documentado do Python, não um deadline de parede.
**Correção:** deadline total real via `signal.SIGALRM` (stdlib, POSIX/macOS), independente do timeout de socket, com valor de referência `300s` (margem sobre o pior caso observado de `259.717s`). **`300s` é política operacional baseada na evidência atual — não é garantia de SLA da Anthropic.** Timeout nunca gera retry: a única tentativa é abortada, o erro é registrado, e a execução termina.

## 6. Testes offline pós-POC (8/8 PASS)

| # | Teste | Resultado |
|---|---|---|
| 1 | JSON válido | PASS |
| 2 | Markdown fence | PASS — rejeitado, sem limpeza automática |
| 3 | Campo obrigatório ausente | PASS — rejeitado |
| 4 | Propriedade inesperada | PASS — rejeitada |
| 5 | Inferência indevida de uso 4G | PASS — detectada pela heurística, sem falso positivo em resposta conforme |
| 6 | Resposta dentro do deadline | PASS — 1 tentativa, sucesso |
| 7 | Deadline excedido | PASS — 1 tentativa, aborta, zero retry |
| 8 | Erro HTTP 500 mock | PASS — 1 tentativa, erro registrado, zero retry |

Nenhum teste acessou `api.anthropic.com` — os testes de rede (6–8) usaram exclusivamente servidor HTTP local em `127.0.0.1` (porta efêmera).

## 7. Evidência dos executores (não incorporados ao repositório nesta etapa)

Os dois executores usados neste Gate são artefatos **temporários, fora de `/Volumes/config` e fora de controle de versão** (diretório `mktemp -d` local à máquina de trabalho). Registro apenas do hash/path como evidência documental — **não foram copiados para o repositório nesta etapa**, por decisão deliberada de não ampliar escopo sem autorização explícita:

| Artefato | SHA-256 | Papel |
|---|---|---|
| `executor_gate52b.py` | `662197bb0f125898288b65ef35e11a39281a0cb1c249f37f7891eceb04d54126` | Executor real usado na única chamada válida (Python 3, stdlib apenas, 1 tentativa HTTP, zero retry/fallback, chave via `getpass`, nunca persistida) |
| `executor_gate52b_pos_poc.py` | `97be4ebf489a25db3944f7da75e85483263120eec7a7c1d0ece488c594cc735c` | Versão corrigida (JSON Schema nativo, regra evidência×inferência reforçada, deadline total via `SIGALRM`) — validada apenas offline, nunca chamada contra a Anthropic real |

**Recomendação (não executada nesta etapa):** avaliar, em Gate futuro, se convém preservar esses scripts como artefato permanente versionado (ex.: `tools/health_check_ia/`) ou manter apenas este registro documental como evidência suficiente. Essa decisão pertence ao usuário e não foi tomada aqui.

## 8. O que esta homologação NÃO significa

- Scheduler **não** está implementado.
- Node-RED definitivo (produção) **não** está implementado — apenas a POC manual do Gate 5.2A.2 existe.
- Execução recorrente **não** está implementada.
- Botão manual **não** está implementado.
- Frequência configurável **não** está implementada.
- Diagnóstico **não** foi persistido no Home Assistant (nenhuma escrita em `sensor.saude_sistema_status` ocorreu a partir da resposta da IA).
- Dashboard **não** foi alterado.
- Notificações **não** foram implementadas.
- A versão pós-POC (`executor_gate52b_pos_poc.py`) **não** foi chamada novamente contra a Anthropic — suas correções são validadas apenas offline.

Esses itens pertencem a um Gate posterior.

## 9. Riscos residuais

1. As 3 correções pós-POC (JSON estruturado, regra evidência×inferência, deadline total) permanecem **não validadas contra o comportamento real do modelo** — apenas testadas offline/localmente.
2. `output_config.format` com schema novo incorre em custo de compilação na primeira chamada (documentado oficialmente pela Anthropic) — pode adicionar latência à próxima chamada real com este schema.
3. A heurística de disciplina evidência×inferência é um validador local por palavras-chave, não uma garantia formal — cobre o padrão observado na POC real, não é prova de ausência de outras formas de overreach.
4. `DEADLINE_TOTAL_SEGUNDOS=300` é estimativa com margem sobre uma única amostra de latência real.
5. Os dois executores continuam como artefatos não versionados fora do repositório (ver seção 7).

## 10. Próximo Gate (apenas declarado, não iniciado)

**GATE 5.3 — ORQUESTRAÇÃO OPERACIONAL DO HEALTH CHECK**

Objetivo conceitual: tirar a execução da dependência operacional do Mac e integrar o mecanismo ao fluxo controlado da Central Operacional. Deverá tratar, no mínimo: execução pelo ambiente Home Assistant/Node-RED; execução manual sob demanda; frequência configurável pelo usuário; controle de consumo de tokens; armazenamento seguro da credencial; persistência controlada da última análise; telemetria; tratamento de indisponibilidade da API; ausência de retry automático inicialmente; separação rigorosa entre diagnóstico analítico e estado determinístico; eventual histórico; eventual apresentação no dashboard. **Nada disso foi implementado nesta etapa.**

## 11. Registro de encerramento

**Decisão:** HOMOLOGAR.

**Status final: AT-HC-02 — HOMOLOGADA.** A camada analítica por IA do Health Check Operacional permanece encerrada nesta entrega, com o estado descrito nas seções 3 e 4 como sua entrega vigente. Nenhuma nova evolução funcional é aberta por este documento — o próximo passo é o Gate 5.3, declarado na seção 10 e ainda não autorizado a iniciar.
