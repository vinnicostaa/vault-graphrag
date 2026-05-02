---
title: Sprint 10 — Web (M1 completo — bloqueador crítico)
type: sprint-spec
tags: [master-sindico, sprint-10, web, tasks]
project: master-sindico
stack: web
sprint: 10
period: "2026-04-23 → 2026-05-18"
status: in-progress
milestone_target: m1
created: 2026-04-24
---

# Sprint 10 Web — M1 completo (auth + cms onboarding + dashboard básico)

**Período**: 2026-04-23 → 2026-05-18 (em curso — pendente decisão macro Opção A/B/C)
**Status**: 🟡 em execução — **bloqueador M1 🔴** (A-FASE10-DEV-005)
**Objetivo**: Executar todo o M1 web num único sprint concentrado — scaffolding real em `apps/auth` + `apps/cms` (morador + síndico onboarding + admin MS base), packages compartilhados (`schemas`, `ui`, core transversal), CI + typecheck + coverage threshold + Sentry + web-vitals.

## Escopo desta sprint (web)

**Sprint mais crítico do projeto**. Web parte praticamente do zero para entregar em 3-4 semanas o que deveria ter levado 8-10 sprints normais. Estimativa [[../../../AUDIT#a-fase10-dev-005]]: 15-25 dias. Com 3-4 semanas é limite — decisão stakeholder sobre Opção A (postergar 18/05) / Opção B (ship 08/05 só backend+mobile; web M1.1 separado) / Opção C (remontagem inline).

## Tasks (agrupadas — tasks detalhadas em [[../../tasks]])

### Setup & infra
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.1 | FE-001..FE-009 setup (scaffold validate, railway.json fix DT-WEB-001, CI, Sentry, web-vitals, Biome custom rules, OpenAPI→TS, preload fonts) | ⏳ | [[../../tasks#setup-infra-compartilhado]] |

### Packages compartilhados
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.2 | FE-010..FE-017 `packages/schemas` Zod canônico (Money, FracaoIdeal, tiers), `packages/ui` atoms+molecules+organism, uno.preset compartilhado | ⏳ | DT-WEB-002/004/005/008/010 |

### Core transversal (por app)
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.3 | FE-020..FE-026 http client (Axios `withCredentials:true`, interceptors 401/403, `X-Request-Id`, fingerprint), SessionContext, route guards, CSRF | ⏳ | CLAUDE.md linha 57-61 (uncomment) |

### App `auth` (M1 completo)
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.4 | FE-030..FE-036 Splash + SignIn + SignUp 5 personas + OTP + Callback Zitadel + Device Mismatch + Logout | ⏳ | AUTH1..AUTH8 |

### App `cms` — Morador M1
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.5 | FE-040..FE-052 M1-M15 (painel, selector condomínio, buscar ID, cadastro unidade, home, timeline, detalhe publicação, plano diretor, eventos, comunicados, avaliar gestão, vínculo, dependentes, encerrar) | ⏳ | M1..M15 |

### App `cms` — Síndico onboarding + base institucional
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.6 | FE-060..FE-068 S3-S6 (cadastrar condomínio, dados gestão, empresa síndica, Home Governança 10 cards — **tela-pivô**), S11-S14 timeline síndico + voice, conta SYS1-SYS3 | ⏳ | S3..S14 + SYS1-3 |

### App `cms` — Admin MS M1
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.7 | FE-070..FE-071 AD1 admin home (gate ABAC duplo) + AD2 gestão tenants (suspender/reativar com audit) | ⏳ | AD1-AD2 |

### App `lp` (scaffold apenas M1)
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.8 | FE-080..FE-081 PUB1 refinar hero + Meta tags + OG + canonical | ⏳ | PUB1 |

### Testing M1
| ID | Título | Status | Ref |
|---|---|---|---|
| web-10.9 | FE-090..FE-093 Vitest + testing-library + MSW + renderWithProviders + fixtures 4 personas + unit tests 75%/85%/90%/90%/95% | ⏳ | coverage-thresholds |

## Dependências

- Sprint anterior: audit trail ([[sprint-9]]).
- Cross-stack: backend Sprint 10 BE-001..BE-015 em paralelo — alguns handlers podem ter contrato ajustado (Unit `unit_type`/`fracao_ideal` vira campo novo na API).
- Decisões: D-020 (stack), D-046 (email Resend M1), D-047 (admin subdomain + `__Host-` cookie + CSP strict), D-057/D-058 (tiers + roles).

## Entregáveis

- 6 apps com conteúdo real (não placeholders) — pelo menos auth + cms + lp.
- M1 user journeys E2E manuais: síndico cria condomínio → convida morador → morador cadastra unidade → timeline read → plano diretor read.
- `bun run check + test + build` verdes em todos os apps.
- Sentry + web-vitals capturando em staging.

## Gates (`<verify>`)

- `bun run check` (biome) ✅ zero warnings
- `tsc --noEmit` por app ✅
- `bun run test` (vitest) ✅ coverage thresholds
- `bun run build` ✅ bundle-size CI
- jest-axe zero violations em tests atoms/molecules
- [[../../tasks#gates-de-entrega-definition-of-done-dod]] DoD 15 pontos ✅

## Pós-sprint

- **O que deu certo**: (preencher)
- **Bloqueadores**: cronograma apertado; possível decisão stakeholder para M1 shipping parcial (só backend+mobile) + web M1.1 em onda 2.
- **Dívidas criadas**: dependentes da Opção escolhida; `packages/core` fica para M2 (DT-WEB-011).
- **Próxima sprint**: Sprint 11 — M2 (Connect Me + Content + Agências + Forum) FE-100..FE-193.

## Links

- [[sprint-9]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../AUDIT#a-fase10-dev-005]]
- [[../../../../ROADMAP]]
- [[../../tasks]]
