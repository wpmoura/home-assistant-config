# Inventario Resumido - Discovery Leve

Data: 2026-05-20
Status: persistido na V20.2 Onda 1

## Objetivo

Persistir o inventario Discovery gerado previamente em modo diagnostico.

Este documento registra apenas inventario resumido. Ele nao mapeia dependencias profundas, nao cruza entidades e nao autoriza implementacao, limpeza ou remocao.

## Packages encontrados

| Package | Classificacao |
| --- | --- |
| `packages/central_operacional_aliases_v20.yaml` | MIGRADO_V20 |
| `packages/motor_eventos_v20.yaml` | MIGRADO_V20 |
| `packages/motor_timeline_v20.yaml` | MIGRADO_V20 |
| `packages/parametros_operacionais_v20.yaml` | MIGRADO_V20 |
| `packages/central_mensagens_corrigido.yaml` | MIGRADO_V20 |
| `packages/wan_4g_engine_v20.yaml` | MIGRADO_V20 |
| `packages/motor_contexto_v20_2.yaml` | MIGRADO_V20 |
| `packages/motor_relevancia_v20_2.yaml` | MIGRADO_V20 |
| `packages/motor_confianca_v20_2.yaml` | MIGRADO_V20 |
| `packages/motor_atividade_operacional_v20.yaml` | MIGRADO_V20 |
| `packages/teste_motor_eventos_v20.yaml` | MIGRADO_V20 |
| `packages/status_casa.yaml` | COMPATIVEL_V20 |
| `packages/energia_contexto.yaml` | COMPATIVEL_V20 |
| `packages/carro.yaml` | COMPATIVEL_V20 |
| `packages/carro_presenca.yaml` | COMPATIVEL_V20 |
| `packages/ventilador_quarto_maior.yaml` | COMPATIVEL_V20 |
| `packages/modo_dormir.yaml` | COMPATIVEL_V20 |
| `packages/ha_inicio.yaml` | COMPATIVEL_V20 |
| `packages/cerebro_backup_formatado.yaml` | COMPATIVEL_V20 |
| `packages/alertas_contextuais_v2_corrigido.yaml` | LEGADO_ATIVO |

## Automacoes encontradas

| Fonte | Encontrado | Classificacao |
| --- | ---: | --- |
| `automations.yaml` | 89 automacoes com alias | LEGADO_ATIVO |
| `packages/alertas_contextuais_v2_corrigido.yaml` | 3 automacoes | LEGADO_ATIVO |
| `packages/carro_presenca.yaml` | 3 automacoes | COMPATIVEL_V20 |
| `packages/ha_inicio.yaml` | 1 automacao | COMPATIVEL_V20 |
| `packages/modo_dormir.yaml` | 4 automacoes | COMPATIVEL_V20 |
| `packages/wan_4g_engine_v20.yaml` | 1 automacao | MIGRADO_V20 |
| `packages/central_mensagens_corrigido.yaml` | 6 automacoes | MIGRADO_V20 |

## Documentos encontrados

- `README.md`
- `docs/ARCHITECTURE.md`
- `docs/CHANGELOG.md`
- `docs/GIT_STRATEGY.md`
- `docs/Informações.txt`
- `docs/ROADMAP.md`
- `docs/architecture_v20_2_context_engine.md`
- `docs/auditoria_legado_v20_1c.md`
- `docs/confianca_operacional_v20_2.md`
- `docs/context_matrix_v20_2.md`
- `docs/dependencias_legado_v20_1d.md`
- `docs/documentação HA.md`
- `docs/dynamic_score_v20_2.md`
- `docs/event_correlation_v20_2.md`
- `docs/execucao_testes_reais_v20_2_fase_1a.md`
- `docs/git_repository_cleanup_strategy.md`
- `docs/harness_testes_shadow_v20_2.md`
- `docs/impacto_limpeza_v20_1e.md`
- `docs/inventario_legacy_migration_v20_1.md`
- `docs/investigacao_carregamento_v19.md`
- `docs/investigacao_casa_tv_ativa_v19.md`
- `docs/mapa_operacional_floorplan_v20_3.md`
- `docs/matriz_testes_reais_v20_2_fase_1a.md`
- `docs/pendencias_atuais_central_operacional.md`
- `docs/plano_execucao_v20_2.md`
- `docs/prompt_com_regras.md`
- `docs/regras_central_operacional_home_assistant_v_17.md`
- `docs/release_central_operacional_v20.md`
- `docs/releases/v20.1n-estabilizacao.md`
- `docs/releases/v20.2-shadow-phaseA.md`
- `docs/relevancia_contextual_v20_2.md`
- `docs/roadmap_central_operacional_semantic_house_v_26.md`
- `docs/saneamento_residuos_v20_1k.md`
- `docs/validacao_operacional_v20_1f.md`
- `docs/validacao_pos_isolamento_v20_1j.md`

## Configuration

| Arquivo | Classificacao |
| --- | --- |
| `configuration.yaml` | COMPATIVEL_V20 |

## Observacao

Este inventario e uma fotografia resumida. Itens classificados como `LEGADO_ATIVO`, `DESCONHECIDO` ou historicos nao devem ser removidos sem fase propria, gates e rollback.
