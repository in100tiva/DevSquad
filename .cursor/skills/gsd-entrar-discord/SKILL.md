---
name: gsd-entrar-discord
description: "Entrar na comunidade Discord do GSD"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-entrar-discord` ou descreve uma tarefa correspondente a esta skill.
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
Exibir o link de convite do Discord para o servidor da comunidade GSD.
</objective>

<output>
# Entre no Discord do GSD

Conecte-se com outros usuários do GSD, obtenha ajuda, compartilhe o que está construindo e fique atualizado.

**Link de convite:** https://discord.gg/gsd

Clique no link ou cole no seu navegador para entrar.
</output>
</output>
