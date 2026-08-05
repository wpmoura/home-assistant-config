# C1 — Saída de casa simulada

## Objetivo e estado atual

Comprovar de forma observacional que o harness consegue simular casa vazia, alterar somente `binary_sensor.casa_efetivamente_vazia` e registrar uma execução laboratorial sem ações físicas.

Não existe uma AT-001 formal no repositório ou no runtime. Foram encontradas as automações legadas `automation.lab_sai` (`LAB - Querida Fui`) e `automation.carro_iniciar_uso_ao_sair_de_casa`; elas não são alteradas nem reutilizadas neste lote.

As pessoas conhecidas são `person.wmoura` e `person.jacira_f_p_moura`. `binary_sensor.casa_presenca_global` foi observado como `unavailable`. Existe `binary_sensor.casa_vazia_v20_2`, mas ele pertence à camada shadow e não pode controlar produção.

## Componentes e lógica

O lote reutiliza os IDs previstos em `docs/harness_testes_shadow_v20_2.md`:

- `input_boolean.teste_v20_2_harness_ativo`;
- `input_boolean.teste_v20_2_simular_casa_vazia`.

Com o harness desligado, `binary_sensor.casa_efetivamente_vazia` permanece `off` por fallback seguro, independentemente da simulação. Com o harness ligado, o sensor reflete o helper de simulação. As fontes reais e shadow aparecem somente como atributos observacionais.

A automação `v20_2c_laboratorio_saida_simulada` registra exclusivamente no Logbook. A Timeline não possui interface pública homologada para publicação avulsa, e o harness existente proíbe publicação na Timeline produtiva. Push também fica fora desta primeira versão, conforme a recomendação do harness de evitar notificações automáticas.

## Critérios de aceite

- Ambos os helpers iniciam e terminam desligados.
- Simulação ligada com harness desligado não altera o sensor efetivo.
- Harness e simulação ligados colocam o sensor efetivo em `on`.
- A transição `off → on` executa a automação uma única vez e gera Logbook com indicação explícita de simulação.
- Nenhum script operacional, luz, tomada, Recovery, NVR, porta, chuva ou presença real é alterado.
- Timeline e push permanecem inalterados.

## Procedimento de teste

1. Confirmar os dois helpers em `off` e o sensor efetivo em `off` com `source: safe_fallback`.
2. Ligar somente a simulação; confirmar que o sensor permanece `off`; desligar a simulação.
3. Ligar o harness e depois a simulação; confirmar sensor `on`, `source: simulation`, um trace e uma entrada no Logbook.
4. Desligar a simulação e depois o harness; confirmar sensor `off` e ausência de efeito residual.

## Rollback

Desligar os dois helpers. Para rollback de código, reverter o commit exclusivo da V20.2C e recarregar os domínios suportados em janela controlada. Nenhuma restauração de estado físico é necessária porque o package não chama serviços físicos.

## Evidências esperadas

Estados e atributos dos três componentes, trace da automação, entrada de Logbook, ausência de erros e confirmação de que Timeline, push e entidades físicas não mudaram.
