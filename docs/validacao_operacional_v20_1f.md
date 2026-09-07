# Validação Operacional V20.1F

## Objetivo

Validar operacionalmente os candidatos de limpeza identificados nas fases V20.1D/V20.1E, sem executar limpeza e sem alterar produção.

Esta etapa não remove entidades, não altera YAML, não altera dashboards, não altera automações e não cria commit.

## Escopo e limites

Validação realizada por inspeção local dos artefatos disponíveis em `/config`:

- `packages/`
- `automations.yaml`
- `scripts.yaml`
- `scripts_erro500.yaml`
- `ui-lovelace.yaml`
- `.storage/lovelace*`
- `.storage/lovelace_dashboards`
- `.storage/core.entity_registry`
- `.storage/core.restore_state`
- `home-assistant_v2.db` em modo somente leitura

Limite importante: a UI do Home Assistant não foi operada diretamente nesta sessão. Os itens de UI abaixo ficam como checklist manual antes de qualquer remoção futura:

- Developer Tools -> States
- Developer Tools -> Template
- Configurações -> Entidades
- Configurações -> Dispositivos
- Dashboards ocultos via UI
- Histórico via UI

## Candidatos avaliados

### 1. `packages/_disabled/DESATIVACAO_V19.md`

Tipo: documentação histórica.

#### Presença

- Arquivo presente em `packages/_disabled/`.
- Não define entidade.
- Não aparece como consumidor técnico em automações, scripts ou dashboards.
- Contém apenas histórico e instruções de desativação da camada V19.

#### Consumo indireto

- `status_casa`: nenhum consumo técnico.
- Templates: nenhum.
- Scripts: nenhum.
- Automações: nenhum.
- `restore_state`: nenhum.
- Registros persistidos: nenhum registro de entidade associado ao arquivo.

#### Checklist manual de UI

- Developer Tools -> States: não aplicável; não cria entidade.
- Developer Tools -> Template: não aplicável.
- Configurações -> Entidades: não aplicável.
- Configurações -> Dispositivos: não aplicável.
- Dashboards ocultos: não aplicável.
- Histórico: não aplicável.

#### Classificação V20.1F

**Validado seguro** do ponto de vista operacional.

Ressalva: se removido futuramente, preservar o conteúdo histórico relevante em `docs/` ou manter o commit V20.1D/E como referência de auditoria.

### 2. `packages/_disabled/status_casa_v19.yaml`

Tipo: pacote YAML histórico/desativado com templates V19.

#### Presença

- Arquivo presente em `packages/_disabled/`.
- Define, dentro do arquivo, sensores e binary sensors V19:
  - `sensor.status_casa_v19`
  - `sensor.casa_score_ativo_v19`
  - `sensor.casa_modo_operacional_v19`
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
- Não há consumo direto de `_v19` em `automations.yaml`, `scripts.yaml`, `scripts_erro500.yaml` ou `ui-lovelace.yaml`.
- `configuration.yaml` usa `homeassistant.packages: !include_dir_named packages/`, portanto o comportamento real de carregamento de subdiretórios deve ser validado na UI/logs antes de qualquer remoção.

#### Consumo indireto

- `status_casa`: não altera `sensor.status_casa`; o arquivo define `sensor.status_casa_v19`.
- Templates: existem templates internos fortes entre entidades V19.
- Scripts: nenhum consumo direto encontrado.
- Automações: nenhum consumo direto encontrado.
- `restore_state`: há estados V19 persistidos em `.storage/core.restore_state`.
- Registros persistidos: há entidades V19 em `.storage/core.entity_registry`.
- Dashboards ocultos: `.storage/lovelace.teste_4` consome várias entidades V19.

#### Histórico local

Consulta somente leitura em `home-assistant_v2.db` encontrou histórico V19 em `states_meta`, incluindo:

- `binary_sensor.casa_tv_ativa_v19`
- `binary_sensor.casa_incidente_ativo_v19`
- `binary_sensor.casa_event_engine_ativo_v19`
- `sensor.status_casa_v19`
- `sensor.casa_score_ativo_v19`
- `sensor.casa_event_feed_v19`
- `sensor.casa_evento_atual_v19`

Achado relevante:

- `binary_sensor.casa_tv_ativa_v19` possui eventos recentes no histórico em 2026-05-19.
- Outros sensores V19 aparecem como `unavailable` em 2026-05-15, sugerindo resíduo ou ciclo de reload/restart ainda não completamente estabilizado.
- Esse achado impede confirmar o pacote como operacionalmente seguro nesta fase.

#### Checklist manual de UI

Antes de qualquer limpeza futura, validar:

- Developer Tools -> States:
  - buscar `_v19`;
  - confirmar se `binary_sensor.casa_tv_ativa_v19` ainda muda estado;
  - confirmar se `sensor.status_casa_v19` está ausente, `unavailable` ou ativo.
- Developer Tools -> Template:
  - avaliar `states('binary_sensor.casa_tv_ativa_v19')`;
  - avaliar `states('sensor.status_casa_v19')`;
  - avaliar se algum template oficial sem versão depende indiretamente de V19.
- Configurações -> Entidades:
  - buscar `_v19`;
  - registrar se entidades estão ativas, restauradas, indisponíveis ou órfãs.
- Configurações -> Dispositivos:
  - confirmar que V19 não está associada a dispositivo real.
- Dashboards ocultos:
  - abrir `teste-4` se possível;
  - confirmar se é usado por alguém ou apenas laboratório histórico.
- Histórico:
  - consultar `binary_sensor.casa_tv_ativa_v19`;
  - confirmar se há eventos após a próxima reinicialização planejada.

#### Classificação V20.1F

**Requer observação**.

Motivo: apesar de não haver consumo direto em automações/scripts/dashboard oficial, há histórico recente de pelo menos uma entidade V19 e registros persistidos ainda presentes.

Não remover até uma validação manual confirmar que as entidades V19 não existem mais como entidades ativas após restart/reload controlado.

### 3. `.storage/lovelace.teste_4`

Tipo: dashboard oculto/laboratório com referências V19.

#### Presença

- `.storage/lovelace_dashboards` registra:
  - `id: teste_4`
  - `title: Laboratório - Casa Inteligente`
  - `url_path: teste-4`
  - `show_in_sidebar: false`
  - `mode: storage`
- `.storage/lovelace.teste_4` contém referências V19.
- `ui-lovelace.yaml` não contém `_v19` nem `_v20_2`.

#### Consumo indireto

- `status_casa`: não consome `sensor.status_casa`; consome `sensor.status_casa_v19`.
- Templates: há vários templates Lovelace usando `states(...)`, `is_state(...)` e `state_attr(...)` com V19.
- Scripts: nenhum consumo direto encontrado.
- Automações: nenhum consumo direto encontrado.
- `restore_state`: o dashboard referencia entidades que também aparecem em `.storage/core.restore_state`.
- Registros persistidos: o dashboard referencia entidades presentes em `.storage/core.entity_registry`.

#### Histórico local

- O dashboard em si não gera histórico operacional.
- Ele aponta para entidades V19 que ainda possuem histórico no recorder.
- Como é `show_in_sidebar: false`, o risco cotidiano é menor, mas ainda há risco de acesso por URL direta `teste-4`.

#### Checklist manual de UI

Antes de qualquer limpeza futura, validar:

- Dashboards ocultos:
  - abrir `/teste-4` ou selecionar o dashboard pela administração da UI;
  - confirmar que ninguém usa a tela como laboratório ativo.
- Developer Tools -> States:
  - confirmar que as entidades V19 exibidas pelo dashboard não são necessárias.
- Histórico:
  - verificar se os sensores exibidos no dashboard continuam atualizando.
- Configurações -> Entidades:
  - confirmar se as entidades V19 do dashboard estão ativas, indisponíveis ou órfãs.

#### Classificação V20.1F

**Requer observação**.

Motivo: dashboard oculto não afeta automações, mas ainda referencia entidades V19 com registros/histórico. Remoção futura deve ser precedida por exportação/migração ou confirmação formal de descarte.

### 4. Entradas V19 em `.storage/core.entity_registry`

Tipo: registro persistido interno.

#### Presença

- Entradas V19 ainda existem em `.storage/core.entity_registry`.
- `disabled_by` aparece como `null` nas entradas observadas, o que reforça a necessidade de validação pela UI em Configurações -> Entidades.

#### Consumo indireto

- `status_casa`: o arquivo também registra `sensor.status_casa`, então qualquer edição manual seria sensível.
- Templates: não define templates, mas mantém identidade de entidades usadas por templates e dashboards.
- Scripts/automações: nenhum consumo direto encontrado.
- `restore_state`: correlacionado com `.storage/core.restore_state`.
- Registros persistidos: é o próprio registro.

#### Classificação V20.1F

**Não remover**.

Motivo: arquivo interno do Home Assistant; não é candidato operacional simples. Só deve ser tratado em fase própria, com HA parado, backup completo e procedimento validado.

### 5. Entradas V19 em `.storage/core.restore_state`

Tipo: estado restaurado interno.

#### Presença

- Estados V19 ainda aparecem em `.storage/core.restore_state`.
- Também existe estado restaurado de `sensor.status_casa`, portanto edição manual é risco direto ao contrato final.

#### Consumo indireto

- `status_casa`: risco por proximidade no mesmo arquivo.
- Templates: não define templates.
- Scripts/automações: nenhum consumo direto encontrado.
- `restore_state`: é o próprio mecanismo.
- Registros persistidos: correlacionado com `core.entity_registry`.

#### Classificação V20.1F

**Não remover**.

Motivo: arquivo interno do Home Assistant; manter fora de qualquer limpeza automática.

## Matriz de decisão V20.1F

| Candidato | V20.1E | V20.1F | Motivo da decisão |
|---|---|---|---|
| `packages/_disabled/DESATIVACAO_V19.md` | seguro remover | validado seguro | Sem consumo técnico ou entidade associada |
| `packages/_disabled/status_casa_v19.yaml` | seguro remover | requer observação | Histórico recente de V19 no recorder e registros persistidos |
| `.storage/lovelace.teste_4` | remover somente após migração | requer observação | Dashboard oculto ainda referencia V19 |
| `.storage/core.entity_registry` entradas V19 | risco alto | não remover | Registro interno sensível |
| `.storage/core.restore_state` entradas V19 | risco alto | não remover | Estado interno sensível |
| `packages/status_casa.yaml` comentário V19 | dependência desconhecida | não remover | Arquivo produtivo sensível; referência é comentário |

## Dependências inesperadas

Foi encontrada dependência/sinal inesperado no histórico:

- `binary_sensor.casa_tv_ativa_v19` ainda possui eventos recentes no recorder em 2026-05-19.

Interpretação provável:

- pode haver entidade V19 ainda viva em memória;
- pode haver efeito de reload/restart incompleto;
- pode haver comportamento de inclusão de subdiretório em `packages/` a confirmar;
- pode ser resíduo persistido, mas o padrão de eventos sugere atualização real até validação em UI.

Esse achado bloqueia a confirmação operacional do pacote `status_casa_v19.yaml` como seguro para remoção imediata.

## Resumo final

- Candidatos seguros confirmados: **1**
- Candidatos reclassificados: **1**
- Candidatos que requerem observação: **2**
- Candidatos marcados como não remover: **3**
- Dependências inesperadas: **1**
- Produção alterada: **não**
- Limpeza executada: **não**
- Commit criado: **não**

## Recomendação para futura V20.2A

Antes de qualquer limpeza, executar uma fase V20.2A de estabilização e observação:

1. Confirmar pela UI se entidades `_v19` ainda existem em States e Entidades.
2. Fazer restart/reload controlado somente se aprovado operacionalmente.
3. Observar por pelo menos um ciclo real de uso da TV se `binary_sensor.casa_tv_ativa_v19` continua atualizando.
4. Confirmar que `sensor.status_casa`, aliases finais e dashboards produtivos permanecem estáveis.
5. Exportar ou descartar formalmente o dashboard oculto `teste-4`.
6. Manter `core.entity_registry` e `core.restore_state` fora de qualquer limpeza automática.

Conclusão: V20.1F não confirma limpeza operacional ampla. A documentação histórica está segura, mas o pacote V19 e o dashboard oculto requerem observação adicional antes de qualquer remoção futura.
