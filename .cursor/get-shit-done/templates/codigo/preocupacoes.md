# Template de Preocupações do Código

Template para `.planning/codebase/CONCERNS.md` - captura problemas conhecidos e áreas que requerem cuidado.

**Propósito:** Evidenciar avisos acionáveis sobre o código-fonte. Focado em "o que observar ao fazer mudanças."

---

## Template do Arquivo

```markdown
# Preocupações do Código

**Data da Análise:** [AAAA-MM-DD]

## Dívida Técnica

**[Área/Componente]:**
- Problema: [Qual é o atalho/solução alternativa]
- Por quê: [Por que foi feito dessa forma]
- Impacto: [O que quebra ou degrada por causa disso]
- Abordagem de correção: [Como resolver adequadamente]

**[Área/Componente]:**
- Problema: [Qual é o atalho/solução alternativa]
- Por quê: [Por que foi feito dessa forma]
- Impacto: [O que quebra ou degrada por causa disso]
- Abordagem de correção: [Como resolver adequadamente]

## Bugs Conhecidos

**[Descrição do bug]:**
- Sintomas: [O que acontece]
- Gatilho: [Como reproduzir]
- Solução alternativa: [Mitigação temporária, se houver]
- Causa raiz: [Se conhecida]
- Bloqueado por: [Se aguardando algo]

**[Descrição do bug]:**
- Sintomas: [O que acontece]
- Gatilho: [Como reproduzir]
- Solução alternativa: [Mitigação temporária, se houver]
- Causa raiz: [Se conhecida]

## Considerações de Segurança

**[Área que requer cuidado de segurança]:**
- Risco: [O que pode dar errado]
- Mitigação atual: [O que está em vigor agora]
- Recomendações: [O que deve ser adicionado]

**[Área que requer cuidado de segurança]:**
- Risco: [O que pode dar errado]
- Mitigação atual: [O que está em vigor agora]
- Recomendações: [O que deve ser adicionado]

## Gargalos de Performance

**[Operação/endpoint lento]:**
- Problema: [O que está lento]
- Medição: [Números reais: "500ms p95", "2s tempo de carregamento"]
- Causa: [Por que está lento]
- Caminho de melhoria: [Como acelerar]

**[Operação/endpoint lento]:**
- Problema: [O que está lento]
- Medição: [Números reais]
- Causa: [Por que está lento]
- Caminho de melhoria: [Como acelerar]

## Áreas Frágeis

**[Componente/Módulo]:**
- Por que frágil: [O que o faz quebrar facilmente]
- Falhas comuns: [O que tipicamente dá errado]
- Modificação segura: [Como alterar sem quebrar]
- Cobertura de testes: [Está testado? Lacunas?]

**[Componente/Módulo]:**
- Por que frágil: [O que o faz quebrar facilmente]
- Falhas comuns: [O que tipicamente dá errado]
- Modificação segura: [Como alterar sem quebrar]
- Cobertura de testes: [Está testado? Lacunas?]

## Limites de Escala

**[Recurso/Sistema]:**
- Capacidade atual: [Números: "100 req/s", "10k usuários"]
- Limite: [Onde quebra]
- Sintomas no limite: [O que acontece]
- Caminho de escala: [Como aumentar capacidade]

## Dependências em Risco

**[Pacote/Serviço]:**
- Risco: [ex., "depreciado", "sem manutenção", "breaking changes chegando"]
- Impacto: [O que quebra se falhar]
- Plano de migração: [Alternativa ou caminho de atualização]

## Funcionalidades Críticas Ausentes

**[Lacuna de funcionalidade]:**
- Problema: [O que está faltando]
- Solução alternativa atual: [Como os usuários lidam]
- Bloqueia: [O que não pode ser feito sem isso]
- Complexidade de implementação: [Estimativa aproximada de esforço]

## Lacunas na Cobertura de Testes

**[Área não testada]:**
- O que não está testado: [Funcionalidade específica]
- Risco: [O que pode quebrar despercebido]
- Prioridade: [Alta/Média/Baixa]
- Dificuldade para testar: [Por que ainda não está testado]

---

*Auditoria de preocupações: [data]*
*Atualize conforme problemas são corrigidos ou novos são descobertos*
```

<good_examples>
```markdown
# Preocupações do Código

**Data da Análise:** 2025-01-20

## Dívida Técnica

**Queries de banco de dados em componentes React:**
- Problema: Queries diretas ao Supabase em 15+ componentes de página ao invés de server actions
- Arquivos: `app/dashboard/page.tsx`, `app/profile/page.tsx`, `app/courses/[id]/page.tsx`, `app/settings/page.tsx` (e mais 11 em `app/`)
- Por quê: Prototipagem rápida durante fase de MVP
- Impacto: Não é possível implementar RLS adequadamente, expõe estrutura do DB ao cliente
- Abordagem de correção: Mover todas as queries para server actions em `app/actions/`, adicionar políticas RLS adequadas

**Validação manual de assinatura de webhook:**
- Problema: Código de verificação de webhook do Stripe copiado e colado em 3 endpoints diferentes
- Arquivos: `app/api/webhooks/stripe/route.ts`, `app/api/webhooks/checkout/route.ts`, `app/api/webhooks/subscription/route.ts`
- Por quê: Cada webhook adicionado ad-hoc sem abstração
- Impacto: Fácil esquecer verificação em novos webhooks (risco de segurança)
- Abordagem de correção: Criar middleware compartilhado `lib/stripe/validate-webhook.ts`

## Bugs Conhecidos

**Condição de corrida na atualização de assinatura:**
- Sintomas: Usuário aparece como tier "gratuito" por 5-10 segundos após pagamento bem-sucedido
- Gatilho: Navegação rápida após redirecionamento do checkout Stripe, antes do webhook processar
- Arquivos: `app/checkout/success/page.tsx` (handler de redirecionamento), `app/api/webhooks/stripe/route.ts` (webhook)
- Solução alternativa: Webhook do Stripe eventualmente atualiza o status (auto-recuperação)
- Causa raiz: Processamento do webhook mais lento que navegação do usuário, sem atualização otimista da UI
- Correção: Adicionar polling em `app/checkout/success/page.tsx` após redirecionamento

**Estado de sessão inconsistente após logout:**
- Sintomas: Usuário redirecionado para /dashboard após logout ao invés de /login
- Gatilho: Logout via botão no nav mobile (desktop funciona normalmente)
- Arquivo: `components/MobileNav.tsx` (linha ~45, handler de logout)
- Solução alternativa: Navegação manual pela URL para /login funciona
- Causa raiz: Componente nav mobile não aguardando supabase.auth.signOut()
- Correção: Adicionar await no handler de logout em `components/MobileNav.tsx`

## Considerações de Segurança

**Verificação de role admin apenas no client-side:**
- Risco: Páginas do dashboard admin verificam isAdmin do cliente Supabase, sem verificação no servidor
- Arquivos: `app/admin/page.tsx`, `app/admin/users/page.tsx`, `components/AdminGuard.tsx`
- Mitigação atual: Nenhuma (dependendo apenas de ocultação na UI)
- Recomendações: Adicionar middleware nas rotas admin em `middleware.ts`, verificar role no servidor

**Uploads de arquivo sem validação:**
- Risco: Usuários podem fazer upload de qualquer tipo de arquivo para o bucket de avatar (sem validação de tamanho/tipo)
- Arquivo: `components/AvatarUpload.tsx` (handler de upload)
- Mitigação atual: Limite do bucket Supabase de 2MB (configurado no dashboard)
- Recomendações: Adicionar validação de tipo de arquivo (somente image/*) em `lib/storage/validate.ts`

## Gargalos de Performance

**Endpoint /api/courses:**
- Problema: Buscando todos os cursos com lições e autores aninhados
- Arquivo: `app/api/courses/route.ts`
- Medição: 1.2s p95 de tempo de resposta com 50+ cursos
- Causa: Padrão de query N+1 (query separada por curso para lições)
- Caminho de melhoria: Usar include do Prisma para eager-load lições em `lib/db/courses.ts`, adicionar cache Redis

**Carregamento inicial do Dashboard:**
- Problema: Cascata de 5 chamadas de API seriais no mount
- Arquivo: `app/dashboard/page.tsx`
- Medição: 3.5s até interativo em 3G lento
- Causa: Cada componente busca seus próprios dados independentemente
- Caminho de melhoria: Converter para Server Component com fetch paralelo único

## Áreas Frágeis

**Cadeia de middleware de autenticação:**
- Arquivo: `middleware.ts`
- Por que frágil: 4 funções de middleware diferentes executam em ordem específica (auth -> role -> subscription -> logging)
- Falhas comuns: Mudança na ordem do middleware quebra tudo, difícil de debugar
- Modificação segura: Adicionar testes antes de mudar a ordem, documentar dependências em comentários
- Cobertura de testes: Sem testes de integração para a cadeia de middleware (apenas testes unitários)

**Tratamento de eventos de webhook do Stripe:**
- Arquivo: `app/api/webhooks/stripe/route.ts`
- Por que frágil: Switch statement gigante com 12 tipos de evento, lógica de transação compartilhada
- Falhas comuns: Novo tipo de evento adicionado sem tratamento, atualizações parciais no DB em erro
- Modificação segura: Extrair cada handler de evento para `lib/stripe/handlers/*.ts`
- Cobertura de testes: Apenas 3 de 12 tipos de evento têm testes

## Limites de Escala

**Tier Gratuito do Supabase:**
- Capacidade atual: 500MB banco de dados, 1GB armazenamento de arquivos, 2GB largura de banda/mês
- Limite: ~5000 usuários estimados antes de atingir limites
- Sintomas no limite: Erros de rate limit 429, escritas no DB falham
- Caminho de escala: Upgrade para Pro (R$125/mês) estende para 8GB DB, 100GB armazenamento

**Bloqueio de renderização server-side:**
- Capacidade atual: ~50 usuários simultâneos antes de lentidão
- Limite: Plano Hobby da Vercel (timeout de função 10s, 100GB-hrs/mês)
- Sintomas no limite: Timeouts 504 de gateway nas páginas de curso
- Caminho de escala: Upgrade para Vercel Pro (R$100/mês), adicionar cache edge

## Dependências em Risco

**react-hot-toast:**
- Risco: Sem manutenção (última atualização há 18 meses), compatibilidade com React 19 desconhecida
- Impacto: Notificações toast quebram, sem degradação graceful
- Plano de migração: Trocar para sonner (mantido ativamente, API similar)

## Funcionalidades Críticas Ausentes

**Tratamento de falha de pagamento:**
- Problema: Sem mecanismo de retry ou notificação ao usuário quando pagamento de assinatura falha
- Solução alternativa atual: Usuários manualmente reinserem dados de pagamento (se perceberem)
- Bloqueia: Não é possível reter usuários com cartões expirados, sem processo de dunning
- Complexidade de implementação: Média (webhooks Stripe + fluxo de email + UI)

**Rastreamento de progresso do curso:**
- Problema: Sem estado persistente de quais lições foram concluídas
- Solução alternativa atual: Usuários rastreiam progresso manualmente
- Bloqueia: Não é possível mostrar percentual de conclusão, recomendar próxima lição
- Complexidade de implementação: Baixa (adicionar tabela de junção completed_lessons)

## Lacunas na Cobertura de Testes

**Fluxo de pagamento end-to-end:**
- O que não está testado: Fluxo completo checkout Stripe -> webhook -> ativação de assinatura
- Risco: Processamento de pagamento pode quebrar silenciosamente (já aconteceu duas vezes)
- Prioridade: Alta
- Dificuldade para testar: Precisa de fixtures de teste do Stripe e configuração de simulação de webhook

**Comportamento de error boundary:**
- O que não está testado: Como o app se comporta quando componentes lançam erros
- Risco: Tela branca da morte para usuários, sem reporte de erros
- Prioridade: Média
- Dificuldade para testar: Precisa disparar erros intencionalmente no ambiente de teste

---

*Auditoria de preocupações: 2025-01-20*
*Atualize conforme problemas são corrigidos ou novos são descobertos*
```
</good_examples>

<guidelines>
**O que pertence ao CONCERNS.md:**
- Dívida técnica com impacto claro e abordagem de correção
- Bugs conhecidos com passos de reprodução
- Lacunas de segurança e recomendações de mitigação
- Gargalos de performance com medições
- Código frágil que quebra facilmente
- Limites de escala com números
- Dependências que precisam de atenção
- Funcionalidades ausentes que bloqueiam fluxos de trabalho
- Lacunas na cobertura de testes

**O que NÃO pertence aqui:**
- Opiniões sem evidência ("código está bagunçado")
- Reclamações sem soluções ("auth é ruim")
- Ideias de funcionalidades futuras (isso é para planejamento de produto)
- TODOs normais (esses ficam em comentários de código)
- Decisões arquiteturais que estão funcionando bem
- Problemas menores de estilo de código

**Ao preencher este template:**
- **Sempre inclua caminhos de arquivo** - Preocupações sem localização não são acionáveis. Use crases: `src/file.ts`
- Seja específico com medições ("500ms p95" não "lento")
- Inclua passos de reprodução para bugs
- Sugira abordagens de correção, não apenas problemas
- Foque em itens acionáveis
- Priorize por risco/impacto
- Atualize conforme problemas são resolvidos
- Adicione novas preocupações conforme descobertas

**Diretrizes de tom:**
- Profissional, não emocional ("padrão de query N+1" não "queries terríveis")
- Orientado a soluções ("Correção: adicionar índice" não "precisa ser corrigido")
- Focado em risco ("Pode expor dados de usuário" não "segurança está ruim")
- Factual ("3.5s de tempo de carregamento" não "muito lento")

**Útil para planejamento de fase quando:**
- Decidindo o que trabalhar a seguir
- Estimando risco de mudanças
- Entendendo onde ter cuidado
- Priorizando melhorias
- Integrando novos contextos do Claude
- Planejando trabalho de refatoração

**Como é preenchido:**
Agentes de exploração detectam esses itens durante mapeamento do código. Adições manuais são bem-vindas para problemas descobertos por humanos. Esta é documentação viva, não uma lista de reclamações.
</guidelines>
