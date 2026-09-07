# Home Assistant Config - Central Operacional

Repositório de configuração do Home Assistant com foco na **Central Operacional**, uma camada de leitura, priorização e publicação de eventos da casa.

## Visão Geral

Este projeto organiza packages, dashboards e documentação para transformar sinais do Home Assistant em uma leitura operacional mais clara: estado da casa, evento dominante, timeline, feed, contexto humano, prioridade e score operacional.

A baseline atual é a **Central Operacional V20**, congelada em 2026-05-13.

## Objetivo da Central Operacional

A Central Operacional tem como objetivo consolidar eventos domésticos relevantes em contratos estáveis, resistentes a estados inválidos e desacoplados de versões experimentais.

Ela busca responder rapidamente:

- O que está acontecendo agora?
- Qual evento é dominante?
- Quais eventos recentes compõem a timeline?
- Existe incidente ativo?
- O dashboard pode funcionar sem depender de sensores versionados?

## Principais Capacidades

- Aliases finais sem versão para dashboards oficiais.
- Motor canônico de evento dominante.
- Timeline operacional com os 6 últimos eventos relevantes.
- Feed operacional desacoplado do evento dominante.
- Registro de eventos secundários mesmo durante incidentes ativos.
- TV ligada/desligada com contexto semântico quando disponível.
- Porta da Sala aberta/fechada e alerta por timeout.
- Timeout parametrizável para Porta da Sala aberta.
- Proteção contra `unknown`, `unavailable`, `none` e vazio.
- Preservação de V19 e legado sem reativação.

## Arquitetura Resumida

A arquitetura V20 é organizada em camadas:

- `central_mensagens_corrigido.yaml`: núcleo operacional real atual de mensagens.
- `central_operacional_aliases_v20.yaml`: aliases públicos sem versão.
- `motor_eventos_v20.yaml`: classificação do evento dominante.
- `motor_timeline_v20.yaml`: publicação de timeline/feed com histórico de 6 eventos.
- `parametros_operacionais_v20.yaml`: helpers e parâmetros complementares.
- `status_casa.yaml`: pesos e parâmetros históricos, preservado como legado/governança.

## Organização dos Packages

Os packages ficam em `packages/` e seguem o padrão moderno do Home Assistant com raiz em dicionário.

Packages núcleo da V20:

- `packages/central_operacional_aliases_v20.yaml`
- `packages/motor_eventos_v20.yaml`
- `packages/motor_timeline_v20.yaml`
- `packages/parametros_operacionais_v20.yaml`

Packages de apoio:

- `packages/teste_motor_eventos_v20.yaml`
- `packages/central_mensagens_corrigido.yaml`
- `packages/status_casa.yaml`

## Evento Dominante

O evento dominante é calculado pelo motor V20 e exposto por:

- `sensor.casa_evento_dominante_v20`
- `sensor.casa_evento_atual_v20`
- `sensor.casa_evento_origem_v20`
- `sensor.casa_evento_tipo_v20`
- `binary_sensor.casa_evento_ativo_v20`

Esse motor decide o evento principal por peso, severidade e origem, sem impedir que eventos secundários sejam registrados na timeline.

## Timeline e Feed Operacional

A timeline V20 registra eventos por transição de estado e mantém até 6 eventos recentes:

- `sensor.casa_timeline_v20`
- `sensor.casa_event_feed_v20`

Aliases finais consumidos por dashboards:

- `sensor.casa_timeline`
- `sensor.casa_event_feed`

Todos os eventos publicados seguem o formato:

```text
HH:MM mensagem
```

## Parâmetros Operacionais

Parâmetros V20 principais:

- `input_number.casa_peso_ups`
- `input_number.casa_timeout_evento_minutos`
- `input_number.casa_confianca_minima_evento`
- `input_number.casa_timeout_porta_aberta_minutos`
- `input_boolean.casa_eventos_visiveis`
- `input_boolean.casa_prioridade_contextual_ativa`

## Status Atual da Baseline

Status: **V20 congelada**.

Documento de release:

- `docs/release_central_operacional_v20.md`

A partir deste ponto, mudanças estruturais devem ser tratadas como V21 ou como hotfix documentado da V20.

## Roadmap Resumido V21+

- V21: criticidade contextual dinâmica.
- V22: motor semântico.
- V23: observabilidade operacional.
- V24: IA/LLM e contexto adaptativo.

Mais detalhes em:

- `docs/ROADMAP.md`

## Estrutura de Diretórios

```text
.
├── docs/
│   ├── ARCHITECTURE.md
│   ├── CHANGELOG.md
│   ├── GIT_STRATEGY.md
│   ├── ROADMAP.md
│   └── release_central_operacional_v20.md
├── packages/
│   ├── central_operacional_aliases_v20.yaml
│   ├── motor_eventos_v20.yaml
│   ├── motor_timeline_v20.yaml
│   ├── parametros_operacionais_v20.yaml
│   └── status_casa.yaml
├── configuration.yaml
├── README.md
└── CHANGELOG.md
```

## Versionamento Git

Fluxo recomendado:

```bash
git status --short --branch --untracked-files=no
git diff --cached --name-status
git commit -m "chore: freeze Central Operacional V20 baseline"
git tag v20-central-operacional
git push -u origin main
git push origin v20-central-operacional
```

Consulte:

- `docs/GIT_STRATEGY.md`

## Arquivos Sensíveis

Nunca commitar:

- `secrets.yaml`
- `home-assistant_v2.db*`
- `*.log`
- `.storage/auth*`
- `.storage/core.config_entries`
- `.storage/core.device_registry`
- `.storage/core.entity_registry`
- chaves SSH
- arquivos temporários/cache

O `.gitignore` foi preparado para proteger esses arquivos.

## Tecnologias Utilizadas

- Home Assistant
- YAML packages
- Jinja templates
- Lovelace dashboards
- Git/GitHub

## Screenshots Futuras

Reservado para imagens dos dashboards oficiais:

- Dashboard principal da Central Operacional.
- Parâmetros Operacionais.
- Engenharia Semântica.
- Debug Operacional V20.

## Estado Atual do Projeto

Central Operacional V20 congelada e documentada. O próximo ciclo recomendado é a V21, com foco em criticidade contextual dinâmica, contexto humano e relevância adaptativa.
