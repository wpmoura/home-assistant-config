# Roadmap V20 Consolidado

Data: 2026-05-20
Status: ATIVO

## Objetivo

Consolidar o roadmap documental da Central Operacional V20 e explicitar dependencias com o backlog tecnico.

Este documento nao altera contratos, nao implementa funcionalidades e nao substitui gates.

## Estado consolidado

| Fase | Estado | Dependencia tecnica |
| --- | --- | --- |
| V20.0 | Concluida e congelada | TD-DOC-001 |
| V20.1A | Concluida | TD-DOC-001 |
| V20.1B | Concluida/parcial por dominio | TD-ARCH-001 |
| V20.1C-V20.1K | Auditorias e saneamento documental executados | TD-LEG-001 |
| V20.1N | Homologada como estabilizacao operacional | TD-UX-001 |
| V20.2A | Proximos passos de evolucao contextual de atividades | TD-ARCH-001 |
| V20.2 shadow | Parcial em shadow mode | TD-PERF-001 |
| V20.2C / CSMR | Promocao limitada consolidada documentalmente; implementacao condicionada ao Gate | TD-ARCH-001 / TD-UX-001 |
| V20.3 | Planejamento futuro | TD-UX-001 |
| V21+ | Planejamento futuro | TD-RES-001 |

## Trilhas

### 1. Governance documental

Objetivo: estabilizar fonte da verdade, constituicao e gates.

Dependencias:

- TD-DOC-001

Entregas:

- `docs/governance/source_of_truth.md`
- `docs/governance/constituicao_central_operacional_v20.md`
- `docs/governance/gates_v20.md`

### 2. Legado e auditorias

Objetivo: manter V17/V18/V19 como historico, sem competir com V20.

Dependencias:

- TD-LEG-001
- TD-ARCH-001

Diretriz:

- Auditorias sao fotografia.
- Nenhuma limpeza sem lote pequeno, rollback e gate.

### 3. Motor operacional V20

Objetivo: preservar contratos finais e consolidar atividades formais.

Dependencias:

- TD-LAB-001
- TD-UX-001
- TD-ARCH-001

Diretriz:

- Timeline e event feed continuam contratos protegidos.
- Aumento de linhas deve preservar aliases finais.
- `teste_motor_eventos_v20` precisa de classificacao formal.

### 4. Contexto V20.2

Objetivo: evoluir contexto, relevancia, confianca e correlacao em shadow mode.

Dependencias:

- TD-PERF-001
- TD-ARCH-001

Diretriz:

- V20.2 permanece desacoplada ate promocao formal.
- A regra geral permanece: Context Engine e demais componentes V20.2 continuam em shadow.
- A promocao limitada do CSMR da V20.2C e a unica excecao formal; ela nao promove a V20.2 inteira e nao autoriza implementacao antes do Gate especifico.
- V20.1O permanece autoridade canônica da Timeline e do Event Feed.
- Despacho subordinado: `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md`.
- Nenhuma inteligencia paralela sem classificacao.
- IA deve permanecer opcional.

### 5. Resiliencia operacional

Objetivo: planejar energia/UPS, internet/failover e HA principal/backup.

Dependencias:

- TD-RES-001

Diretriz:

- Priorizar seguranca, rollback e continuidade.
- Nao substituir automacoes criticas sem auditoria.

## Dependencias entre roadmap e debitos

| Debito | Bloqueia/condiciona |
| --- | --- |
| TD-DOC-001 | Encerramento limpo de fases e consolidacao documental |
| TD-LAB-001 | Promocao ou arquivamento de rotinas de teste V20 |
| TD-LEG-001 | Limpeza futura de documentos/referencias antigas |
| TD-ARCH-001 | Migracao de automacoes legadas e eliminacao de decisoes paralelas |
| TD-UX-001 | Evolucao da timeline/event feed |
| TD-PERF-001 | Camadas IA e assistivas futuras |
| TD-RES-001 | Energia/UPS e HA resiliente principal/backup |

## Proximos passos documentais

1. Usar `source_of_truth.md` como indice raiz.
2. Manter documentos antigos sem mover ate fase propria.
3. Tratar backlog tecnico como fila de fases futuras.
4. Aplicar gates antes de encerrar qualquer nova fase.
5. Consolidar documentos duplicados somente em etapa posterior.
