# Despacho — PEND-018 — Legado "HD Backup" Substituído por Time Machine/Dell (2026-09-11)

## Objetivo

Encerrar a PEND-018: auditoria, higienização e fechamento documental do
mecanismo legado ligado ao helper `input_boolean.hd_backup` e à entidade
física `switch.regua_zigbee_br_l4`.

## Contexto

Entrada original em `docs/auditoria_legado_v20_1c.md` (2026-05-22, Quarentena
C1): as automações `Lab - Liga input_boolean.hd_backup` e
`Lab - Desligar input_boolean.hd_backup` espelhavam (de forma incompleta —
o `off` nunca teve `target` definido) o estado de `switch.regua_zigbee_br_l4`
em `input_boolean.hd_backup`. Nenhum consumidor jamais leu esse helper para
acionar a tomada (nunca houve direção helper → físico).

Entre 2026-05-19 e 2026-09-09, `switch.regua_zigbee_br_l4` foi reaproveitada
para alimentar os discos do Time Machine e passou a ser controlada pelo
mecanismo implementado e homologado em **PEND-017**
(`docs/governance/despacho_pend017_webhooks_macbook_dell_fechamento.md`):

```
LaunchAgent → script dell_p3424we_monitor.sh → webhook HA →
input_boolean.macbook_dell_p3424we_conectado → automação
"Time Machine - Controle tomada pela conexão Dell" → switch.regua_zigbee_br_l4
```

## Sequência de Gates executada

1. **Gate P1 (read-only)** — reconstrução completa do mecanismo via estado
   live, YAML, `git log`, `entity_registry` (incluindo `deleted_entities`),
   `core.restore_state` e histórico do recorder. Achados principais:
   - `input_boolean.hd_backup` removido do `entity_registry` em
     **2026-09-09T20:03:12Z** (`deleted_entities`, `orphaned_timestamp`
     correspondente; último estado real `on` em 2026-09-05).
   - As duas automações originais (`unique_id` `1743452727258` e
     `1743452975044`) não existem mais no `automations.yaml` vigente, mas
     permaneciam como entradas órfãs no `entity_registry`
     (`automation.lab_liga_input_boolean_impressora` /
     `automation.lab_desligar_input_boolean_impressora`), estado
     `unavailable`, `restored: true`.
   - `switch.regua_zigbee_br_l4` (dispositivo Zigbee "Regua Zigbee BR",
     UseeLink 4 tomadas + USB, `zigbee2mqtt`, área `home_office`) segue viva,
     sob controle ativo da automação "Time Machine - Controle tomada pela
     conexão Dell" (habilitada, disparos recentes confirmados).
   - Classificação: **D — SUBSTITUÍDO**.
2. **Gate P2 (higienização controlada)** — após revalidação sem divergência:
   - Removidas do `entity_registry` as duas entradas órfãs
     `automation.lab_liga_input_boolean_impressora` e
     `automation.lab_desligar_input_boolean_impressora` (confirmado, antes de
     remover, que não são as automações reais e ativas
     `automation.liga_input_boolean_impressora` /
     `automation.desliga_input_boolean_impressora`, que têm `unique_id`
     diferente e permaneceram intactas).
   - `switch.regua_zigbee_br_l4` renomeada no `entity_registry` de
     `"HD Backup"` para `"Discos Time Machine"` (`icon` mantido).
   - `emulated_hue.yaml`: comentário do item `switch.regua_zigbee_br_l4`
     corrigido de `# HD Backup` para `# Discos Time Machine` (única linha
     alterada; sem mudança de `hidden`, host, porta ou demais entidades
     expostas).
   - Nenhuma ação física, reload ou restart executado.
3. **Gate P2.1 (documentação + preparação de promoção)** — nota de
   fechamento anexada ao final de `docs/auditoria_legado_v20_1c.md`
   (histórico original preservado, linhas 269-270 não alteradas); este
   despacho criado; isolamento do diff da PEND-018 analisado frente ao
   working tree misto de `/Volumes/config`.

## Estado ao final desta frente

- PEND-018 = **encerrada e higienizada**.
- `input_boolean.hd_backup` = inexistente (confirmado).
- Duas automações órfãs = removidas do `entity_registry` (confirmado).
- `switch.regua_zigbee_br_l4` = viva, nome atual "Discos Time Machine",
  controlada pelo mecanismo Time Machine/Dell (PEND-017), intacto.
- `origin/main`: já contém o mecanismo Time Machine/Dell em
  `automations.yaml` (via PR #22 da PEND-017) e **já não contém** nenhuma
  automação `hd_backup` — ou seja, o estado-alvo de `automations.yaml` já
  está em `main` independentemente desta frente. O único artefato rastreado
  por Git ainda pendente de promoção é o comentário corrigido em
  `emulated_hue.yaml` (`origin/main` ainda tem `# HD Backup` na linha
  correspondente).
- Nenhuma pendência funcional remanescente desta frente.

## Referência cruzada

- `docs/auditoria_legado_v20_1c.md` — entrada original (2026-05-22) e nota
  de fechamento (2026-09-11).
- `docs/governance/despacho_pend017_webhooks_macbook_dell_fechamento.md` —
  implementação e homologação do mecanismo que substituiu o "HD Backup".
