---
name: gsd-validar-fase
description: "Auditar retroativamente e preencher lacunas de validação Nyquist para uma fase concluída"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-validar-fase` ou descreve uma tarefa correspondente a esta skill.
- Trate todo o texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com o Usuário
Quando o workflow precisar de entrada do usuário, solicite conversacionalmente:
- Apresente opções como lista numerada no texto da resposta
- Peça ao usuário para responder com sua escolha
- Para seleção múltipla, peça números separados por vírgula

## C. Uso de Ferramentas
Use estas ferramentas do Cursor ao executar workflows GSD:
- `Shell` para executar comandos (operações de terminal)
- `StrReplace` para editar arquivos existentes
- `Read`, `Write`, `Glob`, `Grep`, `Task`, `WebSearch`, `WebFetch`, `TodoWrite` conforme necessário

## D. Geração de Subagentes
Quando o workflow precisar gerar um subagente:
- Use `Task(subagent_type="generalPurpose", ...)`
- O parâmetro `model` mapeia para as opções de modelo do Cursor (ex: "fast")
</cursor_skill_adapter>

<objective>
Auditar cobertura de validação Nyquist para uma fase concluída. Três estados:
- (A) VALIDACAO.md existe — auditar e preencher lacunas
- (B) Sem VALIDACAO.md, RESUMO.md existe — reconstruir a partir dos artefatos
- (C) Fase não executada — sair com orientação

Saída: VALIDACAO.md atualizado + arquivos de teste gerados.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/validar-fase.md
</execution_context>

<context>
Fase: {{GSD_ARGS}} — opcional, padrão é a última fase concluída.
</context>

<process>
Execute @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/validar-fase.md.
Preserve todas as portas do workflow.
</process>
</output>
