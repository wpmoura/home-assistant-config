# Auditoria Operacional do Recovery 4G — V20.1Q

Data: 2026-07-13
Status: EVIDÊNCIA CONSOLIDADA
Classificação: auditoria histórica/evidencial, não canônica
Autoridade: subordinada à Constituição, ao Source of Truth, à arquitetura e ao Roadmap

## Finalidade

Registrar o estado observado do recovery automático do modem 4G antes da V20.1Q.

Este documento é uma fotografia operacional. Ele não autoriza implementação, limpeza do legado, alteração de automações, remoção de blueprints, edição de `.storage` ou ação física.

## Problema observado

A Central Operacional detecta corretamente a indisponibilidade do Backup 4G.

O defeito não está em:

- detecção;
- Timeline;
- Score;
- Dashboard;
- estado semântico de Internet.

O defeito está exclusivamente na execução do recovery automático do modem 4G.

## Artefatos técnicos observados

- Blueprint avançado preservado: `blueprints/automation/wmoura/wan_4G_AppleWatch_Energia.yaml`.
- Automação derivada preservada em `automations.yaml`, id `1776732366716`, alias `Gestão modem 4G`.
- Detector usado pela automação legada: `binary_sensor.rede_modem_zte`.
- Origem do detector legado: plataforma `ping`, separada da detecção homologada da Central.
- Tomada do modem: `switch.0xa4c1381045aeb344`.
- Helper de energia legado: `input_boolean.energia_falta`.
- Estado observado do helper de energia durante falhas: `unavailable`.
- Ações físicas `switch.turn_off` e `switch.turn_on` permanecem no algoritmo legado.

O blueprint simples `blueprints/automation/wmoura/wan_4G_AppleWatch.yaml` permanece fora do lote por não possuir instância ativa identificada. O saneamento desse blueprint e do motor paralelo `packages/wan_4g_engine_v20.yaml` não é autorizado por esta auditoria.

## Achados aprovados

- O blueprint legado está preservado.
- A automação existente está habilitada no estado auditado.
- A tomada correta já está identificada no código.
- O algoritmo de power cycle está preservado.
- O recovery legado possui detector próprio.
- O helper de energia anteriormente utilizado está indisponível.
- Existe split-brain entre a Central Operacional e o recovery legado.
- A Central conhece o incidente, mas o recovery usa decisão própria.
- O Executor legado pode interpretar fontes diferentes das utilizadas pela Central.
- Traços auditados mostraram a condição de energia falhando antes das ações físicas, sem execução do power cycle nesses ciclos.

## Fluxo observado

```text
Detector legado baseado em ping
        │
        ▼
Automação Gestão modem 4G
        │
        ├── condição input_boolean.energia_falta == off
        │       └── estado observado: unavailable → fluxo bloqueado
        │
        └── se liberado: tomada off → espera → tomada on → validação legada

Central Operacional
        │
        └── detecta e publica Backup 4G indisponível por cadeia própria
```

As duas cadeias não compartilham a mesma autoridade de decisão ou validação.

## Causa arquitetural

Existe separação indevida entre:

- quem detecta e interpreta o estado;
- quem executa a ação física;
- quem valida o resultado;
- quem encerra o incidente.

Essa separação permite decisões concorrentes e impede que a detecção homologada acione de forma confiável o recovery físico.

## Riscos

- Central indicar falha sem recuperação.
- Recovery agir com detector divergente.
- Tentativas não rastreáveis.
- Decisões paralelas.
- Falha silenciosa do modem 4G.
- Perda de resiliência da residência.
- Tomada permanecer sob controle de fluxos concorrentes.
- Alterações no legado eliminarem o rollback antes da homologação.

## Conclusão

O recovery deve ser reorganizado segundo o princípio:

> A Central decide.
> O Executor atua.
> A Central valida.
> A Central encerra.

A auditoria não autoriza limpeza do legado nem implementação por si só. A decisão técnica aprovada está em `docs/arquitetura/despacho_arquitetural_v20_1q.md`, e o plano subordinado está em `docs/releases/implementation_plan_v20_1q.md`.
