# Despacho Arquitetural V20.2C-A1 — Promoção Limitada da Sessão de Monitoramento Remoto

Data: 2026-08-05
Status: DECISÃO ARQUITETURAL CONSOLIDADA; IMPLEMENTAÇÃO BLOQUEADA PELO GATE
Classificação: arquitetura subordinada
Autoridade: subordinada à Constituição, ao Source of Truth, a `architecture.md` e a `docs/ARCHITECTURE.md`

## Finalidade

Consolidar a promoção limitada da V20.2C sem promover a V20.2 inteira, sem alterar a V20.1O e sem autorizar implementação, publicação em runtime ou modificação dos contratos protegidos.

Este despacho foi explicitamente autorizado como novo artefato arquitetural. `architecture.md` e `docs/ARCHITECTURE.md` permanecem canônicos; este documento registra a decisão subordinada e deve ser interpretado junto ao Gate específico da V20.2C.

## Motivação e conflito identificado

A V20.2 foi concebida como Context Engine shadow, sem publicação na Timeline. A V20.2C evoluiu para coordenar ações operacionais após a saída de Wilson e introduziu o conceito de Sessão de Monitoramento Remoto, com início, duração lógica e encerramento.

A publicação imediata foi bloqueada corretamente porque Timeline e Event Feed são contratos protegidos, V20.1O permanece congelada como política canônica e o restante da V20.2 continua shadow. A Constituição permite superar esse bloqueio somente por promoção formal, fase própria, validação e Gate.

## Parecer e promoção limitada

O parecer arquitetural é favorável à promoção exclusiva do subconjunto V20.2C responsável pelo ciclo de vida da Sessão de Monitoramento Remoto.

Classificação oficial:

> Motor oficial de coordenação operacional da Sessão de Monitoramento Remoto, com escopo restrito à saída e ao retorno de Wilson.

O componente recebe o nome oficial **Coordenador da Sessão de Monitoramento Remoto**, sigla **CSMR**. “Remote Monitoring Session Coordinator” é referência secundária; a nomenclatura principal permanece em português.

O Context Engine V20.2 original, Relevance Engine, Correlation Engine, sensores shadow e qualquer outro componente V20.2 permanecem em shadow. A promoção do CSMR não constitui autorização implícita para esses componentes.

## Modelo operacional

```text
person.wmoura
→ fonte contextual

binary_sensor.wilson_ausente_de_casa
→ abstração de autorização

Sessão de Monitoramento Remoto
→ contrato operacional confirmado

Luz da Mesa, Push da Porta e módulos futuros
→ ações subordinadas à sessão
```

A presença bruta não é a sessão. A sessão somente pode ser aberta depois da graça, da revalidação da ausência e da confirmação de que nenhuma sessão já está ativa.

## Autoridade do CSMR

O CSMR é a única autoridade sobre:

- início, duração lógica e encerramento da sessão;
- graça e revalidação da ausência;
- cancelamento antes da abertura;
- sequenciamento dos eventos oficiais;
- liberação das ações subordinadas;
- prevenção de duplicidade e sessões incompletas;
- idempotência do ciclo;
- tratamento de restart e reload;
- coordenação do rollback da sessão.

O CSMR não infere presença, não executa Presence Intelligence, não altera o Context Engine shadow, não armazena histórico, não mantém Event Feed, não implementa deduplicação paralela e não escreve em aliases finais.

Separação de autoridade:

```text
CSMR
→ decide ciclo de vida e ordem da sessão

Publicador canônico V20.1O
→ publica, armazena, deduplica e apresenta
```

## Contrato da sessão

### Entrada

```text
Ausência de Wilson confirmada
→ 📍 Wilson saiu de casa
→ 🛡️ Monitoramento remoto iniciado
→ liberar ações subordinadas
```

### Encerramento

```text
Retorno de Wilson confirmado
→ 📍 Wilson chegou em casa
→ 🛡️ Monitoramento remoto encerrado
```

O encerramento somente pode ocorrer quando existir sessão anteriormente aberta.

## Eventos oficiais autorizados

O escopo protegido desta promoção contém exatamente:

- `📍 Wilson saiu de casa`;
- `🛡️ Monitoramento remoto iniciado`;
- `📍 Wilson chegou em casa`;
- `🛡️ Monitoramento remoto encerrado`.

Nenhum outro evento V20.2C recebe autorização implícita. Alterações futuras exigem avaliação própria.

## Propriedades obrigatórias

- uma única sessão ativa por vez;
- nenhum início antes da graça;
- cancelamento durante a graça não abre sessão;
- retorno sem sessão aberta não publica encerramento;
- cada transição publica seus eventos exatamente uma vez e na ordem definida;
- ciclos consecutivos produzem sessões independentes e completas;
- restart ou reload não cria sessão fantasma;
- Harness não representa fato operacional publicável;
- falha de publicação permanece observável;
- ações subordinadas não avançam silenciosamente diante de falha crítica de abertura;
- rollback preserva V20.1O e o restante da Central.

## Ações subordinadas

- C1.1 — Luz da Mesa: `light.smart_lampada_wifi_1`;
- C1.2 — Alerta da Porta da Sala: `automation.v20_2c_teste_alertar_porta_aberta_apos_saida`;
- C1.3 — Garantia do Push da Porta: `automation.v20_2c_garantia_habilitar_push_da_porta_ao_sair`.

Esses consumidores não possuem autoridade independente para abrir ou encerrar a sessão. Módulos futuros devem declarar se atuam na abertura, durante a sessão ou no encerramento.

## Publicação canônica e proibições

A futura implementação poderá solicitar publicação somente pelo caminho canônico formalmente aprovado. Permanecem proibidos:

- escrita direta em `sensor.casa_timeline` ou `sensor.casa_event_feed`;
- alteração de aliases finais;
- Timeline, Event Feed ou histórico paralelos;
- substituição de `sensor.casa_evento_publicavel_v20`;
- deduplicação concorrente;
- substituição ou reabertura silenciosa da V20.1O.

A V20.1O permanece autoridade sobre política de publicação, armazenamento, histórico, limite, deduplicação e apresentação pública.

## Gate exigido

A implementação depende da aprovação do **Gate de Promoção Limitada V20.2C — Sessão de Monitoramento Remoto**, registrado em `docs/governance/gates_v20.md`. O Gate deve cobrir arquitetura, contrato, comportamento, regressão, falha, rollback e homologação.

## Rollback arquitetural

O rollback deve retirar somente a promoção operacional da sessão e seus consumidores subordinados, preservando V20.1O, Timeline, Event Feed, aliases finais, Context Engine shadow e o restante da Central. Nenhuma implementação poderá deixar execução pendente, sessão fantasma ou estado de Harness inseguro.

## Estado da autorização

```text
Parecer arquitetural: aprovado
Promoção limitada: consolidada documentalmente
CSMR: reconhecido como componente arquitetural oficial
Implementação técnica: não autorizada por este despacho
Publicação runtime: não habilitada
Próximo passo: plano técnico restrito e execução do Gate
```

## Referências

- `architecture.md`;
- `docs/ARCHITECTURE.md`;
- `docs/governance/gates_v20.md`;
- `docs/ROADMAP.md`;
- `docs/roadmap/roadmap_v20_consolidado.md`;
- `docs/v20_2c/README.md`;
- `docs/v20_2c/c1_saida_de_casa.md`.
