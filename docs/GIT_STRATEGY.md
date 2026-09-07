# Estratégia Git

## Objetivo

Manter o repositório versionado de forma segura, auditável e adequada para Home Assistant, evitando vazamento de segredos, registries, banco local e arquivos de runtime.

## Política de Commits

Use mensagens curtas, imperativas e com escopo claro.

Exemplos:

```bash
git commit -m "chore: freeze Central Operacional V20 baseline"
git commit -m "docs: add V21 roadmap"
git commit -m "fix: adjust V20 timeline fallback"
```

Tipos sugeridos:

- `chore`: tarefas de manutenção, freeze, estrutura.
- `docs`: documentação.
- `fix`: correção de bug.
- `feat`: nova capacidade.
- `refactor`: reorganização sem mudança funcional.
- `test`: testes ou validações assistidas.

## Tags e Releases

Use tags para baselines estáveis:

```bash
git tag v20-central-operacional
git push origin v20-central-operacional
```

Padrão sugerido:

- `v20-central-operacional`
- `v20.1-central-operacional-hotfix`
- `v21-central-operacional`

## Fluxo Recomendado de Versionamento

Antes de commitar:

```bash
git status --short --branch --untracked-files=no
git diff --cached --name-status
git diff --cached --name-only
```

Adicionar somente arquivos intencionais:

```bash
git add .gitignore
git add README.md CHANGELOG.md
git add docs/ARCHITECTURE.md docs/ROADMAP.md docs/GIT_STRATEGY.md
git add docs/release_central_operacional_v20.md
git add packages/central_operacional_aliases_v20.yaml
git add packages/motor_eventos_v20.yaml
git add packages/motor_timeline_v20.yaml
git add packages/parametros_operacionais_v20.yaml
```

Commit:

```bash
git commit -m "chore: freeze Central Operacional V20 baseline"
```

Tag:

```bash
git tag v20-central-operacional
```

Push:

```bash
git push -u origin main
git push origin v20-central-operacional
```

## Proteção de Arquivos Sensíveis

Nunca commitar:

- `secrets.yaml`
- `home-assistant_v2.db`
- `home-assistant_v2.db-shm`
- `home-assistant_v2.db-wal`
- `*.log`
- `.storage/auth*`
- `.storage/core.config_entries`
- `.storage/core.device_registry`
- `.storage/core.entity_registry`
- `.storage/core.restore_state`
- chaves SSH
- caches
- backups locais

## O Que Não Deve Ser Commitado

Evite versionar:

- `.storage/` bruto.
- Banco do Home Assistant.
- Logs.
- Backups.
- `custom_components/` baixados automaticamente, salvo decisão explícita.
- `www/community/` gerenciado por HACS, salvo decisão explícita.
- Diretórios de runtime como `deps/`, `tts/`, `zigbee2mqtt/log/`.

## Dashboards

Dashboards em `.storage/` podem conter dados sensíveis ou ruído de runtime.

Recomendação:

- Não versionar `.storage/lovelace*` diretamente.
- Exportar dashboards sanitizados para uma pasta dedicada, se necessário.
- Documentar manualmente mudanças importantes em `docs/`.

## Recuperação de Stage Acidental

Se arquivos sensíveis forem adicionados ao stage por acidente:

```bash
git restore --staged <arquivo>
```

Em repositório inicial sem `HEAD`, use:

```bash
git read-tree --empty
```

Esse comando limpa o índice, mas não apaga arquivos do disco.

## Checklist Antes do Push

- `git diff --cached --name-only` mostra somente arquivos intencionais.
- Nenhum arquivo de `.storage/` está staged.
- Nenhum `secrets.yaml` está staged.
- Nenhum banco ou log está staged.
- `.gitignore` protege arquivos sensíveis.
- Release/tag foi revisada.
