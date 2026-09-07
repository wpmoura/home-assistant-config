# Regras da Central Operacional — Home Assistant

## Arquitetura Base

Os principais pilares definidos para a Central Operacional são:

- Todos os YAML em formato `package`
- Estrutura pronta para `/config/packages/`
- Raiz sempre em dicionário
- Compatibilidade moderna HA 2026
- Nada de snippets parciais
- Sempre gerar arquivo completo final
- `status_casa.yaml` como motor central/orquestrador
- Outros packages atuando como “fontes especializadas”
- Dashboard apenas consome sensores finais
- Evitar sensores redundantes e lógica duplicada
- Prioridade dinâmica/evento dominante
- Timeline operacional persistente
- Sistema contextual e semântico (TV, presença, máquina de lavar, chuva, backup, internet etc.)
- UX dinâmica:
  - só mostrar o que importa
  - esconder ruído
  - destacar incidentes reais
- Sensores com criticidade/peso
- Eventos temporários vs persistentes
- Failover e observabilidade como cidadãos de primeira classe

---

# Arquitetura Atual da Casa

## Home Assistant Principal

- Raspberry Pi 4
- HAOS/Supervisor

## Home Assistant Backup

- Raspberry Pi 5
- Docker
- MQTT isolado

## Zigbee

- SLZB-06 via rede
- Zigbee2MQTT separado

## Acesso Remoto

- Tailscale

## Infraestrutura

- Ubiquiti / UniFi

---

# Conceitos Operacionais Consolidados

## Sensores e Núcleo Operacional

- `status_casa_vXX`
- `atividade_relevante_vXX`
- `timeline_operacional_vXX`
- contexto semântico da TV LG
- detecção contextual:
  - Live TV
  - Apple TV
  - OTT
- monitoramento:
  - WAN
  - 4G
  - UPS
  - Google Backup
  - vazamento
  - portas
  - chuva
  - presença humana
  - movimento relevante

---

# Regras Operacionais Definidas

## Vazamento

- continua sendo incidente
- mas com criticidade menor no contexto atual

## Movimento

- perde prioridade quando há contexto humano ativo

## TV

- contexto da mídia é mais importante que tomada energizada

## Eventos

### Temporários

- porta abriu → expira
- movimento → expira

### Persistentes

- chuva → persistente
- internet → persistente

## Dashboard

- evitar poluição visual
- priorizar narrativa operacional
- leitura rápida estilo “central de comando”

## Notificações

- Apple Watch friendly
- mais semânticas
- menos spam
- migração gradual dos Blueprints legados para o novo barramento central

---

# Objetivo Arquitetural

A arquitetura está migrando de:

> “mundo legado de automações isoladas”

para:

> “arquitetura centralizada orientada a eventos e contexto”

---

# Filosofia da Central Operacional

A Central Operacional deve funcionar como:

- narrativa contextual da casa
- motor operacional inteligente
- sistema observável
- camada semântica sobre sensores físicos
- painel executivo/NOC residencial

Nem tudo que existe no Home Assistant deve virar narrativa operacional.

## Exemplo

### Deve aparecer

- TV ativa
- Globo HD
- Netflix
- Backup falhou
- WAN caiu
- Vazamento detectado
- Máquina lavando
- Chuva

### Não deve aparecer

- ping OK
- heartbeat
- telemetria normal
- sensor online
- ruído operacional contínuo

---

# Hierarquia Narrativa

## Prioridade

1. Incidentes críticos
2. Atividades humanas relevantes
3. Eventos temporários
4. Telemetria

---

# Filosofia Visual

## O dashboard deve ser:

- limpo
- elegante
- operacional
- contextual
- com pouca poluição visual
- estilo painel executivo/NOC

## Não deve ser:

- excessivamente colorido
- arcade/RGB
- cheio de cards competindo visualmente

## Uso correto de cores

### Cor forte apenas para:

- incidentes reais
- falhas críticas
- eventos importantes

### Contextos normais devem permanecer neutros:

- TV ligada
- Apple TV
- Globo HD
- presença
- máquina lavando

---

# Próximos Objetivos

## Evolução prevista

- governança operacional
- parametrização de criticidade/confiança
- memória operacional
- correlação de eventos
- predição operacional
- IA contextual futura

