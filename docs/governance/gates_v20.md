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
- [ ] Cenários 1, 2, 5 e 10 homologados no runtime.
- [ ] Oscilação, `unknown`, `unavailable` e janela zero homologados.
- [ ] Restart, energia, operador e segurança da tomada homologados.
- [ ] Timeline com 16 eventos validada após restart controlado.

O gate permanece aberto. Nenhum reload, restart ou power cycle foi autorizado nesta implementação.

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
