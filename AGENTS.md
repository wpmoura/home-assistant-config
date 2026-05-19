# Contexto Operacional - Central Operacional Home Assistant

## Objetivo

Registrar o estado atual da Central Operacional como fonte única de verdade.

## Estado atual

- V20.0 = concluída e congelada
- V20.1A = implementada
- V20.1B = camada oficial de produção
- V20.1C = auditoria de legado planejada
- V20.2 = parcialmente implementada em shadow mode/paralelo
- V21+ = planejamento futuro

## Regras ativas

- Nunca alterar `sensor.status_casa`
- Não alterar aliases finais sem validação formal
- Dashboards produtivos não consomem `_v20_2`
- V20.2 permanece isolada em shadow mode
- IA é opcional; IA desligada mantém o sistema funcional
- Não substituir automações legadas sem auditoria V20.1C
- Packages novos devem permitir rollback simples
- Radar de Movimento é recurso sob demanda
- Alertas contextuais futuros devem ser assistivos e desacoplados

> Detalhes operacionais e histórico de decisões estão em `docs/ROADMAP.md`, `docs/architecture.md`, `docs/backlog.md` e `docs/CHANGELOG.md`.
