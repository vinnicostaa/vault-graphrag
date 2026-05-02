---
title: Sprint 10 — Mobile (M1 correto — auth + home + timeline)
type: sprint-spec
tags: [master-sindico, sprint-10, mobile, tasks]
project: master-sindico
stack: mobile
sprint: 10
period: "2026-04-23 → 2026-05-18"
status: in-progress
milestone_target: m1
created: 2026-04-24
---

# Sprint 10 Mobile — M1 correto (splash + auth OIDC + home morador + timeline read)

**Período**: 2026-04-23 → 2026-05-18 (em curso)
**Status**: 🟡 em execução — gap vs vault M1 (A-FASE10-DEV-004 🟡)
**Objetivo**: Reescopar M1 mobile conforme ROADMAP real — entregar auth OIDC completo, home shell com bottom nav 4 tabs, timeline read + plano diretor leitura + eventos, onboarding morador (M1-M15 escopo mobile), conta + assinatura, base infra (flavors, Sentry, CI). Aplicar D-048/049/050.

## Escopo desta sprint (mobile)

Sprint 10 mobile é o sprint "reset de escopo" — admite que o Sprint 9 autônomo codou o que não deveria (assembly) e agora prioriza o que vault definiu como M1. Assembly já codado fica como **feature preparada para M3** (não deletar; reutilizar em Sprint 12).

## Tasks (agrupadas — detalhes em [[../../tasks]])

### Setup & infra
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.1 | MB-001 minSdk 19→26 + bundle ID `com.mastersindico.app` | ⏳ | DT-MB-001/002 |
| mobile-10.2 | MB-002 deps M1: `bloc_concurrency, hydrated_bloc, freezed + annotation + json_serializable, hive + hive_flutter, local_auth, device_info_plus, firebase_messaging + core, app_links, crypto, flutter_windowmanager` + `build_runner build` | ⏳ | DT-MB-003 |
| mobile-10.3 | MB-003..MB-009 AppConstants keys, RefreshInterceptor Dio (mutex), NoParams Equatable, paleta Manual IV (#C69C3F/#071A33) ou D-### novo, Sentry placeholder, logger + PII masker | ⏳ | DT-MB-004..007 |
| mobile-10.4 | MB-010..MB-013 easy_localization, Google Services dev, Info.plist permissions (camera/mic/photo/FaceID/ATS), AndroidManifest permissions + network_security_config + intent-filter | ⏳ | DT-MB-009/010 |

### Auth OIDC completo
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.5 | MB-020..MB-027 AuthRemoteDatasource refresh_token + AuthBloc (AuthCheckRequested, AuthRefreshRequested, AuthSessionExpired), OIDC custom scheme bidirecional, LoginForm pt-BR, Logout confirm, DeviceFingerprint + secure storage, Device mismatch page, healthcheck getCurrentUser | ⏳ | APP-AUTH-001/002, APP-SYS-001/004 |

### Design system widgets
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.6 | MB-040..MB-046 AppButton, AppInput, AppCard, AppBadge, AppSpinner, AppEmptyState, AppErrorState + golden tests + SemanticsLabel + ToastHost + SnackbarHost + FLAG_SECURE | ⏳ | — |

### Home shell + morador
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.7 | MB-060..MB-068 Splash bootstrap, ShellRoute bottom nav 4 tabs, selector condomínio, painel morador, timeline lista + pull-to-refresh + detalhe, plano diretor leitura, eventos lista + detalhe + ciência/participação | ⏳ | APP-SYS-001, APP-HOME-001/002, APP-MOR-001..006 |

### Onboarding morador
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.8 | MB-080..MB-085 buscar condomínio, wizard identificação unidade, termo de ciência, status, avaliação obrigatória (GoRouter.redirect gate), gerenciar acessos | ⏳ | APP-MOR-009..014 |

### Conta + assinatura
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.9 | MB-100..MB-105 Conta home, perfil+foto, assinatura + trial countdown + Stripe Custom Tab, segurança + biometria + sessões ativas, LGPD export/delete (biometria obrigatória), sobre | ⏳ | APP-ACC-001..007 |

### Offline M1 (Hive + D-048)
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.10 | MB-120..MB-124 Hive setup + cipher, boxes (user, memberships, timeline:{condId}, events:{condId}, plano-diretor:{condId}), hydrated_bloc + MembershipsBloc, offline banner, NetworkInfoImpl D-048 integration | ⏳ | D-041, D-048 |

### Push M1 (FCM)
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.11 | MB-140..MB-146 Firebase dev/staging/prod, FCM register + onTokenRefresh, handlers fg/bg/terminated + deep-link app_links, Android channels 6 categorias, iOS categories 6, inbox APP-NOT-001/002/003 | ⏳ | BR-MOB-PUSH-003 |

### Testing M1
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.12 | MB-160..MB-166 unit use cases 95%, bloc 90%, integration OIDC + onboarding M1-M8, widget tests LoginPage/HomePage/TimelinePage/EventoDetailPage, golden design system, CI coverage gate 80% | ⏳ | coverage-thresholds |

### Security M1
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.13 | MB-180..MB-186 certificate pinning prod, HTTPS-only (ATS + network_security_config), freeRASP report-only (D-050), local_auth + biometria APP-ACC-004/006, FlutterSecureStorage hardening, Hive cipher wire-up, logout clean (Zitadel + secure + Hive + FCM unsubscribe + hydrated_bloc clear) | ⏳ | D-050, P-APP-003 |

### Build & store
| ID | Título | Status | Ref |
|---|---|---|---|
| mobile-10.14 | MB-200..MB-203 flavors dev/staging/prod, Makefile, CI GitHub Actions (subosito/flutter-action + analyze + test + build_runner + APK debug + iOS no-codesign), TestFlight + Play Console Internal upload manual | ⏳ | DT-MB-011 |

## Dependências

- Sprint anterior: assembly adiantada ([[sprint-9]]).
- Cross-stack: backend Sprint 10 entrega Unit `unit_type`+`fracao_ideal` + rate limit per-user + A-023 audit (impactam payloads mobile).
- Decisões: D-048, D-049, D-050 — **aplicar em código** (A-FASE10-DEV-003).

## Entregáveis

- App M1 navegável fim-a-fim: login Zitadel → home com condomínio selecionado → timeline read → plano diretor read → eventos read → conta + trial countdown.
- Build AAB assinado para Google Play Internal + IPA TestFlight.
- Coverage ≥ 80% global.
- Security checklist MASVS-L1 básico OK (D-050 report-only ativo).

## Gates (`<verify>`)

- `fvm flutter analyze` ✅ zero warning
- `fvm flutter test --coverage` ✅ thresholds
- `fvm dart run build_runner build --delete-conflicting-outputs` ✅
- `fvm flutter build apk --release` ✅
- `fvm flutter build ios --no-codesign` ✅
- [[../../tasks#gates-de-merge-dod]] DoD mobile ✅

## Pós-sprint

- **O que deu certo**: (preencher)
- **Bloqueadores**: assembly mobile já codado (Sprint 9) não será removido — fica pronto para M3; decisão de não deletar evita desperdício.
- **Dívidas criadas**: dependentes da Opção A/B/C; Empresa mobile app continua backlog pós-M3.
- **Próxima sprint**: Sprint 11 — M2 (síndico light + Connect Me + vídeo + votação fornecedor) MB-220..MB-363.

## Links

- [[sprint-9]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../AUDIT#a-fase10-dev-003]]
- [[../../../AUDIT#a-fase10-dev-004]]
- [[../../../../ROADMAP]]
- [[../../tasks]]
