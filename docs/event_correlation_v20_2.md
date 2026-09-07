# Central Operacional V20.2 - Event Correlation

## Objetivo

Definir a estrategia de correlacao de eventos da V20.2 para transformar eventos isolados em situacoes operacionais compostas, sem alterar a V20.1B.

A correlacao deve operar em paralelo, lendo a timeline e os sensores determinísticos V20.1B.

## Principios

- Eventos simples continuam existindo.
- Eventos compostos sao inferencias adicionais.
- A V20.2 nao deve impedir a publicacao da V20.1B.
- A correlacao deve ser explicavel.
- Eventos correlacionados devem expor janela temporal, fontes e confianca.

## Contratos de Entrada

Entradas principais:

- `sensor.casa_timeline_v20`
- `sensor.casa_event_feed_v20`
- `sensor.casa_evento_publicavel_v20`
- sensores determinísticos V20.1B
- atributos `eventos_json`, `linha_1`...`linha_6`

Entradas por dominio:

- Energia: `sensor.casa_energia_estado_v20`
- Internet: `sensor.casa_internet_estado_v20`
- Failover: `sensor.casa_failover_estado_v20`
- Backup: `sensor.casa_backup_google_estado_v20`
- Chuva: `sensor.casa_chuva_estado_v20`
- Janela: sensores `sensor.casa_janela_*_estado_v20`
- Porta: sensores `sensor.casa_porta_*_estado_v20`
- Vazamento: `sensor.casa_vazamento_estado_v20`
- Banho: `sensor.casa_banho_estado_v20`

## Contratos de Saida

Sensores previstos:

- `sensor.casa_event_memory_v20_2`
- `sensor.casa_evento_composto_v20_2`
- `sensor.casa_contexto_causal_v20_2`
- `sensor.casa_correlacao_ativa_v20_2`

Atributos esperados:

- `eventos_relacionados`
- `janela_temporal`
- `evento_raiz`
- `evento_resultante`
- `ordem_semantica`
- `confianca`
- `motivo`
- `timestamp_ocorrencia`
- `timestamp_confirmacao`

## Event Memory / Temporal Buffer

O buffer temporal e uma memoria curta para correlacionar eventos proximos.

Janelas sugeridas:

| Janela | Uso |
|---|---|
| 1 minuto | ordenacao de eventos quase simultaneos |
| 5 minutos | correlacao operacional curta |
| 15 minutos | recuperacao e encadeamento |
| 60 minutos | recorrencia e diagnostico |

O buffer deve guardar:

- tipo do evento;
- origem;
- mensagem;
- timestamp de ocorrencia;
- timestamp de confirmacao, quando aplicavel;
- dominio;
- confianca.

## Eventos Compostos Previstos

### Recuperacao de Internet/Failover

Padrao:

```text
failover_4g_ativo
internet_degradada
failover_4g_encerrado
internet_normalizada
```

Evento composto:

```text
recuperacao_conectividade
```

Narrativa deterministica futura:

```text
Internet voltou ao normal apos encerramento do failover 4G.
```

Debito tecnico registrado:

- A V20.1B pode publicar `Internet normalizada` antes de `Failover 4G encerrado`.
- Isso nao e bug funcional.
- V20.2/V20.3 deve ordenar semanticamente eventos correlacionados.

### Falta de Energia com Degradacao de Internet

Padrao:

```text
falta_energia
failover_4g_ativo
internet_degradada
```

Evento composto:

```text
infraestrutura_degradada_por_energia
```

Relevancia:

- alta se casa ocupada;
- critica se UPS sem leitura;
- media/alta se redundancia ativa.

### Chuva com Janela Aberta

Padrao:

```text
chuva_ativa
janela_aberta
```

Evento composto:

```text
risco_ambiental_chuva_janela
```

Relevancia:

- alta.

### Porta Aberta com Casa Vazia

Padrao:

```text
casa_vazia
porta_sala_aberta
```

Evento composto:

```text
risco_seguranca_porta_casa_vazia
```

Relevancia:

- alta ou critica, conforme horario.

### Banho Semantico

Padrao:

```text
presenca_box
banho_detectado
banho_encerrado_confirmado
```

Evento composto:

```text
banho_em_andamento
```

Observacao temporal:

- Na V20.1B, o horario publicado de `Banho encerrado` representa confirmacao.
- V20.2 deve separar `timestamp_ocorrencia` e `timestamp_confirmacao`.

### Backup Falhando Recorrente

Padrao:

```text
backup_google_falha
backup_google_falha repetido em janela de 24h
```

Evento composto:

```text
risco_backup_recorrente
```

Relevancia:

- alta, mesmo que cada falha isolada seja media.

## Ordenacao Semantica

Objetivo: publicar ou interpretar eventos em ordem operacional, nao apenas em ordem tecnica de disparo.

Exemplo:

Ordem tecnica possivel:

```text
Internet normalizada
Failover 4G encerrado
```

Ordem semantica desejada:

```text
Failover 4G encerrado
Internet normalizada
```

Regra futura:

- Se eventos `failover_encerrado` e `internet_normalizada` ocorrem em uma janela curta, a narrativa/contexto deve ordenar `failover_encerrado` antes de `internet_normalizada`.

## Confidence Engine

Confianca da correlacao:

| Nivel | Criterio |
|---|---|
| Alta | eventos relacionados ocorreram na janela esperada e fontes estao conhecidas |
| Media | eventos relacionados ocorreram, mas alguma fonte esta instavel |
| Baixa | apenas um evento sugere correlacao |
| Desconhecida | fontes indisponiveis |

## Deduplicacao Semantica

Regras:

- Nao repetir evento composto se os eventos base nao mudaram.
- Nao publicar narrativa composta a cada atualizacao de atributo.
- Manter evento simples e composto separados em sensores diferentes na fase inicial.
- Evitar que dashboards produtivos consumam eventos compostos antes de validacao.

## Plano Incremental

1. Criar `sensor.casa_event_memory_v20_2`.
2. Expor ultimos eventos com timestamp e dominio.
3. Criar correlacoes somente como atributos, sem publicacao visual.
4. Criar `sensor.casa_evento_composto_v20_2`.
5. Validar cenarios de internet/failover, energia, chuva/janela e banho.
6. Criar narrativa deterministica.
7. Somente depois avaliar integracao visual.

## Riscos

- Agrupar eventos que nao tem relacao causal.
- Ocultar detalhes importantes atras de evento composto.
- Gerar narrativa em ordem incorreta.
- Aumentar complexidade da timeline.

## Mitigacao

- Sempre expor eventos base.
- Expor `eventos_relacionados`.
- Expor `janela_temporal`.
- Expor `confianca`.
- Manter V20.1B inalterada.
- Validar correlacoes em debug antes de uso produtivo.
