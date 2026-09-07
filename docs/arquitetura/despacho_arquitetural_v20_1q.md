# Despacho Arquitetural V20.1Q — Recovery Automático do Modem 4G

Data: 2026-07-13
Status: DECISÃO TÉCNICA APROVADA
Classificação: arquitetura subordinada
Autoridade: subordinada à Constituição, ao Source of Truth, a `architecture.md` e a `docs/ARCHITECTURE.md`

## Finalidade

Definir a arquitetura oficial da V20.1Q sem substituir os documentos arquiteturais canônicos e sem autorizar implementação fora dos gates.

## Princípio central

> A Central decide.
> O Executor atua.
> A Central valida.
> A Central encerra.

## Responsabilidades da Central

A Central é a única responsável por:

- detectar indisponibilidade do 4G;
- decidir quando iniciar recovery;
- autorizar a tentativa;
- validar se o 4G foi recuperado;
- autorizar eventual segunda tentativa;
- declarar falha definitiva;
- encerrar o ciclo;
- determinar publicação em Timeline e Push conforme política.

## Responsabilidades do Executor

O Executor é responsável somente por:

- receber uma solicitação válida;
- verificar permissões operacionais;
- executar o power cycle;
- respeitar o número da tentativa;
- usar o tempo OFF configurado;
- religar a tomada;
- informar fatos técnicos da execução;
- aguardar a decisão da Central.

## Proibições do Executor

O Executor:

- não possui detector próprio;
- não interpreta bytes;
- não interpreta ping;
- não decide se o 4G está indisponível;
- não decide se houve recuperação;
- não altera o estado semântico da Internet;
- não encerra o incidente;
- não cria inteligência paralela;
- não publica eventos por canal paralelo à Central.

## Contrato Central → Executor

A ordem deve identificar uma solicitação válida e a tentativa autorizada. O Executor deve aceitar somente uma execução por vez, rejeitar duplicidade e expor fatos técnicos correlacionáveis à solicitação.

O retorno do Executor não contém veredito semântico sobre o 4G. Ele informa apenas fatos como:

- solicitação aceita ou bloqueada;
- tentativa iniciada;
- tomada desligada;
- tomada religada;
- power cycle concluído tecnicamente;
- aguardando validação;
- timeout técnico;
- erro ou esgotamento técnico.

## Estados técnicos mínimos

- `ocioso`;
- `bloqueado`;
- `solicitado`;
- `executando_tentativa_1`;
- `executando_tentativa_2`;
- `aguardando_validacao`;
- `concluido_tecnicamente`;
- `timeout_validacao`;
- `esgotado`;
- `cooldown`;
- `erro`.

Esses estados descrevem somente o Executor. `concluido_tecnicamente` significa que o power cycle terminou; não significa que o 4G foi recuperado.

## Power cycle aprovado — adendo corretivo de 2026-07-17

- Cada tentativa usa o mesmo Tempo OFF parametrizado.
- A quantidade é definida exclusivamente por `input_number.casa_recovery_4g_max_tentativas`.
- A Central captura um snapshot imutável desse máximo no início do ciclo.
- Não existe limite funcional interno de duas ou dez tentativas.
- O Executor recebe e executa uma única tentativa de índice genérico.
- Validação exclusivamente pela Central.

## Parametrização aprovada

Devem ser parametrizáveis:

- habilitação do recovery automático;
- máximo de tentativas definido exclusivamente pelo helper, sem teto funcional interno;
- Tempo OFF único para qualquer tentativa;
- cooldown;
- confirmação da queda;
- estabilização contínua do retorno;
- Tempo OFF único;
- timeout de espera pela validação da Central;
- publicação em Timeline;
- envio de Push.

Timeline e Push não controlam a execução física. Recovery desligado impede ação física automática, mas não desativa detecção ou diagnóstico.

Os defaults numéricos de cooldown e timeout de validação permanecem pendentes de decisão antes da implementação funcional.

O sucesso exige retorno contínuo durante `casa_recovery_4g_estabilizacao_retorno_minutos`. Perda para `off`, `unknown` ou `unavailable` invalida a janela. `ultima_execucao` significa o timestamp do último esgotamento completo que iniciou cooldown. Sucesso, restart, falta de energia e cancelamento pelo operador não iniciam cooldown.

## Segurança física

O desenho deve impedir que:

- a tomada permaneça desligada indefinidamente;
- dois ciclos sejam executados simultaneamente;
- uma nova execução ignore cooldown;
- restart ou erro deixe o modem sem energia;
- uma tentativa além do snapshot configurado seja disparada.

Restart, cancelamento e erro durante o período OFF devem possuir caminho seguro de religamento. A estratégia concreta deve ser apresentada no Gate pré-implementação.

## Timeline e Push

A Central permanece como publicadora canônica. O Executor não escreve diretamente nos contratos públicos.

- Os helpers de política podem ser preparados na V20.1Q.1.
- A publicação definitiva pertence à V20.1Q.2, salvo reutilização explicitamente aprovada de publicador canônico sem reabrir V20.1O.
- Desligar Timeline ou Push não desativa o recovery.
- Não pode existir feed, timeline ou canal de push paralelo.

## Separação de fases

### V20.1Q.1 — Recovery Executor e parâmetros

- parâmetros operacionais;
- execução física subordinada;
- tentativas e cooldown;
- timeout de validação;
- concorrência, idempotência e restart seguro;
- painel administrativo de parâmetros;
- preparação dos contratos Timeline e Push.

### V20.1Q.2 — Integração canônica

- publicação canônica de fatos do recovery;
- consumo dos helpers Timeline e Push;
- integração sem duplicidade;
- preservação integral da política V20.1O.

## Contratos protegidos

A V20.1Q não autoriza alterar:

- `sensor.status_casa`;
- aliases finais;
- `sensor.casa_timeline`;
- `sensor.casa_event_feed`;
- motores V20.2 shadow;
- detecção homologada de Internet/4G;
- comportamento de failover;
- Score e contexto;
- dashboards produtivos fora da seção administrativa aprovada.

## Rollback arquitetural

O blueprint e a automação legados devem permanecer preservados durante implementação e homologação. Nenhuma remoção de legado pertence à V20.1Q.1. O recovery novo deve possuir chave de habilitação e lote reversível.

## Referências

- Auditoria: `docs/auditorias/auditoria_operacional_recovery_4g_v20_1q.md`.
- Plano: `docs/releases/implementation_plan_v20_1q.md`.
