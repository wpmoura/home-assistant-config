# Investigação V20.1H - Carregamento efetivo do package V19

## Objetivo

Determinar por que uma entidade originada em `packages/_disabled/status_casa_v19.yaml` continua ativa, sem alterar produção.

Esta investigação não altera YAML, não move arquivos, não remove entidades e não cria commit.

## Arquivo investigado

- `packages/_disabled/status_casa_v19.yaml`

## Resumo executivo

Causa raiz definitiva no escopo local: **include incorreto / isolamento inefetivo de package legado**.

O arquivo V19 foi movido para `packages/_disabled/`, mas continua dentro da árvore incluída por:

```yaml
homeassistant:
  packages: !include_dir_named packages/
```

Como a entidade `binary_sensor.casa_tv_ativa_v19` está registrada como `platform: template`, possui `disabled_by: null` no `entity_registry` e continua alternando estado no recorder, a pasta `_disabled` deve ser tratada como convenção documental, não como desativação operacional garantida.

Correção V20.1I aplicada: os artefatos históricos foram movidos para fora da árvore `packages/`:

- `archive/packages_disabled/status_casa_v19.yaml`
- `archive/packages_disabled/DESATIVACAO_V19.md`

Esta correção não edita `.storage`, não remove entidades manualmente e não altera lógica produtiva.

Validação V20.1J: após validação operacional manual no Home Assistant, `binary_sensor.casa_tv_ativa_v19` permaneceu existente apenas como resíduo `unavailable` e não alternou após teste real da TV. `sensor.status_casa` e `sensor.casa_timeline` seguiram funcionando sem impacto aparente.

## 1. Cadeia de includes

### `configuration.yaml`

Includes relevantes encontrados:

- `sensor: !include sensor_commandline.yaml`
- `homeassistant.customize: !include customize.yaml`
- `homeassistant.packages: !include_dir_named packages/`
- `recorder: !include recorder.yaml`
- `input_boolean: !include input_boolean.yaml`
- `input_text: !include input_text.yaml`
- `input_number: !include input_number.yaml`
- `input_select: !include input_select.yaml`
- `automation: !include automations.yaml`
- `script: !include scripts.yaml`
- `scene: !include scenes.yaml`
- `emulated_hue: !include emulated_hue.yaml`
- `template: !include template_sensorstot.yaml`
- `frontend.themes: !include_dir_merge_named themes`
- `alexa: !include alexa.yaml`
- `alarm_control_panel: !include alarm_control_panel.yaml`
- `utility_meter: !include utility_meter_ac.yaml`

Linha crítica:

```yaml
homeassistant:
  packages: !include_dir_named packages/
```

### Árvore de packages

Antes da V20.1I, o arquivo V19 estava localizado em:

```text
packages/_disabled/status_casa_v19.yaml
```

Após a V20.1I, os artefatos históricos foram isolados em:

```text
archive/packages_disabled/status_casa_v19.yaml
archive/packages_disabled/DESATIVACAO_V19.md
```

Itens relevantes sob `packages/` após a correção:

- `packages/status_casa.yaml`
- demais packages V20/V20.2 ativos

O diretório `packages/_disabled/` foi removido por estar vazio.

### Includes indiretos

Não foram encontrados `!include`, `!include_dir_named`, `!include_dir_merge_named` ou `!include_dir_merge_list` dentro de `packages/_disabled/status_casa_v19.yaml`.

Também não foi encontrada inclusão explícita de `packages/_disabled/status_casa_v19.yaml` em outro arquivo. A cadeia observada é indireta pela inclusão do diretório `packages/`.

### Observação sobre loader

O módulo Python do Home Assistant não está instalado neste ambiente local, então a implementação do loader não pôde ser inspecionada diretamente aqui.

Ainda assim, a evidência operacional local é suficiente para concluir que o arquivo está efetivamente carregado ou que seu conteúdo foi carregado e permanece ativo: a entidade template existe no registry, não está desabilitada e segue atualizando com a TV.

## 2. Duplicações

### `status_casa_v19`

Referências encontradas antes da correção:

- `packages/_disabled/status_casa_v19.yaml`
- `packages/_disabled/DESATIVACAO_V19.md`
- `packages/status_casa.yaml` como comentário
- `.storage/lovelace.teste_4`
- `.storage/core.entity_registry`
- `.storage/core.restore_state`

Não foi encontrada definição duplicada ativa fora do antigo arquivo V19. Após a V20.1I, a cópia preservada fica em `archive/packages_disabled/status_casa_v19.yaml`, fora da árvore carregada por packages.

### `casa_tv_ativa_v19`

Referências encontradas antes da correção:

- definição em `packages/_disabled/status_casa_v19.yaml`
- consumo interno em sensores V19 no mesmo arquivo
- documentação em `packages/_disabled/DESATIVACAO_V19.md`
- consumo em `.storage/lovelace.teste_4`
- registro em `.storage/core.entity_registry`
- estado em `.storage/core.restore_state`

Após a correção, a definição preservada está em `archive/packages_disabled/status_casa_v19.yaml`.

Não foi encontrada duplicação em:

- `automations.yaml`
- diretório `automations/`
- `scripts.yaml`
- `scripts_erro500.yaml`
- `template_sensorstot.yaml`
- `template_sensors.yaml`
- `binary_sensor.yaml`
- `sensor.yaml`
- `ui-lovelace.yaml`
- `.storage/core.config_entries`
- `.storage/core.device_registry`

### `binary_sensor.casa_tv_ativa_v19`

Definição textual única antes da correção:

- `packages/_disabled/status_casa_v19.yaml`

Definição preservada após a correção:

- `archive/packages_disabled/status_casa_v19.yaml`

Consumidor visual:

- `.storage/lovelace.teste_4`

Registros persistidos:

- `.storage/core.entity_registry`
- `.storage/core.restore_state`
- `home-assistant_v2.db`

Conclusão: **não há package duplicado nem template duplicado conhecido**.

## 3. Entidade ativa

### `unique_id`

No YAML:

```yaml
unique_id: casa_tv_ativa_v19
```

No entity registry:

- `unique_id: casa_tv_ativa_v19`

### `entity_id`

No entity registry:

- `entity_id: binary_sensor.casa_tv_ativa_v19`

### Origem

Origem textual antes da correção:

- `packages/_disabled/status_casa_v19.yaml`

Origem preservada após a correção:

- `archive/packages_disabled/status_casa_v19.yaml`

Origem operacional antes da V20.1I:

- template entity carregada pelo Home Assistant

Estado operacional após V20.1J:

- entidade ainda registrada, porém `unavailable`;
- sem alternância após teste real da TV;
- comportamento compatível com resíduo de registry/restore, não com template V19 ainda carregado.

### Integration / platform

No entity registry:

- `platform: template`
- `config_entry_id: null`
- `device_id: null`
- `disabled_by: null`

Interpretação:

- entidade de template YAML, não criada por UI helper/config entry;
- não vinculada a dispositivo;
- não marcada como desabilitada.

### State attributes

Últimos atributos observados no recorder:

```json
{"icon":"mdi:television","friendly_name":"Casa TV Ativa V19"}
```

### Histórico

`home-assistant_v2.db` em modo somente leitura registra:

- `binary_sensor.casa_tv_ativa_v19`: 158 estados entre 2026-05-11 23:52:24 e 2026-05-19 21:25:09.
- `sensor.status_casa_v19`: 107 estados entre 2026-05-11 23:52:24 e 2026-05-15 19:28:24.

Últimos estados de `binary_sensor.casa_tv_ativa_v19`:

| Estado | Timestamp local |
|---|---|
| `off` | 2026-05-19 21:25:09 |
| `on` | 2026-05-19 20:18:07 |
| `off` | 2026-05-19 19:48:09 |
| `on` | 2026-05-19 19:07:07 |
| `off` | 2026-05-19 18:27:33 |
| `on` | 2026-05-19 17:58:16 |

Os contextos recentes não têm `context_user_id` nem `context_parent_id`, padrão compatível com atualização interna de template, não ação manual, script ou automação.

## 4. Determinação da causa raiz

### Hipóteses avaliadas

| Hipótese | Resultado | Evidência |
|---|---|---|
| Include incorreto | Confirmada | Arquivo V19 está sob `packages/`, que é incluído por `!include_dir_named packages/`, e a entidade segue ativa |
| Package duplicado | Não confirmado | Nenhuma segunda definição encontrada |
| Template duplicado | Não confirmado | Única definição textual em `packages/_disabled/status_casa_v19.yaml` |
| Persistência indevida | Parcial, mas não causa raiz | `restore_state` e `entity_registry` existem, mas o recorder mostra alternância real `on/off` |
| Outra causa | Não identificada | Sem evidência de script, automação, MQTT ou helper |

### Causa raiz definitiva

**Include incorreto / isolamento inefetivo**.

O arquivo foi colocado em `_disabled`, mas permaneceu dentro da raiz de packages. Como `configuration.yaml` inclui `packages/`, o conteúdo V19 não pode ser considerado operacionalmente desativado apenas pelo nome do diretório.

Correção V20.1I: mover o arquivo histórico para `archive/packages_disabled/`, fora de `packages/`, corrige o isolamento sem tocar em `.storage` nem remover entidades manualmente.

## Impacto

Impacto direto observado antes da V20.1I:

- `binary_sensor.casa_tv_ativa_v19` continua ativa e atualizando.
- `.storage/lovelace.teste_4` ainda consome a entidade.
- O recorder continua recebendo eventos V19.

Impacto observado após V20.1J:

- `binary_sensor.casa_tv_ativa_v19` ainda existe, mas em estado `unavailable`;
- a entidade não alternou após teste real da TV;
- `sensor.status_casa` permaneceu existente com valor `⚠️ Backup Google com falha`;
- `sensor.casa_timeline` permaneceu existente com valor `22:37 📺 TV desligada`;
- não houve impacto aparente em dashboards produtivos.

Impacto não observado:

- nenhum consumo direto em `sensor.status_casa`;
- nenhum consumo direto em `automations.yaml`;
- nenhum consumo direto em `scripts.yaml` ou `scripts_erro500.yaml`;
- nenhum consumo direto em `ui-lovelace.yaml`;
- nenhuma duplicação em templates oficiais fora do pacote V19.

## Risco

Risco antes da V20.1I: **médio**.

Motivos:

- entidade V19 está ativa em produção;
- o diretório `_disabled` gera falsa sensação de desativação;
- há ruído histórico no recorder;
- dashboards ocultos ainda podem consumir legado.

Risco para aliases finais: **baixo no estado observado**.

Risco para limpeza futura automática: **alto**.

Risco após V20.1J: **baixo para produção atual**, com resíduo conhecido em `.storage` e dashboard oculto.

## Correção V20.1I

Ação aplicada:

- Criado `archive/packages_disabled/`.
- Movido `packages/_disabled/status_casa_v19.yaml` para `archive/packages_disabled/status_casa_v19.yaml`.
- Movido `packages/_disabled/DESATIVACAO_V19.md` para `archive/packages_disabled/DESATIVACAO_V19.md`.
- Removido `packages/_disabled/` por estar vazio.

Escopo preservado:

- `.storage/core.entity_registry` não foi editado.
- `.storage/core.restore_state` não foi editado.
- Nenhuma entidade foi removida manualmente.
- Nenhum dashboard foi alterado.
- Nenhuma automação foi alterada.
- Nenhuma lógica produtiva foi alterada.

Resultado validado na V20.1J:

- o template V19 deixou de operar como entidade ativa após o isolamento;
- `binary_sensor.casa_tv_ativa_v19` ficou `unavailable`;
- a entidade não alternou após teste real da TV;
- registros persistidos continuam existindo em `.storage`, como esperado, e não foram limpos nesta fase.

## Recomendação

Não limpar entidades V19 nesta fase.

Para uma futura fase controlada:

1. Tratar entidades V19 `unavailable` como resíduos conhecidos.
2. Revisar o dashboard oculto `teste-4` antes de qualquer remoção futura.
3. Manter rollback simples antes de qualquer alteração.
4. Revalidar `sensor.status_casa`, automações, scripts, dashboards oficiais e `lovelace.teste_4`.
5. Não editar `.storage/core.entity_registry` nem `.storage/core.restore_state` como solução inicial.

## Conclusão

Antes da V20.1I, `packages/_disabled/status_casa_v19.yaml` continuava efetivo para pelo menos `binary_sensor.casa_tv_ativa_v19`.

A causa raiz não era duplicação nem persistência isolada. Era a permanência do package legado dentro da árvore incluída por `configuration.yaml`.

A V20.1I corrige o isolamento movendo o legado histórico para `archive/packages_disabled/`, fora da árvore `packages/`.

A V20.1J valida o resultado: a entidade V19 deixou de alternar e permanece apenas como resíduo `unavailable`, sem impacto aparente em `sensor.status_casa` ou `sensor.casa_timeline`.
