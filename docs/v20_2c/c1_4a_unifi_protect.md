# C1.4A — Descoberta da integração UniFi Protect

## Escopo e evidências

Levantamento somente leitura realizado em 2026-08-05 no runtime do Home Assistant, na entrada carregada `unifiprotect` intitulada “UNVR Instant WM”. Foram consultados o registro de dispositivos, o registro de entidades, os estados e atributos atuais, o catálogo de serviços, o diagnóstico redigido da integração e a documentação oficial do Home Assistant. Nenhum serviço foi executado e nenhum estado, opção, configuração ou equipamento foi alterado.

A entrada usa UniFi Protect 7.1.87, possui `disable_rtsp: false` e expõe um NVR e duas câmeras. O diagnóstico confirma duas câmeras conectadas, ambas G4 Instant 5.3.95, com gravação por detecções e detecção de movimento habilitadas. Os nomes redigidos do diagnóstico não foram usados para identificar os equipamentos; a identificação abaixo vem dos registros oficiais do Home Assistant.

## Arquitetura encontrada

| Equipamento | Device ID | Modelo / versão | Área | Vínculo |
| --- | --- | --- | --- | --- |
| UNVR Instant WM | `1ccc6a78f064eceee0a8f2b39e0d8f7d` | UNVR Instant / 7.1.87 | Home Office | raiz da integração |
| G4 Instant - Cozinha | `f97d84540016a17704b547299a9d54db` | G4 Instant / 5.3.95 | Cozinha | via UNVR Instant WM |
| G4 Instant - Quarto | `db735aba545ccc720c63a51beb420c4f` | G4 Instant / 5.3.95 | sem área atribuída | via UNVR Instant WM |

O inventário contém 111 entidades UniFi Protect: 84 ativas e 27 desabilitadas pela integração. As entidades adicionais `device_tracker` e `sensor` fornecidas pela integração UniFi Network para os mesmos equipamentos não são capacidades do UniFi Protect e, por isso, não integram a tabela abaixo.

## Estado atual da gravação

| Câmera | Controle | Estado | Opções expostas |
| --- | --- | --- | --- |
| G4 Instant - Cozinha | `select.g4_instant_recording_mode` | `detections` | `adaptive`, `always`, `never`, `schedule`, `detections` |
| G4 Instant - Quarto | `select.g4_instant_recording_mode_2` | `detections` | `adaptive`, `always`, `never`, `schedule`, `detections` |

O NVR informa 100% do vídeo armazenado como detecções, 0% como vídeo contínuo e 0% como timelapse. A utilização de armazenamento observada foi 32,05%. A configuração é individual por câmera; não foi encontrada entidade de modo de gravação global no NVR. O Alarm Manager do NVR está registrado, porém `unavailable`, compatível com a limitação de perfis quando o Alarm Manager do Protect não está em modo Local.

## Inventário completo de entidades

Os estados representam o instante da auditoria. “Desabilitada” significa entidade existente no registro, mas não carregada na máquina de estados; não é sinônimo de recurso inexistente no equipamento.

+| Equipamento | Entity ID | Friendly name | Estado | Registro |
| --- | --- | --- | --- | --- |
| UNVR Instant WM | `alarm_control_panel.unvr_instant_wm_alarm_manager` | UNVR Instant WM Alarm Manager | `unavailable` | ativo |
| UNVR Instant WM | `binary_sensor.unvr_instant_wm_hdd_1` | UNVR Instant WM HDD 1 | `off` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_cpu_temperature` | CPU temperature | `desabilitada` | desabilitada (integration) |
| UNVR Instant WM | `sensor.unvr_instant_wm_cpu_utilization` | CPU utilization | `desabilitada` | desabilitada (integration) |
| UNVR Instant WM | `sensor.unvr_instant_wm_memory_utilization` | Memory utilization | `desabilitada` | desabilitada (integration) |
| UNVR Instant WM | `sensor.unvr_instant_wm_recording_capacity` | UNVR Instant WM Recording capacity | `12267183` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_resolution_4k_video` | UNVR Instant WM Resolution: 4K video | `unknown` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_resolution_free_space` | UNVR Instant WM Resolution: free space | `unknown` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_resolution_hd_video` | UNVR Instant WM Resolution: HD video | `unknown` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_storage_utilization` | UNVR Instant WM Storage utilization | `32.05` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_type_continuous_video` | UNVR Instant WM Type: continuous video | `0` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_type_detections_video` | UNVR Instant WM Type: detections video | `100` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_type_timelapse_video` | UNVR Instant WM Type: timelapse video | `0` | ativo |
| UNVR Instant WM | `sensor.unvr_instant_wm_uptime` | UNVR Instant WM Uptime | `2026-08-02T03:51:00+00:00` | ativo |
| UNVR Instant WM | `switch.unvr_instant_wm_analytics_enabled` | UNVR Instant WM Analytics enabled | `on` | ativo |
| UNVR Instant WM | `switch.unvr_instant_wm_insights_enabled` | UNVR Instant WM Insights enabled | `on` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_is_dark_2` | G4 Instant - Quarto Is dark | `off` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_movimento_2` | G4 Instant - Quarto Movimento | `off` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_animal_detected` | G4 Instant - Quarto Animal detected | `unavailable` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_audio_object_detected` | Audio object detected | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_baby_cry_detected` | G4 Instant - Quarto Baby cry detected | `off` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_co_alarm_detected` | G4 Instant - Quarto CO alarm detected | `off` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_object_detected` | Object detected | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_person_detected` | G4 Instant - Quarto Person detected | `off` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_smoke_alarm_detected` | G4 Instant - Quarto Smoke alarm detected | `off` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_speaking_detected` | G4 Instant - Quarto Speaking detected | `unavailable` | ativo |
| G4 Instant - Quarto | `binary_sensor.g4_instant_quarto_vehicle_detected` | G4 Instant - Quarto Vehicle detected | `unavailable` | ativo |
| G4 Instant - Quarto | `button.g4_instant_reiniciar_2` | Reiniciar | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `button.g4_instant_unadopt_device_2` | Unadopt device | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `camera.g4_instant_high_resolution_channel_2` | G4 Instant - Quarto High resolution channel | `idle` | ativo |
| G4 Instant - Quarto | `event.g4_instant_quarto_vehicle` | G4 Instant - Quarto Vehicle | `unknown` | ativo |
| G4 Instant - Quarto | `media_player.g4_instant_quarto_speaker` | G4 Instant - Quarto Speaker | `idle` | ativo |
| G4 Instant - Quarto | `number.g4_instant_quarto_infrared_custom_lux_trigger` | G4 Instant - Quarto Infrared custom lux trigger | `unavailable` | ativo |
| G4 Instant - Quarto | `number.g4_instant_quarto_microphone_level` | G4 Instant - Quarto Microphone level | `100` | ativo |
| G4 Instant - Quarto | `number.g4_instant_quarto_system_sounds_volume` | G4 Instant - Quarto System sounds volume | `100` | ativo |
| G4 Instant - Quarto | `number.g4_instant_wide_dynamic_range` | G4 Instant - Quarto Wide dynamic range | `unavailable` | ativo |
| G4 Instant - Quarto | `select.g4_instant_quarto_hdr_mode` | G4 Instant - Quarto HDR mode | `auto` | ativo |
| G4 Instant - Quarto | `select.g4_instant_quarto_infrared_mode` | G4 Instant - Quarto Infrared mode | `auto` | ativo |
| G4 Instant - Quarto | `select.g4_instant_recording_mode_2` | G4 Instant - Quarto Recording mode | `detections` | ativo |
| G4 Instant - Quarto | `sensor.g4_instant_disk_write_rate_2` | G4 Instant - Quarto Disk write rate | `0.02158384` | ativo |
| G4 Instant - Quarto | `sensor.g4_instant_last_motion_detected_2` | Last motion detected | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `sensor.g4_instant_oldest_recording_2` | Oldest recording | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `sensor.g4_instant_quarto_wi_fi_signal_strength` | Wi-Fi signal strength | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `sensor.g4_instant_received_data_2` | Received data | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `sensor.g4_instant_storage_used_2` | G4 Instant - Quarto Storage used | `37580.96384` | ativo |
| G4 Instant - Quarto | `sensor.g4_instant_transferred_data_2` | Transferred data | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `sensor.g4_instant_uptime_2` | Uptime | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `switch.g4_instant_motion_2` | G4 Instant - Quarto Motion | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_overlay_show_date_2` | G4 Instant - Quarto Overlay: show date | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_overlay_show_logo_2` | G4 Instant - Quarto Overlay: show logo | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_overlay_show_name_2` | G4 Instant - Quarto Overlay: show name | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_overlay_show_nerd_mode_2` | G4 Instant - Quarto Overlay: show nerd mode | `off` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_animal_detection` | G4 Instant - Quarto Animal detection | `off` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_baby_cry_detection` | G4 Instant - Quarto Baby cry detection | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_co_alarm_detection` | G4 Instant - Quarto CO alarm detection | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_hdr_mode` | HDR mode | `desabilitada` | desabilitada (integration) |
| G4 Instant - Quarto | `switch.g4_instant_quarto_modo_de_privacidade` | G4 Instant - Quarto Modo de privacidade | `off` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_person_detection` | G4 Instant - Quarto Person detection | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_smoke_detection` | G4 Instant - Quarto Smoke detection | `on` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_speaking_detection` | G4 Instant - Quarto Speaking detection | `off` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_status_light` | G4 Instant - Quarto Status light | `off` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_system_sounds` | G4 Instant - Quarto System sounds | `off` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_quarto_vehicle_detection` | G4 Instant - Quarto Vehicle detection | `off` | ativo |
| G4 Instant - Quarto | `switch.g4_instant_ssh_habilitado_2` | SSH habilitado | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_animal_detected` | G4 Instant - Cozinha Animal detected | `unavailable` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_audio_object_detected` | Audio object detected | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_baby_cry_detected` | G4 Instant - Cozinha Baby cry detected | `unavailable` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_co_alarm_detected` | G4 Instant - Cozinha CO alarm detected | `off` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_is_dark` | G4 Instant - Cozinha Is dark | `off` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_movimento` | G4 Instant - Cozinha Movimento | `off` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_object_detected` | Object detected | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_person_detected` | G4 Instant - Cozinha Person detected | `off` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_smoke_alarm_detected` | G4 Instant - Cozinha Smoke alarm detected | `off` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_speaking_detected` | G4 Instant - Cozinha Speaking detected | `off` | ativo |
| G4 Instant - Cozinha | `binary_sensor.g4_instant_vehicle_detected` | G4 Instant - Cozinha Vehicle detected | `off` | ativo |
| G4 Instant - Cozinha | `button.g4_instant_reiniciar` | Reiniciar | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `button.g4_instant_unadopt_device` | Unadopt device | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `camera.g4_instant_high_resolution_channel` | G4 Instant - Cozinha High resolution channel | `idle` | ativo |
| G4 Instant - Cozinha | `event.g4_instant_vehicle` | G4 Instant - Cozinha Vehicle | `unknown` | ativo |
| G4 Instant - Cozinha | `media_player.g4_instant_speaker` | G4 Instant - Cozinha Speaker | `idle` | ativo |
| G4 Instant - Cozinha | `number.g4_instant_infrared_custom_lux_trigger` | G4 Instant - Cozinha Infrared custom lux trigger | `unavailable` | ativo |
| G4 Instant - Cozinha | `number.g4_instant_microphone_level` | G4 Instant - Cozinha Microphone level | `72` | ativo |
| G4 Instant - Cozinha | `number.g4_instant_system_sounds_volume` | G4 Instant - Cozinha System sounds volume | `100` | ativo |
| G4 Instant - Cozinha | `select.g4_instant_hdr_mode` | G4 Instant - Cozinha HDR mode | `auto` | ativo |
| G4 Instant - Cozinha | `select.g4_instant_infrared_mode` | G4 Instant - Cozinha Infrared mode | `auto` | ativo |
| G4 Instant - Cozinha | `select.g4_instant_recording_mode` | G4 Instant - Cozinha Recording mode | `detections` | ativo |
| G4 Instant - Cozinha | `sensor.g4_instant_disk_write_rate` | G4 Instant - Cozinha Disk write rate | `0.04394855` | ativo |
| G4 Instant - Cozinha | `sensor.g4_instant_last_motion_detected` | Last motion detected | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `sensor.g4_instant_oldest_recording` | Oldest recording | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `sensor.g4_instant_received_data` | Received data | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `sensor.g4_instant_storage_used` | G4 Instant - Cozinha Storage used | `236223.20128` | ativo |
| G4 Instant - Cozinha | `sensor.g4_instant_transferred_data` | Transferred data | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `sensor.g4_instant_uptime` | Uptime | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `sensor.g4_instant_wi_fi_signal_strength` | Wi-Fi signal strength | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `switch.g4_instant_animal_detection` | G4 Instant - Cozinha Animal detection | `off` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_baby_cry_detection` | G4 Instant - Cozinha Baby cry detection | `off` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_co_alarm_detection` | G4 Instant - Cozinha CO alarm detection | `on` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_hdr_mode` | HDR mode | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `switch.g4_instant_modo_de_privacidade` | G4 Instant - Cozinha Modo de privacidade | `off` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_motion` | G4 Instant - Cozinha Motion | `on` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_overlay_show_date` | G4 Instant - Cozinha Overlay: show date | `on` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_overlay_show_logo` | G4 Instant - Cozinha Overlay: show logo | `off` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_overlay_show_name` | G4 Instant - Cozinha Overlay: show name | `on` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_overlay_show_nerd_mode` | G4 Instant - Cozinha Overlay: show nerd mode | `off` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_person_detection` | G4 Instant - Cozinha Person detection | `on` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_smoke_detection` | G4 Instant - Cozinha Smoke detection | `on` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_speaking_detection` | G4 Instant - Cozinha Speaking detection | `on` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_ssh_habilitado` | SSH habilitado | `desabilitada` | desabilitada (integration) |
| G4 Instant - Cozinha | `switch.g4_instant_status_light` | G4 Instant - Cozinha Status light | `off` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_system_sounds` | G4 Instant - Cozinha System sounds | `off` | ativo |
| G4 Instant - Cozinha | `switch.g4_instant_vehicle_detection` | G4 Instant - Cozinha Vehicle detection | `on` | ativo |

## Atributos relevantes

- As duas entidades `camera` estão `idle`, com canal de alta resolução ativo em 2688 × 1512, 30 fps e detecção de movimento habilitada.
- Os dois selects de gravação expõem exatamente as cinco opções registradas acima.
- Os selects de infravermelho estão em `auto` e aceitam `auto`, `on`, `auto_filter_only`, `custom` e `off`.
- Os selects HDR estão em `auto` e aceitam `always`, `off` e `auto`.
- Os dois switches de modo de privacidade estão `off`. Segundo o contrato oficial da integração, ativá-los desabilita gravação e microfone e aplica uma máscara integral.
- As duas câmeras expõem eventos de veículo, sensores binários de movimento/objetos e controles por tipo de detecção. Não existe entidade de sirene, spotlight/floodlight ou PTZ neste ambiente.

## Serviços e interfaces disponíveis

### Serviços próprios de `unifiprotect`

| Serviço | Finalidade | Aplicável aos equipamentos encontrados |
| --- | --- | --- |
| `unifiprotect.add_doorbell_text` | adicionar texto de campainha | não; não há campainha |
| `unifiprotect.remove_doorbell_text` | remover texto de campainha | não; não há campainha |
| `unifiprotect.set_chime_paired_doorbells` | parear campainhas a chime | não; não há chime |
| `unifiprotect.remove_privacy_zone` | remover zona de privacidade de câmera | interface disponível; nenhuma ação executada |
| `unifiprotect.get_user_keyring_info` | consultar chaveiros de usuário | interface disponível; fora do escopo de gravação |
| `unifiprotect.ptz_goto_preset` | mover câmera PTZ para preset | não; as G4 Instant não são PTZ |

Nenhum desses seis serviços inicia, interrompe ou altera o modo de gravação.

### Interfaces genéricas aplicáveis

- `select.select_option` pode alterar cada `select.*_recording_mode` para uma das opções efetivamente expostas. Os serviços de navegação do domínio `select` também existem, mas não são necessários para comprovar a capacidade.
- `camera.snapshot` pode salvar uma imagem do stream; `camera.record` pode gravar localmente um trecho do stream no Home Assistant. Esse último não altera nem representa a política de gravação do NVR.
- O media source do UniFi Protect e os endpoints proxy oficiais permitem consultar thumbnails, snapshots históricos e clips/eventos gravados. O runtime não expõe um serviço `unifiprotect` de download de clip.

## Matriz de capacidades reais

| Recurso | Resultado no ambiente |
| --- | --- |
| Iniciar/interromper gravação por serviço dedicado | não disponível |
| Alterar modo de gravação | disponível por câmera via `select.select_option` |
| Gravação contínua | opção `always` disponível por câmera; não ativada na auditoria |
| Gravação por eventos | opção `detections` disponível e ativa nas duas câmeras |
| Gravação por agenda | opção `schedule` disponível por câmera |
| Modo adaptativo | opção `adaptive` disponível por câmera |
| Desabilitar gravação | opção `never` disponível por câmera |
| Perfil/modo global do NVR | não encontrado |
| Configuração individual por câmera | disponível |
| Snapshot atual | disponível via entidade/serviço genérico de câmera |
| Snapshot histórico | disponível pela view proxy oficial do UniFi Protect |
| Clips/eventos gravados | disponíveis via media source e views proxy; sem serviço próprio de download |
| Eventos | eventos de veículo e sensores de detecção presentes |
| Detecção IA | controles e sensores de pessoa, veículo, animal e sons conforme cada câmera |
| Privacy mode | disponível individualmente nas duas câmeras |
| Sirene | não disponível nos equipamentos encontrados |
| Spotlight/floodlight | não disponível nos equipamentos encontrados |
| PTZ | não suportado pelas duas G4 Instant encontradas |

## Limitações comprovadas

- Não há controle global de gravação no dispositivo NVR.
- Não há serviço UniFi Protect específico para iniciar/parar gravação; o controle persistente é o select individual de cada câmera.
- `camera.record` grava o stream no Home Assistant e não deve ser confundido com controle do NVR.
- Não há entidade que represente um “perfil contextual” único para aplicar às duas câmeras.
- A integração não expõe sirene, spotlight, floodlight ou PTZ porque esses equipamentos/capacidades não existem no inventário atual.
- A auditoria confirma as opções expostas, mas não alterou nenhuma delas; portanto, não homologa transição de modo nem efeito físico no NVR.

## Conclusão para C1.4B

A integração suporta a capacidade técnica necessária para uma futura C1.4B somente no nível de alteração individual do modo de gravação das duas câmeras, usando os selects existentes. Ela não suporta um comando global do NVR nem um serviço dedicado de início/parada de gravação. Esta conclusão é de compatibilidade técnica; nenhuma implementação, escolha de modo ou política de restauração foi definida neste lote.
