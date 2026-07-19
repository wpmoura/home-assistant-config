# Implementation Plan V20.1Q — Recovery Automático do Modem 4G

Data: 2026-07-13
Status: HOMOLOGAÇÃO RUNTIME PARCIAL — SUSPENSA POR DECISÃO OPERACIONAL. Sem bloqueio técnico conhecido; pronta para retomada futura.
Classificação: release transitório operacional
Validade: execução e homologação da V20.1Q
Autoridade: subordinada à Constituição, Source of Truth, Arquitetura, Roadmap e Gates

## Adendo corretivo aprovado — 2026-07-17

Este adendo substitui as regras numeradas deste plano onde houver conflito. O Recovery usa laço genérico e snapshot de `casa_recovery_4g_max_tentativas`, sem limite interno de duas ou dez tentativas. Todas as tentativas consomem `casa_recovery_4g_tempo_off_segundos`; os dois helpers numerados permanecem deprecados e sem consumidores. A queda é confirmada por `casa_recovery_4g_confirmacao_queda_minutos`. O sucesso somente ocorre após estabilidade contínua e revalidação final em `on`. Cooldown e gravação de `ultima_execucao` ocorrem somente após esgotamento completo e seguro. Restart, energia e cancelamento do operador não consomem cooldown.

Matriz obrigatória, ainda não executada: máximos 1, 2, 5 e 10; alteração do máximo durante o ciclo; Tempo OFF único e alteração entre tentativas; retorno estável, oscilação, `unknown`, `unavailable`, janela zero e timeout; falha intermediária, erro técnico seguro, esgotamento, bloqueio e expiração do cooldown; restart nas fases OFF, validação e estabilização; cancelamento por energia e operador; inicialização automática sem power cycle antes da confirmação da queda; Timeline com 16 eventos e índices superiores a 2.

### Procedimento de homologação pendente

Pré-condições para todos os casos: autorização operacional explícita, tomada confirmada, rollback preparado, trace habilitado e registro dos valores de todos os helpers antes do ciclo. Para cada cenário, guardar request ID, sequência de eventos, índices executados, estados da tomada, veredito, estado final do Executor, `ciclo_em_andamento`, `tentativa_atual` e timestamp de `ultima_execucao`.

1. Quantidade: executar incidentes controlados com máximo 1, 2, 5 e 10 sem retorno; contar exatamente os índices e confirmar cooldown somente ao final. Em outro ciclo, alterar o helper após o primeiro índice e confirmar que o snapshot original permanece válido. Repetir com retorno estabilizado em índice intermediário e confirmar ausência de índices posteriores.
2. Tempo OFF: definir um valor observável antes do ciclo e medir cada intervalo OFF. Alterar o helper entre duas tentativas e confirmar que o Executor o lê no início de cada tentativa; os helpers numerados não podem produzir efeito.
3. Estabilização: testar retorno que permanece `on`, queda para `off` antes do prazo, múltiplas oscilações, transições para `unknown` e `unavailable`, janela zero e ausência total de retorno até timeout. Somente a janela concluída e revalidada em `on` pode publicar `recuperacao_validada`.
4. Cooldown: confirmar ausência após falha intermediária, retorno instável, erro técnico seguro e sucesso. No esgotamento, confirmar uma única gravação de `ultima_execucao`, estado `cooldown`, bloqueio de nova solicitação e retorno para `ocioso` após expiração.
5. Cancelamentos: reiniciar durante OFF, espera de retorno e estabilização; repetir com falta de energia e desligamento pelo operador. Confirmar tomada ligada quando controlável, ciclo desligado, tentativa zero, nenhum timestamp novo e nenhum cooldown.
6. Inicialização e Timeline: após restart autorizado, confirmar Recovery automático `on`, ausência de power cycle antes da confirmação completa da queda, reconciliação de estado residual e limite padrão de 16 eventos. Gerar tentativa com índice superior a 2 e confirmar renderização dinâmica e apenas um evento final.

Nenhum item deste procedimento foi executado durante a implementação estática. Parte deste procedimento foi posteriormente executada em runtime real — ver "Ata de Homologação Runtime — V20.1Q" abaixo.

### Ata de Homologação Runtime — V20.1Q (2026-07-18)

Status: **Homologação Suspensa por decisão operacional.** Não existe bloqueio técnico conhecido. A implementação permanece válida. Pronta para retomada futura exatamente deste ponto.

Três power cycles reais e controlados foram autorizados e executados pelo usuário (Testes 1, 2 e 3), conforme Gate pré-teste físico. Todos os ajustes de parâmetro usados nos testes foram feitos via valores de helper (operação normal), sem alteração de YAML, scripts, automações, packages ou arquitetura.

#### Teste 1 — Esgotamento mínimo (max=1) + Cooldown

- Objetivo: esgotamento completo com o menor valor de tentativas; entrada e expiração do cooldown; tentativa de janela zero.
- Resultado: **Aprovado** para esgotamento/cooldown. Janela zero não exercitada (estabilização ficou em 1, não 0; o ciclo foi direto a timeout sem passar pela lógica de estabilização).
- Evidências: ciclo completo 21:24–21:27 (2026-07-18); tempo OFF exato (10s) e timeout de validação exato (30s) batendo com os parâmetros configurados; esgotamento declarado exatamente no snapshot (1=1); `ultima_execucao` gravado somente no esgotamento; cooldown expirado corretamente; Timeline com mensagens corretas no alias produtivo `sensor.casa_event_feed`; estado do Executor percorrendo integralmente o contrato documentado (`ocioso→solicitado→executando→erro/timeout→cooldown→ocioso`).
- Critérios do Gate atendidos: Cenário "1"; esgotamento completo; cooldown (entrada e expiração).

#### Teste 2 — Cenário "10" + Integridade do snapshot

- Objetivo: cenário de máximo alto (10); alteração do helper em pleno ciclo; busca de retorno em índice intermediário.
- Resultado: **Aprovado**, com escopo maior que o planejado. A queda real durou mais que o previsto e não se resolveu sozinha — esgotamento completo nas 10 tentativas em vez do retorno intermediário esperado.
- Evidências: `max_tentativas` alterado de 10→3 em pleno ciclo (21:45:34–21:45:36); trace do orquestrador confirma `max_tentativas_ciclo` fixo em 10 do início ao fim — o laço completou as 10 iterações reais, ignorando a mudança; trace explícito da transição de expiração do cooldown (ação real, não apenas inferida por logbook); Timeline com as 10 tentativas registradas.
- Critérios do Gate atendidos: Cenário "10"; **integridade do snapshot** (prova definitiva); reforço de esgotamento e cooldown com valor diferente do Teste 1.

#### Teste 3 — Cenário "5" + tentativa de cancelamento/retorno intermediário/janela zero

- Objetivo: cenário "5"; estratégia adaptativa priorizando retorno intermediário, cancelamento pelo operador e janela zero, nesta ordem.
- Resultado: **Aprovado parcialmente**. Cenário "5" obtido, além de achado não planejado de maior valor (erro técnico seguro). Os três objetivos prioritários não foram atingidos por limitação de monitoramento (polling de ~6s não foi suficiente para reagir dentro da janela de validação de ~30s por tentativa).
- Evidências: tentativas 1 e 2 tiveram atraso real de confirmação da tomada (Zigbee, >5s), disparando o ramo de erro técnico do orquestrador, que religou a tomada por segurança, confirmou o estado e prosseguiu ao próximo índice sem corromper o ciclo; tentativas 3–5 limpas; esgotamento em 5=5; queda real resolveu-se sozinha 45s **depois** do esgotamento.
- Critérios do Gate atendidos: Cenário "5"; **erro técnico seguro / falha intermediária sem decisão autônoma do Executor** (achado não planejado).

#### Itens já homologados (evidência operacional suficiente)

Ciclo completo de sucesso (detecção → confirmação de queda → tentativa → religamento → validação → estabilização → encerramento sem cooldown → Timeline/`status_casa`); restart durante ciclo ativo; segurança da tomada / religamento independente; limite de 16 eventos na Timeline; cenários de máximo 1, 5 e 10; esgotamento completo; cooldown (entrada e expiração); integridade do snapshot; erro técnico seguro; ausência de erros técnicos no error log da API.

#### Itens pendentes para retomada futura

- Cancelamento pelo operador em ciclo ativo — não exercitado com sucesso (limitação de monitoramento).
- Retorno estabilizado em índice intermediário — as quedas reais tentadas duraram mais que a janela de tentativas disponível.
- Janela de estabilização igual a zero — configurada mas nunca exercida de fato.

#### Classificações de não bloqueio

- Cenário "2" isoladamente: considerado suficientemente coberto pela generalização de código (laço genérico único, sem hardcode por valor, confirmado por leitura de código e por execução real em três valores distintos — 1, 5 e 10). Não é bloqueador de encerramento.
- Interrupção por falta de energia em ciclo ativo: classificada como **risco residual aceito**. Mesma condição de código do cancelamento pelo operador, já comprovada por analogia estrutural (leitura de código), mas sem execução real com ciclo ativo. Não deve ser forçada deliberadamente.
- Oscilação/`unknown`/`unavailable` dos sensores: sem ocorrência real observada em nenhum teste; permanece sem evidência, sem classificação de bloqueio no momento.

#### Próxima etapa recomendada

Um único teste combinado e adaptativo: `estabilizacao_retorno_minutos=0`, `max_tentativas` com folga (ex.: 3), monitorado por **assinatura de eventos** (`subscribe_events`/`state_changed` via WebSocket ou mecanismo equivalente) em vez de polling, para reação em tempo real (~1s) em vez de janelas de ~6s. Se a conectividade voltar durante uma tentativa, deixar validar (fecha retorno intermediário + janela zero simultaneamente); se não voltar a tempo, desligar `automatico` durante a janela de validação (fecha cancelamento pelo operador). Estimativa: 1 execução adicional deve bastar.

### Limitação do dashboard Parâmetros

O dashboard ativo foi localizado em `.storage/lovelace.dashboard_lixo`, sem arquivo YAML versionado equivalente. Em respeito à proibição de edição manual de `.storage`, a implementação estática de 2026-07-17 não alterou a interface. Após autorização operacional, a UI deverá adicionar Tempo OFF genérico e Confirmação da Queda, manter Automático e Estabilização e retirar da visualização os dois Tempos OFF numerados legados.

### Atualização do dashboard Parâmetros — 2026-07-17 (item concluído)

Diagnóstico técnico, revisão de UX e resolução dos bloqueios técnicos foram registrados como aprovados pelo usuário nesta sessão de implementação, restritos ao escopo deste item (seção "Recovery 4G" do dashboard `Parâmetros`).

Mecanismo utilizado: API/WebSocket oficial do Home Assistant (`/api/websocket`, comandos `lovelace/config` e `lovelace/config/save`), autenticada com o `HA_TOKEN` da sessão. Nenhum arquivo `.storage` foi editado manualmente; a alteração foi persistida exclusivamente pelo mecanismo oficial que o próprio frontend usa para salvar dashboards Storage. Não houve fallback para edição manual pela interface.

Verificação prévia executada:

- Leitura via `lovelace/dashboards/list` e `lovelace/config` confirmou que a API suporta acesso ao dashboard Storage `dashboard-lixo` (título "Parâmetros").
- Estados reais dos helpers `casa_recovery_4g_*` foram consultados via `/api/states` antes da alteração para confirmar que todas as entidades referenciadas existem e estão disponíveis.
- Backup do JSON completo do dashboard foi salvo antes e depois da escrita para permitir rollback simples.

Alteração aplicada, restrita ao card "Execução automática" da seção "Recovery 4G":

- Removidos: `input_number.casa_recovery_4g_tempo_off_tentativa_1` ("Tempo OFF — tentativa 1") e `input_number.casa_recovery_4g_tempo_off_tentativa_2` ("Tempo OFF — tentativa 2"), helpers legados deprecados e sem consumidores.
- Adicionados: `input_number.casa_recovery_4g_tempo_off_segundos` ("Tempo OFF único"), `input_number.casa_recovery_4g_confirmacao_queda_minutos` ("Confirmação da queda") e `input_number.casa_recovery_4g_estabilizacao_retorno_minutos` ("Estabilização do retorno").
- Mantidos sem alteração: "Recovery automático", "Máximo de tentativas", "Cooldown", "Timeout de validação" e o card "Publicação" (Timeline, Push).

Nenhuma outra seção, view, package, automação, script, entidade ou alias final foi alterada. A alteração foi lida de volta via `lovelace/config` e confirmada idêntica ao esperado. Esta atualização não homologa o Recovery 4G em runtime; o Gate corretivo V20.1Q em `docs/governance/gates_v20.md` permanece aberto para os cenários de teste físico e homologação ainda não executados.

### Reorganização UX conforme ESPEC-UX-PARAM-RECOVERY4G — 2026-07-17

A entrega anterior deste item ficou parcial: adicionou/removeu os campos corretos, mas manteve a estrutura antiga de dois cards (`Execução automática`, `Publicação`) e não aplicou a reorganização de UX já aprovada. A especificação normativa correspondente — `docs/ux/espec_ux_param_recovery4g.md` (ESPEC-UX-PARAM-RECOVERY4G, V20.1Q) — define de forma exaustiva a estrutura correta e passa a ser a referência única desta seção.

Reorganização aplicada, via a mesma API/WebSocket oficial (`lovelace/config` / `lovelace/config/save`), sem edição de `.storage`, sem alteração de helpers, automações, scripts ou arquitetura:

- A seção "Recovery 4G" passou a conter exatamente três cards de entidades, nesta ordem: **Operação**, **Ciclo de Recuperação**, **Avisos**.
- **Operação**: Recuperação Automática, Máximo de Tentativas, Pausa após Esgotar Tentativas (mapeada para `casa_recovery_4g_cooldown_minutos`).
- **Ciclo de Recuperação** (numerado 1–5): Tempo para Confirmar a Queda, Tempo com a Tomada Desligada, Limite de Espera da Tomada (`casa_recovery_4g_timeout_confirmacao_tomada_segundos`, não exposto na entrega anterior), Limite de Espera do 4G (`casa_recovery_4g_timeout_validacao_segundos`), Tempo de Estabilização.
- **Avisos**: Registrar na Timeline, Notificar no Celular.
- O card de texto explicativo (markdown) da entrega anterior foi removido, pois a especificação define conteúdo exaustivo de exatamente três cards por seção.
- O heading "Recovery 4G" foi preservado em inglês, conforme exceção normativa da Seção 3.2 da especificação.

Validação executada contra os 15 critérios de aceitação (Seção 9 da especificação): A1–A7 e A9–A15 verificados por inspeção estrutural do JSON lido de volta da API. A8 (nenhum rótulo quebra em duas linhas em viewport de celular) não pôde ser verificado por inspeção visual real nesta sessão; os rótulos usados são literalmente os oficiais da especificação.

Diagnóstico técnico, revisão de UX e resolução dos bloqueios técnicos para este item foram registrados como aprovados pelo usuário nesta sessão de implementação.

## 1. Objetivo

Restaurar o recovery automático do modem 4G sem criar detector paralelo e sem alterar a lógica semântica da Central.

## 2. Invariantes

- A Central decide, valida e encerra.
- O Executor atua somente mediante ordem válida.
- O Executor não interpreta ping, bytes, latência ou estado do link.
- Nenhuma ação física ocorre sem gates e autorização operacional aplicável.
- `sensor.status_casa`, aliases finais, V20.1O e V20.2 shadow permanecem protegidos.

## 3. Fases

### V20.1Q.1 — Recovery Executor

Escopo:

- política parametrizável;
- helpers;
- executor físico;
- controle de tentativas;
- cooldown;
- timeout de validação;
- proteção contra concorrência;
- proteção contra tomada desligada;
- contratos técnicos para comunicação com a Central;
- painel administrativo de parâmetros;
- preparação dos controles Timeline e Push.

Não inclui publicação definitiva na Timeline, salvo se já houver publicador canônico reutilizável sem reabrir V20.1O e com aprovação no Gate pré-implementação.

### V20.1Q.2 — Integração com Timeline

Escopo posterior:

- publicação canônica dos fatos do recovery;
- respeito aos helpers Timeline e Push;
- integração sem duplicidade;
- ausência de feed paralelo;
- preservação da política V20.1O.

## 4. Helpers esperados

Antes de criar qualquer helper, pesquisar equivalentes e seus consumidores.

Caso não existam equivalentes, usar preferencialmente:

- `input_boolean.casa_recovery_4g_automatico`;
- `input_number.casa_recovery_4g_max_tentativas`;
- `input_number.casa_recovery_4g_tempo_off_segundos`;
- `input_number.casa_recovery_4g_confirmacao_queda_minutos`;
- `input_number.casa_recovery_4g_estabilizacao_retorno_minutos`;
- `input_number.casa_recovery_4g_cooldown_minutos`;
- `input_number.casa_recovery_4g_timeout_validacao_segundos`;
- `input_boolean.casa_recovery_4g_timeline`;
- `input_boolean.casa_recovery_4g_push`.

A nomenclatura deve ser confirmada pela inspeção antes da implementação. Não duplicar helpers semanticamente equivalentes.

Os helpers numerados antigos de Tempo OFF permanecem somente como legado deprecado e sem consumidores até limpeza futura autorizada.

Helpers adicionais para estado, correlação, cancelamento, idempotência ou restart somente podem ser propostos após inspeção e devem ser apresentados antes da alteração funcional.

## 5. Defaults aprovados

- Recovery automático: ligado.
- Máximo de tentativas: definido exclusivamente pelo helper e capturado em snapshot no início do ciclo.
- Tempo OFF: único e parametrizado para qualquer índice.
- Timeline: configurável.
- Push: configurável.

## 6. Cooldown

O cooldown deve ser parametrizável.

O valor numérico anterior não está registrado documentalmente no repositório. Portanto:

- o default numérico é decisão pendente antes da implementação funcional;
- nenhum valor deve ser inventado;
- a Etapa A deve identificar eventual valor no blueprint ou automação existente e apresentá-lo para aprovação;
- o valor legado não está automaticamente aprovado;
- o cooldown impede novo ciclo, mas não interrompe conclusão ou validação de ciclo já iniciado.

## 7. Timeout de validação

O timeout deve ser parametrizável. O Executor não interpreta o resultado.

Ao final do período, o Executor:

- informa timeout técnico;
- permanece subordinado à decisão da Central;
- não declara falha ou sucesso sem comando da Central.

O default numérico é decisão pendente e deve ser validado na Etapa A, sem inferência.

## 8. Estados técnicos

Contrato mínimo do Executor:

- `ocioso`;
- `bloqueado`;
- `solicitado`;
- `executando_tentativa_1`;
- `executando_tentativa_2`;
- `aguardando_validacao`;
- `concluido_tecnicamente`;
- `timeout_validacao`;
- `esgotado`;
- `cooldown`;
- `erro`.

Os estados não substituem os estados semânticos da Central. `concluido_tecnicamente` significa somente término do power cycle.

Caso a inspeção encontre contrato equivalente, a Etapa A deve propor reutilização e registrar a correspondência.

## 9. Persistência, idempotência e restart

Antes da implementação, a Etapa A deve definir explicitamente:

- como impedir ciclo duplicado após restart;
- como correlacionar solicitação, incidente e tentativa;
- como recuperar estado de tentativa;
- como garantir religamento da tomada;
- como tratar cooldown após restart;
- quais helpers são persistentes;
- qual informação é transitória;
- como cancelar com segurança antes, durante e depois do período OFF.

Nenhuma regra de restart deve ser implementada antes de ser apresentada no Gate pré-implementação.

## 10. Entidade da tomada e legado

A entidade exata deve ser confirmada pela auditoria técnica no repositório. A evidência consolidada aponta `switch.0xa4c1381045aeb344`, mas a Etapa A deve validar:

- entity_id atual;
- blueprint e automação atuais;
- consumidores;
- disponibilidade runtime;
- comportamento atual;
- risco de alteração.

O blueprint e a automação legados devem permanecer preservados até homologação. Nenhum decommission pertence à V20.1Q.1.

## 11. Arquivos e componentes

Não fixar previamente um novo package sem inspeção.

A Etapa A deve:

- localizar o blueprint e a automação existentes;
- avaliar reutilização;
- avaliar se o package de parâmetros existente comporta os helpers;
- localizar o painel administrativo existente;
- mapear o publicador canônico de Timeline e Push;
- apresentar a lista exata de arquivos antes de alterar.

Novo package funcional continua proibido até justificativa técnica específica e aprovação constitucional aplicável.

Recursos nativos preferenciais, conforme adequação confirmada:

- scripts;
- automações;
- helpers;
- eventos;
- templates disparados por evento;
- `mode: single` para exclusão mútua;
- `wait_for_trigger` ou contrato equivalente para aguardar a Central;
- Logbook para diagnóstico técnico.

Não criar infraestrutura paralela apenas para simular barramento de mensagens.

## 12. Painel de parâmetros

Adicionar futuramente uma seção `Recovery 4G` no painel de Parâmetros existente, contendo:

- Recovery automático;
- Máximo de tentativas;
- Tempo OFF único;
- Confirmação da queda;
- Estabilização do retorno;
- Cooldown;
- Timeout de validação;
- Timeline;
- Push.

O layout deve:

- seguir o padrão visual atual;
- funcionar bem em celular;
- não criar dashboard separado;
- não confundir Failover 4G com Recovery 4G;
- deixar claro que Timeline e Push são políticas de publicação;
- deixar claro que Recovery automático controla ação física.

## 13. Fluxo técnico esperado

1. A Central emite solicitação válida e correlacionável.
2. O Executor verifica habilitação, concorrência, cooldown e tentativa autorizada.
3. O Executor desliga a tomada.
4. O Executor aguarda o tempo OFF configurado para a tentativa.
5. O Executor religa a tomada, inclusive em caminho de erro quando aplicável.
6. O Executor registra conclusão técnica e aguarda validação.
7. A Central valida e decide encerrar, executar o próximo índice ou declarar esgotamento.
8. O Executor nunca executa além do snapshot capturado pela Central.

## 14. Testes a preparar

Preparar homologação para:

1. Recovery desabilitado.
2. Máximos 1, 2, 5 e 10 com Tempo OFF único.
3. Sucesso estabilizado em tentativa inicial e intermediária.
4. Falha intermediária sem decisão autônoma do Executor.
5. Próximo índice autorizado genericamente pela Central.
6. Oscilação, `unknown` e `unavailable` durante a estabilização.
7. Esgotamento exatamente no limite do snapshot.
8. Cooldown impedindo novo ciclo.
9. Alteração de parâmetros pelo painel.
10. Timeline desligada sem afetar recovery.
11. Push desligado sem afetar recovery.
12. Restart durante solicitação, período OFF ou validação.
13. Erro durante tomada desligada.
14. Concorrência e comando duplicado.
15. Rollback.

Nenhum teste físico será executado sem autorização operacional explícita.

## 15. Rollback

O plano exige:

- backup dos arquivos alterados;
- commits pequenos;
- restauração simples;
- preservação da automação e blueprint anteriores até homologação;
- nenhuma remoção de legado na V20.1Q.1;
- possibilidade de desligar recovery automático pelo helper;
- confirmação de tomada ligada antes de encerrar rollback;
- nenhuma edição manual de `.storage`.

## 16. Gates da implementação

### Gate documental

- Auditoria, despacho e plano presentes e referenciados.
- Cooldown e timeout pendentes declarados.

### Gate pré-implementação

- Helpers equivalentes pesquisados.
- Arquivos e entidades confirmados.
- Estratégia de persistência, idempotência, restart e rollback apresentada.
- Nenhuma alteração funcional antes da aprovação da Etapa A.

### Gate pré-teste físico

- YAML e configuração validados.
- Tomada confirmada.
- Rollback preparado.
- Autorização operacional explícita registrada.

### Gate de homologação

- Cenários de testes executados e evidenciados.
- Ausência de concorrência, execução além do snapshot e detector paralelo confirmada.

### Gate de encerramento

- Documentação e pendências atualizadas.
- Legado preservado ou tratado apenas em fase própria.

## 17. Critérios de aceite

A V20.1Q.1 somente estará apta à homologação se:

- a Central for a única decisora;
- o Executor não possuir detector;
- não houver leitura de bytes ou ping no Executor;
- a quantidade de tentativas vier exclusivamente do snapshot do helper;
- tempos, cooldown e timeout forem parametrizáveis;
- recovery puder ser desligado;
- Timeline e Push forem independentes da execução;
- não houver concorrência ou tentativa além do snapshot;
- a tomada estiver protegida contra permanência desligada;
- `sensor.status_casa` e aliases finais estiverem inalterados;
- V20.1O não tiver sido reaberta;
- rollback estiver documentado;
- nenhuma ação física tiver sido realizada sem autorização.

## 18. Pendências antes da implementação funcional

- Aprovar default numérico do cooldown.
- Aprovar default numérico do timeout de validação.
- Confirmar helpers equivalentes e nomenclatura.
- Confirmar entidade e disponibilidade da tomada.
- Confirmar contrato de energia canônico usado pela Central.
- Definir persistência e reconciliação após restart.
- Confirmar arquivos exatos e necessidade ou não de package novo.
- Confirmar fronteira de reutilização do publicador canônico em V20.1Q.1 versus V20.1Q.2.
