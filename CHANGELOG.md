# Changelog

## [v20.1q-homologacao-runtime-suspensa] - 2026-07-18

### Added

- Ata de Homologação Runtime V20.1Q em `docs/releases/implementation_plan_v20_1q.md`, cobrindo três testes reais e controlados de power cycle (Teste 1: esgotamento mínimo max=1 + cooldown; Teste 2: cenário max=10 + integridade do snapshot; Teste 3: cenário max=5 + achado de erro técnico seguro).

### Changed

- `docs/governance/gates_v20.md`: Gate corretivo V20.1Q atualizado para refletir itens homologados em runtime (cenários 1/5/10, snapshot, cooldown entrada/expiração, religamento de segurança, erro técnico seguro, restart, Timeline de 16 eventos) e itens que permanecem pendentes (cancelamento pelo operador, retorno em índice intermediário, janela de estabilização zero).
- Status do `docs/releases/implementation_plan_v20_1q.md` alterado para "Homologação Runtime Parcial — Suspensa por decisão operacional".

### Notes

- Homologação **suspensa por decisão operacional**, sem bloqueio técnico conhecido; implementação permanece válida.
- Cenário "2" isoladamente e interrupção por falta de energia em ciclo ativo classificados como não bloqueadores (cobertura por generalização de código e risco residual aceito, respectivamente).
- Nenhum helper, script, automação, package, dashboard ou arquitetura foi alterado nesta atividade — apenas documentação.
- Recomendação registrada para a retomada: monitoramento por assinatura de eventos (WebSocket/`subscribe_events`) em vez de polling.

## [v20.1q-dashboard-parametros-recovery-4g-ux] - 2026-07-17

### Changed

- Seção "Recovery 4G" do dashboard `Parâmetros` reorganizada em três cards, conforme `docs/ux/espec_ux_param_recovery4g.md` (ESPEC-UX-PARAM-RECOVERY4G): **Operação** (Recuperação Automática, Máximo de Tentativas, Pausa após Esgotar Tentativas), **Ciclo de Recuperação** (numerado 1–5: Tempo para Confirmar a Queda, Tempo com a Tomada Desligada, Limite de Espera da Tomada, Limite de Espera do 4G, Tempo de Estabilização) e **Avisos** (Registrar na Timeline, Notificar no Celular).
- Card de texto explicativo (markdown) removido da seção, por exceder o conteúdo exaustivo definido pela especificação.
- Entrega anterior (entrada `v20.1q-dashboard-parametros-recovery-4g` acima) ficou parcial: havia corrigido os campos, mas não a estrutura de cards.

### Method

- Mesma via da entrega anterior: API/WebSocket oficial do Home Assistant (`lovelace/config` / `lovelace/config/save`), sem edição manual de `.storage`, sem alteração de helpers, automações, scripts ou arquitetura.

### Notes

- Validado contra os 15 critérios de aceitação da especificação (A1–A15); A8 (quebra de rótulo em celular) não verificável visualmente nesta sessão.
- Homologação runtime do Recovery 4G continua pendente; Gate corretivo V20.1Q permanece aberto.

## [v20.1q-dashboard-parametros-recovery-4g] - 2026-07-17

### Changed

- Seção "Recovery 4G" do dashboard `Parâmetros` (`.storage/lovelace.dashboard_lixo`) atualizada: card "Execução automática" passa a expor Tempo OFF único, Confirmação da queda e Estabilização do retorno; os dois campos legados numerados de Tempo OFF (tentativa 1 e 2) foram retirados da visualização.
- Diagnóstico técnico, revisão de UX e resolução dos bloqueios técnicos registrados como aprovados pelo usuário para este item, conforme `docs/releases/implementation_plan_v20_1q.md`.

### Method

- Alteração aplicada exclusivamente via API/WebSocket oficial do Home Assistant (`lovelace/config` / `lovelace/config/save`) autenticada com `HA_TOKEN`, sem edição manual de `.storage`.
- Backup do JSON do dashboard salvo antes e depois da escrita; leitura de confirmação executada após a gravação.

### Notes

- Nenhuma outra seção do dashboard, package, automação, script, entidade, alias final ou `sensor.status_casa` foi alterada.
- Homologação runtime do Recovery 4G continua pendente; Gate corretivo V20.1Q permanece aberto.

## [v20.1q-recovery-generico-corretivo] - 2026-07-17

### Changed

- Recovery automático passa a iniciar ligado após restart e Timeline passa a 16 eventos por padrão.
- Orquestrador refatorado para tentativas genéricas com snapshot do máximo configurado.
- Tempo OFF único, confirmação de queda própria e estabilização contínua do retorno.
- Sucesso encerra em `ocioso`, sem cooldown; cooldown ocorre somente após esgotamento completo.
- Cancelamentos por restart, energia e operador deixam de gravar `ultima_execucao`.
- `ultima_execucao` passa a significar o último esgotamento completo que iniciou cooldown.
- Dashboard “Parâmetros” identificado somente em `.storage`; ajuste visual ficou pendente para fluxo suportado pela UI.

### Validation

- Validação exclusivamente estática; homologação no Home Assistant permanece pendente.
- Nenhum reload, restart, power cycle ou push executado.

Este projeto segue uma adaptação do padrão [Keep a Changelog](https://keepachangelog.com/), com versões orientadas por baseline arquitetural da Central Operacional.

## [v20.1q-formalizacao-documental] - 2026-07-13

### Added

- Auditoria operacional do recovery 4G em `docs/auditorias/auditoria_operacional_recovery_4g_v20_1q.md`.
- Despacho arquitetural subordinado em `docs/arquitetura/despacho_arquitetural_v20_1q.md`.
- Implementation Plan transitório em `docs/releases/implementation_plan_v20_1q.md`.
- Gates específicos para documentação, pré-implementação, teste físico, homologação e encerramento da V20.1Q.

### Changed

- Roadmap registra a Fase 0 concluída, V20.1Q.1 como próxima fase e V20.1Q.2 como integração posterior.
- Arquitetura e Source of Truth passam a indexar os artefatos subordinados da V20.1Q.
- Marcador GIT-HYGIENE-01 atualizado como resolvido pelo commit `8e59a4be2e09ebb2706d4fc46a87a7258aa646b0`.

### Notes

- Formalização exclusivamente documental.
- Implementação V20.1Q.1 não iniciada.
- Nenhum YAML, helper, automação, script, dashboard ou `.storage` alterado.
- Nenhum teste físico executado.
- Defaults numéricos de cooldown e timeout de validação permanecem pendentes.

## [v20.1o-politica-timeline-push-agregacao] - 2026-05-21

### Frozen

- V20.1O congelado formalmente como estável com débitos aceitos após validação operacional.
- Política individual por evento implementada para timeline, push e agregação.
- Timeline, push e agregação mantidos como controles independentes.
- Banho/chuveiro habilitado por padrão, com encerramento fixo de 2 minutos.
- Push passa a consumir somente evento canônico atual, sem reutilizar texto renderizado da timeline/feed.
- Agregação passa a respeitar flags individuais e contexto operacional ativo.
- Máquina de lavar passa a respeitar transições reais OFF->ON e ON->OFF, sem desligamento falso em reload/primeira leitura.
- Falso positivo de Backup removido da agregação, exigindo falha real em `sensor.backup_google_status`.
- UI existente integrada com controles Timeline / Push / Agrupar.

### Governance

- Arquivos criados: 0.
- Package shadow: 0.
- Documentação nova: 0.
- Fontes de verdade novas: 0.
- Alterações fora do escopo: 0.
- Compatibilidade legada preservada.
- Mudanças futuras devem abrir novo lote formal e citar dependências do V20.1O.

### Validated

- YAML OK.
- `git diff --check` OK.

### Accepted Debt

- Categoria: UX / Narrativa contextual.
- Sintoma: eventos agregados podem ocultar a saída individual de participantes.
- Exemplo: TV + Microondas + Banho ativos, depois TV e Microondas desligam enquanto Banho continua ativo, mas a timeline pode mostrar apenas Banho encerrado.
- Impacto: não afeta estado real, automações, push ou lógica operacional; afeta apenas interpretação humana da timeline.
- Tratamento: reservado para V20.1P — Inteligência contextual da timeline.

## [v20.1n-estabilizacao] - 2026-05-20

### Added

- Checkpoint documental de estabilização V20.1N em `docs/releases/v20.1n-estabilizacao.md`.
- Registro dos marcos homologados: V20.1N.4.1 Motor operacional estabilizado, V20.1N.4.2 Timeline integrada, V20.1N.4.3 Pós-multiatividade e V20.1N.4.3a Consistência visual.
- Decisão arquitetural de próximos passos para V20.2A - Evolução Contextual de Atividades.
- Registro de que monitoramento de banho passa a ser ativo por padrão daqui para frente, inicialmente desacoplado do motor operacional V20.1N.

### Validated

- TV funcionando.
- Multiatividade funcionando.
- Timeline consistente.
- Deduplicação OK.
- Sem `unavailable`.
- Sem spam.

### Notes

- Status: Homologado.
- Checkpoint exclusivamente documental.
- Nenhum YAML, package, automação, dashboard ou `.storage` foi alterado por esta etapa.
- Nenhuma funcionalidade nova ou evolução funcional foi implementada.
- Banho fica como futuro candidato a atividade formal (`🛁 banho`) após estabilização.

## [v20.1c-legacy-audit] - 2026-05-20

### Added

- Início da auditoria de legado V20.1C com diagnóstico formal de dependências.
- Documento de inventário criado em `docs/auditoria_legado_v20_1c.md`.
- Identificação preliminar de `packages/_disabled/status_casa_v19.yaml`, `.storage` com registros V19 e motores V20.2 em shadow.
- Confirmação inicial de que `ui-lovelace.yaml` não consome `_v20_2` no escopo desta busca.

### Closed

- `V20.1C_FECHAMENTO` registrado como fechamento formal de diagnóstico e governança.
- Inventário concluído.
- Dependências classificadas.
- Impacto documentado.
- Automações, scripts, blueprints e packages auditados.
- Risco operacional classificado.
- Quarentena controlada definida.

### Notes

- Decommission continua bloqueado.
- V20.1O permanece congelada.
- Nenhuma limpeza foi autorizada.
- Nenhuma automação foi autorizada para desligamento automático.
- Limpeza futura continua dependente de classificação de risco, consumidores conhecidos, side-effects mapeados, janela de observação e rollback obrigatório.
- Sem alterações de YAML de produção nesta fase.

## [v20.1d-legacy-dependencies] - 2026-05-20

### Added

- Início da análise de dependências do legado V20.1D.
- Documento de dependências criado em `docs/dependencias_legado_v20_1d.md`.
- Inventário detalhado de `_v19`, `_v20_2`, helpers antigos e uso por dashboards, packages, automações e scripts.

### Notes

- Análise documental iniciada; nenhuma alteração de produção foi realizada.
- Esta etapa foca em mapear dependências reais antes de qualquer limpeza ou migração.

## [v20.1i-disabled-package-isolation] - 2026-05-20

### Added

- Documentação da validação operacional V20.1F em `docs/validacao_operacional_v20_1f.md`.
- Investigação da entidade ativa V19 em `docs/investigacao_casa_tv_ativa_v19.md`.
- Investigação do carregamento efetivo de package V19 em `docs/investigacao_carregamento_v19.md`.
- Arquivo histórico V19 arquivado em `archive/packages_disabled/status_casa_v19.yaml`.
- Documento histórico de desativação V19 arquivado em `archive/packages_disabled/DESATIVACAO_V19.md`.

### Changed

- Artefatos históricos V19 foram movidos para fora da árvore `packages/`, impedindo carregamento por `homeassistant.packages: !include_dir_named packages/` após próximo restart/reload controlado.
- O diretório `packages/_disabled/` foi removido por estar vazio.

### Validated

- Validação pós-isolamento V20.1J documentada em `docs/validacao_pos_isolamento_v20_1j.md`.
- `binary_sensor.casa_tv_ativa_v19` permaneceu existente apenas como resíduo `unavailable`.
- A entidade V19 não alternou após teste real da TV.
- `sensor.status_casa` permaneceu funcional com valor `⚠️ Backup Google com falha`.
- `sensor.casa_timeline` permaneceu funcional com valor `22:37 📺 TV desligada`.

### Notes

- Não houve edição de `.storage/core.entity_registry`.
- Não houve edição de `.storage/core.restore_state`.
- Nenhuma entidade foi removida manualmente.
- Nenhuma lógica produtiva, dashboard ou automação foi alterada.
- Limpeza futura de entidades residuais V19 deve ocorrer apenas em fase própria.

## [v20.1k-v19-residue-documentation] - 2026-05-20

### Added

- Saneamento documental dos resíduos V19 pós-isolamento em `docs/saneamento_residuos_v20_1k.md`.
- Classificação dos resíduos restantes em registry, restore_state, recorder, dashboard oculto e documentação.

### Notes

- Nenhum arquivo `.storage` foi editado.
- Nenhuma entidade foi removida.
- Nenhum YAML produtivo foi alterado.
- Nenhum dashboard foi alterado.
- Entidades V19 residuais foram tratadas como candidatas a limpeza futura, não como dependência ativa.
- Dashboard oculto `teste-4` foi classificado inicialmente como não remover automaticamente e posteriormente removido pela UI/fluxo suportado na consolidação V20.2A.
- A referência documental obsoleta em `packages/status_casa.yaml` foi fechada separadamente no commit `6cbfd18 docs: update archived V19 package reference`.

## [v20.2-operational-residual-audit] - 2026-05-20

### Changed

- Roadmap consolidado para refletir o estado real após V20.1K e V20.2A.
- Arquitetura e contexto operacional atualizados com o estado de V20.1B, V20.1K, V20.2A e V20.2B.
- Registro de que a tag `V20.1K_FECHAMENTO` foi criada.
- Registro de que o dashboard legado `teste-4` foi removido pela UI/fluxo suportado.

### Notes

- Não houve alteração em YAML funcional, packages, entidades, automações ou `.storage`.
- Dashboards ativos não possuem resíduos V19 conhecidos.
- Auditoria de automações identificou 21 órfãs; automações críticas não devem ser removidas automaticamente.
- Próximas prioridades: energia/UPS, HA resiliente principal -> backup RPi5, internet/4G/failover, camada contextual futura e limpeza técnica futura.

## [v20.2-shadow-phaseA] - 2026-05-19

### Added

- Checkpoint formal de homologação Fase A para V20.2 em modo shadow.
- Registro de resultados executados: 10.
- Registro de aprovados: 7.
- Registro de bloqueados: 2.
- Registro de parciais: 1.
- Registro de falhas: 0.
- Documento de checkpoint em `docs/releases/v20.2-shadow-phaseA.md`.
- Achados importantes sobre isolamento, contexto de chuva, baseline contaminada, dashboards legados V19 e revisão de aliases.
- Riscos identificados sobre baseline contaminada, dependências externas em testes de TV, dashboards legados V19 e aliases finais.

### Notes

- Checkpoint: Homologação Fase A concluída.
- V20.2 permanece isolada em shadow e não deve ser promovida para produção nesta etapa.
- Próximo passo: avançar para validação das próximas fases com foco em correlação e revisão de aliases.

## [v20-central-operacional] - 2026-05-13

### Added

- Baseline congelada da Central Operacional V20.
- Aliases finais sem versão para consumo oficial dos dashboards.
- Motor canônico de evento dominante em `motor_eventos_v20.yaml`.
- Timeline operacional V20 com 6 eventos recentes.
- Feed operacional desacoplado do evento dominante.
- Registro de eventos secundários coexistindo com incidente dominante.
- Eventos de TV ligada/desligada com preservação de contexto semântico quando disponível.
- Eventos de Porta da Sala aberta, fechada e aberta por timeout.
- Timeout parametrizável por `input_number.casa_timeout_porta_aberta_minutos`.
- Parâmetros operacionais complementares em `parametros_operacionais_v20.yaml`.
- Documento formal de release em `docs/release_central_operacional_v20.md`.
- Documentação de arquitetura, roadmap e estratégia Git.

### Changed

- Dashboards oficiais passam a depender preferencialmente de aliases finais sem versão.
- `sensor.casa_timeline` e `sensor.casa_event_feed` passam a consumir a timeline/feed V20 quando disponíveis.
- `sensor.atividade_relevante` passa a priorizar contexto semântico de TV quando disponível.
- Dashboard de Parâmetros Operacionais passa a incluir temporizações V20.
- A V20 passa a tratar `status_casa.yaml` como base de parâmetros/governança, não como motor principal.
- Atualização de documentação para consolidar o contexto operacional, arquitetura e estado atual da Central Operacional.

### Fixed

- Regressão em que atividade relevante de TV exibia apenas `TV: Live TV`.
- Dependências diretas de sensores V19 em dashboards oficiais.
- Timeline limitada ao último evento textual.
- Ausência de eventos secundários durante incidente dominante.
- Exposição direta de estados inválidos como `unknown`, `unavailable`, `none` e vazio.
- Risco de commit acidental de arquivos sensíveis do Home Assistant via `.gitignore`.

### Notes

- V19 permanece preservada e não reativada.
- Entidades antigas não foram removidas do registry.
- Automações antigas não foram alteradas.
- A timeline V20 não reconstrói histórico antigo do Recorder; ela registra eventos a partir do reload/restart.
- Mudanças futuras devem ser tratadas como V21 ou hotfix V20 documentado.
