# V20.2D — Baseline operacional consolidada da V20.2C

Status: **V20.2C FUNCIONALMENTE CONCLUÍDA**  
Pendência não bloqueante: **I4B.2 — Evidência Operacional** do primeiro ciclo físico.

## Escopo e método

Esta consolidação é exclusivamente estática e documental. Foram consultados `AGENTS.md`, `docs/ROADMAP.md`, `docs/governance/source_of_truth.md`, `docs/governance/gates_v20.md`, `docs/v20_2c/README.md`, `docs/v20_2c/plano_tecnico_csmr.md`, `CHANGELOG.md` e os packages V20.2C. Não houve runtime, Harness, reload, restart ou alteração funcional.

## Auditoria arquitetural

- Não foram removidos helpers, scripts ou automações.
- Os controles `input_boolean.teste_v20_2_harness_ativo` e `input_boolean.teste_v20_2_simular_wilson_ausente` permanecem expostos por decisão de rollback/Harness e são documentados como controles de teste, não como comportamento produtivo.
- O script transacional `script.casa_csmr_transicionar_v20_2c` é consumido pelo dispatcher e pelo Harness; o script canônico de publicação `script.casa_publicar_evento_timeline_v20` é o caminho I1.
- As automações C1.1, C1.2, C1.3 e Protect possuem consumidores e contratos registrados. Não foram identificados órfãos que autorizassem remoção automática.
- As entidades de sessão, fronteira temporal e `return_pending` são persistentes e consumidas pelo dispatcher/consumidores. Não foram encontradas entidades temporárias a remover silenciosamente.
- Referências históricas a “pendente” em seções narrativas anteriores foram tratadas como histórico; o status vigente passa a ser o deste documento e das seções finais dos Gates.

## Catálogo de contratos

| Contrato | Responsabilidade | Entradas | Saídas | Autoridade | Subordinados |
|---|---|---|---|---|---|
| I1 | Publicação canônica de fatos da sessão | `request_id`, `session_id`, `event_code`, mensagem, origem | ACK/ledger V20.1O | V20.1O | CSMR/dispatcher como solicitantes |
| I2 | Máquina transacional serial | ação, request, origem, modo de teste | `idle/starting/active/ending/failed`, sessão e checkpoints | CSMR | dispatcher e consumidores |
| I2A | Reserva e correlação de sessão | reserva, UUID e ACK | sessão reservada/consumida, D1 seguro | CSMR | dispatcher |
| I3 | Coordenação da saída/retorno | presença oficial, graça, I1 e I2A | ciclo correlacionado e publicação homologada | dispatcher sob contratos I1/I2A | CSMR |
| I4 | Autorização temporal dos consumidores | estado CSMR, `return_pending`, `session_id`, `occurred_at` | autorização/invalidação de C1.x | CSMR + fronteira persistente | C1.1, C1.2, C1.3 |
| I5 | Intenção de gravação Protect | solicitação CSMR, override manual, estado observado | `always` ou `detections` nos dois selects | intenção efetiva OR; Protect apenas executor | câmeras G4 Instant |

## Mapa arquitetural

```text
Wilson / person.wmoura
          │
          ▼
     Dispatcher
          │  I1 / I2 / I2A
          ▼
         CSMR
     ┌────┼───────────────┬──────────┐
     ▼    ▼               ▼          ▼
  Timeline C1.1          C1.2       C1.3
  (I1/V20.1O) (luz)       (porta)    (garantia)
                         
                    UniFi Protect
                    (I5: always/detections)
```

CSMR coordena ciclo e autorização; não substitui a autoridade V20.1O de publicação. C1.x e Protect não abrem nem encerram sessão.

## Próximas evoluções elegíveis

Ficam apenas como candidatos documentais, sem implementação: coleta de evidência física I4B.2, promoção operacional posterior do Protect quando necessária, novos consumidores por Gate próprio e futuras integrações externas reproduzíveis. Qualquer evolução deverá abrir Gate específico, preservar os contratos I1–I5 e incluir rollback.
