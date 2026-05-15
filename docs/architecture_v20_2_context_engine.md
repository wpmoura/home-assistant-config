# Central Operacional V20.2 - Context Engine

## Objetivo

A V20.2 evolui a Central Operacional de uma camada deterministica de eventos para uma camada contextual e semantica, capaz de interpretar o estado da casa considerando contexto humano, operacional, temporal e ambiental.

A V20.2 nao substitui a V20.1B. Ela nasce como camada paralela, somente leitura, consumindo os contratos determinísticos ja congelados na V20.1B e produzindo novos sensores versionados para validacao.

## Relacao com a V20.1B Congelada

Baseline preservada:

- Tag de referencia: `v20.1b-lote2-central-operacional`.
- A V20.1B permanece como fonte deterministica de eventos.
- A V20.2 le sensores da V20.1B.
- A V20.1B nao depende da V20.2.
- Nenhum alias final deve apontar para sensores V20.2 nesta fase.
- Nenhum dashboard produtivo deve consumir sensores V20.2 nesta fase.

Contratos V20.1B que podem ser lidos:

- `sensor.casa_evento_publicavel_v20`
- `sensor.casa_timeline_v20`
- `sensor.casa_event_feed_v20`
- `sensor.casa_energia_estado_v20`
- `sensor.casa_internet_estado_v20`
- `sensor.casa_failover_estado_v20`
- `sensor.casa_backup_google_estado_v20`
- `sensor.casa_porta_sala_estado_v20`
- `sensor.casa_porta_banheiro_estado_v20`
- `sensor.casa_porta_quarto_menor_estado_v20`
- `sensor.casa_porta_quarto_maior_estado_v20`
- `sensor.casa_janela_varanda_sala_estado_v20`
- `sensor.casa_janela_quarto_maior_estado_v20`
- `sensor.casa_janela_quarto_menor_estado_v20`
- `sensor.casa_chuva_estado_v20`
- `sensor.casa_vazamento_estado_v20`
- `sensor.casa_banho_estado_v20`

## Principios Arquiteturais

1. Paralelismo seguro: V20.2 deve poder ser desabilitada sem impacto na V20.1B.
2. Leitura explicavel: todo sensor contextual deve expor atributos de origem, regra aplicada, confianca e motivo.
3. Sem acoplamento reverso: V20.1B nao deve consultar sensores V20.2.
4. Sem aliases publicos nesta fase: contratos finais continuam apontando para a arquitetura atual.
5. Sem LLM nesta fase: narrativa deve ser deterministica, com estrutura pronta para LLM futura.
6. Privacidade por padrao: contexto humano deve ser tratado com cuidado, especialmente banho, presenca e modo dormir.
7. Observabilidade antes de automacao: primeiro medir e explicar, depois decidir se vira acao.
8. Compatibilidade com legado: automacoes antigas, V19 e `sensor.status_casa` permanecem intocados.

## Desenho de Componentes

```text
Sensores fisicos e fontes vivas
        |
        v
V20.1B - Deterministic Event Layer
        |
        v
V20.2 - Context Engine
        |
        +--> Human Context
        +--> Operational Context
        +--> Temporal Context
        +--> Environmental Context
        |
        v
V20.2 - Correlation Engine
        |
        v
V20.2 - Relevance Engine
        |
        v
V20.2 - Dynamic Score Engine
        |
        v
V20.2 - Semantic Narrative
```

## Packages Previstos

| Package | Responsabilidade | Status inicial |
|---|---|---|
| `motor_contexto_v20_2.yaml` | Contexto humano, operacional, temporal e ambiental | Novo |
| `motor_correlacao_v20_2.yaml` | Eventos compostos, memoria temporal e agrupamento | Novo |
| `motor_relevancia_v20_2.yaml` | Matriz contexto x evento x relevancia | Novo |
| `motor_score_dinamico_v20_2.yaml` | Score explicavel com modificadores | Novo |
| `motor_narrativa_semantica_v20_2.yaml` | Narrativa deterministica futura | Novo |
| `debug_context_engine_v20_2.yaml` | Observabilidade e diagnostico tecnico | Opcional |

## Sensores Previstos

Contexto:

- `sensor.casa_contexto_humano_v20_2`
- `sensor.casa_contexto_operacional_v20_2`
- `sensor.casa_contexto_temporal_v20_2`
- `sensor.casa_contexto_ambiental_v20_2`
- `binary_sensor.casa_vazia_v20_2`
- `binary_sensor.contexto_noturno_v20_2`
- `binary_sensor.contexto_modo_dormir_v20_2`
- `binary_sensor.contexto_risco_humano_v20_2`

Correlacao:

- `sensor.casa_evento_composto_v20_2`
- `sensor.casa_evento_correlacionado_v20_2`
- `sensor.casa_event_memory_v20_2`
- `sensor.casa_contexto_causal_v20_2`

Relevancia:

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_impacto_humano_v20_2`
- `sensor.casa_prioridade_contextual_v20_2`
- `sensor.casa_confianca_contextual_v20_2`

Score e narrativa:

- `sensor.casa_score_dinamico_v20_2`
- `sensor.casa_score_explicacao_v20_2`
- `sensor.casa_narrativa_semantica_v20_2`

## Contratos de Entrada

Entradas deterministicas:

- estados normalizados V20.1B;
- timeline/feed V20;
- helpers operacionais existentes;
- sensores de presenca e modo dormir;
- horario local;
- sensores ambientais como chuva, janelas, portas e vazamento.

Entradas devem ser tratadas como sinais, nao como verdade absoluta. A V20.2 deve sempre expor confianca quando combinar multiplas fontes.

## Contratos de Saida

Saidas V20.2 devem ser versionadas e explicaveis.

Padrao de atributos recomendado:

```yaml
origem:
eventos_relacionados:
contextos_usados:
regra_aplicada:
score_base:
modificadores:
score_final:
confianca:
timestamp_ocorrencia:
timestamp_confirmacao:
motivo:
```

Nenhuma saida V20.2 deve substituir alias final nesta fase.

## Event Memory / Temporal Buffer

A V20.2 deve introduzir memoria curta de eventos para permitir correlacao. A memoria nao precisa ser historico permanente; ela serve para agrupar eventos proximos.

Janela inicial sugerida:

- curto prazo: 5 minutos;
- medio prazo: 15 minutos;
- longo operacional: 60 minutos, somente para recorrencia.

Uso previsto:

- falta de energia seguida de failover;
- failover encerrado seguido de internet normalizada;
- chuva ativa com janela aberta;
- banho detectado apos presenca no box;
- porta aberta durante casa vazia;
- evento repetido em curto periodo.

## Confidence Engine

O motor de confianca deve indicar o quanto uma inferencia e confiavel.

Fatores:

- quantidade de fontes conhecidas;
- estabilidade temporal;
- concordancia entre sensores;
- ausencia de `unknown`/`unavailable`;
- redundancia operacional;
- confirmacao por transicao real.

Exemplo de classificacao:

| Confianca | Uso |
|---|---|
| `alta` | Pode alimentar prioridade contextual |
| `media` | Pode aparecer em debug ou narrativa |
| `baixa` | Nao deve aumentar criticidade sem confirmacao |
| `desconhecida` | Deve gerar fallback, nao inferencia |

## Motor de Inferencia Contextual

Inferencias iniciais:

- `casa_vazia`
- `casa_ocupada`
- `modo_dormir`
- `contexto_noturno`
- `banho_em_andamento`
- `infraestrutura_degradada`
- `recuperacao_internet`
- `risco_ambiental`
- `risco_humano`
- `anomalia_noturna`

Cada inferencia deve declarar:

- fontes usadas;
- regra;
- confianca;
- motivo;
- se e apenas informativa ou se impacta score.

## Estrategia Incremental por Lotes

### Lote 1 - Contexto Base

- Criar sensores de contexto humano, temporal, operacional e ambiental.
- Nao alterar score.
- Nao alterar aliases.
- Validar atributos e confianca.

### Lote 2 - Relevancia Contextual

- Criar matriz evento x contexto x relevancia.
- Gerar `sensor.casa_relevancia_contextual_v20_2`.
- Expor explicacao de regra.

### Lote 3 - Correlacao e Memoria

- Criar event memory.
- Criar eventos compostos.
- Registrar debito de ordenacao internet/failover.
- Separar `timestamp_ocorrencia` e `timestamp_confirmacao`.

### Lote 4 - Score Dinamico

- Criar score paralelo.
- Comparar com score operacional atual.
- Nao substituir `sensor.casa_score_operacional`.

### Lote 5 - Narrativa Deterministica

- Criar narrativa contextual curta.
- Preparar estrutura para LLM futura.
- Nao integrar LLM ainda.

## Riscos e Mitigacao

| Risco | Impacto | Mitigacao |
|---|---|---|
| Inferencia errada por sensor instavel | Score incorreto | Usar confianca e estabilidade temporal |
| Duplicidade entre evento simples e composto | Timeline ruidosa | Event memory e deduplicacao semantica |
| Exposicao de contexto humano sensivel | Privacidade | Debug restrito e publicacao opcional |
| Acoplamento com V20.1B | Regressao | V20.2 somente leitura |
| Score opaco | Dificuldade de manutencao | Atributos de explicacao obrigatorios |
| LLM prematuro | Comportamento imprevisivel | Narrativa deterministica antes de IA |

## Rollback

Rollback deve ser simples:

1. Desabilitar packages V20.2.
2. Recarregar/reiniciar Home Assistant.
3. V20.1B continua operando.
4. Aliases finais permanecem inalterados.
5. Dashboards produtivos continuam lendo a camada atual.

## Criterios de Aceite da Arquitetura

- V20.2 e paralela.
- V20.2 le V20.1B.
- V20.1B nao depende da V20.2.
- Nenhum alias final aponta para V20.2.
- Nenhum dashboard produtivo consome V20.2.
- Todo sensor contextual possui atributos explicaveis.
- Todo score dinamico possui decomposicao.
- Toda correlacao possui fonte, janela temporal e confianca.
