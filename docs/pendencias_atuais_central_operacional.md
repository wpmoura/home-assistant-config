# Pendências Atuais - Central Operacional Home Assistant

Data do levantamento original: 2026-05-16
Última reconciliação: 2026-09-16

Este arquivo é a fila canônica de pendências concretas da Central Operacional. Roadmaps declaram situação e prioridade; `docs/technical_debt/backlog_tecnico.md` registra dívidas estruturais; Gates registram critérios e evidências. O conteúdo original de maio permanece abaixo como snapshot histórico não saneado.

## Fila operacional atual — reconciliada em 2026-09-16

Classificações permitidas: `ABERTA`, `RESOLVIDA`, `SUPERADA` e `NÃO COMPROVADA`.

### Pendências abertas ou não comprovadas

| ID | Frente | Roadmap | Pendência | Tipo | Bloqueante | Classificação | Próxima ação / evidência necessária |
| --- | --- | --- | --- | --- | --- | --- | --- |
| PEND-004 | Gestão do Carro — zonas | SOC | Registrar entrada e saída nas zonas conhecidas | funcional | Não para a baseline AT-GC; sim para concluir o domínio | ABERTA | Abrir Gate próprio para entidade observada, contrato, GPS, idempotência, sobreposição e mudanças cadastrais |
| PEND-009 | V20.2 shadow | SOC | Concluir ou reclassificar testes ainda pendentes da Fase 1A | teste | Sim para promoção geral; não para manter shadow | ABERTA | Usar `docs/execucao_testes_reais_v20_2_fase_1a.md`; preservar os 7 OK, 1 parcial e 2 bloqueados já registrados |
| PEND-012 | V20.1C/decommission | SOC | Definir e autorizar lotes pequenos de desativação com rollback | decisão | Sim para qualquer remoção | ABERTA | Quarentena C1 integralmente qualificada e executada (Lotes 1-5, 18 objetos removidos). Triagem de C2/C3/Bloqueado (48 itens): Lote 6 removeu 7 itens (5 scripts mortos + 2 automações desabilitadas); Lote 7 removeu as 2 automações de moeda restantes (`Lab - Moeda - Dólar abaixo de...` id `1662142325333`, `Lab - Moeda - Euro abaixo de...` id `1662650425226`) — trigger comum `sensor.yahoofinance_usdbrl_x` confirmado `unavailable`/`restored: true` (fonte de dado estruturalmente morta, não apenas condição não ocorrida); achado lateral não corrigido: a automação "Euro" nunca monitorou Euro de fato, usava o mesmo sensor de USD/BRL. Total C2/C3/Bloqueado removido: 9 objetos. `LAB - Notificar Início do Home Assistant - WM` preservada por NO-GO individual (Lote 6). C3 (10 itens) e Bloqueado (13 itens) permanecem classificados PRESERVAR/NÃO ACIONÁVEIS — nenhuma dívida de decommission real neles. **Único saldo remanescente:** `Homeoffice - Ligar Regua USB` e `Home office - Desliga Regua USB`, desabilitadas via `automation.turn_off` desde `2026-09-13T23:11` (Lote 4), em observação — janela mínima documental de 7 dias encerra-se em aproximadamente `2026-09-20T23:11`; nenhuma decisão antecipada. PEND-012 permanece ABERTA exclusivamente por essa observação |
| PEND-019 | Sensor de vazamento (Lab) — achado lateral do discovery da PEND-011 | — (fora dos roadmaps V20; dispositivo físico isolado por decisão de Wilson) | Ausência de alerta ativo está comprovada; confiabilidade do sensor `binary_sensor.sensor_vazmento_agua_water_leak` e a causa do estado `on` observado permanecem não comprovadas | observação/diagnóstico físico | Não bloqueia a PEND-011 nem nenhuma outra frente; porém a validação da confiabilidade do sensor é pré-condição para ativar qualquer alerta futuro | ABERTA | Achado comprovado: ausência de qualquer alerta ativo — o blueprint `Alerta_Vazamento_com_Led.yaml` existe em `blueprints/automation/wmoura/` mas nunca foi instanciado (`use_blueprint`); nenhuma automação, script ou notificação reage à entidade, que só aparece como indicador visual passivo em 3 dashboards (`dashboard-baterias`, `dashboard-testes`, `lovelace`). Achado NÃO comprovado: a confiabilidade do próprio sensor e a causa do estado `on` observado (`binary_sensor.sensor_vazmento_agua_water_leak = on` no momento do discovery/Gate P2.0) permanecem em aberto — não há evidência que separe detecção real de água de instabilidade de conexão/reconexão do dispositivo. Wilson relatou que a bateria física do sensor está arriada, divergente do que o HA reporta (`voltage: 3025 mV`, `battery: 100%`); troca prevista para a semana de 2026-09-23, ainda não realizada — os valores abaixo são registrados apenas como baseline anterior à troca, não como validação de funcionamento. Gate P2.0 (2026-09-16, restart único do HA Core autorizado e executado por Wilson) habilitou e carregou as quatro entidades de diagnóstico do dispositivo — `sensor.sensor_vazmento_agua_linkquality`, `sensor.sensor_vazmento_agua_voltage`, `sensor.sensor_vazmento_agua_trigger_count`, `sensor.sensor_vazmento_agua_power_outage_count` — sem nenhum efeito colateral em automação, notificação, luz ou tomada. Baseline pré-troca registrado em 2026-09-16 16:14:44: `linkquality=51`, `voltage=3025 mV`, `trigger_count=0`, `power_outage_count=25`. P2.1 é observação read-only, sem loop automático, sem automação e sem alerta; novo baseline e comparação de eventos serão registrados quando a bateria for trocada. `trigger_count=0` e a ausência histórica de transição para `off` não são prova de ausência de água. Não bloqueia a PEND-011 nem nenhuma outra frente, mas a validação da confiabilidade do sensor (pós-troca de bateria e observação P2.1) é pré-condição para qualquer ativação futura de alerta — não deve ser pulada |
| PEND-020 | V20.1C/legado — eletrodomésticos (micro-ondas) | SOC | Duplicidade confirmada em runtime entre duas automações independentes que ligam o micro-ondas pelo mesmo gatilho; decisão sobre como tratar a duplicidade ainda não foi tomada | duplicidade | Não | ABERTA | `Cozinha - Liga Microondas` (`id 1743565284745`) e `LAB - Liga Agenda Micro Ondas` (`id 1743632883172`) disparam ambas em `schedule.agenda_micro_ondas → on` e executam `switch.turn_on` em `switch.micro_ondas_tomada_1`; runtime confirmou as duas disparando no mesmo horário (`last_triggered` idêntico em 2026-09-16 11:00). Ambas permanecem habilitadas e em operação — nenhuma foi alterada. `LAB - Liga Agenda Micro Ondas` tem uma condição de guarda (switch off) que `Cozinha - Liga Microondas` não tem; essa diferença ainda não foi investigada quanto à origem (qual das duas é a automação canônica, se nasceram de contextos distintos, se alguma tem dependência em outro lugar). Decisão ainda necessária, em lote formal (Quarentena C3 de `docs/auditoria_legado_v20_1c.md`, janela mínima 30 dias, rollback obrigatório): como tratar a duplicidade — não se presume aqui que o caminho será desativar uma das duas; a decisão pode incluir consolidar as duas em uma, ajustar a condição de guarda, ou outro tratamento a definir no Gate próprio |
| PEND-021 | V20.1C/legado — blueprint Bloqueado (monitoramento de energia crítica) | SOC | Lacuna de referência de versão de blueprint nas duas automações reais de equipamento crítico; origem da divergência v5/v6 ainda não investigada | lacuna de referência/versão | Não (automações seguem funcionando normalmente com a configuração já resolvida em runtime; risco só se reeditadas pela UI de blueprint) | ABERTA | `Cozinha - Gestão Energia Geladeira` (`id 1777423497558`) e `Dispensa - Gestão Energia Cervejeira` (`id 1777424244998`) usam `use_blueprint: wmoura/monitoramento_energia_v5.yaml`, caminho que não existe nem no Git (`origin/main`) nem na listagem live de blueprints instalados; o único arquivo presente é `blueprints/automation/wmoura/monitoramento_energia_v6.yaml`, classificado Bloqueado em `docs/auditoria_legado_v20_1c.md`, e que não é usado por nenhuma automação hoje. Ambas as automações permanecem habilitadas e em operação normal, com a configuração já resolvida/cacheada pelo HA — nenhuma foi alterada e o funcionamento atual deve ser preservado até que haja evidência e Gate próprio. Investigação ainda necessária: por que `v5` não existe (renomeação, exclusão, nunca versionado, divergência entre edição via UI e o que está no Git) e se `v6` é de fato uma evolução compatível de `v5` ou um blueprint distinto; não se presume aqui que o destino será migrar as instâncias para `v6` — a decisão depende dessa investigação e de Gate próprio antes de qualquer alteração |

### Itens antigos resolvidos ou superados

| Item do snapshot de maio | Classificação | Evidência / destino |
| --- | --- | --- |
| V20.1C descrita como “planejada” | SUPERADA | `V20.1C_FECHAMENTO` registra diagnóstico e governança concluídos; somente decommission continua aberto em PEND-012 |
| Afirmação de que todos os testes V20.2 Fase 1A estavam pendentes | SUPERADA | Execução posterior registra 10 executados: 7 OK, 1 parcial e 2 bloqueados; saldo permanece em PEND-009 |
| Destino do dashboard legado `teste-4` | RESOLVIDA | Removido pela UI/fluxo suportado e registrado em AGENTS, arquitetura e Changelog |
| Bloqueadores iniciais de implantação da V20.2E | RESOLVIDA | `md5`, carregamento das automações, bootstrap fail-closed e consumidor canônico foram corrigidos; cobertura residual encerrada — ver PEND-003 — V20.2E abaixo |
| V20.2F mantida fechada até autorização formal | SUPERADA | Necessidade de zonas confirmada e elevada a backlog priorizado; implementação continua condicionada a Gate próprio em PEND-004 |
| Radar de Movimento listado como “implementado mas não validado” | SUPERADA | Existe apenas como planejamento futuro, sem implementação autorizada |
| V20.1C listada como “ainda sem execução” | SUPERADA | Auditoria/diagnóstico executados; remoção do legado permanece bloqueada em PEND-012 |
| Dashboard V19 `teste-4` como risco de navegação | RESOLVIDA | Remoção suportada confirmada; referências antigas permanecem apenas históricas |
| PEND-013 — destino de `docs/release_central_operacional_v20.md` | RESOLVIDA | O próprio documento registra “Release Baseline”, congelamento em 2026-05-13 e status de baseline V20.0 congelada; permanece histórico e não recebe evoluções posteriores |
| PEND-015 — autoridade de `docs/roadmap_central_operacional_semantic_house_v_26.md` | RESOLVIDA | Classificado como visão estratégica conceitual subordinada; não é roadmap canônico nem declara status operacional |
| PEND-006 — handoff do Health Check | RESOLVIDA | Conteúdo de `main` incorporado seletivamente como `docs/handoffs/HANDOFF_HEALTH_CHECK_ENCERRADO.md`; original preservado, artefato auxiliar e nenhuma autorização histórica transportada |
| PEND-008 — publicação da consolidação documental | RESOLVIDA | PR #16 mergeado em `feature/v20-2c-contextual-automations` pelo merge commit `4a0f63b` |
| PEND-007 — divergência entre `main` e a feature | RESOLVIDA | Histórias reconciliadas e mergeadas em `main` pelo PR #18, merge commit `a25c4713b2f7e1226db77cf90604e3d5529934cb`; referências de segurança e branches anteriores preservadas. A sincronização local permanece separada e bloqueada pela PEND-001 |
| Correção de segurança e identidade do alarme | RESOLVIDA | Código exposto rotacionado; referências migradas para `!secret home_alarm_code`; ID duplicado substituído por `alarme_desativar_automaticamente_ao_acordar_v1`; configuração, reload e coexistência das automações validados. Commit remoto `859531c`; teste integral dos gatilhos permanece em PEND-016 |
| PEND-003 — V20.2E / Guard, Concorrência, Push Matrix, Deduplicação | RESOLVIDA | Guard/Concorrência homologado: branch `event_code=car_use_ended`/`termino_request_id` de `script.carro_liberar_bloqueio_rejected` comprovada em runtime controlado (persistência, bloqueio de reconciliação e recuperação administrativa), com rollback integral confirmado e zero efeito colateral (zero ação física, zero push real, zero publicação indevida na Timeline). Os três cenários antes BLOCKED (indisponibilidade, tipo inválido, concorrência real de `has_value`/`current`) encerrados por aceitação fundamentada de risco residual — implementação existente, revisão estática independente já aprovada em `docs/governance/gates_v20.md`, ausência de defeito conhecido, caminho normal comprovado em runtime (ciclo real de uso do carro em 2026-09-13 concluído com sucesso, ACKs `published`) e desproporcionalidade de criar infraestrutura de teste exclusiva para atributos nativos `current`/`has_value` sem uso produtivo. Push Matrix: independência Push × Timeline confirmada por leitura de código e já aprovada estaticamente; matriz completa 2×2 nunca foi requisito formal obrigatório do Gate canônico; ativação dos helpers `carro_push_inicio_uso`/`carro_push_fim_uso` permanece escolha operacional de Wilson, não dívida técnica da V20.2E. Deduplicação: mecanismo pertence ao consumidor canônico compartilhado `sensor.casa_evento_publicavel_v20`/motor V20.1O, não ao produtor `carro_presenca`; já homologada estática e continuamente em runtime por todos os produtores da Timeline. Saldo técnico final = zero. Critérios normativos e checklist atualizados em `docs/governance/gates_v20.md` |
| PEND-001 — Working tree local / frentes mistas | RESOLVIDA | Gates P1 (auditoria e classificação arquivo/hunk, PASS) e P2 (reconciliação, PASS) de 2026-09-11: base local de `/Volumes/config` realinhada com `origin/main` (`194562e`) por branch de segurança + commit WIP + `checkout -B` + reaplicação seletiva por cópia exata de arquivo — sem `reset --hard`, `clean` ou `pull` destrutivo. Dez itens que apareciam como alteração local eram apenas efeito do HEAD antigo (`e665417`, com `4ef2362` como ancestral distante) e já estavam publicados em `main`; removidos do diff sem perda de conteúdo, com rollback preservado em `backup/pend-001-head-antigo-20260911-133452` e `backup/pend-001-wip-snapshot-20260911-133452`. Resíduos genuinamente ainda não publicados foram identificados, preservados intactos e transferidos às respectivas frentes: CSMR/V20.2C (`docs/v20_2c/c1_saida_de_casa.md`, `docs/v20_2c/plano_tecnico_csmr.md`, `packages/csmr_dispatcher_integracao_v20_2c.yaml`, `packages/v20_2c_contextual_automations.yaml`, `packages/v20_2c_protect_csmr.yaml`) e o arquivo misto CSMR+SmallTV `docs/ARCHITECTURE.md`. Nenhum desses resíduos reabre a PEND-001; CSMR/H4 permanece estacionado. Encerramento documental em `docs/handoffs/HANDOFF_PEND001_ENCERRADO.md` |
| PEND-002 — Recovery 4G / V20.1Q | RESOLVIDA | Gate P1 (PASS) identificou exatamente 3 cenários residuais sem evidência suficiente — cancelamento em ciclo ativo, retorno antes do esgotamento (índice intermediário) e estabilização de retorno = 0; os demais cenários já tinham evidência de 2026-07-18/20 e não foram repetidos. Qualificação da Via A (PASS) classificou o harness de simulação já existente em `main` (`8695c02`, `dd350b0`) como **H1 — REPRESENTATIVO**: reutiliza integralmente a lógica produtiva do orquestrador/Executor, com pontos controlados de mock apenas para o sinal de conectividade e o alvo de atuação física; isolamento comprovado estaticamente em 5/5 pontos de atuação física antes de qualquer execução. Execução via harness (2026-09-11) obteve evidência PASS para os 3 cenários: retorno intermediário e estabilização=0 compartilharam um único ciclo controlado (`request_id=r4g-20260911182637261461`, validado na tentativa 2 de 3, sem cooldown); cancelamento exercitado em ciclo independente (`request_id=r4g-20260911183007444582`, veredito `cancelado_operador`). Durante toda a execução, `switch.0xa4c1381045aeb344` (tomada real) e `binary_sensor.backup_4g_operacional` (sensor real) não tiveram nenhuma transição de estado — isolamento físico comprovado empiricamente, nenhuma ação física real ocorreu. Parâmetros operacionais restaurados aos valores originais ao final (`max_tentativas=2`, `estabilizacao_retorno_minutos=3`, `confirmacao_queda_minutos=3`, `timeout_validacao_segundos=120`, `tempo_off_segundos=5`, `timeout_confirmacao_tomada_segundos=5`, `cooldown_minutos=10`, `automatico=on`, `modo_teste=off`). **Limitação não bloqueante:** o harness comprova a resposta da máquina de estados ao retorno intermediário controlado, mas não reproduz o timing ou comportamento probabilístico real da operadora/ISP. Detalhes completos em `docs/governance/despacho_pend002_recovery_4g_v20_1q_fechamento.md` |
| PEND-016 — Alarme/Alexa — resíduos operacionais | RESOLVIDA | Encerrada, corrigida e homologada: `automation.alarme_modo_casa` deixou de referenciar um script órfão (`script.1743352611708`, ausente do config) e passou a chamar `notify.alexa_media` diretamente; ação TTS desabilitada/redundante em `automation.alarme_alarme_disparado_2_2` removida; `check_config` e `automation.reload` PASS; bloqueio externo da integração `alexa_media` (conta presa em `setup_retry`) resolvido por atualização do Alexa Media Player (5.7.0 → 5.15.7) com restart do HA (evidência humana); homologação física real com ciclo único armar→anúncio→desarmar pelo caminho produtivo normal e confirmação auditiva humana de Wilson da mensagem "Alarme ligado modo Casa". Residuais explicitamente fora do escopo: `media_player.moura_s_echo_dot_2` indisponível (não participa do fluxo), par legado `modo_dormir_autoarmar_alarme`/`modo_dormir_desarmar_alarme_ao_acordar` mirando entidade inexistente. Detalhes em `docs/governance/despacho_pend016_alarme_alexa_fechamento.md` |
| PEND-017 — MacBook/Dell/Time Machine | RESOLVIDA | Encerrada e publicada: os dois `webhook_id` expostos no histórico do PR #21 foram rotacionados e passaram a ser protegidos exclusivamente por `secrets.yaml` (`!secret`) + macOS Keychain, sem literal em nenhum arquivo versionado; `check_config` e `automation.reload` PASS sem restart do HA; homologação física real (conexão/desconexão do monitor) confirmou a cadeia completa script→webhook→helper→automação Time Machine→tomada nos dois sentidos, com os dois IDs antigos comprovadamente inertes; PR #21 fechado sem merge; branch reconstruída a partir do `main` vigente com o commit exposto provadamente não-ancestral, publicada e mergeada em `main` via PR #22 (merge commit `248ec78bc1ad05f414c7f5e330d87a64bc6ad529`). Detalhes em `docs/governance/despacho_pend017_webhooks_macbook_dell_fechamento.md` |
| PEND-018 — HD Backup legado | RESOLVIDA | Mecanismo legado `input_boolean.hd_backup` classificado como D — SUBSTITUÍDO pelo mecanismo MacBook ↔ Dell P3424WE ↔ Discos Time Machine (PEND-017); helper e as duas automações órfãs associadas removidas do `entity_registry`; `switch.regua_zigbee_br_l4` renomeada para "Discos Time Machine" (registry + `emulated_hue.yaml`); fechamento promovido a `main` via PR #26. Nota de fechamento também registrada em `docs/auditoria_legado_v20_1c.md`. Detalhes em `docs/governance/despacho_pend018_hd_backup_timemachine_fechamento.md` |
| PEND-005 — Lavadora / watcher pós-cutover | RESOLVIDA | O watcher temporário `packages/lavadora_homologacao_pos_cutover_watch.yaml` (nunca commitado, por desenho) já não existia no working tree; cumpriu sua única função em 2026-08-18 (disparo único coincidindo com o ACK de `washing_started` do ciclo físico real de homologação, `docs/governance/despacho_lavadora_homologacao_fisica_fechamento.md` §2.3) e não é consumido por nenhum outro componente — a FSM produtiva (`packages/lavadora_sessao.yaml`) é independente dele. Único saldo técnico era o registro órfão remanescente `automation.lavadora_watcher_de_homologacao_pos_cutover` (`unique_id: lavadora_homologacao_pos_cutover_watch`, `state: unavailable`, `restored: true`, zero configuração ativa e zero consumidores confirmados antes da remoção) — removido do `entity_registry` em 2026-09-11. Nenhuma ação física ocorreu, nenhum reload/restart foi necessário. Frente da Lavadora permanece homologada sem ressalvas desde `docs/governance/despacho_lavadora_homologacao_fisica_fechamento.md`; não há mais pendência funcional relacionada a este watcher |
| PEND-010 — V20.1A / evidências dos helpers, painel, limite da Timeline e persistência | RESOLVIDA | Auditoria pragmática (2026-09-11) fechou os 4 tópicos com evidência live/reutilizada, sem novo teste: (1) helpers `input_number.casa_timeline_max_eventos` e os 12 `input_boolean.casa_timeline_evento_*` existem e estão ativamente consumidos em `packages/motor_timeline_v20.yaml`; (2) todos os 13 helpers confirmados presentes no painel "Parâmetros" (`dashboard-lixo`) via busca live; (3) limite da Timeline confirmado funcionando em produção com valor customizado (`max_eventos=30`, alterado por usuário humano em 2026-09-11), `sensor.casa_timeline_v20.limite_eventos` espelhando o valor em tempo real; reforçado por reuso da mesma evidência de V20.1Q/PEND-002 (limite de 16 eventos homologado em produção); (4) persistência pós-reload/restart coberta por evidência estrutural cruzada — mesmo mecanismo nativo `input_number`/`input_boolean` do HA, mesmo arquivo de parâmetros, já comprovado sobrevivendo a restart real em V20.1Q/Recovery 4G, Lavadora M5 e Alarme/Alexa (PEND-016), sem nenhum indício contrário em nenhuma dessas frentes |
| PEND-014 — Dashboards/debug | RESOLVIDA | Auditoria live/read-only da Lovelace (2026-09-11): navegação oficial confirmada em Overview (`lovelace`) e Minha Casa (`minha_casa`); busca cross-dashboard pelo padrão `_v20_2` (sensores experimentais/shadow) retornou zero ocorrências em qualquer um dos 10 dashboards storage-mode, produtivo ou de debug; a única fonte legada residual, `sensor.central_ultima_mensagem`, aparece exclusivamente nos dois dashboards de debug/laboratório (`testes-anterior`, `debug-operacional-v20`), nunca em dashboard produtivo — confirma o que o snapshot histórico já registrava. Nenhuma dependência funcional indevida encontrada; nenhum saldo funcional remanescente. Observação cosmética não bloqueante: `testes-anterior` e `debug-operacional-v20` têm `require_admin: false` (visíveis a usuários não-admin); registrada aqui apenas como nota, sem abrir nova pendência |
| PEND-011 — V20.1B/legado / Auditoria de decommission (side-effects, consumidores, duplicidades) | RESOLVIDA | Auditoria encerrada por decisão de Wilson, reaproveitando integralmente `docs/auditoria_legado_v20_1c.md`, `docs/dependencias_legado_v20_1d.md`, `docs/impacto_limpeza_v20_1e.md` e verificação read-only ao vivo do HA (2026-09-16); nenhuma nova auditoria geral foi aberta. **RESOLVIDA significa que a obrigação de auditoria está encerrada — não que todas as automações legadas foram corrigidas, migradas ou desativadas**; as automações abaixo permanecem ativas em produção exatamente como auditadas. Achados comprovados: (1) as quatro automações de agenda ainda relevantes — `LAB - Liga/Desliga Agenda Maq. de Lavar` e `LAB - Liga/Desligar Agenda Monitor DELL` — seguem habilitadas e disparando normalmente, sem consumidor V20/CSMR, e o par do Monitor Dell controla `switch.regua_zigbee_br_l1` ("Monitor Dell"), fisicamente distinto de `switch.regua_zigbee_br_l4` ("Discos Time Machine") do mecanismo homologado na PEND-017 — confirmado que não há sobreposição, sem reabrir aquela pendência; (2) a assimetria Liga/Desliga do Monitor Dell é achado conhecido desta mesma PEND-011: o commit `036e207` (PR #34, mergeado 2026-09-12) já havia removido um guard quebrado (`sensor.estabilizador_potencia` indisponível) que bloqueava permanentemente `LAB - Desligar Agenda Monitor DELL`, e substituído uma chamada a script órfão por `notify.alexa_media` direto; `LAB - Liga Agenda Monitor DELL` foi preservada sem alteração, registrada no próprio commit como fallback funcional já validado em gate anterior — a baixa frequência de disparo desse lado é esperada, não defeito. Resíduo não bloqueante herdado do mesmo commit: a homologação física da tomada dedicada ao monitor (fora do circuito por manuseio de Wilson em 2026-09-12) segue sem confirmação de reconexão; não bloqueia este fechamento. (3) Dos 6 blueprints classificados Bloqueado em `docs/auditoria_legado_v20_1c.md`, nenhum está de fato instanciado por `use_blueprint` hoje: em 5 deles (`Gestão de Energia - UPS.yaml`, `Pós-retorno de energia.yaml`, `luz_seguranca_portas_noite.yaml`, `wan_4G_AppleWatch_Energia.yaml`, `Alerta_Vazamento_com_Led.yaml`) as proteções físicas críticas estão cobertas por automações inline equivalentes já em produção — não é lacuna de side-effect; o sexto (`monitoramento_energia_v6.yaml`) tem lacuna real de referência de versão, tratada separadamente em PEND-021. (4) Duplicidade real confirmada em runtime entre duas automações de "Liga Microondas", tratada separadamente em PEND-020. (5) O sensor de vazamento (`binary_sensor.sensor_vazmento_agua_water_leak`) foi isolado desta frente por decisão explícita de Wilson e é acompanhado à parte, sem bloquear este fechamento, em PEND-019. Nenhuma lacuna adicional de side-effect, consumidor ou duplicidade permanece sem destino atribuível à PEND-011 |

### Regras de manutenção

- Usar identificadores estáveis no formato `PEND-XXX`; eles servem somente para rastreabilidade e não substituem as classificações `SOC`/`AT`, os níveis `P1`/`P2`/`P3` nem o estado da pendência.
- Não reutilizar nem renumerar um identificador já atribuído, inclusive depois da resolução do item.
- Registrar aqui somente ação ou decisão concreta ainda necessária.
- Dívida estrutural sem ação priorizada pertence a `docs/technical_debt/backlog_tecnico.md`.
- Futuro/ideia sem aprovação pertence ao roadmap, não a esta fila.
- Ao resolver uma pendência, registrar evidência, atualizar o roadmap/Gate aplicável e mover a linha para “resolvidos ou superados”.
- Não considerar pendência resolvida por idade, existência de código ou documento posterior genérico.
- Handoffs podem referenciar IDs desta fila, mas não criar pendência oficial paralela.

## Snapshot histórico original — levantamento de 2026-05-16

O conteúdo abaixo é preservado para rastreabilidade. Seus títulos e estados não prevalecem sobre a fila reconciliada acima.

## Escopo do Diagnóstico

Foram analisados:

- documentação em `docs/`
- packages em `packages/`
- dashboards Lovelace em `.storage/lovelace*`
- roadmap, release, changelog e baseline
- matriz e execução de testes V20.2
- packages V20, V20.1B e V20.2

Este relatório não altera código, packages, dashboards, sensores ou automações. Ele consolida pendências abertas e riscos conhecidos.

## 1. Pendências Abertas por Versão/Fase

### V20.2E — Integração do Uso do Carro à Timeline

Status:

- implementação e correções estáticas concluídas;
- contrato `publicar_timeline` homologado em runtime;
- ciclo real do carro reconciliado com os identificadores originais;
- ocorrência funcional de 11/08/2026 encerrada tecnicamente.

Bloqueadores da primeira implantação resolvidos: o filtro Jinja `hash` incompatível foi substituído por `md5`, as automações foram carregadas e o bootstrap administrativo manual inicializou os sete `input_text` em modo fail-closed. O marcador `input_boolean.carro_checkpoints_inicializados` foi gravado por último; não houve sessão, request, rejeição, publicação ou push espontâneo.

As correções de checkpoints, controles de push, guard administrativo e consumidor canônico foram aprovadas estaticamente. Permanecem como cobertura runtime separada os cenários artificiais do guard, concorrência e matriz completa dos controles de push:

1. **Persistência do `request_id` do término:** comprovada no ciclo real. O reconciliador reutilizou a sessão e os requests originais, publicou início e término uma vez cada e limpou os checkpoints somente depois do ACK `published` do término.
2. **Controles de push:** permanecem independentes da Timeline e restauram a escolha persistida. A matriz runtime completa ligado/desligado continua pendente; isso não reabre o incidente de publicação.

Controle adicional de rejeição: ACK `rejected` de início ou término persiste estado, evento, request e motivo. O bloqueio completo ou parcial sobrevive a restart e impede reconciliação ou novo ciclo. A liberação administrativa exige `event_code` e `request_id`, valida ambos contra a sessão e o checkpoint correspondente, recusa qualquer metadado preenchido divergente ou inválido e admite `reason` vazio somente na recuperação parcial. Seu guard fail-closed usa `has_value` e `state_attr`; somente dois `current` numéricos nativos, não booleanos, não negativos e exatamente zero permitem avançar. A compatibilidade estática foi aprovada; testes runtime artificiais de indisponibilidade, tipo inválido e escritor ativo permanecem pendentes.

Incidente do consumidor canônico encerrado: `publicar_timeline` foi observado como string `"true"` no Core 2026.7.1. A normalização restritiva aceita somente `true` nativo ou string exatamente `true` após `trim`/normalização de caixa; os demais valores permanecem fail-closed. `check_config`, `template.reload` e runtime comprovaram Timeline, `eventos_json`, `request_ids_json`, ledger, ACK e ausência de falso `published`.

A sessão CSMR real de 11/08/2026 foi auditada. `wilson_left_home` foi solicitado três vezes e falhou antes de `open`; o monitoramento nunca chegou a `active`. No retorno houve apenas `cancel_reservation`; `remote_monitoring_started`, `wilson_arrived_home` e `remote_monitoring_ended` não ocorreram e não devem ser retropublicados. O request real de `wilson_left_home` também não será reapresentado: a Timeline atual não possui `occurred_at`, forma `HH:MM` com `now()` e faz prepend sem ordenação histórica. A evolução temporal permanece separada e não bloqueia a correção homologada.

Próximas ações válidas:

1. consolidar este diff documental e funcional homologado em commit autorizado separadamente;
2. manter como cobertura futura os testes artificiais do guard, concorrência e matriz completa de push;
3. tratar temporalidade histórica somente em lote arquitetural próprio, sem retropublicar a ocorrência de 11/08/2026;
4. manter a candidata V20.2F fechada até autorização formal.

Critérios normativos de encerramento permanecem no Gate V20.2E em `docs/governance/gates_v20.md`.

#### Encerramento (PEND-003, 2026-09-14)

Os testes runtime artificiais do guard/concorrência mencionados acima como pendentes foram concluídos em Gates dedicados, sem reabrir esta auditoria:

- Branch `event_code=car_use_ended`/`termino_request_id` de `script.carro_liberar_bloqueio_rejected` homologada com PASS em runtime controlado (persistência, bloqueio de reconciliação e recuperação administrativa), com rollback integral confirmado e zero efeito colateral.
- Os cenários de indisponibilidade, tipo inválido e concorrência real (`has_value`/`current`) permanecem tecnicamente não reproduzíveis por manipulação administrativa de helpers — encerrados por aceitação fundamentada de risco residual (implementação já revisada e aprovada estaticamente, ausência de defeito, caminho normal comprovado em runtime), não por omissão.
- A matriz completa 2×2 dos controles de push nunca constituiu requisito formal obrigatório deste Gate; a independência Push × Timeline já estava aprovada estaticamente. Ativação dos helpers de push permanece escolha operacional de Wilson.
- A deduplicação pertence ao consumidor canônico compartilhado (motor V20.1O), não ao produtor `carro_presenca`, e já possui homologação estática e runtime contínua por todos os produtores da Timeline.

PEND-003 encerrada com saldo técnico final igual a zero. Checklist correspondente atualizado em `docs/governance/gates_v20.md`.

### V20.0 - Baseline Congelada

Status: concluída e congelada.

Pendências residuais:

- Confirmar se o release V20.0 em `docs/release_central_operacional_v20.md` deve ser atualizado para refletir as evoluções V20.1A/V20.1B/V20.2 ou permanecer como documento histórico congelado.
- Resolver a divergência documental entre V20.0 congelada e documentos posteriores que já descrevem V20.1/V20.2.
- Garantir que commits/tags da baseline não incluam arquivos sensíveis ou dashboards `.storage` indevidos.

### V20.1A - Operational Control Layer

Status: implementada.

Pendências:

- Confirmar validação completa dos helpers de controle:
  - `input_number.casa_timeline_max_eventos`
  - `input_boolean.casa_timeline_evento_*`
- Confirmar se todos os helpers aparecem corretamente no painel administrativo.
- Confirmar se a alteração do limite máximo de eventos da timeline funciona após reload/restart.
- Registrar evidências formais dos testes de bloqueio/publicação por helper.

### V20.1B - Legacy Migration Layer

Status: implementada na camada de eventos, com legado preservado.

Pendências:

- Não considerar V20.1B como migração completa das automações da casa.
- Auditar automações, blueprints e scripts legados antes de qualquer desativação.
- Mapear side-effects legados que podem atualizar helpers, `input_text`, sensores, modos, score ou contexto.
- Separar notificações textuais antigas de ações físicas/recovery.
- Validar duplicidades externas de notificação/push fora da timeline V20.
- Resolver em fase futura a ordem semântica de recuperação internet/failover:
  - desejado: `📡 Failover 4G encerrado` antes de `🌐 Internet normalizada`.
- Validar se intensidade de chuva, banho, vazamento, portas internas e janelas estão plenamente documentados com evidências reais.

### V20.1C - Legacy Decommission Audit

Status: planejada.

Pendências:

- Criar inventário de consumidores do legado.
- Classificar automações por risco:
  - mensagem apenas
  - mensagem + helper
  - ação física
  - recovery
  - segurança/alarme
- Validar dashboards, scripts e packages que ainda leem fontes legadas.
- Definir lotes pequenos e reversíveis para desativação futura.
- Criar plano de rollback por domínio.
- Só remover duplicidades após validação real.

### V20.2 Lote 1 - Contexto Base

Status: implementado em paralelo.

Sensores envolvidos:

- `binary_sensor.casa_vazia_v20_2`
- `binary_sensor.contexto_noturno_v20_2`
- `sensor.casa_contexto_temporal_v20_2`
- `sensor.casa_contexto_humano_v20_2`
- `sensor.casa_contexto_operacional_v20_2`
- `sensor.casa_contexto_ambiental_v20_2`

Pendências:

- Registrar evidências de validação manual dos contextos base.
- Confirmar comportamento de presença/casa vazia em cenários reais.
- Confirmar comportamento de contexto ambiental com chuva, banho e janelas.
- Validar que nenhum alias final consome esses sensores.
- Confirmar rollback simples removendo/desabilitando `packages/motor_contexto_v20_2.yaml`.

### V20.2A - Evolução Contextual de Atividades

Status: decisão arquitetural aprovada para próximos passos.

Pendências:

- Tratar monitoramento de banho como ativo por padrão daqui para frente.
- Não tratar banho como funcionalidade opcional.
- Manter banho inicialmente desacoplado do motor operacional V20.1N.
- Utilizar lógica contextual existente com movimento, umidade e sensores relacionados.
- Definir encerramento padrão por ausência de evidências de banho por 2 minutos.
- Validar que a regra evita falso encerramento por oscilação de movimento, estabilização da umidade e pequenas pausas durante banho.
- Após estabilização, avaliar incorporação de banho ao motor operacional como atividade formal (`🛁 banho`).

### V20.2 Lote 2A - Relevância Contextual

Status: implementado como prova mínima.

Sensores envolvidos:

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_evento_relevante_v20_2`
- `sensor.casa_motivo_relevancia_v20_2`

Pendências:

- Registrar evidências reais para as quatro regras mínimas:
  - porta aberta + casa vazia = `alta`
  - chuva ativa + janela aberta = `critica`
  - internet degradada + noturno = `media`
  - energia ausente + alguém em casa = `critica`
- Confirmar que evento crítico concorrente prioriza corretamente `chuva_janela_aberta`.
- Validar comportamento quando não há evento contextual: `baixa` + `nenhum_evento_contextual`.
- Definir se o score contextual futuro será atributo ou sensor próprio.
- Manter sem integração com timeline, aliases ou score oficial até concluir a Fase 1A.

### V20.2 Lote 2B - Confidence & Stability Shadow Sensors

Status: implementado e carregado no Home Assistant.

Sensores envolvidos:

- `sensor.casa_confianca_contextual_v20_2`
- `sensor.casa_estabilidade_contextual_v20_2`

Baseline observada:

- confiança: `alta`
- estabilidade: `estavel`
- `shadow_mode: true`
- `estabilidade_temporal_real: false`
- `fontes_invalidas: nenhuma`
- `fontes_contraditorias: nenhuma`
- `contradicao_detectada: false`
- `dominio_estimado: nenhum`
- `dominio_oscilante: false`

Pendências:

- Executar matriz de testes reais da Fase 1A.
- Validar contradições reais ou simuladas.
- Validar domínios oscilantes como `observando`.
- Confirmar que `indeterminada` não combina com `estavel`.
- Implementar memória temporal real somente em fase posterior.
- Ainda não criar `sensor.casa_evento_contextual_estavel_v20_2`.
- Não conectar confidence à relevância oficial ainda.

## 2. Pendências Técnicas

- Decomposição explicável de `sensor.casa_score_operacional` ainda não implementada.
- `sensor.casa_score_operacional` permanece simplificado.
- WAN/4G ainda possui parte da lógica em motor separado/versionado.
- Contrato final sem versão para WAN/4G ainda pendente.
- Feed histórico real ainda depende da memória atual da timeline; não reconstrói histórico antigo do Recorder.
- Timeline V20 começa a registrar a partir do reload/restart, sem reconstrução retroativa.
- Ordenação semântica de eventos correlacionados ainda pendente.
- Confidence/stability ainda não têm debounce, cooldown, hysteresis ou temporal decay reais.
- Contexto V20.2 é paralelo e ainda não influencia decisão oficial.
- Harness de testes shadow ainda é apenas proposta documental.
- Radar de Movimento por cômodo está registrado como backlog, sem implementação.
- IA/LLM está documentada como camada opcional futura, sem implementação.

### Auditoria operacional residual V20.2B

Status: encerramento provisório documental.

Achados registrados:

- 21 automações órfãs foram identificadas em `.storage/core.entity_registry`.
- Existem automações ou blocos de automação que exigem validação antes de limpeza, incluindo itens sem trigger detectável, referências a entidades possivelmente inexistentes e ações internas desabilitadas.
- Automações críticas não devem ser removidas automaticamente.
- A limpeza futura deve ocorrer por criticidade, preferencialmente pela UI do Home Assistant, sem edição manual de `.storage`.

Categorias de triagem:

| Categoria | Exemplos de domínio | Diretriz |
| --- | --- | --- |
| Crítico operacional | energia, UPS, internet/failover, vazamento, alarme, porta, segurança, ações físicas | manter até validação real; não remover automaticamente |
| Legado/LAB | automações `LAB`, notificações antigas, experiências e testes | revisar utilidade; remover somente após descarte formal |
| Provável remoção | `nova_automacao*`, duplicatas antigas, órfãs sem domínio crítico | candidato a limpeza futura pela UI |
| Precisa validação | automações sem trigger detectável, referências inexistentes, side-effects desconhecidos | inventariar antes de qualquer decisão |

Pendência futura:

- Criar matriz de decisão por automação com nome, entidade/id, domínio, criticidade, última evidência de uso e ação recomendada.
- Separar a limpeza em lotes pequenos: primeiro provável remoção, depois legado/LAB, e por último itens críticos apenas com teste real.

## 3. Pendências de Documentação

- Atualizar `docs/release_central_operacional_v20.md` ou decidir que ele permanece congelado como baseline V20.0 histórica.
- Consolidar `docs/CHANGELOG.md` com V20.1A, V20.1B e V20.2, caso o changelog oficial deva acompanhar fases posteriores.
- Registrar resultados reais em `docs/execucao_testes_reais_v20_2_fase_1a.md`.
- Atualizar documentação de arquitetura com um mapa mais claro entre:
  - V20.1B determinística
  - V20.2 contexto
  - V20.2 relevância
  - V20.2 confiança
- Documentar explicitamente quais arquivos são históricos, quais são baseline e quais são planejamento futuro.
- Decidir se `docs/roadmap_central_operacional_semantic_house_v_26.md` continua como referência ativa ou documento histórico.
- Criar registro de validação dos reloads/restarts necessários por fase.

## 4. Pendências de Testes

Arquivo principal:

- `docs/matriz_testes_reais_v20_2_fase_1a.md`

Arquivo de execução:

- `docs/execucao_testes_reais_v20_2_fase_1a.md`

Pendências:

- Todos os testes U-001 a U-017 permanecem pendentes no arquivo de execução.
- Todos os testes I-001 a I-008 permanecem pendentes.
- Todos os testes R-001 a R-005 permanecem pendentes.
- Todos os testes B-001 a B-006 permanecem pendentes.
- Todos os testes O-001 a O-005 permanecem pendentes.
- Critérios de saída da Fase 1A ainda não foram marcados como `OK`.
- Pelo menos um teste integrado crítico precisa ser validado com evidência.
- Validar que timeline/feed não recebem eventos da camada shadow.
- Validar que aliases finais não apontam para sensores V20.2.
- Validar que `sensor.status_casa` não é alterado por sensores shadow.
- Validar rollback simples dos packages V20.2.

## 5. Pendências de Dashboard/Interface

Dashboards oficiais e administrativos:

- `.storage/lovelace.sistema_casa`
- `.storage/lovelace.dashboard_lixo`
- `.storage/lovelace.testes_anterior`
- `.storage/lovelace.debug_operacional`
- `.storage/lovelace_dashboards`

Pendências:

- Confirmar se o menu lateral aponta apenas para dashboards V20 oficiais.
- Confirmar se dashboards produtivos não consomem sensores V20.2 experimentais.
- Avaliar se haverá um dashboard de debug V20.2 separado para contexto/relevância/confiança.
- Implementar futuramente seção sob demanda `Radar de Movimento`.
- Adicionar controle futuro de IA:
  - `IA desligada`
  - `IA leve`
  - `IA completa`
- Dashboard legado V19 `teste-4` foi validado como removido por fluxo externo/suportado; manter apenas referências documentais/históricas.
- `.storage/lovelace.debug_operacional` e `.storage/lovelace.testes_anterior` ainda exibem fontes legadas como `sensor.central_ultima_mensagem`; isso é aceitável como debug/fonte real, mas não deve virar dependência de decisão V20.2.

## 6. Pendências Futuras / Backlog

### V20.1C

- Legacy Decommission Audit.
- Mapeamento de dependências invisíveis.
- Auditoria de side-effects.
- Desativação controlada do legado.

### V20.2 / V20.3

- Semantic Timeline Refinement:
  - `timestamp_ocorrencia`
  - `timestamp_confirmacao`
- Correlation and Event Ordering.
- Confidence com memória temporal real.
- Cooldown, debounce, hysteresis e temporal decay reais.
- Integração controlada entre relevância, confiança e decisão contextual.
- Harness de testes shadow real em `packages/test_harness_v20_2.yaml`, se a execução manual não for suficiente.
- Radar de Movimento sob demanda.

### V21

- Criticidade contextual dinâmica.
- Score explicável por contexto, impacto, duração, redundância, presença e horário.
- Separação entre criticidade técnica e relevância humana.

### V22

- Motor semântico determinístico.
- Narrativa curta sem depender de LLM.
- Síntese de causa/efeito entre eventos.

### V23

- Observabilidade operacional.
- Métricas por motor.
- Diagnóstico de sensores indisponíveis.
- Histórico de score e prioridade.

### V24

- IA/LLM opcional.
- IA desligada deve manter o sistema 100% determinístico.
- IA ligada apenas enriquece análise, contexto e recomendações.
- IA nunca deve ser dependência obrigatória para eventos críticos.

## 7. Itens Implementados mas Ainda Não Validados Formalmente

- `packages/motor_contexto_v20_2.yaml`
- `packages/motor_relevancia_v20_2.yaml`
- `packages/motor_confianca_v20_2.yaml`
- Matriz de testes reais V20.2 Fase 1A.
- Cópia operacional de execução dos testes.
- Proposta de harness de testes shadow.
- Radar de Movimento registrado no roadmap.
- Princípio arquitetural de IA opcional registrado no roadmap/arquitetura.
- V20.1C registrada como fase futura, mas ainda sem execução.

## 8. Itens que Dependem de Reinício/Reload do Home Assistant

Dependem de reload de templates/packages ou reinício:

- Novos packages:
  - `packages/motor_contexto_v20_2.yaml`
  - `packages/motor_relevancia_v20_2.yaml`
  - `packages/motor_confianca_v20_2.yaml`
- Alterações em helpers de `packages/parametros_operacionais_v20.yaml`.
- Qualquer package futuro:
  - `packages/test_harness_v20_2.yaml`
  - package futuro do Radar de Movimento
  - package futuro de IA/controle de modo IA

Já observado:

- `packages/motor_confianca_v20_2.yaml` carregou corretamente após correção de atributos.

Ainda pendente:

- Registrar evidência de reload/restart para contexto e relevância V20.2.
- Registrar evidência de que os sensores permanecem após reinício.

## 9. Riscos Conhecidos

- Desativar automações antigas sem auditoria pode causar regressões silenciosas.
- Algumas automações legadas podem ter side-effects além de notificações.
- Dashboards legados com V19 podem confundir navegação se aparecerem como oficiais.
- `.git` grande e banco SQLite grande podem continuar impactando backup Google.
- Versionar `.storage` sensível ou banco SQLite continua sendo risco de segurança e tamanho.
- Score contextual futuro pode ficar opaco sem atributos explicáveis.
- Confidence sem memória temporal real pode dar falsa estabilidade.
- Integração prematura de V20.2 com aliases finais pode quebrar estabilidade da V20.1B.
- IA/LLM pode degradar performance se não for opcional e sob demanda.
- Radar de movimento permanente pode poluir dashboard principal; deve ser sob demanda.
- Eventos correlacionados ainda podem aparecer em ordem semanticamente imperfeita.

## 10. Próximo Passo Recomendado

Próximo passo recomendado: executar e preencher a Fase 1A de testes reais antes de implementar novas camadas.

Ordem sugerida:

1. Validar baseline normal:
   - `sensor.casa_relevancia_contextual_v20_2 = baixa`
   - `sensor.casa_evento_relevante_v20_2 = nenhum_evento_contextual`
   - `sensor.casa_confianca_contextual_v20_2 = alta`
   - `sensor.casa_estabilidade_contextual_v20_2 = estavel`
2. Preencher `docs/execucao_testes_reais_v20_2_fase_1a.md`.
3. Executar pelo menos os testes críticos:
   - I-001 porta aberta sem ninguém em casa
   - I-003 chuva + janela aberta
   - I-007 internet degradada + noturno
   - I-008 energia ausente + alguém em casa
4. Confirmar que timeline/feed, aliases finais e `sensor.status_casa` não são alterados pela camada shadow.
5. Só depois decidir se vale criar o harness real `packages/test_harness_v20_2.yaml`.
6. Após validação, preparar commit seletivo apenas dos arquivos V20.2/documentação realmente relacionados.
