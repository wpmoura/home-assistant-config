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
- V20.1Q = pacote corretivo implementado estaticamente; homologação operacional pendente
- V20.2A = concluída; dashboard legado `teste-4` removido pela UI
- V20.2B = auditoria executada, sem ação operacional
- V20.2/V20.3/V21 = planejamento futuro
- V20.2C-A1 = promoção limitada do CSMR consolidada documentalmente; implementação e publicação em runtime bloqueadas pelo Gate específico

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
- V20.2 permanece isolada em shadow, exceto pela promoção arquitetural limitada do CSMR da V20.2C; a exceção não alcança o Context Engine original nem autoriza implementação antes do Gate
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

## Arquitetura Lovelace e dashboard Parâmetros

A arquitetura Lovelace oficial separa definição operacional de entidades e definição visual:

```text
YAML versionado
  └── packages e helpers

Lovelace Storage
  ├── .storage/lovelace_dashboards        cadastro, título, URL e modo
  └── .storage/lovelace.dashboard_lixo    views, seções e cards de Parâmetros
```

O dashboard exibido ao usuário como `Parâmetros` é um dashboard Lovelace Storage. `dashboard_lixo` é somente seu identificador técnico legado; não é o nome funcional da interface. O cadastro fica em `.storage/lovelace_dashboards` e o conteúdo fica em `.storage/lovelace.dashboard_lixo`.

Os helpers consumidos pelo dashboard são definidos em YAML, principalmente em `packages/parametros_operacionais_v20.yaml`. Os cards que expõem esses helpers não são definidos nesse package: criar, remover, renomear ou substituir um helper não atualiza automaticamente o dashboard Storage.

Regra obrigatória de impacto: toda implementação que altere helpers utilizados pela Central Operacional deve verificar se o dashboard `Parâmetros` também precisa ser atualizado. A análise deve declarar separadamente a mudança YAML e a eventual mudança visual Storage.

O modo Storage não determina qual executor deve realizar a alteração. Não se deve presumir que HA-MCP seja obrigatório, nem que Codex seja incapaz de atuar, apenas pelo tipo de armazenamento. A escolha depende do mecanismo operacional disponível, suportado e autorizado. Enquanto não existir mecanismo automatizado homologado para dashboards Storage, a interface do Home Assistant permanece o método suportado conhecido.

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

Contrato corretivo vigente: a Central captura um snapshot de `casa_recovery_4g_max_tentativas`, controla um laço sem índice fixo e só declara sucesso após permanência contínua de `backup_4g_operacional` em `on` durante a janela configurada. Todas as tentativas usam um Tempo OFF único. Cooldown e `ultima_execucao` pertencem exclusivamente ao esgotamento completo; sucesso e cancelamentos retornam sem cooldown.

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

## V20.2C — Sessão de Monitoramento Remoto

### Classificação e estado

A Sessão de Monitoramento Remoto é um contrato operacional confirmado, distinto da presença bruta de Wilson. O subconjunto V20.2C responsável por seu ciclo de vida é promovido de forma limitada como **motor oficial de coordenação operacional**, sob o nome **Coordenador da Sessão de Monitoramento Remoto (CSMR)**.

A promoção está consolidada documentalmente, mas não autoriza implementação ou publicação em runtime. O restante da V20.2, incluindo o Context Engine original, permanece em shadow.

Decisão subordinada: `docs/arquitetura/despacho_arquitetural_v20_2c_a1.md`.

### Modelo e fronteiras

```text
person.wmoura
→ fonte contextual

binary_sensor.wilson_ausente_de_casa
→ abstração de autorização

CSMR / Sessão de Monitoramento Remoto
→ ciclo de vida, ordem e liberação dos consumidores

Publicador canônico V20.1O
→ publicação, histórico, limite, deduplicação e apresentação
```

O CSMR é a única autoridade sobre graça, revalidação, abertura, duração lógica, encerramento, cancelamento prévio, idempotência, prevenção de sessão incompleta, sequenciamento e liberação de ações subordinadas. Ele também deve definir comportamento seguro diante de restart, reload, falha crítica de publicação e rollback.

O CSMR não infere presença, não substitui a V20.1O, não mantém Timeline ou Event Feed próprios, não armazena histórico, não deduplica em paralelo e não escreve diretamente em aliases finais.

### Ciclo de vida e eventos protegidos

Entrada:

```text
Ausência confirmada após graça e revalidação
→ 📍 Wilson saiu de casa
→ 🛡️ Monitoramento remoto iniciado
→ liberar ações subordinadas
```

Encerramento:

```text
Retorno confirmado com sessão aberta
→ 📍 Wilson chegou em casa
→ 🛡️ Monitoramento remoto encerrado
```

Esses quatro textos são os únicos eventos autorizados por esta promoção. O Harness, cancelamentos, revalidações e mudanças internas não são fatos publicáveis.

### Ações subordinadas

| Lote | Consumidor atual | Momento arquitetural |
| --- | --- | --- |
| C1.1 | `light.smart_lampada_wifi_1` | após abertura confirmada |
| C1.2 | `automation.v20_2c_teste_alertar_porta_aberta_apos_saida` | após abertura confirmada |
| C1.3 | `automation.v20_2c_garantia_habilitar_push_da_porta_ao_sair` | após abertura confirmada |

Esses componentes não abrem nem encerram sessão. Módulos futuros devem declarar atuação na abertura, durante a sessão ou no encerramento.

### Propriedades do contrato

- uma única sessão ativa por vez;
- nenhum início antes da graça;
- cancelamento durante a graça não abre sessão;
- retorno sem sessão aberta não publica encerramento;
- dois eventos por transição, exatamente uma vez e na ordem definida;
- ciclos consecutivos completos e independentes;
- nenhum restart ou reload cria sessão fantasma;
- falha crítica de abertura permanece observável e impede progressão silenciosa;
- rollback preserva V20.1O e o restante da Central.

### Publicação canônica

A futura implementação somente poderá solicitar publicação pelo caminho canônico formalmente aprovado. É proibido escrever diretamente em `sensor.casa_timeline`, `sensor.casa_event_feed` ou aliases finais, criar histórico paralelo, criar outra Timeline/Event Feed, substituir `sensor.casa_evento_publicavel_v20` ou implementar deduplicação concorrente.

V20.1O permanece autoridade sobre política de publicação, armazenamento, histórico, limite, deduplicação e apresentação pública.

### Rollback arquitetural

O rollback deve retirar somente a promoção funcional da sessão e seus consumidores subordinados. V20.1O, Timeline, Event Feed, aliases, Context Engine shadow e demais componentes V20.2 permanecem inalterados. Nenhuma sessão, execução pendente ou estado inseguro do Harness pode sobreviver ao rollback.
