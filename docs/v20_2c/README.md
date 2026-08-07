# V20.2C — Automações Contextuais Residenciais

## Objetivo

A V20.2C é um lote corretivo da V20 para ativar recursos mínimos de monitoramento remoto quando Wilson estiver fora da residência. A permanência de Jacira ou de outra pessoa não bloqueia o fluxo. O lote não inicia formalmente a V21, não implementa Presence Intelligence e não representa monitoramento médico ou de bem-estar.

O conceito central passa a ser a **Sessão de Monitoramento Remoto**, contrato operacional confirmado e distinto da presença bruta. Seu componente arquitetural oficial é o **Coordenador da Sessão de Monitoramento Remoto (CSMR)**, promovido de forma limitada como motor de coordenação da saída e do retorno de Wilson.

A promoção e as decisões técnicas estão consolidadas documentalmente. O CSMR operacional completo ainda não foi implementado. O lote V20.2C-I1 implementou e homologou isoladamente a interface canônica V20.1O em `test_mode`, sem publicação real nem conexão de consumidores.

O lote V20.2C-I2 implementou a fundação transacional isolada do CSMR: estados `idle`, `starting`, `active`, `ending` e `failed`, UUID de sessão, idempotência por comando, checkpoint persistente, serialização e recuperação explícita. O componente permanece restrito ao Harness; presença, publicação e C1.x continuam desconectados.

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
