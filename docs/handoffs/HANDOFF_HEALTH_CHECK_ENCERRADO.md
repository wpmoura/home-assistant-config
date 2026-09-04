# Handoff Auxiliar Encerrado — Health Check / Saúde do Sistema

Data da incorporação seletiva: 2026-09-04
Status do handoff: `ENCERRADO`
Roadmap: `SOC`

## Classificação e proveniência

Este é um artefato auxiliar de continuidade. Não é fonte canônica, não autoriza execução e não substitui Constituição, Source of Truth, roadmap, Gates, Changelog, implementação ou evidência de runtime.

Foi produzido pela incorporação seletiva de `docs/governance/HANDOFF_HEALTH_CHECK_ATUAL.md`, existente em `main`. O original extenso permanece preservado naquela história Git. Ele não foi copiado integralmente porque contém instruções e fotografias operacionais de sessões anteriores que poderiam ser confundidas com estado ou autorização atuais.

## Último estado comprovado no handoff de origem

- Health Check implementado, homologado e em operação normal.
- Camada determinística permanece soberana; a camada analítica com IA é opcional e não executa ação física.
- Botão manual e scheduler usam o mesmo pipeline, lock e máquina de estados.
- Scheduler homologado em `1x por dia`, com execução prevista para 08:00 `America/Sao_Paulo` e verificação em janela de 15 minutos.
- Primeira execução automática real auditada em 2026-09-02, com `origem=scheduled`, contrato válido, custo recalculado compatível, contadores coerentes e sem indício de retry ou duplicidade.
- Seleção Haiku/Sonnet e modo MOCK são controláveis pela interface do Home Assistant.
- Diagnóstico de falhas Anthropic preserva o erro original e não deve afirmar causa sem evidência.
- Credencial Anthropic não pode ser exposta, impressa, registrada ou recuperada por documentação.

Esses fatos descrevem o checkpoint histórico do handoff. Estado mutável de Home Assistant, Node-RED, scheduler, modelo, lock, helpers e credenciais deve ser revalidado antes de qualquer nova ação.

## Evidências Git registradas na origem

| Marco | Evidência |
| --- | --- |
| Fundação e Gate 5.3 | PR #2 mergeado em `main` |
| Contrato booleano | PR #4 mergeado em `main` |
| Seleção de modelo | PR #5 mergeado em `main` |
| Diagnóstico de acesso Node-RED | PR #7 mergeado em `main` |
| Integração Haiku/Sonnet | PR #8 mergeado em `main` |
| Entrada em operação normal | PR #10, merge `fbda933` |
| Billing/créditos | PR #11, merge `7fae4e2` |
| Primeira execução automática | PR #12, merge `1a810ed` |
| Fechamento documental | PR #13, tip de `main` no checkpoint `73a870d` |

Os identificadores acima servem para navegação. Antes de reutilizá-los como evidência atual, consultar diretamente Git/GitHub e os documentos canônicos aplicáveis.

## Limites permanentes transportados

- A IA não é fonte da verdade operacional.
- Nenhuma saída da IA pode disparar ação física automaticamente.
- Preservar zero retry automático em chamadas reais.
- Não alterar frequência, modo MOCK, modelo ou comportamento funcional sem Gate e autorização humana aplicáveis.
- Não usar autorização registrada em sessão ou handoff anterior como autorização atual.
- Não misturar Health Check com CSMR, Recovery 4G, Gestão do Carro, Lavadora, Heartbeat ou SmallTV.
- Alterações no Node-RED exigem snapshot/hash e devem evitar substituição integral indiscriminada de flows.

## Pendências e dívidas fora deste handoff

O handoff de origem não deixou pendência funcional aberta para concluir a entrada em operação normal. Questões posteriores ou estruturais devem permanecer nas fontes adequadas, incluindo:

- lógica principal do Node-RED ainda não versionada no Git;
- integração semântica futura com a Timeline, se formalmente aprovada;
- reconciliação entre `main` e `feature/v20-2c-contextual-automations`, que permanece em Gate Git separado.

Este arquivo não reabre nenhuma dessas frentes.

## Próximo uso seguro

Não existe próximo passo operacional autorizado por este handoff. Se a frente Health Check for retomada:

1. consultar Constituição, Source of Truth, roadmap SOC, Gates e backlog técnico;
2. verificar Git/GitHub e runtime atuais;
3. classificar a atividade e o nível do prompt;
4. solicitar nova autorização para qualquer escrita, chamada com custo ou mudança operacional.
