---
title: Tasks — BC assembly
type: spec
tags: [tasks, by-module, assembly, master-sindico, v2]
source: 04-requirements/functional/assembly.md + AUDIT A-033/A-034 + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Tasks — BC `assembly`

Tasks do bounded context **assembly** (Assembleias, edital, pauta, votação real-time, quórum fracional, procuração, ata imutável, live Livekit).

Alinhamento: [[../../04-requirements/functional/assembly]] (ASM-001..ASM-015).

---

## M3 — Kick-off + polish + deploy

### Kick-off (Sprint 14)

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-300 | Config + edital PDF auto/upload UI | 14 | ⏳ |
| T-301 | Check-in UI (app QR + terminal físico) | 14 | ⏳ |
| T-302 | Votação real-time WebSocket (< 200ms) UI | 14 | ⏳ |
| T-307 | Painel presidente UI — quórum + fila + controles | 14 | ⏳ |
| T-304 | Fix A-033 — Livekit join/List retry + DLQ + PBT | 14 | ⏳ |
| T-305 | Fix A-034 — Livekit EndRoom NotFound tolerant + retry | 14 | ⏳ |

### Polish (Sprint 15)

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-303 | Procuração (proxy) UI | 15 | ⏳ |
| T-306 | Ata automática + homologação UI | 15 | ⏳ |
| T-310 | Simulador quórum pré-assembleia | 15 | ⏳ |
| T-311 | Enquetes preliminares consultivas | 15 | ⏳ |

### Relatórios + a11y (Sprint 16)

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-312 | 6 relatórios (presença, votação, procurações, fala, consolidado, transparência) CSV+PDF | 16 | ⏳ |
| T-313 | Fila fala cronometrada UI + alertas 30s | 16 | ⏳ |
| T-314 | Score transparência (10 componentes) dashboard | 16 | ⏳ |
| T-308 | Telão 2 áreas (Presentation + Operational) thin | 16 | ⏳ |

---

## Invariantes cobertos

I-039..I-050 de [[../../01-domain/bounded-contexts#assembly]]:
- I-039 Vote único por (agenda_item, voter) — UNIQUE DB
- I-040 Minutes imutável pós-publish
- I-041 Quórum por fração ideal, não por pessoa
- I-042 Assembly não inicia sem edital publicado
- I-043 ≤ 1 assembleia ativa por condomínio
- I-044 Vote após timeout → `absent`
- I-045 Procuração: proxy não vota diferente do titular
- I-046 Pauta `published_at` → read-only
- I-047 Síndico presidente não vota (imparcialidade)
- I-048 Live room órfã evitada por Saga
- I-049 Fração ideal NUMERIC(10,6), nunca float
- I-050 Mandate `end_date > start_date`

---

## Links

- [[_moc]]
- [[../../04-requirements/functional/assembly]]
- [[../../01-domain/bounded-contexts#assembly]]
- [[../backlog]]
