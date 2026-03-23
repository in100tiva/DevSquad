# Template de Padrões de Teste

Template para `.planning/codebase/TESTING.md` - captura framework e padrões de teste.

**Propósito:** Documentar como testes são escritos e executados. Guia para adicionar testes que seguem os padrões existentes.

---

## Template do Arquivo

```markdown
# Padrões de Teste

**Data da Análise:** [AAAA-MM-DD]

## Framework de Teste

**Runner:**
- [Framework: ex., "Jest 29.x", "Vitest 1.x"]
- [Config: ex., "jest.config.js na raiz do projeto"]

**Biblioteca de Asserção:**
- [Biblioteca: ex., "expect built-in", "chai"]
- [Matchers: ex., "toBe, toEqual, toThrow"]

**Comandos de Execução:**
```bash
[ex., "npm test" ou "npm run test"]              # Executar todos os testes
[ex., "npm test -- --watch"]                     # Modo watch
[ex., "npm test -- path/to/file.test.ts"]       # Arquivo único
[ex., "npm run test:coverage"]                   # Relatório de cobertura
```

## Organização dos Arquivos de Teste

**Localização:**
- [Padrão: ex., "*.test.ts junto com arquivos fonte"]
- [Alternativa: ex., "diretório __tests__/" ou "árvore tests/ separada"]

**Nomenclatura:**
- [Testes unitários: ex., "nome-do-modulo.test.ts"]
- [Integração: ex., "nome-funcionalidade.integration.test.ts"]
- [E2E: ex., "fluxo-usuario.e2e.test.ts"]

**Estrutura:**
```
[Mostre o padrão real do diretório, ex.:
src/
  lib/
    utils.ts
    utils.test.ts
  services/
    user-service.ts
    user-service.test.ts
]
```

## Estrutura do Teste

**Organização de Suite:**
```typescript
[Mostre o padrão real usado, ex.:

describe('NomeDoModulo', () => {
  describe('nomeDaFuncao', () => {
    it('deve tratar caso de sucesso', () => {
      // arrange
      // act
      // assert
    });

    it('deve tratar caso de erro', () => {
      // código do teste
    });
  });
});
]
```

**Padrões:**
- [Setup: ex., "beforeEach para setup compartilhado, evitar beforeAll"]
- [Teardown: ex., "afterEach para limpar, restaurar mocks"]
- [Estrutura: ex., "padrão arrange/act/assert obrigatório"]

## Mocking

**Framework:**
- [Ferramenta: ex., "Mocking built-in do Jest", "vi do Vitest", "Sinon"]
- [Mock de imports: ex., "vi.mock() no topo do arquivo"]

**Padrões:**
```typescript
[Mostre o padrão real de mocking, ex.:

// Mock de dependência externa
vi.mock('./external-service', () => ({
  fetchData: vi.fn()
}));

// Mock no teste
const mockFetch = vi.mocked(fetchData);
mockFetch.mockResolvedValue({ data: 'test' });
]
```

**O que Mockar:**
- [ex., "APIs externas, sistema de arquivos, banco de dados"]
- [ex., "Tempo/datas (usar vi.useFakeTimers)"]
- [ex., "Chamadas de rede (usar mock fetch)"]

**O que NÃO Mockar:**
- [ex., "Funções puras, utilitários"]
- [ex., "Lógica de negócio interna"]

## Fixtures e Factories

**Dados de Teste:**
```typescript
[Mostre o padrão para criar dados de teste, ex.:

// Padrão Factory
function createTestUser(overrides?: Partial<User>): User {
  return {
    id: 'test-id',
    name: 'Usuário Teste',
    email: 'teste@exemplo.com',
    ...overrides
  };
}

// Arquivo de Fixture
// tests/fixtures/users.ts
export const mockUsers = [/* ... */];
]
```

**Localização:**
- [ex., "tests/fixtures/ para fixtures compartilhadas"]
- [ex., "funções factory no arquivo de teste ou tests/factories/"]

## Cobertura

**Requisitos:**
- [Meta: ex., "80% de cobertura de linhas", "sem meta específica"]
- [Aplicação: ex., "CI bloqueia <80%", "cobertura apenas para consciência"]

**Configuração:**
- [Ferramenta: ex., "cobertura built-in via flag --coverage"]
- [Exclusões: ex., "excluir *.test.ts, arquivos de config"]

**Visualizar Cobertura:**
```bash
[ex., "npm run test:coverage"]
[ex., "open coverage/index.html"]
```

## Tipos de Teste

**Testes Unitários:**
- [Escopo: ex., "testar uma única função/classe isoladamente"]
- [Mocking: ex., "mockar todas as dependências externas"]
- [Velocidade: ex., "deve executar em <1s por teste"]

**Testes de Integração:**
- [Escopo: ex., "testar múltiplos módulos juntos"]
- [Mocking: ex., "mockar serviços externos, usar módulos internos reais"]
- [Setup: ex., "usar banco de teste, semear dados"]

**Testes E2E:**
- [Framework: ex., "Playwright para E2E"]
- [Escopo: ex., "testar fluxos completos do usuário"]
- [Localização: ex., "diretório e2e/ separado dos testes unitários"]

## Padrões Comuns

**Teste Assíncrono:**
```typescript
[Mostre o padrão, ex.:

it('deve tratar operação assíncrona', async () => {
  const result = await asyncFunction();
  expect(result).toBe('esperado');
});
]
```

**Teste de Erro:**
```typescript
[Mostre o padrão, ex.:

it('deve lançar em entrada inválida', () => {
  expect(() => functionCall()).toThrow('mensagem de erro');
});

// Erro assíncrono
it('deve rejeitar em falha', async () => {
  await expect(asyncCall()).rejects.toThrow('mensagem de erro');
});
]
```

**Teste de Snapshot:**
- [Uso: ex., "apenas para componentes React" ou "não usado"]
- [Localização: ex., "diretório __snapshots__/"]

---

*Análise de testes: [data]*
*Atualize quando padrões de teste mudarem*
```

<good_examples>
```markdown
# Padrões de Teste

**Data da Análise:** 2025-01-20

## Framework de Teste

**Runner:**
- Vitest 1.0.4
- Config: vitest.config.ts na raiz do projeto

**Biblioteca de Asserção:**
- expect built-in do Vitest
- Matchers: toBe, toEqual, toThrow, toMatchObject

**Comandos de Execução:**
```bash
npm test                              # Executar todos os testes
npm test -- --watch                   # Modo watch
npm test -- path/to/file.test.ts     # Arquivo único
npm run test:coverage                 # Relatório de cobertura
```

## Organização dos Arquivos de Teste

**Localização:**
- *.test.ts junto com arquivos fonte
- Sem diretório tests/ separado

**Nomenclatura:**
- nome-da-unidade.test.ts para todos os testes
- Sem distinção entre unitário/integração no nome do arquivo

**Estrutura:**
```
src/
  lib/
    parser.ts
    parser.test.ts
  services/
    install-service.ts
    install-service.test.ts
  bin/
    install.ts
    (sem teste - testado via integração CLI)
```

## Estrutura do Teste

**Organização de Suite:**
```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';

describe('NomeDoModulo', () => {
  describe('nomeDaFuncao', () => {
    beforeEach(() => {
      // resetar estado
    });

    it('deve tratar entrada válida', () => {
      // arrange
      const input = createTestInput();

      // act
      const result = functionName(input);

      // assert
      expect(result).toEqual(expectedOutput);
    });

    it('deve lançar em entrada inválida', () => {
      expect(() => functionName(null)).toThrow('Entrada inválida');
    });
  });
});
```

**Padrões:**
- Usar beforeEach para setup por-teste, evitar beforeAll
- Usar afterEach para restaurar mocks: vi.restoreAllMocks()
- Comentários explícitos arrange/act/assert em testes complexos
- Um foco de asserção por teste (mas múltiplos expects são OK)

## Mocking

**Framework:**
- Mocking built-in do Vitest (vi)
- Mock de módulo via vi.mock() no topo do arquivo de teste

**Padrões:**
```typescript
import { vi } from 'vitest';
import { externalFunction } from './external';

// Mock do módulo
vi.mock('./external', () => ({
  externalFunction: vi.fn()
}));

describe('suite de teste', () => {
  it('mocka função', () => {
    const mockFn = vi.mocked(externalFunction);
    mockFn.mockReturnValue('resultado mockado');

    // código do teste usando função mockada

    expect(mockFn).toHaveBeenCalledWith('arg esperado');
  });
});
```

**O que Mockar:**
- Operações de sistema de arquivos (fs-extra)
- Execução de child process (child_process.exec)
- Chamadas externas de API
- Variáveis de ambiente (process.env)

**O que NÃO Mockar:**
- Funções puras internas
- Utilitários simples (manipulação de string, helpers de array)
- Tipos TypeScript

## Fixtures e Factories

**Dados de Teste:**
```typescript
// Funções factory no arquivo de teste
function createTestConfig(overrides?: Partial<Config>): Config {
  return {
    targetDir: '/tmp/test',
    global: false,
    ...overrides
  };
}

// Fixtures compartilhadas em tests/fixtures/
// tests/fixtures/sample-command.md
export const sampleCommand = `---
description: Comando de teste
---
Conteúdo aqui`;
```

**Localização:**
- Funções factory: definir no arquivo de teste próximo ao uso
- Fixtures compartilhadas: tests/fixtures/ (para dados de teste multi-arquivo)
- Dados mock: inline no teste quando simples, factory quando complexo

## Cobertura

**Requisitos:**
- Sem meta de cobertura imposta
- Cobertura rastreada para consciência
- Foco em caminhos críticos (parsers, lógica de serviço)

**Configuração:**
- Cobertura do Vitest via c8 (built-in)
- Exclui: *.test.ts, bin/install.ts, arquivos de config

**Visualizar Cobertura:**
```bash
npm run test:coverage
open coverage/index.html
```

## Tipos de Teste

**Testes Unitários:**
- Testar uma única função isoladamente
- Mockar todas as dependências externas (fs, child_process)
- Rápido: cada teste <100ms
- Exemplos: parser.test.ts, validator.test.ts

**Testes de Integração:**
- Testar múltiplos módulos juntos
- Mockar apenas fronteiras externas (sistema de arquivos, process)
- Exemplos: install-service.test.ts (testa serviço + parser)

**Testes E2E:**
- Não utilizados atualmente
- Integração CLI testada manualmente

## Padrões Comuns

**Teste Assíncrono:**
```typescript
it('deve tratar operação assíncrona', async () => {
  const result = await asyncFunction();
  expect(result).toBe('esperado');
});
```

**Teste de Erro:**
```typescript
it('deve lançar em entrada inválida', () => {
  expect(() => parse(null)).toThrow('Não é possível parsear null');
});

// Erro assíncrono
it('deve rejeitar quando arquivo não encontrado', async () => {
  await expect(readConfig('invalid.txt')).rejects.toThrow('ENOENT');
});
```

**Mock de Sistema de Arquivos:**
```typescript
import { vi } from 'vitest';
import * as fs from 'fs-extra';

vi.mock('fs-extra');

it('mocka sistema de arquivos', () => {
  vi.mocked(fs.readFile).mockResolvedValue('conteúdo do arquivo');
  // código do teste
});
```

**Teste de Snapshot:**
- Não usado neste código
- Preferir asserções explícitas para clareza

---

*Análise de testes: 2025-01-20*
*Atualize quando padrões de teste mudarem*
```
</good_examples>

<guidelines>
**O que pertence ao TESTING.md:**
- Configuração de framework e runner de testes
- Padrões de localização e nomenclatura de arquivos de teste
- Estrutura de testes (describe/it, padrões beforeEach)
- Abordagem e exemplos de mocking
- Padrões de fixture/factory
- Requisitos de cobertura
- Como executar testes (comandos)
- Padrões comuns de teste no código real

**O que NÃO pertence aqui:**
- Casos de teste específicos (defira para arquivos de teste reais)
- Escolhas de tecnologia (isso é STACK.md)
- Configuração de CI/CD (isso é docs de deploy)

**Ao preencher este template:**
- Verifique scripts do package.json para comandos de teste
- Encontre o arquivo de config de teste (jest.config.js, vitest.config.ts)
- Leia 3-5 arquivos de teste existentes para identificar padrões
- Procure utilitários de teste em tests/ ou test-utils/
- Verifique configuração de cobertura
- Documente padrões reais usados, não padrões ideais

**Útil para planejamento de fase quando:**
- Adicionando novas funcionalidades (escrever testes que combinam)
- Refatorando (manter padrões de teste)
- Corrigindo bugs (adicionar testes de regressão)
- Entendendo abordagem de verificação
- Configurando infraestrutura de testes

**Abordagem de análise:**
- Verificar package.json para framework de teste e scripts
- Ler arquivo de config de teste para cobertura, setup
- Examinar organização de arquivos de teste (colocados junto vs separados)
- Revisar 5 arquivos de teste para padrões (mocking, estrutura, asserções)
- Procurar utilitários, fixtures, factories de teste
- Notar quaisquer tipos de teste (unitário, integração, e2e)
- Documentar comandos para executar testes
</guidelines>
