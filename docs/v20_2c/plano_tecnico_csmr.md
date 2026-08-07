# V20.2C-T1 — Plano Técnico Restrito do CSMR

Data: 2026-08-06
Status: PLANEJADO; IMPLEMENTAÇÃO BLOQUEADA POR DECISÕES PENDENTES
Classificação: plano técnico auxiliar, subordinado à arquitetura e ao Gate V20.2C
Documento pai/índice: `docs/v20_2c/README.md`

## 1. Finalidade e limites

Este documento define o desenho técnico mínimo do **Coordenador da Sessão de Monitoramento Remoto (CSMR)**. Não implementa código, não promove o restante da V20.2 e não autoriza publicação ou alteração de runtime.

Escopo: ciclo de saída e retorno de Wilson, estado operacional da sessão, graça e revalidação únicas, publicação canônica dos quatro eventos autorizados, liberação ordenada de C1.1 e C1.3, preservação de C1.2, restart/reload, Harness, falhas, rollback e homologação futura.

Fora de escopo: NVR e `input_select.nvr_modo_gravacao`, chuva, Internet, failover, Recovery 4G, energia, Presence Intelligence, IA, novos eventos e qualquer outro componente V20.2.

Nenhum YAML final é especificado neste lote. Entity IDs e eventos novos abaixo são propostas sujeitas às decisões da seção 16.

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

Quando o repositório não define comportamento, este plano apresenta recomendação explicitamente não aprovada. Nenhuma recomendação pendente deve ser convertida em YAML antes da decisão formal correspondente.

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

Ordem recomendada dos consumidores: C1.3 antes de C1.1. Assim, a proteção de alerta é restabelecida antes da ação de conforto/segurança. C1.2 permanece habilitada e passa a observar a abertura confirmada, depois de C1.3, sem graça própria.

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
- Wilson ausente + `inativa`: não abrir silenciosamente. Recomendação pendente: iniciar uma graça completa somente depois de startup estabilizado e apenas para fonte real; Harness nunca reconcilia publicação.
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
→ liberar C1.3, C1.1 e C1.2 em sequência
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

Alteração mínima necessária: remover graça própria e receber liberação somente após C1.3. Preservar porta, Logbook, push, textos, Entity ID e `mode: restart`. A revalidação deve exigir sessão `ativa`. Sem essa adaptação, C1.2 continuaria com dupla graça e poderia ser acionada antes da abertura.

### Cancelamento, testes e rollback

O cancelamento passa a existir somente no coordenador antes da abertura. Depois da abertura, retorno é encerramento, não cancelamento. Os traces atuais continuam evidência histórica dos efeitos e da antiga graça; novos testes comprovam a graça centralizada.

Rollback restaura integralmente os três blocos atuais do package a partir do commit pré-implementação, antes de retirar helpers/scripts do CSMR. O package atual é a referência exata da estrutura anterior.

## 9. Liberação ordenada dos consumidores

Mecanismo recomendado: dispatcher sequencial com chamadas explícitas a scripts consumidores, não observadores simultâneos do boolean/sensor de sessão.

Ordem:

1. C1.3 garante C1.2 habilitada;
2. C1.1 desliga a luz se aplicável;
3. C1.2 avalia a porta e envia push se aplicável.

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

Contrato recomendado, ainda pendente de aprovação:

- evento interno: `casa_sessao_monitoramento_remoto`;
- payload mínimo: `fato`, `session_id`, `event_id`, `source: csmr`, `test_mode: false`;
- fatos aceitos: `wilson_saiu`, `monitoramento_iniciado`, `wilson_chegou`, `monitoramento_encerrado`;
- textos gerados exclusivamente dentro do publicador: os quatro textos oficiais com prefixo `HH:MM`;
- `publicar_timeline: true`, `enviar_push: false`, `permitir_agregacao: false`;
- payload inválido, `test_mode: true` ou fato desconhecido produz `sem_evento` e evidência observacional, nunca texto público.

O CSMR envia fatos, não texto público. Isso preserva formatação e autoridade da V20.1O.

### Confirmação possível

A chamada `event.fire` apenas confirma aceitação local do evento; não confirma atualização da Timeline. A confirmação mínima observável hoje é aguardar que `sensor.casa_timeline_v20` tenha `linha_1` igual ao texto esperado com timestamp, e que `eventos_json` o contenha. O Feed deriva da Timeline e é confirmação secundária, não requisito independente.

Essa confirmação é assíncrona e frágil sem correlação: eventos concorrentes podem mover a linha; a deduplicação pode aceitar o publicador mas não inserir a Timeline; o mesmo texto em outra sessão é indistinguível sem `event_id`. Recomendação: a extensão canônica deve expor atributos de correlação/resultado no próprio publicador ou em evento de ack emitido pela V20.1O. A forma exata é decisão pendente.

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

Se o primeiro evento publicar e o segundo falhar, a sessão não fica ativa e consumidores não são liberados. O estado permanece `erro_abertura`, com o primeiro checkpoint preservado. Não compensar publicando evento de encerramento, pois isso inventaria contrato. Retry cego também é proibido. A retomada depende da política de idempotência pendente.

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
| B: away + off/inativa | recomendação: graça completa após startup estável, apenas fonte real; decisão pendente | somente após graça e decisão aprovada | evitar inferir transição antiga | Harness não abre sessão |
| C: away + on/ativa | restaurar ativa | proibida na inicialização | não republicar entrada | Harness não altera fato restaurado |
| D: home + on/ativa | reconciliar encerramento pela fila | permitida somente com ack/idempotência aprovados | não desligar helper primeiro | Harness não autoriza publicação |
| E: reload durante graça | automação recarregada perde espera; estado segue inativa | proibida | startup/reload não deve abreviar graça; reiniciar graça completa ou aguardar nova transição, decisão pendente | simulação não publica |
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

## 16. Decisões pendentes obrigatórias

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

Enquanto DP-1, DP-2 e DP-3 não forem decididas, o lote de implementação não deve ser autorizado. DP-4 e DP-5 também devem constar do Gate pré-implementação.

## 17. Plano incremental

| Etapa | Arquivos futuros | Comportamento | Risco | Rollback | Validação e critério de saída |
| --- | --- | --- | --- | --- | --- |
| 0 — decisões | documentação/Gate | aprovar DP-1 a DP-5 e contrato | decisão incompleta | manter bloqueio | despacho/lote formal aprovado |
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

## 20. Conclusão

O desenho mínimo é viável com estado persistente por fases, coordenador serial, extensão formal do publicador V20.1O com correlação/ack e consumidores chamados em ordem. Porém, o repositório atual não possui a interface canônica do CSMR, confirmação idempotente nem política aprovada para recuperação parcial/startup ausente.

```text
Plano técnico insuficiente:
existem decisões obrigatórias pendentes antes da implementação
```

O próximo lote autorizado é exclusivamente decisório/documental para resolver DP-1 a DP-5 e atualizar o Gate. Lote de implementação não pode ser autorizado neste estado.
