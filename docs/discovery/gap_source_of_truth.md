# Gap Source of Truth - Discovery

Data: 2026-05-20
Modo: somente documentacao

## Objetivo

Determinar conflitos entre a documentacao existente da Central Operacional e classificar cada documento quanto ao seu papel como fonte da verdade, legado, material a arquivar, consolidar ou migrar.

Esta analise usa como entrada o inventario resumido da Fase A - Discovery leve. Nao houve nova analise de packages, YAML, `.storage` ou dependencias.

## Candidatos a fonte da verdade

- `AGENTS.md`: contexto operacional ativo e regras de execucao.
- `docs/ROADMAP.md`: estado atual, fases e proximos passos.
- `docs/ARCHITECTURE.md`: arquitetura oficial consolidada.
- `CHANGELOG.md` e `docs/CHANGELOG.md`: historico de checkpoints e mudancas.
- `docs/pendencias_atuais_central_operacional.md`: pendencias, riscos e backlog operacional.

## Documentos potencialmente obsoletos

- Documentos V17, V18 e V19 devem ser tratados como historicos, nao como fonte operacional atual.
- Investigacoes antigas devem permanecer como auditorias ou evidencias, mas nao devem competir com roadmap/arquitetura.
- Regras antigas devem ser arquivadas ou migradas para governance se ainda forem validas.

## Documentos duplicados

- Roadmap aparece em mais de uma forma: `docs/ROADMAP.md` e `docs/roadmap_central_operacional_semantic_house_v_26.md`.
- Planejamento aparece em `docs/plano_execucao_v20_2.md`, `docs/pendencias_atuais_central_operacional.md` e documentos de matriz/execucao.
- Releases/checkpoints aparecem em `docs/release_central_operacional_v20.md`, `docs/releases/*` e `CHANGELOG.md`.
- Arquitetura aparece em `docs/ARCHITECTURE.md`, `architecture.md` e documentos especificos de motores V20.2.

## Documentos sem dono claro

- `docs/Informações.txt`
- `docs/documentação HA.md`
- `docs/prompt_com_regras.md`
- `docs/git_repository_cleanup_strategy.md`
- `docs/harness_testes_shadow_v20_2.md`

## Matriz de classificacao

| Documento | Objetivo | Status | Destino recomendado |
| --- | --- | --- | --- |
| `AGENTS.md` | Contexto operacional ativo e regras de trabalho | ATIVO | governance |
| `README.md` | Visao geral do repositorio/projeto | CONSOLIDAR | governance |
| `CHANGELOG.md` | Historico principal de mudancas/checkpoints | ATIVO | governance |
| `docs/CHANGELOG.md` | Historico documental adicional | CONSOLIDAR | governance |
| `docs/ROADMAP.md` | Roadmap operacional atual | ATIVO | roadmap |
| `docs/ARCHITECTURE.md` | Arquitetura oficial consolidada | ATIVO | arquitetura |
| `architecture.md` | Arquitetura resumida em raiz | CONSOLIDAR | arquitetura |
| `docs/GIT_STRATEGY.md` | Estrategia Git | ATIVO | governance |
| `docs/release_central_operacional_v20.md` | Release historico V20.0 | ATIVO | auditorias |
| `docs/releases/v20.1n-estabilizacao.md` | Checkpoint V20.1N | ATIVO | auditorias |
| `docs/releases/v20.2-shadow-phaseA.md` | Checkpoint V20.2 shadow Fase A | ATIVO | auditorias |
| `docs/pendencias_atuais_central_operacional.md` | Pendencias e riscos atuais | ATIVO | technical_debt |
| `docs/plano_execucao_v20_2.md` | Plano de execucao V20.2 | CONSOLIDAR | roadmap |
| `docs/matriz_testes_reais_v20_2_fase_1a.md` | Matriz de testes V20.2 | ATIVO | auditorias |
| `docs/execucao_testes_reais_v20_2_fase_1a.md` | Evidencias de execucao V20.2 | ATIVO | auditorias |
| `docs/architecture_v20_2_context_engine.md` | Arquitetura especifica do Context Engine V20.2 | MIGRAR | arquitetura |
| `docs/context_matrix_v20_2.md` | Matriz contextual V20.2 | MIGRAR | arquitetura |
| `docs/relevancia_contextual_v20_2.md` | Relevancia contextual V20.2 | MIGRAR | arquitetura |
| `docs/confianca_operacional_v20_2.md` | Confianca operacional V20.2 | MIGRAR | arquitetura |
| `docs/event_correlation_v20_2.md` | Correlacao de eventos V20.2 | MIGRAR | arquitetura |
| `docs/dynamic_score_v20_2.md` | Score dinamico V20.2 | MIGRAR | arquitetura |
| `docs/mapa_operacional_floorplan_v20_3.md` | Mapa/floorplan futuro V20.3 | MIGRAR | roadmap |
| `docs/roadmap_central_operacional_semantic_house_v_26.md` | Roadmap amplo/antigo Semantic House | CONSOLIDAR | roadmap |
| `docs/inventario_legacy_migration_v20_1.md` | Inventario de migracao legado V20.1 | ATIVO | auditorias |
| `docs/auditoria_legado_v20_1c.md` | Auditoria de legado V20.1C | ATIVO | auditorias |
| `docs/dependencias_legado_v20_1d.md` | Dependencias de legado V20.1D | ATIVO | auditorias |
| `docs/impacto_limpeza_v20_1e.md` | Impacto de limpeza V20.1E | ATIVO | auditorias |
| `docs/validacao_operacional_v20_1f.md` | Validacao operacional V20.1F | ATIVO | auditorias |
| `docs/investigacao_casa_tv_ativa_v19.md` | Investigacao especifica V19 TV | LEGADO | auditorias |
| `docs/investigacao_carregamento_v19.md` | Investigacao de carregamento V19 | LEGADO | auditorias |
| `docs/validacao_pos_isolamento_v20_1j.md` | Validacao pos-isolamento V20.1J | ATIVO | auditorias |
| `docs/saneamento_residuos_v20_1k.md` | Saneamento documental residuos V19 | ATIVO | auditorias |
| `docs/regras_central_operacional_home_assistant_v_17.md` | Regras antigas V17 | ARQUIVAR | governance |
| `docs/git_repository_cleanup_strategy.md` | Estrategia de limpeza Git | DESCONHECIDO | technical_debt |
| `docs/harness_testes_shadow_v20_2.md` | Harness de testes shadow | MIGRAR | auditorias |
| `docs/prompt_com_regras.md` | Prompt/regras operacionais avulsas | MIGRAR | governance |
| `docs/Informações.txt` | Informacoes gerais sem dono claro | DESCONHECIDO | governance |
| `docs/documentação HA.md` | Documentacao geral Home Assistant | DESCONHECIDO | technical_debt |

## Destinos recomendados

### governance

Deve concentrar regras ativas, estrategia Git, modo de operacao, contexto oficial e convencoes de mudanca.

Documentos candidatos:

- `AGENTS.md`
- `README.md`
- `CHANGELOG.md`
- `docs/GIT_STRATEGY.md`
- `docs/prompt_com_regras.md`
- `docs/regras_central_operacional_home_assistant_v_17.md` apenas como historico/arquivo

### roadmap

Deve concentrar fases, proximos passos, backlog e planejamento futuro.

Documentos candidatos:

- `docs/ROADMAP.md`
- `docs/plano_execucao_v20_2.md`
- `docs/roadmap_central_operacional_semantic_house_v_26.md`
- `docs/mapa_operacional_floorplan_v20_3.md`

### technical_debt

Deve concentrar pendencias, riscos, limpezas futuras e itens sem dono claro.

Documentos candidatos:

- `docs/pendencias_atuais_central_operacional.md`
- `docs/git_repository_cleanup_strategy.md`
- `docs/documentação HA.md`
- `docs/Informações.txt`

### auditorias

Deve concentrar checkpoints, validacoes, investigacoes e evidencias historicas.

Documentos candidatos:

- `docs/releases/*`
- `docs/release_central_operacional_v20.md`
- `docs/inventario_legacy_migration_v20_1.md`
- `docs/auditoria_legado_v20_1c.md`
- `docs/dependencias_legado_v20_1d.md`
- `docs/impacto_limpeza_v20_1e.md`
- `docs/validacao_operacional_v20_1f.md`
- `docs/investigacao_casa_tv_ativa_v19.md`
- `docs/investigacao_carregamento_v19.md`
- `docs/validacao_pos_isolamento_v20_1j.md`
- `docs/saneamento_residuos_v20_1k.md`
- `docs/matriz_testes_reais_v20_2_fase_1a.md`
- `docs/execucao_testes_reais_v20_2_fase_1a.md`
- `docs/harness_testes_shadow_v20_2.md`

### arquitetura

Deve concentrar arquitetura oficial e documentos tecnicos de motores.

Documentos candidatos:

- `docs/ARCHITECTURE.md`
- `architecture.md`
- `docs/architecture_v20_2_context_engine.md`
- `docs/context_matrix_v20_2.md`
- `docs/relevancia_contextual_v20_2.md`
- `docs/confianca_operacional_v20_2.md`
- `docs/event_correlation_v20_2.md`
- `docs/dynamic_score_v20_2.md`

## Conclusao

A fonte da verdade atual esta distribuida entre `AGENTS.md`, `docs/ROADMAP.md`, `docs/ARCHITECTURE.md`, `CHANGELOG.md` e `docs/pendencias_atuais_central_operacional.md`.

O principal conflito documental e a coexistencia de documentos ativos, historicos e especificos de motores sem separacao formal por destino. A recomendacao documental e consolidar por familias: governance, roadmap, technical_debt, auditorias e arquitetura, mantendo documentos V17/V18/V19 como historico e nao como fonte operacional atual.
