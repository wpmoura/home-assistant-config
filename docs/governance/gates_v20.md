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

Status documental: I1/I2 HOMOLOGADOS; EMENDA I2A AUTORIZADA E EM IMPLEMENTAÇÃO; INTEGRAÇÃO REAL PENDENTE

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
- [ ] Reserva, cancelamento e consumo implementados sem novo estado principal.
- [ ] Origem produtiva validada sem publicação e Harness legado preservado.
- [ ] Idempotência, concorrência e persistência da reserva comprovadas.
- [ ] Fluxo D1 simulado sem consumidores e sem Timeline produtiva.

- [ ] Cancelamento antes da graça comprovado sem abertura ou evento.
- [ ] Abertura comprovada exatamente uma vez.
- [ ] Ordem dos dois eventos de entrada comprovada.
- [ ] Consumidores liberados somente depois da abertura publicada.
- [ ] Encerramento comprovado exatamente uma vez.
- [ ] Ordem dos dois eventos de retorno comprovada.
- [ ] Retorno sem sessão aberta comprovado sem publicação.
- [ ] Ciclos consecutivos completos, independentes e sem duplicidade.
- [ ] Restart e reload comprovados sem sessão fantasma.
- [x] Harness do contrato I1 comprovado por ACK `validated_test` como fonte não publicável, sem alteração de Timeline, Event Feed ou aliases.

### Gate de regressão

- [x] C1.1 preservado pelo lote I1.
- [x] C1.2 preservado pelo lote I1.
- [x] C1.3 preservado pelo lote I1.
- [ ] Harness preservado.
- [x] V20.1O preservada como autoridade; extensão interna restrita e retrocompatível.
- [x] Timeline e Event Feed preservados nos testes I1.
- [x] Nenhum outro componente V20.2 promovido implicitamente pelo I1.

### Gate de falha e rollback

- [ ] Falha de publicação permanece observável.
- [ ] Timeout de 10 s, duas repetições com intervalo de 5 s e escalonamento seguro comprovados.
- [x] ACK `duplicate` comprovado para repetição de `request_id` e de identidade lógica no namespace de teste.
- [x] Ledger técnico comprovado com limite de 16; política produtiva de últimos 16 e mínimo de 7 dias validada estaticamente, sem publicação real.
- [ ] Recuperação parcial retoma somente o evento pendente, sem compensação ou reemissão do par.
- [ ] Nenhuma progressão silenciosa após falha crítica de abertura.
- [ ] Rollback restrito à promoção funcional da sessão.
- [ ] V20.1O permanece funcional após rollback.
- [ ] Nenhuma execução ou sessão pendente após rollback.
- [ ] Helpers e Harness terminam em estado seguro.

### Gate de homologação

- [ ] Parser YAML aprovado.
- [ ] Validação estática e configuração Home Assistant aprovadas.
- [ ] Traces e Logbook preservados.
- [ ] Timeline e Event Feed comprovados.
- [ ] Ordem, deduplicação e ciclos sucessivos comprovados.
- [ ] Working tree e commit auditados.
- [ ] Homologação real posterior executada somente com coordenação do operador.

Enquanto qualquer item técnico ou de homologação permanecer aberto, a promoção possui efeito arquitetural/documental, mas a implementação e a publicação em runtime continuam bloqueadas.

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
