---
title: Mobile Sprints — MOC
type: moc
tags: [moc, mobile, sprints, master-sindico]
project: master-sindico
stack: mobile
created: 2026-04-24
updated: 2026-04-24
---

# Mobile Sprints — Map of Content

Índice dos sprints mobile. App Flutter foi criado **2026-04-21 01:33** — Sprints 1-8 são todos N/A (mobile não existia). Sprint 9 entregou assembly (**fora do escopo M1 declarado** — A-FASE10-DEV-004 🟡). Sprint 10 é o reset de escopo para M1 correto.

## Cronograma

| Sprint | Período | Status | Observação |
|---|---|---|---|
| [[sprint-0]] | 2026-04-21 01:33 | ✅ | Scaffold Flutter Clean Arch + BLoC + GetIt + GoRouter + Dio + Dartz |
| [[sprint-1]] | 2026-04-07 → 2026-04-20 | ⏳ N/A | Mobile não existia |
| [[sprint-2]] | 2026-04-21 → 2026-04-27 | ⏳ N/A | Sem feature dev |
| [[sprint-3]] | 2026-04-28 → 2026-05-04 | ⏳ N/A | Sem feature dev |
| [[sprint-4]] | 2026-05-05 → 2026-05-11 | ⏳ N/A | Sem feature dev |
| [[sprint-5]] | 2026-05-12 → 2026-05-18 | ⏳ N/A | Sem feature dev |
| [[sprint-6]] | 2026-05-19 → 2026-05-25 | ⏳ N/A | Sem feature dev |
| [[sprint-7]] | 2026-05-26 → 2026-06-01 | ⏳ N/A | Sem feature dev |
| [[sprint-8]] | 2026-06-02 → 2026-06-08 | ⏳ N/A | Sem feature dev |
| [[sprint-9]] | 2026-04-22 (autônomo) | ✅ mas fora escopo | Assembly full (M3) — A-FASE10-DEV-004 🟡 |
| [[sprint-10]] | 2026-04-23 → 2026-05-18 | 🟡 em execução | M1 correto (auth OIDC + home + timeline + onboarding + conta) |

## Roadmap mobile (marcos)

- **M1 (2026-05-08 ou 05-18)**: splash + auth + home morador + timeline read + onboarding + conta + Hive offline + FCM push → Sprint 10 (MB-001..MB-203)
- **M2 (2026-06-20)**: síndico light + Connect Me + vídeo + votação fornecedor → Sprint 11 (MB-220..MB-363)
- **M3 (2026-07-20)**: assembleia live (já parcialmente codada — reaproveitar) + LMS thin + store submit → Sprint 12 (MB-400..MB-513)

## Dívidas técnicas (DT-MB-###)

Ver [[../../tasks#dt-dividas-tecnicas-hoje-do-diff-appreal-vs-alvo]] — 13 DTs catalogados. Principais Sprint 10:

- **DT-MB-001** minSdk 19→26 — MB-001
- **DT-MB-002** bundle ID normalizar — MB-001
- **DT-MB-003** deps faltantes (`bloc_concurrency`, `hydrated_bloc`, `freezed`, `hive_flutter`, `local_auth`, FCM, `app_links`, etc.) — MB-002
- **DT-MB-005** paleta default Flutter → Manual IV — MB-007
- **DT-MB-008** refresh token sem persist — MB-004
- **DT-MB-009/010** permissions Info.plist + AndroidManifest — MB-012/013

## Findings abertos mobile

| A-### | Severidade | Sprint alvo |
|---|---|---|
| A-FASE10-DEV-003 | 🟠 | Sprint 10 (aplicar D-048/049/050 no código) |
| A-FASE10-DEV-004 | 🟡 | Sprint 10 (reescopar priorizando timeline) |

## Decisões relevantes

- **D-048** Bloc + `bloc_concurrency` + `hydrated_bloc` — fechado vault, não aplicado no código
- **D-049** `freezed` desde M1 — fechado vault, não aplicado
- **D-050** freeRASP report-only M1, gated M3+ — fechado vault, não aplicado
- **D-054** Lock 90d bypass 4-eyes — MB-352 (M2)
- **D-057/D-058** Tiers + roles + Admin D-058 — MB-350..352 (M2)

## Links transversais

- [[../../tasks|tasks monolítico mobile]]
- [[../../../../../50-projects/master-sindico/ROADMAP|ROADMAP]]
- [[../../../../../50-projects/master-sindico/STATE|STATE]]
- [[../../../../../50-projects/master-sindico/AUDIT|AUDIT]]
- [[../../../backend/tasks/by-sprint/_moc|Backend sprints MOC]]
- [[../../../web/tasks/by-sprint/_moc|Web sprints MOC]]
