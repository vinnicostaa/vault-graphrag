---
title: Sprint 14 — Kick-off M3 (Assembleia Live + LMS thin)
type: spec
tags: [tasks, sprint, sprint-14, m3, master-sindico, v2]
source: backlog.md + ROADMAP
created: 2026-04-23
updated: 2026-04-23
---

# Sprint 14 — Kick-off M3

Janela: **2026-06-24 → 2026-07-06** (13 dias).

Meta: **arrancar M3** = Assembleia Live UI + LMS thin + Banco de Talentos + painel presidente, aproveitando backend Sprint 7 e SDK Livekit Sprint 9.

---

## Tasks

### Assembleia Live UI (BC assembly)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-300 | Config + edital PDF auto/upload UI | 2d | web |
| T-301 | Check-in UI (app QR + terminal físico) | 2d | web+mobile |
| T-302 | Votação real-time WebSocket (latência < 200ms) UI | 3d | web+backend |
| T-307 | Painel presidente UI — quórum + fila + controles | 2d | web |

### Livekit fix

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-304 | Fix A-033 — join/ListParticipants retry exponencial + DLQ + PBT | 1d | backend |
| T-305 | Fix A-034 — EndRoom NotFound tolerant + retry | 0.5d | backend |

### LMS thin (BC content)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-250 | LMS estrutura — Course/Module/Lesson + Enrollment | 2d | backend |
| T-250a | Trilha "Síndico Iniciante" — conteúdo seed (vídeo + ebook + quiz) | 1d | content |

### Banco Talentos (BC commercial + content)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-400 | Vídeo-currículo 90s upload + profile profissional (6 perguntas) | 1.5d | mobile+backend |
| T-400a | Busca empresa por skill + CEP + tier (OpenSearch) | 1.5d | web+backend |

---

## Gates de saída

- [ ] AUDIT 🔴 = 0 (A-033, A-034 fixados)
- [ ] Livekit room abre + 10 participantes votando OK
- [ ] LMS trilha iniciante acessível end-to-end
- [ ] Banco Talentos busca funcional

---

## Links

- [[_moc]]
- [[sprint-13]]
- [[sprint-15]]
- [[../backlog#must-m3]]
- [[../../ROADMAP#m3-assembleia--lms--polish]]
