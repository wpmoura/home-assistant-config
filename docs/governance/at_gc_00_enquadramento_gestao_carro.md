# AT-GC-00 — Enquadramento e Arquitetura da Gestão do Carro

Data: 2026-08-21
Status: APROVADA COM RESSALVAS — RESSALVAS INCORPORADAS — ENCERRADA DOCUMENTALMENTE
Classificação: governança de domínio (Gestão do Carro), frente independente
Autoridade: subordinada à Constituição, ao Source of Truth e a `architecture.md`/`docs/ARCHITECTURE.md`

## 1. Objetivo

Definir oficialmente o domínio **Gestão do Carro** dentro do projeto **Central Operacional Home Assistant**, consolidando em uma única arquitetura coerente:

* presença e uso do veículo;
* odômetro;
* abastecimentos;
* consumo;
* manutenção;
* ingestão futura por fotografias;
* integração com Timeline;
* dashboard.

Esta AT é exclusivamente de **enquadramento e arquitetura**.

Não deve implementar alterações de runtime.

---

## 2. Motivação

A auditoria realizada identificou que o domínio Carro já possui componentes funcionais, porém desenvolvidos em momentos e padrões arquiteturais distintos.

Hoje coexistem:

### Uso/presença

Implementação baseada em:

* `carro_presenca`;
* `session_id`;
* `request_id`;
* eventos estruturados;
* Timeline;
* controle de início e término de utilização do veículo.

Este é atualmente o bloco arquiteturalmente mais maduro do domínio.

### Abastecimento/consumo

Existe implementação anterior baseada principalmente em:

* helpers;
* scripts;
* sensores calculados;
* dashboard;
* registro manual de odômetro, litros e valor.

Não existe histórico estruturado próprio de abastecimentos.

### Manutenção

Existe lógica para:

* quilometragem da última troca de óleo;
* data da última troca;
* quilometragem desde a troca;
* quilometragem restante;
* status de vencimento.

A auditoria identificou referências quebradas em parte dessa cadeia.

### Ingestão automática

Ainda não existe.

A nova necessidade prevê identificação automática de:

* fotografias do painel do veículo contendo odômetro;
* screenshots de aplicativo de posto contendo abastecimento;
* data/hora;
* litros;
* valor;
* combustível;
* eventualmente posto e identificador da transação.

As imagens estarão disponíveis em pasta do NAS, alimentada pelo processo existente de backup das fotos.

---

# 3. Princípio arquitetural

**Gestão do Carro é um único domínio.**

Presença, abastecimento, odômetro, consumo, manutenção e processamento de fotografias não deverão evoluir como soluções independentes.

O processamento de fotografias deverá ser considerado uma **fonte de dados do domínio**, e não uma vertical independente.

---

# 4. Capacidades funcionais do domínio

## GC-C1 — Uso do veículo

Responsável por:

* detectar início de uso;
* detectar término de uso;
* manter sessão de utilização;
* contabilizar viagens;
* publicar eventos relevantes na Timeline.

Base existente:

`carro_presenca`

---

## GC-C2 — Odômetro

Responsável por manter a quilometragem válida conhecida do veículo.

O odômetro deverá ser considerado um **dado central do domínio**, porque participa de:

* abastecimento;
* consumo;
* manutenção;
* validação de registros;
* correlação de fotografias;
* cálculo de quilômetros percorridos.

Fontes possíveis no futuro:

* entrada manual;
* foto do painel;
* outra integração do veículo.

A definição de precedência entre fontes será feita em AT posterior.

**Pré-requisito bloqueante (ressalva incorporada):** a definição de autoridade das fontes, precedência e resolução de conflitos é **pré-requisito bloqueante para promoção automática de uma observação de odômetro para o valor canônico**. Até essa definição, a ingestão futura poderá capturar e armazenar uma **observação candidata** de odômetro normalmente — o que fica proibido é o caminho `observação automática → odômetro canônico` sem validação/regra de autoridade explícita. Permanece válida a distinção já definida na seção 5: `evidência → observação → validação → dado/evento confirmado`.

---

## GC-C3 — Abastecimento

Responsável pelo registro de eventos de abastecimento contendo, no mínimo:

* data/hora;
* odômetro associado;
* quantidade de litros;
* valor pago.

Campos adicionais possíveis:

* tipo de combustível;
* valor bruto;
* desconto;
* preço por litro;
* posto;
* ID de transação;
* origem do dado;
* evidência associada.

O domínio deverá suportar tanto entrada automática quanto correção/manual fallback.

---

## GC-C4 — Consumo

Responsável pelos cálculos derivados de abastecimentos válidos:

* quilômetros percorridos;
* km/L;
* R$/km;
* preço/L;
* evolução de consumo.

Esses dados devem ser derivados dos registros canônicos, evitando duplicação desnecessária de informação.

---

## GC-C5 — Manutenção

Responsável inicialmente por:

* troca de óleo;
* quilometragem na troca;
* data da troca;
* km desde a manutenção;
* km restantes;
* status.

A arquitetura deverá permitir futura expansão para:

* pneus;
* bateria;
* filtros;
* revisão;
* outras manutenções.

Não implementar expansão nesta AT.

---

## GC-C6 — Ingestão por imagens

Responsável futuramente por analisar imagens disponíveis no NAS.

Dois tipos iniciais:

### Tipo A — Painel / odômetro

Objetivo:

* identificar imagem válida de painel;
* extrair quilometragem;
* obter data/hora da fotografia;
* produzir uma observação candidata de odômetro.

Para fotografias reais, priorizar metadados adequados, especialmente data original da captura quando disponível.

### Tipo B — Screenshot de abastecimento

Objetivo:

* identificar comprovante/app de abastecimento;
* extrair:

  * data/hora;
  * litros;
  * valor;
  * combustível;
  * outros campos relevantes.

Para screenshots, a data/hora exibida no próprio comprovante poderá ser mais relevante que o timestamp físico do arquivo.

---

# 5. Correlação entre evidências

A ingestão de imagens deverá separar:

**observação**

de

**evento confirmado**.

Exemplo:

`foto painel → observação de odômetro`

`screenshot posto → observação de abastecimento`

Somente depois da correlação e validação o sistema deverá produzir:

`evento canônico de abastecimento`

A correlação poderá considerar:

* proximidade temporal;
* sequência cronológica;
* evolução do odômetro;
* consistência com registros anteriores;
* nível de confiança.

Caso não seja possível determinar associação confiável:

`PENDENTE_REVISAO`

O sistema não deverá inventar associações.

---

# 6. Fontes de dados

O domínio deverá aceitar futuramente múltiplas fontes.

Tipos previstos:

* `manual`
* `photo_odometer`
* `fuel_app_screenshot`
* `automation`
* eventual integração direta com veículo

Todo dado originado automaticamente deverá permitir rastreabilidade até sua fonte.

---

# 7. Idempotência

A arquitetura deverá impedir que a mesma evidência ou abastecimento seja processado múltiplas vezes.

A estratégia definitiva será definida posteriormente, mas deverá considerar elementos como:

* hash do arquivo;
* caminho/identificador da imagem;
* ID de transação;
* timestamp;
* odômetro;
* litros;
* valor.

Um arquivo já processado não deve gerar um novo registro somente porque o scanner rodou novamente.

---

# 8. Histórico

A Gestão do Carro deverá possuir histórico estruturado próprio.

O Home Assistant Recorder poderá continuar registrando estados, mas não deverá ser considerado a única fonte de verdade para eventos históricos de abastecimento/manutenção.

Precisaremos definir posteriormente a persistência canônica.

Candidatos incluem:

* armazenamento interno adequado do Home Assistant;
* SQLite externo/local;
* outro mecanismo estruturado.

A decisão não faz parte desta AT.

---

# 9. Home Assistant

O Home Assistant permanece como:

* camada operacional;
* visualização;
* controle manual;
* automações;
* alertas;
* Timeline.

A ingestão pesada de fotografias não precisa necessariamente ocorrer dentro do Home Assistant.

Arquitetura preliminar:

`iCloud Photos`
→ `Parachute`
→ `NAS`
→ `processador externo`
→ `contrato Gestão do Carro`
→ `Home Assistant`

---

# 10. Dashboard

O dashboard 🚗 Carro existente deverá ser tratado como ativo reaproveitável, sujeito a saneamento e evolução.

Elementos já existentes:

### Abastecimento

* Odômetro
* Litros
* Valor
* Preço/L
* Abastecer

### Consumo

* Km rodados
* Km/L
* R$/km

### Manutenção

* Km restantes
* Km desde óleo
* Status óleo
* Troca óleo

**Saneamento futuro (ressalva incorporada):** o saneamento previsto para o AT-GC-01 deve contemplar explicitamente o item **GC-L06** (seção 16) — os cards "Teste abastecimento" e "Teste troca óleo" apontam hoje para os mesmos scripts de produção, sem isolamento real.

A AT não redesenha o dashboard.

O objetivo é preservar o que fizer sentido após definição do contrato de dados.

---

# 11. Relação com `carro_presenca`

`carro_presenca` não deverá ser substituído nesta fase.

Sua implementação atual serve como referência arquitetural por já possuir:

* sessão;
* request ID;
* esquema estruturado;
* integração com Timeline;
* idempotência/observabilidade mais madura.

A Gestão do Carro deverá absorver conceitualmente essa capacidade sem introduzir regressão.

Não alterar `carro_presenca` nesta AT.

---

# 12. Timeline

Nem toda alteração interna deverá gerar evento de Timeline.

Eventos candidatos futuros incluem:

* início de uso;
* fim de uso;
* abastecimento registrado;
* manutenção realizada;
* situação relevante de manutenção.

Leituras intermediárias de OCR, arquivos rejeitados e cálculos derivados não deverão necessariamente aparecer na Timeline.

A política definitiva será definida posteriormente.

---

# 13. Entrada manual

A automação não elimina o modo manual.

Entrada manual deverá permanecer como fallback para casos como:

* foto ausente;
* OCR inconclusivo;
* correlação ambígua;
* correção de informação;
* indisponibilidade do processador.

A arquitetura deverá permitir distinguir claramente:

`automático`

de

`manual`.

---

# 14. Validações de domínio

Futuramente deverão existir regras mínimas como:

* odômetro não pode regredir sem tratamento explícito;
* salto excessivo deve ser sinalizado;
* litros devem ser positivos;
* valor deve ser válido;
* preço por litro deve estar dentro de faixa plausível;
* evento duplicado não deve ser aceito;
* correlação com baixa confiança deve exigir revisão.

Os limites numéricos ainda não serão definidos nesta AT.

---

# 15. Fronteira com CSMR

A AT-GC-00:

* NÃO altera CSMR;
* NÃO altera reconciliador;
* NÃO altera processador CSMR;
* NÃO modifica lógica de saída/retorno;
* NÃO exige saída física;
* NÃO depende do fechamento do Gate comportamental atualmente pendente.

O desenvolvimento desta frente só poderá tocar componentes compartilhados mediante Gate específico posterior.

**Componente compartilhado da Timeline (ressalva incorporada):** ficam registrados explicitamente como fronteira compartilhada:

* `script.casa_publicar_evento_timeline_v20`;
* `packages/motor_timeline_v20.yaml`.

Ambos já são utilizados por `carro_presenca` e por outros produtores (`csmr_v20_2c`, `lavadora`). Qualquer alteração futura nesses componentes pela Gestão do Carro (por exemplo, ao publicar eventos de abastecimento/manutenção na Timeline — seção 12) deverá ocorrer em Gate específico, considerando os demais produtores que já os utilizam. Nenhuma alteração nesses componentes é feita ou autorizada por esta AT.

---

# 16. Legado identificado pela auditoria

Registrar como backlog técnico, sem correção nesta AT:

### GC-L01

Sensores de manutenção referenciando IDs divergentes das entidades existentes.

### GC-L02

Automações restauradas/órfãs:

* `automation.carro_registrar_abastecimento_pelo_botao`
* `automation.carro_registrar_troca_de_oleo_pelo_botao`

### GC-L03

Abastecimento sem histórico canônico próprio.

### GC-L04

Modelo antigo baseado em helpers como estado principal do fluxo.

### GC-L05

Dashboard em `.storage`, portanto fora do versionamento normal do repositório.

### GC-L06 (ressalva incorporada)

Cards "Teste abastecimento" e "Teste troca óleo" não possuem isolamento real e apontam para scripts de produção (`script.carro_registrar_abastecimento` e `script.carro_registrar_troca_oleo`). Não existe script de teste isolado no domínio — acionar esses cards executa a ação real de produção. Representa risco operacional (disparo acidental de escrita real por engano de ambiente de teste) e deverá ser tratado na AT-GC-01.

Nenhum desses itens deve ser corrigido nesta AT.

---

# 17. Modelo conceitual inicial

A arquitetura deverá distinguir pelo menos estas entidades conceituais:

### Veículo

Representa o carro gerenciado.

### Sessão de uso

Representa uma utilização do veículo.

### Observação de odômetro

Leitura candidata proveniente de foto ou entrada manual.

### Abastecimento

Evento confirmado de combustível.

### Manutenção

Evento confirmado de serviço/manutenção.

### Evidência

Arquivo, fotografia ou screenshot que sustenta uma observação/evento.

---

# 18. Fluxo conceitual alvo

## Uso

`presença`
→ `início/fim de sessão`
→ `evento de uso`
→ `Timeline`

## Odômetro

`foto/manual`
→ `observação`
→ `validação`
→ `odômetro válido`

## Abastecimento

`screenshot do posto`
+
`observação de odômetro próxima`
→ `correlação`
→ `validação`
→ `abastecimento confirmado`
→ `cálculos`
→ `HA`
→ `Timeline quando aplicável`

## Manutenção

`registro manual/automático futuro`
→ `evento de manutenção`
→ `odômetro`
→ `cálculos de vencimento`
→ `HA`

---

# 19. Critérios de aceite da AT-GC-00

A AT poderá ser considerada aprovada quando estiver formalmente definido que:

1. Gestão do Carro é um domínio único;
2. `carro_presenca` integra esse domínio;
3. odômetro é informação central compartilhada;
4. abastecimento passa a ser tratado como evento histórico;
5. ingestão por fotos é fonte de dados, não domínio independente;
6. evidência e evento confirmado são conceitos diferentes;
7. deve existir idempotência;
8. deve existir rastreabilidade de origem;
9. entrada manual permanece como fallback;
10. Home Assistant não precisa executar processamento pesado das imagens;
11. nenhuma implementação será iniciada antes do saneamento e definição do contrato canônico;
12. o trabalho não interfere no Gate atual do CSMR.

---

# 20. Próxima AT proposta

Após aprovação desta AT:

**AT-GC-01 — Saneamento Controlado do Legado Carro**

Objetivo:

* corrigir inconsistências comprovadas pela auditoria;
* eliminar ou regularizar órfãos;
* garantir baseline técnica estável;
* preservar comportamento válido existente;
* preparar o domínio para o contrato canônico.

A AT-GC-01 deverá possuir Gate próprio e não poderá introduzir ainda a automação de fotografias.

---

## 21. Registro de encerramento

**Decisão:** APROVAR COM RESSALVAS.

**Ressalvas incorporadas (4/4):**

1. GC-L06 adicionado ao backlog de legado (seção 16).
2. Seção 10 (Dashboard) referencia GC-L06 como escopo do saneamento futuro.
3. Seção 4 (GC-C2 — Odômetro) explicita que precedência/autoridade de fontes é pré-requisito bloqueante apenas para promoção a odômetro canônico, não para captura de observação candidata.
4. Seção 15 (Fronteira com CSMR) registra `script.casa_publicar_evento_timeline_v20` e `packages/motor_timeline_v20.yaml` como componente compartilhado, sujeito a Gate específico para qualquer alteração futura.

**Status final: AT-GC-00 — APROVADA E ENCERRADA DOCUMENTALMENTE.**

A AT-GC-01 não é aberta por este documento.
