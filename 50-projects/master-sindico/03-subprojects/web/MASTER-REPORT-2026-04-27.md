---
title: Web — Master Report (auditoria + revisão + CVE + coverage + security)
type: audit
tags:
  - audit
  - master-sindico
  - web
  - '2026-04-27'
  - master-report
  - cve
  - coverage
  - security
  - owasp
project: master-sindico
subproject: web
created: '2026-04-27T19:15:00.000Z'
updated: '2026-04-27T19:15:00.000Z'
aliases:
  - web-master-report-2026-04-27
  - master-report-fe-2026-04-27
---

# Web — Master Report 2026-04-27

> **Conteúdo completo (versão FS):** `Development/web/.audit/MASTER-REPORT-2026-04-27.md`
> Espelho aqui no vault para descoberta via graph-rag.

Consolidação total da sessão de auditoria + monitoramento + revisão + CVE + coverage + security do sub-projeto `web/` em 2026-04-27.

## Resumo

| Dimensão | Status |
|---|---|
| Lint (`bun run check`) | ✅ 0 errors / 0 warnings (era 16/14) |
| Tests vitest | ✅ 81 pass / 0 fail (era 80) |
| Coverage schemas | ✅ 100% |
| Coverage ui | ❌ 24% (target 90) |
| Coverage auth | ❌ 5% (target 75) |
| CVE | 🟡 2 moderate (postcss + follow-redirects no admin) |
| CVE critical/high | ✅ 0 |
| Static security XSS | ✅ limpo (zero sinks perigosos) |
| Hardcoded secrets | ✅ limpo |
| Findings catalogados | 30 totais |
| Findings 🟢 resolvidos | 14 |
| Findings 🔴 pendentes | 3 (A-002, A-003, NEW-001) |
| Findings 🟠 pendentes | 4 |
| Findings 🟡 pendentes | 10 |

## Trabalho do Agente 2 (Lotes 2 + 3)

✅ **APROVADO COM CONFIANÇA ALTA** — 11 findings marcados resolvidos: 8 ✅ + 3 ⚠️ ressalva tática + 0 ❌. Padrão sênior, comentários referenciam D-134/STEERING/vault. 3 resoluções extras não marcadas (A-020 cn re-export, A-022 packages/core, .kiro deletado).

Detalhe completo da revisão em [[REVIEW-2026-04-27-AGENTE-1]].

## Findings críticos (3)

- **A-002** OTP code via search params (vazamento URL) — backend coord (`reset_token` opaco)
- **A-003** PKCE gap — auth-context POST email+password vs STEERING #18 — backend coord
- ~~**NEW-001** railway.json 6 apps~~ — 🟢 **RESOLVIDO 2026-04-27 por D-138 / ADR-0045 Cloudflare-only**: 7 railway.json removidos, wrangler.toml criados, deploy scripts package.json. Em troca, novos action items D-138-A (Cloudflare Containers GA?) e D-138-B (Postgres provider externo).

## CVE detalhado

- GHSA-r4q5-vmmm-2653 (follow-redirects ≤1.15.11 via axios) — Custom Auth Headers leak em redirects — fix `bun update follow-redirects`
- GHSA-qx2v-qp2m-jg93 (postcss <8.5.10 via build deps) — XSS em CSS Stringify — fix `bun update postcss`

Ambos no admin, fix rápido pré-deploy prod.

## Coverage gaps (priorização)

- packages/ui: 5/7 componentes 0% (Button, Input, Card, OtpInput, Tabs) — 12h estimado
- apps/auth: 12/14 rotas 0% (sign-in, sign-up, callback, forgot-password 3, verify-email, set-password, welcome, logout, device-mismatch, _auth, __root) — 24h+ estimado
- apps/admin: 0% (sem testes, app novo) — backlog

## Recomendações imediatas

1. Fix NEW-001 (railway 6 apps) — bloqueia deploy prod
2. Fix CVE moderate (`bun update`)
3. Fix NEW-003 (wire requestId)
4. Coordenar backend A-002 + A-003

Detalhamento + matriz priorizada (16 itens) em [[../.audit/MASTER-REPORT-2026-04-27|Master Report FS]].

## Vault graph-rag aplicado

Walking BFS:
- CLAUDE.md (master-sindico) → STATE → STEERING → ROADMAP → 03-subprojects/web/{README, security, design-system, tasks, patterns}
- D-134 → admin-privilegios.md → ui-catalog/admin/main → _components-and-screens
- 18+ notas lidas, 3 atualizadas, 3 novas criadas (AUDIT, REVIEW, MASTER-REPORT)

## Links

- [[AUDIT-2026-04-27]] — auditoria inicial 26 findings
- [[REVIEW-2026-04-27-AGENTE-1]] — review do Agente 2
- [[tasks]] — backlog FE-### + DT-WEB-### (DT-WEB-005 🟢 resolvido)
- [[../../STATE]] — D-134, D-106, D-117
- [[../../STEERING]] — #15 zero-trust, #18 PKCE, #20 PII off-bundle, #25 coverage
- [[security]] — defesa browser-side
- [[../../09-testing/coverage-thresholds]] — gates

## Validação Agente 1

- ✅ `bun run check` exit 0 (16→0 errors)
- ✅ Vitest 81 pass / 0 fail
- ✅ `bun audit` executado — 2 moderate, 0 critical
- ✅ `bun run test:coverage` 3 packages — coverage real medido
- ✅ Static security scan grep — XSS sinks + secrets — limpo
- ✅ OWASP Top 10 mapeamento aplicado
- ✅ Code Calisthenics + SOLID + Clean Code avaliação por arquivo
- ✅ Cross-check Context7 (SolidJS, TanStack, Kobalte, Vitest, Zod)
- ✅ Vault graph-rag BFS walking (18+ notas)
