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

Nenhuma iniciativa identificada no checkpoint de 2026-09-04.

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

## 8. Regra de manutenção

- Cada iniciativa possui um único status principal.
- Pendência não bloqueante não impede conclusão, mas deve permanecer visível.
- Mudança de escopo reabre somente o Gate de Enquadramento.
- O identificador histórico nunca é reutilizado nem reescrito após transferência ao SOC.
