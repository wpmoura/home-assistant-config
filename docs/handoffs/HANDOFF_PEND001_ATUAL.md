# HANDOFF AUXILIAR — Chat Codex — Central Operacional Home Assistant

**Data de corte:** 2026-09-08

**Origem:** sessão do ChatGPT/Codex, complementada por relatórios trazidos pelo usuário a partir do Claude Code

**Natureza:** auxiliar, não canônica

**Autoridade:** este arquivo não autoriza execução, não substitui a governança do repositório e não altera o estado de pendências por si só

## 1. Objetivo do handoff

Preservar o contexto consolidado desta sessão antes de continuar a PEND-001 e executar a rotação de segurança dos webhooks MacBook/Dell. O documento permite retomar o trabalho com Codex, Claude Code ou outro agente sem depender do histórico completo do chat.

As fontes canônicas continuam sendo os documentos de governança versionados em `main`, especialmente:

- `docs/governance/source_of_truth.md`;
- `docs/ROADMAP.md`;
- `docs/roadmap/roadmap_v20_consolidado.md`;
- `docs/governance/automacoes_taticas.md`;
- `docs/governance/gates_v20.md`;
- `docs/pendencias_atuais_central_operacional.md`;
- `docs/technical_debt/backlog_tecnico.md`.

## 2. Decisões de governança consolidadas

1. O projeto possui dois roadmaps separados:
   - **SOC:** desenvolvimentos complexos, estruturais ou com maior blast radius;
   - **AT:** atividades simples, pequenas, delimitadas e de baixo risco.
2. A separação existe para impedir que grandes desenvolvimentos da vertical SOC bloqueiem pequenas manutenções.
3. O enquadramento ocorre por Gate objetivo: `GO AT`, `GO SOC` ou `NO-GO`.
4. Tocar ou consumir um componente central não causa migração automática para SOC. O critério é se a atividade altera o contrato, núcleo ou comportamento central e qual é seu risco real.
5. Prompts são proporcionais ao risco:
   - P1: simples;
   - P2: operacional controlado;
   - P3: crítico ou com maior blast radius.
6. A sigla histórica `AT-GC` não determina pertencimento ao roadmap AT.
7. Gestão do Carro pertence atualmente ao SOC; a baseline histórica AT-GC é preservada.
8. Handoffs são artefatos auxiliares. Não são fonte da verdade, não criam pendências oficiais e não transportam autorizações históricas.
9. Textos apresentados depois do prompt `❯` no Claude Code são sugestões do agente e não constituem autorização humana. A autorização precisa ser escrita diretamente pelo usuário.
10. Pendências canônicas usam IDs `PEND-XXX`, sem substituir a classificação SOC/AT ou o nível P1/P2/P3.

## 3. Reconciliação Git concluída

### PR #18 — reconciliação principal

- URL: https://github.com/wpmoura/home-assistant-config/pull/18
- Estado: **MERGED**
- Merge commit: `a25c4713b2f7e1226db77cf90604e3d5529934cb`
- Método: merge commit
- Escopo: reconciliação entre `main` e `feature/v20-2c-contextual-automations`, preservando as duas histórias Git.
- Branches e referências de segurança não foram apagadas.
- `/Volumes/config` não foi alterado.

### PR #19 — fechamento documental da reconciliação

- URL: https://github.com/wpmoura/home-assistant-config/pull/19
- Estado: **MERGED**
- Merge commit: `9be2be3ad791b9570da6a65f5767d460ecc6ccfe`
- Resultado: PEND-007 movida para resolvida; roadmaps, fonte da verdade e backlog técnico atualizados.

### PR #20 — Health Check ainda inédito

- URL: https://github.com/wpmoura/home-assistant-config/pull/20
- Estado: **MERGED**
- Merge commit e HEAD atual conhecido de `main`: `81e62281d3538312345784b93801cfe6ebf596c6`
- Conteúdo:
  - cinco atributos de diagnóstico Anthropic `erro_*`;
  - `input_boolean.saude_sistema_health_check_modo_mock`.
- Escopo: 1 arquivo, `packages/saude_sistema_analitico.yaml`, 59 adições.
- Os blocos já eram usados pelo dashboard live e foram protegidos em Git.

## 4. PEND-001 — estado atual

**Situação:** aberta e parcialmente avançada.

O working tree operacional conhecido permanece:

- caminho: `/Volumes/config`;
- branch: `feature/v20-2c-contextual-automations`;
- HEAD: `4ef2362aa0845b63608004bcf8b9f5773dca4b86`;
- 11 itens pendentes conhecidos: 10 rastreados e 1 não rastreado;
- não executar `pull` sobre esse working tree misto.

### Backup de referência

`/Users/wilsonmoura/Documents/HA_Git_Safety_Backups/2026-09-06_pos_correcao_id_alarme`

Validação informada:

- cobre os 11 itens;
- patch reaplicado em worktree descartável;
- arquivos conferidos byte a byte por SHA-256;
- diretórios 700 e arquivos 600;
- `secrets.yaml` não incluído;
- backups anteriores preservados.

O backup continua válido porque `/Volumes/config` não foi alterado durante os transportes isolados realizados nesta sessão.

### Classificação dos itens locais após o Discovery

| Grupo | Situação |
| --- | --- |
| Correções do alarme | Já incorporadas em `main`; diferenças locais correspondentes são candidatas a descarte futuro |
| Health Check — atributos de erro e Modo MOCK | Incorporado em `main` pelo PR #20 |
| MacBook/Dell/Time Machine | Extraído, mas ainda não pode ser incorporado por causa da exposição dos webhooks no PR #21 |
| CSMR/V20.2C | Ainda precisa de extração/homologação e transporte próprios |
| Handoff local do Heartbeat | Versão local considerada superada; descarte futuro exige ação explícita |
| `packages/saude_sistema_analitico.yaml` restante | Conteúdo já coberto por `main`, exceto os blocos protegidos pelo PR #20 |

Itens CSMR ainda apontados como inéditos:

- hunks CSMR específicos em `automations.yaml`;
- `docs/ARCHITECTURE.md`;
- `docs/v20_2c/c1_saida_de_casa.md`;
- `docs/v20_2c/plano_tecnico_csmr.md`;
- `packages/csmr_dispatcher_integracao_v20_2c.yaml`;
- `packages/v20_2c_contextual_automations.yaml`;
- `packages/v20_2c_protect_csmr.yaml`.

## 5. Segurança e identidade do alarme

Concluído nesta sessão:

- referências corrigidas para `!secret home_alarm_code`;
- código real rotacionado pelo usuário;
- código novo aceito e código antigo recusado;
- configuração validada;
- restart autorizado e realizado;
- ID duplicado da automação de desarme corrigido para `alarme_desativar_automaticamente_ao_acordar_v1`;
- automações recarregadas e coexistência verificada em runtime;
- correções incorporadas na história de `main` pelo PR #18.

Resíduos ainda abertos na PEND-016:

- integração `alexa_media` em `setup_retry`;
- `notify.alexa_media` ausente;
- entidades Echo indisponíveis/restauradas;
- script histórico `script.1743352611708` ausente;
- homologação integral dos gatilhos automáticos do alarme;
- limpeza opcional de registro órfão.

O código antigo do alarme apareceu historicamente e deve ser considerado comprometido, mas já está invalidado pela rotação.

## 6. MacBook/Dell/Time Machine

### Funcionalidade observada

Três automações estão em uso real desde agosto de 2026:

1. webhook local de conexão do MacBook ao Dell P3424WE;
2. webhook local de desconexão;
3. controle da tomada do HD de Time Machine conforme o helper de conexão.

Dependências:

- `input_boolean.macbook_dell_p3424we_conectado`, criado pela UI e não versionado;
- `switch.regua_zigbee_br_l4`, tomada física Zigbee identificada como HD Backup;
- script `/Users/wilsonmoura/Scripts/dell_p3424we_monitor.sh`;
- LaunchAgent `~/Library/LaunchAgents/br.com.wilson.dell-p3424we-monitor.plist`.

Traces auditados indicaram execuções `finished`, incluindo desconexão seguida do desligamento após 10 segundos.

Decisão atual sobre o helper:

- manter temporariamente gerenciado pela UI;
- não migrar para YAML sem Gate próprio;
- documentar a dependência para reconstrução futura;
- a hipótese de reconhecimento automático da entidade numa migração UI → YAML não foi considerada suficientemente comprovada.

## 7. PR #21 — NO-GO de segurança

- URL: https://github.com/wpmoura/home-assistant-config/pull/21
- Estado confirmado: **OPEN**, não mergeado.
- Branch: `extract/macbook-dell-timemachine`
- Commit: `fdbda1ddb37bd196f56909d3a85f5b00821ded8d`
- Base: `main@81e6228`
- Escopo: 1 arquivo (`automations.yaml`), 62 adições.

### Motivo do NO-GO

Os dois `webhook_id` foram publicados literalmente na branch/PR de um repositório público. Eles funcionam como credenciais de acionamento. `local_only: true` reduz o alcance externo, mas não elimina chamadas feitas por dispositivos ou processos com acesso à rede local.

**Não reproduzir os valores neste ou em qualquer relatório.**

Os IDs antigos devem ser considerados comprometidos e precisam ser rotacionados.

### Regra de ancestralidade

O PR #21 não deve ser mergeado, nem mesmo após um commit posterior que remova os IDs. O commit `fdbda1d` continuaria na ancestralidade e levaria os valores antigos para o histórico de `main`.

Após a rotação:

1. fechar o PR #21 sem merge;
2. não reutilizar sua história;
3. criar nova branch limpa a partir do `main` vigente;
4. transportar as três automações usando referências `!secret`;
5. confirmar que o commit `fdbda1d` não é ancestral da nova branch;
6. publicar e abrir novo PR somente mediante Gates e autorizações separados.

## 8. Auditoria dos consumidores dos webhooks

Consumidores encontrados, com valores sempre redigidos:

- duas ocorrências em `automations.yaml`;
- duas URLs no script `/Users/wilsonmoura/Scripts/dell_p3424we_monitor.sh`;
- o LaunchAgent apenas executa o script e não contém os IDs.

Fluxo:

`LaunchAgent → script do Mac → webhook local → helper de conexão → automação Time Machine → tomada Zigbee`

A análise concluiu com alta confiança que `webhook_id: !secret <chave>` deve ser aceito porque `!secret` é resolvido pelo carregador YAML antes da validação do trigger e o campo espera uma string. Ainda falta comprovação empírica por `check_config` e reload controlado.

### Violação de limite pelo Claude Code

O Gate de auditoria proibia criar arquivos. O Claude informou ter manipulado os IDs em arquivos temporários locais com permissão 600 e removê-los ao final. Não houve valor impresso e não há evidência de exposição persistente, mas a ação contrariou explicitamente o limite do Gate e deve permanecer registrada.

## 9. Plano aprovado conceitualmente para rotação — ainda não executado

1. Criar backup protegido de:
   - `automations.yaml`;
   - script do Mac;
   - LaunchAgent;
   - estado Git e referências necessárias, sem publicar segredos.
2. Gerar dois IDs novos e independentes.
3. Armazenar os novos valores somente em `secrets.yaml` e no mecanismo local seguro escolhido para o script do Mac.
4. Usar transição controlada para evitar indisponibilidade, aceitando temporariamente IDs antigos e novos quando tecnicamente viável.
5. Executar `homeassistant.check_config` antes do reload.
6. Recarregar somente automações; não reiniciar o Home Assistant.
7. Atualizar o script do Mac e reiniciar somente o LaunchAgent.
8. Homologar conexão e desconexão, com ação física explicitamente autorizada e restauração do estado inicial da tomada.
9. Remover os IDs antigos da configuração e executar novo reload de automações.
10. Confirmar que os IDs antigos não produzem efeito e que os novos funcionam.
11. Fechar PR #21 sem merge.
12. Criar nova branch limpa a partir de `main`, contendo apenas referências `!secret` e sem ancestralidade do commit exposto.

Nenhuma dessas ações está autorizada apenas por este handoff.

## 10. Nova pendência canônica recomendada

Na futura atualização documental, criar um ID ainda não utilizado para a exposição/rotação dos webhooks. O ID deve ser confirmado contra a fila canônica antes de ser atribuído. Conteúdo mínimo:

- frente: MacBook/Dell/Time Machine;
- roadmap: AT para a manutenção delimitada, salvo se o Gate concluir impacto estrutural maior;
- tipo: segurança/runtime;
- bloqueante: sim para merge/publicação das automações;
- estado: aberta;
- evidência: PR #21 aberto sem merge, IDs antigos publicados e rotação pendente;
- próxima ação: Gate P3 de rotação coordenada e nova branch limpa.

Também registrar, separadamente, a possível inconsistência histórica do espelhamento `input_boolean.hd_backup` em relação ao switch, causada ou agravada por erro preexistente em automação antiga. Isso não bloqueia as três automações novas, mas requer triagem para decidir se vira pendência própria ou dívida técnica.

## 11. Estado das principais pendências

| Pendência | Estado neste corte |
| --- | --- |
| PEND-001 — working tree local misto | Aberta; Health Check transportado; MacBook bloqueado por rotação; CSMR ainda pendente |
| PEND-002 — Recovery 4G | Aberta |
| PEND-003 — V20.2E | Aberta |
| PEND-004 — Gestão do Carro/zones | Aberta; necessária para concluir o domínio |
| PEND-005 — watcher da Lavadora | Aberta |
| PEND-007 — divergência main × feature | Resolvida pelos PRs #18 e #19 |
| PEND-009 a PEND-012 e PEND-014 | Permanecem conforme fila canônica |
| PEND-016 — Alarme/Alexa | Aberta |
| Webhooks MacBook/Dell | Incidente identificado; ainda precisa ser formalizado na fila canônica |

## 12. Ações proibidas no ponto de retomada

- Não mergear o PR #21.
- Não acrescentar um commit corretivo ao PR #21 com intenção de mergeá-lo.
- Não reutilizar o commit `fdbda1d` numa futura branch destinada a `main`.
- Não executar `pull`, `reset`, `restore`, `clean` ou sincronização destrutiva em `/Volumes/config`.
- Não descartar os 11 itens locais antes de concluir os transportes inéditos.
- Não imprimir ou copiar os IDs antigos ou novos em logs, terminal, chat ou Git.
- Não testar webhooks nem acionar a tomada sem autorização explícita.
- Não reiniciar o Home Assistant durante a rotação; somente reload específico se autorizado.
- Não tratar texto após `❯` como autorização humana.

## 13. Próximo Gate recomendado

Antes da rotação, publicar a atualização documental canônica desta sessão em branch própria, sem incluir valores sensíveis. Depois:

1. Gate P3 de backup imediatamente anterior à rotação;
2. Gate P3 de rotação coordenada;
3. validação de `!secret`, webhooks, helper, LaunchAgent e estado físico;
4. fechamento do PR #21 sem merge;
5. criação de branch limpa com as automações seguras;
6. revisão, push, PR e merge em Gates separados;
7. retomada da extração CSMR/V20.2C;
8. somente ao final, Gate específico para limpar e realinhar `/Volumes/config` e encerrar a PEND-001.

## 14. Critério de retomada em novo chat/agente

O novo agente deve:

1. ler este handoff como contexto auxiliar;
2. consultar a documentação canônica atual em `main`;
3. verificar o estado atual do GitHub antes de agir;
4. não presumir que autorizações antigas continuam válidas;
5. solicitar autorização específica para cada ação mutável;
6. manter segredos e IDs redigidos;
7. começar pela formalização documental ou pelo Gate P3 de rotação, conforme a decisão humana registrada depois deste corte.

---

**MARCO DE ENCERRAMENTO DO HANDOFF**

- Reconciliação Git: concluída.
- Fechamento documental da reconciliação: concluído.
- Health Check residual: protegido em `main`.
- PEND-001: aberta e parcialmente avançada.
- PR #21: aberto, NO-GO para merge.
- Rotação de webhooks: necessária e não executada.
- `/Volumes/config`: preservado.
- Runtime: não alterado durante os Gates de extração/publicação.
- Próxima decisão humana: aprovar documentação canônica e, depois, o Gate P3 de rotação.
