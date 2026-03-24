<purpose>
Gerar testes unitários e E2E para uma fase concluída baseando-se no SUMMARY.md, CONTEXT.md e implementação. Classifica cada arquivo alterado nas categorias TDD (unitário), E2E (navegador) ou Pular, apresenta um plano de testes para aprovação do usuário, e então gera testes seguindo convenções RED-GREEN.

Atualmente os usuários criam prompts manuais de `/gsd-quick` para geração de testes após cada fase. Este workflow padroniza o processo com classificação adequada, gates de qualidade e relatório de lacunas.
</purpose>

<required_reading>
Leia todos os arquivos referenciados pelo execution_context do prompt invocador antes de começar.
</required_reading>

<process>

<step name="parse_arguments">
Analisar `{{GSD_ARGS}}` para:
- Número da fase (inteiro, decimal ou sufixo de letra) → armazenar como `$PHASE_ARG`
- Texto restante após o número da fase → armazenar como `$EXTRA_INSTRUCTIONS` (opcional)

Exemplo: `/gsd-add-tests 12 focar em casos extremos` → `$PHASE_ARG=12`, `$EXTRA_INSTRUCTIONS="focar em casos extremos"`

Se nenhum argumento de fase fornecido:

```
ERRO: Número da fase obrigatório
Uso: /gsd-add-tests <fase> [instruções adicionais]
Exemplo: /gsd-add-tests 12
Exemplo: /gsd-add-tests 12 focar em casos extremos no módulo de preços
```

Encerrar.
</step>

<step name="init_context">
Carregar contexto da operação de fase:

```bash
INIT=$(node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" init phase-op "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Extrair do JSON de init: `phase_dir`, `phase_number`, `phase_name`.

Verificar se o diretório da fase existe. Se não:
```
ERRO: Diretório da fase não encontrado para fase ${PHASE_ARG}
Verifique se a fase existe em .planning/phases/
```
Encerrar.

Ler os artefatos da fase (em ordem de prioridade):
1. `${phase_dir}/*-SUMMARY.md` — o que foi implementado, arquivos alterados
2. `${phase_dir}/CONTEXT.md` — critérios de aceite, decisões
3. `${phase_dir}/*-VERIFICATION.md` — cenários verificados pelo usuário (se TAU foi feito)

Se nenhum SUMMARY.md existir:
```
ERRO: Nenhum SUMMARY.md encontrado para fase ${PHASE_ARG}
Este comando funciona em fases concluídas. Execute /gsd-execute-phase primeiro.
```
Encerrar.

Apresentar banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► ADICIONAR TESTES — Fase ${phase_number}: ${phase_name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</step>

<step name="analyze_implementation">
Extrair a lista de arquivos modificados pela fase do SUMMARY.md (seção "Arquivos Alterados" ou equivalente).

Para cada arquivo, classificar em uma das três categorias:

| Categoria | Critérios | Tipo de Teste |
|-----------|-----------|---------------|
| **TDD** | Funções puras onde `expect(fn(input)).toBe(output)` é possível | Testes unitários |
| **E2E** | Comportamento de UI verificável por automação de navegador | Testes Playwright/E2E |
| **Pular** | Não testável de forma significativa ou já coberto | Nenhum |

**Classificação TDD — aplicar quando:**
- Lógica de negócio: cálculos, preços, regras fiscais, validação
- Transformações de dados: mapeamento, filtragem, agregação, formatação
- Parsers: CSV, JSON, XML, parsing de formato customizado
- Validadores: validação de entrada, validação de schema, regras de negócio
- Máquinas de estado: transições de status, etapas de workflow
- Utilitários: manipulação de strings, tratamento de datas, formatação de números

**Classificação E2E — aplicar quando:**
- Atalhos de teclado: teclas de atalho, teclas modificadoras, sequências de acordes
- Navegação: transições de página, roteamento, breadcrumbs, voltar/avançar
- Interações de formulário: envio, erros de validação, foco em campo, autocomplete
- Seleção: seleção de linha, multi-seleção, intervalos com shift-click
- Arrastar e soltar: reordenação, mover entre containers
- Diálogos modais: abrir, fechar, confirmar, cancelar
- Grids de dados: ordenação, filtragem, edição inline, redimensionar coluna

**Classificação Pular — aplicar quando:**
- Layout/estilização de UI: classes CSS, aparência visual, breakpoints responsivos
- Configuração: arquivos de config, variáveis de ambiente, feature flags
- Código de cola: setup de injeção de dependência, registro de middleware, tabelas de rotas
- Migrações: migrações de banco, alterações de schema
- CRUD simples: criar/ler/atualizar/deletar básico sem lógica de negócio
- Definições de tipo: registros, DTOs, interfaces sem lógica

Ler cada arquivo para verificar a classificação. Não classificar baseado apenas no nome do arquivo.
</step>

<step name="present_classification">
Apresentar a classificação ao usuário para confirmação antes de prosseguir:

```
conversational prompting(
  header: "Classificação de Testes",
  question: |
    ## Arquivos classificados para testes

    ### TDD (Testes Unitários) — {N} arquivos
    {lista de arquivos com breve razão}

    ### E2E (Testes de Navegador) — {M} arquivos
    {lista de arquivos com breve razão}

    ### Pular — {K} arquivos
    {lista de arquivos com breve razão}

    {se $EXTRA_INSTRUCTIONS: "Instruções adicionais: ${EXTRA_INSTRUCTIONS}"}

    Como deseja prosseguir?
  options:
    - "Aprovar e gerar plano de testes"
    - "Ajustar classificação (vou especificar mudanças)"
    - "Cancelar"
)
```

Se o usuário selecionar "Ajustar classificação": aplicar as mudanças e reapresentar.
Se o usuário selecionar "Cancelar": encerrar graciosamente.
</step>

<step name="discover_test_structure">
Antes de gerar o plano de testes, descobrir a estrutura de testes existente do projeto:

```bash
# Encontrar diretórios de testes existentes
find . -type d -name "*test*" -o -name "*spec*" -o -name "*__tests__*" 2>/dev/null | head -20
# Encontrar arquivos de teste existentes para corresponder convenções
find . -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "*Tests.fs" -o -name "*Test.fs" \) 2>/dev/null | head -20
# Verificar runners de teste
ls package.json *.sln 2>/dev/null
```

Identificar:
- Estrutura de diretórios de teste (onde ficam testes unitários, onde ficam testes E2E)
- Convenções de nomenclatura (`.test.ts`, `.spec.ts`, `*Tests.fs`, etc.)
- Comandos do runner de teste (como executar testes unitários, como executar testes E2E)
- Framework de testes (xUnit, NUnit, Jest, Playwright, etc.)

Se a estrutura de testes for ambígua, perguntar ao usuário:
```
conversational prompting(
  header: "Estrutura de Testes",
  question: "Encontrei múltiplos locais de teste. Onde devo criar os testes?",
  options: [listar locais descobertos]
)
```
</step>

<step name="generate_test_plan">
Para cada arquivo aprovado, criar um plano de testes detalhado.

**Para arquivos TDD**, planejar testes seguindo RED-GREEN-REFACTOR:
1. Identificar funções/métodos testáveis no arquivo
2. Para cada função: listar cenários de entrada, saídas esperadas, casos extremos
3. Nota: como o código já existe, testes podem passar imediatamente — tudo bem, mas verificar se testam o comportamento CORRETO

**Para arquivos E2E**, planejar testes seguindo gates RED-GREEN:
1. Identificar cenários de usuário do CONTEXT.md/VERIFICATION.md
2. Para cada cenário: descrever a ação do usuário, resultado esperado, asserções
3. Nota: gate RED significa confirmar que o teste falharia se a funcionalidade estivesse quebrada

Apresentar o plano de testes completo:

```
conversational prompting(
  header: "Plano de Testes",
  question: |
    ## Plano de Geração de Testes

    ### Testes Unitários ({N} testes em {M} arquivos)
    {para cada arquivo: caminho do arquivo de teste, lista de casos de teste}

    ### Testes E2E ({P} testes em {Q} arquivos)
    {para cada arquivo: caminho do arquivo de teste, lista de cenários de teste}

    ### Comandos de Teste
    - Unitários: {comando de teste descoberto}
    - E2E: {comando e2e descoberto}

    Pronto para gerar?
  options:
    - "Gerar todos"
    - "Selecionar específicos (vou especificar quais)"
    - "Ajustar plano"
)
```

Se "Selecionar específicos": perguntar ao usuário quais testes incluir.
Se "Ajustar plano": aplicar mudanças e reapresentar.
</step>

<step name="execute_tdd_generation">
Para cada teste TDD aprovado:

1. **Criar arquivo de teste** seguindo as convenções do projeto descobertas (diretório, nomenclatura, imports)

2. **Escrever teste** com estrutura clara arrange/act/assert:
   ```
   // Arrange — preparar entradas e saídas esperadas
   // Act — chamar a função sob teste
   // Assert — verificar se a saída corresponde às expectativas
   ```

3. **Executar o teste**:
   ```bash
   {comando de teste descoberto}
   ```

4. **Avaliar resultado:**
   - **Teste passa**: Bom — a implementação satisfaz o teste. Verificar se o teste checa comportamento significativo (não apenas que compila).
   - **Teste falha com erro de asserção**: Pode ser um bug genuíno descoberto pelo teste. Sinalizar:
     ```
     ⚠️ Bug potencial encontrado: {nome do teste}
     Esperado: {esperado}
     Real: {real}
     Arquivo: {arquivo de implementação}
     ```
     NÃO corrigir a implementação — este é um comando de geração de testes, não de correção. Registrar a descoberta.
   - **Teste falha com erro (import, sintaxe, etc.)**: É um erro do teste. Corrigir o teste e re-executar.
</step>

<step name="execute_e2e_generation">
Para cada teste E2E aprovado:

1. **Verificar testes existentes** cobrindo o mesmo cenário:
   ```bash
   grep -r "{palavra-chave do cenário}" {diretório de testes e2e} 2>/dev/null
   ```
   Se encontrado, estender em vez de duplicar.

2. **Criar arquivo de teste** visando o cenário do usuário do CONTEXT.md/VERIFICATION.md

3. **Executar o teste E2E**:
   ```bash
   {comando e2e descoberto}
   ```

4. **Avaliar resultado:**
   - **GREEN (passa)**: Registrar sucesso
   - **RED (falha)**: Determinar se é problema do teste ou bug genuíno da aplicação. Sinalizar bugs:
     ```
     ⚠️ Falha E2E: {nome do teste}
     Cenário: {descrição}
     Erro: {mensagem de erro}
     ```
   - **Não é possível executar**: Reportar bloqueador. NÃO marcar como completo.
     ```
     🛑 Bloqueador E2E: {razão pela qual os testes não podem executar}
     ```

**Regra de não-pular:** Se testes E2E não podem executar (dependências faltando, problemas de ambiente), reportar o bloqueador e marcar o teste como incompleto. Nunca marcar sucesso sem realmente executar o teste.
</step>

<step name="summary_and_commit">
Criar relatório de cobertura de testes e apresentar ao usuário:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► GERAÇÃO DE TESTES COMPLETA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Resultados

| Categoria | Gerados | Passando | Falhando | Bloqueados |
|-----------|---------|----------|----------|------------|
| Unitários | {N}     | {n1}     | {n2}     | {n3}       |
| E2E       | {M}     | {m1}     | {m2}     | {m3}       |

## Arquivos Criados/Modificados
{lista de arquivos de teste com caminhos}

## Lacunas de Cobertura
{áreas que não puderam ser testadas e por quê}

## Bugs Descobertos
{quaisquer falhas de asserção que indicam bugs de implementação}
```

Registrar geração de testes no estado do projeto:
```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" state-snapshot
```

Se houver testes passando para commitar:

```bash
git add {arquivos de teste}
git commit -m "test(phase-${phase_number}): add unit and E2E tests from add-tests command"
```

Apresentar próximos passos:

```
---

## ▶ Próximo Passo

{se bugs descobertos:}
**Corrigir bugs descobertos:** `/gsd-quick corrigir as {N} falhas de teste descobertas na fase ${phase_number}`

{se testes bloqueados:}
**Resolver bloqueadores de teste:** {descrição do que é necessário}

{caso contrário:}
**Todos os testes passando!** Fase ${phase_number} está totalmente testada.

---

**Também disponível:**
- `/gsd-add-tests {próxima_fase}` — testar outra fase
- `/gsd-verify-work {phase_number}` — executar verificação TAU

---
```
</step>

</process>

<success_criteria>
- [ ] Artefatos da fase carregados (SUMMARY.md, CONTEXT.md, opcionalmente VERIFICATION.md)
- [ ] Todos os arquivos alterados classificados nas categorias TDD/E2E/Pular
- [ ] Classificação apresentada ao usuário e aprovada
- [ ] Estrutura de testes do projeto descoberta (diretórios, convenções, runners)
- [ ] Plano de testes apresentado ao usuário e aprovado
- [ ] Testes TDD gerados com estrutura arrange/act/assert
- [ ] Testes E2E gerados visando cenários do usuário
- [ ] Todos os testes executados — nenhum teste não executado marcado como passando
- [ ] Bugs descobertos por testes sinalizados (não corrigidos)
- [ ] Arquivos de teste commitados com mensagem adequada
- [ ] Lacunas de cobertura documentadas
- [ ] Próximos passos apresentados ao usuário
</success_criteria>
