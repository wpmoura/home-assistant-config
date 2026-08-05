# C1 — Saída de casa simulada

## Objetivo e estado atual

Comprovar que o harness consegue simular casa vazia e, em C1.1, controlar exclusivamente `light.luz_led_mesa` depois de um tempo de graça cancelável e de uma revalidação integral. A presença real continua fora de escopo.

Não existe uma AT-001 formal no repositório ou no runtime. Foram encontradas as automações legadas `automation.lab_sai` (`LAB - Querida Fui`) e `automation.carro_iniciar_uso_ao_sair_de_casa`; elas não são alteradas nem reutilizadas neste lote.

As pessoas conhecidas são `person.wmoura` e `person.jacira_f_p_moura`. `binary_sensor.casa_presenca_global` foi observado como `unavailable`. Existe `binary_sensor.casa_vazia_v20_2`, mas ele pertence à camada shadow e não pode controlar produção.

## Componentes e lógica

O lote reutiliza os IDs previstos em `docs/harness_testes_shadow_v20_2.md`:

- `input_boolean.teste_v20_2_harness_ativo`;
- `input_boolean.teste_v20_2_simular_casa_vazia`.

Com o harness desligado, `binary_sensor.casa_efetivamente_vazia` permanece `off` por fallback seguro, independentemente da simulação. Com o harness ligado, o sensor reflete o helper de simulação. As fontes reais e shadow aparecem somente como atributos observacionais.

A automação `v20_2c_laboratorio_saida_simulada` permanece exclusivamente observacional. A automação separada `v20_2c_saida_teste_desligar_luz_mesa` exige os dois helpers ligados e a luz ligada, registra o início, aguarda `input_number.teste_v20_2_tempo_graca_saida`, cancela se o harness, a simulação ou o sensor efetivo forem desligados e revalida todos os estados antes de chamar `light.turn_off`.

O helper de graça varia de 5 a 300 segundos e inicia em 15 segundos para homologação. Nenhum parâmetro de iluminação por movimento foi reutilizado porque sua semântica é diferente da confirmação de saída de casa.

## Entidade física e concorrência

O único alvo físico autorizado é `light.luz_led_mesa`, fornecido pela integração Yeelight e associado ao Home Office. As automações legadas de queda/retorno de conectividade podem ligá-la como sinalização. As automações noturnas ligam ou desligam a mesma luz conforme `binary_sensor.movimento_piso_quarto_maior_occupancy`, entre 23:30 e 06:10. O blueprint comercial encontrado controla `light.smart_lampada_wifi_1`, que é outra entidade.

Essas automações não são modificadas. Durante homologação, seus traces e o histórico da luz devem ser observados para distinguir a ação C1.1 de eventual religamento concorrente.

## Critérios de aceite

- Ambos os helpers iniciam e terminam desligados.
- Simulação ligada com harness desligado não altera o sensor efetivo.
- Harness e simulação ligados colocam o sensor efetivo em `on`.
- A transição `off → on` executa a automação uma única vez e gera Logbook com indicação explícita de simulação.
- Nenhum script operacional, tomada, Recovery, NVR, porta, chuva ou presença real é alterado; a única ação física permitida é desligar `light.luz_led_mesa` em C1.1.
- Timeline e push permanecem inalterados.
- Com a luz desligada no início, a automação física não executa.
- Cancelar harness ou simulação durante a graça mantém a luz ligada.
- Com a luz ligada e a simulação mantida, a graça termina, todas as condições são revalidadas e ocorre exatamente um `light.turn_off`.
- A luz nunca é ligada automaticamente pelo harness; a preparação física pertence ao operador.

## Procedimento de teste

1. Confirmar os dois helpers em `off` e o sensor efetivo em `off` com `source: safe_fallback`.
2. Ligar somente a simulação; confirmar que o sensor permanece `off`; desligar a simulação.
3. Ligar o harness e depois a simulação; confirmar sensor `on`, `source: simulation`, um trace e uma entrada no Logbook.
4. Desligar a simulação e depois o harness; confirmar sensor `off` e ausência de efeito residual.
5. Manter a luz desligada, repetir a saída simulada e confirmar que a automação física não inicia.
6. Com ambos os helpers desligados, o operador liga manualmente a luz. Iniciar a simulação e desligá-la antes de 15 segundos; confirmar cancelamento e luz ligada.
7. Repetir a preparação manual, manter a simulação por toda a graça e confirmar um único desligamento e os dois registros de Logbook.
8. Repetir a preparação manual e desligar o harness antes do timeout; confirmar cancelamento e luz ligada.
9. Encerrar com harness e simulação desligados e sem execução pendente.

## Rollback

Desligar os dois helpers e confirmar que a automação física não possui execução pendente. O operador define manualmente o estado final desejado da luz. Para rollback de código, reverter o commit exclusivo de C1.1 e recarregar `input_number` e `automation` em janela controlada; não modificar as automações legadas.

## Evidências esperadas

Estados dos helpers e da luz, valor da graça, traces observacional e físico, Logbook, serviço executado, cancelamentos, ausência de erros e eventuais traces concorrentes sobre a luz.

## Homologação C1.1 — 2026-08-05

- Luz desligada no início: condições físicas não atendidas e nenhuma ação executada (`98271da357915892b0c5b5676b16ebb3`).
- Cancelamento pela simulação: graça interrompida com 14,76 segundos restantes, luz preservada ligada e execução abortada antes de `light.turn_off` (`b4daeb3ebaad0db90972adb9a485e295`).
- Desligamento válido: timeout de 15 segundos, quatro revalidações aprovadas, um único `light.turn_off` e conclusão registrada (`dc3f752509fe5d5dd49ae53a875cbdeb`).
- Cancelamento pelo harness: graça interrompida com 14,79 segundos restantes, luz preservada ligada e execução abortada antes de `light.turn_off` (`7b8cc23cf75175bc7dbc2f0a6826fb71`).
- Nenhuma das cinco automações legadas que referenciam a luz disparou durante a janela dos testes.
- Estado final: harness e simulação desligados, nenhuma execução pendente e estado da luz definido pelo operador.
