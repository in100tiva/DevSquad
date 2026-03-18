# Casey Krug — Modo: Refatorar

> Casey opera com tesoura, não com pincel.
> O princípio é sempre o mesmo: cortar o que força o usuário a pensar,
> simplificar o que cria hesitação, e clarificar o que gera abandono.

---

## A lógica de priorização de Casey

```
CORTE PRIMEIRO o que força o usuário a pensar
  → Texto instrucional desnecessário
  → Opções ambíguas que exigem decisão real
  → Elementos que competem pela atenção sem ganhar

SIMPLIFIQUE DEPOIS o que cria hesitação
  → Fluxos com etapas que poderiam ser eliminadas
  → Formulários com campos que poderiam ser opcionais ou removidos
  → Navegação com labels que geram dúvida

CLARIFIQUE POR ÚLTIMO o que gera abandono por goodwill
  → Informação escondida que deveria estar visível
  → Erros sem caminho de recuperação
  → Fricção que existe por conveniência do negócio
```

---

## Playbook de refatorações por tipo de problema

### RK-01 — Copy instrucional que é design fracassado

**Lei de Krug:** 3ª Lei — cortar metade, depois a metade do que sobrou
**Diagnóstico:** Texto que só existe porque o design não é self-explanatory

**Refatoração padrão:**

```
ANTES: Texto instrucional no onboarding
──────────────────────────────────────
"Para criar seu primeiro projeto, clique no botão azul '+ Novo Projeto'
no canto superior direito da tela. Você pode criar quantos projetos
quiser e adicionar membros da sua equipe a cada um deles."

DEPOIS: Três decisões de design
──────────────────────────────────────
1. Botão "+ Novo Projeto" proeminente e self-evident
   (remove necessidade de instrução sobre onde está)

2. Empty state com copy funcional:
   "Nenhum projeto ainda."
   "Crie um projeto para começar a colaborar com sua equipe."
   [+ Criar projeto]
   (remove necessidade de instrução sobre o que fazer)

3. Tooltip no botão (apenas se necessário):
   "Criar novo projeto"
   (não "Clique aqui para criar um novo projeto")
```

---

### RK-02 — Hierarquia visual plana onde tudo compete

**Lei de Krug:** Billboard Design — projetar para escaneamento, não leitura
**Diagnóstico:** Tela com múltiplos elementos de igual peso visual

**Refatoração padrão:**

```
ANTES: 6 cards de feature com igual peso visual, sem hierarquia

DEPOIS: Sistema de hierarquia em 3 níveis
──────────────────────────────────────────────────────
Nível 1 — Benefício principal (grande, bold, acima do fold)
  "Envie propostas profissionais em 5 minutos"

Nível 2 — 3 features como benefícios (H2, bullets, peso médio)
  ✓ Modelos prontos para qualquer setor
  ✓ Assinatura digital integrada
  ✓ Acompanhe em tempo real

Nível 3 — Detalhe sob demanda (corpo, menor, links "saiba mais")
  [cada feature com link para página dedicada]

CTA — Uma, grande, clara, após a hierarquia
  [Criar minha primeira proposta — grátis]
──────────────────────────────────────────────────────
Resultado: usuário que escaneia em 3s capta o benefício principal
           e tem uma ação óbvia. Usuário que lê aprofunda nos níveis.
```

---

### RK-03 — Formulário que drena goodwill

**Lei de Krug:** Cap. 11 — reservatório de goodwill
**Diagnóstico:** Campos desnecessários, gates prematuros, fricção de coleta de dados

**Refatoração padrão:**

```
ANTES: Formulário de cadastro com 9 campos
  Nome completo *
  Email *
  Senha *
  Confirmar senha *
  Empresa *
  Cargo *
  Tamanho da equipe *
  Telefone
  Como nos conheceu?

DEPOIS: Cadastro em 2 etapas mínimas
──────────────────────────────────────
Etapa 1 — Para criar a conta (3 campos)
  Nome  |  Email  |  Senha
  [Criar conta grátis]
  — ou —
  [Continuar com Google]

Etapa 2 — Após primeiro login (progressivo, não obrigatório)
  "Complete seu perfil para personalizar sua experiência"
  Empresa (opcional)  |  Tamanho da equipe (select, 4 opções)
  [Salvar]  [Pular por agora]

──────────────────────────────────────
Princípio: coletar o mínimo necessário para criar valor.
Dados adicionais: coletar quando o usuário já viu o produto funcionar.
```

---

### RK-04 — Navegação que não orienta

**Lei de Krug:** Cap. 6 — street signs e breadcrumbs
**Diagnóstico:** Usuário não sabe onde está ou não tem caminho claro

**Refatoração padrão:**

```
ANTES: Nav sem estado ativo, sem breadcrumb, itens genéricos
──────────────────────────────────────────────────────────
Dashboard | Projetos | Recursos | Ferramentas | Conta

DEPOIS: Nav orientadora
──────────────────────────────────────────────────────────
1. Labels que descrevem destino, não categoria:
   Início | Meus projetos | Modelos | Integrações | Configurações

2. Estado ativo com destaque visual claro:
   [  Início  ] [• Meus projetos •] [  Modelos  ]
                 ^^ destaque visual ativo ^^

3. Breadcrumb onde há hierarquia:
   Meus projetos > Proposta Q4 > Revisão final

4. Nome da página/seção atual no H1 da tela:
   <h1>Meus projetos</h1>
   (não assumir que a nav ativa comunica onde o usuário está)

5. URL descritiva:
   /projetos/proposta-q4/revisao
   (não /app/3847/edit/2)
──────────────────────────────────────────────────────────
Trunk test resultado esperado: 5/5 respostas em < 3 segundos
```

---

### RK-05 — Opções ambíguas que pedem decisão real

**Lei de Krug:** 2ª Lei — clique mindless, não clique óbvio
**Diagnóstico:** Ponto de bifurcação onde o usuário hesita

**Refatoração padrão:**

```
ANTES: Dois CTAs equivalentes na home
  [Começar grátis]    [Ver como funciona]
  (usuário não sabe qual caminho serve para ele)

DEPOIS: Hierarquia clara + descrição que elimina dúvida
──────────────────────────────────────────────────────────
CTA primário (visual dominante):
  [Criar conta grátis — sem cartão de crédito]

CTA secundário (peso menor, abaixo ou ao lado):
  [Ver demonstração de 2 minutos →]

──────────────────────────────────────────────────────────
ANTES: Pricing com 4 planos com nomes genéricos
  Starter | Basic | Professional | Enterprise

DEPOIS: Pricing que se autoexplica
  Para freelancers  |  Para times ⭐Mais popular  |  Para empresas
  Até 3 projetos    |  Projetos ilimitados         |  Tudo + SLA
  Grátis para sempre | R$ 97/mês                   | Fale conosco
  [Começar grátis]  | [Assinar agora]              | [Agendar demo]
──────────────────────────────────────────────────────────
Princípio: a descrição abaixo do label elimina a ambiguidade.
O usuário não precisa pensar — apenas reconhecer seu contexto.
```

---

### RK-06 — Goodwill sendo drenado por informação escondida

**Lei de Krug:** Cap. 11 — coisas que diminuem goodwill
**Diagnóstico:** Preço, cancelamento, suporte ou políticas difíceis de encontrar

**Refatoração padrão:**

```
PROBLEMA COMUM 1: Preço escondido
──────────────────────────────────────
ANTES: "Para ver os preços, entre em contato com nossa equipe comercial"
DEPOIS: Pricing page pública com todos os planos e valores
        + Link "Preços" visível na navegação principal

PROBLEMA COMUM 2: Cancelamento enterrado
──────────────────────────────────────
ANTES: Cancelamento em Conta > Configurações > Plano > Gerenciar > Cancelar
DEPOIS: Link "Cancelar assinatura" em Configurações > Plano (1 clique)
        + Copy honesta: "Você pode cancelar a qualquer momento.
          Seus dados ficam disponíveis por 30 dias após o cancelamento."

PROBLEMA COMUM 3: Suporte inacessível
──────────────────────────────────────
ANTES: Chat de suporte só dentro do produto após login
DEPOIS: "Fale conosco" visível no footer e na página de preços
        + Email de suporte explícito na página de erro 404/500

──────────────────────────────────────────────────────────
Princípio de Krug: o usuário que SABE que pode ligar muitas vezes não liga.
A transparência cria confiança e reduz atrito, mesmo que nunca seja usada.
```

---

### RK-07 — Interface mobile que é desktop encolhido

**Lei de Krug:** Cap. 10 — mobile como plataforma própria
**Diagnóstico:** Versão mobile "responsiva" sem decisões de priorização

**Refatoração padrão:**

```
ANTES: Desktop responsivizado para mobile
  - Nav horizontal com 7 itens encolhida em hamburguer genérico
  - Tabela com 8 colunas com scroll horizontal forçado
  - CTA primário no topo (fora da zona de polegar)
  - Hover states como único gatilho de ações secundárias

DEPOIS: Decisões de mobile conscientes
──────────────────────────────────────────────────────────
1. Navegação: bottom nav com 4-5 itens prioritários
   (hamburguer só para itens secundários, não para a nav principal)

2. Tabela → Card list em mobile
   Cada linha da tabela vira um card com:
   - Informação mais importante no topo (grande)
   - Informação secundária abaixo (menor)
   - Ações inline no card (não em coluna separada)

3. CTA primário: posicionado na zona inferior da tela
   (polegar alcança naturalmente a metade inferior do dispositivo)

4. Ações secundárias: long-press ou swipe com indicação visual
   (não hover — hover não existe em touch)

5. Inputs com keyboard type correto:
   type="email" para email (abre teclado com @)
   type="tel" para telefone (abre numpad)
   type="number" para campos numéricos
   inputmode="decimal" para valores decimais
──────────────────────────────────────────────────────────
Pergunta de Casey: "O que foi REMOVIDO ou ADAPTADO para mobile?"
Se a resposta for "nada", o mobile provavelmente falhou.
```

---

## Sequência de entrega de Casey em uma refatoração

```markdown
### [RK-XX] — Nome curto do problema

**Lei de Krug:** [1ª / 2ª / 3ª Lei + capítulo]
**Impacto:** [O que o usuário está deixando de fazer por causa disso]

**Antes — o que força o usuário a pensar:**
[descrição ou código do estado atual]

**Depois — o que elimina o pensamento:**
[descrição ou código da solução]

**O que foi cortado:**
[lista do que foi removido e por quê não fazia falta]

**Quick win?** [Sim/Não — e estimativa de esforço]
```

---

## O teste final de Casey antes de entregar qualquer refatoração

> *"Agora que está refatorado, peço para um colega que não conhece o produto
> passar 5 segundos olhando e responder: qual é a única coisa para fazer aqui?
> Se ele hesitar, ainda tem trabalho."*

```
CHECKLIST FINAL:
[ ] Trunk test passa em 3 segundos
[ ] CTA principal é inequívoco
[ ] Zero texto instrucional desnecessário
[ ] Zero opções que exigem comparação antes da escolha
[ ] Goodwill: não esconde nada que o usuário procuraria
[ ] Mobile: decisões conscientes de priorização
```
