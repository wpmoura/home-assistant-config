# Despacho — PEND-016 — Alarme / Alexa — Fechamento (2026-09-09)

## Objetivo

Encerrar a PEND-016: corrigir e homologar o caminho produtivo do alarme
(`alarm_control_panel.home_alarm`) que anuncia por voz via Alexa Media
Player ao armar em modo casa, e eliminar a duplicidade de anúncio na
automação de disparo.

## Defeito original

- `automation.alarme_modo_casa` chamava `script.1743352611708` ("Anunciar
  via Alexa") — entidade órfã, removida do config atual (só restava um
  estado `restored` fantasma).
- `automation.alarme_alarme_disparado_2_2` continha uma ação TTS via
  `notify.alexa_media` já desabilitada (`enabled: false`) e redundante com
  o `media_player.play_media` já existente em `automation.alarme_alarme_disparado_2`
  (mesmo evento, mesmo destino, mesmo arquivo de som).
- Bloqueio externo concomitante: integração `alexa_media` (entrada
  `amazon.com.br`) presa em `setup_retry`, `notify.alexa_media` ausente
  do registro de serviços, ambos os Echo (`echo_show_de_wilson_2`,
  `moura_s_echo_dot_2`) `unavailable`.

## Correção aplicada (Gate de correção)

Em `automations.yaml`, exclusivamente:

- `automation.alarme_modo_casa`: chamada ao script ausente substituída por
  `notify.alexa_media` direto (mesmo padrão TTS já usado no fluxo de
  disparo), eliminando o ID numérico opaco.
- `automation.alarme_alarme_disparado_2_2`: ação TTS desabilitada/redundante
  removida; a automação mantém apenas a notificação push.

Nenhuma outra automação de alarme foi tocada. `homeassistant.check_config`
e `automation.reload` (somente automações) executados com sucesso, sem
efeito produtivo.

## Resolução do bloqueio externo (evidência humana)

Reportado por Wilson e revalidado tecnicamente nesta frente: o Alexa Media
Player (HACS) foi atualizado de `5.7.0` para `5.15.7`; o Home Assistant foi
reiniciado para carregar a nova versão; o erro `Connection Error during
login` (`KeyError` recorrente em `alexalogin.login`) cessou. Revalidação
técnica pós-restart confirmou: entrada `amazon.com.br` = `loaded`;
`notify.alexa_media` presente no registro de serviços; `echo_show_de_wilson_2`
disponível, com interação real recente registrada pela própria integração.
A entrada `amazon.com` permanece `not_loaded`/desabilitada — decisão
anterior preservada, não alterada.

## Homologação física real (autorização humana explícita)

Ciclo único, real, pelo caminho produtivo normal (sem simulação):

1. `automation.ativar_alarme_automaticamente_e_notificar` disparada →
   `alarm_control_panel.home_alarm`: `disarmed` → `armed_home`.
2. `automation.alarme_modo_casa` disparada pela mudança de estado; push ao
   iPhone confirmado; chamada `notify.alexa_media` executada **sem erro**
   em log (nenhuma exceção, nenhum `service_not_found`).
3. **Confirmação auditiva humana de Wilson:** ouviu no Echo Show a
   mensagem "Alarme ligado modo Casa".
4. `automation.desativar_alarme_ao_acordar` disparada →
   `alarm_control_panel.home_alarm`: `armed_home` → `disarmed`.
5. `automation.alarme_desarmado` disparada (push + parada de mídia).
   Estado final estável, sem erro, sem efeito inesperado.

Nenhuma porta/janela foi aberta deliberadamente, nenhuma sirene, nenhum
disparo, nenhum ciclo adicional.

## Estado ao final desta frente

- PEND-016 = **encerrada, corrigida e homologada**.
- Caminho produtivo Alarme → Alexa funcional e comprovado por execução
  real com confirmação humana.
- Nenhuma credencial, senha ou token da conta Amazon foi manuseado,
  armazenado ou exposto por este agente em nenhum momento desta frente.

## Residuais conhecidos — explicitamente fora deste fechamento

- `media_player.moura_s_echo_dot_2` permanece `unavailable`; não participa
  do fluxo do alarme, não bloqueia este fechamento.
- Par legado `automation.modo_dormir_autoarmar_alarme` /
  `modo_dormir_desarmar_alarme_ao_acordar`, visando a entidade inexistente
  `alarm_control_panel.alarme`, permanece morto e não tratado.
- Três automações fora do escopo do alarme (`lab_desligar_agenda_monitor_dell`,
  `rotina_desligar_home_office_voice`,
  `area_de_servico_avisar_jacira_fim_da_lavagem_de_roupas`) referenciam
  serviços Alexa quebrados pela mesma causa raiz histórica; não tocadas.
- `automations.yaml` permanece com a correção desta frente **não
  commitada** — parte do working tree misto de `/Volumes/config`, junto
  com outras frentes (CSMR) ainda em andamento. Publicação em histórico
  limpo é item separado, não autorizado neste Gate.

## Referência cruzada

- `docs/governance/gates_v20.md` — log resumido dos Gates desta frente.
- `AGENTS.md` — bullet correspondente em "Estado atual".
