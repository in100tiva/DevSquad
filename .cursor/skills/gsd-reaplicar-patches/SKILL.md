---
name: gsd-reaplicar-patches
description: "Reaplicar modificações locais após uma atualização do GSD"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-reaplicar-patches` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de input do usuário, pergunte conversacionalmente:
- Apresente opções como lista numerada no texto da resposta
- Peça ao usuário para responder com sua escolha
- Para seleção múltipla, peça números separados por vírgula

## C. Uso de Ferramentas
Use estas ferramentas do Cursor ao executar workflows GSD:
- `Shell` para executar comandos (operações de terminal)
- `StrReplace` para editar arquivos existentes
- `Read`, `Write`, `Glob`, `Grep`, `Task`, `WebSearch`, `WebFetch`, `TodoWrite` conforme necessário

## D. Criação de Subagentes
Quando o workflow precisar criar um subagente:
- Use `Task(subagent_type="generalPurpose", ...)`
- O parâmetro `model` mapeia para as opções de modelo do Cursor (ex: "fast")
</cursor_skill_adapter>

<purpose>
Após uma atualização do GSD limpar e reinstalar arquivos, este comando mescla as modificações locais previamente salvas do usuário de volta na nova versão. Usa comparação inteligente para lidar com casos em que o arquivo upstream também mudou.
</purpose>

<process>

## Passo 1: Detectar patches salvos

Verificar diretório de patches locais:

```bash
# Instalação global — detectar diretório de configuração runtime
if [ -d "$HOME/.config/opencode/gsd-local-patches" ]; then
  PATCHES_DIR="$HOME/.config/opencode/gsd-local-patches"
elif [ -d "$HOME/.opencode/gsd-local-patches" ]; then
  PATCHES_DIR="$HOME/.opencode/gsd-local-patches"
elif [ -d "$HOME/.gemini/gsd-local-patches" ]; then
  PATCHES_DIR="$HOME/.gemini/gsd-local-patches"
else
  PATCHES_DIR="D:/projetos/Estudo/devsquad/.cursor/gsd-local-patches"
fi
# Fallback de instalação local — verificar todos os diretórios runtime
if [ ! -d "$PATCHES_DIR" ]; then
  for dir in .config/opencode .opencode .gemini .claude; do
    if [ -d "./$dir/gsd-local-patches" ]; then
      PATCHES_DIR="./$dir/gsd-local-patches"
      break
    fi
  done
fi
```

Ler `backup-meta.json` do diretório de patches.

**Se nenhum patch encontrado:**
```
Nenhum patch local encontrado. Nada para reaplicar.

Patches locais são salvos automaticamente quando você executa /gsd-atualizar
após modificar qualquer arquivo de workflow, comando ou agente do GSD.
```
Sair.

## Passo 2: Mostrar resumo dos patches

```
## Patches Locais para Reaplicar

**Salvos da:** v{from_version}
**Versão atual:** {ler arquivo VERSION}
**Arquivos modificados:** {quantidade}

| # | Arquivo | Status |
|---|---------|--------|
| 1 | {caminho_arquivo} | Pendente |
| 2 | {caminho_arquivo} | Pendente |
```

## Passo 3: Mesclar cada arquivo

Para cada arquivo em `backup-meta.json`:

1. **Ler a versão salva** (cópia modificada do usuário de `gsd-local-patches/`)
2. **Ler a versão recém-instalada** (arquivo atual após atualização)
3. **Comparar e mesclar:**

   - Se o novo arquivo é idêntico ao salvo: pular (modificação foi incorporada ao upstream)
   - Se o novo arquivo difere: identificar as modificações do usuário e aplicá-las à nova versão

   **Estratégia de mesclagem:**
   - Ler ambas as versões completamente
   - Identificar seções que o usuário adicionou ou modificou (procurar adições, não apenas diferenças da substituição de caminho)
   - Aplicar adições/modificações do usuário à nova versão
   - Se uma seção que o usuário modificou também foi alterada no upstream: sinalizar como conflito, mostrar ambas as versões, perguntar ao usuário qual manter

4. **Escrever resultado mesclado** no local de instalação
5. **Reportar status:**
   - `Mesclado` — modificações do usuário aplicadas sem conflito
   - `Pulado` — modificação já no upstream
   - `Conflito` — usuário escolheu resolução

## Passo 4: Atualizar manifesto

Após reaplicar, regenerar o manifesto de arquivos para que atualizações futuras detectem corretamente como modificações do usuário:

```bash
# O manifesto será regenerado na próxima /gsd-atualizar
# Por enquanto, apenas anotar quais arquivos foram modificados
```

## Passo 5: Opção de limpeza

Perguntar ao usuário:
- "Manter backups dos patches para referência?" → preservar `gsd-local-patches/`
- "Limpar backups dos patches?" → remover diretório `gsd-local-patches/`

## Passo 6: Relatório

```
## Patches Reaplicados

| # | Arquivo | Status |
|---|---------|--------|
| 1 | {caminho_arquivo} | ✓ Mesclado |
| 2 | {caminho_arquivo} | ○ Pulado (já no upstream) |
| 3 | {caminho_arquivo} | ⚠ Conflito resolvido |

{quantidade} arquivo(s) atualizado(s). Suas modificações locais estão ativas novamente.
```

</process>

<success_criteria>
- [ ] Todos os patches salvos processados
- [ ] Modificações do usuário mescladas na nova versão
- [ ] Conflitos resolvidos com input do usuário
- [ ] Status reportado para cada arquivo
</success_criteria>
</output>
