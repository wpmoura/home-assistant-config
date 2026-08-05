# C1 — Saída de Wilson e monitoramento remoto

## Regra oficial e estado atual

O C1 representa a saída de Wilson e a ativação do monitoramento remoto da residência sob sua perspectiva operacional. A decisão depende exclusivamente de `person.wmoura`: a presença de `person.jacira_f_p_moura` é contexto observacional e não bloqueia, cancela ou limita C1.1, C1.2 ou integrações futuras.

O estado `not_home` autoriza o fluxo. `home` o mantém inativo. `unknown`, `unavailable` ou qualquer valor não reconhecido aplicam fallback seguro e não autorizam ações. Ausência de movimento, presença global, trackers de Jacira e sensores shadow não participam da decisão.

Não existe AT-001 formal. As automações legadas `automation.lab_sai` e `automation.carro_iniciar_uso_ao_sair_de_casa` não são alteradas nem reutilizadas.

## Fonte, abstração e harness

O tracker selecionado por `person.wmoura` é `device_tracker.iphone69`, fornecido pelo Companion App. O histórico recente apresenta transições coerentes entre `home`, `not_home` e zonas conhecidas. Outros trackers vinculados são auxiliares ou antigos e não são consultados diretamente pelo C1; a entidade `person.wmoura` permanece a abstração oficial.

O helper de simulação é `input_boolean.teste_v20_2_simular_wilson_ausente`. Quando `input_boolean.teste_v20_2_harness_ativo` está ligado, ele controla o fluxo. Com o harness desligado, somente `person.wmoura` controla o fluxo real.

O único sensor oficial do C1 é `binary_sensor.wilson_ausente_de_casa`, com nome amigável “Wilson Ausente de Casa” e `unique_id: wilson_ausente_de_casa`. Como a V20.2C ainda permanece em branch isolada, sua identidade foi consolidada antes da integração, sem alias, espelho ou contrato de compatibilidade.

```text
Harness ON  → simulação controla Wilson Ausente de Casa
Harness OFF → person.wmoura = not_home autoriza o fluxo
Estado inválido da fonte real → fallback seguro OFF
```

## C1.1 — Luz da Mesa

A automação `v20_2c_saida_teste_desligar_luz_mesa` controla somente `light.smart_lampada_wifi_1`, a Luz Mesa Tuya da área Home Office. `light.luz_led_mesa` é outro dispositivo e permanece fora do fluxo.

Após a transição de Wilson Ausente de Casa para `on`, a automação exige fonte autorizada e luz ligada, registra o início, aguarda `input_number.teste_v20_2_tempo_graca_saida` (15 segundos na homologação), cancela se o sensor efetivo voltar a `off`, revalida a fonte, o sensor e a luz e faz uma única chamada a `light.turn_off`. O modo é `restart`.

As evidências iniciais sobre `light.luz_led_mesa` permanecem apenas como validação histórica do mecanismo: `98271da357915892b0c5b5676b16ebb3`, `b4daeb3ebaad0db90972adb9a485e295`, `dc3f752509fe5d5dd49ae53a875cbdeb` e `7b8cc23cf75175bc7dbc2f0a6826fb71`.

A homologação sobre a entidade correta preserva os traces:

- luz desligada e condição recusada: `26b49c76c5e3eb699f1e1479732c19c8`;
- cancelamento pela simulação: `620ce693594953c944c2ca6b882412a6`;
- desligamento válido e único: `63c75ab8b7708c2f22e6b1bf1a35af23`;
- cancelamento pelo harness: `fd618aa47cfd093134732a2079cbb608`.

Essas evidências homologam graça, cancelamento, revalidação e alvo físico sob o harness. A mudança atual apenas substitui a fonte semântica e não altera o serviço, o alvo ou o tempo.

## C1.2 — Porta da Sala

A Porta da Sala é `binary_sensor.sensor_porta_sala_contact`, Aqara via Zigbee2MQTT/MQTT. `off` significa fechada e `on`, aberta. O push oficial é `notify.mobile_app_iphonewm`.

A automação `v20_2c_saida_teste_alertar_porta_aberta` usa a mesma transição e o mesmo tempo de graça. Se a ausência de Wilson continuar válida e a porta estiver aberta ao final, registra no Logbook e envia exatamente um push. Não altera a porta nem automações legadas; o modo é `restart`.

Evidências preservadas:

- porta fechada, sem alerta: `5d9bce5b6a189c9cbbecb771d99ec62e`;
- cancelamento pela simulação: `c32b50f54843ed8a230fc6e99366e4b3`;
- porta aberta, Logbook e um push confirmado: `46a0f352316a4d974af07494c73847fe`;
- cancelamento pelo harness: `3f43023e94173dfb178bc326d8911ada`.

Essas evidências permanecem válidas como homologação do mecanismo sob harness. A automação legada `automation.central_porta_sala_contextual_v2` foi observada como concorrência esperada e não foi modificada.

## Critérios de aceite

- `person.wmoura = not_home` com harness desligado ativa o sensor com `source: real`.
- `person.wmoura = home` mantém o sensor desligado com `source: real`.
- Fonte real `unknown`, `unavailable` ou não reconhecida mantém o sensor desligado com `source: safe_fallback`.
- `person.jacira_f_p_moura = home` não bloqueia nem cancela C1.1 ou C1.2.
- Harness ligado preserva integralmente os cenários simulados já homologados.
- Retorno de Wilson antes do fim da graça desliga o sensor e cancela as ações pendentes.
- C1.1 preserva luz, graça, cancelamento, revalidação e uma única ação física.
- C1.2 preserva porta, graça, cancelamento, Logbook e push único.
- Nenhuma automação legada é alterada e nenhum recurso futuro é ativado nesta correção.

## Validação observacional

1. Com Wilson em `home`, harness e simulação desligados, confirmar sensor `off`, `source: real` e motivo `wilson_is_home`.
2. Ligar apenas a simulação com harness desligado; confirmar que ela é ignorada e restaurá-la para `off`.
3. Ligar o harness e a simulação; confirmar sensor `on`, `source: simulation` e restaurar ambos para `off`.
4. Confirmar estaticamente que estados reais inválidos produzem fallback seguro.
5. Não executar saída física nesta correção; o primeiro teste real deverá ser coordenado separadamente.

## Recursos futuros mapeados, não implementados

| Recurso | Referência atual | Comportamento futuro esperado |
| --- | --- | --- |
| Push Porta | `notify.mobile_app_iphonewm` | alertar porta aberta após a graça |
| Recovery 4G | `input_boolean.casa_recovery_4g_automatico` | habilitar política operacional quando Wilson sair |
| Failover | `binary_sensor.internet_em_failover_4g` | observar mudança de rota e comunicar exceções |
| Monitoramento de Internet | `sensor.internet_estado_operacional` | ampliar observação remota da conectividade |
| Alertas de Chuva | `sensor.casa_chuva_estado_v20` | avisar condições relevantes sem inferência comportamental |
| Luz da Mesa | `light.smart_lampada_wifi_1` | desligar após graça e revalidação (C1.1) |
| NVR | sem entidade oficial definida | integrar somente em lote futuro e governado |

## C1.3 — Garantia do Push da Porta

A entidade oficial de habilitação do Push Porta é a própria automação homologada C1.2, `automation.v20_2c_teste_alertar_porta_aberta_apos_saida`. Seu estado `on` mantém o fluxo habilitado; `off` desabilita seus gatilhos. O serviço `notify.mobile_app_iphonewm` continua sendo o destino, mas não é uma entidade de habilitação.

A automação `v20_2c_garantir_push_porta_ao_sair` observa exclusivamente `binary_sensor.wilson_ausente_de_casa`, aguarda o mesmo tempo de graça, cancela se a ausência terminar e revalida a abstração. Após a graça, chama `automation.turn_on` apenas quando C1.2 estiver `off` e registra no Logbook somente quando essa correção ocorrer. Se C1.2 já estiver `on`, a execução termina sem serviço redundante e sem falsa correção.

O harness permanece a única fonte de testes deste lote. Os cenários cobrem C1.2 já habilitada, reabilitação automática, cancelamento durante a graça, nova saída após cancelamento e porta fechada sem push. Não há abertura física da porta, mudança no destino do push ou alteração na lógica homologada de C1.2.

### Homologação C1.3 — 2026-08-05

- C1.2 já habilitada: a graça terminou, a condição de estado `off` foi recusada e nenhuma ação de correção foi executada (`5e812f1cf00b761c7dc885c5600d9af9`).
- C1.2 desabilitada: após a graça e a revalidação houve uma chamada a `automation.turn_on` e um registro `logbook.log` (`120aaa6d256639b4737a3f278d080385`).
- Cancelamento durante a graça: a abstração voltou a `off` com 11,73 segundos restantes; C1.2 permaneceu desabilitada e nenhuma correção foi registrada (`86d9d1bb790977a7410a610cdad0ab6c`).
- Nova saída após cancelamento: a graça terminou e houve exatamente uma nova correção com Logbook (`b9f80b66f72208a886b1832ed51a6f45`).
- Compatibilidade com C1.2: com a porta fechada, a condição física foi recusada antes do Logbook e de `notify.mobile_app_iphonewm` (`7669c1bd3e40c63ef6e4734092476655`).

Nenhuma porta foi aberta, nenhum push foi enviado e nenhum erro relacionado à V20.2C foi encontrado no log do sistema. Estado final: C1.2 habilitada, harness e simulação desligados, sensor de ausência desligado e porta fechada.

## Riscos e rollback

O principal risco é uma transição transitória de `person.wmoura`. O tempo de graça e a revalidação final são a proteção inicial; não são introduzidos scoring, quorum, aprendizado ou lógica da V21.

Para rollback operacional, desligar harness e simulação e confirmar ausência de execução pendente. Para rollback de código, reverter somente o commit corretivo e recarregar template, helpers e automações em janela controlada. Não alterar automações legadas.
