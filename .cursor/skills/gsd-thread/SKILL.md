---
name: gsd-thread
description: "Gerenciar threads de contexto persistente para trabalho entre sessões"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-thread` ou descreve uma tarefa correspondente a esta skill.
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
Criar, listar ou retomar threads de contexto persistente. Threads são armazenamentos
de conhecimento leves entre sessões para trabalho que abrange múltiplas sessões mas
não pertence a nenhuma fase específica.
</objective>

<process>

**Analise {{GSD_ARGS}} para determinar o modo:**

<mode_list>
**Se sem argumentos ou {{GSD_ARGS}} está vazio:**

Listar todas as threads:
```bash
ls .planning/threads/*.md 2>/dev/null
```

Para cada thread, leia as primeiras linhas para mostrar título e status:
```
## Threads Ativas

| Thread | Status | Última Atualização |
|--------|--------|--------------------|
| fix-deploy-key-auth | ABERTA | 2026-03-15 |
| pasta-tcp-timeout | RESOLVIDA | 2026-03-12 |
| perf-investigation | EM PROGRESSO | 2026-03-17 |
```

Se nenhuma thread existir, mostrar:
```
Nenhuma thread encontrada. Crie uma com: /gsd-thread <descrição>
```
</mode_list>

<mode_resume>
**Se {{GSD_ARGS}} corresponde a um nome de thread existente (arquivo existe):**

Retomar a thread — carregar seu contexto na sessão atual:
```bash
cat ".planning/threads/${THREAD_NAME}.md"
```

Exiba o conteúdo da thread e pergunte no que o usuário quer trabalhar em seguida.
Atualize o status da thread para `EM PROGRESSO` se estava `ABERTA`.
</mode_resume>

<mode_create>
**Se {{GSD_ARGS}} é uma nova descrição (nenhum arquivo de thread correspondente):**

Criar uma nova thread:

1. Gerar slug a partir da descrição:
   ```bash
   SLUG=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" generate-slug "{{GSD_ARGS}}")
   ```

2. Criar o diretório de threads se necessário:
   ```bash
   mkdir -p .planning/threads
   ```

3. Escrever o arquivo da thread:
   ```bash
   cat > ".planning/threads/${SLUG}.md" << 'EOF'
   # Thread: {descrição}

   ## Status: ABERTA

   ## Objetivo

   {descrição}

   ## Contexto

   *Criada a partir da conversa em {data de hoje}.*

   ## Referências

   - *(adicione links, caminhos de arquivos ou números de issues)*

   ## Próximos Passos

   - *(o que a próxima sessão deve fazer primeiro)*
   EOF
   ```

4. Se houver contexto relevante na conversa atual (trechos de código,
   mensagens de erro, resultados de investigação), extraia e adicione à
   seção Contexto.

5. Commit:
   ```bash
   node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: criar thread — ${ARGUMENTS}" --files ".planning/threads/${SLUG}.md"
   ```

6. Relatório:
   ```
   ## 🧵 Thread Criada

   Thread: {slug}
   Arquivo: .planning/threads/{slug}.md

   Retome a qualquer momento com: /gsd-thread {slug}
   ```
</mode_create>

</process>

<notes>
- Threads NÃO são vinculadas a fases — existem independentemente do roteiro
- Mais leve que /gsd-pausar-trabalho — sem estado de fase, sem contexto de plano
- O valor está em Contexto e Próximos Passos — uma sessão cold-start pode retomar imediatamente
- Threads podem ser promovidas a fases ou itens de backlog quando amadurecerem:
  /gsd-adicionar-fase ou /gsd-adicionar-backlog com contexto da thread
- Arquivos de thread ficam em .planning/threads/ — sem colisão com fases ou outras estruturas GSD
</notes>
</output>
