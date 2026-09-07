# Dependências de Legado V20.1D

## Objetivo

Mapear dependências reais do legado antes de qualquer limpeza, remoção ou migração.

Regra central: apenas analisar e documentar. Nenhuma alteração de YAML de produção foi feita.

## Escopo

- Entidades `_v19`
- Entidades `_v20_2`
- Helpers antigos (`input_boolean`, `input_number`, `input_select`, `input_text`)
- Automations em `automations.yaml`
- Scripts em `scripts.yaml` e `scripts_erro500.yaml`
- Dashboards em `ui-lovelace.yaml` e `.storage/lovelace*`
- Pacotes em `packages/`

## Metodologia

1. Escaneamento de referências diretas em arquivos de configuração e storage.
2. Separação entre elementos ativos de produção e artefatos de dashboard/storage.
3. Classificação preliminar baseada em presença em produção, dashboard ou restore/registry.

## Inventário resumido

- Referências `_v19` em configuração/storage: **193**
- Referências `_v20_2` em configuração/storage: **225**
- Referências `_v19` em documentação: **87**
- Referências `_v20_2` em documentação: **225**
- `input_boolean.` referências: **478** em 24 arquivos
- `input_number.` referências: **211** em 14 arquivos
- `input_select.` referências: **87** em 12 arquivos
- `input_text.` referências: **106** em 11 arquivos

## V20.1D-A: Validação de consistência do inventário

### Diferença entre V20.1C e V20.1D

- V20.1C foi um diagnóstico preliminar com escopo mais restrito. Os totais originais de `_v19 = 160` e `_v20_2 = 195` refletiam uma busca inicial que não capturou todos os artefatos históricos e de desativação.
- V20.1D atualizou o inventário para incluir fontes adicionais legítimas, mantendo a regra de não alterar YAML de produção.
- As principais origens das diferenças são:
  - `_v19`: inclusão de arquivos legados/desativados e registros históricos adicionais (`packages/_disabled/DESATIVACAO_V19.md`, `.storage/core.entity_registry`, `.storage/core.restore_state`, `.storage/lovelace.teste_4`).
  - `_v20_2`: inclusão de registros históricos adicionais em `.storage/core.restore_state`, além de manter as referências já encontradas em pacotes shadow.

### Quebra por origem

#### `_v19`

- packages: 58
- automations: 0
- scripts: 0
- ui-lovelace: 0
- .storage (Lovelace armazenado): 59
- restore_state: 16
- entity_registry: 30
- documentação: 87

#### `_v20_2`

- packages: 173
- automations: 0
- scripts: 0
- ui-lovelace: 0
- .storage (Lovelace armazenado): 0
- restore_state: 30
- entity_registry: 22
- documentação: 225

### Validação de origem

- Produção ativa:
  - `_v19`: 1 referência em `packages/status_casa.yaml`
  - `_v20_2`: 173 referências em `packages/motor_confianca_v20_2.yaml`, `packages/motor_contexto_v20_2.yaml` e `packages/motor_relevancia_v20_2.yaml`
- Registro histórico:
  - `.storage/core.entity_registry`
  - `.storage/core.restore_state`
  - `.storage/lovelace.teste_4`
  - arquivos desativados em `packages/_disabled`
- Documentação:
  - referências encontradas em `docs/` e em `packages/_disabled/DESATIVACAO_V19.md`
- Candidatos reais de limpeza:
  - `packages/_disabled/status_casa_v19.yaml`
  - `packages/_disabled/DESATIVACAO_V19.md`
  - `.storage/lovelace.teste_4`

> Nota: `.storage/core.restore_state` e `.storage/core.entity_registry` são históricos e devem ser tratados como dados de registro; a limpeza só deve ocorrer após validação completa, backup e Home Assistant parado.

## Resultados principais

### Entidades `_v19`

Encontradas em:

- `.storage/lovelace.teste_4` — **59** referências
- `packages/_disabled/status_casa_v19.yaml` — **57** referências
- `.storage/core.entity_registry` — **30** referências
- `packages/_disabled/DESATIVACAO_V19.md` — **30** referências
- `.storage/core.restore_state` — **16** referências
- `packages/status_casa.yaml` — **1** referência

Observação: não há referências `_v19` diretas em `automations.yaml`, `scripts.yaml`, `scripts_erro500.yaml` ou `ui-lovelace.yaml`.

### Entidades `_v20_2`

Encontradas em:

- `packages/motor_confianca_v20_2.yaml` — **120** referências
- `packages/motor_relevancia_v20_2.yaml` — **42** referências
- `.storage/core.restore_state` — **30** referências
- `.storage/core.entity_registry` — **22** referências
- `packages/motor_contexto_v20_2.yaml` — **11** referências

Observação: não há referências `_v20_2` diretas em `ui-lovelace.yaml`.

### Helpers antigos

Helpers ativos detectados no inventário e suas referências em configuração ativa:

- `input_boolean.*` — uso ativo em `automations.yaml`, `scripts_erro500.yaml`, e vários pacotes V20/V20.2
- `input_number.*` — uso ativo em pacotes V20, automações e scripts
- `input_select.*` — uso ativo em automações e pacotes
- `input_text.*` — uso ativo em automações, scripts e pacotes

O escaneamento mostrou que helpers ainda são consumidos por pacotes e automações ativos. Não há evidência de renomeação ou exclusão de helpers como parte desta fase.

### Automations e scripts

- `automations.yaml` — **97** aliases
- `scripts.yaml` / `scripts_erro500.yaml` — **14** aliases

Dependências diretas encontradas:

- `automations.yaml` não contém `_v19` ou `_v20_2`
- `scripts.yaml` e `scripts_erro500.yaml` não contêm `_v19` ou `_v20_2`

Isso indica que a camada de automações/scritps ativos não consome diretamente entidades de versão de legado ou shadow.

### Dashboards

- `ui-lovelace.yaml` — não contém referências `_v19` ou `_v20_2`
- `.storage/lovelace.teste_4` — contém **59** referências `_v19`
- `.storage` de lovelace adicionais não mostraram `_v20_2` diretas no escaneamento de arquivo principal para dashboards

Conclusão: o dashboard oficial `ui-lovelace.yaml` está limpo de `_v19` e `_v20_2`, mas há artefatos de dashboard armazenado que ainda carregam legado.

### Packages

#### Ativos em produção

- `packages/central_operacional_aliases_v20.yaml`
- `packages/motor_eventos_v20.yaml`
- `packages/motor_timeline_v20.yaml`
- `packages/parametros_operacionais_v20.yaml`
- `packages/status_casa.yaml`
- `packages/alertas_contextuais_v2_corrigido.yaml`
- `packages/energia_contexto.yaml`
- `packages/ha_inicio.yaml`
- `packages/modo_dormir.yaml`
- `packages/carro.yaml`
- `packages/carro_presenca.yaml`
- `packages/ventilador_quarto_maior.yaml`
- `packages/wan_4g_engine_v20.yaml`
- `packages/teste_motor_eventos_v20.yaml`

#### Shadow

- `packages/motor_confianca_v20_2.yaml`
- `packages/motor_contexto_v20_2.yaml`
- `packages/motor_relevancia_v20_2.yaml`

#### Legado / desativados

- `packages/_disabled/status_casa_v19.yaml`
- `packages/_disabled/DESATIVACAO_V19.md`

## Classificação de dependências

### Usado em produção

- `automations.yaml`
- `scripts.yaml`
- `scripts_erro500.yaml`
- `ui-lovelace.yaml`
- `packages/central_operacional_aliases_v20.yaml`
- `packages/motor_eventos_v20.yaml`
- `packages/motor_timeline_v20.yaml`
- `packages/parametros_operacionais_v20.yaml`
- `packages/status_casa.yaml`
- `packages/alertas_contextuais_v2_corrigido.yaml`
- `packages/energia_contexto.yaml`
- `packages/ha_inicio.yaml`
- `packages/modo_dormir.yaml`
- `packages/carro.yaml`
- `packages/carro_presenca.yaml`
- `packages/ventilador_quarto_maior.yaml`
- `packages/wan_4g_engine_v20.yaml`

### Usado apenas em dashboard

- `.storage/lovelace.teste_4` — dashboard de teste/debug com referências `_v19`

### Usado apenas em restore_state / registry

- `.storage/core.entity_registry` — registros de entidades V19 e V20.2
- `.storage/core.restore_state` — histórico de estado de entidades V19 e V20.2

### Candidato a legado

- `packages/_disabled/status_casa_v19.yaml`
- `packages/_disabled/DESATIVACAO_V19.md`
- `.storage/lovelace.teste_4`
- `.storage/core.entity_registry` (entradas V19 e V20.2)
- `.storage/core.restore_state` (entradas V19 e V20.2)

### Candidato a remoção futura

- `packages/_disabled/status_casa_v19.yaml`
- `packages/_disabled/DESATIVACAO_V19.md`
- `.storage/lovelace.teste_4`

> Nota: a remoção de arquivos em `.storage` deve ser realizada apenas após validação com Home Assistant parado e backup completo.

### Dúvida / requer validação manual

- `packages/status_casa.yaml` — contém uma referência `_v19` e é usado em produção; merece revisão de dependência de legado antes de qualquer refatoração.
- `packages/motor_contexto_v20_2.yaml` — possui referências `status_casa` e helpers; deve ser validado quanto ao grau de acoplamento entre shadow e camada oficial.
- `packages/motor_eventos_v20.yaml` e `packages/motor_timeline_v20.yaml` — contêm helpers fortes e referências de configuração operativa; qualquer alteração deve ser revisada manualmente.
- `packages/central_mensagens_corrigido.yaml` e `packages/teste_motor_eventos_v20.yaml` — artefatos de teste e integração que podem exigir contexto operacional específico antes de remoção.

## Riscos encontrados

- Artefatos de `.storage` podem dar falsa impressão de dependência ativa. A limpeza direta de `.storage` é de alto risco sem verificação adicional.
- `packages/status_casa.yaml` mantém um único ponto de legado `_v19` em uma área sensível de governança; essa dependência exige revisão cuidadosa.
- A presença de helpers antigos em automações e pacotes sugere que a migração de controle operacional deve ser faseada.
- A existência de dashboard armazenado `lovelace.teste_4` com `_v19` mostra que há pontos de visibilidade de legado fora do dashboard oficial.

## Entidades críticas

- `sensor.status_casa` e seus derivados de governança
- `sensor.casa_contexto_*` e `sensor.casa_relevancia_contextual_v20_2` como partes do motor shadow
- `sensor.casa_evento_relevante_v20_2` e `sensor.casa_confianca_contextual_v20_2` usados para análise de estabilidade
- `binary_sensor.casa_vazia_v20_2` e `binary_sensor.contexto_noturno_v20_2`

## Itens que não devem ser removidos

- Qualquer arquivo em `packages/` que não esteja explicitamente em `_disabled/`
- `automations.yaml`
- `scripts.yaml`
- `scripts_erro500.yaml`
- `ui-lovelace.yaml`
- `.storage/core.entity_registry`
- `.storage/core.restore_state`

## Candidatos seguros para limpeza futura

- `packages/_disabled/status_casa_v19.yaml`
- `packages/_disabled/DESATIVACAO_V19.md`
- `.storage/lovelace.teste_4` (quando confirmado como dashboard de teste não utilizado)

## Próximos passos recomendados

1. Validar manualmente `packages/status_casa.yaml` e seu único `_v19`.
2. Revisar a função de `packages/motor_contexto_v20_2.yaml` e as dependências de shadow antes de qualquer migração.
3. Documentar consumo real dos helpers antigos em workflows de automação.
4. Manter o dashboard oficial `ui-lovelace.yaml` isolado de `_v19` e `_v20_2`.
5. Não remover `.storage` sem backup completo e Home Assistant parado.

---

> Estado de análise: V20.1D iniciado. Esta etapa é documental e não altera configuração de produção.