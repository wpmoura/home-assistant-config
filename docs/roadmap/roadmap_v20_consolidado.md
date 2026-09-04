# Roadmap SOC — Sistema Operacional Casa

Data de consolidação: 2026-09-04
Status: ATIVO

## Objetivo

Consolidar o roadmap de desenvolvimentos complexos da Central Operacional e explicitar dependencias com o backlog tecnico.

Este documento nao altera contratos, nao implementa funcionalidades e nao substitui gates.

Iniciativas pequenas e delimitadas pertencem ao roadmap AT em `docs/governance/automacoes_taticas.md`. O enquadramento obrigatório está em `docs/governance/gates_v20.md`.

## Estado operacional auditado

### Concluído

| Frente | Evidência consolidada | Pendência não bloqueante ou dívida separada |
| --- | --- | --- |
| Health Check | Implementado, homologado, documentado e mergeado em `main`; entrada em operação normal e scheduler diário homologados | Node-RED não versionado; integração semântica com Timeline adiada |
| CSMR — baseline publicada | Gates até I5B homologados; V20.2C funcionalmente concluída | I4B.2: evidência operacional natural não bloqueante |
| Lavadora/FSM | FSM, Harness, contrato, cutover, restart e ciclo físico real homologados sem ressalvas | Destino do watcher pós-cutover |
| Heartbeat HA → Timeline → SmallTV | PRs funcional e documental mergeados; runtime homologado | Allowlist replicada em três pontos |
| Gestão do Carro — baseline AT-GC | AT-GC-00 a AT-GC-08 homologada; histórico AT preservado | Domínio transferido ao SOC e ainda possui backlog funcional |

### Em fechamento

Nenhuma frente identificada neste checkpoint.

### Em andamento

| Frente | Condição | Próxima necessidade |
| --- | --- | --- |
| Recovery 4G | Homologação suspensa por decisão operacional; implementação permanece válida | Retomar somente os cenários pendentes definidos no Gate V20.1Q |
| CSMR — alterações locais posteriores | NO-GO para publicação | Auditar os oito arquivos locais preservados no working tree original |
| V20.2E — Uso do carro na Timeline | Implementação e correções existentes; núcleo do contrato e ciclo real possuem evidências, mas o Gate ainda registra homologação runtime complementar pendente | Auditar estado local/publicado e executar somente a cobertura residual necessária |
| V20.2 shadow — motores contextuais | Implementação parcial em paralelo, sem promoção geral para produção | Consolidar quais lotes continuam ativos e quais são apenas experimentais antes de novo avanço |

### Backlog priorizado

| Frente | Escopo conhecido | Próximo Gate |
| --- | --- | --- |
| Gestão do Carro — entrada/saída em zonas conhecidas | Casa da Fernanda, Casa da Camila e demais zonas cadastradas; zona Casa permanece com o CSMR | Auditar entidade observada, contrato, GPS, idempotência, sobreposição e mudanças cadastrais |

### Dívida técnica

- Allowlist de eventos replicada no contrato, motor Timeline e SmallTV.
- Lógica principal do Health Check no Node-RED não está versionada no Git.
- Pendências residuais da homologação Recovery 4G permanecem registradas no Gate próprio.

### Dívida de governança

- Branches `main` e `feature/v20-2c-contextual-automations` possuem histórias e documentação divergentes.
- O working tree original possui oito alterações CSMR ainda sem classificação.
- Política de handoffs definida; permanece pendente incorporar seletivamente o handoff do Health Check existente em `main` e sanear os títulos/localizações históricos sem migração em massa.
- Status antigos conflitantes precisam de saneamento controlado sem apagar evidências históricas.
- A política de prompts `P1/P2/P3` está definida; acompanhar sua aplicação prática e ajustar somente quando houver evidência de excesso ou insuficiência.

### Futuro / ideias

Itens estratégicos ainda não aprovados permanecem nas seções de planejamento deste documento e em `docs/ROADMAP.md`; não constituem autorização de execução.

- Radar de Movimento sob demanda e fases posteriores de mapa/histórico.
- V21 — Criticidade Contextual Dinâmica.
- V22 — Motor Semântico.
- V23 — Observabilidade Operacional ampliada.
- V24 — IA/LLM e Contexto Adaptativo.
- Gestão inteligente de energia/UPS e Home Assistant principal/backup.

## Estado consolidado histórico de 2026-05-20

Esta tabela preserva o checkpoint original. Quando houver divergência, prevalece o estado operacional auditado de 2026-09-04 acima.

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
