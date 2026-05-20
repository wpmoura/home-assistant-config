# Changelog

Este projeto segue uma adaptação do padrão [Keep a Changelog](https://keepachangelog.com/), com versões orientadas por baseline arquitetural da Central Operacional.

## [v20.1c-legacy-audit] - 2026-05-20

### Added

- Início da auditoria de legado V20.1C com diagnóstico formal de dependências.
- Documento de inventário criado em `docs/auditoria_legado_v20_1c.md`.
- Identificação preliminar de `packages/_disabled/status_casa_v19.yaml`, `.storage` com registros V19 e motores V20.2 em shadow.
- Confirmação inicial de que `ui-lovelace.yaml` não consome `_v20_2` no escopo desta busca.

### Notes

- Auditoria de legado iniciada; sem alterações de YAML de produção nesta fase.
- Próximo passo: detalhar classificação de automações e scripts por risco e dependência.

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
