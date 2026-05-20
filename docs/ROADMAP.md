# Roadmap da Central Operacional

## Estado Atual

- V20.0 = concluída e congelada
- V20.1A = implementada
- V20.1B = camada oficial de produção
- V20.1C = auditoria de legado planejada
- V20.2 = parcialmente implementada em shadow mode/paralelo
- V21+ = planejamento futuro

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

Status: concluída em camada de eventos, com legado preservado.

Escopo real:

- migração da camada de eventos
- timeline V20
- feed operacional V20
- sensores determinísticos V20
- camada semântica inicial
- redução de parsing textual
- publicação estruturada de eventos operacionais

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

Status: planejada.

Objetivo: auditar o legado antes de qualquer desativação controlada.

Direções:

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

## V20.2 - Dedicated Engines + UX/Operational Layout

Status: parcialmente implementada em shadow mode/paralelo; checkpoint de homologação Fase A concluído.

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
