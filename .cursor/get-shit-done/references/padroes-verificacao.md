# Padrões de Verificação

Como verificar que diferentes tipos de artefatos são implementações reais, não stubs ou placeholders.

<core_principle>
**Existência ≠ Implementação**

Um arquivo existir não significa que a funcionalidade funciona. Verificação deve checar:
1. **Existe** - Arquivo está presente no caminho esperado
2. **Substancial** - Conteúdo é implementação real, não placeholder
3. **Conectado** - Ligado ao resto do sistema
4. **Funcional** - Realmente funciona quando invocado

Níveis 1-3 podem ser verificados programaticamente. Nível 4 frequentemente requer verificação humana.
</core_principle>

<stub_detection>

## Padrões Universais de Stub

Estes padrões indicam código placeholder independente do tipo de arquivo:

**Stubs baseados em comentário:**
```bash
# Padrões grep para comentários stub
grep -E "(TODO|FIXME|XXX|HACK|PLACEHOLDER)" "$file"
grep -E "implement|add later|coming soon|will be" "$file" -i
grep -E "// \.\.\.|/\* \.\.\. \*/|# \.\.\." "$file"
```

**Texto placeholder na saída:**
```bash
# Padrões de placeholder de UI
grep -E "placeholder|lorem ipsum|coming soon|under construction" "$file" -i
grep -E "sample|example|test data|dummy" "$file" -i
grep -E "\[.*\]|<.*>|\{.*\}" "$file"  # Colchetes de template restantes
```

**Implementações vazias ou triviais:**
```bash
# Funções que não fazem nada
grep -E "return null|return undefined|return \{\}|return \[\]" "$file"
grep -E "pass$|\.\.\.|\bnothing\b" "$file"
grep -E "console\.(log|warn|error).*only" "$file"  # Funções somente-log
```

**Valores hardcoded onde dinâmico é esperado:**
```bash
# IDs, contagens ou conteúdo hardcoded
grep -E "id.*=.*['\"].*['\"]" "$file"  # IDs de string hardcoded
grep -E "count.*=.*\d+|length.*=.*\d+" "$file"  # Contagens hardcoded
grep -E "\\\$\d+\.\d{2}|\d+ items" "$file"  # Valores de exibição hardcoded
```

</stub_detection>

<react_components>

## Componentes React/Next.js

**Verificação de existência:**
```bash
# Arquivo existe e exporta componente
[ -f "$component_path" ] && grep -E "export (default |)function|export const.*=.*\(" "$component_path"
```

**Verificação substancial:**
```bash
# Retorna JSX real, não placeholder
grep -E "return.*<" "$component_path" | grep -v "return.*null" | grep -v "placeholder" -i

# Tem conteúdo significativo (não apenas wrapper div)
grep -E "<[A-Z][a-zA-Z]+|className=|onClick=|onChange=" "$component_path"

# Usa props ou state (não estático)
grep -E "props\.|useState|useEffect|useContext|\{.*\}" "$component_path"
```

**Padrões de stub específicos de React:**
```javascript
// RED FLAGS - Estes são stubs:
return <div>Component</div>
return <div>Placeholder</div>
return <div>{/* TODO */}</div>
return <p>Coming soon</p>
return null
return <></>

// Também stubs - handlers vazios:
onClick={() => {}}
onChange={() => console.log('clicked')}
onSubmit={(e) => e.preventDefault()}  // Apenas previne default, não faz nada
```

**Verificação de conexão:**
```bash
# Componente importa o que precisa
grep -E "^import.*from" "$component_path"

# Props são realmente usados (não apenas recebidos)
# Procurar por destructuring ou uso de props.X
grep -E "\{ .* \}.*props|\bprops\.[a-zA-Z]+" "$component_path"

# Chamadas de API existem (para componentes que buscam dados)
grep -E "fetch\(|axios\.|useSWR|useQuery|getServerSideProps|getStaticProps" "$component_path"
```

**Verificação funcional (requer humano):**
- O componente renderiza conteúdo visível?
- Elementos interativos respondem a cliques?
- Dados carregam e exibem?
- Estados de erro aparecem apropriadamente?

</react_components>

<api_routes>

## Rotas de API (Next.js App Router / Express / etc.)

**Verificação de existência:**
```bash
# Arquivo de rota existe
[ -f "$route_path" ]

# Exporta handlers de método HTTP (Next.js App Router)
grep -E "export (async )?(function|const) (GET|POST|PUT|PATCH|DELETE)" "$route_path"

# Ou handlers estilo Express
grep -E "\.(get|post|put|patch|delete)\(" "$route_path"
```

**Verificação substancial:**
```bash
# Tem lógica real, não apenas declaração return
wc -l "$route_path"  # Mais de 10-15 linhas sugere implementação real

# Interage com fonte de dados
grep -E "prisma\.|db\.|mongoose\.|sql|query|find|create|update|delete" "$route_path" -i

# Tem tratamento de erros
grep -E "try|catch|throw|error|Error" "$route_path"

# Retorna resposta significativa
grep -E "Response\.json|res\.json|res\.send|return.*\{" "$route_path" | grep -v "message.*not implemented" -i
```

**Padrões de stub específicos de rotas de API:**
```typescript
// RED FLAGS - Estes são stubs:
export async function POST() {
  return Response.json({ message: "Not implemented" })
}

export async function GET() {
  return Response.json([])  // Array vazio sem query ao DB
}

export async function PUT() {
  return new Response()  // Resposta vazia
}

// Somente console log:
export async function POST(req) {
  console.log(await req.json())
  return Response.json({ ok: true })
}
```

**Verificação de conexão:**
```bash
# Importa clientes de database/serviço
grep -E "^import.*prisma|^import.*db|^import.*client" "$route_path"

# Realmente usa corpo do request (para POST/PUT)
grep -E "req\.json\(\)|req\.body|request\.json\(\)" "$route_path"

# Valida input (não apenas confiando no request)
grep -E "schema\.parse|validate|zod|yup|joi" "$route_path"
```

**Verificação funcional (humana ou automatizada):**
- GET retorna dados reais do banco de dados?
- POST realmente cria um registro?
- Resposta de erro tem código de status correto?
- Verificações de auth são realmente aplicadas?

</api_routes>

<database_schema>

## Schema de Banco de Dados (Prisma / Drizzle / SQL)

**Verificação de existência:**
```bash
# Arquivo de schema existe
[ -f "prisma/schema.prisma" ] || [ -f "drizzle/schema.ts" ] || [ -f "src/db/schema.sql" ]

# Modelo/tabela está definido
grep -E "^model $model_name|CREATE TABLE $table_name|export const $table_name" "$schema_path"
```

**Verificação substancial:**
```bash
# Tem campos esperados (não apenas id)
grep -A 20 "model $model_name" "$schema_path" | grep -E "^\s+\w+\s+\w+"

# Tem relacionamentos se esperado
grep -E "@relation|REFERENCES|FOREIGN KEY" "$schema_path"

# Tem tipos de campo apropriados (não tudo String)
grep -A 20 "model $model_name" "$schema_path" | grep -E "Int|DateTime|Boolean|Float|Decimal|Json"
```

**Padrões de stub específicos de schemas:**
```prisma
// RED FLAGS - Estes são stubs:
model User {
  id String @id
  // TODO: adicionar campos
}

model Message {
  id        String @id
  content   String  // Apenas um campo real
}

// Faltam campos críticos:
model Order {
  id     String @id
  // Sem: userId, items, total, status, createdAt
}
```

**Verificação de conexão:**
```bash
# Migrações existem e foram aplicadas
ls prisma/migrations/ 2>/dev/null | wc -l  # Deve ser > 0
npx prisma migrate status 2>/dev/null | grep -v "pending"

# Client foi gerado
[ -d "node_modules/.prisma/client" ]
```

**Verificação funcional:**
```bash
# Pode consultar a tabela (automatizado)
npx prisma db execute --stdin <<< "SELECT COUNT(*) FROM $table_name"
```

</database_schema>

<hooks_utilities>

## Hooks Customizados e Utilitários

**Verificação de existência:**
```bash
# Arquivo existe e exporta função
[ -f "$hook_path" ] && grep -E "export (default )?(function|const)" "$hook_path"
```

**Verificação substancial:**
```bash
# Hook usa React hooks (para hooks customizados)
grep -E "useState|useEffect|useCallback|useMemo|useRef|useContext" "$hook_path"

# Tem valor de retorno significativo
grep -E "return \{|return \[" "$hook_path"

# Mais que comprimento trivial
[ $(wc -l < "$hook_path") -gt 10 ]
```

**Padrões de stub específicos de hooks:**
```typescript
// RED FLAGS - Estes são stubs:
export function useAuth() {
  return { user: null, login: () => {}, logout: () => {} }
}

export function useCart() {
  const [items, setItems] = useState([])
  return { items, addItem: () => console.log('add'), removeItem: () => {} }
}

// Retorno hardcoded:
export function useUser() {
  return { name: "Test User", email: "test@example.com" }
}
```

**Verificação de conexão:**
```bash
# Hook é realmente importado em algum lugar
grep -r "import.*$hook_name" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"

# Hook é realmente chamado
grep -r "$hook_name()" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"
```

</hooks_utilities>

<environment_config>

## Variáveis de Ambiente e Configuração

**Verificação de existência:**
```bash
# Arquivo .env existe
[ -f ".env" ] || [ -f ".env.local" ]

# Variável obrigatória está definida
grep -E "^$VAR_NAME=" .env .env.local 2>/dev/null
```

**Verificação substancial:**
```bash
# Variável tem valor real (não placeholder)
grep -E "^$VAR_NAME=.+" .env .env.local 2>/dev/null | grep -v "your-.*-here|xxx|placeholder|TODO" -i

# Valor parece válido para o tipo:
# - URLs devem começar com http
# - Chaves devem ser longas o suficiente
# - Booleanos devem ser true/false
```

**Padrões de stub específicos de env:**
```bash
# RED FLAGS - Estes são stubs:
DATABASE_URL=your-database-url-here
STRIPE_SECRET_KEY=sk_test_xxx
API_KEY=placeholder
NEXT_PUBLIC_API_URL=http://localhost:3000  # Ainda apontando para localhost em prod
```

**Verificação de conexão:**
```bash
# Variável é realmente usada no código
grep -r "process\.env\.$VAR_NAME|env\.$VAR_NAME" src/ --include="*.ts" --include="*.tsx"

# Variável está no schema de validação (se usando zod/etc para env)
grep -E "$VAR_NAME" src/env.ts src/env.mjs 2>/dev/null
```

</environment_config>

<wiring_verification>

## Padrões de Verificação de Conexão

Verificação de conexão checa que componentes realmente se comunicam. É aqui que a maioria dos stubs se escondem.

### Padrão: Componente → API

**Verificar:** O componente realmente chama a API?

```bash
# Encontrar a chamada fetch/axios
grep -E "fetch\(['\"].*$api_path|axios\.(get|post).*$api_path" "$component_path"

# Verificar que não está comentado
grep -E "fetch\(|axios\." "$component_path" | grep -v "^.*//.*fetch"

# Verificar que a resposta é usada
grep -E "await.*fetch|\.then\(|setData|setState" "$component_path"
```

**Red flags:**
```typescript
// Fetch existe mas resposta ignorada:
fetch('/api/messages')  // Sem await, sem .then, sem atribuição

// Fetch em comentário:
// fetch('/api/messages').then(r => r.json()).then(setMessages)

// Fetch para endpoint errado:
fetch('/api/message')  // Typo - deveria ser /api/messages
```

### Padrão: API → Banco de Dados

**Verificar:** A rota de API realmente consulta o banco de dados?

```bash
# Encontrar a chamada ao banco de dados
grep -E "prisma\.$model|db\.query|Model\.find" "$route_path"

# Verificar que é awaited
grep -E "await.*prisma|await.*db\." "$route_path"

# Verificar que resultado é retornado
grep -E "return.*json.*data|res\.json.*result" "$route_path"
```

**Red flags:**
```typescript
// Query existe mas resultado não retornado:
await prisma.message.findMany()
return Response.json({ ok: true })  // Retorna estático, não resultado da query

// Query não awaited:
const messages = prisma.message.findMany()  // Faltando await
return Response.json(messages)  // Retorna Promise, não dados
```

### Padrão: Formulário → Handler

**Verificar:** O submit do formulário realmente faz algo?

```bash
# Encontrar handler onSubmit
grep -E "onSubmit=\{|handleSubmit" "$component_path"

# Verificar que handler tem conteúdo
grep -A 10 "onSubmit.*=" "$component_path" | grep -E "fetch|axios|mutate|dispatch"

# Verificar que não é apenas preventDefault
grep -A 5 "onSubmit" "$component_path" | grep -v "only.*preventDefault" -i
```

**Red flags:**
```typescript
// Handler apenas previne default:
onSubmit={(e) => e.preventDefault()}

// Handler apenas loga:
const handleSubmit = (data) => {
  console.log(data)
}

// Handler está vazio:
onSubmit={() => {}}
```

### Padrão: Estado → Renderização

**Verificar:** O componente renderiza estado, não conteúdo hardcoded?

```bash
# Encontrar uso de estado no JSX
grep -E "\{.*messages.*\}|\{.*data.*\}|\{.*items.*\}" "$component_path"

# Verificar map/render de estado
grep -E "\.map\(|\.filter\(|\.reduce\(" "$component_path"

# Verificar conteúdo dinâmico
grep -E "\{[a-zA-Z_]+\." "$component_path"  # Interpolação de variável
```

**Red flags:**
```tsx
// Hardcoded ao invés de estado:
return <div>
  <p>Mensagem 1</p>
  <p>Mensagem 2</p>
</div>

// Estado existe mas não renderizado:
const [messages, setMessages] = useState([])
return <div>Sem mensagens</div>  // Sempre mostra "sem mensagens"

// Estado errado renderizado:
const [messages, setMessages] = useState([])
return <div>{otherData.map(...)}</div>  // Usa dados diferentes
```

</wiring_verification>

<verification_checklist>

## Checklist Rápido de Verificação

Para cada tipo de artefato, percorra este checklist:

### Checklist de Componente
- [ ] Arquivo existe no caminho esperado
- [ ] Exporta um componente function/const
- [ ] Retorna JSX (não null/vazio)
- [ ] Sem texto placeholder na renderização
- [ ] Usa props ou state (não estático)
- [ ] Event handlers têm implementações reais
- [ ] Imports resolvem corretamente
- [ ] Usado em algum lugar no app

### Checklist de Rota de API
- [ ] Arquivo existe no caminho esperado
- [ ] Exporta handlers de método HTTP
- [ ] Handlers têm mais de 5 linhas
- [ ] Consulta banco de dados ou serviço
- [ ] Retorna resposta significativa (não vazia/placeholder)
- [ ] Tem tratamento de erros
- [ ] Valida input
- [ ] Chamado do frontend

### Checklist de Schema
- [ ] Modelo/tabela definido
- [ ] Tem todos os campos esperados
- [ ] Campos têm tipos apropriados
- [ ] Relacionamentos definidos se necessário
- [ ] Migrações existem e aplicadas
- [ ] Client gerado

### Checklist de Hook/Utilitário
- [ ] Arquivo existe no caminho esperado
- [ ] Exporta função
- [ ] Tem implementação significativa (não retornos vazios)
- [ ] Usado em algum lugar no app
- [ ] Valores de retorno consumidos

### Checklist de Conexão
- [ ] Componente → API: chamada fetch/axios existe e usa resposta
- [ ] API → Banco de Dados: query existe e resultado retornado
- [ ] Formulário → Handler: onSubmit chama API/mutation
- [ ] Estado → Renderização: variáveis de estado aparecem no JSX

</verification_checklist>

<automated_verification_script>

## Abordagem de Verificação Automatizada

Para o subagente de verificação, use este padrão:

```bash
# 1. Verificar existência
check_exists() {
  [ -f "$1" ] && echo "EXISTE: $1" || echo "FALTANDO: $1"
}

# 2. Verificar padrões de stub
check_stubs() {
  local file="$1"
  local stubs=$(grep -c -E "TODO|FIXME|placeholder|not implemented" "$file" 2>/dev/null || echo 0)
  [ "$stubs" -gt 0 ] && echo "PADRÕES_STUB: $stubs em $file"
}

# 3. Verificar conexão (componente chama API)
check_wiring() {
  local component="$1"
  local api_path="$2"
  grep -q "$api_path" "$component" && echo "CONECTADO: $component → $api_path" || echo "NÃO_CONECTADO: $component → $api_path"
}

# 4. Verificar substância (mais de N linhas, tem padrões esperados)
check_substantive() {
  local file="$1"
  local min_lines="$2"
  local pattern="$3"
  local lines=$(wc -l < "$file" 2>/dev/null || echo 0)
  local has_pattern=$(grep -c -E "$pattern" "$file" 2>/dev/null || echo 0)
  [ "$lines" -ge "$min_lines" ] && [ "$has_pattern" -gt 0 ] && echo "SUBSTANCIAL: $file" || echo "FRACO: $file ($lines linhas, $has_pattern matches)"
}
```

Execute estas verificações contra cada artefato obrigatório. Agregue resultados em VERIFICATION.md.

</automated_verification_script>

<human_verification_triggers>

## Quando Exigir Verificação Humana

Algumas coisas não podem ser verificadas programaticamente. Sinalize estas para teste humano:

**Sempre humano:**
- Aparência visual (parece correto?)
- Conclusão de fluxo do usuário (você consegue realmente fazer a coisa?)
- Comportamento em tempo real (WebSocket, SSE)
- Integração com serviço externo (Stripe, envio de email)
- Clareza de mensagem de erro (a mensagem é útil?)
- Sensação de performance (parece rápido?)

**Humano se incerto:**
- Conexão complexa que grep não consegue rastrear
- Comportamento dinâmico dependendo de estado
- Edge cases e estados de erro
- Responsividade mobile
- Acessibilidade

**Formato para pedido de verificação humana:**
```markdown
## Verificação Humana Necessária

### 1. Envio de mensagem no chat
**Teste:** Digite uma mensagem e clique Enviar
**Esperado:** Mensagem aparece na lista, input limpa
**Verificar:** Mensagem persiste após refresh?

### 2. Tratamento de erros
**Teste:** Desconecte a rede, tente enviar
**Esperado:** Mensagem de erro aparece, mensagem não perdida
**Verificar:** Pode tentar novamente após reconectar?
```

</human_verification_triggers>

<checkpoint_automation_reference>

## Automação Pré-Ponto de Verificação

Para padrões de pontos de verificação com automação primeiro, gerenciamento de ciclo de vida de servidor, tratamento de instalação de CLI e protocolos de recuperação de erros, veja:

**@D:/projetos/Estudo/devsquad/.cursor/get-shit-done/references/pontos-verificacao.md** → seção `<automation_reference>`

Princípios chave:
- Claude configura ambiente de verificação ANTES de apresentar pontos de verificação
- Usuários nunca executam comandos CLI (apenas visitam URLs)
- Ciclo de vida do servidor: iniciar antes do ponto de verificação, tratar conflitos de porta, manter rodando durante duração
- Instalação de CLI: auto-instalar onde seguro, ponto de verificação para escolha do usuário caso contrário
- Tratamento de erros: corrigir ambiente quebrado antes do ponto de verificação, nunca apresentar ponto de verificação com setup falhado

</checkpoint_automation_reference>
