# Automações Táticas (Vertical B)

Data: 2026-08-20
Status: ATIVO

## 1. Finalidade do registro

Este documento é o registro canônico da Vertical B — Automações Táticas (AT) da Central Operacional. Ele existe para dar rastreabilidade mínima a automações locais de baixo risco, sem reproduzir o aparato de Gates da Vertical A — Sistema Operacional Casa (SOC), definido em `docs/governance/gates_v20.md`.

Este documento é subordinado à Constituição (`docs/governance/constituicao_central_operacional_v20.md`) e ao Source of Truth (`docs/governance/source_of_truth.md`), conforme reconhecido em ambos.

## 2. Vertical B — Automações Táticas (resumo)

Automações Táticas cobrem conforto, iluminação, notificações, rotinas, pequenas integrações, automações locais, experimentos controlados e melhorias operacionais.

Não incluem arquitetura, componentes compartilhados, dashboards centrais, timeline, engines, observabilidade, segurança, governança, infraestrutura estrutural, integrações centrais ou roadmap estratégico — esses permanecem na Vertical A — Sistema Operacional Casa (SOC), governada pelo fluxo constitucional completo (Constituição → Source of Truth → AGENTS → Architecture → Roadmap → Implementação → Homologação → Auditoria).

## 3. Fluxo reduzido

Requisito → Projeto → Implementação → Teste → Documentação resumida (este registro).

Não há Gates numerados para AT. O encerramento de uma AT usa os critérios mínimos da seção 6.

## 4. Critérios de escalonamento para SOC

Uma AT migra automaticamente para o SOC quando:

- altera componentes centrais;
- altera arquitetura;
- modifica dashboards globais;
- cria engines novas;
- cria novos contratos entre módulos;
- modifica a Timeline;
- altera o Executor;
- altera o Scheduler;
- altera o Recovery;
- altera infraestrutura estrutural do sistema;
- altera segurança.

**Esclarecimento**: utilizar ou acionar uma entidade ou infraestrutura residencial já existente (por exemplo, ligar/desligar um dispositivo Zigbee já cadastrado, como `switch.regua_zigbee_br_l4`) não constitui, por si só, alteração estrutural de infraestrutura para fins deste critério. Controle operacional de um dispositivo existente é diferente de alteração estrutural da infraestrutura do sistema (rede, coordenador Zigbee, add-ons, integrações centrais, backend do Home Assistant).

## 5. Convenção AT-nnn

Cada Automação Tática recebe um identificador sequencial `AT-nnn`, começando em `AT-001`. O identificador é atribuído na ordem de registro neste documento e não é reutilizado.

## 6. Critérios mínimos de encerramento de uma AT

Uma AT pode ser marcada como concluída/homologada quando:

- requisito definido;
- desenho/projeto identificado;
- implementação concluída;
- teste executado;
- documentação resumida atualizada (este registro);
- nenhuma condição de escalonamento para SOC (seção 4) identificada.

## 7. Inventário resumido

| ID | Nome | Status | Pendência não bloqueante |
| --- | --- | --- | --- |
| AT-001 | Controle automático de energia da dock Time Machine pela conexão Dell P3424WE | HOMOLOGADA | Validação com carga real após instalação da dock/HD |

## 8. Registros resumidos das ATs

### AT-001 — Controle automático de energia da dock Time Machine pela conexão Dell P3424WE

- **ID**: AT-001
- **Nome**: Controle automático de energia da dock Time Machine pela conexão Dell P3424WE
- **Objetivo**: Energizar automaticamente a tomada destinada à dock/HD do Time Machine quando o MacBook estiver conectado via USB-C ao Dell P3424WE, e desligá-la após a desconexão.
- **Requisito**: evitar consumo permanente da régua/tomada quando o Dell P3424WE (e periféricos USB-C acoplados) não estiver conectado ao MacBook Pro.
- **Desenho/fluxo**:
  - Dell conectado → detector macOS → webhook ON → helper ON → tomada ON
  - Dell desconectado → detector macOS → webhook OFF → helper OFF → 10 segundos → tomada OFF
- **Componentes**:
  - macOS LaunchAgent: `br.com.wilson.dell-p3424we-monitor`
  - Script: `~/Scripts/dell_p3424we_monitor.sh`
  - Detecção: `system_profiler SPDisplaysDataType` / DELL P3424WE
  - Dois webhooks locais HA: ON/OFF
  - Helper: `input_boolean.macbook_dell_p3424we_conectado`
  - Tomada: `switch.regua_zigbee_br_l4`
- **Comportamento**: Dell conectado → helper ON → tomada ON; Dell desconectado → helper OFF → delay 10 s → tomada OFF.
- **Testes executados/homologados**:
  - detecção Dell conectado
  - detecção Dell desconectado
  - comportamento com display sleep
  - webhook ON
  - webhook OFF
  - helper ON/OFF
  - execução automática via launchd
  - ausência de publicação periódica redundante após correção
  - tomada ON automática
  - tomada OFF automática após 10 s
  - ciclo ON/OFF end-to-end sem intervenção no Terminal
- **Status**: HOMOLOGADA
- **Regra operacional**: o usuário deve ejetar manualmente o volume do Time Machine no macOS antes de desconectar fisicamente o USB-C do Dell.
- **Pendências**: validação operacional com carga real após instalação da dock e HD do Time Machine em `switch.regua_zigbee_br_l4` (não bloqueante).
- **Segurança**: UUIDs dos webhooks não são registrados neste documento; webhooks configurados como `local_only`; nenhum segredo consta nesta documentação.
