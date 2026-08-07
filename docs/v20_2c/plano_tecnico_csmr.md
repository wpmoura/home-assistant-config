# V20.2C-T1 — Plano Técnico Restrito do CSMR

Data: 2026-08-06
Status: I1, I2, I2A E I3A HOMOLOGADOS; PUBLICAÇÃO PRODUTIVA E CONSUMIDORES PENDENTES
Classificação: plano técnico auxiliar, subordinado à arquitetura e ao Gate V20.2C
Documento pai/índice: `docs/v20_2c/README.md`

## 1. Finalidade e limites

Este documento define o desenho técnico mínimo do **Coordenador da Sessão de Monitoramento Remoto (CSMR)**. Não implementa código, não promove o restante da V20.2 e não autoriza publicação ou alteração de runtime.

Escopo: ciclo de saída e retorno de Wilson, estado operacional da sessão, graça e revalidação únicas, publicação canônica dos quatro eventos autorizados, liberação ordenada de C1.1 e C1.3, preservação de C1.2, restart/reload, Harness, falhas, rollback e homologação futura.

Fora de escopo: NVR e `input_select.nvr_modo_gravacao`, chuva, Internet, failover, Recovery 4G, energia, Presence Intelligence, IA, novos eventos e qualquer outro componente V20.2.

Nenhum YAML final é especificado neste lote. A seção 20 registra a decisão V20.2C-D1 e prevalece sobre recomendações ou pendências históricas das seções anteriores.

## 2. Gate de conhecimento prévio

### Artefatos consultados

- Constituição: `docs/governance/constituicao_central_operacional_v20.md`;
- Source of Truth: `docs/governance/source_of_truth.md`;
- regras operacionais: `AGENTS.md`;
- arquitetura: `architecture.md` e `docs/ARCHITECTURE.md`;
- despacho A1: `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md`;
- Roadmap Executivo: `docs/ROADMAP.md`;
- Roadmap Consolidado: `docs/roadmap/roadmap_v20_consolidado.md`;
- Gate integral: `docs/governance/gates_v20.md`;
- V20.2C: `docs/v20_2c/README.md` e `docs/v20_2c/c1_saida_de_casa.md`;
- histórico/checkpoints: `CHANGELOG.md`, `docs/CHANGELOG.md` e `docs/pendencias_atuais_central_operacional.md`;
- implementação atual: `packages/v20_2c_contextual_automations.yaml`, `packages/motor_timeline_v20.yaml`, `packages/central_operacional_aliases_v20.yaml` e `.storage/core.entity_registry` somente em leitura.

### Artefatos não encontrados

- `docs/architecture.md`, citado textualmente no rodapé de `AGENTS.md`, não existe com essa capitalização; o documento encontrado e usado é `docs/ARCHITECTURE.md`;
- não existe plano técnico anterior do CSMR;
- não existe script, serviço ou evento de entrada do CSMR no publicador canônico;
- não existe confirmação/ack transacional de publicação na V20.1O;
- não existe contrato documentado para retomar uma transição parcial da sessão.

### Divergências e achados

- Não há conflito entre Constituição, arquitetura, despacho, Roadmaps, Gate e documentação V20.2C: todos bloqueiam implementação até aprovação do plano e do Gate.
- A descrição histórica de C1.2 ainda menciona graça própria; isso é o estado atual, não a arquitetura-alvo.
- O `id` YAML de C1.3 é `v20_2c_garantir_push_porta_ao_sair`, enquanto o Entity ID confirmado no registry é `automation.v20_2c_garantia_habilitar_push_da_porta_ao_sair`. Não renomear nenhum deles no lote futuro.
- O publicador não oferece uma entrada genérica. O evento `casa_recovery_4g_central` é precedente interno, mas seu payload é exclusivo do Recovery 4G e não constitui contrato reutilizável pelo CSMR.
- `sensor.casa_evento_publicavel_v20` não é gravável. Ele é um trigger-based template sensor que transforma fontes enumeradas em texto público.

### Hipótese de trabalho

Na versão T1 inicial, lacunas foram registradas sem suposição. A decisão V20.2C-D1 da seção 20 resolveu essas lacunas e é a referência normativa vigente.

## 3. Estado atual auditado

| Papel | Implementação atual | Observação |
| --- | --- | --- |
| autorização contextual | `binary_sensor.wilson_ausente_de_casa` | fonte real ou Harness; não é sessão |
| Harness | `input_boolean.teste_v20_2_harness_ativo` | `initial: false` |
| simulação | `input_boolean.teste_v20_2_simular_wilson_ausente` | `initial: false` |
| graça | `input_number.teste_v20_2_tempo_graca_saida` | 15 s homologados |
| C1.1 | `automation.v20_2c_teste_desligar_luz_da_mesa_apos_saida` | `mode: restart`; graça própria; alvo `light.smart_lampada_wifi_1` |
| C1.2 | `automation.v20_2c_teste_alertar_porta_aberta_apos_saida` | `mode: restart`; graça própria; porta e push homologados |
| C1.3 | `automation.v20_2c_garantia_habilitar_push_da_porta_ao_sair` | `mode: restart`; graça própria; habilita C1.2 idempotentemente |
| publicador | `sensor.casa_evento_publicavel_v20` | fontes fixas; gera `HH:MM mensagem` |
| histórico | `sensor.casa_timeline_v20` | lista limitada; deduplica evento consecutivo sem timestamp |
| feed | `sensor.casa_event_feed_v20` | espelha Timeline |
| contratos públicos | `sensor.casa_timeline`, `sensor.casa_event_feed` | aliases somente leitura para o CSMR |

O package atual não contém estado de sessão, CSMR, evento de sessão, checkpoint transacional ou reconciliação de startup.

## 4. Arquitetura-alvo

```text
binary_sensor.wilson_ausente_de_casa
        │
        ▼
automação CSMR (graça + revalidação + exclusão mútua)
        │
        ├── estado/checkpoint persistente da sessão
        │
        ├── solicitação formal ao publicador V20.1O
        │       └── sensor.casa_evento_publicavel_v20
        │              └── sensor.casa_timeline_v20
        │                     └── sensor.casa_event_feed_v20 + aliases
        │
        └── dispatcher sequencial após confirmação
                ├── C1.3 garantir C1.2 habilitada
                └── C1.1 desligar Luz da Mesa
```

Ordem das ações pontuais: C1.3 antes de C1.1. Assim, a proteção de alerta é restabelecida antes da ação de conforto/segurança. C1.2 não é executada pelo dispatcher; permanece consumidor reativo da abertura física da porta enquanto a sessão estiver ativa.

## 5. Representação da sessão

### Proposta

Usar um `input_select`, e não um `input_boolean` isolado:

- Entity ID: `input_select.casa_sessao_monitoramento_remoto_estado`;
- domínio: `input_select`;
- friendly name: `Casa — Sessão de Monitoramento Remoto — Estado`;
- opções propostas: `inativa`, `abrindo_evento_saida`, `abrindo_evento_monitoramento`, `ativa`, `encerrando_evento_chegada`, `encerrando_evento_monitoramento`, `erro_abertura`, `erro_encerramento`;
- estado inicial de primeira instalação: `inativa`;
- após restart: restaurar último estado; não declarar `initial` após implantação, pois isso destruiria o checkpoint;
- única autoridade lógica: automação principal do CSMR e rotina de rollback/reconciliação explicitamente autorizada.

Um binary sensor derivado, `binary_sensor.casa_sessao_monitoramento_remoto_ativa`, pode expor somente `estado == ativa`. Ele é leitura pública operacional, não autoridade e não substitui o helper de fase.

Um boolean sozinho não distingue abertura parcial, sessão ativa e encerramento parcial. Um template binary sensor derivado da ausência confundiria autorização contextual com confirmação operacional e não sobreviveria de forma segura a transações parciais.

### Proteção e persistência

Home Assistant não oferece ACL por entidade capaz de impedir que um administrador altere um helper. A proteção possível é operacional: não expor o helper em dashboard, documentar “não manipular”, registrar transições em Logbook, validar todas as mudanças no CSMR e reconciliar combinações inválidas. Essa limitação deve ser aceita formalmente.

Recomenda-se um identificador persistente adicional (`input_text.casa_sessao_monitoramento_remoto_id`) e um identificador da última solicitação confirmada (`input_text.casa_sessao_monitoramento_remoto_ultimo_evento`). Eles tornam retomada e auditoria possíveis, mas não criam histórico paralelo: guardam somente o ciclo corrente/último checkpoint técnico.

### Startup

- Wilson em casa + `inativa`: permanecer inativa; não publicar.
- Wilson ausente + `inativa`: permanecer inativa e aguardar futuro ciclo real `home → not_home`; não publicar nem liberar consumidores.
- Wilson ausente + `ativa`: restaurar ativa; não republicar entrada.
- Wilson em casa + `ativa`: iniciar reconciliação de encerramento, sem apagar o estado antes da confirmação.
- fase parcial: retomar do checkpoint somente se o contrato de idempotência/ack for aprovado; caso contrário, bloquear em erro observável.

### Rollback do estado

Primeiro impedir novos triggers, aguardar/cancelar execuções, restaurar consumidores legados, e só então colocar o helper em `inativa` e limpar IDs técnicos. Nunca remover helper enquanto automações ainda o referenciam.

## 6. Componentes propostos

Todos permaneceriam no package existente `packages/v20_2c_contextual_automations.yaml`, preservando rollback isolado.

| Componente | ID sugerido | Mode | Responsabilidade |
| --- | --- | --- | --- |
| coordenador | `automation.v20_2c_csmr_coordenar_sessao` | `queued`, `max: 1` | serializar saída, retorno e startup; decidir transições |
| publicar e confirmar | `script.v20_2c_csmr_publicar_evento` | `queued` | solicitar um fato ao canal canônico e aguardar evidência/timeout |
| liberar abertura | `script.v20_2c_csmr_liberar_consumidores` | `single` | chamar consumidores em ordem e isolar falhas |
| consumidor C1.1 | `script.v20_2c_csmr_consumidor_luz_mesa` | `single` | revalidar sessão/luz e desligar uma vez |
| consumidor C1.3 | `script.v20_2c_csmr_consumidor_garantir_push_porta` | `single` | revalidar sessão e habilitar C1.2 se necessário |
| reconciliação | preferencialmente branch `homeassistant.start` do coordenador | — | impedir sessão fantasma e tratar checkpoints |

O coordenador calcula uma vez: `origem`, `test_mode`, `grace_seconds`, `session_id`, fase e motivo do trigger. O snapshot da graça impede alteração do helper no meio do ciclo.

Concorrência: fila única e revalidação do estado ao consumir cada item. Triggers repetidos incompatíveis com a fase atual terminam sem publicação. Retorno durante graça cancela a espera; retorno durante abertura entra na fila e é tratado antes de consumidores, de acordo com a decisão de falha parcial.

## 7. Fluxo de entrada

Pseudofluxo proposto:

```text
trigger off → on da ausência
→ recusar se Harness para o caminho publicável
→ exigir estado inativa e fonte válida
→ capturar graça
→ esperar retorno ou timeout
→ se retornou: terminar sem evento
→ revalidar ausência, fonte real e estado inativa
→ criar session_id
→ fase abrindo_evento_saida
→ solicitar fato wilson_saiu
→ confirmar publicação ou entrar em erro_abertura
→ fase abrindo_evento_monitoramento
→ solicitar fato monitoramento_iniciado
→ confirmar publicação ou entrar em erro_abertura
→ fase ativa
→ executar C1.3 e depois C1.1
→ manter C1.2 reativo durante a sessão
```

A sessão somente é oficialmente aberta após confirmação do segundo evento e transição para `ativa`. Ativar antes das publicações permitiria consumidores prematuros e esconderia abertura incompleta.

## 8. Graça única e migração

### C1.1

Remover/adaptar: trigger de ausência, Logbook “graça iniciada”, `wait_for_trigger`, timeout e condições de “graça concluída”. Preservar condições da luz, revalidação de sessão ativa, `light.turn_off`, Logbook, alvo e comportamento idempotente.

Novo gatilho recomendado: chamada explícita pelo dispatcher. `mode: restart` deixa de ser necessário no script consumidor porque não há espera; a exclusão `single` é mais segura. Se a automação atual for mantida como casca para preservar trace/Entity ID, ela deve disparar por evento interno de liberação direcionado e continuar `mode: restart`, sem delay.

### C1.3

Remover/adaptar: trigger de ausência, `wait_for_trigger`, timeout e revalidações relativas à graça. Preservar teste de C1.2 `off`, `automation.turn_on`, Logbook e Entity ID atual.

Novo gatilho: chamada explícita pelo dispatcher, antes de C1.2. Operação continua idempotente; se C1.2 já estiver `on`, termina sem serviço.

### C1.2

Alteração mínima necessária: remover graça própria e o trigger de saída. C1.2 permanece automação reativa à porta, habilitada por C1.3 e autorizada somente durante sessão ativa. Seu gatilho futuro é a abertura física de `binary_sensor.sensor_porta_sala_contact`. Preservar porta, Logbook, push, textos, Entity ID e `mode: restart`. Ela não integra a sequência de chamadas pontuais do dispatcher.

### Cancelamento, testes e rollback

O cancelamento passa a existir somente no coordenador antes da abertura. Depois da abertura, retorno é encerramento, não cancelamento. Os traces atuais continuam evidência histórica dos efeitos e da antiga graça; novos testes comprovam a graça centralizada.

Rollback restaura integralmente os três blocos atuais do package a partir do commit pré-implementação, antes de retirar helpers/scripts do CSMR. O package atual é a referência exata da estrutura anterior.

## 9. Liberação ordenada dos consumidores

Mecanismo recomendado: dispatcher sequencial com chamadas explícitas a scripts consumidores, não observadores simultâneos do boolean/sensor de sessão.

Ordem das ações pontuais:

1. C1.3 garante C1.2 habilitada;
2. C1.1 desliga a luz se aplicável;

Depois dessas ações, C1.2 permanece armado e reage de forma independente à abertura física da porta durante a sessão. O estado ativo é autorização, não gatilho para enviar push.

Cada consumidor revalida `sessão == ativa` e `session_id`. O trace do dispatcher e os traces dos scripts comprovam início/fim e ordem. Falha de consumidor é subordinada: deve ser registrada, mas os seguintes continuam por branches isolados com resposta/resultado explícito. Falha não volta a sessão para “abrindo” nem republica eventos.

Novos consumidores são adicionados como scripts independentes ao dispatcher, com fase declarada, contrato de entrada (`session_id`) e resultado. Esse padrão mantém ordem sem fazer cada módulo conhecer a Timeline. A lista, porém, continua deliberadamente explícita; não usar descoberta dinâmica de entidades.

## 10. Publicação canônica V20.1O

### Mecanismo existente

O caminho real atual é:

```text
fontes enumeradas / evento casa_recovery_4g_central
→ trigger-based template sensor.casa_evento_publicavel_v20
→ sensor.casa_timeline_v20
→ sensor.casa_event_feed_v20
→ aliases sensor.casa_timeline e sensor.casa_event_feed
```

`sensor.casa_evento_publicavel_v20` gera o timestamp e os textos, além dos atributos `publicar_timeline`, `enviar_push` e `permitir_agregacao`. A Timeline reage à mudança do publicador, recusa valores vazios/inválidos e deduplica apenas o mesmo texto-base consecutivo (ignorando `HH:MM`). O Event Feed espelha a Timeline. O CSMR nunca chama ou escreve nos quatro sensores.

### Lacuna concreta

Não existe mecanismo oficial de entrada para os quatro fatos do CSMR. A implementação exige um lote formal que estenda `packages/motor_timeline_v20.yaml`, preservando o mesmo sensor e sem reabrir silenciosamente a política V20.1O.

Proposta histórica T1, substituída pelo contrato definitivo da seção 20:

- evento interno: `casa_sessao_monitoramento_remoto`;
- payload mínimo atualizado pelo I1: `request_id`, `session_id`, `event_code`, `message`, `source: csmr_v20_2c`, `schema_version: 1`, `test_mode: false`;
- fatos aceitos: `wilson_saiu`, `monitoramento_iniciado`, `wilson_chegou`, `monitoramento_encerrado`;
- textos gerados exclusivamente dentro do publicador: os quatro textos oficiais com prefixo `HH:MM`;
- `publicar_timeline: true`, `enviar_push: false`, `permitir_agregacao: false`;
- payload inválido, `test_mode: true` ou fato desconhecido produz `sem_evento` e evidência observacional, nunca texto público.

O CSMR envia fatos, não texto público. Isso preserva formatação e autoridade da V20.1O.

### Confirmação possível

A chamada `event.fire` apenas confirma aceitação local do evento; não confirma atualização da Timeline. A confirmação mínima observável hoje é aguardar que `sensor.casa_timeline_v20` tenha `linha_1` igual ao texto esperado com timestamp, e que `eventos_json` o contenha. O Feed deriva da Timeline e é confirmação secundária, não requisito independente.

Essa confirmação observacional isolada é assíncrona e frágil. A seção 20 resolve a lacuna por script canônico, `request_id`, ACK e ledger da V20.1O.

Timeout recomendado, sujeito a homologação: 10 segundos por evento. Abertura não avança no timeout. O valor não tem fundamento runtime ainda e precisa ser validado.

## 11. Semântica de sucesso e falha parcial

Sucesso mínimo futuro: solicitação aceita **e** ack correlacionado pelo mesmo `event_id`, com resultado `published` ou `deduplicated_expected`, acompanhado da presença na Timeline quando `published`. Apenas chamar serviço não basta.

Requisito runtime recomendado:

1. enviar um fato;
2. aguardar ack correlacionado;
3. confirmar que Timeline registrou o texto quando o ack disser `published`;
4. persistir o checkpoint;
5. seguir ao próximo fato.

Deduplicação de um fato novo da sessão não pode ser tratada automaticamente como sucesso: pode ocultar duplicidade ou colisão. Deve resultar em erro observável até o contrato distinguir retry idempotente do mesmo `event_id` de uma nova solicitação.

Se o primeiro evento publicar e o segundo falhar, a sessão não fica ativa e consumidores não são liberados. O checkpoint é preservado e somente o request pendente é repetido conforme DP-3, sem compensação ou reemissão do par.

Falha ao ativar `ativa` depois de dois acks deve impedir consumidores e gerar erro observacional. A falta de transação atômica entre Timeline e helper é risco residual que exige reconciliação explícita.

## 12. Encerramento

Gatilho: transição de `binary_sensor.wilson_ausente_de_casa` de `on` para `off`, processada pela mesma fila. Não há fundamento para graça de retorno; portanto, não se propõe nenhuma.

```text
retorno
→ exigir fase ativa
→ fase encerrando_evento_chegada
→ publicar/confirmar wilson_chegou
→ fase encerrando_evento_monitoramento
→ publicar/confirmar monitoramento_encerrado
→ fase inativa e limpeza do session_id corrente
```

Retorno em `inativa` termina sem publicação. Retorno durante graça cancela abertura sem evento. Retorno durante abertura parcial não pode publicar encerramento nem apagar checkpoint; entra em falha crítica de abertura até decisão de recuperação. Oscilação rápida é serializada: somente uma sessão ativa pode encerrar e nenhuma nova saída inicia enquanto o estado não voltar a `inativa`.

Falha parcial de encerramento mantém `erro_encerramento`, sessão logicamente não encerrada e consumidores de encerramento bloqueados. O próximo ciclo não começa até reconciliação. Após os dois acks e transição para `inativa`, um novo ciclo pode iniciar.

## 13. Restart e reload

| Cenário | Estado esperado e correção | Publicação | Duplicidade/reconciliação | Harness |
| --- | --- | --- | --- | --- |
| A: home + off/inativa | permanecer inativa | proibida | nenhuma | termina/permanece desligado |
| B: away + off/inativa | permanecer inativa até novo ciclo real | proibida | não inferir transição antiga | Harness não abre sessão real |
| C: away + on/ativa | restaurar ativa | proibida na inicialização | não republicar entrada | Harness não altera fato restaurado |
| D: home + on/ativa | reconciliar encerramento pela fila | permitida somente com ack/idempotência aprovados | não desligar helper primeiro | Harness não autoriza publicação |
| E: reload durante graça | abortar graça e voltar a inativa | proibida | exigir nova transição real | simulação não publica |
| F: reload entre eventos de entrada | checkpoint parcial restaurado | proibido retry cego | retomar pelo `event_id` somente após contrato idempotente; senão `erro_abertura` | teste termina sem evento público |
| G: reload durante encerramento | checkpoint parcial restaurado | proibido retry cego | mesma regra; manter bloqueio de novo ciclo | teste termina sem evento público |

Reload de automações não emite `homeassistant.start`; por isso uma rotina só de startup é insuficiente. O plano futuro deve diferenciar startup de reload ou aceitar que fases parciais ficam em erro seguro até reconciliação administrativa. Não usar trigger de mudança do helper como autorização para publicar.

## 14. Harness

O Harness continua controlando `binary_sensor.wilson_ausente_de_casa`, mas `source: simulation` deve bloquear o branch publicável e qualquer ação física durante homologação inicial. Ele valida:

- graça central única;
- cancelamento;
- ausência de abertura/publicação simulada;
- ciclos lógicos em shadow técnico;
- serialização e guards;
- estado final desligado.

A tensão documental é resolvida separando duas provas:

1. **Harness:** prova que simulação não vira fato público e valida o coordenador sem efeitos;
2. **caminho canônico:** testes estáticos/isolados do contrato do publicador e, depois, um ciclo real controlado autorizado pelo operador comprovam os quatro eventos.

Publicar os textos oficiais a partir do Harness violaria a regra de fonte não publicável. Marcar textos como teste criaria eventos não autorizados. Criar canal isolado alteraria a arquitetura. Portanto, nenhuma dessas opções é recomendada sem nova decisão arquitetural. A homologação real posterior é obrigatória para Timeline/Event Feed.

Ao final de qualquer teste: Harness e simulação `off`, fase `inativa` salvo teste explícito de restauração, nenhuma execução pendente e nenhum consumidor físico acionado por simulação.

## 15. Falhas e rollback

### Classificação

| Falha | Classe | Comportamento seguro |
| --- | --- | --- |
| primeira/segunda publicação de entrada, helper de fase, template de autorização | crítica de abertura | não ativar sessão; não liberar consumidores; checkpoint + Logbook/trace |
| C1.1 ou C1.3 | consumidor subordinado | registrar resultado e continuar consumidores independentes; sessão permanece ativa |
| primeira/segunda publicação de retorno | crítica de encerramento | não declarar inativa; bloquear ciclo seguinte; checkpoint + Logbook/trace |
| Logbook/atributo diagnóstico | observacional | não mudar decisão já confirmada; registrar em trace/log do HA quando possível |
| publicador/helper indisponível, automação CSMR desabilitada | crítica conforme fase | fail closed; nenhuma progressão silenciosa |
| restart no fluxo | crítica até reconciliação | restaurar checkpoint; nunca retry cego |

Erros de template devem produzir recusa segura e nunca default permissivo. Indisponibilidade temporária do publicador pode aguardar até o timeout; após isso exige reconciliação, não loop ilimitado.

### Rollback técnico

Arquivos funcionais futuros previstos:

- `packages/v20_2c_contextual_automations.yaml`;
- `packages/motor_timeline_v20.yaml` somente após lote formal de extensão V20.1O;
- nenhuma alteração em `packages/central_operacional_aliases_v20.yaml`.

Ordem:

1. desligar Harness e simulação e bloquear novos inícios;
2. confirmar/cancelar execuções do CSMR em janela operacional;
3. se houver fase parcial, registrar checkpoint e decidir reconciliação antes de apagar estado;
4. restaurar C1.1/C1.2/C1.3 atuais com graça própria;
5. remover/desabilitar dispatcher e coordenador;
6. reverter a extensão de entrada CSMR na V20.1O, preservando todo o restante do motor;
7. colocar helpers do CSMR em estado seguro e só então removê-los em lote posterior, se autorizado;
8. validar V20.1O, aliases e consumidores.

Um reload controlado de templates/helpers/automações pode bastar se a validação de configuração confirmar suporte; restart somente se exigido pelo mecanismo realmente implementado. A escolha não pode ser fixada antes do YAML futuro. Nenhum reload/restart é parte deste lote.

## 16. Registro histórico das lacunas T1 — resolvidas pela seção 20

### DP-1 — Interface canônica de entrada e ack

- Pergunta: a V20.1O poderá receber o evento `casa_sessao_monitoramento_remoto` e emitir confirmação correlacionada?
- Evidência: só há fontes enumeradas e precedente exclusivo `casa_recovery_4g_central`; não existe entrada CSMR nem ack.
- Opções: evento tipado no mesmo publicador; nova fonte helper observada pelo mesmo publicador; extensão com ack explícito.
- Riscos: reabrir V20.1O silenciosamente, colisão, ausência de confirmação e duplicidade.
- Recomendação: lote formal de extensão do mesmo publicador, evento tipado + `event_id` + ack, sem sensor/histórico paralelo.
- Runtime futuro: sim, para validar latência, ack e Timeline.

### DP-2 — Política de idempotência e recuperação parcial

- Pergunta: como reprocessar o mesmo `event_id` após reload sem duplicar e como concluir/abortar pares parciais?
- Evidência: deduplicação atual compara apenas texto consecutivo sem timestamp e não conhece sessão/evento.
- Opções: ack/cache idempotente limitado no publicador; falha manual sem retry; transação persistente específica.
- Riscos: evento duplicado, sessão incompleta ou bloqueio permanente.
- Recomendação: idempotência correlacionada no caminho canônico, com retenção mínima suficiente para restart; sem histórico paralelo no CSMR.
- Runtime futuro: sim.

### DP-3 — Ausência já vigente no startup

- Pergunta: startup com Wilson ausente e sessão inativa deve abrir após nova graça ou permanecer inativo até nova transição?
- Evidência: documentos exigem análise, mas não escolhem política; “sessão fantasma” proíbe inferência insegura.
- Opções: graça completa após estabilização; aguardar nova transição; intervenção manual.
- Riscos: perder monitoramento real versus criar sessão baseada em ausência antiga.
- Recomendação: graça completa apenas para fonte real válida, após startup estabilizado, se formalmente aprovada.
- Runtime futuro: sim, inclusive restart.

### DP-4 — Segurança administrativa do helper

- Pergunta: aceita-se proteção operacional, já que não há ACL de escrita por entidade para administradores?
- Evidência: helper nativo é manipulável por serviço/UI com permissão adequada.
- Opções: helper oculto/documentado; estado interno de integração customizada; persistência externa.
- Riscos: manipulação indevida versus complexidade não nativa.
- Recomendação: helper nativo não exposto, guards e Logbook; aceitar limitação explicitamente.
- Runtime futuro: não para decidir; sim para homologar guards.

### DP-5 — Timeout e semântica de deduplicação

- Pergunta: qual timeout e quais resultados de ack contam como sucesso?
- Evidência: não há medição nem ack hoje.
- Opções: 5/10/30 s; `published` apenas; `published` e retry idempotente confirmado.
- Riscos: falso timeout ou falso sucesso.
- Recomendação: 10 s inicial; sucesso em `published` ou retry do mesmo `event_id` comprovadamente já publicado, nunca deduplicação textual genérica.
- Runtime futuro: sim.

DP-1 a DP-5 foram resolvidas integralmente pela decisão V20.2C-D1 da seção 20.

## 17. Plano incremental

| Etapa | Arquivos futuros | Comportamento | Risco | Rollback | Validação e critério de saída |
| --- | --- | --- | --- | --- | --- |
| 0 — decisões | documentação/Gate | resolver DP-1 a DP-5 e contrato | nenhum risco funcional | reverter commit documental | concluída pela V20.2C-D1 |
| 1 — estado | package V20.2C | helpers de fase/ID e sensor derivado, sem consumidor | restauração errada | remover referências antes dos helpers | parser, restart matrix sem publicação |
| 2 — CSMR shadow | package V20.2C | graça, cancelamento e fila; Harness sem publicação/efeito | sessão fantasma | desabilitar coordenador | traces de cancelamento e ciclos shadow |
| 3 — publicador canônico | motor Timeline + package | entrada CSMR e ack, ainda sem consumidores | regressão V20.1O | reverter somente branch CSMR do template | eventos sintéticos controlados conforme autorização, aliases intactos |
| 4 — abertura/encerramento | package V20.2C | pares sequenciais e checkpoints | parcial/retry | voltar shadow | ordem, ack, timeout e restart aprovados |
| 5 — migrar C1.3 | package V20.2C | remover graça própria; primeira liberação | push ficar off | restaurar bloco atual | traces on/off/cancelamento |
| 6 — migrar C1.1 | package V20.2C | remover graça própria; ação após C1.3 | ação física indevida | restaurar bloco atual | luz off/on, idempotência, ordem |
| 7 — migrar C1.2 | package V20.2C | retirar dupla graça e liberar após C1.3 | regressão do push | restaurar bloco atual | porta fechada/aberta e push único |
| 8 — homologação | sem novo escopo | ciclos, falhas, reload/restart, real coordenado | efeito produtivo | rollback preparado | matriz Gate completa e estado seguro |
| 9 — promoção runtime | docs/checkpoint | habilitar apenas CSMR aprovado | regressão tardia | rollback funcional restrito | observação, commit e Gate encerrados |

Cada etapa funcional deve ser commit pequeno e reversível; a referência de “um único commit” aplica-se somente a este plano documental.

## 18. Matriz completa do Gate V20.2C

| Critério | Implementação prevista | Evidência futura |
| --- | --- | --- |
| despacho registrado | preservado, sem mudança | link e diff documental |
| CSMR classificado | package restrito e aliases próprios | inventário YAML |
| restante V20.2 shadow | nenhuma dependência de motores `_v20_2` | busca estática |
| V20.1O preservada | extensão formal no mesmo publicador | diff restrito + regressão |
| fronteiras definidas | coordenador/publicador/consumidores separados | traces e revisão |
| plano técnico aprovado | este documento após decisões | aprovação explícita |
| quatro eventos somente | enum de quatro fatos no publicador | testes de payload inválido |
| formato `HH:MM mensagem` | texto gerado pela V20.1O | Timeline observada |
| sem escrita direta | CSMR usa evento tipado | busca de serviços/alvos |
| sem histórico/dedup paralelo | somente checkpoint corrente | inspeção YAML |
| caminho concreto validado | DP-1 implementada no sensor existente | trace + ack |
| cancelamento antes da graça | espera única no CSMR | trace sem evento/efeito |
| abertura uma vez | fase + session/event IDs | trace e Timeline |
| ordem da entrada | chamadas/acks sequenciais | timestamps, ack e trace |
| consumidor após publicação | dispatcher após fase ativa | trace causal |
| encerramento uma vez | guard `ativa` + IDs | trace e Timeline |
| ordem do retorno | chamadas/acks sequenciais | ack e trace |
| retorno sem sessão | guard `inativa` | trace recusado |
| ciclos independentes | novo `session_id` por ciclo | dois ciclos completos |
| restart/reload sem fantasma | checkpoint e reconciliação | cenários A–G |
| Harness não publicável | guard `test_mode/source` | testes negativos |
| C1.1 preservado | script mantém alvo/condições | trace e luz real autorizada |
| C1.2 preservado | porta/push/textos mantidos | trace e push único |
| C1.3 preservado | garantia idempotente mantida | trace enabled/disabled |
| Harness preservado | mesmos helpers, default off | ciclos simulados |
| V20.1O preservada | testes de eventos existentes | regressão Timeline/Push |
| Timeline/Feed preservados | aliases intocados | estados/atributos comparados |
| nenhuma promoção implícita | busca por consumidores/fontes | diff auditado |
| falha observável | fases de erro + Logbook/trace | injeção de timeout/indisponível |
| sem progressão após falha | dispatcher bloqueado | trace de falha crítica |
| rollback restrito | dois arquivos funcionais previstos | ensaio de reversão |
| V20.1O após rollback | branch CSMR removível | regressão pós-rollback |
| sem execução/sessão pendente | ordem de rollback | inspeção de traces/estado |
| helpers/Harness seguros | `inativa`, IDs limpos, booleans off | snapshot final |
| parser YAML | validação estática | saída do parser |
| configuração HA | check config futuro | saída registrada |
| traces/Logbook | modos e logs preservados | IDs de traces |
| Timeline/Feed comprovados | ack + conteúdo público | snapshot/atributos |
| ordem/dedup/ciclos | matriz de cenários | relatório de homologação |
| working tree/commit | lotes auditados | status, diff e hashes |
| homologação real coordenada | etapa 8 com operador | autorização e evidências |

Os Gates gerais 0–6 também se aplicam: escopo e limites deste plano estão declarados; contratos protegidos não são alterados neste lote; arquitetura é respeitada; evidência documental é registrada; rollback futuro está definido; este plano funciona como checkpoint documental; status final é `Planejada/Bloqueada` até decisões.

## 19. Riscos residuais

- Event Bus do Home Assistant não é fila durável; restart pode perder solicitação ou ack.
- Helpers não formam transação atômica com trigger-based template sensors.
- Deduplicação textual atual não prova idempotência transacional.
- Evento concorrente pode mover `linha_1` antes da confirmação observacional.
- Entity IDs de automações derivados de aliases podem divergir do `id` YAML; usar registry auditado e não renomear.
- Reload e restart têm semânticas diferentes; ambos precisam de prova.
- C1.2 desabilitada não pode receber trigger; por isso C1.3 deve precedê-la.
- Um consumidor físico não deve ser testado pelo Harness sem autorização explícita.

## 20. V20.2C-D1 — decisões técnicas definitivas

Esta seção resolve DP-1 a DP-5 e substitui toda marcação “pendente”, “recomendação” ou alternativa aberta registrada anteriormente neste plano. Nenhum parâmetro fica delegado ao lote de implementação.

### DP-1 — entrada canônica e ACK

#### Escolha

A API pública escolhida é o script canônico `script.casa_publicar_evento_timeline_v20`, pertencente à V20.1O. Ele valida, solicita a publicação ao `sensor.casa_evento_publicavel_v20`, aguarda o resultado canônico e retorna resposta estruturada.

Um evento público isolado foi rejeitado porque comprova apenas despacho. Helpers de comando/resposta foram rejeitados porque permitem colisão entre solicitações. Internamente, o script pode disparar um evento privado para alimentar o trigger-based template sensor; esse evento não constitui API autorizada para o CSMR ou para a V20.2 geral.

#### Contrato de entrada — schema 1

| Campo | Tipo e regra |
| --- | --- |
| `schema_version` | inteiro obrigatório, exatamente `1` |
| `request_id` | UUID obrigatório, criado e persistido antes da chamada; invariável em retries |
| `source` | enum autorizada; neste lote somente `csmr_v20_2c` |
| `session_id` | UUID obrigatório e único por ciclo real ou ciclo Harness |
| `event_code` | enum dos quatro códigos autorizados |
| `message` | string obrigatória, não vazia e exatamente correspondente ao `event_code` |
| `test_mode` | boolean obrigatório |

Códigos e mensagens validados exclusivamente pela V20.1O:

| `event_code` | mensagem pública derivada |
| --- | --- |
| `wilson_left_home` | `📍 Wilson saiu de casa` |
| `remote_monitoring_started` | `🛡️ Monitoramento remoto iniciado` |
| `wilson_arrived_home` | `📍 Wilson chegou em casa` |
| `remote_monitoring_ended` | `🛡️ Monitoramento remoto encerrado` |

O solicitante envia `message`, mas não envia `timestamp`, flags de Timeline/Push/agregação nem Entity ID de destino. A V20.1O exige correspondência exata entre código e mensagem, gera `HH:MM`, fixa `enviar_push: false` e `permitir_agregacao: false`, e valida fonte, schema, tipos, evento e identidades.

Rastreabilidade V20.2C-I1: o contrato decisório original usava `source: csmr`, códigos `wilson_left`/`wilson_arrived` e mensagem derivada. A autorização formal do lote I1 substituiu esses valores por `csmr_v20_2c`, `wilson_left_home`/`wilson_arrived_home` e `message` obrigatório. A substituição é evolução formal do contrato, não altera os papéis arquiteturais: o CSMR continua solicitante futuro e a V20.1O continua única autoridade publicadora. Os valores anteriores são rejeitados pelo schema 1 atualizado.

#### Contrato de ACK — schema 1

O ACK é retornado pelo script e também emitido pela V20.1O no evento privado observacional `casa_timeline_publicacao_ack_v20`.

| Campo | Tipo e regra |
| --- | --- |
| `schema_version` | inteiro `1` |
| `request_id` | correlação exata |
| `source` | fonte validada |
| `session_id` | sessão validada |
| `event_code` | código validado |
| `status` | `published`, `validated_test`, `duplicate`, `rejected` ou `failed` |
| `ack_at` | timestamp de processamento emitido pela V20.1O |
| `reason` | código estável; vazio apenas em sucesso |

`published` somente ocorre depois de a entrada estar incorporada à Timeline e seu request confirmado pelo caminho canônico. `validated_test` confirma payload, fonte, identidade e texto sem persistir Timeline/Feed. `duplicate` significa `request_id` já processado ou identidade lógica já registrada no namespace correspondente. Payload inválido ou fonte não autorizada recebe `rejected`, sem alteração pública. Erro técnico recebe `failed`.

Correlação usa `request_id`. Identidade lógica usa `source + session_id + event_code`. Reenvio do mesmo request ou novo request para identidade lógica já registrada retorna `duplicate` e referencia o processamento original; sessão diferente permanece identidade diferente.

Impacto futuro: extensão formal e restrita de `packages/motor_timeline_v20.yaml`, preservando sensores, aliases, formato, limite e fontes existentes. Rollback remove script, trigger privado, ACK e ledger do CSMR; Recovery e demais fontes permanecem intactos.

### DP-2 — máquina de estados e persistência

Estados oficiais:

```text
idle
opening_wait
opening_event_1_pending
opening_event_1_confirmed
opening_event_2_pending
active
closing_event_1_pending
closing_event_1_confirmed
closing_event_2_pending
error_opening
error_closing
```

Transições permitidas:

```text
idle → opening_wait
opening_wait → idle | opening_event_1_pending
opening_event_1_pending → opening_event_1_confirmed | error_opening
opening_event_1_confirmed → opening_event_2_pending
opening_event_2_pending → active | error_opening
active → closing_event_1_pending
closing_event_1_pending → closing_event_1_confirmed | error_closing
closing_event_1_confirmed → closing_event_2_pending
closing_event_2_pending → idle | error_closing
error_opening → estado pending correspondente, por retomada governada
error_closing → estado pending correspondente, por retomada governada
```

Nenhuma outra transição é válida. Dados persistentes mínimos: fase, `session_id`, `current_request_id`, `current_event_code`, `last_confirmed_event_code`, `last_ack_status`, `last_ack_reason`, `created_at`, `source` e `test_mode`. Guardam apenas a transação corrente, não histórico paralelo.

O request é persistido antes da chamada. Após restart/reload, uma fase `pending` repete somente o request corrente com o mesmo ID; se ele já tiver sido publicado, a V20.1O responde `duplicate`. `active` é restaurado sem republicação. O estado `opening_wait` perdido por reload volta a `idle`, pois nenhum evento foi publicado e uma nova transição real é exigida.

Reconciliação é acionada em `homeassistant.start`, no evento nativo de reload das automações e quando o coordenador muda de desabilitado para habilitado. Automação desabilitada não progride; seus dados persistem. Reabilitação não cria sessão: apenas reconcilia a fase registrada.

Abortar é permitido somente em `opening_wait`, antes de qualquer publicação. Depois do primeiro ACK, o par deve ser concluído ou permanecer em erro observável. Após conclusão de encerramento, limpar IDs, ACK, timestamps e origem, mantendo apenas `idle`.

### DP-3 — recuperação de par parcial

Estratégia escolhida: retomar somente o evento pendente, com o mesmo `session_id` e o mesmo `request_id`. Nunca reemitir o par completo e nunca publicar evento compensatório.

Política fechada:

- uma tentativa inicial e no máximo duas repetições;
- 5 segundos entre repetições;
- mesmo `request_id` em todas;
- `duplicate` é sucesso somente quando ID e identidade lógica coincidem no ledger;
- `rejected` nunca recebe retry automático;
- `failed` e timeout recebem retry até o limite;
- esgotamento leva a `error_opening` ou `error_closing`;
- recuperação manual corrige a causa e retoma o mesmo request; não edita Timeline nem limpa checkpoint;
- abertura parcial nunca libera consumidores;
- encerramento parcial bloqueia o próximo ciclo.

Se Wilson retornar durante abertura parcial, o CSMR conclui o segundo evento de entrada sem liberar consumidores e, em seguida, publica o par de encerramento da mesma sessão. Isso preserva os quatro eventos, a ordem e a rastreabilidade sem sessão operacional aberta silenciosamente.

### DP-4 — startup com Wilson já ausente

Política oficial: **não abrir automaticamente** quando Wilson estiver ausente e a fase persistida for `idle`.

Não se publica “Wilson saiu” sem observar a saída. Ausência prolongada permanece sem sessão e sem consumidores até que ocorra um futuro ciclo real `home → not_home`. Não há comando manual para forçar os eventos oficiais.

- Harness: `away + idle` em teste pode iniciar apenas ciclo `test_mode`, sem Timeline pública ou ação física;
- sessão persistida `active` + Wilson ausente: restaurar `active`, sem republicar;
- sessão persistida `active` + Wilson em casa: reconciliar o par de encerramento;
- Wilson em casa + `idle`: permanecer `idle`;
- consumidores: nenhum é liberado em `away + idle`;
- risco residual aceito: uma ausência real prolongada pode ficar sem monitoramento após perda/ausência de sessão persistida;
- teste futuro obrigatório: restart nos quatro estados básicos e prova negativa de `away + idle`.

### DP-5 — timeout, unicidade e retenção

Timeout de ACK: 10 segundos por chamada. Depois de timeout/`failed`, aguardar 5 segundos e repetir até duas vezes. Após três chamadas totais, entrar em erro. Os valores são normativos; qualquer ajuste futuro exige lote formal.

`duplicate` só equivale a sucesso quando o ledger confirma simultaneamente o mesmo `request_id`, `source`, `session_id` e `event_code`. Deduplicação visual por texto nunca é ACK e não participa da idempotência transacional.

A V20.1O retém requests publicados até que ambas as condições sejam excedidas: últimos 16 eventos **e** no mínimo 7 dias. Requests `validated_test` ficam em ledger técnico não público por 24 horas. O CSMR mantém a transação corrente até conclusão; se o ledger expirar e não houver prova inequívoca, permanece em erro para recuperação manual, sem republicar.

Sessões diferentes podem publicar a mesma mensagem porque possuem `session_id` distinto. A mesma identidade com request diferente é conflito; o mesmo request com identidade diferente também é conflito.

### Harness definitivo

O Harness usa a API canônica com `test_mode: true`. A V20.1O executa todas as validações, deriva os textos e devolve `validated_test`, mas não altera `sensor.casa_evento_publicavel_v20`, Timeline, Event Feed ou aliases. Ações físicas permanecem bloqueadas.

ACKs e traces comprovam correlação e ordem. O Event Feed não pode ser comprovado dinamicamente por teste que deliberadamente não persiste: sua derivação é validada estaticamente e, depois, um ciclo real coordenado e autorizado comprova Timeline e Event Feed com os quatro textos oficiais. O Harness termina desligado, simulação desligada e fase de teste limpa.

### Consumidores

```text
ACK wilson_left_home
→ ACK remote_monitoring_started
→ fase active
→ dispatcher executa C1.3
→ dispatcher executa C1.1
→ C1.2 permanece reativo à porta durante active
```

C1.3 e C1.1 são ações pontuais de preparação. C1.2 não é chamada pelo dispatcher e não envia push ao simples início da sessão; C1.3 apenas garante que ela esteja habilitada. C1.2 reage a `binary_sensor.sensor_porta_sala_contact` e exige sessão ativa.

### Quadro decisório

| Decisão | Escolha | Fundamentação | Risco residual | Impacto futuro |
| --- | --- | --- | --- | --- |
| DP-1 | script canônico + ACK | validação pertence à V20.1O | atomicidade script/evento | extensão restrita do motor |
| DP-2 | fase persistente + ledger | retry seguro após restart | expiração do ledger | helpers e atributos internos |
| DP-3 | retomar somente pendente | exatamente uma ocorrência e ordem | bloqueio em erro | reconciliação governada |
| DP-4 | não abrir `away + idle` | não afirmar saída falsa | ausência sem sessão | teste negativo obrigatório |
| DP-5 | 10 s, 2 retries, 5 s, retenção 16/7 dias | limite determinístico | ajuste exige novo lote | matriz de falhas |

### Matriz final de autorização

| Item | Resolvido? | Evidência documental |
| --- | --- | --- |
| Entrada canônica | Sim | DP-1, contrato de entrada |
| ACK correlacionado | Sim | DP-1, contrato de ACK |
| Máquina de estados | Sim | DP-2 |
| Restart/reload | Sim | DP-2 |
| Par parcial | Sim | DP-3 |
| Startup ausente | Sim | DP-4 |
| Timeout | Sim | DP-5 |
| Idempotência | Sim | DP-2 e DP-5 |
| Harness | Sim | Harness definitivo |
| Liberação de consumidores | Sim | Consumidores |

## 21. Conclusão

O desenho mínimo está fechado com estado persistente por fases, coordenador serial, script canônico V20.1O, ACK correlacionado, ledger idempotente, retomada exclusiva do evento pendente, startup conservador e Harness por ACK sem persistência pública. A interface ainda não existe no runtime; sua implementação pertence ao próximo lote.

```text
Decisões obrigatórias resolvidas:
lote técnico de implementação pode ser autorizado.
```

A autorização é documental e condicionada ao Gate pré-implementação. Nenhum código ou comportamento foi habilitado por esta decisão.

## 22. V20.2C-I1 — contrato canônico implementado

O lote I1 implementa somente a interface V20.1O, sem conectar o CSMR ou qualquer consumidor. As entidades introduzidas são `script.casa_publicar_evento_timeline_v20` e `sensor.casa_timeline_publicacao_ack_v20`. O script opera em `mode: queued`, serializando verificação, publicação e atualização do ledger para impedir corrida entre identidades concorrentes.

O ACK é retornado como resposta do serviço e projetado no sensor persistente. O sensor mantém namespaces separados `ledger_producao` e `ledger_teste`; produção conserva todos os registros dos últimos 7 dias e, mesmo após essa janela, pelo menos os 16 mais recentes. O namespace técnico conserva no máximo 16 entradas por 24 horas e nunca bloqueia uma futura sessão produtiva.

Em `test_mode: true`, todas as validações e verificações de idempotência são executadas, o ACK válido é `validated_test` e nenhum evento alcança `sensor.casa_evento_publicavel_v20`. No caminho real, não exercitado neste lote, o script usa o evento interno restrito `casa_timeline_publicar_canonico_v20`, aguarda até 10 segundos pela incorporação do `request_id` e faz no máximo duas repetições, separadas por 5 segundos, reutilizando o mesmo ID. O ACK `published` só é emitido após confirmação; esgotamento resulta em `failed`.

Homologação controlada em 2026-08-06: payload válido retornou `validated_test`; reenvio do request e nova requisição da mesma identidade retornaram `duplicate`; sessão diferente foi aceita; source/códigos antigos, mensagem ausente/vazia/incompatível e schema inválido foram rejeitados. Após reload parcial, o request persistido continuou duplicado. Dezessete entradas técnicas provaram limite, ordem e descarte do mais antigo no ledger de teste, mantendo `ledger_producao` vazio. Timeline, Event Feed e aliases não receberam nenhum dos quatro textos oficiais.

Rollback: remover `packages/contrato_publicacao_timeline_v20.yaml` e os blocos identificados como `contrato_canonico_v20`/`request_ids_json` em `packages/motor_timeline_v20.yaml`, validar a configuração e recarregar templates e scripts. Não há restauração de banco, Timeline, Event Feed, V20.1Q ou C1.x.

## 23. V20.2C-I2 — estado transacional isolado implementado

O lote I2 implementa somente a fundação persistente do CSMR, sem presença, graça, publicação, dispatcher ou consumidores. A representação observável é `sensor.casa_csmr_estado_v20_2c`; o único writer autorizado é `script.casa_csmr_transicionar_v20_2c`, restrito ao Harness `test_mode: true` e às ações `open`, `close` e `recover`.

### Modelo e equivalência

| Estado I2 | Semântica | Equivalência futura no modelo detalhado D1 |
| --- | --- | --- |
| `idle` | nenhuma sessão | `idle` |
| `starting` | abertura transacional em curso | fases `opening_*` posteriores à graça |
| `active` | sessão lógica ativa | `active` |
| `ending` | encerramento transacional em curso | fases `closing_*` |
| `failed` | falha preservada | `error_opening` ou `error_closing`, distinguido por `transition_action` e `last_consistent_state` |

O I2 não elimina nem redefine as fases detalhadas necessárias quando a publicação for conectada. Ele implementa a camada mínima autorizada e mantém nos atributos a etapa, o estado anterior e o último estado consistente.

Transições válidas: `idle → starting → active`, `active → ending → idle`, `starting → failed`, `ending → failed` e `failed → idle` exclusivamente por `recover`. Qualquer outra combinação é rejeitada. Repetição do mesmo `request_id` recupera o resultado do ledger sem nova transição.

### Identidade, persistência e concorrência

O CSMR gera um UUID v4 de `session_id` uma única vez ao aceitar `open`; o ID permanece invariável em `starting`, `active`, `ending` e eventual `failed`, e é arquivado em `last_session_id` ao retornar a `idle`. Cada comando exige UUID distinto de `request_id`. O checkpoint registra timestamps, origem `harness_i2`, motivo, erro, estado anterior, estado consistente e até 16 conclusões técnicas.

O sensor trigger-based com `unique_id` é restaurado pelo Home Assistant após reload/restart. Estados transitórios não disparam qualquer reconciliação automática: permanecem observáveis e bloqueiam novos comandos incompatíveis; `failed` exige recuperação explícita. Não há trigger de `person.wmoura`, `homeassistant.start` ou ausência, portanto `away + idle` não cria sessão.

O script usa `mode: queued`; todas as requisições revalidam o checkpoint ao sair da fila. Duas aberturas concorrentes resultam deterministicamente em uma conclusão e uma rejeição `session_already_active`, preservando um único `session_id`.

### Homologação I2

Em 2026-08-06 foram comprovados: abertura `starting → active`; reenvio idempotente; rejeição de nova abertura ativa; encerramento `ending → idle`; encerramento repetido; rejeição em `idle`; preservação de sessão e ledger após reload de templates/scripts; serialização de duas aberturas simultâneas; falhas simuladas de abertura e encerramento; recuperação explícita; e bloqueio de `test_mode: false`.

Timeline, Event Feed, contrato I1, presença, C1.1, C1.2, C1.3, Recovery 4G e modos de gravação do Protect permaneceram fora do package e inalterados. O teste de restart físico não foi necessário: persistência foi comprovada por reload e a restauração após restart permanece propriedade nativa documentada do trigger-based template sensor, a ser reobservada junto ao futuro lote de integração.

Rollback I2: remover `packages/csmr_estado_transacional_v20_2c.yaml`, validar a configuração e recarregar templates/scripts. Como o lote não publicou nem acionou dispositivos, não há banco, histórico, consumidor ou estado físico a restaurar.

## 24. V20.2C-I2A — emenda operacional limitada

A emenda I2A resolve formalmente os bloqueios encontrados antes do I3 sem alterar os cinco estados principais nem o contrato I1. O CSMR permanece autoridade exclusiva para gerar `session_id` e passa a oferecer reserva persistente antes da publicação de `wilson_left_home`.

### Reserva e cancelamento

`reserve` somente é aceito em `idle`, sem sessão ou reserva corrente. O CSMR gera UUID v4, mantém o estado principal em `idle` e persiste `session_reserved`, `reserved_session_id`, `reservation_request_id`, `reserved_at`, origem, motivo e modo. Reenvio do mesmo request recupera a mesma reserva; novo request concorrente não gera outro UUID.

`cancel_reservation` somente atua sobre reserva não consumida em `idle`. O cancelamento é explícito, idempotente, registrado no ledger e limpa o checkpoint corrente sem publicar ou entrar em `failed`. Retorno durante a graça continua anterior à reserva e não cria qualquer identidade.

`open` produtivo consome exatamente o UUID reservado e exige que o chamador apresente o mesmo `session_id`. O vínculo com `reservation_request_id` fica em `consumed_reservation_request_id`. O Harness legado permanece autorizado a abrir sem reserva e gerar sua sessão internamente.

### Modos e origens

Lista fechada:

| Modo | Origem | Uso |
| --- | --- | --- |
| `test_mode: true` | `harness_i2` | homologação isolada; ausência histórica de `source` é normalizada para este valor |
| `test_mode: false` | `csmr_dispatcher_v20_2c` | dispatcher operacional oficial do futuro I3 |

Qualquer outra combinação é rejeitada. `test_mode: false` não autoriza publicação por si só e falha simulada permanece proibida em modo produtivo. A emenda apenas permite transições de estado; durante sua homologação não há presença, I1 ou Timeline.

### Retorno durante abertura

Fica preservada a decisão D1. Depois que `wilson_left_home` foi confirmado, retorno antes de `open` ou durante `starting` não desfaz a saída: a fila conclui `open`, publica `remote_monitoring_started`, não libera consumidores, publica `wilson_arrived_home`, conclui `close` e então publica `remote_monitoring_ended`, sempre com o mesmo `session_id`.

Rastreabilidade do conflito: a autorização inicial do I3 continha a regra “não publicar `remote_monitoring_started` após retorno confirmado durante `starting`”. A revisão preventiva identificou incompatibilidade com D1 e risco de deixar `wilson_left_home` isolado. O despacho I2A substitui essa regra pela conclusão transacional completa acima. Falha de `open` ou da publicação de início continua interrompendo a cadeia e exige recuperação explícita.

### Persistência e rollback

A reserva usa o mesmo sensor trigger-based restaurável do I2 e o mesmo ledger limitado a 16 conclusões. Reload/startup não consome, abre ou cancela reserva automaticamente. Rollback da emenda restaura a versão I2 do package e recarrega templates/scripts; não exige restaurar Timeline, banco, consumidores ou dispositivos.

### Homologação isolada I2A

Em 2026-08-06, `reserve` manteve `idle` e criou UUID persistente; reenvio devolveu o mesmo UUID; novo request e reserva concorrente não criaram segunda identidade. Reload preservou reserva, request e origem. `cancel_reservation` limpou o checkpoint e sua repetição foi idempotente.

A origem `csmr_dispatcher_v20_2c` com `test_mode: false` foi aceita exclusivamente no estado transacional, sem I1 ou Timeline. Origem externa, combinação de modo incorreta, UUID divergente, `open` produtivo sem reserva e simulação de falha produtiva foram rejeitados. O `open` consumiu exatamente o UUID reservado e `close` arquivou a mesma sessão.

O Harness legado sem `source` continuou operando como `harness_i2`, incluindo geração interna, fechamento, falha simulada e recuperação. Na simulação D1, `close` chegou enquanto `open` estava em `starting`; `mode: queued` concluiu `active` e depois `idle` com o mesmo UUID. Nenhuma Timeline, presença, ação C1.x ou dispositivo participou do teste.

## 26. V20.2C-I4A — Integração temporal dos consumidores

Status: HOMOLOGADO em 2026-08-07; commit funcional `b02e05d`.

O Gate confirmou a subordinação de C1.1, C1.2 e C1.3 ao CSMR, sem alterar a máquina I2/I2A, I1, Timeline, Event Feed, Recovery, Protect ou `person.wmoura`. O `input_boolean.casa_csmr_retorno_pendente_v20_2c` é persistente e bloqueia consumidores durante o retorno e D1.

A autorização operacional exige cumulativamente:

- `CSMR == active`;
- `return_pending == false`;
- `session_id` do evento igual à sessão autorizada;
- `occurred_at > consumer_authorized_since`.

`consumer_authorized_since` e a sessão autorizada são persistidos por helpers mínimos. A fronteira é criada somente após `active` operacional e ACK de `remote_monitoring_started`; é invalidada em `idle`, `starting`, `ending`, `failed` e ao ligar `return_pending`. C1.2 captura o instante físico por `trigger.to_state.last_changed`; o Harness carrega `occurred_at` explícito. C1.2 permanece reativo à abertura física da porta, e C1.3 executou uma única garantia sem duplicação indevida.

### Evidências de homologação

- `homeassistant.check_config` aprovado;
- reloads parciais de helpers, templates, scripts e automações retornaram HTTP 200;
- Starting e Ending rejeitaram eventos ocorridos fora da janela autorizada;
- Failed não acionou consumidores;
- D1 manteve a fronteira inválida e não acionou C1.1, C1.2 ou C1.3;
- Active nominal produziu uma execução de C1.2 e uma execução de C1.3;
- reload com `return_pending=true` preservou o bloqueio;
- nenhum restart foi executado e nenhum componente protegido foi alterado.

Estado final homologado: `CSMR=idle`, `return_pending=off`, `consumer_authorized_since=1970-01-01 00:00:00`.

Permanecem pendentes a promoção operacional real dos consumidores, a validação por ciclo real de saída/retorno, UniFi Protect e demais consumidores futuros. O próximo Gate é V20.2C-I4B.

## 25. V20.2C-I3A — integração lógica homologada

O I3A integra a presença oficial `person.wmoura`, o helper existente `input_number.teste_v20_2_tempo_graca_saida`, o dispatcher, a reserva I2A, o estado transacional e o contrato I1. A implementação funcional está no commit `b11309bf20985a9385fb7918e82883dff4c8867e`, em `packages/csmr_dispatcher_integracao_v20_2c.yaml`.

Todas as quatro chamadas ao I1 permanecem em `test_mode: true`. As chamadas internas `reserve`, `open`, `close` e `cancel_reservation` usam a origem fechada `csmr_dispatcher_v20_2c`; isso movimenta somente o checkpoint transacional e não autoriza publicação produtiva.

### Evidências homologadas

| Cenário | Evidência e resultado |
| --- | --- |
| ciclo nominal | graça de 15 s, reserva, consumo, `active`, retorno e `idle`; quatro ACKs `validated_test` em ordem |
| identidade | um `session_id` por ciclo; `request_id` próprio e determinístico por evento |
| retorno durante graça | `cancelled_during_grace`; sem reserva, sessão ou novo ACK I1 |
| retorno durante `starting` / D1 | fila `queued` concluiu abertura e encerramento; quatro ACKs ordenados na mesma sessão; nenhum consumidor |
| reload durante graça | execução original preservada uma única vez; nenhuma nova reserva/sessão ou republicação; encerramento controlado posterior |
| reload com sessão/checkpoint | I2/I2A já comprovaram persistência de sessão/reserva; I3A não criou sessão fantasma nem trigger de startup |
| duplicate, retry e idempotência | contrato I1 comprovou `duplicate`, timeout de 10 s e até duas repetições de 5 s com o mesmo request; I2/I2A comprovaram reenvio e reserva idempotentes |
| concorrência e cancelamento de reserva | serialização I2/I2A preservada; I3A serializou saída/retorno e manteve um único UUID |
| falhas | I1 comprovou rejeição/falha sem progressão; I2 comprovou falhas simuladas de abertura/encerramento e recuperação explícita |

Sessões observadas na homologação I3A incluíram `15fb53f7-23f2-4fbf-8571-441f9cb98d0c` no ciclo nominal e `9892bf79-46fb-49ba-8e98-a890e39c5dab` no cenário D1. Em ambos, os eventos mantiveram o mesmo UUID durante todo o ciclo e request IDs distintos.

### Ausência de impacto e limites

Timeline e Event Feed produtivos não receberam os quatro textos; somente o ledger técnico I1 foi atualizado. V20.1Q, Recovery 4G, C1.1, C1.2, C1.3, UniFi Protect, dashboards e `person.wmoura` não foram modificados. Nenhum consumidor foi conectado ou executado.

Permanecem fora do I3A: publicação produtiva, promoção operacional, C1.1, C1.2, C1.3, UniFi Protect e consumidores futuros. O próximo Gate autorizado é exclusivamente V20.2C-I3B, limitado à troca de `test_mode: true` por `test_mode: false` nas quatro chamadas I1 e à homologação operacional controlada, sem outra alteração arquitetural ou funcional.

## 27. V20.2C-A2 — Governança de homologação

O despacho A2 institui a distinção permanente entre **Homologação Técnica** e **Evidência Operacional**. A primeira valida os contratos equivalentes por Harness, incluindo runtime, estados, concorrência, persistência, consumidores, publicação, rollback, recovery e reload. A segunda observa o comportamento natural em produção, sem alterar implementação; sua ausência não bloqueia o Roadmap quando a equivalência técnica estiver comprovada.

O I4B é reestruturado em duas etapas:

- **I4B.1 — Promoção Operacional Controlada:** homologação técnica pelo Harness dos consumidores, dispatcher, CSMR, Timeline, publicação, autorização, push, D1, reload, rollback e proteção dos componentes.
- **I4B.2 — Evidência Operacional:** registro do primeiro ciclo físico real `home → not_home → home`, sem correções durante a coleta. Comportamento inesperado abre Gate corretivo próprio.

I4B.1 libera a evolução do Roadmap após aprovação. I4B.2 fica `PENDENTE DE EVIDÊNCIA OPERACIONAL` e não bloqueia I5, I6, consumidores futuros ou UniFi Protect. Hardware ou integrações sem Harness equivalente continuam sujeitos a validação física obrigatória.

### Homologação I4B.1

Em 2026-08-07, o Harness Dispatcher percorreu graça, abertura, publicação canônica, sessão `active`, autorização temporal e encerramento. O ciclo usou `session_id=739eb841-6033-48b0-8907-ba27c1459b43`; C1.1 e C1.3 executaram uma vez na autorização e C1.2 uma vez após `occurred_at` posterior à fronteira. O retorno invalidou a fronteira, terminou em `idle` e a porta pós-retorno não acionou C1.2. Reloads parciais retornaram HTTP 200 sem reabertura ou evento retroativo. Um `cycle_id` residual foi reconciliado por checkpoint `idle`, sem publicação ou alteração funcional. Estado final: CSMR/dispatcher `idle`, `return_pending=off`, fronteira inválida e IDs vazios. I4B.1 está HOMOLOGADO; I4B.2 permanece PENDENTE DE EVIDÊNCIA OPERACIONAL.

## 28. V20.2C-I5A — Integração controlada do UniFi Protect

O I5A foi homologado por Harness em 2026-08-07. O modelo separa `csmr_recording_requested`, `manual_override`, `effective_state` e o estado observado dos selects `select.g4_instant_recording_mode` e `select.g4_instant_recording_mode_2`. A função canônica é `effective = csmr_recording_requested OR manual_override`: qualquer intenção ativa aplica `always`; ambas inativas restauram o baseline `detections`.

O Harness cobriu idle, active autorizado, retorno pendente, ending, failed, transições manuais, retorno com override preservado, reload, concorrência e idempotência. `return_pending` bloqueia somente a intenção automática do CSMR. Não foram criados eventos de Timeline nem alterados CSMR, dispatcher, consumidores C1.x, Recovery, V20.1Q ou dashboards. O I5B fica reservado à promoção operacional controlada; a evidência física permanece posterior e não altera este contrato.
