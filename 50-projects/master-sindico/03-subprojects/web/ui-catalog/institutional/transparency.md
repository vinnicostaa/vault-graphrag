---
title: Transparency Dashboard — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, institutional, transparency]
bc: institutional
source: _chaos/11-TRANSPARENCY-DASHBOARD.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 11 — Painel de Transparência

> **Origem**: `_chaos/11-TRANSPARENCY-DASHBOARD.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096).

> Sprint 2 · App `cms` · Rotas: `/transparencia`, `/transparencia/editar`, `/transparencia/pdf`
> Exclusivo Síndico `pro` (edição) e Moradores (visualização readonly).

---

## Regras de negócio

- **Plano de Reeleição**: "O que prometi" vs "O que fiz".
- **Percentual de execução** do Plano Diretor por item.
- **Timeline da gestão** (eventos cronológicos — alimentada pelo BC `institutional` Timeline imutável — ADR-0033).
- **Exportar gestão em PDF**.
- **Morador** vê readonly; **Síndico `pro`** edita.

## Layout

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  PAINEL DE TRANSPARÊNCIA      [✏ Editar]      │
│          │                                                │
│          │  PLANO DE GESTÃO                               │
│          │  ┌──────────────────────────────────────────┐ │
│          │  │ 1. Reforma da fachada           [80%]    │ │
│          │  │ ████████████████░░░░                      │ │
│          │  │ 2. Troca dos elevadores         [40%]    │ │
│          │  │ ████████░░░░░░░░░░░░                      │ │
│          │  │ 3. Paisagismo entrada          [100%]    │ │
│          │  │ ████████████████████ ✓                    │ │
│          │  │                                          │ │
│          │  │ Execução geral: ██████████░░ 73%         │ │
│          │  └──────────────────────────────────────────┘ │
│          │                                                │
│          │  TIMELINE DA GESTÃO                            │
│          │  ● Mar/2026 — Início obra fachada             │
│          │  │  Contratada: Alpha • R$45.000               │
│          │  │  [📹 Vídeo] [📄 Edital]                    │
│          │  ● Fev/2026 — Assembleia Ordinária            │
│          │  │  5 pautas • 78% participação                │
│          │  ● Jan/2026 — Manutenção preventiva           │
│          │     Elevadores — inspeção periódica            │
│          │                                                │
│          │  [📄 Exportar Gestão em PDF]                   │
└──────────────────────────────────────────────────────────┘
```

## CSS

```css
.plan-item { margin-bottom: 16px; }
.plan-item-header { display: flex; justify-content: space-between; margin-bottom: 6px; }
.plan-item-title { font-family: 'Manrope'; font-size: 14px; font-weight: 600; }
.plan-item-percent {
  font-family: 'Manrope'; font-size: 14px; font-weight: 700; color: var(--primary);
}
.plan-item-bar {
  height: 8px; background: var(--muted); border-radius: 4px; overflow: hidden;
}
.plan-item-fill {
  height: 100%; background: var(--primary); border-radius: 4px;
  transition: width var(--duration-slower) var(--ease-out);
}
.plan-item-fill.complete { background: var(--success); }

.plan-total {
  margin-top: 24px; padding-top: 16px; border-top: 1px solid var(--border);
}
.plan-total-label { font-family: 'Michroma'; font-size: 14px; color: var(--foreground); }
.plan-total-bar { height: 12px; margin-top: 8px; }

.timeline { position: relative; padding-left: 32px; }
.timeline::before {
  content: ''; position: absolute; left: 5px; top: 0; bottom: 0;
  width: 2px; background: var(--border);
}
.timeline-item { position: relative; margin-bottom: 24px; }
.timeline-dot {
  position: absolute; left: -32px; top: 2px;
  width: 12px; height: 12px; border-radius: 50%;
  background: var(--primary); border: 3px solid var(--card);
  box-shadow: 0 0 0 2px var(--border);
}
.timeline-date {
  font-family: 'Manrope'; font-size: 12px;
  color: var(--muted-foreground); font-weight: 600;
}
.timeline-title {
  font-family: 'Manrope'; font-size: 14px; font-weight: 600;
  color: var(--foreground); margin-top: 2px;
}
.timeline-meta { font-size: 13px; color: var(--muted-foreground); margin-top: 4px; }
.timeline-actions { display: flex; gap: 8px; margin-top: 8px; }

.pdf-export-btn {
  display: inline-flex; align-items: center; gap: 8px;
  padding: 12px 24px; background: var(--secondary); color: var(--secondary-foreground);
  border-radius: var(--radius-md); font-weight: 600; cursor: pointer;
}
```

## Form de edição (Síndico `pro`)

Para cada item do plano: Título, Descrição, Percentual (slider 0–100), Observações.
Botões [+Adicionar atividade], [Remover], reordenar com drag.

> Edição não afeta Timeline imutável (R3 nada deletado / ADR-0033). Histórico de versões preservado.

## Componentes

`TransparencyDashboard`, `PlanItemList`, `PlanItem`, `PlanItemForm`, `PlanTotalBar`, `Timeline`, `TimelineItem`, `PDFExportButton`.

## Links

- [[../../../../00-product/sub-produtos/01-governanca-institucional]]
- [[../../../web/ui-catalog#s27-—-dashboard-síndico]] — S27 dashboard síndico
- [[../../../../00-product/business-model#41-matriz-consolidada]] — gate "Exportar PDF" Síndico `pro`
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
