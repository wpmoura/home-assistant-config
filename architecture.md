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
- V20.1C possui `V20.1C_FECHAMENTO` registrado: diagnóstico e governança concluídos, com decommission bloqueado.
- V20.1N está homologada como checkpoint de estabilização operacional.
- V20.1O está congelada como política produtiva de Timeline / Push / Agregação.
- V20.1Q implementa o Recovery 4G pelo princípio “A Central decide, o Executor atua, a Central valida e a Central encerra”; a homologação operacional do pacote corretivo permanece pendente.
- V20.2 opera em shadow mode e não deve alterar dashboards produtivos. A única exceção arquitetural formal é a promoção limitada do Coordenador da Sessão de Monitoramento Remoto (CSMR) da V20.2C, ainda sem implementação autorizada até aprovação de seu Gate específico.
- O CSMR é classificado como motor oficial de coordenação operacional, com escopo restrito à saída e ao retorno de Wilson; essa classificação não promove o restante da V20.2.
- V20.1O permanece como motor oficial da Timeline, do Event Feed e da política de publicação. O CSMR não mantém histórico, não deduplica em paralelo e não escreve diretamente nos aliases finais.
- V20.2E amplia de forma estritamente aditiva o publicador canônico para `source: carro_presenca` e eventos `car_use_started`/`car_use_ended`; os contratos e quatro eventos do CSMR permanecem inalterados.
- IA é opcional; quando desligada, o sistema deve permanecer 100% funcional.
- `sensor.status_casa` não deve ser alterado por implementações experimentais.
- Aliases finais só podem ser alterados com validação formal.

Decisão arquitetural subordinada:

- `docs/arquitetura/despacho_arquitetural_v20_1q.md` — Recovery Automático do Modem 4G com tentativas genéricas, retorno estabilizado e cooldown somente por esgotamento.
- `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md` — promoção limitada do CSMR, condicionada ao Gate e sem autorização de implementação ou publicação em runtime.
