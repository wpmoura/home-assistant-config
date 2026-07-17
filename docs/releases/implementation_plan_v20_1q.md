# Implementation Plan V20.1Q — Recovery Automático do Modem 4G

Data: 2026-07-13
Status: PACOTE CORRETIVO IMPLEMENTADO ESTATICAMENTE; HOMOLOGAÇÃO OPERACIONAL PENDENTE
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

Nenhum item deste procedimento foi executado durante a implementação estática.

### Limitação do dashboard Parâmetros

O dashboard ativo foi localizado em `.storage/lovelace.dashboard_lixo`, sem arquivo YAML versionado equivalente. Em respeito à proibição de edição manual de `.storage`, esta implementação não alterou a interface. Após autorização operacional, a UI deverá adicionar Tempo OFF genérico e Confirmação da Queda, manter Automático e Estabilização e retirar da visualização os dois Tempos OFF numerados legados.

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
