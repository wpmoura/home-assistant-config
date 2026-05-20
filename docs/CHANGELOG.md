# Central Operacional V20 - Changelog

## 2026-05-13 - Fases 1, 2 e 3

### Resumo

Iniciada a migração da Central Operacional para a arquitetura V20, com foco em contratos finais sem versão, proteção dos dashboards contra estados inválidos e desacoplamento gradual dos sensores V17/V18/V19.

A base operacional atual passa a considerar `central_mensagens_corrigido.yaml` como núcleo real, enquanto `status_casa.yaml` permanece como arquivo de parâmetros, pesos e governança.

## Alterações por Package

### `packages/central_operacional_aliases_v20.yaml`

Package criado para concentrar os aliases finais sem versão consumidos pelos dashboards e automações futuras.

Sensores finais criados:

- `sensor.status_casa`
- `sensor.atividade_relevante`
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- `sensor.casa_contexto_humano`
- `sensor.casa_prioridade`
- `sensor.casa_score_operacional`

Fonte semântica interna criada:

- `sensor.central_tv_contexto_fonte`

Fontes operacionais usadas:

- `sensor.central_alerta_principal`
- `sensor.central_atividade_relevante_fonte`
- `sensor.central_ultima_mensagem`
- `sensor.central_categoria_alerta`
- `input_select.central_mensagem_prioridade`
- `sensor.internet_estado_operacional`
- `sensor.backup_google_status`
- `media_player.lg_webos_tv_oled55g3psa`

Fallbacks oficiais aplicados:

- `status_casa`: `Casa operando normalmente`
- `atividade_relevante`: `Nenhuma atividade relevante`
- `casa_timeline`: `Aguardando eventos`
- `casa_event_feed`: `Aguardando eventos`
- `casa_contexto_humano`: `Contexto não determinado`
- `casa_prioridade`: `normal`
- `casa_score_operacional`: `0`

### `packages/central_mensagens_corrigido.yaml`

A fonte antiga `Atividade Relevante` foi convertida em fonte interna:

- antes: `sensor.atividade_relevante`
- depois: `sensor.central_atividade_relevante_fonte`

Essa mudança liberou o nome final `sensor.atividade_relevante` para ser usado como alias oficial V20, sem conflito ativo de `unique_id`.

### `packages/status_casa.yaml`

Sem alteração funcional nesta rodada.

O arquivo permanece como base de parâmetros, pesos, criticidade, confiança mínima, timeout e visibilidade de eventos.

### `packages/wan_4g_engine_v20.yaml`

Sem alteração nesta rodada.

O motor WAN/4G existente ainda não foi promovido ao contrato final dos dashboards administrativos, porque seus sensores têm sufixo `_v20`. A normalização sem versão fica pendente para uma fase futura.

## Dashboards Alterados

### `.storage/lovelace.sistema_casa`

Dashboard principal da Central Operacional migrado para consumir somente os aliases finais:

- `sensor.status_casa`
- `sensor.atividade_relevante`
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- `sensor.casa_contexto_humano`
- `sensor.casa_prioridade`
- `sensor.casa_score_operacional`

Mudanças principais:

- Removidas referências diretas a sensores V19.
- Cards convertidos para leituras com fallback visual.
- Removidas duplicidades entre evento lógico, evento atual e feed.
- Adicionadas leituras consolidadas de score, prioridade, contexto, atividade e contrato V20.

### `.storage/lovelace.dashboard_lixo`

Dashboard administrativo "Parâmetros" atualizado para governança operacional V20.

Áreas cobertas:

- Internet
- Energia
- WAN
- Failover 4G
- Backup Google
- UPS
- Criticidade de internet
- Timeout de evento
- Confiança mínima
- Visibilidade de eventos

Removida dependência direta de:

- `binary_sensor.casa_event_engine_ativo_v19`

### `.storage/lovelace.testes_anterior`

Dashboard "Engenharia Semântica" migrado de painel V19 para painel semântico V20.

O painel agora apresenta:

- status operacional atual
- score operacional atual
- contexto humano atual
- atividade relevante atual
- timeline atual
- feed atual
- fontes operacionais reais

Removidas as dependências diretas de sensores V19 como:

- `sensor.status_casa_v19`
- `sensor.casa_modo_operacional_v19`
- `sensor.casa_score_ativo_v19`
- `sensor.casa_contexto_humano_v19`
- `sensor.atividade_relevante_v19`
- `sensor.casa_event_feed_v19`
- `sensor.casa_evento_atual_v19`
- `binary_sensor.casa_incidente_ativo_v19`
- `binary_sensor.casa_event_engine_ativo_v19`

### `.storage/lovelace.debug_operacional`

Dashboard novo criado: "Debug Operacional V20".

Conteúdo:

- aliases vivos
- fontes reais usadas pelos aliases
- fallback aplicado
- origem dominante
- score atual
- prioridade atual
- estado da internet
- estado do backup
- contexto semântico da TV
- motores vivos

### `.storage/lovelace_dashboards`

Registro Lovelace atualizado para expor o novo dashboard:

- `Debug Operacional V20`
- caminho: `debug-operacional-v20`

## Sensores Removidos

Nenhuma entidade foi removida do registry.

Nenhum sensor V17, V18 ou V19 foi apagado.

Mudança estrutural relevante:

- `sensor.atividade_relevante` deixou de ser produzido diretamente por `central_mensagens_corrigido.yaml`.
- O mesmo nome foi assumido pelo alias final V20 em `central_operacional_aliases_v20.yaml`.
- A fonte anterior foi preservada como `sensor.central_atividade_relevante_fonte`.

## Mudanças de Comportamento

- Dashboards oficiais deixam de consumir sensores versionados diretamente.
- `sensor.atividade_relevante` agora prioriza o contexto semântico da TV quando disponível.
- Conteúdo de TV usa, em ordem:
  - `media_title`
  - `media_channel`
  - `app_name`
  - `source`
  - `📺 TV: Live TV`
  - `📺 TV ligada`
  - `Nenhuma atividade relevante`
- Dashboards oficiais deixam de exibir `unknown`, `unavailable`, `none` ou vazio em cards críticos.
- O dashboard principal passa a tratar score e prioridade como contrato operacional estável.
- O painel administrativo passa a separar governança de parâmetros, leitura operacional e política de visibilidade.
- O debug V20 passa a explicar a origem dos aliases sem recuperar o debug V19.

## Melhorias

- Criação de aliases finais resilientes sem versão.
- Proteção visual dos dashboards contra estados inválidos.
- Redução de dependências cruzadas entre V17, V18, V19 e V20.
- Reintrodução do contexto semântico rico da TV sem reativar V19.
- Separação mais clara entre:
  - motor operacional
  - aliases finais
  - governança
  - engenharia semântica
  - debug operacional
- Novo dashboard de debug desacoplado, focado em fontes vivas e fallbacks.

## Correções

- Corrigida regressão em que `sensor.atividade_relevante` mostrava apenas `📺 TV: Live TV`.
- Corrigido dashboard principal para não depender de sensores V19 desativados.
- Corrigido painel administrativo para remover `binary_sensor.casa_event_engine_ativo_v19`.
- Corrigido painel de Engenharia Semântica para não depender mais do Core V19.
- Corrigidos cards que poderiam renderizar estados inválidos diretamente.

## Breaking Changes

- Dashboards oficiais não devem mais usar sensores versionados diretamente.
- `sensor.atividade_relevante` passa a ser um alias final V20, não mais a fonte bruta do motor atual.
- O dashboard principal deixou de apresentar decomposição V19 de score por atributo.
- O painel "Engenharia Semântica" deixou de ser um debug V19 e passou a ser uma leitura semântica V20.
- O painel "Parâmetros" foi reorganizado como governança operacional, não como painel de pesos legado.

## Próximos Passos

- Criar `motor_eventos_v20` sem versão nas entidades públicas.
- Criar contrato final sem versão para WAN/4G, evitando uso direto de entidades `_v20` em dashboards.
- Implementar feed histórico real para `sensor.casa_event_feed`.
- Implementar timeline multi-evento para `sensor.casa_timeline`.
- Criar decomposição explicável de `sensor.casa_score_operacional`.
- Normalizar origem dominante do evento em sensor final sem versão.
- Consolidar `motor_contexto_v20`, `motor_prioridade_v20`, `motor_timeline_v20` e `motor_presenca_v20`.
- Revisar dashboards legados preservados, especialmente `.storage/lovelace.teste_4`, quando a migração oficial estiver estável.
