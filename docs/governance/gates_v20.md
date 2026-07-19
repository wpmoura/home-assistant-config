# Gates V20

Data: 2026-05-20
Status: ATIVO

## Objetivo

Definir criterios obrigatorios para encerramento de fases da Central Operacional V20.

Nenhuma fase deve ser considerada concluida sem gate documental correspondente.

## Gate corretivo V20.1Q — Recovery 4G

- [x] Tentativas genéricas e snapshot do máximo implementados estaticamente.
- [x] Tempo OFF único e helpers numerados marcados como legado.
- [x] Confirmação de queda e estabilização de retorno parametrizadas separadamente.
- [x] Cooldown restrito ao esgotamento e `ultima_execucao` com semântica documentada.
- [x] Cancelamentos sem cooldown implementados.
- [x] Validação YAML e buscas estáticas executadas.
- [x] Dashboard "Parâmetros" reorganizado em três cards (Operação, Ciclo de Recuperação, Avisos) conforme especificação oficial de UX.
- [x] Especificação oficial de UX documentada e versionada em `docs/ux/espec_ux_param_recovery4g.md`.
- [x] Commit e push da entrega de UX do dashboard "Parâmetros" realizados, com rastreabilidade em `CHANGELOG.md` e `docs/releases/implementation_plan_v20_1q.md`.
- [ ] Validação visual do critério A8 da especificação de UX (nenhum rótulo quebra em duas linhas em viewport de celular) — sem evidência visual registrada.
- [x] Cenários 1, 5 e 10 homologados no runtime com quedas reais controladas (Testes 1, 3 e 2, respectivamente). Cenário 2 considerado suficientemente coberto por generalização de código (laço genérico único, sem hardcode por valor, confirmado por leitura de código e por execução real em três valores distintos) e não é bloqueador de encerramento.
- [x] Snapshot dos parâmetros (`max_tentativas_ciclo`) validado contra alteração de helper em pleno ciclo — prova definitiva no Teste 2.
- [x] Cooldown homologado — entrada e expiração, com trace de ação real da transição `cooldown → ocioso` (Testes 1 e 2).
- [x] Religamento de segurança e proteção da tomada contra permanência desligada homologados (Testes 1–3).
- [x] Erro técnico seguro / falha intermediária sem decisão autônoma do Executor homologado (achado real não planejado no Teste 3: atraso de confirmação da tomada tratado corretamente, sem corromper o ciclo).
- [x] Restart durante ciclo ativo homologado (reconciliação limpa, sem cooldown).
- [x] Timeline validada com o limite de 16 eventos em produção.
- [ ] Oscilação, `unknown` e `unavailable` dos sensores — sem ocorrência real observada em nenhum teste; permanece sem evidência.
- [ ] Janela de estabilização igual a zero — parametrizada em dois testes, mas nunca exercida de fato (o ciclo foi direto a timeout antes de alcançar a lógica de estabilização nas duas vezes).
- [ ] Retorno estabilizado em índice intermediário (sucesso antes do esgotamento) — não obtido; as quedas reais tentadas duraram mais que a janela de tentativas disponível.
- [ ] Cancelamento pelo operador em ciclo ativo — tentativa dedicada não obteve sucesso por limitação de monitoramento (reação tardia); recomenda-se assinatura de eventos (WebSocket/`subscribe_events`) em vez de polling na próxima tentativa.
- Interrupção por falta de energia em ciclo ativo — **classificada como risco residual aceito**, não bloqueador. Mesma condição de código do cancelamento pelo operador, já comprovada por analogia estrutural (leitura de código), mas sem execução real com ciclo ativo. Não deve ser forçada deliberadamente.

### Estado da homologação runtime — Suspensa

**Status:** Homologação Suspensa.

**Motivo:** interrupção por decisão operacional. Não existe bloqueio técnico conhecido. A implementação permanece válida.

**Evidências preservadas (não precisam ser repetidas):** Recovery 4G funcional de ponta a ponta; snapshot dos parâmetros; parametrização das tentativas (cenários 1, 5 e 10); cooldown; expiração do cooldown; religamento de segurança; tratamento de erro técnico; Timeline; estados do Executor.

**Permanecem para retomada futura:** cancelamento pelo operador em ciclo ativo; retorno antes do esgotamento (índice intermediário); janela de estabilização igual a zero.

Detalhamento completo dos três testes executados (Teste 1, Teste 2, Teste 3), evidências e próxima etapa recomendada em `docs/releases/implementation_plan_v20_1q.md`.

Três power cycles reais e controlados foram autorizados e executados pelo usuário durante esta rodada de homologação runtime (Testes 1, 2 e 3), conforme Gate pré-teste físico.

## Gate 0 - Escopo

Obrigatorio antes de iniciar.

- Fase nomeada.
- Objetivo declarado.
- Limites declarados.
- Itens fora de escopo declarados.
- Tipo da fase definido: documentacao, auditoria, shadow, implementacao, migracao ou limpeza.

## Gate 1 - Contratos protegidos

Obrigatorio para qualquer fase que possa afetar operacao.

Validar que nao houve alteracao indevida em:

- `sensor.status_casa`
- aliases finais sem versao
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- dashboards produtivos
- automacoes criticas

## Gate 2 - Arquitetura

Obrigatorio para novas camadas, motores ou decisoes operacionais.

- A fase respeita a Constituicao V20.
- Nao cria inteligencia paralela sem classificacao.
- Roadmap nao esta sendo usado como implementacao.
- YAML nao redefine arquitetura.
- Camada shadow permanece desacoplada ate promocao formal.

## Gate 3 - Evidencia

Obrigatorio para homologacao.

- Evidencias ou resultados registrados.
- Falhas, bloqueios e parciais declarados.
- Ausencia de spam, duplicidade e `unavailable` avaliada quando aplicavel.
- Impacto em timeline/event feed declarado quando aplicavel.

## Gate 4 - Rollback

Obrigatorio para implementacao, migracao ou limpeza.

- Rollback simples identificado.
- Lote pequeno e reversivel.
- Nenhuma edicao manual de `.storage` sem fase propria.
- Nenhuma remocao automatica de automacoes orfas/desabilitadas.

## Gate 5 - Documentacao

Obrigatorio no encerramento.

- Changelog ou checkpoint atualizado.
- Roadmap consolidado atualizado quando houver impacto futuro.
- Backlog tecnico atualizado quando houver pendencia.
- Auditoria registrada quando a fase for diagnostica.

## Gate 6 - Status final

Status permitido:

- `Planejada`
- `Em diagnostico`
- `Implementada em shadow`
- `Homologada`
- `Concluida`
- `Bloqueada`
- `Parcial`
- `Arquivada`

Uma fase homologada deve ter escopo fechado e nao deve continuar acumulando mudancas sem nova fase.

## Gates especificos - V20.1Q Recovery 4G

### Gate documental

- Auditoria, despacho arquitetural e Implementation Plan presentes e referenciados.
- Classificacao e subordinacao documental declaradas.
- Lacunas de cooldown e timeout numericos registradas sem inferencia.

### Gate pre-implementacao

- Nova Etapa A executada sobre `develop` sincronizada.
- Helpers equivalentes, consumidores, tomada, blueprint e automacao confirmados.
- Lista exata de arquivos apresentada antes de alteracao funcional.
- Persistencia, idempotencia, restart seguro, cancelamento e rollback definidos.
- Fronteira V20.1Q.1/V20.1Q.2 preservada.

### Gate pre-teste fisico

- YAML e configuracao Home Assistant validados.
- Tomada correta e caminho de religamento confirmados.
- Ausencia de detector proprio, ping ou interpretacao de bytes no Executor confirmada.
- Rollback preparado.
- Autorizacao operacional explicita do usuario registrada antes de qualquer power cycle.

### Gate de homologacao

- Cenarios de recovery desabilitado, tentativas 1 e 2, cooldown, timeout, concorrencia, restart e erro executados.
- Nenhuma tentativa além do snapshot configurado observada.
- Central confirmada como unica decisora e validadora.
- Timeline, Push, aliases finais, `sensor.status_casa` e V20.1O preservados.

### Gate de encerramento

- Evidencias e pendencias registradas.
- Changelog/checkpoint e Roadmap atualizados.
- Legado preservado ou tratado somente por fase propria.
- Nenhuma edicao manual de `.storage`.
