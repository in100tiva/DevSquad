# Parsing de Argumento de Fase

Parsear e normalizar argumentos de fase para comandos que operam em fases.

## Extração

De `{{GSD_ARGS}}`:
- Extrair número da fase (primeiro argumento numérico)
- Extrair flags (prefixadas com `--`)
- Texto restante é descrição (para comandos insert/add)

## Usando gsd-tools

O comando `find-phase` trata normalização e validação em um passo:

```bash
PHASE_INFO=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" find-phase "${PHASE}")
```

Retorna JSON com:
- `found`: true/false
- `directory`: Caminho completo para diretório da fase
- `phase_number`: Número normalizado (ex: "06", "06.1")
- `phase_name`: Parte do nome (ex: "foundation")
- `plans`: Array de arquivos PLAN.md
- `summaries`: Array de arquivos SUMMARY.md

## Normalização Manual (Legado)

Preencher com zero fases inteiras para 2 dígitos. Preservar sufixos decimais.

```bash
# Normalizar número da fase
if [[ "$PHASE" =~ ^[0-9]+$ ]]; then
  # Inteiro: 8 → 08
  PHASE=$(printf "%02d" "$PHASE")
elif [[ "$PHASE" =~ ^([0-9]+)\.([0-9]+)$ ]]; then
  # Decimal: 2.1 → 02.1
  PHASE=$(printf "%02d.%s" "${BASH_REMATCH[1]}" "${BASH_REMATCH[2]}")
fi
```

## Validação

Use `roadmap get-phase` para validar que a fase existe:

```bash
PHASE_CHECK=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" roadmap get-phase "${PHASE}" --pick found)
if [ "$PHASE_CHECK" = "false" ]; then
  echo "ERRO: Fase ${PHASE} não encontrada no roadmap"
  exit 1
fi
```

## Consulta de Diretório

Use `find-phase` para consulta de diretório:

```bash
PHASE_DIR=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" find-phase "${PHASE}" --raw)
```
