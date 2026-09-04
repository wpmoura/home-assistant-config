# Roadmap da Central Operacional

Data do checkpoint de consolidação: 2026-09-04
Função: índice executivo dos roadmaps SOC e AT

## Organização canônica

A Central Operacional possui dois roadmaps permanentes:

| Roadmap | Finalidade | Documento canônico |
| --- | --- | --- |
| SOC — Sistema Operacional Casa | Desenvolvimentos complexos, arquitetura, contratos, motores e componentes centrais | `docs/roadmap/roadmap_v20_consolidado.md` |
| AT — Automações Táticas | Melhorias pequenas e delimitadas que consomem interfaces e entidades existentes | `docs/governance/automacoes_taticas.md` |

O Gate de Enquadramento em `docs/governance/gates_v20.md` decide `GO AT`, `GO SOC` ou `NO-GO`. Prefixos históricos não determinam pertencimento atual.

## Resumo executivo auditado em 2026-09-04

| Frente | Roadmap | Status principal | Condição ou pendência |
| --- | --- | --- | --- |
| Health Check | SOC | Concluído | Dívida separada: lógica principal Node-RED fora do Git; integração semântica com Timeline adiada |
| Recovery 4G | SOC | Em andamento | Homologação suspensa por decisão operacional; implementação existente e válida |
| CSMR — baseline publicada | SOC | Concluído | I4B.2 permanece como evidência operacional natural não bloqueante |
| CSMR — alterações locais posteriores | SOC | Em andamento | NO-GO para publicação até auditar os oito arquivos locais registrados no handoff |
| Lavadora/FSM | SOC | Concluído | Decidir destino do watcher pós-cutover, sem bloquear a baseline |
| Heartbeat HA → Timeline → SmallTV | SOC | Concluído | Dívida técnica: allowlist replicada em contrato, motor e SmallTV |
| Gestão do Carro — baseline AT-GC | SOC atual; origem AT | Concluído somente para a baseline AT-GC-00 a AT-GC-08 | Domínio não está integralmente concluído; zonas permanecem pendentes |
| Entrada/saída em zonas conhecidas | SOC | Backlog priorizado | Não iniciada; inclui Casa da Fernanda, Casa da Camila e demais zonas cadastradas |
| V20.2E — Uso do carro na Timeline | SOC | Em andamento | Núcleo funcional recuperado e evidenciado; homologação runtime complementar e estado local/publicação ainda precisam de consolidação |
| V20.2 shadow — motores contextuais | SOC | Em andamento | Implementação parcial em paralelo; promoção produtiva não autorizada como conjunto |
| AT-001 Dell/Time Machine | AT | Concluído | Validação com carga real permanece não bloqueante |

### Nota sobre Gestão do Carro

A execução histórica AT-GC permanece preservada e homologada. O crescimento do domínio motivou seu reenquadramento para SOC. Odômetro, abastecimentos, ingestão por imagens, manutenção e dashboard compõem a baseline concluída; o domínio não deve ser declarado integralmente encerrado enquanto o registro de entrada e saída nas zonas conhecidas permanecer pendente.

A zona Casa continua sob responsabilidade do CSMR para evitar duplicidade semântica. O desenho das demais zonas deve decidir contrato, idempotência, oscilações de GPS, troca direta, sobreposição e mudanças cadastrais antes da implementação.

### Dívidas de governança prioritárias

- Divergência estrutural entre `main`, `feature/v20-2c-contextual-automations` e a branch histórica de governança AT.
- Oito alterações locais da frente CSMR ainda não auditadas no working tree original.
- Política de handoffs definida: artefatos auxiliares, no máximo um ativo por frente, sem autoridade de decisão ou autorização; handoff do Health Check incorporado seletivamente como artefato encerrado, preservando o original em `main`.
- Documentos antigos mantêm status superados e precisam ser interpretados por este checkpoint até saneamento controlado.
- A política de prompts proporcionais foi definida em `docs/governance/gates_v20.md` com níveis `P1`, `P2` e `P3`; sua aplicação deve ser auditada nas próximas atividades antes de qualquer refinamento.
- A fila operacional canônica foi reconciliada em `docs/pendencias_atuais_central_operacional.md`; o conteúdo de maio permanece como snapshot histórico e não implica resolução automática.

### Futuro / ideias sem autorização de execução

- Radar de Movimento sob demanda e suas fases posteriores de mapa/histórico.
- V21 — Criticidade Contextual Dinâmica.
- V22 — Motor Semântico.
- V23 — Observabilidade Operacional ampliada.
- V24 — IA/LLM e Contexto Adaptativo.
- Gestão inteligente de energia/UPS e Home Assistant principal/backup.

Esses itens permanecem visíveis para planejamento, mas não constituem backlog priorizado nem autorização de implementação.

## Estado histórico anterior ao checkpoint de 2026-09-04

Esta seção é preservada como memória do planejamento acumulado. Quando houver divergência de status, prevalece o resumo executivo auditado acima até o saneamento controlado do conteúdo histórico.

- V20.0 = concluída e congelada
- V20.1A = concluída
- V20.1B lote 1 = concluída
- V20.1B lote 2 = concluída/parcial para energia, internet, failover e backup
- V20.1C = `V20.1C_FECHAMENTO` registrado; diagnóstico e governança concluídos, decommission bloqueado
- V20.1D/E = checkpoint documental de dependências e impacto concluído; limpeza não autorizada
- V20.1F/G/H = validação e investigação de V19 concluídas
- V20.1I/J = isolamento controlado de packages desativados aplicado e validado
- V20.1K = concluída; tag `V20.1K_FECHAMENTO` criada
- V20.1N = homologada; checkpoint de estabilização registrado
- V20.1O = estável com débitos aceitos; política Timeline / Push / Agregação estabilizada
- V20.1Q = pacote corretivo implementado estaticamente; validação operacional pendente
- V20.2A = concluída; dashboard legado `teste-4` removido pela UI
- V20.2B = auditoria executada, sem ação operacional
- V20.2C-A1 = promoção limitada do CSMR consolidada documentalmente; implementação bloqueada pelo Gate específico
- V20.2E = integração aditiva do uso do carro à Timeline implementada estaticamente; homologação runtime pendente
- V20.2 geral/V20.3/V21 = planejamento futuro; exceção V20.2C-A1 registrada separadamente
- A autorização V20.2E é restrita ao produtor `carro_presenca` e não promove o restante da V20.2

Estado operacional consolidado:

- Workspace limpo no último checkpoint validado.
- Sem resíduos V19 conhecidos em dashboards ativos.
- Dashboard legado `teste-4` removido pela UI/fluxo suportado.
- Auditoria de automações identificou 21 órfãs; automações críticas não devem ser removidas automaticamente.
- Limpeza técnica futura deve seguir criticidade, em lotes pequenos e reversíveis.

Checkpoint de governança e prioridade operacional:

- GIT-HYGIENE-01 resolvido pela Fase 0 no commit `8e59a4be2e09ebb2706d4fc46a87a7258aa646b0`.
- Branch `develop` publicada e sincronizada, sem alteração de `main`.
- V20.1Q — Recovery Automático do Modem 4G registrada como prioridade operacional aprovada.
- V20.1Q.1 — Executor genérico, parâmetros, estabilização e política de cooldown implementados no repositório.
- V20.1Q.2 — eventos canônicos integrados à Timeline; homologação runtime permanece bloqueada até autorização explícita.
- Governança constitucional ajustada: Constituição prevalece sobre `source_of_truth`; `source_of_truth` atua como índice/roteador documental.
- Radar/Mapa Operacional Fase 1 está desenhado no ROADMAP, mas não autorizado para implementação.
- Bloco técnico do Radar/Mapa está sinalizado como candidato futuro à extração controlada, sem extração autorizada.
- Lote 1 C1 da auditoria de legado e plano de observação operacional de 7 dias estão definidos.
- Decommission permanece bloqueado.
- Os três artefatos formais da V20.1Q estão referenciados na seção própria deste Roadmap.

## Arquitetura Oficial

Fluxo oficial de processamento:

- Sensores físicos
- V20.1B Deterministic Event Layer
- Context Engine V20.2
- Relevance Engine V20.2
- Correlation Engine V20.2
- Dynamic Score
- Narrativa determinística
- Aliases finais futuros

## Regras Obrigatórias

- Nunca alterar `sensor.status_casa`
- Nunca alterar aliases finais sem validação
- Dashboards produtivos não consomem `_v20_2`
- V20.2 permanece isolada em shadow, exceto pelo CSMR da V20.2C promovido de forma limitada e ainda bloqueado pelo Gate específico
- IA é opcional
- IA desligada mantém sistema 100% funcional
- Não substituir automações legadas sem auditoria V20.1C
- Packages novos devem permitir rollback simples
- Dashboard Radar de Movimento deve ser sob demanda
- Alertas contextuais futuros devem ser assistivos e desacoplados

## Backlog Estratégico

- Gestão inteligente de energia/UPS
- HA resiliente com principal -> backup RPi5
- Internet/4G/failover
- Camada contextual futura
- V20.2A - Evolução Contextual de Atividades
- Limpeza técnica futura
- Radar de Movimento sob demanda
- Assistente Contextual Preventivo
- Weather Risk Engine
- Presence Intelligence
- Observabilidade operacional
- Energy Brain
- House Exposure Engine
- IA contextual opcional

### Backlog priorizado V20.2F — Registro de entrada e saída das zonas cadastradas

Status: backlog priorizado SOC — não iniciado.

Finalidade funcional preliminar: registrar na Timeline a entrada e a saída de Wilson das zonas cadastradas no Home Assistant.

Premissa preliminar: a zona Casa permanece sob responsabilidade do CSMR, evitando duplicidade semântica com os eventos homologados de saída e retorno de casa.

Este registro não constitui fase ativa nem autoriza implementação sem Gate próprio. A necessidade pertence à continuidade da Gestão do Carro no SOC e não integra a baseline concluída AT-GC nem o escopo histórico da V20.2E.

Decisões pendentes de análise formal:

- contrato e campos necessários para identificar a zona;
- identidade lógica e idempotência;
- tratamento de `not_home`, `unknown` e `unavailable`;
- oscilações do GPS nas fronteiras;
- troca direta entre zonas;
- zonas sobrepostas;
- criação, renomeação e remoção de zonas;
- necessidade ou não de pushes;
- critérios de abertura e Gate próprio.

## V20.0 - Baseline Finalizada

Status: concluída e congelada em 2026-05-13.

Entregas principais:

- Aliases finais sem versão.
- Motor de evento dominante.
- Timeline/feed operacional com 6 eventos.
- Registro de eventos secundários.
- TV com contexto semântico.
- Porta da Sala com timeout parametrizável.
- Proteção contra estados inválidos.
- Preservação de V19 e legado.
- Documentação formal de release.

## V20.1 - Legacy Migration Layer + Operational Control Layer

Status: planejada dentro do ciclo V20, sem alterar a baseline V20.0 congelada.

Objetivo: criar uma camada de controle operacional e migração gradual para parametrizar a publicação de eventos sem quebrar aliases finais, dashboards oficiais ou legado.

Direções:

- Parametrizar a quantidade máxima de linhas da timeline.
- Criar helper operacional para limite de linhas.
- Definir mínimo `3`, máximo `100` e default `6`.
- Permitir ajuste via painel administrativo.
- Criar controle operacional do que entra na timeline/feed.
- Usar `input_boolean` individual por classe de evento.
- Exemplos de controles:
  - `timeline_tv`
  - `timeline_chuva`
  - `timeline_porta`
  - `timeline_backup`
  - `timeline_energia`
- Semântica dos controles:
  - ligado: publica na timeline/feed.
  - desligado: ignora na timeline/feed.
- Consolidar os controles operacionais no painel administrativo.

Critério de compatibilidade:

- A V20.1 deve preservar os contratos públicos da V20.0.
- A V20.1 não deve reativar V19.
- A V20.1 não deve alterar `sensor.status_casa` legado.

### V20.1A - Operational Control Layer

Status: concluída.

Escopo:

- parametrização da quantidade máxima de eventos da timeline/feed
- controles liga/desliga por domínio de publicação
- consolidação de controles operacionais no painel administrativo
- preservação dos aliases finais e dashboards produtivos

### V20.1B - Legacy Migration Layer

Status: lote 1 concluído; lote 2 concluído/parcial para energia, internet, failover e backup, com legado preservado.

Escopo real:

- migração da camada de eventos
- timeline V20
- feed operacional V20
- sensores determinísticos V20
- camada semântica inicial
- redução de parsing textual
- publicação estruturada de eventos operacionais
- cobertura parcial dos domínios energia, internet, failover e backup

Limites explícitos:

- V20.1B não representa migração completa das automações da casa.
- V20.1B não autoriza remover automações antigas.
- V20.1B não autoriza desativar blueprints antigos.
- V20.1B não garante que todos os side-effects legados estejam mapeados.
- V20.1B não garante que dependências indiretas estejam conhecidas.

Risco arquitetural:

- automações antigas podem alimentar múltiplos componentes
- notificações antigas podem atualizar helpers, `input_text` ou sensores derivados
- algumas automações podem impactar score, contexto, modos operacionais ou ações físicas
- remover legado sem auditoria pode causar regressões silenciosas

Decisão:

- manter legado ativo
- usar V20.1B como camada paralela determinística
- tratar remoção de legado como fase própria

### V20.1C - Legacy Decommission Audit

Status: `V20.1C_FECHAMENTO` registrado. Diagnóstico e governança concluídos; decommission permanece bloqueado.

Objetivo: auditar o legado antes de qualquer desativação controlada.

Direções:
- Documento inicial criado em `docs/auditoria_legado_v20_1c.md`.
- Inventário preliminar de V19/V20.2 e pacotes de suporte concluído como diagnóstico documental.
- Dependências classificadas em `docs/dependencias_legado_v20_1d.md`.
- Impacto de limpeza simulado em `docs/impacto_limpeza_v20_1e.md`.
- Automações, scripts, blueprints e packages auditados no documento V20.1C.
- Risco operacional classificado e quarentena controlada definida.
- Decommission continua bloqueado.
- Nenhuma limpeza ou automação está autorizada para desligamento automático.
- Limpeza futura continua dependente de classificação de risco, consumidores conhecidos, side-effects mapeados, janela de observação e rollback obrigatório.

### V20.1D - Legacy Dependency Analysis

Status: concluída em checkpoint documental com V20.1E.

Objetivo: mapear dependências reais do legado antes de qualquer limpeza, remoção ou migração.

Direções:
- Documento de análise criado em `docs/dependencias_legado_v20_1d.md`.
- Validar uso em produção versus uso apenas em dashboard ou no restore/registry.
- Identificar candidatos a legado e candidatos a remoção futura.
- Preservar helpers, dashboards, automações e scripts durante a análise.

- mapear dependências diretas e indiretas
- identificar consumidores do legado
- validar side-effects de automações, scripts e blueprints
- separar mensagens textuais de ações físicas/recovery
- classificar automações por risco
- desativar legado somente em lotes pequenos e reversíveis
- remover duplicidades apenas após validação

Matriz de prontidão prevista:

| Domínio | Camada V20 criada? | Timeline usando V20? | Legado ainda ativo? | Automações auditadas? | Risco de regressão? | Pronto para desativação? |
|---|---|---|---|---|---|---|
| Porta da Sala | Sim | Sim | Sim | Parcial | Alto | Não |
| Portas internas | Sim | Sim, opcional | Sim | Parcial | Médio | Não |
| Janelas/contatos | Sim | Sim, opcional | Sim | Parcial | Médio | Não |
| Chuva | Sim | Sim | Sim | Parcial | Alto | Não |
| Vazamento | Sim | Sim, opcional | Sim | Parcial | Alto | Não |
| Banho semântico | Sim | Sim | Sim | Parcial | Médio | Não |
| Energia | Sim | Sim | Sim | Parcial | Alto | Não |
| Internet | Sim | Sim | Sim | Parcial | Alto | Não |
| Failover 4G | Sim | Sim | Sim | Parcial | Alto | Não |
| Backup Google | Sim | Sim | Sim | Parcial | Médio | Não |
| TV | Parcial | Sim | Sim | Parcial | Médio | Não |
| Segurança/alarme | Não | Não | Sim | Não | Crítico | Não |
| Recovery/ações físicas | Não | Não | Sim | Não | Crítico | Não |

### V20.1E - Cleanup Impact Simulation

Status: concluída.

Objetivo: simular impacto de limpeza de candidatos de legado sem executar remoção.

Resultado:

- Relatório criado em `docs/impacto_limpeza_v20_1e.md`.
- Candidatos em `_disabled/` foram inicialmente classificados como seguros do ponto de vista documental.
- `.storage/core.entity_registry` e `.storage/core.restore_state` permaneceram fora de escopo de limpeza.
- Nenhum YAML produtivo, dashboard, automação ou entidade foi alterado.

### V20.1F - Operational Validation

Status: concluída em investigação documental.

Objetivo: validar operacionalmente os candidatos de limpeza antes de qualquer remoção.

Resultado:

- Relatório criado em `docs/validacao_operacional_v20_1f.md`.
- `packages/_disabled/status_casa_v19.yaml` foi reclassificado como requer observação.
- Foi identificado histórico recente de `binary_sensor.casa_tv_ativa_v19`.
- Limpeza de V19 permaneceu bloqueada.

### V20.1G - Active V19 Dependency Trace

Status: concluída.

Objetivo: investigar por que `binary_sensor.casa_tv_ativa_v19` ainda possuía eventos recentes.

Resultado:

- Relatório criado em `docs/investigacao_casa_tv_ativa_v19.md`.
- A entidade foi classificada como realmente ativa.
- A origem foi atribuída ao template V19 preservado em package.
- Não foi encontrado consumo direto por `sensor.status_casa`, automações, scripts ou dashboards oficiais.

### V20.1H - Effective Package Loading Investigation

Status: concluída.

Objetivo: determinar por que o package V19 em `_disabled` continuava efetivo.

Resultado:

- Relatório criado em `docs/investigacao_carregamento_v19.md`.
- Causa raiz identificada: isolamento inefetivo por manter artefatos históricos dentro da árvore `packages/`.
- `packages/_disabled/` foi confirmado como convenção documental insuficiente para desativação operacional neste ambiente.

### V20.1I - Disabled Package Isolation

Status: aplicado e validado pela V20.1J.

Objetivo: impedir que artefatos históricos em `_disabled` sejam carregados por `homeassistant.packages`.

Alterações controladas:

- Criado `archive/packages_disabled/`.
- Movido `packages/_disabled/status_casa_v19.yaml` para `archive/packages_disabled/status_casa_v19.yaml`.
- Movido `packages/_disabled/DESATIVACAO_V19.md` para `archive/packages_disabled/DESATIVACAO_V19.md`.
- Removido `packages/_disabled/` por estar vazio.

Limites:

- `.storage/core.entity_registry` não foi editado.
- `.storage/core.restore_state` não foi editado.
- Nenhuma entidade foi removida manualmente.
- Nenhuma lógica produtiva foi alterada.
- Nenhum dashboard ou automação foi alterado.

Validação V20.1J:

- `binary_sensor.casa_tv_ativa_v19` existe, mas ficou `unavailable`.
- A entidade não alternou após teste real da TV.
- `sensor.status_casa` existe e manteve valor `⚠️ Backup Google com falha`.
- `sensor.casa_timeline` existe e manteve valor `22:37 📺 TV desligada`.
- Registros persistidos em `.storage` seguem intocados até fase própria.

### V20.1J - Post-Isolation Validation

Status: aprovada.

Objetivo: validar se a movimentação dos artefatos V19 para `archive/packages_disabled/` impediu o carregamento do template legado.

Resultado:

- Relatório criado em `docs/validacao_pos_isolamento_v20_1j.md`.
- O isolamento de `packages/_disabled/` funcionou.
- `binary_sensor.casa_tv_ativa_v19` ficou apenas como resíduo `unavailable`.
- Não houve alternância da entidade V19 após teste real da TV.
- Não houve impacto aparente em `sensor.status_casa`.
- Não houve impacto aparente em `sensor.casa_timeline`.

Recomendação:

- Commits documentais e de isolamento controlado estão liberados.
- Limpeza de `.storage`, entidades residuais e dashboard oculto `teste-4` deve permanecer fora desta fase.

### V20.1K - V19 Residue Documentation Cleanup

Status: concluída em análise documental.

Objetivo: mapear e classificar resíduos V19 restantes após o isolamento de `packages/_disabled/`, sem remover nada.

Resultado:

- Relatório criado em `docs/saneamento_residuos_v20_1k.md`.
- Entidades V19 residuais foram classificadas como candidatas a limpeza futura, com estado final `unavailable`.
- `.storage/core.entity_registry` foi classificado como candidato a limpeza futura, sem edição manual.
- `.storage/core.restore_state` foi classificado como não remover automaticamente.
- Dashboard oculto `teste-4` foi inicialmente classificado como não remover automaticamente e posteriormente removido pela UI/fluxo suportado na consolidação V20.2A.
- Comentário antigo em `packages/status_casa.yaml` foi identificado como referência documental obsoleta e corrigido na V20.1K.1 pelo commit `6cbfd18 docs: update archived V19 package reference`.

Recomendação:

- V20.2A pode avançar sem bloquear por resíduos V19.
- Limpeza de registry, restore_state e dashboard oculto deve ser fase própria, reversível e validada pela UI do Home Assistant.

### V20.1N - Operational Stabilization Checkpoint

Status: homologada.

Objetivo: registrar o encerramento do ciclo V20.1N após homologação completa, sem implementar novas funcionalidades e sem alterar YAML.

Marcos registrados:

- V20.1N.4.1 Motor operacional estabilizado.
- V20.1N.4.2 Timeline integrada.
- V20.1N.4.3 Pós-multiatividade.
- V20.1N.4.3a Consistência visual.

Resultados homologados:

- TV funcionando.
- Multiatividade funcionando.
- Timeline consistente.
- Deduplicação OK.
- Sem `unavailable`.
- Sem spam.

Referência:

- Checkpoint documental em `docs/releases/v20.1n-estabilizacao.md`.

### V20.1O - Politica Timeline / Push / Agregacao

Status: ESTÁVEL COM DÉBITOS ACEITOS.

Objetivo: encerrar formalmente a política individual de publicação para eventos da Central Operacional V20, mantendo a arquitetura canônica da timeline estabilizada.

Escopo congelado:

- Política individual por evento implementada.
- Timeline independente.
- Push independente.
- Agregação independente.
- Banho habilitado por padrão.
- Encerramento do banho em 2 minutos.
- Push consumindo evento canônico atual.
- Agregação respeitando flags individuais.
- Agregação considerando contexto operacional ativo.
- Máquina de lavar respeitando transições reais OFF->ON e ON->OFF.
- Correção de falso positivo de Backup.
- UI integrada com Timeline / Push / Agrupar.
- Compatibilidade legada preservada.

Governança do congelamento:

- Arquivos criados = 0.
- Package shadow = 0.
- Documentação nova = 0.
- Fontes de verdade novas = 0.
- Alterações fora do escopo = 0.
- Não reabrir V20.1O silenciosamente.
- Mudanças futuras devem abrir V20.1P, V20.2 ou outro lote formal.

Referências residuais conhecidas:

- Atributos `linha_1` a `linha_6` permanecem como compatibilidade legada; a renderização final produtiva usa `sensor.casa_event_feed` com `eventos_json` e `limite_eventos`.
- Helpers visuais legados de banho podem existir no ambiente, mas o fluxo V20.1O usa banho habilitado por padrão e encerramento fixo de 2 minutos.
- Contextos agregáveis dependem de sensores canônicos V20 e, no caso de Backup, também de falha real em `sensor.backup_google_status`.

Débito aceito pós-V20.1O:

- Categoria: UX / Narrativa contextual.
- Sintoma: eventos agregados podem ocultar a saída individual de participantes.
- Exemplo: TV + Microondas + Banho ativos, depois TV e Microondas desligam enquanto Banho continua ativo, mas a timeline pode mostrar apenas Banho encerrado.
- Impacto: não afeta estado real, automações, push ou lógica operacional; afeta apenas interpretação humana da timeline.
- Tratamento: reservado para V20.1P — Inteligência contextual da timeline.

## V20.1Q - Recovery Automático do Modem 4G

Status: formalizada documentalmente; implementação não iniciada.

Prioridade: resiliência operacional da conectividade remota.

Fases:

- V20.1Q Fase 0 — saneamento Git concluído no commit `8e59a4be2e09ebb2706d4fc46a87a7258aa646b0`.
- V20.1Q.1 — Recovery Executor parametrizável e painel administrativo de parâmetros.
- V20.1Q.2 — integração definitiva com Timeline e publicadores canônicos.

Princípio:

> A Central decide. O Executor atua. A Central valida. A Central encerra.

Referências:

- Auditoria: `docs/auditorias/auditoria_operacional_recovery_4g_v20_1q.md`.
- Despacho: `docs/arquitetura/despacho_arquitetural_v20_1q.md`.
- Plano: `docs/releases/implementation_plan_v20_1q.md`.

Pendências pré-implementação:

- default numérico de cooldown;
- default numérico de timeout de validação;
- confirmação de helpers equivalentes, arquivos e estratégia de restart;
- autorização posterior após nova Etapa A.

Nenhum teste físico, alteração funcional ou decommission de legado está autorizado pela formalização documental.

## V20.2 - Dedicated Engines + UX/Operational Layout

Status: parcialmente implementada em shadow mode/paralelo; checkpoint de homologação Fase A concluído; auditoria operacional residual iniciada.

A regra geral de shadow permanece vigente. A única exceção arquitetural formal é o CSMR da V20.2C, promovido de forma limitada e ainda sem implementação autorizada.

Checkpoint: Homologação Fase A concluída.

Referências de alinhamento:

- Arquitetura oficial detalhada em `architecture.md`
- Backlog estratégico detalhado em `backlog.md`

Objetivo: melhorar organização visual, ergonomia operacional e separação interna por motores especializados, sem avançar ainda para criticidade contextual dinâmica da V21.

Direções:

- Evoluir visualmente a Central Operacional.
- Criar layout operacional contextual.
- Organizar a arquitetura por motores especializados.
- Melhorar a visualização da timeline/feed.
- Preparar filtros, contexto e drill-down operacional.
- Separar melhor leitura executiva, debug e governança.
- Manter dashboards produtivos consumindo aliases finais sem versão.

### V20.2C-A1 — Promoção Arquitetural da Sessão de Monitoramento Remoto

Status: PROMOÇÃO LIMITADA CONSOLIDADA DOCUMENTALMENTE; IMPLEMENTAÇÃO E PUBLICAÇÃO EM RUNTIME BLOQUEADAS PELO GATE.

Objetivo: reconhecer o Coordenador da Sessão de Monitoramento Remoto (CSMR) como motor oficial de coordenação operacional com escopo restrito à saída e ao retorno de Wilson, sem promover o restante da V20.2 e sem reabrir a V20.1O.

Decisões consolidadas:

- Sessão de Monitoramento Remoto é contrato operacional distinto da presença bruta;
- CSMR possui autoridade exclusiva sobre ciclo de vida e ordem da sessão;
- V20.1O permanece publicador canônico e autoridade sobre histórico, limite, deduplicação e apresentação;
- somente quatro eventos de sessão foram autorizados arquiteturalmente;
- C1.1, C1.2 e C1.3 são consumidores subordinados;
- implementação depende de plano técnico restrito e aprovação do Gate específico;
- Context Engine e demais componentes V20.2 permanecem shadow.

Referências:

- `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md`;
- `docs/governance/gates_v20.md`;
- `docs/v20_2c/README.md`;
- `docs/v20_2c/c1_saida_de_casa.md`.

### V20.2A - Legacy Dashboard Review

Status: validada por inspeção e remoção externa confirmada.

Objetivo: avaliar o dashboard oculto legado `teste-4` antes de qualquer remoção.

Resultado:

- Dashboard `Laboratório - Casa Inteligente` (`teste-4`) foi classificado como legado V19 sem vínculo operacional encontrado.
- Remoção foi validada posteriormente por inspeção: `teste_4` não aparece mais em `.storage/lovelace_dashboards` e `.storage/lovelace.teste_4` não existe mais.
- Referências V19 não aparecem em dashboards ativos após a remoção.
- Permanecem apenas referências documentais/históricas.

### V20.2A - Evolução Contextual de Atividades

Status: decisão arquitetural aprovada para próximos passos.

Decisão: o monitoramento de banho passa a ser ativo por padrão daqui para frente.

Regras:

- Não tratar banho como funcionalidade opcional.
- Manter inicialmente desacoplado do motor operacional V20.1N.
- Utilizar lógica contextual existente, combinando movimento, umidade e sensores relacionados.

Regra padrão de encerramento:

```text
Ausência de evidências de banho por 2 minutos
↓
Encerrar banho
```

Objetivo:

- Evitar falso encerramento causado por oscilações de movimento.
- Evitar falso encerramento causado por estabilização da umidade.
- Evitar falso encerramento causado por pequenas pausas durante banho.

Futuro candidato:

- Incorporar banho ao motor operacional como atividade formal (`🛁 banho`) após estabilização.

### V20.2B - Automation Residual Audit

Status: encerramento provisório documental.

Objetivo: registrar achados da auditoria de automações órfãs/desabilitadas sem executar limpeza.

Resultado:

- Foram identificadas 21 automações órfãs em `.storage/core.entity_registry`.
- Automações críticas não devem ser removidas automaticamente.
- A limpeza futura deve ocorrer por criticidade, em lotes pequenos, reversíveis e validados pela UI do Home Assistant.

Categorias de triagem:

| Categoria | Critério | Diretriz |
| --- | --- | --- |
| Crítico operacional | energia, UPS, internet/failover, vazamento, alarme, porta, segurança ou ações físicas | não remover automaticamente; validar funcionamento real |
| Legado/LAB | aliases com `LAB`, testes, notificações antigas, experiências de dashboard ou laboratório | revisar e remover somente se descartado formalmente |
| Provável remoção | `nova_automacao*`, duplicatas antigas, órfãs sem domínio crítico e sem referência operacional | candidato a limpeza futura pela UI |
| Precisa validação | automações sem trigger detectável, entidades inexistentes ou side-effects pouco claros | inventariar antes de qualquer mudança |

Princípio:

- Nenhuma automação deve ser habilitada, desabilitada ou removida sem validação de domínio e rollback.

### Semantic Timeline Refinement

Objetivo futuro: separar temporalmente o instante físico do evento e o instante em que o motor confirmou o evento após tolerância, debounce ou grace period.

Motivação:

- Na V20.1B, `🚿 Banho encerrado` usa `input_number.casa_timeout_banho_minutos`.
- O horário publicado hoje representa o momento de confirmação após o timeout.
- Essa decisão é mantida na V20.1B porque o evento semântico representa confirmação com confiança, não apenas uma leitura física transitória.

Modelo futuro:

- `timestamp_ocorrencia`: quando o evento físico ocorreu.
- `timestamp_confirmacao`: quando o motor confirmou o evento.

Exemplo futuro:

- Sensor do box ficou `off` às `09:20`.
- Timeout confirmou encerramento às `09:23`.
- Timeline pode exibir `09:20 🚿 Banho encerrado`.
- Atributo interno pode registrar `confirmado_em: 09:23`.

Regras de compatibilidade:

- Não alterar o comportamento da V20.1B.
- Não remover a tolerância parametrizável.
- Preservar a semântica de evento confirmado.
- Evoluir atributos internos antes de alterar a apresentação pública da timeline.

### Correlation And Event Ordering

Objetivo futuro: ordenar e agrupar eventos correlacionados que pertencem à mesma recuperação operacional.

Débito técnico registrado na V20.1B lote 2:

- Em recuperação completa de internet/failover, a timeline pode publicar `🌐 Internet normalizada` antes de `📡 Failover 4G encerrado`.
- A recuperação está funcionalmente correta.
- A ordem semanticamente desejada é:
  - `📡 Failover 4G encerrado`
  - `🌐 Internet normalizada`

Tratamento previsto:

- Refinar a máquina de estados de internet/failover.
- Introduzir correlação contextual de eventos próximos.
- Permitir agrupamento ou ordenação semântica antes da publicação final na timeline.
- Resolver em V20.2 ou V20.3 sem alterar a baseline V20.1B validada.

### Radar de Movimento da Casa - Modo Sob Demanda

Status: backlog futuro.

Nota de governanca documental: esta secao contem detalhamento tecnico acima do nivel executivo esperado para o ROADMAP. O Radar/Mapa Operacional e candidato futuro a extracao controlada para documento classificado, mas a extracao nao esta autorizada neste momento. Nao mover conteudo automaticamente; aguardar aprovacao explicita. O ROADMAP permanece documento executivo.

Objetivo: criar uma interface opcional de monitoramento de movimento por cômodo, ativada manualmente pelo usuário, para uso eventual e não permanente no dashboard principal.

Requisitos funcionais:

- Criar um helper de controle, preferencialmente `input_boolean.casa_radar_movimento_ativo`.
- Quando o helper estiver desligado, nenhum cartão, badge ou chip de radar de movimento deve aparecer na interface principal.
- Quando o helper estiver ligado, exibir uma seção ou cartão chamado `Radar de Movimento`.
- A seção deve mostrar somente os cômodos com movimento ativo naquele momento.
- Se nenhum cômodo estiver com movimento ativo, não devem aparecer botões, badges ou chips de cômodos.
- Evitar poluição visual no dashboard principal.
- Implementar futuramente com packages completos em `/config/packages/`.
- O dashboard deve consumir sensores finais ou semânticos, sem lógica pesada embutida.
- Não implementar agora; manter como backlog/pendência futura.

Diretriz arquitetural:

- Criar futuramente uma camada semântica de movimento por cômodo.
- Consolidar sensores físicos como PIR, mmWave, presença, ocupação e sensores específicos, como o sensor do box do banheiro.
- Separar leitura física de movimento/presença da apresentação visual no dashboard.
- Manter o radar como experiência sob demanda, não como elemento fixo da Central Operacional.

Inventário preparatório por ambiente:

Este inventário é base de descoberta para o Radar/Mapa Operacional. Ele não cria nova fonte de verdade, não autoriza implementação e deve ser validado em runtime antes de qualquer package, helper ou dashboard futuro.

| Ambiente | Sensores físicos | Sensores V20 derivados | Função operacional | Fonte candidata para camada semântica | Elegível para Radar |
|---|---|---|---|---|---|
| Área de Serviço | `binary_sensor.movimento_area_servico_presence`; sensor específico de vazamento registrado como `binary_sensor.sensor_vazmento_agua_water_leak` | `sensor.casa_vazamento_estado_v20` para vazamento; sem derivado V20 de presença por ambiente | Presença local e alerta específico de água/vazamento | Presença local como atividade; vazamento como alerta contextual, não como movimento | SIM |
| Banheiro | `binary_sensor.movimento_banheiro_occupancy`; `binary_sensor.movimento_alto_banheiro_occupancy`; `binary_sensor.sensor_porta_banheiro_contact` | `sensor.casa_banho_estado_v20`; `sensor.casa_porta_banheiro_estado_v20` | Movimento geral, sensor de box/chuveiro e porta interna | Separar presença do banheiro de evento semântico de banho; sensor alto não deve virar movimento genérico | SIM |
| Cozinha | `binary_sensor.movimento_cozinha_2_occupancy`; `binary_sensor.camera_hub_g2h_7624_motion_sensor`; referências legadas a `binary_sensor.sensor_movimento_cozinha_occupancy`/`_2` em automações | Sem derivado V20 por ambiente identificado | Movimento/atividade local; câmera pode ser fonte complementar se validada | Priorizar `movimento_cozinha_2_occupancy`; validar se câmera e sensores legados ainda são fontes reais | SIM |
| Corredor | `binary_sensor.movimento_corredor_occupancy` | Sem derivado V20 por ambiente identificado | Passagem/circulação | Fonte direta para ambiente ativo, com possível debounce futuro | SIM |
| Dispensa | `binary_sensor.movimento_dispensa_occupancy` | Sem derivado V20 por ambiente identificado | Movimento local | Fonte direta para ambiente ativo | SIM |
| Home office | Nenhum sensor físico de movimento/presença por ambiente identificado nesta descoberta | Sem derivado V20 por ambiente identificado | Rotina local existe por helpers/automação, mas não por sensor de presença encontrado | Requer sensor físico ou fonte semântica própria antes de entrar no Radar | NÃO |
| Quarto Maior | `binary_sensor.movimento_piso_quarto_maior_occupancy`; `binary_sensor.sensor_movimento_quarto_maior_occupancy`; referência a `binary_sensor.presence_sensor_master_bedroom_occupancy`; `binary_sensor.0x00158d0006b0a28b_contact`; `binary_sensor.sensor_janela_quarto_maior_contact`; `binary_sensor.grp_movimento_quarto_maior` | `sensor.casa_porta_quarto_maior_estado_v20`; `sensor.casa_janela_quarto_maior_estado_v20` | Presença/movimento, porta interna e janela | Usar grupo de movimento como candidato semântico; validar sensor de porta marcado no registry como quebrado | SIM |
| Quarto Menor | `binary_sensor.movimento_quarto_menor_occupancy`; `binary_sensor.movimento_piso_quarto_menor_occupancy`; `binary_sensor.sensor_porta_sec_quarto_contact`; `binary_sensor.sensor_janela_quarto_menor_contact`; `binary_sensor.grp_movimento_quarto_menor` | `sensor.casa_porta_quarto_menor_estado_v20`; `sensor.casa_janela_quarto_menor_estado_v20` | Presença/movimento, porta interna e janela | Usar grupo de movimento como candidato semântico; validar área do sensor de piso antes de uso final | SIM |
| Sala de Estar | `binary_sensor.movimento_sala_estar_occupancy`; `binary_sensor.sensor_porta_sala_contact`; `binary_sensor.sensor_varanda_sala_contact` | `sensor.casa_porta_sala_estado_v20`; `sensor.casa_janela_varanda_sala_estado_v20` | Movimento, porta principal e abertura da varanda | Fonte direta para presença local; portas/janelas como contexto do ambiente | SIM |
| Sala de Jantar | `binary_sensor.movimento_sala_jantar_occupancy` | Sem derivado V20 por ambiente identificado | Movimento/atividade local | Fonte direta para ambiente ativo | SIM |
| Casa/global | `binary_sensor.casa_presenca_global`; `binary_sensor.casa_tem_movimento`; legados `binary_sensor.casa_presenca_global_2`, `binary_sensor.casa_presenca_global_v13`, `binary_sensor.casa_atividade_corredor_v17` | `binary_sensor.casa_vazia_v20_2` em shadow; contexto humano V20.2 consome presença global e movimento por cômodo | Estado agregado de presença/casa vazia | Usar apenas como fallback/estado agregado; não representa um cômodo para o mapa | NÃO |
| Lab/técnico/externo | `binary_sensor.sensor_chuva_girasol_rain`; sensores de infraestrutura e entidades registradas em área técnica | `sensor.casa_chuva_estado_v20`; `sensor.casa_vazamento_estado_v20` quando registrado tecnicamente fora do cômodo real | Chuva, vazamento e sinais técnicos/ambientais | Entrar como camada contextual/alerta, não como presença de ambiente | NÃO |

Evolução futura:

#### Fase 1 - Radar simples sob demanda

Objetivo: entregar somente uma visão operacional simples, acionada manualmente, sem ocupar espaço permanente no dashboard principal.

- Criar helper liga/desliga, preferencialmente `input_boolean.casa_radar_movimento_ativo`.
- Exibir o Radar de Movimento apenas quando o helper estiver ligado.
- Listar apenas ambientes com atividade/movimento real no momento.
- Não listar ambientes inativos.
- Não criar inferência semântica nesta fase.
- Não alterar lógica canônica da timeline, contexto ou aliases finais.

Design técnico previsto:

- Package futuro: `packages/radar_movimento_operacional_v20.yaml`.
- O package deve ser isolado, reversível e sem dependência de V20.2 shadow para consumo por dashboard produtivo.
- O package não deve alterar `sensor.status_casa`, V20.1O, aliases finais, timeline, push, agregação ou automações existentes.
- Dashboard produtivo deve consumir somente sensores finais do Radar, nunca sensores brutos nem entidades `_v20_2`.

Helpers previstos:

| Helper | Tipo | Função | Default previsto |
|---|---|---|---|
| `input_boolean.casa_radar_movimento_ativo` | liga/desliga | Habilitar visualização sob demanda do Radar no dashboard | `off` |
| `input_number.casa_radar_movimento_retencao_segundos` | número | Retenção curta para evitar piscar ambiente em transições rápidas | `30` |

Sensores semânticos previstos por ambiente:

| Ambiente | Sensor semântico futuro | Sensores físicos alimentadores | Regra inicial | Entra na Fase 1 |
|---|---|---|---|---|
| Área de Serviço | `binary_sensor.casa_radar_area_servico_ativa_v20` | `binary_sensor.movimento_area_servico_presence` | Ativo quando presença/movimento local estiver `on` | SIM |
| Banheiro | `binary_sensor.casa_radar_banheiro_ativo_v20` | `binary_sensor.movimento_banheiro_occupancy` | Ativo por movimento geral; `binary_sensor.movimento_alto_banheiro_occupancy` fica reservado para banho e não deve ser usado como movimento genérico na Fase 1 | SIM |
| Cozinha | `binary_sensor.casa_radar_cozinha_ativa_v20` | `binary_sensor.movimento_cozinha_2_occupancy` | Ativo quando movimento local estiver `on`; câmera e sensores legados ficam apenas como candidatos futuros | SIM |
| Corredor | `binary_sensor.casa_radar_corredor_ativo_v20` | `binary_sensor.movimento_corredor_occupancy` | Ativo quando movimento local estiver `on` | SIM |
| Dispensa | `binary_sensor.casa_radar_dispensa_ativa_v20` | `binary_sensor.movimento_dispensa_occupancy` | Ativo quando movimento local estiver `on` | SIM |
| Quarto Maior | `binary_sensor.casa_radar_quarto_maior_ativo_v20` | `binary_sensor.grp_movimento_quarto_maior`; fallback documentado: `binary_sensor.movimento_piso_quarto_maior_occupancy`, `binary_sensor.sensor_movimento_quarto_maior_occupancy` | Ativo pelo grupo semântico se disponível; fallback só após validação runtime | SIM |
| Quarto Menor | `binary_sensor.casa_radar_quarto_menor_ativo_v20` | `binary_sensor.grp_movimento_quarto_menor`; fallback documentado: `binary_sensor.movimento_quarto_menor_occupancy`, `binary_sensor.movimento_piso_quarto_menor_occupancy` | Ativo pelo grupo semântico se disponível; fallback só após validação runtime | SIM |
| Sala de Estar | `binary_sensor.casa_radar_sala_estar_ativa_v20` | `binary_sensor.movimento_sala_estar_occupancy` | Ativo quando movimento local estiver `on`; porta/varanda entram apenas como contexto, não como presença | SIM |
| Sala de Jantar | `binary_sensor.casa_radar_sala_jantar_ativa_v20` | `binary_sensor.movimento_sala_jantar_occupancy` | Ativo quando movimento local estiver `on` | SIM |

Sensores agregados previstos:

| Sensor | Tipo | Função |
|---|---|---|
| `binary_sensor.casa_radar_algum_ambiente_ativo_v20` | binary_sensor | Indicar se qualquer ambiente elegível está ativo |
| `sensor.casa_radar_ambientes_ativos_v20` | sensor | Expor lista curta de ambientes ativos para o dashboard |
| `sensor.casa_radar_resumo_v20` | sensor | Expor texto simples, por exemplo `Banheiro, Cozinha` ou `Sem movimento` |

Regras de elegibilidade da Fase 1:

- Ambiente entra apenas se possuir sensor físico ou grupo semântico local já identificado.
- Sensores de porta, janela, chuva, vazamento e infraestrutura não contam como presença/movimento de ambiente na Fase 1.
- Sensores globais como `binary_sensor.casa_presenca_global` e `binary_sensor.casa_tem_movimento` podem ser referência de fallback diagnóstico, mas não devem acender cômodos individualmente.
- Sensores `_v20_2` não devem ser consumidos por dashboard produtivo.
- Sensor com área indefinida ou estado suspeito deve ficar documentado como candidato, mas não deve entrar como fonte primária sem validação runtime.
- Ambientes sem fonte local, como Home office nesta descoberta, ficam fora da Fase 1.

Consumo pelo dashboard:

- O dashboard deve exibir ou ocultar a seção do Radar a partir de `input_boolean.casa_radar_movimento_ativo`.
- O dashboard deve consumir `sensor.casa_radar_ambientes_ativos_v20` e/ou os `binary_sensor.casa_radar_*_ativo_v20` finais.
- O dashboard não deve calcular presença, agrupar sensores físicos, aplicar debounce ou consultar `_v20_2`.
- Quando o helper estiver `off`, nenhum card, badge ou chip do Radar deve aparecer.
- Quando o helper estiver `on` e não houver ambiente ativo, o dashboard deve mostrar estado vazio simples ou ocultar chips de ambiente.

Critérios de aceite da Fase 1:

- Com helper `off`, Radar invisível no dashboard.
- Com helper `on`, somente ambientes ativos aparecem.
- Ambientes inativos não aparecem.
- Atividade em cada sensor físico primário acende apenas o ambiente correspondente.
- Banho/chuveiro não vira movimento genérico do banheiro por causa de `binary_sensor.movimento_alto_banheiro_occupancy`.
- Dashboard produtivo não consome entidades `_v20_2`.
- Nenhuma alteração em `sensor.status_casa`, V20.1O, timeline, push, agregação, automações ou aliases finais.
- IA desligada não altera o funcionamento do Radar.

Rollback previsto:

- Desligar `input_boolean.casa_radar_movimento_ativo` remove o Radar do dashboard.
- Remover futuramente o package `packages/radar_movimento_operacional_v20.yaml` deve eliminar apenas sensores/helpers do Radar, sem afetar contratos V20 existentes.
- Dashboard deve continuar funcional sem o package do Radar.
- Se um sensor físico gerar falso positivo, retirar o ambiente da lista sem alterar motores V20.1O ou V20.2.

#### Fase 2 - Camada semântica de presença por ambiente

Objetivo: consolidar leituras físicas em estados semânticos por ambiente.

- Unificar PIR, mmWave, ocupação, presença e sensores específicos por cômodo.
- Separar sensor físico bruto de presença semântica por ambiente.
- Expor estados finais do tipo ambiente ativo/inativo, sem lógica pesada no dashboard.
- Tratar múltiplas fontes do mesmo ambiente com prioridade e tolerância a ruído.
- Manter a experiência sob demanda.

#### Fase 3 - Planta baixa operacional

Objetivo: evoluir de lista textual para mapa visual da casa.

- Criar planta baixa da casa em fase futura, sem substituir a lista simples da Fase 1.
- Fazer ambientes acenderem/destacarem conforme atividade.
- Manter fallback textual para ambientes ativos.
- Evitar que a planta baixa vire fonte de verdade; ela deve consumir sensores finais/semânticos.

#### Fase 4 - Histórico de movimentação e integração contextual

Objetivo: permitir leitura temporal da movimentação sem transformar o radar em log permanente.

- Registrar histórico dos últimos movimentos por ambiente.
- Integrar com timeline/contexto apenas quando houver evento operacional relevante.
- Evitar spam de movimento bruto na timeline.
- Diferenciar movimento atual, último ambiente ativo e sequência recente.
- Permitir reconstrução humana de contexto recente sem alterar prioridades canônicas.

#### Fase 5 - Integração futura com CMDB e IA opcional

Objetivo: enriquecer interpretação da movimentação com contexto de inventário e IA, mantendo o sistema determinístico por padrão.

- Relacionar ambientes, dispositivos e sensores via CMDB futura.
- Permitir que IA opcional explique padrões de movimentação ou anomalias.
- IA não deve ser dependência para detecção de movimento, presença ou alertas críticos.
- IA desligada deve manter o Radar/Mapa Operacional funcional.
- Usar IA apenas para explicação, recomendação e enriquecimento contextual.

Critério de compatibilidade:

- A V20.2 pode melhorar UX e organização, mas não deve quebrar os contratos da V20.0.
- Mudanças estruturais profundas de criticidade devem permanecer no escopo da V21.

## V21 - Criticidade Contextual Dinâmica

Objetivo: transformar pesos estáticos em uma matriz contextual.

Direções:

- Combinar peso base, severidade, horário, presença, modo dormir e recorrência.
- Ajustar criticidade conforme contexto humano.
- Criar decomposição explicável do score operacional.
- Separar criticidade técnica de relevância humana.
- Normalizar contratos finais sem expor sensores versionados em dashboards.

## V22 - Motor Semântico

Objetivo: evoluir a leitura operacional para uma camada semântica determinística.

Direções:

- Criar motor de narrativa curta baseado nos eventos recentes.
- Consolidar contexto humano atual.
- Criar síntese de causa/efeito entre eventos.
- Melhorar fallback textual sem depender de LLM.
- Reduzir ruído de eventos repetitivos.

## V23 - Observabilidade Operacional

Objetivo: tornar o sistema mais auditável e diagnosticável.

Direções:

- Painel de saúde dos motores.
- Métricas de eventos por origem.
- Diagnóstico de sensores indisponíveis.
- Histórico de score e prioridade.
- Testes assistidos mais completos.
- Validação de contratos finais.

## V24 - IA, LLM e Contexto Adaptativo

Objetivo: adicionar inteligência contextual opcional sem substituir a camada determinística.

Regra principal:

- IA desligada: a Central Operacional continua funcionando 100% pelo motor determinístico.
- IA ligada: a IA apenas enriquece análise, contexto, explicações e recomendações.
- IA nunca deve ser dependência obrigatória para eventos críticos.
- Eventos críticos devem continuar sendo detectados, publicados e tratados por sensores, templates e motores determinísticos.
- O dashboard deve ter controle claro de estado da IA:
  - `IA desligada`
  - `IA ativa`
- Idealmente, o controle deve evoluir para modos:
  - `IA desligada`
  - `IA leve`
  - `IA completa`

Direções:

- Resumos semânticos opcionais.
- Explicação contextual de incidentes.
- Sugestões de ação baseadas em estado da casa.
- Detecção de padrões recorrentes.
- Integração com LLM apenas sobre dados já normalizados.
- Manter IA como camada opcional de enriquecimento, nunca como motor primário de decisão crítica.

## Princípios de Evolução

- Preservar aliases finais sem versão.
- Não acoplar dashboards a sensores versionados.
- Não reativar V19.
- Não alterar automações antigas para corrigir motores V20/V21.
- Documentar todo hotfix de baseline.
- Preferir motores determinísticos antes de IA/LLM.
- Garantir que o sistema continue operacional quando a IA estiver desligada.
- Nunca tornar IA dependência obrigatória para eventos críticos.
