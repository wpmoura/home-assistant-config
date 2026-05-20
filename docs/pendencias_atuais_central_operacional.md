# Pendências Atuais - Central Operacional Home Assistant

Data do levantamento: 2026-05-16

## Escopo do Diagnóstico

Foram analisados:

- documentação em `docs/`
- packages em `packages/`
- dashboards Lovelace em `.storage/lovelace*`
- roadmap, release, changelog e baseline
- matriz e execução de testes V20.2
- packages V20, V20.1B e V20.2

Este relatório não altera código, packages, dashboards, sensores ou automações. Ele consolida pendências abertas e riscos conhecidos.

## 1. Pendências Abertas por Versão/Fase

### V20.0 - Baseline Congelada

Status: concluída e congelada.

Pendências residuais:

- Confirmar se o release V20.0 em `docs/release_central_operacional_v20.md` deve ser atualizado para refletir as evoluções V20.1A/V20.1B/V20.2 ou permanecer como documento histórico congelado.
- Resolver a divergência documental entre V20.0 congelada e documentos posteriores que já descrevem V20.1/V20.2.
- Garantir que commits/tags da baseline não incluam arquivos sensíveis ou dashboards `.storage` indevidos.

### V20.1A - Operational Control Layer

Status: implementada.

Pendências:

- Confirmar validação completa dos helpers de controle:
  - `input_number.casa_timeline_max_eventos`
  - `input_boolean.casa_timeline_evento_*`
- Confirmar se todos os helpers aparecem corretamente no painel administrativo.
- Confirmar se a alteração do limite máximo de eventos da timeline funciona após reload/restart.
- Registrar evidências formais dos testes de bloqueio/publicação por helper.

### V20.1B - Legacy Migration Layer

Status: implementada na camada de eventos, com legado preservado.

Pendências:

- Não considerar V20.1B como migração completa das automações da casa.
- Auditar automações, blueprints e scripts legados antes de qualquer desativação.
- Mapear side-effects legados que podem atualizar helpers, `input_text`, sensores, modos, score ou contexto.
- Separar notificações textuais antigas de ações físicas/recovery.
- Validar duplicidades externas de notificação/push fora da timeline V20.
- Resolver em fase futura a ordem semântica de recuperação internet/failover:
  - desejado: `📡 Failover 4G encerrado` antes de `🌐 Internet normalizada`.
- Validar se intensidade de chuva, banho, vazamento, portas internas e janelas estão plenamente documentados com evidências reais.

### V20.1C - Legacy Decommission Audit

Status: planejada.

Pendências:

- Criar inventário de consumidores do legado.
- Classificar automações por risco:
  - mensagem apenas
  - mensagem + helper
  - ação física
  - recovery
  - segurança/alarme
- Validar dashboards, scripts e packages que ainda leem fontes legadas.
- Definir lotes pequenos e reversíveis para desativação futura.
- Criar plano de rollback por domínio.
- Só remover duplicidades após validação real.

### V20.2 Lote 1 - Contexto Base

Status: implementado em paralelo.

Sensores envolvidos:

- `binary_sensor.casa_vazia_v20_2`
- `binary_sensor.contexto_noturno_v20_2`
- `sensor.casa_contexto_temporal_v20_2`
- `sensor.casa_contexto_humano_v20_2`
- `sensor.casa_contexto_operacional_v20_2`
- `sensor.casa_contexto_ambiental_v20_2`

Pendências:

- Registrar evidências de validação manual dos contextos base.
- Confirmar comportamento de presença/casa vazia em cenários reais.
- Confirmar comportamento de contexto ambiental com chuva, banho e janelas.
- Validar que nenhum alias final consome esses sensores.
- Confirmar rollback simples removendo/desabilitando `packages/motor_contexto_v20_2.yaml`.

### V20.2 Lote 2A - Relevância Contextual

Status: implementado como prova mínima.

Sensores envolvidos:

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_evento_relevante_v20_2`
- `sensor.casa_motivo_relevancia_v20_2`

Pendências:

- Registrar evidências reais para as quatro regras mínimas:
  - porta aberta + casa vazia = `alta`
  - chuva ativa + janela aberta = `critica`
  - internet degradada + noturno = `media`
  - energia ausente + alguém em casa = `critica`
- Confirmar que evento crítico concorrente prioriza corretamente `chuva_janela_aberta`.
- Validar comportamento quando não há evento contextual: `baixa` + `nenhum_evento_contextual`.
- Definir se o score contextual futuro será atributo ou sensor próprio.
- Manter sem integração com timeline, aliases ou score oficial até concluir a Fase 1A.

### V20.2 Lote 2B - Confidence & Stability Shadow Sensors

Status: implementado e carregado no Home Assistant.

Sensores envolvidos:

- `sensor.casa_confianca_contextual_v20_2`
- `sensor.casa_estabilidade_contextual_v20_2`

Baseline observada:

- confiança: `alta`
- estabilidade: `estavel`
- `shadow_mode: true`
- `estabilidade_temporal_real: false`
- `fontes_invalidas: nenhuma`
- `fontes_contraditorias: nenhuma`
- `contradicao_detectada: false`
- `dominio_estimado: nenhum`
- `dominio_oscilante: false`

Pendências:

- Executar matriz de testes reais da Fase 1A.
- Validar contradições reais ou simuladas.
- Validar domínios oscilantes como `observando`.
- Confirmar que `indeterminada` não combina com `estavel`.
- Implementar memória temporal real somente em fase posterior.
- Ainda não criar `sensor.casa_evento_contextual_estavel_v20_2`.
- Não conectar confidence à relevância oficial ainda.

## 2. Pendências Técnicas

- Decomposição explicável de `sensor.casa_score_operacional` ainda não implementada.
- `sensor.casa_score_operacional` permanece simplificado.
- WAN/4G ainda possui parte da lógica em motor separado/versionado.
- Contrato final sem versão para WAN/4G ainda pendente.
- Feed histórico real ainda depende da memória atual da timeline; não reconstrói histórico antigo do Recorder.
- Timeline V20 começa a registrar a partir do reload/restart, sem reconstrução retroativa.
- Ordenação semântica de eventos correlacionados ainda pendente.
- Confidence/stability ainda não têm debounce, cooldown, hysteresis ou temporal decay reais.
- Contexto V20.2 é paralelo e ainda não influencia decisão oficial.
- Harness de testes shadow ainda é apenas proposta documental.
- Radar de Movimento por cômodo está registrado como backlog, sem implementação.
- IA/LLM está documentada como camada opcional futura, sem implementação.

## 3. Pendências de Documentação

- Atualizar `docs/release_central_operacional_v20.md` ou decidir que ele permanece congelado como baseline V20.0 histórica.
- Consolidar `docs/CHANGELOG.md` com V20.1A, V20.1B e V20.2, caso o changelog oficial deva acompanhar fases posteriores.
- Registrar resultados reais em `docs/execucao_testes_reais_v20_2_fase_1a.md`.
- Atualizar documentação de arquitetura com um mapa mais claro entre:
  - V20.1B determinística
  - V20.2 contexto
  - V20.2 relevância
  - V20.2 confiança
- Documentar explicitamente quais arquivos são históricos, quais são baseline e quais são planejamento futuro.
- Decidir se `docs/roadmap_central_operacional_semantic_house_v_26.md` continua como referência ativa ou documento histórico.
- Criar registro de validação dos reloads/restarts necessários por fase.

## 4. Pendências de Testes

Arquivo principal:

- `docs/matriz_testes_reais_v20_2_fase_1a.md`

Arquivo de execução:

- `docs/execucao_testes_reais_v20_2_fase_1a.md`

Pendências:

- Todos os testes U-001 a U-017 permanecem pendentes no arquivo de execução.
- Todos os testes I-001 a I-008 permanecem pendentes.
- Todos os testes R-001 a R-005 permanecem pendentes.
- Todos os testes B-001 a B-006 permanecem pendentes.
- Todos os testes O-001 a O-005 permanecem pendentes.
- Critérios de saída da Fase 1A ainda não foram marcados como `OK`.
- Pelo menos um teste integrado crítico precisa ser validado com evidência.
- Validar que timeline/feed não recebem eventos da camada shadow.
- Validar que aliases finais não apontam para sensores V20.2.
- Validar que `sensor.status_casa` não é alterado por sensores shadow.
- Validar rollback simples dos packages V20.2.

## 5. Pendências de Dashboard/Interface

Dashboards oficiais e administrativos:

- `.storage/lovelace.sistema_casa`
- `.storage/lovelace.dashboard_lixo`
- `.storage/lovelace.testes_anterior`
- `.storage/lovelace.debug_operacional`
- `.storage/lovelace_dashboards`

Pendências:

- Confirmar se o menu lateral aponta apenas para dashboards V20 oficiais.
- Confirmar se dashboards produtivos não consomem sensores V20.2 experimentais.
- Avaliar se haverá um dashboard de debug V20.2 separado para contexto/relevância/confiança.
- Implementar futuramente seção sob demanda `Radar de Movimento`.
- Adicionar controle futuro de IA:
  - `IA desligada`
  - `IA leve`
  - `IA completa`
- Validar que dashboards legados com V19 continuam preservados apenas como laboratório/histórico.
- `.storage/lovelace.teste_4` ainda contém múltiplas referências V19; isso é aceitável somente se permanecer legado/laboratório.
- `.storage/lovelace.debug_operacional` e `.storage/lovelace.testes_anterior` ainda exibem fontes legadas como `sensor.central_ultima_mensagem`; isso é aceitável como debug/fonte real, mas não deve virar dependência de decisão V20.2.

## 6. Pendências Futuras / Backlog

### V20.1C

- Legacy Decommission Audit.
- Mapeamento de dependências invisíveis.
- Auditoria de side-effects.
- Desativação controlada do legado.

### V20.2 / V20.3

- Semantic Timeline Refinement:
  - `timestamp_ocorrencia`
  - `timestamp_confirmacao`
- Correlation and Event Ordering.
- Confidence com memória temporal real.
- Cooldown, debounce, hysteresis e temporal decay reais.
- Integração controlada entre relevância, confiança e decisão contextual.
- Harness de testes shadow real em `packages/test_harness_v20_2.yaml`, se a execução manual não for suficiente.
- Radar de Movimento sob demanda.

### V21

- Criticidade contextual dinâmica.
- Score explicável por contexto, impacto, duração, redundância, presença e horário.
- Separação entre criticidade técnica e relevância humana.

### V22

- Motor semântico determinístico.
- Narrativa curta sem depender de LLM.
- Síntese de causa/efeito entre eventos.

### V23

- Observabilidade operacional.
- Métricas por motor.
- Diagnóstico de sensores indisponíveis.
- Histórico de score e prioridade.

### V24

- IA/LLM opcional.
- IA desligada deve manter o sistema 100% determinístico.
- IA ligada apenas enriquece análise, contexto e recomendações.
- IA nunca deve ser dependência obrigatória para eventos críticos.

## 7. Itens Implementados mas Ainda Não Validados Formalmente

- `packages/motor_contexto_v20_2.yaml`
- `packages/motor_relevancia_v20_2.yaml`
- `packages/motor_confianca_v20_2.yaml`
- Matriz de testes reais V20.2 Fase 1A.
- Cópia operacional de execução dos testes.
- Proposta de harness de testes shadow.
- Radar de Movimento registrado no roadmap.
- Princípio arquitetural de IA opcional registrado no roadmap/arquitetura.
- V20.1C registrada como fase futura, mas ainda sem execução.

## 8. Itens que Dependem de Reinício/Reload do Home Assistant

Dependem de reload de templates/packages ou reinício:

- Novos packages:
  - `packages/motor_contexto_v20_2.yaml`
  - `packages/motor_relevancia_v20_2.yaml`
  - `packages/motor_confianca_v20_2.yaml`
- Alterações em helpers de `packages/parametros_operacionais_v20.yaml`.
- Qualquer package futuro:
  - `packages/test_harness_v20_2.yaml`
  - package futuro do Radar de Movimento
  - package futuro de IA/controle de modo IA

Já observado:

- `packages/motor_confianca_v20_2.yaml` carregou corretamente após correção de atributos.

Ainda pendente:

- Registrar evidência de reload/restart para contexto e relevância V20.2.
- Registrar evidência de que os sensores permanecem após reinício.

## 9. Riscos Conhecidos

- Desativar automações antigas sem auditoria pode causar regressões silenciosas.
- Algumas automações legadas podem ter side-effects além de notificações.
- Dashboards legados com V19 podem confundir navegação se aparecerem como oficiais.
- `.git` grande e banco SQLite grande podem continuar impactando backup Google.
- Versionar `.storage` sensível ou banco SQLite continua sendo risco de segurança e tamanho.
- Score contextual futuro pode ficar opaco sem atributos explicáveis.
- Confidence sem memória temporal real pode dar falsa estabilidade.
- Integração prematura de V20.2 com aliases finais pode quebrar estabilidade da V20.1B.
- IA/LLM pode degradar performance se não for opcional e sob demanda.
- Radar de movimento permanente pode poluir dashboard principal; deve ser sob demanda.
- Eventos correlacionados ainda podem aparecer em ordem semanticamente imperfeita.

## 10. Próximo Passo Recomendado

Próximo passo recomendado: executar e preencher a Fase 1A de testes reais antes de implementar novas camadas.

Ordem sugerida:

1. Validar baseline normal:
   - `sensor.casa_relevancia_contextual_v20_2 = baixa`
   - `sensor.casa_evento_relevante_v20_2 = nenhum_evento_contextual`
   - `sensor.casa_confianca_contextual_v20_2 = alta`
   - `sensor.casa_estabilidade_contextual_v20_2 = estavel`
2. Preencher `docs/execucao_testes_reais_v20_2_fase_1a.md`.
3. Executar pelo menos os testes críticos:
   - I-001 porta aberta sem ninguém em casa
   - I-003 chuva + janela aberta
   - I-007 internet degradada + noturno
   - I-008 energia ausente + alguém em casa
4. Confirmar que timeline/feed, aliases finais e `sensor.status_casa` não são alterados pela camada shadow.
5. Só depois decidir se vale criar o harness real `packages/test_harness_v20_2.yaml`.
6. Após validação, preparar commit seletivo apenas dos arquivos V20.2/documentação realmente relacionados.

