# AT-GC-01 — Saneamento Controlado do Legado Carro

Data: 2026-08-21
Status: EXECUTADA — APROVADA COM RESSALVAS
Classificação: governança de domínio (Gestão do Carro), frente independente
Autoridade: subordinada à Constituição, ao Source of Truth e a `docs/governance/at_gc_00_enquadramento_gestao_carro.md`

## Resultado por item

| Item | Status | Resumo |
|---|---|---|
| GC-L01 | **RESOLVIDO** | 9 referências divergentes (`sensor.carro_*_troca_oleo` → `sensor.carro_*_troca_de_oleo`) corrigidas em `packages/carro.yaml`: 2 sensores template, 2 automações de alerta (trigger + mensagem) e 3 entradas do `recorder.include`. Validado em runtime: `sensor.carro_status_troca_de_oleo` passou de `Vencida` (incorreto) para `Ok` (correto). |
| GC-L02 | **RESOLVIDO** | `automation.carro_registrar_abastecimento_pelo_botao` e `automation.carro_registrar_troca_de_oleo_pelo_botao` removidas via API de registro de entidades (`restored: true`, sem config correspondente confirmado por nova varredura antes da remoção). Nenhuma automação foi recriada. |
| GC-L03 | **NÃO CORRIGIDO** (fora de escopo por definição da AT) | Ausência de histórico canônico de abastecimento/manutenção preservada como dívida — tratamento previsto na AT de contrato canônico. |
| GC-L04 | **NÃO REFATORADO** (fora de escopo por definição da AT) | Helpers continuam sendo o estado principal do fluxo de abastecimento; nenhuma alteração estrutural feita. |
| GC-L05 | **NÃO MIGRADO** (fora de escopo por definição da AT) | Dashboard `🚗 Carro` permanece em `.storage`, fora do versionamento de YAML. |
| GC-L06 | **RESOLVIDO** | Card "Teste scripts carro" (itens "Teste abastecimento"/"Teste troca óleo") removido do dashboard via `ha_config_set_dashboard` (API suportada, sem edição direta de `.storage`). Botões reais "Abastecer" e "Troca óleo" preservados intactos. |

## Arquivos efetivamente alterados

- `packages/carro.yaml` — 9 referências de entidade corrigidas (ver diff no despacho de entrega da AT).
- Dashboard `dashboard-carro` (`.storage/lovelace.dashboard_carro`) — 1 card removido via API; arquivo não editado diretamente.

## Validações executadas

- YAML de `packages/carro.yaml` validado localmente (`yaml.safe_load`) antes do reload.
- Reload aplicado apenas nos alvos necessários: `template.reload` e `automation.reload` (sem restart do HA).
- Runtime pós-reload conferido: os 3 sensores de manutenção, as 2 automações de alerta, os 4 sensores de abastecimento/consumo (inalterados), `script.carro_registrar_abastecimento`/`carro_registrar_troca_oleo` (inalterados, nenhum registro fictício disparado) e os componentes de `carro_presenca` (automações de uso, `input_boolean.carro_em_uso`, `counter.carro_viagens`, reconciliador) — todos disponíveis e com `last_triggered`/estado idênticos aos anteriores à intervenção, confirmando ausência de efeito colateral.
- Dashboard reobtido após a escrita: estrutura de 3 seções preservada, apenas o card de teste ausente.

## Riscos residuais

- GC-L03, GC-L04 e GC-L05 permanecem como dívida técnica conhecida, deliberadamente não tratada nesta AT.
- A correção do GC-L01 é uma correção de referência (aponta para o `entity_id` real já existente) — não resolve a causa raiz de por que o `unique_id`/nome diverge do `entity_id` gerado; isso é aceitável para esta AT (preservação de IDs existentes), mas deve ser considerado se o contrato canônico futuro exigir nomes consistentes.
- Nenhum risco novo identificado nos componentes protegidos (`carro_presenca`, CSMR, Timeline compartilhada) — nenhum deles foi tocado.

## Recomendação

AT-GC-01 **APROVADA COM RESSALVAS** (as três dívidas GC-L03/L04/L05, deliberadamente fora de escopo, permanecem registradas para a próxima frente — contrato canônico de odômetro/abastecimento).
