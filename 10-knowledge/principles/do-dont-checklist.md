---
title: Do / Don't — checklist consolidado
type: guide
tags: [principles, checklist, do-dont]
source: .kiro/steering/do-dont.md
created: 2026-04-22
aliases: [Do Don't, Regras duras, Checklist consolidado]
---

# Do / Don't — checklist consolidado

Consolidação cross-linguagem das regras duras. Este arquivo é o **checklist**; detalhes específicos vivem nos arquivos temáticos (`architecture/`, `testing/`, `security/`, etc.). **Se houver conflito, este arquivo vence** pro tópico coberto.

Versão Go-específica deste mesmo checklist vive em `20-stacks/go/` *(a ingerir)*. Este é o cross-linguagem.

## ✅ FAZER

### Antes de qualquer implementação

- [ ] Ler `specs/tasks/{sprint}.md`, `requirements/*.md` e `design/*.md` relacionados
- [ ] Ler código existente do módulo (`src/modules/{ctx}/` ou equivalente)
- [ ] Ler `specs/reference/antipatterns-to-avoid.md` (ou equivalente) — saber o que deu errado antes
- [ ] Checar STATE.md — bloqueadores e decisões em aberto
- [ ] Consultar doc oficial via **Context7 MCP** antes de usar qualquer SDK externo
- [ ] Declarar plano com `<verify>` antes de codar — ver [[../methodology/sdd-workflow]] *(a criar)*
- [ ] **TDD obrigatório**: teste que falha (Red) **antes** da implementação. Exceção: spikes descartáveis

### Código

- [ ] Early returns e guard clauses — nunca if/else aninhado profundo
- [ ] Propagar erros com contexto (`fmt.Errorf("X: %w", err)` em Go; `throw new X({ cause: err })` em TS; etc.)
- [ ] Usar mecanismo tipado pra inspecionar erro (`errors.Is/As`, `instanceof`, pattern-matching)
- [ ] Compile-time interface check quando a linguagem permite
- [ ] Context/cancellationToken como primeiro param em I/O
- [ ] **Nomes completos** — `unitOfWork`, `condominium`, `subscriptionRepository`
- [ ] Entidades com estado privado + getters explícitos + factories `NewX()` / `ReconstructX()`
- [ ] Value Objects imutáveis, validados na factory, sem setters
- [ ] **Um use case por arquivo**, um handler por arquivo, um provider por arquivo

### Arquitetura

- [ ] Ordem de implementação: **migration → codegen → domain → application → infrastructure → routes → wire-up**
- [ ] Interfaces no `domain/`, implementações no `infrastructure/`
- [ ] `IUnitOfWork.Run()` (ou equivalente) em operações que precisam atomicidade
- [ ] Bounded contexts se comunicam **via domain events**, nunca chamadas diretas
- [ ] **Tenant isolation**: `WHERE tenant_id = $ctx.tenantID` em toda query multi-tenant
- [ ] Handlers usam contratos abstratos (`HTTPRouter`), **nunca** framework cru (Gin, Express, Fastify)

### Banco de dados

- [ ] Migration antes do código
- [ ] Ferramenta versionada com up/down (goose, Flyway, Prisma migrate, etc.)
- [ ] CHECK constraints pra enums e invariantes simples (**sempre**)
- [ ] Listar colunas explicitamente — **nunca** `SELECT *`
- [ ] Codegen atualizado (sqlc, Prisma, drizzle-kit) e **commitado**
- [ ] Build limpo após criar migration (valida embed)
- [ ] Índice em toda FK e em colunas de filtro frequente
- [ ] Validar `EXPLAIN ANALYZE` via MCP antes de commit em query nova

### Segurança

- [ ] Verificar assinatura HMAC do webhook **antes** de parsing (Stripe, Mux, GitHub, etc.)
- [ ] **Idempotency keys** em escritas de provider externo
- [ ] Mascarar PII em logs (`***.***.123-45` para CPF)
- [ ] Tokens em log: apenas prefixo (`sk_test_...`)
- [ ] **Nunca** retornar PII em mensagem de erro ao cliente
- [ ] Signed URLs com TTL curto pra arquivos privados
- [ ] Service Account Key (JWT Profile) pra introspection OIDC — **não** Client Credentials
- [ ] Validação server-side **sempre** ("never trust the frontend")
- [ ] CHECK constraint no banco pra invariantes críticas (**não só** validação na aplicação)
- [ ] ABAC engine **falha no startup** se matriz ausente/inválida

### Testes (TDD)

- [ ] **Red → Green → Refactor**: teste primeiro, implementação depois
- [ ] Testes de integração com banco real (testcontainers + reuse de container + TRUNCATE CASCADE entre testes)
- [ ] **Property-based testing** em regras críticas: fração ideal, quotas, ABAC, trial, tenant isolation
- [ ] Build tag / suite separada pros testes de integração
- [ ] `-race` (ou equivalente) no CI sempre
- [ ] Coverage thresholds por camada — gate duro
- [ ] Spike exploratório pode preceder TDD se SDK é instável — spike é **sempre** descartado; código final nasce com testes

### Transações

- [ ] Operação em 1 banco → UnitOfWork (ACID local)
- [ ] Operação cruzando banco + provider externo (Stripe, Mux, SES) → **Saga orquestrada** com compensação explícita
- [ ] Long-running (> 5s) → Saga com state persistido em `saga_state`
- [ ] Eventual consistency (notificação, cache invalidação) → domain event + handler **async idempotent**
- [ ] Concorrência em quota/recurso → `SELECT ... FOR UPDATE` em tx, **não** subir isolation level

### Code Calisthenics

- [ ] Máximo **1 nível de indentação** por método — early returns
- [ ] **Sem `else`** — guard clauses. Switch OK.
- [ ] Primitivos envolvidos em Value Objects (Email, CPF, CNPJ, Money) — auto-validados
- [ ] Collections com invariantes em tipo próprio (`VoteCollection`, não `[]Vote` solto)
- [ ] **Um ponto por linha** (Demeter) — evitar `a.b().c().d()` em cascata de entidades
- [ ] Nomes completos — sem abreviação fora de idiomático (`ctx`, `err`, `i`)
- [ ] Entidade com **5–10 campos**. Acima: cheiro de God Object → dividir agregados. Ver [[../antipatterns/god-object|AP-002]].
- [ ] Use case com **3–5 deps**. Acima de 7: cheiro → Saga ou composição
- [ ] **Sem `SetX`** em entidades — métodos de domínio que expressam intenção (`Cancel(reason)`, `Activate()`)

### Validação antes de concluir task

```bash
<build> ./...                                 # compila sem erro
<lint/vet> ./...                              # sem warning
<codegen> && git diff --exit-code <gen-path>  # generated em dia
<test> -race -count=1 ./...                   # unit passa
<test> -tags=integration -count=1 ./...       # integração passa
<coverage> --threshold-file=.coverage.yml     # cobertura por camada
<security-scanner> -severity high ./...       # sem findings graves
<vuln-check> ./...                            # sem CVEs abertos
```

## ❌ NÃO FAZER

### Código

- ❌ Descartar erro silenciosamente (`_ = fn()`, `catch {}` vazio) — **PROIBIDO absoluto**. Ver [[../runtime-antipatterns/_moc|OP-020]].
- ❌ Comparar erro por string (`err.Error() == "not found"`) — use tipo/sentinel
- ❌ `float` / `double` pra dinheiro — integer (centavos)
- ❌ `float` pra proporção com regra de negócio — `DECIMAL/NUMERIC`
- ❌ `panic`/`throw` em produção fora de inicialização irrecuperável
- ❌ `any`/`interface{}` onde tipo concreto serve
- ❌ Abreviações: `uow`, `condo`, `q`, `repo`
- ❌ Global mutation setters (`SetGlobalLevel`) — quebra testes paralelos

### Arquitetura

- ❌ Importar framework HTTP em `core/` ou `domain/` — viola Clean Arch
- ❌ Importar SDK externo (Stripe, Mux, Twilio) em `domain/`
- ❌ Chamar use case de **outro** bounded context diretamente — use domain events. Ver [[../runtime-antipatterns/_moc|OP-009, OP-010]].
- ❌ Misturar commands e queries no mesmo use case (CQRS)
- ❌ Lógica de negócio em handlers — handlers só parse/call/serialize
- ❌ Lógica de negócio em middleware — middleware é transversal
- ❌ Abstrações sem 2+ implementações reais (YAGNI)
- ❌ Criar muitos arquivos pequenos — prefira implementar em arquivo existente
- ❌ Features não solicitadas ("enquanto estou aqui...")
- ❌ Mudar arquitetura sem discussão explícita com o time

### Banco de dados

- ❌ `SELECT *` em query gerada — liste colunas
- ❌ Editar arquivos de codegen manualmente
- ❌ Migration destrutiva em prod: `DROP TABLE`, `DROP COLUMN`, `ALTER COLUMN` que perde dado
- ❌ `FLOAT` / `DOUBLE` pra dinheiro ou proporção
- ❌ UPDATE/DELETE em tabela imutável por design (`audit_trail`, `assembly_votes`, etc.)
- ❌ Query em tabela com soft delete sem `WHERE deleted_at IS NULL`
- ❌ Query multi-tenant sem filtro de `tenant_id`
- ❌ Begin/Commit manual fora de `UnitOfWork.Run()`
- ❌ Rodar migration em múltiplas instâncias simultâneas (tasks ECS paralelas)

### Segurança

- ❌ Commitar `.env` ou qualquer arquivo com secret
- ❌ Tokens em `localStorage` / `sessionStorage` (frontend)
- ❌ `sk_live_*` em ambiente de dev — **nunca**
- ❌ `InsecureSkipVerify: true` em TLS
- ❌ Processar webhook sem validar assinatura. Ver [[../runtime-antipatterns/_moc|OP-023]].
- ❌ Logar CPF/CNPJ, senhas, tokens completos, emails sem mascarar em debug. Ver [[../runtime-antipatterns/_moc|OP-022]].
- ❌ Retornar PII em mensagem de erro ao cliente
- ❌ CORS com `*` em rota protegida
- ❌ Validar input apenas no frontend — **sempre** server-side
- ❌ Combinar MCP de busca web + MCP destrutivo sem aprovação humana (prompt injection)

### Processo

- ❌ Implementar sem ler código existente primeiro
- ❌ Avançar pra próxima task sem a atual compilar
- ❌ Ignorar erro de compilação — corrija antes de avançar
- ❌ Aceitar info de terceiro sem verificar doc oficial via Context7
- ❌ Pular fase `Discuss` porque "já sei o que fazer"
- ❌ Pular TDD — "testes depois" é anti-padrão provado pelo legacy
- ❌ Skip de build+lint+test antes de commit
- ❌ `git push --force` em branch compartilhada
- ❌ Commit com secret — se acontecer, **rotacionar imediatamente**
- ❌ Usar projeto legado como fonte de verdade — é **referência**, contém decisões já revisadas. Ver [[../methodology/legacy-as-reference]]
- ❌ Copiar código/convenções do legado sem validar contra requirements atualizados

## Links

- [[_moc]] — princípios
- [[../antipatterns/_moc]] — catálogo detalhado com AP-XXX
- [[../methodology/sdd-workflow]] *(a criar)*
- [[../methodology/legacy-as-reference]]
- [[../../30-agents/autonomous-execution]] — protocolo de decisão autônoma
