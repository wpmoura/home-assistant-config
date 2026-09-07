# AT-GC-07 — Versionamento e Encerramento da Gestão do Carro

Data: 2026-08-24
Status: EXECUTADA — ENCERRAMENTO FORMAL DA FRENTE
Classificação: governança de domínio (Gestão do Carro), frente independente
Autoridade: subordinada a `docs/governance/at_gc_00_enquadramento_gestao_carro.md`, `at_gc_01_saneamento_carro.md` e `at_gc_02_contrato_canonico_abastecimento.md`
Baseline: commit `aba0b33` (AT-GC-04)

## Objetivo original (recuperado da AT-GC-00)

Consolidar presença/uso, odômetro, abastecimento, consumo, manutenção e ingestão futura por fotografias em um único domínio coerente (`docs/governance/at_gc_00_enquadramento_gestao_carro.md`, seção 3), com histórico estruturado próprio, idempotência, rastreabilidade de origem e distinção clara entre observação e evento confirmado — sem exigir processamento pesado dentro do Home Assistant.

## Entrega funcional

- **Ingestão automática de evidências fotográficas** — varredura incremental de `NAS/Photos Library/<ano>` (`src/scanner.py`), checkpoint SQLite local (`data/checkpoint.db`) por `(path, size, mtime)`, sem reprocessar arquivos inalterados.
- **OCR/classificação on-device** (Swift + Vision, `src/photo_helper.swift`), sem dependência de nuvem.
- **Distinção estrutural** entre `ODOMETER_TOTAL` (único elegível ao contrato canônico), `ODOMETER_TRIP` (registrado só para auditoria, nunca lido por `integrate.py`), `FUEL_CANDIDATE` e `REVIEW` (dado insuficiente/ambíguo, nunca promovido automaticamente).
- **Integração automática com o Home Assistant** via REST (`src/ha_client.py` → `script.carro_confirmar_abastecimento` / `script.carro_registrar_leitura_odometro`), fluxos de odômetro e abastecimento independentes (correlação é enriquecimento opcional, nunca pré-requisito).
- **Histórico canônico de abastecimentos** (`sensor.carro_historico_de_abastecimentos`) e de odômetro (`sensor.carro_historico_de_odometro`), sensores trigger-based com ledger append-only, mesmo padrão já comprovado em `contrato_publicacao_timeline_v20.yaml`.
- **Idempotência em duas camadas**: cache local de tentativas (`integration_attempts`) e `dedup_key` autoritativo no contrato HA — reprocessar um candidato já confirmado nunca duplica.
- **Fallback temporal correto**: hierarquia `event_at_content → file_timestamp → pendente` (`resolve_event_at()`), nunca o horário de processamento como substituto silencioso.
- **Correção controlada de registro já persistido**: extensão mínima do contrato (`correction_of_dedup_key`), permitindo corrigir `event_at`/`evidence_ref` de um registro específico in-place, com verificação de identidade (`liters`/`amount_brl`/`source_id`), sem mutação ampla do ledger.
- **Projeção declarativa do ledger para o estado funcional do dashboard** (correção de rastreabilidade dashboard×ledger): `sensor.carro_ultimo_abastecimento` (trigger-based, seleção por maior `event_at` elegível, nunca por posição física na lista) e reescrita das 4 fórmulas de consumo (`carro_preco_por_litro`, `carro_km_rodados_desde_ultimo_abastecimento`, `carro_consumo_medio_ultimo_abastecimento`, `carro_custo_por_km`) para derivar do ledger em vez dos helpers manuais legados — ausência de dado (`odometer_km` nulo) representada como `unknown`, nunca como `0` fictício.
- **Execução automática recorrente** via LaunchAgent (`com.wilson.carro.foto-ingestao`, `StartInterval: 3600`), credenciais via Keychain, sem hardcode/versionamento de token.
- **Proteção estrutural contra promoção silenciosa de `REVIEW`**: `integrate.py` nunca lê `status=REVIEW`, apenas `ODOMETER_CANDIDATE`/`FUEL_CANDIDATE`.

## Fluxo automático final

```
launchd (1h) → run-ingestao.sh → scanner.py (classificação/checkpoint)
            → integrate.py (Fluxo A odômetro + Fluxo B abastecimento, independentes)
            → ha_client.py (REST) → contrato canônico HA → ledger
```

## Estado operacional na data de encerramento

- Execução automática **ativa** (`FORCE_DRY_RUN=false`, deliberado — ativação autorizada em Gate dedicado e auditada na AT-GC-06);
- LaunchAgent operacional (30 execuções, `last exit code: 0`, sem erro real registrado);
- Sem backlog confirmável pendente — todos os candidatos aptos já processados;
- `IMG_7555.PNG` permanece em `FUEL_CANDIDATE`/pendente (dados insuficientes: `liters: null`) — nunca confirmado automaticamente, por design;
- `input_number.carro_odometro = 98.882 km`, protegido contra regressão em toda a cadeia.

## Integridade do histórico

20 abastecimentos no ledger canônico, `dedup_key` únicos, sem duplicata, ordem cronológica coerente (`2025-10-03` a `2026-04-18`), nenhuma data futura. `IMG_8514.PNG` (`source_id 238493935`) — originalmente persistido com `event_at` igual ao horário de processamento por ausência de fallback — corrigido via o mecanismo de correção controlada; permanece corrigido (`event_at: 2026-04-18T05:46:06`, `event_at_source=file_timestamp`).

## Risco residual — Lock órfão após `SIGKILL`

**Descrição:** o mecanismo de exclusão mútua do wrapper (`mkdir "$LOCKDIR"` + `trap 'rm -rf "$LOCKDIR"' EXIT`) não é acionado por `SIGKILL`/encerramento abrupto do processo — um lock órfão bloquearia execuções subsequentes (`exit 8` indefinidamente) até intervenção manual. Padrão de falha já observado, nesta mesma sessão, em componente irmão (`mount-home-assistant-config.sh`, lock parado por ~3 dias até detecção manual).

**Classificação:** risco operacional de hardening. Não compromete ledger, idempotência ou integridade de dado. Não observado como falha ativa da ingestão atual (nenhum lock órfão presente hoje). Correção proposta (checar idade do lock e removê-lo automaticamente se muito além do intervalo normal) permanece **não implementada**, registrada como item futuro.

## Itens futuros recomendados (não implementados)

1. Hardening do lock do wrapper contra `SIGKILL` (acima).
2. Mecanismo de "aging"/alerta para candidatos `REVIEW` acumulados (172 no momento do encerramento), hoje sem prazo de revisão.
3. Extensão do fallback temporal / âncora de odômetro para o cenário em que o abastecimento mais recente por `event_at` e o abastecimento mais recente com `odometer_km` correlacionado sejam registros diferentes — hoje a projeção usa `sensor.carro_ultimo_abastecimento` como referência única; funcionalmente correto para o backlog atual, mas vale revisitar se o padrão de correlação mudar.

## Recomendação

APROVADA — encerramento formal da AT Gestão do Carro. Riscos residuais são aceitáveis, documentados, e não bloqueiam a operação automática já validada na AT-GC-06.
