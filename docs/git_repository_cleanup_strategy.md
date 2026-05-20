# Estratégia de Saneamento do Repositório Git

## Contexto

Durante a análise da Central Operacional/Home Assistant foi identificado crescimento excessivo do repositório Git local e impacto direto no tamanho dos backups.

Medições realizadas em `/config`:

| Item | Tamanho aproximado |
|---|---:|
| `.git` | 15 GB |
| `.git/objects` | 15 GB |
| `.git/objects/pack` | 15 GB |
| `home-assistant_v2.db` | 1.7 GB |
| `home-assistant_v2.db-wal` | 8.6 MB |
| `.storage` | 6.7 MB |
| `zigbee2mqtt` | 103 MB |
| `custom_components` | 50 MB |

Arquivos de pack encontrados:

| Pack | Tamanho aproximado | Data |
|---|---:|---|
| `pack-12f65f8f...pack` | 7.3 GB | 2026-04-10 |
| `pack-583e42e9...pack` | 7.3 GB | 2026-04-29 |
| `pack-5d045e03...pack` | 412 MB | 2026-05-13 |
| `pack-748caad3...pack` | 410 MB | 2026-05-12 |

O backup Google foi observado com aproximadamente 22.5 GB.

## Diagnóstico

O principal fator de crescimento do backup não é somente o banco SQLite.

A composição provável é:

| Componente | Impacto provável |
|---|---:|
| `.git` | ~15 GB |
| `home-assistant_v2.db` | ~1.7 GB |
| demais arquivos de configuração, integrações e metadados | centenas de MB |
| overhead/compressão/estrutura do backup | variável |

Portanto, mesmo que o `recorder` seja otimizado, o backup continuará grande enquanto `.git` for incluído.

## Causas Prováveis

O diretório `.git/objects/pack` concentra praticamente todo o espaço usado pelo Git.

As causas mais prováveis são:

1. Arquivos grandes foram adicionados ao histórico Git em algum momento.
2. Esses arquivos podem ter sido removidos do working tree depois, mas permaneceram nos objetos do Git.
3. Possíveis candidatos:
   - `home-assistant_v2.db`
   - arquivos `.db-wal` ou `.db-shm`
   - backups locais
   - arquivos de mídia
   - pastas de runtime do Home Assistant
   - `.storage`
   - `zigbee2mqtt`
   - `custom_components`
   - dumps temporários
4. O branch atual parece não referenciar objetos grandes diretamente, mas os packs continuam ocupando espaço.
5. O `.gitignore` atual protege o futuro, mas não remove objetos já gravados em packs antigos.

## Evidências

`git count-objects -vH` indicou:

```text
count: 2749
size: 68.44 MiB
in-pack: 4
packs: 4
size-pack: 15.32 GiB
prune-packable: 0
garbage: 0 bytes
```

A listagem de objetos alcançáveis pelo branch atual mostrou apenas arquivos pequenos da baseline V20, como:

- `.gitignore`
- `docs/release_central_operacional_v20.md`
- `packages/motor_eventos_v20.yaml`
- `packages/motor_timeline_v20.yaml`
- `packages/parametros_operacionais_v20.yaml`
- `packages/central_operacional_aliases_v20.yaml`

Isso sugere que os packs grandes podem estar relacionados a objetos antigos, objetos inalcançáveis, commits anteriores, tentativa anterior de versionamento ou histórico que precisa ser auditado com ferramenta própria antes de qualquer limpeza.

## Relação Entre `.git`, Recorder e Backup

### Recorder

O banco `home-assistant_v2.db` tem aproximadamente 1.7 GB.

Esse tamanho é relevante e deve ser reduzido com:

- `purge_keep_days` menor
- exclusão de entidades verbosas
- `recorder.purge` com `repack`
- preservação de long term statistics e Energy Dashboard

### Git

O `.git` tem aproximadamente 15 GB.

Esse tamanho não é reduzido por:

- `.gitignore`
- remover arquivos do working tree
- reduzir o `recorder`
- apagar arquivos já removidos do projeto

Ele só será reduzido com uma estratégia específica de saneamento Git.

### Backup Google

Se o backup inclui a pasta `/config` inteira e não exclui `.git`, então `.git` é copiado junto.

Com base nos tamanhos atuais, é altamente provável que `.git` esteja sendo incluído no backup, pois:

- `.git` tem ~15 GB
- `home-assistant_v2.db` tem ~1.7 GB
- backup Google chega a ~22.5 GB

Não foi encontrada, dentro de `/config`, uma configuração explícita de exclusão do `.git` para o backup Google. A configuração efetiva pode estar no add-on/supervisor, fora do escopo direto deste diretório.

## Revisão do `.gitignore`

O `.gitignore` atual já protege itens importantes:

- `.storage/`
- `home-assistant_v2.db*`
- `*.db`
- `*.log`
- `secrets.yaml`
- `ssh/`
- `custom_components/`
- `zigbee2mqtt/`
- `esphome/`
- `node-red/`
- `backupwm/`
- `image/`
- `backups/`
- `backup/`

Essa proteção é boa para evitar novos commits acidentais.

Limitação importante:

> `.gitignore` não remove arquivos já presentes no histórico Git nem reduz packs existentes.

## Plano de Remediação

### Fase 1 - Curto Prazo, Baixo Risco

Objetivo: reduzir imediatamente o tamanho dos backups sem tocar no histórico Git.

1. Excluir `.git` do backup Google/Home Assistant.
2. Confirmar se o add-on Google Drive Backup permite exclusões por pasta.
3. Se permitir, adicionar:

```text
/config/.git
```

ou equivalente conforme a interface do add-on.

4. Manter `.gitignore` atual.
5. Otimizar `recorder.yaml` separadamente para reduzir `home-assistant_v2.db`.
6. Rodar purge/repack do recorder em janela segura:

```yaml
action: recorder.purge
data:
  keep_days: 7
  repack: true
  apply_filter: true
```

7. Gerar novo backup e comparar tamanho.

Ganho esperado:

| Ação | Ganho estimado |
|---|---:|
| Excluir `.git` do backup | até ~15 GB |
| Otimizar recorder | ~0.5 a 1.5 GB inicialmente |
| Excluir caches/runtime extras | centenas de MB |

Resultado esperado:

- Backup pode cair de ~22.5 GB para algo próximo de 5-7 GB, dependendo dos add-ons incluídos e compressão.

Risco:

- Baixo.
- Não altera Git.
- Não altera baseline V20.
- Não remove histórico.

Rollback:

- Remover a exclusão de `.git` no add-on de backup.
- Fazer novo backup completo se desejar preservar o repositório local dentro do backup.

### Fase 2 - Médio Prazo, Auditoria do Histórico

Objetivo: descobrir exatamente quais objetos causaram os packs gigantes.

Atividades recomendadas:

1. Clonar ou copiar o repositório para uma área temporária fora do `/config`.
2. Trabalhar na cópia, nunca diretamente na baseline produtiva.
3. Auditar objetos grandes com ferramentas adequadas:

```bash
git count-objects -vH
git rev-list --objects --all
git verify-pack -v .git/objects/pack/*.idx
```

4. Se disponível, usar ferramenta própria para análise:

```bash
git-sizer
```

5. Identificar se os objetos grandes são:
   - banco SQLite
   - backups
   - mídia
   - `.storage`
   - pastas inteiras de runtime

Ganho esperado:

- Nenhum ganho imediato.
- Gera mapa seguro para decidir limpeza profunda.

Risco:

- Baixo se feito em cópia.
- Médio se executado no repositório produtivo.

Rollback:

- Descartar a cópia temporária.
- Não tocar no repo original.

### Fase 3 - Limpeza Profunda Futura

Objetivo: reduzir `.git` de 15 GB para poucos MB.

Opções seguras:

#### Opção A - Repositório limpo novo

Recomendação preferida para este caso.

1. Criar uma nova pasta/repositório limpo.
2. Copiar apenas arquivos versionáveis:
   - `README.md`
   - `CHANGELOG.md`
   - `docs/`
   - `packages/`
   - arquivos YAML sanitizados necessários
   - `.gitignore`
3. Não copiar:
   - `.git`
   - `.storage`
   - `home-assistant_v2.db*`
   - `secrets.yaml`
   - `zigbee2mqtt`
   - `custom_components`
   - backups
   - logs
4. Inicializar Git novo.
5. Criar commit da baseline V20.
6. Criar tag:

```bash
git tag v20-central-operacional
```

Ganho esperado:

- `.git` novo provavelmente abaixo de 10-50 MB.
- Backup reduzido se o `.git` novo ainda for incluído.

Risco:

- Baixo a médio.
- Perde histórico antigo problemático, mas preserva baseline limpa.

Rollback:

- Manter o repositório antigo arquivado antes da troca.
- Não apagar `.git` antigo até validar o novo.

#### Opção B - Reescrever histórico

Usar `git filter-repo` para remover arquivos grandes do histórico.

Não recomendado como primeira opção dentro de `/config`.

Exemplo conceitual, não executar sem cópia e validação:

```bash
git filter-repo --path home-assistant_v2.db --invert-paths
```

Também seria necessário filtrar padrões como:

```text
home-assistant_v2.db*
.storage/
*.log
backups/
backup/
zigbee2mqtt/
custom_components/
```

Depois disso, seria necessário:

```bash
git gc --prune=now --aggressive
```

Risco:

- Alto.
- Reescreve histórico.
- Pode quebrar tags/remotes.
- Pode exigir force push.
- Pode ser pesado demais para rodar dentro do ambiente do Home Assistant.

Rollback:

- Só é seguro se houver cópia completa do repositório antes.
- Não executar sem backup externo.

## Estratégia Segura de Rollback

Antes de qualquer limpeza profunda:

1. Fazer backup completo do diretório `.git` atual em mídia externa ou local fora do backup automático.
2. Exportar os arquivos versionáveis da baseline V20 para uma pasta limpa.
3. Confirmar que `git status` e `git log` funcionam no repositório atual.
4. Não apagar o `.git` antigo até o novo repositório estar validado.
5. Validar que o Home Assistant inicia sem depender de qualquer arquivo dentro de `.git`.

## Recomendações

### Recomendação Imediata

Excluir `.git` do Google Drive Backup.

Essa ação é o maior ganho com menor risco.

### Recomendação Para a Baseline V20

Congelar a baseline V20 em um repositório limpo futuro, com apenas arquivos necessários e documentação.

O repositório atual pode ser tratado como área de transição.

### Recomendação Para o Recorder

Otimizar `recorder.yaml`, mas tratar isso como segundo problema.

O recorder explica ~1.7 GB.

O Git explica ~15 GB.

### Recomendação Para Git

Evitar `git filter-repo`, `git gc --aggressive`, remoção manual de objects ou reset/reclone diretamente no `/config` produtivo.

Preferir:

1. excluir `.git` do backup;
2. criar repo limpo paralelo;
3. validar baseline;
4. só depois aposentar o repo antigo.

## Critérios de Sucesso

- Backup Google cai significativamente após excluir `.git`.
- Baseline V20 permanece intacta.
- Home Assistant continua iniciando normalmente.
- `.gitignore` impede novos arquivos sensíveis.
- Novo repositório limpo contém somente arquivos versionáveis.
- Nenhum arquivo sensível é versionado.

## Próximo Passo Recomendado

Configurar a exclusão de `.git` no mecanismo de backup Google/Home Assistant e executar um novo backup de teste.

Depois comparar:

- tamanho anterior: ~22.5 GB
- tamanho novo esperado: aproximadamente 5-7 GB, dependendo de add-ons e banco

Somente após essa validação iniciar a criação de um repositório limpo para a baseline V20.
