# V20.2C — Automações Contextuais Residenciais

## Objetivo

A V20.2C é um lote corretivo da V20 para ativar recursos mínimos de monitoramento remoto quando Wilson estiver fora da residência. A permanência de Jacira ou de outra pessoa não bloqueia o fluxo. O lote não inicia formalmente a V21, não implementa Presence Intelligence e não representa monitoramento médico ou de bem-estar.

O conceito central passa a ser a **Sessão de Monitoramento Remoto**, contrato operacional confirmado e distinto da presença bruta. Seu componente arquitetural oficial é o **Coordenador da Sessão de Monitoramento Remoto (CSMR)**, promovido de forma limitada como motor de coordenação da saída e do retorno de Wilson.

A promoção e as decisões técnicas estão consolidadas documentalmente. O CSMR operacional completo ainda não foi promovido. O lote V20.2C-I1 implementou e homologou isoladamente a interface canônica V20.1O em `test_mode`, sem publicação real nem conexão de consumidores.

O lote V20.2C-I2 implementou a fundação transacional isolada do CSMR: estados `idle`, `starting`, `active`, `ending` e `failed`, UUID de sessão, idempotência por comando, checkpoint persistente, serialização e recuperação explícita. O componente permanece restrito ao Harness; presença, publicação e C1.x continuam desconectados.

A emenda V20.2C-I2A autoriza reserva prévia do `session_id` pelo próprio CSMR e a origem produtiva fechada `csmr_dispatcher_v20_2c`. Durante a emenda, ambos permanecem isolados de presença e Timeline. Retorno após saída já publicada preserva D1: concluir abertura sem consumidores e encerrar imediatamente a mesma sessão.

O Gate V20.2C-I3A foi homologado em 2026-08-07 pelo commit funcional `b11309bf20985a9385fb7918e82883dff4c8867e`. O dispatcher integra `person.wmoura`, a graça existente, revalidação, reserva/consumo do CSMR e o contrato I1 exclusivamente em `test_mode: true`. Ciclo nominal, cancelamento durante a graça, concorrência/D1, reload, correlação de ACKs e identidade única de sessão foram comprovados sem publicação produtiva ou consumidores.

O Gate V20.2C-I3B foi homologado em 2026-08-07, promovendo as quatro chamadas I1 para `test_mode: false` e preservando os contratos protegidos.

O Gate V20.2C-I4A foi homologado após a correção da corrida temporal entre eventos de consumidores e a transição para `active`, pelo commit funcional `b02e05d`. C1.1, C1.2 e C1.3 permanecem subordinados ao CSMR. O `return_pending` persistente bloqueia a passagem D1; uma fronteira temporal persistente, vinculada ao `session_id`, somente autoriza consumidores quando `CSMR == active`, `return_pending == false` e `occurred_at > consumer_authorized_since`. C1.2 usa `trigger.to_state.last_changed`, e o Harness exige `occurred_at` explícito. Starting, Ending, Failed, D1 e reload foram homologados sem consumidor indevido; o estado final foi `idle`, `return_pending=off` e `consumer_authorized_since=1970-01-01 00:00:00`.

Continuam pendentes a validação por ciclo real de saída/retorno e a evidência operacional do UniFi Protect. O próximo Gate autorizado é **V20.2C-I5B**.

## Gate V20.2C-I5A — Integração controlada do UniFi Protect

Homologado em 2026-08-07 por Harness, sem promoção operacional real. O package `packages/v20_2c_protect_csmr.yaml` separa a intenção automática `csmr_recording_requested` da intenção manual `manual_override`; a intenção efetiva é a disjunção das duas. `always` é aplicado aos selects `select.g4_instant_recording_mode` e `select.g4_instant_recording_mode_2` quando qualquer intenção está ativa; com ambas inativas, o baseline é `detections`.

Foram validados idle, sessão ativa autorizada, retorno pendente, ending, failed, override manual em casa, retorno com e sem override, reload, concorrência de intenção e idempotência. O retorno retira apenas a intenção CSMR; não desliga gravação manual. Não houve novos eventos de Timeline, nem alterações em CSMR, dispatcher, C1.x, Recovery, V20.1Q, Protect além dos selects durante o Harness ou dashboards. I5B permanece reservado à promoção operacional controlada.

## Governança A2 — Homologação Técnica e Evidência Operacional

O despacho V20.2C-A2 estabelece que Homologação Técnica e Evidência Operacional são etapas distintas. A Homologação Técnica valida arquitetura, implementação, runtime, contratos, estados, concorrência, persistência, consumidores, publicação, rollback, recovery e reload; quando o Harness reproduz integralmente esses contratos, ela pode ser concluída sem aguardar oportunidade física. A Evidência Operacional registra posteriormente a observação natural em produção e não altera código, arquitetura ou runtime.

Aplicação imediata: o Gate V20.2C-I4B foi dividido em I4B.1 — Promoção Operacional Controlada por Harness — e I4B.2 — Evidência Operacional do primeiro ciclo físico real. I4B.1 foi homologado em 2026-08-07 pelo Harness Dispatcher, incluindo sessão ativa, consumidores, retorno, D1, reload e bloqueio pós-retorno. I4B.2 permanece pendente e não bloqueia I5, I6, consumidores futuros ou UniFi Protect. Se hardware ou integração não forem reproduzíveis por Harness, o Gate correspondente continuará exigindo evidência física.

## Plano técnico restrito

O plano V20.2C-T1 está registrado em `plano_tecnico_csmr.md`. A decisão V20.2C-D1 nele incorporada resolve DP-1 a DP-5. O lote I1 criou `script.casa_publicar_evento_timeline_v20`, ACK correlacionado, ledger persistente e `test_mode`; não criou a máquina de estados do CSMR, dispatcher, sessão ou migração C1.x. O contrato vigente exige `source: csmr_v20_2c`, códigos `wilson_left_home`/`wilson_arrived_home` e `message` obrigatório.

## Relação com a V20.1Q e Git

Esta branch foi criada diretamente sobre o commit `486aa1a` da branch V20.1Q ainda não integrada. O empilhamento foi uma decisão consciente para preservar o runtime atual e manter as mudanças próprias da V20.2C em commit isolado. Depois da integração da V20.1Q em `develop`, a V20.2C deverá ser rebaseada ou reconciliada antes de integração.

## Escopo

- C0 — infraestrutura mínima: harness desligado por padrão, simulação isolada e fallback seguro.
- C1 — saída de Wilson: fonte real baseada exclusivamente em `person.wmoura`, com substituição controlada pelo harness.
- C1.1 — desligamento da Luz da Mesa após tempo de graça e revalidação.
- C1.2 — consumidor reativo da Porta da Sala durante sessão ativa, sem execução pontual pelo dispatcher.
- C1.3 — garantia idempotente de que a automação C1.2 esteja habilitada após a saída de Wilson.
- C1.4A — descoberta somente leitura das capacidades reais do UniFi Protect para gravação contextual, documentada em `c1_4a_unifi_protect.md`.
- A1 — promoção arquitetural limitada do CSMR, registrada em `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md` e condicionada ao Gate.

## Fronteiras da promoção

- V20.2 geral e Context Engine original permanecem em shadow.
- V20.1O permanece autoridade da Timeline e do Event Feed.
- CSMR decide ciclo de vida, ordem e liberação dos consumidores, mas não publica por canal paralelo.
- C1.1, C1.2 e C1.3 são ações subordinadas e não abrem nem encerram sessão.
- Somente os quatro eventos oficiais da sessão pertencem à promoção limitada.
- Nenhum package, helper, automação ou runtime é autorizado por esta consolidação documental.

## Fora de escopo

Integração funcional com Recovery 4G, failover, monitoramento de Internet, chuva, NVR, gestão de energia, Presence Intelligence, IA, inferência comportamental e qualquer monitoramento médico ou de bem-estar. O C1.4A inventaria o NVR, mas não implementa comportamento. O encerramento arquitetural da sessão pelo retorno de Wilson não autoriza automações funcionais de chegada.
