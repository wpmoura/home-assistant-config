# Inventário Legacy Migration V20.1B

## Objetivo

Mapear fontes legadas de eventos, mensagens e notificações ainda relevantes para a Central Operacional, antes de migrar para sensores determinísticos e publicadores estruturados da camada V20.

Esta etapa não altera automações, packages, dashboards, `sensor.status_casa`, V19 ou registry.

## Escopo Avaliado

- `packages/central_mensagens_corrigido.yaml`
- `packages/alertas_contextuais_v2_corrigido.yaml`
- `packages/motor_eventos_v20.yaml`
- `packages/motor_timeline_v20.yaml`
- `automations.yaml`
- blueprints locais em `blueprints/automation/wmoura/`
- documentação operacional em `docs/`

## Classificação

| Item encontrado | Domínio | Mensagem/evento gerado | Destino recomendado | Risco de duplicidade | Prioridade | Ação recomendada |
|---|---|---|---|---|---|---|
| `sensor.central_ultima_mensagem` | Central legado | Último texto publicado em `input_text.central_ultimo_evento` | Manter como fallback temporário dos aliases finais | Alto, pois recebe eventos de V2 e scripts centrais | Alta | Migrar agora para V20.1B como fonte legada observada, sem parsing como contrato principal |
| `input_text.central_ultimo_evento` | Central legado | Texto livre de último evento | Criar adaptador estruturado futuro, somente leitura | Alto | Alta | Migrar agora para camada de compatibilidade V20.1B |
| `input_text.central_ultima_notificacao` | Central legado | Última notificação enviada por V2 | Manter legado por enquanto | Médio | Média | Inventariar, mas não publicar na timeline sem deduplicação |
| `script.central_publicar_mensagem` | Central legado | Publica título, detalhe, origem e prioridade | Manter como entrada controlada do legado | Médio | Alta | Migrar agora como publicador legado estruturável |
| `script.central_limpar_mensagem` | Central legado | Limpa mensagem central e alerta ativo | Manter legado por enquanto | Baixo | Média | Preservar; futura migração para reset estruturado |
| `sensor.central_alerta_principal` | Central legado/alias interno | Estado principal textual da central antiga | Manter como fonte de `sensor.status_casa` até substituição segura | Médio | Alta | Manter legado por enquanto |
| `sensor.central_categoria_alerta` | Central legado/semântico | Categoria por origem textual e estados de internet/backup | Migrar para evento dominante/tipo estruturado | Médio | Média | Migrar depois de consolidar origem/tipo V20 |
| `sensor.central_atividade_relevante_fonte` | Central legado/atividade | Atividade relevante por sensores antigos | Manter como fonte interna até motor de contexto | Médio | Média | Manter legado por enquanto |
| `sensor.casa_timeline_eventos_v2` | Timeline V2 | Espelha `input_text.central_ultimo_evento` | Descartar/desativar futuramente | Alto | Média | Não migrar diretamente; substituir por `sensor.casa_timeline_v20` |
| `automation.alerta_contextual_porta_sala_v2` | Porta | `🚪 Porta da sala abriu durante a <período>.` | `motor_timeline_v20` já cobre porta sala aberta | Alto | Alta | Manter legado por enquanto; preparar desativação futura após validação V20 |
| `automation.alerta_contextual_chuva_inicio_v2` | Chuva | `⛈️ Chove forte agora.`, `🌧️ Chuva moderada detectada.`, `🌦️ Começou a chover.` | `motor_timeline_v20` já cobre chuva iniciada; intensidade ainda é mais rica no legado | Médio | Média | Migrar agora apenas a classificação determinística de intensidade se necessário |
| `automation.alerta_contextual_chuva_fim_v2` | Chuva | `🌤️ Parou de chover.` | `motor_timeline_v20` já cobre chuva encerrada | Alto | Média | Manter legado por enquanto; futura desativação quando timeline for fonte oficial |
| `automation.Home Office - Internet - Sem Conexão com Internet` | Internet | `Internet não responde, fluxo de dados por 4G!` | `sensor.internet_estado_operacional` e `motor_timeline_v20` | Alto | Alta | Migrar agora para evento estruturado de internet/failover; manter ação operacional de religar fibra separada |
| `automation.Home Office - Internet - Retorno da Conexão` | Internet | `Já temos internet !` | `motor_timeline_v20` já cobre internet normalizada | Alto | Alta | Migrar agora como evento de retorno; preservar notificação antiga até validação |
| `automation.Home Office - Monitorar falta de energia e desligar após 30 minutos` | Energia | Falta/retorno de energia, alertas periódicos, possível shutdown desabilitado | `binary_sensor.energia_concessionaria`, `motor_eventos_v20`, `motor_timeline_v20` | Alto | Alta | Manter legado por segurança; migrar publicação de eventos para V20.1B sem alterar lógica de proteção |
| `automation.Quarto - Está chovendo` | Chuva | `☔️ Chovendo agora!` | `motor_timeline_v20` | Alto | Média | Manter legado por enquanto; candidato a desativação futura |
| `automation.Quarto - Parou de chover` | Chuva | `☂️ Sem chuva, agora !` | `motor_timeline_v20` | Alto | Média | Manter legado por enquanto; candidato a desativação futura |
| `automation.Quarto - Notificação intensidade de Chuva` | Chuva | `⛈️ Chove forte agora!` | Futuro `motor_contexto_v20` ou extensão de chuva V20 | Médio | Baixa | Manter legado por enquanto; migrar depois se intensidade entrar no contrato |
| `automation.Sala - Porta da sala abriu` | Porta | `⚠️ Porta da Sala abriu !` | `motor_timeline_v20` já cobre porta sala aberta | Alto | Alta | Manter legado por enquanto; candidato a desativação após validação de push V20 |
| `automation.LAB - Alarme - Dispara alarme casa` | Segurança | Dispara alarme se porta sala abre com alarme armado | Fora do feed operacional comum | Baixo | Alta | Manter legado; não migrar para timeline sem motor de segurança |
| `automation.LAB - Alarme - Alarme disparado` | Segurança | `Alarme disparado! Verifique sua casa !` e sirene/áudio | Futuro motor de segurança, não V20.1B | Médio | Alta | Manter legado por enquanto |
| `automation.Corredor - Gestão Luz Segurança versus portas` | Portas internas | Acende luz com portas internas à noite | `motor_timeline_v20` já tem publicação opcional para portas internas | Baixo | Média | Manter legado; não migrar automação de luz, apenas eventos se helper estiver ativo |
| `blueprints/automation/wmoura/luz_seguranca_portas_noite.yaml` | Portas internas/iluminação | Gestão de luz por portas | Fora da timeline textual | Baixo | Baixa | Manter legado por enquanto |
| `blueprints/automation/wmoura/Gestão de Energia - UPS.yaml` | Energia/UPS | Falta de energia, retorno, alertas periódicos e logbook | `motor_eventos_v20` já classifica energia/UPS parcialmente | Alto | Alta | Migrar agora só publicação determinística; preservar ação de proteção |
| `blueprints/automation/wmoura/Pós-retorno de energia.yaml` | Energia | Alertas de tomadas críticas após retorno | Futuro motor de energia/infraestrutura | Médio | Média | Manter legado por enquanto |
| `blueprints/automation/wmoura/wan_4G_AppleWatch_Energia.yaml` | Failover/4G | 4G offline, tentativa de recovery, retorno/falha crítica | `motor_timeline_v20` cobre failover; recovery deve permanecer blueprint | Médio | Alta | Migrar agora eventos finais, manter recovery legado |
| `blueprints/automation/wmoura/wan_4G_AppleWatch.yaml` | Failover/4G | Alertas de 4G e recovery | `motor_timeline_v20` e futuro contrato WAN/4G sem versão | Médio | Média | Manter legado por enquanto |
| `blueprints/automation/wmoura/Alerta_Vazamento_com_Led.yaml` | Vazamento | `💦 Vazamento detectado...`, `😌 ... cessou` | `motor_timeline_v20` já tem vazamento opcional desligado por padrão | Médio | Média | Migrar depois de confirmar sensor real e política de publicação |
| `packages/cerebro_backup_formatado.yaml` | Vazamento/contexto | Usa `binary_sensor.vazamento` para composição semântica | Futuro motor de contexto | Médio | Baixa | Manter legado por enquanto |
| `packages/ha_inicio.yaml` | Sistema | Notificação de início do HA | Fora do feed operacional de incidentes | Baixo | Baixa | Manter legado |
| `packages/modo_dormir.yaml` | Contexto humano | Notificações de modo dormir | Futuro motor de contexto humano | Baixo | Baixa | Manter legado por enquanto |
| `packages/carro_presenca.yaml` | Presença/carro | Notificações diretas de presença/veículo | Futuro motor de presença V20/V21 | Baixo | Baixa | Manter legado por enquanto |
| `blueprints/automation/wmoura/monitoramento_energia_v6.yaml` | Energia/equipamentos | Alertas de potência, tensão e corrente de equipamentos | Futuro motor de infraestrutura/equipamentos | Médio | Baixa | Manter legado por enquanto |
| `packages/_disabled/status_casa_v19.yaml` | V19 desativado | Sensores e eventos V19 | Não usar | Alto | Alta | Preservar desativado; não migrar direto |

## Eventos Já Migrados Para V20

| Evento | Fonte V20 atual | Observação |
|---|---|---|
| Porta sala aberta | `sensor.casa_porta_sala_estado_v20` em `motor_timeline_v20.yaml` | Publicação controlada por `input_boolean.casa_timeline_evento_porta_sala` |
| Porta sala fechada | `sensor.casa_porta_sala_estado_v20` em `motor_timeline_v20.yaml` | Publicação controlada por helper |
| Porta sala aberta por timeout | `input_number.casa_timeout_porta_aberta_minutos` + porta sala | Timeout permanece exclusivo da Porta Sala |
| Chuva iniciada | `binary_sensor.sensor_chuva_girasol_rain` e `sensor.casa_chuva_estado_v20` | Publicação controlada por helper; ignora restauração inicial |
| Chuva encerrada | `binary_sensor.sensor_chuva_girasol_rain` e `sensor.casa_chuva_estado_v20` | Publicação controlada por helper; transição física `on` -> `off` |
| Intensidade da chuva | `sensor.intensidade_da_chuva` | Publica apenas com chuva ativa e mudança real de intensidade |
| Falta de energia | `sensor.casa_energia_estado_v20` | Publicação controlada por helper |
| Energia restabelecida | `sensor.casa_energia_estado_v20` | Publicação controlada por helper |
| Internet degradada | `sensor.casa_internet_estado_v20` | Publicação controlada por helper |
| Internet indisponível | `sensor.casa_internet_estado_v20` | Publicação controlada por helper |
| Internet normalizada | `sensor.casa_internet_estado_v20` | Publicação controlada por helper |
| Failover 4G ativado | `sensor.casa_failover_estado_v20` | Publicação controlada por helper |
| Failover 4G encerrado | `sensor.casa_failover_estado_v20` | Publicação controlada por helper |
| Backup Google com falha | `sensor.casa_backup_google_estado_v20` | Publicação controlada por helper |
| Backup Google normalizado | `sensor.casa_backup_google_estado_v20` | Publicação controlada por helper |
| TV ligada/desligada/contexto | `media_player.lg_webos_tv_oled55g3psa` e `sensor.central_tv_contexto_fonte` | Publicação controlada por helper |
| Portas internas | `sensor.casa_porta_banheiro_estado_v20`, `sensor.casa_porta_quarto_menor_estado_v20`, `sensor.casa_porta_quarto_maior_estado_v20` | Opcional, desligado por padrão |
| Janelas/contatos | `sensor.casa_janela_varanda_sala_estado_v20`, `sensor.casa_janela_quarto_maior_estado_v20`, `sensor.casa_janela_quarto_menor_estado_v20` | Opcional, desligado por padrão |
| Vazamento | `sensor.casa_vazamento_estado_v20` | Opcional, desligado por padrão |

## Lote 2 Implementado - Energia, Internet, Failover 4G e Backup Google

Data de referência: 2026-05-14.

Implementação realizada apenas na camada V20/paralela, sem alterar automações antigas, blueprints, dashboards produtivos, V19, Git ou `sensor.status_casa`.

### Fontes Reais Inventariadas

| Domínio | Fontes reais/operacionais | Observação |
|---|---|---|
| Energia | `binary_sensor.energia_concessionaria`, `sensor.ups_voltagem`, `sensor.ups_potencia` | `binary_sensor.energia_concessionaria` é derivado de leitura da UPS e permanece fonte real da presença da concessionária. UPS fica como atributo de contexto neste lote. |
| Internet | `sensor.internet_estado_operacional`, `binary_sensor.internet_wan_principal_ok`, `binary_sensor.internet_wan2_4g_ok`, `sensor.casa_internet_estado_semantico_v20` | `sensor.internet_estado_operacional` segue como fonte operacional viva; WAN V20 entra como fonte complementar quando disponível. |
| Failover 4G | `binary_sensor.internet_em_failover_4g`, `sensor.internet_estado_operacional`, `binary_sensor.casa_failover_ativo_v20` | O sensor V20 normaliza o estado final para `ativo`/`inativo`. |
| Backup Google | `sensor.backup_google_status`, `sensor.backup_state` | `sensor.backup_google_status` já normaliza o add-on; V20.1B cria camada determinística final para timeline. |

### Sensores Determinísticos Criados

| Sensor | Domínio | Fontes reais | Estado normalizado | Helper respeitado |
|---|---|---|---|---|
| `sensor.casa_energia_estado_v20` | energia | `binary_sensor.energia_concessionaria`, `sensor.ups_voltagem`, `sensor.ups_potencia` | `presente`, `ausente`, `desconhecida` | `input_boolean.casa_timeline_evento_energia` |
| `sensor.casa_internet_estado_v20` | internet | `sensor.internet_estado_operacional`, `sensor.casa_internet_estado_semantico_v20` | `normal`, `degradada`, `indisponivel`, `desconhecida` | `input_boolean.casa_timeline_evento_internet` |
| `sensor.casa_failover_estado_v20` | failover 4G | `binary_sensor.internet_em_failover_4g`, `sensor.internet_estado_operacional`, `binary_sensor.casa_failover_ativo_v20` | `ativo`, `inativo`, `desconhecido` | `input_boolean.casa_timeline_evento_failover` |
| `sensor.casa_backup_google_estado_v20` | backup Google | `sensor.backup_google_status`, `sensor.backup_state` | `ok`, `falha`, `desconhecido` | `input_boolean.casa_timeline_evento_backup_google` |

### Eventos Publicados

| Evento | Transição determinística | Regra anti-startup |
|---|---|---|
| `⚡ Falta de energia` | `presente` -> `ausente` | Não publica `unknown/desconhecida` -> `ausente` |
| `⚡ Energia restabelecida` | `ausente` -> `presente` | Não publica `unknown/desconhecida` -> `presente` |
| `🌐 Internet degradada` | `normal`/`indisponivel` -> `degradada` | Não publica estado inicial restaurado |
| `🌐 Internet indisponível` | `normal`/`degradada` -> `indisponivel` | Não publica estado inicial restaurado |
| `🌐 Internet normalizada` | `degradada`/`indisponivel` -> `normal` | Não publica estado inicial restaurado |
| `📡 Failover 4G ativado` | `inativo` -> `ativo` | Não publica estado inicial restaurado |
| `📡 Failover 4G encerrado` | `ativo` -> `inativo` | Não publica estado inicial restaurado |
| `⚠️ Backup Google com falha` | `ok` -> `falha` | Não publica `desconhecido` -> `falha` |
| `✅ Backup Google normalizado` | `falha` -> `ok` | Não publica `desconhecido` -> `ok` |

### Itens Legados Preservados

- Automações antigas de internet, energia, failover e backup permanecem ativas.
- Blueprints de recovery/segurança permanecem inalterados.
- A timeline V20 não depende de `sensor.central_ultima_mensagem` nem de parsing textual para esses domínios.
- Possível duplicidade externa ainda pode ocorrer por notificações/push legados, mas a timeline V20 agora usa contratos determinísticos próprios.

### Correção de Prioridade - Internet

Correção aplicada após validação do lote 2:

- `sensor.casa_internet_estado_v20` não deve dar prioridade a `sensor.casa_internet_estado_semantico_v20` quando as fontes primárias indicam normalidade.
- Regra consolidada:
  - se `sensor.internet_estado_operacional = normal` e `binary_sensor.internet_wan_principal_ok = on`, o estado final deve ser `normal`.
  - `binary_sensor.internet_wan2_4g_ok = on` indica redundância ativa, mas não degradação.
  - `sensor.casa_internet_estado_semantico_v20` é apenas fonte auxiliar/fallback quando as fontes primárias estão indisponíveis ou inconclusivas.
- Essa correção evita falso `indisponivel` quando o WAN engine V20 auxiliar está desalinhado, mas a internet operacional real está normal.

### Correção da Máquina de Transição - Internet Normalizada

Correção aplicada após validação de recuperação completa:

- `sensor.casa_internet_estado_v20` passou a observar também `binary_sensor.internet_em_failover_4g`.
- Enquanto `binary_sensor.internet_em_failover_4g = on`, a internet permanece semanticamente `degradada`, mesmo que a WAN principal já tenha voltado.
- O estado `normal` só é assumido quando:
  - `sensor.internet_estado_operacional = normal`
  - `binary_sensor.internet_wan_principal_ok = on`
  - `binary_sensor.internet_em_failover_4g` não está `on`
- Isso preserva a sequência operacional:
  - `📡 Failover 4G ativado`
  - `🌐 Internet degradada`
  - `📡 Failover 4G encerrado`
  - `🌐 Internet normalizada`
- A publicação continua condicionada à transição real `degradada/indisponivel -> normal`, evitando normalização falsa em startup/reload.

### Débito Técnico - Ordenação Semântica Internet/Failover

Status: registrado para V20.2 ou V20.3.

Não é bug funcional. A recuperação ocorre corretamente.

Cenário observado:

- Durante recuperação completa, a timeline pode publicar:
  - `🌐 Internet normalizada`
  - `📡 Failover 4G encerrado`

Ordem semanticamente desejada:

- `📡 Failover 4G encerrado`
- `🌐 Internet normalizada`

Classificação:

- Refinamento de máquina de estados.
- Ordenação de eventos correlacionados.
- Futuro agrupamento contextual de recuperação operacional.

Decisão:

- Não alterar V20.1B lote 2 agora.
- Resolver futuramente junto com correlação contextual e agrupamento de eventos.

## Lote 1 Implementado - Porta, Chuva e Vazamento

Data de referência: 2026-05-14.

Implementação realizada apenas na camada V20/paralela, sem alterar automações antigas, blueprints, dashboards, V19 ou `sensor.status_casa`.

### Sensores Determinísticos Criados

| Sensor | Domínio | Fontes reais | Estado normalizado | Helper respeitado |
|---|---|---|---|---|
| `sensor.casa_porta_sala_estado_v20` | porta | `binary_sensor.sensor_porta_sala_contact` | `aberta`, `fechada`, `desconhecida` | `input_boolean.casa_timeline_evento_porta_sala` |
| `sensor.casa_porta_banheiro_estado_v20` | porta interna | `binary_sensor.sensor_porta_banheiro_contact` | `aberta`, `fechada`, `desconhecida` | `input_boolean.casa_timeline_evento_portas_internas` |
| `sensor.casa_porta_quarto_menor_estado_v20` | porta interna | `binary_sensor.sensor_porta_sec_quarto_contact` | `aberta`, `fechada`, `desconhecida` | `input_boolean.casa_timeline_evento_portas_internas` |
| `sensor.casa_porta_quarto_maior_estado_v20` | porta interna | `binary_sensor.0x00158d0006b0a28b_contact` | `aberta`, `fechada`, `desconhecida` | `input_boolean.casa_timeline_evento_portas_internas` |
| `sensor.casa_janela_varanda_sala_estado_v20` | janela/contato | `binary_sensor.sensor_varanda_sala_contact` | `aberta`, `fechada`, `desconhecida` | `input_boolean.casa_timeline_evento_janelas` |
| `sensor.casa_janela_quarto_maior_estado_v20` | janela | `binary_sensor.sensor_janela_quarto_maior_contact` | `aberta`, `fechada`, `desconhecida` | `input_boolean.casa_timeline_evento_janelas` |
| `sensor.casa_janela_quarto_menor_estado_v20` | janela | `binary_sensor.sensor_janela_quarto_menor_contact` | `aberta`, `fechada`, `desconhecida` | `input_boolean.casa_timeline_evento_janelas` |
| `sensor.casa_chuva_estado_v20` | chuva | `binary_sensor.sensor_chuva_girasol_rain`, `sensor.intensidade_da_chuva` | `ativa`, `inativa`, `desconhecida` | `input_boolean.casa_timeline_evento_chuva` |
| `sensor.casa_vazamento_estado_v20` | vazamento | `binary_sensor.sensor_vazmento_agua_water_leak`, `binary_sensor.vazamento` | `detectado`, `normal`, `desconhecido` | `input_boolean.casa_timeline_evento_vazamento` |
| `sensor.casa_banho_estado_v20` | contexto humano / banho | `binary_sensor.movimento_alto_banheiro_occupancy` | `banho_detectado`, `sem_banho`, `desconhecido` | `input_boolean.casa_timeline_evento_banho`, `input_number.casa_timeout_banho_minutos` |

### Extensão Semântica - Banho

Correção adicionada ao escopo V20.1B para tratar o sensor do box/chuveiro como evento semântico humano, não como movimento genérico.

Entidade real identificada:

- `binary_sensor.movimento_alto_banheiro_occupancy`

Evidências encontradas:

- Grupo legado `grp_movimento_banheiro` combina `binary_sensor.movimento_banheiro_occupancy` e `binary_sensor.movimento_alto_banheiro_occupancy`.
- Dashboards legados exibem `binary_sensor.movimento_alto_banheiro_occupancy` como sensor do `Box`.
- Roadmap semântico já classificava `box ativo` como `banho`.

Implementação V20.1B:

- Criado `sensor.casa_banho_estado_v20`.
- Criado helper `input_boolean.casa_timeline_evento_banho`.
- Criado helper `input_number.casa_timeout_banho_minutos`.
- Helper inicia desligado por padrão por ser um evento humano/privado novo na timeline operacional.
- O encerramento do banho possui tolerância parametrizável, com default inicial de 3 minutos.
- Publicação respeita apenas transições reais e estabilizadas:
  - `sem_banho` -> `banho_detectado`: `🚿 Banho iniciado`
  - `banho_detectado` -> `sem_banho`, somente após o sensor do box permanecer `off` por `input_number.casa_timeout_banho_minutos`: `🚿 Banho encerrado`
- Se `binary_sensor.movimento_alto_banheiro_occupancy` voltar para `on` antes do timeout, o encerramento é cancelado pelo próprio gatilho com `for:`.
- Estados `unknown`, `unavailable` e restauração inicial não publicam evento.
- O sensor não entra em motores de movimento genérico.
- O painel administrativo de parâmetros operacionais passou a expor o timeout de banho para calibração sem edição de YAML.

Nota arquitetural para V20.2:

- Na V20.1B, o horário publicado em `🚿 Banho encerrado` representa o momento de confirmação após o timeout.
- Essa decisão permanece válida porque o evento semântico indica confirmação com confiança.
- Refinamento futuro previsto em `V20.2 - Semantic Timeline Refinement`:
  - `timestamp_ocorrencia`: momento em que o sensor físico ficou `off`.
  - `timestamp_confirmacao`: momento em que o motor confirmou o encerramento após `input_number.casa_timeout_banho_minutos`.
  - A timeline poderá exibir o horário de ocorrência e manter o horário de confirmação como atributo interno.

### Correção de Nomenclatura - Quartos e Janelas

Correção aplicada após validação do lote 1:

| Fonte real | Nome operacional V20 | Sensor determinístico | Mensagens publicadas |
|---|---|---|---|
| `binary_sensor.sensor_porta_sec_quarto_contact` | Porta Quarto Menor | `sensor.casa_porta_quarto_menor_estado_v20` | `🚪 Porta Quarto Menor aberta`, `🚪 Porta Quarto Menor fechada` |
| `binary_sensor.0x00158d0006b0a28b_contact` | Porta Quarto Maior | `sensor.casa_porta_quarto_maior_estado_v20` | `🚪 Porta Quarto Maior aberta`, `🚪 Porta Quarto Maior fechada` |
| `binary_sensor.sensor_janela_quarto_maior_contact` | Janela Quarto Maior | `sensor.casa_janela_quarto_maior_estado_v20` | `🪟 Janela Quarto Maior aberta`, `🪟 Janela Quarto Maior fechada` |
| `binary_sensor.sensor_janela_quarto_menor_contact` | Janela Quarto Menor | `sensor.casa_janela_quarto_menor_estado_v20` | `🪟 Janela Quarto Menor aberta`, `🪟 Janela Quarto Menor fechada` |

`sensor.casa_janela_varanda_sala_estado_v20` foi preservado sem alteração funcional.

### Correção do Rain Engine

Correção aplicada após validação do lote 1:

- `binary_sensor.sensor_chuva_girasol_rain` voltou a ser gatilho explícito de publicação.
- `sensor.intensidade_da_chuva` passou a ser gatilho explícito de mudança de intensidade.
- `sensor.casa_chuva_estado_v20` permanece como camada determinística.
- Eventos de chuva só publicam se `input_boolean.casa_timeline_evento_chuva` não estiver `off`.
- O início da chuva exige transição física real `off` -> `on`, evitando falso positivo em startup/reload.
- O encerramento da chuva exige transição física real `on` -> `off`.
- Mudança de intensidade só publica quando:
  - `binary_sensor.sensor_chuva_girasol_rain = on`
  - `sensor.casa_chuva_estado_v20 = ativa`
  - valor novo e anterior são válidos
  - valor novo é diferente do anterior
  - valor novo não é `Sem chuva`

Correção adicional aplicada após teste de falso positivo:

- `sensor.casa_chuva_estado_v20` passou a tratar o sensor físico `binary_sensor.sensor_chuva_girasol_rain` como autoridade para `ativa`/`inativa`.
- `sensor.intensidade_da_chuva` permanece apenas como atributo complementar de chuva ativa.
- Mudança de intensidade com `binary_sensor.sensor_chuva_girasol_rain = off` é ignorada completamente, mesmo que a intensidade restaurada esteja como `Forte`, `Moderada` ou `Fraca`.
- Os blocos de publicação por estado determinístico também exigem coerência com o sensor físico antes de gerar `Chuva iniciada` ou `Chuva encerrada`.

Mensagens publicadas:

- `🌧️ Chuva iniciada`
- `🌧️ Chuva iniciada: <intensidade>`
- `🌧️ Intensidade da chuva: <valor>`
- `🌤️ Chuva encerrada`

### Publicação V20 Atualizada

O `sensor.casa_evento_publicavel_v20` passou a publicar porta, chuva, janelas/contatos e vazamento a partir dos sensores determinísticos acima.

O timeout de Porta Sala continua usando diretamente `binary_sensor.sensor_porta_sala_contact`, porque o gatilho com `for:` precisa observar o sensor físico da porta aberta pelo tempo configurado.

### Itens do Inventário Atendidos no Lote 1

| Item do inventário | Resultado do lote 1 |
|---|---|
| `automation.alerta_contextual_porta_sala_v2` | Mantida, mas a timeline V20 não depende mais dela |
| `automation.Sala - Porta da sala abriu` | Mantida, mas a timeline V20 não depende mais dela |
| `automation.alerta_contextual_chuva_inicio_v2` | Mantida, mas a timeline V20 usa `sensor.casa_chuva_estado_v20` |
| `automation.alerta_contextual_chuva_fim_v2` | Mantida, mas a timeline V20 usa `sensor.casa_chuva_estado_v20` |
| `automation.Quarto - Está chovendo` | Mantida, mas duplicidade pode ser removida futuramente |
| `automation.Quarto - Parou de chover` | Mantida, mas duplicidade pode ser removida futuramente |
| `blueprints/automation/wmoura/Alerta_Vazamento_com_Led.yaml` | Mantido, mas a timeline V20 usa `sensor.casa_vazamento_estado_v20` quando o helper de vazamento estiver ligado |

### O Que Continua Legado

- Pushes/notificações antigas continuam ativos.
- Blueprints antigos continuam ativos.
- `sensor.central_ultima_mensagem` continua como fallback histórico dos aliases finais.
- Vazamento permanece desligado por padrão na timeline, respeitando a decisão conservadora da V20.1A.

## Migrar Agora Para V20.1B

Prioridade para a próxima etapa de implementação:

1. Criar camada de observação legado → V20 para `input_text.central_ultimo_evento`, sem depender dele como contrato final.
2. Criar sensores determinísticos para eventos legados de internet, energia, backup e porta que ainda publicam mensagens diretas.
3. Mapear `script.central_publicar_mensagem` como publicador legado estruturado, usando campos `prioridade`, `titulo`, `detalhe` e `origem`.
4. Criar deduplicação entre publicações legadas e `motor_timeline_v20`, evitando que a mesma porta/chuva/internet apareça duas vezes.
5. Preservar notificações diretas até existir um publicador V20 validado para push/voz/logbook.

## Manter Legado Por Enquanto

- Automação de desligamento/proteção por falta de energia.
- Blueprints de recovery 4G/modem.
- Blueprints de energia de equipamentos.
- Alarmes e segurança.
- Modo dormir.
- Presença/carro.
- Notificações de laboratório/rotinas fora da Central Operacional.

## Descartar ou Desativar Futuramente

Após validação completa da V20.1B e futura V20.2:

- `sensor.casa_timeline_eventos_v2`
- automações contextuais V2 de porta/chuva que só duplicam timeline
- notificações diretas simples de porta e chuva, se substituídas por publicador V20
- uso direto de `sensor.central_ultima_mensagem` em dashboards, mantendo apenas como fallback histórico

## Riscos Principais

| Risco | Impacto | Mitigação recomendada |
|---|---|---|
| Duplicidade de porta sala | Timeline e push repetidos | Deduplicar por domínio + tipo + janela curta de tempo |
| Duplicidade de chuva | Eventos de chuva por binário e intensidade | Normalizar origem preferencial e publicar uma vez por transição |
| Energia com automações críticas | Quebra de proteção da casa | Não alterar automações de proteção; migrar somente publicação |
| 4G/recovery misturado com timeline | Eventos operacionais e ações de recovery confundidos | Separar evento publicado de ação corretiva |
| `central_ultimo_evento` textual | Parsing frágil | Tratar como compatibilidade, não como fonte canônica |
| V19 desativado | Reintrodução acidental | Não referenciar sensores `_v19` |

## Próxima Ação Recomendada

Implementar a V20.1B em um package novo e paralelo, sem alterar automações antigas:

- `packages/legacy_migration_v20_1.yaml`

Esse package deve criar sensores de compatibilidade e diagnóstico, como:

- sensor de último evento legado observado
- sensor de domínio legado inferido por origem estruturada quando disponível
- binary sensor indicando possível duplicidade com timeline V20
- atributos de origem, prioridade, timestamp e ação recomendada

Depois disso, a migração de notificações diretas deve ser feita por domínio, começando por porta, chuva, internet e energia.
