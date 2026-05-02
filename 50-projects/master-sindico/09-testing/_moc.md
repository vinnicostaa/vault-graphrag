---
title: MOC — 09-testing
type: moc
tags: [moc, master-sindico, v2, testing]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 09-testing

Test strategy pirâmide **70-20-10** (unit/integration/E2E), PBT obrigatório em invariantes críticos, gates CI duros, coverage thresholds por camada. Especializa [[../../vault/10-knowledge/testing/testing-strategy]] com stack Go 1.26 + testcontainers-go + `pgregory.net/rapid` + Playwright + k6.

---

## Arquivos

- [[test-strategy]] — Pirâmide 70-20-10, quando cada tipo, estimativa por BC (~1600 tests total), gates CI duros (go build/vet/test-race/integration/gosec/govulncheck + coverage parser).
- [[unit]] — Go unit tests: table-driven obrigatório, stubs hand-rolled (sem mockery), testify/require, AAA, nomenclatura `TestX_when_Y_should_Z`, FakeClock canônico, cobertura por camada.
- [[integration]] — testcontainers-go (PG 18 + Redis 7 real), tag `//go:build integration`, helper reusable, cross-tenant isolation 100+, TOCTOU concorrente, idempotency webhook, rate limit cleanup.
- [[e2e]] — Playwright (web SolidJS) + Patrol/integration_test (Flutter), cenários críticos (signup, Connect Me LGPD, upload Mux 90d, assembleia live), smoke tests pós-deploy, data isolation.
- [[load]] — k6: auth storm (20rpm burst 5), assembleia 10k viewers M3, upload vídeo burst, Connect Me burst, webhook Stripe retry; SLO targets (99.5% avail, p95 < 500ms).
- [[security]] — SAST (gosec HIGH), SCA (govulncheck 0 reachable), secret scan (trufflehog), Semgrep custom (PII, SELECT *, HMAC-before-parse), DAST OWASP ZAP baseline (M3+ stretch), pen-test checklist OWASP Top 10 + LGPD + multi-tenant.
- [[pbt]] — `pgregory.net/rapid` em invariantes críticos: INV-021 fração (100 casos), INV-016 quota (300), INV-086 ABAC (400, ~22ms), INV-071 vote TOCTOU, INV-043 evaluation TOCTOU, pagination, Money.
- [[coverage-thresholds]] — domain 95%, application 90%, infrastructure/database 85%, infrastructure/http 75%, middleware 90%, core+pkg 95%, global ≥85%; `.coverage.yml` canônico; script parser Python; CI gate.

---

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
- [[../STEERING]] — não-negociáveis (coverage §25, gates §26, TDD §28)
- [[../01-domain/invariants]] — INV-### testáveis (ver mapping por tipo enforcement)
- [[../01-domain/business-rules]] — BR-###
- [[../07-quality/_moc]] — quality (clean code/arch, patterns, antipatterns)
- [[../07-quality/PATTERNS]] — patterns testados em cada tipo
- [[../07-quality/antipatterns]] — AP-### com detecção em CI
- [[../08-security/_moc]] — security posture BeyondCorp
- [[../08-security/cve-process]] — fluxo de CVE
- [[../11-audit/AUDIT]] — findings A-### legados
- [[../13-research/google-arch/_destilado]] — SRE SLI/SLO/SLA
- [[../13-research/netflix/_destilado]] — resilience patterns + chaos
