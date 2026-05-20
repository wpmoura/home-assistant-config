# 🧠 Roadmap — Central Operacional / Semantic House Platform


---

# 📜 Visão Estratégica

A Central Operacional evoluiu de um conjunto de automações isoladas para uma plataforma operacional contextual da casa.

O objetivo de longo prazo é transformar o Home Assistant em:

# Semantic Operational Context Platform

capaz de:

- interpretar contexto
- correlacionar eventos
- entender presença humana
- gerar narrativas operacionais
- inferir comportamentos
- suportar IA contextual futura
- operar com alta confiabilidade

---

# 🏗️ ERA PRÉ-SEMÂNTICA
# V0 → V16

---

# 🔹 V0 — Automação Base

## Objetivo
Automatizar dispositivos e eventos simples.

## Características

- automações isoladas
- lógica procedural
- notificações diretas
- sem contexto operacional

## Exemplos

- porta abriu → envia push
- vazamento → alerta
- chuva → notificação

---

# 🔹 V1 → V4 — Consolidação da Casa

## Objetivo
Consolidar infraestrutura da automação.

## Entradas principais

- Zigbee
- MQTT
- Tuya
- LG WebOS
- tomadas inteligentes
- sensores de presença
- sensores ambientais

---

# 🔹 V5 → V8 — Observabilidade

## Objetivo
A casa começou a “falar”.

## Entradas principais

- dashboards operacionais
- radar operacional
- monitoramento WAN
- monitoramento 4G
- UPS
- backup
- failover
- health checks

## Resultado
Nasceu a ideia de:

# “estado da casa”

---

# 🔹 V9 → V11 — Contexto Operacional

## Objetivo
Introduzir contexto humano e operacional.

## Entradas principais

- atividade relevante
- timeline operacional
- contexto humano inicial
- TV contextual
- máquina de lavar contextual
- narrativa operacional

## Exemplos

```text
📺 Assistindo Globo HD
🧺 Máquina centrifugando
🌧️ Chovendo agora
```

---

# 🔹 V12 → V14 — Semântica Experimental

## Objetivo
Primeiras tentativas de interpretação semântica.

## Entradas principais

- score operacional
- criticidade
- relevância
- confiança
- narrativa dinâmica

## Problemas descobertos

- parsing textual excessivo
- acoplamento forte
- baixa determinística

---

# 🔹 V15 → V16 — Central Operacional Moderna

## Objetivo
Modernização da UX operacional.

## Entradas principais

- dashboards fluidos
- timeline visual
- chips dinâmicos
- radar operacional
- layout estilo Netflix/Apple
- UX contextual

## Resultado
Nasceu:

# a Central Operacional moderna.

---

# ✅ V17 — Consolidação Semântica

## Status
Concluída

## Objetivo
Criar o primeiro “sistema nervoso central” da casa.

## Entradas principais

- timeline operacional
- contexto humano
- status central da casa
- consolidação de eventos
- narrativa operacional integrada

## Eventos tratados

- porta
- chuva
- push
- vazamento
- TV

## Problemas descobertos

- semântica ainda textual
- dependência entre sensores
- herança excessiva

---

# ✅ V18 — Parametrização Operacional

## Status
Concluída

## Objetivo
Transformar pesos e criticidade em parâmetros vivos.

## Entradas principais

- sliders
- helpers operacionais
- criticidade configurável
- timeout configurável
- visibilidade dinâmica
- parâmetros persistentes

## Conceitos introduzidos

- pesos operacionais
- criticidade
- confiança mínima
- timeout temporal
- governança operacional

---

# ✅ V19 — Event Engine & Timeline Semântica

## Status
Concluída / Encerrada

## Objetivo
Transformar eventos em memória operacional temporal.

## Entradas principais

- Event Engine
- feed temporal
- timeline dinâmica
- score ativo
- contexto humano
- modo operacional
- dashboards separados

## Dashboards criados

- operacional
- administrativo
- engenharia
- comparativo legado

## Funcionalidades importantes

- TV contextual
- timeline temporal
- score breakdown
- feed de eventos
- incidentes ativos
- contexto operacional dinâmico

## Problemas descobertos

- parsing textual frágil
- loops semânticos
- sensores herdados
- perda de confiança operacional
- acoplamento V17/V18/V19

---

# 🚀 V20 — Operational Core Rewrite

## Status
Próxima grande evolução

## Objetivo

# restaurar confiança operacional

---

# 🔹 V20.1 — Legacy Migration Layer

## Objetivo
Migrar sensores e eventos legados para engines determinísticas.

## Já migrados

- chuva
- porta
- TV (parcial)

## Pendentes

### Vazamento

Hoje:

```text
sensor textual legado
```

Futuro:

```text
binary_sensor.vazamento_real
```

---

### Backup Google

Hoje:

```text
parsing textual
```

Futuro:

```text
binary_sensor.backup_google_falhando
```

---

### WAN / 4G

Futuro:

```text
binary_sensor.wan_online
binary_sensor.failover_ativo
binary_sensor.modem_4g_operacional
```

---

### Máquina de lavar

Futuro:

```text
sensor.lavadora_estado_semantico
```

Estados previstos:

- lavando
- molho
- centrifugando
- finalizada

---

### Micro-ondas

Futuro:

```text
binary_sensor.microondas_em_uso
```

---

# 🔹 V20.2 — Dedicated Operational Engines

## Objetivo
Cada domínio operacional terá sua própria engine.

## Engines previstas

### Backup Engine

- retry
- timeout
- degradação
- health state
- recuperação

### TV Engine

- conteúdo
- HDMI
- app
- live TV
- estado real do media_player

### Door Engine

- abertura contextual
- timeout
- persistência

### Rain Engine

- chuva leve
- moderada
- intensa
- persistência temporal

### Leak Engine

- detecção real
- criticidade contextual

---

# 🔹 V20.3 — Operational Weight Matrix

## Objetivo
Separar:

- peso operacional
- peso contextual
- peso semântico
- peso cognitivo futuro

## Conceito

A casa passa a distinguir:

# saúde operacional

de:

# atividade humana/contextual

---

## Exemplo

| Evento | Operacional | Contextual |
|---|---|---|
| Internet crítica | 100 | 100 |
| Sem energia | 95 | 100 |
| Failover 4G | 85 | 70 |
| Vazamento | 30 | depende da presença |
| Backup Google erro | 20 | 10 |
| TV ligada | 0 | 30 |

---

## Resultado esperado

A casa pode estar:

```text
Operacionalmente saudável
```

mas:

```text
Contextualmente ativa
```

---

# 🚀 V21 — Presence Intelligence Engine

## Objetivo
A casa começa a entender pessoas.

---

# 🔹 V21.1 — Presence Core

## Entradas principais

- zonas semânticas
- contexto espacial
- chegada/saída
- ocupação contextual
- deslocamento

## Exemplos

```text
🏢 Wilson trabalhando
🚗 Wilson em deslocamento
🏠 Wilson chegando
🍿 Casa em entretenimento
```

---

# 🔹 V21.2 — Criticidade Contextual

## Objetivo
Peso muda dinamicamente conforme contexto.

## Exemplos

```text
porta abriu + casa vazia = crítica
porta abriu + morador em casa = baixa
```

```text
vazamento + ninguém em casa = alto
vazamento + usuário presente = médio
```

---

# 🔹 V21.3 — Mobility Context Engine

## Integrações previstas

- carro
- GPS
- velocidade
- distância
- zonas
- tempo fora
- deslocamento urbano

---

# 🔹 V21.4 — Human Activity Inference

## Objetivo
A casa passa a inferir atividades humanas.

## Casos previstos

| Evento | Inferência |
|---|---|
| box ativo | banho |
| TV + sofá | entretenimento |
| cama ocupada | dormindo |
| cozinha ativa | preparando refeição |
| mesa + notebook | trabalhando |

---

# 🚿 Shower Presence Engine

## Sensor estratégico

Existe um sensor de presença dentro do box do banheiro.

Isso permitirá inferir:

```text
🚿 Alguém tomando banho
```

## Impactos futuros

- evitar anúncios Alexa
- evitar apagar luz
- evitar modo dormir
- enriquecer contexto humano
- melhorar inferência contextual

---

# 🚀 V22 — Energy Intelligence

## Objetivo
A casa começa a entender consumo energético.

## Entradas principais

- consumo contextual
- consumo fantasma
- standby inteligente
- UPS intelligence
- blackout
- degradação elétrica

## Exemplos

```text
⚡ Casa em economia
🔋 Operando em UPS
🔥 Consumo anormal detectado
```

---

# 🚀 V23 — Predictive Behaviors

## Objetivo
A casa começa a prever comportamento.

## Entradas principais

- predição de presença
- detecção de anomalias
- comportamento recorrente
- rotinas adaptativas

## Exemplos

```text
Wilson normalmente chega às 19h.
Preparar ambiente.
```

---

# 🚀 V24 — Self-Healing Automations

## Objetivo
A casa começa a corrigir problemas automaticamente.

## Entradas principais

- reinício automático
- recuperação semântica
- failover automático
- restart de integrações
- remediação contextual

## Exemplos

```text
Backup falhou 3 vezes.
Reiniciar integração automaticamente.
```

---

# 🚀 V25 — Semantic Brain

## Objetivo
Introduzir inteligência contextual cognitiva.

## Entradas principais

- memória operacional
- contexto histórico
- recorrência
- causa raiz
- narrativa cognitiva

## Objetivo futuro

Preparar a plataforma para:

# LLM contextual

---

# 🚀 V26 — Multi-Agent House Intelligence

## Objetivo
Múltiplos agentes especializados.

## Agentes previstos

| Agente | Função |
|---|---|
| Segurança | portas, alarmes, presença |
| Energia | UPS, consumo |
| Entretenimento | TV, mídia |
| Mobilidade | carro, deslocamento |
| Infraestrutura | HA, rede, backup |
| Saúde da casa | estabilidade geral |

---

# 🧠 Arquitetura futura prevista

## Camada 1
Sensores reais

## Camada 2
Core operacional determinístico

## Camada 3
Contexto semântico

## Camada 4
LLM cognitivo

---

# 🤖 Papel futuro do LLM

## O LLM NÃO será:

- fonte da verdade
- executor direto
- controlador operacional

## O LLM SERÁ:

- camada cognitiva
- interpretação contextual
- inferência
- narrativa
- previsão
- análise comportamental

---

# 🧠 Exemplos futuros

```text
Wilson chegou em casa há 12 minutos.
A TV foi ligada logo em seguida.
A casa está em modo entretenimento.
```

```text
Comportamento incomum detectado:
porta aberta às 03:14 sem retorno de moradores.
```

```text
A casa está operacionalmente saudável,
mas altamente ativa.
```

---

# 🎯 Estado atual do projeto

| Área | Status |
|---|---|
| UX | alta |
| Timeline | alta |
| Contexto | média |
| Semântica | média |
| Confiabilidade operacional | baixa/média |
| Core determinístico | ainda não |
| Presence Intelligence | ainda não |
| IA contextual | futura |

---

# 🎯 Prioridade atual

## NÃO evoluir features agora.

## Primeiro:

# V20 — restaurar confiança operacional.

