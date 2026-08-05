# V20.2C — Automações Contextuais Residenciais

## Objetivo

A V20.2C é um lote corretivo da V20 para criar uma fundação mínima, isolada e reversível de testes contextuais residenciais. Ela não inicia formalmente a V21 nem implementa Presence Intelligence.

## Relação com a V20.1Q e Git

Esta branch foi criada diretamente sobre o commit `486aa1a` da branch V20.1Q ainda não integrada. O empilhamento foi uma decisão consciente para preservar o runtime atual e manter as mudanças próprias da V20.2C em commit isolado. Depois da integração da V20.1Q em `develop`, a V20.2C deverá ser rebaseada ou reconciliada antes de integração.

## Escopo

- C0 — infraestrutura mínima de teste: harness desligado por padrão, simulação isolada e fallback seguro.
- C1 — saída de casa: detecção exclusivamente laboratorial de uma transição simulada para casa vazia.

## Fora de escopo

Tempo de graça, luz da mesa, push da porta, Recovery 4G, NVR, chuva, chegada, gestão de energia, Presence Intelligence, IA e inferência comportamental.
