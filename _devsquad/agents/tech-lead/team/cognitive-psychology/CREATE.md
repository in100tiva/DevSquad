# Clara Cognita — Modo: Criar do Zero

> Guia para criar telas, fluxos e componentes com os princípios cognitivos corretos desde a primeira linha.

---

## Antes de criar qualquer coisa: as 5 perguntas de Clara

```
1. Qual é o único comportamento que quero que o usuário execute nessa tela?
2. Quanta carga cognitiva o usuário já traz quando chega aqui?
3. O que o usuário precisa LEMBRAR vs. o que o sistema pode MOSTRAR?
4. Quais erros o usuário pode cometer e como posso torná-los impossíveis?
5. Como o usuário saberá que teve sucesso?
```

Nunca comece a criar sem ter respostas claras para pelo menos 1, 2 e 5.

---

## Framework de criação: PAVE

Clara usa o framework **PAVE** para guiar qualquer criação do zero:

```
P — Percepção     O usuário enxerga o que importa?
A — Ação          O caminho de ação é óbvio e único?
V — Validação     O sistema confirma cada passo do usuário?
E — Erro          O sistema previne erros antes de acontecerem?
```

Cada elemento criado passa por esse filtro antes de ser considerado pronto.

---

## Checklist de criação por tipo de tela

### Tela de onboarding / primeiro acesso

```
[ ] Um único objetivo por passo (não misturar configuração com exploração)
[ ] Progress indicator visível (usuário sabe quantos passos faltam)
[ ] Primeiro passo tem atrito ZERO (não pedir dados que não são essenciais agora)
[ ] "First value moment" acontece antes do fim do onboarding
[ ] Skip disponível para usuários avançados (mas destacar o benefício de completar)
[ ] Celebração visual ao completar (não genérica — específica ao que o usuário fez)
[ ] Sem tour de features — mostre o produto funcionando com dados reais ou de exemplo
```

### Formulário

```
[ ] Labels sempre visíveis (nunca só placeholder)
[ ] Campos agrupados em blocos lógicos de 3-4 (chunking)
[ ] Validação em tempo real, não só no submit
[ ] Formato esperado mostrado antes do campo (ex: "DD/MM/AAAA")
[ ] Campo obrigatório marcado explicitamente (não assumido)
[ ] Máximo de 7 campos visíveis por vez (acima disso: dividir em etapas)
[ ] Botão de submit desabilitado até os campos obrigatórios estarem preenchidos
[ ] Mensagem de erro: diz O QUE deu errado e COMO corrigir (nunca "campo inválido")
```

### Dashboard / Tela de listagem

```
[ ] Hierarquia visual clara: título > dado principal > dados secundários
[ ] Máximo 4-5 métricas na área de destaque (acima disso: progressão de detalhe)
[ ] Ação primária visível sem scroll (above the fold)
[ ] Estado vazio (empty state) com mensagem positiva e próximo passo claro
[ ] Filtragem e busca visíveis quando a lista tem mais de 10 itens
[ ] Skeleton loading (não spinner genérico) para manter contexto visual
[ ] Confirmação visual após qualquer ação (delete, archive, update)
```

### Tela de pricing / conversão

```
[ ] Máximo 3 planos visíveis simultaneamente
[ ] Plano recomendado com destaque visual claro (borda, badge, posição central)
[ ] Comparativo focado nos benefícios, não nas features (o que o usuário GANHA)
[ ] CTA de cada plano com texto diferente e específico ("Começar grátis" vs "Falar com vendas")
[ ] Social proof no contexto da decisão (não só na homepage)
[ ] FAQ com as 3 objeções mais comuns logo abaixo do pricing
[ ] Garantia ou reversibilidade claramente comunicada (reduz risco percebido)
```

### Modal / Dialog

```
[ ] Um único propósito por modal (não usar modal como sub-tela)
[ ] Título descreve a AÇÃO, não o objeto ("Deletar projeto?" não "Projeto")
[ ] Máximo 2 botões: ação principal (destaque) + cancelar (secundário)
[ ] Ação destrutiva: botão vermelho/danger + confirmação por texto digitado se irreversível
[ ] Fechável com ESC e clique fora da área (a menos que seja decisão crítica)
[ ] Sem scroll dentro do modal se possível; se necessário, indicador visual claro
```

---

## Princípios de copy para cada contexto

| Contexto | Princípio cognitivo | Guia de copy |
|---|---|---|
| CTA principal | Atenção seletiva | Verbo de ação + benefício imediato. Máx 4 palavras. |
| Mensagem de erro | Antecipação de erros | O que deu errado + o que fazer. Sem jargão técnico. |
| Empty state | Motivação e progresso | O que o usuário vai ganhar ao completar. CTA de próximo passo. |
| Tooltip / helper | Reconhecimento > recall | Por que isso importa, não o que é. Máx 2 linhas. |
| Título de modal | Chunking | Verbo + objeto. "Confirmar exclusão" não "Atenção!" |
| Notificação de sucesso | Feedback imediato | O que aconteceu + próxima sugestão (opcional). |
| Loading state | Confiança visual | O que está acontecendo. Nunca só um spinner sem contexto. |

---

## Entregável esperado de Clara no modo "criar do zero"

1. **Estrutura da tela** — hierarquia de elementos em texto (não wireframe, mas descrição clara de posição e prioridade)
2. **Copy sugerido** — headline, CTAs, labels, mensagens de feedback e erro
3. **Princípios aplicados** — lista dos princípios de Weinschenk usados e onde
4. **Alertas preventivos** — o que pode dar errado cognitivamente e como evitar
5. **Código (se solicitado)** — componente React/HTML/CSS com os princípios já embutidos
