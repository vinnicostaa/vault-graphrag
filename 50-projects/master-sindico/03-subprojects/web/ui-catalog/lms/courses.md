---
title: LMS — Cursos & Treinamentos (UI Spec)
type: ui-spec
tags: [ui-catalog, master-sindico, lms, courses, content]
bc: content
source: _chaos/15-LMS-COURSES.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 15 — Sistema de Cursos e Treinamentos (LMS)

> **Origem**: `_chaos/15-LMS-COURSES.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).

> Sprint 4 · App `lms` (porta 3002) · Rotas: `/cursos`, `/cursos/:id`, `/cursos/:id/aula/:aulaId`, `/cursos/criar` (Empresa), `/meus-cursos`
> Estrutura hierárquica: **Curso → Módulo → Aula**. Player estilo YouTube / Udemy. Certificado ao 100 %.

---

## Regras de negócio

### Estrutura hierárquica
- **Curso**: módulos + descrição + thumbnail + instrutor (empresa).
- **Módulo**: agrupamento de aulas dentro do curso.
- **Aula**: vídeo + descrição + anexo opcional (PDF / ebook).

### Progresso
- `% = (aulas_concluidas / total_aulas) * 100`.
- Aula concluída se ≥ 70 % do vídeo assistido.
- Progresso por módulo e por curso.

### Certificado
- Gerado automaticamente ao 100 %.
- PDF dinâmico: nome do aluno, nome do curso, data de conclusão, QR Code de validação.
- Download disponível em "Meus Cursos".

### Permissões (matriz canônica — ver [[../../../../00-product/business-model#41-matriz-consolidada]])

| Ação | Morador `base` | Síndico `base` | Síndico `plus`/`pro` | Empresa `plus` | Empresa `pro` | Marketing |
|---|---|---|---|---|---|---|
| Acessar cursos | ❌ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Certificado emitido | ❌ | ❌ | ✅ | ✅ | ✅ | — |
| Criar cursos | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ (em nome da empresa) |
| Ebook | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |

> Parceiro Vizinhança (`local_company`): **bloqueado** em LMS (D-062).

---

## Layout — Catálogo de cursos

Rota: `/cursos`.

Estrutura: header + filter pills + seção "Continuar Estudando" (condicional) + categorias + grid "Todos os Cursos".

### CSS — Course Card

```css
.course-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: var(--radius-xl);
  overflow: hidden;
  cursor: pointer;
  transition: all var(--duration-moderate) var(--ease-out);
}
.course-card:hover {
  border-color: var(--primary);
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
}
.course-card-thumb {
  width: 100%; aspect-ratio: 16/9;
  object-fit: cover; background: var(--muted);
}
.course-card-progress-bar {
  position: absolute; bottom: 0; left: 0; right: 0;
  height: 4px; background: oklch(0 0 0 / 0.3);
}
.course-card-progress-fill {
  height: 100%; background: var(--primary);
  transition: width var(--duration-slow);
}
.course-card-body { padding: 14px; }
.course-card-title {
  font-family: 'Manrope'; font-size: 15px; font-weight: 700;
  color: var(--foreground); margin-bottom: 6px; line-height: 1.3;
  display: -webkit-box; -webkit-line-clamp: 2;
  -webkit-box-orient: vertical; overflow: hidden;
}
.course-card-progress-badge {
  display: inline-flex; align-items: center; gap: 4px;
  padding: 2px 8px; border-radius: var(--radius-full);
  font-size: 11px; font-weight: 600;
  background: oklch(0.715 0.120 84.2 / 0.1); color: var(--primary);
  margin-bottom: 8px;
}
.course-card-completed-badge {
  background: oklch(0.627 0.170 149.2 / 0.1); color: var(--success);
}
.course-grid {
  display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px;
}
@media (max-width: 1023px) { .course-grid { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 767px)  { .course-grid { grid-template-columns: 1fr; } }
```

---

## Layout — Detalhe do curso

Rota: `/cursos/:id`.

Banner 16:9 + título + meta (módulos / aulas / duração) + barra de progresso + botão "Continuar" + accordion de módulos + material complementar + bloco certificado.

### CSS — Module Accordion

```css
.module-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 14px 16px;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md);
  cursor: pointer; margin-bottom: 2px;
  transition: all var(--duration-moderate) var(--ease-out);
}
.module-header:hover { border-color: var(--primary); }
.module-header[data-expanded] {
  border-radius: var(--radius-md) var(--radius-md) 0 0;
  border-bottom-color: transparent;
}
.module-status-badge {
  padding: 2px 8px; border-radius: var(--radius-full);
  font-size: 11px; font-weight: 600;
}
.module-status-badge.completed   { background: oklch(0.627 0.170 149.2 / 0.1); color: var(--success); }
.module-status-badge.in-progress { background: oklch(0.715 0.120 84.2 / 0.1); color: var(--primary); }
.module-status-badge.locked      { background: var(--muted); color: var(--muted-foreground); }

.lesson-item {
  display: flex; align-items: center; gap: 12px;
  padding: 12px 16px;
  border-bottom: 1px solid var(--border);
  cursor: pointer; transition: all var(--duration-quick);
}
.lesson-item:hover { background: oklch(0.715 0.120 84.2 / 0.03); }
.lesson-status-icon.completed { color: var(--success); }
.lesson-status-icon.current   { color: var(--primary); }
.lesson-status-icon.pending   { color: var(--muted-foreground); }
```

---

## Layout — Player de aula

Rota: `/cursos/:id/aula/:aulaId`.

Layout estilo YouTube / Udemy: vídeo à esquerda, sidebar com índice à direita.

```css
.lesson-layout {
  display: grid; grid-template-columns: 1fr 320px; gap: 0;
  height: calc(100vh - 64px);
}
@media (max-width: 1023px) {
  .lesson-layout { grid-template-columns: 1fr; height: auto; }
}
.lesson-video-container {
  width: 100%; aspect-ratio: 16/9; max-height: 70vh;
  background: #000; position: relative;
}
.lesson-sidebar {
  border-left: 1px solid var(--border);
  overflow-y: auto; background: var(--card);
}
.lesson-sidebar .lesson-item.active {
  background: oklch(0.715 0.120 84.2 / 0.08);
  border-left: 3px solid var(--primary);
}
```

Mobile: vídeo full-width topo, sidebar de conteúdo vira accordion abaixo.

---

## Wizard de criação (Empresa `pro` / Marketing)

Rota: `/cursos/criar`. 4 steps similares ao wizard de assembleias:

```
[1. Info do Curso] → [2. Módulos] → [3. Aulas/Upload] → [4. Revisão]
```

- **Step 1**: Título, Descrição, Categoria (31 categorias), Thumbnail.
- **Step 2**: Adicionar módulos com drag & drop.
- **Step 3**: Aulas por módulo (vídeo via Mux upload + descrição + anexo PDF).
- **Step 4**: Preview + [Publicar] / [Salvar Rascunho].

```css
.module-drag-handle {
  width: 24px; height: 24px;
  color: var(--muted-foreground); cursor: grab;
}
.module-drag-handle:active { cursor: grabbing; }
.module-editable-item {
  display: flex; align-items: center; gap: 12px;
  padding: 14px 16px;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md); margin-bottom: 8px;
}
```

---

## Certificado (100 %)

```
┌───────────────────────────────────────────┐
│  🏆 CERTIFICADO DISPONÍVEL               │
│                                           │
│  ┌─────────────────────────────────────┐  │
│  │   MASTER SÍNDICO                    │  │
│  │   Certificado de Conclusão          │  │
│  │                                     │  │
│  │   João Silva                        │  │
│  │   concluiu com êxito o curso        │  │
│  │   "Gestão de Contratos"             │  │
│  │                                     │  │
│  │   Data: 19/02/2026                  │  │
│  │   [QR Code]                         │  │
│  └─────────────────────────────────────┘  │
│                                           │
│  [📥 Baixar PDF]                         │
└───────────────────────────────────────────┘
```

```css
.certificate-card {
  background: oklch(0.715 0.120 84.2 / 0.05);
  border: 2px solid var(--primary);
  border-radius: var(--radius-2xl);
  padding: 24px; text-align: center;
}
.certificate-preview-name {
  font-family: 'Michroma'; font-size: 20px;
  color: var(--foreground); margin-bottom: 8px;
}
```

## Componentes

`CourseCard`, `CourseContinueCard`, `CourseDetail`, `ModuleAccordion`, `LessonItem`, `LessonPlayer` (compõe [[../content/video-player|video-player]]), `LessonSidebar`, `CourseProgress`, `CertificateCard`, `CourseWizard`, `ModuleDragList`, `CategoryPills`.

## Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Links

- [[../../../web/ui-catalog#c-app-lms-lms1-lms15]] — telas LMS1-LMS15
- [[../content/video-player|content/video-player]] — player base
- [[../content/video-upload|content/video-upload]] — upload de aulas
- [[../../../../00-product/business-model#41-matriz-consolidada]] — matriz LMS
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
