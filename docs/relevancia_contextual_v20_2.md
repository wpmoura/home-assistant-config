# V20.2 Lote 2 - Relevancia Contextual

## Objetivo

O Lote 2 da V20.2 introduz o planejamento da camada de relevancia contextual da Central Operacional.

Essa camada deve interpretar eventos determinísticos já produzidos pela V20.1B em conjunto com os contextos base criados no Lote 1 da V20.2, sem substituir o score operacional atual, sem publicar eventos na timeline e sem alterar aliases finais.

O objetivo técnico é preparar um motor paralelo capaz de responder:

- qual evento é mais relevante agora
- qual domínio operacional está envolvido
- qual contexto aumenta ou reduz a criticidade
- qual score contextual foi calculado
- qual motivo explica a decisão

## Principios de Relevancia Contextual

- A relevancia deve ser explicável por atributos, não apenas por um estado final.
- O mesmo evento pode ter criticidade diferente conforme presença, horário, ambiente e operação.
- A camada V20.2 deve ler a V20.1B, mas a V20.1B não deve depender da V20.2.
- Nenhum alias final deve apontar para sensores de relevancia nesta fase.
- Nenhum dashboard produtivo deve consumir diretamente a relevancia contextual nesta fase.
- Estados inválidos devem resultar em `indeterminada`, não em alerta artificial.
- O score contextual deve ser paralelo e reversível.

## Sensores Planejados

Package futuro:

`packages/motor_relevancia_v20_2.yaml`

Sensores previstos:

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_evento_relevante_v20_2`
- `sensor.casa_relevancia_dominio_v20_2`
- `sensor.casa_motivo_relevancia_v20_2`

Estados sugeridos:

- `baixa`
- `media`
- `alta`
- `critica`
- `indeterminada`

## Matriz Evento x Contexto

| Evento base | Contexto | Relevancia sugerida | Motivo |
|---|---|---:|---|
| Porta aberta | Casa vazia | Alta | Risco de acesso com ausência humana |
| Porta aberta | Alguém em casa | Média | Evento relevante, mas com presença local |
| Porta aberta | Noturno/madrugada | Alta | Abertura em período sensível |
| Chuva ativa | Janela aberta | Crítica | Risco ambiental direto |
| Chuva ativa | Janelas fechadas | Baixa/Média | Condição ambiental sem exposição interna |
| Internet degradada | Manhã/tarde | Alta | Possível impacto operacional em horário útil |
| Internet degradada | Noite/madrugada | Média | Impacto menor, salvo contexto especial futuro |
| Energia ausente | Casa ocupada | Crítica | Impacto direto em conforto, segurança e operação |
| Energia ausente | Casa vazia | Alta | Incidente relevante, mas sem impacto humano imediato |
| Backup Google com falha | Falha simples | Média | Requer atenção, sem impacto imediato |
| Backup Google com falha | Falha recorrente | Alta | Aumenta risco de perda de histórico/configuração |
| Banho detectado | Alguém em casa | Baixa/Média | Contexto humano sem criticidade por si só |
| Movimento interno | Casa vazia + madrugada | Crítica | Possível anomalia de presença |

## Regras de Score

O score contextual deve ser calculado em paralelo e não deve alterar:

- `sensor.casa_score_operacional`
- `sensor.status_casa`
- aliases finais sem versão
- timeline/feed V20.1B

Faixas sugeridas:

| Score final | Relevancia |
|---:|---|
| 0-24 | baixa |
| 25-49 | media |
| 50-79 | alta |
| 80-100 | critica |

Modelo inicial:

- evento base define `score_base`
- contexto humano aplica modificadores
- contexto temporal aplica modificadores
- contexto operacional aplica modificadores
- contexto ambiental aplica modificadores
- confiança reduz ou limita o resultado quando fontes forem inconclusivas

Exemplo conceitual:

```yaml
evento_base: porta_aberta
score_base: 45
modificadores:
  casa_vazia: 25
  contexto_noturno: 15
score_final: 85
relevancia: critica
motivo: Porta aberta com casa vazia em período noturno
```

## Saturacao e Clamp de Score

O score final deve ser limitado entre `0` e `100`.

Regras:

- valores negativos devem ser convertidos para `0`
- valores acima de `100` devem ser convertidos para `100`
- eventos com fontes inválidas devem retornar `indeterminada`
- baixa confiança deve limitar a relevancia máxima, salvo eventos críticos explícitos

Clamp sugerido por confiança:

| Confiança | Limite sugerido |
|---|---:|
| alta | 100 |
| média | 79 |
| baixa | 49 |
| indeterminada | sem score confiável |

## Cooldown

Cooldown é o intervalo mínimo para evitar repetição excessiva da mesma decisão contextual.

Na V20.2 Lote 2, o cooldown deve ser apenas planejado. Ele não deve publicar eventos nem alterar timeline.

Uso futuro:

- evitar que a mesma porta aberta gere múltiplas relevancias equivalentes
- evitar repetição contínua de internet degradada
- evitar escalada repetida de backup em falha
- permitir nova avaliação quando o contexto mudar de fato

Exemplo:

- porta aberta com casa vazia gera `alta`
- enquanto evento e contexto não mudarem, a relevancia permanece estável
- se entrar em período noturno, pode recalcular e subir para `critica`

## Decaimento Temporal

Decaimento temporal reduz a relevancia de eventos antigos que permanecem sem nova evidência.

Na V20.2 Lote 2, o decaimento também deve ser apenas planejado.

Uso futuro:

- backup com falha pode aumentar por recorrência, não por repetição bruta
- evento ambiental antigo pode perder relevancia se não houver mudança
- eventos humanos podem expirar mais rápido que incidentes operacionais

Diretriz:

- duração pode aumentar criticidade em incidentes persistentes
- ausência de mudança pode reduzir ruído visual
- eventos confirmados por sensores físicos têm prioridade sobre inferências antigas

## Atributos Explicaveis Obrigatorios

Todo sensor de relevancia contextual deve expor atributos suficientes para auditoria:

- `evento_base`
- `dominio`
- `contexto_humano`
- `contexto_temporal`
- `contexto_operacional`
- `contexto_ambiental`
- `score_base`
- `modificadores`
- `score_final`
- `motivo`
- `confianca`
- `fontes_usadas`
- `timestamp`

## Riscos de Score Opaco

Riscos principais:

- score final sem explicação clara
- modificadores acumulando criticidade demais
- presença instável elevando falsamente eventos
- regra ambiental competindo com regra operacional
- dependência circular entre contexto, relevancia e score
- usuários não entenderem por que um evento ficou crítico

Mitigações:

- manter atributos explicáveis obrigatórios
- começar com poucas regras
- limitar score por confiança
- não alterar score operacional oficial nesta fase
- não publicar timeline antes de validação manual
- não consumir aliases finais como fonte de decisão

## Estrategia Incremental

### Lote 2A - Contrato

- documentar matriz contextual
- definir sensores planejados
- definir atributos obrigatórios
- validar dependências com V20.1B e V20.2 Lote 1

### Lote 2B - Implementacao Paralela

- criar `motor_relevancia_v20_2.yaml`
- gerar sensores versionados `_v20_2`
- não publicar timeline
- não alterar aliases finais

### Lote 2C - Validacao Manual

- testar porta, chuva, energia, internet, backup e banho
- validar score, motivo, domínio e confiança
- revisar falsos positivos

### Lote 2D - Observabilidade

- preparar painel de debug opcional
- manter fora dos dashboards produtivos
- expor motivo e fontes usadas

### Lote 2E - Preparacao para Score Dinamico

- avaliar se o score contextual pode alimentar um motor futuro
- não substituir `sensor.casa_score_operacional` sem fase específica

## Compatibilidade com V20.1B

Regras obrigatórias:

- V20.2 Lote 2 deve ler sensores determinísticos da V20.1B.
- V20.1B não deve depender da V20.2.
- Nenhum sensor V20.1B deve ser alterado para atender a relevancia.
- Timeline/feed V20.1B não devem ser modificados.
- Anti-spam da timeline não deve ser reutilizado como regra de relevancia.
- Eventos legados não devem voltar a ser fonte principal.

## Compatibilidade com V20.2 Lote 1

Regras obrigatórias:

- usar os contextos base como entrada principal
- tratar `indeterminado` como baixa confiança
- não duplicar lógica de presença, temporalidade, operação ou ambiente
- não criar dependência circular entre contexto e relevancia

Entradas previstas:

- `binary_sensor.casa_vazia_v20_2`
- `binary_sensor.contexto_noturno_v20_2`
- `sensor.casa_contexto_temporal_v20_2`
- `sensor.casa_contexto_humano_v20_2`
- `sensor.casa_contexto_operacional_v20_2`
- `sensor.casa_contexto_ambiental_v20_2`

## Fora de Escopo Nesta Fase

- implementar YAML
- alterar aliases finais
- alterar dashboards produtivos
- publicar eventos na timeline
- substituir score operacional
- implementar narrativa semântica
- implementar correlação composta
- usar LLM
- remover legado

