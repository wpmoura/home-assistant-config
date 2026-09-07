# 🧠 Central Operacional da Casa
## Arquitetura Semântica, Motor Contextual e Roadmap Evolutivo
### Home Assistant — Sistema Operacional Residencial Contextual

---

## 📌 Objetivo do Documento

Este documento consolida:

- arquitetura lógica da Central Operacional
- fundamentos conceituais
- regras semânticas
- modelo de priorização
- motores contextuais
- evolução do sistema
- roadmap estratégico
- visão futura da plataforma

O objetivo é transformar o ambiente Home Assistant em um:

## 🧠 Sistema Operacional Residencial Contextual

capaz de:

- interpretar eventos
- entender contexto humano
- reduzir ruído operacional
- gerar narrativas inteligentes
- priorizar situações
- evoluir para IA contextual

---

## 🏗️ Visão Arquitetural

A arquitetura foi concebida em múltiplas camadas semânticas.

---

## 1. Camada Física

Representa o mundo real.

### Dispositivos

- sensores Zigbee
- sensores BLE
- sensores Wi-Fi
- tomadas inteligentes
- LG WebOS
- Aqara FP2
- dispositivos Tuya
- dispositivos UniFi
- UPS
- modem 4G
- roteadores
- smart TVs
- eletrodomésticos
- sensores ambientais

---

## 2. Camada de Telemetria

Responsável pela aquisição de estados brutos.

### Exemplos

| Entidade | Informação |
|---|---|
| binary_sensor | on/off |
| sensor.power | potência |
| sensor.temperature | temperatura |
| sensor.presence | presença |
| media_player | mídia |
| device_tracker | localização |

---

## 3. Camada Semântica

Transforma sinais físicos em significado operacional.

### Exemplo — TV

#### Entrada física

| Sensor | Valor |
|---|---|
| tomada TV | energizada |
| LG WebOS | ligada |
| media_title | Globo HD |

#### Interpretação semântica

```text
📺 TV ativa assistindo Globo HD
```

### Exemplo — Máquina de lavar

#### Entrada física

| Sensor | Valor |
|---|---|
| potência | 18W |
| variação cíclica | presente |

#### Interpretação semântica

```text
🧺 Máquina lavando roupas
```

### Exemplo — Box banheiro

#### Entrada física

| Sensor | Valor |
|---|---|
| presença FP2 | ocupada |

#### Interpretação semântica

```text
🚿 Alguém tomando banho
```

---

## 4. Camada Contextual

A camada contextual é o núcleo da Central Operacional.

Ela consolida múltiplos eventos em uma interpretação única da casa.

### Sensores Contextuais Principais

| Sensor | Objetivo |
|---|---|
| `sensor.status_casa` | estado principal |
| `sensor.casa_narrativa` | narrativa operacional |
| `sensor.casa_evento_dominante` | evento prioritário |
| `sensor.casa_modo_operacional` | modo contextual |
| `sensor.casa_timeline_operacional` | histórico semântico |
| `sensor.casa_contexto_humano` | interpretação humana |
| `sensor.casa_atividade_relevante` | atividade principal |
| `binary_sensor.casa_incidente_ativo` | incidente crítico |

---

## 🧠 Filosofia da Plataforma

O sistema deixou de ser:

```text
automações independentes
```

e passou a ser:

```text
um motor contextual unificado
```

A arquitetura busca:

- reduzir ruído
- eliminar redundância
- consolidar eventos
- gerar interpretação contextual
- aproximar comportamento humano

---

## ⚙️ Modelo Operacional

### Eventos

Tudo na casa passa a ser tratado como evento.

#### Exemplos

- porta abriu
- chuva iniciou
- TV ligada
- presença detectada
- UPS descarregando
- internet caiu
- vazamento detectado

---

## Estrutura de Evento

Cada evento possui atributos.

| atributo | significado |
|---|---|
| criticidade | impacto |
| confiança | certeza |
| persistência | duração |
| categoria | tipo |
| contexto | relevância contextual |
| prioridade | peso final |
| temporalidade | TTL |
| origem | sensor fonte |

---

## 🧮 Motor de Prioridade

A casa decide:

> qual evento é mais importante neste momento?

### Exemplo

| Evento | Peso |
|---|---|
| Vazamento | 95 |
| Failover WAN | 80 |
| Porta abriu | 35 |
| Movimento corredor | 10 |

Resultado:

```text
🚨 Vazamento detectado
```

---

## Critérios de Priorização

O score final considera:

- criticidade
- contexto humano
- horário
- persistência
- relevância operacional
- presença humana
- modo da casa

---

## 🧍 Contexto Humano

Eventos mudam de importância conforme comportamento humano.

### Exemplos

| Situação | Peso |
|---|---|
| porta abriu com casa vazia | alto |
| porta abriu com moradores acordados | médio |
| movimento noturno sem ninguém | crítico |
| TV ligada à noite | entretenimento |
| banho detectado | ocupação privada |

### Contextos Reconhecidos

- alguém em casa
- casa vazia
- dormindo
- entretenimento
- banho
- rotina ativa
- madrugada
- ausência prolongada
- atividade operacional

---

## 🕒 Eventos Temporários

Eventos possuem duração contextual.

### Exemplos

| Evento | duração |
|---|---|
| porta abriu | 2 min |
| chuva | enquanto durar |
| TV ligada | contínuo |
| banho | enquanto presença ativa |
| microondas | instantâneo |

---

## 🚫 Sistema Anti Flood

Objetivo: evitar ruído operacional.

### Evita

- spam visual
- spam de notificações
- repetição excessiva
- eventos redundantes

### Estratégias

- cooldown
- supressão temporal
- agrupamento de eventos
- debounce semântico
- consolidação contextual

---

## 🏠 Modos Operacionais da Casa

A casa opera em estados contextuais.

| modo | significado |
|---|---|
| tranquilo | operação normal |
| entretenimento | mídia ativa |
| atividade | rotina ativa |
| atenção | evento relevante |
| emergência | incidente crítico |
| failover | operação degradada |

---

## 🧠 Narrativa Operacional

A narrativa operacional traduz eventos técnicos em linguagem humana.

### Exemplo técnico

```text
binary_sensor.internet = off
```

### Narrativa operacional

```text
⚠️ Internet principal indisponível — Failover 4G ativo
```

---

## 📜 Timeline Operacional

A timeline virou memória operacional semântica.

### Exemplo

```text
🌧️ Chovendo
🚪 Porta aberta
📺 Globo HD
⚠️ Failover WAN
```

### Benefícios

- leitura humana
- análise rápida
- rastreabilidade
- contexto histórico
- depuração operacional

---

## 📺 Sistema Semântico de TV

A TV deixou de ser apenas:

```text
tomada energizada
```

e passou a representar contexto humano.

### Contextos Reconhecidos

| estado | interpretação |
|---|---|
| Live TV | canal ao vivo |
| OTT | streaming |
| HDMI | console/dispositivo |
| standby | ignorado |

### Objetivos

- detectar entretenimento real
- compor narrativa
- identificar ocupação
- enriquecer contexto humano

---

## 🌐 Sistema de Failover

A casa possui monitoramento contextual de conectividade.

### Recursos

- internet principal
- failover 4G
- UPS
- degradação operacional
- recuperação automática

### Objetivos

- alta disponibilidade
- continuidade operacional
- visibilidade contextual
- automação resiliente

---

## 🔋 Sistema Energético

O sistema monitora:

- concessionária
- UPS
- bateria
- consumo
- autonomia
- degradação

### Futuro

- previsão de autonomia
- shutdown inteligente
- priorização energética
- preservação de carga

---

## 🚿 Contexto de Banho

O sensor do box representa um avanço contextual importante.

### Benefícios

- privacidade contextual
- supressão de alertas
- ocupação humana real
- narrativa contextual

### Exemplo

```text
🚿 Banho em andamento
```

---

## 🧠 Motor de Confiança

A plataforma evolui para calcular:

> quão confiável é um evento?

### Objetivo

Evitar falsas interpretações.

### Exemplo

```text
TV ativa = 92%
```

### Critérios

| fator | exemplo |
|---|---|
| redundância | TV + WebOS |
| consenso | múltiplos sensores |
| estabilidade | bouncing |
| histórico | padrão conhecido |
| confiabilidade | Zigbee vs Tuya |

---

## 🧩 Consolidação do Legado

Hoje coexistem:

| camada | status |
|---|---|
| automações antigas | parcialmente ativas |
| central operacional | principal |
| blueprints externos | coexistindo |

### Objetivo

Migrar tudo para:

```text
pipeline contextual único
```

### Eventos a Migrar

- porta aberta
- chuva
- vazamento
- bateria
- UPS
- segurança
- sensores antigos
- alertas herdados

---

## 🎛️ Painel de Parametrização

Planejado para permitir ajustes sem edição YAML.

### Funcionalidades Futuras

- pesos
- criticidade
- thresholds
- persistência
- horários silenciosos
- relevância
- confiança

---

## 🧠 IA Operacional

### Objetivo

A casa deixar de apenas reagir e começar a interpretar.

### Narrativa Inteligente

Hoje:

```text
⚠️ Porta aberta
```

Futuro:

```text
⚠️ Porta da sala abriu com casa vazia às 02:14
```

### IA Contextual

LLM poderá inferir:

- anomalias
- padrões
- hábitos
- exceções
- previsões
- insights operacionais

### Memória Operacional

A casa poderá lembrar:

- rotinas
- padrões
- horários
- hábitos
- sequências recorrentes

### Sistema Preditivo

```text
⚠️ Geladeira consumindo acima do padrão há 3 dias
```

```text
⚠️ UPS deve durar apenas 18 minutos
```

---

## 🏗️ Roadmap Consolidado

---

## ✅ Fase 1 — Fundação Semântica

### Entregue

- status casa
- narrativa
- timeline
- TV contextual
- sensores semânticos
- failover
- anti flood
- eventos temporários

---

## ✅ Fase 2 — Central Operacional

### Entregue

- dashboards dinâmicos
- evento dominante
- contexto humano
- incidentes ativos
- narrativa operacional

---

## 🚧 Fase 3 — Context Intelligence

### Em evolução

- pesos contextuais dinâmicos
- relevância humana
- motor de confiança
- painel de parametrização
- consolidação contextual

---

## 🚧 Fase 4 — Consolidação do Legado

### Objetivo

Pipeline semântico unificado.

### Escopo

- porta aberta
- chuva
- vazamento
- bateria
- UPS
- modem 4G
- segurança
- blueprints externos
- automações antigas

---

## 🚧 Fase 5 — IA Operacional

### Objetivo

Narrativa inteligente e inferência contextual.

### Possibilidades

- análise de padrões
- detecção de anomalias
- recomendações
- explicação de eventos
- sumarização de timeline
- apoio à tomada de decisão

---

## 🚧 Fase 6 — Casa Cognitiva

### Objetivo

A casa começar a:

- prever
- interpretar
- decidir
- adaptar
- negociar prioridade
- aprender comportamento

---

## 📍 Estado Atual do Projeto

Atualmente o projeto está aproximadamente em:

```text
Fase 3 — Context Intelligence
├── contextualização
├── pesos dinâmicos
├── confiança
├── relevância humana
└── consolidação do pipeline
```

---

## 🎯 Visão Estratégica Final

O projeto deixou de ser:

```text
um dashboard Home Assistant
```

e passou a evoluir para:

## 🧠 Sistema Operacional Residencial Contextual

capaz de:

- interpretar contexto
- consolidar eventos
- reduzir ruído
- inferir relevância
- priorizar situações
- gerar narrativa operacional
- evoluir para IA contextual
- criar memória operacional
- prever comportamentos
- automatizar decisões

Roadmap/versionamento