# Template de Configuração do Usuário

Template para `.planning/phases/XX-nome/{fase}-USER-SETUP.md` - configuração humana necessária que o Claude não pode automatizar.

**Propósito:** Documentar tarefas de configuração que literalmente requerem ação humana - criação de conta, configuração de dashboard, recuperação de secrets. Claude automatiza tudo que é possível; este arquivo captura apenas o que resta.

---

## Template do Arquivo

```markdown
# Fase {X}: Configuração do Usuário Necessária

**Gerado:** [AAAA-MM-DD]
**Fase:** {nome-da-fase}
**Status:** Incompleto

Complete estes itens para a integração funcionar. Claude automatizou tudo que foi possível; estes itens requerem acesso humano a dashboards/contas externas.

## Variáveis de Ambiente

| Status | Variável | Origem | Adicionar em |
|--------|----------|--------|--------------|
| [ ] | `NOME_VAR_ENV` | [Dashboard do Serviço → Caminho → Para → Valor] | `.env.local` |
| [ ] | `OUTRA_VAR` | [Dashboard do Serviço → Caminho → Para → Valor] | `.env.local` |

## Configuração de Conta

[Apenas se criação de nova conta for necessária]

- [ ] **Criar conta no [Serviço]**
  - URL: [URL de cadastro]
  - Pular se: Já tem conta

## Configuração de Dashboard

[Apenas se configuração de dashboard for necessária]

- [ ] **[Tarefa de configuração]**
  - Local: [Dashboard do Serviço → Caminho → Para → Configuração]
  - Definir como: [Valor ou configuração necessária]
  - Notas: [Quaisquer detalhes importantes]

## Verificação

Após completar a configuração, verifique com:

```bash
# [Comandos de verificação]
```

Resultados esperados:
- [Como o sucesso se parece]

---

**Quando todos os itens estiverem completos:** Marque o status como "Completo" no topo do arquivo.
```

---

## Quando Gerar

Gere `{fase}-USER-SETUP.md` quando o frontmatter do plano contém campo `user_setup`.

**Gatilho:** `user_setup` existe no frontmatter do PLAN.md e tem itens.

**Local:** Mesmo diretório que PLAN.md e SUMMARY.md.

**Timing:** Gerado durante execute-plan.md após tarefas serem concluídas, antes da criação do SUMMARY.md.

---

## Schema do Frontmatter

No PLAN.md, `user_setup` declara configuração que requer ação humana:

```yaml
user_setup:
  - service: stripe
    why: "Processamento de pagamento requer chaves de API"
    env_vars:
      - name: STRIPE_SECRET_KEY
        source: "Stripe Dashboard → Developers → API keys → Secret key"
      - name: STRIPE_WEBHOOK_SECRET
        source: "Stripe Dashboard → Developers → Webhooks → Signing secret"
    dashboard_config:
      - task: "Criar endpoint de webhook"
        location: "Stripe Dashboard → Developers → Webhooks → Add endpoint"
        details: "URL: https://[seu-dominio]/api/webhooks/stripe, Eventos: checkout.session.completed, customer.subscription.*"
    local_dev:
      - "Execute: stripe listen --forward-to localhost:3000/api/webhooks/stripe"
      - "Use o webhook secret da saída do CLI para testes locais"
```

---

## A Regra Automação-Primeiro

**USER-SETUP.md contém APENAS o que o Claude literalmente não pode fazer.**

| Claude PODE Fazer (não no USER-SETUP) | Claude NÃO PODE Fazer (→ USER-SETUP) |
|----------------------------------------|---------------------------------------|
| `npm install stripe` | Criar conta Stripe |
| Escrever código do webhook handler | Obter chaves de API do dashboard |
| Criar estrutura do arquivo `.env.local` | Copiar valores reais de secrets |
| Executar `stripe listen` | Autenticar Stripe CLI (OAuth no navegador) |
| Configurar package.json | Acessar dashboards de serviços externos |
| Escrever qualquer código | Recuperar secrets de sistemas terceiros |

**O teste:** "Isto requer um humano em um navegador, acessando uma conta para a qual o Claude não tem credenciais?"
- Sim → USER-SETUP.md
- Não → Claude faz automaticamente

---

## Exemplos por Serviço

<stripe_example>
```markdown
# Fase 10: Configuração do Usuário Necessária

**Gerado:** 2025-01-14
**Fase:** 10-monetizacao
**Status:** Incompleto

Complete estes itens para a integração Stripe funcionar.

## Variáveis de Ambiente

| Status | Variável | Origem | Adicionar em |
|--------|----------|--------|--------------|
| [ ] | `STRIPE_SECRET_KEY` | Stripe Dashboard → Developers → API keys → Secret key | `.env.local` |
| [ ] | `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe Dashboard → Developers → API keys → Publishable key | `.env.local` |
| [ ] | `STRIPE_WEBHOOK_SECRET` | Stripe Dashboard → Developers → Webhooks → [endpoint] → Signing secret | `.env.local` |

## Configuração de Conta

- [ ] **Criar conta Stripe** (se necessário)
  - URL: https://dashboard.stripe.com/register
  - Pular se: Já tem conta Stripe

## Configuração de Dashboard

- [ ] **Criar endpoint de webhook**
  - Local: Stripe Dashboard → Developers → Webhooks → Add endpoint
  - URL do endpoint: `https://[seu-dominio]/api/webhooks/stripe`
  - Eventos para enviar:
    - `checkout.session.completed`
    - `customer.subscription.created`
    - `customer.subscription.updated`
    - `customer.subscription.deleted`

- [ ] **Criar produtos e preços** (se usando planos de assinatura)
  - Local: Stripe Dashboard → Products → Add product
  - Criar cada plano de assinatura
  - Copiar Price IDs para:
    - `STRIPE_STARTER_PRICE_ID`
    - `STRIPE_PRO_PRICE_ID`

## Desenvolvimento Local

Para teste local de webhook:
```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```
Use o webhook signing secret da saída do CLI (começa com `whsec_`).

## Verificação

Após completar a configuração:

```bash
# Verificar variáveis de ambiente
grep STRIPE .env.local

# Verificar build
npm run build

# Testar endpoint de webhook (deve retornar 400 bad signature, não 500 crash)
curl -X POST http://localhost:3000/api/webhooks/stripe \
  -H "Content-Type: application/json" \
  -d '{}'
```

Esperado: Build passa, webhook retorna 400 (validação de assinatura funcionando).

---

**Quando todos os itens estiverem completos:** Marque o status como "Completo" no topo do arquivo.
```
</stripe_example>

<supabase_example>
```markdown
# Fase 2: Configuração do Usuário Necessária

**Gerado:** 2025-01-14
**Fase:** 02-autenticacao
**Status:** Incompleto

Complete estes itens para o Supabase Auth funcionar.

## Variáveis de Ambiente

| Status | Variável | Origem | Adicionar em |
|--------|----------|--------|--------------|
| [ ] | `NEXT_PUBLIC_SUPABASE_URL` | Supabase Dashboard → Settings → API → Project URL | `.env.local` |
| [ ] | `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase Dashboard → Settings → API → anon public | `.env.local` |
| [ ] | `SUPABASE_SERVICE_ROLE_KEY` | Supabase Dashboard → Settings → API → service_role | `.env.local` |

## Configuração de Conta

- [ ] **Criar projeto Supabase**
  - URL: https://supabase.com/dashboard/new
  - Pular se: Já tem projeto para este app

## Configuração de Dashboard

- [ ] **Habilitar Auth por Email**
  - Local: Supabase Dashboard → Authentication → Providers
  - Habilitar: Provider de Email
  - Configurar: Confirmar email (on/off conforme preferência)

- [ ] **Configurar provedores OAuth** (se usando login social)
  - Local: Supabase Dashboard → Authentication → Providers
  - Para Google: Adicionar Client ID e Secret do Google Cloud Console
  - Para GitHub: Adicionar Client ID e Secret do GitHub OAuth Apps

## Verificação

Após completar a configuração:

```bash
# Verificar variáveis de ambiente
grep SUPABASE .env.local

# Verificar conexão (executar no diretório do projeto)
npx supabase status
```

---

**Quando todos os itens estiverem completos:** Marque o status como "Completo" no topo do arquivo.
```
</supabase_example>

<sendgrid_example>
```markdown
# Fase 5: Configuração do Usuário Necessária

**Gerado:** 2025-01-14
**Fase:** 05-notificacoes
**Status:** Incompleto

Complete estes itens para o email SendGrid funcionar.

## Variáveis de Ambiente

| Status | Variável | Origem | Adicionar em |
|--------|----------|--------|--------------|
| [ ] | `SENDGRID_API_KEY` | SendGrid Dashboard → Settings → API Keys → Create API Key | `.env.local` |
| [ ] | `SENDGRID_FROM_EMAIL` | Seu endereço de email de remetente verificado | `.env.local` |

## Configuração de Conta

- [ ] **Criar conta SendGrid**
  - URL: https://signup.sendgrid.com/
  - Pular se: Já tem conta

## Configuração de Dashboard

- [ ] **Verificar identidade do remetente**
  - Local: SendGrid Dashboard → Settings → Sender Authentication
  - Opção 1: Single Sender Verification (rápido, para dev)
  - Opção 2: Domain Authentication (produção)

- [ ] **Criar API Key**
  - Local: SendGrid Dashboard → Settings → API Keys → Create API Key
  - Permissão: Restricted Access → Mail Send (Full Access)
  - Copiar chave imediatamente (mostrada apenas uma vez)

## Verificação

Após completar a configuração:

```bash
# Verificar variável de ambiente
grep SENDGRID .env.local

# Testar envio de email (substitua com seu email de teste)
curl -X POST http://localhost:3000/api/test-email \
  -H "Content-Type: application/json" \
  -d '{"to": "seu@email.com"}'
```

---

**Quando todos os itens estiverem completos:** Marque o status como "Completo" no topo do arquivo.
```
</sendgrid_example>

---

## Diretrizes

**Nunca inclua:** Valores reais de secrets. Passos que o Claude pode automatizar (instalação de pacotes, alterações de código).

**Nomenclatura:** `{fase}-USER-SETUP.md` segue o padrão de número da fase.
**Rastreamento de status:** Usuário marca checkboxes e atualiza linha de status quando completo.
**Buscabilidade:** `grep -r "USER-SETUP" .planning/` encontra todas as fases com requisitos do usuário.
