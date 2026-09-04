# Handoff Canônico — Heartbeat HA → Timeline → SmallTV/GeekMagic

**Status: IMPLEMENTADA · HOMOLOGADA · DOCUMENTADA · MERGEADA · ENCERRADA.**

Este documento consolida o estado final da frente, pós-merge, para continuidade futura. Não substitui o histórico detalhado já registrado em `docs/governance/gates_v20.md` (seções "V20.2C — Heartbeat HA → Timeline (Gate de implementação)" e "Carga runtime e homologação real (2026-09-03)") e em `CHANGELOG.md` (entradas `[heartbeat-ha-timeline-implementacao]` e `[heartbeat-ha-timeline-homologacao]`) — é um resumo de handoff, o histórico completo permanece nesses dois arquivos.

## 1. Objetivo da funcionalidade

Fazer a automação já existente de uptime do HA ("Note HA uptime") produzir, além da notificação pessoal já existente ao Watch/iPhone, uma segunda saída operacional: um heartbeat `"🟢 HA ativo"` publicado na Timeline canônica da Central Operacional e, por consequência, visível na SmallTV/GeekMagic.

## 2. Arquitetura final

```
automation.note_ha_uptime
  (use_blueprint: wmoura/Uptime_HA_Hostv2.yaml)
        │
        ├── CANAL PESSOAL (inalterado)
        │     notify.mobile_app_iphonewm — mensagem de uptime formatada
        │
        └── CANAL OPERACIONAL (novo, condicional a publicar_timeline_heartbeat)
              script.turn_on → script.casa_publicar_evento_timeline_v20
              (fire-and-forget, continue_on_error: true)
                     │
                     ▼
              contrato canônico (packages/contrato_publicacao_timeline_v20.yaml)
                     │  valida source/event_code/message/schema_version, dedup
                     ▼
              evento casa_timeline_publicar_canonico_v20
                     │
                     ▼
              motor Timeline (packages/motor_timeline_v20.yaml)
              sensor.casa_evento_publicavel_v20 → sensor.casa_timeline_v20
                     │
                     ▼
              SmallTV/GeekMagic (packages/smalltv_publicacao_v20.yaml)
              automation.casa_smalltv_publicar_evento_v20 → geekmagic.notify
```

Os dois canais são independentes: uma falha na publicação da Timeline nunca impede nem atrasa a notificação pessoal (ela é a primeira ação da sequência, e a segunda ação roda com `continue_on_error`).

## 3. Entidades, scripts e arquivos envolvidos

| Papel | Referência |
|---|---|
| Automação | `automation.note_ha_uptime` |
| Blueprint | `blueprints/automation/wmoura/Uptime_HA_Hostv2.yaml` |
| Script canônico | `script.casa_publicar_evento_timeline_v20` |
| Contrato | `packages/contrato_publicacao_timeline_v20.yaml` |
| Motor Timeline | `packages/motor_timeline_v20.yaml` — sensores `Casa Evento Publicavel V20` e `Casa Timeline V20` |
| Consumidor SmallTV | `packages/smalltv_publicacao_v20.yaml` — `automation.casa_smalltv_publicar_evento_v20` |
| ACK | `sensor.casa_timeline_publicacao_ack_v20` |
| Timeline visível | `sensor.casa_timeline_v20` (atributo `linha_1`) |

## 4. Frequência parametrizável

Único mecanismo de periodicidade: o input `intervalo_horas` já existente no Blueprint (`condition: {{ (now().hour % (intervalo_horas|int(1))) == 0 }}`, disparado por `time_pattern minutes:"0"`). O valor observado em runtime (3 horas) é a configuração atual da instância — **não é hardcode funcional**; nada no código fixa esse número. Nenhum segundo parâmetro de periodicidade foi criado — decisão deliberada para eliminar risco de dessincronia entre o canal pessoal e o operacional.

## 5. Separação canal pessoal × canal operacional

Ação 1 (canal pessoal): mensagem ao Watch/iPhone, **texto byte a byte inalterado** em relação ao Blueprint original. Ação 2 (canal operacional): novo input booleano `publicar_timeline_heartbeat` (default `true`) controla exclusivamente a publicação na Timeline, sem afetar a ação 1.

## 6. Contrato source/event_code/message

```
source:     ha_uptime
event_code: heartbeat
message:    "🟢 HA ativo"   (fixa, cadastrada no contrato — não é texto livre)
```

`request_id`/`session_id` gerados por `md5(context.id ~ ':ha_uptime:heartbeat:<request|session>')`, formatados UUIDv4-like — mesmo padrão já validado em produção por `lavadora_sessao.yaml`/`carro_presenca.yaml`. Cada execução gera identidade nova, nunca derivada de mensagem ou horário.

## 7. Comportamento de deduplicação

Achado do Discovery: o sensor `Casa Timeline V20` deduplicava por texto (`evento_base == anterior_base`), o que faria heartbeats consecutivos idênticos nunca materializar e o contrato esgotar retries com ACK `failed`. Corrigido com uma **exceção governada, escopada exclusivamente a `source=ha_uptime`/`event_code=heartbeat`**: novo atributo `ultimo_request_id_heartbeat_v20` guarda o `request_id` do último heartbeat materializado; a supressão por texto só se aplica quando o `request_id` atual repete esse valor (= retry da mesma transação). Para todos os demais produtores, a deduplicação por texto permanece exatamente como era. Validado por simulação determinística (`ha_eval_template`) e, na parte de identidade transacional, por dados reais de produção (dois `request_id`s distintos, 15:00 e 18:00, ambos `published`).

## 8. Relação com a Timeline

O heartbeat é só mais um produtor autorizado do contrato canônico — não há bypass, não há escrita direta em `sensor.casa_timeline_v20` (confirmado por auditoria: toda referência fora da própria definição do sensor é leitura, `states()`/`state_attr()`).

## 9. Relação com SmallTV/GeekMagic

Consumidora indireta e desacoplada: observa o evento bruto `casa_timeline_publicar_canonico_v20` já publicado pelo contrato, via whitelist mínima própria (`source=ha_uptime`/`event_code=heartbeat` adicionado a `whitelist_smalltv_v20`), com sua própria deduplicação por `request_id` (protege contra retries do produtor sem tratamento especial adicional) e seu próprio kill-switch (`input_boolean.casa_smalltv_habilitado_v20`, preservado). O Blueprint não chama `geekmagic.notify` em nenhum momento.

## 10. Evidências de homologação

- `homeassistant.check_config`: **PASS**
- `script.reload` / `template.reload` / `automation.reload`: **PASS**
- Restart do HA: **não necessário**
- Duas execuções reais, naturais, sem disparo manual, em 2026-09-03:

| Execução | `request_id` | ACK | Timeline (`linha_1`) | SmallTV |
|---|---|---|---|---|
| 15:00 | `dc591f96-7ec7-4314-8dc0-9bc4e209771a` | `published` | `"15:00 🟢 HA ativo"` | disparada (mesma cadeia causal) |
| 18:00 | `ea9bc680-55ba-4179-874b-50484a01ba06` | `published` | `"18:00 🟢 HA ativo"` | disparada (mesma cadeia causal) |

- Confirmação humana do operador: **Watch OK · iPhone OK · Timeline OK · GeekMagic OK.**
- **Limitação explícita:** entre as duas execuções reais ocorreram outros eventos genuínos na Timeline (ex.: `17:59 🧺 Lavagem iniciada`). O cenário "dois heartbeats imediatamente consecutivos sem nenhum evento entre eles" **não foi observado naturalmente em produção** — permanece validado apenas por simulação determinística (item 7). Nenhum teste artificial foi criado para forçar esse cenário.

## 11. Publicação Git

- **PR:** [#14](https://github.com/wpmoura/home-assistant-config/pull/14) — `MERGED`
- **Base:** `feature/v20-2c-contextual-automations`
- **Head:** `feat/ha-uptime-heartbeat-timeline-smalltv`
- **Commit do Heartbeat:** `2059d8a99eae6b86c3cb979791f5badb08b57a8d`
- **Merge commit:** `5c89dc54a1a09606789ce7c781dd594e7ec55fa9`
- **Novo tip de `origin/feature/v20-2c-contextual-automations`:** `5c89dc5...`

## 12. Decisões arquiteturais tomadas

- **Manter o Blueprint** existente (evoluído com uma segunda ação condicional), em vez de migrar para package — decisão do Discovery, mantida sem redesenho.
- **`24596b2`** (commit órfão "docs(health-check): add canonical post-merge handoff") **deliberadamente não publicado** por esta frente — seu conteúdo já está em `main` por outro PR (#3, via `ffb534c`/`6ad1bbf`).
- **Baseline de publicação:** `origin/feature/v20-2c-contextual-automations` (não `origin/main`), comprovada por análise de blob hash e merge-base — `origin/main` é uma linha irmã, estruturalmente divergente, sem superset da árvore funcional desta frente.

## 13. Limitações conhecidas

- Cenário de heartbeats consecutivos sem evento intermediário: validado só por simulação, não por observação real (ver item 10).
- `packages/smalltv_publicacao_v20.yaml` chegou ao git pela primeira vez neste PR, já com todo o baseline SmallTV pré-existente (2026-08-21/22) embutido — não era dívida nova, mas fica registrado como fato de proveniência.

## 14. Dívidas técnicas residuais

1. **Allowlist triplicada** — `source`/`event_code`/`message` duplicados independentemente em `contrato_publicacao_timeline_v20.yaml`, `motor_timeline_v20.yaml` e `smalltv_publicacao_v20.yaml`. Pré-existente ao Heartbeat, não refatorada nesta frente.
2. **Cenário de repetição sem evento intermediário** ainda não observado naturalmente em runtime (item 10/13) — aguardando observação futura, sem ação necessária.
3. **`feature/v20-2c-contextual-automations` muito à frente de `main`** (84 commits próprios desde o merge-base `16ae780`, contra 26 exclusivos de `main`) — dívida de governança estrutural pré-existente ao Heartbeat, identificada durante esta frente mas **não é objeto de correção aqui**; registrada apenas para conhecimento.

## 15. Estado do working tree original

O worktree em `/Volumes/config` (branch `feature/v20-2c-contextual-automations`) permaneceu, durante toda esta frente, com **HEAD local em `4ef2362`** (o commit do Heartbeat antes de qualquer isolamento) — **não foi atualizado** para o novo tip pós-merge (`5c89dc5`), porque nenhuma sincronização (pull/checkout/merge/reset) foi autorizada sobre esse working tree por conter 8 modificações CSMR não commitadas, que permaneceram intocadas do início ao fim:

```
automations.yaml
docs/ARCHITECTURE.md
docs/v20_2c/c1_saida_de_casa.md
docs/v20_2c/plano_tecnico_csmr.md
packages/csmr_dispatcher_integracao_v20_2c.yaml
packages/saude_sistema_analitico.yaml
packages/v20_2c_contextual_automations.yaml
packages/v20_2c_protect_csmr.yaml
```

**Nota para a próxima frente que tocar este worktree:** ele está tecnicamente atrás do remoto (`origin/feature/v20-2c-contextual-automations` já avançou para `5c89dc5`); sincronizá-lo faz parte do escopo dessa próxima frente, não desta.

## 16. Próximos passos

Nenhum obrigatório para esta frente — está encerrada. Sugestões para o futuro, sem urgência:
- Sincronizar o working tree local com o novo tip remoto, preservando as 8 alterações CSMR (matéria da própria frente CSMR, não desta).
- Observar naturalmente, sem forçar, o cenário de heartbeats consecutivos sem evento intermediário.
- Avaliar, em algum momento futuro, consolidar a allowlist triplicada — não urgente, sem impacto funcional atual.
