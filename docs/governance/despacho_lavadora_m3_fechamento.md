# Despacho de Governança — Encerramento do Gate M3 (Lavadora)

Data: 2026-08-15
Status: DECISÃO DE GOVERNANÇA CONSOLIDADA; DOCUMENTAL — NENHUMA ALTERAÇÃO FUNCIONAL, RELOAD, RESTART OU PUBLICAÇÃO AUTORIZADA POR ESTE DESPACHO
Classificação: governança subordinada, frente independente (fora da numeração V20.x)
Autoridade: subordinada à Constituição, ao Source of Truth, a `architecture.md`/`docs/ARCHITECTURE.md` e a `AGENTS.md`

## Finalidade

Consolidar como decisão formal de governança as três hipóteses de engenharia (EH-1, EH-2, EH-3) adotadas na implementação da FSM de sessão da lavadora (M2) e homologadas pelo Harness de teste isolado (M3), registrando o estado arquitetural efetivamente comprovado em runtime shadow. Este despacho não promove `source = lavadora`, não promove nenhum `event_code`, não altera o classificador produtivo e não autoriza M4. Referencia `packages/lavadora_sessao.yaml` (M1+M2+M3) e a revisão formal EH-1/EH-2/EH-3 realizada antes da abertura do M3.

## 1. Decisões formalizadas

### EH-1 — Indisponibilidade durante CONFIRMING_START

**Contrato aprovado e homologado:**

```text
CONFIRMING_START
      ↓
unknown/unavailable
      ↓
IDLE
```

**Consequências formalizadas:**

- a confirmação de início é cancelada, nunca congelada;
- nenhum `session_id` novo é criado nesse caminho;
- dado indisponível nunca confirma atividade física;
- uma nova sequência válida de potência deverá satisfazer novamente, do zero, o critério de início (`P_start` sustentado por `T_start`).

**Evidência M3:** cenário M3-03 — duas execuções independentes, uma com `unavailable` e outra com `unknown`, ambas partindo de `idle → confirming_start` (200 W) e retornando a `idle` antes do vencimento de `T_start`, com `session_id` inalterado em ambas. Corroborado indiretamente por M3-02 (pulso sem dado ausente, mesmo destino) e M3-15 (spike alto em `idle`, que nunca chega a comprometer a identidade da sessão).

### EH-2 — Indisponibilidade durante CONFIRMING_END

**Contrato aprovado e homologado:**

```text
CONFIRMING_END
      ↓
unknown/unavailable
      ↓
ACTIVE
```

**Consequências formalizadas:**

- a confirmação de término é invalidada, nunca parcialmente contabilizada;
- `fim_candidato_desde` é limpo;
- `session_id` é preservado;
- `centrifugacao_detectada` é preservada;
- tempo indisponível nunca contribui para `Tfim`;
- um novo período integralmente válido de silêncio é exigido para reabrir a confirmação de término.

**Evidência M3:**
- M3-07 — duas execuções (`unavailable` e `unknown`) a partir de `confirming_end`, ambas retornando a `active` com `fim_candidato_desde` limpo, `session_id` e `centrifugacao_detectada` preservados.
- M3-08 — alternância repetida `unavailable` ↔ silêncio válido por ~40 s sem jamais atingir `idle` por timeout, comprovando que a indisponibilidade não acumula tempo a favor de `Tfim`.
- M3-09 — cenário combinado obrigatório: silêncio legítimo → `confirming_end` → dropout (`unknown`) → reset via EH-2 para `active` → novo silêncio → retomada de atividade real antes de um novo `Tfim`. Nenhuma finalização, nenhuma sessão nova, `session_id` preservado do início ao fim da sequência.

### EH-3 — Retenção da identidade após encerramento

**Contrato aprovado e homologado:**

```text
sessão ABC termina
      ↓
estado_fsm = idle
session_id = ABC
      ↓
nova atividade candidata
      ↓
confirming_start
session_id = ABC
      ↓
nova sessão confirmada
      ↓
active
session_id = XYZ
```

**Formalização obrigatória do contrato de leitura:**

> `lavadora_sessao_id` nunca deve ser interpretado isoladamente como indicação de sessão ativa. A interpretação correta é sempre pareada: `estado_fsm + session_id`.

```text
idle + ABC             = última sessão encerrada identificada como ABC
confirming_start + ABC = nova sessão ainda não confirmada; ABC continua sendo a identidade anterior
active + XYZ           = sessão XYZ atualmente confirmada
confirming_end + XYZ   = sessão XYZ continua ativa semanticamente, aguardando confirmação de término
```

O novo `session_id` nasce **exclusivamente** na transição `CONFIRMING_START → ACTIVE`, exatamente uma vez por sessão confirmada.

**Evidência M3:** cenário M3-13 (leitura pareada `idle + session_id` capturada imediatamente após o término real de M3-06, confirmando retenção) seguido por M3-14 (segunda sessão consecutiva): o `session_id` da sessão anterior (`51a239cd-…`) permaneceu presente durante todo o `confirming_start` da nova tentativa e só foi substituído por um novo identificador (`1ac9f265-…`) no instante exato em que `T_start` venceu e a FSM confirmou `active` — troca ocorrida exatamente uma vez, nunca antes.

## 2. Evidência do Harness

O Gate M3 encerrou com:

```text
GO — FSM shadow homologada pelo Harness; apta a solicitar abertura de M4
```

Todos os cenários obrigatórios da matriz M3-01 a M3-16 (início normal, pulso falso, EH-1, ativa/múltiplas leituras, pausa interna, término real, EH-2 breve e prolongada, pausa + dropout combinados, centrifugação simples e múltipla, ciclo sem centrifugação, EH-3/retenção, segunda sessão consecutiva, spike fora de sessão, timestamp malformado) registraram **PASS**, com prova negativa de Timeline e prova de não interferência do classificador produtivo igualmente **PASS**.

Cenários de reload, explicitamente distintos dos demais por dependerem de infraestrutura que o Gate não reproduz integralmente:

```text
M3-17 / reload durante ACTIVE:          PARCIALMENTE VALIDADO
M3-18 / reload durante CONFIRMING_END:  PARCIALMENTE VALIDADO
```

Limitação explícita registrada em ambos: `reload != restart completo do Home Assistant`. `automation.reload` recarrega apenas as definições de trigger/ação da automação; os helpers `input_*` que carregam `session_id`, `estado_fsm`, `centrifugacao_detectada` e `fim_candidato_desde` não são recriados por esse caminho, portanto o mecanismo de recuperação de estado (`RestoreEntity` a partir do Recorder, que é o que de fato entra em jogo num restart real) **não foi exercitado**. Os dois cenários comprovam apenas que um reload de automação não perturba uma sessão em curso — não constituem evidência de comportamento pós-restart físico, e não devem ser lidos como tal.

## 3. Defeitos encontrados pelo M3

Registrados como evidência de efetividade do Harness — ambos foram encontrados, corrigidos e revalidados dentro do próprio Gate M3, antes do seu fechamento.

### TypeError em `fim_elapsed_min`

```text
Causa:
  datetime nativo reaproveitado entre chaves de `variables:`
  → rerenderização como string na chave seguinte
  → subtração datetime - string

Correção homologada:
  recalcular/converter com as_datetime(fim_raw)
  dentro da mesma expressão responsável pela subtração
  (nunca passar o datetime cru entre variáveis)
```

Este defeito só se manifestava quando `fim_candidato_desde` continha um timestamp real dentro de `confirming_end` — condição nunca exercitada durante M2 (que nunca chegou a popular esse campo com um valor não vazio em runtime). O caminho foi reexecutado após a correção (cenários M3-07 em diante) sem qualquer recorrência do erro.

### Schema inválido `restore`

`homeassistant.check_config` **não detectou** a chave de configuração inválida (`restore:` em `input_boolean`/`input_select`/`input_number`, chave que não existe nesses schemas) nas duas vezes em que essa condição esteve presente no arquivo. Apenas o `reload` real dos respectivos componentes revelou o erro de validação.

**Aprendizado de governança registrado:**

> `check_config` aprovado não substitui validação de materialização/reload quando o Gate altera entidades ou configuração carregável.

Este aprendizado é restrito à evidência observada nesta sessão (chaves extras de schema em helpers `input_*`); não se generaliza a outras classes de erro de configuração sem nova evidência.

## 4. Shadow mode homologado

```text
FSM nova:                                          homologada em shadow mode
Timeline:                                           nenhuma publicação da nova FSM
source = lavadora:                                  ainda NÃO promovido
event_codes:                                        ainda NÃO promovidos
classificador bruto:                                continua produtivo
input_boolean.atividade_maquina_lavar_habilitada:   on
M4:                                                 não executado
```

## 5. Estado da frente

```text
Diagnóstico                         CONCLUÍDO
Calibração histórica n=5            CONCLUÍDA
Desenho FSM                         CONCLUÍDO
Governança contrato Timeline        GO
M1 — Helpers/parâmetros             GO
M2 — FSM shadow                     CONCLUÍDO
EH-1/EH-2/EH-3                      HOMOLOGADAS
M3 — Harness                        GO
M4 — Promoção contrato              PENDENTE / NÃO AUTORIZADO
M5 — Cutover                        NÃO AUTORIZADO
```

Este despacho não declara a nova solução produtiva. O classificador bruto (`sensor.casa_atividade_operacional_v20`) permanece a única autoridade operacional da lavadora; a FSM nova permanece exclusivamente em shadow mode.

## Próximo Gate

Os critérios técnicos para **solicitar** a abertura do M4 foram atingidos pelo Harness (Seção 2). Este despacho não abre, autoriza ou executa M4 — apenas registra que o pedido formal de M4 pode ser feito. M4 continua expressamente **NÃO AUTORIZADO** até Gate específico com aprovação explícita.

## Anexos

- `docs/governance/evidencia_lavadora_ciclo_real_2026_08_15.md` — evidência de campo de um ciclo físico real completo exercitando esta FSM shadow em 15/08/2026, posterior a este despacho. Não constitui homologação produtiva nem autoriza M5; ver ressalva obrigatória no próprio anexo.
