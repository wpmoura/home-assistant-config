# AT-GC-02 — Contrato Canônico de Odômetro/Abastecimento

Data: 2026-08-21
Status: EXECUTADA — AGUARDANDO REVISÃO PARA COMMIT
Classificação: governança de domínio (Gestão do Carro), frente independente
Autoridade: subordinada a `docs/governance/at_gc_00_enquadramento_gestao_carro.md` e `docs/governance/at_gc_01_saneamento_carro.md`
Baseline: commit `681df3d` (AT-GC-00/01)

## Contrato escolhido

Sensor de template *trigger-based* (`sensor.carro_historico_de_abastecimentos`), disparado pelo evento `carro_abastecimento_confirmado`, acumulando registros em `attributes.registros` via `this.attributes.registros | default([], true)` — o mesmo padrão nativo já comprovado em produção por `sensor.casa_timeline_publicacao_ack_v20` (`packages/contrato_publicacao_timeline_v20.yaml`), sem dependência nova (sem SQLite/pyscript/AppDaemon). Diferença deliberada: sem expurgo por tempo/contagem — este é histórico permanente, não uma trilha de ACK transitória.

## Armazenamento e campos

Cada item de `registros` (mais recente primeiro):

`id`, `dedup_key`, `event_at`, `confirmed_at`, `odometer_km`, `liters`, `amount_brl`, `price_per_liter`, `source`, `source_id`, `evidence_ref`, `confidence`, `status`.

Atributo `contrato` autodescreve o schema (`schema_version`, fontes autorizadas/futuras, política de retenção, estados suportados) diretamente na entidade — introspectável por um futuro processador externo sem depender de documentação fora do HA.

## Idempotência

`dedup_key` = `source_id` quando fornecido; senão `md5(source|minuto do evento|odômetro|litros|valor)`. Verificação feita no script canônico *antes* de publicar o evento (leitura de `registros` já existentes) — o acumulador do sensor não reimplementa a checagem, mantendo a separação evidência→observação→validação→confirmado.

## Regras de validação (script `carro_confirmar_abastecimento`)

- odômetro numérico ≥ 0;
- litros > 0; valor > 0;
- odômetro não pode regredir em relação ao último registro confirmado (ou, na ausência de histórico, em relação a `input_number.carro_odometro_ultimo_abastecimento`) — regressão é **rejeitada**, não fabricada/clampada;
- duplicidade (mesma `dedup_key`) é **rejeitada** sem gerar novo evento.

Nenhuma observação automática pode promover odômetro sem passar por esta validação (`source` fica registrado em cada evento; fontes automáticas futuras herdam a mesma validação, sem caminho de bypass).

## Fluxo manual → canônico

`script.carro_registrar_abastecimento` (botão do dashboard, mesmo `entity_id`) agora só lê os helpers atuais e delega para `script.carro_confirmar_abastecimento` (`source: manual`) — ponto único de confirmação, pronto para receber fontes automáticas futuras sem duplicar lógica.

## Testabilidade

Campo `test_mode` no script canônico: executa toda a validação/idempotência e retorna o resultado via `response_variable`, sem publicar evento nem mutar estado. Nenhum botão de teste foi adicionado ao dashboard (evita repetir o padrão do GC-L06).

## Resultado dos Gates

| Gate | Resultado |
|---|---|
| A — Decisão arquitetural | Concluído; investigação do Recorder confirmou que o `include` do `carro.yaml` não restringe o Recorder globalmente (evidência: histórico de outros domínios intacto); nenhum impacto arquitetural imprevisto — prosseguiu sem pausa. |
| B — Núcleo canônico | Implementado em `packages/carro.yaml`: sensor trigger-based, script canônico, wrapper manual, `recorder.include` atualizado. Validado via `yaml.safe_load` antes de cada reload; `template.reload` + `script.reload` aplicados (sem restart). |
| C — Compatibilidade manual | Verificado via 3 chamadas `test_mode: true` (aprovação, rejeição por regressão, rejeição por litros inválidos) + verificação estática isolada (`ha_eval_template`) da lógica de deduplicação. Nenhum abastecimento real foi criado; `script.carro_registrar_abastecimento` nunca foi acionado nesta AT (seu `last_triggered` permanece `2026-04-18`, inalterado). |
| D — Runtime | Entidades novas confirmadas (`sensor.carro_historico_de_abastecimentos`, `script.carro_confirmar_abastecimento`); nenhum erro em `error_log`/`system log` atribuível às mudanças; sensores/automações de manutenção (GC-L01) sem regressão; dashboard, `carro_presenca` e componentes protegidos (CSMR, Timeline compartilhada) bit-a-bit inalterados (hash do dashboard idêntico; timestamps de `carro_presenca` idênticos). |

## Achado durante a implementação

O nome do sensor (`Carro histórico de abastecimentos`) gera `entity_id` com "de" (`sensor.carro_historico_de_abastecimentos`), divergente do `unique_id` escolhido (`carro_historico_abastecimentos`) — a mesma classe de divergência do GC-L01. Diferente do GC-L01, aqui a referência errada foi pega **antes** do commit (verificação de runtime logo após o primeiro reload) e corrigida na mesma execução; `unique_id` foi mantido como está (consistente com a convenção já existente no arquivo) e todas as referências de leitura foram corrigidas para o `entity_id` real.

## Riscos residuais

- `sensor.carro_historico_de_abastecimentos` não tem expurgo — aceitável na escala de um veículo pessoal (dezenas de registros/ano); se o volume mudar de ordem de grandeza no futuro, uma migração de armazenamento deve ser reavaliada (fora de escopo aqui).
- A regra de não-regressão compara apenas contra o último valor conhecido — não detecta saltos "para cima" implausíveis (ex.: erro de digitação que ainda seja maior que o anterior); a AT não introduziu limite arbitrário, por instrução explícita.
- `status: pendente_revisao` está documentado no contrato (`contrato.estados_suportados`) mas nenhum produtor o gera ainda — não há fonte automática nesta AT. É extensão prevista, não pendência funcional.
- Cálculos de consumo (km rodados/km-L/preço-L/R$-km) continuam lendo os helpers legados (`input_number.carro_odometro_ultimo_abastecimento`), não o novo histórico — preservado deliberadamente por compatibilidade (seção 9 da AT), não redesenhado.

## Recomendação

APROVADA COM RESSALVAS — riscos residuais acima são aceitáveis e não bloqueiam uso; commit pendente de revisão do usuário.
