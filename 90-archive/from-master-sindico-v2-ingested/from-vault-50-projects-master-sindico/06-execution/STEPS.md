---
title: STEPS — Zero → Produção · Master Síndico
type: project
tags: [master-sindico, execution, steps, zero-to-prod]
project: master-sindico
created: 2026-04-22
---

# STEPS — Zero → Produção

Sequência macro do projeto. Cada passo tem **gate** pra passar ao próximo.

---

## 0. Discovery (concluído)

- [x] Visão de produto estabelecida
- [x] Personas mapeadas (5)
- [x] Modelo de negócio (trial por persona, reputação Bronze→Diamante)
- [x] Concorrência avaliada (ERPs condominiais tradicionais vs proposta de governança)
- [x] Ubiquitous language validada com dev do cliente

**Gate**: visão escrita, aprovada, em [[../00-product/vision]].

---

## 1. Onboarding agente ↔ dev (concluído)

- [x] Dev respondeu questionário (ver [[../../../30-agents/playbooks/onboarding-novo-projeto]])
- [x] Scaffold instanciado
- [x] Stack escolhida (D-001..D-010)
- [x] Não-negociáveis registrados em [[../STEERING]]
- [x] Constituição técnica aplicada ([[../../../CLAUDE#8]])

**Gate**: CLAUDE.md do projeto preenchido; STATE com kick-off D-000.

---

## 2. Setup de repo + CI (concluído)

- [x] Monorepo com backend/, web/, app/
- [x] Cada sub-projeto com `.claude/settings.json` próprio
- [x] Branch strategy: `main` sempre deployable; feature branches curtas
- [x] GitHub Actions CI: build + lint + test + coverage + gosec + govulncheck
- [x] Railway linked (deploy em push to main de `backend/`)
- [x] Stripe CLI + webhook local funcionando

**Gate**: CI verde em PR de exemplo; primeira rota `/health` em staging.

---

## 3. Sprint 0 — Base Clean Architecture (concluído)

- [x] `core/contracts` (IModule, IUseCase, IRepository, IUnitOfWork, HTTPRouter, Context)
- [x] `core/errors` (AppError + sentinels + ValidationError)
- [x] `shared/providers/database` (pgxpool + UoW + Migrator)
- [x] `shared/providers/cache` (Redis)
- [x] `shared/providers/auth` (Zitadel)
- [x] `shared/middleware` (BeyondCorp stack)
- [x] `server/GinAdapter` (isola Gin)
- [x] `cmd/api/main.go` (entry com DI explícito)
- [x] `/health` + `/health/ready` (liveness + readiness)

**Gate**: servidor sobe; PG e Redis acessam; Zitadel auth OIDC funciona end-to-end (`/auth/login` → `/auth/callback` → `/api/v1/auth/me`).

---

## 4. Sprint 1..N — Features por módulo (em curso)

Loop para cada feature:

1. **Discuss**: ler requirement + design + AUDIT + STATE
2. **Plan**: escrever `<plan>` com `<verify>` (contrato)
3. **Execute**: seguir ordem canônica (ver [[development-cycle]])
4. **Verify**: gates verdes (build + vet + test + coverage + security)
5. **Ship**: commit imperativo, PR, review, merge, deploy staging
6. **Audit** (fim do sprint): `sprint-auditor` → `AUDIT.md` atualizado

Status atual:
- ✅ Sprints 0-9 entregues
- ⏳ Sprint 10 em curso (finalização M1)

**Gate entre sprints**: `AUDIT.md` sem 🔴 (high/critical). Sem 🔴🟡 aberto.

---

## 5. QA de sprint (em cada fim de sprint)

- Rodar `sprint-auditor` (skill local em `.claude/skills/`)
- Materializar findings em `11-audit/AUDIT.md`
- Propor fixes pra próxima sprint (criar task-executor entries)
- **Dual-check** (ver [[../../../30-agents/playbooks/dual-check]]) em decisões arquiteturais
- Revisão de dependências + CVE ([[../../../30-agents/playbooks/dependency-audit]] + [[../../../30-agents/playbooks/cve-scan]])
- Performance: `go tool pprof` em endpoints críticos; `k6` em rotas de alta concorrência (em Sprint 11+)

**Gate**: coverage global ≥ 85%; gosec -severity=high = 0; govulncheck = 0.

---

## 6. Staging deploy (entre sprints; contínuo)

- PR mergeado em `main` → GitHub Actions → Railway staging
- Smoke test automatizado em staging (Playwright básico)
- Monitor OTel + Sentry 24h
- Se erro rate > 0.5% ou p95 > 500ms por 15min consecutivos: **rollback**

**Gate pro prod**: 48h em staging sem regressão detectada.

---

## 7. Produção deploy (por marco)

Pré-deploy:
- [ ] AUDIT.md sem 🔴🟡
- [ ] STATE sem B-### (bloqueadores)
- [ ] CHANGELOG atualizado
- [ ] Release Notes na PR body
- [ ] DB snapshot pré-migration (se aplicável)
- [ ] Feature flags criadas (se necessário rollout progressivo)
- [ ] Runbook atualizado se mudança operacional
- [ ] Oncall notificado (canal)

Deploy:
- Railway promote staging → prod (ou canary se feature flag)
- `/health/ready` verifica PG + Redis + Zitadel reachable
- Smoke test em prod (endpoint crítico por sub-projeto)

Pós-deploy (24h):
- Monitorar métricas (error rate, p95, throughput)
- Monitorar Sentry
- Stripe/Mux/Livekit webhooks recebidos sem erro

**Rollback** se: error rate > 0.5%, p99 > 1s por 15min, Sentry incident, webhook falha sistemática.

Ver [[../../../30-agents/playbooks/rollback]] e [[playbooks/rollback]].

---

## 8. Operação contínua

- **Oncall** em Sprint 11+ (assembleias live exigem)
- **SLO review** mensal: uptime, p95, error budget
- **Post-mortem** de qualquer incidente > 15min (formato blameless)
- **Dep-audit recorrente** (semanal ou por release)
- **CVE response** conforme SLA ([[../../../30-agents/playbooks/cve-scan]])
- **LGPD cumprimento** — requests de exclusão atendidas em ≤ 15 dias

---

## 9. Iteração contínua

- Retro semanal (sprint-auditor + self-review dev)
- Feedback de usuários via canais definidos
- Métricas de produto (ativação, retenção, NPS) → input pra próximo sprint
- Backlog em [[../05-tasks/backlog]] revisado mensalmente

---

## Marco de transição entre fases

| De → Para | Gate |
|---|---|
| Sprint 0 → 1 | Base Clean Arch + `/health` funcionando em staging |
| Sprint N → N+1 | AUDIT 🔴🟡 = 0 + coverage OK + gates verdes |
| Pré-M1 → M1 deploy prod | Sprint 10 finalizada + staging 48h limpo + runbook |
| M1 → M2 | M1 critérios de saída atingidos (ver [[../ROADMAP]]) |
| M3 → Escala | SLO 99.5% mantido 2 semanas + load test aprovado + D-### infra migração |

---

## Links

- [[_moc]]
- [[development-cycle]]
- [[qa-cycle]]
- [[deploy-cycle]]
- [[playbooks/rollback]]
- [[playbooks/incident-response]]
- [[../ROADMAP]]
- [[../STATE]]
- [[../11-audit/AUDIT]]
- [[../../../10-knowledge/methodology/sdd-workflow]]
- [[../../../30-agents/playbooks/plan-and-execute]]
