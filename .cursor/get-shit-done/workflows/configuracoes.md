<purpose>
Configuração interativa dos agentes de workflow GSD (pesquisa, verificação de plano, verificador) e seleção de perfil de modelo via prompt de múltiplas perguntas. Atualiza .planning/config.json com as preferências do usuário. Opcionalmente salva configurações como padrões globais (~/.gsd/defaults.json) para projetos futuros.
</purpose>

<required_reading>
Ler todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="ensure_and_load_config">
Garantir que config existe e carregar estado atual:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" config-ensure-section
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state load)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Cria `.planning/config.json` com padrões se ausente e carrega valores atuais da config.
</step>

<step name="read_current">
```bash
cat .planning/config.json
```

Analisar valores atuais (padrão `true` se não presente):
- `workflow.research` — disparar pesquisador durante plan-phase
- `workflow.plan_check` — disparar verificador de plano durante plan-phase
- `workflow.verifier` — disparar verificador durante execute-phase
- `workflow.nyquist_validation` — pesquisa de arquitetura de validação durante plan-phase (padrão: true se ausente)
- `workflow.ui_phase` — gerar contratos de design UI-SPEC.md para fases de frontend (padrão: true se ausente)
- `workflow.ui_safety_gate` — solicitar execução de /gsd-ui-phase antes de planejar fases de frontend (padrão: true se ausente)
- `model_profile` — qual modelo cada agente usa (padrão: `balanced`)
- `git.branching_strategy` — abordagem de branching (padrão: `"none"`)
</step>

<step name="present_settings">
Usar conversational prompting com valores atuais pré-selecionados:

```
conversational prompting([
  {
    question: "Qual perfil de modelo para agentes?",
    header: "Modelo",
    multiSelect: false,
    options: [
      { label: "Quality", description: "Opus em todo lugar exceto verificação (custo mais alto)" },
      { label: "Balanced (Recomendado)", description: "Opus para planejamento, Sonnet para pesquisa/execução/verificação" },
      { label: "Budget", description: "Sonnet para escrita, Haiku para pesquisa/verificação (custo mais baixo)" },
      { label: "Inherit", description: "Usar modelo da sessão atual para todos os agentes (melhor para OpenRouter, modelos locais, ou troca de modelo em runtime)" }
    ]
  },
  {
    question: "Disparar Pesquisador de Plano? (pesquisa domínio antes do planejamento)",
    header: "Pesquisa",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Pesquisar objetivos da fase antes de planejar" },
      { label: "Não", description: "Pular pesquisa, planejar diretamente" }
    ]
  },
  {
    question: "Disparar Verificador de Plano? (verifica planos antes da execução)",
    header: "Verificação de Plano",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Verificar se planos atendem objetivos da fase" },
      { label: "Não", description: "Pular verificação de plano" }
    ]
  },
  {
    question: "Disparar Verificador de Execução? (verifica conclusão da fase)",
    header: "Verificador",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Verificar must-haves após execução" },
      { label: "Não", description: "Pular verificação pós-execução" }
    ]
  },
  {
    question: "Pipeline de auto-avanço? (discutir → planejar → executar automaticamente)",
    header: "Auto",
    multiSelect: false,
    options: [
      { label: "Não (Recomendado)", description: "/clear manual + colar entre estágios" },
      { label: "Sim", description: "Encadear estágios via subagentes Task() (mesmo isolamento)" }
    ]
  },
  {
    question: "Habilitar Validação Nyquist? (pesquisa cobertura de testes durante planejamento)",
    header: "Nyquist",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Pesquisar cobertura automatizada de testes durante plan-phase. Adiciona requisitos de validação aos planos. Bloqueia aprovação se tarefas não tiverem verify automatizado." },
      { label: "Não", description: "Pular pesquisa de validação. Bom para prototipagem rápida ou fases sem testes." }
    ]
  },
  // Nota: Validação Nyquist depende da saída de pesquisa. Se pesquisa estiver desabilitada,
  // plan-phase automaticamente pula passos Nyquist (sem RESEARCH.md para extrair).
  {
    question: "Habilitar Fase UI? (gera contratos de design UI-SPEC.md para fases de frontend)",
    header: "Fase UI",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Gerar contratos de design UI antes de planejar fases de frontend. Trava espaçamento, tipografia, cor e copywriting." },
      { label: "Não", description: "Pular geração de UI-SPEC. Bom para projetos backend-only ou fases de API." }
    ]
  },
  {
    question: "Habilitar Portão de Segurança UI? (solicita executar /gsd-ui-phase antes de planejar fases de frontend)",
    header: "Portão UI",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "plan-phase pede para executar /gsd-ui-phase primeiro quando indicadores de frontend são detectados." },
      { label: "Não", description: "Sem solicitação — plan-phase prossegue sem verificação de UI-SPEC." }
    ]
  },
  {
    question: "Estratégia de branching git?",
    header: "Branching",
    multiSelect: false,
    options: [
      { label: "Nenhuma (Recomendado)", description: "Commitar diretamente na branch atual" },
      { label: "Por Fase", description: "Criar branch para cada fase (gsd/phase-{N}-{name})" },
      { label: "Por Marco", description: "Criar branch para todo o marco (gsd/{version}-{name})" }
    ]
  },
  {
    question: "Habilitar avisos de janela de contexto? (injeta mensagens consultivas quando contexto está enchendo)",
    header: "Avisos de Ctx",
    multiSelect: false,
    options: [
      { label: "Sim (Recomendado)", description: "Avisar quando uso de contexto exceder 65%. Ajuda a evitar perda de trabalho." },
      { label: "Não", description: "Desabilitar avisos. Permite o Claude alcançar auto-compact naturalmente. Bom para execuções longas desatendidas." }
    ]
  },
  {
    question: "Pesquisar melhores práticas antes de fazer perguntas? (busca web durante new-project e discuss-phase)",
    header: "Pesq. Perguntas",
    multiSelect: false,
    options: [
      { label: "Não (Recomendado)", description: "Fazer perguntas diretamente. Mais rápido, usa menos tokens." },
      { label: "Sim", description: "Buscar web por melhores práticas antes de cada grupo de perguntas. Perguntas mais informadas mas usa mais tokens." }
    ]
  },
  {
    question: "Pular discuss-phase no modo autônomo? (usar objetivos de fase do ROADMAP como spec)",
    header: "Pular Discussão",
    multiSelect: false,
    options: [
      { label: "Não (Recomendado)", description: "Executar discussão inteligente antes de cada fase — levanta áreas cinzentas e captura decisões." },
      { label: "Sim", description: "Pular discuss no /gsd-autonomous — encadear direto para planejar. Melhor para trabalho backend/pipeline onde descrições de fase são a spec." }
    ]
  }
])
```
</step>

<step name="update_config">
Mesclar novas configurações no config.json existente:

```json
{
  ...existing_config,
  "model_profile": "quality" | "balanced" | "budget" | "inherit",
  "workflow": {
    "research": true/false,
    "plan_check": true/false,
    "verifier": true/false,
    "auto_advance": true/false,
    "nyquist_validation": true/false,
    "ui_phase": true/false,
    "ui_safety_gate": true/false,
    "text_mode": true/false,
    "research_before_questions": true/false,
    "discuss_mode": "discuss" | "assumptions",
    "skip_discuss": true/false
  },
  "git": {
    "branching_strategy": "none" | "phase" | "milestone",
    "quick_branch_template": <string|null>
  },
  "hooks": {
    "context_warnings": true/false,
    "workflow_guard": true/false
  }
}
```

Escrever config atualizada em `.planning/config.json`.
</step>

<step name="save_as_defaults">
Perguntar se deseja salvar estas configurações como padrões globais para projetos futuros:

```
conversational prompting([
  {
    question: "Salvar estas como configurações padrão para todos os novos projetos?",
    header: "Padrões",
    multiSelect: false,
    options: [
      { label: "Sim", description: "Novos projetos iniciam com estas configurações (salvo em ~/.gsd/defaults.json)" },
      { label: "Não", description: "Aplicar apenas a este projeto" }
    ]
  }
])
```

Se "Sim": escrever o mesmo objeto de config (menos campos específicos do projeto como `brave_search`) em `~/.gsd/defaults.json`:

```bash
mkdir -p ~/.gsd
```

Escrever `~/.gsd/defaults.json` com:
```json
{
  "mode": <atual>,
  "granularity": <atual>,
  "model_profile": <atual>,
  "commit_docs": <atual>,
  "parallelization": <atual>,
  "branching_strategy": <atual>,
  "quick_branch_template": <atual>,
  "workflow": {
    "research": <atual>,
    "plan_check": <atual>,
    "verifier": <atual>,
    "auto_advance": <atual>,
    "nyquist_validation": <atual>,
    "ui_phase": <atual>,
    "ui_safety_gate": <atual>,
    "skip_discuss": <atual>
  }
}
```
</step>

<step name="confirm">
Exibir:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► CONFIGURAÇÕES ATUALIZADAS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Configuração           | Valor |
|------------------------|-------|
| Perfil de Modelo       | {quality/balanced/budget/inherit} |
| Pesquisador de Plano   | {Ligado/Desligado} |
| Verificador de Plano   | {Ligado/Desligado} |
| Verificador de Execução| {Ligado/Desligado} |
| Auto-Avanço            | {Ligado/Desligado} |
| Validação Nyquist      | {Ligado/Desligado} |
| Fase UI                | {Ligado/Desligado} |
| Portão de Segurança UI | {Ligado/Desligado} |
| Branching Git          | {Nenhuma/Por Fase/Por Marco} |
| Pular Discussão        | {Ligado/Desligado} |
| Avisos de Contexto     | {Ligado/Desligado} |
| Salvo como Padrões     | {Sim/Não} |

Estas configurações se aplicam a futuras execuções de /gsd-plan-phase e /gsd-execute-phase.

Comandos rápidos:
- /gsd-set-profile <perfil> — trocar perfil de modelo
- /gsd-plan-phase --research — forçar pesquisa
- /gsd-plan-phase --skip-research — pular pesquisa
- /gsd-plan-phase --skip-verify — pular verificação de plano
```
</step>

</process>

<success_criteria>
- [ ] Config atual lida
- [ ] Usuário apresentado com 10 configurações (perfil + 8 toggles de workflow + branching git)
- [ ] Config atualizada com seções model_profile, workflow e git
- [ ] Oferecido ao usuário salvar como padrões globais (~/.gsd/defaults.json)
- [ ] Mudanças confirmadas ao usuário
</success_criteria>
