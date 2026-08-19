# Despacho de Governança — Fechamento Formal da Homologação Física Pós-Cutover (Lavadora)

Data: 2026-08-19
Status: DECISÃO DE GOVERNANÇA CONSOLIDADA; DOCUMENTAL — NENHUMA ALTERAÇÃO FUNCIONAL, RELOAD, RESTART OU PUBLICAÇÃO AUTORIZADA POR ESTE DESPACHO
Classificação: governança subordinada, frente independente (fora da numeração V20.x)
Autoridade: subordinada à Constituição, ao Source of Truth, a `architecture.md`/`docs/ARCHITECTURE.md` e a `AGENTS.md`

## Finalidade

Registrar como decisão formal de governança o resultado da Homologação Física Pós-Cutover da FSM semântica da lavadora (`packages/lavadora_sessao.yaml`), auditada sobre o primeiro ciclo físico real ocorrido após o cutover (M5/M5.1/M5.2), e encerrar formalmente a frente. Este despacho consolida a baseline M1–M5.2 já registrada em `despacho_lavadora_m3_fechamento.md` e em `AGENTS.md`, soma a evidência de produção real e declara o resultado da homologação. Não reabre, não altera e não estende o escopo funcional já congelado em M1–M5.2.

## 1. Baseline consolidada (M1–M5.2)

```text
M1 — Helpers/parâmetros                    CONCLUÍDO
M2 — FSM shadow                            CONCLUÍDO
EH-1/EH-2/EH-3                             HOMOLOGADAS
M3 — Harness (18 cenários)                 CONCLUÍDO, GO
M4 — Promoção do contrato Timeline         CONCLUÍDO
M5 — Cutover (test_mode=false)             CONCLUÍDO
M5.1 — Guard homeassistant.start           CONCLUÍDO
M5.2 — Remoção de initial:true na origem   CONCLUÍDO
```

Checkpoints já commitados/publicados no remote (`origin/feature/v20-2c-contextual-automations`): `8825bbf` (M1–M3 + despacho + evidência shadow 15/08), `087f615` (M5), `dfcfc67` (M5.1), `bad8e54` (M5.2), `f5ab43a` (AGENTS.md + CHANGELOG.md do cutover). Nenhum arquivo desta baseline foi alterado por este despacho.

## 2. Ciclo físico real auditado

`session_id = 211c29bd-b0bb-474f-87f7-9a818e5f0fa1` — 18/08/2026, 17:35:58 → 19:13:00 (≈1h37min). Primeira lavagem física real ocorrida após o cutover M5.

Auditoria realizada exclusivamente por leitura de estado/histórico/ledger via Home Assistant (HA-MCP), sem qualquer escrita, reload, restart ou execução de Harness.

### 2.1 Linha do tempo da FSM

| Hora (18/08) | Transição | Observação |
|---|---|---|
| 17:35:58 | `idle → confirming_start` | |
| 17:36:18 | `confirming_start → active` | `T_start` ≈ 20 s, conforme `lavadora_sessao_tempo_confirmacao_inicio` |
| 17:36–19:02 | 9 oscilações `active ⇄ confirming_end` | compatível com ruído real de potência de um ciclo físico (lavagem + centrifugação intermitente); nenhuma gerou publicação extra |
| 19:02:11 | última atividade elétrica relevante da sessão | |
| 19:13:00 | `confirming_end → idle` | `Tfim` ≈ 10 min 51 s, dentro da tolerância de agendamento de `lavadora_sessao_tempo_confirmacao_fim` (10 min) |

### 2.2 Publicações canônicas (fonte: `sensor.casa_timeline_publicacao_ack_v20`, ledger de produção)

| event_code | request_id | processed_at | status | test_mode |
|---|---|---|---|---|
| `washing_started` | `b02f9b79-c40c-4d6f-8d3e-9c07972bd882` | 17:36:18.340 | `published` | `false` |
| `spinning_detected` | `05588923-1035-478d-880d-b0a7d13eadbf` | 18:58:01.033 | `published` | `false` |
| `washing_finished` | `82d5362c-144d-48f0-8987-fee7debf720c` | 19:13:00.615 | `published` | `false` |

- Exatamente 1× `washing_started`, exatamente 1× `spinning_detected` (≤1 exigido), exatamente 1× `washing_finished`.
- **Mesmo `session_id`** nas três publicações.
- `script.casa_publicar_evento_timeline_v20` disparou exatamente 3 vezes nessa janela (17:36:18 / 18:58:00 / 19:13:00) — correspondência 1:1 com o ledger; nenhuma chamada adicional, nenhum `rejected`, nenhum `duplicate`.
- Zero mensagens no padrão legado (`"Máquina de lavar ativa"` / `"Máquina finalizada"`) em todo o ledger de produção auditado (janela 11/08–19/08).

### 2.3 Watcher de homologação

`automation.lavadora_homologacao_pos_cutover_watch` (entidade `automation.lavadora_watcher_de_homologacao_pos_cutover`) registrou `last_triggered = 17:36:18.372`, coincidindo com o ACK de `washing_started` — disparo único (mode `single`), sem erro em log de sistema associado a "lavadora".

### 2.4 Classificador bruto legado (kill-switch) e sobrevivência a restart real

`input_boolean.atividade_maquina_lavar_habilitada` apresentou um único registro de estado desde `2026-08-16T11:52:22` (cutover M5) até a auditoria: **sempre `off`**, sem nenhuma reversão para `on`.

Nesse intervalo ocorreu um **restart real do Home Assistant** (evidenciado por `last_changed` sincronizado em ~00:42 de 19/08 em múltiplos helpers restaurando seus últimos valores conhecidos). O kill-switch permaneceu `off` através desse restart.

**Isto encerra formalmente a ressalva registrada em M5/M5.1/M5.2 e em `despacho_lavadora_m3_fechamento.md` (seção 2) de que a persistência pós-restart era apenas "estruturalmente validada, não empiricamente provada"**: o hardening de duas camadas (guard `homeassistant.start` + remoção de `initial: true`) foi exercitado por um restart físico real e se comportou exatamente como projetado.

### 2.5 Ausência de regressão em outros domínios

A automação legada não relacionada `automation.area_de_servico_avisar_jacira_fim_da_lavagem_de_roupas` — fora do escopo da neutralização do M5, que atua sobre `input_boolean.atividade_maquina_lavar_habilitada` — não disparou durante a janela do ciclo (0 entradas de logbook em 72h cobrindo o período). Nenhuma interferência observada em TV, micro-ondas, banho ou demais domínios publicados pela Timeline canônica.

## 3. Critérios de homologação e resultado por critério

| Critério | Resultado |
|---|---|
| Exatamente 1× `washing_started` publicado | ATENDIDO |
| No máximo 1× `spinning_detected` publicado | ATENDIDO (exatamente 1×) |
| Exatamente 1× `washing_finished` publicado | ATENDIDO |
| Mesmo `session_id` nos três eventos | ATENDIDO |
| Ausência de spam legado | ATENDIDO |
| Ausência de duplicidade/rejeição no ledger | ATENDIDO |
| Comportamento coerente de ACK/idempotência | ATENDIDO |
| Ausência de regressão em outros domínios | ATENDIDO |

## 4. Decisão

```text
HOMOLOGADO — SEM RESSALVAS
```

A FSM semântica é a autoridade produtiva definitiva da lavadora. O classificador bruto legado permanece neutralizado e seu estado `off` está agora empiricamente comprovado como resiliente a restart real. A frente da Lavadora está formalmente encerrada nesta baseline.

## 5. Estado final da frente

```text
Diagnóstico                                   CONCLUÍDO
Calibração histórica n=5                      CONCLUÍDA
Desenho FSM                                   CONCLUÍDO
M1 — Helpers/parâmetros                       CONCLUÍDO
M2 — FSM shadow                               CONCLUÍDO
EH-1/EH-2/EH-3                                HOMOLOGADAS
M3 — Harness                                  CONCLUÍDO
M4 — Promoção do contrato Timeline            CONCLUÍDO
M5 — Cutover                                  CONCLUÍDO
M5.1 — Guard homeassistant.start              CONCLUÍDO
M5.2 — Remoção de initial:true na origem      CONCLUÍDO
Restart real / sobrevivência do kill-switch   COMPROVADO EMPIRICAMENTE
Homologação Física Pós-Cutover                HOMOLOGADO — SEM RESSALVAS
```

Não há pendência funcional aberta na frente da Lavadora.

## 6. Pendência não funcional (decisão operacional, não de engenharia)

`packages/lavadora_homologacao_pos_cutover_watch.yaml` permanece ativo no working tree, ainda não commitado. Sua função (notificar a primeira lavagem física pós-cutover para viabilizar esta auditoria) já foi cumprida. Este despacho **não decide** seu destino — permanece pendente decisão explícita e futura entre:

- remover, por ser um mecanismo de propósito único já cumprido; ou
- converter em mecanismo permanente de observabilidade da lavadora.

Nenhuma ação sobre esse arquivo foi tomada por este despacho.

## Anexos

- `docs/governance/despacho_lavadora_m3_fechamento.md` — baseline M1–M3, EH-1/EH-2/EH-3, e evidência de campo em shadow mode (15/08/2026).
- `docs/governance/evidencia_lavadora_ciclo_real_2026_08_15.md` — evidência de campo do ciclo em shadow mode que precedeu este cutover.
