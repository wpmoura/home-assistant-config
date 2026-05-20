# Investigação V20.1G - `binary_sensor.casa_tv_ativa_v19`

## Objetivo

Descobrir por que `binary_sensor.casa_tv_ativa_v19` ainda possui eventos recentes no histórico, sem alterar produção.

Esta investigação não altera YAML, não remove entidades, não altera dashboards e não cria commit.

## Entidade investigada

- `binary_sensor.casa_tv_ativa_v19`

## Resumo executivo

Classificação: **entidade realmente ativa**.

A entidade não é apenas resíduo de `restore_state`, registro persistido ou falso positivo. O recorder mostra alternâncias reais `on/off` em 2026-05-19, alinhadas no mesmo segundo com mudanças de `media_player.lg_webos_tv_oled55g3psa`.

Causa raiz operacional provável: o template V19 ainda está sendo carregado pelo Home Assistant a partir de `packages/_disabled/status_casa_v19.yaml`, porque o arquivo permanece dentro da árvore incluída por `homeassistant.packages: !include_dir_named packages/`. O nome `_disabled` funciona como convenção documental, mas nesta configuração não deve ser tratado como garantia de exclusão operacional até validação pela UI/logs.

## 1. Origem

### Package

Origem textual encontrada:

- `packages/_disabled/status_casa_v19.yaml`

Trecho lógico identificado:

- nome: `Casa TV Ativa V19`
- `unique_id: casa_tv_ativa_v19`
- entidade gerada: `binary_sensor.casa_tv_ativa_v19`

Fontes usadas pelo template:

- `media_player.lg_webos_tv_oled55g3psa`
- atributo `source` de `media_player.lg_webos_tv_oled55g3psa`
- atributo `app_name` de `media_player.lg_webos_tv_oled55g3psa`
- `binary_sensor.casa_tv_ativa_v18`
- `binary_sensor.casa_tv_ativa_v17`

Não foi encontrada outra definição YAML da entidade fora de `packages/_disabled/status_casa_v19.yaml`.

### Template

O `entity_registry` registra a entidade como:

- `platform: template`
- `entity_id: binary_sensor.casa_tv_ativa_v19`
- `unique_id: casa_tv_ativa_v19`
- `original_name: Casa TV Ativa V19`
- `disabled_by: null`

Conclusão: a origem operacional é template, não MQTT, helper, script ou automação.

### Helper

Não há helper com este nome.

Helpers aparecem apenas como dependências indiretas de outros sensores V19 no mesmo pacote, não como origem de `binary_sensor.casa_tv_ativa_v19`.

### Script

Nenhum consumo ou atualização direta encontrada em:

- `scripts.yaml`
- `scripts_erro500.yaml`

### Automação

Nenhum consumo ou atualização direta encontrada em:

- `automations.yaml`

### MQTT

Nenhuma definição ou consumo MQTT encontrado para `binary_sensor.casa_tv_ativa_v19`.

### `restore_state`

Existe estado restaurado em `.storage/core.restore_state`:

- entidade: `binary_sensor.casa_tv_ativa_v19`
- último estado registrado localmente: `off`
- timestamp: `2026-05-20T00:25:09.054246+00:00`

O `restore_state` explica a persistência do último estado, mas não explica sozinho as alternâncias recentes `on/off`.

### `entity_registry`

Existe registro persistido em `.storage/core.entity_registry`.

Achado relevante:

- `disabled_by: null`

Interpretação: a entidade não aparece marcada como desabilitada no registro persistido. Isso reforça a necessidade de confirmação em Configurações -> Entidades antes de qualquer limpeza.

## 2. Consumidores

### `status_casa`

Não foi encontrado consumo de `binary_sensor.casa_tv_ativa_v19` pelo alias final `sensor.status_casa`.

O consumo ocorre dentro do próprio pacote V19:

- `sensor.casa_score_ativo_v19`
- `sensor.casa_modo_operacional_v19`
- `sensor.status_casa_v19`
- `sensor.atividade_relevante_v19`
- `sensor.casa_evento_canonico_v19`

Conclusão: não há evidência de impacto direto no contrato final `sensor.status_casa`, mas há acoplamento interno na camada V19.

### Sensores derivados

Consumidores V19 internos encontrados em `packages/_disabled/status_casa_v19.yaml`:

- `sensor.casa_score_ativo_v19`
- `sensor.casa_modo_operacional_v19`
- `sensor.status_casa_v19`
- `sensor.atividade_relevante_v19`
- `sensor.casa_evento_canonico_v19`

Esses sensores são legado e não devem ser migrados automaticamente.

### Dashboards

Consumidor direto encontrado:

- `.storage/lovelace.teste_4`

Uso identificado:

- card condicional com `entity: binary_sensor.casa_tv_ativa_v19`

Não foi encontrado consumo em `ui-lovelace.yaml`.

### Automações

Nenhum consumo direto encontrado em `automations.yaml`.

### Scripts

Nenhum consumo direto encontrado em `scripts.yaml` ou `scripts_erro500.yaml`.

### Templates

Consumidores template estão concentrados no pacote V19 preservado em `_disabled/`.

Também há templates V20 ativos que consomem `media_player.lg_webos_tv_oled55g3psa`, mas não consomem `binary_sensor.casa_tv_ativa_v19`.

### Grupos

Nenhum grupo com referência direta à entidade foi encontrado no escopo analisado.

## 3. Histórico

### Timestamps recentes

O recorder em `home-assistant_v2.db`, consultado em modo somente leitura, contém eventos recentes para `binary_sensor.casa_tv_ativa_v19`.

Últimas alternâncias relevantes:

| Estado | Timestamp local |
|---|---|
| `on` | 2026-05-19 17:58:16 |
| `off` | 2026-05-19 18:27:33 |
| `on` | 2026-05-19 19:07:07 |
| `off` | 2026-05-19 19:48:09 |
| `on` | 2026-05-19 20:18:07 |
| `off` | 2026-05-19 21:25:09 |

Distribuição por dia no recorder:

- 2026-05-19: 5 eventos `on`, 5 eventos `off`
- 2026-05-18: 3 eventos `on`, 3 eventos `off`
- 2026-05-17: 1 evento `on`, 2 eventos `off`
- 2026-05-16: 1 evento `on`, 1 evento `off`
- 2026-05-15: 9 eventos `on`, 8 eventos `off`, 5 `unknown`, 1 `unavailable`

### Correlação com a TV física

O histórico mostra coincidência temporal entre a TV física e a entidade V19.

Exemplo:

| Entidade | Estado | Timestamp local |
|---|---|---|
| `media_player.lg_webos_tv_oled55g3psa` | `on` | 2026-05-19 17:58:16 |
| `binary_sensor.casa_tv_ativa_v19` | `on` | 2026-05-19 17:58:16 |
| `media_player.lg_webos_tv_oled55g3psa` | `off` | 2026-05-19 18:27:33 |
| `binary_sensor.casa_tv_ativa_v19` | `off` | 2026-05-19 18:27:33 |

Essa correlação é compatível com o template V19, que considera a TV ativa quando o `media_player` não está em `off`, `standby`, `unknown` ou `unavailable`, ou quando atributos como `source`/`app_name` estão presentes.

### Quem gerou a alteração

O recorder não possui eventos `state_changed` disponíveis na tabela `events` para esta análise.

Nos registros de `states`, as mudanças recentes de `binary_sensor.casa_tv_ativa_v19` aparecem com:

- `context_user_id`: vazio
- `context_parent_id`: vazio

Interpretação: não há evidência de usuário, script ou automação chamando a mudança. O padrão é compatível com atualização interna de entidade/template pelo Home Assistant.

### Padrão de eventos

O padrão observado é:

1. `media_player.lg_webos_tv_oled55g3psa` muda para estado ativo.
2. `binary_sensor.casa_tv_ativa_v19` muda para `on` no mesmo segundo.
3. `media_player.lg_webos_tv_oled55g3psa` muda para `off`.
4. `binary_sensor.casa_tv_ativa_v19` muda para `off` no mesmo segundo.

Conclusão: a entidade segue a TV física e não se comporta como registro parado.

## 4. Classificação

| Hipótese | Resultado | Evidência |
|---|---|---|
| Entidade realmente ativa | Confirmada | Alterna `on/off` com a TV em 2026-05-19 |
| Entidade shadow | Não | Nome V19 e origem template legado |
| Persistida apenas por registro | Não | Registro existe, mas há eventos recentes reais |
| Falso positivo | Não | Correlação temporal com `media_player` |

Classificação final: **entidade realmente ativa**.

## Causa raiz

A causa raiz é que `binary_sensor.casa_tv_ativa_v19` ainda está sendo avaliado como entidade template ativa.

A origem textual única encontrada é `packages/_disabled/status_casa_v19.yaml`, e `configuration.yaml` inclui packages por:

```yaml
homeassistant:
  packages: !include_dir_named packages/
```

Portanto, a hipótese operacional mais forte é:

- o arquivo `packages/_disabled/status_casa_v19.yaml` ainda permanece dentro do escopo de carregamento de packages;
- a pasta `_disabled` não deve ser considerada desativação efetiva enquanto estiver sob `packages/`;
- o template de TV V19 continua reagindo ao estado real da TV.

## Impacto

Impacto direto em produção: **baixo**, porque não há consumo direto em automações, scripts, `ui-lovelace.yaml` ou `sensor.status_casa`.

Impacto indireto:

- mantém entidade V19 viva;
- mantém ruído no recorder;
- mantém `lovelace.teste_4` funcional/parcialmente funcional com V19;
- invalida a classificação de `packages/_disabled/status_casa_v19.yaml` como seguro para remoção sem validação adicional;
- pode confundir auditorias futuras de legado.

## Risco

Risco atual: **médio**.

Motivos:

- entidade V19 ativa em produção, ainda que sem consumo crítico identificado;
- origem em pacote considerado desativado documentalmente;
- possibilidade de outras entidades V19 também serem carregadas, mesmo que estejam `unavailable`;
- edição/remoção sem procedimento pode causar efeitos não previstos se algum uso manual ou dashboard oculto ainda depender dela.

Risco para `sensor.status_casa`: **baixo no estado observado**, pois não há consumo direto encontrado.

Risco para limpeza futura: **alto se feita automaticamente**, porque a entidade ainda está viva.

## Recomendação

Não executar limpeza agora.

Próximos passos recomendados:

1. Confirmar pela UI em Developer Tools -> States se `binary_sensor.casa_tv_ativa_v19` existe e muda com a TV.
2. Confirmar em Configurações -> Entidades se a entidade aparece ativa e com `platform: template`.
3. Validar em logs/config check como o Home Assistant trata `packages/_disabled/status_casa_v19.yaml` dentro de `!include_dir_named packages/`.
4. Planejar uma fase própria para isolar V19 fora de `packages/`, por exemplo mover artefatos históricos para `docs/` ou para um diretório fora do caminho de packages.
5. Antes de qualquer alteração, registrar rollback e validar que `sensor.status_casa`, aliases finais, automações, scripts e dashboards oficiais permanecem sem dependência V19.
6. Manter `.storage/core.entity_registry` e `.storage/core.restore_state` intocados.

## Conclusão

`binary_sensor.casa_tv_ativa_v19` não é falso positivo. É uma entidade template V19 ainda ativa, com eventos recentes e correlação direta com o estado da TV física.

A limpeza de V19 deve ser pausada até uma fase controlada de isolamento real do pacote legado, sem tocar em `.storage` e sem alterar contratos finais.
