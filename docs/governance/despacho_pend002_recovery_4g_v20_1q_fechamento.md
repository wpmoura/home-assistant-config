# Despacho — PEND-002 — Recovery 4G / V20.1Q — Fechamento (2026-09-11)

## Objetivo

Encerrar a PEND-002: os três cenários residuais da homologação runtime do Recovery Automático do Modem 4G (V20.1Q.1) que permaneciam sem evidência suficiente desde a Ata de Homologação de 2026-07-18/20.

## Contexto

A implementação V20.1Q.1 já estava funcionalmente completa e amplamente homologada — 4 quedas reais controladas (Testes 1, 2, 3 e repetição do Teste 3) comprovaram esgotamento, cooldown, integridade do snapshot, religamento de segurança, erro técnico seguro, restart em ciclo ativo, Timeline e o guard rail `tomada_ja_desligada`. A homologação runtime ficou **suspensa por decisão operacional** (não por bloqueio técnico) com exatamente 3 cenários pendentes:

1. cancelamento pelo operador em ciclo ativo;
2. retorno antes do esgotamento (sucesso em índice intermediário);
3. janela de estabilização igual a zero exercida de fato.

## Saldo inicial e evidências pré-existentes

Registrados em `docs/governance/gates_v20.md` ("Gate corretivo V20.1Q") e `docs/releases/implementation_plan_v20_1q.md` (Ata de Homologação Runtime). Os 12 demais cenários já possuíam evidência operacional suficiente e não foram objeto desta frente.

## Gate P1 — Auditoria de evidências — PASS

Confrontou os três cenários da fila canônica contra toda a evidência documental e o histórico live disponível (janela de 10 dias, limite do recorder). Confirmou que os três continuavam genuinamente sem evidência — nenhum fora comprovado organicamente sem reconciliação documental. Identificou, como achado adicional, a existência de um harness de simulação já publicado em `main` (commits `8695c02` e `dd350b0`, agosto de 2026) capaz de reduzir o risco dos testes pendentes de FÍSICO/REDE para potencialmente LÓGICO CONTROLADO — nunca antes exercitado de ponta a ponta.

## Qualificação da Via A (harness) — PASS — H1 REPRESENTATIVO

Auditoria estática completa do harness, sem executar nenhum dos três cenários:

- **Arquitetura:** o orquestrador (`script.central_orquestrar_recovery_4g`) e o Executor (`script.casa_recovery_4g_executar_tentativa`) são exatamente o mesmo código em produção e em teste. Os únicos pontos de mock são o sinal de entrada (`binary_sensor.backup_4g_operacional_efetivo`, que em modo teste espelha `input_boolean.casa_recovery_4g_sim_backup_operacional` em vez do sensor real) e o alvo de atuação física (`sim_tomada` em vez de `switch.0xa4c1381045aeb344`), comutados por uma variável imutável (`modo_execucao_capturado`) capturada uma única vez no nascimento de cada execução — nunca relida do toggle global durante o ciclo.
- **Isolamento:** os 5 pontos de atuação física do sistema (Executor, religamento de segurança, reconciliação de restart, interrupção por falta de energia, cancelamento por desabilitação) implementam o mesmo padrão de 3 ramos — `producao` → tomada real; `teste` → `sim_tomada`; estado ambíguo → **fail-closed, nenhuma ação física, apenas notificação**. Duas camadas adicionais: automação-tripwire que alarma qualquer mudança real na tomada durante `modo_teste=on`, e bloqueio forçado da automação legada `gestao_modem_4g` durante testes.
- **Matriz comparativa** (produção × harness) cobriu criação/início de ciclo, controle de tentativa, contador, estado do Executor, request_id/deduplicação, retorno de conectividade, estabilização, cancelamento, cooldown, encerramento, Timeline/eventos e ações físicas — concluindo lógica idêntica em todos os aspectos, exceto o alvo físico final (mock por desenho).
- **Resposta à pergunta de suficiência:** cancelamento e estabilização=0 — evidência via harness plenamente suficiente (questões puras de máquina de estados). Retorno intermediário — suficiente para o critério técnico exigido pelos Gates (correção do código), com a ressalva não bloqueante, já registrada desde 2026-07-20, de que a simulação não caracteriza o timing real da operadora/ISP.

## Execução Via A — PASS nos 3 cenários (2026-09-11)

Parâmetros de tempo temporariamente reduzidos para viabilizar execução em janela curta (registrados e restaurados — ver seção Restauração).

### Ciclo 1 — retorno intermediário + estabilização = 0

`request_id=r4g-20260911182637261461`. Tentativa 1 expirou por timeout sem retorno simulado (deliberado). Na **tentativa 2 de 3**, o retorno simulado foi injetado: o sistema disparou `retorno_detectado`, reconheceu `estabilizacao_retorno_minutos=0` e validou **instantaneamente** (`veredito=...:2:validado`), retornando a `ocioso` **sem cooldown** (correto — sucesso não inicia cooldown) e sem que a tentativa 3 chegasse a disparar.

Uma primeira tentativa deste ciclo, com margem de tempo insuficiente da minha parte, resultou em esgotamento completo (todas as 3 tentativas expiraram por timeout antes da injeção do retorno chegar a tempo) — reportado com transparência antes da repetição bem-sucedida. Isolamento físico confirmado também nessa tentativa malformada.

### Ciclo 2 — cancelamento em ciclo ativo

`request_id=r4g-20260911183007444582`, ciclo independente do Ciclo 1. Com a tentativa 1 em `aguardando_validacao` (ciclo genuinamente ativo, `ciclo_em_andamento=on`), `input_boolean.casa_recovery_4g_automatico` foi desligado. Resultado: `veredito=...:1:cancelado_operador`; orquestrador e Executor mortos com segurança (`script.turn_off`); `sim_tomada` religada pelo ramo de reconciliação específico do modo teste; `ciclo_em_andamento` e `tentativa_atual` resetados.

## Isolamento físico — confirmado empiricamente

Histórico consultado de `switch.0xa4c1381045aeb344` (tomada real do modem) e `binary_sensor.backup_4g_operacional` (sensor real de conectividade da casa) para toda a janela de execução (15:20:00–18:31:00 de 2026-09-11, cobrindo os dois ciclos bem-sucedidos e a tentativa malformada): **nenhuma transição de estado em nenhum dos dois, em nenhum momento.** O isolamento se comportou exatamente como a qualificação H1 previu — nenhuma ação física real ocorreu.

## Restauração

Todos os parâmetros temporariamente reduzidos foram restaurados e verificados individualmente ao final:

| Parâmetro | Valor restaurado |
|---|---|
| `max_tentativas` | 2 |
| `estabilizacao_retorno_minutos` | 3 |
| `confirmacao_queda_minutos` | 3 |
| `timeout_validacao_segundos` | 120 |
| `tempo_off_segundos` | 5 |
| `timeout_confirmacao_tomada_segundos` | 5 |
| `cooldown_minutos` | 10 |
| `automatico` | on |
| `modo_teste` | off |
| `sim_backup_operacional` | off |
| `sim_tomada` | on (repouso) |
| `bloqueio_legado_harness` | on (inalterado durante todo o processo) |
| `harness_checkpoint` | idle (barreiras R2/R3 não foram necessárias) |

## Limitação registrada — risco residual não bloqueante

> O harness comprova a resposta da máquina de estados ao retorno intermediário controlado, mas não reproduz o timing ou comportamento probabilístico real da operadora/ISP.

Esta limitação já era conhecida desde a Ata de Homologação de 2026-07-20 (classificada ali como "hipótese em aberto, não achado") e não bloqueia o fechamento técnico da PEND-002.

## Riscos residuais

- Nenhum risco físico ou operacional identificado — isolamento comprovado antes e depois da execução.
- A limitação de timing real acima permanece como conhecimento registrado, não como pendência.

## Conclusão

**PEND-002 — RESOLVIDA.** Saldo residual identificado objetivamente no Gate P1; harness qualificado antes de qualquer execução; três cenários residuais exercitados com PASS; nenhuma ação física real ocorreu; estado operacional restaurado e verificado; nenhum cenário residual conhecido permanece sem evidência bloqueante. **Não há necessidade de novo teste para o fechamento da PEND-002.**

## Referência cruzada

- `docs/pendencias_atuais_central_operacional.md` — PEND-002 movida para "Itens antigos resolvidos ou superados".
- `docs/releases/implementation_plan_v20_1q.md` — seção "Fechamento via harness — PEND-002 (2026-09-11)", histórico original preservado sem alteração.
- `docs/governance/gates_v20.md` — Gate corretivo V20.1Q (evidências pré-existentes, não alterado por este despacho).
