# Evidência de Campo — Ciclo Físico Real da Lavadora (15/08/2026)

Data do registro: 2026-08-16
Status: EVIDÊNCIA DE CAMPO, SHADOW MODE — NÃO CONSTITUI HOMOLOGAÇÃO PRODUTIVA
Classificação: anexo formal ao despacho de encerramento do M3
Referenciado por: `docs/governance/despacho_lavadora_m3_fechamento.md`

## Finalidade

Registrar que a FSM shadow (`packages/lavadora_sessao.yaml`, M1–M3) foi exercitada com sucesso por um ciclo físico real completo da lavadora em 15/08/2026, posterior ao encerramento do M3 e anterior a qualquer M5/cutover — que continua não executado. Este documento não reabre nem duplica as decisões já formalizadas no despacho do M3; apenas anexa evidência de campo adicional àquela homologação.

## Ressalva obrigatória

> Isto **NÃO** constitui homologação produtiva, pois M5/cutover ainda não foi executado. A FSM permanece em shadow mode e o caminho bruto antigo (`sensor.casa_atividade_operacional_v20` / `motor_timeline_v20`) continua sendo a autoridade produtiva da lavadora.

## Evidência do ciclo

```text
Data: 15/08/2026

session_id:
d3a052cd-bd36-4dc8-89f8-96ed08b7b07f

sessão confirmada (confirming_start -> active):
18:24:41

centrifugação detectada:
19:13:45

última atividade:
19:46:48

fim_candidato_desde:
19:46:54

retorno a idle:
19:57:00
```

Registro adicional:

- 1 sessão física; `session_id` alterado exatamente uma vez (criado em `18:24:41`, retido após `idle`, coerente com EH-3).
- 6 pausas internas (`active -> confirming_end -> active`); maior pausa interna ≈ 5min35s (19:30:12→19:35:47); nenhuma pausa atingiu `Tfim = 10min`.
- 2 cruzamentos de potência acima de `P_centrifugacao = 480W` (732,3W às 19:13:45 e 745,4W às 19:29:02); `centrifugacao_detectada` mudou apenas uma vez, de `off` para `on`, no primeiro cruzamento — o segundo não gerou nova mudança (idempotência confirmada).
- Nenhuma leitura `unknown`/`unavailable` em potência ou corrente durante todo o ciclo.
- EH-1/EH-2 **não foram exercitadas fisicamente** neste ciclo (nenhuma indisponibilidade ocorreu); permanecem validadas apenas pelo Harness sintético do M3.
- EH-3 foi coerente com o comportamento observado: `session_id` reteve a identidade da sessão encerrada após `idle`, sem ambiguidade.
- Nenhuma exceção nova nos logs durante a janela (18:20–20:00 de 15/08); as únicas ocorrências de "lavadora" nos logs do período são as duas já corrigidas e revalidadas no M3.
- Nenhuma regressão observada nos demais domínios (porta sala, chuva, CSMR/carro_presença, banho continuaram publicando normalmente antes, durante e depois da janela).

## Parâmetros efetivos durante o ciclo

```text
P_start = 5W
T_start = 20s
P_silence = 5W
Tfim = 10min
P_centrifugacao = 480W
```

Idênticos aos valores restaurados ao final do M3; não foram alterados pelo M4 nem por este registro.

## Spam legado observado

Durante esta mesma lavagem, o classificador bruto (`sensor.casa_atividade_operacional_v20` → `motor_timeline_v20`) publicou **14 eventos antigos** na Timeline produtiva — 7× `🌧️🧺 Chuva + Máquina ativos` e 7× `🧺 Máquina finalizada`, um par para cada início/fim de pulso que a FSM shadow corretamente tratou como uma única sessão. Isso é **compatível com o fato de M5 ainda não ter ocorrido**: o caminho bruto legado permanece a única autoridade produtiva, e nenhum evento do novo contrato (`🧺 Lavagem iniciada` / `🌀 Centrifugação detectada` / `✅ Lavagem finalizada`) foi ou poderia ter sido publicado, pois a FSM não está conectada a nenhum publicador.

## Resultado formal

```text
HOMOLOGADO COM RESSALVAS — a FSM shadow aderiu integralmente ao contrato
interno M1–M3 em um ciclo físico real complexo.
```

A ressalva é exatamente a registrada na seção "Ressalva obrigatória" acima.

## Estado da frente após este registro

```text
M3 — GO
Evidência física shadow — REGISTRADA
M4 — GO
M5 — PENDENTE / NÃO AUTORIZADO
```

M5 permanece apto a ser **solicitado** (os critérios técnicos já atingidos em M3/M4 não são alterados por este registro), mas continua **NÃO AUTORIZADO** — este documento não abre, autoriza nem executa M5.
