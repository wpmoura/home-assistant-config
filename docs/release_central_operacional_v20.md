# Central Operacional Home Assistant V20 - Release Baseline

Data de congelamento: 2026-05-13  
Status: baseline V20 congelada para início da V21

## Objetivo da V20

A V20 consolida a Central Operacional em uma camada paralela, resiliente e desacoplada das versões anteriores. O objetivo principal foi estabilizar contratos públicos sem versão para dashboards, criar um motor canônico de eventos, publicar timeline/feed com histórico mínimo e preservar a operação legada sem reativar V19.

## Principais evoluções

- Criação de aliases finais sem versão para consumo oficial.
- Separação entre evento dominante, timeline/feed e parâmetros operacionais.
- Timeline operacional com os 6 últimos eventos relevantes.
- Registro de eventos secundários mesmo quando existe incidente dominante ativo.
- Publicação de eventos com timestamp visível no formato `HH:MM`.
- Inclusão de TV ligada/desligada com contexto semântico quando disponível.
- Restrição da publicação de portas à Porta da Sala.
- Timeout parametrizável para Porta da Sala aberta.
- Proteção contra `unknown`, `unavailable`, `none` e estados vazios nos contratos finais.
- Preservação de V19, dashboards legados e entidades antigas no registry.

## Arquitetura consolidada

A V20 fica organizada em camadas:

- Núcleo atual de mensagens: `packages/central_mensagens_corrigido.yaml`
- Aliases finais públicos: `packages/central_operacional_aliases_v20.yaml`
- Motor canônico de evento dominante: `packages/motor_eventos_v20.yaml`
- Motor de timeline/feed operacional: `packages/motor_timeline_v20.yaml`
- Parâmetros complementares: `packages/parametros_operacionais_v20.yaml`
- Parâmetros legados e pesos históricos: `packages/status_casa.yaml`
- Testes assistidos: `packages/teste_motor_eventos_v20.yaml`

Os dashboards oficiais devem consumir preferencialmente aliases finais sem versão:

- `sensor.status_casa`
- `sensor.atividade_relevante`
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- `sensor.casa_contexto_humano`
- `sensor.casa_prioridade`
- `sensor.casa_score_operacional`

## Eventos suportados

A baseline V20 registra e/ou classifica:

- Backup Google com erro/atenção.
- Internet crítica.
- Internet normalizada.
- Failover 4G ativo.
- Failover 4G encerrado.
- Falta de energia.
- Retorno de energia.
- UPS sem leitura operacional no motor dominante.
- TV ligada.
- TV desligada.
- TV com contexto semântico, como app, canal, programação ou fonte.
- Porta sala aberta.
- Porta sala fechada.
- Porta sala aberta por timeout parametrizável.
- Chuva iniciada.
- Chuva encerrada.
- Vazamento detectado no motor dominante.

## Timeline e feed

A timeline V20 é publicada por:

- `sensor.casa_timeline_v20`
- `sensor.casa_event_feed_v20`

Os aliases finais sem versão consomem essa camada quando disponível:

- `sensor.casa_timeline`
- `sensor.casa_event_feed`

Características:

- Mantém até 6 linhas recentes por atributos `linha_1` a `linha_6`.
- Publica o evento mais recente no estado principal.
- Prefixa eventos com horário no formato `HH:MM`.
- Preserva fallback antigo `Aguardando eventos`.
- Evita duplicação consecutiva do mesmo evento ignorando o timestamp.
- Registra eventos secundários por transição de fonte, sem depender do evento dominante.

## Evento dominante

O evento dominante é calculado em `packages/motor_eventos_v20.yaml`.

Sensores principais:

- `sensor.casa_evento_dominante_v20`
- `sensor.casa_evento_atual_v20`
- `sensor.casa_evento_origem_v20`
- `sensor.casa_evento_tipo_v20`
- `binary_sensor.casa_evento_ativo_v20`

O motor dominante usa pesos operacionais e prioridade por score para escolher o evento principal, sem impedir que a timeline registre eventos secundários.

## Parâmetros operacionais

Os parâmetros V20 ficam em `packages/parametros_operacionais_v20.yaml`.

Helpers principais:

- `input_number.casa_peso_ups`
- `input_number.casa_timeout_evento_minutos`
- `input_number.casa_confianca_minima_evento`
- `input_number.casa_timeout_porta_aberta_minutos`
- `input_boolean.casa_eventos_visiveis`
- `input_boolean.casa_prioridade_contextual_ativa`

Os pesos históricos e criticidades permanecem em `packages/status_casa.yaml`.

## Timeout parametrizável

O timeout de Porta da Sala aberta usa:

- `input_number.casa_timeout_porta_aberta_minutos`

Configuração da baseline:

- inicial: `5`
- mínimo: `1`
- máximo: `60`
- unidade: `min`

O timeout vale somente para:

- `binary_sensor.sensor_porta_sala_contact`

O evento publicado é:

- `HH:MM 🚪 Porta sala aberta há X min`

## Dashboard operacional

Dashboards relacionados à V20:

- `.storage/lovelace.sistema_casa`: dashboard principal oficial.
- `.storage/lovelace.dashboard_lixo`: Parâmetros Operacionais.
- `.storage/lovelace.testes_anterior`: Engenharia Semântica em modo V20.
- `.storage/lovelace.debug_operacional`: Debug Operacional V20.
- `.storage/lovelace_dashboards`: mapeamento do menu lateral.

O dashboard de Parâmetros Operacionais contém a seção:

- `⏱️ Temporizações Operacionais`

Com:

- `input_number.casa_timeout_porta_aberta_minutos`
- `input_number.casa_timeout_evento_minutos`
- `input_number.casa_confianca_minima_evento`

## Anti-spam

A V20 evita spam por três mecanismos:

- Eventos são publicados por transição de estado ou por timeout específico.
- A timeline não registra o mesmo evento consecutivo quando só muda o timestamp.
- O timeout de Porta da Sala dispara uma vez por permanência aberta, não continuamente.

## Compatibilidade com V19 e legado

A baseline V20 preserva compatibilidade com o legado:

- Não reativa V19.
- Não remove entidades V17/V18/V19 do registry.
- Não altera `sensor.status_casa` legado fora do contrato já existente.
- Não altera automações antigas.
- Não exige mudança imediata em dashboards legados.
- Mantém fallback para `sensor.central_ultima_mensagem` quando a timeline V20 ainda não publicou eventos.

## Baseline congelada da V20

A partir deste release, a V20 deve ser tratada como baseline congelada.

Pode ser alterado sem quebrar compatibilidade:

- Textos de documentação.
- Ajustes visuais em dashboards, desde que continuem consumindo aliases finais.
- Novos atributos diagnósticos em sensores V20.
- Novos helpers opcionais que não mudem comportamento existente.
- Correções de fallback que mantenham os mesmos entity IDs públicos.

Não deve ser alterado diretamente:

- Entity IDs finais sem versão usados por dashboards oficiais.
- Semântica básica de `sensor.casa_timeline` e `sensor.casa_event_feed`.
- Nome dos sensores do motor dominante V20.
- Nome dos sensores da timeline V20.
- `sensor.status_casa` legado como forma de resolver mudanças da V20.
- Packages V19 em `_disabled`.
- Automações antigas como parte de correções da V20.

Packages considerados núcleo da V20:

- `packages/central_operacional_aliases_v20.yaml`
- `packages/motor_eventos_v20.yaml`
- `packages/motor_timeline_v20.yaml`
- `packages/parametros_operacionais_v20.yaml`

Packages de apoio:

- `packages/teste_motor_eventos_v20.yaml`
- `packages/status_casa.yaml`
- `packages/central_mensagens_corrigido.yaml`

## Mapa dos principais arquivos

### `packages/motor_eventos_v20.yaml`

Motor canônico de evento dominante. Classifica o evento principal, origem, tipo, severidade, score base, mensagem e estado ativo.

### `packages/motor_timeline_v20.yaml`

Motor de publicação da timeline/feed. Registra eventos por transição, mantém 6 eventos, trata TV, energia, internet, failover, chuva, Backup Google e Porta da Sala.

### `packages/parametros_operacionais_v20.yaml`

Helpers complementares da V20, incluindo peso de UPS, timeout de evento, confiança mínima, eventos visíveis, prioridade contextual e timeout da Porta da Sala.

### `packages/central_operacional_aliases_v20.yaml`

Contratos finais sem versão. Protege dashboards e automações futuras de sensores versionados e estados inválidos.

### Dashboards relacionados

- `.storage/lovelace.sistema_casa`
- `.storage/lovelace.dashboard_lixo`
- `.storage/lovelace.testes_anterior`
- `.storage/lovelace.debug_operacional`
- `.storage/lovelace_dashboards`

## Critérios de aceite atendidos

- Timeline com 6 eventos.
- Feed desacoplado do evento dominante.
- TV ligada/desligada registrada no feed.
- Contexto semântico de TV preservado quando disponível.
- Porta sala aberta registrada.
- Porta sala fechada registrada.
- Porta sala aberta por timeout registrada.
- Timeout parametrizável por helper.
- Eventos secundários coexistem com dominante ativo.
- Anti-spam funcional para eventos consecutivos.
- Aliases finais preservados.
- V19 preservada e não reativada.
- Dashboards produtivos mantidos.
- Automações antigas preservadas.

## Débitos técnicos conhecidos

- A timeline V20 começa a registrar eventos a partir do reload/restart; não reconstrói histórico antigo do Recorder.
- O motor dominante ainda considera outras portas no cálculo de dominância, embora a timeline/feed publique somente Porta da Sala.
- O score operacional final ainda é simplificado e não possui decomposição explicável completa.
- WAN/4G ainda tem parte da lógica em motor separado com nomenclatura versionada.
- Contexto humano e criticidade contextual ainda são iniciais.
- Testes assistidos validam cenários selecionados, mas não substituem testes de transição real no Home Assistant.

## Roadmap pós-V20

### V21 - Criticidade contextual dinâmica

Introduzir uma matriz de criticidade que combine peso base, contexto da casa, presença, horário, modo dormir, clima, conectividade e recorrência.

### Contexto humano

Evoluir `sensor.casa_contexto_humano` para descrever melhor a situação operacional, com linguagem natural controlada e sem depender de uma única mensagem textual.

### Relevância adaptativa

Ajustar relevância conforme frequência, horário, ocupação, severidade e histórico recente, reduzindo ruído sem esconder eventos importantes.

### Contexto temporal

Adicionar regras específicas para madrugada, manhã, expediente, noite, ausência prolongada, modo dormir e finais de semana.

### Integração semântica/LLM futura

Planejar uma camada opcional de síntese semântica, capaz de resumir múltiplos sinais em narrativa operacional, sem substituir os sensores determinísticos da V20.

## Próximo passo recomendado

Iniciar a V21 somente após validar em operação real:

- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- `sensor.casa_evento_dominante_v20`
- `sensor.casa_evento_publicavel_v20`
- `input_number.casa_timeout_porta_aberta_minutos`

Com a baseline V20 congelada, mudanças futuras devem ser tratadas como V21 ou hotfix V20 documentado.
