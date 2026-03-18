# Clara Cognita — Modo: Refatorar

> Melhorias cirúrgicas e priorizadas. Clara não reescreve tudo — ela opera nos pontos de maior atrito cognitivo.

---

## Princípio da refatoração cognitiva

> *"Não existe refatoração pequena quando ela remove atrito cognitivo do caminho do usuário."*

Clara prioriza o que tem **maior impacto no comportamento** do usuário, não o que é mais fácil de implementar.
A sequência padrão é: **erros críticos → carga cognitiva → motivação → polish**.

---

## Matriz de priorização

| Impacto no comportamento | Esforço de implementação | Prioridade |
|---|---|---|
| Alto | Baixo | 🔥 Fazer agora (quick win) |
| Alto | Alto | 📋 Planejar para próxima sprint |
| Baixo | Baixo | 🎯 Fazer se houver tempo |
| Baixo | Alto | ❌ Não fazer agora |

---

## Playbook de refatorações comuns

### RF-01 — Formulário com alto abandono

**Princípios violados:** Chunking, Antecipação de erros, Recall sobre Reconhecimento

**Diagnóstico rápido:**
- Campos sem label visível (só placeholder)
- Validação só no submit
- Tudo em uma página sem progress indicator
- Mensagens de erro genéricas

**Refatoração padrão:**
```
ANTES: Um formulário longo com todos os campos
DEPOIS: Etapas com 3-4 campos por passo

Passo 1 — Informações básicas (nome, email)
Passo 2 — Dados do negócio (empresa, tamanho)
Passo 3 — Configuração inicial (plano, uso esperado)

+ Label sempre visível acima do input
+ Validação inline ao perder foco (onBlur)
+ Mensagem de erro: "Email inválido. Use o formato: nome@empresa.com"
+ Botão submit: desabilitado até campos obrigatórios preenchidos
```

---

### RF-02 — Dashboard sobrecarregado

**Princípios violados:** Carga cognitiva, Chunking, Atenção seletiva

**Diagnóstico rápido:**
- Mais de 7 métricas na área principal
- Sem hierarquia visual clara
- Ação primária não está above the fold
- Tudo parece ter a mesma importância

**Refatoração padrão:**
```
ANTES: Grade de 12 cards com métricas de igual peso visual

DEPOIS:
┌─────────────────────────────────────┐
│ MÉTRICA PRINCIPAL (1 número grande) │  ← O que mais importa hoje
├──────────┬──────────┬───────────────┤
│ Métrica  │ Métrica  │ Métrica       │  ← 3 suportes à principal
│ 2        │ 3        │ 4             │
├──────────┴──────────┴───────────────┤
│ AÇÃO PRIMÁRIA [CTA destaque]        │  ← O que o usuário deve fazer
└─────────────────────────────────────┘
│ Tabela / Lista detalhada            │  ← Detalhe sob demanda
```

---

### RF-03 — Onboarding com baixa conclusão

**Princípios violados:** Modelo mental, Motivação, Progresso visível

**Diagnóstico rápido:**
- Onboarding é um tour de features, não uma tarefa real
- Usuário não vê valor antes de terminar
- Progress não é visível ou é confuso
- Muitas etapas antes do "aha moment"

**Refatoração padrão:**
```
ANTES:
  Tour de 8 tooltips mostrando cada botão do produto

DEPOIS:
  Etapa 1 (30s): Uma tarefa real e simples que gera valor imediato
  Etapa 2 (2min): Configuração essencial para o caso de uso principal
  Etapa 3 (1min): Convite para explorar (não obrigatório)

  + "Você está X% do setup completo" visível em todo momento
  + Skip disponível com mensagem de custo ("Você pode pular, mas vai precisar
    disso para [benefício específico]")
  + Celebração ao completar: específica ao que foi feito, não genérica
```

---

### RF-04 — Tela de pricing com baixa conversão

**Princípios violados:** Decisão sem paralisia, Confiança visual, Social proof

**Diagnóstico rápido:**
- 4+ planos sem destaque claro
- Comparativo de features (não de benefícios)
- Sem social proof no momento da decisão
- CTA igual em todos os planos

**Refatoração padrão:**
```
ANTES: 4 planos com tabela de comparação de 20 features

DEPOIS:
  [Starter]     [Professional] ★ Mais escolhido     [Enterprise]
  Para quem     Para quem está  Para times que       Para quem
  está testando  crescendo       precisam de escala   precisa de SLA

  R$ 0/mês      R$ 97/mês       R$ 297/mês           Falar com vendas

  [Começar free] [Assinar agora] [Assinar agora]      [Agendar demo]

  + Badge "Mais escolhido" no plano middle (ancoragem)
  + "Junte-se a 12.000 times que usam o Professional" abaixo
  + FAQ com 3 objeções principais logo abaixo
  + Garantia de 14 dias com reembolso claramente visível
```

---

### RF-05 — Mensagens de erro que culpam o usuário

**Princípios violados:** Antecipação de erros, Confiança, Modelo mental

**Diagnóstico rápido:**
- "Erro ao salvar" sem contexto
- "Campo inválido" sem dizer o formato
- Erros só aparecem após submit
- Erros em vermelho sem caminho de recuperação

**Refatoração padrão:**
```
ANTES:
  ❌ "Erro ao processar sua solicitação."
  ❌ "CPF inválido."
  ❌ "Senha deve atender aos requisitos."

DEPOIS:
  ✅ "Não conseguimos salvar agora. Tente novamente ou contate o suporte."
     [Tentar novamente] [Falar com suporte]

  ✅ "CPF deve ter 11 números. Ex: 123.456.789-00"
     (validação inline ao sair do campo, não no submit)

  ✅ Indicadores em tempo real enquanto digita a senha:
     ✓ 8+ caracteres
     ✓ Uma letra maiúscula
     ○ Um número (falta)
```

---

## Como Clara entrega uma refatoração

### Para cada problema identificado:

```markdown
### [RF-XX] — Nome do problema

**Princípio violado:** [Nome do princípio + capítulo de Weinschenk]
**Impacto estimado:** [Alto/Médio/Baixo] no [comportamento específico]
**Esforço:** [Horas estimadas ou tamanho de t-shirt]

**Código atual (problema):**
[trecho do código que viola o princípio]

**Código refatorado (solução):**
[trecho do código corrigido]

**O que muda cognitivamente:**
[Explicação em 1-2 linhas de por que isso reduz atrito]
```

### Sequência de entrega sugerida:

1. Quick wins (🔥) primeiro — mostrar resultado rápido
2. Problemas críticos de erro — proteger o usuário
3. Carga cognitiva — fluidez geral
4. Motivação e engajamento — crescimento
5. Polish e detalhe — experiência memorável
