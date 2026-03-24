---
name: gsd-revisar-backlog
description: "Revisar e promover itens do backlog para o marco ativo"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-revisar-backlog` ou descreve uma tarefa correspondente a esta skill.
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

<objective>
Revisar todos os itens 999.x do backlog e opcionalmente promovê-los para a
sequência do marco ativo ou remover entradas obsoletas.
</objective>

<process>

1. **Listar itens do backlog:**
   ```bash
   ls -d .planning/phases/999* 2>/dev/null || echo "Nenhum item de backlog encontrado"
   ```

2. **Ler ROADMAP.md** e extrair todas as entradas de fase 999.x:
   ```bash
   cat .planning/ROADMAP.md
   ```
   Mostrar cada item do backlog com sua descrição, contexto acumulado (CONTEXT.md, RESEARCH.md) e data de criação.

3. **Apresentar a lista ao usuário** via interação conversacional:
   - Para cada item do backlog, mostrar: número da fase, descrição, artefatos acumulados
   - Opções por item: **Promover** (mover para ativo), **Manter** (deixar no backlog), **Remover** (deletar)

4. **Para itens a PROMOVER:**
   - Encontrar o próximo número de fase sequencial no marco ativo
   - Renomear o diretório de `999.x-slug` para `{novo_num}-slug`:
     ```bash
     NEW_NUM=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase add "${DESCRIPTION}" --raw)
     ```
   - Mover artefatos acumulados para o novo diretório de fase
   - Atualizar ROADMAP.md: mover a entrada da seção `## Backlog` para a lista de fases ativas
   - Remover marcador `(BACKLOG)`
   - Adicionar campo `**Depende de:**` apropriado

5. **Para itens a REMOVER:**
   - Deletar o diretório da fase
   - Remover a entrada da seção `## Backlog` do ROADMAP.md

6. **Commitar alterações:**
   ```bash
   node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: revisar backlog — promovidos N, removidos M" --files .planning/ROADMAP.md
   ```

7. **Relatório resumo:**
   ```
   ## 📋 Revisão do Backlog Concluída

   Promovidos: {lista de itens promovidos com novos números de fase}
   Mantidos: {lista de itens mantidos no backlog}
   Removidos: {lista de itens deletados}
   ```

</process>
</output>
