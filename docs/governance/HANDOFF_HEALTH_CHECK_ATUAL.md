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

## 9. GATE — Seleção controlada do modelo analítico (Haiku/Sonnet) — EM ANDAMENTO, NÃO CONCLUÍDO

**Nota:** as Seções 1-8 acima predatam esta atividade e alguns eventos posteriores já ocorridos nesta linha do tempo (fix de `contrato_ok`, a 5ª chamada real Sonnet, correções de coerência do dashboard) — não foram reescritas aqui; esta seção documenta exclusivamente o estado desta atividade específica.

**Objetivo:** permitir seleção controlada entre Claude Haiku 4.5 (econômico, padrão) e Claude Sonnet 5 (avançado) para o Health Check Analítico, com whitelist estrita no Node-RED.

**Estado atual — INCOMPLETO:**
- Commit local `d236c8b` (branch `gate/modelo-analitico-select`, worktree `config-gate-modelo-analitico`, cortado de `origin/main`) contém **somente** a criação do helper `input_select.saude_sistema_health_check_modelo_analitico` (YAML, `packages/saude_sistema_analitico.yaml`) — opções "Claude Haiku 4.5 — Econômico" / "Claude Sonnet 5 — Avançado", inicial = Haiku.
- **Helper ainda NÃO implantado no runtime** (nem PR aberto, nem merge, nem `template.reload`/deploy executado).
- **Seletor ainda NÃO presente no dashboard** — nenhum card foi adicionado (evitado deliberadamente, para não referenciar uma entidade inexistente em produção).
- **Node-RED ainda usa `claude-sonnet-5` hardcoded** — nenhuma alteração foi feita no Node-RED nesta atividade (sem ferramenta de acesso ao Node-RED nesta sessão); o pipeline real não lê o novo helper.
- **Portanto: Haiku ainda NÃO está operacional.** Selecionar a opção Haiku no helper (quando implantado) não terá nenhum efeito real até a integração Node-RED ser feita.
- Correção dinâmica do card "Telemetria e Uso" (remoção da frase hardcoded "todas via motor MOCK até o momento", agora derivada de `sensor.saude_sistema_analitico_status.modelo`) **já foi aplicada em `.storage/lovelace.sistema_casa`** — essa parte está concluída e ao vivo.
- Scheduler permanece `Desativado`. Pipeline permanece DARK/MOCK. **Zero nova chamada Anthropic realizada nesta atividade. Nenhuma chamada Haiku foi realizada.**
- **Próxima chamada real continua proibida sem autorização humana explícita e específica** — mesma regra permanente desta frente.

**Objetivo pendente (nesta ordem):**
1. ~~Proteger/versionar o helper pelo fluxo de governança~~ — **CONCLUÍDO**: PR #5 mergeado em `main` (merge commit `984519fbf4f0d5e4f5b9bb10ce368cb222d72ac6`).
2. ~~Implantar o helper no runtime~~ — **CONCLUÍDO**: `input_select.saude_sistema_health_check_modelo_analitico` implantado via sync do arquivo + `input_select.reload` (não `template.reload` — reload específico do domínio, sem restart).
3. ~~Adicionar o seletor ao dashboard~~ — **CONCLUÍDO**: tile presente em `.storage/lovelace.sistema_casa`, seção "Health Check Analítico (IA)", com legenda explícita "(em preparação)" para não induzir o usuário a acreditar que já está operacional.
4. Alterar Node-RED (`gate53c_fn_prep_manual`/executor) para ler o helper e aplicar whitelist estrita — **AINDA NÃO FEITO**:
   - "Claude Haiku 4.5 — Econômico" → `claude-haiku-4-5`
   - "Claude Sonnet 5 — Avançado" → `claude-sonnet-5`
   - Qualquer valor fora da whitelist = falha segura (`modelo_analitico_invalido`), zero chamada.
5. Validar ambos os caminhos **sem chamada externa** (MOCK/fixture/estrutural) — **PARCIALMENTE FEITO**: ver Seção 9.1 abaixo.
6. **Somente depois**, solicitar autorização humana explícita para **UMA** chamada real Haiku (mesmo protocolo rigoroso já usado para a 5ª chamada Sonnet: pré-voo, guard temporário, zero retry, restauração imediata para DARK/MOCK).

### 9.1 Atualização — tentativa de integração Node-RED (validação apenas como lógica isolada; Node-RED real NÃO alterado)

**Estado runtime confirmado nesta atualização:**
- Helper **implantado e homologado**: `input_select.saude_sistema_health_check_modelo_analitico`, `state` atual = **"Claude Haiku 4.5 — Econômico"**, `options` = `["Claude Haiku 4.5 — Econômico", "Claude Sonnet 5 — Avançado"]`.
- Seletor **presente no dashboard**, junto aos controles de "Frequência automática"/"Executar Health Check agora", com legenda "(em preparação)".
- **Whitelist Haiku/Sonnet validada apenas como lógica isolada** (Node.js local, zero rede, zero Node-RED real, zero Anthropic): 10/10 casos PASS — Haiku→`claude-haiku-4-5`, Sonnet→`claude-sonnet-5`, valores inválidos (id de API bruto, outro provedor, payload de injeção) e estados HA não-operacionais (`unavailable`/`unknown`/vazio/`undefined`/`null`) → `modelo_analitico_invalido`, zero POST simulado. **Isso prova apenas a lógica proposta, não o comportamento do Node-RED real.**
- **Node-RED real NÃO foi alterado** — sem ferramenta de acesso ao Node-RED nesta sessão (mesma limitação já registrada nos gates de pré-voo da 5ª chamada). Código proposto (função `resolverModeloAnalitico`) entregue como sugestão para aplicação futura por quem tiver acesso direto.
- **Executor real continua hardcoded em `claude-sonnet-5`.**
- **Portanto: Haiku AINDA NÃO está operacional** — a seleção no helper não tem nenhum efeito sobre qual modelo é efetivamente chamado.
- **Nenhuma chamada Haiku foi realizada. Nenhuma nova chamada Sonnet foi realizada.**
- Pipeline permanece **DARK/MOCK**. Scheduler permanece **Desativado**.
- `anthropic_calls_this_gate` **continua sendo um débito técnico separado** (escopo/persistência do contador) — deliberadamente não tocado nem usado para validar esta atividade, para não misturar frentes.
- **Próxima atividade:** aplicar e homologar a integração diretamente no Node-RED (Fase 1 de leitura/documentação real do código + Fase 2 de implementação + Deploy controlado, por sessão/pessoa com acesso).
- **Qualquer chamada real continua proibida sem autorização humana explícita e específica.**

### 9.2 Diagnóstico de acesso Claude Code → Node-RED (somente leitura, nenhuma credencial testada)

**Objetivo desta subseção:** registrar por que esta e sessões futuras do Claude Code não conseguem operar diretamente nos flows do Node-RED, e o que seria necessário para restaurar esse acesso — sem executar nenhuma correção agora.

**Diagnóstico:**
- Node-RED é alcançável pela LAN em `192.168.50.237:1880` — **conectividade de rede não é o problema** (correção de um diagnóstico anterior desta mesma frente, que havia testado endereços incorretos: IP interno Docker `172.30.32.1` e hostname mDNS `.local.hass.io`, nenhum dos dois alcançável a partir de uma sessão externa).
- A tentativa somente-leitura à Admin API (`GET /`, `GET /auth/token`, `GET /flows`, sem enviar nenhuma credencial) retorna **HTTP 401** em todas as rotas testadas, com corpo/cabeçalhos idênticos: `Server: nginx`, `WWW-Authenticate: Basic realm="Home Assistant Authentication"`.
- **Portanto: o bloqueio não é de conectividade — existe uma barreira de autenticação antes do acesso à Admin API.**
- Nesta instalação do Claude Code **NÃO existe MCP dedicado ao Node-RED configurado**. `~/.claude.json` contém apenas o servidor `ha-mcp`; nenhuma entrada para Node-RED existe (global ou por projeto).
- O acesso histórico utilizado nesta frente está documentado (`docs/governance/incidente_node_red_post_flows_gate53b1.md`) como chamadas HTTP diretas à Admin API do Node-RED, incluindo `GET /flows`, `POST /flows`, `PUT /flow/<id>` — mas **esse documento não registra a credencial/mecanismo usado para atravessar a autenticação**.
- **Não foi possível comprovar nesta investigação** qual credencial ou mecanismo concreto era utilizado historicamente.
- **Não foi encontrada** nenhuma credencial reutilizável, variável de ambiente, script auxiliar ou conta de automação Node-RED documentada em nenhum arquivo lido (`.claude/settings.local.json`, docs de governança).
- O diagnóstico atual aponta para **autenticação Basic via proxy associado ao ambiente Home Assistant/Supervisor** (realm explicitamente identificado como "Home Assistant Authentication", resposta servida por `nginx`, não pelo próprio Node-RED) — consistente com o campo `protected: true` já observado na configuração do add-on. **Essa é uma hipótese fundamentada, não uma confirmação** — a autenticação efetiva ainda precisa ser comprovada por teste controlado antes de ser tratada como mecanismo operacional definitivo.
- **Nenhuma senha, token, `credential_secret` ou outro segredo foi registrado nesta investigação ou neste handoff.**

**Risco histórico importante — reafirmado:** existe incidente documentado nesta frente (`incidente_node_red_post_flows_gate53b1.md`, Gate 5.3B.1) envolvendo `POST /flows`, que provocou substituição integral do conjunto de flows e perda de 58 nós (recuperado sem perda residual). Por isso, **se o acesso automatizado ao Node-RED for restaurado no futuro**:
- `POST /flows` fica **proibido** para alterações normais;
- preferir `PUT /flow/<id>` para alteração cirúrgica (atualiza uma tab por vez, preserva as demais);
- snapshot/export antes de qualquer escrita;
- hash SHA-256 antes/depois;
- zero retry automático;
- verificação pós-escrita (`GET /flows` de conferência estrutural completa);
- escopo mínimo por alteração;
- Deploy controlado, nunca automático/silencioso.

**Próximo passo técnico (nesta ordem):**
1. Comprovar de forma controlada o mecanismo de autenticação real da Admin API (não presumir a hipótese acima como definitiva).
2. Preferencialmente utilizar uma identidade dedicada para automação, evitando reutilizar credencial pessoal do usuário.
3. Nunca persistir senha em texto claro em repositório ou `settings.local.json`.
4. Somente depois, restaurar ao Claude Code acesso controlado à Admin API.
5. Só então realizar a integração real Haiku/Sonnet no Node-RED (item 4 pendente da Seção 9, acima).

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
