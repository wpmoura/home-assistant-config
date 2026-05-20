# Harness de Testes Shadow V20.2 - Proposta Tecnica

## Objetivo

Propor uma estratégia de automação controlada para reduzir a execução manual da matriz `docs/matriz_testes_reais_v20_2_fase_1a.md`.

Este documento é apenas desenho técnico. Nenhum package, sensor, helper, dashboard, automação ou alias é criado nesta etapa.

## Escopo

Incluído:

- proposta de harness paralelo
- classificação dos testes da Fase 1A
- estratégia de simulação controlada
- snapshots antes/depois
- validações automáticas
- critérios para decidir se vale criar o package real

Fora de escopo:

- alterar sensores produtivos
- alterar `sensor.status_casa`
- alterar timeline/feed
- alterar dashboards
- alterar aliases finais
- substituir sensores reais
- acionar automações físicas
- desligar internet/energia real
- criar package nesta etapa
- fazer commit

## Estratégia de Testes Automatizados

### Testes 100% Automatizáveis

São testes que podem ser executados sem tocar em entidades físicas e sem depender de sensores reais.

Exemplos:

- validação de estados ENUM
- ausência de dependência com V19
- ausência de alteração em aliases finais
- ausência de publicação pela camada shadow
- existência de atributos obrigatórios
- validação de contradições com fontes simuladas

### Testes Parcialmente Automatizáveis

São testes em que o harness consegue simular as entradas e validar a lógica, mas ainda precisa de confirmação manual em produção.

Exemplos:

- porta aberta + casa vazia
- chuva + janela aberta
- internet degradada + noturno
- energia ausente + alguém em casa
- fonte inválida
- domínio oscilante

### Testes Obrigatoriamente Manuais

São testes que dependem de comportamento físico real, sensores reais ou validação operacional da casa.

Exemplos:

- abertura real de porta
- chuva real no sensor físico
- falta/retorno real de energia
- banho detectado pelo sensor do box
- TV ligada/desligada
- automações legadas preservadas

### Testes Não Recomendados para Simulação

São testes com risco operacional se simulados diretamente.

Exemplos:

- desligar internet real
- provocar falta de energia real
- disparar ações físicas de recuperação
- manipular sensores produtivos via estado forçado
- simular segurança/alarme fora de janela controlada

## Arquitetura do Harness

Package futuro proposto:

`/config/packages/test_harness_v20_2.yaml`

O harness deve nascer completamente separado da produção.

Namespace sugerido:

`teste_v20_2_*`

### Componentes Planejados

Helpers:

- `input_boolean.teste_v20_2_harness_ativo`
- `input_select.teste_v20_2_cenario`
- `input_text.teste_v20_2_snapshot_antes`
- `input_text.teste_v20_2_snapshot_depois`
- `input_text.teste_v20_2_resultado`
- `input_boolean.teste_v20_2_simular_casa_vazia`
- `input_boolean.teste_v20_2_simular_noturno`
- `input_boolean.teste_v20_2_simular_chuva`
- `input_boolean.teste_v20_2_simular_janela_aberta`
- `input_boolean.teste_v20_2_simular_porta_aberta`
- `input_boolean.teste_v20_2_simular_energia_ausente`
- `input_boolean.teste_v20_2_simular_internet_degradada`
- `input_boolean.teste_v20_2_simular_backup_falha`

Sensores simulados:

- `sensor.teste_v20_2_relevancia_contextual`
- `sensor.teste_v20_2_evento_relevante`
- `sensor.teste_v20_2_motivo_relevancia`
- `sensor.teste_v20_2_confianca_contextual`
- `sensor.teste_v20_2_estabilidade_contextual`

Seletores real vs simulado:

- `sensor.teste_v20_2_fonte_casa_vazia`
- `sensor.teste_v20_2_fonte_noturno`
- `sensor.teste_v20_2_fonte_contexto_humano`
- `sensor.teste_v20_2_fonte_contexto_operacional`
- `sensor.teste_v20_2_fonte_contexto_ambiental`

### Camada de Seleção Real vs Simulado

Regra:

- se `input_boolean.teste_v20_2_harness_ativo = off`, sensores de teste apenas espelham produção ou ficam inativos
- se `input_boolean.teste_v20_2_harness_ativo = on`, sensores de teste leem helpers simulados
- sensores produtivos nunca devem ler sensores simulados
- aliases finais nunca devem apontar para sensores simulados

Fluxo:

1. Selecionar cenário em `input_select.teste_v20_2_cenario`.
2. Ativar harness.
3. Gerar snapshot antes.
4. Aplicar helpers simulados.
5. Calcular sensores shadow simulados.
6. Gerar snapshot depois.
7. Comparar esperado vs obtido.
8. Registrar resultado.
9. Desativar harness.

## Snapshots

Snapshot antes:

- estados reais dos sensores V20.2
- atributos principais
- estado da timeline/feed
- estado dos aliases finais

Snapshot depois:

- estados simulados do harness
- resultado esperado
- resultado obtido
- divergências

Campos mínimos:

- `cenario`
- `timestamp`
- `relevancia`
- `evento`
- `motivo`
- `confianca`
- `estabilidade`
- `fontes_invalidas`
- `fontes_contraditorias`
- `contradicao_detectada`
- `timeline_alterada`
- `aliases_alterados`

## Log de Execução

Opções possíveis:

- `input_text.teste_v20_2_resultado`
- Logbook com evento informativo
- notificação persistente não intrusiva
- arquivo markdown preenchido manualmente

Recomendação inicial:

- evitar automações e notificações na primeira versão real do harness
- usar sensores e input_text para observabilidade
- manter relatório final em markdown manual

## Regras de Segurança

- harness desligado por padrão
- namespace exclusivo `teste_v20_2_*`
- nunca publicar em timeline/feed produtivo
- nunca acionar automações físicas
- nunca alterar `sensor.status_casa`
- nunca alterar aliases finais
- nunca substituir sensores reais
- nunca desligar internet real
- nunca simular falta de energia real em produção
- nunca reativar V19
- rollback simples: remover/desabilitar `packages/test_harness_v20_2.yaml`

## Classificação dos Testes da Matriz

### Testes Unitários por Domínio

| ID | Classificação | Justificativa |
|---|---|---|
| U-001 | Automatizável | Estado normal pode ser validado por leitura/snapshot sem ação física |
| U-002 | Parcial | Lógica pode ser simulada, mas abertura real da porta valida sensor físico |
| U-003 | Parcial | Fechamento pode ser simulado, mas validação final deve ser real |
| U-004 | Parcial | Chuva pode ser simulada no harness, mas sensor físico precisa de teste real |
| U-005 | Parcial | Intensidade pode ser simulada, mas chuva real valida cadeia física |
| U-006 | Automatizável | Estado normal de energia pode ser observado por snapshot |
| U-007 | Não recomendado simular | Falta de energia real não deve ser provocada; usar observação ou simulação isolada |
| U-008 | Manual | Retorno de energia depende de evento real ou janela segura |
| U-009 | Automatizável | Internet normal pode ser observada por snapshot |
| U-010 | Não recomendado simular | Não derrubar internet real; apenas simulação isolada no harness |
| U-011 | Manual | Retorno real de internet depende de incidente ou janela segura |
| U-012 | Automatizável | Backup normal pode ser observado por snapshot |
| U-013 | Parcial | Falha pode ser simulada no harness; falha real deve ser observada sem provocar risco |
| U-014 | Parcial | Movimento pode ser testado fisicamente; lógica pode ser simulada |
| U-015 | Manual | Banho depende do sensor físico do box e do timeout real |
| U-016 | Manual | TV ligada depende do media_player real |
| U-017 | Manual | TV desligada depende do media_player real |

### Testes Integrados entre Domínios

| ID | Classificação | Justificativa |
|---|---|---|
| I-001 | Parcial | Simulação valida regra; abertura real + casa vazia valida operação |
| I-002 | Parcial | Simulação valida regra; presença real valida contexto |
| I-003 | Parcial | Simulação valida regra crítica; chuva/janela real validam cadeia física |
| I-004 | Parcial | Simulação valida que porta + chuva não vira crítica indevida |
| I-005 | Não recomendado simular | Energia + internet offline real tem risco operacional |
| I-006 | Parcial | Simulação valida prioridade; falha real de backup deve ser observada |
| I-007 | Parcial | Simulação valida regra; não derrubar internet real sem janela segura |
| I-008 | Não recomendado simular | Falta de energia real não deve ser provocada |

### Testes de Regressão

| ID | Classificação | Justificativa |
|---|---|---|
| R-001 | Automatizável | Busca por dependências V19 e leitura de entidades |
| R-002 | Automatizável | Snapshot antes/depois de `sensor.status_casa` |
| R-003 | Automatizável | Snapshot antes/depois de timeline/feed |
| R-004 | Automatizável | Busca por aliases apontando para V20.2 |
| R-005 | Parcial | Harness não deve interferir; efeitos legados precisam de observação real |

### Testes de Robustez e Contradição

| ID | Classificação | Justificativa |
|---|---|---|
| B-001 | Automatizável | Fonte inválida pode ser simulada em namespace de teste |
| B-002 | Automatizável | Contradição casa vazia/presença pode ser simulada |
| B-003 | Automatizável | Relevância sem evento pode ser simulada |
| B-004 | Automatizável | Chuva ativa + ambiental normal pode ser simulada |
| B-005 | Parcial | Flapping pode ser simulado; flapping real precisa observação |
| B-006 | Parcial | Chuva flapping pode ser simulada; sensor físico precisa observação |

### Testes de Observabilidade e Documentação

| ID | Classificação | Justificativa |
|---|---|---|
| O-001 | Automatizável | Atributos podem ser validados por template/snapshot |
| O-002 | Automatizável | Busca por entidade inexistente pode ser feita sem risco |
| O-003 | Automatizável | Estados ENUM podem ser comparados |
| O-004 | Manual | Documentação precisa revisão humana |
| O-005 | Manual | Rollback deve ser planejado; execução só em janela controlada |

## Proposta de Package Futuro

Arquivo futuro:

`/config/packages/test_harness_v20_2.yaml`

Escopo inicial sugerido:

- helpers simulados
- sensores shadow simulados
- comparação esperado vs obtido
- snapshot textual
- nenhum acionamento físico
- nenhuma publicação em timeline

Não incluir na primeira versão:

- automações de execução em lote
- notificações automáticas
- alteração de helpers produtivos
- integração com dashboards produtivos
- chamadas de serviços físicos

## Critério para Criar o Package Real

Vale criar o package real se:

- houver necessidade de repetir testes muitas vezes
- a matriz manual consumir tempo excessivo
- os cenários críticos precisarem ser revalidados após cada mudança
- a camada V20.2 começar a integrar confiança/relevância/decisão
- houver risco de regressão por mudanças futuras

Não vale criar ainda se:

- a validação manual da Fase 1A for suficiente
- as regras ainda mudarem muito
- o harness introduzir mais complexidade que benefício
- houver risco de confundir simulação com produção

Critério mínimo recomendado:

- concluir pelo menos a baseline manual normal
- validar ao menos um cenário crítico real
- validar que shadow mode não altera timeline/aliases/status
- confirmar que a equipe quer regressão repetível antes da V20.2 avançar

## Próximo Passo Recomendado

Executar a matriz manual da Fase 1A primeiro.

Depois, se houver repetição frequente ou divergências difíceis de reproduzir, criar o package `test_harness_v20_2.yaml` como camada isolada de simulação.

