# Source of Truth - Central Operacional V20

Data: 2026-05-20
Status: ATIVO

## Indice documental oficial

Este documento atua como indice e roteador documental da Central Operacional V20.

Ele nao possui autoridade superior a Constituicao, nao substitui automaticamente documentos historicos, auditorias ou checkpoints e nao altera regras permanentes. Em caso de conflito, a Constituicao prevalece.

## Hierarquia oficial

1. `docs/governance/constituicao_central_operacional_v20.md`
   - regras permanentes, limites constitucionais e precedencia maxima.

2. `docs/governance/source_of_truth.md`
   - indice, roteador documental e mapa de familias documentais.

3. `AGENTS.md`
   - regras operacionais resumidas para execucao assistida.

4. Arquitetura
   - `architecture.md`, `docs/ARCHITECTURE.md` e documentos tecnicos de arquitetura.

5. Roadmap
   - `docs/ROADMAP.md`, `docs/roadmap/roadmap_v20_consolidado.md` e planejamentos subordinados.

6. Auditorias
   - documentos de auditoria, investigacao, validacao e impacto.

7. Artefatos auxiliares
   - releases, checklists, discovery, technical debt e historicos.

Documentos complementares dentro de `docs/governance/`, como `docs/governance/gates_v20.md`, permanecem subordinados a Constituicao e devem ser interpretados conforme esta hierarquia.

## Fontes atuais consolidadas

Os documentos abaixo continuam ativos como origem historica ou detalhamento, mas devem ser interpretados pela hierarquia acima:

- `AGENTS.md`
- `README.md`
- `CHANGELOG.md`
- `docs/CHANGELOG.md`
- `docs/ROADMAP.md`
- `docs/ARCHITECTURE.md`
- `architecture.md`
- `docs/pendencias_atuais_central_operacional.md`
- `docs/discovery/gap_source_of_truth.md`

## Regras de precedencia

- Constituicao define regras permanentes e prevalece sobre este indice.
- `source_of_truth` roteia e referencia documentos, mas nao muda canonicidade sozinho.
- `AGENTS.md` orienta execucao operacional, mas nao substitui a Constituicao.
- Arquitetura define desenho tecnico, mas nao substitui a Constituicao.
- Roadmap define direcao, mas nao altera contratos.
- Auditorias registram fotografia de um momento.
- Changelog registra historico, mas nao aprova mudanca funcional sozinho.
- YAML implementa configuracao, mas nao redefine arquitetura.
- Nenhum arquivo markdown criado durante execucao torna-se fonte de verdade automaticamente.
- Documentos operacionais e releases sao transitorios por padrao.
- Antes de criar novo arquivo, reutilizar documento existente quando houver categoria compativel.

## Politica constitucional de crescimento documental

A politica detalhada de crescimento documental fica na Constituicao em `docs/governance/constituicao_central_operacional_v20.md`.

Este indice apenas define a precedencia: a Constituicao governa criacao de arquivos, extracao documental, classificacao, canonicidade e prevencao de fontes paralelas da verdade. Nenhum documento grande autoriza divisao automatica ou criacao automatica de artefato.

## Familias documentais

### governance

Concentra regras ativas, fonte da verdade, gates e constituicao operacional.

### roadmap

Concentra planejamento, fases futuras, dependencias e relacao com debitos tecnicos.

### technical_debt

Concentra pendencias, riscos, inconsistencias e candidatos a saneamento.

### auditorias

Concentra checkpoints, investigacoes, validacoes e evidencias historicas.

- `docs/auditorias/auditoria_operacional_recovery_4g_v20_1q.md`
  - Classificacao: auditoria historica/evidencial, nao canonica.
  - Finalidade: registrar o estado observado do recovery 4G.
  - Limite: nao autoriza implementacao ou limpeza isoladamente.

### arquitetura

Concentra arquitetura oficial e documentos tecnicos de motores.

- `docs/arquitetura/despacho_arquitetural_v20_1q.md`
  - Classificacao: arquitetura subordinada.
  - Finalidade: registrar a decisao tecnica aprovada da V20.1Q.
  - Autoridade: nao substitui `architecture.md` nem `docs/ARCHITECTURE.md`.

## Documentos historicos

Documentos V17, V18, V19, investigacoes antigas e regras antigas permanecem como memoria operacional. Eles nao devem competir com governance, roadmap consolidado ou arquitetura V20.

## Documentos operacionais e releases

Documentos em `docs/releases/` registram checkpoints, validacoes, planejamentos de lote, checklists de implantacao e historico operacional. Eles nao criam regras permanentes, nao alteram contratos e nao competem com a hierarquia oficial.

- `docs/releases/implementation_plan_v20_1q.md`
  - Classificacao: release transitorio operacional.
  - Validade: execucao e homologacao da V20.1Q.
  - Limite: nao cria regra permanente e nao declara implementacao concluida.

Classificacao explicita:

- `docs/releases/v20.1n_checklist_implantacao.md`
  - Tipo documental: checklist operacional de implantacao.
  - Autoridade: historica/transitoria; subordinada a `docs/governance/source_of_truth.md`, `docs/governance/constituicao_central_operacional_v20.md` e `docs/governance/gates_v20.md`.
  - Validade: valida apenas como preparacao e evidencia operacional do lote V20.1N - Atividade Operacional.
  - Destino final: permanecer em `docs/releases/` como registro historico; nao migrar para governance, roadmap, technical_debt ou arquitetura salvo se uma fase futura identificar conhecimento permanente ainda nao incorporado.
  - Estado atual do lote V20.1N: PREPARADO, NAO HOMOLOGADO EM RUNTIME.
  - Backup pre-implantacao criado em `/private/tmp/v20_1n_implantacao_pre/`.
  - Nenhum reload/restart foi executado.
  - Testes runtime/shadow reais nao foram executados.
  - `packages/motor_atividade_operacional_v20.yaml` ainda nao esta rastreado no git.
  - Proximo passo obrigatorio: reload/restart controlado e validacao manual de TV, microondas, maquina de lavar e multiatividade.
  - Rollback deve permanecer preparado antes do reload/restart.
  - Gate pré-homologação:
    - [ ] Verificar git status
    - [ ] Confirmar motor_atividade_operacional_v20.yaml rastreado
    - [ ] Confirmar backup existente
    - [ ] Executar reload/restart controlado
    - [ ] Validar TV
    - [ ] Validar microondas
    - [ ] Validar máquina de lavar
    - [ ] Validar multiatividade
    - [ ] Confirmar timeline
    - [ ] Confirmar ausência de impacto em sensor.status_casa

## Regra de manutencao

Toda nova fase deve atualizar, quando aplicavel:

- roadmap consolidado;
- backlog tecnico;
- gates;
- changelog/checkpoint;
- auditoria correspondente.

Nenhuma fase deve ser considerada encerrada sem passar pelos gates definidos em `docs/governance/gates_v20.md`.
