Regras fortes para o 

homeassistant:
packages: !include_dir_named packages

Todos os arquivos YAML devem ser gerados:

* em formato de package
* com raiz em dicionário
* usando padrão moderno compatível com HA 2026
* mantendo identação consistente
* prontos para salvar diretamente em /config/packages/
Central Operacional = status_casa.yaml como motor principal
Outros packages = fontes especializadas
Dashboard = só consome os sensores finais

NÃO gerar:

* listas soltas na raiz (- sensor:)
* snippets parciais
* template: duplicado desnecessariamente
* sintaxe antiga/deprecated

Sempre gerar o arquivo COMPLETO final.

## Modelo obrigatório de registro por fase V20

```markdown
# V20 — Fase X — Nome da fase

## Objetivo
(explica rapidamente o propósito técnico)

## Escopo implementado
(lista objetiva do que entrou)

## Arquivos criados
- caminho
- caminho

## Arquivos alterados
- caminho
- caminho

## Sensores criados
- sensor.x
- sensor.y

## Helpers utilizados
- helper.x

## Dependências utilizadas
- sensor.a
- binary_sensor.b

## Regras implementadas
- regra 1
- regra 2

## Fallbacks implementados
- fallback 1

## Compatibilidade
- Não depende da V19
- Não altera aliases finais
- Não altera dashboards
- etc

## Limitações atuais
- item 1
- item 2

## Pendências próximas fases
- timeout
- contexto
- multi-evento
- IA contextual
- etc

## Validação
- YAML OK
- Templates OK
- Sem referências órfãs
- Sem dependências quebradas

## Próximo passo recomendado
(descrever exatamente a próxima ação)
```
