---
title: MOC — 03-subprojects
type: moc
tags: [moc, master-sindico, v2, subprojects]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 03-subprojects

Três sub-projetos com stacks dedicadas; paridade de domínio (mesmos 6 BCs + cross-domain) e respeito às mesmas regras globais ([[../STEERING]], [[../CLAUDE]]).

- **Backend** — Go 1.26 + Gin abstraído (`contracts.HTTPRouter`) + pgx/v5 + sqlc + goose + zerolog + Redis 7 + PostgreSQL 18. Clean Arch estrita + DDD + CQRS. Único implementador de domínio + regras. Provider externo abstraído (Stripe, Mux, Livekit, Zitadel, SES, Twilio, FCM). Ver [[backend/README]].
- **Web** — SolidJS + TS + Vite + Tailwind. Feature-sliced (1 feature por BC). Consome API backend via cookie httpOnly `.mastersindico.com.br`. WCAG 2.1 AA. Never trust the frontend — backend é fonte de verdade. Ver [[web/README]].
- **Mobile** — Flutter + Dart + Clean Arch + Feature First. Consome backend via Bearer em `flutter_secure_storage`. OIDC PKCE via browser externo. Cert pinning prod. Offline-first seletivo. Ver [[mobile/README]].

---

## Estrutura canônica por sub-projeto

Cada sub-projeto mantém os mesmos artefatos:

- `README.md` — stack + estrutura + contratos externos.
- `architecture.md` — detalhamento arquitetural específico da stack.
- `patterns.md` — idiomática + patterns aplicados.
- `requirements.md` — requirements específicos além dos canônicos.
- `security.md` — security posture da stack.
- `tasks.md` — seed de tasks por sprint M1-M3.

Web e mobile têm dois artefatos adicionais:

- `ui-catalog.md` — catálogo exaustivo de telas.
- `design-system.md` (web only) — tokens, cores Manual IV, tipografia, componentes.

---

## Sub-MOCs

- [[backend/_moc]]
- [[web/_moc]]
- [[mobile/_moc]]

---

## Marcos por sub-projeto

| Sub-projeto | M1 (2026-05-08) | M2 (2026-06-20) | M3 (2026-07-20) |
|---|---|---|---|
| Backend | identity + billing + institutional base + cross-domain (ABAC/Audit/Outbox) | commercial (Connect Me + Reputação + Vizinhança) + content (Mux + Moderação + Lock 90d) | assembly (live + votação + ata) + LMS thin |
| Web | Onboarding morador + síndico + central + timeline + eventos + conta | Connect Me 4 fluxos + empresa dashboard + vídeo player + reputação + compliance | Assembleia live + LMS + admin panel + relatórios + LP |
| Mobile | Splash + auth OIDC + home morador + timeline read + eventos + onboarding + conta | Síndico light (voice-first) + Connect Me morador + vídeo-currículo + votação player | Assembleia live (Livekit SDK Flutter) + LMS thin + store submission |

Detalhamento em [[backend/tasks]], [[web/tasks]], [[mobile/tasks]].

---

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STEERING]] — não-negociáveis
- [[../STATE]] — decisões vivas (D-###)
- [[../ROADMAP]] — marcos
- [[../01-domain/bounded-contexts]] — 6 BCs mapeados
- [[../02-architecture/clean-arch-mapping]] — camadas
- [[../04-requirements/_moc]] — requirements canônicos
- [[../08-security/BEYOND_CORP]] — security posture
