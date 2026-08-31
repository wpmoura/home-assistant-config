# AT-HC-01 — Persistência do Health Check Operacional (Gate 3)

Data: 2026-08-25
Status: **IMPLEMENTADO — aguardando homologação**
Classificação: governança de domínio (Saúde do Sistema / Health Check Operacional), frente independente
Autoridade: subordinada aos Gates 1, 2 e 2.1 do Health Check Operacional (auditoria de runtime, contrato de observabilidade e fechamento de lacunas)
Escopo deste Gate: **somente a camada de persistência**. A inteligência semântica (observar o runtime, interpretar evidências, classificar saúde) continua fora do Home Assistant, feita por Claude + HA-MCP.

## 1. Finalidade

Permitir que o Home Assistant **receba, valide, persista e exponha como entidade** o resultado de um Health Check Operacional produzido externamente, sem reimplementar nenhuma lógica de diagnóstico em Jinja/YAML. Este package não decide se algo está saudável — ele só aceita ou rejeita a FORMA de um payload já classificado, e nunca corrige, infere ou completa um payload incompleto.

## 2. Arquitetura

```
Claude + HA-MCP
  → observa runtime (system_health, logs, traces, config entries, etc.)
  → interpreta evidências e classifica saúde
  → publica evento `saude_sistema_diagnostico_publicado`
  → Home Assistant persiste o resultado em sensor.saude_sistema_status
  → sensor.saude_sistema_watchdog detecta se a publicação ficou desatualizada
  → (futuro Gate) dashboard consome as duas entidades acima, sem lógica própria
```

Padrão reaproveitado: **trigger-based template sensor alimentado por evento customizado**, o mesmo mecanismo já comprovado em produção em `packages/carro.yaml` (`sensor.carro_historico_de_odometro`, `sensor.carro_historico_de_abastecimentos`) e em `packages/contrato_publicacao_timeline_v20.yaml` (`sensor.casa_timeline_publicacao_ack_v20`). A variável `this` é usada para reaproveitar o valor anterior quando o payload recebido é inválido — idêntico ao mecanismo de idempotência/correção já usado nesses dois packages.

Arquivo: `packages/saude_sistema.yaml` (novo, isolado — não modifica nenhum outro package).

## 3. Entidades criadas

| Entidade | Tipo | Finalidade |
|---|---|---|
| `sensor.saude_sistema_status` | trigger-based template sensor | Snapshot mais recente do Health Check |
| `sensor.saude_sistema_watchdog` | trigger-based template sensor (time_pattern + evento) | Detecta se o próprio Health Check ficou desatualizado |
| `input_number.saude_sistema_watchdog_limite_horas` | helper YAML | Limite (em horas) usado pelo watchdog — configurável, valor inicial provisório |

Nota de nomenclatura: o `name` inicialmente usado ("Saúde do Sistema - ...") gera por slugificação `sensor.saude_do_sistema_...`, diferente do nome candidato definido no Gate 3. Os `entity_id` foram renomeados explicitamente via registro de entidades para `sensor.saude_sistema_status` e `sensor.saude_sistema_watchdog` (a `friendly_name` de exibição continua "Saúde do Sistema - ...").

## 4. Evento e contrato do payload

**Evento:** `saude_sistema_diagnostico_publicado`

**Campos obrigatórios:**

| Campo | Tipo | Regra de validação |
|---|---|---|
| `status` | string | Deve ser um de: `healthy`, `degraded`, `failed`, `indeterminate` |
| `checked_at` | string (ISO 8601) | Deve ser parseável como data/hora |
| `summary` | string | Não pode ser vazio |
| `source` | string | Não pode ser vazio (ex.: `gate3_test`, ou identificador real do diagnóstico) |
| `contract_version` | string | Não pode ser vazio (versão atual: `"1.0.0"`) |
| `red_count` | número | — |
| `yellow_count` | número | — |
| `indeterminate_count` | número | — |

**Campos opcionais:**

| Campo | Tipo | Regra |
|---|---|---|
| `action_today` | string | Livre |
| `dominios` | mapping | Se presente, deve ser um mapeamento (dict); cada chave é um domínio (`home_assistant`, `infraestrutura`, `rede_wan`, `automacoes`, `central_operacional`, `backups`, `integracoes_dispositivos`, `atualizacoes`, etc.) com pelo menos `status`/`summary`, e opcionalmente `evidence`/`action`/`urgency` |

**Validação (tudo ou nada):** se qualquer campo obrigatório estiver ausente, com tipo errado, ou `status` fora do enum, ou `checked_at` não parseável, ou `dominios` presente mas não for um mapeamento — **o payload inteiro é rejeitado**. Nenhum campo é parcialmente escrito; o último resultado válido é preservado integralmente, e os atributos `payload_rejeitado`/`ultima_rejeicao_em` registram que uma tentativa inválida ocorreu (sem alterar o resto do estado).

## 5. Estados permitidos e semântica

`sensor.saude_sistema_status.state` aceita exclusivamente: `healthy`, `degraded`, `failed`, `indeterminate`.

Os estados `dormant_expected` e `informational` (Gate 2/2.1) vivem **dentro** de `dominios.<nome>.status` (por domínio), nunca como estado geral do sensor principal — e, por definição, não contaminam o estado geral: a decisão de agregar tudo isso em um farol único é responsabilidade de quem publica o evento (o processo de diagnóstico), não deste sensor. Este package não recalcula nem agrega os status de `dominios`.

A apresentação visual (🟢🟡🔴⚪) é responsabilidade do futuro dashboard — este sensor só expõe os valores textuais.

## 6. Watchdog

`sensor.saude_sistema_watchdog` recalcula a cada 15 minutos (trigger `time_pattern`) e também a cada novo evento de diagnóstico, comparando `now()` com `state_attr('sensor.saude_sistema_status', 'checked_at')` — não com `last_changed`/`last_updated` da entidade, porque esses só mudam quando o **texto do estado muda**; `checked_at` muda em toda publicação válida, mesmo quando o status textual permanece igual entre duas execuções.

Estados: `ok` (dentro do limite), `atrasado` (entre 1x e 2x o limite), `sem_execucao` (acima de 2x o limite), `desconhecido` (nenhuma publicação válida ocorreu ainda — nunca é tratado como `sem_execucao`).

**O limite (`input_number.saude_sistema_watchdog_limite_horas`) é um valor provisório de 48h**, sem nenhuma cadência diária aprovada por trás — existe apenas para o mecanismo não ficar sem parâmetro nenhum, e deve ser revisto assim que uma cadência de execução real for aprovada em um gate futuro. Isso está documentado tanto no comentário do YAML quanto no atributo `contrato` do próprio sensor.

## 7. Lacunas conhecidas (preservadas como tal, não meta deste Gate)

- **CPU/memória/temperatura do host** — sem fonte instrumentada no ambiente; este package não tenta resolver isso, e o campo `dominios.infraestrutura`, se publicado, deve refletir essa lacuna como `indeterminate`/`informational` quando aplicável.
- **Timeline/ACK e Monitoramento remoto** — descobertos incidentalmente durante a implementação deste Gate ao inspecionar `packages/contrato_publicacao_timeline_v20.yaml`: o ACK da Timeline é o evento `casa_timeline_contrato_ack_v20` → `sensor.casa_timeline_publicacao_ack_v20` (campo `status`: `rejected`/`duplicate`/`validated_test`/`publish`/`published`/`failed`, com `ack_at`); os eventos `remote_monitoring_started`/`remote_monitoring_ended` são `event_code`s reais da fonte `csmr_v20_2c` nesse mesmo contrato. Este achado **não foi implementado nem instrumentado** neste Gate — fica registrado aqui para uso em um diagnóstico futuro, sem ampliar o escopo desta camada de persistência.
- **Access Points/gateway UniFi individuais** — sem evidência de vitalidade individual confirmada (Gate 2.1); não instrumentado aqui.
- **Recovery 4G** — mantém a interpretação do Gate 2.1 (`dormant_expected` quando sem execução recente); nenhum novo `input_datetime` de heartbeat foi criado neste Gate, conforme instruído.

## 8. Procedimento de publicação (para quem/o que gera o diagnóstico)

Publicar via evento customizado (ex.: `ha_call_event` do HA-MCP, ou `POST /api/events/saude_sistema_diagnostico_publicado` autenticado):

```json
{
  "source": "identificador do processo de diagnóstico",
  "status": "healthy | degraded | failed | indeterminate",
  "checked_at": "2026-08-25T15:10:00-03:00",
  "contract_version": "1.0.0",
  "summary": "resumo executivo em até ~5 linhas",
  "action_today": "texto livre, ou omitir",
  "red_count": 0,
  "yellow_count": 1,
  "indeterminate_count": 1,
  "dominios": {
    "home_assistant": {"status": "healthy", "summary": "..."},
    "backups": {"status": "informational", "summary": "..."}
  }
}
```

Um payload que falhar qualquer regra da seção 4 é silenciosamente rejeitado do ponto de vista de estado (o último resultado válido permanece).

## 9. Procedimento de diagnóstico (consulta)

- `sensor.saude_sistema_status` — estado e atributos completos do último diagnóstico válido.
- `sensor.saude_sistema_watchdog` — se esse último diagnóstico ainda é considerado atual.
- Nenhuma automação, script ou dashboard consome essas entidades ainda (fora do escopo deste Gate).

## 10. Referência de implementação

Arquivo: `packages/saude_sistema.yaml` (commit ainda não realizado — aguardando autorização/homologação).
