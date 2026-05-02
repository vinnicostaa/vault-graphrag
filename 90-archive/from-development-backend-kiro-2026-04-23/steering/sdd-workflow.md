---
inclusion: always
---

# SDD Workflow — Como Executar Uma Task

**SDD (Spec-Driven Development)** é o processo oficial do Master Síndico. Inspirado em parte pelo [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) — que adotamos em **3 conceitos específicos** (não como framework completo):

1. **Ciclo explícito de 5 fases** por task (Discuss → Plan → Execute → Verify → Ship)
2. **`<verify>` declarativo no plano** — testes e critérios de aceite declarados **antes** da implementação (casa com TDD — ver `testing.md`)
3. **`.kiro/STATE.md` como arquivo vivo** de bloqueadores, decisões em aberto e dívidas técnicas

Spec canônica vive em `.kiro/specs/master-sindico/` (requirements + design + tasks + reference). Sem spec = sem implementação. Pular fases produz retrabalho.

Toda task segue **5 fases** antes de virar PR:

```
Discuss  →  Plan  →  Execute  →  Verify  →  Ship
 contexto   plano    código     testes +   PR +
 + spec     + <verify>+ tests   coverage   review
```

Esse ciclo é **obrigatório** — inclui o `feature-dev` plugin (exploration agent na fase Discuss, architecture design agent na fase Plan, quality review agent na fase Verify) e o `superpowers` plugin (brainstorm, write-plan, TDD red-green skills). Ver `steering/plugins.md` para integração plugin × fase.

## Fase 1 — Discuss

**Antes de tocar em qualquer arquivo**, leia nesta ordem:

1. `.kiro/specs/master-sindico/tasks/{sprint}.md` — descrição da task
2. `.kiro/specs/master-sindico/requirements/*.md` relacionado — o quê e por quê
3. `.kiro/specs/master-sindico/design/*.md` relacionado — como (interfaces, contratos, fluxos)
4. Código existente que vai ser modificado (`internal/modules/{context}/`)
5. `.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` — o que deu errado antes
6. `.kiro/STATE.md` — bloqueadores e decisões em aberto que impactam a task

**Não** avance se qualquer desses itens estiver incompleto ou ambíguo. Atualize o spec **antes** de implementar.

## Fase 2 — Plan

Escreva um **plano explícito** com os seguintes blocos (mental ou em comentário, conforme tamanho da task):

```xml
<plan task="1.3 Stripe adapter">
  <files>
    internal/modules/billing/domain/providers/i_payment_gateway.go       [novo]
    internal/modules/billing/infrastructure/providers/stripe_gateway.go  [novo]
    internal/shared/config/config.go                                      [editar — add Stripe config]
  </files>

  <docs-oficiais>
    - stripe-go v82: Context7 resolve-library-id("stripe/stripe-go")
    - Webhook signature: stripe.Webhook.ConstructEvent — validar ANTES de parsing
    - Idempotency keys: automático no SDK
  </docs-oficiais>

  <verify>
    - go build ./... compila sem erro
    - go vet ./... sem warning
    - Teste unitário: NewStripeGateway com config inválida retorna erro
    - Teste integração: CreateSubscription com customer_id real no sandbox retorna ID válido
    - Teste: webhook com assinatura inválida retorna 400
    - Coverage target: 85% em infrastructure/providers/
  </verify>

  <subagents>
    - Se a task explodir para 10+ arquivos, delegar por camada: domain, application, infrastructure
    - Subagent retorna APENAS resumo (o que mudou, o que ficou pendente) — não polui contexto
  </subagents>
</plan>
```

**Regra dura**: se a task usa SDK externo (Zitadel, Stripe, pgx, sqlc, goose, zerolog, etc), o plano deve declarar **qual fonte oficial** foi consultada via **Context7 MCP** (não memória de treino). Ver `.kiro/steering/mcp-integration.md`.

## Fase 3 — Execute

**Ordem de implementação por feature** (para o backend Go):

```
1. Migration SQL (goose) + CHECK constraints
   └── internal/shared/providers/database/migrations/NNNNN_*.sql
   └── Numeração por módulo: identity 001-099, billing 100-199,
       institutional 200-299, commercial 300-399, content 400-499, assembly 500-599

2. sqlc query tipada
   └── internal/modules/{ctx}/infrastructure/database/queries/*.sql
   └── sqlc generate (raiz do backend)

3. Domain entity / value object
   └── Estado privado, factory NewX() / ReconstructX(), métodos de domínio

4. Repository interface (domínio)
   └── internal/modules/{ctx}/domain/repositories/i_*_repository.go

5. Domain event (se cruzar bounded contexts)
   └── internal/modules/{ctx}/domain/events/*.go

6. Use case (CQRS: Command ≠ Query, arquivo separado)
   └── internal/modules/{ctx}/application/use_cases/*_use_case.go
   └── var _ contracts.IUseCase[I, O] = (*MyUseCase)(nil)

7. Mapper (entidade → DTO)

8. Repository implementation (infraestrutura, pgx+sqlc)
   └── Usar database.DBFromContext(ctx, pool) para suporte a transação

9. Provider externo (SDK isolado)
   └── Interface no domain/providers/, implementação em infrastructure/providers/

10. Handler + rota (contracts.HandlerFunc, nunca gin.HandlerFunc)

11. Wire-up no module.go e cmd/api/main.go

12. Testes (unidade + integração)
```

**Nunca** codifique fora dessa ordem. Pular a migration antes do Go é o erro mais caro.

### Orchestrator-Worker (para tasks grandes)

Tasks que envolvem **módulo inteiro** (ex: "implementar billing completo") devem ser **delegadas em subagentes**, cada um com contexto próprio:

```
Main agent (orquestrador)
 ├── Subagent 1: domain/ — entities, value objects, repo interfaces, events
 ├── Subagent 2: application/ — use cases CQRS, mappers
 ├── Subagent 3: infrastructure/database/ — repo implementations, sqlc queries
 └── Subagent 4: infrastructure/http/ — handlers, routes, wire-up
```

Cada subagente recebe: task do spec, arquivos já criados pelos anteriores, critério de aceitação específico da camada. Main agent **apenas coordena e revisa** — não polui o próprio contexto com o detalhe de cada camada.

## Fase 4 — Verify

**Rode localmente, nesta ordem**, antes de considerar a task pronta:

```bash
go build ./...                        # deve compilar sem erro
go vet ./...                          # deve passar sem warning
sqlc generate && git diff --exit-code # sqlc gerado deve estar em dia
go test ./... -count=1                # testes unitários
go test ./... -tags=integration       # testes de integração (testcontainers-go)
go test ./... -coverprofile=coverage.out
go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
```

**Thresholds mínimos** (ver `steering/testing.md`):
- `internal/modules/*/domain/` → **95%**
- `internal/modules/*/application/` → **90%**
- `internal/modules/*/infrastructure/database/` → **85%**
- `internal/modules/*/infrastructure/http/` → **75%**
- `internal/shared/middleware/` → **90%**
- `internal/core/` → **95%**
- **Global** → **≥ 85%**

Task que não atende o threshold **não é mergeada**. Se o threshold é injusto para o caso (ex: handler 74% por um único error branch impossível), documente no plano com `<verify-exception>` e justifique.

### Security gates (por task de segurança)

Quando a task envolve auth, billing, webhook, input externo, dados sensíveis (LGPD):
- `gosec -severity high ./...` deve passar
- `govulncheck ./...` deve passar
- Validação de input server-side: **sempre**. Frontend nunca é fonte de verdade.
- Webhook signature validada **antes** de qualquer parsing
- Secrets **nunca** em código, commit, log ou mensagem de erro

## Fase 5 — Ship

1. Commit com mensagem imperativa, prefixada pelo domínio quando escopo claro:
   ```
   billing: Add Stripe webhook signature validation
   ```
2. **Atualizar README.md do sub-projeto** quando a mudança é feature-level (não micro-fix / não dívida paga em isolamento):
   - `backend/README.md` quando a mudança expõe endpoint novo, tabela nova, domínio novo, integração de SDK nova, invariante novo na matriz ABAC, ou contrato cross-module novo em `shared/types/`.
   - `web/README.md` quando novo app SolidJS, nova rota TanStack, nova feature flag, nova variável de env.
   - `app/README.md` quando novo feature slice Flutter, novo provider, nova tela principal, nova integração nativa.
   - Seções a manter em dia: stack usada, endpoints/rotas principais, variáveis de ambiente, comandos de dev, e status de sprint (marcos + % coberto).
   - Regra de escopo: bug-fix pontual **não** atualiza README. Nova feature **sempre** atualiza.
3. Abrir PR via GitHub MCP (ver `steering/mcp-integration.md`)
4. PR body obrigatoriamente tem seção `Release Notes:`
5. CI valida: build + vet + test + coverage thresholds + gosec + govulncheck
6. Review: pelo menos uma pessoa revisa ou, em trabalho solo, aguardar 30min antes de merge para reler com olhos frescos

## Fase 6 — Sprint Audit (gate antes do próximo sprint)

Ao fim de cada sprint, antes de qualquer task do sprint seguinte, rodar a skill **`sprint-auditor`** (ver `.claude/skills/sprint-auditor.md`). Ela lê `AUDIT.md`, revisa sistematicamente o que foi entregue (seguindo o check-list do `.claude/skills/go-review.md`) e materializa findings em `.kiro/AUDIT.md`.

```
Sprint {N} tasks ✅ → sprint-auditor → AUDIT.md com findings
                                        ↓
  task-executor próxima sessão lê AUDIT.md PRIMEIRO
                                        ↓
              itens 🔴/🟡 abertos? — resolver antes da próxima task
                                        ↓
      sprint-auditor re-valida (modo fechamento) → move ✅ → Histórico
                                        ↓
                      próximo sprint liberado para Discuss
```

**Regras duras**:

- `task-executor` é **obrigado** a abrir `.kiro/AUDIT.md` no início de cada sessão. Itens `🔴 Aberto` / `🟡 Em andamento` **bloqueiam** pegada de task nova.
- `sprint-auditor` **nunca** implementa fix — apenas aponta. Fix é responsabilidade do `task-executor` na sessão seguinte, tratando cada item como micro-task (mesmo ciclo 5-fase, escala reduzida).
- Distinção vs `STATE.md`: STATE lista bloqueadores/decisões/dívidas **conscientes** (com sprint alvo); AUDIT lista bugs/má implementação **descobertos em revisão** (sem sprint alvo — prioridade máxima).
- Severidades em AUDIT: `critical` (bloqueia deploy) → `high` (bug funcional) → `medium` (cheiro forte) → `low` (cosmético). `critical` e `high` são blocker obrigatório do próximo sprint.

Invocação: `/sprint-auditor` (manual ao fim de sprint) ou implicitamente na primeira sessão pós-sprint se `AUDIT.md` tiver items abertos.

## `<verify>` é contrato, não sugestão

O bloco `<verify>` do plano é um **compromisso**. Se você escreveu "teste unitário X passa", esse teste **existe** ao final da task. Não existe "depois eu faço" — se ficar fora do escopo, documente explicitamente em `.kiro/STATE.md` como dívida.

## `STATE.md` — arquivo vivo

`.kiro/STATE.md` rastreia o que o `tasks.md` (estático) não comporta:
- Bloqueadores no momento (com responsável, data, impacto)
- Decisões em aberto (com alternativas e prazo para resolver)
- Dívidas técnicas registradas (com sprint alvo para pagar)
- Questões pendentes de produto que afetam spec

Atualize `STATE.md` **sempre** que descobrir um bloqueador, abrir uma decisão, ou registrar débito técnico. Sem isso, a próxima sessão do agente vai trombar no mesmo muro.

## Comandos Úteis (quick reference)

```bash
# Rodar a API
go run ./cmd/api

# Subir infraestrutura local (PostgreSQL 18 + Redis 7 + NATS)
docker compose up -d

# Health checks
curl http://localhost:8000/health
curl http://localhost:8000/health/ready

# Migrations (manual em dev; auto no startup via database.Migrate)
go run ./cmd/migrate up
go run ./cmd/migrate status

# sqlc
sqlc generate
git diff --name-only                     # não deve ter diff em generated/

# Mocks (mockery — adicionar via go install quando necessário)
mockery --name=ISubscriptionRepository \
        --dir=internal/modules/billing/domain/repositories \
        --output=internal/modules/billing/application/mocks

# Coverage por arquivo
go test ./... -count=1 -coverprofile=coverage.out
go tool cover -func=coverage.out | grep -v "100.0%"

# Stripe webhook local
stripe listen --forward-to localhost:8000/webhooks/stripe
```

## Adicionando um Bounded Context

```bash
ctx=<billing|institutional|commercial|content|assembly>

mkdir -p internal/modules/$ctx/{domain/{entities,value_objects,repositories,events,providers},application/{use_cases,mappers},infrastructure/{database/{queries,generated},http/routes,providers,jobs}}
```

Depois: criar `internal/modules/$ctx/module.go` implementando `contracts.IModule` e registrar em `cmd/api/main.go`.

## Regras de PR

- Título imperativo, capitalizado, sem prefixos `fix:`/`feat:`, sem pontuação final
- Prefixo de domínio quando escopo claro: `billing: Add Stripe webhook handler`
- Corpo descreve **o quê e por quê**, não como
- Sempre seção `Release Notes:` no final do body
- Checkout de `Release Notes:` vai para CHANGELOG quando publicado

## Não Fazer

- Não commitar `.env` ou qualquer arquivo com secrets
- Não editar arquivos em `*/infrastructure/database/generated/` manualmente
- Não criar abstrações sem 2+ implementações reais (YAGNI)
- Não adicionar features não solicitadas ("enquanto estou aqui...")
- Não misturar lógica de negócio em handlers ou middleware
- Não usar `interface{}` / `any` onde tipo concreto serve
- Não pular a fase Discuss porque "já sei o que fazer"
