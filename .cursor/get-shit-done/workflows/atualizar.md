<purpose>
Verificar atualizações do GSD via npm, exibir changelog para versões entre a instalada e a mais recente, obter confirmação do usuário e executar instalação limpa com limpeza de cache.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="get_installed_version">
Detectar se GSD está instalado local ou globalmente verificando ambos os locais e validando integridade da instalação.

Primeiro, derivar `PREFERRED_RUNTIME` do caminho `execution_context` do prompt invocador:
- Caminho contém `/.codex/` -> `codex`
- Caminho contém `/.gemini/` -> `gemini`
- Caminho contém `/.config/opencode/` ou `/.opencode/` -> `opencode`
- Caso contrário -> `claude`

Usar `PREFERRED_RUNTIME` como o primeiro runtime verificado para que `/gsd-update` direcione o runtime que o invocou.

```bash
# Candidatos de runtime: "<runtime>:<config-dir>" armazenados como array.
# Usar array ao invés de string separada por espaço garante iteração correta
# tanto em bash quanto em zsh (zsh não faz word-split de variáveis
# sem aspas por padrão). Corrige #1173.
RUNTIME_DIRS=( "claude:.claude" "opencode:.config/opencode" "opencode:.opencode" "gemini:.gemini" "codex:.codex" )

# PREFERRED_RUNTIME deve ser definido do execution_context antes de executar este bloco.
# Se não definido, inferir de variáveis de ambiente do runtime; fallback para claude.
if [ -z "$PREFERRED_RUNTIME" ]; then
  if [ -n "$CODEX_HOME" ]; then
    PREFERRED_RUNTIME="codex"
  elif [ -n "$GEMINI_CONFIG_DIR" ]; then
    PREFERRED_RUNTIME="gemini"
  elif [ -n "$OPENCODE_CONFIG_DIR" ] || [ -n "$OPENCODE_CONFIG" ]; then
    PREFERRED_RUNTIME="opencode"
  elif [ -n "$CLAUDE_CONFIG_DIR" ]; then
    PREFERRED_RUNTIME="claude"
  else
    PREFERRED_RUNTIME="claude"
  fi
fi

# Reordenar entradas para que runtime preferido seja verificado primeiro.
ORDERED_RUNTIME_DIRS=()
for entry in "${RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  if [ "$runtime" = "$PREFERRED_RUNTIME" ]; then
    ORDERED_RUNTIME_DIRS+=( "$entry" )
  fi
done
for entry in "${RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  if [ "$runtime" != "$PREFERRED_RUNTIME" ]; then
    ORDERED_RUNTIME_DIRS+=( "$entry" )
  fi
done

# Verificar local primeiro (tem prioridade apenas se válido e distinto do global)
LOCAL_VERSION_FILE="" LOCAL_MARKER_FILE="" LOCAL_DIR="" LOCAL_RUNTIME=""
for entry in "${ORDERED_RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  dir="${entry#*:}"
  if [ -f "./$dir/get-shit-done/VERSION" ] || [ -f "./$dir/get-shit-done/workflows/atualizar.md" ]; then
    LOCAL_RUNTIME="$runtime"
    LOCAL_VERSION_FILE="./$dir/get-shit-done/VERSION"
    LOCAL_MARKER_FILE="./$dir/get-shit-done/workflows/atualizar.md"
    LOCAL_DIR="$(cd "./$dir" 2>/dev/null && pwd)"
    break
  fi
done

GLOBAL_VERSION_FILE="" GLOBAL_MARKER_FILE="" GLOBAL_DIR="" GLOBAL_RUNTIME=""
for entry in "${ORDERED_RUNTIME_DIRS[@]}"; do
  runtime="${entry%%:*}"
  dir="${entry#*:}"
  if [ -f "$HOME/$dir/get-shit-done/VERSION" ] || [ -f "$HOME/$dir/get-shit-done/workflows/atualizar.md" ]; then
    GLOBAL_RUNTIME="$runtime"
    GLOBAL_VERSION_FILE="$HOME/$dir/get-shit-done/VERSION"
    GLOBAL_MARKER_FILE="$HOME/$dir/get-shit-done/workflows/atualizar.md"
    GLOBAL_DIR="$(cd "$HOME/$dir" 2>/dev/null && pwd)"
    break
  fi
done

# Tratar como LOCAL apenas se os caminhos resolvidos forem diferentes (previne detecção errada quando CWD=$HOME)
IS_LOCAL=false
if [ -n "$LOCAL_VERSION_FILE" ] && [ -f "$LOCAL_VERSION_FILE" ] && [ -f "$LOCAL_MARKER_FILE" ] && grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+' "$LOCAL_VERSION_FILE"; then
  if [ -z "$GLOBAL_DIR" ] || [ "$LOCAL_DIR" != "$GLOBAL_DIR" ]; then
    IS_LOCAL=true
  fi
fi

if [ "$IS_LOCAL" = true ]; then
  INSTALLED_VERSION="$(cat "$LOCAL_VERSION_FILE")"
  INSTALL_SCOPE="LOCAL"
  TARGET_RUNTIME="$LOCAL_RUNTIME"
elif [ -n "$GLOBAL_VERSION_FILE" ] && [ -f "$GLOBAL_VERSION_FILE" ] && [ -f "$GLOBAL_MARKER_FILE" ] && grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+' "$GLOBAL_VERSION_FILE"; then
  INSTALLED_VERSION="$(cat "$GLOBAL_VERSION_FILE")"
  INSTALL_SCOPE="GLOBAL"
  TARGET_RUNTIME="$GLOBAL_RUNTIME"
elif [ -n "$LOCAL_RUNTIME" ] && [ -f "$LOCAL_MARKER_FILE" ]; then
  # Runtime detectado mas VERSION ausente/corrupto: tratar como versão desconhecida, manter runtime alvo
  INSTALLED_VERSION="0.0.0"
  INSTALL_SCOPE="LOCAL"
  TARGET_RUNTIME="$LOCAL_RUNTIME"
elif [ -n "$GLOBAL_RUNTIME" ] && [ -f "$GLOBAL_MARKER_FILE" ]; then
  INSTALLED_VERSION="0.0.0"
  INSTALL_SCOPE="GLOBAL"
  TARGET_RUNTIME="$GLOBAL_RUNTIME"
else
  INSTALLED_VERSION="0.0.0"
  INSTALL_SCOPE="UNKNOWN"
  TARGET_RUNTIME="claude"
fi

echo "$INSTALLED_VERSION"
echo "$INSTALL_SCOPE"
echo "$TARGET_RUNTIME"
```

Extrair da saída:
- Linha 1 = versão instalada (`0.0.0` significa versão desconhecida)
- Linha 2 = escopo da instalação (`LOCAL`, `GLOBAL`, ou `UNKNOWN`)
- Linha 3 = runtime alvo (`claude`, `opencode`, `gemini`, ou `codex`)
- Se escopo for `UNKNOWN`, prosseguir para etapa de instalação usando fallback `--claude --global`.

Se múltiplas instalações de runtime forem detectadas e o runtime invocador não puder ser determinado do execution_context, perguntar ao usuário qual runtime atualizar antes de executar instalação.

**Se arquivo VERSION ausente:**
```
## Atualização GSD

**Versão instalada:** Desconhecida

Sua instalação não inclui rastreamento de versão.

Executando instalação limpa...
```

Prosseguir para etapa de instalação (tratar como versão 0.0.0 para comparação).
</step>

<step name="check_latest_version">
Verificar npm para versão mais recente:

```bash
npm view get-shit-done-cc version 2>/dev/null
```

**Se verificação npm falhar:**
```
Não foi possível verificar atualizações (offline ou npm indisponível).

Para atualizar manualmente: `npx get-shit-done-cc --global`
```

Sair.
</step>

<step name="compare_versions">
Comparar instalada vs mais recente:

**Se instalada == mais recente:**
```
## Atualização GSD

**Instalada:** X.Y.Z
**Mais recente:** X.Y.Z

Você já está na versão mais recente.
```

Sair.

**Se instalada > mais recente:**
```
## Atualização GSD

**Instalada:** X.Y.Z
**Mais recente:** A.B.C

Você está à frente do último release (versão de desenvolvimento?).
```

Sair.
</step>

<step name="show_changes_and_confirm">
**Se atualização disponível**, buscar e mostrar o que há de novo ANTES de atualizar:

1. Buscar changelog da URL raw do GitHub
2. Extrair entradas entre versões instalada e mais recente
3. Exibir prévia e pedir confirmação:

```
## Atualização GSD Disponível

**Instalada:** 1.5.10
**Mais recente:** 1.5.15

### O Que Há de Novo
────────────────────────────────────────────────────────────

## [1.5.15] - 2026-01-20

### Adicionado
- Funcionalidade X

## [1.5.14] - 2026-01-18

### Corrigido
- Correção de bug Y

────────────────────────────────────────────────────────────

⚠️  **Nota:** O instalador realiza uma instalação limpa das pastas GSD:
- `commands/gsd/` será limpo e substituído
- `get-shit-done/` será limpo e substituído
- Arquivos `agents/gsd-*` serão substituídos

(Caminhos são relativos ao local de instalação do runtime detectado:
global: `D:/projetos/Estudo/devsquad/.cursor/`, `~/.config/opencode/`, `~/.opencode/`, `~/.gemini/`, ou `~/.codex/`
local: `./.cursor/`, `./.config/opencode/`, `./.opencode/`, `./.gemini/`, ou `./.codex/`)

Seus arquivos personalizados em outros locais são preservados:
- Comandos personalizados fora de `commands/gsd/` ✓
- Agentes personalizados sem prefixo `gsd-` ✓
- Hooks personalizados ✓
- Seus arquivos .cursor/rules/ ✓

Se você modificou algum arquivo GSD diretamente, eles serão automaticamente copiados para `gsd-local-patches/` e podem ser reaplicados com `/gsd-reapply-patches` após a atualização.
```

Usar conversational prompting:
- Pergunta: "Prosseguir com a atualização?"
- Opções:
  - "Sim, atualizar agora"
  - "Não, cancelar"

**Se usuário cancelar:** Sair.
</step>

<step name="run_update">
Executar a atualização usando o tipo de instalação detectado no passo 1:

Construir flag de runtime do passo 1:
```bash
RUNTIME_FLAG="--$TARGET_RUNTIME"
```

**Se instalação LOCAL:**
```bash
npx -y get-shit-done-cc@latest "$RUNTIME_FLAG" --local
```

**Se instalação GLOBAL:**
```bash
npx -y get-shit-done-cc@latest "$RUNTIME_FLAG" --global
```

**Se instalação UNKNOWN:**
```bash
npx -y get-shit-done-cc@latest --claude --global
```

Capturar saída. Se instalação falhar, mostrar erro e sair.

Limpar cache de atualização para que indicador da statusline desapareça:

```bash
# Limpar cache de atualização em todos os diretórios de runtime
for dir in .claude .config/opencode .opencode .gemini .codex; do
  rm -f "./$dir/cache/gsd-update-check.json"
  rm -f "$HOME/$dir/cache/gsd-update-check.json"
done
```

O hook SessionStart (`gsd-check-update.js`) escreve no diretório de cache do runtime detectado, então todos os caminhos devem ser limpos para prevenir indicadores de atualização obsoletos.
</step>

<step name="display_result">
Formatar mensagem de conclusão (changelog já foi mostrado na etapa de confirmação):

```
╔═══════════════════════════════════════════════════════════╗
║  GSD Atualizado: v1.5.10 → v1.5.15                         ║
╚═══════════════════════════════════════════════════════════╝

⚠️  Reinicie seu runtime para usar os novos comandos.

[Ver changelog completo](https://github.com/glittercowboy/get-shit-done/blob/main/CHANGELOG.md)
```
</step>


<step name="check_local_patches">
Após atualização concluir, verificar se o instalador detectou e fez backup de arquivos modificados localmente:

Verificar gsd-local-patches/backup-meta.json no diretório de configuração.

**Se patches encontrados:**

```
Patches locais foram copiados antes da atualização.
Execute /gsd-reapply-patches para mesclar suas modificações na nova versão.
```

**Se sem patches:** Continuar normalmente.
</step>
</process>

<success_criteria>
- [ ] Versão instalada lida corretamente
- [ ] Versão mais recente verificada via npm
- [ ] Atualização pulada se já atualizado
- [ ] Changelog buscado e exibido ANTES da atualização
- [ ] Aviso de instalação limpa mostrado
- [ ] Confirmação do usuário obtida
- [ ] Atualização executada com sucesso
- [ ] Lembrete de reinício mostrado
</success_criteria>
