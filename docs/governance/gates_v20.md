# Gates V20

Data: 2026-05-20
Status: ATIVO

## Objetivo

Definir criterios obrigatorios para encerramento de fases da Central Operacional V20.

Nenhuma fase deve ser considerada concluida sem gate documental correspondente.

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
