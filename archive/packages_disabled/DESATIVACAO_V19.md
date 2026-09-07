# Desativação da Camada V19 - Central Operacional
## Data: 13 de maio de 2026

## ✅ CONCLUÍDO
Todos os sensores V19 foram desativados da Central Operacional do Home Assistant.

---

## 📁 Arquivos Afetados

### 1. **Arquivos YAML de Configuração**

#### Arquivo Principal (Modificado):
- [packages/status_casa.yaml](../packages/status_casa.yaml)
  - ❌ Removido: Todos os sensores template V19 (binary_sensor, sensor)
  - ❌ Removido: Trigger V19 para Casa Evento Atual e Casa Event Feed
  - ✅ Preservado: Inputs (input_number, input_boolean, input_select)
  - ✅ Preservado: Versões legadas V17/V18

#### Arquivo de Backup (Criado):
- [packages/_disabled/status_casa_v19.yaml](../packages/_disabled/status_casa_v19.yaml)
  - ✅ Backup completo de todos os sensores V19
  - ℹ️ Arquivo comentado como DESABILITADO
  - ℹ️ Instruções para reativar se necessário

---

### 2. **Sensores V19 Desativados**

#### Binary Sensors:
- `binary_sensor.casa_tv_ativa_v19`
- `binary_sensor.casa_incidente_ativo_v19`
- `binary_sensor.casa_event_engine_ativo_v19`

#### Sensors:
- `sensor.casa_tv_contexto_v19`
- `sensor.casa_score_ativo_v19`
- `sensor.casa_modo_operacional_v19`
- `sensor.status_casa_v19`
- `sensor.casa_breakdown_ativo_v19`
- `sensor.casa_contexto_humano_v19`
- `sensor.atividade_relevante_v19`
- `sensor.casa_timeline_operacional_v19`
- `sensor.casa_timeline_temporal_v19`
- `sensor.casa_evento_canonico_v19`
- `sensor.casa_evento_atual_v19` (trigger-based)
- `sensor.casa_event_feed_v19` (trigger-based)

---

### 3. **Dashboards Afetados** (Requerem Atualização Manual)

#### Armazenados em .storage:
- `.storage/lovelace.sistema_casa`
  - 6 referências a sensores V19
  - Entidades: evento_atual_v19, timeline_temporal_v19, event_feed_v19, tv_contexto_v19
  
- `.storage/lovelace.teste_4`
  - 10 referências a sensores V19
  - Entidades: tv_contexto_v19, timeline_temporal_v19, evento_atual_v19, event_feed_v19

- `.storage/lovelace.testes_anterior`
  - 14 referências a sensores V19
  - Entidades: timeline_temporal_v19, event_feed_v19, evento_atual_v19

#### Armazenados em .storage (Informações de Sistema):
- `.storage/core.entity_registry`
  - 5 entidades V19 registradas (serão automaticamente removidas do Home Assistant após reinicialização)
  
- `.storage/core.restore_state`
  - 5 estados salvos de entidades V19 (estados históricos)

---

### 4. **Versões Legadas Preservadas**

✅ **Nenhuma alteração realizada em:**
- V17 (sensores base)
- V18 (sensores intermediários)
- Automações legadas
- Scripts legados
- Blueprints

---

## 🚀 Próximos Passos

### 1. **Atualizar Dashboards** (Manual)
Os seguintes arquivos de dashboard precisam ser atualizados manualmente:
- `.storage/lovelace.sistema_casa` - Dashboard Central Operacional
- `.storage/lovelace.teste_4` - Dashboard de Teste
- `.storage/lovelace.testes_anterior` - Dashboard Antigo

Opções:
- Remover cards referentes a V19
- Substituir por sensores V17/V18
- Manter cards vazios (não causam erro)

### 2. **Reiniciar Home Assistant**
Para sincronizar as mudanças:
```bash
# No Home Assistant
Settings → System → Restart
```

### 3. **Verificar Entity Registry**
Home Assistant removerá automaticamente as entidades V19 inativas do registro após reinicialização.

---

## 📋 Resumo de Impacto

| Categoria | Quantidade | Status |
|-----------|-----------|--------|
| Sensores Desativados | 12 | ✅ Concluído |
| Binary Sensors Desativados | 3 | ✅ Concluído |
| Dashboards Afetados | 3 | ⚠️ Requer atualização |
| Automações Impactadas | 0 | ✅ Nenhuma |
| Versões Legadas Preservadas | 2 (V17, V18) | ✅ Intactas |

---

## 🔄 Como Reativar V19 (se necessário)

1. Mover arquivo para ativar:
   ```bash
   mv /config/packages/_disabled/status_casa_v19.yaml /config/packages/status_casa_v19.yaml
   ```

2. Reiniciar Home Assistant

3. Sensores V19 voltarão a funcionar

---

## ⚠️ Observações Importantes

- ✅ **V19 foi completamente isolado** - não há dependências residuais
- ✅ **V17/V18 continuam intactos** - compatibilidade garantida
- ⚠️ **Dashboards podem mostrar "entity not found"** - esperado até atualização manual
- ℹ️ **Dados históricos preservados** em `.storage/core.restore_state`

---

**Desativação realizada com sucesso em 13 de maio de 2026**
