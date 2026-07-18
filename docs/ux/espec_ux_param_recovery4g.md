# CENTRAL OPERACIONAL HOME ASSISTANT
## ESPECIFICAÇÃO OFICIAL DE UX
### DASHBOARD "PARÂMETROS" — SEÇÃO RECOVERY 4G

| Campo | Valor |
|---|---|
| Documento | ESPEC-UX-PARAM-RECOVERY4G |
| Versão | V20.1Q |
| Natureza | Normativo |
| Escopo | Interface do operador — seção Recovery 4G do dashboard "Parâmetros" |
| Fora de escopo | Arquitetura, implementação, YAML, Lovelace, automações, scripts, helpers |
| Status | Oficial — referência única para implementações futuras |

**Convenção normativa:** os termos DEVE, NÃO DEVE e PODE são empregados neste documento com valor obrigatório. Cláusulas marcadas com DEVE ou NÃO DEVE constituem requisito de aceitação.

---

## 1. OBJETIVO DO PAINEL

A seção Recovery 4G do dashboard "Parâmetros" é o ponto único de ajuste do comportamento da rotina de recuperação automática de conectividade.

O painel existe para permitir que o operador execute três ações, e somente estas três:

1. **Habilitar ou inibir** a recuperação automática.
2. **Calibrar** os tempos e limites do ciclo de recuperação.
3. **Controlar** a emissão de avisos decorrentes da recuperação.

O painel é um instrumento de **configuração**, não de **observação**.

O painel NÃO tem função de diagnóstico. O painel NÃO tem função de monitoramento. O painel NÃO reporta o que o Recovery está fazendo, fez ou concluiu. Estas funções pertencem à Timeline e ao Radar Operacional, e a separação entre configurar e observar é estrutural nesta Central.

O painel DEVE ser operável sob incidente, com uma das mãos, em tela de celular, sem consulta a documentação.

---

## 2. PRINCÍPIOS DE UX

Os princípios abaixo são obrigatórios. Qualquer implementação futura que os contrarie DEVE ser rejeitada em revisão, ainda que funcionalmente correta.

**P1 — Organização cronológica.**
Parâmetros que atuam em fases distintas do ciclo DEVEM ser apresentados na ordem em que o algoritmo os consome. A ordem visual é a documentação do comportamento.

**P2 — Mínima carga cognitiva.**
O operador NÃO DEVE precisar reconstruir mentalmente o ciclo de recuperação para localizar um campo. A posição do campo DEVE revelar sua função.

**P3 — Operação rápida em incidente.**
O controle de maior consequência DEVE estar acessível sem rolagem, na primeira linha do primeiro card.

**P4 — Compatibilidade com celular.**
A tela de referência do projeto é o aplicativo Home Assistant em viewport estreita. Todo rótulo DEVE caber em uma linha. Todo agrupamento DEVE ser legível em aproximadamente uma altura de tela.

**P5 — Ausência de jargão técnico.**
Termos técnicos DEVEM ser traduzidos quando existir alternativa compreensível sem perda de precisão. Ver Seção 6.

**P6 — Parâmetros primeiro.**
O painel apresenta exclusivamente valores que o operador ajusta. Nada que o sistema apenas informa pertence a este painel.

**P7 — Estados internos não aparecem.**
Helpers de estado interno NÃO DEVEM ser expostos neste painel, sob nenhuma condição ou justificativa de conveniência. Ver Seção 7.

**P8 — Separação por intenção.**
O agrupamento DEVE refletir a intenção do operador ao abrir o painel, não a tipagem do dado subjacente.

**P9 — Proporcionalidade visual.**
Controles de consequência distinta NÃO DEVEM ter apresentação visual equivalente. A chave mestra não pode parecer um interruptor de notificação.

**P10 — Unidade explícita.**
Todo valor de duração DEVE exibir sua unidade junto ao valor. Nenhum número aparece desacompanhado.

---

## 3. ORGANIZAÇÃO VISUAL

A estrutura oficial da seção é composta por três cards, nesta ordem fixa:

```
Recovery 4G
    ↓
Operação
    ↓
Ciclo de Recuperação
    ↓
Avisos
```

Esta ordem NÃO DEVE ser alterada.

### 3.1 Motivo da organização

A estrutura deriva de três eixos convergentes.

**Eixo da intenção.** O operador abre este painel por um de três motivos: interromper o sistema, calibrar o sistema, ou silenciar o sistema. Cada motivo tem exatamente um destino. Um card único obrigaria as três intenções a percorrer a mesma lista.

**Eixo da frequência.** Operação define política e muda raramente. Ciclo de Recuperação é ajuste fino e muda ocasionalmente. Avisos é situacional. O que muda menos vem primeiro porque é o que decide se o resto tem efeito.

**Eixo da consequência.** A ordem é decrescente em impacto. Desabilitar a Recuperação Automática anula o painel inteiro. Alterar um tempo altera o comportamento. Desligar um aviso altera apenas o ruído. Isolar os avisos em card próprio é medida de proteção contra toque incorreto, não preferência estética.

### 3.2 Título da seção

O título "Recovery 4G" DEVE ser preservado em inglês. É o nome operacional consolidado da funcionalidade na Central e funciona como identificador, não como rótulo de controle. A regra de tradução da Seção 6 aplica-se aos controles, não à denominação da funcionalidade.

---

## 4. CONTEÚDO DE CADA CARD

O conteúdo abaixo é exaustivo. Nenhum item adicional PODE ser incluído sem revisão formal desta especificação.

### 4.1 Card — Operação

| # | Rótulo oficial | Natureza |
|---|---|---|
| 1 | Recuperação Automática | Interruptor |
| 2 | Máximo de Tentativas | Numérico |
| 3 | Pausa após Esgotar Tentativas | Duração |

### 4.2 Card — Ciclo de Recuperação

| # | Rótulo oficial | Natureza |
|---|---|---|
| 1 | Tempo para Confirmar a Queda | Duração |
| 2 | Tempo com a Tomada Desligada | Duração |
| 3 | Limite de Espera da Tomada | Duração |
| 4 | Limite de Espera do 4G | Duração |
| 5 | Tempo de Estabilização | Duração |

Os cinco itens DEVEM ser numerados de 1 a 5 na interface. A numeração é requisito, não ornamento: ela converte cinco campos independentes em uma sequência legível e comunica o algoritmo ao operador sem exigir consulta a documentação.

### 4.3 Card — Avisos

| # | Rótulo oficial | Natureza |
|---|---|---|
| 1 | Registrar na Timeline | Interruptor |
| 2 | Notificar no Celular | Interruptor |

### 4.4 Alocação da Pausa após Esgotar Tentativas

"Pausa após Esgotar Tentativas" é uma duração e, por tipagem, pertenceria ao card Ciclo de Recuperação. Ela DEVE permanecer no card Operação.

Justificativa normativa: este parâmetro só possui significado em função do Máximo de Tentativas e é sempre ajustado em conjunto com ele. Separá-los para satisfazer coerência de tipo produziria consistência taxonômica ao custo de coerência operacional. Prevalece o princípio P8.

Esta alocação NÃO DEVE ser "corrigida" em implementações futuras.

---

## 5. ORDEM OBRIGATÓRIA

A sequência dos parâmetros espelha a cronologia do algoritmo de recuperação. Cada item ocupa a posição correspondente ao momento em que atua.

### 5.1 Card Operação — camada de política

| Posição | Parâmetro | Momento de atuação |
|---|---|---|
| 1 | Recuperação Automática | Antes de tudo. Determina se o ciclo pode iniciar. |
| 2 | Máximo de Tentativas | Delimita quantas vezes o ciclo pode repetir. |
| 3 | Pausa após Esgotar Tentativas | Atua ao fim da última tentativa fracassada. |

A camada de política é lida antes da camada de tempo porque responde à pergunta anterior: *se* e *quantas vezes*, antes de *quanto tempo*.

### 5.2 Card Ciclo de Recuperação — camada cronológica

| Posição | Parâmetro | Fase do ciclo |
|---|---|---|
| 1 | Tempo para Confirmar a Queda | Detecção. Período de observação antes de tratar a perda como real. |
| 2 | Tempo com a Tomada Desligada | Corte. Intervalo em que a tomada permanece desligada. |
| 3 | Limite de Espera da Tomada | Religamento. Tolerância máxima para a tomada confirmar retorno. |
| 4 | Limite de Espera do 4G | Validação. Tolerância máxima para o enlace 4G ser considerado válido. |
| 5 | Tempo de Estabilização | Consolidação. Período de assentamento antes de encerrar a tentativa. |

A leitura vertical do card reproduz a linha do tempo do evento: **cai → confirma → desliga → religa → valida → estabiliza**.

### 5.3 Card Avisos — camada de saída

Os avisos são consequência do ciclo e ocupam a última posição por serem posteriores a ele em todos os cenários.

### 5.4 Cláusula de imutabilidade

A ordem definida nesta seção NÃO DEVE ser alterada por conveniência de implementação, por ordem alfabética, por ordem de criação de entidades ou por qualquer outro critério que não seja a cronologia do algoritmo. Alteração na ordem dos parâmetros DEVE ser precedida de alteração desta especificação.

---

## 6. REGRAS DE NOMENCLATURA

### 6.1 Motivo da tradução

Após a remoção dos helpers legados Tentativa 1 e Tentativa 2, o painel passou a depender integralmente dos rótulos para transmitir significado. O contexto que anteriormente derivava da numeração das tentativas deixou de existir. Cada rótulo precisa agora ser autossuficiente.

Termos em inglês herdados da implementação carregavam significado técnico que não sobrevive à leitura sob pressão. A tradução não é preferência linguística: é remoção de dependência de conhecimento tácito.

### 6.2 Tabela normativa de conversão

| Termo de origem | Rótulo oficial | Fundamento |
|---|---|---|
| Recovery Automático | Recuperação Automática | Único termo em inglês no controle mais crítico. Tradução direta sem perda. |
| Cooldown após Esgotamento | Pausa após Esgotar Tentativas | "Cooldown" é jargão importado. "Esgotamento" isolado é ambíguo quanto ao objeto. |
| Confirmação da Queda | Tempo para Confirmar a Queda | O rótulo de origem sugere estado, não ajuste de duração. |
| Tempo OFF | Tempo com a Tomada Desligada | "OFF" não identifica o objeto desligado. Rótulo de origem incompleto após remoção dos legados. |
| Timeout de Confirmação da Tomada | Limite de Espera da Tomada | "Timeout" é técnico. "Confirmação" colidia semanticamente com "Confirmação da Queda". |
| Timeout de Validação do 4G | Limite de Espera do 4G | Paralelismo obrigatório com o item anterior. |
| Tempo de Estabilização do Retorno | Tempo de Estabilização | "do Retorno" é redundante dentro do card e penaliza a largura em celular. |
| Timeline | Registrar na Timeline | Substantivo isolado sugere navegação, não interruptor. |
| Push | Notificar no Celular | Termo mais opaco do painel. Não descreve o efeito. |

### 6.3 Quando utilizar "Tempo"

O prefixo **Tempo** DEVE ser utilizado quando a duração é **decidida pelo sistema e cumprida integralmente**. O valor representa quanto tempo a rotina permanece deliberadamente em determinada fase.

Aplica-se a: Tempo para Confirmar a Queda, Tempo com a Tomada Desligada, Tempo de Estabilização.

### 6.4 Quando utilizar "Limite"

O prefixo **Limite** DEVE ser utilizado quando a duração é **teto de tolerância** e a fase pode encerrar antes. O valor representa o máximo que a rotina aguarda por um evento externo.

Aplica-se a: Limite de Espera da Tomada, Limite de Espera do 4G.

A distinção entre 6.3 e 6.4 é semântica e normativa: **"Tempo" é duração cumprida; "Limite" é duração tolerada.** Ela informa ao operador se o valor prolonga o ciclo sempre ou apenas no pior caso.

### 6.5 Quando utilizar verbos

Rótulos de **interruptor de saída** DEVEM iniciar por verbo no infinitivo. O verbo revela a natureza binária do controle e o distingue de rótulos de navegação.

Aplica-se a: Registrar na Timeline, Notificar no Celular.

O interruptor Recuperação Automática é exceção deliberada: trata-se de chave mestra e não de saída, e seu rótulo já é inequívoco.

### 6.6 Quando manter nomes técnicos

Nomes técnicos DEVEM ser preservados em duas situações:

1. **Denominação de funcionalidade.** "Recovery 4G" é identificador consolidado da Central. Traduzir o identificador geraria mais atrito do que resolve.
2. **Nome próprio de componente da Central.** "Timeline" é seção existente e nomeada do sistema. Traduzi-la romperia a correspondência com o destino ao qual o registro é enviado.

Fora destas duas situações, termo técnico com alternativa compreensível DEVE ser traduzido.

### 6.7 Regras estruturais

- Todo rótulo de duração DEVE iniciar por "Tempo" ou "Limite".
- Todo rótulo de interruptor de saída DEVE iniciar por verbo.
- Nenhum rótulo PODE conter sufixo numérico de tentativa.
- Nenhum rótulo PODE exceder a largura de uma linha na viewport de referência.

---

## 7. ITENS PROIBIDOS

Os helpers de estado interno NÃO fazem parte do dashboard "Parâmetros" e NÃO DEVEM ser exibidos nesta seção sob nenhuma hipótese.

Ficam explicitamente proibidos neste painel:

| Item proibido | Natureza |
|---|---|
| Request ID | Estado interno de execução |
| Estado Executor | Estado interno de execução |
| Tentativa Atual | Estado interno de execução |
| Veredito | Resultado de execução |
| Última Tentativa | Registro histórico |
| Última Execução | Registro histórico |

### 7.1 Fundamento da proibição

O painel "Parâmetros" é instrumento de configuração. Os itens acima são estados observados, não valores ajustáveis. Sua presença violaria o princípio P6 e produziria três danos concretos:

1. **Diluição do painel.** Campos não ajustáveis competem por atenção com campos ajustáveis, aumentando o tempo de localização sob incidente.
2. **Ambiguidade de affordance.** O operador não distingue de imediato o que pode alterar do que apenas lê. Risco de tentativa de edição em campo de estado.
3. **Duplicação de responsabilidade.** Observação pertence à Timeline e ao Radar Operacional. Reproduzi-la aqui cria duas fontes para o mesmo fato e abre divergência.

### 7.2 Extensão da regra

A proibição alcança qualquer helper de estado interno criado no futuro, ainda que não listado nominalmente na tabela acima. O critério é funcional: **se o operador não pode alterar o valor, o valor não pertence a este painel.**

### 7.3 Proibições adicionais

- NÃO DEVE ser exibido qualquer parâmetro com sufixo numérico de tentativa. Os helpers legados Tentativa 1 e Tentativa 2 foram removidos e não retornam.
- NÃO DEVE ser exibido valor de duração sem unidade.
- NÃO DEVE ser exibido controle de recuperação fora dos três cards especificados.

---

## 8. MOCKUP TEXTUAL

Organização oficial da seção. Valores exibidos são ilustrativos e não constituem especificação de default.

```
Recovery 4G

┌─ OPERAÇÃO ────────────────────────────────┐
│  Recuperação Automática          [ ON  ]  │
│  Máximo de Tentativas               [ 3 ] │
│  Pausa após Esgotar Tentativas   [ 30 min]│
└───────────────────────────────────────────┘

┌─ CICLO DE RECUPERAÇÃO ────────────────────┐
│  1 · Tempo para Confirmar a Queda  [ 60 s]│
│  2 · Tempo com a Tomada Desligada  [ 30 s]│
│  3 · Limite de Espera da Tomada    [ 20 s]│
│  4 · Limite de Espera do 4G        [120 s]│
│  5 · Tempo de Estabilização        [ 60 s]│
└───────────────────────────────────────────┘

┌─ AVISOS ──────────────────────────────────┐
│  Registrar na Timeline           [ ON  ]  │
│  Notificar no Celular            [ ON  ]  │
└───────────────────────────────────────────┘
```

### 8.1 Elementos normativos do mockup

| Elemento | Status |
|---|---|
| Três cards, nesta ordem | Obrigatório |
| Títulos dos cards conforme grafados | Obrigatório |
| Numeração 1–5 no card Ciclo de Recuperação | Obrigatório |
| Ausência de numeração nos cards Operação e Avisos | Obrigatório |
| Unidade visível em todo valor de duração | Obrigatório |
| Rótulo à esquerda, valor à direita | Obrigatório |
| Valores exibidos no mockup | Ilustrativos |
| Caracteres de moldura | Ilustrativos |

---

## 9. CRITÉRIOS DE ACEITAÇÃO

A UX é considerada implementada quando todos os critérios abaixo são satisfeitos. Critérios são objetivos e verificáveis por inspeção visual na viewport de referência.

| # | Critério | Verificação |
|---|---|---|
| A1 | A seção contém exatamente três cards, nomeados Operação, Ciclo de Recuperação e Avisos. | Inspeção |
| A2 | Os cards aparecem nesta ordem, de cima para baixo. | Inspeção |
| A3 | O card Operação contém exatamente os três itens da Seção 4.1, nesta ordem. | Inspeção |
| A4 | O card Ciclo de Recuperação contém exatamente os cinco itens da Seção 4.2, nesta ordem. | Inspeção |
| A5 | O card Avisos contém exatamente os dois itens da Seção 4.3, nesta ordem. | Inspeção |
| A6 | Os cinco itens do card Ciclo exibem numeração visível de 1 a 5. | Inspeção |
| A7 | Todos os rótulos correspondem literalmente aos rótulos oficiais da Seção 4. | Comparação textual |
| A8 | Nenhum rótulo quebra em duas linhas na viewport de referência. | Inspeção em celular |
| A9 | Todo valor de duração exibe unidade. | Inspeção |
| A10 | Nenhum item da Seção 7 aparece na seção Recovery 4G. | Inspeção |
| A11 | Nenhum rótulo contém sufixo numérico de tentativa. | Comparação textual |
| A12 | Nenhum termo em inglês aparece em rótulo de controle. | Comparação textual |
| A13 | O interruptor Recuperação Automática é alcançável sem rolagem ao abrir a seção. | Inspeção em celular |
| A14 | O card Avisos é visualmente distinto e não adjacente ao interruptor Recuperação Automática. | Inspeção |
| A15 | Campos de duração permitem ajuste por incremento discreto, não exclusivamente por arraste contínuo. | Interação em celular |

**Regra de aceitação:** a UX está implementada quando A1 a A15 são satisfeitos integralmente. Reprovação em qualquer critério reprova a entrega, independentemente de correção funcional.

---

## 10. RACIONAL ARQUITETURAL

Esta seção justifica a especificação. Não contém requisito.

### 10.1 A interface documenta o algoritmo

A organização cronológica faz com que a leitura do painel ensine o funcionamento do Recovery. Um operador que nunca leu a documentação, ao percorrer o card Ciclo de Recuperação de cima para baixo, aprende que o sistema confirma a queda, desliga a tomada, aguarda o retorno, valida o 4G e estabiliza. A interface passa a ser a explicação, e a documentação deixa de ser pré-requisito de operação.

Isto reduz a dependência de memória e o custo de reaprendizado após meses sem incidente — que é a condição normal de uso deste painel.

### 10.2 A separação por intenção reduz o tempo até a ação

Sob incidente, o custo dominante não é ajustar: é localizar. Três cards com intenções distintas convertem uma busca linear em uma escolha entre três. O operador que precisa interromper o sistema não percorre tempos de calibração para chegar à chave mestra.

### 10.3 O isolamento dos avisos é medida de segurança

Três interruptores adjacentes com consequências incomparáveis constituem risco operacional. Desligar a Recuperação Automática acreditando desligar a notificação é falha plausível e de custo alto — o sistema deixa de recuperar sem que ninguém perceba, precisamente porque a notificação continuou ligada e nada mudou na percepção do operador. A separação física em cards elimina a classe inteira de erro.

### 10.4 A nomenclatura absorve a perda dos legados

A remoção de Tentativa 1 e Tentativa 2 retirou do painel o contexto que a numeração fornecia. Rótulos autossuficientes restauram esse contexto sem reintroduzir os helpers. "Tempo com a Tomada Desligada" carrega sozinho o que "Tempo OFF" só significava por vizinhança.

### 10.5 A distinção Tempo/Limite informa o custo do ajuste

Ao saber que "Tempo" é sempre cumprido e "Limite" é apenas tolerado, o operador compreende de imediato qual parâmetro alonga o ciclo em toda execução e qual só age no pior caso. A escolha de calibração torna-se informada pelo próprio rótulo.

### 10.6 Coerência com a Central

O modelo de duas personas da Central separa controle de observação. O dashboard "Parâmetros" é superfície de controle; Timeline e Radar Operacional são superfícies de observação. A proibição da Seção 7 não é restrição local: é a aplicação, nesta seção, de uma regra estrutural da Central. Admitir estado interno neste painel abriria precedente para a erosão da separação em todas as demais seções.

### 10.7 Governança

A ausência deste documento produziu a falha que o originou: a decisão de UX existia, estava aprovada, e não foi implementada — porque a implementação segue a documentação, e não a conversa. Este documento encerra essa lacuna. A partir da V20.1Q, decisão de UX sem documento normativo é decisão inexistente para efeitos de implementação.

---

## CONTROLE DO DOCUMENTO

| Item | Valor |
|---|---|
| Referência oficial | ESPEC-UX-PARAM-RECOVERY4G · V20.1Q |
| Substitui | Nenhum documento anterior — primeira emissão |
| Revoga | Decisões de UX registradas exclusivamente em conversa técnica |
| Precedência | Este documento prevalece sobre qualquer registro conversacional de UX da seção Recovery 4G |
| Alteração | Qualquer desvio dos requisitos DEVE ser precedido de revisão formal desta especificação |
| Aplicável a | Implementações futuras da seção Recovery 4G do dashboard "Parâmetros" |

---

**FIM DO DOCUMENTO**
