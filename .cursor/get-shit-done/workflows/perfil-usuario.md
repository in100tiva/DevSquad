<purpose>
Orquestrar o fluxo completo de perfilamento do desenvolvedor: consentimento, análise de sessão (ou fallback por questionário), geração de perfil, exibição dos resultados e criação de artefatos.

Este workflow conecta a Fase 1 (pipeline de sessão) e a Fase 2 (motor de perfilamento) numa experiência coesa para o usuário. O trabalho pesado fica nos subcomandos existentes de gsd-tools.cjs e no agente gsd-perfilador-usuario — este workflow orquestra a sequência, trata ramificações e fornece a UX.
</purpose>

<required_reading>
Antes de começar, leia todos os arquivos referenciados pelo execution_context do prompt invocador.

Referências principais:
- @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/marca-ui.md (padrões de exibição)
- @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/agents/gsd-perfilador-usuario.md (definição do agente de perfilamento)
- @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/perfilamento-usuario.md (documento de referência de perfilamento)
</required_reading>

<process>

## 1. Inicializar

Analise flags em {{GSD_ARGS}}:
- Detectar flag `--questionnaire` (pular análise de sessão, só questionário)
- Detectar flag `--refresh` (reconstruir o perfil mesmo quando já existir)

Verificar se já existe perfil:

```bash
PROFILE_PATH="D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md"
[ -f "$PROFILE_PATH" ] && echo "EXISTS" || echo "NOT_FOUND"
```

**Se o perfil existir E --refresh NÃO estiver definido E --questionnaire NÃO estiver definido:**

Use prompt conversacional:
- header: "Perfil existente"
- question: "Você já tem um perfil. O que deseja fazer?"
- options:
  - "Ver" — Exibir cartão-resumo com os dados do perfil existente e encerrar
  - "Atualizar" — Continuar com o comportamento de --refresh
  - "Cancelar" — Encerrar o workflow

Se "Ver": leia USER-PROFILE.md, exiba o conteúdo formatado como cartão-resumo e encerre.
Se "Atualizar": Defina o comportamento de --refresh e continue.
Se "Cancelar": Exiba "Nenhuma alteração feita." e encerre.

**Se o perfil existir E --refresh estiver definido:**

Fazer backup do perfil existente:
```bash
cp "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md" "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.backup.md"
```

Exibir: "Reanalisando suas sessões para atualizar seu perfil."
Continuar para o passo 2.

**Se não existir perfil:** Continuar para o passo 2.

---

## 2. Portão de consentimento (ACTV-06)

**Pular se** a flag `--questionnaire` estiver definida (não há leitura de JSONL — ir direto ao passo 4b).

Exibir tela de consentimento:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD > PERFILAR SEU ESTILO DE CÓDIGO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

O Claude começa toda conversa de forma genérica. Um perfil ensina ao Claude
como VOCÊ realmente trabalha — não como você acha que trabalha.

## O que vamos analisar

Suas sessões recentes do Cursor, buscando padrões nestas
8 dimensões comportamentais:

| Dimensão              | O que mede                                      |
|-----------------------|-------------------------------------------------|
| Estilo de comunicação | Como você formula pedidos (objetivo vs. detalhado) |
| Velocidade de decisão | Como você escolhe entre opções                  |
| Profundidade de explicação | Quanta explicação você quer junto ao código |
| Abordagem de depuração | Como você enfrenta erros e bugs              |
| Filosofia de UX       | Quanto você prioriza design vs. função          |
| Filosofia de fornecedores | Como você avalia bibliotecas e ferramentas   |
| Gatilhos de frustração | O que faz você corrigir o Claude              |
| Estilo de aprendizado | Como você prefere aprender coisas novas         |

## Tratamento dos dados

✓ Lê arquivos de sessão localmente (somente leitura, nada é alterado)
✓ Analisa padrões de mensagens (não o significado do conteúdo)
✓ Armazena o perfil em D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md
✗ Nada é enviado a serviços externos
✗ Conteúdo sensível (chaves de API, senhas) é excluído automaticamente
```

**Se for o fluxo --refresh:**
Mostrar consentimento abreviado:

```
Reanalisando suas sessões para atualizar seu perfil.
Seu perfil existente foi copiado para USER-PROFILE.backup.md.
```

Use prompt conversacional:
- header: "Atualizar"
- question: "Continuar com a atualização do perfil?"
- options:
  - "Continuar" — Seguir para o passo 3
  - "Cancelar" — Encerrar o workflow

**Se for o fluxo padrão (sem --refresh):**

Use prompt conversacional:
- header: "Pronto?"
- question: "Pronto para analisar suas sessões?"
- options:
  - "Vamos lá" — Seguir para o passo 3 (análise de sessão)
  - "Usar questionário em vez disso" — Ir para o passo 4b (fluxo do questionário)
  - "Agora não" — Exibir "Sem problemas. Execute /gsd-perfil-usuario quando estiver pronto." e encerrar

---

## 3. Varredura de sessões

Exibir: "◆ Varrendo sessões..."

Executar varredura de sessões:
```bash
SCAN_RESULT=$(node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs scan-sessions --json 2>/dev/null)
```

Analisar a saída JSON para obter a contagem de sessões e de projetos.

Exibir: "✓ Encontradas N sessões em M projetos"

**Avaliar suficiência de dados:**
- Contar o total de mensagens disponíveis no resultado da varredura (somar sessões entre projetos)
- Se forem encontradas 0 sessões: Exibir "Nenhuma sessão encontrada. Mudando para o questionário." e ir para o passo 4b
- Se houver sessões: Continuar para o passo 4a

---

## 4a. Fluxo de análise de sessão

Exibir: "◆ Amostrando mensagens..."

Executar amostragem para perfil:
```bash
SAMPLE_RESULT=$(node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs profile-sample --json 2>/dev/null)
```

Analisar a saída JSON para obter o caminho do diretório temporário e a contagem de mensagens.

Exibir: "✓ Amostradas N mensagens de M projetos"

Exibir: "◆ Analisando padrões..."

**Disparar o agente gsd-perfilador-usuario com a ferramenta Task:**

Use a ferramenta Task para disparar o agente `gsd-perfilador-usuario`. Forneça:
- O caminho do arquivo JSONL amostrado, vindo da saída de profile-sample
- O documento de referência de perfilamento em `D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/perfilamento-usuario.md`

O prompt do agente deve seguir esta estrutura:
```
Leia o documento de referência de perfilamento e as mensagens de sessão amostradas; em seguida analise os padrões comportamentais do desenvolvedor nas 8 dimensões.

Referência: @D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/perfilamento-usuario.md
Dados de sessão: @{temp_dir}/profile-sample.jsonl

Analise essas mensagens e devolva sua análise no formato JSON <analysis> especificado no documento de referência.
```

**Analisar a saída do agente:**
- Extrair o bloco JSON `<analysis>` da resposta do agente
- Salvar o JSON de análise num arquivo temporário (no mesmo diretório temporário criado por profile-sample)

```bash
ANALYSIS_PATH="{temp_dir}/analysis.json"
```

Grave o JSON de análise em `$ANALYSIS_PATH`.

Exibir: "✓ Análise concluída (N dimensões pontuadas)"

**Verificar dados escassos:**
- Leia o JSON de análise e confira a contagem total de mensagens
- Se tiverem sido analisadas < 50 mensagens: Observar que um complemento por questionário pode melhorar a precisão. Exibir: "Observação: poucos dados de sessão (N mensagens). Os resultados podem ter menor confiança."

Continuar para o passo 5.

---

## 4b. Fluxo do questionário

Exibir: "Usando o questionário para montar seu perfil."

**Obter perguntas:**
```bash
QUESTIONS=$(node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs profile-questionnaire --json 2>/dev/null)
```

Analisar o JSON de perguntas. Contém 8 perguntas, uma por dimensão.

**Apresentar cada pergunta ao usuário via prompt conversacional:**

Para cada item do array de perguntas:
- header: Nome da dimensão (ex.: "Estilo de comunicação")
- question: Texto da pergunta
- options: Opções de resposta da definição da pergunta

Reunir todas as respostas num objeto JSON de respostas mapeando chaves de dimensão para valores selecionados.

**Salvar respostas em arquivo temporário:**
```bash
ANSWERS_PATH=$(mktemp /tmp/gsd-profile-answers-XXXXXX.json)
```

Grave o JSON de respostas em `$ANSWERS_PATH`.

**Converter respostas em análise:**
```bash
ANALYSIS_RESULT=$(node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs profile-questionnaire --answers "$ANSWERS_PATH" --json 2>/dev/null)
```

Analisar o JSON de análise do resultado.

Salvar o JSON de análise num arquivo temporário:
```bash
ANALYSIS_PATH=$(mktemp /tmp/gsd-profile-analysis-XXXXXX.json)
```

Grave o JSON de análise em `$ANALYSIS_PATH`.

Continuar para o passo 5 (pular resolução de splits, pois o questionário trata ambiguidade internamente).

---

## 5. Resolução de splits

**Pular se** for só o fluxo do questionário (splits já tratados internamente).

Leia o JSON de análise de `$ANALYSIS_PATH`.

Verificar em cada dimensão se `cross_project_consistent: false`.

**Para cada split detectado:**

Use prompt conversacional:
- header: Nome da dimensão (ex.: "Estilo de comunicação")
- question: "Suas sessões mostram padrões diferentes:" seguido do contexto do split (ex.: "projetos CLI/backend -> terse-direct, projetos Frontend/UI -> detailed-structured")
- options:
  - Opção de rating A (ex.: "terse-direct")
  - Opção de rating B (ex.: "detailed-structured")
  - "Dependente de contexto (manter ambos)"

**Se o usuário escolher um rating específico:** Atualizar o campo `rating` da dimensão no JSON de análise para o valor selecionado.

**Se o usuário escolher "Dependente de contexto":** Manter o rating dominante no campo `rating`. Adicionar um `context_note` ao resumo da dimensão descrevendo o split (ex.: "Dependente de contexto: objetivo em projetos CLI, detalhado em projetos frontend").

Grave o JSON de análise atualizado de volta em `$ANALYSIS_PATH`.

---

## 6. Gravação do perfil

Exibir: "◆ Gravando perfil..."

```bash
node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs write-profile --input "$ANALYSIS_PATH" --json 2>/dev/null
```

Exibir: "✓ Perfil gravado em D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md"

---

## 7. Exibição do resultado

Leia o JSON de análise de `$ANALYSIS_PATH` para montar a exibição.

**Mostrar tabela tipo boletim:**

```
## Seu perfil

| Dimensão              | Rating               | Confiança |
|-----------------------|----------------------|-----------|
| Estilo de comunicação | detailed-structured  | HIGH      |
| Velocidade de decisão | deliberate-informed  | MEDIUM    |
| Profundidade de explicação | concise         | HIGH      |
| Abordagem de depuração | hypothesis-driven | MEDIUM    |
| Filosofia de UX       | pragmatic            | LOW       |
| Filosofia de fornecedores | thorough-evaluator | HIGH    |
| Gatilhos de frustração | scope-creep         | MEDIUM    |
| Estilo de aprendizado | self-directed        | HIGH      |
```

(Preencher com valores reais do JSON de análise.)

**Mostrar destaques:**

Escolher 3–4 dimensões com maior confiança e mais sinais de evidência. Formatar assim:

```
## Destaques

- **Comunicação (HIGH):** Você costuma fornecer contexto estruturado com
  cabeçalhos e enunciados do problema antes de fazer pedidos
- **Escolha de bibliotecas (HIGH):** Você pesquisa alternativas a fundo — comparando
  documentação, atividade no GitHub e tamanho do bundle antes de decidir
- **Frustrações (MEDIUM):** Você corrige o Claude com mais frequência por fazer coisas
  que não pediu — escopo crescente é seu principal gatilho
```

Montar os destaques a partir do array `evidence` e dos campos `summary` no JSON de análise. Usar as citações de evidência mais convincentes. Formatar cada uma como "Você tende a..." ou "Você costuma..." com atribuição da evidência.

**Oferecer visualização completa do perfil:**

Use prompt conversacional:
- header: "Perfil"
- question: "Quer ver o perfil completo?"
- options:
  - "Sim" — Leia e exiba o conteúdo completo de USER-PROFILE.md; em seguida continue para o passo 8
  - "Ir para artefatos" — Ir direto ao passo 8

---

## 8. Seleção de artefatos (ACTV-05)

Use prompt conversacional com multiSelect:
- header: "Artefatos"
- question: "Quais artefatos devo gerar?"
- options (TODAS pré-selecionadas por padrão):
  - "Arquivo de comando /gsd-preferencias-dev" — "Carregue suas preferências em qualquer sessão"
  - "Seção de perfil em .cursor/rules/" — "Adicionar perfil ao .cursor/rules/ deste projeto"
  - ".cursor/rules/ global" — "Adicionar perfil em D:/projetos/Estudo/devsquad/.cursor/.cursor/rules/ para todos os projetos"

**Se nenhum artefato for selecionado:** Exibir "Nenhum artefato gerado. Seu perfil está salvo em D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md" e ir para o passo 10.

---

## 9. Geração de artefatos

Gerar os artefatos selecionados em sequência (I/O de arquivo é rápido; não há ganho com agentes em paralelo):

**Para /gsd-preferencias-dev (se selecionado):**

```bash
node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs generate-dev-preferences --analysis "$ANALYSIS_PATH" --json 2>/dev/null
```

Exibir: "✓ Gerado /gsd-preferencias-dev em D:/projetos/Estudo/devsquad/.cursor/commands/gsd/dev-preferences.md"

**Para seção de perfil em .cursor/rules/ (se selecionado):**

```bash
node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs generate-claude-profile --analysis "$ANALYSIS_PATH" --json 2>/dev/null
```

Exibir: "✓ Seção de perfil adicionada em .cursor/rules/"

**Para .cursor/rules/ global (se selecionado):**

```bash
node D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs generate-claude-profile --analysis "$ANALYSIS_PATH" --global --json 2>/dev/null
```

Exibir: "✓ Seção de perfil adicionada em D:/projetos/Estudo/devsquad/.cursor/.cursor/rules/"

**Tratamento de erros:** Se alguma chamada a gsd-tools.cjs falhar, exibir a mensagem de erro e usar prompt conversacional para oferecer "Tentar de novo" ou "Pular este artefato". Ao tentar de novo, executar o comando novamente. Ao pular, seguir para o próximo artefato.

---

## 10. Resumo e diff de atualização

**Se for o fluxo --refresh:**

Leia o backup antigo e a nova análise para comparar ratings e confiança por dimensão.

Leia o perfil em backup:
```bash
BACKUP_PATH="D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.backup.md"
```

Comparar rating e confiança de cada dimensão entre antigo e novo. Exibir tabela de diff apenas com dimensões alteradas:

```
## Alterações

| Dimensão     | Antes                       | Depois                       |
|--------------|-----------------------------|------------------------------|
| Comunicação  | terse-direct (LOW)          | detailed-structured (HIGH)   |
| Depuração    | fix-first (MEDIUM)          | hypothesis-driven (MEDIUM)   |
```

Se nada mudou: Exibir "Nenhuma alteração detectada — seu perfil já está atualizado."

**Exibir resumo final:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD > PERFIL CONCLUÍDO ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Seu perfil:    D:/projetos/Estudo/devsquad/.cursor/get-shit-done/USER-PROFILE.md
```

Em seguida listar os caminhos de cada artefato gerado:
```
Artefatos:
  ✓ /gsd-preferencias-dev   D:/projetos/Estudo/devsquad/.cursor/commands/gsd/dev-preferences.md
  ✓ seção .cursor/rules/         .cursor/rules/
  ✓ .cursor/rules/ global        D:/projetos/Estudo/devsquad/.cursor/.cursor/rules/
```

(Mostrar apenas artefatos que foram de fato gerados.)

**Limpar arquivos temporários:**

Remover o diretório temporário criado por profile-sample (contém JSONL de amostra e JSON de análise):
```bash
rm -rf "$TEMP_DIR"
```

Remover também arquivos temporários avulsos criados para respostas do questionário:
```bash
rm -f "$ANSWERS_PATH" 2>/dev/null
rm -f "$ANALYSIS_PATH" 2>/dev/null
```

(Limpar apenas caminhos temporários que tenham sido criados nesta execução do workflow.)

</process>

<success_criteria>
- [ ] A inicialização detecta perfil existente e trata as três respostas (ver/atualizar/cancelar)
- [ ] O portão de consentimento é exibido no fluxo de análise de sessão e omitido no fluxo do questionário
- [ ] A varredura de sessões encontra sessões e reporta estatísticas
- [ ] Fluxo de análise de sessão: amostra mensagens, dispara o agente de perfilamento e extrai o JSON de análise
- [ ] Fluxo do questionário: apresenta 8 perguntas, coleta respostas e converte em JSON de análise
- [ ] A resolução de splits apresenta splits dependentes de contexto com opções de resolução para o usuário
- [ ] O perfil é gravado em USER-PROFILE.md via subcomando write-profile
- [ ] A exibição do resultado mostra a tabela tipo boletim e os destaques com evidências
- [ ] A seleção de artefatos usa multiSelect com todas as opções pré-selecionadas
- [ ] Os artefatos são gerados em sequência via subcomandos de gsd-tools.cjs
- [ ] O diff de atualização mostra dimensões alteradas quando --refresh foi usado
- [ ] Arquivos temporários são removidos ao concluir
</success_criteria>
