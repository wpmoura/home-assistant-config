# Contexto Operacional - Central Operacional Home Assistant

## Objetivo

Registrar o estado atual da Central Operacional como fonte única de verdade.

## Estado atual

- V20.0 = concluída e congelada
- V20.1A = concluída
- V20.1B lote 1 = concluída
- V20.1B lote 2 = concluída/parcial para energia, internet, failover e backup
- V20.1K = concluída; tag `V20.1K_FECHAMENTO` criada
- V20.1N = homologada; checkpoint de estabilização registrado
- V20.1O = estável com débitos aceitos; política Timeline / Push / Agregação estabilizada
- V20.2A = concluída; dashboard legado `teste-4` removido pela UI
- V20.2B = auditoria executada; nenhuma ação operacional realizada
- V20.2/V20.3/V21 = planejamento futuro
- V21+ = planejamento futuro

## Regras ativas

- Nunca alterar `sensor.status_casa`
- Não alterar aliases finais sem validação formal
- Dashboards produtivos não consomem `_v20_2`
- V20.2 permanece isolada em shadow mode
- IA é opcional; IA desligada mantém o sistema funcional
- Não substituir automações legadas sem auditoria V20.1C
- Automações órfãs/desabilitadas não devem ser removidas automaticamente; limpeza futura deve seguir criticidade
- Packages novos devem permitir rollback simples
- Radar de Movimento é recurso sob demanda
- Alertas contextuais futuros devem ser assistivos e desacoplados
- V20.1O não deve ser reaberta silenciosamente; mudanças futuras devem abrir lote formal e citar dependências do V20.1O

> Detalhes operacionais e histórico de decisões estão em `docs/ROADMAP.md`, `docs/architecture.md`, `docs/pendencias_atuais_central_operacional.md` e `CHANGELOG.md`.
