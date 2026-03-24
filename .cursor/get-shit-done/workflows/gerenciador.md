<purpose>

Centro de comando interativo para gerenciar um marco a partir de um único terminal. Mostra um painel de todas as fases com status visual, despacha discussão inline e planejar/executar como agentes em background, e retorna ao painel após cada ação. Habilita trabalho paralelo de fases a partir de um terminal.

</purpose>

<required_reading>

Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.

</required_reading>

<process>

<step name="initialize" priority="first">

## 1. Inicializar

Inicialização via manager init:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init manager)
```

Extrair do JSON: `milestone_version`, `milestone_name`, `phase_count`, `completed_count`, `in_progress_count`, `phases`, `recommended_actions`, `all_complete`, `waiting_signal`.

**Se erro:** Exibir a mensagem de erro e sair.

Exibir banner de inicialização:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► GERENCIADOR
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 {milestone_version} — {milestone_name}
 {phase_count} fases · {completed_count} concluídas

 ✓ Discutir → inline    ◆ Planejar/Executar → background
 Painel atualiza automaticamente quando trabalho em background está ativo.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Prosseguir para etapa do painel.

</step>

<step name="dashboard">

## 2. Painel (Ponto de Atualização)

**Toda vez que esta etapa for alcançada**, re-ler estado do disco para captar mudanças de agentes em background:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init manager)
```

Extrair o JSON completo. Construir exibição do painel.

Construir painel a partir do JSON. Símbolos: `✓` feito, `◆` ativo, `○` pendente, `·` na fila. Barra de progresso: 20 caracteres `█░`.

**Mapeamento de status** (disk_status → D P E Status):

- `complete` → `✓ ✓ ✓` `✓ Concluída`
- `partial` → `✓ ✓ ◆` `◆ Executando...`
- `planned` → `✓ ✓ ○` `○ Pronta para executar`
- `discussed` → `✓ ○ ·` `○ Pronta para planejar`
- `researched` → `◆ · ·` `○ Pronta para planejar`
- `empty`/`no_directory` + `is_next_to_discuss` → `○ · ·` `○ Pronta para discutir`
- `empty`/`no_directory` caso contrário → `· · ·` `· Em seguida`
- Se `is_active`, substituir ícone de status com `◆` e anexar `(ativo)`

Se houver fases `is_active`, mostrar: `◆ Background: {ação} Fase {N}, ...` acima da grade.

Usar `display_name` (não `name`) para a coluna Fase — está pré-truncado para 20 caracteres com `…` se cortado. Preencher todos os nomes de fase com a mesma largura para alinhamento.

Usar `deps_display` do JSON de init para a coluna Deps — mostra de quais fases esta fase depende (ex: `1,3`) ou `—` para nenhuma.

Exemplo de saída:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► PAINEL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ████████████░░░░░░░░ 60%  (3/5 fases)
 ◆ Background: Planejando Fase 4
 | # | Fase                 | Deps | D | P | E | Status              |
 |---|----------------------|------|---|---|---|---------------------|
 | 1 | Fundação             | —    | ✓ | ✓ | ✓ | ✓ Concluída         |
 | 2 | Camada API           | 1    | ✓ | ✓ | ◆ | ◆ Executando (ativo)|
 | 3 | Sistema de Auth      | 1    | ✓ | ✓ | ○ | ○ Pronta p/ executar|
 | 4 | Painel UI & Conf…    | 1,2  | ✓ | ◆ | · | ◆ Planejando (ativo)|
 | 5 | Notificações         | —    | ○ | · | · | ○ Pronta p/ discutir|
 | 6 | Polimento & Email…   | 1-5  | · | · | · | · Em seguida        |
```

**Seção de recomendações:**

Se `all_complete` for true:

```
╔══════════════════════════════════════════════════════════════╗
║  MARCO CONCLUÍDO                                              ║
╚══════════════════════════════════════════════════════════════╝

Todas as {phase_count} fases concluídas. Prontas para etapas finais:
  → /gsd-verify-work — executar testes de aceite
  → /gsd-complete-milestone — arquivar e finalizar
```

Perguntar ao usuário via conversational prompting:
- **question:** "Todas as fases concluídas. O que fazer agora?"
- **options:** "Verificar trabalho" / "Concluir marco" / "Sair do gerenciador"

Tratar respostas:
- "Verificar trabalho": `Skill(skill="gsd-verify-work")` depois voltar ao painel.
- "Concluir marco": `Skill(skill="gsd-complete-milestone")` depois sair.
- "Sair do gerenciador": Ir para etapa de saída.

**Se NÃO all_complete**, construir opções compostas de `recommended_actions`:

**Lógica de opção composta:** Agrupar ações de background (planejar/executar) juntas, e combiná-las com a ação inline única (discutir) quando existir. O objetivo é apresentar o menor número de opções possível — uma opção pode despachar múltiplos agentes de background mais uma ação inline.

**Construindo opções:**

1. Coletar todas as ações de background (recomendações de executar e planejar) — pode haver múltiplas de cada.
2. Coletar a ação inline (recomendação de discutir, se houver — no máximo uma já que discutir é sequencial).
3. Construir opções compostas:

   **Se houver QUAISQUER ações recomendadas (background, inline, ou ambas):**
   Criar UMA opção primária "Continuar" que despacha TODAS juntas:
   - Label: `"Continuar"` — sempre esta palavra exata
   - Abaixo da label, listar cada ação que acontecerá. Enumerar TODAS as ações recomendadas — não limitar ou truncar:
     ```
     Continuar:
       → Executar Fase 32 (background)
       → Planejar Fase 34 (background)
       → Discutir Fase 35 (inline)
     ```
   - Isso despacha todos os agentes de background primeiro, depois executa a discussão inline (se houver).
   - Se não houver discussão inline, o painel atualiza após disparar agentes de background.

   **Importante:** A opção Continuar deve incluir CADA ação de `recommended_actions` — não apenas 2. Se houver 3 ações, listar 3. Se houver 5, listar 5.

4. Sempre adicionar:
   - `"Atualizar painel"`
   - `"Sair do gerenciador"`

Exibir recomendações compactamente:

```
───────────────────────────────────────────────────────────────
▶ Próximos Passos
───────────────────────────────────────────────────────────────

Continuar:
  → Executar Fase 32 (background)
  → Planejar Fase 34 (background)
  → Discutir Fase 35 (inline)
```

**Auto-atualização:** Se agentes de background estão rodando (`is_active` é true para qualquer fase), definir ciclo de auto-atualização de 60 segundos. Após apresentar o menu de ação, se nenhuma entrada do usuário for recebida em 60 segundos, atualizar o painel automaticamente. Este intervalo é configurável via `manager_refresh_interval` na config GSD (padrão: 60 segundos, definir como 0 para desabilitar).

Apresentar via conversational prompting:
- **question:** "O que você gostaria de fazer?"
- **options:** (opções compostas construídas acima + atualizar + sair, conversational prompting auto-adiciona "Outro")

**Em "Outro" (texto livre):** Analisar intenção — se mencionar um número de fase e ação, despachar apropriadamente. Se não for claro, exibir ações disponíveis e voltar ao action_menu.

Prosseguir para etapa handle_action com a ação selecionada.

</step>

<step name="handle_action">

## 4. Tratar Ação

### Atualizar Painel

Voltar para etapa do painel.

### Sair do Gerenciador

Ir para etapa de saída.

### Ação Composta (background + inline)

Quando o usuário seleciona uma opção composta:

1. **Disparar todos os agentes de background primeiro** (planejar/executar) — despachá-los em paralelo usando os tratadores Planejar Fase N / Executar Fase N abaixo.
2. **Depois executar a discussão inline:**

```
Skill(skill="gsd-discuss-phase", args="{PHASE_NUM}")
```

Após discussão concluir, voltar para etapa do painel (agentes de background continuam rodando).

### Discutir Fase N

Discussão é interativa — precisa de entrada do usuário. Executar inline:

```
Skill(skill="gsd-discuss-phase", args="{PHASE_NUM}")
```

Após discussão concluir, voltar para etapa do painel.

### Planejar Fase N

Planejamento roda autonomamente. Disparar agente de background:

```
Task(
  description="Planejar fase {N}: {phase_name}",
  run_in_background=true,
  prompt="Você está executando o workflow GSD plan-phase para fase {N} do projeto.

Diretório de trabalho: {cwd}
Fase: {N} — {phase_name}
Objetivo: {goal}

Passos:
1. Ler o workflow plan-phase: cat D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/planejar-fase.md
2. Executar: node \"D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs\" init plan-phase {N}
3. Seguir os passos do workflow para produzir arquivos PLAN.md para esta fase.
4. Se pesquisa estiver habilitada na config, executar o passo de pesquisa primeiro.
5. Disparar subagente gsd-planejador via Task() para criar os planos.
6. Se verificador de plano estiver habilitado, disparar subagente gsd-verificador-plano para verificar.
7. Commitar arquivos de plano quando concluído.

Importante: Você está rodando em background. NÃO use conversational prompting — tome decisões autônomas baseadas no contexto do projeto. Se encontrar um bloqueio, escreva no STATE.md como bloqueio e pare. NÃO contorne silenciosamente erros de permissão ou acesso a arquivos — deixe falhar para que o gerenciador possa exibir com dicas de resolução."
)
```

Exibir:

```
◆ Disparando planejador para Fase {N}: {phase_name}...
```

Voltar para etapa do painel.

### Executar Fase N

Execução roda autonomamente. Disparar agente de background:

```
Task(
  description="Executar fase {N}: {phase_name}",
  run_in_background=true,
  prompt="Você está executando o workflow GSD execute-phase para fase {N} do projeto.

Diretório de trabalho: {cwd}
Fase: {N} — {phase_name}
Objetivo: {goal}

Passos:
1. Ler o workflow execute-phase: cat D:/projetos/Estudo/devsquad/.cursor/get-shit-done/workflows/executar-fase.md
2. Executar: node \"D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs\" init execute-phase {N}
3. Seguir os passos do workflow: descobrir planos, analisar dependências, agrupar em ondas.
4. Para cada onda, disparar subagentes gsd-executor via Task() para executar planos em paralelo.
5. Após todas as ondas concluírem, disparar subagente gsd-verificador se verificador estiver habilitado.
6. Atualizar ROADMAP.md e STATE.md com progresso.
7. Commitar todas as mudanças.

Importante: Você está rodando em background. NÃO use conversational prompting — tome decisões autônomas. Use --no-verify em commits git. Se encontrar erro de permissão, bloqueio de arquivo, ou qualquer problema de acesso, NÃO contorne — deixe falhar e escreva o erro no STATE.md como bloqueio para que o gerenciador possa exibir com orientação de resolução."
)
```

Exibir:

```
◆ Disparando executor para Fase {N}: {phase_name}...
```

Voltar para etapa do painel.

</step>

<step name="background_completion">

## 5. Conclusão de Agente de Background

Quando notificado que um agente de background concluiu:

1. Ler a mensagem de resultado do agente.
2. Exibir uma breve notificação:

```
✓ {description}
  {resumo breve do resultado do agente}
```

3. Voltar para etapa do painel.

**Se o agente reportou um erro ou bloqueio:**

Classificar o erro:

**Erro de permissão / acesso a ferramenta** (ex: ferramenta não permitida, permissão negada, restrição de sandbox):
- Analisar o erro para identificar qual ferramenta ou comando foi bloqueado.
- Exibir o erro claramente, depois oferecer correção:
  - **question:** "Fase {N} falhou — permissão negada para `{tool_or_command}`. Quer que eu adicione no settings.local.json para que seja permitido?"
  - **options:** "Adicionar permissão e tentar novamente" / "Executar esta fase inline ao invés" / "Pular e continuar"
  - "Adicionar permissão e tentar novamente": Usar `Skill(skill="update-config")` para adicionar a permissão no `settings.local.json`, depois re-disparar o agente de background. Voltar ao painel.
  - "Executar esta fase inline ao invés": Despachar a mesma ação (planejar/executar) inline via `Skill()` ao invés de um Task em background. Voltar ao painel depois.
  - "Pular e continuar": Voltar ao painel (fase permanece no estado atual).

**Outros erros** (bloqueio git, conflito de arquivo, erro de lógica, etc.):
- Exibir o erro, depois oferecer opções via conversational prompting:
  - **question:** "Agente de background para Fase {N} encontrou um problema: {error}. O que fazer?"
  - **options:** "Tentar novamente" / "Executar inline ao invés" / "Pular e continuar" / "Ver detalhes"
  - "Tentar novamente": Re-disparar o mesmo agente de background. Voltar ao painel.
  - "Executar inline ao invés": Despachar a ação inline via `Skill()`. Voltar ao painel depois.
  - "Pular e continuar": Voltar ao painel (fase permanece no estado atual).
  - "Ver detalhes": Ler seção de bloqueios do STATE.md, exibir, depois re-apresentar opções.

</step>

<step name="exit">

## 6. Saída

Exibir status final com barra de progresso:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► FIM DA SESSÃO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 {milestone_version} — {milestone_name}
 {PROGRESS_BAR} {progress_pct}%  ({completed_count}/{phase_count} fases)

 Retome a qualquer momento: /gsd-manager
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Nota:** Qualquer agente de background ainda rodando continuará até concluir. Seus resultados serão visíveis na próxima invocação de `/gsd-manager` ou `/gsd-progress`.

</step>

</process>

<success_criteria>
- [ ] Painel exibe todas as fases com indicadores corretos de status (colunas D/P/E/V)
- [ ] Barra de progresso mostra porcentagem precisa de conclusão
- [ ] Resolução de dependências: fases bloqueadas mostram quais deps estão faltando
- [ ] Recomendações priorizam: executar > planejar > discutir
- [ ] Fases de discussão rodam inline via Skill() — perguntas interativas funcionam
- [ ] Fases de planejamento disparam agentes Task em background — retorna ao painel imediatamente
- [ ] Fases de execução disparam agentes Task em background — retorna ao painel imediatamente
- [ ] Atualizações do painel captam mudanças de agentes em background via estado no disco
- [ ] Conclusão de agente de background dispara notificação e atualização do painel
- [ ] Erros de agente de background apresentam opções de tentar novamente/pular
- [ ] Estado all-complete oferece verify-work e complete-milestone
- [ ] Saída mostra status final com instruções de retomada
- [ ] Entrada de texto livre "Outro" é analisada para número de fase e ação
- [ ] Loop do gerenciador continua até o usuário sair ou marco ser concluído
</success_criteria>
