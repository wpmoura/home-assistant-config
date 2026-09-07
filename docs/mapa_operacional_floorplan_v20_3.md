# Mapa Operacional / Floorplan V20.3 - Backlog Tecnico

## Objetivo

Registrar a frente futura de implementação de um mapa visual do apartamento no Home Assistant usando Picture Elements.

O Mapa Operacional deve ser uma camada visual de observabilidade, não um motor de decisão. Ele deve ajudar a enxergar estado da casa por cômodo, dispositivo ou domínio, sem alterar sensores produtivos, aliases finais, timeline/feed, `sensor.status_casa` ou automações existentes.

Imagem base prevista:

- `/config/www/floorplan/apto.png`

## Status

Status: backlog futuro.

Não implementar nesta fase.

## Escopo Futuro

O mapa deve exibir camadas sob demanda:

- WiFi / UniFi
- Câmeras
- Movimento / Presença
- Segurança / Portas
- Eventos contextuais futuros

Cada camada deve poder ser ligada/desligada de forma explícita para evitar poluição visual.

## Diretrizes Arquiteturais

- Não integrar ainda com decisão oficial V20.2.
- Não publicar eventos na timeline.
- Não alterar `sensor.status_casa`.
- Não alterar aliases finais.
- Não alterar automações produtivas.
- Não consumir sensores shadow como fonte produtiva.
- Tratar o mapa como dashboard visual/observabilidade.
- Manter o Radar de Movimento sob demanda, controlado por helper.
- Quando o helper de radar estiver desligado, o mapa deve permanecer limpo.
- IA/LLM não deve ser usada nesta fase.
- A futura camada de IA deve continuar opcional e desligável.

## Relação com a Central Operacional

O Mapa Operacional deve ficar abaixo dos motores determinísticos.

Fluxo correto:

```text
Sensores reais
        │
        ▼
Sensores determinísticos / semânticos
        │
        ▼
Mapa operacional visual
```

Anti-padrão:

```text
Mapa operacional
        │
        ▼
Decisão crítica / timeline / score
```

O mapa pode consumir estados semânticos estabilizados, mas não deve virar fonte de decisão operacional.

## Radar de Movimento Sob Demanda

O Radar de Movimento deve ser tratado como uma camada opcional dentro do mapa.

Helper futuro sugerido:

- `input_boolean.casa_radar_movimento_ativo`

Regras:

- desligado: não exibir badges, chips, ícones ou cartões de movimento
- ligado: exibir somente cômodos com movimento/presença ativa naquele momento
- sem movimento ativo: não exibir botões, badges ou chips de cômodos
- não ocupar espaço fixo no dashboard principal
- não publicar eventos na timeline
- não alterar relevância, confiança ou score

## Camadas Previstas

### WiFi / UniFi

Objetivo:

- visualizar estado geral de rede e pontos relevantes
- destacar falhas de conectividade, se houver sensores finais adequados

Regras:

- não desligar/reiniciar equipamentos pelo mapa nesta fase
- apenas leitura/observabilidade

### Câmeras

Objetivo:

- exibir pontos de câmera ou atalhos visuais
- permitir acesso rápido a visualizações existentes

Regras:

- não criar automações de segurança nesta fase
- não publicar movimento detectado por câmera na timeline sem motor próprio

### Movimento / Presença

Objetivo:

- consolidar sensores PIR, mmWave, presença, ocupação e sensores específicos por cômodo
- incluir sensores especiais como o box do banheiro sem tratá-los como movimento genérico quando houver semântica própria

Regras:

- radar desligado por padrão
- não poluir dashboard principal
- usar camada semântica futura por cômodo

### Segurança / Portas

Objetivo:

- visualizar portas/janelas abertas ou fechadas por cômodo
- usar sensores determinísticos já existentes quando disponíveis

Regras:

- não substituir alertas de segurança
- não alterar automações de alarme
- não alterar timeline/feed

### Eventos Contextuais Futuros

Objetivo:

- permitir indicação visual futura de eventos contextuais relevantes
- exemplo: chuva + janela aberta, energia ausente, internet degradada, presença inesperada

Regras:

- só integrar após validação da V20.2
- não consumir sensores shadow como fonte produtiva
- usar apenas contratos finais/semânticos promovidos em fase própria

## Arquivos Futuros Propostos

Packages:

- `packages/mapa_operacional_v20_3.yaml`

Dashboard/Lovelace:

- `dashboards/mapa_operacional_v20_3.yaml`
- ou documentação equivalente para Lovelace em modo storage

Assets:

- `/config/www/floorplan/apto.png`
- `/config/www/floorplan/icons/`

## Helpers Futuros Sugeridos

Helpers possíveis:

- `input_boolean.casa_mapa_operacional_ativo`
- `input_boolean.casa_radar_movimento_ativo`
- `input_boolean.casa_mapa_camada_wifi`
- `input_boolean.casa_mapa_camada_cameras`
- `input_boolean.casa_mapa_camada_movimento`
- `input_boolean.casa_mapa_camada_seguranca`
- `input_boolean.casa_mapa_camada_eventos_contextuais`

Esses helpers são apenas proposta. Não devem ser criados antes da validação da V20.2 Fase 1A.

## Sensores Semânticos Futuros

Possíveis sensores futuros:

- `sensor.casa_comodos_movimento_ativo`
- `sensor.casa_radar_movimento_resumo`
- `sensor.casa_mapa_estado_visual`
- `sensor.casa_mapa_evento_contextual_visual`

Diretriz:

- criar sensores finais/semânticos antes de colocar lógica pesada no dashboard
- evitar templates complexos dentro do Picture Elements
- manter dashboard como camada de apresentação

## Ordem Recomendada

1. Finalizar testes reais V20.2 Fase 1A.
2. Implementar helpers do mapa.
3. Criar dashboard Picture Elements.
4. Integrar Radar de Movimento sob demanda.
5. Integrar eventos contextuais e IA opcional somente em fase posterior.

## Critérios para Avançar

Antes de implementar o mapa:

- V20.2 Fase 1A validada com evidências.
- Sensores shadow confirmados como não produtivos.
- Timeline/feed sem dependência da camada visual.
- Aliases finais preservados.
- `sensor.status_casa` preservado.
- Imagem base `apto.png` disponível em `/config/www/floorplan/`.
- Estratégia de dashboard definida: YAML mode, storage mode ou documentação Lovelace manual.

## Riscos

- Poluição visual no dashboard principal.
- Misturar observabilidade com decisão operacional.
- Criar lógica pesada dentro do Lovelace.
- Consumir sensores shadow como se fossem produtivos.
- Transformar radar de movimento em vigilância permanente, em vez de ferramenta sob demanda.
- Introduzir IA/LLM antes de estabilizar contratos determinísticos.

## Mitigações

- Usar helper de ativação para radar.
- Manter mapa como dashboard/camada separada ou seção condicional.
- Consumir apenas sensores finais/semânticos promovidos.
- Não publicar em timeline/feed.
- Não alterar score, aliases ou `sensor.status_casa`.
- Manter IA opcional, desligável e fora de decisões críticas.

