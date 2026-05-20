# Matriz de Testes Reais V20.2 - Fase 1A

## Objetivo

Documentar os testes reais necessários para validar a camada shadow da V20.2 antes de qualquer integração entre relevância, confiança e decisão contextual.

Esta fase valida observabilidade, coerência semântica e comportamento read-only.

## Escopo

Incluído:

- sensores determinísticos V20.1B
- contexto base V20.2
- relevância contextual V20.2 Lote 2A
- confiança/estabilidade shadow V20.2 Lote 2B

Fora de escopo:

- alterar YAML
- alterar packages
- alterar dashboards
- alterar timeline/feed
- alterar aliases finais
- criar sensores
- alterar score oficial
- publicar eventos novos
- remover legado

## Sensores Shadow Observados

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_evento_relevante_v20_2`
- `sensor.casa_motivo_relevancia_v20_2`
- `sensor.casa_confianca_contextual_v20_2`
- `sensor.casa_estabilidade_contextual_v20_2`

## Campos de Evidencia

Para cada teste, coletar:

- print ou leitura dos sensores V20.2
- estado das entidades fonte
- atributos `fontes_invalidas`
- atributos `fontes_contraditorias`
- atributo `contradicao_detectada`
- atributo `dominio_estimado`
- atributo `dominio_oscilante`
- estado da timeline/feed, quando aplicável apenas para verificar que não houve alteração indevida
- observação manual do comportamento físico

## 1. Testes Unitarios por Dominio

| ID | Evento testado | Entidades envolvidas | Pré-condição | Ação real a executar | Resultado esperado na V20/V20.2 shadow | Impacto esperado na confiança | Impacto esperado na relevância | Evidência a coletar | Status | Observações |
|---|---|---|---|---|---|---|---|---|---|---|
| U-001 | Estado normal da casa | `sensor.casa_relevancia_contextual_v20_2`, `sensor.casa_confianca_contextual_v20_2`, `sensor.casa_estabilidade_contextual_v20_2` | Sem incidentes ativos | Apenas observar por alguns minutos | Relevância `baixa`, evento `nenhum_evento_contextual`, confiança `alta`, estabilidade `estavel` | `alta`; sem fontes inválidas ou contraditórias | `baixa` | Estados e atributos dos cinco sensores shadow | Pendente | Baseline de referência |
| U-002 | Porta da sala aberta | `binary_sensor.sensor_porta_sala_contact`, `sensor.casa_porta_sala_estado_v20` | Helper de porta sala ligado | Abrir Porta da Sala | V20.1B deve registrar porta; V20.2 pode elevar se contexto indicar casa vazia | `alta` se fontes coerentes; `baixa` se houver contradição de presença | `alta` se casa vazia; caso contrário pode permanecer `baixa` | Estado da porta, contexto humano, relevância, confiança | Pendente | Validar com casa ocupada e vazia |
| U-003 | Porta da sala fechada | `binary_sensor.sensor_porta_sala_contact`, `sensor.casa_porta_sala_estado_v20` | Porta aberta | Fechar Porta da Sala | Evento contextual deve retornar a normal se não houver outro incidente | `alta` se fontes coerentes | `baixa` ou evento nenhum | Leitura antes/depois | Pendente | Verificar ausência de contradição |
| U-004 | Chuva fraca iniciada | `binary_sensor.sensor_chuva_girasol_rain`, `sensor.intensidade_da_chuva`, `sensor.casa_chuva_estado_v20` | Sem chuva | Expor/acionar sensor de chuva com intensidade fraca | Chuva ativa na V20.1B; sem janela aberta, relevância não deve virar crítica | `media` ou `alta`, domínio oscilante pode aparecer | `baixa`/`media`, conforme contexto ambiental | Estado chuva, intensidade, domínio oscilante | Pendente | Não deve depender de parsing textual |
| U-005 | Chuva forte | `sensor.intensidade_da_chuva`, `sensor.casa_chuva_estado_v20` | Chuva ativa | Alterar intensidade real para forte | V20.1B registra intensidade; V20.2 shadow observa domínio ambiental | `media` ou `alta` se coerente | Sem janela aberta, não deve ser crítica | Leitura de intensidade e sensores shadow | Pendente | Confirmar que intensidade não publica se chuva física off |
| U-006 | Energia normal | `binary_sensor.energia_concessionaria`, `sensor.casa_energia_estado_v20` | Energia presente | Observar estado normal | Contexto operacional normal ou sem criticidade | `alta` | `baixa` | Estado energia e contexto operacional | Pendente | Baseline de energia |
| U-007 | Falta de energia | `binary_sensor.energia_concessionaria`, `sensor.casa_energia_estado_v20` | Energia presente | Simular/observar falta real de concessionária | V20.1B publica falta; V20.2 pode marcar evento crítico se alguém em casa | `alta` se fontes coerentes | `critica` se casa ocupada | Estados energia, humano, relevância, confiança | Pendente | Não executar ações inseguras |
| U-008 | Retorno de energia | `binary_sensor.energia_concessionaria`, `sensor.casa_energia_estado_v20` | Energia ausente | Aguardar retorno real | V20.1B publica retorno; V20.2 deve normalizar se não houver outro incidente | `alta` | `baixa`/normal contextual | Estados antes/depois | Pendente | Verificar ausência de normalização falsa |
| U-009 | Internet normal | `sensor.internet_estado_operacional`, `sensor.casa_internet_estado_v20` | Internet normal | Observar por alguns minutos | Contexto operacional normal | `alta` | `baixa` | Estado internet e sensores shadow | Pendente | Baseline de conectividade |
| U-010 | Falha/degradação de internet | `sensor.internet_estado_operacional`, `binary_sensor.internet_wan_principal_ok`, `sensor.casa_internet_estado_v20` | Internet normal | Simular falha segura ou aguardar evento real | V20.1B publica degradação/indisponibilidade; V20.2 pode marcar média se noturno | `media` se domínio oscilante; `alta` se coerente e estável no instante | `media` se noturno; caso contrário pode ficar `baixa` | Estados WAN, internet, contexto temporal | Pendente | Não derrubar conectividade crítica sem planejamento |
| U-011 | Retorno de internet | `sensor.internet_estado_operacional`, `sensor.casa_internet_estado_v20` | Internet degradada/offline | Aguardar retorno real | V20.1B publica normalização; V20.2 retorna a baixa se sem outro incidente | `alta` | `baixa` | Sequência de estados | Pendente | Validar sem duplicidade semântica |
| U-012 | Backup Google normal | `sensor.backup_google_status`, `sensor.casa_backup_google_estado_v20` | Backup OK | Observar estado normal | Sem relevância contextual crítica | `alta` | `baixa` | Estado backup e contexto operacional | Pendente | Baseline de backup |
| U-013 | Backup Google falha | `sensor.backup_google_status`, `sensor.casa_backup_google_estado_v20` | Backup OK | Aguardar falha real ou validar em janela segura | V20.1B publica falha; V20.2 não deve virar crítica sozinha nesta fase | `alta` se fonte coerente | Sem regra mínima específica, tende a `baixa` | Estado backup e relevância | Pendente | Relevância recorrente fica para fase futura |
| U-014 | Presença/movimento | `binary_sensor.casa_presenca_global`, sensores de movimento, `binary_sensor.casa_vazia_v20_2` | Estado conhecido | Gerar movimento real em área monitorada | Contexto humano deve refletir presença | `alta` se coerente; `baixa` se presença contraditória | Impacta regras de porta/energia | Contexto humano, casa vazia, confiança | Pendente | Observar possível oscilação |
| U-015 | Banho detectado pelo box | `binary_sensor.movimento_alto_banheiro_occupancy`, `sensor.casa_banho_estado_v20` | Sem banho | Entrar no box/acionar sensor real | Banho semântico detectado; relevância não deve virar crítica por si só | `alta` se coerente; `media` se transição recente | `baixa`/`media` | Estado banho e contexto ambiental | Pendente | Validar timeout de encerramento sem mexer em YAML |
| U-016 | TV ligada | `media_player.lg_webos_tv_oled55g3psa`, `sensor.central_tv_contexto_fonte` | TV desligada | Ligar TV | V20.1B registra TV; V20.2 shadow não deve elevar criticidade | `alta` se fontes coerentes | `baixa` | Estado TV e relevância | Pendente | Confirmar contexto semântico se disponível |
| U-017 | TV desligada | `media_player.lg_webos_tv_oled55g3psa`, `sensor.central_tv_contexto_fonte` | TV ligada | Desligar TV | V20.1B registra TV desligada; V20.2 permanece sem criticidade | `alta` | `baixa` | Estado TV e sensores shadow | Pendente | Não deve gerar contradição |

## 2. Testes Integrados entre Dominios

| ID | Evento testado | Entidades envolvidas | Pré-condição | Ação real a executar | Resultado esperado na V20/V20.2 shadow | Impacto esperado na confiança | Impacto esperado na relevância | Evidência a coletar | Status | Observações |
|---|---|---|---|---|---|---|---|---|---|---|
| I-001 | Porta aberta sem ninguém em casa | `sensor.casa_porta_sala_estado_v20`, `binary_sensor.casa_vazia_v20_2`, `sensor.casa_contexto_humano_v20_2` | Casa vazia confirmada | Abrir Porta da Sala | Evento relevante `porta_aberta_casa_vazia`; relevância `alta` | `alta` se sem contradição | `alta` | Estado porta, casa vazia, evento relevante | Pendente | Um dos principais testes da V20.2 |
| I-002 | Porta aberta com alguém em casa | Mesmas fontes de I-001 | Presença confirmada | Abrir Porta da Sala | Porta registra na V20.1B, mas regra contextual de casa vazia não deve ativar | `alta` | `baixa` ou sem evento contextual | Contexto humano e relevância | Pendente | Testar diferença semântica |
| I-003 | Chuva + janela aberta | `sensor.casa_chuva_estado_v20`, sensores de janela V20, `sensor.casa_contexto_ambiental_v20_2` | Chuva ativa ou janela aberta | Abrir janela durante chuva ou iniciar chuva com janela aberta | Evento `chuva_janela_aberta`; relevância `critica` | `media`/`alta`; estabilidade `observando` por domínio oscilante | `critica` | Estado chuva, janela, ambiental, confiança | Pendente | Regra crítica principal ambiental |
| I-004 | Chuva + porta aberta | `sensor.casa_chuva_estado_v20`, `sensor.casa_porta_sala_estado_v20` | Chuva ativa | Abrir Porta da Sala | V20.1B registra ambos; V20.2 só deve virar crítica se houver janela aberta conforme regra atual | `alta` se coerente | Não crítica por esta combinação, salvo outra regra ativa | Estados porta/chuva/relevância | Pendente | Evita extrapolação indevida |
| I-005 | Energia ausente + internet offline | `sensor.casa_energia_estado_v20`, `sensor.casa_internet_estado_v20`, `sensor.casa_contexto_operacional_v20_2` | Tudo normal | Observar incidente real ou teste seguro | V20.1B registra ambos; V20.2 pode priorizar energia se alguém em casa | `alta` se coerente | `critica` se energia ausente + alguém em casa | Estados energia, internet, humano, relevância | Pendente | Sem correlação composta ainda |
| I-006 | Backup falhando durante outro incidente | `sensor.casa_backup_google_estado_v20`, outro domínio ativo | Backup OK e outro incidente ativo | Observar falha durante incidente real | V20.1B registra backup; V20.2 não deve substituir regra crítica principal nesta fase | `alta` se fontes coerentes | Mantém relevância do evento mais crítico existente | Estados backup, evento relevante, motivo | Pendente | Relevância recorrente fica para fases futuras |
| I-007 | Internet degradada + contexto noturno | `sensor.casa_internet_estado_v20`, `binary_sensor.contexto_noturno_v20_2` | Noturno confirmado | Degradar internet de forma segura ou observar evento real | Evento `internet_degradada_noturno`; relevância `media`; estabilidade `observando` | `media`, pois domínio oscilante | `media` | Contexto temporal/noturno e internet | Pendente | Regra mínima da prova de relevância |
| I-008 | Energia ausente + alguém em casa | `sensor.casa_energia_estado_v20`, `sensor.casa_contexto_humano_v20_2` | Presença confirmada | Simular/observar falta de energia | Evento `energia_ausente_casa_ocupada`; relevância `critica` | `alta` se contexto coerente | `critica` | Energia, humano, evento relevante | Pendente | Regra crítica operacional |

## 3. Testes de Regressao contra V19/status atual

| ID | Evento testado | Entidades envolvidas | Pré-condição | Ação real a executar | Resultado esperado na V20/V20.2 shadow | Impacto esperado na confiança | Impacto esperado na relevância | Evidência a coletar | Status | Observações |
|---|---|---|---|---|---|---|---|---|---|---|
| R-001 | V19 permanece desativada | Packages `_disabled`, sensores V19 | V19 desativada | Observar após reload/restart | Nenhum sensor shadow deve depender de V19 | Sem impacto | Sem impacto | Busca por `_v19` e estados shadow | Pendente | Não reativar V19 |
| R-002 | `sensor.status_casa` não muda por shadow | `sensor.status_casa`, sensores shadow | Estado normal | Observar durante testes | Shadow não deve alterar status legado/final | Sem impacto direto | Sem impacto direto | Estado antes/depois de `sensor.status_casa` | Pendente | Confirmar isolamento |
| R-003 | Timeline/feed não recebem eventos da camada shadow | `sensor.casa_timeline`, `sensor.casa_event_feed`, sensores shadow | Qualquer cenário de teste | Executar testes V20.2 | Timeline só muda por V20.1B, não por shadow | Sem impacto direto | Sem publicação nova | Linha da timeline antes/depois | Pendente | Shadow mode real |
| R-004 | Aliases finais permanecem iguais | Aliases finais V20 | Estado conhecido | Observar após testes | Nenhum alias final aponta para sensores V20.2 shadow | Sem impacto | Sem impacto | Estados e histórico dos aliases | Pendente | Não alterar dashboards |
| R-005 | Automação legada continua ativa | Automações antigas relevantes | Legado preservado | Observar evento real | Shadow não deve bloquear ou duplicar ação legada | Sem impacto | Sem impacto | Logs/notificações antigas se existirem | Pendente | V20.1C auditará decommission |

## 4. Testes de Robustez e Contradicao

| ID | Evento testado | Entidades envolvidas | Pré-condição | Ação real a executar | Resultado esperado na V20/V20.2 shadow | Impacto esperado na confiança | Impacto esperado na relevância | Evidência a coletar | Status | Observações |
|---|---|---|---|---|---|---|---|---|---|---|
| B-001 | Fonte inválida | Uma fonte V20.2 em `unknown/unavailable` | Janela de manutenção ou falha real | Observar quando fonte ficar inválida | `fontes_invalidas` lista a fonte; não deve ficar `alta` indevidamente | `indeterminada` ou `baixa` | Pode virar `indeterminada` | Atributo `fontes_invalidas` | Pendente | Não forçar estados perigosos |
| B-002 | Contradição casa vazia/presença | `binary_sensor.casa_vazia_v20_2`, `sensor.casa_contexto_humano_v20_2` | Contradição observável | Observar caso real de oscilação | `contradicao_detectada: true`; estabilidade não deve ser `estavel` | `baixa` | Pode manter ou rebaixar conforme regra | `fontes_contraditorias` | Pendente | Importante para presença |
| B-003 | Relevância sem evento | `sensor.casa_relevancia_contextual_v20_2`, `sensor.casa_evento_relevante_v20_2` | Evento nenhum | Observar se relevância subir sem evento | Deve registrar contradição se relevância `media/alta/critica` com evento nenhum | `baixa` | Não aceitar promoção | Atributos de contradição | Pendente | Detecta bug de regra |
| B-004 | Chuva ativa mas contexto ambiental normal | `sensor.casa_chuva_estado_v20`, `sensor.casa_contexto_ambiental_v20_2` | Chuva ativa | Observar contexto ambiental | Se houver janela aberta e ambiental normal, contradição deve aparecer | `baixa` | Evitar falsa crítica sem coerência | Ambiental + contradição | Pendente | Depende do contexto ambiental |
| B-005 | Internet flapping | `sensor.casa_internet_estado_v20` | Ambiente seguro | Observar transições rápidas realistas | Domínio oscilante deve ficar `observando`, não `estavel` | `media` | `media` se regra noturna ativa | Estados sequenciais | Pendente | Sem memória temporal real ainda |
| B-006 | Chuva flapping | `sensor.casa_chuva_estado_v20` | Chuva instável | Observar alternância real | Domínio ambiental deve ser tratado com cautela | `media`/`baixa` | Crítica só com janela aberta e coerência | Chuva, janela, estabilidade | Pendente | Fase futura terá debounce real |

## 5. Testes de Observabilidade e Documentacao

| ID | Evento testado | Entidades envolvidas | Pré-condição | Ação real a executar | Resultado esperado na V20/V20.2 shadow | Impacto esperado na confiança | Impacto esperado na relevância | Evidência a coletar | Status | Observações |
|---|---|---|---|---|---|---|---|---|---|---|
| O-001 | Atributos obrigatórios existem | Sensores shadow | Sensores carregados | Abrir entidades no HA | Todos os atributos documentados devem aparecer | Sem impacto | Sem impacto | Print dos atributos | Pendente | Baseline documental |
| O-002 | `motor_confianca_v20_2` não é entidade | Package e sensores | HA carregado | Buscar entidade `motor_confianca_v20_2` | Não deve existir entidade com nome do package | Sem impacto | Sem impacto | Resultado da busca | Pendente | Confirma entendimento operacional |
| O-003 | Estados ENUM simples | Sensores shadow | Cenários variados | Observar estados | Apenas ENUMs previstos devem aparecer | Sem impacto | Sem impacto | Histórico de estados | Pendente | Evita acentos/variações |
| O-004 | Documentação acompanha estado real | Docs V20.2 | Após testes | Atualizar status manualmente depois | Evidências devem bater com documentos | Sem impacto | Sem impacto | Links/prints/notas | Pendente | Não alterar YAML |
| O-005 | Rollback simples | Package shadow | Janela de manutenção planejada | Apenas planejar rollback, sem executar | Remover package deve bastar em caso futuro | Sem impacto | Sem impacto | Procedimento validado em papel | Pendente | Não remover agora |

## Critério de Saída da Fase 1A

A próxima etapa só deve ser liberada quando todos os critérios abaixo estiverem atendidos:

1. Os sensores shadow permanecem carregados após reinício/reload.
2. Estado normal da casa produz:
   - relevância `baixa`
   - evento `nenhum_evento_contextual`
   - confiança `alta`
   - estabilidade `estavel`
3. As quatro regras mínimas da relevância contextual foram validadas:
   - porta aberta + casa vazia = `alta`
   - chuva ativa + janela aberta = `critica`
   - internet degradada + noturno = `media`
   - energia ausente + alguém em casa = `critica`
4. `fontes_invalidas` fica `nenhuma` em baseline normal.
5. `fontes_contraditorias` fica `nenhuma` em baseline normal.
6. Contradições reais ou simuladas por estado operacional aparecem nos atributos corretos.
7. Domínios oscilantes ficam `observando` quando aplicável.
8. Timeline/feed não recebem eventos da camada shadow.
9. Aliases finais não apontam para sensores V20.2.
10. `sensor.status_casa` não é alterado pela camada shadow.
11. Nenhuma dependência operacional com V17/V18/V19 foi introduzida.
12. Pelo menos um teste integrado crítico foi validado com evidência.
13. O rollback permanece simples: remover/desabilitar package shadow sem afetar V20.1B.

Quando esses critérios forem atendidos, a V20.2 pode avançar para desenho da integração controlada entre relevância, confiança e decisão contextual.

