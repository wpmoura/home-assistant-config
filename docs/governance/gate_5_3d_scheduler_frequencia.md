# Gate 5.3D — Scheduler e frequência configurável (MOCK only)

## Objetivo

Permitir que o usuário escolha, pela UI do Home Assistant, a frequência
automática do Health Check — sem editar nenhum flow Node-RED — convergindo
manual e automático para o MESMO pipeline já homologado no Gate 5.3C/5.3C.1.
Toda execução automática deste Gate usa coleta real + payload real +
inferência MOCK. Zero chamada Anthropic real.

## Helper de configuração

`input_select.saude_sistema_health_check_frequencia` ("Saúde do Sistema -
Health Check - Frequência automática"), opções fixas e definitivas:
`Desativado`, `1x por semana`, `2x por semana`, `3x por semana`,
`1x por dia`. Default: `Desativado` (seguro — nenhuma execução automática
até o usuário optar). Auditoria prévia confirmou não existir helper
equivalente (único `input_select` pré-existente era `input_select.ciclo`,
da máquina de lavar, sem relação).

## Política determinística de dias e horário

| Frequência | Dias (UTC `getUTCDay`) |
|---|---|
| Desativado | nenhum |
| 1x por semana | segunda (1) |
| 2x por semana | segunda (1), quinta (4) |
| 3x por semana | segunda (1), quarta (3), sexta (5) |
| 1x por dia | todos (0–6) |

Horário fixo: **08:00 America/São_Paulo = 11:00 UTC**. Toda a lógica do
scheduler opera em **UTC** (`getUTCDay`/`getUTCHours`) deliberadamente,
para não depender do fuso horário do container onde o Node-RED roda
(evita a classe de bug "o container está em UTC mas eu assumi horário
local"). Constante `HORA_EXECUCAO_UTC = 11`, documentada no próprio nó de
decisão, fácil de alterar. Não configurável pela UI neste Gate (Seção 5:
sem requisito prévio de horário configurável, evitando complexidade
desnecessária).

## Arquitetura — pipeline único

```
[tick do scheduler, 15/15min]  [input_button press]
        |                              |
  le frequencia REAL             (Gate 5.3C, inalterado)
        |                              |
  le checkpoint REAL                   |
        |                              |
  decide se e devido                   |
        | (sim)                        |
  marca checkpoint + monta             |
  payload {origem:'scheduled', ...}    |
        \_____________________________/
                     |
        gate53b_fn_lock_check (MESMO ponto de entrada)
                     |
        preparing -> router coleta (manual OU scheduled = REAL)
                     |
        coleta real (Gate 5.2A) -> payload real -> calling -> MOCK
                     |
        validando -> validacao contrato -> finalize -> persistencia
```

Nenhuma duplicação: lock, FSM, coleta, payload, executor, validador,
persistência e telemetria são exatamente os mesmos nós já homologados. A
única distinção entre manual e automático é o campo `origem`
(`manual`/`scheduled`) e, para scheduled, os campos adicionais
`frequencia`/`janela` que acompanham o payload até a persistência.

## Anti-duplicidade e checkpoint persistente

`ultima_janela_scheduled_atendida` (atributo em
`sensor.saude_sistema_analitico_status`, evento dedicado
`health_check_scheduled_checkpoint`) guarda a última janela de calendário
("YYYY-MM-DD") em que uma execução scheduled foi **despachada** — marcado
**antes** do lock check, portanto válido independentemente do resultado
(aceita, rejeitada ou falha). Isso implementa o guardrail estrutural da
Seção 20: no máximo 1 tentativa scheduled por janela, sempre.

Por viver no Home Assistant (não em flow-context do Node-RED, que é
memory-only), o checkpoint sobrevive a restart do Node-RED e a restart do
HA (restauração nativa de trigger-based template sensor) — comprovado
estruturalmente pelo próprio mecanismo de leitura (o scheduler sempre lê o
checkpoint do HA a cada tick, nunca de memória local).

**Limitação de design conhecida**: o checkpoint é um valor único (não um
conjunto de janelas atendidas). Isso é suficiente e correto para uso real
(o relógio avança cronologicamente, então só é preciso saber se "hoje" já
foi atendido). Só se torna observável em testes fixture que pulam datas
fora de ordem cronológica — não afeta o comportamento em produção.

## Missed run (execução perdida)

Política conservadora adotada (Seção 13): **não há catch-up**. Se o
Node-RED estiver indisponível durante a janela de 08:00, ela é
simplesmente perdida — o scheduler nunca dispara retroativamente uma
execução para um horário já passado, pois a checagem de hora
(`getUTCHours() === HORA_EXECUCAO_UTC`) só é verdadeira durante a própria
janela avaliada em tempo real. Nenhuma lógica de "atrasado" foi
implementada, deliberadamente, para não gerar chamadas inesperadas.

## Mecanismo de teste acelerado

Um nó dedicado (`gate53d_test_fixture_prep`) injeta `msg.teste_agora`
(timestamp ISO fixture) na MESMA cadeia real de leitura de
frequência/checkpoint — apenas o relógio é simulado; frequência e
checkpoint continuam lidos ao vivo do HA. As opções do
`input_select` de produção permanecem exclusivamente as 5 definitivas;
nenhuma opção de teste foi adicionada à UI.

## Testes executados (todos com MOCK apenas na inferência)

| # | Teste | Resultado |
|---|---|---|
| 1 | Desativado | PASS — `motivo: desativado`, zero execução scheduled |
| 1b | Desativado + manual | PASS — manual continua funcional |
| 2 | 1x/dia (quarta) | PASS — devido, executado |
| 3 | 1x/semana (segunda) | PASS — devido, checkpoint marcado |
| 3b | 1x/semana (mesma janela, repetição) | PASS — `janela_ja_atendida`, bloqueado antes do lock |
| 3c | 1x/semana (terça, dia errado) | PASS — `dia_nao_programado` |
| 4 | 2x/semana (quinta) | PASS — devido, 2ª execução scheduled da semana |
| 5 | 3x/semana (sexta) | PASS — devido, 3ª execução scheduled da semana |
| Hora errada | segunda 14:00 UTC | PASS — `fora_da_hora` |
| 6 | Concorrência manual ativo → scheduled | PASS — scheduled rejeitada, checkpoint marcado mesmo assim |
| 7 | Concorrência scheduled ativo → manual | PASS — manual rejeitado, scheduled preservada |
| 10 | Desativar após ativo | PASS — futuras scheduled cessam; manual continua |
| 11 | Origem | PASS — `scheduled` em todas as automáticas, `manual` nas manuais |
| 12 | Pipeline real por scheduled | PASS — coleta real (12 entidades, 9973 bytes) em toda execução scheduled |
| 13 | Zero Anthropic | PASS |
| 14 | Regressão Node-RED | PASS — 112 nós; Home Office/Reset buttons/Reseta Everthing/Gate 5.2A intactos |
| — | Regressão execução manual pós-scheduler | PASS — mesma coleta/payload/MOCK/validador/persistência, zero Anthropic |

Contadores observados ao final: `contagem_execucoes_total=8`,
`contagem_execucoes_manual=2`, `contagem_execucoes_scheduled=5`
(consistente com o roteiro de testes acima).

## Alterações no Node-RED (sem POST /flows integral)

Único `PUT /flow/gate53b_tab`: 45→58 nós (+13: tick de produção, 2 leituras
`api-current-state` reais, função de decisão, função de checkpoint+
despacho, função de fixture de teste, 6 injects de teste). Router de coleta
e `lock_check`/`finalize` tiveram edições pontuais (condição estendida para
`scheduled`; propagação de `frequencia_execucao`/`janela_execucao`).
Nenhuma outra tab tocada (112 nós totais, Home Office/Reset
buttons/Reseta Everthing/Gate 5.2A inalterados).

## Custo e projeção (estimativa, não compromisso)

Custo Anthropic real deste Gate: **US$ 0,00**. Referência histórica
(Gate 5.2B, US$ 0,036712/execução): 1x/semana ≈ US$0,16/mês;
2x/semana ≈ US$0,32/mês; 3x/semana ≈ US$0,48/mês; 1x/dia ≈ US$1,10/mês —
todas puramente ilustrativas (multiplicação simples), sem qualquer
chamada real realizada para validá-las.

## Confirmações finais

Zero chamadas Anthropic. Zero uso de `ANTHROPIC_API_KEY` real. Zero ação
física. Zero retry automático (scheduler nunca reavalia a mesma janela
após marcá-la, independente do resultado). `sensor.saude_sistema_status`
inalterado. Nenhuma regressão. Nenhum commit, nenhum push.

## Riscos residuais

- Já registrados nos Gates anteriores (cosmético `contrato_ok` como
  string; reload de templates toca timestamp de sensores não relacionados).
- Limitação de design do checkpoint single-value (ver acima) — sem
  impacto em produção, apenas em testes fixture fora de ordem.
- `link call` da coleta mantém timeout de 30s (não testado empiricamente
  neste Gate — apenas o timeout da futura chamada Anthropic foi validado).
