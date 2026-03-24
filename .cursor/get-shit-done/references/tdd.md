<overview>
TDD é sobre qualidade de design, não métricas de cobertura. O ciclo red-green-refactor te força a pensar sobre comportamento antes de implementação, produzindo interfaces mais limpas e código mais testável.

**Princípio:** Se você pode descrever o comportamento como `expect(fn(input)).toBe(output)` antes de escrever `fn`, TDD melhora o resultado.

**Insight chave:** Trabalho TDD é fundamentalmente mais pesado que tarefas padrão — requer 2-3 ciclos de execução (RED → GREEN → REFACTOR), cada um com leitura de arquivos, execução de testes e debugging potencial. Funcionalidades TDD ganham planos dedicados para garantir contexto completo disponível durante todo o ciclo.
</overview>

<when_to_use_tdd>
## Quando TDD Melhora a Qualidade

**Candidatos a TDD (criar um plano TDD):**
- Lógica de negócio com entradas/saídas definidas
- Endpoints de API com contratos de request/response
- Transformações de dados, parsing, formatação
- Regras de validação e restrições
- Algoritmos com comportamento testável
- Máquinas de estado e workflows
- Funções utilitárias com especificações claras

**Pular TDD (usar plano padrão com tarefas `type="auto"`):**
- Layout de UI, estilização, componentes visuais
- Mudanças de configuração
- Código de cola conectando componentes existentes
- Scripts únicos e migrações
- CRUD simples sem lógica de negócio
- Prototipagem exploratória

**Heurística:** Você pode escrever `expect(fn(input)).toBe(output)` antes de escrever `fn`?
→ Sim: Criar um plano TDD
→ Não: Usar plano padrão, adicionar testes depois se necessário
</when_to_use_tdd>

<tdd_plan_structure>
## Estrutura do Plano TDD

Cada plano TDD implementa **uma funcionalidade** através do ciclo completo RED-GREEN-REFACTOR.

```markdown
---
phase: XX-name
plan: NN
type: tdd
---

<objective>
[Qual funcionalidade e por quê]
Propósito: [Benefício de design do TDD para esta funcionalidade]
Saída: [Funcionalidade funcionando e testada]
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@relevant/source/files.ts
</context>

<feature>
  <name>[Nome da funcionalidade]</name>
  <files>[arquivo fonte, arquivo de teste]</files>
  <behavior>
    [Comportamento esperado em termos testáveis]
    Casos: entrada → saída esperada
  </behavior>
  <implementation>[Como implementar quando testes passarem]</implementation>
</feature>

<verification>
[Comando de teste que prova que funcionalidade funciona]
</verification>

<success_criteria>
- Teste falhando escrito e commitado
- Implementação passa o teste
- Refactor completo (se necessário)
- Todos os 2-3 commits presentes
</success_criteria>

<output>
Após conclusão, criar SUMMARY.md com:
- RED: Qual teste foi escrito, por que falhou
- GREEN: Qual implementação fez passar
- REFACTOR: Qual limpeza foi feita (se alguma)
- Commits: Lista de commits produzidos
</output>
```

**Uma funcionalidade por plano TDD.** Se funcionalidades são triviais o suficiente para agrupar, são triviais o suficiente para pular TDD — use um plano padrão e adicione testes depois.
</tdd_plan_structure>

<execution_flow>
## Ciclo Red-Green-Refactor

**RED - Escrever teste falhando:**
1. Criar arquivo de teste seguindo convenções do projeto
2. Escrever teste descrevendo comportamento esperado (do elemento `<behavior>`)
3. Executar teste - DEVE falhar
4. Se teste passa: funcionalidade já existe ou teste está errado. Investigar.
5. Commit: `test({fase}-{plano}): add failing test for [funcionalidade]`

**GREEN - Implementar para passar:**
1. Escrever código mínimo para fazer teste passar
2. Sem esperteza, sem otimização - apenas fazer funcionar
3. Executar teste - DEVE passar
4. Commit: `feat({fase}-{plano}): implement [funcionalidade]`

**REFACTOR (se necessário):**
1. Limpar implementação se melhorias óbvias existirem
2. Executar testes - DEVEM continuar passando
3. Só commitar se mudanças feitas: `refactor({fase}-{plano}): clean up [funcionalidade]`

**Resultado:** Cada plano TDD produz 2-3 commits atômicos.
</execution_flow>

<test_quality>
## Bons Testes vs Testes Ruins

**Testar comportamento, não implementação:**
- Bom: "retorna string de data formatada"
- Ruim: "chama helper formatDate com parâmetros corretos"
- Testes devem sobreviver a refactors

**Um conceito por teste:**
- Bom: Testes separados para entrada válida, entrada vazia, entrada malformada
- Ruim: Teste único verificando todos os edge cases com múltiplas asserções

**Nomes descritivos:**
- Bom: "deve rejeitar email vazio", "retorna null para ID inválido"
- Ruim: "test1", "trata erro", "funciona corretamente"

**Sem detalhes de implementação:**
- Bom: Testar API pública, comportamento observável
- Ruim: Mockar internos, testar métodos privados, assertar em estado interno
</test_quality>

<framework_setup>
## Setup de Framework de Teste (Se Nenhum Existe)

Quando executando um plano TDD mas nenhum framework de teste está configurado, configure como parte da fase RED:

**1. Detectar tipo de projeto:**
```bash
# JavaScript/TypeScript
if [ -f package.json ]; then echo "node"; fi

# Python
if [ -f requirements.txt ] || [ -f pyproject.toml ]; then echo "python"; fi

# Go
if [ -f go.mod ]; then echo "go"; fi

# Rust
if [ -f Cargo.toml ]; then echo "rust"; fi
```

**2. Instalar framework mínimo:**
| Projeto | Framework | Instalar |
|---------|-----------|----------|
| Node.js | Jest | `npm install -D jest @types/jest ts-jest` |
| Node.js (Vite) | Vitest | `npm install -D vitest` |
| Python | pytest | `pip install pytest` |
| Go | testing | Embutido |
| Rust | cargo test | Embutido |

**3. Criar config se necessário:**
- Jest: `jest.config.js` com preset ts-jest
- Vitest: `vitest.config.ts` com test globals
- pytest: `pytest.ini` ou seção no `pyproject.toml`

**4. Verificar setup:**
```bash
# Executar suíte de testes vazia - deve passar com 0 testes
npm test  # Node
pytest    # Python
go test ./...  # Go
cargo test    # Rust
```

**5. Criar primeiro arquivo de teste:**
Seguir convenções do projeto para localização de testes:
- `*.test.ts` / `*.spec.ts` ao lado do fonte
- Diretório `__tests__/`
- Diretório `tests/` na raiz

Setup de framework é um custo único incluído na fase RED do primeiro plano TDD.
</framework_setup>

<error_handling>
## Tratamento de Erros

**Teste não falha na fase RED:**
- Funcionalidade pode já existir - investigar
- Teste pode estar errado (não testando o que você pensa)
- Corrigir antes de prosseguir

**Teste não passa na fase GREEN:**
- Debugar implementação
- Não pular para refactor
- Continuar iterando até verde

**Testes falham na fase REFACTOR:**
- Desfazer refactor
- Commit foi prematuro
- Refatorar em passos menores

**Testes não relacionados quebram:**
- Parar e investigar
- Pode indicar problema de acoplamento
- Corrigir antes de prosseguir
</error_handling>

<commit_pattern>
## Padrão de Commit para Planos TDD

Planos TDD produzem 2-3 commits atômicos (um por fase):

```
test(08-02): add failing test for email validation

- Testa formatos de email válidos aceitos
- Testa formatos inválidos rejeitados
- Testa tratamento de entrada vazia

feat(08-02): implement email validation

- Padrão regex corresponde RFC 5322
- Retorna booleano para validade
- Trata edge cases (vazio, null)

refactor(08-02): extract regex to constant (opcional)

- Moveu padrão para constante EMAIL_REGEX
- Sem mudanças de comportamento
- Testes continuam passando
```

**Comparação com planos padrão:**
- Planos padrão: 1 commit por tarefa, 2-4 commits por plano
- Planos TDD: 2-3 commits para funcionalidade única

Ambos seguem mesmo formato: `{tipo}({fase}-{plano}): {descrição}`

**Benefícios:**
- Cada commit independentemente reversível
- Git bisect funciona no nível de commit
- Histórico claro mostrando disciplina TDD
- Consistente com estratégia geral de commit
</commit_pattern>

<context_budget>
## Orçamento de Contexto

Planos TDD visam **~40% de uso de contexto** (menor que os ~50% dos planos padrão).

Por que menor:
- Fase RED: escrever teste, executar teste, potencialmente debugar por que não falhou
- Fase GREEN: implementar, executar teste, potencialmente iterar em falhas
- Fase REFACTOR: modificar código, executar testes, verificar sem regressões

Cada fase envolve leitura de arquivos, execução de comandos, análise de saída. O vai-e-volta é inerentemente mais pesado que execução linear de tarefas.

Foco em funcionalidade única garante qualidade total durante todo o ciclo.
</context_budget>
