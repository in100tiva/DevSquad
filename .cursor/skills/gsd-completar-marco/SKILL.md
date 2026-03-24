---
name: gsd-completar-marco
description: "Arquivar marco concluído e preparar para a próxima versão"
---

<cursor_skill_adapter>
## A. Invocação da Skill
- Esta skill é invocada quando o usuário menciona `gsd-completar-marco` ou descreve uma tarefa correspondente a esta skill.
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
Marcar marco {{version}} como concluído, arquivar em milestones/ e atualizar ROADMAP.md e REQUIREMENTS.md.

Propósito: Criar registro histórico da versão entregue, arquivar artefatos do marco (roteiro + requisitos), e preparar para o próximo marco.
Saída: Marco arquivado (roteiro + requisitos), PROJECT.md evoluído, tag git criada.
</objective>

<execution_context>
**Carregue estes arquivos AGORA (antes de prosseguir):**

- @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/completar-marco.md (workflow principal)
- @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/templates/arquivo-marco.md (template de arquivo)
  </execution_context>

<context>
**Arquivos do projeto:**
- `.planning/ROADMAP.md`
- `.planning/REQUIREMENTS.md`
- `.planning/STATE.md`
- `.planning/PROJECT.md`

**Entrada do usuário:**

- Versão: {{version}} (ex: "1.0", "1.1", "2.0")
  </context>

<process>

**Siga o workflow completar-marco.md:**

0. **Verificar auditoria:**

   - Procure por `.planning/v{{version}}-MILESTONE-AUDIT.md`
   - Se ausente ou desatualizado: recomendar `/gsd-auditar-marco` primeiro
   - Se status da auditoria é `gaps_found`: recomendar `/gsd-planejar-lacunas-marco` primeiro
   - Se status da auditoria é `passed`: prosseguir para o passo 1

   ```markdown
   ## Verificação Pré-voo

   {Se não há v{{version}}-MILESTONE-AUDIT.md:}
   ⚠ Nenhuma auditoria de marco encontrada. Execute `/gsd-auditar-marco` primeiro para verificar
   cobertura de requisitos, integração entre fases e fluxos E2E.

   {Se a auditoria tem lacunas:}
   ⚠ Auditoria de marco encontrou lacunas. Execute `/gsd-planejar-lacunas-marco` para criar
   fases que fechem as lacunas, ou prossiga mesmo assim para aceitar como débito técnico.

   {Se a auditoria passou:}
   ✓ Auditoria de marco aprovada. Prosseguindo com a conclusão.
   ```

1. **Verificar prontidão:**

   - Verificar se todas as fases do marco têm planos concluídos (SUMMARY.md existe)
   - Apresentar escopo e estatísticas do marco
   - Aguardar confirmação

2. **Coletar estatísticas:**

   - Contar fases, planos, tarefas
   - Calcular intervalo git, alterações de arquivos, LOC
   - Extrair linha do tempo do git log
   - Apresentar resumo, confirmar

3. **Extrair conquistas:**

   - Ler todos os arquivos SUMMARY.md das fases no intervalo do marco
   - Extrair 4-6 conquistas principais
   - Apresentar para aprovação

4. **Arquivar marco:**

   - Criar `.planning/milestones/v{{version}}-ROADMAP.md`
   - Extrair detalhes completos das fases do ROADMAP.md
   - Preencher template arquivo-marco.md
   - Atualizar ROADMAP.md para resumo de uma linha com link

5. **Arquivar requisitos:**

   - Criar `.planning/milestones/v{{version}}-REQUIREMENTS.md`
   - Marcar todos os requisitos v1 como completos (checkboxes marcados)
   - Anotar resultados dos requisitos (validado, ajustado, descartado)
   - Deletar `.planning/REQUIREMENTS.md` (novo criado para próximo marco)

6. **Atualizar PROJECT.md:**

   - Adicionar seção "Estado Atual" com versão entregue
   - Adicionar seção "Objetivos do Próximo Marco"
   - Arquivar conteúdo anterior em `<details>` (se v1.1+)

7. **Commit e tag:**

   - Stage: MILESTONES.md, PROJECT.md, ROADMAP.md, STATE.md, arquivos de arquivo
   - Commit: `chore: archive v{{version}} milestone`
   - Tag: `git tag -a v{{version}} -m "[resumo do marco]"`
   - Perguntar sobre push da tag

8. **Oferecer próximos passos:**
   - `/gsd-novo-marco` — iniciar próximo marco (questionamento → pesquisa → requisitos → roteiro)

</process>

<success_criteria>

- Marco arquivado em `.planning/milestones/v{{version}}-ROADMAP.md`
- Requisitos arquivados em `.planning/milestones/v{{version}}-REQUIREMENTS.md`
- `.planning/REQUIREMENTS.md` deletado (novo para próximo marco)
- ROADMAP.md condensado para entrada de uma linha
- PROJECT.md atualizado com estado atual
- Tag git v{{version}} criada
- Commit bem-sucedido
- Usuário sabe dos próximos passos (incluindo necessidade de novos requisitos)
  </success_criteria>

<critical_rules>

- **Carregar workflow primeiro:** Ler completar-marco.md antes de executar
- **Verificar conclusão:** Todas as fases devem ter arquivos SUMMARY.md
- **Confirmação do usuário:** Aguardar aprovação nos portões de verificação
- **Arquivar antes de deletar:** Sempre criar arquivos de arquivo antes de atualizar/deletar originais
- **Resumo de uma linha:** Marco condensado no ROADMAP.md deve ser uma linha única com link
- **Eficiência de contexto:** Arquivo mantém ROADMAP.md e REQUIREMENTS.md com tamanho constante por marco
- **Novos requisitos:** Próximo marco começa com `/gsd-novo-marco` que inclui definição de requisitos
  </critical_rules>
</output>
