# C1 — Saída de casa simulada

## Objetivo e estado atual

Comprovar que o harness consegue simular casa vazia e, em C1.1, controlar exclusivamente a Luz da Mesa `light.smart_lampada_wifi_1` depois de um tempo de graça cancelável e de uma revalidação integral. A presença real continua fora de escopo.

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

Existem dois dispositivos distintos. `light.luz_led_mesa` é a Luz LED da Mesa, uma fita Yeelight usada por sinalizações de conectividade e segurança noturna; ela foi usada incorretamente no primeiro ciclo C1.1. O alvo correto é `light.smart_lampada_wifi_1`, friendly name “Luz Mesa”, dispositivo Tuya “Smart Lâmpada Wi-Fi”, área Home Office.

A entidade correta é ligada e desligada pelas rotinas legadas de Home Office e por duas automações baseadas em `binary_sensor.sensor_movimento_quarto_maior_occupancy`, com janelas 08:30–19:00 e 17:30–22:30. O histórico confirmou períodos em `on` associados ao uso da mesa de trabalho, incluindo permanência entre 21:42 e 23:20 em 2026-08-04.

Essas automações não são modificadas. Durante homologação, seus traces e o histórico da luz devem ser observados para distinguir a ação C1.1 de eventual religamento concorrente.

## Critérios de aceite

- Ambos os helpers iniciam e terminam desligados.
- Simulação ligada com harness desligado não altera o sensor efetivo.
- Harness e simulação ligados colocam o sensor efetivo em `on`.
- A transição `off → on` executa a automação uma única vez e gera Logbook com indicação explícita de simulação.
- Nenhum script operacional, tomada, Recovery, NVR, porta, chuva ou presença real é alterado; a única ação física permitida é desligar `light.smart_lampada_wifi_1` em C1.1.
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

As evidências abaixo permanecem válidas como homologação do mecanismo de graça, cancelamento e revalidação, mas não homologam o alvo físico real porque foram executadas sobre a Luz LED da Mesa `light.luz_led_mesa`:

- Luz LED desligada no início: condições físicas não atendidas e nenhuma ação executada (`98271da357915892b0c5b5676b16ebb3`).
- Cancelamento pela simulação: graça interrompida com 14,76 segundos restantes e execução abortada antes de `light.turn_off` (`b4daeb3ebaad0db90972adb9a485e295`).
- Desligamento do alvo incorreto: timeout de 15 segundos, quatro revalidações aprovadas e um único `light.turn_off` sobre `light.luz_led_mesa` (`dc3f752509fe5d5dd49ae53a875cbdeb`).
- Cancelamento pelo harness: graça interrompida com 14,79 segundos restantes e execução abortada antes de `light.turn_off` (`7b8cc23cf75175bc7dbc2f0a6826fb71`).

Após a identificação inequívoca da Luz da Mesa Tuya, os quatro testes foram repetidos sobre `light.smart_lampada_wifi_1`:

- Luz correta desligada no início: a condição física falhou e nenhuma ação foi executada (`26b49c76c5e3eb699f1e1479732c19c8`).
- Cancelamento pela simulação: a graça foi interrompida com 12,87 segundos restantes, a execução foi abortada antes de `light.turn_off` e a luz correta permaneceu ligada (`620ce693594953c944c2ca6b882412a6`).
- Desligamento válido: o timeout de 15 segundos terminou, as quatro revalidações foram aprovadas e houve exatamente um `light.turn_off` com alvo `light.smart_lampada_wifi_1` (`63c75ab8b7708c2f22e6b1bf1a35af23`).
- Cancelamento pelo harness: a graça foi interrompida com 12,89 segundos restantes, a execução foi abortada antes de `light.turn_off` e ambas as luzes permaneceram ligadas (`fd618aa47cfd093134732a2079cbb608`).

O Logbook registrou os três inícios aplicáveis e somente uma conclusão, vinculados à entidade correta. O histórico de `light.luz_led_mesa` não apresentou transição durante os Testes 2 e 3; no Teste 4 ela permaneceu ligada, com o mesmo `last_changed` anterior ao teste. Nenhuma das quatro automações legadas associadas à Luz da Mesa disparou durante a homologação e não foram encontrados erros de sistema relacionados à V20.2C.

Estado final: harness e simulação desligados, nenhuma execução pendente, Luz da Mesa e Luz LED da Mesa ligadas conforme preparação manual do operador. O C1.1 está homologado sobre o alvo físico correto; a evidência anterior permanece preservada apenas como validação histórica do mecanismo no alvo incorreto.
