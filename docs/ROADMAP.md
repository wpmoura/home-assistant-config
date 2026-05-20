# Roadmap da Central Operacional

## Estado Atual

- V20.0 = concluída e congelada
- V20.1A = concluída
- V20.1B lote 1 = concluída
- V20.1B lote 2 = concluída/parcial para energia, internet, failover e backup
- V20.1C = auditoria de legado concluída em diagnóstico
- V20.1D/E = checkpoint documental de dependências e impacto concluído
- V20.1F/G/H = validação e investigação de V19 concluídas
- V20.1I/J = isolamento controlado de packages desativados aplicado e validado
- V20.1K = concluída; tag `V20.1K_FECHAMENTO` criada
- V20.2A = concluída; dashboard legado `teste-4` removido pela UI
- V20.2B = auditoria executada, sem ação operacional
- V20.2/V20.3/V21 = planejamento futuro

Estado operacional consolidado:

- Workspace limpo no último checkpoint validado.
- Sem resíduos V19 conhecidos em dashboards ativos.
- Dashboard legado `teste-4` removido pela UI/fluxo suportado.
- Auditoria de automações identificou 21 órfãs; automações críticas não devem ser removidas automaticamente.
- Limpeza técnica futura deve seguir criticidade, em lotes pequenos e reversíveis.

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
- V20.2 permanece isolada em shadow
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
- Limpeza técnica futura
- Radar de Movimento sob demanda
- Assistente Contextual Preventivo
- Weather Risk Engine
- Presence Intelligence
- Observabilidade operacional
- Energy Brain
- House Exposure Engine
- IA contextual opcional

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

Status: concluída em diagnóstico.

Objetivo: auditar o legado antes de qualquer desativação controlada.

Direções:
- Documento inicial criado em `docs/auditoria_legado_v20_1c.md`.
- Inventário preliminar de V19/V20.2 e pacotes de suporte em análise.

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

## V20.2 - Dedicated Engines + UX/Operational Layout

Status: parcialmente implementada em shadow mode/paralelo; checkpoint de homologação Fase A concluído; auditoria operacional residual iniciada.

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

### V20.2A - Legacy Dashboard Review

Status: validada por inspeção e remoção externa confirmada.

Objetivo: avaliar o dashboard oculto legado `teste-4` antes de qualquer remoção.

Resultado:

- Dashboard `Laboratório - Casa Inteligente` (`teste-4`) foi classificado como legado V19 sem vínculo operacional encontrado.
- Remoção foi validada posteriormente por inspeção: `teste_4` não aparece mais em `.storage/lovelace_dashboards` e `.storage/lovelace.teste_4` não existe mais.
- Referências V19 não aparecem em dashboards ativos após a remoção.
- Permanecem apenas referências documentais/históricas.

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

Evolução futura:

- Mapa/planta baixa da casa com cômodos acendendo conforme movimento.
- Histórico dos últimos movimentos.
- Modo monitoramento temporário.
- Possível integração futura com IA/LLM para interpretação contextual da movimentação.

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
