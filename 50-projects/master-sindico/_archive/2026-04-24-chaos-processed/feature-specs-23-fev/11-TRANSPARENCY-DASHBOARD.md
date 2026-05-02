# 11 — PAINEL DE TRANSPARÊNCIA

> Sprint 2 · Rotas: /transparencia, /transparencia/editar, /transparencia/pdf
> Exclusivo Síndico N3 (edição) e Moradores (visualização readonly)

---

## REGRAS DE NEGÓCIO

- Plano de Reeleição: "O que prometi" vs "O que fiz"
- Percentual de execução do Plano Diretor por item
- Timeline da gestão (eventos cronológicos)
- Exportar gestão em PDF
- Morador vê readonly. Síndico N3 edita.

## LAYOUT

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
.plan-item-percent { font-family: 'Manrope'; font-size: 14px; font-weight: 700; color: var(--primary); }
.plan-item-bar { height: 8px; background: var(--muted); border-radius: 4px; overflow: hidden; }
.plan-item-fill { height: 100%; background: var(--primary); border-radius: 4px; transition: width 500ms ease; }
.plan-item-fill.complete { background: var(--success); }

.plan-total { margin-top: 24px; padding-top: 16px; border-top: 1px solid var(--border); }
.plan-total-label { font-family: 'Michroma'; font-size: 14px; color: var(--foreground); }
.plan-total-bar { height: 12px; margin-top: 8px; }

.timeline { position: relative; padding-left: 32px; }
.timeline::before { content: ''; position: absolute; left: 5px; top: 0; bottom: 0; width: 2px; background: var(--border); }
.timeline-item { position: relative; margin-bottom: 24px; }
.timeline-dot {
  position: absolute; left: -32px; top: 2px;
  width: 12px; height: 12px; border-radius: 50%;
  background: var(--primary); border: 3px solid var(--card);
  box-shadow: 0 0 0 2px var(--border);
}
.timeline-date { font-family: 'Manrope'; font-size: 12px; color: var(--muted-foreground); font-weight: 600; }
.timeline-title { font-family: 'Manrope'; font-size: 14px; font-weight: 600; color: var(--foreground); margin-top: 2px; }
.timeline-meta { font-size: 13px; color: var(--muted-foreground); margin-top: 4px; }
.timeline-actions { display: flex; gap: 8px; margin-top: 8px; }

.pdf-export-btn {
  display: inline-flex; align-items: center; gap: 8px;
  padding: 12px 24px; background: var(--secondary); color: var(--secondary-foreground);
  border-radius: var(--radius); font-weight: 600; cursor: pointer;
}
```

## FORM DE EDIÇÃO (Síndico N3)

Para cada item do plano: Título, Descrição, Percentual (slider 0-100), Observações
Botão [+Adicionar atividade], [Remover], reordenar com drag.

## COMPONENTES

TransparencyDashboard, PlanItemList, PlanItem, PlanItemForm, PlanTotalBar, Timeline, TimelineItem, PDFExportButton
