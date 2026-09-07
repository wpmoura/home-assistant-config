# Impacto de Limpeza V20.1E

## Objetivo

Simular o impacto operacional de uma limpeza futura dos candidatos de legado identificados na V20.1C/V20.1D, sem alterar produção.

Esta etapa é apenas analítica. Nenhum YAML produtivo, entidade, dashboard ou commit foi alterado.

## Regras respeitadas

- Não alterar YAML.
- Não remover entidades.
- Não alterar dashboards.
- Não fazer commit.
- Não modificar `sensor.status_casa`.
- Não alterar aliases finais.
- Manter V20.2 isolada em shadow mode.

## Escopo analisado

Fontes verificadas:

- `packages/`
- `automations.yaml`
- `scripts.yaml`
- `scripts_erro500.yaml`
- `ui-lovelace.yaml`
- `.storage/lovelace*`
- `.storage/lovelace_dashboards`
- `.storage/core.entity_registry`
- `.storage/core.restore_state`
- documentação V20.1C/V20.1D

## Candidatos avaliados

### 1. `packages/_disabled/status_casa_v19.yaml`

Simulação: remover o arquivo histórico desativado.

Consumidores diretos:

- Nenhum consumidor ativo em `automations.yaml`.
- Nenhum consumidor ativo em `scripts.yaml` ou `scripts_erro500.yaml`.
- Nenhum consumidor ativo em `ui-lovelace.yaml`.
- O arquivo é autocontido e está dentro de `_disabled/`.

Dependências indiretas:

- Dentro do próprio arquivo, sensores V19 dependem entre si:
  - `sensor.casa_score_ativo_v19`
  - `sensor.casa_modo_operacional_v19`
  - `sensor.status_casa_v19`
  - `sensor.casa_contexto_humano_v19`
  - `sensor.atividade_relevante_v19`
  - `sensor.casa_timeline_operacional_v19`
  - `sensor.casa_timeline_temporal_v19`
  - `sensor.casa_evento_canonico_v19`
  - `sensor.casa_evento_atual_v19`
  - `sensor.casa_event_feed_v19`
  - `binary_sensor.casa_tv_ativa_v19`
  - `binary_sensor.casa_incidente_ativo_v19`
  - `binary_sensor.casa_event_engine_ativo_v19`
- Também referencia fontes legadas V17/V18 e helpers atuais, mas somente dentro de código desativado:
  - `sensor.status_casa_v17`
  - `sensor.casa_evento_porta_v17`
  - `sensor.casa_evento_chuva_v17`
  - `sensor.casa_evento_vazamento_v17`
  - `sensor.casa_tv_contexto_v18`
  - `sensor.casa_tv_contexto_v17`
  - `binary_sensor.casa_tv_ativa_v18`
  - `binary_sensor.casa_tv_ativa_v17`
  - `input_number.casa_peso_*`
  - `input_boolean.casa_event_engine_ativo`
  - `input_boolean.casa_mostrar_*`
  - `input_select.casa_criticidade_*`

Dependências via template:

- Fortes, mas restritas ao pacote desativado.
- O arquivo contém template sensors e trigger-based sensors V19.
- Como está em `_disabled/`, a remoção não deve alterar execução ativa, desde que não haja processo externo reativando este arquivo manualmente.

Dependências via `sensor.status_casa`:

- Não altera `sensor.status_casa`.
- `packages/status_casa.yaml` contém apenas comentário informando que V19 foi movida para `_disabled/`.
- O candidato cria `sensor.status_casa_v19`, não o alias final `sensor.status_casa`.

Dependências via `restore_state`:

- `.storage/core.restore_state` ainda guarda estados V19 restaurados.
- Remover o arquivo histórico não limpa esses estados automaticamente.
- Impacto esperado: nenhum runtime ativo; apenas permanência de histórico até limpeza controlada do storage.

Dependências via dashboards ocultos:

- `.storage/lovelace.teste_4` consome entidades V19.
- `lovelace_dashboards` registra `teste_4` com `show_in_sidebar: false`, título `Laboratório - Casa Inteligente`.
- Remover o arquivo preservado não muda a situação atual se V19 já está desativada: o dashboard oculto continuará apontando para entidades inexistentes/históricas.

Classificação: **seguro remover**, com ressalva de manter backup externo ou histórico em git antes da limpeza definitiva.

Impacto simulado:

- Produção: baixo.
- Dashboards oficiais: nenhum impacto observado.
- Dashboard oculto `teste_4`: continua com referências quebradas a V19, já esperadas pelo estado atual.
- Rollback: simples se o arquivo estiver versionado ou copiado antes da remoção futura.

### 2. `packages/_disabled/DESATIVACAO_V19.md`

Simulação: remover a documentação histórica da desativação V19.

Consumidores diretos:

- Nenhum consumidor técnico.
- Referenciado apenas como documentação/histórico humano.

Dependências indiretas:

- Não define entidades.
- Não executa templates.
- Não alimenta automações, scripts, dashboards ou aliases.

Dependências via template:

- Nenhuma.

Dependências via `sensor.status_casa`:

- Nenhuma alteração em `sensor.status_casa`.
- O arquivo apenas documenta que `sensor.status_casa_v19` foi desativado.

Dependências via `restore_state`:

- Nenhuma.

Dependências via dashboards ocultos:

- Nenhuma.

Classificação: **seguro remover** do ponto de vista operacional, mas a utilidade histórica recomenda mover/arquivar somente depois de congelar a auditoria V20.1C/V20.1D.

Impacto simulado:

- Produção: nenhum.
- Operação: nenhum.
- Auditoria futura: perda de contexto se não houver substituto em `docs/`.

### 3. `.storage/lovelace.teste_4`

Simulação: remover dashboard storage oculto com referências V19.

Consumidores diretos:

- Registrado em `.storage/lovelace_dashboards` como:
  - `id: teste_4`
  - `url_path: teste-4`
  - `title: Laboratório - Casa Inteligente`
  - `show_in_sidebar: false`
  - `mode: storage`
- Não aparece em `ui-lovelace.yaml`.
- Não há referências `_v20_2` em dashboards Lovelace analisados.

Dependências indiretas:

- Consome entidades V19 que já estão desativadas/históricas:
  - `sensor.status_casa_v19`
  - `sensor.casa_modo_operacional_v19`
  - `sensor.casa_score_ativo_v19`
  - `binary_sensor.casa_incidente_ativo_v19`
  - `binary_sensor.casa_tv_ativa_v19`
  - `sensor.casa_tv_contexto_v19`
  - `sensor.casa_contexto_humano_v19`
  - `binary_sensor.casa_event_engine_ativo_v19`
  - `sensor.casa_evento_atual_v19`
  - `sensor.casa_timeline_temporal_v19`
  - `sensor.casa_event_feed_v19`
  - `sensor.casa_timeline_operacional_v19`
  - `sensor.atividade_relevante_v19`

Dependências via template:

- Alta presença de templates Lovelace com `states(...)`, `is_state(...)` e `state_attr(...)`.
- Os templates são apenas de apresentação, não de automação.
- Não há evidência de side-effect operacional.

Dependências via `sensor.status_casa`:

- Não consome `sensor.status_casa`.
- Consome `sensor.status_casa_v19`, que é legado.

Dependências via `restore_state`:

- Algumas entidades V19 consumidas pelo dashboard ainda existem em `.storage/core.restore_state`.
- Remover o dashboard não remove estados restaurados nem registros de entidade.

Dependências via dashboards ocultos:

- Este é o principal dashboard oculto com V19.
- Como `show_in_sidebar: false`, o risco é baixo para uso cotidiano, mas ainda existe risco de acesso direto por URL ou uso como laboratório antigo.

Classificação: **remover somente após migração**.

Impacto simulado:

- Produção: baixo.
- Visibilidade histórica/laboratório: média, pois remove uma tela comparativa antiga.
- Risco técnico: médio-baixo se houver confirmação formal de que `teste-4` não é usado.
- Recomendação: antes de remover, exportar/registrar evidência do dashboard ou migrar qualquer informação ainda útil para documentação.

### 4. Entradas V19 em `.storage/core.entity_registry`

Simulação: remover manualmente registros V19 do entity registry.

Consumidores diretos:

- Home Assistant usa este arquivo como registro interno.
- Foram encontrados registros V19 para 15 entidades, incluindo:
  - `sensor.status_casa_v19`
  - `sensor.casa_event_feed_v19`
  - `sensor.casa_evento_atual_v19`
  - `sensor.casa_evento_canonico_v19`
  - `sensor.casa_timeline_temporal_v19`
  - `binary_sensor.casa_event_engine_ativo_v19`

Dependências indiretas:

- Pode afetar IDs, histórico de entidade e consistência interna do Home Assistant.
- Não há consumo ativo em automações/scripts, mas o arquivo é infraestrutura interna.

Dependências via template:

- Não executa templates, mas registra entidades usadas por templates de dashboard e restore.

Dependências via `sensor.status_casa`:

- Contém registro de `sensor.status_casa` e de `sensor.status_casa_v19`.
- Qualquer edição manual equivocada aqui pode atingir o contrato final, portanto deve ser tratada como crítica.

Dependências via `restore_state`:

- Relaciona-se diretamente com estados preservados em `.storage/core.restore_state`.
- Limpar somente registry sem estratégia para restore pode deixar resíduos ou inconsistência de auditoria.

Dependências via dashboards ocultos:

- `.storage/lovelace.teste_4` ainda referencia entidades V19 registradas/históricas.

Classificação: **risco alto**.

Impacto simulado:

- Produção: risco alto se editado manualmente.
- Recomendação: não tratar como limpeza simples. Só considerar com Home Assistant parado, backup completo e procedimento próprio.

### 5. Entradas V19 em `.storage/core.restore_state`

Simulação: remover manualmente estados restaurados V19.

Consumidores diretos:

- Home Assistant usa este arquivo para restaurar últimos estados.
- Foram encontrados estados V19 com timestamps históricos, incluindo `sensor.status_casa_v19`, timeline/feed V19 e binary sensors V19.

Dependências indiretas:

- Afeta somente histórico/restauração, mas o arquivo é sensível.
- Pode mascarar evidências de auditoria se limpo cedo demais.

Dependências via template:

- Não define templates.
- Alguns dashboards antigos podem exibir dados históricos se entidades ainda forem resolvidas.

Dependências via `sensor.status_casa`:

- Contém estado restaurado de `sensor.status_casa` e de `sensor.status_casa_v19`.
- Edição manual tem risco de tocar no alias final por proximidade no mesmo arquivo.

Dependências via `restore_state`:

- É a própria dependência.

Dependências via dashboards ocultos:

- `.storage/lovelace.teste_4` referencia entidades que também aparecem em restore_state.

Classificação: **risco alto**.

Impacto simulado:

- Produção: risco alto se editado manualmente.
- Operação: baixo se feito corretamente com HA parado, mas não recomendado nesta fase.
- Recomendação: manter intocado até existir procedimento específico de limpeza de storage.

### 6. `packages/status_casa.yaml`

Simulação: alterar/remover a referência V19 encontrada.

Consumidores diretos:

- Arquivo ativo de produção.
- Contém helpers e parâmetros operacionais usados pela Central.

Dependências indiretas:

- Área sensível por proximidade com `sensor.status_casa`.
- A referência V19 atual é comentário:
  - `# V19 DESATIVADO - Todos os sensores V19 foram movidos para /config/packages/_disabled/status_casa_v19.yaml`

Dependências via template:

- Nenhuma dependência V19 ativa neste arquivo foi encontrada.

Dependências via `sensor.status_casa`:

- Área crítica. Mesmo uma limpeza de comentário não deve ser feita durante simulação.

Dependências via `restore_state`:

- Não aplicável ao comentário.

Dependências via dashboards ocultos:

- Não aplicável ao comentário.

Classificação: **dependência desconhecida** para qualquer alteração funcional; **sem ação recomendada** para a referência atual, por ser comentário em arquivo produtivo.

Impacto simulado:

- Remover apenas o comentário teria impacto operacional nulo, mas não agrega valor suficiente para mexer em arquivo sensível.
- Recomendação: não alterar.

## Entidades críticas identificadas

Críticas por contrato final:

- `sensor.status_casa`
- `sensor.atividade_relevante`
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- `sensor.casa_contexto_humano`
- `sensor.casa_prioridade`
- `sensor.casa_score_operacional`

Críticas por legado V19 ainda visível em storage/dashboard:

- `sensor.status_casa_v19`
- `sensor.casa_event_feed_v19`
- `sensor.casa_evento_atual_v19`
- `sensor.casa_evento_canonico_v19`
- `sensor.casa_timeline_temporal_v19`
- `sensor.casa_score_ativo_v19`
- `binary_sensor.casa_incidente_ativo_v19`
- `binary_sensor.casa_event_engine_ativo_v19`

Críticas por shadow V20.2 que não devem ser limpas nesta fase:

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_evento_relevante_v20_2`
- `sensor.casa_motivo_relevancia_v20_2`
- `sensor.casa_confianca_contextual_v20_2`
- `sensor.casa_estabilidade_contextual_v20_2`
- `sensor.casa_contexto_humano_v20_2`
- `sensor.casa_contexto_ambiental_v20_2`
- `sensor.casa_contexto_operacional_v20_2`
- `sensor.casa_contexto_temporal_v20_2`
- `binary_sensor.casa_vazia_v20_2`
- `binary_sensor.contexto_noturno_v20_2`

## Matriz de classificação

| Candidato | Classificação | Motivo |
|---|---|---|
| `packages/_disabled/DESATIVACAO_V19.md` | seguro remover | Documentação histórica sem consumo técnico |
| `packages/_disabled/status_casa_v19.yaml` | seguro remover | Pacote desativado, sem consumo ativo em produção |
| `.storage/lovelace.teste_4` | remover somente após migração | Dashboard oculto com templates V19 e valor histórico/laboratório |
| `.storage/core.entity_registry` entradas V19 | risco alto | Registro interno do Home Assistant |
| `.storage/core.restore_state` entradas V19 | risco alto | Estado interno restaurado e evidência histórica |
| `packages/status_casa.yaml` comentário V19 | dependência desconhecida | Arquivo produtivo sensível; referência é comentário, sem ganho em alterar |

## Resumo executivo

- Total de candidatos seguros: **2**
- Total de candidatos com risco: **4**
- Candidatos de risco médio: **1**
- Candidatos de risco alto: **2**
- Candidatos com dependência desconhecida: **1**
- Produção modificada: **não**
- Dashboards modificados: **não**
- Entidades removidas: **não**

## Ordem sugerida de limpeza futura

1. Arquivar ou consolidar `packages/_disabled/DESATIVACAO_V19.md` em `docs/` se a informação ainda for útil.
2. Remover `packages/_disabled/status_casa_v19.yaml` somente após confirmar que o rollback está coberto por git/backup.
3. Migrar ou exportar o conteúdo útil de `.storage/lovelace.teste_4`; depois remover o dashboard pelo fluxo normal da UI ou procedimento controlado.
4. Validar que nenhum usuário acessa `teste-4` por URL direta.
5. Somente em uma fase própria, com Home Assistant parado e backup completo, avaliar limpeza de entradas V19 em `core.entity_registry` e `core.restore_state`.
6. Não alterar `packages/status_casa.yaml` por causa do comentário V19; manter o contrato final intacto.

## Conclusão

A limpeza mais segura é documental e em `_disabled/`. O único dashboard candidato real é oculto, mas ainda tem templates V19 e deve passar por migração/exportação antes de remoção. Arquivos internos de `.storage` não devem ser tratados como limpeza simples nesta fase.

V20.1E não autoriza remoção operacional. Ela apenas estabelece uma ordem futura de limpeza com risco controlado.
