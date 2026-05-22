# Contexto Operacional - Central Operacional Home Assistant

## Objetivo

Registrar o estado atual da Central Operacional como fonte única de verdade.

## Estado atual

- V20.0 = concluída e congelada
- V20.1A = concluída
- V20.1B lote 1 = concluída
- V20.1B lote 2 = concluída/parcial para energia, internet, failover e backup
- V20.1C = `V20.1C_FECHAMENTO` registrado; diagnóstico e governança concluídos, decommission bloqueado
- V20.1D/E = checkpoint documental de dependências e impacto concluído; limpeza não autorizada
- V20.1K = concluída; tag `V20.1K_FECHAMENTO` criada
- V20.1N = homologada; checkpoint de estabilização registrado
- V20.1O = estável com débitos aceitos; política Timeline / Push / Agregação estabilizada
- V20.2A = concluída; dashboard legado `teste-4` removido pela UI
- V20.2B = auditoria executada; nenhuma ação operacional realizada
- V20.2/V20.3/V21 = planejamento futuro
- V21+ = planejamento futuro

## Regras ativas

- Nunca alterar `sensor.status_casa`
- Não alterar aliases finais sem validação formal
- Dashboards produtivos não consomem `_v20_2`
- V20.2 permanece isolada em shadow mode
- IA é opcional; IA desligada mantém o sistema funcional
- Não substituir automações legadas sem auditoria V20.1C
- Automações órfãs/desabilitadas não devem ser removidas automaticamente; limpeza futura deve seguir criticidade
- Packages novos devem permitir rollback simples
- Radar de Movimento é recurso sob demanda
- Alertas contextuais futuros devem ser assistivos e desacoplados
- V20.1O não deve ser reaberta silenciosamente; mudanças futuras devem abrir lote formal e citar dependências do V20.1O
- V20.1C não autoriza decommission; nenhuma limpeza ou desativação automática está autorizada e ações futuras exigem rollback

## Gate obrigatório - conhecimento prévio

Objetivo: impedir redescoberta de conhecimento já existente e evitar auditorias iniciadas do zero quando o projeto já possui histórico, inventários ou diagnósticos anteriores.

Antes de iniciar qualquer auditoria, diagnóstico, migração, limpeza, refatoração, investigação, plano de implementação ou análise de impacto, deve ser executada consulta aos artefatos existentes do projeto.

Fluxo obrigatório:

1. Procurar documentação existente relacionada ao tema:
   - `AGENTS.md`
   - `docs/ROADMAP.md`
   - `CHANGELOG.md`
   - `architecture.md`
   - `docs/ARCHITECTURE.md`
   - inventários existentes
   - auditorias anteriores
   - documentos de dependências
   - análises de impacto
   - documentação histórica relevante
2. Declarar explicitamente:
   - artefatos consultados;
   - artefatos não encontrados;
   - divergências encontradas;
   - hipótese assumida quando existir ausência de informação.
3. É proibido iniciar descoberta do zero quando existir material prévio relacionado.
4. Inventários resumidos são apenas índice de navegação:
   - não substituem documentação completa;
   - não autorizam remoção;
   - não substituem análise de dependências.
5. Em caso de múltiplos documentos relacionados:
   - usar o documento mais profundo/canônico;
   - utilizar resumos apenas como referência rápida.
6. Caso existam conflitos entre documentos:
   - interromper execução;
   - apontar conflito;
   - solicitar decisão explícita antes de prosseguir.

Resultado esperado: toda nova auditoria ou migração deve começar utilizando conhecimento acumulado do projeto e não por redescoberta manual.

> Detalhes operacionais e histórico de decisões estão em `docs/ROADMAP.md`, `docs/architecture.md`, `docs/pendencias_atuais_central_operacional.md` e `CHANGELOG.md`.
