## Handoff — Health Check / Saúde do Sistema (pós-merge PR #2)

**Data deste handoff:** 2026-08-31
**Autor:** sessão Claude Code anterior (compactada múltiplas vezes) — este documento existe exatamente para que uma NOVA sessão, sem esse contexto, não precise reconstruir nada por memória.

**Regra de ouro deste documento:** tudo aqui foi verificado por leitura direta (Git/GitHub, runtime HA/Node-RED, arquivos versionados) no momento em que foi escrito. Runtime muda; releia antes de agir. Ver "REGRAS PARA A PRÓXIMA SESSÃO" no final.

---

## 1. Objetivo da frente

Implementar um Health Check Operacional diário/sob-demanda para a Central Operacional Home Assistant, com uma camada de evidências determinísticas + indicadores + payload homologado, orquestrada por Node-RED, complementada por um diagnóstico opcional gerado pela API Anthropic (Claude), com persistência controlada e apresentação segura.

**Princípio permanente, inegociável, reafirmado em toda a frente:**

> **A IA NÃO É FONTE DA VERDADE OPERACIONAL.**

Consequências diretas desse princípio:
- A camada analítica (IA) nunca escreve em `sensor.saude_sistema_status` (o sensor determinístico permanece soberano).
- Nenhuma ação física é disparada automaticamente a partir de uma saída da IA — apenas recomendações para avaliação humana.
- Zero retry automático em qualquer chamada Anthropic real.
- Nenhuma credencial é impressa, logada, hasheada ou exposta em qualquer artefato (código, doc, log, sensor).

## 2. Arquitetura homologada

```
SOLICITAÇÃO (manual: input_button | scheduled: input_select frequência)
   → LOCK (flow.hc_em_andamento — mutex único, serializa manual × scheduled)
      → FSM: idle → preparing → calling → processing → validating → success|failed|interrupted
         → COLETA REAL DE EVIDÊNCIAS (Gate 5.2A, via link call)
            → EVIDENCE-BUNDLE (JSON estruturado de entidades HA)
               → PAYLOAD ANALÍTICO (execution_id + origem + coleta)
                  → BOUNDARY MOCK/ANTHROPIC (dark-path toggle em `gate53c_fn_prep_manual`;
                    fora do dark-path = MOCK sempre, sem nenhuma chamada de rede)
                     → [se dark-path habilitado] guard de contagem (>= N bloqueia) → HTTP real Anthropic
                     → VALIDAÇÃO DE CONTRATO (9 chaves pós-POC, schema Gate 5.2B)
                        → PERSISTÊNCIA ANALÍTICA (sensor.saude_sistema_analitico_status,
                          apenas campos estruturados, nunca a resposta íntegra)
                           → TELEMETRIA (tokens/custo, extraídos de resp.usage, nunca fabricados)
                              → FINALIZE (libera o lock, publica evento health_check_state_changed)
```

Dois sensores, dois namespaces, deliberadamente separados:
- **`sensor.saude_sistema_status`** (AT-HC-01, `packages/saude_sistema.yaml`) — determinístico, alimentado pelo evento `saude_sistema_diagnostico_publicado`. **Nota de precisão:** esse evento é documentado como `publicado_por: 'Claude + HA-MCP'` — ou seja, um mecanismo **diferente e mais antigo** (Gate 3), humano-supervisionado (uma sessão Claude/HA-MCP publica após avaliação, não um pipeline automatizado). O pipeline Gate 5.3B–5.3F.6.1 (auditado e mergeado no PR #2) **nunca** dispara esse evento nem escreve nesse sensor — confirmado estruturalmente (nenhum nó do flow Gate 5.3B+ referencia `saude_sistema_diagnostico_publicado`).
- **`sensor.saude_sistema_analitico_status`** (AT-HC-02/03, `packages/saude_sistema_analitico.yaml`) — camada analítica complementar, alimentada por 4 eventos internos do Node-RED (`health_check_requested`, `health_check_rejected`, `health_check_state_changed`, `health_check_scheduled_checkpoint`). É este sensor, e só este, que reflete o pipeline Gate 5.3B–5.3F.6.1.

**Achado estrutural importante:** o código Node-RED que implementa a FSM/lock/guard/parser (`gate53b_fn_finalize`, `gate53b_sf_fn_parse_real_response`, `gate53b_sf_fn_build_real_request`) **não está versionado neste repositório Git** — vive exclusivamente no runtime do add-on Node-RED (armazenamento interno do add-on, fora de `/Volumes/config`). Qualquer verificação desse código exige leitura direta e segura (`GET`) da instância Node-RED, não apenas do Git.

## 3. Gates concluídos (cronologia consolidada)

| Gate/Fase | Objetivo | Resultado | Chamada Anthropic real? | Decisão final |
|---|---|---|---|---|
| AT-HC-01 (Gate 3) | Persistência do Health Check determinístico | Concluído | Não (evento externo) | Sensor `sensor.saude_sistema_status` em produção |
| AT-HC-02 | Homologação da 1ª chamada real (Gate 5.2B, executor isolado) | Concluído | Sim — 1 chamada (fora deste PR, herança histórica; custo de referência US$ 0,036712, **nunca reutilizado** como valor real de chamadas futuras) | Viabilidade técnica confirmada |
| Gate 5.3B/5.3B.1 | Fundação operacional: FSM, lock, reconciliação pós-restart, credencial nativa | Concluído | Não (100% MOCK) | Base homologada |
| Incidente Node-RED `POST /flows` | Um `POST /flows` truncou o corpo da requisição, excluindo temporariamente um nó | Recuperado sem perda de dados | Não | Regra permanente adotada: **nunca mais `POST /flows` integral**; usar `PUT /flow/<tab_id>` ou `/flow/global` (menor *blast radius*), sempre com snapshot/hash antes e depois |
| Gate 5.3C/5.3C.1 | Execução manual controlada + validação de contrato (9 chaves) + coleta real integrada | Concluído | Não (MOCK) | Contrato pós-POC homologado |
| Gate 5.3D | Scheduler com frequência configurável, checkpoint anti-duplicidade | Concluído | Não | Scheduler operacional, padrão `Desativado` |
| Gate 5.3E | Interface operacional / homologação de runtime | Concluído | Não | UI validada humanamente |
| Gate 5.3F Fase 2 | Preparação para chamada real (deadline nativo, headers, guard inicial `>=1`) | Concluído | Não | Pipeline pronto para 1ª chamada real controlada |
| Gate 5.3F Fase 3 | **1ª chamada real autorizada** | Executada | **Sim (1ª)** — falhou com `resposta_real_sem_texto` (sem HTTP code capturado; gap de instrumentação reconhecido honestamente) | Zero retry; gap de instrumentação registrado para fase futura |
| Gate 5.3F Fase 3.2 | Correção offline do HTTP request e da instrumentação (sem nova chamada) | Concluído | Não | Instrumentação corrigida para próxima fase |
| Gate 5.3F Fase 4 | **2ª chamada real autorizada** (guard `>=1→>=2`) | Executada | **Sim (2ª)** — **HTTP 401** `invalid x-api-key` | Falha de credencial classificada honestamente; zero segunda tentativa |
| Gate 5.3F Fase 4.1 | Investigação forense offline do 401 + handoff de correção manual da credencial ao usuário | Concluído | Não (zero contato Anthropic) | Usuário substituiu a credencial manualmente na UI do Node-RED |
| Gate 5.3F Fase 5 | **3ª chamada real autorizada** (guard `>=2→>=3`), nova credencial | Executada | **Sim (3ª)** — **HTTP 401** novamente (nova credencial também não aceita) | Usuário substituiu a credencial uma 2ª vez |
| Gate 5.3F Fase 6 | **4ª chamada real autorizada** (guard `>=3→>=4`), nova credencial | Executada | **Sim (4ª)** — **HTTP 200**, contrato analítico **PASS** (9 chaves validadas) | Sucesso; lacuna honesta reportada: telemetria de tokens/custo permaneceu `null` apesar do sucesso |
| Gate 5.3F.6.1 | Correção offline da lacuna de telemetria (zero chamadas Anthropic) | Concluído | **Não (zero chamadas nesta fase)** | 2 pontos de perda corrigidos (Node-RED `gate53b_fn_finalize` + YAML `saude_sistema_analitico.yaml`); validado com 8 testes locais via fixture |
| **PR #2** | Consolidar e mergear a frente Health Check em `main` | **MERGED** | Não (zero chamada durante todo o processo de PR/merge) | `main` agora contém os 18 arquivos da frente |

**Total histórico de chamadas Anthropic reais: 4** (1 falha sem HTTP capturado, 2 falhas HTTP 401, 1 sucesso HTTP 200). Guard atual bloqueia qualquer 5ª chamada sem nova alteração/autorização explícita.

## 4. PR #2 e merge (estado Git/GitHub confirmado nesta sessão)

- **PR:** #2 — "Health Check — Gate 5.3: orquestração, scheduler, UI e homologação runtime"
- **Estado:** `MERGED`
- **Merge commit:** `827ac7ebc7896cc9e2d9047ccea19cb812b732c0` (pais: `16ae78089102eed020fd799099c91589598b3bbd` = `main` anterior, `ab518079c05aedf9ed1037587e906ebbf246ff8d` = head do PR)
- **`main` local confirmado no merge commit** (`git log -1 origin/main` = `827ac7e`).
- **18 arquivos incorporados**, confirmados individualmente presentes em `origin/main`:
  - `docs/governance/at_hc_01_persistencia_health_check.md`
  - `docs/governance/at_hc_02_homologacao_camada_analitica_ia.md`
  - `docs/governance/gate_5_3c_1_coleta_real_integrada.md`
  - `docs/governance/gate_5_3c_execucao_manual_mock.md`
  - `docs/governance/gate_5_3d_scheduler_frequencia.md`
  - `docs/governance/gate_5_3e_interface_operacional.md`
  - `docs/governance/gate_5_3f_fase2_preparacao_real.md`
  - `docs/governance/gate_5_3f_fase3_2_correcao_offline.md`
  - `docs/governance/gate_5_3f_fase3_execucao_real.md`
  - `docs/governance/gate_5_3f_fase4_1_correcao_credencial.md`
  - `docs/governance/gate_5_3f_fase4_segunda_execucao_real.md`
  - `docs/governance/gate_5_3f_fase5_terceira_execucao_real.md`
  - `docs/governance/gate_5_3f_fase6_1_telemetria_tokens_custo.md`
  - `docs/governance/gate_5_3f_fase6_quarta_execucao_real.md`
  - `docs/governance/gate_5_3f_homologacao_runtime_fase1.md`
  - `docs/governance/incidente_node_red_post_flows_gate53b1.md`
  - `packages/saude_sistema.yaml`
  - `packages/saude_sistema_analitico.yaml`
- **Escopo exclusivo Health Check confirmado:** diff `16ae780..827ac7e` = exatamente esses 18 arquivos, zero código de CSMR/V20.2C/Recovery 4G/Gestão do Carro/Lavadora/SmallTV.
- **Nenhum segredo versionado** (varredura de padrões de credencial, limpa).
- **Branch `feature/health-check-gate-5-3`** permanece publicada no remoto como histórico do PR já mergeado — não precisa ser tocada; pode ser deletada quando o mantenedor desejar (não obrigatório).
- **CI:** **não configurado neste repositório** (nenhum `.github/workflows/`) — isso significa que não há verificação automatizada; qualquer validação de qualidade depende de auditoria manual/humana, como a que gerou este PR.
- **Working tree de outras frentes:** no checkout usado por esta sessão (`/Volumes/config`, branch `feature/v20-2c-contextual-automations`), existem alterações não commitadas de **outra frente (CSMR/V20.2C)**: `CHANGELOG.md`, `automations.yaml`, `docs/ARCHITECTURE.md`, `docs/governance/gates_v20.md`, `docs/v20_2c/c1_saida_de_casa.md`, `docs/v20_2c/plano_tecnico_csmr.md`, `packages/csmr_dispatcher_integracao_v20_2c.yaml`, `packages/v20_2c_contextual_automations.yaml`, `packages/v20_2c_protect_csmr.yaml` (todos `M`), mais `packages/smalltv_publicacao_v20.yaml` (`??`, untracked). **Isso é pré-existente, de outra frente, e não deve ser tocado, commitado ou descartado por quem trabalhar em Health Check.**

## 5. Estado runtime atual (verificado por leitura ao vivo, `GET`-only, nesta sessão)

| Item | Valor confirmado |
|---|---|
| Scheduler (`input_select.saude_sistema_health_check_frequencia`) | **Desativado** |
| Lock (`flow.hc_em_andamento`, contexto de flow `gate53b_tab`) | **`false`** (livre) |
| Caminho Anthropic real (`gate53c_fn_prep_manual`) | **DARK** — `mock_mode: 'success'` |
| Guard de chamadas reais (`gate53b_sf_fn_build_real_request`, subflow em `/flow/global`) | **`contagemAtual >= 4`** (bloqueia a 5ª chamada) |
| Contador histórico (`anthropic_calls_this_gate`, contexto do nó `gate53b_subflow_instance`) | **4** |
| Nós na tab `gate53b_tab` | **70** (68 originais + 2 do harness de teste da Fase 6.1) |
| `gate53b_fn_finalize` propaga telemetria | **Sim** (`msg.telemetria_real` presente na função) |
| `sensor.saude_sistema_status` (determinístico) | `degraded` — não escrito por este pipeline, valor independente |
| `sensor.saude_sistema_analitico_status` (analítico) | `success`; última transição registrada foi um disparo de teste MOCK (`origem=ha_bidirectional_test_531`, `modelo=mock`, `contrato_ok=true`) — **não é a 4ª chamada real**, é um teste bidirecional posterior, sem tokens/custo (esperado, MOCK) |
| `contagem_execucoes_total` / `manual` / `scheduled` | 24 / 7 / 5 (cumulativo desde o início da frente; inclui execuções MOCK, testes e as 4 chamadas reais) |
| `input_boolean.gate53b1_teste_disparo_ha` (helper de teste bidirecional) | `off` |
| `input_text.gate_5_3f_6_1_disparo_de_teste_local_de_telemetria_nao_e_producao` (helper de teste da Fase 6.1) | existe, último valor `E` (último caso de teste executado) — **helper de teste, não de produção**; não está versionado em nenhum YAML (é um helper de storage do HA, criado via UI/API) |

**Nenhuma divergência runtime × documentação foi encontrada** nesta verificação.

## 6. Estado da credencial Anthropic (sem revelar segredo)

- A credencial atual está armazenada como variável de ambiente do tipo `credential` na instância do subflow `HC Executor (mock)` (Node-RED), nunca em texto plano em código, flow JSON exportável, documentação ou log.
- Foi substituída manualmente pelo usuário **2 vezes** ao longo da frente (após a Fase 4 e após a Fase 5, ambas por HTTP 401).
- A credencial atualmente configurada foi a que produziu o **HTTP 200 da Fase 6** — não há garantia de que continue válida indefinidamente (rate limits, expiração, revogação são possíveis a qualquer momento, fora do controle desta frente).
- **Nenhuma nova sessão deve tentar ler, testar, "confirmar" ou inferir o valor desta credencial.** Presença booleana (`credencial_presente`) é o único sinal já usado, e apenas dentro do MOCK local (sem contato de rede).

## 7. Telemetria de tokens/custo (Gate 5.3F.6.1)

- Extraída de `resp.usage` da resposta real da Anthropic, em `gate53b_sf_fn_parse_real_response` → `msg.telemetria_real` → propagada por `gate53b_fn_finalize` → evento `health_check_state_changed` → lida por `evt.*` em `saude_sistema_analitico.yaml` (nos estados `success`/`failed`, preservando valor anterior nos demais).
- 12 campos: `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`, `custo_input_usd`, `custo_output_usd`, `custo_cache_write_usd`, `custo_cache_read_usd`, `custo_usd`, `modelo_retornado`, `stop_reason`, `request_id_ultima_chamada`.
- Preços oficiais usados (Claude Sonnet 5, consultados em `platform.claude.com/docs/en/about-claude/pricing` nesta fase): input US$2/MTok, output US$10/MTok, cache write (5m) US$2,50/MTok, cache read US$0,20/MTok. **O valor histórico US$0,036712 do Gate 5.2B nunca deve ser reutilizado como custo real de uma chamada específica.**
- Campos ausentes/não numéricos permanecem `null` — **nunca fabricados**. `custo_usd` soma apenas os componentes numéricos conhecidos quando parte dos dados está ausente (comportamento documentado, não um bug oculto).
- Validado com 8 testes locais via fixture (zero chamadas Anthropic): usage completo, cache zero, cache presente, usage ausente, output ausente, persistência, MOCK sem custo fabricado, regressão completa — todos PASS.
- **Ainda não validado com uma NOVA chamada real** desde a correção (a correção foi testada apenas com fixtures locais). Isso é uma pendência explícita (ver Seção 8).

## 8. Pendências pós-merge

### A. Correções técnicas
1. Corrigir `contrato_ok` (hoje persiste como **string** `"true"`/`"false"`, não booleano nativo — risco: uma checagem Jinja ingênua trataria `"false"` como truthy).

### B. Observabilidade
2. Validar futuramente a telemetria real de tokens/custo em uma **nova execução real autorizada** (a correção da Fase 6.1 só foi testada com fixtures locais, nunca com uma chamada Anthropic real pós-correção).
3. Revisar a política de `recorder` para os sensores Health Check (atualmente deliberadamente fora do `recorder.include`; reavaliar se isso continua correto).

### C. UI
4. Revisar coerência visual/temporal da página "Saúde do Sistema" entre dados históricos de homologação e a última análise real.

### D. Governança/documentação
5. Atualizar o comentário "Ainda 100% MOCK... neste Gate" no bloco `input_button` de `saude_sistema_analitico.yaml` — ficou desatualizado desde o Gate 5.3F, quando esse mesmo botão passou a poder disparar chamadas reais via dark-path.
6. Revisar a ambiguidade documental entre AT-HC-01 (`sensor.saude_sistema_status`, alimentado por "Claude + HA-MCP") e o princípio "IA não é fonte da verdade operacional" — esclarecer que se trata de um mecanismo humano-supervisionado diferente do pipeline automatizado Gate 5.3B+.

### E. Infraestrutura/Git
7. Avaliar um mecanismo de versionamento/export seguro do flow Node-RED (hoje toda a lógica de FSM/lock/guard/parser vive fora do controle de versão deste repositório).

### F. Melhorias futuras
- Nenhuma additional formalmente proposta além das listadas acima; qualquer nova funcionalidade (ex.: cache prompt real, nova frequência de scheduler, dashboard novo) deve passar por um Gate explícito, autorizado por humano, com o mesmo rigor desta frente.

## REGRAS PARA A PRÓXIMA SESSÃO

- **Leia este handoff primeiro**, antes de qualquer ação na frente Health Check.
- **Valide o runtime atual antes de qualquer escrita** — o estado descrito aqui é uma fotografia de 2026-08-31; Node-RED/HA podem ter mudado desde então.
- **Não confie apenas em memória conversacional** (nem a de sessões anteriores, nem eventual resumo automático) — reconstrua a partir de `main`, runtime e Git/GitHub, nessa ordem.
- **Nenhuma chamada Anthropic sem autorização humana explícita e específica** (formato "GATE X — AUTORIZAÇÃO EXPLÍCITA", como em todas as fases anteriores desta frente) — mesmo para "apenas testar a credencial".
- **Não habilitar o scheduler automaticamente** — ele é `Desativado` por padrão, e essa é uma escolha deliberada até uma decisão humana explícita em contrário.
- **Não expor, imprimir, logar ou tentar ler a credencial Anthropic** em nenhuma circunstância.
- **Não executar nenhuma ação física** a partir de uma saída da camada analítica — ela produz apenas recomendações.
- **Preservar `sensor.saude_sistema_status` como soberano** — nenhuma alteração desta frente deve escrever nele.
- **Preservar o pipeline único manual/scheduled** (mesmo lock, mesma FSM) — não criar um caminho paralelo.
- **Preservar zero retry automático** em qualquer chamada real.
- **Tratar qualquer nova alteração funcional através de um Gate explícito**, com autorização humana, snapshot/hash antes e depois, e relatório honesto (inclusive de falhas).
- **Não misturar Health Check com outras frentes** que existam no working tree (CSMR/V20.2C, Recovery 4G, Gestão do Carro, Lavadora, SmallTV) — se encontrar arquivos modificados/untracked de outras frentes, documente e preserve, nunca descarte ou commit junto.
- **Nunca usar `POST /flows` integral** no Node-RED — usar `PUT /flow/<tab_id>` ou `/flow/global`, sempre com snapshot/hash antes e depois (regra permanente desde o incidente Gate 5.3B.1).
