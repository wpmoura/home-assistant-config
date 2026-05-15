# Central Operacional V20.2 - Dynamic Score

## Objetivo

Definir a arquitetura do score operacional dinamico da V20.2.

O score dinamico deve considerar evento, contexto, impacto humano, duracao, redundancia, recorrencia e confianca. Ele nasce paralelo e nao substitui `sensor.casa_score_operacional` nesta fase.

## Principios

- Score deve ser explicavel.
- Score deve ser rastreavel por atributos.
- Score nao deve depender de texto livre.
- Score fixo atual permanece intacto.
- Score dinamico deve ser validado antes de qualquer promocao para alias final.

## Relacao com V20.1B

A V20.1B fornece eventos determinísticos e estados normalizados.

A V20.2 calcula score dinamico lendo:

- sensores determinísticos V20.1B;
- matriz contextual V20.2;
- correlacoes V20.2;
- contexto humano, operacional, temporal e ambiental.

A V20.1B nao le o score dinamico.

## Sensor Previsto

Principal:

- `sensor.casa_score_dinamico_v20_2`

Auxiliares:

- `sensor.casa_score_explicacao_v20_2`
- `sensor.casa_score_modificadores_v20_2`
- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_confianca_contextual_v20_2`

## Formula Conceitual

```text
score_final =
  score_base_evento
  + impacto_contexto
  + impacto_humano
  + impacto_temporal
  + impacto_duracao
  + impacto_recorrencia
  - mitigacao_redundancia
  - mitigacao_confirmacao_baixa
```

Limites:

```text
score_final minimo = 0
score_final maximo = 100
```

## Componentes do Score

### Score Base do Evento

| Evento | Base inicial sugerida |
|---|---:|
| Internet indisponivel | 80 |
| Internet degradada | 45 |
| Failover 4G ativo | 50 |
| Falta de energia | 80 |
| Backup Google falha | 40 |
| Vazamento detectado | 75 |
| Porta sala aberta | 35 |
| Chuva ativa | 20 |
| Janela aberta | 20 |
| Banho em andamento | 10 |
| TV ativa | 5 |

### Impacto Contextual

| Contexto | Modificador |
|---|---:|
| Casa vazia + porta aberta | +35 |
| Modo dormir + porta aberta | +30 |
| Chuva + janela aberta | +45 |
| Internet degradada + horario comercial | +20 |
| Falta energia + UPS sem leitura | +25 |
| Movimento interno + casa vazia | +45 |
| Backup falhando recorrente | +25 |

### Mitigacoes

| Mitigacao | Modificador |
|---|---:|
| Failover 4G ativo e funcional | -15 |
| UPS com leitura valida | -10 |
| Casa ocupada para evento resolvivel localmente | -5 |
| Evento com baixa confianca | -20 |
| Evento ja normalizado | -30 |

### Duracao

| Duracao | Modificador |
|---|---:|
| ate 1 minuto | 0 |
| 1 a 5 minutos | +5 |
| 5 a 15 minutos | +10 |
| mais de 15 minutos | +20 |
| recorrente em 24h | +15 |

## Exemplo de Calculo

### Internet Degradada Durante Horario Comercial

```text
score_base_evento = 45
impacto_temporal = +20
redundancia_4g = -15
confianca_alta = 0
score_final = 50
```

### Chuva com Janela Aberta

```text
score_base_chuva = 20
impacto_ambiental_janela = +45
score_final = 65
```

### Porta Aberta com Casa Vazia

```text
score_base_porta = 35
impacto_casa_vazia = +35
impacto_temporal_madrugada = +20
score_final = 90
```

### Falta de Energia com UPS sem Leitura

```text
score_base_energia = 80
impacto_ups_sem_leitura = +25
limite_maximo = 100
score_final = 100
```

## Atributos Obrigatorios

`sensor.casa_score_dinamico_v20_2` deve expor:

```yaml
score_base:
impacto_contexto:
impacto_humano:
impacto_temporal:
impacto_duracao:
impacto_recorrencia:
mitigacao_redundancia:
mitigacao_confianca:
score_final:
evento_dominante:
eventos_relacionados:
contextos_aplicados:
confianca:
motivo:
```

## Confidence Engine

O score nao deve subir agressivamente quando a confianca for baixa.

Regras sugeridas:

| Confianca | Regra |
|---|---|
| Alta | aplica modificadores completos |
| Media | aplica modificadores com limite |
| Baixa | nao eleva para critica |
| Desconhecida | score deve permanecer conservador |

Exemplo:

```text
porta_aberta + casa_vazia
se confianca da casa_vazia = baixa
nao elevar automaticamente para critica
```

## Score e Eventos Compostos

Eventos compostos podem alterar o score.

Exemplos:

| Evento composto | Efeito |
|---|---|
| `infraestrutura_degradada_por_energia` | aumenta score de energia/internet |
| `risco_ambiental_chuva_janela` | aumenta score ambiental |
| `risco_seguranca_porta_casa_vazia` | aumenta score de seguranca |
| `backup_falha_recorrente` | aumenta score operacional |
| `recuperacao_conectividade` | reduz score apos normalizacao |

## Compatibilidade

Na fase inicial:

- `sensor.casa_score_operacional` permanece inalterado.
- `sensor.casa_score_dinamico_v20_2` e apenas comparativo.
- Dashboards produtivos nao devem consumir o score dinamico.
- Aliases finais nao devem apontar para V20.2.

## Plano Incremental

### Lote 1 - Score Read Only

- Criar score dinamico paralelo.
- Expor atributos completos.
- Nao alterar dashboards produtivos.

### Lote 2 - Comparacao

- Comparar score fixo atual e score dinamico.
- Registrar divergencias em debug.

### Lote 3 - Ajuste de Pesos

- Calibrar modificadores.
- Adicionar helpers apenas se necessario.

### Lote 4 - Validacao Operacional

- Rodar cenarios manuais.
- Avaliar falsos positivos.
- Validar eventos compostos.

### Lote 5 - Promocao Futura

- Somente apos validacao, considerar alias final ou dashboard produtivo.

## Riscos

| Risco | Mitigacao |
|---|---|
| Score opaco | atributos explicativos obrigatorios |
| Score instavel | debounce/contexto temporal |
| Excesso de criticidade | limites por confianca |
| Conflito com score atual | manter paralelo |
| Regressao em dashboards | nao consumir em producao inicialmente |

## Rollback

Rollback do score dinamico:

1. Desabilitar package V20.2 de score.
2. Manter V20.1B operando.
3. Aliases finais continuam intactos.
4. Dashboards produtivos continuam usando score atual.

## Criterios de Aceite

- Score final entre 0 e 100.
- Atributos explicam todos os modificadores.
- Nenhum alias final alterado.
- Nenhum dashboard produtivo alterado.
- V20.1B continua independente.
- Score dinamico reage a contexto sem depender de texto livre.
