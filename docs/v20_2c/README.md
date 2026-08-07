# V20.2C — Automações Contextuais Residenciais

## Objetivo

A V20.2C é um lote corretivo da V20 para ativar recursos mínimos de monitoramento remoto quando Wilson estiver fora da residência. A permanência de Jacira ou de outra pessoa não bloqueia o fluxo. O lote não inicia formalmente a V21, não implementa Presence Intelligence e não representa monitoramento médico ou de bem-estar.

O conceito central passa a ser a **Sessão de Monitoramento Remoto**, contrato operacional confirmado e distinto da presença bruta. Seu componente arquitetural oficial é o **Coordenador da Sessão de Monitoramento Remoto (CSMR)**, promovido de forma limitada como motor de coordenação da saída e do retorno de Wilson.

A promoção está consolidada apenas no plano documental e arquitetural. O CSMR ainda não foi implementado, a publicação em runtime permanece desabilitada e o próximo passo obrigatório é um plano técnico restrito seguido da execução do Gate específico.

## Plano técnico restrito

O plano V20.2C-T1 está registrado em `plano_tecnico_csmr.md`. Ele confirma a viabilidade do desenho, mas identifica decisões obrigatórias ainda ausentes no contrato de entrada/ack da V20.1O, na idempotência de recuperação parcial e na reconciliação de startup. Por isso, o lote de implementação permanece bloqueado; o próximo lote é exclusivamente decisório/documental.

## Relação com a V20.1Q e Git

Esta branch foi criada diretamente sobre o commit `486aa1a` da branch V20.1Q ainda não integrada. O empilhamento foi uma decisão consciente para preservar o runtime atual e manter as mudanças próprias da V20.2C em commit isolado. Depois da integração da V20.1Q em `develop`, a V20.2C deverá ser rebaseada ou reconciliada antes de integração.

## Escopo

- C0 — infraestrutura mínima: harness desligado por padrão, simulação isolada e fallback seguro.
- C1 — saída de Wilson: fonte real baseada exclusivamente em `person.wmoura`, com substituição controlada pelo harness.
- C1.1 — desligamento da Luz da Mesa após tempo de graça e revalidação.
- C1.2 — alerta da Porta da Sala aberta após o mesmo tempo de graça.
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
