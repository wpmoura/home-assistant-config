# AT-GC-08 — Homologação Final da Gestão do Carro (Ciclo DNG / ODO / Dashboard 2026)

Data: 2026-08-24
Status: **HOMOLOGADA**
Classificação: governança de domínio (Gestão do Carro), frente independente
Autoridade: subordinada a `at_gc_00_enquadramento_gestao_carro.md`, `at_gc_01_saneamento_carro.md`, `at_gc_02_contrato_canonico_abastecimento.md` e `at_gc_07_encerramento_gestao_carro.md`
Baseline anterior: AT-GC-07 (commit `aba0b33`)

## 1. Objetivo desta homologação

Encerrar formalmente, com homologação, o ciclo de trabalho realizado após o fechamento da AT-GC-07, que estendeu a Gestão do Carro sem redesenhar o domínio: suporte a `.DNG`, persistência independente de leituras de odômetro, distinção segura entre ODO e ocorrências não canônicas (TRIP/km/h/km/L), idempotência por evidência, reconciliação tardia sem inferência arbitrária, recuperação do histórico ODO de 2026 e a camada visual completa do dashboard Carro.

Esta homologação **não amplia escopo funcional** além do que já foi implementado, testado e validado visualmente nas rodadas que a precederam.

## 2. Escopo entregue e homologado

1. **Pipeline de ingestão de fotos do carro** — varredura incremental do NAS, checkpoint local, OCR/classificação on-device (`src/scanner.py`, `src/photo_helper.swift`), integração automática com o Home Assistant (`src/integrate.py`, `src/ha_client.py`), execução recorrente via LaunchAgent.
2. **Suporte a `.DNG`** — extensão anteriormente bloqueada por engano (`IRRELEVANT_EXTS`); corrigida e validada com fotos reais do painel.
3. **Distinção segura entre ODO e ocorrências não canônicas** — `find_number_near_label()` corrigido para não confundir "km" puro com `km/h` (velocidade) ou `km/L` (consumo); validado com 15/15 testes controlados sobre casos reais já auditados (falsos-positivos eliminados, um falso-negativo real recuperado). TRIP permanece estruturalmente isolado do odômetro canônico.
4. **Persistência independente de ODO** — uma leitura de odômetro é evidência própria, nunca dependente de haver abastecimento associado (`script.carro_registrar_leitura_odometro`, Fluxo A).
5. **Idempotência por evidência** — `source_id` (sha1 real do arquivo) tornou-se obrigatório para `source: photo_odometer`; a mesma evidência produz sempre o mesmo `dedup_key`, independentemente do caminho de publicação (manual ou automático). Validado por teste de idempotência real antes da limpeza do ledger.
6. **Reconciliação tardia segura** — abastecimentos confirmados sem odômetro são enriquecidos automaticamente apenas quando existe correlação temporal inequívoca (janela de 60 min) com uma leitura real de ODO; nunca por coincidência de data, nunca por inferência.
7. **Histórico ODO 2026 recuperado e publicado** — 31 leituras reais adicionais recuperadas do acervo já inventariado (extensão `.DNG` bloqueada e classificação antiga), publicadas pelo fluxo canônico, sem escrita manual de estado.
8. **Dashboard Carro** — resumo atual (odômetro, último abastecimento, litros, valor, preço/L), indicadores derivados (km rodados, consumo, custo/km, pendências), gráfico de evolução do odômetro, gráfico de preço do combustível (ApexCharts, instalado e configurado nesta AT), histórico de abastecimentos, manutenção com "Km desde óleo" corrigido.

## 3. Estado homologado (evidência quantitativa, confirmada em runtime)

| Item | Valor |
|---|---|
| `input_number.carro_odometro` | 103.602 km |
| `sensor.carro_historico_de_odometro` | 34 leituras únicas de 2026 |
| Primeiro ODO 2026 | 94.846 km — 03/01/2026 |
| Último ODO | 103.602 km — 21/08/2026 |
| Abastecimentos canônicos | 20 |
| Abastecimentos com ODO associado | 11 (1 legado + 10 reconciliados automaticamente) |
| Abastecimentos aguardando ODO | 9 |
| Km desde óleo | 4.720 km |
| Km restantes (troca de óleo) | 5.280 km |
| TRIP/km/h/km/L no histórico ODO | 0 (zero) |

Sequência de odômetro monotonicamente crescente, sem regressão, sem par duplicado (`event_at` + `odometer_km`).

## 4. REVIEW conhecido — não bloqueante

Os dois casos abaixo foram identificados, extraídos corretamente pelo mecanismo real, e **deliberadamente não publicados** por falharem no controle de plausibilidade cronológica. Permanecem como pendência de revisão manual, fora do canônico, e **não bloqueiam** esta homologação:

| Arquivo | Data | Valor extraído | Motivo |
|---|---|---:|---|
| `IMG_7925.DNG` | 27/02/2026 | 97.804 km | Regressão cronológica frente à leitura seguinte válida (97.504 km, 13/03/2026) |
| `IMG_9301.DNG` | 19/06/2026 (noite) | 1.011 km | Incompatível com a leitura da mesma manhã (101.082 km) — OCR incompleto |

## 5. Escopo temporal — 2025 e anos anteriores

A implantação operacional homologada nesta AT cobre o ano-calendário **2026**. Isso é uma decisão de escopo, não um resultado de auditoria de ausência de eventos: **2025 e anos anteriores não foram descartados** e devem ser tratados como **backfill histórico futuro**, em atividade própria, usando eventualmente outras fontes de acervo além do NAS atual — que não contém necessariamente o histórico fotográfico completo anterior a 2026. Ausência de fotos de um período não deve ser interpretada como ausência de eventos reais do veículo naquele período.

## 6. TRIP não é requisito funcional

TRIP (contador parcial/resetável do painel) não é requisito funcional desta AT. Sua extração tem uma limitação de arredondamento decimal conhecida e não corrigida (registrada como item técnico futuro, sem prioridade). TRIP nunca alimenta o odômetro canônico e nunca deve fazê-lo — essa é uma regra estrutural, não uma pendência.

## 7. Base da homologação

Esta homologação foi concedida com base em **resultado visual real conferido pelo usuário diretamente no Home Assistant** — dashboard Carro, gráfico de evolução do odômetro, histórico de abastecimentos e card de manutenção — e não apenas em testes automatizados ou respostas de API. As correções de causa raiz (extrator, idempotência, card de manutenção) foram validadas por teste controlado antes de qualquer publicação real, e o resultado final foi confirmado visualmente antes desta homologação.

## 8. Commits de referência

**Pipeline** (`carro-fotos-ingestao`):
- `1028513`, `9d208ad`, `99d2a6f` — base da ingestão automática (AT-GC anteriores)
- `f490ae8` — suporte a DNG e reconciliação tardia
- `3561fd5` — distinção segura km puro / km/h / km/L (referência principal desta homologação)

**Home Assistant** (`/Volumes/config`):
- `681df3d`, `9461a73`, `aba0b33`, `c7070b9` — baseline canônico (AT-GC anteriores, até AT-GC-07)
- `4c08fb8` — extensão do contrato de correção para `odometer_km`
- `4e0eaa9` — identidade de dedup por evidência + correção do card de manutenção (referência principal desta homologação)

A camada visual do dashboard (`ApexCharts`, cards de resumo/indicadores/histórico) vive em `.storage/lovelace.dashboard_carro`, fora do controle de versão do repositório (legado GC-L05, já registrado na AT-GC-00) — validada por conferência visual direta, não por commit.

## 9. Riscos residuais e pendências registradas (não bloqueantes)

1. 2 casos REVIEW (seção 4), aguardando decisão manual futura.
2. Backfill histórico de 2025 e anteriores (seção 5), fora do escopo operacional atual.
3. Limitação de arredondamento decimal na extração de TRIP (seção 6) — não corrigida, sem impacto no canônico.
4. Risco residual de lock órfão pós-`SIGKILL` já mitigado em AT-GC-H01 (hardening aplicado e homologado anteriormente).

## 10. Registro de encerramento

**Decisão:** HOMOLOGAR.

**Status final: AT-GC-08 — HOMOLOGADA. A Gestão do Carro permanece encerrada como frente, com o estado descrito nas seções 2 e 3 como sua entrega vigente.**

Nenhuma nova evolução funcional é aberta por este documento.
