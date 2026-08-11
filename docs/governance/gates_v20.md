# Gates V20

Data: 2026-05-20
Status: ATIVO

## Objetivo

Definir criterios obrigatorios para encerramento de fases da Central Operacional V20.

Nenhuma fase deve ser considerada concluida sem gate documental correspondente.

## Gate corretivo V20.1Q — Recovery 4G

- [x] Tentativas genéricas e snapshot do máximo implementados estaticamente.
- [x] Tempo OFF único e helpers numerados marcados como legado.
- [x] Confirmação de queda e estabilização de retorno parametrizadas separadamente.
- [x] Cooldown restrito ao esgotamento e `ultima_execucao` com semântica documentada.
- [x] Cancelamentos sem cooldown implementados.
- [x] Validação YAML e buscas estáticas executadas.
- [x] Dashboard "Parâmetros" reorganizado em três cards (Operação, Ciclo de Recuperação, Avisos) conforme especificação oficial de UX.
- [x] Especificação oficial de UX documentada e versionada em `docs/ux/espec_ux_param_recovery4g.md`.
- [x] Commit e push da entrega de UX do dashboard "Parâmetros" realizados, com rastreabilidade em `CHANGELOG.md` e `docs/releases/implementation_plan_v20_1q.md`.
- [ ] Validação visual do critério A8 da especificação de UX (nenhum rótulo quebra em duas linhas em viewport de celular) — sem evidência visual registrada.
- [x] Cenários 1, 5 e 10 homologados no runtime com quedas reais controladas (Testes 1, 3 e 2, respectivamente). Cenário 2 considerado suficientemente coberto por generalização de código (laço genérico único, sem hardcode por valor, confirmado por leitura de código e por execução real em três valores distintos) e não é bloqueador de encerramento.
- [x] Snapshot dos parâmetros (`max_tentativas_ciclo`) validado contra alteração de helper em pleno ciclo — prova definitiva no Teste 2.
- [x] Cooldown homologado — entrada e expiração, com trace de ação real da transição `cooldown → ocioso` (Testes 1 e 2).
- [x] Religamento de segurança e proteção da tomada contra permanência desligada homologados (Testes 1–3).
- [x] Erro técnico seguro / falha intermediária sem decisão autônoma do Executor homologado (achado real não planejado no Teste 3: atraso de confirmação da tomada tratado corretamente, sem corromper o ciclo).
- [x] Restart durante ciclo ativo homologado (reconciliação limpa, sem cooldown).
- [x] Timeline validada com o limite de 16 eventos em produção.
- [ ] Oscilação, `unknown` e `unavailable` dos sensores — sem ocorrência real observada em nenhum teste; permanece sem evidência.
- [ ] Janela de estabilização igual a zero — parametrizada em três execuções (incluindo a repetição do Teste 3 em 2026-07-20 com `estabilizacao_retorno_minutos=0` e monitoramento por assinatura de eventos), mas nunca exercida de fato: em nenhuma tentativa houve retorno de `backup_4g_operacional` durante a janela de validação.
- [ ] Retorno estabilizado em índice intermediário (sucesso antes do esgotamento) — não obtido em nenhuma das quatro quedas reais tentadas até agora; a duração real das quedas variou de ~2min30s a ~6min55s, sempre excedendo a janela de tentativas configurada no momento.
- [ ] Cancelamento pelo operador em ciclo ativo — não exercitado. Na primeira tentativa (Teste 3, 2026-07-18) faltou por limitação de monitoramento (polling); na repetição (2026-07-20), o monitor por assinatura de eventos foi validado e usado com sucesso, mas o cancelamento não foi tentado porque a execução foi mantida estritamente passiva por instrução explícita do usuário (sem alteração automática de helper).
- Interrupção por falta de energia em ciclo ativo — **classificada como risco residual aceito**, não bloqueador. Mesma condição de código do cancelamento pelo operador, já comprovada por analogia estrutural (leitura de código), mas sem execução real com ciclo ativo. Não deve ser forçada deliberadamente.
- [x] Achado arquitetural: guard rail `tomada_ja_desligada` confirmado — o orquestrador recusa iniciar um ciclo se a tomada já estiver desligada no momento da solicitação (`fato: solicitacao_bloqueada`). Descoberto na repetição do Teste 3 (2026-07-20); implica que o mecanismo de "provocar queda" via tomada deve sempre religá-la antes da janela de confirmação, ou usar uma fonte de queda que não envolva a tomada.
- [x] Guard de manutenção implementado estaticamente: `input_boolean.casa_comunicacao_modo_manutencao` inicia desligado e bloqueia somente novas execuções de `automation.central_recovery_4g_solicitar` quando ligado. O orquestrador, ciclos já iniciados e `automation.central_recovery_4g_religamento_seguranca` não consomem esse helper.

### Nota de rigor metodológico — 2026-07-20

Uma afirmação anterior de que o padrão observado de "retorno real poucos segundos após o esgotamento" seria "quase estrutural" foi revisada e **reclassificada como hipótese não comprovada**, não como achado. Evidência disponível (leitura de código do sensor `backup_4g_operacional`/`internet_wan2_4g_ok` e histórico das sondas de latência subjacentes) mostra que a detecção está próxima do retorno real (sem indício de retorno silencioso não detectado), mas a amostra de apenas 3 quase-acertos e 1 queda longa, com duração total variando de ~2min30s a ~6min55s, não sustenta a existência de um tempo de reconexão fixo/estrutural da operadora. Ver detalhamento completo em `docs/releases/implementation_plan_v20_1q.md`.

### Estado da homologação runtime — Suspensa

**Status:** Homologação Suspensa.

**Motivo:** interrupção por decisão operacional. Não existe bloqueio técnico conhecido. A implementação permanece válida.

**Evidências preservadas (não precisam ser repetidas):** Recovery 4G funcional de ponta a ponta; snapshot dos parâmetros; parametrização das tentativas (cenários 1, 5 e 10); cooldown; expiração do cooldown; religamento de segurança; tratamento de erro técnico; Timeline; estados do Executor; guard rail `tomada_ja_desligada`; monitoramento por assinatura de eventos (WebSocket) formalmente validado (6/6 critérios) e usado com sucesso.

**Permanecem para retomada futura:** cancelamento pelo operador em ciclo ativo; retorno antes do esgotamento (índice intermediário); janela de estabilização igual a zero.

**Limitação metodológica registrada:** a proximidade observada entre esgotamento e retorno real da conectividade não deve ser tratada como propriedade estrutural do sistema — amostra pequena (3 casos), duração real das quedas variável (~2min30s–6min55s). Ver "Nota de rigor metodológico" acima.

Detalhamento completo dos testes executados (Teste 1, Teste 2, Teste 3 e sua repetição em 2026-07-20), evidências, fatos vs. hipóteses e próxima etapa recomendada em `docs/releases/implementation_plan_v20_1q.md`.

Quatro power cycles reais e controlados foram autorizados e executados pelo usuário durante as rodadas de homologação runtime (Testes 1, 2, 3 e sua repetição), conforme Gate pré-teste físico.

## Gate 0 - Escopo

Obrigatorio antes de iniciar.

- Fase nomeada.
- Objetivo declarado.
- Limites declarados.
- Itens fora de escopo declarados.
- Tipo da fase definido: documentacao, auditoria, shadow, implementacao, migracao ou limpeza.

## Gate 1 - Contratos protegidos

Obrigatorio para qualquer fase que possa afetar operacao.

Validar que nao houve alteracao indevida em:

- `sensor.status_casa`
- aliases finais sem versao
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- dashboards produtivos
- automacoes criticas

## Gate 2 - Arquitetura

Obrigatorio para novas camadas, motores ou decisoes operacionais.

- A fase respeita a Constituicao V20.
- Nao cria inteligencia paralela sem classificacao.
- Roadmap nao esta sendo usado como implementacao.
- YAML nao redefine arquitetura.
- Camada shadow permanece desacoplada ate promocao formal.

## Gate 3 - Evidencia

Obrigatorio para homologacao.

- Evidencias ou resultados registrados.
- Falhas, bloqueios e parciais declarados.
- Ausencia de spam, duplicidade e `unavailable` avaliada quando aplicavel.
- Impacto em timeline/event feed declarado quando aplicavel.

## Gate 4 - Rollback

Obrigatorio para implementacao, migracao ou limpeza.

- Rollback simples identificado.
- Lote pequeno e reversivel.
- Nenhuma edicao manual de `.storage` sem fase propria.
- Nenhuma remocao automatica de automacoes orfas/desabilitadas.

## Gate 5 - Documentacao

Obrigatorio no encerramento.

- Changelog ou checkpoint atualizado.
- Roadmap consolidado atualizado quando houver impacto futuro.
- Backlog tecnico atualizado quando houver pendencia.
- Auditoria registrada quando a fase for diagnostica.

## Gate 6 - Status final

Status permitido:

- `Planejada`
- `Em diagnostico`
- `Implementada em shadow`
- `Homologada`
- `Concluida`
- `Bloqueada`
- `Parcial`
- `Arquivada`

Uma fase homologada deve ter escopo fechado e nao deve continuar acumulando mudancas sem nova fase.

## Gate de Promoção Limitada V20.2C — Sessão de Monitoramento Remoto

Status documental: I1/I2/I2A/I3A/I3B HOMOLOGADOS; CONSUMIDORES PERMANECEM PENDENTES

Este Gate é obrigatório antes de habilitar o Coordenador da Sessão de Monitoramento Remoto (CSMR), publicar eventos da V20.2C na Timeline ou liberar consumidores subordinados pelo contrato de sessão.

### Gate arquitetural

- [x] Promoção limitada registrada no despacho `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md`.
- [x] CSMR classificado como motor oficial de coordenação operacional com escopo restrito.
- [x] Restante da V20.2 e Context Engine original preservados em shadow.
- [x] V20.1O preservada como autoridade canônica da Timeline e Event Feed.
- [x] Fronteiras entre CSMR, publicador e consumidores definidas.
- [x] Plano técnico restrito definido e decisões DP-1 a DP-5 resolvidas documentalmente em `docs/v20_2c/plano_tecnico_csmr.md`.

### Gate de contrato

- [x] Eventos autorizados limitados a `📍 Wilson saiu de casa`, `🛡️ Monitoramento remoto iniciado`, `📍 Wilson chegou em casa` e `🛡️ Monitoramento remoto encerrado`.
- [x] Formato público `HH:MM mensagem` preservado.
- [x] Escrita direta em aliases finais, Timeline e Event Feed proibida.
- [x] Timeline, Event Feed, histórico e deduplicação paralelos proibidos.
- [x] Caminho canônico decidido: script V20.1O com payload versionado, ACK correlacionado e preservação de `sensor.casa_evento_publicavel_v20`.
- [x] Script canônico, ACK e ledger idempotente implementados e validados em `test_mode` pelo lote V20.2C-I1.

### Gate de comportamento

Fundação transacional I2 homologada:

- [x] Estados mínimos `idle`, `starting`, `active`, `ending` e `failed` observados no Harness isolado.
- [x] `session_id` único preservado durante abertura, atividade, encerramento e falha.
- [x] Reenvio por `request_id`, transições inválidas e abertura concorrente tratados deterministicamente.
- [x] Falhas de abertura/encerramento e recuperação explícita comprovadas sem publicação ou consumidor.
- [x] Checkpoint ativo e idempotência preservados após reload parcial.
- [x] Ausência de trigger de presença/startup comprovada; `away + idle` não possui caminho para abertura no I2.

Gate isolado da emenda I2A:

- [x] Autoridade exclusiva do CSMR sobre geração do `session_id` preservada documentalmente.
- [x] Origem operacional fechada definida como `csmr_dispatcher_v20_2c`.
- [x] Política D1 para retorno durante `starting` reafirmada; regra conflitante do I3 substituída com rastreabilidade.
- [x] Reserva, cancelamento e consumo implementados sem novo estado principal.
- [x] Origem produtiva validada sem publicação e Harness legado preservado.
- [x] Idempotência, concorrência e persistência da reserva comprovadas.
- [x] Fluxo D1 simulado sem consumidores e sem Timeline produtiva.

Gate V20.2C-I3A — **STATUS: HOMOLOGADO**:

- [x] Commit funcional auditado: `b11309bf20985a9385fb7918e82883dff4c8867e`.
- [x] Integração lógica entre `person.wmoura`, graça, revalidação, dispatcher, reserva I2A, CSMR e I1 comprovada em Harness.
- [x] Reserva e consumo preservaram um único `session_id`; cada evento recebeu `request_id` próprio.
- [x] Ciclo nominal concluiu `idle → active → idle` com quatro ACKs `validated_test` correlacionados e ordenados.
- [x] Retorno durante a graça cancelou sem reserva, sessão ou publicação.
- [x] Retorno durante `starting` aplicou D1 e concluiu abertura/encerramento na mesma sessão, sem consumidores.
- [x] Concorrência foi serializada por `mode: queued`; nenhuma sessão ou UUID concorrente foi criado.
- [x] Reload durante a graça preservou uma única execução e não criou abertura, reserva ou publicação duplicada.
- [x] Persistência, duplicate, retry, cancelamento de reserva e idempotência foram comprovados pelo conjunto I1/I2/I2A/I3A, sem repetição desnecessária de testes.
- [x] Timeout/falha de publicação referenciados à homologação I1; falhas de abertura/encerramento e recuperação referenciadas à homologação I2.
- [x] Parser YAML, `config_check`, reload parcial, traces, ACKs, `git diff --check` e commit foram auditados.
- [x] Timeline e Event Feed permaneceram sem os quatro eventos produtivos; todas as chamadas I1 usaram `test_mode: true`.
- [x] V20.1Q, Recovery 4G, C1.x, UniFi Protect, dashboards e consumidores permaneceram inalterados.

- [x] Cancelamento durante a graça comprovado sem abertura ou evento.
- [x] Abertura lógica comprovada exatamente uma vez em Harness.
- [x] Ordem dos dois eventos de entrada comprovada por ACKs `validated_test`.
- [ ] Consumidores liberados somente depois da abertura publicada.
- [x] Encerramento lógico comprovado exatamente uma vez em Harness.
- [x] Ordem dos dois eventos de retorno comprovada por ACKs `validated_test`.
- [x] Retorno sem sessão aberta comprovado sem publicação.
- [ ] Ciclos consecutivos completos, independentes e sem duplicidade.
- [x] Reload comprovado sem sessão fantasma; startup/restart conservador permanece coberto pela ausência de trigger e persistência homologada, sem restart físico no I3A.
- [x] Harness do contrato I1 comprovado por ACK `validated_test` como fonte não publicável, sem alteração de Timeline, Event Feed ou aliases.

### Gate de regressão

- [x] C1.1 preservado pelo lote I1.
- [x] C1.2 preservado pelo lote I1.
- [x] C1.3 preservado pelo lote I1.
- [x] Harness preservado.
- [x] V20.1O preservada como autoridade; extensão interna restrita e retrocompatível.
- [x] Timeline e Event Feed preservados nos testes I1.
- [x] Nenhum outro componente V20.2 promovido implicitamente pelo I1.

### Gate de falha e rollback

- [x] Falha de publicação permanece observável pelo contrato I1 e interrompe encadeamento.
- [x] Timeout de 10 s, duas repetições com intervalo de 5 s e escalonamento seguro comprovados no I1 e reutilizados sem mecanismo paralelo no I3A.
- [x] ACK `duplicate` comprovado para repetição de `request_id` e de identidade lógica no namespace de teste.
- [x] Ledger técnico comprovado com limite de 16; política produtiva de últimos 16 e mínimo de 7 dias validada estaticamente, sem publicação real.
- [ ] Recuperação parcial retoma somente o evento pendente, sem compensação ou reemissão do par.
- [x] Nenhuma progressão silenciosa após falha crítica de abertura, conforme guards I3A e testes de falha I1/I2.
- [ ] Rollback restrito à promoção funcional da sessão.
- [ ] V20.1O permanece funcional após rollback.
- [ ] Nenhuma execução ou sessão pendente após rollback.
- [ ] Helpers e Harness terminam em estado seguro.

### Gate de homologação

- [x] Parser YAML aprovado.
- [x] Validação estática e configuração Home Assistant aprovadas.
- [x] Traces e ACKs preservados.
- [x] Timeline e Event Feed comprovados sem publicação produtiva no I3A.
- [x] Ordem, deduplicação e ciclos lógicos comprovados pelo conjunto I1/I2/I2A/I3A.
- [x] Working tree e commit funcional auditados.
- [ ] Homologação real posterior executada somente com coordenação do operador.

O Gate I3A está encerrado e não deve acumular novas mudanças. Permanecem abertos somente itens de promoção produtiva, consumidores e rollback operacional pertencentes a Gates posteriores.

O Gate seguinte ao I3A foi o **V20.2C-I3B — Promoção Operacional**, limitado à substituição de `test_mode: true` por `test_mode: false` nas quatro chamadas I1 e à homologação operacional controlada. Sua conclusão está registrada abaixo.

### Gate V20.2C-I3B — Promoção Operacional

**STATUS: HOMOLOGADO**

- [x] Commit funcional `20eb9d15b6a4b2c59b7bf52426c2e6f61c01bf37` alterou exclusivamente quatro valores I1 de `test_mode: true` para `test_mode: false` no dispatcher homologado.
- [x] Parser YAML, `homeassistant.check_config`, `git diff --check` e reload parcial de automações aprovados.
- [x] Um único ciclo operacional controlado foi executado pelo Harness, sem alterar `person.wmoura` ou o helper de graça.
- [x] Sessão única `37c7be2f-4da1-46b4-8a4d-217ce73f4d14` preservada nos quatro eventos.
- [x] `wilson_left_home`: request `9e57d3e1-977e-4a54-8d3c-9197c66da8aa`, ACK `published` em `2026-08-07T07:48:41.974521-03:00`.
- [x] `remote_monitoring_started`: request `3ee45057-30f2-46f8-8b61-5e7c10ced008`, ACK `published` em `2026-08-07T07:48:42.352951-03:00`.
- [x] `wilson_arrived_home`: request `b2b3d3d0-7e9a-4ce9-89f8-be08897ece62`, ACK `published` em `2026-08-07T07:49:19.462676-03:00`.
- [x] `remote_monitoring_ended`: request `8751c680-69a9-4e47-82ab-79c364f8105d`, ACK `published` em `2026-08-07T07:49:19.846991-03:00`.
- [x] Timeline e Event Feed persistiram os quatro textos oficiais na ordem causal; origem e IDs foram preservados no ledger canônico V20.1O.
- [x] CSMR terminou `idle`, sem reserva e com a sessão arquivada em `last_session_id`.
- [x] Reload pós-ciclo preservou quatro registros no ledger e não republicou nenhum evento.
- [x] Permanência contínua não gerou nova sessão ou publicação; traces de saída e retorno terminaram sem erro.
- [x] C1.1, C1.2 e C1.3 mantiveram estado, `last_triggered` e ausência de execução pelo dispatcher.
- [x] Protect cozinha/quarto permaneceram em `detections`; Recovery 4G automático permaneceu `on`.
- [x] V20.1Q, contrato I1, estado I2/I2A, dashboards, presença, helper de graça e consumidores não foram modificados.

O I3B encerra somente a promoção produtiva dos quatro eventos. Integração de C1.x, UniFi Protect e qualquer consumidor futuro continua bloqueada até Gate próprio. Rollback funcional: reverter `20eb9d15b6a4b2c59b7bf52426c2e6f61c01bf37`, validar configuração e recarregar automações; os quatro fatos já publicados permanecem como histórico legítimo do ciclo homologado e não exigem edição da Timeline.

### Gate V20.2C-I4A — Integração dos Consumidores

**STATUS: HOMOLOGADO**

O Gate I4A fechou a integração temporal dos consumidores pelo commit funcional `b02e05d`. C1.1, C1.2 e C1.3 permanecem subordinados ao CSMR; não houve alteração em V20.1Q, Recovery, I1, I2/I2A, Timeline, Event Feed, UniFi Protect, dashboards, sensores físicos ou `person.wmoura`.

- [x] `return_pending` persistente bloqueia consumidores durante retorno e D1.
- [x] Fronteira persistente `consumer_authorized_since` vinculada ao `session_id`.
- [x] Autorização exige `active`, `return_pending=false`, sessão correspondente e `occurred_at > consumer_authorized_since`.
- [x] C1.2 usa `trigger.to_state.last_changed` e permanece reativa à abertura física da porta.
- [x] Harness exige `occurred_at` explícito.
- [x] Fronteira invalidada em `idle`, `starting`, `ending`, `failed` e retorno pendente.
- [x] `homeassistant.check_config` aprovado e reloads parciais retornaram HTTP 200.
- [x] Starting e Ending rejeitados; Failed não acionou consumidor.
- [x] D1 não acionou C1.1, C1.2 ou C1.3.
- [x] Active nominal produziu uma execução de C1.2 e uma de C1.3, sem duplicação indevida.
- [x] Reload com `return_pending=true` preservou a proteção.
- [x] Nenhum restart foi executado e nenhum componente protegido foi alterado.

Estado final: `CSMR=idle`, `return_pending=off`, `consumer_authorized_since=1970-01-01 00:00:00`.

Continuam pendentes a promoção operacional real dos consumidores, a validação por ciclo real de saída/retorno, UniFi Protect e demais consumidores futuros. O próximo Gate autorizado é **V20.2C-I4B.1**.

### Despacho V20.2C-A2 — Governança de homologação

O projeto distingue formalmente **Homologação Técnica** de **Evidência Operacional**. Homologação Técnica pode usar Harness quando ele reproduz integralmente contratos funcionais, transacionais, estados, concorrência, idempotência, rollback, recovery, reload, consumidores e efeitos esperados. Evidência Operacional é apenas a observação natural posterior em produção; sua ausência não bloqueia o Roadmap. Hardware ou integração sem Harness equivalente permanece sujeito a Gate físico.

### Gate V20.2C-I4B.1 — Promoção Operacional Controlada

**STATUS: HOMOLOGADO**

Esta etapa valida exclusivamente pelo Harness homologado o runtime produtivo, dispatcher, Timeline, CSMR, consumidores C1.1/C1.2/C1.3, publicação, autorização temporal, push, D1, reload, rollback, ausência de duplicidade e proteção dos componentes. Não altera código funcional.

- [x] Graça, abertura, publicação canônica e sessão `active` observadas pelo Harness Dispatcher.
- [x] C1.1 e C1.3 executaram uma vez; C1.2 executou uma vez após a fronteira temporal.
- [x] Retorno elevou o bloqueio, encerrou a sessão e invalidou a fronteira.
- [x] Porta pós-retorno não acionou consumidor.
- [x] Reloads HTTP 200 não reabriram sessão nem produziram evento retroativo.
- [x] Resíduo de `cycle_id` foi reconciliado por checkpoint `idle`, sem publicação ou alteração funcional.
- [x] Estado final seguro e componentes protegidos inalterados.

### Gate V20.2C-I4B.2 — Evidência Operacional

**STATUS: PENDENTE DE EVIDÊNCIA OPERACIONAL**

Esta etapa registrará o primeiro ciclo físico natural `home → not_home → home`, coletando traces, Timeline, Event Feed, ACKs, `session_id`, `request_id`, estados, consumidores, push e encerramento. Nenhuma correção será feita durante a coleta; comportamento inesperado exigirá Gate corretivo. I4B.2 não bloqueia I5, I6, consumidores futuros ou UniFi Protect após I4B.1 homologado.

### Gate V20.2C-I5A — Integração controlada do UniFi Protect

**STATUS: HOMOLOGADO (Harness)**

O I5A homologou o modelo mínimo de intenção automática e manual para os modos de gravação das câmeras G4 Instant. `csmr_recording_requested` depende do contexto operacional autorizado do CSMR; `manual_override` é uma solicitação explícita independente. A intenção efetiva é `csmr_recording_requested OR manual_override`, mapeada exclusivamente para `always`; com ambas desligadas, o baseline é `detections`.

Foram validados os dois selects Protect, retorno pendente, ending, failed, manualização em idle, retorno com override manual preservado, reload, falha parcial documentada e comandos idempotentes. Não houve novos `event_code`, publicação na Timeline/Event Feed, alteração de CSMR, dispatcher, I1/I2, C1.x, Recovery, V20.1Q, dashboards ou sensores físicos.

### Gate V20.2C-I5B — Promoção operacional controlada

**STATUS: HOMOLOGADO** em 2026-08-07. O Harness executou CSMR ativo, retorno, override manual, `return_pending`, `failed`, reload, idempotência e divergência controlada de uma câmera. O baseline final ficou em `detections`; nenhum componente protegido foi alterado.

## V20.2D — Consolidação da baseline V20.2C

**STATUS: V20.2C FUNCIONALMENTE CONCLUÍDA**

## Gate V20.2E — Integração do Uso do Carro à Timeline

**Status: APROVADA ESTATICAMENTE PARA COMMIT; HOMOLOGAÇÃO RUNTIME PENDENTE**

### Escopo e arquitetura

- [x] Lote formal V20.2E autorizado como ampliação estritamente aditiva da V20.1O.
- [x] Produtor limitado a `source: carro_presenca` e códigos `car_use_started`/`car_use_ended`.
- [x] Quatro eventos, source e semântica do CSMR preservados integralmente.
- [x] Escrita direta em Timeline, Event Feed e aliases finais proibida.
- [x] Detecção existente do carro e destinatário `notify.mobile_app_iphonewm` preservados.

### Contrato e comportamento

- [x] Publicador canônico ampliado sem ledger, retry ou idempotência paralelos.
- [x] Um `session_id` persistente correlaciona início e término; cada publicação usa `request_id` próprio.
- [x] ACK `published`, `duplicate` ou `validated_test` permite conclusão segura; `rejected`, `failed` e timeout preservam o checkpoint.
- [x] Término sem sessão válida não publica evento órfão nem bloqueia o comportamento funcional legado.
- [x] Push de início e término parametrizados independentemente; Timeline não depende desses parâmetros.
- [x] Reinício com ciclo ativo preserva a correlação por helpers restauráveis.

### Validação e homologação

- [x] Persistência e reconciliação controlada do `request_id` de `car_use_ended` implementadas e verificadas estaticamente contra o contrato canônico; homologação runtime permanece pendente.
- [x] Ausência de `initial` nos dois controles de push verificada estaticamente, permitindo primeira criação nativa em `off` e restauração posterior da escolha do usuário; ativação inicial controlada permanece pendente de implantação.
- [ ] Tratamento de checkpoint de término vazio/válido/inválido e bloqueio de estados parciais verificados estaticamente; confirmação runtime permanece pendente.
- [ ] Persistência de `rejected` para início/término, bloqueio da reconciliação e liberação governada verificados estaticamente; confirmação runtime permanece pendente.
- [ ] Recuperação administrativa de metadados `rejected` parciais, validação contra o ciclo e trava contra escritor ativo verificadas estaticamente; confirmação runtime permanece pendente.
- [ ] Guard fail-closed de concorrência, sem default zero e com validação explícita de existência, disponibilidade e `current`, verificado estaticamente; teste de concorrência runtime permanece pendente.
- [x] Compatibilidade do guard fail-closed com `has_value`/`state_attr` aprovada na revisão estática final independente: `none` preservado, validação numérica estrita, booleanos e negativos recusados, ausência de fallback zero, stops separados antes da limpeza e lógica administrativa preservada. Classificação: **A. APROVADA ESTATICAMENTE PARA ATUALIZAÇÃO DO GATE E COMMIT.**
- [ ] Validação no ambiente real do atributo `current`, testes funcionais e de concorrência, implantação e homologação operacional permanecem pendentes.
- [x] Parser YAML disponível e parser JSON Storage aprovados; `check_config` nativo permanece pendente por indisponibilidade do Home Assistant neste ambiente.
- [x] `git diff --check`, referências, IDs e ausência de escrita direta aprovados.
- [x] Cenários de pushes ligados/desligados preparados.
- [x] Duplicidade, falha/timeout, término sem sessão e reinício com ciclo ativo preparados.
- [x] Nenhum reload, restart, push real ou publicação produtiva executado sem autorização específica.

A sequência operacional e os bloqueios detalhados permanecem consolidados na seção V20.2E de `docs/pendencias_atuais_central_operacional.md`.

I4B.2 permanece exclusivamente como evidência operacional futura e não bloqueante. O catálogo de contratos, mapa arquitetural, auditoria estática e próximas evoluções elegíveis estão consolidados em `docs/v20_2c/baseline_v20_2d.md`.

## Gates especificos - V20.1Q Recovery 4G

### Gate documental

- Auditoria, despacho arquitetural e Implementation Plan presentes e referenciados.
- Classificacao e subordinacao documental declaradas.
- Lacunas de cooldown e timeout numericos registradas sem inferencia.

### Gate pre-implementacao

- Nova Etapa A executada sobre `develop` sincronizada.
- Helpers equivalentes, consumidores, tomada, blueprint e automacao confirmados.
- Lista exata de arquivos apresentada antes de alteracao funcional.
- Persistencia, idempotencia, restart seguro, cancelamento e rollback definidos.
- Fronteira V20.1Q.1/V20.1Q.2 preservada.

### Gate pre-teste fisico

- YAML e configuracao Home Assistant validados.
- Tomada correta e caminho de religamento confirmados.
- Ausencia de detector proprio, ping ou interpretacao de bytes no Executor confirmada.
- Rollback preparado.
- Autorizacao operacional explicita do usuario registrada antes de qualquer power cycle.

### Gate de homologacao

- Cenarios de recovery desabilitado, tentativas 1 e 2, cooldown, timeout, concorrencia, restart e erro executados.
- Nenhuma tentativa além do snapshot configurado observada.
- Central confirmada como unica decisora e validadora.
- Timeline, Push, aliases finais, `sensor.status_casa` e V20.1O preservados.

### Gate de encerramento

- Evidencias e pendencias registradas.
- Changelog/checkpoint e Roadmap atualizados.
- Legado preservado ou tratado somente por fase propria.
- Nenhuma edicao manual de `.storage`.
