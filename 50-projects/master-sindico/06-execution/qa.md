---
title: QA — Sprint-auditor weekly
type: guide
tags: [execution, qa, sprint-auditor, master-sindico, v2]
source: legado 06-execution/qa-cycle + playbooks dep-audit/cve-scan/dual-check
created: 2026-04-23
updated: 2026-04-23
---

# QA — Sprint-auditor weekly

QA acontece **semanalmente, domingo**. Executado pelo `sprint-auditor` (skill Claude Code) + review humano.

---

## Cadência

| Dia | Atividade |
|---|---|
| **Dom 22:00** | `sprint-auditor` roda (skill local em `.claude/skills/`) |
| Dom 22:30 | Dev revisa findings, atualiza AUDIT, plano pra próxima sprint |
| Dom 23:00 | SESSION_CHARTER da próxima sprint preenchido |
| Seg 09:00 | Sprint planning — tasks populadas com findings resolvidos |

---

## Sprint-auditor — escopo

### O que ele lê

- Commits da semana (`git log --since="7 days ago"`)
- PRs merged (via `gh pr list`)
- Código tocado (`git diff <sprint-start>..HEAD`)
- [[../11-audit/AUDIT]] — estado atual
- [[../STATE]] — decisões da semana
- [[../SESSION_CHARTER]] — ordens da sprint

### O que ele valida (checklist)

**Código**:
- [ ] `SELECT *` em queries sqlc? (A-021 anti-pattern)
- [ ] `PII em logs` (grep CPF/CNPJ/email/token sem `Masked()`)? (A-010 / DR-019)
- [ ] Guard clauses / early return respeitadas (sem `else`)?
- [ ] Tenant isolation — toda query escopada `WHERE condominium_id` quando aplicável
- [ ] Webhook HMAC verificado **antes** de parse?
- [ ] Use case autoriza via middleware `RequirePermission`, não duplica authorize dentro
- [ ] Provider externo via interface (IR-001 DIP)?
- [ ] Cache invalidado em mutation (via domain event)? (A-007 / AP-007 mitigado)
- [ ] `SELECT ... FOR UPDATE` em pessimistic lock de quota/vote?

**Testes**:
- [ ] Coverage domain ≥ 95%, app ≥ 90%, infra ≥ 85%, http ≥ 75%, global ≥ 85%
- [ ] PBT em invariants críticos (fração ideal, quotas, ABAC matrix, vote único, pagination)
- [ ] Integration tests em novos use cases (testcontainers-go)

**Gates duros** (CI):
- [ ] `go build`
- [ ] `go vet`
- [ ] `go test -race`
- [ ] `go test -tags=integration`
- [ ] `gosec -severity=high` — 0 findings
- [ ] `govulncheck` — 0 CVEs reachable
- [ ] Frontend: `bun lint / bun test / bun run build`
- [ ] Mobile: `dart analyze / flutter test`

**Dependências**:
- [ ] `dep-audit` roda via playbook ([[playbooks/dep-audit]])
- [ ] `cve-scan` roda via playbook ([[playbooks/cve-scan]])

**Docs**:
- [ ] Nova task fechada tem doc correspondente atualizada?
- [ ] OpenAPI sincronizada?
- [ ] CHANGELOG atualizado?
- [ ] ADR criada pra decisão arquitetural nova?

**Processo**:
- [ ] Dual-check executado em decisão não-trivial? (log em [[../11-audit/dual-check-log/_moc]])
- [ ] STATE atualizado com D-### novos?
- [ ] SESSION_CHARTER fechado com progresso?

### O que ele gera

- Findings A-### novos (🔴 crítico / 🟡 relevante / 🟢 resolvido)
- Log em `[[../11-audit/audit-log/<YYYY-MM-DD>.md]]`
- Propostas de T-### pra próxima sprint no backlog
- Relatório resumo (post retrospective)

---

## Dual-check em revisão semanal

Além do dual-check pontual em decisões, sprint-auditor agenda **revisão contextual cruzada** (dual-check no macro):

1. Propositor (sprint-auditor) lista decisões arquiteturais da semana
2. Revisor (segundo agente contexto limpo) revisa sem viés
3. Consolidação — registrar em `[[../11-audit/dual-check-log/<YYYY-MM-DD>.md]]`

Playbook: [[playbooks/dual-check]].

---

## Dep-audit + CVE — recorrente

**Semanal**:
- `dep-audit` completo ([[playbooks/dep-audit]])
  - License check
  - Manutenção ativa (último commit ≤ 6m)
  - Alternativas consideradas
- `cve-scan` ([[playbooks/cve-scan]])
  - `govulncheck` (Go)
  - `npm audit` / `bun audit` (web)
  - `dart pub deps --style=compact` + cross-check CVE db (mobile)

**Por release** (pré-M1/M2/M3): re-rodar dep-audit + cve-scan + pen-test interno.

---

## Performance — pprof + k6

Em endpoints com p99 > 1s ou que tocam votação/assembleia real-time:

```bash
# pprof backend Go
curl -s http://localhost:8080/debug/pprof/profile?seconds=30 > cpu.prof
go tool pprof -http=:7070 cpu.prof

# k6 rotas críticas
k6 run --vus 100 --duration 5m scripts/rfp-flow.js
```

A partir de **Sprint 11+**, k6 roda em cada deploy staging (pipeline automático).

---

## Gates obrigatórios pra fechar a sprint

- [ ] AUDIT 🔴 = 0 (🟡 com plano documentado)
- [ ] Coverage global ≥ 85%
- [ ] Gates CI verdes
- [ ] Staging 48h estável (error rate < 0.5%, p95 < 500ms)
- [ ] Runbooks tocados pela sprint foram atualizados
- [ ] Dual-check executado em decisões não-triviais
- [ ] Dep-audit + CVE-scan sem críticos

---

## Links

- [[STEPS]]
- [[dev]]
- [[playbooks/dep-audit]]
- [[playbooks/cve-scan]]
- [[playbooks/dual-check]]
- [[../11-audit/AUDIT]]
- [[../11-audit/audit-log/_moc]]
- [[../11-audit/dual-check-log/_moc]]
- [[../../vault/30-agents/playbooks/dependency-audit]]
- [[../../vault/30-agents/playbooks/cve-scan]]
