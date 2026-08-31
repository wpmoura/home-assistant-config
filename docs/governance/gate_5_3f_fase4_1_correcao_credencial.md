# Gate 5.3F — Fase 4.1: Correção da Credencial e Validação Offline Pós-HTTP 401

**Data:** 2026-08-30
**Escopo:** investigação forense do HTTP 401 da Fase 4, correção manual da credencial pelo usuário, validação offline pós-correção. **Zero chamadas Anthropic nesta fase.**

## 1. Evidência do HTTP 401 (Fase 4), lida diretamente do trace persistido

- `execution_id`: `hc-mtfz4wx5-a0qgbbqh`
- `status_http`: **401**
- `request-id`: `req_011CeZBZaafyTbgzzUvg5aXd`
- `corpo_erro` (sanitizado): `{"type":"authentication_error","message":"invalid x-api-key"}`
- Nenhum erro adicional além da autenticação presente no trace.

## 2. Análise: resposta genuína da Anthropic (não proxy/intermediário)

Evidência convergente: corpo no formato JSON estruturado exato da Anthropic Messages API (`type`/`message`), `content-type: application/json`, request-id no padrão `req_<alfanumérico>` da Anthropic. Contraste direto com o incidente do harness local (Fase 3.2), onde uma requisição mal-roteada bateu no proxy nginx do add-on e retornou HTML `text/html` "401 Authorization Required" — um padrão visivelmente distinto. A evidência documental disponível aponta fortemente para uma resposta genuína da Anthropic (não é possível prova absoluta sem inspeção de rede direta, fora do escopo desta fase).

Evidência indireta de que o header `x-api-key` foi de fato enviado (sem inspecionar seu valor): o código de `gate53b_sf_fn_build_real_request` só monta e envia a requisição HTTP **depois** de passar pelo guard `if (!chave) { executor_erro='credencial_ausente' }`. Como o resultado observado foi um 401 vindo da própria Anthropic (não `credencial_ausente`), confirma-se estruturalmente que uma credencial não-vazia existia e foi incluída no header.

## 3. Classificação da causa

Não é possível distinguir, apenas com evidência offline, entre chave inválida, revogada/desativada, ou colada incorretamente — a mensagem `"invalid x-api-key"` da Anthropic é genérica o suficiente para cobrir os três casos.

**Classificação:** "credencial apresentada à Anthropic não foi aceita; causa específica da invalidade não determinável offline."

Descartadas por evidência: chave inexistente no runtime (refutado — passou o guard `!chave`); header não enviado (refutado); mecanismo de credencial não entregou o segredo ao runtime (refutado — `env.get()` retornou valor não-vazio).

## 4. Verificação do mecanismo de credencial (sem exposição de segredo)

- Variável `anthropic_api_key` existe no subflow `gate53b_subflow_executor`, tipo `cred` — confirmado.
- Nem o template do subflow nem a instância `gate53b_subflow_instance` expõem campo `value` no JSON exportado via GET — apenas `name` e `type`. Confirmado estruturalmente (leitura do flow completo, checagem de chaves presentes).
- O valor real vive exclusivamente no armazenamento de credenciais criptografado do Node-RED, nunca no JSON do flow.
- Ausência em Git: confirmada (`git status --short` filtrado por termos relacionados a segredo/chave/credencial não retornou nenhum resultado).
- Ausência em estados/atributos do HA: confirmada (o sensor `sensor.saude_sistema_analitico_status` nunca carrega o valor, apenas o booleano `credencial_presente`).

## 5. Correção manual (ação do usuário)

O usuário substituiu manualmente a credencial na instância "HC Executor (mock)" (aba Gate 5.3B) e realizou Deploy normal pela interface do Node-RED. Nenhuma chave foi recebida, solicitada, escrita em arquivo, código, flow JSON ou documentação por este processo.

## 6. Validação offline pós-correção (zero chamadas Anthropic)

Mecanismo de credencial testado exclusivamente via caminho **MOCK/local**: disparo do teste bidirecional já existente (`input_boolean.gate53b1_teste_disparo_ha`, origem `ha_bidirectional_test_531`, que não passa pela coleta real nem pelo executor real — bypass direto para o executor MOCK). Execução `hc-mtfzr9sg-kfmbp9t5` concluída com `contract_ok:true`, `success`, **`credencial_presente_no_mock:true`**. Nenhuma requisição saiu para `api.anthropic.com`.

| Item | Resultado |
|---|---|
| 1. Credencial presente | **SIM** (`credencial_presente_no_mock:true`) |
| 2. Mecanismo credential funcional | **SIM** |
| 3. Segredo ausente em GET/export do flow | **SIM** |
| 4. Segredo ausente em logs | **SIM** (nenhum ponto do código loga a credencial) |
| 5. Segredo ausente no Git | **SIM** |
| 6. Segredo ausente em estados HA | **SIM** |
| 7. Pipeline MOCK | **SIM** |
| 8. Caminho Anthropic DARK | **SIM** |
| 9. Scheduler Desativado | **SIM** |
| 10. Lock livre | **SIM** |
| 11. Nenhuma execução real ocorreu | **SIM** |
| 12. Contador histórico permanece 2 | **SIM** |

## 7. Limite explícito desta fase

Esta fase **não pode e não afirma** que a credencial é aceita pela Anthropic — isso exigiria uma chamada externa, fora do escopo autorizado. A conclusão máxima permitida e aqui declarada é: **credencial presente e recuperável localmente pelo runtime**. A aceitação real pela Anthropic só poderá ser comprovada em eventual execução real futura, mediante nova autorização humana explícita (terceira chamada, exigindo ajuste do guard `>= 2` → `>= 3`).

## 8. Riscos residuais

- A aceitação da nova credencial pela Anthropic permanece não verificada até uma eventual terceira chamada real futuramente autorizada.
- O guard permanece em `>= 2`; uma terceira chamada exigirá ajuste explícito e documentado do mesmo, seguindo o padrão de menor-alteração-possível já estabelecido.
- A condição de corrida de reconciliação (Fases 3.1/3.2) permanece um limite de design aceito, mitigado pela espera de ≥15-20s pós-deploy.

## 9. Princípio permanente reafirmado

A IA não é fonte da verdade operacional. `sensor.saude_sistema_status` permanece intocado. Nenhuma ação física foi disparada. Nenhuma credencial foi impressa, logada ou reproduzida em texto — apenas presença booleana. Zero chamadas Anthropic nesta fase.
