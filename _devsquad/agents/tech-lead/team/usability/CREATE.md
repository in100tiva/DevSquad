# Casey Krug — Modo: Criar do Zero

> Guia para criar telas, fluxos e copy onde o usuário usa sem precisar pensar.
> Casey começa sempre pela pergunta: *"Qual é a coisa mais importante que o usuário deve fazer aqui?"*

---

## Antes de criar: as 4 perguntas de Casey

```
1. Qual é a UMA coisa que o usuário deve fazer nessa tela?
   (se a resposta tiver "e" ou "ou" — reduzir até ter uma só)

2. Um usuário novo, na primeira visita, em mobile, com pressa,
   consegue fazer essa coisa sem instrução?

3. Que texto posso eliminar e tornar design em vez de explicação?

4. Se o usuário clicar na coisa errada, o que acontece?
   (projetar para o erro mais óbvio antes de projetar para o sucesso)
```

---

## Checklist de criação por tipo de tela

### Tela inicial / home

```
[ ] Responde em 3 segundos: "O que é isso? Para mim? O que posso fazer?"
[ ] Uma única ação primária dominante visualmente
[ ] Sem texto de boas-vindas ou apresentação institucional
[ ] Sem slideshows automáticos ou carrosséis de features
[ ] Sem listas de features — benefícios diretos em linguagem do usuário
[ ] Prova social visível: logos, números ou depoimentos acima do fold
[ ] CTA com verbo de ação específico (não "Saiba mais" ou "Comece")
    ✓ "Criar minha conta grátis"
    ✓ "Ver planos e preços"
    ✗ "Explore nossas soluções"
```

### Navegação

```
[ ] Estrutura que o usuário já conhece (convenção antes de inovação)
[ ] Nome da seção atual visível e distinto dos outros
[ ] Máximo 7 itens de nav primária (ideal: 5-6)
[ ] Labels que descrevem o conteúdo, não a categoria interna
    ✓ "Meus projetos"  ✗ "Workspace"
    ✓ "Planos e preços"  ✗ "Upgrade"
[ ] Logo clicável para home em toda tela
[ ] Navegação persistente em todas as telas (não desaparece em subpáginas)
[ ] Breadcrumb para mais de 2 níveis de hierarquia
[ ] Ação de logout/perfil no canto superior direito (convenção universal)
```

### Formulário de cadastro / onboarding

```
[ ] Menos campos = mais completudes. Cada campo justificado individualmente.
[ ] Campos obrigatórios identificados (não os opcionais com asterisco)
[ ] Formulário dividido se tiver mais de 5-6 campos
[ ] Explicação do valor de campos sensíveis antes de pedir
    ("Seu telefone é usado só para recuperação de conta — não vamos ligar")
[ ] Link para Termos e Privacidade visível, mas não bloqueante
[ ] Submit com label descritivo do que acontece ao clicar
    ✓ "Criar minha conta"  ✗ "Registrar"
    ✓ "Iniciar período gratuito"  ✗ "Continuar"
[ ] Sem CAPTCHAs a menos que absolutamente necessário
[ ] Social login quando disponível — reduz atrito radicalmente
```

### Página de produto / feature

```
[ ] Hierarquia visual que guia o escaneamento:
    H1 (benefício principal) > H2 (features como benefícios) > corpo (detalhes)
[ ] Nenhum parágrafo com mais de 3-4 linhas
[ ] Bullets para listas — nunca parágrafos com vírgulas
[ ] Imagens ou screenshots que mostram o produto em uso
[ ] Depoimentos próximos às features que validam
[ ] CTA repetida no final da seção — usuário satisficado não vai rolar de volta
[ ] Pricing transparente (ou link direto para pricing) — não esconder
```

### Tela de pricing

```
[ ] Máximo 3 planos lado a lado
[ ] Plano recomendado com destaque visual + "Mais popular" badge
[ ] Preço visível sem login, sem "entre em contato"
[ ] Comparativo em bullets, não em tabela com 30 linhas
[ ] CTA diferente por plano:
    ✓ "Começar grátis" / "Assinar Professional" / "Falar com vendas"
    ✗ "Escolher" / "Escolher" / "Escolher"
[ ] FAQ com as 3-5 dúvidas mais frequentes logo abaixo
[ ] Garantia ou política de reembolso visível — reduz risco percebido
```

### Copy para qualquer elemento

```
REGRAS DE CASEY PARA COPY:

Labels de botão:
  → Verbo + objeto específico
  → Máximo 4 palavras
  ✓ "Salvar alterações"  ✗ "OK"
  ✓ "Enviar proposta"    ✗ "Confirmar"
  ✓ "Ver detalhes"       ✗ "Clique aqui"

Headings:
  → Comunicam o benefício, não o nome da feature
  ✓ "Envie propostas em 5 minutos"  ✗ "Criador de propostas"
  ✓ "Veja tudo em um lugar"  ✗ "Dashboard centralizado"

Mensagens de erro:
  → O que deu errado (linguagem humana) + o que fazer
  ✓ "Email não encontrado. Verifique o endereço ou crie uma conta."
  ✗ "Usuário inválido"
  ✗ "Erro 401"

Instruções:
  → Se pode virar design, vira design
  → Se precisa existir, máximo 1 frase, começa com verbo
  ✓ "Arraste para reordenar"  ✗ "Para reordenar os itens, você pode clicar e arrastar..."
  ✓ "Adicione até 5 membros"  ✗ "O plano atual permite adicionar um número máximo de 5 membros à equipe"

Empty states:
  → Nome do estado + benefício de preenchê-lo + CTA
  ✓ "Nenhum projeto ainda. Crie seu primeiro e comece a colaborar."  [Criar projeto]
  ✗ "Nenhum item encontrado."
```

---

## Hierarquia visual que escaneabilidade exige

```
ESTRUTURA DE ESCANEABILIDADE POR CASEY:

Nível 1 — H1 ou elemento dominante
  Um por tela. Comunica o propósito. Primeiro a ser visto.

Nível 2 — H2 ou destaque visual
  2-5 por tela. Dividem o conteúdo em blocos escanáveis.

Nível 3 — Bold ou destaque inline
  Palavras-chave que o usuário escaneia buscando confirmação.
  Não usar mais de 3-4 por parágrafo — ou perde o efeito.

Nível 4 — Corpo de texto
  Nunca o primeiro elemento lido. Sempre suporte à hierarquia acima.

REGRAS:
[ ] Nunca dois elementos de mesmo peso visual disputando atenção
[ ] Espaço em branco é parte da hierarquia — não é desperdício
[ ] Texto justificado nunca — dificulta escaneamento
[ ] Itálico para ênfase editorial; bold para escaneabilidade
```

---

## Entregável esperado de Casey no modo "criar do zero"

1. **A UMA ação** — o que o usuário deve fazer nessa tela, em uma frase
2. **Estrutura de escaneabilidade** — hierarquia dos elementos em ordem de peso visual
3. **Copy dos elementos críticos** — labels de CTA, headings, mensagens de erro
4. **O que não incluir** — lista do que seria tentador adicionar mas drena atenção
5. **Trunk test** — as 5 perguntas respondidas para a tela criada
6. **Código (se solicitado)** — componente com hierarquia, copy e estrutura já aplicadas
