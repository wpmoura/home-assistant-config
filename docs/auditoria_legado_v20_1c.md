# Auditoria de Legado V20.1C

## Objetivo

Documentar o diagnóstico inicial da camada de legado antes de qualquer desativação ou remoção controlada.

O foco desta fase é identificar dependências, classificar componentes e mapear riscos indiretos entre:

- V19 / legado preservado
- V20.1B / camada oficial de produção
- V20.2 / motores em shadow mode
- artefatos de teste e de configuração desativada

## Escopo do diagnóstico

1. Inventário preliminar de pacotes e arquivos em `packages/`.
2. Identificação de helpers (`input_boolean`, `input_number`, `input_select`, `input_text`).
3. Mapeamento de `automations.yaml`, `scripts.yaml` e `scripts_erro500.yaml`.
4. Revisão de dashboards oficiais e `.storage` de Lovelace.
5. Contagem de referências `_v19` e `_v20_2` como indicadores de legado e shadow.

## Inventário preliminar

### 1. Pacotes

Encontrados os seguintes itens de pacotes relevantes:

- `packages/_disabled/status_casa_v19.yaml` — pacote V19 desativado.
- `packages/_disabled/DESATIVACAO_V19.md` — documentação de desativação do legado.
- `packages/central_operacional_aliases_v20.yaml` — aliases operacionais V20.
- `packages/motor_confianca_v20_2.yaml` — motor de confiança V20.2 (shadow).
- `packages/motor_contexto_v20_2.yaml` — motor de contexto V20.2 (shadow).
- `packages/motor_relevancia_v20_2.yaml` — motor de relevância V20.2 (shadow).
- `packages/motor_eventos_v20.yaml` — motor de eventos V20.
- `packages/motor_timeline_v20.yaml` — motor de timeline V20.
- `packages/parametros_operacionais_v20.yaml` — parâmetros operacionais V20.
- `packages/status_casa.yaml` — pacote de governança/status de casa.
- `packages/alertas_contextuais_v2_corrigido.yaml` — alertas contextuais V2 corrigidos.
- `packages/energia_contexto.yaml`, `packages/ha_inicio.yaml`, `packages/modo_dormir.yaml`, `packages/carro.yaml`, `packages/carro_presenca.yaml`, `packages/ventilador_quarto_maior.yaml`, `packages/wan_4g_engine_v20.yaml` — pacotes de suporte e domínios específicos.
- `packages/teste_motor_eventos_v20.yaml` — pacote de teste.

### 2. Helpers

Arquivos de helpers presentes no repositório:

- `input_boolean.yaml`
- `input_number.yaml`
- `input_select.yaml`
- `input_text.yaml`

Estes arquivos são parte do inventário de controle operacional e devem ser considerados ao avaliar a dependência de automações legadas.

### 3. Automações e scripts

- `automations.yaml` — arquivo único de automações principal.
- `automations/` — não foram encontrados arquivos `.yaml` adicionais no diretório para esta auditoria inicial.
- `scripts.yaml` — script principal.
- `scripts_erro500.yaml` — scripts de fallback/erro.

Contagens relevantes:

- `automations.yaml` contém 97 entradas `alias:`.
- `scripts*.yaml` contém pelo menos 14 entradas `alias:`.

### 4. Dashboards

Arquivos de Lovelace sob revisão:

- `ui-lovelace.yaml` — dashboard principal oficial.
- `.storage/lovelace.*` — dashboards armazenados, incluindo:
  - `lovelace.dashboard_alarme`
  - `lovelace.dashboard_baterias`
  - `lovelace.dashboard_carro`
  - `lovelace.dashboard_lixo`
  - `lovelace.dashboard_testes`
  - `lovelace.debug_operacional`
  - `lovelace.lovelace`
  - `lovelace.map`
  - `lovelace.minha_casa`
  - `lovelace.sistema_casa`
  - `lovelace.teste_4`
  - `lovelace.testes_anterior`
  - `lovelace_dashboards`
  - `lovelace_resources`

### 5. Entidades e referências de versão

Referências detectadas:

- `160` ocorrências de `_v19` em `packages/` e `.storage`.
- `195` ocorrências de `_v20_2` em `packages/` e `.storage`.

Observações:

- Não foram encontradas referências `_v20_2` em `ui-lovelace.yaml` no escopo desta busca.
- Referências `_v20_2` aparecem em `.storage` principalmente em `core.entity_registry` e `core.restore_state`, indicando presença de entidades shadow no registro e estado restaurado.
- Referências `_v19` aparecem em `.storage` nas mesmas áreas, além do pacote desativado `packages/_disabled/status_casa_v19.yaml`.

## Achados iniciais

1. O legado V19 está preservado como um artefato controlado, com código de suporte desativado em `packages/_disabled`.
2. A camada V20.2 está presente em shadow, com registros de entidades e estados em `.storage`, mas sem evidência de consumo direto em `ui-lovelace.yaml`.
3. `packages/_disabled/DESATIVACAO_V19.md` reforça o desenho de desativação, portanto a auditoria deve tratar o pacote V19 como histórico, não como ativo de produção.
4. O dashboard armazenado `lovelace.teste_4` já foi identificado como fonte de referências `_v19`; deve ser considerado um artefato de teste/debug e não um dashboard oficial.
5. Automations e scripts ainda são um ponto crítico: 97 aliases em `automations.yaml` significam muitos fluxos a serem auditados para dependências e side-effects.

## Classificação preliminar

### CORE

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

### LEGADO

- `packages/_disabled/status_casa_v19.yaml`
- `.storage/core.entity_registry` com registros de entidades V19
- `.storage/core.restore_state` com estados restaurados de entidades V19
- `lovelace.teste_4` (dashboard de teste com referências V19)

### SHADOW

- `packages/motor_confianca_v20_2.yaml`
- `packages/motor_contexto_v20_2.yaml`
- `packages/motor_relevancia_v20_2.yaml`
- `.storage/core.entity_registry` com registros de entidades V20.2
- `.storage/core.restore_state` com histórico de entidades V20.2

### OBSOLETO / ARTEFATO DE DOCUMENTAÇÃO

- `packages/_disabled/DESATIVACAO_V19.md`
- `packages/teste_motor_eventos_v20.yaml` (artefato de teste)
- dashboards `.storage` de teste e debug que não são parte do arquivo oficial `ui-lovelace.yaml`

## Riscos imediatos identificados

- Dependências indiretas de legacy V19 em sensores e dashboards podem não ser visíveis apenas pelo grep de `_v19`.
- A presença de entidades V19 e V20.2 em `.storage` pode confundir validações se consideradas como configuração ativa.
- Remoção de qualquer pacote legado sem auditoria dos aliases e scripts pode causar regressão em notificações e controles operacionais.
- A ausência de consumo `_v20_2` em `ui-lovelace.yaml` é um sinal positivo, mas deve ser validada em todas as fontes de dashboard oficiais.

## Próximos passos

1. Expandir o inventário para incluir a lista completa de `alias:` e seus destinos de ações.
2. Auditar automações críticas que atualizam helpers (`input_boolean`, `input_number`, `input_text`) usados por V20.1B.
3. Verificar dependência de V19 em dashboards dinâmicos além de `ui-lovelace.yaml`.
4. Classificar pacotes e automações por risco de remoção e visibilidade de side-effect.
5. Preparar a fase de desativação gradual com lotes reversíveis e rollback simples.

---

> Nota: esta auditoria é documental e identificatória. Nenhuma alteração de YAML de produção foi realizada nesta fase.
