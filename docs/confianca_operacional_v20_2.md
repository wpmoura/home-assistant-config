# V20.2 Lote 2B - Confidence & Stability Layer

## Objetivo

Este documento formaliza o contrato operacional da camada de confiança e estabilidade da V20.2.

A camada deve nascer em modo paralelo e observável, sem interferir na relevância oficial, timeline, aliases finais, dashboards produtivos ou score principal.

Package futuro:

`packages/motor_confianca_v20_2.yaml`

Sensores planejados:

- `sensor.casa_confianca_contextual_v20_2`
- `sensor.casa_estabilidade_contextual_v20_2`
- `sensor.casa_evento_contextual_estavel_v20_2`

## Principios

- A confiança deve ser explicável.
- A estabilidade deve ser temporal, não apenas booleana.
- A camada deve operar em shadow mode na primeira fase.
- A V20.2 pode ler V20.1B, mas V20.1B não deve depender da V20.2.
- Nenhum alias final deve apontar para esta camada nesta fase.
- Nenhum dashboard produtivo deve depender desta camada nesta fase.
- A camada não deve publicar eventos na timeline.
- A camada não deve substituir `sensor.casa_score_operacional`.

## Pipeline Operacional Oficial

Pipeline conceitual:

1. Evento determinístico V20.1B
2. Contexto base V20.2 Lote 1
3. Relevância contextual V20.2 Lote 2A
4. Confidence & Stability V20.2 Lote 2B
5. Observabilidade/debug
6. Integração futura com score dinâmico, se aprovada

Nesta fase, o passo 4 observa e classifica, mas não altera o resultado do passo 3.

## Maquina de Estados da Confidence

Estados:

- `alta`
- `media`
- `baixa`
- `indeterminada`

### alta

Condição geral:

- fontes principais válidas
- evento/contexto coerente
- estabilidade mínima atingida
- sem flapping recente relevante

Exemplo:

- chuva ativa por tempo mínimo
- janela aberta confirmada
- contexto ambiental consistente

### media

Condição geral:

- fontes válidas
- evento ainda recente
- contexto parcialmente estável
- pequena oscilação tolerável

Exemplo:

- internet degradada há poucos segundos, mas sem retorno imediato

### baixa

Condição geral:

- fontes oscilantes
- evento muito curto
- contexto contraditório
- flapping detectado

Exemplo:

- presença alternando rapidamente entre casa vazia e ocupada

### indeterminada

Condição geral:

- fontes críticas em `unknown`, `unavailable`, `none` ou vazias
- ausência de sensores determinísticos suficientes
- contexto base indisponível

## Maquina de Estados da Estabilidade

Estados:

- `instavel`
- `observando`
- `estavel`
- `indeterminada`

### instavel

O estado observado mudou múltiplas vezes dentro da janela de estabilidade.

Exemplos:

- chuva ativa/inativa alternando rapidamente
- porta abrindo e fechando repetidamente
- internet degradada/normal em sequência curta

### observando

O evento foi detectado, mas ainda não atingiu a permanência mínima.

Exemplo:

- porta abriu há 5 segundos e o threshold é 10 segundos

### estavel

O evento permanece coerente por tempo suficiente.

Exemplo:

- chuva ativa por 60 segundos
- energia ausente por 10 segundos

### indeterminada

Não há fontes suficientes para avaliar estabilidade.

## Thresholds Oficiais por Dominio

Valores iniciais planejados:

| Domínio | Estabilidade mínima | Cooldown | Observação |
|---|---:|---:|---|
| Porta | 10s | 30s | Evita abre/fecha acidental |
| Janela | 15s | 60s | Evita contato oscilante |
| Chuva | 60s | 3min | Evita falso positivo de sensor climático |
| Presença | 2min | 5min | Evita casa vazia por ausência temporária de movimento |
| Internet | 30s | 2min | Evita degradações curtas |
| Energia | 10s | 1min | Deve reagir rápido, mas sem startup falso |
| Banho | timeout existente | 3min | Reusa tolerância de banho da V20.1B |
| Failover | 20s | 2min | Evita failover transitório |
| Backup Google | 5min | 30min | Falhas tendem a ser menos instantâneas |

Esses thresholds devem começar como contrato documental. Helpers só devem ser criados se a calibração operacional mostrar necessidade real.

## Debounce

Debounce é a exigência de permanência mínima antes de aceitar uma mudança contextual.

Regras:

- mudanças rápidas devem entrar como `observando`
- se o evento retornar ao estado anterior antes do threshold, a mudança não é promovida
- debounce não deve apagar o evento bruto da V20.1B
- debounce não deve bloquear timeline V20.1B

Exemplo:

- internet muda para `degradada`
- aguarda 30 segundos
- se continuar degradada, estabilidade pode virar `estavel`
- se normalizar antes, a confidence permanece baixa ou média

## Cooldown

Cooldown evita reprocessamento excessivo da mesma condição contextual.

Regras:

- cooldown atua sobre evento + domínio + contexto
- cooldown não impede correção de estado
- cooldown não deve esconder evento crítico novo
- mudança real de contexto pode encerrar o cooldown

Exemplo:

- `porta_aberta_casa_vazia` foi classificado como alta
- enquanto a porta continuar aberta e casa vazia, não recalcular repetidamente
- se a casa deixar de estar vazia, o contexto muda e o cooldown deixa de se aplicar

## Hysteresis

Hysteresis evita oscilação entre níveis de confiança/relevância.

Regras:

- subir criticidade pode exigir menos tempo que baixar
- baixar criticidade exige confirmação mais estável
- estados críticos devem ter saída conservadora

Exemplo:

- `chuva + janela aberta` sobe para crítica rapidamente após estabilidade mínima
- para sair de crítica, exige janela fechada ou chuva encerrada de forma estável

## Temporal Decay

Temporal decay reduz o peso de eventos antigos sem nova evidência.

Regras:

- eventos persistentes podem manter relevância, mas não devem gerar repetição artificial
- eventos antigos sem confirmação nova perdem confiança
- incidentes operacionais persistentes podem subir por duração em motor futuro, mas não nesta fase

Exemplo:

- backup com falha continua relevante
- sem nova evidência, a camada mantém estado estável, mas não promove indefinidamente

## Regras de Bypass Critico

Alguns eventos podem furar debounce/cooldown parcial por segurança operacional.

Eventos candidatos:

- vazamento detectado
- energia ausente
- porta aberta com casa vazia em contexto noturno
- chuva ativa com janela aberta

Regras:

- bypass não elimina atributos de confiança
- bypass deve marcar `bypass_critico: true`
- bypass não publica timeline nesta fase
- bypass deve continuar protegido contra `unknown` e startup/reload

## Comportamento para Unknown/Unavailable

Estados inválidos:

- `unknown`
- `unavailable`
- `none`
- `None`
- vazio

Regras:

- não promover relevância
- não declarar estabilidade alta
- confidence deve ser `indeterminada` ou `baixa`
- evento contextual estável deve ser `nenhum` ou `indeterminado`
- atributos devem listar quais fontes estavam inválidas

## Atributos Padrao dos Sensores

Todo sensor desta camada deve expor:

- `evento_observado`
- `evento_estavel`
- `dominio`
- `estado_anterior`
- `estado_atual`
- `duracao_estado`
- `threshold_aplicado`
- `cooldown_ativo`
- `debounce_aplicado`
- `hysteresis_aplicada`
- `temporal_decay_aplicado`
- `bypass_critico`
- `confidence_score`
- `confianca`
- `estabilidade`
- `motivo`
- `fontes_usadas`
- `fontes_invalidas`
- `timestamp`

## Shadow Mode

Na primeira implementação, a camada deve operar em shadow mode.

Shadow mode significa:

- calcular confiança e estabilidade
- expor atributos para auditoria
- não alterar `sensor.casa_relevancia_contextual_v20_2`
- não alterar timeline/feed
- não alterar aliases finais
- não alterar dashboards produtivos
- não alterar score principal

Objetivo do shadow mode:

- medir comportamento real
- detectar falsos positivos
- calibrar thresholds
- validar estabilidade antes de influenciar decisões

## Metricas de Validacao

Métricas sugeridas:

- quantidade de mudanças de contexto por hora
- quantidade de eventos classificados como instáveis
- tempo médio até estabilidade
- quantidade de eventos com confidence baixa
- quantidade de eventos em bypass crítico
- quantidade de flaps por domínio
- divergência entre relevância contextual e estabilidade
- falsos positivos observados manualmente

## Criterios de Promocao

Um evento pode ser promovido quando:

- fontes principais são válidas
- evento determinístico está ativo
- contexto é coerente
- estabilidade mínima foi atingida
- cooldown não bloqueia nova decisão
- confidence é `media` ou `alta`

Eventos com bypass crítico podem ser promovidos antes da estabilidade total, mas devem registrar o bypass em atributo.

## Criterios de Rebaixamento

Um evento pode ser rebaixado quando:

- evento determinístico encerrou
- contexto deixou de sustentar a relevância
- fontes ficaram inválidas
- temporal decay reduziu confiança
- hysteresis confirmou retorno

Rebaixamento de eventos críticos deve ser conservador.

## Riscos

- atrasar incidentes reais
- reduzir visibilidade de eventos curtos importantes
- criar lógica temporal difícil de manter em templates
- duplicar anti-spam já existente da timeline
- gerar confiança artificial por thresholds mal calibrados
- tornar o motor contextual opaco

## Mitigacoes

- iniciar em shadow mode
- expor atributos explicáveis
- não bloquear timeline V20.1B
- não substituir score oficial
- usar poucos domínios inicialmente
- revisar thresholds após validação real
- manter rollback simples

## Rollback

Rollback planejado:

1. Desabilitar ou remover `packages/motor_confianca_v20_2.yaml`.
2. Reiniciar ou recarregar templates/packages.
3. Confirmar que V20.1B e V20.2 Lote 1/2A continuam operando.

Como a camada nasce paralela, o rollback não deve afetar:

- timeline/feed
- aliases finais
- dashboards produtivos
- score operacional atual
- sensores determinísticos V20.1B

## Estrategia Incremental

### Etapa 1 - Contrato

- documentar máquinas de estado
- definir thresholds oficiais
- definir atributos obrigatórios
- manter sem YAML

### Etapa 2 - Sensores Observáveis

- criar sensores em `motor_confianca_v20_2.yaml`
- expor confidence e estabilidade
- manter shadow mode

## Baseline Operacional - V20.2 Lote 2B Fase 1

Status: carregada corretamente no Home Assistant.

Package:

- `packages/motor_confianca_v20_2.yaml`

Sensores carregados:

- `sensor.casa_confianca_contextual_v20_2`
- `sensor.casa_estabilidade_contextual_v20_2`

Estado inicial observado:

| Entidade | Estado inicial |
|---|---|
| `sensor.casa_confianca_contextual_v20_2` | `alta` |
| `sensor.casa_estabilidade_contextual_v20_2` | `estavel` |

Atributos observados:

| Atributo | Valor |
|---|---|
| `shadow_mode` | `true` |
| `estabilidade_temporal_real` | `false` |
| `fontes_invalidas` | `nenhuma` |
| `fontes_contraditorias` | `nenhuma` |
| `contradicao_detectada` | `false` |
| `dominio_estimado` | `nenhum` |
| `dominio_oscilante` | `false` |

Observação operacional:

- `motor_confianca_v20_2` é o nome do package, não uma entidade do Home Assistant.
- Não existe entidade esperada chamada `sensor.motor_confianca_v20_2` ou equivalente.
- O carregamento correto da camada é confirmado pela existência dos dois sensores shadow listados acima.
- Nesta fase, a camada permanece read-only e não interfere em relevância, timeline, aliases finais, dashboards ou score oficial.

### Etapa 3 - Validação Manual

- testar chuva flapping
- testar presença oscilando
- testar internet degradando rapidamente
- testar porta abre/fecha
- testar banho curto
- testar failover rápido

### Etapa 4 - Calibração

- ajustar thresholds
- decidir se helpers são necessários
- medir falsos positivos

### Etapa 5 - Integração Futura

- permitir que relevância contextual leia confiança
- ainda sem alterar score principal
- preparar score dinâmico em fase posterior
