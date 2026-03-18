# DevSquad MCP Check — `_devsquad/core/mcp-check.md`

Este arquivo é lido pelo runner antes de qualquer execução do DevSquad.
Define quais MCPs são necessários, como verificar se estão ativos e o que fazer quando estão ausentes.

---

## MCPs necessários para o DevSquad

| MCP | Status | Usado por | Se ausente |
|-----|--------|-----------|------------|
| `context7` | OBRIGATÓRIO | César, Camila, Diana, Giovana, Rafael | bloquear + instruir instalação |
| `playwright` | OBRIGATÓRIO | Nadia (validação DX), César (testes) | bloquear + instruir instalação |
| `MCP_DOCKER` | OPCIONAL | todos (gerenciamento de ambiente) | avisar, não bloquear |

---

## Como o runner verifica os MCPs

O runner executa este check **antes de carregar Lucas** (antes do Passo 0 do SKILL.md).

### Verificação

```
PASSO 0-PRE — MCP Check

Lucas verifica se os MCPs obrigatórios estão disponíveis no contexto do Cursor:

1. Verificar context7:
   Tentar: resolve-library-id com query "test"
   SE funcionar → context7 ativo
   SE falhar   → context7 ausente

2. Verificar playwright:
   Verificar se o servidor playwright está listado nas ferramentas disponíveis
   SE disponível → playwright ativo
   SE ausente   → playwright ausente

3. Verificar MCP_DOCKER (opcional):
   Verificar se MCP_DOCKER está listado
   SE disponível → registrar como disponível para uso contextual
   SE ausente   → registrar como indisponível (não bloqueia)
```

### Comportamento por resultado

**Cenário A — Todos os obrigatórios ativos:**
→ Continuar normalmente para o Passo 0 do SKILL.md

**Cenário B — Context7 ausente:**
→ Exibir mensagem de bloqueio:

```
DevSquad não pode iniciar.

Context7 MCP está ausente — ele é obrigatório para que os agentes
usem documentação atualizada das bibliotecas do seu projeto.

Para instalar:
  1. Acesse context7.com/dashboard e crie uma conta gratuita
  2. Gere uma chave de API no dashboard
  3. Adicione ao ~/.cursor/mcp.json:

  {
    "mcpServers": {
      "context7": {
        "url": "https://mcp.context7.com/mcp",
        "headers": { "CONTEXT7_API_KEY": "SUA_CHAVE_AQUI" }
      }
    }
  }

  4. Reinicie o Cursor
  5. Consulte _devsquad/PREREQUISITES.md para instruções detalhadas
```

**Cenário C — Playwright ausente:**
→ Exibir mensagem de bloqueio:

```
DevSquad não pode iniciar.

Playwright MCP está ausente — ele é obrigatório para que os agentes
possam validar interfaces e executar testes no browser.

Para instalar, adicione ao ~/.cursor/mcp.json:

  {
    "mcpServers": {
      "playwright": {
        "command": "npx",
        "args": ["-y", "@playwright/mcp@latest"]
      }
    }
  }

Depois reinicie o Cursor.
Consulte _devsquad/PREREQUISITES.md para instruções detalhadas.
```

**Cenário D — Ambos ausentes:**
→ Exibir ambas as mensagens juntas, com link para PREREQUISITES.md

**Cenário E — Apenas Docker ausente:**
→ Avisar sem bloquear:

```
Docker MCP Toolkit não detectado.
Isso é opcional — o DevSquad funciona normalmente sem ele.
Se quiser gerenciar MCPs com interface gráfica, consulte _devsquad/PREREQUISITES.md.
```

---

## Como os agentes usam cada MCP

### Context7 — usado por todos os membros da equipe

Quando um membro propõe código que usa uma biblioteca externa:

```
Padrão de uso automático:
  1. Identificar a biblioteca no código ou no contexto
     ex: "Supabase", "React", "TypeScript", "Prisma"
  2. Chamar context7 resolve-library-id para obter o ID correto
  3. Chamar context7 query-docs com o ID e a query específica
  4. Usar a documentação retornada para embasar a proposta de código

Exemplos de quando acionar:
  César     → propondo código TypeScript com padrões de alguma lib
  Camila    → definindo ports e adapters com Supabase
  Diana     → modelando domain events com estrutura de TypeScript
  Giovana   → aplicando padrão Strategy ou Observer em TypeScript
  Rafael    → avaliando padrões arquiteturais com stack específica
  Nadia     → verificando affordances de APIs públicas de uma lib
```

### Playwright — usado por Nadia e César

```
Nadia — validação de DX (experiência do desenvolvedor):
  Quando a interface de um componente React é proposta ou refatorada,
  Nadia pode usar Playwright para:
    - Navegar até o componente em dev server
    - Verificar acessibilidade e affordances reais
    - Tomar screenshot para comparar before/after
    - Validar que affordances estão funcionando como esperado

César — validação de testes:
  Quando refatoração envolve testes Playwright existentes,
  César pode usar Playwright para:
    - Rodar os testes de caracterização antes de refatorar
    - Confirmar que testes passam após cada passo atômico
    - Gerar novos testes baseados no comportamento observado
```

### Docker MCP Toolkit — uso contextual

```
Quando disponível, Lucas pode sugerir ao usuário:
  - Adicionar MCPs complementares via catálogo Docker
    ex: "Quer adicionar o GitHub MCP para Lucas criar issues
        diretamente do plano de refatoração?"
  - Verificar se há MCPs relevantes para o projeto no catálogo
    ex: banco de dados, monitoramento, CI/CD
```

---

## Localização do arquivo de instruções completo

```
_devsquad/PREREQUISITES.md
```

Este arquivo contém as instruções passo a passo para cada MCP, incluindo:
- Links de cadastro e geração de chaves
- Limites de uso da versão gratuita
- Configuração completa do `~/.cursor/mcp.json`
- Troubleshooting de problemas comuns
