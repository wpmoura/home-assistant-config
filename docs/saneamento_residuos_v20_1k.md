# Saneamento documental de residuos V19 - V20.1K

## Objetivo

Mapear e classificar os residuos restantes apos o isolamento de `packages/_disabled/`, sem remover entidades, sem editar `.storage`, sem alterar YAML produtivo, sem alterar dashboards e sem criar commit automatico.

## Escopo validado

Itens investigados:

- `binary_sensor.casa_tv_ativa_v19`
- referencias a `packages/_disabled`
- `status_casa_v19`
- dashboard oculto `.storage/lovelace.teste_4`
- referencias V19 em `.storage/core.entity_registry`
- referencias V19 em `.storage/core.restore_state`
- preservacao historica em `archive/packages_disabled/`

## Resumo executivo

O isolamento aplicado na V20.1I e validado na V20.1J funcionou: as entidades V19 nao operam mais como templates ativos e aparecem no recorder com estado final `unavailable`.

Os residuos restantes sao esperados e devem ser tratados como saneamento futuro, nao como falha ativa de producao.

Classificacao geral:

- risco operacional atual: **baixo**
- risco documental: **medio**, por referencias antigas ainda apontarem para `packages/_disabled`
- risco de limpeza automatica: **alto**, porque `.storage`, dashboard oculto e historico ainda contem V19

## 1. Entidades V19 residuais

Consulta somente leitura ao recorder encontrou os estados finais abaixo:

| Entidade | Estado final | Ultima atualizacao local | Classificacao |
| --- | --- | --- | --- |
| `binary_sensor.casa_tv_ativa_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `binary_sensor.casa_incidente_ativo_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `binary_sensor.casa_event_engine_ativo_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.status_casa_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.casa_tv_contexto_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.casa_score_ativo_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.casa_breakdown_ativo_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.casa_modo_operacional_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.casa_contexto_humano_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.atividade_relevante_v19` | sem estado final novo nesta consulta | historico antigo | candidato a limpeza futura |
| `sensor.casa_timeline_operacional_v19` | sem estado final novo nesta consulta | historico antigo | candidato a limpeza futura |
| `sensor.casa_timeline_temporal_v19` | sem estado final novo nesta consulta | historico antigo | candidato a limpeza futura |
| `sensor.casa_evento_canonico_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.casa_evento_atual_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |
| `sensor.casa_event_feed_v19` | `unavailable` | 2026-05-19 22:32:54 | candidato a limpeza futura |

Interpretacao:

- `unavailable` em bloco apos o isolamento e compativel com entidade registrada sem template carregado;
- nao ha evidencia de alternancia ativa de `binary_sensor.casa_tv_ativa_v19` apos a validacao manual da V20.1J;
- nenhuma entidade V19 deve ser removida manualmente nesta fase.

## 2. `packages/_disabled`

Resultado:

- o diretorio `packages/_disabled/` nao existe mais;
- `packages/_disabled/status_casa_v19.yaml` nao existe mais dentro de `packages/`;
- `packages/_disabled/DESATIVACAO_V19.md` nao existe mais dentro de `packages/`.

Residuo encontrado na V20.1K:

- comentario em `packages/status_casa.yaml` ainda referencia `/config/packages/_disabled/status_casa_v19.yaml`.

Status apos V20.1K.1:

- corrigido no commit `6cbfd18 docs: update archived V19 package reference`;
- o comentario agora aponta para `/config/archive/packages_disabled/status_casa_v19.yaml`;
- a alteracao foi apenas documental, sem impacto funcional.

Classificacao:

- diretorio removido: **inofensivo**
- comentario obsoleto em YAML produtivo: **resolvido na V20.1K.1**

Acao futura:

- nenhuma acao pendente para esse comentario;
- manter novas referencias operacionais apontando para `archive/packages_disabled/`.

## 3. `status_casa_v19`

Residuos encontrados:

- `.storage/core.entity_registry`
- `.storage/core.restore_state`
- `.storage/lovelace.teste_4`
- `home-assistant_v2.db`
- documentacao historica anterior a V20.1I
- arquivo preservado em `archive/packages_disabled/status_casa_v19.yaml`

Classificacao:

- registry/restore: **candidato a limpeza futura**
- recorder/historico: **nao remover automaticamente**
- arquivo arquivado: **inofensivo**
- referencias documentais antigas: **requer observacao**

Risco:

- baixo para producao atual;
- medio para confusao operacional se alguem interpretar documentos antigos como estado atual;
- alto se uma limpeza automatica tentar editar `.storage` diretamente.

## 4. Dashboard oculto `.storage/lovelace.teste_4`

Resultado:

- dashboard registrado em `.storage/lovelace_dashboards`:
  - `id: teste_4`
  - `title: Laboratório - Casa Inteligente`
  - `url_path: teste-4`
  - `show_in_sidebar: false`
  - `mode: storage`
- dashboard contem multiplas referencias V19, incluindo:
  - `sensor.status_casa_v19`
  - `sensor.casa_modo_operacional_v19`
  - `sensor.casa_score_ativo_v19`
  - `binary_sensor.casa_incidente_ativo_v19`
  - `binary_sensor.casa_tv_ativa_v19`
  - `sensor.casa_tv_contexto_v19`
  - `sensor.casa_contexto_humano_v19`
  - `sensor.casa_timeline_temporal_v19`
  - `binary_sensor.casa_event_engine_ativo_v19`
  - `sensor.casa_evento_atual_v19`
  - `sensor.casa_event_feed_v19`
  - `sensor.casa_timeline_operacional_v19`

Classificacao: **nao remover automaticamente**.

Motivo:

- e um dashboard oculto/laboratorial, mas ainda e uma configuracao real em `.storage`;
- pode ser referencia historica util para comparar V19/V20;
- alterar ou remover dashboards viola o escopo desta fase.

Acao futura:

- decidir em fase propria se `teste-4` deve ser arquivado, migrado para aliases finais ou removido pela UI;
- se mantido, rotular explicitamente como historico V19 para evitar uso operacional.

## 5. `.storage/core.entity_registry`

Foram encontradas referencias V19 no registry, incluindo:

- `binary_sensor.casa_tv_ativa_v19`
- `binary_sensor.casa_incidente_ativo_v19`
- `binary_sensor.casa_event_engine_ativo_v19`
- `sensor.casa_tv_contexto_v19`
- `sensor.casa_score_ativo_v19`
- `sensor.casa_breakdown_ativo_v19`
- `sensor.casa_modo_operacional_v19`
- `sensor.status_casa_v19`
- `sensor.casa_contexto_humano_v19`
- `sensor.atividade_relevante_v19`
- `sensor.casa_timeline_operacional_v19`
- `sensor.casa_timeline_temporal_v19`
- `sensor.casa_evento_atual_v19`
- `sensor.casa_event_feed_v19`
- `sensor.casa_evento_canonico_v19`

Classificacao: **candidato a limpeza futura**.

Regra:

- nao editar `.storage/core.entity_registry` manualmente;
- se a limpeza for desejada, executar por fluxo suportado do Home Assistant, preferencialmente via UI e com backup.

## 6. `.storage/core.restore_state`

Foram encontrados estados restaurados V19, incluindo:

- `binary_sensor.casa_tv_ativa_v19`
- `binary_sensor.casa_incidente_ativo_v19`
- `binary_sensor.casa_event_engine_ativo_v19`
- `sensor.status_casa_v19`
- `sensor.casa_event_feed_v19`
- `sensor.casa_evento_atual_v19`
- `sensor.casa_tv_contexto_v19`
- `sensor.casa_score_ativo_v19`
- `sensor.casa_modo_operacional_v19`
- `sensor.casa_contexto_humano_v19`
- `sensor.casa_timeline_operacional_v19`
- `sensor.casa_timeline_temporal_v19`
- `sensor.casa_evento_canonico_v19`

Classificacao: **nao remover automaticamente**.

Motivo:

- restore_state e mecanismo interno do Home Assistant;
- editar manualmente pode causar inconsistencias;
- os estados sao residuos esperados apos isolamento de YAML.

## 7. Arquivo historico arquivado

Arquivos preservados:

- `archive/packages_disabled/status_casa_v19.yaml`
- `archive/packages_disabled/DESATIVACAO_V19.md`

Classificacao: **inofensivo**.

Observacao:

- `archive/packages_disabled/DESATIVACAO_V19.md` ainda contem instrucoes antigas apontando para `packages/_disabled/`;
- manter como documento historico e nao como procedimento operacional atual.

## Matriz de classificacao

| Residuo | Local | Classificacao | Acao |
| --- | --- | --- | --- |
| Entidades V19 `unavailable` | registry/recorder | candidato a limpeza futura | limpar somente por fase propria e fluxo suportado |
| `binary_sensor.casa_tv_ativa_v19` | registry/restore/recorder/dashboard oculto | candidato a limpeza futura | observar; nao remover manualmente |
| `status_casa_v19` | registry/restore/dashboard oculto/archive | candidato a limpeza futura | tratar junto do pacote V19 residual |
| `packages/_disabled` inexistente | filesystem | inofensivo | nenhuma |
| Comentario antigo em `packages/status_casa.yaml` | YAML produtivo | resolvido na V20.1K.1 | commit `6cbfd18` atualizou apenas documentacao |
| `.storage/lovelace.teste_4` | dashboard oculto | nao remover automaticamente | revisar por UI em fase propria |
| `.storage/core.entity_registry` | registry | candidato a limpeza futura | nao editar manualmente |
| `.storage/core.restore_state` | restore_state | nao remover automaticamente | deixar o HA gerenciar |
| `archive/packages_disabled/` | arquivo historico | inofensivo | preservar |
| Documentos antigos com `packages/_disabled` | docs | requer observacao | atualizar se forem usados como guia atual |

## Riscos

### Baixo

- Entidades V19 residuais estao `unavailable`.
- `binary_sensor.casa_tv_ativa_v19` nao alternou apos teste real da TV.
- `sensor.status_casa` e `sensor.casa_timeline` seguiram sem impacto aparente na V20.1J.

### Medio

- Documentos historicos antigos ainda podem sugerir que `packages/_disabled/` e o local antigo do backup.
- Dashboard oculto `teste-4` ainda referencia entidades V19 e pode confundir uma leitura manual.

### Alto

- Limpeza automatica ou edicao manual de `.storage`.
- Remocao do dashboard oculto sem decisao explicita.
- Remocao de registros residuais sem backup e sem validar a UI do Home Assistant.

## O que pode esperar

- Limpeza de `.storage/core.entity_registry`.
- Limpeza de `.storage/core.restore_state`.
- Remocao ou migracao do dashboard oculto `teste-4`.
- Atualizacao de documentos historicos antigos que nao sao guias operacionais atuais.
- Remocao de historico no recorder.

## O que exige acao futura

- Decidir o destino do dashboard oculto `teste-4`.
- Corrigir referencias documentais obsoletas para `packages/_disabled` somente quando fizerem parte de documentacao operacional atual.
- Definir procedimento seguro para entidades V19 residuais `unavailable`.
- Confirmar se a limpeza deve ser feita via UI do Home Assistant, nao por edicao direta de `.storage`.

## Recomendacao para V20.2A

Avancar para V20.2A sem bloquear por residuos V19, desde que:

1. V19 seja tratado como residuo inativo e nao como dependencia produtiva.
2. Nenhuma limpeza automatica de `.storage` seja executada.
3. O dashboard oculto `teste-4` fique fora dos dashboards produtivos.
4. A documentacao operacional atual aponte para `archive/packages_disabled/`, nao para `packages/_disabled/`.
5. A limpeza de registry/restore/dashboard oculto seja planejada como fase propria, reversivel e validada pela UI do Home Assistant.

Conclusao: **V20.1K aprova o saneamento documental como mapa de residuos, mas nao autoriza limpeza operacional ainda**.
