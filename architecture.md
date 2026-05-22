# Arquitetura Operacional

O documento registra a arquitetura oficial da Central Operacional Home Assistant.

Fluxo oficial de processamento:

- Sensores físicos
- V20.1B Deterministic Event Layer
- Context Engine V20.2
- Relevance Engine V20.2
- Correlation Engine V20.2
- Dynamic Score
- Narrativa determinística
- Aliases finais futuros

Princípios principais:

- V20.1B é a camada de produção determinística.
- V20.1N está homologada como checkpoint de estabilização operacional.
- V20.1O está congelada como política produtiva de Timeline / Push / Agregação.
- V20.2 opera em shadow mode e não deve alterar dashboards produtivos.
- IA é opcional; quando desligada, o sistema deve permanecer 100% funcional.
- `sensor.status_casa` não deve ser alterado por implementações experimentais.
- Aliases finais só podem ser alterados com validação formal.
