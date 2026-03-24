<purpose>
Executar uma tarefa trivial inline sem overhead de subagente. Sem PLAN.md, sem disparo de Task,
sem pesquisa, sem verificação de plano. Apenas: entender → fazer → commitar → registrar.

Para tarefas como: corrigir um typo, atualizar um valor de configuração, adicionar um import faltando, renomear uma
variável, commitar trabalho não commitado, adicionar uma entrada no .gitignore, incrementar número de versão.

Use /gsd-quick para qualquer coisa que precise de planejamento multi-etapa ou pesquisa.
</purpose>

<process>

<step name="parse_task">
Analisar `{{GSD_ARGS}}` para a descrição da tarefa.

Se vazio, perguntar:
```
Qual é a correção rápida? (uma frase)
```

Armazenar como `$TASK`.
</step>

<step name="scope_check">
**Antes de fazer qualquer coisa, verificar se isso é realmente trivial.**

Uma tarefa é trivial se pode ser concluída em:
- ≤ 3 edições de arquivo
- ≤ 1 minuto de trabalho
- Sem novas dependências ou mudanças de arquitetura
- Sem pesquisa necessária

Se a tarefa parecer não-trivial (refatoração multi-arquivo, nova funcionalidade, precisa de pesquisa),
dizer:

```
Isso parece precisar de planejamento. Use /gsd-quick no lugar:
  /gsd-quick "{descrição da tarefa}"
```

E parar.
</step>

<step name="execute_inline">
Fazer o trabalho diretamente:

1. Ler o(s) arquivo(s) relevante(s)
2. Fazer a(s) mudança(s)
3. Verificar se a mudança funciona (rodar testes existentes se aplicável, ou fazer uma verificação rápida de sanidade)

**Sem PLAN.md.** Apenas faça.
</step>

<step name="commit">
Commitar a mudança atomicamente:

```bash
git add -A
git commit -m "fix: {descrição concisa do que mudou}"
```

Usar formato de commit convencional: `fix:`, `feat:`, `docs:`, `chore:`, `refactor:` conforme apropriado.
</step>

<step name="log_to_state">
Se `.planning/STATE.md` existir, adicionar à tabela "Quick Tasks Completed".
Se a tabela não existir, pular esta etapa silenciosamente.

```bash
# Verificar se STATE.md tem tabela de tarefas rápidas
if grep -q "Quick Tasks Completed" .planning/STATE.md 2>/dev/null; then
  # Adicionar entrada — o workflow cuida do formato
  echo "| $(date +%Y-%m-%d) | fast | $TASK | ✅ |" >> .planning/STATE.md
fi
```
</step>

<step name="done">
Reportar conclusão:

```
✅ Feito: {o que foi alterado}
   Commit: {hash curto}
   Arquivos: {lista de arquivos alterados}
```

Sem sugestões de próximos passos. Sem roteamento de workflow. Apenas feito.
</step>

</process>

<guardrails>
- NUNCA disparar um Task/subagente — isso roda inline
- NUNCA criar arquivos PLAN.md ou SUMMARY.md
- NUNCA executar pesquisa ou verificação de plano
- Se a tarefa levar mais de 3 edições de arquivo, PARAR e redirecionar para /gsd-quick
- Se estiver inseguro sobre como implementar, PARAR e redirecionar para /gsd-quick
</guardrails>

<success_criteria>
- [ ] Tarefa concluída no contexto atual (sem subagentes)
- [ ] Commit git atômico com mensagem convencional
- [ ] STATE.md atualizado se existir
- [ ] Operação total abaixo de 2 minutos de tempo real
</success_criteria>
