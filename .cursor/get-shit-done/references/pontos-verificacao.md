<overview>
Planos executam autonomamente. Pontos de verificação formalizam pontos de interação onde verificação humana ou decisões são necessárias.

**Princípio central:** Claude automatiza tudo com CLI/API. Pontos de verificação são para verificação e decisões, não trabalho manual.

**Regras de ouro:**
1. **Se Claude pode executar, Claude executa** - Nunca peça ao usuário para executar comandos CLI, iniciar servidores ou rodar builds
2. **Claude configura o ambiente de verificação** - Inicia servidores dev, popula bancos de dados, configura variáveis de ambiente
3. **Usuário só faz o que requer julgamento humano** - Verificações visuais, avaliação de UX, "isso parece correto?"
4. **Segredos vêm do usuário, automação vem do Claude** - Peça chaves de API, depois Claude as usa via CLI
5. **Modo auto ignora pontos de verificação/decisão** — Quando `workflow._auto_chain_active` ou `workflow.auto_advance` é true na config: human-verify auto-aprova, decision auto-seleciona primeira opção, human-action ainda para (gates de autenticação não podem ser automatizados)
</overview>

<checkpoint_types>

<type name="human-verify">
## checkpoint:human-verify (Mais Comum - 90%)

**Quando:** Claude completou trabalho automatizado, humano confirma que funciona corretamente.

**Usar para:**
- Verificações visuais de UI (layout, estilização, responsividade)
- Fluxos interativos (clicar em wizard, testar fluxos do usuário)
- Verificação funcional (funcionalidade funciona como esperado)
- Qualidade de reprodução de áudio/vídeo
- Suavidade de animações
- Testes de acessibilidade

**Estrutura:**
```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[O que Claude automatizou e implantou/construiu]</what-built>
  <how-to-verify>
    [Passos exatos para testar - URLs, comandos, comportamento esperado]
  </how-to-verify>
  <resume-signal>[Como continuar - "aprovado", "sim", ou descreva problemas]</resume-signal>
</task>
```

**Exemplo: Componente UI (mostra padrão chave: Claude inicia servidor ANTES do ponto de verificação)**
```xml
<task type="auto">
  <name>Construir layout de dashboard responsivo</name>
  <files>src/components/Dashboard.tsx, src/app/dashboard/page.tsx</files>
  <action>Criar dashboard com sidebar, header e área de conteúdo. Usar classes responsivas do Tailwind para mobile.</action>
  <verify>npm run build funciona, sem erros TypeScript</verify>
  <done>Componente Dashboard compila sem erros</done>
</task>

<task type="auto">
  <name>Iniciar servidor dev para verificação</name>
  <action>Executar `npm run dev` em background, aguardar mensagem "ready", capturar porta</action>
  <verify>fetch http://localhost:3000 retorna 200</verify>
  <done>Servidor dev rodando em http://localhost:3000</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Layout de dashboard responsivo - servidor dev rodando em http://localhost:3000</what-built>
  <how-to-verify>
    Visite http://localhost:3000/dashboard e verifique:
    1. Desktop (>1024px): Sidebar à esquerda, conteúdo à direita, header no topo
    2. Tablet (768px): Sidebar colapsa para menu hamburger
    3. Mobile (375px): Layout de coluna única, navegação inferior aparece
    4. Sem deslocamento de layout ou scroll horizontal em qualquer tamanho
  </how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva problemas de layout</resume-signal>
</task>
```

**Exemplo: Build Xcode**
```xml
<task type="auto">
  <name>Compilar app macOS com Xcode</name>
  <files>App.xcodeproj, Sources/</files>
  <action>Executar `xcodebuild -project App.xcodeproj -scheme App build`. Verificar erros de compilação na saída.</action>
  <verify>Saída do build contém "BUILD SUCCEEDED", sem erros</verify>
  <done>App compila com sucesso</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>App macOS compilado em DerivedData/Build/Products/Debug/App.app</what-built>
  <how-to-verify>
    Abra App.app e teste:
    - App inicia sem crashes
    - Ícone da barra de menu aparece
    - Janela de preferências abre corretamente
    - Sem falhas visuais ou problemas de layout
  </how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva problemas</resume-signal>
</task>
```
</type>

<type name="decision">
## checkpoint:decision (9%)

**Quando:** Humano deve fazer escolha que afeta a direção da implementação.

**Usar para:**
- Seleção de tecnologia (qual provedor de auth, qual banco de dados)
- Decisões de arquitetura (monorepo vs repositórios separados)
- Escolhas de design (esquema de cores, abordagem de layout)
- Priorização de funcionalidades (qual variante construir)
- Decisões de modelo de dados (estrutura do schema)

**Estrutura:**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>[O que está sendo decidido]</decision>
  <context>[Por que esta decisão importa]</context>
  <options>
    <option id="option-a">
      <name>[Nome da opção]</name>
      <pros>[Benefícios]</pros>
      <cons>[Desvantagens]</cons>
    </option>
    <option id="option-b">
      <name>[Nome da opção]</name>
      <pros>[Benefícios]</pros>
      <cons>[Desvantagens]</cons>
    </option>
  </options>
  <resume-signal>[Como indicar a escolha]</resume-signal>
</task>
```

**Exemplo: Seleção de Provedor de Auth**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Selecionar provedor de autenticação</decision>
  <context>
    Necessita de autenticação de usuário para o app. Três opções sólidas com diferentes desvantagens.
  </context>
  <options>
    <option id="supabase">
      <name>Supabase Auth</name>
      <pros>Integrado com Supabase DB que estamos usando, plano gratuito generoso, integração com row-level security</pros>
      <cons>UI menos customizável, preso ao ecossistema Supabase</cons>
    </option>
    <option id="clerk">
      <name>Clerk</name>
      <pros>UI pré-construída bonita, melhor experiência de desenvolvedor, documentação excelente</pros>
      <cons>Pago após 10k MAU, vendor lock-in</cons>
    </option>
    <option id="nextauth">
      <name>NextAuth.js</name>
      <pros>Gratuito, auto-hospedado, controle máximo, amplamente adotado</pros>
      <cons>Mais trabalho de setup, você gerencia atualizações de segurança, UI é faça você mesmo</cons>
    </option>
  </options>
  <resume-signal>Selecione: supabase, clerk, ou nextauth</resume-signal>
</task>
```

**Exemplo: Seleção de Banco de Dados**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Selecionar banco de dados para dados de usuário</decision>
  <context>
    App precisa de armazenamento persistente para usuários, sessões e conteúdo gerado pelo usuário.
    Escala esperada: 10k usuários, 1M registros no primeiro ano.
  </context>
  <options>
    <option id="supabase">
      <name>Supabase (Postgres)</name>
      <pros>SQL completo, plano gratuito generoso, auth integrado, assinaturas em tempo real</pros>
      <cons>Vendor lock-in para funcionalidades em tempo real, menos flexível que Postgres puro</cons>
    </option>
    <option id="planetscale">
      <name>PlanetScale (MySQL)</name>
      <pros>Escalonamento serverless, fluxo de trabalho com branching, excelente DX</pros>
      <cons>MySQL não Postgres, sem foreign keys no plano gratuito</cons>
    </option>
    <option id="convex">
      <name>Convex</name>
      <pros>Tempo real por padrão, nativo TypeScript, cache automático</pros>
      <cons>Plataforma mais nova, modelo mental diferente, menos flexibilidade SQL</cons>
    </option>
  </options>
  <resume-signal>Selecione: supabase, planetscale, ou convex</resume-signal>
</task>
```
</type>

<type name="human-action">
## checkpoint:human-action (1% - Raro)

**Quando:** Ação NÃO tem CLI/API e requer interação exclusivamente humana, OU Claude encontrou um gate de autenticação durante a automação.

**Usar SOMENTE para:**
- **Gates de autenticação** - Claude tentou CLI/API mas precisa de credenciais (isso NÃO é uma falha)
- Links de verificação por email (clicar no email)
- Códigos 2FA por SMS (verificação por telefone)
- Aprovações manuais de conta (plataforma requer revisão humana)
- Fluxos 3D Secure de cartão de crédito (autorização de pagamento baseada em web)
- Aprovações de app OAuth (aprovação baseada em web)

**NÃO usar para trabalho manual pré-planejado:**
- Deploy (usar CLI - gate de auth se necessário)
- Criar webhooks/bancos de dados (usar API/CLI - gate de auth se necessário)
- Rodar builds/testes (usar ferramenta Bash)
- Criar arquivos (usar ferramenta Write)

**Estrutura:**
```xml
<task type="checkpoint:human-action" gate="blocking">
  <action>[O que humano deve fazer - Claude já fez tudo automatizável]</action>
  <instructions>
    [O que Claude já automatizou]
    [A ÚNICA coisa que requer ação humana]
  </instructions>
  <verification>[O que Claude pode verificar depois]</verification>
  <resume-signal>[Como continuar]</resume-signal>
</task>
```

**Exemplo: Verificação de Email**
```xml
<task type="auto">
  <name>Criar conta SendGrid via API</name>
  <action>Usar API SendGrid para criar conta de subusuário com email fornecido. Solicitar email de verificação.</action>
  <verify>API retorna 201, conta criada</verify>
  <done>Conta criada, email de verificação enviado</done>
</task>

<task type="checkpoint:human-action" gate="blocking">
  <action>Completar verificação de email para conta SendGrid</action>
  <instructions>
    Eu criei a conta e solicitei o email de verificação.
    Verifique sua caixa de entrada para o link de verificação do SendGrid e clique nele.
  </instructions>
  <verification>Chave de API do SendGrid funciona: teste curl funciona</verification>
  <resume-signal>Digite "pronto" quando email verificado</resume-signal>
</task>
```

**Exemplo: Gate de Autenticação (Ponto de Verificação Dinâmico)**
```xml
<task type="auto">
  <name>Deploy no Vercel</name>
  <files>.vercel/, vercel.json</files>
  <action>Executar `vercel --yes` para deploy</action>
  <verify>vercel ls mostra deployment, fetch retorna 200</verify>
</task>

<!-- Se vercel retorna "Error: Not authenticated", Claude cria ponto de verificação dinamicamente -->

<task type="checkpoint:human-action" gate="blocking">
  <action>Autenticar CLI do Vercel para que eu possa continuar o deployment</action>
  <instructions>
    Eu tentei fazer deploy mas recebi erro de autenticação.
    Execute: vercel login
    Isso abrirá seu navegador - complete o fluxo de autenticação.
  </instructions>
  <verification>vercel whoami retorna seu email da conta</verification>
  <resume-signal>Digite "pronto" quando autenticado</resume-signal>
</task>

<!-- Após autenticação, Claude tenta novamente o deployment -->

<task type="auto">
  <name>Tentar novamente deployment no Vercel</name>
  <action>Executar `vercel --yes` (agora autenticado)</action>
  <verify>vercel ls mostra deployment, fetch retorna 200</verify>
</task>
```

**Distinção chave:** Gates de auth são criados dinamicamente quando Claude encontra erros de auth. NÃO pré-planejados — Claude automatiza primeiro, pede credenciais somente quando bloqueado.
</type>
</checkpoint_types>

<execution_protocol>

Quando Claude encontra `type="checkpoint:*"`:

1. **Parar imediatamente** - não prosseguir para próxima tarefa
2. **Exibir ponto de verificação claramente** usando o formato abaixo
3. **Aguardar resposta do usuário** - não alucinar conclusão
4. **Verificar se possível** - checar arquivos, rodar testes, o que for especificado
5. **Retomar execução** - continuar para próxima tarefa somente após confirmação

**Para checkpoint:human-verify:**
```
╔═══════════════════════════════════════════════════════╗
║  PONTO DE VERIFICAÇÃO: Verificação Necessária         ║
╚═══════════════════════════════════════════════════════╝

Progresso: 5/8 tarefas completas
Tarefa: Layout de dashboard responsivo

Construído: Dashboard responsivo em /dashboard

Como verificar:
  1. Visite: http://localhost:3000/dashboard
  2. Desktop (>1024px): Sidebar visível, conteúdo preenche espaço restante
  3. Tablet (768px): Sidebar colapsa para ícones
  4. Mobile (375px): Sidebar escondida, menu hamburger aparece

────────────────────────────────────────────────────────
→ SUA AÇÃO: Digite "aprovado" ou descreva problemas
────────────────────────────────────────────────────────
```

**Para checkpoint:decision:**
```
╔═══════════════════════════════════════════════════════╗
║  PONTO DE VERIFICAÇÃO: Decisão Necessária             ║
╚═══════════════════════════════════════════════════════╝

Progresso: 2/6 tarefas completas
Tarefa: Selecionar provedor de autenticação

Decisão: Qual provedor de auth devemos usar?

Contexto: Necessita de autenticação de usuário. Três opções com diferentes desvantagens.

Opções:
  1. supabase - Integrado com nosso DB, plano gratuito
     Prós: Integração com row-level security, plano gratuito generoso
     Contras: UI menos customizável, lock-in no ecossistema

  2. clerk - Melhor DX, pago após 10k usuários
     Prós: UI pré-construída bonita, documentação excelente
     Contras: Vendor lock-in, preço em escala

  3. nextauth - Auto-hospedado, controle máximo
     Prós: Gratuito, sem vendor lock-in, amplamente adotado
     Contras: Mais trabalho de setup, atualizações de segurança por conta própria

────────────────────────────────────────────────────────
→ SUA AÇÃO: Selecione supabase, clerk, ou nextauth
────────────────────────────────────────────────────────
```

**Para checkpoint:human-action:**
```
╔═══════════════════════════════════════════════════════╗
║  PONTO DE VERIFICAÇÃO: Ação Necessária                ║
╚═══════════════════════════════════════════════════════╝

Progresso: 3/8 tarefas completas
Tarefa: Deploy no Vercel

Tentado: vercel --yes
Erro: Não autenticado. Por favor execute 'vercel login'

O que você precisa fazer:
  1. Execute: vercel login
  2. Complete a autenticação no navegador quando abrir
  3. Retorne aqui quando terminar

Eu vou verificar: vercel whoami retorna sua conta

────────────────────────────────────────────────────────
→ SUA AÇÃO: Digite "pronto" quando autenticado
────────────────────────────────────────────────────────
```
</execution_protocol>

<authentication_gates>

**Gate de auth = Claude tentou CLI/API, recebeu erro de auth.** Não é uma falha — é um gate que requer input humano para desbloquear.

**Padrão:** Claude tenta automação → erro de auth → cria checkpoint:human-action → usuário autentica → Claude tenta novamente → continua

**Protocolo de gate:**
1. Reconhecer que não é uma falha - auth ausente é esperado
2. Parar tarefa atual - não tentar repetidamente
3. Criar checkpoint:human-action dinamicamente
4. Fornecer passos exatos de autenticação
5. Verificar que autenticação funciona
6. Tentar novamente a tarefa original
7. Continuar normalmente

**Distinção chave:**
- Ponto de verificação pré-planejado: "Preciso que você faça X" (errado - Claude deveria automatizar)
- Gate de auth: "Tentei automatizar X mas preciso de credenciais" (correto - desbloqueia automação)

</authentication_gates>

<automation_reference>

**A regra:** Se tem CLI/API, Claude faz. Nunca peça ao humano para realizar trabalho automatizável.

## Referência de CLI de Serviços

| Serviço | CLI/API | Comandos Chave | Gate de Auth |
|---------|---------|----------------|--------------|
| Vercel | `vercel` | `--yes`, `env add`, `--prod`, `ls` | `vercel login` |
| Railway | `railway` | `init`, `up`, `variables set` | `railway login` |
| Fly | `fly` | `launch`, `deploy`, `secrets set` | `fly auth login` |
| Stripe | `stripe` + API | `listen`, `trigger`, chamadas API | Chave API no .env |
| Supabase | `supabase` | `init`, `link`, `db push`, `gen types` | `supabase login` |
| Upstash | `upstash` | `redis create`, `redis get` | `upstash auth login` |
| PlanetScale | `pscale` | `database create`, `branch create` | `pscale auth login` |
| GitHub | `gh` | `repo create`, `pr create`, `secret set` | `gh auth login` |
| Node | `npm`/`pnpm` | `install`, `run build`, `test`, `run dev` | N/A |
| Xcode | `xcodebuild` | `-project`, `-scheme`, `build`, `test` | N/A |
| Convex | `npx convex` | `dev`, `deploy`, `env set`, `env get` | `npx convex login` |

## Automação de Variáveis de Ambiente

**Arquivos env:** Usar ferramentas Write/Edit. Nunca pedir ao humano para criar .env manualmente.

**Variáveis env do dashboard via CLI:**

| Plataforma | Comando CLI | Exemplo |
|------------|-------------|---------|
| Convex | `npx convex env set` | `npx convex env set OPENAI_API_KEY sk-...` |
| Vercel | `vercel env add` | `vercel env add STRIPE_KEY production` |
| Railway | `railway variables set` | `railway variables set API_KEY=value` |
| Fly | `fly secrets set` | `fly secrets set DATABASE_URL=...` |
| Supabase | `supabase secrets set` | `supabase secrets set MY_SECRET=value` |

**Padrão de coleta de segredos:**
```xml
<!-- ERRADO: Pedindo ao usuário para adicionar vars env no dashboard -->
<task type="checkpoint:human-action">
  <action>Adicionar OPENAI_API_KEY no dashboard do Convex</action>
  <instructions>Vá para dashboard.convex.dev → Settings → Environment Variables → Add</instructions>
</task>

<!-- CERTO: Claude pede o valor, depois adiciona via CLI -->
<task type="checkpoint:human-action">
  <action>Forneça sua chave de API do OpenAI</action>
  <instructions>
    Eu preciso da sua chave de API do OpenAI para o backend Convex.
    Obtenha em: https://platform.openai.com/api-keys
    Cole a chave (começa com sk-)
  </instructions>
  <verification>Eu vou adicionar via `npx convex env set` e verificar</verification>
  <resume-signal>Cole sua chave de API</resume-signal>
</task>

<task type="auto">
  <name>Configurar chave OpenAI no Convex</name>
  <action>Executar `npx convex env set OPENAI_API_KEY {chave-fornecida-pelo-usuario}`</action>
  <verify>`npx convex env get OPENAI_API_KEY` retorna a chave (mascarada)</verify>
</task>
```

## Automação de Servidor Dev

| Framework | Comando de Início | Sinal de Pronto | URL Padrão |
|-----------|-------------------|-----------------|------------|
| Next.js | `npm run dev` | "Ready in" ou "started server" | http://localhost:3000 |
| Vite | `npm run dev` | "ready in" | http://localhost:5173 |
| Convex | `npx convex dev` | "Convex functions ready" | N/A (somente backend) |
| Express | `npm start` | "listening on port" | http://localhost:3000 |
| Django | `python manage.py runserver` | "Starting development server" | http://localhost:8000 |

**Ciclo de vida do servidor:**
```bash
# Executar em background, capturar PID
npm run dev &
DEV_SERVER_PID=$!

# Aguardar pronto (máx 30s) — usa fetch() para compatibilidade cross-platform
timeout 30 bash -c 'until node -e "fetch(\"http://localhost:3000\").then(r=>{process.exit(r.ok?0:1)}).catch(()=>process.exit(1))" 2>/dev/null; do sleep 1; done'
```

**Conflitos de porta:** Matar processo antigo (`lsof -ti:3000 | xargs kill`) ou usar porta alternativa (`--port 3001`).

**Servidor continua rodando** através dos pontos de verificação. Só matar quando plano completo, mudando para produção, ou porta necessária para outro serviço.

## Tratamento de Instalação de CLI

| CLI | Auto-instalar? | Comando |
|-----|----------------|---------|
| npm/pnpm/yarn | Não - perguntar ao usuário | Usuário escolhe gerenciador de pacotes |
| vercel | Sim | `npm i -g vercel` |
| gh (GitHub) | Sim | `brew install gh` (macOS) ou `apt install gh` (Linux) |
| stripe | Sim | `npm i -g stripe` |
| supabase | Sim | `npm i -g supabase` |
| convex | Não - usar npx | `npx convex` (sem instalação necessária) |
| fly | Sim | `brew install flyctl` ou instalador curl |
| railway | Sim | `npm i -g @railway/cli` |

**Protocolo:** Tentar comando → "command not found" → auto-instalável? → sim: instalar silenciosamente, tentar novamente → não: ponto de verificação pedindo ao usuário para instalar.

## Falhas de Automação Pré-Ponto de Verificação

| Falha | Resposta |
|-------|----------|
| Servidor não inicia | Verificar erro, corrigir problema, tentar novamente (não prosseguir para ponto de verificação) |
| Porta em uso | Matar processo antigo ou usar porta alternativa |
| Dependência faltando | Executar `npm install`, tentar novamente |
| Erro de build | Corrigir o erro primeiro (bug, não problema de ponto de verificação) |
| Erro de auth | Criar ponto de verificação de gate de auth |
| Timeout de rede | Tentar novamente com backoff, depois ponto de verificação se persistente |

**Nunca apresente um ponto de verificação com ambiente de verificação quebrado.** Se o servidor local não está respondendo, não peça ao usuário para "visitar localhost:3000".

> **Nota cross-platform:** Use `node -e "fetch('http://localhost:3000').then(r=>console.log(r.status))"` ao invés de `curl` para verificações de saúde. `curl` tem problemas no Windows MSYS/Git Bash devido a problemas de SSL/manipulação de path.

```xml
<!-- ERRADO: Ponto de verificação com ambiente quebrado -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard (servidor falhou ao iniciar)</what-built>
  <how-to-verify>Visite http://localhost:3000...</how-to-verify>
</task>

<!-- CERTO: Corrigir primeiro, depois ponto de verificação -->
<task type="auto">
  <name>Corrigir problema de inicialização do servidor</name>
  <action>Investigar erro, corrigir causa raiz, reiniciar servidor</action>
  <verify>fetch http://localhost:3000 retorna 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Dashboard - servidor rodando em http://localhost:3000</what-built>
  <how-to-verify>Visite http://localhost:3000/dashboard...</how-to-verify>
</task>
```

## Referência Rápida de Automatizáveis

| Ação | Automatizável? | Claude faz? |
|------|----------------|-------------|
| Deploy no Vercel | Sim (`vercel`) | SIM |
| Criar webhook Stripe | Sim (API) | SIM |
| Escrever arquivo .env | Sim (ferramenta Write) | SIM |
| Criar DB Upstash | Sim (`upstash`) | SIM |
| Executar testes | Sim (`npm test`) | SIM |
| Iniciar servidor dev | Sim (`npm run dev`) | SIM |
| Adicionar vars env no Convex | Sim (`npx convex env set`) | SIM |
| Adicionar vars env no Vercel | Sim (`vercel env add`) | SIM |
| Popular banco de dados | Sim (CLI/API) | SIM |
| Clicar link de verificação de email | Não | NÃO |
| Inserir cartão de crédito com 3DS | Não | NÃO |
| Completar OAuth no navegador | Não | NÃO |
| Verificar visualmente se UI parece correta | Não | NÃO |
| Testar fluxos interativos do usuário | Não | NÃO |

</automation_reference>

<writing_guidelines>

**FAÇA:**
- Automatize tudo com CLI/API antes do ponto de verificação
- Seja específico: "Visite https://meuapp.vercel.app" não "verifique o deployment"
- Numere os passos de verificação
- Declare resultados esperados: "Você deve ver X"
- Forneça contexto: por que este ponto de verificação existe

**NÃO FAÇA:**
- Pedir ao humano para fazer trabalho que Claude pode automatizar ❌
- Assumir conhecimento: "Configure as configurações usuais" ❌
- Pular passos: "Configure o banco de dados" (muito vago) ❌
- Misturar múltiplas verificações em um ponto de verificação ❌

**Posicionamento:**
- **Após automação completar** - não antes de Claude fazer o trabalho
- **Após construção de UI** - antes de declarar fase completa
- **Antes de trabalho dependente** - decisões antes de implementação
- **Em pontos de integração** - após configurar serviços externos

**Posicionamento ruim:** Antes da automação ❌ | Muito frequente ❌ | Muito tarde (tarefas dependentes já precisavam do resultado) ❌
</writing_guidelines>

<examples>

### Exemplo 1: Setup de Banco de Dados (Sem Ponto de Verificação Necessário)

```xml
<task type="auto">
  <name>Criar banco de dados Redis no Upstash</name>
  <files>.env</files>
  <action>
    1. Executar `upstash redis create myapp-cache --region us-east-1`
    2. Capturar URL de conexão da saída
    3. Escrever em .env: UPSTASH_REDIS_URL={url}
    4. Verificar conexão com comando de teste
  </action>
  <verify>
    - upstash redis list mostra banco de dados
    - .env contém UPSTASH_REDIS_URL
    - Teste de conexão funciona
  </verify>
  <done>Banco de dados Redis criado e configurado</done>
</task>

<!-- SEM PONTO DE VERIFICAÇÃO NECESSÁRIO - Claude automatizou tudo e verificou programaticamente -->
```

### Exemplo 2: Fluxo Completo de Auth (Ponto de verificação único no final)

```xml
<task type="auto">
  <name>Criar schema de usuário</name>
  <files>src/db/schema.ts</files>
  <action>Definir tabelas User, Session, Account com Drizzle ORM</action>
  <verify>npm run db:generate funciona</verify>
</task>

<task type="auto">
  <name>Criar rotas de API de auth</name>
  <files>src/app/api/auth/[...nextauth]/route.ts</files>
  <action>Configurar NextAuth com provedor GitHub, estratégia JWT</action>
  <verify>TypeScript compila, sem erros</verify>
</task>

<task type="auto">
  <name>Criar UI de login</name>
  <files>src/app/login/page.tsx, src/components/LoginButton.tsx</files>
  <action>Criar página de login com botão OAuth do GitHub</action>
  <verify>npm run build funciona</verify>
</task>

<task type="auto">
  <name>Iniciar servidor dev para teste de auth</name>
  <action>Executar `npm run dev` em background, aguardar sinal de pronto</action>
  <verify>fetch http://localhost:3000 retorna 200</verify>
  <done>Servidor dev rodando em http://localhost:3000</done>
</task>

<!-- UM ponto de verificação no final verifica o fluxo completo -->
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Fluxo completo de autenticação - servidor dev rodando em http://localhost:3000</what-built>
  <how-to-verify>
    1. Visite: http://localhost:3000/login
    2. Clique "Entrar com GitHub"
    3. Complete o fluxo OAuth do GitHub
    4. Verifique: Redirecionado para /dashboard, nome do usuário exibido
    5. Atualize a página: Sessão persiste
    6. Clique logout: Sessão limpa
  </how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva problemas</resume-signal>
</task>
```
</examples>

<anti_patterns>

### ❌ RUIM: Pedindo ao usuário para iniciar servidor dev

```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Componente Dashboard</what-built>
  <how-to-verify>
    1. Execute: npm run dev
    2. Visite: http://localhost:3000/dashboard
    3. Verifique se layout está correto
  </how-to-verify>
</task>
```

**Por que ruim:** Claude pode executar `npm run dev`. Usuário deve apenas visitar URLs, não executar comandos.

### ✅ BOM: Claude inicia servidor, usuário visita

```xml
<task type="auto">
  <name>Iniciar servidor dev</name>
  <action>Executar `npm run dev` em background</action>
  <verify>fetch http://localhost:3000 retorna 200</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard em http://localhost:3000/dashboard (servidor rodando)</what-built>
  <how-to-verify>
    Visite http://localhost:3000/dashboard e verifique:
    1. Layout corresponde ao design
    2. Sem erros no console
  </how-to-verify>
</task>
```

### ❌ RUIM: Pedindo ao humano para fazer deploy / ✅ BOM: Claude automatiza

```xml
<!-- RUIM: Pedindo ao usuário para fazer deploy via dashboard -->
<task type="checkpoint:human-action" gate="blocking">
  <action>Deploy no Vercel</action>
  <instructions>Visite vercel.com/new → Importe repo → Clique Deploy → Copie URL</instructions>
</task>

<!-- BOM: Claude faz deploy, usuário verifica -->
<task type="auto">
  <name>Deploy no Vercel</name>
  <action>Executar `vercel --yes`. Capturar URL.</action>
  <verify>vercel ls mostra deployment, fetch retorna 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Deployed em {url}</what-built>
  <how-to-verify>Visite {url}, verifique se homepage carrega</how-to-verify>
  <resume-signal>Digite "aprovado"</resume-signal>
</task>
```

### ❌ RUIM: Muitos pontos de verificação / ✅ BOM: Ponto de verificação único

```xml
<!-- RUIM: Ponto de verificação após cada tarefa -->
<task type="auto">Criar schema</task>
<task type="checkpoint:human-verify">Verificar schema</task>
<task type="auto">Criar rota de API</task>
<task type="checkpoint:human-verify">Verificar API</task>
<task type="auto">Criar formulário UI</task>
<task type="checkpoint:human-verify">Verificar formulário</task>

<!-- BOM: Um ponto de verificação no final -->
<task type="auto">Criar schema</task>
<task type="auto">Criar rota de API</task>
<task type="auto">Criar formulário UI</task>

<task type="checkpoint:human-verify">
  <what-built>Fluxo completo de auth (schema + API + UI)</what-built>
  <how-to-verify>Testar fluxo completo: registrar, login, acessar página protegida</how-to-verify>
  <resume-signal>Digite "aprovado"</resume-signal>
</task>
```

### ❌ RUIM: Verificação vaga / ✅ BOM: Passos específicos

```xml
<!-- RUIM -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard</what-built>
  <how-to-verify>Verifique se funciona</how-to-verify>
</task>

<!-- BOM -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard responsivo - servidor rodando em http://localhost:3000</what-built>
  <how-to-verify>
    Visite http://localhost:3000/dashboard e verifique:
    1. Desktop (>1024px): Sidebar visível, área de conteúdo preenche espaço restante
    2. Tablet (768px): Sidebar colapsa para ícones
    3. Mobile (375px): Sidebar escondida, menu hamburger no header
    4. Sem scroll horizontal em qualquer tamanho
  </how-to-verify>
  <resume-signal>Digite "aprovado" ou descreva problemas de layout</resume-signal>
</task>
```

### ❌ RUIM: Pedindo ao usuário para executar comandos CLI

```xml
<task type="checkpoint:human-action">
  <action>Executar migrações do banco de dados</action>
  <instructions>Execute: npx prisma migrate deploy && npx prisma db seed</instructions>
</task>
```

**Por que ruim:** Claude pode executar esses comandos. Usuário nunca deve executar comandos CLI.

### ❌ RUIM: Pedindo ao usuário para copiar valores entre serviços

```xml
<task type="checkpoint:human-action">
  <action>Configurar URL de webhook no Stripe</action>
  <instructions>Copie URL de deployment → Stripe Dashboard → Webhooks → Add endpoint → Copie secret → Adicione ao .env</instructions>
</task>
```

**Por que ruim:** Stripe tem uma API. Claude deve criar o webhook via API e escrever no .env diretamente.

</anti_patterns>

<summary>

Pontos de verificação formalizam pontos de interação humana para verificação e decisões, não trabalho manual.

**A regra de ouro:** Se Claude PODE automatizar, Claude DEVE automatizar.

**Prioridade de pontos de verificação:**
1. **checkpoint:human-verify** (90%) - Claude automatizou tudo, humano confirma correção visual/funcional
2. **checkpoint:decision** (9%) - Humano faz escolhas arquiteturais/tecnológicas
3. **checkpoint:human-action** (1%) - Passos manuais verdadeiramente inevitáveis sem API/CLI

**Quando NÃO usar pontos de verificação:**
- Coisas que Claude pode verificar programaticamente (testes, builds)
- Operações de arquivo (Claude pode ler arquivos)
- Correção de código (testes e análise estática)
- Qualquer coisa automatizável via CLI/API
</summary>
