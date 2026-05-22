# Constituicao da Central Operacional V20

Data: 2026-05-20
Status: ATIVO

## Proposito

Estabelecer as regras permanentes da Central Operacional V20 para impedir divergencia entre documentacao, arquitetura, roadmap e implementacao.

## Principios

### 1. Fonte unica de verdade

A Central Operacional deve manter uma hierarquia documental clara. Documentos antigos, auditorias e checkpoints nao substituem a fonte da verdade ativa.

### 2. Nenhuma inteligencia paralela

Nao deve existir nova inteligencia operacional concorrendo com a V20 sem classificacao formal.

Qualquer motor, automacao ou camada contextual deve ser uma destas categorias:

- motor oficial;
- camada shadow;
- legado ativo;
- auditoria;
- experimento documentado.

### 3. Roadmap nao altera contratos

Roadmap, backlog e planejamento nao alteram sensores finais, aliases publicos, dashboards produtivos ou comportamento operacional.

Toda alteracao de contrato exige fase propria, validacao e gate.

### 4. Auditorias sao fotografia

Auditorias registram o estado observado em uma data. Elas nao autorizam limpeza, migracao, remocao ou mudanca operacional por si so.

### 5. YAML nao redefine arquitetura

Packages, automacoes e templates implementam decisao aprovada. A existencia de YAML nao torna uma arquitetura oficial.

### 6. Nenhuma fase encerra sem gates

Toda fase deve declarar escopo, evidencias, riscos, rollback e status antes de ser encerrada.

### 7. Contratos publicos sao protegidos

Sensores finais, aliases publicos, timeline e event feed devem ser preservados ate decisao formal.

Contratos protegidos incluem:

- `sensor.status_casa`
- `sensor.casa_timeline`
- `sensor.casa_event_feed`
- `sensor.atividade_relevante`
- aliases finais sem versao

### 8. V20.2 permanece shadow ate promocao formal

Camadas V20.2 podem observar e produzir sensores versionados, mas nao devem alterar producao sem fase de promocao.

### 9. IA e opcional

Qualquer camada de IA deve ser assistiva, desacoplada e explicitamente controlavel. A casa deve funcionar sem IA.

### 10. Limpeza deve ser pequena e reversivel

Limpeza de legado, automacoes, dashboards, helpers ou entidades residuais deve ocorrer em lotes pequenos, com rollback e validacao.

### 11. Crescimento documental deve ser governado

Documentos grandes, historicos ou em expansao nao autorizam criacao automatica de novos arquivos, extracao automatica, mudanca de canonicidade ou decisao arquitetural implicita.

## Politica Constitucional de Crescimento Documental

### Principios gerais

1. Arquivos novos permanecem proibidos por padrao.
2. Arquivos novos somente podem ser criados mediante aprovacao explicita, justificativa, classificacao e referencia canonica.
3. Documento grande demais nao autoriza automaticamente novo arquivo, extracao, refatoracao documental ou mudanca estrutural.
4. Codex pode medir, sinalizar e recomendar. Codex nao pode decidir sozinho, executar extracao automaticamente ou alterar canonicidade.
5. Extracao documental deve ser controlada, aprovada, registrada, rastreavel e vinculada ao documento pai.
6. Documento pai permanece como fonte canonica ou indice canonico.
7. Documento extraido deve possuir classificacao obrigatoria:
   - canonico;
   - auxiliar;
   - release;
   - historico;
   - transitorio.
8. Documento auxiliar nao substitui fonte canonica.
9. Documento transitorio deve possuir objetivo, duracao e criterio de encerramento.
10. Nenhum documento criado pode autorizar implementacao, substituir aprovacao formal ou criar decisao arquitetural implicita.

### Criterios objetivos para sinalizacao

Sinalizar documento candidato a revisao arquitetural/documental quando ocorrer qualquer condicao:

- mais de 1000 linhas;
- ou mais de 10 secoes principais;
- ou mais de 3 temas distintos misturados;
- ou mais de 30% do conteudo tratar tema diferente do objetivo original;
- ou mais de 5 revisoes consecutivas adicionarem apenas anexos, observacoes, detalhes, notas ou apendices.

Os criterios acima:

- nao autorizam divisao automatica;
- nao autorizam criacao automatica de arquivo;
- nao autorizam mudanca estrutural;
- apenas acionam revisao arquitetural/documental.

### Tratamento obrigatorio

Quando documento exceder criterios:

1. Reorganizar secoes internas se conteudo ainda pertencer ao mesmo dominio.
2. Identificar blocos candidatos a dominio independente.
3. Propor extracao controlada.
4. Nao executar automaticamente.
5. Solicitar aprovacao explicita.
6. Apos aprovacao, registrar:
   - classificacao;
   - finalidade;
   - responsavel;
   - canonicidade;
   - encerramento, se transitorio.
7. Manter indice ou resumo no documento pai.
8. Referenciar documento novo no pai.
9. Registrar se o documento pai permanece canonico ou se nova canonicidade foi formalmente transferida.
10. Garantir ausencia de fonte paralela da verdade.

### Documentos ja grandes

Documentos existentes que ja estiverem grandes:

- nao devem ser divididos automaticamente;
- nao devem ter conteudo movido automaticamente;
- devem ser sinalizados como candidatos quando aplicavel;
- devem listar motivo;
- devem listar bloco candidato;
- devem listar classificacao sugerida;
- devem aguardar aprovacao explicita.

### Exemplos especificos do projeto

- `docs/ROADMAP.md`
  - Deve permanecer executivo.
  - Nao deve absorver detalhes tecnicos extensos.
  - Detalhes podem ser extraidos somente mediante aprovacao explicita.
- `docs/auditoria_legado_v20_1c.md`
  - Pode sinalizar extracao futura.
  - Candidatos naturais: quarentena, observacao operacional e lotes.
  - A extracao nao autoriza decommission.
- `docs/ARCHITECTURE.md`
  - Pode crescer moderadamente.
  - Nao deve absorver backlog operacional.
- `CHANGELOG.md`
  - Crescimento historico e esperado.
  - Nao aprova mudanca funcional sozinho.
- `AGENTS.md`
  - Deve permanecer curto, normativo e operacional.
  - Deve apontar para documentos constitucionais/canonicos quando o detalhe pertencer a governanca, roadmap, arquitetura, auditoria ou backlog.

## Classificacoes oficiais

- `MIGRADO_V20`: item alinhado aos motores, aliases ou governanca V20.
- `COMPATIVEL_V20`: item que pode permanecer sem conflito conhecido.
- `LEGADO_ATIVO`: item funcional que decide ou notifica fora da consciencia V20.
- `ORFAO_INATIVO`: item existente sem uso ativo aparente.
- `CANDIDATO_REMOCAO`: item que pode ser removido futuramente apos validacao.
- `DESCONHECIDO`: item que exige inspecao manual.

## Regra de conflito

Quando houver conflito entre documentos:

1. Constituicao define regras permanentes e prevalece sobre os demais documentos.
2. `source_of_truth.md` atua como indice/roteador documental e nao define precedencia acima da Constituicao.
3. `AGENTS.md` orienta execucao operacional resumida.
4. Arquitetura define desenho tecnico aprovado.
5. Roadmap consolidado define direcao.
6. Auditorias, releases e backlog tecnico servem como evidencias historicas e registro de pendencias.
