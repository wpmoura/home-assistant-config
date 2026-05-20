# Validacao pos-isolamento V20.1J

## Objetivo

Validar se a movimentacao dos artefatos V19 para `archive/packages_disabled/` impediu o carregamento do template legado, sem alterar arquivos produtivos, sem editar `.storage`, sem remover entidades manualmente e sem criar commit.

## Escopo

Validacao inicial realizada em 2026-05-19 22:17:32 -03 por inspecao local de arquivos, registry/restore em modo somente leitura e recorder em modo somente leitura.

Validacao operacional manual realizada posteriormente no Home Assistant:

- `binary_sensor.casa_tv_ativa_v19`: existe, estado `unavailable`, nao alternou apos teste real da TV.
- `sensor.status_casa`: existe, valor `⚠️ Backup Google com falha`, sem impacto aparente.
- `sensor.casa_timeline`: existe, valor `22:37 📺 TV desligada`, sem impacto aparente.

Conclusao operacional: **o isolamento de `packages/_disabled/` funcionou**. A entidade V19 permaneceu apenas como residuo registrado em estado `unavailable`, sem alternancia real apos teste da TV.

Alteracao desta fase:

- criacao deste documento de validacao.

Nao foram alterados YAML produtivos, dashboards, `.storage`, entidades ou automacoes.

## 1. Validacao de configuracao do Home Assistant

Resultado no shell: **nao validado neste shell**.

Tentativas locais:

- `ha core check`: indisponivel, comando `ha` nao encontrado.
- `hass`: indisponivel, comando `hass` nao encontrado.
- modulo Python `homeassistant`: indisponivel neste ambiente local.

Conclusao: a validacao formal por CLI nao ficou disponivel neste ambiente. A validacao operacional foi realizada manualmente no Home Assistant e nao indicou impacto aparente.

## 2. Reinicio ou reload controlado

Resultado no shell: **nao executado neste shell**.

Motivo:

- o CLI `ha` nao esta disponivel no ambiente atual;
- nao foi usado nenhum mecanismo alternativo de UI/API para reiniciar ou recarregar o Home Assistant;
- a regra desta fase proibe alteracao manual de entidades e edicao de `.storage`.

Conclusao: o controle por CLI nao estava disponivel neste shell. A etapa operacional manual posterior confirmou que o template V19 nao voltou a alternar depois do isolamento e do teste real da TV.

## 3. Verificacao da entidade V19

Entidade investigada:

- `binary_sensor.casa_tv_ativa_v19`

### Presenca em registry/restore

A entidade ainda aparece em registros persistidos:

- `.storage/core.entity_registry`
- `.storage/core.restore_state`

Isto e esperado apos isolamento de YAML, porque a V20.1I nao removeu entidades manualmente nem editou `.storage`.

Validacao manual no Home Assistant:

- existe: sim;
- estado: `unavailable`;
- alternou apos teste real da TV: nao.

Interpretacao: a entidade deixou de operar como template ativo e ficou apenas como residuo persistido.

### Presenca em dashboards ocultos

A entidade ainda aparece em:

- `.storage/lovelace.teste_4`

Isto confirma preservacao de uma dependencia visual historica/laboratorial. Nenhum dashboard produtivo YAML foi alterado nesta fase.

### Historico recente

Consulta somente leitura ao recorder encontrou os eventos mais recentes da entidade:

| Entidade | Estado | Timestamp local |
| --- | --- | --- |
| `binary_sensor.casa_tv_ativa_v19` | `off` | 2026-05-19 21:25:09 |
| `binary_sensor.casa_tv_ativa_v19` | `on` | 2026-05-19 20:18:07 |
| `binary_sensor.casa_tv_ativa_v19` | `off` | 2026-05-19 19:48:09 |

Nao foi observado evento posterior a 2026-05-19 21:25:09 no snapshot local consultado as 22:14-22:17.

A validacao manual posterior confirmou que a entidade nao alternou apos teste real da TV.

Classificacao nesta fase: **validado aprovado**.

## 4. Impacto em `sensor.status_casa`

Consulta somente leitura ao recorder encontrou estado recente de `sensor.status_casa`:

| Entidade | Estado | Timestamp local |
| --- | --- | --- |
| `sensor.status_casa` | `⚠️ Backup Google com falha` | 2026-05-19 20:14:04 |

Conclusao local: **nenhum impacto observado** em `sensor.status_casa` pelos artefatos locais disponiveis.

Validacao manual no Home Assistant:

- existe: sim;
- valor: `⚠️ Backup Google com falha`;
- impacto aparente: nenhum.

## 5. Aliases finais

Entidades finais avaliadas em registry/recorder:

| Entidade | Resultado local |
| --- | --- |
| `sensor.casa_event_feed` | estado recente encontrado no recorder |
| `sensor.casa_contexto_humano` | estado recente encontrado no recorder |
| `sensor.casa_prioridade` | estado recente encontrado no recorder |
| `sensor.casa_score_operacional` | estado recente encontrado no recorder |
| `sensor.casa_timeline` | presente em package/registry; validado manualmente com valor `22:37 📺 TV desligada` |

Conclusao: os aliases finais principais seguem funcionais no escopo validado. `sensor.casa_timeline`, que nao apareceu no snapshot inicial do recorder, foi confirmado manualmente no Home Assistant.

## 6. Greps pos-isolamento

### `packages/`

Resultado: **sem referencias encontradas** para `casa_tv_ativa_v19` em:

- `packages/`
- `automations.yaml`
- `scripts.yaml`
- `ui-lovelace.yaml`

Tambem nao existe mais `status_casa_v19.yaml` dentro de `packages/`.

### `archive/`

Resultado: **preservacao historica confirmada**.

Referencias encontradas em:

- `archive/packages_disabled/status_casa_v19.yaml`
- `archive/packages_disabled/DESATIVACAO_V19.md`

### `.storage`

Resultado: **referencias persistidas esperadas**.

Referencias encontradas em:

- `.storage/lovelace.teste_4`
- `.storage/core.entity_registry`
- `.storage/core.restore_state`

Nenhum arquivo `.storage` foi editado.

## 7. Classificacao V20.1J

| Item | Resultado |
| --- | --- |
| HA validou configuracao? | Validacao por CLI indisponivel neste shell; validacao operacional manual aprovada |
| HA reiniciou/recarregou sem erro? | Controle por CLI indisponivel neste shell; ambiente HA validado manualmente sem impacto aparente |
| Entidade V19 parou de alternar? | Sim; ficou `unavailable` e nao alternou apos teste real da TV |
| Houve impacto em `sensor.status_casa`? | Nao; valor validado como `⚠️ Backup Google com falha` |
| Houve impacto em dashboards? | Nenhum dashboard produtivo alterado; dashboard oculto `teste_4` ainda referencia V19 |

## Recomendacao

**Aprovado para commit documental e de isolamento controlado.**

O isolamento de arquivos esta correto do ponto de vista local e operacional: `status_casa_v19.yaml` saiu de `packages/`, foi preservado em `archive/packages_disabled/`, nao ha mais referencia V19 na arvore carregada por packages e `binary_sensor.casa_tv_ativa_v19` nao alterna mais apos teste real da TV.

Pendencias futuras:

1. manter `.storage/core.entity_registry` e `.storage/core.restore_state` intocados ate uma fase propria de limpeza;
2. revisar o dashboard oculto `teste-4` antes de qualquer remocao futura;
3. tratar entidades V19 `unavailable` como residuos conhecidos, nao como dependencia ativa.
