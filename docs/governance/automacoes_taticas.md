# Roadmap AT — Automações Táticas

Data de consolidação: 2026-09-04
Status: ATIVO
Classificação: roadmap canônico da Vertical AT, subordinado à Constituição e ao Source of Truth

## 1. Finalidade

Registrar melhorias pequenas, delimitadas e rápidas que utilizem interfaces e entidades existentes sem alterar contratos, motores ou componentes centrais da Central Operacional.

AT não significa ausência de controle. Toda iniciativa passa pelo Gate de Enquadramento de `docs/governance/gates_v20.md`; o rigor do Gate de execução continua proporcional ao risco e ao blast radius.

## 2. Limite entre consumir e alterar

- Consumir uma interface canônica existente não altera o componente consumido.
- Chamar `script.casa_publicar_evento_timeline_v20` com combinação já autorizada de `source` e `event_code` não significa, isoladamente, alterar a Timeline.
- Incluir nova fonte ou evento, alterar allowlist, contrato, deduplicação, persistência, idempotência ou o motor significa alterar componente central e exige `GO SOC`.

## 3. Gate enxuto de enquadramento

Antes da implementação, responder:

1. A melhoria apenas consome interfaces existentes?
2. Modifica contrato, motor ou componente central?
3. O impacto é local ou alcança outros domínios?
4. O rollback é simples e imediato?
5. Existem dúvidas ou dependências não auditadas?

Resultados permitidos:

- `GO AT`: apenas consome interfaces existentes, possui impacto local e rollback simples.
- `GO SOC`: modifica contrato, motor ou componente central, ou possui impacto sistêmico.
- `NO-GO`: faltam evidências para decidir; auditar somente a dúvida antes de implementar.

## 4. Fluxo reduzido

Requisito → Gate de Enquadramento → Implementação → Teste → registro neste roadmap.

Não criar documento ou Gate adicional quando este registro e as evidências existentes forem suficientes.

## 5. Critérios de conclusão

Uma AT pode ser concluída quando:

- requisito e escopo estão definidos;
- implementação e teste foram concluídos;
- resultado foi registrado;
- não existe pendência bloqueante;
- nenhuma condição descoberta durante a execução exige reenquadramento para SOC.

## 6. Estado atual

### Concluído

| ID | Iniciativa | Evidência | Pendência não bloqueante |
| --- | --- | --- | --- |
| AT-001 | Controle automático de energia da dock Time Machine pela conexão Dell P3424WE | Ciclo ON/OFF end-to-end homologado | Validar com carga real após instalação da dock/HD |

### Transferido ao SOC

| Identificação histórica | Iniciativa | Motivo | Destino |
| --- | --- | --- | --- |
| AT-GC-00 a AT-GC-08 | Gestão do Carro | O domínio cresceu para odômetro e abastecimento canônicos, ingestão de imagens, histórico, manutenção, dashboard e integrações centrais | Roadmap SOC; histórico AT-GC preservado |

### Em fechamento

Nenhuma iniciativa identificada no checkpoint de 2026-09-04.

### Em andamento

| ID | Iniciativa | Estado | Próximo Gate |
| --- | --- | --- | --- |
| AT-002 | Inferência observacional "Wilson Deitado" para iluminação do quarto | Implementação local no worktree `feature/at002-wilson-deitado-observacao` — não implantada, não commitada, não mergeada | Gate de implantação (P2 controlado) — homologação de 7 a 14 dias em modo observação antes de qualquer integração com luzes |

### Backlog priorizado

Nenhuma iniciativa identificada no checkpoint de 2026-09-04.

### Dívida técnica

Nenhuma dívida exclusiva da Vertical AT identificada neste checkpoint.

### Dívida de governança

- Incorporar este documento à baseline consolidada; sua formalização anterior permaneceu isolada na branch `docs/governance/at001-tactical-automations`.
- Prefixos históricos como `AT-GC` e `AT-HC` não comprovam, isoladamente, o pertencimento atual ao roadmap AT.

### Futuro / ideias

Novas ideias somente entram aqui após o Gate de Enquadramento resultar em `GO AT`.

## 7. AT-001 — resumo operacional

- MacBook conectado ao Dell P3424WE → webhook local → helper ligado → tomada ligada.
- MacBook desconectado → helper desligado → atraso de 10 segundos → tomada desligada.
- Webhooks permanecem `local_only`; seus UUIDs não devem ser documentados.
- O volume do Time Machine deve ser ejetado no macOS antes da desconexão física.

## 7A. AT-002 — resumo operacional

- **Título**: Inferência observacional "Wilson Deitado" para iluminação do quarto.
- **Classificação**: AT / P2 — operacional controlado, modo observação.
- **Status**: Implementação em branch isolada (`feature/at002-wilson-deitado-observacao`) — não implantada no Home Assistant. `/Volumes/config` (repositório operacional real) não foi alterado.
- **Objetivo**: estimar, apenas para observação e diagnóstico, se Wilson está deitado, sem acionar luzes nem qualquer outra ação, para futuramente subsidiar um Gate de implantação separado.
- **Package criado**: `packages/at002_wilson_deitado_observacao.yaml`, com dois `binary_sensor` template:
  - `binary_sensor.wilson_deitado_candidato` (`unique_id: at002_wilson_deitado_candidato`) — reflexo imediato, sem memória.
  - `binary_sensor.wilson_deitado_estimado` (`unique_id: at002_wilson_deitado_estimado`) — confirma o candidato após permanência contínua.
- **Regra do candidato**: `janela_noturna (23:30–06:10) E input_boolean.wilson_dormindo = on E iphone_conectado_energia`. Não usa movimento da casa, `input_boolean.modo_dormir_ativo` nem `binary_sensor.todos_dormindo` — ambos representam estado coletivo (Wilson e Jacira), não pessoal.
- **Estabilização**: `binary_sensor.wilson_deitado_estimado` usa `delay_on: minutes: 5` nativo do template binary_sensor; desliga imediatamente (sem `delay_off`) quando o candidato deixar de ser verdadeiro.
- **Ausência de ação física**: nenhum dos dois sensores comanda luz, cria automação, publica na Timeline ou cria helper persistente; é puramente diagnóstico.
- **Mapeamento do iPhone**: `iphone_conectado_energia` aceita provisoriamente apenas os literais `Charging`/`Full` de `sensor.iphone_de_wilson_battery_state`; atributo fixo `mapeamento_iphone: "provisorio"` sinaliza que esses valores não foram comprovados no histórico real (14 dias auditados só mostraram `"Not Charging"`). Qualquer outro valor resulta em `false`, nunca em confirmação positiva.
- **Indisponibilidade**: ambos os sensores usam `availability:` — ficam `unavailable` (não `off`) quando `wilson_dormindo` ou o sensor do iPhone estiverem `unknown`/`unavailable`/vazios.
- **Próximo passo**: a integração com as luzes está fora da fase observacional atual; nenhuma ação física está autorizada agora. Após 7 a 14 dias de evidências reais de observação (etapa futura, fora deste worktree), a continuidade da AT-002 será submetida a novo Gate, que decidirá se a integração com as luzes permanece como próxima fase desta mesma AT-002 ou exige novo enquadramento. Essa integração poderá continuar como P2 controlado e não é automaticamente P3.

## 8. Regra de manutenção

- Cada iniciativa possui um único status principal.
- Pendência não bloqueante não impede conclusão, mas deve permanecer visível.
- Mudança de escopo reabre somente o Gate de Enquadramento.
- O identificador histórico nunca é reutilizado nem reescrito após transferência ao SOC.
