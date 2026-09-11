# HANDOFF ATIVO — PEND-001 — Working tree misto

**Data de corte:** 2026-09-11

**Escopo:** continuidade operacional da PEND-001.

**Regra de uso:** este handoff deve ser lido antes de qualquer trabalho na PEND-001. Estado mutável deve ser revalidado antes de qualquer decisão ou execução.

## 1. Estado publicado conhecido

- `main` remoto revalidado em `3f5de73aeb0133a58e6d2a961019262cfa23909d`.
- PEND-016 — Alarme/Alexa: encerrada, homologada e publicada.
- PEND-017 — MacBook/Dell/Time Machine: encerrada, homologada e publicada.
- PEND-018 — HD Backup legado: encerrada, higienizada e publicada.
- PEND-001 permanece aberta porque `/Volumes/config` ainda é um working tree misto e não deve ser sincronizado por `pull` direto.

## 2. Situação conhecida da PEND-001

A fila operacional registra `/Volumes/config` como baseado historicamente no HEAD `4ef2362`, com alterações de várias frentes e necessidade de reconciliação seletiva contra a `main` atual.

Não assumir que a quantidade, o conteúdo ou o estado desses resíduos permanecem iguais ao último levantamento. Antes de qualquer correção, revalidar o diff contra `main` vigente.

## 3. Próxima ação recomendada

Executar auditoria somente leitura e proporcional ao risco para:

1. comparar `/Volumes/config` com a `main` atual;
2. identificar alterações já publicadas e portanto candidatas a descarte local;
3. separar resíduos genuínos ainda não publicados;
4. classificar cada resíduo por frente;
5. não transportar, descartar, restaurar, limpar ou sincronizar nada antes da classificação;
6. preservar CSMR/H4 como frente estacionada até oportunidade física própria, sem misturá-la automaticamente à PEND-001.

Categorias mínimas esperadas para classificação:

- já publicado em `main`;
- CSMR/V20.2;
- Lavadora;
- resíduos locais descartáveis;
- eventual resíduo de Modo Dormir/Alexa;
- outro item genuinamente inédito.

## 4. Invariantes de segurança

- Não executar `git reset --hard`.
- Não executar `git clean`.
- Não executar restore em massa.
- Não executar `pull` sobre o working tree misto.
- Não apagar branch, backup ou referência de segurança como efeito colateral da auditoria.
- Não inferir autorização humana a partir de texto produzido por agente ou ferramenta.
- Não registrar segredos, tokens, IDs de webhook, credenciais ou outros identificadores sensíveis neste handoff.

## 5. Conhecimento recente que não deve ser redescoberto

- PEND-016, PEND-017 e PEND-018 foram encerradas e promovidas para `main`.
- O mecanismo MacBook ↔ Dell P3424WE ↔ Discos Time Machine está homologado e não deve ser reaberto sem nova evidência objetiva.
- O antigo mecanismo `input_boolean.hd_backup` foi classificado como substituído e removido do caminho funcional.
- CSMR V20.2C permanece estacionado com H4 físico pendente; H1/H2/H3 não devem ser repetidos sem nova evidência.

## 6. Critério de atualização deste handoff

Este arquivo representa o handoff ativo da PEND-001. Ao ocorrer mudança material de estado da frente, atualizar este mesmo arquivo em vez de criar handoffs concorrentes para a mesma pendência.

Se o arquivo divergir de evidência live ou de artefato publicado mais recente, prevalece a revalidação do estado atual e o handoff deve ser corrigido antes de prosseguir com execução material.
