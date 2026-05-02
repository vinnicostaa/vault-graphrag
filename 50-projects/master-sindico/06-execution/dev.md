---
title: Dev — Fluxo diário
type: guide
tags: [execution, dev, sdd, daily-flow, master-sindico, v2]
source: legado 06-execution + vault/10-knowledge/methodology/sdd-workflow + playbooks research-loop/dual-check/plan-and-execute
created: 2026-04-23
updated: 2026-04-23
---

# Dev — Fluxo diário

Fluxo padrão de uma sessão de desenvolvimento. Aderência a **SDD 5-fase** (Discuss → Plan → Execute → Verify → Ship) + **Gate 0** (abrir AUDIT) + **research-loop** + **dual-check** + **commit format**.

---

## 0. Gate 0 — abrir AUDIT (obrigatório toda sessão)

```
1. Abrir vault/50-projects/master-sindico/11-audit/AUDIT.md (canônico legado)
   + master-sindico-v2/11-audit/AUDIT.md (canônico v2)
2. Checar 🔴 (crítico) e 🟡 (relevante)
3. Se 🔴 aberto: bloqueia task nova — prioridade absoluta
4. Se 🟡 aberto relacionado à task da sessão: resolver junto ou registrar plano
```

**Regra dura**: sem Gate 0 não começa nada.

---

## 1. Task pick

1. Abrir `[[../05-tasks/by-sprint/sprint-<N>]]` (sprint em curso)
2. Selecionar task MUST da sprint que esteja com DoR (Definition of Ready) atendido ([[../DoR-DoD#definition-of-ready-dor]])
3. Se DoR não está atendido: research-loop + dual-check pra resolver dependência antes

---

## 2. SDD 5-fase

### 2.1 Discuss

- Ler o **requirement** específico (`04-requirements/functional/<module>.md`)
- Ler o **aggregate** e **invariantes** afetados (`01-domain/aggregates/` + `01-domain/invariants.md`)
- Ler **AUDIT** em busca de findings relacionados (A-###)
- Ler **STATE** em busca de decisões relacionadas (D-###)
- Alinhar com playbook de módulo (ex: `02-architecture/patterns.md`)

Saída: `<discuss>` mental ou em rascunho; objetivo claro + restrições explicitadas.

### 2.2 Plan

Escrever `<plan>` com:
- **Escopo**: o que muda (migrations, entidades, use cases, handlers, frontend, docs)
- **Arquivos tocados**: lista (adicionar/modificar/remover)
- **Contratos**: OpenAPI, domain events publicados/consumidos, tests que passam
- **Verify block**: `<verify>` com AC (acceptance criteria) testáveis
- **Rollback plan**: como reverter se for bad commit

Saída: `<plan>` em comentário na task ou em rascunho local.

### 2.3 Execute — ordem canônica

```
1. Migration SQL + CHECK constraints (particionadas por módulo, p.ex. 001_identity_..._0001.sql)
2. sqlc query (colunas explícitas — nunca SELECT *; AP-021 mitigado)
3. Domain entity / VO
   - Entidade: campos privados + getters
   - Factory NewX(ctx, ...inputs) → (*Entity, error) com validação
   - Factory ReconstructX(row, ...) → *Entity sem validação (veio do repo)
4. Repo interface em domain/
5. Domain event (se cross-context)
6. Use case CQRS
   - Command: *_command.go com Execute(ctx, input) → (Output, error)
   - Query: *_query.go
   - Arquivos separados
7. Mapper (row ↔ aggregate)
8. Repo impl (infrastructure/database)
9. Provider externo (se SDK; infrastructure/providers/<provider>/)
   - Interface em shared/providers/<kind>/interface.go
   - Impl em infrastructure/providers/<provider>/
10. Handler + rota (OpenAPI-first — spec antes do código)
11. Wire-up no module + main (DI manual explícito)
12. Testes:
    - Unit: table-driven + edge cases + PBT em invariants críticos
    - Integration: testcontainers-go (DB + Redis reais)
    - HTTP: handler isolado com mocks
```

Code Calisthenics Go-flavor em todo o caminho:
- Sem bloco `else` — early return
- Sem `any`/`interface{}` onde tipo concreto serve
- Máximo 1 nível indent em métodos razoáveis
- Sem `try/catch` defensivo (só captura se tem ação útil)
- Sem abstração prematura — 2+ implementações reais ou não abstrai (YAGNI)

### 2.4 Verify — gates duros

```bash
# Obrigatórios (não mergeia sem):
go build ./...
go vet ./...
go test -race -count=1 ./...
go test -tags=integration -count=1 ./...
gosec -severity=high ./...          # 0 findings
govulncheck ./...                    # 0 CVEs reachable

# Coverage check:
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out    # domain 95%, app 90%, infra 85%, http 75%, global ≥ 85%

# Frontend:
cd web && bun lint && bun test && bun run build
# Mobile:
cd app && dart analyze && flutter test
```

### 2.5 Ship

Commit format (imperativo, ≤ 72 chars primeira linha):
```
<type>(<scope>): <what>

<why — breve>

<footer se relevante>
Refs: T-### (task) / A-### (finding) / D-### (decision)
Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `sec`.

Scopes: `identity`, `billing`, `institutional`, `commercial`, `content`, `assembly`, `cross-domain`, `web`, `mobile`, `ops`, `docs`.

Exemplo:
```
feat(commercial): add Connect Me síndico→empresa RFP flow

Implementa RFP estruturado com quota anual (I-030). Timeline entry
gerada no publish via evento de domínio (BR-003).

Refs: T-100, COM-002, D-004
Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
```

PR:
- Título claro, mesmo estilo do commit primeiro
- Body: summary + test plan + risk
- Label: `m1`, `m2`, `m3`, `hotfix`, `dependabot`
- Reviewers: pelo menos 1 humano + CI

Merge: squash-merge (histórico limpo em `main`); delete branch.

Deploy staging: automático via Railway após merge.

---

## 3. Research-loop (antes de decidir)

Quando bater em decisão arquitetural/pattern/SDK não-trivial:

1. **Analisar o que já existe** — grep em `90-ingested/`, `vault/10-knowledge/`, `vault/50-projects/master-sindico/`
2. **Doc oficial** — Context7 MCP ou WebFetch direto no site oficial (nunca tutorial aleatório)
3. **Big tech engineering blogs** — buscar em `13-research/` ou procurar (Netflix/Google/Meta/Stripe)
4. **Patterns aplicáveis** — mapear em `02-architecture/patterns.md`
5. **Dual-check** — segundo agente em contexto limpo revisa
6. **Registrar** — D-### em STATE + ADR em `02-architecture/adr/` se impacto arquitetural

Playbook completo: [[playbooks/research-loop]].

---

## 4. Dual-check (obrigatório)

Em:
- arquitetura (novo BC, event bus, ABAC nova regra)
- contratos públicos (OpenAPI, domain events schema)
- schema DB (migration destrutiva, índice)
- security policy (CORS, rate limit, ABAC matrix)
- regra de negócio em prod
- conflito de fontes
- modo autônomo

Formato: [[playbooks/dual-check]] + log em [[../11-audit/dual-check-log/_moc]].

---

## 5. Dep-audit + CVE scan

Antes de adicionar dep nova:
- [[playbooks/dep-audit]] — verificar license, manutenção ativa, alternativas
- [[playbooks/cve-scan]] — `govulncheck` deve passar

Recorrente (sprint-auditor domingo): ambos.

---

## 6. Modo autônomo — limites

Herda [[../../vault/30-agents/autonomous-execution]]. Pausa obrigatória em:
1. `DROP TABLE/COLUMN` ou migration destrutiva sem expand-contract
2. Deploy em produção
3. `git push --force` em main/develop
4. Remover dep crítica (pgx, gin-adapter, go-redis)
5. Operação em dado pessoal real (LGPD)

Em dúvida técnica **não-destrutiva**: research-loop + dual-check resolve; **nunca** pausar por "não sei decidir X" — o playbook resolve.

---

## 7. Checklist fim de sessão

- [ ] STATE atualizado (D-### / DT-### novos)
- [ ] AUDIT com findings novos/fechados
- [ ] SESSION_CHARTER reflete progresso
- [ ] Notas novas com frontmatter + ≥ 2 `[[links]]`
- [ ] MOCs impactados atualizados
- [ ] Gates verdes
- [ ] `tasks.md` reflete progresso
- [ ] Docs sincronizadas (README, OpenAPI, CHANGELOG)

---

## Links

- [[STEPS]] — macro zero → prod
- [[qa]] — QA weekly
- [[deploy]] — deploy flow
- [[rollback]]
- [[incident]]
- [[playbooks/_moc]]
- [[../DoR-DoD]]
- [[../CLAUDE]]
- [[../../vault/10-knowledge/methodology/sdd-workflow]]
- [[../../vault/30-agents/playbooks/plan-and-execute]]
