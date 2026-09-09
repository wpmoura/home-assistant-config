# Despacho — PEND-017 — Rotação e Fechamento dos Webhooks MacBook/Dell P3424WE (2026-09-08/09)

## Objetivo

Encerrar a PEND-017: rotação coordenada dos dois `webhook_id` da integração
MacBook → Dell P3424WE → Home Assistant → Time Machine, expostos no histórico
do PR #21, e reconstrução da implementação em histórico Git limpo.

## Contexto

Fluxo operacional:

`LaunchAgent → script dell_p3424we_monitor.sh → webhook HA →
input_boolean.macbook_dell_p3424we_conectado → automação Time Machine →
switch.regua_zigbee_br_l4`

O PR #21 (`extract/macbook-dell-timemachine`, base `main`) continha os dois
`webhook_id` originais em texto plano em `automations.yaml`. Ambos foram
considerados comprometidos assim que o histórico do PR se tornou público.

## Sequência de Gates executada

1. **Pré-voo (read-only)** — auditoria completa de Git, HA, `secrets.yaml`,
   script/LaunchAgent do Mac e backups existentes. Nenhum rastro documental
   prévio da PEND-017 foi encontrado (lacuna declarada, não pressuposta).
   Veredito: `READY FOR PEND-017 EXECUTION GATE`.
2. **Execução da rotação** —
   - Backup dedicado criado fora do repositório (diretório `700`, arquivos
     `600`, manifesto SHA-256), cobrindo `automations.yaml`, `secrets.yaml`,
     o script do Mac e o LaunchAgent.
   - Dois novos `webhook_id` gerados, armazenados exclusivamente em
     `secrets.yaml` (chaves `macbook_dell_webhook_connected` /
     `macbook_dell_webhook_disconnected`) e no macOS Keychain (entradas
     dedicadas, acessíveis sem prompt ao processo do LaunchAgent).
   - As duas automações passaram a usar `webhook_id: !secret <chave>`.
   - `homeassistant.check_config` → **PASS** (sem `persistent_notification.invalid_config`,
     sem erro novo em log, sem repair novo).
   - `automation.reload` → **PASS** (as duas automações + a automação Time
     Machine recarregadas, `state: on`, `id` preservado).
   - Script do Mac reescrito para buscar os IDs no Keychain em runtime
     (`security find-generic-password`); zero literais no arquivo.
   - LaunchAgent reiniciado isoladamente (`launchctl kickstart`) — sem tocar
     HA, sem restart de sistema.
3. **Homologação física (autorização humana explícita, transições reais)** —
   - **OFF real:** desconexão física do Dell → script publica o novo webhook
     → automação dispara → helper `off` → automação Time Machine dispara
     (debounce de 10s) → `switch.regua_zigbee_br_l4` `off`. Cadeia causal
     confirmada por `context.parent_id`. **PASS.**
   - **ON real:** reconexão física → mesma cadeia, sem debounce, latência
     total ~0,4s até a tomada. **PASS.**
   - **Teste negativo dos dois IDs antigos:** chamados diretamente; HTTP 200
     em ambos (comportamento padrão do endpoint de webhook do HA, não é
     evidência de sucesso). Evidência decisiva: `last_triggered` de ambas as
     automações, estado do helper e da tomada permaneceram **idênticos**
     antes/depois — nenhum efeito produzido pelos IDs antigos. **PASS.**
   - **Estado final restaurado organicamente** (sem alteração manual de
     helper/tomada): Dell conectado, helper `on`, tomada `on`.
4. **Fechamento** —
   - PR #21 fechado **sem merge** (`state: CLOSED`, `mergedAt: null`),
     comentário sem menção a IDs.
   - Nova branch `feature/macbook-dell-timemachine-clean` criada a partir do
     `main` remoto vigente (SHA `81e62281d3538312345784b93801cfe6ebf596c6`
     à época), em clone isolado fora de `/Volumes/config`. Prova formal de
     que o commit exposto do PR #21 **não é ancestral** da nova branch.
   - Transporte seletivo: apenas o bloco das 3 automações (com `!secret`,
     sem nenhum literal) copiado de `automations.yaml`; nada mais do PR
     original foi reaproveitado (nenhum cherry-pick).
   - Auditoria de segurança no diff da branch limpa: `git diff --check`
     PASS; zero padrões de UUID; zero `/api/webhook/` literal; zero menção
     a CSMR; zero `secrets.yaml`; verificação interna (sem impressão)
     confirmou que nenhum dos quatro valores (2 IDs antigos + 2 novos)
     aparece em qualquer lugar do diff. Commit local único criado
     (`a3d86014ce68dee9b6511cc218cfd5e65e17fbe9`).
5. **Publicação e merge** —
   - Branch `feature/macbook-dell-timemachine-clean` publicada em `origin`
     (SHA remoto idêntico ao local). PR #22
     (`feat(macbook): secure Dell webhook integration`) aberto contra
     `main`, sem menção a IDs/URLs no corpo. Auditoria de segredos repetida
     no diff hospedado pelo GitHub: zero UUID, zero `/api/webhook/`
     literal, zero dos 4 valores reais.
   - PR #22 **mergeado** em `main` (squash), merge commit
     `248ec78bc1ad05f414c7f5e330d87a64bc6ad529`. Único arquivo incorporado:
     `automations.yaml` (+62/-0). PR #21 permaneceu `CLOSED`/`mergedAt: null`,
     inalterado pelo merge do #22.

## Segurança

Em nenhum momento, em nenhum dos Gates desta frente, um `webhook_id` (antigo
ou novo), uma URL completa de webhook, o conteúdo de `secrets.yaml` ou um
valor do Keychain foi impresso em relatório, documentação, commit ou log.

## Estado ao final desta frente

- PEND-017 = **encerrada, homologada e publicada em `main`**.
- PR #21 = fechado sem merge.
- PR #22 = **mergeado** em `main` (merge commit `248ec78bc1ad05f414c7f5e330d87a64bc6ad529`).
- Implementação segura (`!secret` + Keychain) ativa em produção no HA e
  presente em `main` do repositório.
- Nenhuma pendência remanescente desta frente.

## Referência cruzada

- `docs/governance/gates_v20.md` — log resumido dos Gates P3 desta frente.
- `AGENTS.md` — bullet correspondente em "Estado atual".
