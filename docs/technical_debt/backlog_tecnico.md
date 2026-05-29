# Backlog Tecnico

Data: 2026-05-20
Status: ATIVO

## Objetivo

Registrar debitos tecnicos iniciais a partir do inventario resumido e do gap de fonte da verdade.

Este backlog nao autoriza implementacao, limpeza ou remocao. Cada item exige fase propria e gates.

## Debitos iniciais

| ID | Titulo | Origem | Classificacao | Status | Dependencia |
| --- | --- | --- | --- | --- | --- |
| TD-DOC-001 | Eliminar multiplos candidatos a fonte de verdade | gap_source_of_truth | Documentacao | Aberto | Governance consolidada |
| TD-LAB-001 | Validar `teste_motor_eventos_v20` | inventario_resumido | LAB/Testes | Aberto | Auditoria leve |
| TD-LEG-001 | Remover referencias V17/V18/V19 residuais | gap_source_of_truth | Legado | Aberto | Auditoria e rollback |
| TD-ARCH-001 | Eliminar decisoes paralelas fora da V20 | inventario_resumido | Arquitetura | Aberto | Matriz de automacoes legadas |
| TD-UX-001 | Aumentar quantidade de linhas exibidas na timeline/event feed | inventario_resumido | UX | Aberto | Gate de contrato da timeline |
| TD-PERF-001 | Controle explicito liga/desliga camada IA | gap_source_of_truth | Governanca/IA | Aberto | Constituicao e UI futura |
| TD-RES-001 | Gerenciamento inteligente UPS + HA principal/backup | inventario_resumido | Resiliencia | Aberto | Energia/UPS e arquitetura HA |

## Detalhamento

### TD-DOC-001 - Eliminar multiplos candidatos a fonte de verdade

Problema: a fonte da verdade esta distribuida entre `AGENTS.md`, `docs/ROADMAP.md`, `docs/ARCHITECTURE.md`, changelogs e pendencias.

Resultado esperado: familia `governance` passa a definir precedencia; documentos antigos permanecem como historico.

### TD-LAB-001 - Validar `teste_motor_eventos_v20`

Problema: package de teste aparece como `MIGRADO_V20`, mas seu papel operacional precisa ser confirmado.

Resultado esperado: classificar como teste ativo, harness, auditoria ou candidato a arquivamento.

### TD-LEG-001 - Remover referencias V17/V18/V19 residuais

Problema: documentos e residuos antigos ainda aparecem como memoria historica e podem competir com a leitura atual.

Resultado esperado: manter referencias em auditorias, arquivar regras antigas e impedir uso como fonte atual.

### TD-ARCH-001 - Eliminar decisoes paralelas fora da V20

Problema: automacoes antigas e rotinas legadas podem decidir ou notificar fora da consciencia V20.

Resultado esperado: classificar cada decisao paralela como migrada, compativel, legado ativo ou candidata a remocao.

### TD-UX-001 - Aumentar linhas da timeline/event feed

Problema: a exibicao da timeline/event feed pode nao representar volume suficiente de eventos.

Resultado esperado: ampliar exibicao sem quebrar contrato dos sensores finais.

### TD-PERF-001 - Controle explicito liga/desliga camada IA

Problema: IA deve ser opcional e desacoplada, mas precisa de controle explicito quando existir camada assistiva.

Resultado esperado: estado deterministico funcional com IA desligada e controle visivel futuro.

### TD-RES-001 - Gerenciamento inteligente UPS + HA principal/backup

Problema: energia, UPS e HA principal/backup exigem desenho resiliente dedicado.

Resultado esperado: plano de resiliencia com fallback, prioridades, rollback e gates proprios.

## Regras de uso

- Nenhum debito pode ser resolvido sem fase propria.
- Nenhum debito autoriza edicao de YAML por si so.
- Nenhum debito autoriza remocao de legado sem auditoria.
- Todo debito resolvido deve apontar evidencia e gate final.
