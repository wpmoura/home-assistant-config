## Handoff — Health Check / Saúde do Sistema (pós-merge PR #2)

**Data deste handoff:** 2026-08-31
**Última atualização de conteúdo:** 2026-09-02 (Seções 5, 7, 8, 9.4-9.9, 10 e 11 — fechamento da seleção Haiku/Sonnet, correção de `effort`, homologação real Haiku, correção de precificação por model ID versionado, publicação determinística real, acabamentos de dashboard, auditoria Health Check → Timeline, entrada em operação normal e diagnóstico de billing/créditos Anthropic). Conteúdo anterior preservado; campos superados marcados explicitamente em vez de removidos.
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

## 5. Estado runtime atual (verificado por leitura ao vivo, `GET`-only)

**Fotografia atual (verificada nesta rodada, após as Seções 9.4-9.9):**

| Item | Valor confirmado (atual) |
|---|---|
| Scheduler | **Desativado** |
| Lock (`hc_em_andamento`) | **`false`** (livre) |
| Caminho Anthropic real | **DARK** — `mock_mode: 'success'` |
| Guard de chamadas reais (`anthropic_calls_this_gate`) | **REMOVIDO** — ratchet retirado do código (ver "Riscos residuais" da Seção 9), ausente também do runtime (`/context/flow/gate53b_tab` não retorna mais essa chave) |
| Helper de modelo analítico | `"Claude Haiku 4.5 — Econômico"` selecionado; whitelist Haiku/Sonnet operacional (Seção 9.4) |
| `sensor.saude_sistema_status` (determinístico) | `degraded` — publicação **real** (Seção 9.7), `source=claude_hamcp_manual`, não mais `gate3_test`; `red_count=0`/`yellow_count=1`/`indeterminate_count=0` |
| `sensor.saude_sistema_watchdog` | `ok` |
| `sensor.saude_sistema_analitico_status` (analítico) | `success`; `modelo=claude-haiku-4-5-20251001`; `custo_usd=0,014671` (Seções 9.5/9.6) |
| `contagem_execucoes_total` / `manual` / `scheduled` | `30` / `12` / `5` |

**Tabela original desta seção (fotografia de 2026-08-31, histórica — preservada para rastreabilidade, campos superados marcados explicitamente):**

| Item | Valor confirmado |
|---|---|
| Scheduler (`input_select.saude_sistema_health_check_frequencia`) | Desativado |
| Lock (`flow.hc_em_andamento`, contexto de flow `gate53b_tab`) | `false` (livre) |
| Caminho Anthropic real (`gate53c_fn_prep_manual`) | DARK — `mock_mode: 'success'` |
| Guard de chamadas reais (`gate53b_sf_fn_build_real_request`, subflow em `/flow/global`) | `contagemAtual >= 4` (bloqueia a 5ª chamada) — **SUPERADO, ratchet removido, ver tabela atual acima** |
| Contador histórico (`anthropic_calls_this_gate`, contexto do nó `gate53b_subflow_instance`) | 4 — **SUPERADO, removido, ver tabela atual acima** |
| Nós na tab `gate53b_tab` | 70 (68 originais + 2 do harness de teste da Fase 6.1) |
| `gate53b_fn_finalize` propaga telemetria | Sim (`msg.telemetria_real` presente na função) |
| `sensor.saude_sistema_status` (determinístico) | `degraded` — não escrito por este pipeline, valor independente — **SUPERADO, era o fixture sintético `gate3_test`, ver Seção 9.7 para a publicação real que o substituiu** |
| `sensor.saude_sistema_analitico_status` (analítico) | `success`; última transição registrada foi um disparo de teste MOCK (`origem=ha_bidirectional_test_531`, `modelo=mock`, `contrato_ok=true`) — não é a 4ª chamada real, é um teste bidirecional posterior, sem tokens/custo (esperado, MOCK) — **SUPERADO, ver tabela atual acima** |
| `contagem_execucoes_total` / `manual` / `scheduled` | 24 / 7 / 5 (cumulativo desde o início da frente; inclui execuções MOCK, testes e as 4 chamadas reais) — **SUPERADO, ver tabela atual acima** |
| `input_boolean.gate53b1_teste_disparo_ha` (helper de teste bidirecional) | `off` |
| `input_text.gate_5_3f_6_1_disparo_de_teste_local_de_telemetria_nao_e_producao` (helper de teste da Fase 6.1) | existe, último valor `E` (último caso de teste executado) — **helper de teste, não de produção**; não está versionado em nenhum YAML (é um helper de storage do HA, criado via UI/API) |

**Nenhuma divergência runtime × documentação foi encontrada** em nenhuma das duas verificações (2026-08-31 e a rodada atual).

## 6. Estado da credencial Anthropic (sem revelar segredo)

- A credencial atual está armazenada como variável de ambiente do tipo `credential` na instância do subflow `HC Executor (mock)` (Node-RED), nunca em texto plano em código, flow JSON exportável, documentação ou log.
- Foi substituída manualmente pelo usuário **2 vezes** ao longo da frente (após a Fase 4 e após a Fase 5, ambas por HTTP 401).
- A credencial atualmente configurada foi a que produziu o **HTTP 200 da Fase 6** — não há garantia de que continue válida indefinidamente (rate limits, expiração, revogação são possíveis a qualquer momento, fora do controle desta frente).
- **Nenhuma nova sessão deve tentar ler, testar, "confirmar" ou inferir o valor desta credencial.** Presença booleana (`credencial_presente`) é o único sinal já usado, e apenas dentro do MOCK local (sem contato de rede).

## 7. Telemetria de tokens/custo (Gate 5.3F.6.1)

- Extraída de `resp.usage` da resposta real da Anthropic, em `gate53b_sf_fn_parse_real_response` → `msg.telemetria_real` → propagada por `gate53b_fn_finalize` → evento `health_check_state_changed` → lida por `evt.*` em `saude_sistema_analitico.yaml` (nos estados `success`/`failed`, preservando valor anterior nos demais).
- 12 campos: `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`, `custo_input_usd`, `custo_output_usd`, `custo_cache_write_usd`, `custo_cache_read_usd`, `custo_usd`, `modelo_retornado`, `stop_reason`, `request_id_ultima_chamada`.
- Preços oficiais usados (consultados em `platform.claude.com/docs/en/about-claude/pricing`): **Claude Sonnet 5** — input US$2/MTok, output US$10/MTok, cache write (5m) US$2,50/MTok, cache read US$0,20/MTok. **Claude Haiku 4.5** — input US$1/MTok, output US$5/MTok (preços de cache do Haiku não homologados nesta frente — nunca inventados; ausência do campo produz `null`, nunca `0`). **O valor histórico US$0,036712 do Gate 5.2B nunca deve ser reutilizado como custo real de uma chamada específica.**
- Campos ausentes/não numéricos permanecem `null` — **nunca fabricados**. `custo_usd` soma apenas os componentes numéricos conhecidos quando parte dos dados está ausente (comportamento documentado, não um bug oculto).
- Validado com 8 testes locais via fixture (zero chamadas Anthropic): usage completo, cache zero, cache presente, usage ausente, output ausente, persistência, MOCK sem custo fabricado, regressão completa — todos PASS.
- ~~Ainda não validado com uma NOVA chamada real desde a correção~~ — **RESOLVIDO**: validado com uma chamada real Haiku (Seção 9.5), que revelou um segundo bug (model ID versionado não batendo com a chave da tabela de preços, causando `custo_usd=null` mesmo com tokens presentes — Seção 9.6), corrigido e re-validado com uma nova chamada real: `custo_usd` persistido = recalculado independentemente = **US$ 0,014671** (MATCH exato).

## 8. Pendências pós-merge

### A. Correções técnicas
1. Corrigir `contrato_ok` (hoje persiste como **string** `"true"`/`"false"`, não booleano nativo — risco: uma checagem Jinja ingênua trataria `"false"` como truthy).

### B. Observabilidade
2. ~~Validar futuramente a telemetria real de tokens/custo em uma nova execução real autorizada~~ — **RESOLVIDA**. Validado com chamada real Haiku pós-correção do effort (Seção 9.5); revelou e permitiu corrigir um segundo bug de precificação por model ID versionado (Seção 9.6), com validação final `custo_usd` persistido = recalculado = US$ 0,014671 (MATCH).
3. Revisar a política de `recorder` para os sensores Health Check (atualmente deliberadamente fora do `recorder.include`; reavaliar se isso continua correto).

### C. UI
4. Revisar coerência visual/temporal da página "Saúde do Sistema" entre dados históricos de homologação e a última análise real — **PARCIALMENTE RESOLVIDA**: o item específico de horário local (cards exibindo hora em UTC em vez de America/Sao_Paulo) foi corrigido via `as_local` (Seção 9.8). Não confirmado se há outros aspectos de coerência visual/temporal fora desse item específico — item mantido aberto para eventual revisão adicional.

### D. Governança/documentação
5. Atualizar o comentário "Ainda 100% MOCK... neste Gate" no bloco `input_button` de `saude_sistema_analitico.yaml` — ficou desatualizado desde o Gate 5.3F, quando esse mesmo botão passou a poder disparar chamadas reais via dark-path.
6. Revisar a ambiguidade documental entre AT-HC-01 (`sensor.saude_sistema_status`, alimentado por "Claude + HA-MCP") e o princípio "IA não é fonte da verdade operacional" — esclarecer que se trata de um mecanismo humano-supervisionado diferente do pipeline automatizado Gate 5.3B+.

### E. Infraestrutura/Git
7. Avaliar um mecanismo de versionamento/export seguro do flow Node-RED (hoje toda a lógica de FSM/lock/guard/parser vive fora do controle de versão deste repositório).

### F. Melhorias futuras
- Nenhuma additional formalmente proposta além das listadas acima; qualquer nova funcionalidade (ex.: cache prompt real, nova frequência de scheduler, dashboard novo) deve passar por um Gate explícito, autorizado por humano, com o mesmo rigor desta frente.

## 9. GATE — Seleção controlada do modelo analítico (Haiku/Sonnet) — CONCLUÍDO E HOMOLOGADO (ver Seções 9.4-9.9)

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

### 9.3 INTEGRAÇÃO REAL HAIKU/SONNET NO NODE-RED — CONCLUÍDA E AUDITADA

**Estado homologado:**
- Acesso programático Claude Code → Node-RED **restaurado** via HTTP Basic Auth com conta dedicada (não a pessoal do usuário).
- `GET /flows` autenticado retornou HTTP 200.
- Node-RED real confirmado contendo `Gate 5.3B`, `HC Executor`, subflow `gate53b_subflow_executor`.

**Alteração aplicada:**
- Mecanismo utilizado: **`PUT /flow/global`** (`GET /flow/<id>` de subflow retorna 404 neste runtime — só tabs são endereçáveis individualmente; `/flow/global` é o mecanismo oficial para configs/subflows).
- **`POST /flows` NÃO foi utilizado.**
- **Exatamente 1** `PUT /flow/global` executado, **zero retry**.

**Escopo da alteração, dentro de `gate53b_subflow_executor`:**
1. Criado: `gate53b_sf_st_modelo_analitico` (tipo `api-current-state`, mesmo padrão nativo já usado por `gate53d_st_frequencia` para o helper de frequência — não um `global.get` inventado).
2. Alterado somente: `gate53b_sf_fn_router.wires[1]` — de `gate53b_sf_fn_build_real_request` para `gate53b_sf_st_modelo_analitico`.
3. Alterado: `gate53b_sf_fn_build_real_request` — whitelist estrita:
   ```
   "Claude Haiku 4.5 — Econômico" -> "claude-haiku-4-5"
   "Claude Sonnet 5 — Avançado"   -> "claude-sonnet-5"
   qualquer outro valor -> executor_erro = 'modelo_analitico_invalido', bloqueio ANTES do HTTP POST
   ```
   Hardcode `model: 'claude-sonnet-5'` substituído por `model: modeloResolvido`.

**Preservações confirmadas:** caminho MOCK, DARK/MOCK, zero retry, lock, timeout, contrato JSON, coleta, credencial Anthropic (nunca exposta), scheduler, frequência, telemetria existente — todos intocados. HTTP request Anthropic **não foi executado** em nenhum momento. Nenhum segredo trafegou em `GET /flow/global` (campo `env` do subflow, tipo `cred`, sem `value` na leitura).

**Validação:**
- `PUT /flow/global` → HTTP 200; `GET` pós-escrita → HTTP 200.
- Diff estrutural (hash antes/depois) confirmou **somente**: 1 nó criado, 1 wire alterado, 1 Function alterada — nenhuma outra diferença.
- Whitelist validada isoladamente (Node.js local, antes do deploy): **10/10 PASS** — Haiku→`claude-haiku-4-5`, Sonnet→`claude-sonnet-5`, inválido/`unavailable`/`unknown`/ausente→`modelo_analitico_invalido`. Código implantado confirmado textualmente idêntico ao validado.
- **Zero chamada Anthropic.** `contagem_execucoes_total = 26`. `hc_em_andamento = false` (lido diretamente via `GET /context/flow/gate53b_tab`, autenticado). `scheduler = Desativado`. Helper atual = `"Claude Haiku 4.5 — Econômico"`.

**Riscos residuais / débitos técnicos (estado original desta subseção — ver resolução abaixo):**
1. ~~`anthropic_calls_this_gate` permanece `4`...~~ — **RESOLVIDO.** O ratchet foi classificado (auditoria dedicada) como mecanismo temporário de homologação da Gate 5.3F, não um controle de segurança permanente, e foi **removido** do código de `gate53b_sf_fn_build_real_request` (grep exaustivo dos 131 objetos do flow confirmou zero referência remanescente). Confirmado ausente também em runtime (`/context/flow/gate53b_tab` não retorna mais a chave `anthropic_calls_this_gate`).
2. ~~Existe segundo hardcode em `gate53b_sf_fn_parse_real_response`...~~ — **RESOLVIDO.** `msg.modelo_usado` foi consolidado em uma única atribuição (`(resp && resp.model) || msg.modeloAnaliticoRequisitado || null`, cobrindo sucesso e erro HTTP) e o mesmo padrão foi aplicado em `gate53b_sf_fn_real_error` (catch de transporte). Confirmado pela telemetria real: mesmo na chamada Haiku que falhou por HTTP 400 (Seção 9.5), o campo `modelo` registrou corretamente `claude-haiku-4-5`, nunca `claude-sonnet-5`.
3. O único `config` global do Home Assistant (`87769718.457718`) teve seu campo `_users` atualizado automaticamente pelo próprio runtime do Node-RED (registro de que o novo nó passou a referenciá-lo) — **não foi uma alteração deliberada desta atividade**, é comportamento padrão do sistema ao criar um nó que referencia esse config node; **reincidiu em todas as escritas subsequentes desta frente**, sempre com o mesmo caráter benigno. Nenhum outro campo do config mudou; nenhuma credencial exposta.

**Estado final desta subseção (SUPERADO — ver Seções 9.4-9.9 para o fechamento completo da frente):**
- Helper Haiku/Sonnet operacional. Seletor presente no dashboard. Node-RED integrado ao seletor. Haiku tecnicamente operacional no executor.
- ~~Nenhuma chamada Haiku real executada ainda.~~ **SUPERADO**: a primeira chamada real Haiku foi executada e homologada com sucesso (Seção 9.5). Pipeline foi restaurado para DARK/MOCK imediatamente após, como em toda a frente.
- Regra permanente mantida: qualquer nova chamada real continua exigindo autorização humana explícita e específica.

**Governança futura para Node-RED (regra permanente, reafirmada):**
`GET` para auditoria · `PUT /flow/<tab_id>` quando aplicável · `PUT /flow/global` somente quando necessário para subflow (tabs não endereçam subflows individualmente neste runtime) · `POST /flows` proibido para alterações normais · snapshot/hash antes · diff estrutural pós-escrita · zero retry · leitura pós-escrita · segredo nunca em log/repositório.

### 9.4 Correção de `output_config.effort` condicional por modelo — CONCLUÍDA E AUDITADA

**Causa raiz (comprovada por resposta HTTP real da Anthropic, não presumida):** `gate53b_sf_fn_build_real_request` montava `output_config: { effort: 'medium', format: {...} }` de forma **incondicional**, para qualquer modelo. Uma tentativa real de chamada Haiku (pós-integração da Seção 9.3, antes desta correção) foi rejeitada com **HTTP 400**, corpo: `{"type":"invalid_request_error","message":"This model does not support the effort parameter."}` — capturado via `global.get('gate53b_trace')`, `request_id` real `req_011Cebxy7MVKPUBaV3eTuqBv`. Zero retry; DARK restaurado imediatamente.

**Correção aplicada:**
```js
const outputConfig = { format: { type: 'json_schema', schema: SCHEMA } };
if (modeloResolvido === 'claude-sonnet-5') {
  outputConfig.effort = 'medium';
}
```
- Haiku: **não envia** `output_config.effort`.
- Sonnet: continua enviando `output_config.effort = 'medium'` (comportamento já homologado na Fase 6, preservado).
- `output_config.format`/`json_schema` preservado, incondicional, para os dois modelos.
- Escopo: **1 nó** (`gate53b_sf_fn_build_real_request`), **1 campo** (`func`). Endpoint: `PUT /flow/global`, exatamente 1, zero retry.
- Diff estrutural pós-escrita confirmou: nenhum outro nó/wire/config alterado. Zero segredo exposto.
- Validação local prévia (zero chamada externa): Haiku sem `effort`/com `format`; Sonnet com `effort=medium`/com `format`; modelo fora da whitelist bloqueado antes de qualquer montagem de request.

### 9.5 Primeira chamada real Haiku — HOMOLOGADA COM SUCESSO

Após a correção da Seção 9.4, autorizada exatamente **1** nova chamada real Haiku (protocolo de sempre: pré-voo somente-leitura, armar `mock_mode: 'real_anthropic'` cirurgicamente, disparar via botão manual, capturar evidências, restaurar DARK, zero retry).

| Campo | Valor real observado |
|---|---|
| `execution_id` | `hc-mti44lq0-pimyz4c9` |
| HTTP status | **200** |
| `request_id` | `req_011Cec1v22XEzL2HRX5aTM4i` |
| `modelo`/`modelo_retornado` | `claude-haiku-4-5-20251001` (versionado, retornado pela própria Anthropic) |
| `contrato_ok` | `true` (9 chaves validadas) |
| `duracao_segundos` | `21.125` |
| `contagem_execucoes_total` / `manual` | `29 → 30` / `11 → 12` |

Pipeline restaurado para DARK/MOCK imediatamente após a captura de evidências; `hc_em_andamento` confirmado `false`; scheduler confirmado `Desativado`; helper confirmado inalterado (`Claude Haiku 4.5 — Econômico`). Zero segunda chamada.

### 9.6 Correção de precificação por model ID versionado — CONCLUÍDA E AUDITADA

**Causa raiz (comprovada, não presumida):** a chamada da Seção 9.5 retornou `custo_usd = null` apesar de tokens presentes (`input_tokens=5709`, `output_tokens=2450` nesse teste específico). `gate53b_fn_finalize` indexava `TABELA_PRECOS_POR_MTOK[msg.modelo_usado]` diretamente — mas `msg.modelo_usado` carrega o model ID **real/versionado** retornado pela Anthropic (`claude-haiku-4-5-20251001`), que não bate com a chave curta `'claude-haiku-4-5'` da tabela. `precos` ficava `undefined`; o guard de `custoPorTokens` (que nunca fabrica custo) retornava `null` corretamente, mas o custo real não era persistido.

**Correção aplicada** (dentro de `gate53b_fn_finalize`, sem tocar `msg.modelo_usado` nem a telemetria real):
```js
const CHAVE_PRECIFICACAO_POR_MODELO_REAL = {
  'claude-haiku-4-5': 'claude-haiku-4-5',
  'claude-haiku-4-5-20251001': 'claude-haiku-4-5',
  'claude-sonnet-5': 'claude-sonnet-5'
  // whitelist explicita, sem regex/startsWith; nenhum ID de Sonnet versionado
  // foi observado ate o momento, entao nao foi adicionado por precaucao.
};
const chavePrecificacao = CHAVE_PRECIFICACAO_POR_MODELO_REAL[msg.modelo_usado];
const precos = chavePrecificacao ? TABELA_PRECOS_POR_MTOK[chavePrecificacao] : undefined;
```
- Telemetria do model ID real (`modelo`, `modelo_retornado`) **preservada integralmente** — nunca substituída pela chave curta.
- Modelo fora da whitelist continua produzindo `custo_usd: null` — nunca `US$ 0,00` fabricado.
- Escopo: **1 nó** (`gate53b_fn_finalize`), **1 campo** (`func`). Endpoint: `PUT /flow/gate53b_tab`, exatamente 1, zero retry. Diff estrutural confirmou byte-a-byte que nada mais mudou.
- **Validação real, nova chamada Haiku, pós-correção:**

| Campo | Valor |
|---|---|
| `input_tokens` / `output_tokens` | `5711` / `1792` |
| `custo_usd` persistido | `US$ 0,014671` |
| `custo_usd` recalculado independentemente ((5711/1e6×1)+(1792/1e6×5)) | `US$ 0,014671` |
| Resultado | **MATCH exato** |

### 9.7 Camada determinística — publicação real substituindo o fixture `gate3_test`

**Achado (auditoria dedicada, não relacionada ao pipeline analítico):** `sensor.saude_sistema_status` (Gate 3, AT-HC-01, Seção 2) permanecia travado desde 2026-08-25 em um payload **sintético de teste** (`source: gate3_test`, `summary` explicitamente marcado como teste), porque o mecanismo que o alimenta (evento `saude_sistema_diagnostico_publicado`, "publicado por Claude + HA-MCP") nunca havia sido operacionalizado além do teste inicial — confirmado por grep exaustivo: nenhum automation/script/Node-RED dispara esse evento automaticamente. Isso é **arquiteturalmente correto** (a camada analítica/IA nunca deveria alimentar esse sensor diretamente — princípio da Seção 1 preservado), mas o mecanismo humano-supervisionado nunca foi de fato exercido em produção.

**Ação realizada:** uma publicação real, via o mesmo evento já contratado, coletando evidências reais do runtime (uptime de núcleo/host, CPU/memória/disco, backups, conectividade, atualização pendente) e aplicando classificação determinística (pior-domínio-prevalece, sem forçar VERDE).

| Campo | Valor |
|---|---|
| `source` | `claude_hamcp_manual` (distinto de `gate3_test`) |
| `checked_at` | `2026-09-01T01:01:25-03:00` |
| `status` (naquele instante, ver nota abaixo) | `degraded` |
| `red_count` / `yellow_count` / `indeterminate_count` | `0` / `1` / `0` |
| `watchdog` | `sem_execucao → ok` |

**Nota importante:** `degraded` reflete a **classificação correspondente às evidências observadas naquele instante específico** (internet primária degradada de forma real e contínua há ~23,7h, backup 4G disponível mas sem uso ativo confirmado) — **não é um estado permanente do sistema**, apenas o resultado determinístico do payload publicado. Domínios avaliados: `home_assistant` (healthy — uptime núcleo ~192h/host ~800h sem restart, CPU/memória/disco nominais), `backups` (healthy), `conectividade` (degraded — motivo acima), `atualizacoes` (informational — update HA Core pendente há 8 dias, não tratado como degradação). Zero chamada Anthropic nesta publicação — mecanismo inteiramente determinístico/observacional.

### 9.8 Acabamentos do dashboard — CONCLUÍDOS E AUDITADOS

- **Horário local (2 cards)**: cards "Última execução concluída em" (Estado da Execução) e "Resultado da última análise" corrigidos com o filtro nativo `as_local` do Home Assistant, inserido entre `as_datetime()` e `.strftime()`. **Timestamp bruto UTC preservado sem alteração** — a conversão é aplicada **somente na apresentação**, nunca no dado persistido, e usa a timezone configurada no HA (`America/Sao_Paulo`), **sem offset `-03:00` hardcoded**. Exemplo real validado: `2026-09-01T03:34:03.390Z` (UTC, bruto) → `01/09/2026 00:34:03` (renderizado). Formato `DD/MM/YYYY HH:MM:SS` preservado.
- **Texto obsoleto do seletor de modelo**: o card que dizia *"Modelo analítico (em preparação): esta seleção ainda não está conectada ao executor real..."* foi substituído por texto factual: *"Modelo analítico: seleção conectada ao executor real do Health Check. Opções disponíveis: Claude Haiku 4.5 — Econômico / Claude Sonnet 5 — Avançado, ambas homologadas com chamada real à Anthropic."*
- Escopo: **3 cards** alterados no total (2 de horário + 1 de texto), todos em `.storage/lovelace.sistema_casa`, via `ha_config_set_dashboard`/`python_transform` protegido por `config_hash`. Diff estrutural confirmou que somente o campo `content` desses 3 cards mudou.

### 9.9 Auditoria Health Check → evento semântico → Timeline — NÃO IMPLEMENTADA (adiada para Gate dedicado)

Avaliação read-only de como o Health Check poderia publicar conclusões na Timeline/Event Feed central. **Confirmado por leitura de código, não presumido: nenhum produtor publica diretamente na Timeline** — tudo passa por `sensor.casa_evento_publicavel_v20` (`packages/motor_timeline_v20.yaml`), que hoje só aceita, para produtores externos, um contrato fechado (`packages/contrato_publicacao_timeline_v20.yaml`, V20.1O) com **exatamente 3 fontes autorizadas** (`csmr_v20_2c`, `carro_presenca`, `lavadora`) e **9 `event_code`/mensagem fixos**, sem suporte a mensagem variável.

- **Produtor semântico já existente**: `health_check_state_changed` (disparado por `gate53b_fn_finalize` → `gate53b_fire_event`) já carrega tudo que seria necessário (`estado`, `origem`, `modelo`, `contrato_ok`, `duracao_segundos`, `custo_usd`) — nenhuma alteração necessária aqui.
- **Caminho para a Timeline** exigiria estender a whitelist do contrato V20.1O já existente (novo `source`, ex. `saude_sistema`, + 1-2 novos `event_code` com mensagem fixa) em **2 arquivos centrais**: `contrato_publicacao_timeline_v20.yaml` e `motor_timeline_v20.yaml` — ambos compartilhados com o CSMR.
- **Esforço classificado: MODERADO** (extensão de padrão já existente, não mecanismo novo).
- **Recomendação registrada**: tratar como Gate dedicado e separado, por tocar arquivos centrais compartilhados com o CSMR (área historicamente protegida nesta base, ver `packages/v20_2c_protect_csmr.yaml`) — **não implementado nesta rodada**.
- **Zero alteração em `motor_timeline_v20.yaml`, `contrato_publicacao_timeline_v20.yaml` ou qualquer arquivo CSMR.** Apenas leitura.

## 10. GATE — Entrada em operação normal — CONCLUÍDO E HOMOLOGADO

**Objetivo:** eliminar a dependência de Terminal/Claude Code para cada execução — fazer o botão manual e o scheduler operarem REAL por padrão, usando o modelo selecionado no dashboard, mantendo MOCK disponível apenas como capacidade administrativa/teste, sem duplicar executor nem criar pipeline paralelo.

### 10.1 Achado da auditoria (antes desta implementação)

Confirmado por leitura direta do runtime: **tanto o botão manual (`gate53c_fn_prep_manual`) quanto o scheduler (`gate53d_fn_marcar_checkpoint`) tinham `mock_mode: 'success'` hardcoded** diretamente no literal do payload — nenhum dos dois jamais alcançava o ramo REAL do executor. A única forma histórica de obter uma execução real era uma edição cirúrgica temporária do próprio código Node-RED, revertida logo em seguida — sem nenhum controle de UI. Isso significava, na prática, que **a seleção de modelo (Haiku/Sonnet) não tinha efeito algum** em uso normal, apesar de já estar corretamente implementada (Seção 9).

### 10.2 Implementação (A — Home Assistant)

Novo helper, `packages/saude_sistema_analitico.yaml`, sibling de `gate53b1_teste_disparo_ha` sob a chave `input_boolean:` já existente:
```yaml
saude_sistema_health_check_modo_mock:
  name: "Saúde do Sistema - Health Check - Modo MOCK"
  icon: mdi:test-tube
  initial: false
```
`OFF` (padrão) = operação REAL · `ON` = MOCK (administrativo/teste). Implantado via `input_boolean.reload`; confirmado `state=off` após criação.

### 10.3 Implementação (B — Node-RED)

**Escopo mínimo, ponto único de convergência, sem duplicar executor:**
- `gate53c_fn_prep_manual` e `gate53d_fn_marcar_checkpoint`: removido o literal `mock_mode: 'success'` de seus payloads (mantêm apenas `origem`/`mock_duration_ms`/demais campos próprios).
- **2 nós novos**, inseridos entre os dois pontos de disparo e o lock (`gate53b_fn_lock_check`), reaproveitando exatamente o padrão já usado por `gate53d_st_frequencia`/`gate53b_sf_st_modelo_analitico` (não inventado):
  - `gate53_st_modo_mock_admin` (`api-current-state`, lê `input_boolean.saude_sistema_health_check_modo_mock` em `msg.modoMockSelecionado`, nunca em `msg.payload` — evita colisão).
  - `gate53_fn_aplicar_modo_mock` (function): `msg.payload.mock_mode = (msg.modoMockSelecionado === 'on') ? 'success' : 'real_anthropic'`.
- Manual e scheduled **continuam convergindo exatamente no mesmo `gate53b_fn_lock_check`** — nenhum pipeline paralelo criado.
- Escopo confirmado por diff estrutural: **2 nós existentes alterados** (`func`+`wires`) + **2 nós novos criados**, nenhum outro nó tocado. `PUT /flow/gate53b_tab`, exatamente 1, zero retry, hash pós-escrita idêntico ao proposto.

### 10.4 Implementação (dashboard)

Adicionados 2 cards ao final da seção "Health Check Analítico (IA)": tile do novo toggle + markdown explicativo inequívoco ("Desligado (OFF) = operação REAL... Ligado (ON) = modo MOCK... controle administrativo/teste — não é uso normal"). Nenhum outro card alterado.

### 10.5 Validação sem Anthropic (pré-homologação)

Simulação local do código exatamente implantado, 4 casos, zero chamada externa: toggle ON (manual e scheduled) → roteia para MOCK; toggle OFF (manual e scheduled) → roteia para REAL, parado deliberadamente antes do HTTP Anthropic (simulação, não execução).

### 10.6 Homologação MOCK (toggle ON, execução real do runtime)

Com o toggle ligado, 1 execução manual real disparada: `modelo="mock"`, `contrato_ok=true`, `custo_usd=null`, `request_id_ultima_chamada=null`, `contagem_execucoes_total` 30→31, lock liberado ao final. Confirmou que o novo gate preserva MOCK integralmente quando solicitado. Toggle devolvido a `OFF` em seguida.

### 10.7 Homologação final — execução REAL pelo caminho normal do usuário

Com o toggle já em `OFF` (sem nenhuma intervenção administrativa), 1 execução manual disparada **exclusivamente via `input_button.press`** — o mesmo mecanismo disponível ao usuário no dashboard. Zero `PUT` ao Node-RED, zero edição de `mock_mode`, zero chamada `curl` direta à Anthropic nesta homologação.

| Campo | Valor real observado |
|---|---|
| `execution_id` | `hc-mtje33ra-soqejvq9` |
| HTTP status | **200** |
| `request_id` | `req_011Cedi2Cf4nAhaDrzw3xDzR` |
| `modelo`/`modelo_retornado` | `claude-haiku-4-5-20251001` (versionado, família Haiku selecionada respeitada) |
| `contrato_ok` | `true` |
| `input_tokens` / `output_tokens` | `6100` / `2155` |
| `custo_usd` persistido | `US$ 0,016875` |
| `custo_usd` recalculado ((6100/1e6×1)+(2155/1e6×5)) | `US$ 0,016875` — **MATCH exato**, confirma zero fallback para preço Sonnet |
| `stop_reason` | `end_turn` |
| `contagem_execucoes_total` / `manual` | `31 → 32` / `13 → 14` |
| `hc_em_andamento` final | `false` |

Toggle MOCK permaneceu `OFF`, scheduler permaneceu `Desativado`, modelo selecionado permaneceu `Claude Haiku 4.5 — Econômico` — nenhum dos três precisou ser tocado para que a execução real acontecesse pelo caminho normal.

### 10.8 Estado final — HEALTH CHECK OPERACIONAL PARA USO NORMAL

- Botão manual: **REAL por padrão**.
- Scheduler: **REAL por padrão quando habilitado** (mesmo executor, mesmo ponto de convergência do manual).
- MOCK: disponível **somente** via `input_boolean.saude_sistema_health_check_modo_mock` (toggle administrativo no dashboard).
- Nenhuma dependência de Terminal/Claude Code para uso normal — modelo, botão, frequência e o novo toggle administrativo são todos controláveis pela UI do HA.
- **Pendência não resolvida nesta atividade** (fora de escopo, ver Seção 9.9): a camada determinística (`sensor.saude_sistema_status`) continua sem renovação automática a partir do pipeline analítico — o watchdog pode voltar a `sem_execucao` mesmo com o Health Check operando normalmente em produção. Candidato a Gate dedicado futuro.
- Timeline/CSMR, camada determinística, credenciais: **não tocados** nesta atividade.

## 11. GATE — Diagnóstico de billing/créditos Anthropic — CONCLUÍDO E HOMOLOGADO SEM CUSTO

**Objetivo:** antes de habilitar o scheduler, dar visibilidade operacional a falhas reais de billing/limite/autenticação/indisponibilidade da API Anthropic — preservando sempre o HTTP status e o corpo de erro originais, nunca afirmando causa não comprovada (ex.: nunca "créditos acabaram" a menos que a própria resposta permita essa afirmação).

### 11.1 Achado da auditoria

`gate53b_sf_fn_parse_real_response` já capturava `status_http`/`corpo_erro` (JSON truncado a 500 chars) em erros HTTP reais, mas **esse dado nunca chegava ao HA** — `gate53b_fn_finalize` nunca o repassava. Erros de **transporte** (timeout/DNS/conexão, capturados em `gate53b_sf_fn_real_error`) não tinham nenhum dado estruturado capturado. Nenhum mecanismo de Push desacoplado de Timeline/CSMR existia para esse propósito.

### 11.2 Classificação implementada (dado real, nunca inferido)

Nova função `classificarErroAnthropic` em `gate53b_fn_finalize`, aplicada **somente** quando `motivo` começa com `http_error_real_` (nunca em MOCK nem em falhas de pipeline como `credencial_ausente`):

| Categoria | Gatilho | Push? |
|---|---|---|
| `billing` | palavras-chave reais no `message` (credit balance/billing/payment method/insufficient credit/purchase credits) | Sim |
| `limite` | HTTP 429 ou `type=rate_limit_error` | Sim |
| `autenticacao` | HTTP 401/403 ou `type=authentication_error/permission_error` | Sim |
| `temporario` | HTTP ≥500 ou `type=api_error/overloaded_error` | Não (evita spam de instabilidade transitória) |
| `transporte` | sem HTTP status (timeout/DNS/conexão) | Não |
| `desconhecido` | nenhum dos anteriores — **nunca inventa causa** | Não |

`erro_status_http`/`erro_tipo`/`erro_mensagem` (dado bruto original, nunca descartado, mesmo se o corpo truncado em 500 chars não for JSON válido) + `erro_categoria`/`erro_diagnostico` (rótulo operacional) persistidos em `sensor.saude_sistema_analitico_status` — 5 novos atributos, `packages/saude_sistema_analitico.yaml` (mesmo padrão de `custo_usd` etc.), `template.reload`.

### 11.3 Node-RED (2 PUTs distintos)

- `gate53b_sf_fn_real_error` (subflow, `PUT /flow/global`): passa a capturar `telemetria_real` também em falha de transporte.
- `gate53b_fn_finalize` (tab, `PUT /flow/gate53b_tab`): classificação + `outputs` 1→2 (novo output dedicado a Push).
- `gate53f61_fn_build_test_msg`: casos **F-J** adicionados (mesmo harness de 5.3F.6.1, zero rede) — billing/limite/autenticação/5xx/desconhecido sintéticos, injetados direto em `gate53b_fn_finalize`.
- **1 nó novo**: `gate53_call_push_billing` (`api-call-service`, `notify.mobile_app_iphonewm`).

**Push**: dedup via `flow.get/set('gate_billing_ultima_categoria_pushed')` — só reenvia quando a categoria muda; reseta no primeiro sucesso real. Independente de Timeline/CSMR. Zero ação física.

### 11.4 Achado da própria homologação, corrigido em tempo real

O primeiro teste (caso F/billing) revelou que os 5 novos campos, embora corretamente publicados por Node-RED no evento `health_check_state_changed`, não chegavam ao sensor HA — gap idêntico ao já documentado historicamente para `custo_usd` (Seção 7/9.6). Corrigido estendendo o template YAML (5 novos blocos `evt.*` seguindo o padrão idêntico já existente) + `template.reload`; reconfirmado com sucesso no reteste.

### 11.5 Homologação (matriz completa, zero chamada Anthropic)

Todos os 6 testes via harness `gate53f61_fn_build_test_msg` (mesmo mecanismo já auditado desde 5.3F.6.1 — zero rede, injeção direta em `gate53b_fn_finalize`, ignora lock/coleta/HTTP real):

| Caso | Resultado |
|---|---|
| billing (F) | `erro_categoria=billing`, push disparado |
| limite (G) | `erro_categoria=limite`, push disparado (categoria mudou) |
| auth (H) | `erro_categoria=autenticacao`, push disparado |
| 5xx (I) | `erro_categoria=temporario`, **push não disparado** |
| desconhecido (J) | `erro_categoria=desconhecido`, sem falsa afirmação, push não disparado |
| success MOCK (A) | `custo_usd` normal (Sonnet, US$0,037), 5 campos novos `null`, dedup resetado para `null` |

Lock liberado em todos os casos (ressalva: harness bypassa o lock por design, nunca exercitado sob contenção real). Zero retry, zero segunda tentativa em cada caso. Scheduler e toggle MOCK permaneceram inalterados durante toda a homologação.

### 11.6 Acabamento visual do dashboard

Card "Estado da Execução" (`.storage/lovelace.sistema_casa`) estendido — **1 card alterado**, nenhum outro tocado. Novo bloco: categoria traduzida (billing/pagamento, limite de uso/gasto/rate limit, autenticação/autorização, indisponibilidade temporária/API, desconhecido), HTTP status e tipo Anthropic quando disponíveis, diagnóstico+ação recomendada (`erro_diagnostico`, único campo existente — não foi inventado um campo separado de "ação recomendada"), mensagem original da API preservada e legível, horário via `as_local` (reaproveita `fin_fmt` já calculado no mesmo card, sem offset hardcoded). Quando não há erro classificado e a última execução foi `success`, mostra "✅ Anthropic: operacional" — nunca fabricado quando não há evidência (`idle`/outros estados sem erro não mostram nada). Validado com o motor Jinja real do HA (`ha_eval_template`) para os 6 cenários + estado neutro, e confirmado ao vivo contra o sensor real (estado `success` → "Anthropic: operacional").

### 11.7 Estado final

Toggle MOCK `off`, scheduler `Desativado`, lock livre, modelo selecionado inalterado, Timeline/CSMR/SmallTV não tocados, zero chamada Anthropic em toda a atividade (auditoria + implementação + homologação + acabamento visual).

**Riscos/pendências residuais:**
1. Classificação `billing` depende de correspondência de texto no `message` real — se a Anthropic mudar a redação da mensagem, pode cair em `desconhecido` (seguro por design: nunca falha silenciosamente, o erro original continua sempre visível).
2. Push nunca testado sob lock realmente ocupado (limitação do harness, não da implementação).

## REGRAS PARA A PRÓXIMA SESSÃO

- **Leia este handoff primeiro**, antes de qualquer ação na frente Health Check.
- **Valide o runtime atual antes de qualquer escrita** — o estado descrito aqui é uma fotografia de 2026-08-31 (Seções 1-8) e 2026-09-01/02 (Seções 9-11); Node-RED/HA podem ter mudado desde então.
- **Não confie apenas em memória conversacional** (nem a de sessões anteriores, nem eventual resumo automático) — reconstrua a partir de `main`, runtime e Git/GitHub, nessa ordem.
- **Regra de autorização para chamada Anthropic — atualizada pela Seção 10**: durante a fase de homologação (Seções 1-9), toda chamada real exigia autorização humana explícita e específica por chamada. **A partir da Seção 10, o Health Check está em operação normal** — botão manual e scheduler (quando habilitado) fazem chamadas reais **sem exigir uma nova autorização a cada disparo**, pois essa é agora a operação padrão pretendida e homologada. Isso não dispensa autorização humana para **alterar** esse comportamento (voltar a MOCK por padrão, mudar o mecanismo, etc.) — apenas para o uso normal já homologado.
- **Não habilitar o scheduler automaticamente** — ele é `Desativado` por padrão, e essa é uma escolha deliberada até uma decisão humana explícita em contrário.
- **Não expor, imprimir, logar ou tentar ler a credencial Anthropic** em nenhuma circunstância.
- **Não executar nenhuma ação física** a partir de uma saída da camada analítica — ela produz apenas recomendações.
- **Preservar `sensor.saude_sistema_status` como soberano** — nenhuma alteração desta frente deve escrever nele.
- **Preservar o pipeline único manual/scheduled** (mesmo lock, mesma FSM) — não criar um caminho paralelo.
- **Preservar zero retry automático** em qualquer chamada real.
- **Tratar qualquer nova alteração funcional através de um Gate explícito**, com autorização humana, snapshot/hash antes e depois, e relatório honesto (inclusive de falhas).
- **Não misturar Health Check com outras frentes** que existam no working tree (CSMR/V20.2C, Recovery 4G, Gestão do Carro, Lavadora, SmallTV) — se encontrar arquivos modificados/untracked de outras frentes, documente e preserve, nunca descarte ou commit junto.
- **Nunca usar `POST /flows` integral** no Node-RED — usar `PUT /flow/<tab_id>` ou `/flow/global`, sempre com snapshot/hash antes e depois (regra permanente desde o incidente Gate 5.3B.1).
