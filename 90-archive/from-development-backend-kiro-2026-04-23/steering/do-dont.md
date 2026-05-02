---
inclusion: always
---

# Do / Don't — Master Síndico Backend

Consolidação das regras duras. Este arquivo é o **checklist**; os outros arquivos de steering têm o **detalhe**. Se houver conflito, este vence.

## FAZER ✅

### Antes de qualquer implementação

- Ler `.kiro/specs/master-sindico/tasks/{sprint}.md`, `requirements/*.md` e `design/*.md` relacionados
- Ler o código existente do módulo (`internal/modules/{ctx}/`)
- Ler `.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` — saber o que deu errado antes
- Checar `.kiro/STATE.md` — bloqueadores e decisões em aberto
- Consultar doc oficial via **Context7 MCP** (ou plugin Context7) antes de usar qualquer SDK externo
- Declarar plano com `<verify>` antes de codar — ver `steering/sdd-workflow.md`
- **TDD obrigatório**: escrever teste que falha (Red) **antes** da implementação. Sem exceção além de spikes descartáveis — ver `steering/testing.md`

### Código Go

- Early returns e guard clauses (nunca if/else aninhado profundo)
- Propagar erros com `fmt.Errorf("contexto: %w", err)`
- `errors.Is` / `errors.As` para inspecionar
- Compile-time interface check: `var _ IFoo = (*Impl)(nil)`
- `context.Context` como primeiro parâmetro em funções com I/O
- Nomes completos: `unitOfWork`, `condominium`, `subscriptionRepository`
- Entidades com estado privado + getters explícitos + factories `NewX()` / `ReconstructX()`
- Value Objects imutáveis, validados na factory, sem setters
- Um use case por arquivo, um handler por arquivo, um provider por arquivo

### Arquitetura

- Ordem de implementação: migration → sqlc → domain → application → infrastructure → routes → wire-up
- Interfaces no `domain/`, implementações no `infrastructure/`
- `IUnitOfWork.Run()` para operações que precisam de atomicidade
- Comunicar entre bounded contexts via **domain events**, nunca chamadas diretas
- Toda query de dados de condomínio: `WHERE condominium_id = $tenantID`
- Handlers usam `contracts.Context` / `contracts.HandlerFunc`, nunca `gin.Context`

### Banco de dados

- Migration antes do código Go
- goose com `-- +goose Up` / `-- +goose Down` + StatementBegin/End
- CHECK constraints para enums e invariantes simples (sempre)
- Listar colunas explicitamente em queries sqlc — nunca `SELECT *`
- `sqlc generate` após modificar query
- `go build ./...` após criar migration (valida `embed.FS`)
- Índice em toda FK e em colunas de filtro frequente
- Validar `EXPLAIN ANALYZE` em query nova via **Postgres MCP** antes de commit

### Segurança

- Verificar assinatura de webhook **antes** de parsing (Stripe, Mux)
- Idempotency keys em escritas de provider externo
- Mascarar CPF/CNPJ em logs (`***.***.123-45`)
- Tokens em logs: apenas prefixo (`sk_test_...`)
- Nunca retornar PII em mensagem de erro
- Signed URLs com TTL curto para arquivos privados
- JWT Profile (service account) para introspection Zitadel — não Client Credentials
- Validação server-side sempre (**never trust the frontend**)
- CHECK constraint no banco para invariantes críticas (não só validação na aplicação)
- ABAC engine falha no startup se matriz ausente/inválida

### Testes (TDD)

- **Red → Green → Refactor**: teste primeiro, implementação depois
- Testes de integração com banco real (testcontainers-go + reuse de container + TRUNCATE CASCADE entre testes)
- PBT (`pgregory.net/rapid`) em regras críticas: fração ideal, quotas, ABAC, trial, tenant isolation
- Tag `//go:build integration` nos testes de integração
- `-race` no CI sempre
- Coverage thresholds por camada (ver `steering/testing.md`) — gate duro
- Um spike exploratório pode preceder TDD se SDK externo for instável — spike é **sempre** descartado; código final nasce com testes

### Transações

- Operação em 1 banco → `IUnitOfWork.Run` (ACID local via pgx tx)
- Operação cruzando banco + provider externo (Stripe, Mux, SES, Twilio) → **Saga orquestrada** com compensação explícita (ver `steering/transaction-patterns.md`)
- Operação long-running (> 5s) → Saga com state persistido em `saga_state`
- Eventual consistency (notificação, cache invalidação) → domain event + handler async idempotent
- Concorrência em quota/recurso → `SELECT ... FOR UPDATE` dentro de tx, **não** subir isolation level

### Code Calisthenics (ver `steering/code-calisthenics.md`)

- Máximo 1 nível de indentação por método — early returns
- Sem `else` — guard clauses; switch é ok
- Envolver primitivos com value objects (Email, CPF, CNPJ, Money) — auto-validados
- Wrap em tipo próprio para collections com invariantes (`VoteCollection`, não `[]Vote` solto)
- Um ponto por linha (Law of Demeter) — evitar `a.b().c().d()` em cascata de entidades
- Nomes completos: sem abreviações fora de convenção idiomática (`ctx`, `err`, `i`, receivers)
- Entidade com 5-10 campos. Acima: cheiro de God Object → dividir agregados
- Use case com 3-5 dependências. Acima de 7: cheiro → Saga ou composição
- Sem `SetX` em entidades — usar métodos de domínio que expressam intenção (`Cancel(reason)`, `Activate()`)

### Validação antes de concluir task

```bash
go build ./...                        # compila sem erro
go vet ./...                          # sem warning
sqlc generate && git diff --exit-code # generated em dia
go test -race -count=1 ./...          # testes unit passam
go test -tags=integration -count=1 ./... # integração passa
go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
gosec -severity high ./...            # sem findings
govulncheck ./...                     # sem vulns
```

---

## NÃO FAZER ❌

### Código Go

- ❌ `_ = fn()` em operação falível — **PROIBIDO** absoluto
- ❌ `err.Error() == "not found"` — use `errors.Is`/`errors.As`
- ❌ `float64` / `float32` para dinheiro — `int64` centavos
- ❌ `float` para fração ideal — `DECIMAL/NUMERIC(10,6)` no PG
- ❌ `panic` em produção — apenas em `init()` com config irrecuperável
- ❌ `interface{}` / `any` onde tipo concreto serve
- ❌ Abreviações em variável: `uow`, `condo`, `q`, `repo`
- ❌ `zerolog.SetGlobalLevel()` — mutação global quebra testes paralelos
- ❌ `sync.Mutex` em struct exportada sem documentar invariante (esconde complexidade)

### Arquitetura

- ❌ Importar Gin em `core/` ou `domain/` — viola Clean Architecture
- ❌ Importar SDK externo (Stripe, Mux, Twilio, Zitadel) em `domain/`
- ❌ Chamar use case de outro bounded context diretamente — use domain events
- ❌ Misturar commands e queries no mesmo use case (CQRS)
- ❌ Lógica de negócio em handlers — handlers só parsear/chamar/serializar
- ❌ Lógica de negócio em middleware — middleware é transversal, não regra
- ❌ Abstrações sem 2+ implementações reais (YAGNI)
- ❌ Criar muitos arquivos pequenos — prefira implementar em arquivo existente
- ❌ Features não solicitadas ("enquanto estou aqui...")
- ❌ Mudar arquitetura sem discussão explícita com o time

### Banco de dados

- ❌ `SELECT *` em query sqlc — liste colunas
- ❌ Editar arquivos em `*/infrastructure/database/generated/` manualmente
- ❌ Migration destrutiva em prod: `DROP TABLE`, `DROP COLUMN`, `ALTER COLUMN` que perde dados
- ❌ `FLOAT` / `DOUBLE` para dinheiro ou fração ideal
- ❌ UPDATE/DELETE em tabela imutável (`timeline_entries`, `audit_trail`, `assembly_votes`)
- ❌ Query em tabela com soft delete sem `WHERE deleted_at IS NULL`
- ❌ Query de dados de condomínio sem `WHERE condominium_id = $tenantID`
- ❌ Begin/Commit manual fora de `UnitOfWork.Run()`
- ❌ Rodar migration em múltiplas instâncias simultâneas (ECS tasks paralelas)

### Segurança

- ❌ Commitar `.env` ou qualquer arquivo com secret
- ❌ Tokens em `localStorage` / `sessionStorage` (frontend)
- ❌ `sk_live_*` em ambiente de dev — nunca
- ❌ `InsecureSkipVerify: true` em `tls.Config`
- ❌ Processar webhook sem validar assinatura
- ❌ Logar CPF/CNPJ, senhas, tokens completos, emails sem mascarar em debug
- ❌ Retornar PII em mensagem de erro ao cliente
- ❌ CORS com `*` em rota protegida
- ❌ Validar input apenas no frontend — sempre server-side
- ❌ Combinar MCP de busca web + MCP destrutivo sem aprovação humana (prompt injection)

### Processo

- ❌ Implementar sem ler código existente primeiro
- ❌ Avançar para próxima task sem a atual compilar
- ❌ Ignorar erro de compilação — corrija antes de avançar
- ❌ Aceitar info de terceiro sem verificar doc oficial via Context7
- ❌ Pular fase `Discuss` porque "já sei o que fazer"
- ❌ Pular TDD — "testes depois" é anti-padrão provado pelo legacy
- ❌ Skip de `go build && go vet && go test` antes de commit
- ❌ `git push --force` em branch compartilhada
- ❌ Commit com secret acidentalmente — se acontecer, rotacionar imediatamente
- ❌ Usar `full-stack/` como fonte de verdade — é legacy, contém decisões já revisadas
- ❌ Copiar código/convenções do `full-stack/` sem validar contra backend atual + requirements atualizado

---

## Antipatterns Vindos do Legacy (não repetir)

Ver `.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` para detalhe. Resumo das 12 armadilhas:

| # | O que deu errado no full-stack TypeScript | Regra no Go |
|---|---|---|
| A1 | `PermissionService` comentado → autorização desligada sem aviso | ABAC falha no startup se não configurado; teste de integração valida |
| A2 | Profile importa `UserRole` de User (vazamento de domínio) | Cada BC define tipos próprios; cruzar via `shared/types/` ou eventos |
| A3 | Switch de 5 cases em `UpdateProfileUseCase` (if/else hell) | Strategy pattern: `ProfileUpdater` interface + implementação por tipo |
| A4 | `this.authorize()` duplicado em 4 métodos de update | Middleware `RequirePermission()` na rota, nunca no use case |
| A5 | Ordem inconsistente do fluxo de autorização | Padrão rígido: `Fetch → ValidateExists → Authorize → Mutate → Save` |
| A6 | `PermissionService` sem log de negação | ABAC retorna `{granted, reason}`; log estruturado em cada denial |
| A7 | Falta de domain events (update não invalida cache) | `IDomainEventPublisher` em `core/contracts/` desde Sprint 2 |
| A8 | Race condition em `incrementUsage()` sem lock | `SELECT ... FOR UPDATE` em tx, ou advisory lock |
| A9 | Validação só na aplicação (DB aceitava lixo) | CHECK constraints + validator Go no handler |
| A10 | Logs inconsistentes (user_id às vezes) | Campos mínimos obrigatórios no middleware Logger: `request_id, user_id, tenant_id` |
| A11 | Drizzle v1-beta instável | sqlc v1.30 (estável) + pgx/v5 — já feito no Go ✅ |
| A12 | Mistura User/Profile/Subscription | Separação rígida por bounded context: `identity.User` ≠ `institutional.Profile` ≠ `billing.Subscription` |

---

## Fontes Oficiais Obrigatórias (consultar via Context7 MCP)

| SDK / Tech | Context7 ID | Documentação |
|---|---|---|
| Zitadel | `zitadel/zitadel` (trust 9.5) | https://zitadel.com/docs/examples/secure-api/go |
| pgx v5 | `jackc/pgx` | https://pkg.go.dev/github.com/jackc/pgx/v5 |
| sqlc | `sqlc-dev/sqlc` | https://docs.sqlc.dev |
| stripe-go | `stripe/stripe-go` (trust 8.9) | https://docs.stripe.com/api |
| goose | `pressly/goose` | https://github.com/pressly/goose |
| zerolog | `rs/zerolog` | https://github.com/rs/zerolog |
| Gin | `gin-gonic/gin` | https://gin-gonic.com/docs/ |
| Mux (vídeo) | `muxinc/mux-go` | https://docs.mux.com |
| Twilio | `twilio/twilio-go` | https://www.twilio.com/docs |

**Regra dura**: nunca codar integração sem puxar docs via Context7. Treino do modelo pode estar desatualizado; Context7 entrega a versão ativa do `go.mod`.

Ver `steering/mcp-integration.md` para fluxo completo.
