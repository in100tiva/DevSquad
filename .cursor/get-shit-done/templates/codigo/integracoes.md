# Template de Integrações Externas

Template para `.planning/codebase/INTEGRATIONS.md` - captura dependências de serviços externos.

**Propósito:** Documentar com quais sistemas externos este código se comunica. Focado em "o que vive fora do nosso código e do qual dependemos."

---

## Template do Arquivo

```markdown
# Integrações Externas

**Data da Análise:** [AAAA-MM-DD]

## APIs e Serviços Externos

**Processamento de Pagamentos:**
- [Serviço] - [Para que é usado: ex., "cobrança de assinatura, pagamentos únicos"]
  - SDK/Cliente: [ex., "pacote npm stripe v14.x"]
  - Autenticação: [ex., "Chave de API na variável de ambiente STRIPE_SECRET_KEY"]
  - Endpoints usados: [ex., "sessões de checkout, webhooks"]

**Email/SMS:**
- [Serviço] - [Para que é usado: ex., "emails transacionais"]
  - SDK/Cliente: [ex., "sendgrid/mail v8.x"]
  - Autenticação: [ex., "Chave de API na variável de ambiente SENDGRID_API_KEY"]
  - Templates: [ex., "gerenciados no dashboard do SendGrid"]

**APIs Externas:**
- [Serviço] - [Para que é usado]
  - Método de integração: [ex., "API REST via fetch", "Cliente GraphQL"]
  - Autenticação: [ex., "Token OAuth2 na variável de ambiente AUTH_TOKEN"]
  - Limites de taxa: [se aplicável]

## Armazenamento de Dados

**Bancos de Dados:**
- [Tipo/Provedor] - [ex., "PostgreSQL no Supabase"]
  - Conexão: [ex., "via variável de ambiente DATABASE_URL"]
  - Cliente: [ex., "Prisma ORM v5.x"]
  - Migrações: [ex., "prisma migrate em migrations/"]

**Armazenamento de Arquivos:**
- [Serviço] - [ex., "AWS S3 para uploads de usuários"]
  - SDK/Cliente: [ex., "@aws-sdk/client-s3"]
  - Autenticação: [ex., "Credenciais IAM em variáveis AWS_*"]
  - Buckets: [ex., "prod-uploads, dev-uploads"]

**Cache:**
- [Serviço] - [ex., "Redis para armazenamento de sessão"]
  - Conexão: [ex., "variável de ambiente REDIS_URL"]
  - Cliente: [ex., "ioredis v5.x"]

## Autenticação e Identidade

**Provedor de Auth:**
- [Serviço] - [ex., "Supabase Auth", "Auth0", "JWT customizado"]
  - Implementação: [ex., "SDK cliente do Supabase"]
  - Armazenamento de token: [ex., "cookies httpOnly", "localStorage"]
  - Gerenciamento de sessão: [ex., "JWT refresh tokens"]

**Integrações OAuth:**
- [Provedor] - [ex., "Google OAuth para login"]
  - Credenciais: [ex., "GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET"]
  - Escopos: [ex., "email, profile"]

## Monitoramento e Observabilidade

**Rastreamento de Erros:**
- [Serviço] - [ex., "Sentry"]
  - DSN: [ex., "variável de ambiente SENTRY_DSN"]
  - Rastreamento de release: [ex., "via SENTRY_RELEASE"]

**Analytics:**
- [Serviço] - [ex., "Mixpanel para analytics de produto"]
  - Token: [ex., "variável de ambiente MIXPANEL_TOKEN"]
  - Eventos rastreados: [ex., "ações do usuário, visualizações de página"]

**Logs:**
- [Serviço] - [ex., "CloudWatch", "Datadog", "nenhum (apenas stdout)"]
  - Integração: [ex., "Built-in do AWS Lambda"]

## CI/CD e Deploy

**Hospedagem:**
- [Plataforma] - [ex., "Vercel", "AWS Lambda", "Container Docker no ECS"]
  - Deploy: [ex., "automático ao push na branch main"]
  - Variáveis de ambiente: [ex., "configuradas no dashboard da Vercel"]

**Pipeline de CI:**
- [Serviço] - [ex., "GitHub Actions"]
  - Workflows: [ex., "test.yml, deploy.yml"]
  - Secrets: [ex., "armazenados nos secrets do repositório GitHub"]

## Configuração de Ambiente

**Desenvolvimento:**
- Variáveis de ambiente obrigatórias: [Listar variáveis críticas]
- Localização dos secrets: [ex., ".env.local (gitignored)", "vault 1Password"]
- Serviços mock/stub: [ex., "Stripe modo teste", "PostgreSQL local"]

**Staging:**
- Diferenças específicas do ambiente: [ex., "usa conta Stripe de staging"]
- Dados: [ex., "banco de dados separado para staging"]

**Produção:**
- Gerenciamento de secrets: [ex., "Variáveis de ambiente da Vercel"]
- Failover/redundância: [ex., "Replicação de DB multi-região"]

## Webhooks e Callbacks

**Recebidos:**
- [Serviço] - [Endpoint: ex., "/api/webhooks/stripe"]
  - Verificação: [ex., "validação de assinatura via stripe.webhooks.constructEvent"]
  - Eventos: [ex., "payment_intent.succeeded, customer.subscription.updated"]

**Enviados:**
- [Serviço] - [O que dispara]
  - Endpoint: [ex., "webhook CRM externo no signup do usuário"]
  - Lógica de retry: [se aplicável]

---

*Auditoria de integrações: [data]*
*Atualize ao adicionar/remover serviços externos*
```

<good_examples>
```markdown
# Integrações Externas

**Data da Análise:** 2025-01-20

## APIs e Serviços Externos

**Processamento de Pagamentos:**
- Stripe - Cobrança de assinatura e pagamentos únicos de cursos
  - SDK/Cliente: pacote npm stripe v14.8
  - Autenticação: Chave de API na variável de ambiente STRIPE_SECRET_KEY
  - Endpoints usados: sessões de checkout, portal do cliente, webhooks

**Email/SMS:**
- SendGrid - Emails transacionais (recibos, redefinição de senha)
  - SDK/Cliente: @sendgrid/mail v8.1
  - Autenticação: Chave de API na variável de ambiente SENDGRID_API_KEY
  - Templates: Gerenciados no dashboard do SendGrid (IDs dos templates no código)

**APIs Externas:**
- API da OpenAI - Geração de conteúdo de curso
  - Método de integração: API REST via pacote npm openai v4.x
  - Autenticação: Bearer token na variável de ambiente OPENAI_API_KEY
  - Limites de taxa: 3500 requisições/min (tier 3)

## Armazenamento de Dados

**Bancos de Dados:**
- PostgreSQL no Supabase - Armazenamento de dados primário
  - Conexão: via variável de ambiente DATABASE_URL
  - Cliente: Prisma ORM v5.8
  - Migrações: prisma migrate em prisma/migrations/

**Armazenamento de Arquivos:**
- Supabase Storage - Uploads de usuários (imagens de perfil, materiais de curso)
  - SDK/Cliente: @supabase/supabase-js v2.x
  - Autenticação: Chave de role de serviço em SUPABASE_SERVICE_ROLE_KEY
  - Buckets: avatars (público), course-materials (privado)

**Cache:**
- Nenhum atualmente (todas queries no banco, sem Redis)

## Autenticação e Identidade

**Provedor de Auth:**
- Supabase Auth - Email/senha + OAuth
  - Implementação: SDK cliente do Supabase com gerenciamento de sessão server-side
  - Armazenamento de token: cookies httpOnly via @supabase/ssr
  - Gerenciamento de sessão: JWT refresh tokens gerenciados pelo Supabase

**Integrações OAuth:**
- Google OAuth - Login social
  - Credenciais: GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET (dashboard do Supabase)
  - Escopos: email, profile

## Monitoramento e Observabilidade

**Rastreamento de Erros:**
- Sentry - Erros de servidor e cliente
  - DSN: variável de ambiente SENTRY_DSN
  - Rastreamento de release: SHA do commit Git via SENTRY_RELEASE

**Analytics:**
- Nenhum (planejado: Mixpanel)

**Logs:**
- Logs da Vercel - apenas stdout/stderr
  - Retenção: 7 dias no plano Pro

## CI/CD e Deploy

**Hospedagem:**
- Vercel - Hospedagem de app Next.js
  - Deploy: Automático ao push na branch main
  - Variáveis de ambiente: Configuradas no dashboard da Vercel (sincronizadas com .env.example)

**Pipeline de CI:**
- GitHub Actions - Testes e verificação de tipos
  - Workflows: .github/workflows/ci.yml
  - Secrets: Nenhum necessário (testes de repo público apenas)

## Configuração de Ambiente

**Desenvolvimento:**
- Variáveis de ambiente obrigatórias: DATABASE_URL, NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY
- Localização dos secrets: .env.local (gitignored), compartilhados pelo time via vault 1Password
- Serviços mock/stub: Stripe modo teste, projeto dev local do Supabase

**Staging:**
- Usa projeto Supabase separado para staging
- Stripe modo teste
- Mesma conta Vercel, ambiente diferente

**Produção:**
- Gerenciamento de secrets: Variáveis de ambiente da Vercel
- Banco de dados: Projeto Supabase de produção com backups diários

## Webhooks e Callbacks

**Recebidos:**
- Stripe - /api/webhooks/stripe
  - Verificação: Validação de assinatura via stripe.webhooks.constructEvent
  - Eventos: payment_intent.succeeded, customer.subscription.updated, customer.subscription.deleted

**Enviados:**
- Nenhum

---

*Auditoria de integrações: 2025-01-20*
*Atualize ao adicionar/remover serviços externos*
```
</good_examples>

<guidelines>
**O que pertence ao INTEGRATIONS.md:**
- Serviços externos com os quais o código se comunica
- Padrões de autenticação (onde os secrets ficam, não os secrets em si)
- SDKs e bibliotecas cliente usadas
- Nomes de variáveis de ambiente (não valores)
- Endpoints de webhook e métodos de verificação
- Padrões de conexão com banco de dados
- Localizações de armazenamento de arquivos
- Serviços de monitoramento e logging

**O que NÃO pertence aqui:**
- Chaves de API ou secrets reais (NUNCA escreva esses)
- Arquitetura interna (isso é ARCHITECTURE.md)
- Padrões de código (isso é PATTERNS.md)
- Escolhas de tecnologia (isso é STACK.md)
- Problemas de performance (isso é CONCERNS.md)

**Ao preencher este template:**
- Verifique .env.example ou .env.template para variáveis de ambiente obrigatórias
- Procure imports de SDK (stripe, @sendgrid/mail, etc.)
- Verifique handlers de webhook nas rotas/endpoints
- Note onde secrets são gerenciados (não os secrets)
- Documente diferenças específicas de ambiente (dev/staging/prod)
- Inclua padrões de autenticação para cada serviço

**Útil para planejamento de fase quando:**
- Adicionando novas integrações de serviços externos
- Depurando problemas de autenticação
- Entendendo fluxo de dados fora da aplicação
- Configurando novos ambientes
- Auditando dependências de terceiros
- Planejando para indisponibilidade ou migrações de serviços

**Nota de segurança:**
Documente ONDE os secrets ficam (variáveis de ambiente, dashboard da Vercel, 1Password), nunca QUAIS são os secrets.
</guidelines>
