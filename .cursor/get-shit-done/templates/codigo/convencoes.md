# Template de Convenções de Código

Template para `.planning/codebase/CONVENTIONS.md` - captura estilo de código e padrões.

**Propósito:** Documentar como o código é escrito neste projeto. Guia prescritivo para o Claude seguir o estilo existente.

---

## Template do Arquivo

```markdown
# Convenções de Código

**Data da Análise:** [AAAA-MM-DD]

## Padrões de Nomenclatura

**Arquivos:**
- [Padrão: ex., "kebab-case para todos os arquivos"]
- [Arquivos de teste: ex., "*.test.ts junto com o fonte"]
- [Componentes: ex., "PascalCase.tsx para componentes React"]

**Funções:**
- [Padrão: ex., "camelCase para todas as funções"]
- [Async: ex., "sem prefixo especial para funções async"]
- [Handlers: ex., "handleNomeEvento para handlers de evento"]

**Variáveis:**
- [Padrão: ex., "camelCase para variáveis"]
- [Constantes: ex., "UPPER_SNAKE_CASE para constantes"]
- [Privadas: ex., "_prefixo para membros privados" ou "sem prefixo"]

**Tipos:**
- [Interfaces: ex., "PascalCase, sem prefixo I"]
- [Types: ex., "PascalCase para type aliases"]
- [Enums: ex., "PascalCase para nome do enum, UPPER_CASE para valores"]

## Estilo de Código

**Formatação:**
- [Ferramenta: ex., "Prettier com config em .prettierrc"]
- [Comprimento de linha: ex., "100 caracteres máx"]
- [Aspas: ex., "aspas simples para strings"]
- [Ponto e vírgula: ex., "obrigatório" ou "omitido"]

**Linting:**
- [Ferramenta: ex., "ESLint com eslint.config.js"]
- [Regras: ex., "extends airbnb-base, sem console em produção"]
- [Execução: ex., "npm run lint"]

## Organização de Imports

**Ordem:**
1. [ex., "Pacotes externos (react, express, etc.)"]
2. [ex., "Módulos internos (@/lib, @/components)"]
3. [ex., "Imports relativos (., ..)"]
4. [ex., "Imports de tipo (import type {})"]

**Agrupamento:**
- [Linhas em branco: ex., "linha em branco entre grupos"]
- [Ordenação: ex., "alfabética dentro de cada grupo"]

**Path Aliases:**
- [Aliases usados: ex., "@/ para src/, @components/ para src/components/"]

## Tratamento de Erros

**Padrões:**
- [Estratégia: ex., "lançar erros, capturar nas fronteiras"]
- [Erros customizados: ex., "estender classe Error, nomeados *Error"]
- [Async: ex., "usar try/catch, sem cadeias .catch()"]

**Tipos de Erro:**
- [Quando lançar: ex., "entrada inválida, dependências ausentes"]
- [Quando retornar: ex., "falhas esperadas retornam Result<T, E>"]
- [Logging: ex., "logar erro com contexto antes de lançar"]

## Logging

**Framework:**
- [Ferramenta: ex., "console.log, pino, winston"]
- [Níveis: ex., "debug, info, warn, error"]

**Padrões:**
- [Formato: ex., "logging estruturado com objeto de contexto"]
- [Quando: ex., "logar transições de estado, chamadas externas"]
- [Onde: ex., "logar nas fronteiras de serviço, não em utils"]

## Comentários

**Quando Comentar:**
- [ex., "explicar o porquê, não o quê"]
- [ex., "documentar lógica de negócio, algoritmos, casos extremos"]
- [ex., "evitar comentários óbvios como // incrementar contador"]

**JSDoc/TSDoc:**
- [Uso: ex., "obrigatório para APIs públicas, opcional para internas"]
- [Formato: ex., "usar tags @param, @returns, @throws"]

**Comentários TODO:**
- [Padrão: ex., "// TODO(usuario): descrição"]
- [Rastreamento: ex., "linkar para número da issue se disponível"]

## Design de Funções

**Tamanho:**
- [ex., "manter abaixo de 50 linhas, extrair helpers"]

**Parâmetros:**
- [ex., "máximo 3 parâmetros, usar objeto para mais"]
- [ex., "desestruturar objetos na lista de parâmetros"]

**Valores de Retorno:**
- [ex., "retornos explícitos, sem undefined implícito"]
- [ex., "retorno antecipado para cláusulas de guarda"]

## Design de Módulos

**Exports:**
- [ex., "exports nomeados preferidos, default exports para componentes React"]
- [ex., "exportar de index.ts para API pública"]

**Barrel Files:**
- [ex., "usar index.ts para re-exportar API pública"]
- [ex., "evitar dependências circulares"]

---

*Análise de convenções: [data]*
*Atualize quando padrões mudarem*
```

<good_examples>
```markdown
# Convenções de Código

**Data da Análise:** 2025-01-20

## Padrões de Nomenclatura

**Arquivos:**
- kebab-case para todos os arquivos (command-handler.ts, user-service.ts)
- *.test.ts junto com arquivos fonte
- index.ts para barrel exports

**Funções:**
- camelCase para todas as funções
- Sem prefixo especial para funções async
- handleNomeEvento para handlers de evento (handleClick, handleSubmit)

**Variáveis:**
- camelCase para variáveis
- UPPER_SNAKE_CASE para constantes (MAX_RETRIES, API_BASE_URL)
- Sem prefixo underscore (sem marcador privado no TS)

**Tipos:**
- PascalCase para interfaces, sem prefixo I (User, não IUser)
- PascalCase para type aliases (UserConfig, ResponseData)
- PascalCase para nomes de enum, UPPER_CASE para valores (Status.PENDING)

## Estilo de Código

**Formatação:**
- Prettier com .prettierrc
- 100 caracteres de comprimento de linha
- Aspas simples para strings
- Ponto e vírgula obrigatório
- Indentação de 2 espaços

**Linting:**
- ESLint com eslint.config.js
- Extends @typescript-eslint/recommended
- Sem console.log em código de produção (usar logger)
- Execução: npm run lint

## Organização de Imports

**Ordem:**
1. Pacotes externos (react, express, commander)
2. Módulos internos (@/lib, @/services)
3. Imports relativos (./utils, ../types)
4. Imports de tipo (import type { User })

**Agrupamento:**
- Linha em branco entre grupos
- Alfabético dentro de cada grupo
- Imports de tipo por último dentro de cada grupo

**Path Aliases:**
- @/ mapeia para src/
- Nenhum outro alias definido

## Tratamento de Erros

**Padrões:**
- Lançar erros, capturar nas fronteiras (handlers de rota, funções main)
- Estender classe Error para erros customizados (ValidationError, NotFoundError)
- Funções async usam try/catch, sem cadeias .catch()

**Tipos de Erro:**
- Lançar em entrada inválida, dependências ausentes, violações de invariante
- Logar erro com contexto antes de lançar: logger.error({ err, userId }, 'Falha ao processar')
- Incluir causa na mensagem de erro: new Error('Falha ao X', { cause: originalError })

## Logging

**Framework:**
- Instância de logger pino exportada de lib/logger.ts
- Níveis: debug, info, warn, error (sem trace)

**Padrões:**
- Logging estruturado com contexto: logger.info({ userId, action }, 'Ação do usuário')
- Logar nas fronteiras de serviço, não em funções utilitárias
- Logar transições de estado, chamadas externas de API, erros
- Sem console.log em código commitado

## Comentários

**Quando Comentar:**
- Explicar o porquê, não o quê: // Retry 3 vezes porque API tem falhas transientes
- Documentar regras de negócio: // Usuários devem verificar email em 24 horas
- Explicar algoritmos não-óbvios ou workarounds
- Evitar comentários óbvios: // definir count como 0

**JSDoc/TSDoc:**
- Obrigatório para funções de API pública
- Opcional para funções internas se assinatura é autoexplicativa
- Usar tags @param, @returns, @throws

**Comentários TODO:**
- Formato: // TODO: descrição (sem nome de usuário, usando git blame)
- Linkar para issue se existir: // TODO: Corrigir condição de corrida (issue #123)

## Design de Funções

**Tamanho:**
- Manter abaixo de 50 linhas
- Extrair helpers para lógica complexa
- Um nível de abstração por função

**Parâmetros:**
- Máximo 3 parâmetros
- Usar objeto de opções para 4+ parâmetros: function create(options: CreateOptions)
- Desestruturar na lista de parâmetros: function process({ id, name }: ProcessParams)

**Valores de Retorno:**
- Instruções de retorno explícitas
- Retorno antecipado para cláusulas de guarda
- Usar tipo Result<T, E> para falhas esperadas

## Design de Módulos

**Exports:**
- Exports nomeados preferidos
- Default exports apenas para componentes React
- Exportar API pública de barrel files index.ts

**Barrel Files:**
- index.ts re-exporta API pública
- Manter helpers internos privados (não exportar do index)
- Evitar dependências circulares (importar de arquivos específicos se necessário)

---

*Análise de convenções: 2025-01-20*
*Atualize quando padrões mudarem*
```
</good_examples>

<guidelines>
**O que pertence ao CONVENTIONS.md:**
- Padrões de nomenclatura observados no código
- Regras de formatação (config do Prettier, regras de linting)
- Padrões de organização de imports
- Estratégia de tratamento de erros
- Abordagem de logging
- Convenções de comentários
- Padrões de design de funções e módulos

**O que NÃO pertence aqui:**
- Decisões de arquitetura (isso é ARCHITECTURE.md)
- Escolhas de tecnologia (isso é STACK.md)
- Padrões de teste (isso é TESTING.md)
- Organização de arquivos (isso é STRUCTURE.md)

**Ao preencher este template:**
- Verifique .prettierrc, .eslintrc ou arquivos de config similares
- Examine 5-10 arquivos fonte representativos para padrões
- Procure consistência: se 80%+ segue um padrão, documente-o
- Seja prescritivo: "Use X" não "Às vezes Y é usado"
- Note desvios: "Código legado usa Y, código novo deve usar X"
- Mantenha abaixo de ~150 linhas no total

**Útil para planejamento de fase quando:**
- Escrevendo código novo (seguir estilo existente)
- Adicionando funcionalidades (seguir padrões de nomenclatura)
- Refatorando (aplicar convenções consistentes)
- Code review (verificar contra padrões documentados)
- Onboarding (entender expectativas de estilo)

**Abordagem de análise:**
- Escanear diretório src/ para padrões de nomenclatura de arquivos
- Verificar scripts do package.json para comandos lint/format
- Ler 5-10 arquivos para identificar nomenclatura de funções, tratamento de erros
- Procurar arquivos de config (.prettierrc, eslint.config.js)
- Notar padrões em imports, comentários, assinaturas de funções
</guidelines>
