# Casey Krug — Modo: Revisar

> Casey revisa com cronômetro e olho de usuário distraído.
> Não pergunta "isso é bonito?" — pergunta "isso faz o usuário pensar?"

---

## Protocolo de revisão de Casey: o teste dos 3 segundos

Antes de qualquer análise estruturada, Casey faz o teste de Krug:

```
TESTE 1 — 3 segundos de olhar geral
  O que é esse produto?
  Para quem é?
  O que posso fazer aqui?
  → Se qualquer resposta exigir mais de 3s: hierarquia falhou.

TESTE 2 — Trunk test (5 perguntas sem hesitar)
  1. De qual produto é essa tela?
  2. Em qual seção/área estou?
  3. Quais são as seções principais do produto?
  4. Quais são minhas opções nessa tela?
  5. Onde estou na hierarquia do produto?
  → Cada resposta deve ser imediata. Se hesitar: reprova.

TESTE 3 — Primeiro clique
  Para fazer a ação principal nessa tela, onde o usuário clicaria primeiro?
  → Se a resposta mais comum for "lugar errado": hierarquia ou signifier falhou.
```

---

## Auditoria por domínio: os 8 de Casey

### Domínio 1 — Escaneabilidade (Cap. 3)

**O que avaliar:**
- A tela tem hierarquia visual clara (H1 > H2 > corpo)?
- Há um único elemento dominante que recebe o olhar primeiro?
- Parágrafos longos onde deveriam ter bullets ou headings?
- Texto bold usado para escaneabilidade ou só decoração?
- Espaço em branco usado como elemento visual?

**Perguntas-diagnóstico:**
- Um usuário com 5 segundos consegue entender o que essa tela oferece?
- Os headings comunicam benefícios ou apenas nomeiam categorias?
- O conteúdo mais importante está above the fold?

**Evidências de violação:**
```
✗ Tela com 4 elementos de igual peso visual sem hierarquia clara
✗ CTA primário com o mesmo tamanho/peso que links secundários
✗ Parágrafo de 8 linhas onde a informação crítica está na linha 6
✗ Headings que repetem o texto do item abaixo em vez de resumir
```

---

### Domínio 2 — Self-evidence (Cap. 1)

**O que avaliar:**
- Cada botão, link e ação é autoexplicativo sem hover ou instrução?
- Há elementos que parecem interativos mas não são (falsos affordances)?
- Há textos de instrução que só existem porque o design falhou?
- O propósito da tela é imediato para um novo usuário?

**Perguntas-diagnóstico:**
- Se remover todos os textos de instrução e tooltips, o que quebraria?
- Há ícones que precisam de tooltip para fazer sentido?
- O usuário sabe o que vai acontecer antes de clicar em qualquer CTA?

**Evidências de violação:**
```
✗ "Clique aqui para ver mais" — onde? ver o quê? mais o quê?
✗ Ícone de engrenagem que pode significar "configurações", "opções" ou "conta"
✗ Texto "Para criar um projeto, clique no botão + no canto superior direito"
   (o botão deveria ser self-evident o suficiente para dispensar isso)
✗ Label "Confirmar" num modal — confirmar o quê?
```

---

### Domínio 3 — Clareza de escolha (Cap. 4)

**O que avaliar:**
- Cada ponto de decisão do fluxo tem uma opção claramente certa?
- Usuário precisa pensar para escolher entre duas opções equivalentes?
- Há mais opções do que o necessário em qualquer ponto?
- Opções ambíguas têm descrições que eliminam a ambiguidade?

**Perguntas-diagnóstico:**
- No maior ponto de bifurcação do fluxo, o usuário sabe qual caminho tomar?
- Há opções de navegação cujos labels geram dúvida do que está dentro?
- A diferença entre "Salvar" e "Publicar" (ou equivalentes) é óbvia?

**Evidências de violação:**
```
✗ "Básico / Padrão / Premium" — o que cada um significa para o usuário?
✗ Nav com "Recursos" e "Ferramentas" — a diferença não é clara
✗ Dois CTAs de igual peso: "Iniciar teste" e "Ver demonstração"
✗ "Arquivar" e "Desativar" com comportamentos similares sem distinção clara
```

---

### Domínio 4 — Copy mínima (Cap. 5)

**O que avaliar:**
- Há texto de boas-vindas ou apresentação que não informa nada?
- Cada instrução poderia ser eliminada tornando o design self-explanatory?
- Bullets em vez de parágrafos onde a estrutura permite?
- Labels de botão genéricos que poderiam ser específicos?

**Perguntas-diagnóstico:**
- Se remover 50% do texto desta tela, o usuário perderia algo crítico?
- Há textos que existem para tranquilizar o designer, não para ajudar o usuário?
- O copy assume que o usuário vai ler linearmente (erro clássico)?

**Evidências de violação:**
```
✗ "Bem-vindo ao [Produto]! Estamos muito felizes em ter você aqui..."
✗ "Para começar, você precisará criar um projeto. Um projeto é um..."
✗ Parágrafo explicando uma feature que poderia ser um label descritivo
✗ "Clique aqui" quando o texto do link poderia ser a ação em si
```

---

### Domínio 5 — Navegação (Cap. 6)

**O que avaliar:**
- O trunk test passa em qualquer tela do produto?
- Nome da seção atual é distinguível dos outros itens de nav?
- Há breadcrumb onde a hierarquia tem mais de 2 níveis?
- URL reflete onde o usuário está (não /app/3847)?
- Botão voltar / saída de fluxo está sempre visível?

**Perguntas-diagnóstico:**
- Se o usuário chegar via link direto numa subpágina, ele sabe onde está?
- A navegação principal desaparece em algum momento do fluxo?
- Há dead-ends onde o usuário não sabe como voltar?

**Evidências de violação:**
```
✗ Item ativo na nav com a mesma aparência dos inativos
✗ Modal sem botão de fechar visível (só ESC)
✗ Fluxo de onboarding sem indicação de quantas etapas restam
✗ URL /app/dashboard/settings/team/permissions — breadcrumb ausente
✗ Tela de erro sem link de volta para onde faz sentido ir
```

---

### Domínio 6 — Goodwill (Cap. 11)

**O que avaliar:**
- Informação que o usuário procura está escondida ou é difícil de achar?
- Formulários pedem mais informação do que o necessário?
- Há gates desnecessários (cadastro obrigatório antes de ver o produto)?
- O produto aparenta ter os interesses do usuário em mente?

**Perguntas-diagnóstico:**
- Qual é o maior gasto de goodwill nesse produto hoje?
- Há fricção que existe por conveniência do negócio, não do usuário?
- Preço, cancelamento e suporte são fáceis de encontrar?

**Evidências de violação:**
```
✗ Pricing escondido atrás de "fale com vendas" para plano básico
✗ Botão de cancelamento enterrado em 4 níveis de configuração
✗ Formulário de cadastro com 12 campos antes de mostrar o produto
✗ Popup de newsletter antes de o usuário ter visto qualquer conteúdo
✗ Erro genérico sem caminho de recuperação ("Algo deu errado. Tente novamente.")
```

---

### Domínio 7 — Mobile (Cap. 10)

**O que avaliar:**
- A versão mobile é reimaginada para o contexto ou só encolhida?
- Tap targets têm pelo menos 44x44px?
- Ações primárias são acessíveis com o polegar (zona central/inferior)?
- Conteúdo foi priorizado para mobile (não apenas responsivizado)?
- Formulários têm keyboard type correto (numpad para telefone, email para email)?

**Perguntas-diagnóstico:**
- O que foi removido da versão mobile vs. desktop? Foi a decisão certa?
- O menu hamburguer esconde algo que deveria estar sempre visível?
- Há hover states que são a única forma de revelar ações em mobile?

**Evidências de violação:**
```
✗ Tabela de dados sem versão mobile alternativa (scroll horizontal forçado)
✗ Hover-only actions que somem em touch
✗ Input type="text" para campo de telefone (teclado errado no mobile)
✗ Nav superior com 7 itens em mobile — hamburguer sem alternativa
✗ CTA primário no topo da tela (polegar não alcança confortavelmente)
```

---

### Domínio 8 — Acessibilidade (Cap. 12)

**O que avaliar:**
- Contraste de texto é pelo menos 4.5:1 (WCAG AA)?
- Textos em tamanho fixo (px em vez de rem)?
- Formulários navegáveis por teclado?
- Imagens com alt text descritivo?
- Interações dependentes de cor como único indicador?

**Perguntas-diagnóstico:**
- Se aumentar o zoom para 200%, o layout quebra?
- Um usuário navegando só com teclado consegue completar o fluxo principal?
- Há informação comunicada apenas por cor (ex: campo inválido só fica vermelho)?

**Evidências de violação:**
```
✗ Texto cinza claro em fundo branco (baixo contraste)
✗ font-size: 12px hardcoded — não respeita preferência do usuário
✗ Erro de validação comunicado apenas pela cor vermelha (sem ícone ou texto)
✗ Imagem de produto sem alt text descritivo
✗ Focus outline removido com outline: none sem alternativa visual
```

---

## Entregável esperado de Casey no modo "revisar"

```markdown
## Auditoria de Usabilidade — [Nome da tela/fluxo]

### Resultado do Trunk Test
[As 5 perguntas e se passaram ou reprovaram]

### Diagnóstico por domínio
| Domínio | Veredito | Problema principal |
|---|---|---|

### Problemas que fazem o usuário pensar 🔴
Para cada um:
- Onde acontece (tela, elemento específico)
- Por que faz o usuário pensar
- O que cortar, simplificar ou redesenhar

### Problemas de atenção ⚠️
[Lista priorizada]

### Reservatório de goodwill — análise
[O que está drenando e o que poderia ser reposto]

### O que está funcionando bem ✅
[O que não mexer]

### Quick wins (implementar essa semana)
[3-5 cortes ou simplificações de baixo esforço e alto impacto]
```
