# DevSquad Project Analyzer — `_devsquad/core/project-analyzer.md`

## Objetivo

Dar contexto aos agentes e ao usuário **antes** do review. Usado pelo comando `/devsquad project`.

---

## Ações

1. **Estrutura de diretórios:** listar pastas relevantes (ex.: `src/`, raiz do projeto), excluindo `node_modules`, `.git`, build outputs.
2. **Linguagens:** detectar por extensão (`.ts`, `.tsx`, `.js`, `.py`, etc.) e/ou por config (`tsconfig.json`, `package.json`, `requirements.txt`).
3. **Métricas básicas:** tamanho aproximado de arquivos/pastas, quantidade de arquivos por tipo.
4. **Testes:** verificar presença de testes (pastas `test/`, `__tests__/`, `*.test.ts`, `*.spec.ts`, etc.) e framework (Jest, Vitest, pytest, etc.).
5. **Configuração:** ler `package.json`, `tsconfig.json`, ou equivalente para stack e scripts.

---

## Saída (resumo em markdown)

- Árvore resumida do projeto (pastas principais e arquivos de config).
- Linguagens e stack detectados.
- Métricas básicas (número de arquivos, presença de testes).
- **Sugestões de foco** para cada modo:
  - **CREATE:** onde começar (feature, módulo, camada).
  - **ANALYZE:** áreas de maior risco ou maior acoplamento.
  - **REFACTOR:** ordem sugerida (estrutura → domínio → padrões → código).

---

## Uso

Quando o usuário invoca **/devsquad project**, o agente carrega este arquivo e executa as ações acima, entregando o resumo antes de qualquer revisão ou plano do Lucas.
