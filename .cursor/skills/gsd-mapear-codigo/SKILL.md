---
name: gsd-mapear-codigo
description: "Analisar codebase com agentes mapeadores paralelos para produzir documentos em .planning/codebase/"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-mapear-codigo` ou descreve uma tarefa correspondente a esta skill.
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
Analisar codebase existente usando agentes gsd-mapeador-codigo paralelos para produzir documentos estruturados do codebase.

Cada agente mapeador explora uma área de foco e **escreve documentos diretamente** em `.planning/codebase/`. O orquestrador apenas recebe confirmações, mantendo o uso de contexto mínimo.

Saída: pasta .planning/codebase/ com 7 documentos estruturados sobre o estado do codebase.
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/mapear-codigo.md
</execution_context>

<context>
Área de foco: {{GSD_ARGS}} (opcional - se fornecido, direciona agentes para focar em subsistema específico)

**Carregar estado do projeto se existir:**
Verificar .planning/STATE.md - carrega contexto se projeto já inicializado

**Este comando pode executar:**
- Antes de /gsd-novo-projeto (codebases brownfield) - cria mapa do codebase primeiro
- Após /gsd-novo-projeto (codebases greenfield) - atualiza mapa do codebase conforme código evolui
- A qualquer momento para atualizar entendimento do codebase
</context>

<when_to_use>
**Use mapear-codigo para:**
- Projetos brownfield antes da inicialização (entender código existente primeiro)
- Atualizar mapa do codebase após mudanças significativas
- Onboarding em um codebase desconhecido
- Antes de refatorações grandes (entender estado atual)
- Quando STATE.md referencia informações desatualizadas do codebase

**Pule mapear-codigo para:**
- Projetos greenfield sem código ainda (nada para mapear)
- Codebases triviais (<5 arquivos)
</when_to_use>

<process>
1. Verificar se .planning/codebase/ já existe (oferecer atualizar ou pular)
2. Criar estrutura do diretório .planning/codebase/
3. Gerar 4 agentes gsd-mapeador-codigo paralelos:
   - Agente 1: foco tech → escreve STACK.md, INTEGRATIONS.md
   - Agente 2: foco arch → escreve ARCHITECTURE.md, STRUCTURE.md
   - Agente 3: foco quality → escreve CONVENTIONS.md, TESTING.md
   - Agente 4: foco concerns → escreve CONCERNS.md
4. Aguardar agentes completarem, coletar confirmações (NÃO conteúdos dos documentos)
5. Verificar que todos os 7 documentos existem com contagem de linhas
6. Commitar mapa do codebase
7. Oferecer próximos passos (tipicamente: /gsd-novo-projeto ou /gsd-planejar-fase)
</process>

<success_criteria>
- [ ] Diretório .planning/codebase/ criado
- [ ] Todos os 7 documentos do codebase escritos pelos agentes mapeadores
- [ ] Documentos seguem estrutura do template
- [ ] Agentes paralelos completaram sem erros
- [ ] Usuário sabe os próximos passos
</success_criteria>
</output>
