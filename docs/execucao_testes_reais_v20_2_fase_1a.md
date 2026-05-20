# Execução de Testes Reais V20.2 - Fase 1A

## Objetivo

Preparar e registrar a execução real da Fase 1A da V20.2 em modo shadow sem alterar a produção.

## Regras da execução

- Não alterar YAML
- Não alterar packages
- Não alterar dashboards
- Não alterar aliases finais
- Não alterar automações existentes
- Não remover legado
- Não reativar V19
- Registrar evidência antes de avançar para o próximo teste

## Sensores observados

- `sensor.casa_relevancia_contextual_v20_2`
- `sensor.casa_evento_relevante_v20_2`
- `sensor.casa_motivo_relevancia_v20_2`
- `sensor.casa_confianca_contextual_v20_2`
- `sensor.casa_estabilidade_contextual_v20_2`

## Modelo de checklist de execução

Cada teste deve incluir:

- Objetivo
- Entidades envolvidas
- Ação física real
- Resultado esperado
- Evidência
- Status

### Escopo da Fase A

Os testes de Fase A incluem:
- U-001
- R-001 a R-005
- U-006 a U-013
- U-016 a U-017

## 1. Testes Unitários por Domínio

| ID | Objetivo | Entidades envolvidas | Ação física real | Resultado esperado | Evidência | Status |
|---|---|---|---|---|---|---|
| U-001 | Validar baseline com casa normal | `sensor.casa_relevancia_contextual_v20_2`, `sensor.casa_confianca_contextual_v20_2`, `sensor.casa_estabilidade_contextual_v20_2` | Observar estado normal sem incidentes por alguns minutos | Relevância `baixa`, evento `nenhum_evento_contextual`, confiança `alta`, estabilidade `estavel` | Leitura dos sensores e atributos | Bloqueado |
| U-002 | Validar porta da sala aberta | `binary_sensor.sensor_porta_sala_contact`, `sensor.casa_porta_sala_estado_v20`, contexto humano | Abrir a Porta da Sala fisicamente | V20.1B registra porta; V20.2 pode elevar relevância para `alta` se casa vazia | Estado da porta e sensores de presença/contexto | Pendente |
| U-003 | Validar porta da sala fechada | `binary_sensor.sensor_porta_sala_contact`, `sensor.casa_porta_sala_estado_v20` | Fechar a Porta da Sala fisicamente | Evento contextual retorna ao normal se não houver outro incidente | Leitura antes/depois da porta | Pendente |
| U-004 | Validar chuva fraca | `binary_sensor.sensor_chuva_girasol_rain`, `sensor.intensidade_da_chuva`, `sensor.casa_chuva_estado_v20` | Gerar chuva fraca real ou simular sensor de chuva sem janela aberta | Chuva ativa em V20.1B; relevância não vira crítica sem janela aberta | Intensidade e estado de chuva | OK |
| U-005 | Validar chuva forte | `sensor.intensidade_da_chuva`, `sensor.casa_chuva_estado_v20` | Aumentar intensidade de chuva real ou simular chuva forte | V20.1B registra intensidade; V20.2 observa domínio ambiental; não crítico sem janela aberta | Intensidade, estado de chuva e sensores shadow | OK |
| U-006 | Validar energia normal | `binary_sensor.energia_concessionaria`, `sensor.casa_energia_estado_v20` | Inspecionar energia de concessionária normal | Contexto operacional normal; relevância baixa | Estado de energia e contexto operacional | OK |
| U-007 | Validar falta de energia | `binary_sensor.energia_concessionaria`, `sensor.casa_energia_estado_v20`, contexto humano | Simular ou observar falta real de energia | V20.1B publica falta; V20.2 pode marcar crítico se alguém em casa | Estado de energia, presença e sensores de relevância | Pendente |
| U-008 | Validar retorno de energia | `binary_sensor.energia_concessionaria`, `sensor.casa_energia_estado_v20` | Aguardar o retorno real de energia | V20.1B publica retorno; V20.2 normaliza se não houver outro incidente | Estados antes/depois de energia | Pendente |
| U-009 | Validar internet normal | `sensor.internet_estado_operacional`, `sensor.casa_internet_estado_v20` | Observar internet normal por alguns minutos | Contexto operacional normal; relevância baixa | Estado de internet e sensores shadow | OK |
| U-010 | Validar degradação de internet | `sensor.internet_estado_operacional`, `binary_sensor.internet_wan_principal_ok`, `sensor.casa_internet_estado_v20` | Simular falha segura ou observar degradação real | V20.1B publica degradação; V20.2 pode marcar média se noturno | Estados WAN, internet e contexto temporal | Pendente |
| U-011 | Validar retorno de internet | `sensor.internet_estado_operacional`, `sensor.casa_internet_estado_v20` | Aguardar internet retornar real | V20.1B publica normalização; V20.2 retorna para baixa se não houver outro incidente | Sequência de estados antes/depois | Pendente |
| U-012 | Validar backup Google normal | `sensor.backup_google_status`, `sensor.casa_backup_google_estado_v20` | Observar backup normal | Sem relevância contextual crítica | Estado de backup e contexto operacional | Pendente |
| U-013 | Validar falha de backup Google | `sensor.backup_google_status`, `sensor.casa_backup_google_estado_v20` | Aguardar falha real ou observar janela de falha | V20.1B publica falha; V20.2 não vira crítica sozinha nesta fase | Estado de backup e relevância | Pendente |
| U-014 | Validar presença/movimento | `binary_sensor.casa_presenca_global`, sensores de movimento, `binary_sensor.casa_vazia_v20_2` | Gerar movimento real em área monitorada | Contexto humano reflete presença | Estado de presença e confiança | Pendente |
| U-015 | Validar banho detectado pelo box | `binary_sensor.movimento_alto_banheiro_occupancy`, `sensor.casa_banho_estado_v20` | Entrar no box ou acionar o sensor real | Banho semântico detectado; relevância não vira crítica por si só | Estado de banho e contexto ambiental | Pendente |
| U-016 | Validar TV ligada | `media_player.lg_webos_tv_oled55g3psa`, `sensor.central_tv_contexto_fonte` | Ligar a TV fisicamente | V20.1B registra TV; V20.2 shadow não eleva criticidade | Estado da TV e relevância | OK |
| U-017 | Validar TV desligada | `media_player.lg_webos_tv_oled55g3psa`, `sensor.central_tv_contexto_fonte` | Desligar a TV fisicamente | V20.1B registra TV desligada; V20.2 permanece sem criticidade | Estado da TV e sensores shadow | Bloqueado |

## 2. Testes Integrados entre Domínios

| ID | Objetivo | Entidades envolvidas | Ação física real | Resultado esperado | Evidência | Status |
|---|---|---|---|---|---|---|
| I-001 | Validar porta aberta sem ninguém em casa | `sensor.casa_porta_sala_estado_v20`, `binary_sensor.casa_vazia_v20_2`, `sensor.casa_contexto_humano_v20_2` | Abrir a Porta da Sala com casa vazia | Evento `porta_aberta_casa_vazia`; relevância `alta` | Estado da porta, casa vazia e sensores relevantes | Pendente |
| I-002 | Validar porta aberta com alguém em casa | mesmas entidades de I-001 | Abrir a Porta da Sala com presença confirmada | V20.1B registra porta; regra de casa vazia não ativa | Contexto humano e relevância | Pendente |
| I-003 | Validar chuva + janela aberta | `sensor.casa_chuva_estado_v20`, sensores de janela V20, `sensor.casa_contexto_ambiental_v20_2` | Abrir janela durante chuva real | Evento `chuva_janela_aberta`; relevância `critica`; estabilidade `observando` | Estado chuva, janela e atributos de confiança | Pendente |
| I-004 | Validar chuva + porta aberta | `sensor.casa_chuva_estado_v20`, `sensor.casa_porta_sala_estado_v20` | Abrir Porta da Sala durante chuva | V20.1B registra ambos; V20.2 só vira crítica se houver janela aberta | Estado de chuva, porta e relevância | Pendente |
| I-005 | Validar energia ausente + internet offline | `sensor.casa_energia_estado_v20`, `sensor.casa_internet_estado_v20`, `sensor.casa_contexto_operacional_v20_2` | Observar ambos os incidentes reais | V20.1B registra ambos; V20.2 pode priorizar energia se há alguém em casa | Estados de energia, internet e contexto humano | Pendente |
| I-006 | Validar backup falhando durante outro incidente | `sensor.casa_backup_google_estado_v20`, outro domínio ativo | Observar falha de backup enquanto outro incidente está ativo | Backup não substitui regra crítica principal nesta fase | Estados de backup e outros eventos | Pendente |
| I-007 | Validar internet degradada à noite | `sensor.casa_internet_estado_v20`, `binary_sensor.contexto_noturno_v20_2` | Observar degradação de internet durante o período noturno | Evento `internet_degradada_noturno`; relevância `media`; estabilidade `observando` | Estado de internet e contexto noturno | Pendente |
| I-008 | Validar energia ausente com alguém em casa | `sensor.casa_energia_estado_v20`, `sensor.casa_contexto_humano_v20_2` | Observar falta de energia com presença confirmada | Evento `energia_ausente_casa_ocupada`; relevância `critica` | Estado de energia e presença | Pendente |

## 3. Testes de Regressão contra V19/status atual

| ID | Objetivo | Entidades envolvidas | Ação física real | Resultado esperado | Evidência | Status |
|---|---|---|---|---|---|---|
| R-001 | Validar que V19 permanece desativada | packages `_disabled`, sensores V19 | Inspecionar após reload/restart | Nenhum sensor shadow depende de V19 | Busca por `_v19` e estados shadow | Pendente |
| R-002 | Validar que `sensor.status_casa` não muda por shadow | `sensor.status_casa`, sensores shadow | Observar durante os testes | Shadow não altera `sensor.status_casa` | Estado antes/depois de `sensor.status_casa` | OK |
| R-003 | Validar que timeline/feed não recebem eventos shadow | `sensor.casa_timeline`, `sensor.casa_event_feed`, sensores shadow | Observar a timeline/feed durante testes | Timeline só muda por V20.1B, não por shadow | Leitura da timeline/feed antes/depois | OK |
| R-004 | Validar que aliases finais permanecem iguais | aliases finais V20 | Observar após testes | Nenhum alias final aponta para sensores V20.2 shadow | Estado dos aliases e registros | Parcial |
| R-005 | Validar que automação legada continua ativa | automações antigas relevantes | Observar evento real de legado | Shadow não bloqueia ou duplica ação legada | Logs ou notificações de legado | Pendente |

## 4. Testes de Robustez e Contradição

| ID | Objetivo | Entidades envolvidas | Ação física real | Resultado esperado | Evidência | Status |
|---|---|---|---|---|---|---|
| O-001 | Validar fonte inválida | fonte V20.2 em `unknown/unavailable` | Observar fonte inválida real ou em manutenção | `fontes_invalidas` lista a fonte; confiança não fica `alta` indevidamente | Atributo `fontes_invalidas` | Pendente |
| O-002 | Validar contradição casa vazia/presença | `binary_sensor.casa_vazia_v20_2`, `sensor.casa_contexto_humano_v20_2` | Observar situação real de contradição | `contradicao_detectada: true`; estabilidade não `estavel` | `fontes_contraditorias` e atributos de estabilidade | Pendente |
| O-003 | Validar relevância sem evento | `sensor.casa_relevancia_contextual_v20_2`, `sensor.casa_evento_relevante_v20_2` | Observar sem eventos operacionais | Relevância não sobe indevidamente sem evento; contradição registrada se ocorrer | Atributos de evento e relevância | Pendente |
| O-004 | Validar chuva ativa com contexto ambiental normal | `sensor.casa_chuva_estado_v20`, `sensor.casa_contexto_ambiental_v20_2` | Observar chuva ativa com ambiente normal | Se janela aberta e contexto normal, contradição deve aparecer | Ambiental e contradição | Pendente |
| O-005 | Validar internet flapping | `sensor.casa_internet_estado_v20` | Observar transições rápidas de internet | Domínio oscilante fica `observando`, não `estavel` | Estados sequenciais e atributos | Pendente |

## Resumo executivo de saída

- Total executados: 10
- Aprovados: 7
- Parciais: 1
- Bloqueados: 2
- Falhas: 0

> Atualize os valores conforme a execução real e registre as evidências no corpo deste documento.
### Evidências 
# Execução V20.2 — Sessão de Homologação Real

Data: 2026-05-19  
Ambiente: Produção + V20.2 Shadow  
Escopo: Fase A — Baseline, Regressão e Sensores Básicos

---

## Resultados detalhados

| ID | Teste | Status | Evidência | Resultado |
|---|---:|---|---|---|
| U-001 | Baseline casa sem incidentes | Bloqueado ⚠️ | Backup Google em falha | Pré-condição não atendida |
| U-004 | Chuva detectada | OK ✅ | Chuva ativa detectada | Domínio ambiental respondeu corretamente |
| U-005 | Chuva + contexto | OK ✅ | `evento_base=chuva_janela_aberta` | Relevância contextual crítica correta |
| U-006 | Energia normal | OK ✅ | `binary_sensor.energia_concessionaria=on` ; `sensor.casa_energia_estado_v20=presente` | Energia operacional normal |
| U-009 | Internet normal | OK ✅ | `sensor.internet_estado_operacional=normal`; `sensor.casa_internet_estado_v20=normal`; `binary_sensor.internet_wan_principal_ok=on` | Conectividade íntegra |
| U-016 | TV ligada | OK ✅ | `switch.tv_tomada_1=on`; `sensor.consumo_tv=380.69` | TV identificada como ativa |
| U-017 | TV desligada | Bloqueado ⚠️ | TV em uso por terceiro | Dependência operacional externa |
| R-002 | Validar que `sensor.status_casa` não muda por shadow | OK ✅ | `sensor.status_casa=⚠️ Backup Google com falha`; origem=`backup` | Shadow não alterou produção |
| R-003 | Validar que timeline/feed não recebem eventos shadow | OK ✅ | `sensor.casa_timeline=20:18 📺 Globo HD`; `sensor.casa_event_feed=20:18 📺 Globo HD` | Shadow não publicou eventos |
| R-004 | Validar aliases finais | Parcial ⚠️ | Alguns contextos só existem em `_v20_2` | Requer revisão futura |

---

## Evidências detalhadas

### U-004/U-005 — Chuva contextual

Entidades observadas:

```text
sensor.casa_relevancia_contextual_v20_2 = critica
evento_base = chuva_janela_aberta
motivo = Chuva ativa com janela aberta
confianca = alta
R-001

Status:
OK

Escopo pesquisado:

- packages
- automations
- scripts
- dashboards
- ui-lovelace
- .storage

Resultado:

Packages:

✓ status_casa_v19.yaml
→ localizado em _disabled
→ histórico/desativado

✓ DESATIVACAO_V19.md
→ documentação histórica

✓ status_casa.yaml
→ comentário passivo

Automations:

✓ nenhuma referência encontrada

Scripts:

✓ nenhuma referência encontrada

Dashboards:

⚠️ Encontradas referências V19 em:

lovelace.teste_4

Entidades:

sensor.status_casa_v19
sensor.casa_modo_operacional_v19
sensor.casa_score_ativo_v19
binary_sensor.casa_incidente_ativo_v19
binary_sensor.casa_tv_ativa_v19
sensor.casa_tv_contexto_v19
sensor.casa_event_feed_v19
sensor.casa_evento_atual_v19
sensor.casa_contexto_humano_v19
#
V19 permanece desativada operacionalmente.

Não há dependências ativas em:
- automações
- scripts
- produção principal

Achado:

Existe dashboard legado (lovelace.teste_4)
referenciando entidades V19.    
#
R-001 ✅
R-002 ✅
R-003 ✅
R-004 ⚠️ Parcial

U-004/U-005 ✅
U-006 ✅
U-009 ✅
U-016 ✅

U-001 ⚠️
U-017 ⚠️

Falhas: 0 🎉

## Lições aprendidas

- V20.2 permaneceu isolada da produção durante toda a sessão, confirmando o modo shadow.
- Chuva com janela aberta gerou o contexto correto de `chuva_janela_aberta` e elevou a relevância de forma adequada.
- Falha no backup Google contaminou a baseline e bloqueou a validação de alguns testes.
- Foram encontrados dashboards legados V19 (`lovelace.teste_4`) com referências a entidades `_v19`.
- Aliases finais ainda precisam de revisão para separar claramente o shadow V20.2 dos endpoints de produção.

