# V20.2C — Automações Contextuais Residenciais

## Objetivo

A V20.2C é um lote corretivo da V20 para ativar recursos mínimos de monitoramento remoto quando Wilson estiver fora da residência. A permanência de Jacira ou de outra pessoa não bloqueia o fluxo. O lote não inicia formalmente a V21, não implementa Presence Intelligence e não representa monitoramento médico ou de bem-estar.

## Relação com a V20.1Q e Git

Esta branch foi criada diretamente sobre o commit `486aa1a` da branch V20.1Q ainda não integrada. O empilhamento foi uma decisão consciente para preservar o runtime atual e manter as mudanças próprias da V20.2C em commit isolado. Depois da integração da V20.1Q em `develop`, a V20.2C deverá ser rebaseada ou reconciliada antes de integração.

## Escopo

- C0 — infraestrutura mínima: harness desligado por padrão, simulação isolada e fallback seguro.
- C1 — saída de Wilson: fonte real baseada exclusivamente em `person.wmoura`, com substituição controlada pelo harness.
- C1.1 — desligamento da Luz da Mesa após tempo de graça e revalidação.
- C1.2 — alerta da Porta da Sala aberta após o mesmo tempo de graça.
- C1.3 — garantia idempotente de que a automação C1.2 esteja habilitada após a saída de Wilson.

## Fora de escopo

Integração com Recovery 4G, failover, monitoramento de Internet, chuva, NVR, chegada, gestão de energia, Presence Intelligence, IA, inferência comportamental e qualquer monitoramento médico ou de bem-estar.
