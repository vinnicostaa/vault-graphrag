---
title: MOC — Mobile Requirements (por feature)
type: moc
tags: [moc, master-sindico, mobile, flutter, requirements]
project: master-sindico
stack: mobile
created: 2026-04-24
---

# MOC — Mobile Requirements

Requirements mobile destilados do `requirements.md` monolítico (legado 20k). Cada feature vira um arquivo focado, com IDs estáveis (`FR-MOB-<feat>-N` / `NFR-MOB-*`) e priorização M1/M2/M3.

> Regra: arquivo de feature = 150-300 linhas. Sempre linkar web equivalente quando aplicável.

---

## M1 (MVP — Sprint 10)

- [[splash]] — launch + pre-check integrity + freeze se `hooked`
- [[auth-oidc]] — login Zitadel OIDC+PKCE, 1-device, refresh, logout
- [[onboarding]] — 4 fluxos (morador/síndico/empresa/parceiro), auto-save, 4-7 telas por persona
- [[home-morador]] — painel 4-6 cards
- [[home-sindico]] — central de governança, 13 cards (subset M1: home + condominio + timeline read)
- [[timeline-read]] — lista LT + detalhes (append-only, read-only)
- [[comunicados]] — read, notificação push

## M2 (Sprint 11)

- [[eventos]] — lista, ciente, confirmar participação, RSVP
- [[avaliar-gestao]] — bimestral 3 critérios 1-10 (M12 equivalente)
- [[connect-me-mobile]] — morador envia, síndico recebe (M→S)

## M3 (Sprint 12)

- [[assembly-check-in]] — check-in ao vivo via app (QR ou numérico)
- [[assembly-vote]] — votação real-time WebSocket, fração ideal aplicada
- [[vizinhanca-mobile]] — feed, ativar promoção (VM1-VM7)

---

## Padrões transversais (referência)

- **BLoC** (D-048) + `bloc_concurrency.droppable()` em botões críticos
- **hydrated_bloc** para memberships/timeline/plano-diretor
- **freezed** (D-049) em entities + states + events
- **freeRASP** (D-050) report-only em M1 → gated em M3
- **go_router** ShellRoute + deep-linking com allowlist
- **flutter_secure_storage** para tokens (nunca SharedPreferences)

---

## Vizinhos

- [[../../../_moc]]
- [[../../web/requirements/_moc]] (espelho web quando aplicável)
- [[../../../STATE#D-048|D-048]] · [[../../../STATE#D-049|D-049]] · [[../../../STATE#D-050|D-050]]
