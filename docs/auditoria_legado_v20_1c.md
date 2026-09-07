# Auditoria de Legado V20.1C

## Objetivo

Documentar o diagnóstico inicial da camada de legado antes de qualquer desativação ou remoção controlada.

O foco desta fase é identificar dependências, classificar componentes e mapear riscos indiretos entre:

- V19 / legado preservado
- V20.1B / camada oficial de produção
- V20.2 / motores em shadow mode
- artefatos de teste e de configuração desativada

## Escopo do diagnóstico

1. Inventário preliminar de pacotes e arquivos em `packages/`.
2. Identificação de helpers (`input_boolean`, `input_number`, `input_select`, `input_text`).
3. Mapeamento de `automations.yaml`, `scripts.yaml` e `scripts_erro500.yaml`.
4. Revisão de dashboards oficiais e `.storage` de Lovelace.
5. Contagem de referências `_v19` e `_v20_2` como indicadores de legado e shadow.

## Inventário preliminar

### 1. Pacotes

Encontrados os seguintes itens de pacotes relevantes:

- `packages/_disabled/status_casa_v19.yaml` — pacote V19 desativado.
- `packages/_disabled/DESATIVACAO_V19.md` — documentação de desativação do legado.
- `packages/central_operacional_aliases_v20.yaml` — aliases operacionais V20.
- `packages/motor_confianca_v20_2.yaml` — motor de confiança V20.2 (shadow).
- `packages/motor_contexto_v20_2.yaml` — motor de contexto V20.2 (shadow).
- `packages/motor_relevancia_v20_2.yaml` — motor de relevância V20.2 (shadow).
- `packages/motor_eventos_v20.yaml` — motor de eventos V20.
- `packages/motor_timeline_v20.yaml` — motor de timeline V20.
- `packages/parametros_operacionais_v20.yaml` — parâmetros operacionais V20.
- `packages/status_casa.yaml` — pacote de governança/status de casa.
- `packages/alertas_contextuais_v2_corrigido.yaml` — alertas contextuais V2 corrigidos.
- `packages/energia_contexto.yaml`, `packages/ha_inicio.yaml`, `packages/modo_dormir.yaml`, `packages/carro.yaml`, `packages/carro_presenca.yaml`, `packages/ventilador_quarto_maior.yaml`, `packages/wan_4g_engine_v20.yaml` — pacotes de suporte e domínios específicos.
- `packages/teste_motor_eventos_v20.yaml` — pacote de teste.

### 2. Helpers

Arquivos de helpers presentes no repositório:

- `input_boolean.yaml`
- `input_number.yaml`
- `input_select.yaml`
- `input_text.yaml`

Estes arquivos são parte do inventário de controle operacional e devem ser considerados ao avaliar a dependência de automações legadas.

### 3. Automações e scripts

- `automations.yaml` — arquivo único de automações principal.
- `automations/` — não foram encontrados arquivos `.yaml` adicionais no diretório para esta auditoria inicial.
- `scripts.yaml` — script principal.
- `scripts_erro500.yaml` — scripts de fallback/erro.

Contagens relevantes:

- `automations.yaml` contém 97 entradas `alias:`.
- `scripts*.yaml` contém pelo menos 14 entradas `alias:`.

### 4. Dashboards

Arquivos de Lovelace sob revisão:

- `ui-lovelace.yaml` — dashboard principal oficial.
- `.storage/lovelace.*` — dashboards armazenados, incluindo:
  - `lovelace.dashboard_alarme`
  - `lovelace.dashboard_baterias`
  - `lovelace.dashboard_carro`
  - `lovelace.dashboard_lixo`
  - `lovelace.dashboard_testes`
  - `lovelace.debug_operacional`
  - `lovelace.lovelace`
  - `lovelace.map`
  - `lovelace.minha_casa`
  - `lovelace.sistema_casa`
  - `lovelace.teste_4`
  - `lovelace.testes_anterior`
  - `lovelace_dashboards`
  - `lovelace_resources`

### 5. Entidades e referências de versão

Referências detectadas:

- `160` ocorrências de `_v19` em `packages/` e `.storage`.
- `195` ocorrências de `_v20_2` em `packages/` e `.storage`.

Observações:

- Não foram encontradas referências `_v20_2` em `ui-lovelace.yaml` no escopo desta busca.
- Referências `_v20_2` aparecem em `.storage` principalmente em `core.entity_registry` e `core.restore_state`, indicando presença de entidades shadow no registro e estado restaurado.
- Referências `_v19` aparecem em `.storage` nas mesmas áreas, além do pacote desativado `packages/_disabled/status_casa_v19.yaml`.

## Achados iniciais

1. O legado V19 está preservado como um artefato controlado, com código de suporte desativado em `packages/_disabled`.
2. A camada V20.2 está presente em shadow, com registros de entidades e estados em `.storage`, mas sem evidência de consumo direto em `ui-lovelace.yaml`.
3. `packages/_disabled/DESATIVACAO_V19.md` reforça o desenho de desativação, portanto a auditoria deve tratar o pacote V19 como histórico, não como ativo de produção.
4. O dashboard armazenado `lovelace.teste_4` já foi identificado como fonte de referências `_v19`; deve ser considerado um artefato de teste/debug e não um dashboard oficial.
5. Automations e scripts ainda são um ponto crítico: 97 aliases em `automations.yaml` significam muitos fluxos a serem auditados para dependências e side-effects.

## Classificação preliminar

### CORE

- `packages/central_operacional_aliases_v20.yaml`
- `packages/motor_eventos_v20.yaml`
- `packages/motor_timeline_v20.yaml`
- `packages/parametros_operacionais_v20.yaml`
- `packages/status_casa.yaml`
- `packages/alertas_contextuais_v2_corrigido.yaml`
- `packages/energia_contexto.yaml`
- `packages/ha_inicio.yaml`
- `packages/modo_dormir.yaml`
- `packages/carro.yaml`
- `packages/carro_presenca.yaml`
- `packages/ventilador_quarto_maior.yaml`
- `packages/wan_4g_engine_v20.yaml`

### LEGADO

- `packages/_disabled/status_casa_v19.yaml`
- `.storage/core.entity_registry` com registros de entidades V19
- `.storage/core.restore_state` com estados restaurados de entidades V19
- `lovelace.teste_4` (dashboard de teste com referências V19)

### SHADOW

- `packages/motor_confianca_v20_2.yaml`
- `packages/motor_contexto_v20_2.yaml`
- `packages/motor_relevancia_v20_2.yaml`
- `.storage/core.entity_registry` com registros de entidades V20.2
- `.storage/core.restore_state` com histórico de entidades V20.2

### OBSOLETO / ARTEFATO DE DOCUMENTAÇÃO

- `packages/_disabled/DESATIVACAO_V19.md`
- `packages/teste_motor_eventos_v20.yaml` (artefato de teste)
- dashboards `.storage` de teste e debug que não são parte do arquivo oficial `ui-lovelace.yaml`

## Riscos imediatos identificados

- Dependências indiretas de legacy V19 em sensores e dashboards podem não ser visíveis apenas pelo grep de `_v19`.
- A presença de entidades V19 e V20.2 em `.storage` pode confundir validações se consideradas como configuração ativa.
- Remoção de qualquer pacote legado sem auditoria dos aliases e scripts pode causar regressão em notificações e controles operacionais.
- A ausência de consumo `_v20_2` em `ui-lovelace.yaml` é um sinal positivo, mas deve ser validada em todas as fontes de dashboard oficiais.

## Próximos passos

1. Expandir o inventário para incluir a lista completa de `alias:` e seus destinos de ações.
2. Auditar automações críticas que atualizam helpers (`input_boolean`, `input_number`, `input_text`) usados por V20.1B.
3. Verificar dependência de V19 em dashboards dinâmicos além de `ui-lovelace.yaml`.
4. Classificar pacotes e automações por risco de remoção e visibilidade de side-effect.
5. Preparar a fase de desativação gradual com lotes reversíveis e rollback simples.

## V20.1C - Trilha operacional remanescente

Data de atualização: 2026-05-22.

Modo: documentação apenas. Nenhum YAML produtivo, automação, script, dashboard, entity, `.storage`, package ou commit foi alterado nesta etapa.

### Artefatos consultados

- `AGENTS.md`
- `docs/ROADMAP.md`
- `CHANGELOG.md`
- `architecture.md`
- `docs/ARCHITECTURE.md`
- `docs/inventario_legacy_migration_v20_1.md`
- `docs/auditoria_legado_v20_1c.md`
- `docs/dependencias_legado_v20_1d.md`
- `docs/impacto_limpeza_v20_1e.md`
- `automations.yaml`
- `scripts.yaml`
- `scripts_erro500.yaml`
- `blueprints/automation/*`
- `packages/*`

### Artefatos não encontrados ou vazios

- `scripts.yaml` existe, mas está vazio.
- Não há diretório `automations/` com arquivos YAML adicionais neste escopo.
- A auditoria não encontrou autorização para limpeza, desativação ou migração operacional.

### Hipótese aplicada

V20.1C possui diagnóstico documental concluído, mas a trilha operacional de auditoria/decommission permanece aberta. V20.1C não autoriza desligamento de automações, scripts, blueprints, packages, helpers, dashboards ou entidades. Qualquer limpeza futura depende de classificação de risco, consumidores conhecidos, side-effects mapeados, período de observação e rollback.

### Critério de classificação

- **CRÍTICO**: energia, UPS, internet, failover, alarme, recovery e segurança.
- **MÉDIO**: contexto, notificações, modo dormir, presença e carro.
- **BAIXO**: duplicidades, timeline, mensagens sem ação física crítica e laboratório.

### Matriz por domínio operacional

| Domínio | Itens auditados | Ação física / helper atualizado | Side-effects e consumidores conhecidos | Risco | Pode migrar? | Pode desativar? | Rollback necessário? |
|---|---|---|---|---|---|---|---|
| Internet / failover / recovery | `Home Office - Sem Conexão com Internet`; `Home Office - Internet - Retorno da Conexão`; `Home office - Queda link Fibra`; `Home Office - Retorno Link Fibra`; `Home office - Queda link principal - Fibra`; `LAB - Caiu link contingência`; `Home Office - Retorno Contingência ZTE`; `Home office - Notificar mudança de conexão ativa`; `Home Office - Notificar mudança de conexão ativa_WM`; `Home Office - Desliga Pi-Hole por 5 min`; `LAB - Pi-hole Caiu !!`; `Gestão modem 4G`; `script Home office - Religar Link Fibra após 10min`; blueprints `wan_4G_AppleWatch.yaml` e `wan_4G_AppleWatch_Energia.yaml`; package `wan_4g_engine_v20.yaml` | Desliga/liga porta ou tomada de rede, religa link fibra, reinicia modem/4G/Pi-hole, envia push e cria notificações persistentes | Consumidores/contratos V20 relacionados: `sensor.internet_estado_operacional`, `binary_sensor.internet_wan_principal_ok`, `binary_sensor.internet_wan2_4g_ok`, `binary_sensor.internet_em_failover_4g`, timeline/feed V20 e WAN engine. Side-effect físico crítico: recovery de conectividade | CRÍTICO | Sim, somente publicação/evento; preservar recovery físico | Não | Obrigatório |
| Energia / UPS / tomadas críticas | `Home Office - Monitorar falta de energia e desligar após 30 minutos`; `LAB - Liga Estabilizador via Agenda`; `LAB - Desliga Estabilizador via Agenda`; `LAB - Desligar Agenda Monitor DELL`; `LAB - Querida Cheguei`; `LAB - Querida Fui`; `Gestão de Energia - UPS`; `Gestão tomada teste humidificador`; `Cozinha - Gestão Energia Geladeira`; `Dispensa - Gestão Energia Cervejeira`; blueprints `Gestão de Energia - UPS.yaml`, `Pós-retorno de energia.yaml`, `monitoramento_energia_v6.yaml`; packages `energia_contexto.yaml`, `cerebro_backup_formatado.yaml` | Liga/desliga tomadas, monitora UPS, pode executar `hassio.host_shutdown`, atualiza `input_boolean.energia_falta`, envia push/logbook | Consumidores/contratos V20 relacionados: `binary_sensor.energia_concessionaria`, `sensor.ups_voltagem`, `sensor.ups_potencia`, `sensor.casa_energia_estado_v20`, timeline/feed V20. Side-effect físico crítico: proteção elétrica e desligamento de host | CRÍTICO | Sim, somente publicação/evento; preservar proteção/recovery | Não | Obrigatório |
| Alarme / segurança / vazamento | `LAB - Alarme - Armado Modo Casa`; `LAB - Alarme - Desarmado`; `LAB - Alarme - Armado Modo Ausente`; `LAB - Alarme - Dispara alarme casa`; duas automações `LAB - Alarme - Alarme disparado`; `Ativar alarme automaticamente e notificar`; `Desativar alarme ao acordar`; `Quarto - Liga luz de segurança`; `LAB - Liga Automação Luz Segurança`; `Quarto - Desligar luz segurança`; blueprint `luz_seguranca_portas_noite.yaml`; blueprint `Alerta_Vazamento_com_Led.yaml`; package `modo_dormir.yaml` quando arma/desarma alarme | Arma/desarma/dispara `alarm_control_panel.home_alarm`, toca mídia, acende luz de segurança, envia push/Alexa, atualiza helpers de modo dormir/alarme | Consumidores conhecidos: alarme físico/HA, modo dormir, presença humana, automações de porta. Side-effect crítico: segurança residencial e sinalização | CRÍTICO | Sim, apenas publicação/observação; preservar ação de segurança | Não | Obrigatório |
| Máquina de lavar / microondas / eletrodomésticos | `Cozinha - Liga Microondas`; `LAB - Desliga Agenda Micro Ondas`; `LAB - Liga Agenda Micro Ondas`; `LAB - Liga Agenda Maq. de Lavar`; `LAB - Desliga Agenda Maq. de Lavar`; `Area de Serviço - Maquina de Lavar - avisar Jacira`; `Área de Serviços - Ciclos da Máquina de Lavar`; `Máquina de lavar - início do ciclo`; `Máquina de lavar - máquina de estados`; `Máquina de lavar - fim do ciclo`; `Máquina de lavar - notificar fim`; `Máquina de lavar - reset para desligada` | Liga/desliga tomadas, atualiza `input_boolean.lavando_roupas`, `input_boolean.maquina_lavar_ciclo_ativo`, `input_select.maquina_lavar_estado`, incrementa `counter.ciclos_lava_roupa`, envia push | Consumidores conhecidos: `motor_atividade_operacional_v20.yaml`, `motor_timeline_v20.yaml`, política V20.1O de máquina/microondas e sensores de potência/corrente | MÉDIO | Sim, se contrato V20 preservar estado de ciclo | Não sem observação | Recomendado |
| Modo dormir / contexto humano | `Quarto - Hora de dormir 0:45`; `Quarto - Hora de dormir 1:30`; `Quarto - Hora de dormir 2:30`; `Quarto - Hora de dormir 3:30`; `Ativar Cena Boa Noite ao Ativar Modo Dormir`; `Registrar ativação do modo dormir`; scripts `Forçar Modo Dormir`, `Acordar Wilson`, `Teste Botão Acordar`; blueprints `Modo_Dormir_por_Inatividade_Geral.yaml` e `modo_dormir_inatividade_geral_v2.yaml`; packages `modo_dormir.yaml`, `motor_contexto_v20_2.yaml` | Aciona cena, luzes, alarme, helpers `input_boolean.modo_dormir_ativo`, `input_boolean.wilson_dormindo`, `input_boolean.jacira_dormindo`, `input_text.ultima_ativacao_modo_dormir` | Consumidores conhecidos: Context Engine V20.2 shadow, modo dormir, automações de alarme e cenas de conforto | MÉDIO, com subpartes CRÍTICAS quando envolvem alarme | Sim, se separar contexto de ações físicas | Não sem observação | Recomendado; obrigatório quando alarme estiver envolvido |
| Presença / carro | packages `carro.yaml` e `carro_presenca.yaml`; automações de saída/retorno do carro; scripts relacionados a chegada/saída quando acionam casa | Atualiza `input_boolean.carro_em_uso`, `input_datetime.ultima_saida_carro`, `input_datetime.ultimo_retorno_carro`, `counter.carro_viagens`, odômetro/manutenção e envia push | Consumidores conhecidos: dashboards/registro de carro, presença humana, futuras camadas de presença V20/V21 | MÉDIO | Sim, se houver contrato V20 de presença/carro | Não sem observação | Recomendado |
| Notificações e mensagens legadas | `Lab - Moeda - Dólar abaixo de ...`; `Lab - Moeda - Euro abaixo de ...`; `Quarto - Está chovendo`; `Quarto - Parou de chover`; `Quarto - Monitorar estado do umidificador`; `Lab - Notificar quando downloads forem concluídos_WM`; `Quarto - Notificação intensidade de Chuva`; `Sala - Porta da sala abriu`; `LAB - Notificar Início do Home Assistant - WM`; `Notificação de Pesagem - WM`; `Note HA uptime`; `Monitora Baterias`; scripts Alexa; packages `alertas_contextuais_v2_corrigido.yaml` e `ha_inicio.yaml` | Envia push/Alexa/logbook; alguns escrevem `input_text.central_ultimo_evento` e `input_text.central_ultima_notificacao` | Consumidores conhecidos: `sensor.central_ultima_mensagem`, aliases finais, timeline/feed V20, usuário via push. Risco principal: duplicidade e divergência narrativa | MÉDIO | Sim, migrar para política V20 de push/timeline quando houver equivalência | Não sem observação | Recomendado |
| Iluminação / conforto / laboratório | Automações de movimento do corredor/banheiro/cozinha; rotinas de Home Office; régua USB; altura de mesa; ar-condicionado/umidificador; tema; luz de mesa; blueprints de luz/movimento; package `ventilador_quarto_maior.yaml`; scripts de luz/expediente/desliga tudo | Liga/desliga luzes, cenas, réguas, ventilador, mesa, ar/umidificador; atualiza helpers locais (`input_boolean.office`, `input_boolean.usb`, `input_boolean.altura_mesa`, `input_number.nivel_atual_umidificador`) | Consumidores conhecidos: dispositivos físicos de conforto e helpers de rotina; não são fonte canônica V20, mas podem afetar experiência diária | BAIXO, exceto quando ligado a segurança/energia | Sim, se virar evento observável; ações físicas podem permanecer legadas | Sim, somente após validar uso real/duplicidade | Simples |
| Timeline / aliases / motores V20 | packages `central_mensagens_corrigido.yaml`, `central_operacional_aliases_v20.yaml`, `motor_eventos_v20.yaml`, `motor_timeline_v20.yaml`, `parametros_operacionais_v20.yaml`, `motor_atividade_operacional_v20.yaml`, `status_casa.yaml` | Produzem sensores, aliases, políticas, feed/timeline/push e mensagens centrais; atualizam helpers centrais em scripts da Central | Consumidores conhecidos: dashboards produtivos, aliases finais, timeline/feed V20, `sensor.status_casa`. São parte da arquitetura V20, não legado removível | CRÍTICO para aliases/status; MÉDIO para motores shadow/atividade | Não é migração de legado; tratar por lote formal próprio | Não | Obrigatório |
| V20.2 shadow / harness / teste | packages `motor_contexto_v20_2.yaml`, `motor_relevancia_v20_2.yaml`, `motor_confianca_v20_2.yaml`, `teste_motor_eventos_v20.yaml`; automações/labs de teste | Criam sensores shadow e cenários de teste; não devem alterar aliases finais nem dashboards produtivos | Consumidores conhecidos: auditoria/harness V20.2; risco de confusão se tratados como produção | MÉDIO | Sim, apenas por promoção formal V20.2+ | Não sem decisão formal | Recomendado |

### Resultado por tipo de artefato

| Tipo | Resultado |
|---|---|
| `automations.yaml` | 98 automações auditadas; há blocos críticos de internet, energia, alarme e recovery que não podem ser desligados. Blocos de máquina, modo dormir, presença e notificações exigem observação. Blocos de iluminação/laboratório são candidatos de menor risco, mas ainda dependem de validação de uso real. |
| `scripts.yaml` | Arquivo vazio; nenhum script ativo encontrado. |
| `scripts_erro500.yaml` | 14 scripts legados auditados. Scripts de alarme, acordar e religar fibra são críticos. Scripts de Alexa/modo dormir são médios. Scripts de luz/conforto são baixos. |
| `blueprints/automation/*` | 19 blueprints auditados. Energia/UPS, WAN/4G, pós-retorno de energia, vazamento e luz de segurança são críticos. Modo dormir, presença, bateria e uptime são médios. Luz/movimento puro é baixo. |
| `packages/*` | 20 packages auditados. Packages V20 canônicos e `status_casa.yaml` são críticos. Packages V20.2 shadow e atividade/contexto são médios. Packages de carro/presença/notificação são médios. `ventilador_quarto_maior.yaml` é baixo. |

### Conclusão operacional V20.1C

A trilha operacional remanescente está classificada, mas não encerrada para decommission. O conjunto crítico ainda mistura publicação textual, recuperação física, segurança, energia e conectividade. Portanto:

- automações críticas não podem ser desativadas;
- scripts críticos não podem ser removidos;
- blueprints críticos não podem ser arquivados sem migração e rollback;
- packages V20 canônicos não são candidatos de limpeza;
- duplicidades de notificação/timeline podem ser migradas, mas apenas após validação de consumidores;
- iluminação/laboratório pode entrar em lote futuro de saneamento de baixo risco, desde que validado por uso real.

## Legado em quarentena controlada

Data de atualização: 2026-05-22.

Esta classificação não autoriza desligamento imediato. Ela define apenas o nível de quarentena documental para orientar observação futura, migração controlada e eventual desativação por lote próprio.

### Categorias de quarentena

| Categoria | Significado | Janela mínima | Apto para futura desativação |
|---|---|---:|---|
| Quarentena C1 | Baixo risco; conforto, laboratório, duplicidade ou iluminação sem vínculo crítico identificado | 7 dias | SIM |
| Quarentena C2 | Médio risco; notificações, contexto, modo dormir, presença, carro ou helpers consumidos por fluxos humanos | 14 dias | SIM |
| Quarentena C3 | Alto risco; controla tomada, cena, helper operacional ou rotina com possível efeito físico relevante, mas não é infraestrutura crítica isolada | 30 dias | NÃO |
| Bloqueado | Energia, UPS, internet, failover, alarme, recovery, segurança ou contrato V20 canônico | Não aplicável | NÃO |

### Quarentena C1 - baixo risco

| Nome | Categoria | Risco | Consumidores conhecidos | Side-effects | Motivo da quarentena | Janela mínima | Rollback necessário | Apto futura desativação |
|---|---|---|---|---|---|---:|---|---|
| `Corredor - Parou Movimento` | iluminação/conforto | BAIXO | Luzes do corredor | `light.turn_off` | Automação física simples e substituível por blueprint/rotina de presença | 7 dias | Simples | SIM |
| `Corredor - Iniciou Movimento` | iluminação/conforto | BAIXO | Luzes do corredor | `light.turn_on` | Automação física simples e observável | 7 dias | Simples | SIM |
| `Banheiro - Iniciou Movimento` | iluminação/conforto | BAIXO | Luz do banheiro | `light.turn_on` | Baixo acoplamento com V20; validar uso real | 7 dias | Simples | SIM |
| `Banheiro - Parou Movimento` | iluminação/conforto | BAIXO | Luz do banheiro | `light.turn_off` | Baixo acoplamento com V20; validar uso real | 7 dias | Simples | SIM |
| `Cozinha - Iniciou Movimento` | iluminação/conforto | BAIXO | Luz da cozinha | `light.turn_on` | Rotina física simples de presença | 7 dias | Simples | SIM |
| `Cozinha - Parou Movimento` | iluminação/conforto | BAIXO | Luz da cozinha | `light.turn_off` | Rotina física simples de presença | 7 dias | Simples | SIM |
| `Desliga everything` | laboratório/conforto | BAIXO | `script.desliga_tudo`, `input_boolean.everything` | Aciona script de desligamento geral | Entrada legada de rotina, precisa validar se ainda é usada | 7 dias | Simples | SIM |
| `Lab - Muda o Tema Automaticamente` | laboratório/UI | BAIXO | Tema do frontend | tema automático | Não afeta contratos V20 | 7 dias | Simples | SIM |
| `Homeoffice - Ligar Regua USB` | conforto | BAIXO | `switch.regua_zigbee_br_l5`, `input_boolean.usb` | Liga régua USB | Rotina local substituível; validar uso | 7 dias | Simples | SIM |
| `Home office - Desliga Regua USB` | conforto | BAIXO | `switch.regua_zigbee_br_l5`, `input_boolean.usb` | Desliga régua USB | Rotina local substituível; validar uso | 7 dias | Simples | SIM |
| `Home office - Baixar Mesa` | conforto | BAIXO | Cenas de mesa, `input_boolean.altura_mesa` | `scene.turn_on` | Rotina local de ergonomia | 7 dias | Simples | SIM |
| `Home office - Cena Memória 4` | conforto | BAIXO | Cenas de mesa, `input_boolean.altura_mesa` | `scene.turn_on` | Rotina local de ergonomia | 7 dias | Simples | SIM |
| `Home office - Levantar Mesa` | conforto | BAIXO | Cenas de mesa, `input_boolean.altura_mesa` | `scene.turn_on` | Rotina local de ergonomia | 7 dias | Simples | SIM |
| `Salvar Último Nível de Água` | conforto | BAIXO | `input_number.nivel_atual_umidificador` | Atualiza helper do umidificador | Helper local, sem contrato V20 conhecido | 7 dias | Simples | SIM |
| `Lab - Liga input_boolean.hd_backup` | laboratório | BAIXO | `input_boolean.hd_backup` | Atualiza helper | Agenda/lab, não é backup operacional V20 | 7 dias | Simples | SIM |
| `Lab - Desligar input_boolean.hd_backup` | laboratório | BAIXO | `input_boolean.hd_backup` | Atualiza helper | Agenda/lab, não é backup operacional V20 | 7 dias | Simples | SIM |
| `Lab - Liga input_boolean.impressora` | laboratório | BAIXO | `input_boolean.impressora` | Atualiza helper | Rotina local/lab | 7 dias | Simples | SIM |
| `Lab - Desliga input_boolean.impressora` | laboratório | BAIXO | `input_boolean.impressora` | Atualiza helper | Rotina local/lab | 7 dias | Simples | SIM |
| `Lab - input_boolean.monitor_aoc` | laboratório | BAIXO | `input_boolean.monitor_aoc` | Atualiza helper | Rotina local/lab | 7 dias | Simples | SIM |
| `Lab - Desliga input_boolean.monitor_aoc` | laboratório | BAIXO | `input_boolean.monitor_aoc` | Atualiza helper | Rotina local/lab | 7 dias | Simples | SIM |
| `Quarto - Ligar Ar Condicionado - Frio e Umidificador` | conforto | BAIXO | Tomada mesa/ar/umidificador | `switch.turn_on` | Rotina local de conforto; validar uso sazonal | 7 dias | Simples | SIM |
| `Quarto - Ligar Ar Condicionado - Auto e Umidificador` | conforto | BAIXO | Tomada mesa/ar/umidificador | `switch.turn_on` | Rotina local de conforto; validar uso sazonal | 7 dias | Simples | SIM |
| `Quarto - Desliga Ar Condicionado e Umidificador` | conforto | BAIXO | Tomada mesa/ar/umidificador | `switch.turn_off` | Rotina local de conforto; validar uso sazonal | 7 dias | Simples | SIM |
| `Quarto Gestão luz - 17:30 as 22:30 todos os dias` | iluminação | BAIXO | Luz do quarto | Blueprint de luz | Já padronizável por blueprint | 7 dias | Simples | SIM |
| `Quarto - Gestão Luz Mesa - Horario comercial` | iluminação | BAIXO | Luz de mesa | Blueprint de luz | Já padronizável por blueprint | 7 dias | Simples | SIM |
| `Quarto Maior - Gestão Luz da Mesa - 17:30 as 22:30 - dias de semana` | iluminação | BAIXO | Luz de mesa | Blueprint de luz | Já padronizável por blueprint | 7 dias | Simples | SIM |
| `Corredor - Apaga luz corredor pela manhã` | iluminação | BAIXO | Luz corredor | `light.turn_off` | Rotina simples por horário | 7 dias | Simples | SIM |
| `Coloca o tema Claro` | UI/laboratório | BAIXO | Tema do frontend | tema claro | Sem vínculo operacional V20 | 7 dias | Simples | SIM |
| `Home office - Iniciar expediente` | script legado | BAIXO | Luzes do Home Office | `light.turn_on` | Script local vazio/legado de rotina | 7 dias | Simples | SIM |
| `Home office - Iniciar expediente Strip Light` | script legado | BAIXO | Fita/luzes do Home Office | `light.turn_on`, `light.turn_off` | Script local de conforto | 7 dias | Simples | SIM |
| `Desliga tudo` | script legado | BAIXO | Luzes/réguas | `light.turn_off`, `switch.turn_off` | Rotina física local; validar acionadores antes | 7 dias | Simples | SIM |
| `Verifica subida do sistema HA` | script legado | BAIXO | Luz/indicador | `light.turn_off` | Sem consumidor crítico identificado | 7 dias | Simples | SIM |
| `Resetar Nível do Umidificador` | script legado | BAIXO | `input_number.nivel_atual_umidificador` | Atualiza helper | Helper local | 7 dias | Simples | SIM |
| `LAB - Reset do Não desligar Monitor` | script legado | BAIXO | `input_boolean.naodesliguemonitorhoje` | Desliga helper | Rotina lab/local | 7 dias | Simples | SIM |
| Blueprints de luz/movimento puros | blueprint | BAIXO | Luzes e sensores de presença | `light.turn_on`, `light.turn_off` | Candidatos a padronização, não a remoção cega | 7 dias | Simples | SIM |

### Quarentena C2 - médio risco

| Nome | Categoria | Risco | Consumidores conhecidos | Side-effects | Motivo da quarentena | Janela mínima | Rollback necessário | Apto futura desativação |
|---|---|---|---|---|---|---:|---|---|
| `Quarto - Está chovendo` | notificação | MÉDIO | Usuário/push; timeline V20 já cobre chuva | `notify.mobile_app_iphonewm` | Duplicidade provável com timeline/push V20 | 14 dias | Recomendado | SIM |
| `Quarto - Parou de chover` | notificação | MÉDIO | Usuário/push; timeline V20 já cobre chuva | `notify.mobile_app_iphonewm` | Duplicidade provável com timeline/push V20 | 14 dias | Recomendado | SIM |
| `Quarto - Notificação intensidade de Chuva` | notificação | MÉDIO | Usuário/push; intensidade de chuva | `notify.mobile_app_iphonewm` | Conteúdo mais rico que V20; migrar antes de desligar | 14 dias | Recomendado | SIM |
| `Sala - Porta da sala abriu` | notificação | MÉDIO | Usuário/push; timeline V20 porta | `notify.mobile_app_iphonewm` | Duplicidade provável com evento V20 de porta | 14 dias | Recomendado | SIM |
| `Lab - Moeda - Dólar abaixo de ...` | notificação/lab | MÉDIO | Usuário/push/Alexa | Push, Alexa, luz | Notificação não operacional; validar utilidade | 14 dias | Recomendado | SIM |
| `Lab - Moeda - Euro abaixo de ...` | notificação/lab | MÉDIO | Usuário/push/Alexa | Push, Alexa, luz | Notificação não operacional; validar utilidade | 14 dias | Recomendado | SIM |
| `Quarto - Monitorar estado do umidificador` | notificação | MÉDIO | Usuário/push | `notify.mobile_app_iphone14` | Pode ser útil ao conforto; não remover sem observação | 14 dias | Recomendado | SIM |
| `Lab - Notificar quando downloads forem concluídos_WM` | notificação/lab | MÉDIO | Usuário/push | `notify.mobile_app_iphone14` | Lab/rotina pessoal; validar uso real | 14 dias | Recomendado | SIM |
| `LAB - Notificar Início do Home Assistant - WM` | notificação | MÉDIO | Usuário/push | `notify.mobile_app_iphonewm` | Pode duplicar health/uptime; validar utilidade | 14 dias | Recomendado | SIM |
| `Notificação de Pesagem - WM` | notificação | MÉDIO | Usuário/push | `notify.mobile_app_iphonewm` | Rotina pessoal; validar uso | 14 dias | Recomendado | SIM |
| `Note HA uptime` | notificação | MÉDIO | Usuário/push | blueprint uptime | Health informativo; migrável para observabilidade | 14 dias | Recomendado | SIM |
| `Monitora Baterias` | notificação | MÉDIO | Usuário/push | blueprint bateria | Útil operacionalmente; não desligar sem substituto | 14 dias | Recomendado | SIM |
| `Home office - Alexa - Let's rock` | script legado | MÉDIO | Alexa/Home Office | volume/media/push Alexa | Rotina pessoal com mídia; validar acionadores | 14 dias | Recomendado | SIM |
| `Home office - Alexa - Good bye` | script legado | MÉDIO | Alexa/Home Office | volume/media stop/push Alexa | Rotina pessoal com mídia; validar acionadores | 14 dias | Recomendado | SIM |
| `Anunciar via Alexa` | script legado | MÉDIO | Alexa | Blueprint de anúncio | Consumidor pode ser indireto | 14 dias | Recomendado | SIM |
| `Teste Botão Acordar` | script legado | MÉDIO | Usuário/teste | `notify.notify` | Artefato de teste com notificação | 14 dias | Recomendado | SIM |
| Máquina de lavar - fluxo de estado | máquina/eletro | MÉDIO | `motor_atividade_operacional_v20`, `motor_timeline_v20`, helpers de ciclo | `input_boolean.maquina_lavar_ciclo_ativo`, `input_select.maquina_lavar_estado`, push | Ainda alimenta estado operacional da máquina; migrar antes | 14 dias | Recomendado | SIM |
| `Area de Serviço - Maquina de Lavar - avisar Jacira` | máquina/notificação | MÉDIO | Usuário/push, helper `lavando_roupas` | push e helper | Pode duplicar V20, mas tem destinatário humano específico | 14 dias | Recomendado | SIM |
| `Área de Serviços - Ciclos da Máquina de Lavar` | máquina/contador | MÉDIO | `counter.ciclos_lava_roupa`, usuário | contador e push | Contabilização pode ser útil; validar consumidor | 14 dias | Recomendado | SIM |
| Modo dormir - horários e cena | contexto | MÉDIO | Modo dormir, cenas, usuário | luzes/cena/helpers | Afeta rotina humana; não desligar sem observação | 14 dias | Recomendado | SIM |
| `Registrar ativação do modo dormir` | contexto | MÉDIO | `input_text.ultima_ativacao_modo_dormir` | Atualiza helper | Helper consumido por contexto/histórico | 14 dias | Recomendado | SIM |
| `Forçar Modo Dormir` | script legado | MÉDIO | `input_boolean.modo_dormir_ativo` | Liga helper | Entrada manual/rotina; validar uso | 14 dias | Recomendado | SIM |
| Blueprints de modo dormir/inatividade | blueprint/contexto | MÉDIO | Helpers de dormir/pessoas | Liga helpers e notifica | Pode alimentar V20.2 contexto | 14 dias | Recomendado | SIM |
| Packages/rotinas de carro e presença | presença/carro | MÉDIO | Dashboards/registro de carro | Helpers, contador, push | Domínio futuro V20/V21; não desligar sem substituto | 14 dias | Recomendado | SIM |
| `packages/alertas_contextuais_v2_corrigido.yaml` automações V2 | mensagens legadas | MÉDIO | `input_text.central_ultimo_evento`, `sensor.central_ultima_mensagem`, push | Atualiza último evento/notificação | Fonte legada ainda consumida indiretamente | 14 dias | Recomendado | SIM |

### Quarentena C3 - alto risco

| Nome | Categoria | Risco | Consumidores conhecidos | Side-effects | Motivo da quarentena | Janela mínima | Rollback necessário | Apto futura desativação |
|---|---|---|---|---|---|---:|---|---|
| `Cozinha - Liga Microondas` | eletro/tomada | ALTO | Tomada do microondas; política V20 microondas observa consumo | `switch.turn_on` | Controla energia de eletrodoméstico; validar rotina física | 30 dias | Obrigatório | NÃO |
| `LAB - Desliga Agenda Micro Ondas` | eletro/tomada | ALTO | Tomada do microondas | `switch.turn_off` | Pode desligar alimentação real; validar agendamento | 30 dias | Obrigatório | NÃO |
| `LAB - Liga Agenda Micro Ondas` | eletro/tomada | ALTO | Tomada do microondas | `switch.turn_on` | Pode ligar alimentação real; validar agendamento | 30 dias | Obrigatório | NÃO |
| `LAB - Liga Agenda Maq. de Lavar` | eletro/tomada | ALTO | Tomada da máquina | `switch.turn_on` | Alimentação física de eletrodoméstico | 30 dias | Obrigatório | NÃO |
| `LAB - Desliga Agenda Maq. de Lavar` | eletro/tomada | ALTO | Tomada da máquina | `switch.turn_off` | Alimentação física de eletrodoméstico | 30 dias | Obrigatório | NÃO |
| `Home Office - Rotina Ligar Home office Voice` | conforto com tomada | ALTO | Luzes/réguas Home Office | `light.turn_on`, `switch.turn_on`, scripts | Liga equipamentos físicos; depende de rotina diária | 30 dias | Obrigatório | NÃO |
| `Lab - Rotina Desligar Home office Voice` | conforto com tomada | ALTO | Luzes/réguas Home Office | `light.turn_off`, `switch.turn_off`, script | Desliga equipamentos físicos; depende de rotina diária | 30 dias | Obrigatório | NÃO |
| `LAB - Liga Agenda Monitor DELL` | tomada/equipamento | ALTO | Régua/monitor | `switch.turn_on` | Liga equipamento físico por agenda | 30 dias | Obrigatório | NÃO |
| `LAB - Desligar Agenda Monitor DELL` | tomada/equipamento | ALTO | Régua/monitor, `input_boolean.naodesliguemonitorhoje` | `switch.turn_off` e helper | Desliga equipamento físico com exceção por helper | 30 dias | Obrigatório | NÃO |
| `Acordar Wilson` | script composto | ALTO | Alarme, modo dormir, luzes, ventilador, usuário | Desarma alarme, altera helpers, liga luz/switch | Mistura segurança, presença e conforto | 30 dias | Obrigatório | NÃO |

### Bloqueado

| Nome | Categoria | Risco | Consumidores conhecidos | Side-effects | Motivo da quarentena | Janela mínima | Rollback necessário | Apto futura desativação |
|---|---|---|---|---|---|---:|---|---|
| Automações de internet/fibra/link/contingência | internet/failover/recovery | CRÍTICO | WAN V20, timeline/feed, usuário, conectividade real | Desliga/liga porta/tomada, religa fibra, push | Recovery de conectividade não pode ser removido por auditoria documental | N/A | Obrigatório | NÃO |
| `Home Office - Desliga Pi-Hole por 5 min` | internet/recovery | CRÍTICO | DNS/Pi-hole, rede doméstica | Desliga/liga Pi-hole e helper | Recovery/mitigação de rede | N/A | Obrigatório | NÃO |
| Blueprints `wan_4G_AppleWatch*` | failover/recovery | CRÍTICO | Link 4G, modem, usuário | Desliga/liga modem/tomada, notifica, logbook | Auto-recovery de conectividade | N/A | Obrigatório | NÃO |
| `Home Office - Monitorar falta de energia e desligar após 30 minutos` | energia/UPS | CRÍTICO | UPS/host HA/usuário | `hassio.host_shutdown`, helper energia, push | Proteção de host e energia | N/A | Obrigatório | NÃO |
| Blueprint `Gestão de Energia - UPS.yaml` | energia/UPS | CRÍTICO | UPS/energia/usuário | helper falta energia, push, logbook, shutdown opcional | Proteção de energia | N/A | Obrigatório | NÃO |
| Blueprint `Pós-retorno de energia.yaml` | energia/recovery | CRÍTICO | Tomadas críticas | religa tomadas, notifica | Recovery pós-retorno de energia | N/A | Obrigatório | NÃO |
| Blueprints `monitoramento_energia_v6.yaml` e instâncias de geladeira/cervejeira | energia/equipamento | CRÍTICO | Tomadas/sensores críticos | notifica e pode religar tomada | Equipamentos críticos e energia | N/A | Obrigatório | NÃO |
| Automações do estabilizador e tomadas críticas | energia/equipamento | CRÍTICO | Estabilizador/tomadas | liga/desliga tomada | Energia de equipamentos físicos | N/A | Obrigatório | NÃO |
| Automações de alarme armado/desarmado/disparo | alarme/segurança | CRÍTICO | `alarm_control_panel.home_alarm`, usuário, Alexa | arma/desarma/dispara, toca mídia, push | Segurança residencial | N/A | Obrigatório | NÃO |
| `Ativar alarme automaticamente e notificar` | alarme/segurança | CRÍTICO | Modo dormir, alarme | arma alarme, helper automático | Segurança residencial atrelada a modo dormir | N/A | Obrigatório | NÃO |
| `Desativar alarme ao acordar` | alarme/segurança | CRÍTICO | Modo dormir, alarme | desarma alarme | Segurança residencial | N/A | Obrigatório | NÃO |
| Blueprints `luz_seguranca_portas_noite.yaml` e `Alerta_Vazamento_com_Led.yaml` | segurança/vazamento | CRÍTICO | Portas, vazamento, usuário | luz de segurança, cena, push | Segurança/sinalização física | N/A | Obrigatório | NÃO |
| Packages V20 canônicos e `status_casa.yaml` | contrato V20 | CRÍTICO | Dashboards produtivos, aliases finais, `sensor.status_casa` | sensores, aliases, timeline/feed/push | Não são legado em decommission | N/A | Obrigatório | NÃO |

## Matriz operacional de tratamento do legado

Esta matriz define como cada classe de quarentena pode ser tratada em lotes futuros. Ela não autoriza execução automática. Qualquer ação operacional futura precisa de escopo formal, validação prévia, rollback definido e autorização explícita.

| Classe | Ação permitida | Ação proibida | Tempo mínimo de observação | Tamanho máximo do lote | Rollback obrigatório | Critérios para promoção | Critérios para desativação | Critérios de bloqueio |
|---|---|---|---:|---:|---|---|---|---|
| Quarentena C1 | Observar uso real; comparar com automação/blueprint equivalente; preparar proposta de desativação manual; documentar consumidores encontrados | Desativar em massa; remover sem validar acionadores; misturar com itens C2/C3/Bloqueado | 7 dias | 5 itens | Simples: reativar automação/script/blueprint e restaurar helper anterior se aplicável | Promover para C2 se surgir consumidor humano, helper compartilhado, notificação ou dependência de rotina diária | Pode ser candidata a desativação se não houver acionamento relevante, consumidor conhecido, side-effect crítico ou dependência de dashboard/rotina | Qualquer evidência de energia, rede, alarme, segurança, recovery, modo dormir crítico ou contrato V20 |
| Quarentena C2 | Observar por janela completa; mapear consumidores; criar substituto V20 quando for publicação/notificação; testar desduplicação | Desligar sem substituto; remover helper; alterar sem validar consumidor; misturar com itens C3/Bloqueado | 14 dias | 3 itens | Recomendado e documentado: reativação + restauração de helper/estado se houver | Promover para C3 se controlar tomada, cena física relevante, rotina diária sensível ou helper operacional compartilhado | Pode ser candidata a desativação apenas após substituto validado ou confirmação de ausência de uso real | Consumidor desconhecido, helper usado por motor V20/V20.2, push crítico, rotina de presença/modo dormir não mapeada |
| Quarentena C3 | Apenas observar; separar publicação textual da ação física; desenhar migração sem desligar ação; executar teste manual assistido em lote futuro | Desativar; remover; arquivar blueprint; alterar tomada/cena/helper; juntar com limpeza de C1/C2 | 30 dias | 1 item | Obrigatório, testado antes: reativação, restauração de helper e plano de retorno físico | Promover para Bloqueado se envolver energia, rede, alarme, recovery, segurança ou contrato V20 | Não pode ser desativada na V20.1C; só pode virar candidata após migração da ação física ou comprovação formal de inutilidade | Qualquer falha de observação, acionamento recente, dependência física, automação de rotina diária ou ausência de rollback testado |
| Bloqueado | Somente documentar; monitorar; planejar migração futura por lote formal específico; preservar comportamento atual | Desativar; remover; alterar; arquivar; editar `.storage`; alterar `sensor.status_casa`; reabrir V20.1O | Não aplicável | 0 itens | Obrigatório para qualquer lote futuro, mas bloqueado nesta fase | Só sai de Bloqueado mediante nova fase formal, fonte V20 equivalente, teste real e autorização explícita | Não aplicável nesta fase | Energia/UPS, internet/failover, alarme, recovery, segurança, contrato V20 canônico, aliases finais, dashboard produtivo ou `.storage` |

### Estratégia de execução futura

Nenhum lote abaixo está autorizado para execução pela V20.1C. A estratégia apenas define uma ordem segura para fases futuras.

| Lote | Classe alvo | Escopo máximo | Objetivo | Pré-condições | Saída esperada |
|---|---|---:|---|---|---|
| Lote 1 | Quarentena C1 | até 5 itens | Validar duplicidades simples de iluminação/conforto/laboratório e confirmar ausência de uso real | Observação mínima de 7 dias concluída; consumidores não encontrados; rollback simples definido | Lista de candidatos C1 aptos ou reclassificados |
| Lote 2 | Quarentena C1 remanescente | até 5 itens | Repetir validação para C1 restante sem misturar domínios | Lote 1 encerrado sem regressão; nenhum item crítico misturado | Nova lista de candidatos C1 aptos ou reclassificados |
| Lote 3 | Quarentena C2 - notificações e mensagens | até 3 itens | Reduzir duplicidade de push/timeline apenas quando política V20 equivalente existir | Observação mínima de 14 dias; consumidores mapeados; substituto V20 validado | Proposta de migração/desativação futura de notificações duplicadas |
| Lote 4 | Quarentena C2 - máquina, modo dormir, presença/carro | até 3 itens | Separar estado operacional de notificação humana e preservar helpers consumidos | Observação mínima de 14 dias; dependências de helpers conhecidas; teste manual planejado | Itens mantidos, migrados ou promovidos para C3 |
| Lote 5 | Quarentena C3 | 1 item | Estudar item de alto risco sem desligar ação física | Observação mínima de 30 dias; rollback testado; janela manual definida | Decisão explícita: manter, migrar ação, ou bloquear |
| Lote 6 | Bloqueado | 0 itens | Nenhuma execução; somente planejamento formal futuro | Nova autorização, fase própria, rollback e teste real | Manter bloqueado até novo lote formal |

### Lote 1 - candidatos reais para observação operacional

Lista executável inicial extraída somente do material já auditado nesta V20.1C. O lote respeita o limite de até 5 itens da matriz operacional e não autoriza desativação, remoção ou alteração de YAML; a ação permitida é observação operacional por no mínimo 7 dias, com rollback simples previamente conhecido.

| Nome | Tipo | Motivo da classificação | Consumidores conhecidos | Side-effects | Impacto esperado | Rollback | Apto para observação |
|---|---|---|---|---|---|---|---|
| `Lab - Muda o Tema Automaticamente` | automação | C1 por ser laboratório/UI, sem vínculo crítico identificado com contratos V20 | Tema do frontend | Troca automática de tema | Impacto visual apenas; não altera estado real, timeline, push ou motor V20 | Reativar automação e ajustar tema manualmente se necessário | SIM |
| `Coloca o tema Claro` | automação | C1 por ser UI/laboratório, sem dependência operacional V20 conhecida | Tema do frontend | Aplica tema claro | Impacto visual apenas; sem efeito operacional esperado | Reativar automação e restaurar tema anterior manualmente se necessário | SIM |
| `Verifica subida do sistema HA` | script | C1 por script legado com side-effect simples e sem consumidor crítico identificado | Luz/indicador local | Executa `light.turn_off` | Pode alterar somente indicador/luz local após subida do HA | Reativar script ou restaurar estado da luz manualmente | SIM |
| `Resetar Nível do Umidificador` | script | C1 por atuar em helper local sem vínculo crítico identificado | `input_number.nivel_atual_umidificador` | Atualiza helper de nível do umidificador | Pode deixar valor auxiliar do umidificador sem reset automático durante observação | Reativar script e restaurar valor do helper manualmente se necessário | SIM |
| `LAB - Reset do Não desligar Monitor` | script | C1 por rotina lab/local com helper próprio | `input_boolean.naodesliguemonitorhoje` | Desliga helper de exceção do monitor | Pode alterar apenas exceção local de não desligar monitor | Reativar script e religar o helper se necessário | SIM |

Itens C1 não incluídos neste Lote 1 permanecem em quarentena para lotes futuros. Automações de iluminação, rotinas de Home Office e blueprints com ação física ficam fora do primeiro lote por terem maior chance de impacto humano perceptível, mesmo quando classificados como baixo risco.

### Plano de observação operacional - Lote 1 C1

Este plano não autoriza decommission. Ele define somente como observar uso real dos 5 candidatos C1 iniciais, quais evidências coletar e quais critérios devem orientar uma decisão futura em lote próprio.

| Nome | Como observar se ainda é usado | Evidência a coletar | Tempo de observação | Critério para manter | Critério para desativar futuramente | Rollback | Risco residual |
|---|---|---|---:|---|---|---|---|
| `Lab - Muda o Tema Automaticamente` | Verificar histórico/logbook de execução da automação e percepção visual de troca automática de tema | Registro de acionamento, horário, usuário afetado e necessidade real da troca de tema | 7 dias | Qualquer acionamento útil, uso recorrente ou dependência visual percebida | Nenhum acionamento relevante, nenhum consumidor humano identificado e tema ajustável manualmente sem perda operacional | Reativar automação e ajustar tema manualmente para o estado esperado | Baixo; possível incômodo visual se a troca automática for esperada por algum usuário |
| `Coloca o tema Claro` | Verificar se a automação é acionada por rotina, dashboard ou usuário e se há dependência do tema claro | Registro de acionamento, origem do gatilho e efeito visual observado | 7 dias | Uso manual/automático recorrente ou dependência de leitura/visibilidade em algum painel | Ausência de acionamento e ausência de reclamação/necessidade de tema claro automático | Reativar automação e restaurar tema claro manualmente | Baixo; possível alteração de preferência visual |
| `Verifica subida do sistema HA` | Observar após restart/reload do HA se o script é chamado e qual luz/indicador é desligado | Log de chamada do script, entidade de luz afetada e efeito após subida do HA | 7 dias, incluindo ao menos 1 restart/reload observado se ocorrer naturalmente | Se o script corrigir indicador real pós-subida ou evitar estado visual incorreto | Se não houver chamada, ou se a luz afetada não tiver função operacional/visual necessária | Reativar script ou ajustar a luz manualmente ao estado esperado | Baixo a médio; pode deixar indicador/luz em estado visual inesperado após restart |
| `Resetar Nível do Umidificador` | Monitorar alterações em `input_number.nivel_atual_umidificador` e verificar se alguma rotina espera o reset | Histórico do helper, origem da alteração, uso por painel/rotina e impacto percebido no umidificador | 7 dias | Qualquer consumidor real do helper, rotina dependente do reset ou necessidade operacional do valor | Nenhum consumo identificado e ausência de impacto ao manter o helper sem reset automático | Reativar script e restaurar manualmente o valor do helper | Baixo; pode manter valor auxiliar incorreto no painel ou em rotina local |
| `LAB - Reset do Não desligar Monitor` | Observar estado de `input_boolean.naodesliguemonitorhoje` e se o reset diário/local ainda é usado | Histórico do helper, acionamento do script, usuário/rotina que depende da exceção do monitor | 7 dias | Uso real da exceção ou dependência de rotina para evitar/desfazer desligamento do monitor | Nenhum acionamento relevante e ausência de consumidor humano/rotina dependente | Reativar script e religar/desligar o helper conforme necessidade real | Baixo; monitor pode seguir política padrão quando exceção era desejada |

Condição de saída do Lote 1: ao fim da observação, cada item deve ser classificado como manter, propor desativação futura em lote formal, ou reclassificar para C2/C3/Bloqueado se surgir consumidor, side-effect relevante ou risco não mapeado. Nenhuma ação de desativação é autorizada por este plano.

### Regras de promoção e rebaixamento

- C1 passa para C2 se houver notificação, helper compartilhado, consumidor humano ou uso recorrente.
- C2 passa para C3 se houver controle físico de tomada, cena sensível, rotina diária essencial ou helper operacional crítico.
- C3 passa para Bloqueado se tocar energia, UPS, internet, failover, alarme, recovery, segurança, contrato V20, aliases finais, dashboard produtivo ou `.storage`.
- Um item só pode ser rebaixado de classe após observação completa, ausência de acionamento relevante, consumidores mapeados e rollback validado.

### Critérios gerais de desativação futura

Uma desativação futura só pode ser proposta quando todos os critérios abaixo estiverem atendidos:

- janela mínima de observação concluída;
- consumidores conhecidos e documentados;
- side-effects identificados;
- substituto V20 validado quando houver publicação/notificação;
- rollback testado ou claramente executável;
- lote pequeno e homogêneo;
- autorização explícita antes da execução.

### Critérios gerais de bloqueio

Qualquer item deve permanecer bloqueado se houver:

- ação física crítica;
- controle de energia, UPS, internet, failover, alarme, recovery ou segurança;
- dependência de `sensor.status_casa`, aliases finais ou dashboards produtivos;
- consumo por motor V20/V20.2 ainda não entendido;
- helper compartilhado sem dono claro;
- ausência de rollback;
- divergência documental entre inventário, dependências e impacto.

## V20.1C_FECHAMENTO

Data de registro: 2026-05-22.

Status: fase V20.1C encerrada formalmente como diagnóstico e governança.

### Entregas consolidadas

- Inventário concluído.
- Dependências classificadas.
- Impacto documentado.
- Automações auditadas.
- Scripts auditados.
- Blueprints auditados.
- Packages auditados.
- Risco operacional classificado.
- Quarentena controlada definida.

### Restrições preservadas

- Decommission continua bloqueado.
- V20.1O permanece congelada.
- Nenhuma limpeza está autorizada.
- Nenhuma automação está autorizada para desligamento automático.
- Nenhum script está autorizado para remoção automática.
- Nenhum blueprint está autorizado para arquivamento automático.
- Nenhuma alteração em entities, dashboards, `.storage`, packages ou YAML produtivo foi autorizada.
- Rollback é obrigatório para qualquer ação futura de limpeza, migração ou desativação.

### Próximos passos autorizados

- Observação operacional das categorias Quarentena C1 e C2 durante suas janelas mínimas.
- Preparação de lotes futuros pequenos, reversíveis e explicitamente autorizados.
- Migração futura apenas de publicação/notificação quando houver contrato V20 equivalente e consumidores conhecidos.
- Nova decisão explícita antes de qualquer desativação, limpeza ou remoção.

### Próximos passos não autorizados

- Não desligar automações automaticamente.
- Não remover scripts.
- Não arquivar blueprints.
- Não editar `.storage`.
- Não alterar `sensor.status_casa`.
- Não reabrir V20.1O.
- Não executar decommission sem lote formal, rollback e autorização explícita.

---

> Nota: esta auditoria é documental e identificatória. Nenhuma alteração de YAML de produção foi realizada nesta fase.
