<purpose>
Orquestrar agentes mapeadores de codebase em paralelo para analisar o codebase e produzir documentos estruturados em .planning/codebase/

Cada agente tem contexto fresco, explora uma área de foco específica, e **escreve documentos diretamente**. O orquestrador só recebe confirmação + contagem de linhas, depois escreve um resumo.

Saída: pasta .planning/codebase/ com 7 documentos estruturados sobre o estado do codebase.
</purpose>

<philosophy>
**Por que agentes mapeadores dedicados:**
- Contexto fresco por domínio (sem contaminação de tokens)
- Agentes escrevem documentos diretamente (sem transferência de contexto de volta ao orquestrador)
- Orquestrador só resume o que foi criado (uso mínimo de contexto)
- Execução mais rápida (agentes rodam simultaneamente)

**Qualidade do documento sobre comprimento:**
Incluir detalhe suficiente para ser útil como referência. Priorizar exemplos práticos (especialmente padrões de código) sobre brevidade arbitrária.

**Sempre incluir caminhos de arquivo:**
Documentos são material de referência para o Claude ao planejar/executar. Sempre incluir caminhos reais de arquivo formatados com crases: `src/services/user.ts`.
</philosophy>

<process>

<step name="init_context" priority="first">
Carregar contexto de mapeamento do codebase:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init map-codebase)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `mapper_model`, `commit_docs`, `codebase_dir`, `existing_maps`, `has_maps`, `codebase_dir_exists`.
</step>

<step name="check_existing">
Verificar se .planning/codebase/ já existe usando `has_maps` do contexto de init.

Se `codebase_dir_exists` for true:
```bash
ls -la .planning/codebase/
```

**Se existir:**

```
.planning/codebase/ já existe com estes documentos:
[Listar arquivos encontrados]

O que fazer?
1. Atualizar - Deletar existentes e remapear codebase
2. Atualizar parcial - Manter existentes, atualizar apenas documentos específicos
3. Pular - Usar mapa de codebase existente como está
```

Aguardar resposta do usuário.

Se "Atualizar": Deletar .planning/codebase/, continuar para create_structure
Se "Atualizar parcial": Perguntar quais documentos atualizar, continuar para spawn_agents (filtrado)
Se "Pular": Sair do workflow

**Se não existir:**
Continuar para create_structure.
</step>

<step name="create_structure">
Criar diretório .planning/codebase/:

```bash
mkdir -p .planning/codebase
```

**Arquivos de saída esperados:**
- STACK.md (do mapeador tech)
- INTEGRATIONS.md (do mapeador tech)
- ARCHITECTURE.md (do mapeador arch)
- STRUCTURE.md (do mapeador arch)
- CONVENTIONS.md (do mapeador quality)
- TESTING.md (do mapeador quality)
- CONCERNS.md (do mapeador concerns)

Continuar para spawn_agents.
</step>

<step name="detect_runtime_capabilities">
Antes de invocar agentes, detectar se o runtime atual suporta a ferramenta `Task` para delegação a subagentes.

**Runtimes com ferramenta Task:** Cursor, OpenCode (suporte nativo a subagentes via `Task` ou `task`)
**Runtimes SEM ferramenta Task:** Antigravity, Gemini CLI, Codex e outros

**Como detectar:** Verificar se você tem acesso a uma ferramenta `Task` ou `task` (qualquer capitalização conta). Se você NÃO tem uma ferramenta Task/task (ou só tem ferramentas como `browser_subagent` que é para navegação web, NÃO análise de código):

→ **Pular `spawn_agents` e `collect_confirmations`** — ir direto para `sequential_mapping`.

**CRÍTICO:** Nunca use `browser_subagent` ou `Explore` como substituto para `Task`. A ferramenta `browser_subagent` é exclusivamente para interação com páginas web e falhará para análise de codebase. Se `Task` não estiver disponível, realize o mapeamento sequencialmente no contexto.
</step>

<step name="spawn_agents" condition="Ferramenta Task está disponível">
Invocar 4 agentes gsd-mapeador-codigo em paralelo.

Usar ferramenta Task com `subagent_type="gsd-mapeador-codigo"`, `model="{mapper_model}"`, e `run_in_background=true` para execução paralela.

**CRÍTICO:** Usar o agente dedicado `gsd-mapeador-codigo`, NÃO `Explore` ou `browser_subagent`. O agente mapeador escreve documentos diretamente.

**Agente 1: Foco Tech**

```
Task(
  subagent_type="gsd-mapeador-codigo",
  model="{mapper_model}",
  run_in_background=true,
  description="Mapear stack tech do codebase",
  prompt="Foco: tech

Analisar este codebase para stack tecnológico e integrações externas.

Escrever estes documentos em .planning/codebase/:
- STACK.md - Linguagens, runtime, frameworks, dependências, configuração
- INTEGRATIONS.md - APIs externas, bancos de dados, provedores de auth, webhooks

Explorar completamente. Escrever documentos diretamente usando templates. Retornar apenas confirmação."
)
```

**Agente 2: Foco Arquitetura**

```
Task(
  subagent_type="gsd-mapeador-codigo",
  model="{mapper_model}",
  run_in_background=true,
  description="Mapear arquitetura do codebase",
  prompt="Foco: arch

Analisar a arquitetura e estrutura de diretórios deste codebase.

Escrever estes documentos em .planning/codebase/:
- ARCHITECTURE.md - Padrão, camadas, fluxo de dados, abstrações, pontos de entrada
- STRUCTURE.md - Layout de diretórios, locais-chave, convenções de nomeação

Explorar completamente. Escrever documentos diretamente usando templates. Retornar apenas confirmação."
)
```

**Agente 3: Foco Qualidade**

```
Task(
  subagent_type="gsd-mapeador-codigo",
  model="{mapper_model}",
  run_in_background=true,
  description="Mapear convenções do codebase",
  prompt="Foco: quality

Analisar este codebase para convenções de código e padrões de teste.

Escrever estes documentos em .planning/codebase/:
- CONVENTIONS.md - Estilo de código, nomeação, padrões, tratamento de erros
- TESTING.md - Framework, estrutura, mocking, cobertura

Explorar completamente. Escrever documentos diretamente usando templates. Retornar apenas confirmação."
)
```

**Agente 4: Foco Preocupações**

```
Task(
  subagent_type="gsd-mapeador-codigo",
  model="{mapper_model}",
  run_in_background=true,
  description="Mapear preocupações do codebase",
  prompt="Foco: concerns

Analisar este codebase para dívida técnica, problemas conhecidos e áreas de preocupação.

Escrever este documento em .planning/codebase/:
- CONCERNS.md - Dívida técnica, bugs, segurança, performance, áreas frágeis

Explorar completamente. Escrever documento diretamente usando template. Retornar apenas confirmação."
)
```

Continuar para collect_confirmations.
</step>

<step name="collect_confirmations">
Aguardar todos os 4 agentes completarem usando ferramenta TaskOutput.

**Para cada task_id do agente retornado pelas chamadas da ferramenta Agent acima:**
```
TaskOutput tool:
  task_id: "{task_id do resultado do Agent}"
  block: true
  timeout: 300000
```

Chamar TaskOutput para todos os 4 agentes em paralelo (mensagem única com 4 chamadas TaskOutput).

Quando todas as chamadas TaskOutput retornarem, ler o arquivo de saída de cada agente para coletar confirmações.

**Formato de confirmação esperado de cada agente:**
```
## Mapeamento Completo

**Foco:** {foco}
**Documentos escritos:**
- `.planning/codebase/{DOC1}.md` ({N} linhas)
- `.planning/codebase/{DOC2}.md` ({N} linhas)

Pronto para resumo do orquestrador.
```

**O que você recebe:** Apenas caminhos de arquivo e contagens de linhas. NÃO conteúdo dos documentos.

Se algum agente falhou, anotar a falha e continuar com documentos bem-sucedidos.

Continuar para verify_output.
</step>

<step name="sequential_mapping" condition="Ferramenta Task/task NÃO está disponível (ex: Antigravity, Gemini CLI, Codex)">
Quando a ferramenta `Task` não estiver disponível, realizar mapeamento do codebase sequencialmente no contexto atual. Isso substitui `spawn_agents` e `collect_confirmations`.

**IMPORTANTE:** NÃO use `browser_subagent`, `Explore`, ou qualquer ferramenta baseada em navegador. Use apenas ferramentas de sistema de arquivos (Read, Bash, Write, Grep, Glob, list_dir, view_file, grep_search ou ferramentas equivalentes disponíveis no seu runtime).

Realizar todas as 4 passagens de mapeamento sequencialmente:

**Passagem 1: Foco Tech**
- Explorar package.json/Cargo.toml/go.mod/requirements.txt, arquivos de config, árvores de dependência
- Escrever `.planning/codebase/STACK.md` — Linguagens, runtime, frameworks, dependências, configuração
- Escrever `.planning/codebase/INTEGRATIONS.md` — APIs externas, bancos de dados, provedores de auth, webhooks

**Passagem 2: Foco Arquitetura**
- Explorar estrutura de diretórios, pontos de entrada, limites de módulos, fluxo de dados
- Escrever `.planning/codebase/ARCHITECTURE.md` — Padrão, camadas, fluxo de dados, abstrações, pontos de entrada
- Escrever `.planning/codebase/STRUCTURE.md` — Layout de diretórios, locais-chave, convenções de nomeação

**Passagem 3: Foco Qualidade**
- Explorar estilo de código, padrões de tratamento de erros, arquivos de teste, config de CI
- Escrever `.planning/codebase/CONVENTIONS.md` — Estilo de código, nomeação, padrões, tratamento de erros
- Escrever `.planning/codebase/TESTING.md` — Framework, estrutura, mocking, cobertura

**Passagem 4: Foco Preocupações**
- Explorar TODOs, problemas conhecidos, áreas frágeis, padrões de segurança
- Escrever `.planning/codebase/CONCERNS.md` — Dívida técnica, bugs, segurança, performance, áreas frágeis

Usar os mesmos templates de documento que o agente `gsd-mapeador-codigo`. Incluir caminhos reais de arquivo formatados com crases.

Continuar para verify_output.
</step>

<step name="verify_output">
Verificar que todos os documentos foram criados com sucesso:

```bash
ls -la .planning/codebase/
wc -l .planning/codebase/*.md
```

**Checklist de verificação:**
- Todos 7 documentos existem
- Nenhum documento vazio (cada um deve ter >20 linhas)

Se algum documento estiver faltando ou vazio, anotar quais agentes podem ter falhado.

Continuar para scan_for_secrets.
</step>

<step name="scan_for_secrets">
**VERIFICAÇÃO CRÍTICA DE SEGURANÇA:** Varrer arquivos de saída para segredos acidentalmente vazados antes de commitar.

Executar detecção de padrões de segredo:

```bash
# Verificar padrões comuns de chaves de API nos docs gerados
grep -E '(sk-[a-zA-Z0-9]{20,}|sk_live_[a-zA-Z0-9]+|sk_test_[a-zA-Z0-9]+|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|glpat-[a-zA-Z0-9_-]+|AKIA[A-Z0-9]{16}|xox[baprs]-[a-zA-Z0-9-]+|-----BEGIN.*PRIVATE KEY|eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.)' .planning/codebase/*.md 2>/dev/null && SECRETS_FOUND=true || SECRETS_FOUND=false
```

**Se SECRETS_FOUND=true:**

```
⚠️  ALERTA DE SEGURANÇA: Potenciais segredos detectados nos documentos do codebase!

Encontrados padrões que parecem chaves de API ou tokens em:
[mostrar saída do grep]

Isso exporia credenciais se commitado.

**Ação necessária:**
1. Revise o conteúdo sinalizado acima
2. Se estes são segredos reais, devem ser removidos antes de commitar
3. Considere adicionar arquivos sensíveis às permissões "Deny" do Cursor

Pausando antes do commit. Responda "seguro para prosseguir" se o conteúdo sinalizado não é realmente sensível, ou edite os arquivos primeiro.
```

Aguardar confirmação do usuário antes de continuar para commit_codebase_map.

**Se SECRETS_FOUND=false:**

Continuar para commit_codebase_map.
</step>

<step name="commit_codebase_map">
Commitar o mapa do codebase:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "docs: mapear codebase existente" --files .planning/codebase/*.md
```

Continuar para offer_next.
</step>

<step name="offer_next">
Apresentar resumo de conclusão e próximos passos.

**Obter contagens de linhas:**
```bash
wc -l .planning/codebase/*.md
```

**Formato de saída:**

```
Mapeamento do codebase concluído.

Criado .planning/codebase/:
- STACK.md ([N] linhas) - Tecnologias e dependências
- ARCHITECTURE.md ([N] linhas) - Design do sistema e padrões
- STRUCTURE.md ([N] linhas) - Layout e organização de diretórios
- CONVENTIONS.md ([N] linhas) - Estilo e padrões de código
- TESTING.md ([N] linhas) - Estrutura e práticas de teste
- INTEGRATIONS.md ([N] linhas) - Serviços e APIs externos
- CONCERNS.md ([N] linhas) - Dívida técnica e problemas


---

## ▶ Próximo

**Inicializar projeto** — usar contexto do codebase para planejamento

`/gsd-novo-projeto`

<sub>`/clear` primeiro → janela de contexto limpa</sub>

---

**Também disponível:**
- Re-executar mapeamento: `/gsd-mapear-codigo`
- Revisar arquivo específico: `cat .planning/codebase/STACK.md`
- Editar qualquer documento antes de prosseguir

---
```

Finalizar workflow.
</step>

</process>

<success_criteria>
- Diretório .planning/codebase/ criado
- Se ferramenta Task disponível: 4 agentes gsd-mapeador-codigo paralelos invocados com run_in_background=true
- Se ferramenta Task NÃO disponível: 4 passagens de mapeamento sequenciais realizadas inline (nunca usando browser_subagent)
- Todos 7 documentos de codebase existem
- Nenhum documento vazio (cada um deve ter >20 linhas)
- Resumo claro de conclusão com contagens de linhas
- Usuário recebe próximos passos claros no estilo GSD
</success_criteria>
</output>
