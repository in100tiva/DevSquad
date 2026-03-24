---
name: gsd-adicionar-backlog
description: "Adicionar uma ideia ao estacionamento de backlog (numeração 999.x)"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-adicionar-backlog` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de entrada do usuário, pergunte de forma conversacional:
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
Adicionar um item de backlog ao roteiro usando numeração 999.x. Itens de backlog são
ideias não sequenciadas que ainda não estão prontas para planejamento ativo — vivem fora
da sequência normal de fases e acumulam contexto ao longo do tempo.
</objective>

<process>

1. **Ler ROADMAP.md** para encontrar entradas de backlog existentes:
   ```bash
   cat .planning/ROADMAP.md
   ```

2. **Encontrar próximo número de backlog:**
   ```bash
   NEXT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" phase next-decimal 999 --raw)
   ```
   Se não existirem fases 999.x, comece em 999.1.

3. **Criar o diretório da fase:**
   ```bash
   SLUG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" generate-slug "{{GSD_ARGS}}")
   mkdir -p ".planning/phases/${NEXT}-${SLUG}"
   touch ".planning/phases/${NEXT}-${SLUG}/.gitkeep"
   ```

4. **Adicionar ao ROADMAP.md** sob uma seção `## Backlog`. Se a seção não existir, crie-a no final:

   ```markdown
   ## Backlog

   ### Fase {NEXT}: {descrição} (BACKLOG)

   **Objetivo:** [Capturado para planejamento futuro]
   **Requisitos:** A definir
   **Planos:** 0 planos

   Planos:
   - [ ] A definir (promova com /gsd-revisar-backlog quando pronto)
   ```

5. **Commit:**
   ```bash
   node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: add backlog item ${NEXT} — ${ARGUMENTS}" --files .planning/ROADMAP.md ".planning/phases/${NEXT}-${SLUG}/.gitkeep"
   ```

6. **Relatório:**
   ```
   ## 📋 Item de Backlog Adicionado

   Fase {NEXT}: {descrição}
   Diretório: .planning/phases/{NEXT}-{slug}/

   Este item está no estacionamento de backlog.
   Use /gsd-discutir-fase {NEXT} para explorá-lo mais.
   Use /gsd-revisar-backlog para promover itens ao marco ativo.
   ```

</process>

<notes>
- Numeração 999.x mantém itens de backlog fora da sequência ativa de fases
- Diretórios de fase são criados imediatamente, então /gsd-discutir-fase e /gsd-planejar-fase funcionam neles
- Sem campo `Depende de:` — itens de backlog são não sequenciados por definição
- Numeração esparsa é aceitável (999.1, 999.3) — sempre usa next-decimal
</notes>
</output>
