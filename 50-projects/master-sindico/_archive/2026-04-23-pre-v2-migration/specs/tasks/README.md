---
title: Tasks — Índice + Formato 5-fase
version: 1.0
status: stable
---

# Tasks — Master Síndico

Backlog operacional do projeto, organizado por Sprint. Cada arquivo `sprint-N.md` contém tasks **auto-contidas** em formato de 5 fases (Discuss → Plan → Execute → Verify → Ship).

## Navegação

| Arquivo | Período | Foco |
|---|---|---|
| `sprint-0.md` | 23/03 → 06/04/2026 | Fundação (concluído, ver status final) |
| `sprint-1.md` | 07/04 → 20/04/2026 | Identity + Billing (em andamento) |
| `sprint-2.md` | 21/04 → 04/05/2026 | Institutional I — Registry + Governance base |
| `sprint-3.md` | 05/05 → 18/05/2026 | Institutional II — Governance completa |
| `sprint-4.md` | 19/05 → 01/06/2026 | Commercial — Connect Me + Marketplace |
| `sprint-5.md` | 02/06 → 15/06/2026 | Content — Vídeos + Search |
| `sprint-6.md` | 16/06 → 29/06/2026 | Integrações |
| `sprint-7.md` | 30/06 → 13/07/2026 | Assembly + Polish + Observabilidade |
| `sprint-8.md` | 14/07 → 08/08/2026 | QA + Deploy prod |
| `post-launch.md` | 09/08/2026+ | Academia LMS, Vizinhança, Banco Talentos, Admin |

**Marco 1**: 2026-05-08 (MVP contratual) · **Marco 2**: 2026-06-20 · **Marco 3**: 2026-07-20

## Formato 5-fase — o contrato de cada task

Cada task individual tem este formato:

````markdown
## Task X.Y — <título curto imperativo>

**Estado**: `pending | in_progress | blocked | completed | deferred`
**Sprint**: <número>
**Depende de**: <lista de task IDs> (ou "nenhuma")
**Estima**: <horas> (opcional)

### Discuss — contexto e motivação

- **Requisitos envolvidos**: Req X, Req Y (links para `requirements/*.md`)
- **Design envolvido**: `design/module-X.md` seção Y
- **Decisão em aberto?**: link para `.kiro/STATE.md` se houver
- **Lições do legacy**: antipatterns A1, A3 aplicáveis (se sim) — ver `reference/antipatterns-to-avoid.md`

### Plan — arquivos a criar/modificar

**Criar**:
- `internal/modules/{ctx}/domain/entities/Foo.go` — estado privado, factory, métodos
- `internal/modules/{ctx}/application/use_cases/create_foo_use_case.go` — CQRS command
- `internal/modules/{ctx}/infrastructure/database/foo_repository.go` — impl via pgx+sqlc

**Modificar**:
- `cmd/api/main.go` — wire-up do módulo (se novo)
- `internal/shared/providers/database/migrations/NNNNN_create_foo.sql` — schema

**Não tocar**:
- `modules/{outro_ctx}/` — fora do escopo desta task

**Docs oficiais a consultar via Context7**:
- `stripe/stripe-go` (trust 8.9) — topic "CreateSubscription com trial"
- `jackc/pgx` — topic "BeginTxFunc pattern"

**Subagentes?**: Se task cruzar 4+ camadas, dividir (domain, application, infra) conforme `steering/sdd-workflow.md` seção Orchestrator-Worker.

### `<verify>` — critérios de aceitação testáveis

Escrever **antes** de codar (TDD):

- [ ] Teste unitário: `TestCreateFooUseCase_Execute_CreatesFoo` passa
- [ ] Teste unitário: `TestCreateFooUseCase_Execute_FailsWhenAlreadyExists` passa
- [ ] Teste de integração: `TestFooRepository_SaveAndFind` com testcontainers-go passa
- [ ] Coverage em `application/` ≥ 90%, em `domain/` ≥ 95%
- [ ] `go build ./...` limpo
- [ ] `go vet ./...` sem warning
- [ ] `gosec -severity high` sem findings em arquivos tocados
- [ ] Migration aplica + reverte idempotentemente (`goose up && goose down && goose up`)

### Execute — ordem de implementação

1. Migration SQL + CHECK constraints → `go build ./...` valida embed.FS
2. Query sqlc `.sql` → `sqlc generate`
3. Teste unitário (RED)
4. Domain entity + value objects
5. Repository interface
6. Use case
7. Repository impl (GREEN para teste de integração)
8. Handler + route
9. Wire-up
10. Refactor (com suite verde)

### Ship — checklist final

- [ ] Todos os gates verdes (build, vet, test, coverage, gosec, govulncheck)
- [ ] PR aberto com título imperativo + seção `Release Notes:`
- [ ] `.kiro/STATE.md` atualizado se descobriu dívida técnica ou bloqueador novo
- [ ] Task marcada `completed` neste arquivo

### Notas pós-implementação (opcional)

Decisões feitas durante, issues encontrados, coisas a refatorar depois.

---
````

## Convenções

- **Task ID**: `sprint.task` (ex: `1.3` = Sprint 1, task 3)
- **Sub-task**: `sprint.task.sub` (ex: `1.0.1` = Sprint 1, pré-req 0, sub 1)
- **Estado**:
  - `pending` — ainda não iniciado
  - `in_progress` — em execução agora
  - `blocked` — depende de algo externo; detalhar em `.kiro/STATE.md`
  - `completed` — terminado + mergeado
  - `deferred` — fora do escopo do sprint atual, re-agendado
- **Checkbox**: `[ ]` pending · `[x]` completed · `[-]` blocked/deferred

## Workflow do agente

Quando pedido para implementar task:

1. Ler `sprint-N.md`, localizar task
2. Ler `requirements/` e `design/` referenciados
3. Ler `STATE.md` para bloqueadores
4. Ler `reference/antipatterns-to-avoid.md` se Task tocar áreas sensíveis (auth, quotas, timeline, votação)
5. Consultar Context7 para docs oficiais dos SDKs
6. **Escrever testes primeiro** (RED — TDD)
7. Implementar na ordem declarada em Plan
8. Rodar gates até todos verdes
9. Abrir PR via `github` MCP/plugin
10. Atualizar estado da task para `completed`

## Tasks que não seguem formato 5-fase

Raras exceções — tarefas puramente operacionais:

- Configurar infra (criar bucket S3) — seguem runbook, não 5-fase
- Upgrade de dependência menor — se for bump de patch sem breaking change
- Edit de texto em doc — se for só correção de typo

**Todo o resto** segue formato acima. Task que não cabe em 5-fase deve ser **dividida** em sub-tasks que cabem.
