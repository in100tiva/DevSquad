# Instalação do DevSquad

Depois de clonar este repositório, use o DevSquad em **qualquer projeto** onde você usa o Cursor.

## 1. Clone o repositório (se ainda não tiver)

```bash
git clone https://github.com/in100tiva/DevSquad.git
cd DevSquad
```

## 2. Copie o DevSquad para o seu projeto

No **projeto onde você quer usar** o DevSquad (a pasta do seu código):

- Copie a pasta **`_devsquad`** para a **raiz** do projeto.
- Crie a pasta **`.claude/skills/`** na raiz do projeto (se não existir).
- Copie a pasta **`devsquad`** que está em `.claude/skills/devsquad` para dentro do seu `.claude/skills/`.

**Exemplo (PowerShell, a partir da pasta do DevSquad clonada):**

```powershell
$meuProjeto = "C:\caminho\do\meu\projeto"
Copy-Item -Path _devsquad -Destination $meuProjeto\_devsquad -Recurse -Force
New-Item -ItemType Directory -Force -Path "$meuProjeto\.claude\skills" | Out-Null
Copy-Item -Path .claude\skills\devsquad -Destination "$meuProjeto\.claude\skills\devsquad" -Recurse -Force
```

## 3. Configure os MCPs no Cursor

Antes de usar, configure os MCPs obrigatórios no Cursor:

1. Leia **`_devsquad/PREREQUISITES.md`** no seu projeto (instruções completas).
2. Configure no Cursor:
   - **Context7** (obrigatório) — documentação atualizada de libs.
   - **Playwright** (obrigatório) — validação de interface e testes.

## 4. Use no Cursor

Abra o projeto no Cursor e digite:

- **`/devsquad`** — menu principal
- **`/devsquad analyze`** — analisar código
- **`/devsquad create`** — criar do zero
- **`/devsquad refactor`** — refatorar
- **`/devsquad team`** — listar equipe
- **`/devsquad help`** — ajuda

Pronto. O Lucas Tech Lead e a equipe estão disponíveis no seu projeto.
