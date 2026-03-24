---
name: gsd-auditar-marco
description: "Auditar conclusão do marco contra a intenção original antes de arquivar"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-auditar-marco` ou descreve uma tarefa correspondente a esta skill.
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
Verificar se o marco atingiu sua definição de pronto. Verificar cobertura de requisitos, integração entre fases e fluxos ponta a ponta.

**Este comando É o orquestrador.** Lê arquivos VERIFICATION.md existentes (fases já verificadas durante executar-fase), agrega débito técnico e lacunas diferidas, então invoca verificador de integração para conexões entre fases.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/auditar-marco.md
</execution_context>

<context>
Versão: {{GSD_ARGS}} (opcional — padrão é o marco atual)

Arquivos centrais de planejamento são resolvidos dentro do workflow (`init milestone-op`) e carregados apenas conforme necessário.

**Trabalho Concluído:**
Glob: .planning/phases/*/*-SUMMARY.md
Glob: .planning/phases/*/*-VERIFICATION.md
</context>

<process>
Execute o workflow auditar-marco de @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/auditar-marco.md do início ao fim.
Preserve todas as portas do workflow (determinação de escopo, leitura de verificação, verificação de integração, cobertura de requisitos, roteamento).
</process>
</output>
