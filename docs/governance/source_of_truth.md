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
   - `docs/ROADMAP.md` — índice executivo dos dois roadmaps;
   - `docs/roadmap/roadmap_v20_consolidado.md` — roadmap SOC;
   - `docs/governance/automacoes_taticas.md` — roadmap AT.

6. Auditorias
   - documentos de auditoria, investigacao, validacao e impacto.

7. Artefatos auxiliares
   - handoffs, releases, checklists, discovery, technical debt e historicos.

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
- Handoff transporta contexto; roadmap declara situacao; Gate registra decisao; Changelog registra historia; implementacao e homologacao comprovam entrega.

## Politica constitucional de crescimento documental

A politica detalhada de crescimento documental fica na Constituicao em `docs/governance/constituicao_central_operacional_v20.md`.

Este indice apenas define a precedencia: a Constituicao governa criacao de arquivos, extracao documental, classificacao, canonicidade e prevencao de fontes paralelas da verdade. Nenhum documento grande autoriza divisao automatica ou criacao automatica de artefato.

## Familias documentais

### governance

Concentra regras ativas, fonte da verdade, gates e constituicao operacional.

- `docs/governance/automacoes_taticas.md`
  - Classificacao: roadmap canonico da Vertical AT.
  - Finalidade: registrar melhorias pequenas e locais sob fluxo reduzido.
  - Autoridade: subordinado a Constituicao, ao Source of Truth e ao Gate de Enquadramento.

### roadmap

Concentra planejamento, fases futuras, dependencias e relacao com debitos tecnicos.

- `docs/ROADMAP.md`: indice executivo e roteador; nao duplica os roadmaps detalhados.
- `docs/roadmap/roadmap_v20_consolidado.md`: roadmap canonico SOC.
- `docs/governance/automacoes_taticas.md`: roadmap canonico AT, mantido em governance por tambem registrar seu fluxo reduzido.
- `docs/roadmap_central_operacional_semantic_house_v_26.md`: visao estrategica conceitual subordinada; nao e roadmap canonico e nao declara status operacional.

### releases congelados

- `docs/release_central_operacional_v20.md`: baseline historica V20.0 congelada em 2026-05-13; nao deve absorver evolucoes V20.1/V20.2 posteriores.

### technical_debt

Concentra pendencias, riscos, inconsistencias e candidatos a saneamento.

### pendencias operacionais

- `docs/pendencias_atuais_central_operacional.md`
  - Classificacao: fila canonica de acoes e decisoes concretas ainda necessarias.
  - Limite: nao substitui roadmap, backlog tecnico ou Gate; nao recebe ideias futuras sem aprovacao.
  - Estados: `ABERTA`, `RESOLVIDA`, `SUPERADA` e `NAO COMPROVADA`.

### handoffs

Concentra contexto auxiliar de continuidade entre sessoes, agentes e ferramentas.

- Local preferencial para novos arquivos: `docs/handoffs/`.
- Autoridade: auxiliar; nunca substitui Constituicao, Source of Truth, arquitetura, roadmap, Gates, Changelog, implementacao ou homologacao.
- Estados permitidos: `ATIVO`, `SUPERADO` e `ENCERRADO`.
- Limite: no maximo um handoff ativo por frente.
- Criar somente quando houver troca de sessao/agente/ferramenta, interrupcao de atividade complexa, working tree nao publicado, bloqueio relevante ou proximo passo sensivel ainda nao executado.
- Nao criar apos toda conversa ou Gate, nem para repetir roadmap ou Changelog.
- Conteudo minimo: frente, data/hora, status do handoff, roadmap, branch/HEAD, estado do working tree, ultimo fato comprovado, pendencias, riscos/bloqueios, proximo passo seguro, acoes nao autorizadas e referencias canonicas.
- Autorizações passadas não permanecem automaticamente válidas em nova sessão.
- Handoffs existentes em `docs/governance/` permanecem validos como auxiliares ate saneamento controlado; sua localizacao ou titulo nao lhes concede canonicidade.
- Transcricoes internas de agentes, incluindo `.jsonl` do Claude Code, nao sao handoffs do projeto.

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

- `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md`
  - Classificacao: arquitetura subordinada.
  - Finalidade: registrar a promocao limitada do Coordenador da Sessao de Monitoramento Remoto (CSMR).
  - Autoridade: nao substitui `architecture.md`, `docs/ARCHITECTURE.md`, a Constituicao nem o Gate especifico da V20.2C.
  - Limite: consolida a decisao arquitetural, mas nao autoriza implementacao ou publicacao em runtime.

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
