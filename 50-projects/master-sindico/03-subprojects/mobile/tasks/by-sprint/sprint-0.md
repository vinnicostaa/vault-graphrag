---
title: Sprint 0 — Mobile (Scaffolding Flutter)
type: sprint-spec
tags: [master-sindico, sprint-0, mobile, tasks]
project: master-sindico
stack: mobile
sprint: 0
period: "2026-04-21 (1h33 — scaffolding)"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 0 Mobile — Scaffolding Flutter

**Período**: 2026-04-21 01:33 (criação do app)
**Status**: ✅ concluído (scaffolding)
**Objetivo**: Criar o esqueleto Flutter do app cliente morador/síndico — estrutura Clean Architecture, roteamento GoRouter, BLoC + GetIt/Injectable, Dio, Dartz, OIDC, testes Mocktail.

## Escopo desta sprint (mobile)

Mobile só começou a existir em 21/04/2026 — bem depois do backend Sprint 0 (março) e do web Sprint 0 (13/04). Ou seja, mobile "pulou" os sprints 1-8 cronologicamente. Sprint 9 mobile foi a implementação adiantada da feature assembly (que é M3 no roadmap, mas foi codada 22/04 por A-FASE10-DEV-004 🟡).

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| mobile-0.1 | `flutter create` com bundle `com.mastersindico.br.mastersindico` (DT-MB-002 normalizar) | ✅ | `Development/app/` | DT-MB-002 | — |
| mobile-0.2 | `pubspec.yaml` deps base: `flutter_bloc, equatable, get_it, injectable, go_router, dio, dartz, oidc, mocktail, bloc_test` | ✅ | `Development/app/pubspec.yaml` | DT-MB-003 | — |
| mobile-0.3 | Estrutura Clean Arch `lib/{core, features/{auth, home, assembly}, shared}` | ✅ | `Development/app/lib/` | — | — |
| mobile-0.4 | `core/network` com `NetworkInfoImpl` e interceptors stub | ✅ | `Development/app/lib/core/network/` | — | — |
| mobile-0.5 | `core/di/injection.dart` + `injection.config.dart` gerado | ✅ | `Development/app/lib/core/di/` | — | — |
| mobile-0.6 | `AuthRemoteDatasourceImpl` + `AuthLocalDatasourceImpl` (chaves não-canônicas; DT-MB-006/007) | ✅ parcial | `Development/app/lib/features/auth/data/` | DT-MB-006, DT-MB-007 | — |
| mobile-0.7 | `AuthBloc` básico com `AuthLoggedIn`/`AuthLoggedOut` (sem `AuthCheckRequested`/`AuthRefreshRequested` ainda) | ✅ básico | `Development/app/lib/features/auth/presentation/bloc/` | — | — |
| mobile-0.8 | `main.dart` + `App` widget + theme básico com paleta `#6C63FF` (default Flutter — DT-MB-005) | ✅ | `Development/app/lib/main.dart` | DT-MB-005 | — |
| mobile-0.9 | Android `build.gradle.kts` `minSdk=19` (DT-MB-001) | ✅ débito | `Development/app/android/app/build.gradle.kts` | DT-MB-001 | — |
| mobile-0.10 | iOS `Info.plist` sem permissions (DT-MB-009) | ✅ débito | `Development/app/ios/Runner/Info.plist` | DT-MB-009 | — |

## Dependências

- Sprint anterior: nenhuma (primeiro sprint mobile).
- Cross-stack: backend OIDC pronto.
- Decisões: D-048 (Bloc + `bloc_concurrency` + `hydrated_bloc`), D-049 (`freezed` desde M1), D-050 (freeRASP report-only M1) — **NÃO APLICADAS** ainda no código (A-FASE10-DEV-003).

## Entregáveis

- `fvm flutter analyze` passa.
- `fvm flutter test` passa (testes unitários básicos).
- App compila Android + iOS (sem codesign) localmente.

## Gates (`<verify>`)

- `fvm flutter analyze` ✅ zero warning
- `fvm flutter test` ✅
- `fvm flutter build apk --debug` ✅
- `fvm flutter build ios --no-codesign` ✅

## Pós-sprint

- **O que deu certo**: Clean Arch + BLoC + GetIt em lugar desde o dia 1.
- **Bloqueadores**: D-048/049/050 não aplicadas (A-FASE10-DEV-003 🟠).
- **Dívidas criadas**: DT-MB-001..013 (15 dívidas catalogadas — ver [[../../tasks#dt-dividas-tecnicas-hoje-do-diff-appreal-vs-alvo]]).
- **Próxima sprint**: Sprints 1-8 são **N/A** (mobile não existia); pulo para [[sprint-9]] que materializou assembly.

## Links

- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
- [[../../tasks|mobile tasks monolítico]]
