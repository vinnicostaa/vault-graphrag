---
title: Sprint 15 — M3 Polish (Procuração + Ata + Simulador)
type: spec
tags: [tasks, sprint, sprint-15, m3, master-sindico, v2]
source: backlog.md + ROADMAP
created: 2026-04-23
updated: 2026-04-23
---

# Sprint 15 — M3 Polish

Janela: **2026-07-01 → 2026-07-13** (13 dias).

Meta: fechar features M3 restantes — procuração, ata imutável, simulador quórum, enquetes pré-assembleia, compliance UI.

---

## Tasks

### Assembleia — restantes (BC assembly)

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-303 | Procuração (proxy) UI — morador outorga; proxy vota | 2d | web |
| T-306 | Ata automática + homologação UI | 2d | web+backend |
| T-310 | Simulador quórum pré-assembleia | 1d | web |
| T-311 | Enquetes preliminares consultivas | 1d | web |

### Institutional — restantes M3

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-050 | Event condomínio 13 tipos UI | 1.5d | web |
| T-051 | Compliance anual UI + bloqueio exportação se pendente | 1d | web |

### LMS + Banco Talentos polish

| ID | Título | Estimate | Dono |
|---|---|---|---|
| T-251 | Ebooks biblioteca base (PDF S3 signed URL) | 1d | backend |
| T-260 | LMS — outras 2 trilhas (Intermediário, Profissional) seed | 1d | content |
| T-261 | Certificados auto ao 100% + URL pública verificável | 1d | backend |

---

## Gates de saída

- [ ] AUDIT 🔴 = 0
- [ ] Ata imutável pós-publish (tentativa UPDATE bloqueada DB+app)
- [ ] Procuração E2E (outorga → proxy vota → resultado replicado)
- [ ] Simulador quórum matemática correta (PBT)

---

## Links

- [[_moc]]
- [[sprint-14]]
- [[sprint-16]]
- [[../backlog]]
