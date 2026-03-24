<purpose>

Arquivar diretórios de fases acumulados de marcos concluídos em `.planning/milestones/v{X.Y}-phases/`. Identifica quais fases pertencem a cada marco concluído, mostra resumo dry-run e move diretórios após confirmação.

</purpose>

<required_reading>

1. `.planning/MILESTONES.md`
2. Listagem do diretório `.planning/milestones/`
3. Listagem do diretório `.planning/phases/`

</required_reading>

<process>

<step name="identify_completed_milestones">

Ler `.planning/MILESTONES.md` para identificar marcos concluídos e suas versões.

```bash
cat .planning/MILESTONES.md
```

Extrair cada versão de marco (ex: v1.0, v1.1, v2.0).

Verificar quais diretórios de arquivo de marco já existem:

```bash
ls -d .planning/milestones/v*-phases 2>/dev/null
```

Filtrar para marcos que NÃO possuem diretório de arquivo `-phases`.

Se todos os marcos já tiverem arquivos de fases:

```
Todos os marcos concluídos já possuem diretórios de fases arquivados. Nada para limpar.
```

Parar aqui.

</step>

<step name="determine_phase_membership">

Para cada marco concluído sem arquivo `-phases`, ler o snapshot de ROADMAP arquivado para determinar quais fases pertencem a ele:

```bash
cat .planning/milestones/v{X.Y}-ROADMAP.md
```

Extrair números e nomes de fases do roteiro arquivado (ex: Fase 1: Fundação, Fase 2: Auth).

Verificar quais desses diretórios de fase ainda existem em `.planning/phases/`:

```bash
ls -d .planning/phases/*/ 2>/dev/null
```

Combinar diretórios de fase com pertencimento ao marco. Incluir apenas diretórios que ainda existem em `.planning/phases/`.

</step>

<step name="show_dry_run">

Apresentar resumo dry-run para cada marco:

```
## Resumo da Limpeza

### v{X.Y} — {Nome do Marco}
Estes diretórios de fase serão arquivados:
- 01-foundation/
- 02-auth/
- 03-core-features/

Destino: .planning/milestones/v{X.Y}-phases/

### v{X.Z} — {Nome do Marco}
Estes diretórios de fase serão arquivados:
- 04-security/
- 05-hardening/

Destino: .planning/milestones/v{X.Z}-phases/
```

Se nenhum diretório de fase restar para arquivar (todos já movidos ou deletados):

```
Nenhum diretório de fase encontrado para arquivar. Fases podem ter sido removidas ou arquivadas anteriormente.
```

Parar aqui.

conversational prompting: "Prosseguir com o arquivamento?" com opções: "Sim — arquivar fases listadas" | "Cancelar"

Se "Cancelar": Parar.

</step>

<step name="archive_phases">

Para cada marco, mover diretórios de fases:

```bash
mkdir -p .planning/milestones/v{X.Y}-phases
```

Para cada diretório de fase pertencente a este marco:

```bash
mv .planning/phases/{dir} .planning/milestones/v{X.Y}-phases/
```

Repetir para todos os marcos no conjunto de limpeza.

</step>

<step name="commit">

Commitar as mudanças:

```bash
node "D:/projetos/Estudo/devsquad/.cursor/get-shit-done/bin/gsd-tools.cjs" commit "chore: arquivar diretórios de fases de marcos concluídos" --files .planning/milestones/ .planning/phases/
```

</step>

<step name="report">

```
Arquivado:
{Para cada marco}
- v{X.Y}: {N} diretórios de fases → .planning/milestones/v{X.Y}-phases/

.planning/phases/ limpo.
```

</step>

</process>

<success_criteria>

- [ ] Todos os marcos concluídos sem arquivos de fase existentes identificados
- [ ] Pertencimento de fase determinado a partir de snapshots de ROADMAP arquivados
- [ ] Resumo dry-run mostrado e usuário confirmou
- [ ] Diretórios de fases movidos para `.planning/milestones/v{X.Y}-phases/`
- [ ] Mudanças commitadas

</success_criteria>
