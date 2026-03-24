---
name: gsd-atualizar
description: "Atualizar GSD para a versão mais recente com exibição de changelog"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-atualizar` ou descreve uma tarefa correspondente a esta skill.
- Trate todo texto do usuário após a menção da skill como `{{GSD_ARGS}}`.
- Se não houver argumentos, trate `{{GSD_ARGS}}` como vazio.

## B. Interação com Usuário
Quando o workflow precisar de input do usuário, pergunte de forma conversacional:
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
Verificar atualizações do GSD, instalar se disponível e exibir o que mudou.

Direciona para o workflow de atualização que gerencia:
- Detecção de versão (instalação local vs global)
- Verificação de versão no npm
- Busca e exibição do changelog
- Confirmação do usuário com aviso de instalação limpa
- Execução da atualização e limpeza de cache
- Lembrete de reinicialização
</objective>

<execution_context>
@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/atualizar.md
</execution_context>

<process>
**Siga o workflow de atualização** de `@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/atualizar.md`.

O workflow gerencia toda a lógica incluindo:
1. Detecção de versão instalada (local/global)
2. Verificação da versão mais recente via npm
3. Comparação de versões
4. Busca e extração do changelog
5. Exibição de aviso de instalação limpa
6. Confirmação do usuário
7. Execução da atualização
8. Limpeza de cache
</process>
