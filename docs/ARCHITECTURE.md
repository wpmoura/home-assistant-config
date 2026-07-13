# Arquitetura da Central Operacional

## Visão Arquitetural

A Central Operacional é uma camada de interpretação sobre o Home Assistant. Ela transforma fontes operacionais em contratos estáveis para dashboards, diagnóstico e futuras automações.

A baseline V20 separa quatro responsabilidades:

- Coleta e mensagens atuais.
- Classificação do evento dominante.
- Publicação de timeline/feed.
- Aliases finais sem versão.

## Contexto Operacional Atual

A fonte única de verdade da Central Operacional deve consolidar o estado atual do projeto e manter alinhamento entre arquitetura, roadmap e implementação.

Estado oficial:

- V20.0 = concluída e congelada
- V20.1A = concluída
- V20.1B lote 1 = concluída
- V20.1B lote 2 = concluída/parcial para energia, internet, failover e backup
- V20.1C = `V20.1C_FECHAMENTO` registrado; diagnóstico e governança concluídos, decommission bloqueado
- V20.1D/E = checkpoint documental de dependências e impacto concluído; limpeza não autorizada
- V20.1K = concluída; tag `V20.1K_FECHAMENTO` criada
- V20.1N = homologada; checkpoint de estabilização registrado
- V20.1O = congelada; política Timeline / Push / Agregação estabilizada
- V20.1Q = decisão arquitetural formalizada; implementação não iniciada
- V20.2A = concluída; dashboard legado `teste-4` removido pela UI
- V20.2B = auditoria executada, sem ação operacional
- V20.2/V20.3/V21 = planejamento futuro

Arquitetura oficial:

- Sensores físicos
- V20.1B Deterministic Event Layer
- Context Engine V20.2
- Relevance Engine V20.2
- Correlation Engine V20.2
- Dynamic Score
- Narrativa determinística
- Aliases finais futuros

Regras fundamentais:

- Nunca alterar `sensor.status_casa`
- Nunca alterar aliases finais sem validação
- Dashboards produtivos não consomem `_v20_2`
- V20.1O não deve ser alterada diretamente após congelamento; correções futuras devem abrir lote formal
- V20.2 permanece isolada em shadow
- IA é opcional; IA desligada mantém o sistema funcional
- Não substituir automações legadas sem auditoria V20.1C
- V20.1C não autoriza decommission; nenhuma limpeza ou desativação automática está autorizada
- Packages novos devem permitir rollback simples
- Radar de Movimento deve ser sob demanda

## Princípio de IA Opcional

A arquitetura da Central Operacional deve permanecer determinística por padrão.

Regra principal:

- IA desligada: o sistema continua funcionando 100% pelo motor determinístico.
- IA ligada: a IA apenas enriquece análise, contexto, explicações e recomendações.
- IA nunca deve ser dependência obrigatória para eventos críticos.
- Eventos críticos devem ser detectados, publicados e tratados por sensores, templates e motores determinísticos.
- O dashboard deve expor controle claro para `IA ativa` / `IA desligada`.
- A evolução ideal deve prever modos `IA desligada`, `IA leve` e `IA completa`.

Pipeline correto:

```text
Sensores reais
        │
        ▼
Motores determinísticos
        │
        ▼
Evento, relevância, contexto e score
        │
        ├── dashboards e alertas críticos
        │
        └── IA opcional para enriquecimento
```

Anti-padrão:

```text
Sensores reais
        │
        ▼
IA
        │
        ▼
Decisão crítica
```

Essa regra protege performance, previsibilidade e resiliência do Home Assistant.

## Motores Principais

### Núcleo de Mensagens Atual

Arquivo:

- `packages/central_mensagens_corrigido.yaml`

Responsabilidade:

- Produzir mensagens operacionais reais.
- Servir como fonte viva do estado textual atual.
- Alimentar aliases finais quando a timeline V20 ainda não tem eventos.

### Motor de Evento Dominante

Arquivo:

- `packages/motor_eventos_v20.yaml`

Entidades principais:

- `sensor.casa_evento_dominante_v20`
- `sensor.casa_evento_atual_v20`
- `sensor.casa_evento_origem_v20`
- `sensor.casa_evento_tipo_v20`
- `binary_sensor.casa_evento_ativo_v20`

Responsabilidade:

- Identificar o evento principal.
- Expor origem, tipo, severidade, score base e mensagem.
- Operar em paralelo à timeline.

### Motor de Timeline e Feed

Arquivo:

- `packages/motor_timeline_v20.yaml`

Entidades principais:

- `sensor.casa_evento_publicavel_v20`
- `sensor.casa_timeline_v20`
- `sensor.casa_event_feed_v20`

Responsabilidade:

- Registrar eventos relevantes por transição.
- Manter os 6 eventos mais recentes.
- Publicar eventos secundários mesmo quando há dominante ativo.
- Evitar spam de eventos consecutivos.

## Aliases

Arquivo:

- `packages/central_operacional_aliases_v20.yaml`

Aliases finais:

- `sensor.status_casa`
- `sensor.atividade_relevante`
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- `sensor.casa_contexto_humano`
- `sensor.casa_prioridade`
- `sensor.casa_score_operacional`

Responsabilidade:

- Servir como contrato público sem versão.
- Proteger dashboards contra sensores experimentais.
- Aplicar fallback para estados inválidos.

## Timeline

A timeline V20 usa o formato:

```text
HH:MM mensagem
```

Exemplos:

```text
09:02 📺 TV desligada
09:14 ⚠️ Backup Google com erro
09:18 🚪 Porta sala aberta
09:20 🌧️ Chuva iniciada
```

A timeline mantém:

- `linha_1`
- `linha_2`
- `linha_3`
- `linha_4`
- `linha_5`
- `linha_6`

## Feed

O feed operacional é um espelho da timeline preparado para consumo visual.

Entidades:

- `sensor.casa_event_feed_v20`
- `sensor.casa_event_feed`

O feed não depende do evento dominante. Ele registra eventos por transição de fonte.

## Parâmetros

Arquivo:

- `packages/parametros_operacionais_v20.yaml`

Helpers principais:

- `input_number.casa_peso_ups`
- `input_number.casa_timeout_evento_minutos`
- `input_number.casa_confianca_minima_evento`
- `input_number.casa_timeout_porta_aberta_minutos`
- `input_boolean.casa_eventos_visiveis`
- `input_boolean.casa_prioridade_contextual_ativa`

## Contexto

O contexto V20 ainda é uma camada inicial.

Fontes atuais:

- `sensor.central_tv_contexto_fonte`
- `sensor.central_atividade_relevante_fonte`
- `sensor.central_categoria_alerta`
- `sensor.casa_contexto_humano`

A evolução natural fica no roadmap V21/V22.

## Recovery 4G — V20.1Q

Princípio arquitetural aprovado:

> A Central decide. O Executor atua. A Central valida. A Central encerra.

A Central mantém detecção, decisão, validação, encerramento e publicação canônica. O Executor futuro fica restrito à execução física subordinada e ao relato de fatos técnicos, sem detector próprio, interpretação de ping/bytes ou decisão semântica.

O despacho subordinado está em `docs/arquitetura/despacho_arquitetural_v20_1q.md`. A implementação permanece bloqueada pelos gates e pelo plano `docs/releases/implementation_plan_v20_1q.md`.

## Separação Legado e V20

Estado após V20.1K/V20.2A:

- Artefatos V19 históricos ficam em `archive/packages_disabled/`, fora da árvore carregada por `packages/`.
- Dashboard legado `teste-4` foi removido pela UI/fluxo suportado.
- Dashboards ativos não possuem resíduos V19 conhecidos.
- Entidades V19 residuais em registry/restore_state não devem ser limpas manualmente.
- Auditoria V20.2B identificou 21 automações órfãs; automações críticas não devem ser removidas automaticamente.

Regras da baseline:

- Dashboards oficiais não devem consumir sensores V19 diretamente.
- `status_casa.yaml` permanece como governança/pesos históricos.
- `sensor.status_casa` é alias final e deve permanecer estável.
- Novos motores devem nascer paralelos e só depois serem promovidos aos aliases.

## Relação Entre Packages

```text
central_mensagens_corrigido.yaml
        │
        ├── central_operacional_aliases_v20.yaml ── aliases finais
        │
        └── motor_eventos_v20.yaml ── evento dominante

motor_timeline_v20.yaml ── timeline/feed V20
        │
        └── central_operacional_aliases_v20.yaml ── sensor.casa_timeline / sensor.casa_event_feed

parametros_operacionais_v20.yaml ── helpers V20
        ├── motor_eventos_v20.yaml
        └── motor_timeline_v20.yaml

status_casa.yaml ── pesos/criticidade/governança histórica
```

## Contratos que Devem Permanecer Estáveis

- Entity IDs finais sem versão.
- Sensores do motor dominante V20.
- Sensores da timeline/feed V20.
- Helpers V20 já publicados.
- Fallbacks dos aliases finais.
