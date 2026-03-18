# DevSquad — Pré-requisitos de MCPs

Antes de usar o DevSquad, você precisa instalar e configurar os MCPs abaixo.
Eles estendem as capacidades do Cursor Agent com documentação atualizada, automação de browser e gerenciamento seguro de servidores.

> **Sessão e HANDOFF:** O DevSquad opera dentro de uma sessão do Cursor. Se você fechar o chat, o contexto do HANDOFF é perdido — uma nova sessão recomeça do início.

---

## Resumo rápido

| MCP | Obrigatoriedade | Chave de API | Custo |
|-----|----------------|--------------|-------|
| Context7 | OBRIGATÓRIO | Gratuita (opcional, mas recomendada) | Free |
| Playwright | OBRIGATÓRIO | Não precisa | Free |
| Docker MCP Toolkit | OPCIONAL | Não precisa (usa Docker Desktop) | Free pessoal |

---

## 1. Context7 MCP — OBRIGATÓRIO

### Para que serve

Context7 resolve um problema crítico do Cursor Agent: LLMs são treinados com dados antigos.
Quando você pede para o agente usar `Supabase`, `React 18`, `TypeScript 5` ou qualquer outra biblioteca,
ele pode gerar código baseado em versões desatualizadas, APIs que mudaram ou funções que não existem mais.

Context7 intercepta as perguntas de código, busca a documentação **atualizada e específica da versão** diretamente
da fonte, e injeta essa documentação no contexto do agente antes de ele responder.

Resultado: zero código desatualizado, zero APIs alucinadas, zero exemplos de versões antigas.

Para o DevSquad especificamente, Context7 é usado pelos agentes (especialmente César, Camila e Diana)
ao propor código TypeScript, configurações do Supabase ou padrões do React — garantindo que as sugestões
reflitam a API real da versão que você usa.

### Limites da versão gratuita

- **Sem chave de API:** funciona, mas com rate limits baixos (pode falhar em uso intenso)
- **Com chave de API gratuita:** rate limits mais altos — suficiente para uso normal de desenvolvimento
- Chave de API gratuita não tem custo — basta criar uma conta

### Como obter a chave de API gratuita

1. Acesse **[context7.com/dashboard](https://context7.com/dashboard)**
2. Crie uma conta (Google OAuth ou e-mail)
3. No dashboard, localize a seção **API Keys**
4. Clique em **Create API Key** e copie a chave gerada

### Como instalar no Cursor

**Opção A — HTTP (recomendado para Cursor):**

Abra ou crie o arquivo `~/.cursor/mcp.json` (configuração global) e adicione:

```json
{
  "mcpServers": {
    "context7": {
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "SUA_CHAVE_AQUI"
      }
    }
  }
}
```

**Opção B — npx (alternativa local):**

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp", "--api-key", "SUA_CHAVE_AQUI"]
    }
  }
}
```

**Pré-requisito:** Node.js 18+ instalado (`node -v` para verificar).

### Regra recomendada para o Cursor

Adicione esta regra em **Cursor → Settings → Rules** para que Context7 seja acionado automaticamente:

```
Always use Context7 MCP when I need library/API documentation, code generation,
setup or configuration steps, or library/API documentation — without me having to explicitly ask.
Automatically use the Context7 MCP tools to resolve library ID and get library docs.
```

### Verificar instalação

No Cursor, peça: *"use context7 to get the latest Supabase docs"*
Se retornar documentação atualizada, está funcionando.

---

## 2. Playwright MCP — OBRIGATÓRIO

### Para que serve

Playwright MCP fornece automação real de browser para o Cursor Agent.
Em vez de apenas escrever código de teste, o agente pode **executar** e **observar** sessões de browser ao vivo.

Para o DevSquad, Playwright é usado quando os agentes precisam:
- Validar componentes React que foram criados ou refatorados
- Executar testes e2e para verificar que refatorações não quebraram nada
- Inspecionar o DOM de uma UI para análise de experiência (Nadia)
- Gerar testes Playwright baseados na interação real com a interface

**Não precisa de chave de API.** É completamente gratuito e open source.

### Como instalar no Cursor

**Opção recomendada — Microsoft Playwright MCP (oficial):**

Adicione ao seu `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

**Alternativa — ExecuteAutomation (mais ferramentas):**

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@executeautomation/playwright-mcp-server"]
    }
  }
}
```

> Use a versão da Microsoft para projetos que já usam Playwright nos testes.
> Use a ExecuteAutomation se quiser mais ferramentas de scraping e automação.

**Pré-requisito:** Node.js 18+ instalado. Os browsers (Chromium, Firefox, WebKit) são instalados automaticamente
na primeira execução — não precisa fazer nada.

### Instalar browsers manualmente (se preferir)

```bash
npx playwright install
```

### Verificar instalação

No Cursor, peça: *"use playwright to open google.com and take a screenshot"*
Se uma janela do browser abrir e retornar um screenshot, está funcionando.

---

## 3. Docker MCP Toolkit — OPCIONAL

### Para que serve

Docker MCP Toolkit é um gerenciador de MCPs integrado ao Docker Desktop.
Em vez de configurar cada MCP manualmente em JSON, você tem uma interface gráfica com
um catálogo de 300+ servidores verificados — instalação com um clique.

**Por que é opcional para o DevSquad:**
- Os agentes do DevSquad funcionam completamente sem ele
- Útil se você quiser adicionar outros MCPs ao seu workflow de desenvolvimento
  (GitHub, Postgres, Stripe, Notion, etc.) de forma segura e centralizada
- Especialmente útil para isolar os MCPs em containers, evitando conflitos de dependência

**Não precisa de chave de API separada.** É uma feature gratuita do Docker Desktop.

### Limites / custo

- **Docker Desktop Personal:** gratuito para uso pessoal, educação e empresas pequenas
  (menos de 250 funcionários e menos de USD 10M de faturamento)
- **MCP Toolkit:** feature gratuita incluída no Docker Desktop — sem custo adicional
- Alguns MCPs do catálogo podem requerer chaves de API dos serviços que integram
  (ex: GitHub MCP precisa de token do GitHub)

### Como instalar

1. Baixe e instale **[Docker Desktop](https://www.docker.com/products/docker-desktop/)** (versão 4.40+)
2. Abra Docker Desktop
3. Acesse **Settings → Features in Development**
4. Ative **Docker MCP Toolkit**
5. Clique em **Apply**
6. A seção **MCP Toolkit** aparecerá na sidebar do Docker Desktop

### Como conectar ao Cursor

1. No Docker Desktop, abra **MCP Toolkit**
2. Vá para a aba **Clients**
3. Localize **Cursor** na lista
4. Clique em **Connect**

O Docker Desktop configura automaticamente o MCP Gateway no Cursor.
Todos os servidores que você adicionar ao perfil ficam disponíveis automaticamente.

### Verificar instalação

No Cursor, verifique em **Settings → Tools & MCP** se `MCP_DOCKER` aparece como servidor instalado.

---

## Configuração final do `~/.cursor/mcp.json`

Com Context7 e Playwright configurados (Docker é opcional e se configura automaticamente), seu arquivo ficará assim:

```json
{
  "mcpServers": {
    "context7": {
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "SUA_CHAVE_CONTEXT7_AQUI"
      }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

> Se estiver usando Docker MCP Toolkit, o `MCP_DOCKER` é adicionado automaticamente
> pelo Docker Desktop — não precisa adicionar manualmente ao JSON.

---

## Checklist de verificação

```
[ ] Node.js 18+ instalado  (node -v)
[ ] Context7: chave de API criada em context7.com/dashboard
[ ] Context7: configurado em ~/.cursor/mcp.json
[ ] Context7: regra automática adicionada no Cursor Settings → Rules
[ ] Playwright: configurado em ~/.cursor/mcp.json
[ ] Playwright: browsers instalados (automático na primeira execução)
[ ] Docker Desktop: instalado e MCP Toolkit ativado  [OPCIONAL]
[ ] Cursor reiniciado após configurar o mcp.json
```

---

## Troubleshooting rápido

**Context7 retorna erro de rate limit:**
→ Confirme que a `CONTEXT7_API_KEY` está correta no mcp.json
→ Verifique se a chave foi criada em context7.com/dashboard

**Playwright não abre browser:**
→ Rode `npx playwright install` manualmente no terminal
→ Confirme que Node.js 18+ está instalado

**Docker MCP Toolkit não aparece no Cursor:**
→ Feche e reabra o Cursor após conectar no Docker Desktop
→ Verifique Settings → Tools & MCP para confirmar se `MCP_DOCKER` está listado

**Cursor não encontra o mcp.json:**
→ O arquivo global fica em `~/.cursor/mcp.json` (macOS/Linux) ou `%USERPROFILE%\.cursor\mcp.json` (Windows)
→ Para configuração por projeto: `.cursor/mcp.json` na raiz do projeto
