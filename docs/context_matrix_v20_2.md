# Central Operacional V20.2 - Matriz Contextual

## Objetivo

Definir a matriz inicial de contexto x evento x relevancia para a V20.2. Esta matriz nao implementa regras ainda; ela orienta os futuros sensores do Context Engine e do Relevance Engine.

## Principios

- A relevancia depende do contexto, nao apenas do evento.
- Um mesmo evento pode ser baixo, medio, alto ou critico dependendo de presenca, horario, estado operacional, ambiente e recorrencia.
- A matriz deve ser explicavel e auditavel.
- A V20.2 deve produzir sensores paralelos, sem substituir V20.1B.

## Contextos

### Contexto Humano

| Contexto | Definicao | Fontes provaveis |
|---|---|---|
| `casa_ocupada` | ha presenca humana provavel | sensores de presenca, movimento, dispositivos pessoais |
| `casa_vazia` | nao ha presenca humana provavel | ausencia de presenca + modo fora |
| `modo_dormir` | casa em periodo de sono | helper de modo dormir |
| `banho_em_andamento` | sensor do box indica banho semantico | `sensor.casa_banho_estado_v20` |
| `atividade_relevante` | evento humano/operacional em curso | `sensor.atividade_relevante`, V20.1B |

### Contexto Temporal

| Contexto | Definicao |
|---|---|
| `madrugada` | periodo sensivel, baixa atividade esperada |
| `manha` | rotina ativa |
| `horario_comercial` | impacto maior para internet/energia |
| `noite` | risco maior para seguranca e sono |
| `fim_de_semana` | peso contextual diferente para rotina |

### Contexto Operacional

| Contexto | Definicao |
|---|---|
| `infra_normal` | energia, internet e backup sem incidentes |
| `infra_degradada` | redundancia acionada ou link parcial |
| `infra_indisponivel` | falha de conectividade ou energia relevante |
| `recuperacao_operacional` | estado voltando ao normal apos incidente |
| `redundancia_ativa` | 4G/UPS/backup operando como mitigacao |

### Contexto Ambiental

| Contexto | Definicao |
|---|---|
| `chuva_ativa` | chuva detectada |
| `janela_aberta` | qualquer janela/contato relevante aberto |
| `vazamento_detectado` | sensor de vazamento ativo |
| `porta_aberta` | porta relevante aberta |

## Matriz Evento x Contexto x Relevancia

| Evento | Contexto | Relevancia | Motivo |
|---|---|---|---|
| Porta sala aberta | Casa ocupada, horario diurno | media | Pode ser uso normal |
| Porta sala aberta | Casa vazia | alta | Risco de seguranca |
| Porta sala aberta | Madrugada | alta | Evento inesperado |
| Porta sala aberta | Modo dormir | alta | Pode indicar risco ou interrupcao |
| Porta interna aberta | Opcionais desligados | baixa | Evento informativo |
| Janela aberta | Sem chuva | baixa/media | Contexto ambiental normal |
| Janela aberta | Chuva ativa | alta | Risco ambiental direto |
| Chuva ativa | Janelas fechadas | baixa | Evento ambiental controlado |
| Chuva ativa | Janela aberta | alta | Evento composto relevante |
| Vazamento detectado | Casa ocupada | alta | Requer acao humana |
| Vazamento detectado | Casa vazia | critica | Alto risco sem resposta local |
| Banho iniciado | Presenca no banheiro | baixa | Contexto humano esperado |
| Banho encerrado | Timeout confirmado | baixa | Evento semantico informativo |
| Movimento interno | Casa ocupada | baixa/media | Pode ser normal |
| Movimento interno | Casa vazia | critica | Anomalia de seguranca |
| Movimento interno | Madrugada | alta | Contextual |
| Falta de energia | UPS com leitura | alta | Incidente mitigado temporariamente |
| Falta de energia | UPS sem leitura | critica | Falta de visibilidade e energia |
| Energia restabelecida | Apos falta | media | Recuperacao operacional |
| Internet degradada | Horario comercial | alta | Impacto operacional |
| Internet degradada | Madrugada | media | Menor impacto humano imediato |
| Internet indisponivel | Casa ocupada | alta | Impacto humano provavel |
| Internet indisponivel | Casa vazia | media/alta | Impacto operacional |
| Failover 4G ativado | Energia presente | media | Redundancia funcionando |
| Failover 4G ativado | Falta de energia | alta | Infraestrutura degradada correlacionada |
| Backup Google falha | Evento unico | media | Risco operacional acumulativo |
| Backup Google falha | Recorrente | alta | Risco de continuidade |
| TV ligada | Contexto normal | baixa | Atividade humana |
| TV ligada | Casa vazia | media/alta | Possivel anomalia ou automacao |

## Modificadores de Relevancia

| Modificador | Efeito |
|---|---|
| Casa vazia | aumenta eventos de seguranca |
| Modo dormir | aumenta eventos de porta/movimento |
| Madrugada | aumenta anomalias |
| Horario comercial | aumenta internet/energia |
| Chuva ativa | aumenta janelas abertas |
| Redundancia ativa | reduz severidade final, mas mantem degradacao |
| Evento recorrente | aumenta relevancia |
| Duracao longa | aumenta relevancia |
| Confianca baixa | limita aumento automatico |

## Regras Iniciais de Relevancia

### Porta Aberta

```text
porta_sala_aberta + casa_vazia = alta
porta_sala_aberta + modo_dormir = alta
porta_sala_aberta + casa_ocupada + dia = media
```

### Chuva e Janela

```text
chuva_ativa + janela_aberta = alta
chuva_ativa + janelas_fechadas = baixa
```

### Internet

```text
internet_degradada + horario_comercial = alta
internet_degradada + redundancia_ativa = media/alta
internet_indisponivel + casa_ocupada = alta
internet_indisponivel + evento_recorrente = critica
```

### Energia

```text
falta_energia + ups_ok = alta
falta_energia + ups_sem_leitura = critica
energia_restabelecida + internet_normalizada = recuperacao_operacional
```

### Banho

```text
box_on + banheiro_presenca = banho_em_andamento
box_off_por_timeout = banho_encerrado_confirmado
```

## Sensores de Saida Previstos

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_contexto_humano_v20_2`
- `sensor.casa_contexto_operacional_v20_2`
- `sensor.casa_contexto_temporal_v20_2`
- `sensor.casa_contexto_ambiental_v20_2`
- `sensor.casa_impacto_humano_v20_2`

## Atributos Obrigatorios

Cada sensor de relevancia deve expor:

- `evento_base`
- `contextos_aplicados`
- `regra_aplicada`
- `relevancia_base`
- `relevancia_final`
- `confianca`
- `motivo`
- `fontes`

## Plano Incremental

1. Implementar contexto humano e temporal.
2. Implementar contexto ambiental.
3. Implementar contexto operacional.
4. Aplicar matriz de relevancia somente em sensores paralelos.
5. Validar por cenarios manuais.
6. Somente depois considerar uso em dashboards/debug.

## Riscos

- Contexto humano incorreto pode elevar criticidade indevidamente.
- Sensores de presenca podem oscilar.
- Excesso de regras pode dificultar manutencao.
- Relevancia pode conflitar com score fixo atual.

## Mitigacao

- Incluir `confianca`.
- Expor `motivo`.
- Evitar acao automatica na V20.2 inicial.
- Criar painel de debug antes de uso produtivo.
- Manter V20.1B como baseline operacional.
