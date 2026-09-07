# Plano de Execução V20.2 - Fase 1A

## Objetivo

Transformar o documento de testes em rotina operacional executável para validar a V20.2 em modo shadow sem alterar produção.

## Regras gerais

- Não alterar YAML
- Não alterar dashboards
- Não alterar automações
- Não alterar aliases finais
- Não remover legado
- Não reativar V19
- Registrar evidência em `docs/execucao_testes_reais_v20_2_fase_1a.md`

## Estrutura operacional

A execução é organizada em quatro fases sequenciais:

- Fase A: baseline, regressão e sensores básicos
- Fase B: contexto humano, chuva, portas e presença
- Fase C: eventos compostos
- Fase D: robustez, contradições e flapping

## Resumo executivo

- Esforço total estimado: 2 a 3 dias de execução, dependendo da disponibilidade de incidentes físicos reais
- Quantidade de testes: 33 testes principais (17 U + 8 I + 5 R + 5 O)
- Risco geral: médio, com foco em validar isolamento da camada shadow e evitar impacto em produção
- Bloqueadores: ausência de evidências reais, indisponibilidade de cenários físicos, necessidade de suporte do ambiente operador

---

## Fase A — Baseline, regressão e sensores básicos

Duração estimada: 6 a 8 horas
Risco: baixo a médio
Dependências:
- `docs/execucao_testes_reais_v20_2_fase_1a.md`
- Sensores V20.2 carregados
- Ambiente normalizado sem incidentes ativos
Pré-condições:
- Home Assistant com V20.2 carregado em shadow mode
- Legado ativo e V19 desativado
- Dashboards e aliases finais intactos
Critério de saída:
- Baseline validada sem evidências de alteração de produção
- Regressões de V19 e `sensor.status_casa` confirmadas como não afetadas
- Sensores básicos de energia, internet, backup e TV respondendo como esperado

Testes incluídos:
- U-001: baseline casa normal
- R-001: V19 permanece desativada
- R-002: `sensor.status_casa` isolado
- R-003: timeline/feed não recebem eventos shadow
- R-004: aliases finais permanecem inalterados
- R-005: automação legada continua ativa
- U-006: energia normal
- U-007: falta de energia
- U-008: retorno de energia
- U-009: internet normal
- U-010: degradação de internet
- U-011: retorno de internet
- U-012: backup Google normal
- U-013: backup Google falha
- U-016: TV ligada
- U-017: TV desligada

## Fila de execução imediata

A fila abaixo prioriza a ordem prática para iniciar a Fase A com foco em proteção de produção e validação básica de sensores.

| Ordem | ID | Duração estimada | Dependências | Risco |
|---|---|---|---|---|
| 1 | U-001 | 30 min | Sensores V20.2 carregados | Baixo |
| 2 | R-001 | 15 min | U-001 | Baixo |
| 3 | R-002 | 15 min | U-001 | Baixo |
| 4 | R-003 | 20 min | U-001 | Baixo |
| 5 | R-004 | 20 min | R-003 | Baixo |
| 6 | R-005 | 20 min | U-001 e legado ativo | Baixo |
| 7 | U-006 | 30 min | Fase A iniciada | Baixo |
| 8 | U-007 | 45 min | U-006 | Médio |
| 9 | U-008 | 45 min | U-007 | Médio |
| 10 | U-009 | 30 min | Fase A iniciada | Baixo |
| 11 | U-010 | 45 min | U-009 | Médio |
| 12 | U-011 | 45 min | U-010 | Médio |
| 13 | U-012 | 30 min | Fase A iniciada | Baixo |
| 14 | U-013 | 45 min | U-012 | Médio |
| 15 | U-016 | 30 min | Fase A iniciada | Baixo |
| 16 | U-017 | 30 min | U-016 | Baixo |

---

## Fase B — Contexto humano, chuva, portas e presença

Duração estimada: 8 a 10 horas
Risco: médio
Dependências:
- Conforto de teste com presença humana disponível
- Acesso seguro a portas e janelas
- Sensores de chuva e temperatura funcionando
Pré-condições:
- Fase A concluída e critérios atendidos
- Sensores de presença e contexto humano operacionais
Critério de saída:
- Regras de contexto humano e chuva validadas
- Portas e presença respondendo sem gerar eventos críticos indevidos
- Coerência entre fontes físicas e sensores V20.2

Testes incluídos:
- U-002: porta da sala aberta
- U-003: porta da sala fechada
- U-004: chuva fraca iniciada
- U-005: chuva forte
- U-014: presença/movimento
- U-015: banho detectado pelo box
- I-001: porta aberta sem ninguém em casa
- I-002: porta aberta com alguém em casa
- I-003: chuva + janela aberta
- I-004: chuva + porta aberta

---

## Fase C — Eventos compostos

Duração estimada: 4 a 6 horas
Risco: médio a alto
Dependências:
- Cenários compostos físicos disponíveis ou simuláveis de forma segura
- Acesso a rede e energia para validar eventos simultâneos
Pré-condições:
- Fase B concluída com saídas documentadas
- Sensores de contexto ambientais ativos
Critério de saída:
- Eventos compostos validados sem romper a camada shadow
- Relevância e confiança comportando-se conforme regras definidas

Testes incluídos:
- I-005: energia ausente + internet offline
- I-006: backup falhando durante outro incidente
- I-007: internet degradada + contexto noturno
- I-008: energia ausente com alguém em casa

---

## Fase D — Robustez, contradições e flapping

Duração estimada: 4 a 6 horas
Risco: médio
Dependências:
- Cenários de flapping ou falhas intermitentes observáveis
- Métricas de `fontes_invalidas`, `fontes_contraditorias` e `contradicao_detectada`
Pré-condições:
- Fase C concluída com critérios atendidos
- Sensores de qualidade de dado operacionais
Critério de saída:
- Robustez validada contra fontes inválidas e contradições
- Flapping tratado como domínio oscilante, não como estado estável

Testes incluídos:
- O-001: fonte inválida
- O-002: contradição casa vazia/presença
- O-003: relevância sem evento
- O-004: chuva ativa com contexto ambiental normal
- O-005: internet flapping

---

## Resumo executivo de saída

Total de testes: 33
Esforço total estimado: 2 a 3 dias
Risco geral: médio
Bloqueadores esperados:
- indisponibilidade de cenários físicos reais
- falta de evidência visual de sensores V20.2
- necessidade de validação humana para presença/porta/janela

> Este plano deve ser usado em conjunto com `docs/execucao_testes_reais_v20_2_fase_1a.md` para registro de evidências e status de cada teste.
