# 15 — SISTEMA DE CURSOS E TREINAMENTOS (LMS)

> Sprint 4 · Rotas: /cursos, /cursos/:id, /cursos/:id/aula/:aulaId, /cursos/criar (Empresa), /meus-cursos
> Estrutura hierárquica: Curso → Módulo → Aula. Player estilo YouTube/Udemy. Certificado ao 100%.

---

## REGRAS DE NEGÓCIO

### Estrutura Hierárquica
- **Curso**: Contém módulos + descrição + thumbnail + instrutor (empresa)
- **Módulo**: Agrupamento de aulas dentro do curso
- **Aula**: Vídeo + Descrição + Anexo opcional (PDF/ebook)

### Progresso
- Progresso calculado: `% = (aulas_concluidas / total_aulas) * 100`
- Aula contabilizada como concluída se >70% do vídeo foi assistido
- Progresso por módulo e por curso
- Barra visual em todos os cards

### Certificado
- Gerado automaticamente ao atingir 100% do curso
- PDF dinâmico com: nome do aluno, nome do curso, data de conclusão, QR Code de validação
- Download disponível na área "Meus Cursos"

### Permissões (Matriz Funcional)
| Ação | Base | Morador Pg | Síndico N1 | Síndico N2/N3 | Emp. Plus | Emp. Pro | Marketing |
|------|------|-----------|-----------|---------------|-----------|---------|-----------|
| Acessar cursos | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| Módulos/cursos certificados | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ (criar) | ✅ (criar) |
| Criar cursos | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Ebook | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ✅ |

### Quem cria cursos
- Empresa Pro e Marketing criam cursos
- Síndico N2/N3 consome
- Empresa Plus NÃO cria cursos

---

## LAYOUT — CATÁLOGO DE CURSOS

Rota: `/cursos`

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  CURSOS E TREINAMENTOS             🔍 [Buscar...]     │
│          │  h2 Michroma 20px                                      │
│          │                                                        │
│          │  [Todos] [Em Andamento] [Concluídos] [Novos]          │
│          │   ← pills/tabs                                         │
│          │                                                        │
│          │  ─── CONTINUAR ESTUDANDO ──────────────────────────   │
│          │  ← seção só aparece se tem cursos em andamento         │
│          │                                                        │
│          │  ┌─────────────────────────────┐                       │
│          │  │ [thumbnail 16:9          ]  │                       │
│          │  │ ████████████░░░░ 72%        │                       │
│          │  │                             │                       │
│          │  │ Gestão de Contratos         │                       │
│          │  │ 📚 8 módulos • 24 aulas     │                       │
│          │  │ 🏢 JurisCondo Consultoria   │                       │
│          │  │                             │                       │
│          │  │ [Continuar →]              │                       │
│          │  └─────────────────────────────┘                       │
│          │                                                        │
│          │  ─── CATEGORIAS ───────────────────────────────────   │
│          │  [📋Gestão] [⚖️Jurídico] [🔧Manutenção] [💰Finanças] │
│          │  [🏗️Engenharia] [🔒Segurança] [🌱ESG] [📢Oratória]   │
│          │                                                        │
│          │  ─── TODOS OS CURSOS ──────────────────────────────   │
│          │                                                        │
│          │  ┌────────────┐ ┌────────────┐ ┌────────────┐        │
│          │  │ [thumb]    │ │ [thumb]    │ │ [thumb]    │        │
│          │  │ Prevenção  │ │ AVCB na    │ │ Elevadores │        │
│          │  │ Incêndio   │ │ Prática    │ │ Modernos   │        │
│          │  │            │ │            │ │            │        │
│          │  │ 4 módulos  │ │ 3 módulos  │ │ 6 módulos  │        │
│          │  │ 12 aulas   │ │ 9 aulas    │ │ 18 aulas   │        │
│          │  │            │ │            │ │            │        │
│          │  │ 🏢 SafePro │ │ 🏢 CorpFire│ │ 🏢 ElevTech│        │
│          │  │            │ │            │ │            │        │
│          │  │ [Acessar →]│ │ [Acessar →]│ │ [Acessar →]│        │
│          │  └────────────┘ └────────────┘ └────────────┘        │
│          │                                                        │
│          │  [Carregar mais...]                                    │
└──────────────────────────────────────────────────────────────────┘
```

### CSS — Course Card

```css
.course-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 200ms;
}
.course-card:hover {
  border-color: var(--primary);
  box-shadow: 0 4px 12px oklch(0 0 0 / 0.06);
  transform: translateY(-2px);
}

.course-card-thumb {
  width: 100%;
  aspect-ratio: 16/9;
  object-fit: cover;
  background: var(--muted);
}

/* Progress bar sobreposta na thumbnail (se curso em andamento) */
.course-card-progress-overlay {
  position: relative;
}
.course-card-progress-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: oklch(0 0 0 / 0.3);
}
.course-card-progress-fill {
  height: 100%;
  background: var(--primary);
  transition: width 300ms;
}

.course-card-body { padding: 14px; }

.course-card-title {
  font-family: 'Manrope';
  font-size: 15px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 6px;
  line-height: 1.3;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.course-card-meta {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  margin-bottom: 8px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.course-card-company {
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
}

.course-card-company-icon {
  font-size: 14px;
}

/* Badge de progresso (quando em andamento) */
.course-card-progress-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: 9999px;
  font-size: 11px;
  font-weight: 600;
  background: oklch(0.715 0.120 84.2 / 0.1);
  color: var(--primary);
  margin-bottom: 8px;
}

/* Badge de concluído */
.course-card-completed-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: 9999px;
  font-size: 11px;
  font-weight: 600;
  background: oklch(0.627 0.170 149.2 / 0.1);
  color: var(--success);
}

/* Grid de cursos */
.course-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

@media (max-width: 1023px) {
  .course-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (max-width: 767px) {
  .course-grid { grid-template-columns: 1fr; }
}
```

### CSS — Continuar Estudando (card destaque)

```css
.course-continue-card {
  background: var(--card);
  border: 1px solid var(--primary);
  border-radius: 12px;
  overflow: hidden;
  display: flex;
  gap: 0;
  margin-bottom: 28px;
}

@media (max-width: 767px) {
  .course-continue-card { flex-direction: column; }
}

.course-continue-thumb {
  width: 260px;
  min-height: 160px;
  object-fit: cover;
  background: var(--muted);
  position: relative;
}

@media (max-width: 767px) {
  .course-continue-thumb {
    width: 100%;
    height: 180px;
  }
}

.course-continue-body {
  flex: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.course-continue-progress {
  margin-bottom: 12px;
}
.course-continue-progress-bar {
  height: 6px;
  background: var(--muted);
  border-radius: 3px;
  margin-bottom: 4px;
}
.course-continue-progress-fill {
  height: 100%;
  background: var(--primary);
  border-radius: 3px;
}
.course-continue-progress-text {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--primary);
  font-weight: 600;
}

.course-continue-title {
  font-family: 'Manrope';
  font-size: 18px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 6px;
}

.course-continue-btn {
  margin-top: 12px;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 10px 20px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  width: fit-content;
  transition: all 200ms;
}
.course-continue-btn:hover { background: oklch(0.65 0.120 84.2); }
```

---

## LAYOUT — DETALHE DO CURSO

Rota: `/cursos/:id`

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  ┌─────────────────────────────────────────────────┐   │
│          │  │                                                 │   │
│          │  │         [BANNER DO CURSO 16:9]                  │   │
│          │  │              max-height: 320px                   │   │
│          │  │                                                 │   │
│          │  └─────────────────────────────────────────────────┘   │
│          │                                                        │
│          │  Gestão de Contratos Condominiais                      │
│          │  h1 Michroma 24px                                      │
│          │                                                        │
│          │  🏢 JurisCondo Consultoria                             │
│          │  📚 8 módulos • 24 aulas • ~4h30min                    │
│          │  📂 Jurídico e Direito Imobiliário                     │
│          │                                                        │
│          │  ████████████░░░░ 72% concluído                        │
│          │  [Continuar de onde parei →]                           │
│          │                                                        │
│          │  ─── SOBRE O CURSO ─────────────────────────────────  │
│          │  Este curso aborda os principais aspectos jurídicos    │
│          │  da gestão de contratos em condomínios, incluindo...   │
│          │  [Ler mais ▼]                                          │
│          │                                                        │
│          │  ─── CONTEÚDO DO CURSO ─────────────────────────────  │
│          │                                                        │
│          │  ▼ Módulo 1: Fundamentos (4 aulas)     ✅ Concluído   │
│          │  │  ✅ 1.1 Introdução ao Direito...   12:30  ▶       │
│          │  │  ✅ 1.2 Tipos de Contrato           08:45  ▶       │
│          │  │  ✅ 1.3 Cláusulas Essenciais        15:20  ▶       │
│          │  │  ✅ 1.4 Erros Comuns                 10:00  ▶       │
│          │  │                                                     │
│          │  ▼ Módulo 2: Análise Prática (3 aulas)  ████░░ 66%   │
│          │  │  ✅ 2.1 Caso: Reforma Fachada        18:30  ▶       │
│          │  │  ✅ 2.2 Caso: Portaria 24h           14:15  ▶       │
│          │  │  ⬜ 2.3 Caso: Manutenção Elev.       16:00  ▶       │
│          │  │                                                     │
│          │  ▶ Módulo 3: Negociação (3 aulas)       ⬜ Bloqueado  │
│          │  ▶ Módulo 4: ... (collapsed)                           │
│          │                                                        │
│          │  ─── MATERIAL COMPLEMENTAR ─────────────────────────  │
│          │  📄 Ebook: Guia de Contratos (PDF)     [📥 Download]  │
│          │                                                        │
│          │  ─── CERTIFICADO ───────────────────────────────────  │
│          │  🏆 Complete 100% do curso para gerar seu certificado │
│          │  ████████████░░░░ 72%                                  │
│          │  ou:                                                    │
│          │  🏆 Parabéns! Certificado disponível.                  │
│          │  [📥 Baixar Certificado PDF]                           │
└──────────────────────────────────────────────────────────────────┘
```

### CSS — Course Detail

```css
.course-detail { max-width: 900px; margin: 0 auto; }

.course-detail-banner {
  width: 100%;
  max-height: 320px;
  object-fit: cover;
  border-radius: 16px;
  background: var(--muted);
  margin-bottom: 24px;
}

@media (max-width: 767px) {
  .course-detail-banner {
    max-height: 200px;
    border-radius: 0;
  }
}

.course-detail-title {
  font-family: 'Michroma';
  font-size: 24px;
  color: var(--foreground);
  letter-spacing: 0.05em;
  margin-bottom: 12px;
  line-height: 1.3;
}

@media (max-width: 767px) {
  .course-detail-title { font-size: 20px; }
}

.course-detail-instructor {
  display: flex;
  align-items: center;
  gap: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--muted-foreground);
  margin-bottom: 6px;
}

.course-detail-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--muted-foreground);
  margin-bottom: 20px;
}

/* Barra de progresso do curso */
.course-progress-section { margin-bottom: 24px; }

.course-progress-bar {
  height: 8px;
  background: var(--muted);
  border-radius: 4px;
  margin-bottom: 6px;
}
.course-progress-fill {
  height: 100%;
  background: var(--primary);
  border-radius: 4px;
  transition: width 500ms ease;
}
.course-progress-text {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--primary);
  font-weight: 600;
}

/* Seções */
.course-section {
  margin-bottom: 28px;
}

.course-section-title {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--muted-foreground);
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border);
}

.course-about-text {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  line-height: 1.7;
}

.course-read-more {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--primary);
  font-weight: 600;
  cursor: pointer;
  background: none;
  border: none;
  margin-top: 4px;
}
```

### CSS — Módulos (Accordion)

```css
/* Módulo header (collapsible) */
.module-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 10px;
  cursor: pointer;
  margin-bottom: 2px;
  transition: all 200ms;
}
.module-header:hover { border-color: var(--primary); }
.module-header[data-expanded] {
  border-radius: 10px 10px 0 0;
  border-bottom-color: transparent;
}

.module-header-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.module-chevron {
  width: 18px;
  height: 18px;
  color: var(--muted-foreground);
  transition: transform 200ms;
}
.module-header[data-expanded] .module-chevron {
  transform: rotate(90deg);
}

.module-title {
  font-family: 'Manrope';
  font-size: 15px;
  font-weight: 600;
  color: var(--foreground);
}

.module-meta {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
}

/* Módulo badge de status */
.module-status-badge {
  padding: 2px 8px;
  border-radius: 9999px;
  font-size: 11px;
  font-weight: 600;
}
.module-status-badge.completed {
  background: oklch(0.627 0.170 149.2 / 0.1);
  color: var(--success);
}
.module-status-badge.in-progress {
  background: oklch(0.715 0.120 84.2 / 0.1);
  color: var(--primary);
}
.module-status-badge.locked {
  background: var(--muted);
  color: var(--muted-foreground);
}

/* Aula item (dentro do módulo expandido) */
.module-content {
  border: 1px solid var(--border);
  border-top: none;
  border-radius: 0 0 10px 10px;
  overflow: hidden;
  margin-bottom: 8px;
}

.lesson-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  border-bottom: 1px solid var(--border);
  cursor: pointer;
  transition: all 150ms;
}
.lesson-item:last-child { border-bottom: none; }
.lesson-item:hover { background: oklch(0.715 0.120 84.2 / 0.03); }

.lesson-status-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}
.lesson-status-icon.completed { color: var(--success); }
.lesson-status-icon.current { color: var(--primary); }
.lesson-status-icon.pending { color: var(--muted-foreground); }

.lesson-title {
  flex: 1;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
}
.lesson-item.completed .lesson-title { color: var(--muted-foreground); }

.lesson-duration {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
  flex-shrink: 0;
}

.lesson-play-icon {
  width: 16px;
  height: 16px;
  color: var(--primary);
  flex-shrink: 0;
  opacity: 0;
  transition: opacity 150ms;
}
.lesson-item:hover .lesson-play-icon { opacity: 1; }
```

---

## LAYOUT — PLAYER DE AULA

Rota: `/cursos/:id/aula/:aulaId`

Layout estilo YouTube/Udemy: vídeo à esquerda, sidebar com índice à direita.

```
┌────────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                          │
│          │  ┌──────────────────────────────────┐ ┌──────────────┐  │
│          │  │                                  │ │ CONTEÚDO     │  │
│          │  │                                  │ │              │  │
│          │  │        [VIDEO PLAYER]             │ │ ▼ Módulo 1  │  │
│          │  │         16:9 ratio                │ │  ✅ 1.1 Int │  │
│          │  │         max-h: 70vh               │ │  ✅ 1.2 Tip │  │
│          │  │                                  │ │ ▶ 1.3 Cla   │  │
│          │  │   [controls: play/pause etc]      │ │  ⬜ 1.4 Err │  │
│          │  │   [gold progress bar]             │ │              │  │
│          │  │                                  │ │ ▼ Módulo 2  │  │
│          │  └──────────────────────────────────┘ │  ⬜ 2.1 ...  │  │
│          │                                       │  ⬜ 2.2 ...  │  │
│          │  Cláusulas Essenciais em Contratos    │              │  │
│          │  h2 Manrope 20px bold                 │ scroll       │  │
│          │                                       │ sticky 320px │  │
│          │  🏢 JurisCondo Consultoria            │              │  │
│          │  Módulo 1, Aula 3 de 4               │              │  │
│          │                                       │              │  │
│          │  ── Descrição ──                      │              │  │
│          │  Nesta aula, abordamos as cláusulas   │              │  │
│          │  essenciais que todo contrato...       │              │  │
│          │                                       │              │  │
│          │  ── Anexos ──                          │              │  │
│          │  📄 Modelo de Contrato (PDF) [📥]     │              │  │
│          │                                       └──────────────┘  │
│          │  [← Aula Anterior]    [Próxima Aula →]                  │
└────────────────────────────────────────────────────────────────────┘
```

### Mobile: vídeo full-width topo, sidebar de conteúdo vira accordion abaixo

```
┌──────────────────────────────┐
│  [← Voltar]  Aula 3/4       │
│                              │
│  ┌──────────────────────────┐│
│  │                          ││
│  │    [VIDEO PLAYER]        ││
│  │    full width            ││
│  │                          ││
│  └──────────────────────────┘│
│                              │
│  Cláusulas Essenciais        │
│  Módulo 1 • JurisCondo       │
│                              │
│  [▼ Conteúdo do Curso]       │
│   ← accordion collapsed     │
│                              │
│  Descrição da aula...        │
│                              │
│  [← Anterior] [Próxima →]   │
└──────────────────────────────┘
```

### CSS — Player Layout

```css
.lesson-layout {
  display: grid;
  grid-template-columns: 1fr 320px;
  gap: 0;
  height: calc(100vh - 64px); /* desconta topbar */
}

@media (max-width: 1023px) {
  .lesson-layout {
    grid-template-columns: 1fr;
    height: auto;
  }
}

.lesson-main {
  overflow-y: auto;
  padding-bottom: 40px;
}

.lesson-video-container {
  width: 100%;
  aspect-ratio: 16/9;
  max-height: 70vh;
  background: #000;
  position: relative;
}

@media (max-width: 1023px) {
  .lesson-video-container {
    max-height: none;
  }
}

.lesson-info {
  padding: 20px 24px;
}

@media (max-width: 767px) {
  .lesson-info { padding: 16px; }
}

.lesson-title-text {
  font-family: 'Manrope';
  font-size: 20px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 8px;
  line-height: 1.3;
}

.lesson-breadcrumb {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  margin-bottom: 16px;
}

.lesson-description {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  line-height: 1.7;
  margin-bottom: 20px;
}

.lesson-attachments {
  margin-bottom: 24px;
}

.lesson-attachment-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  background: var(--muted);
  border-radius: 8px;
  margin-bottom: 8px;
}

.lesson-attachment-icon { font-size: 18px; }

.lesson-attachment-name {
  flex: 1;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
}

.lesson-attachment-download {
  padding: 4px 10px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 6px;
  font-family: 'Manrope';
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
}

/* Navegação entre aulas */
.lesson-nav {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  padding-top: 16px;
  border-top: 1px solid var(--border);
}

.lesson-nav-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 16px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  cursor: pointer;
  transition: all 200ms;
}
.lesson-nav-btn:hover {
  border-color: var(--primary);
  color: var(--primary);
}
.lesson-nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
```

### CSS — Sidebar de Conteúdo (sticky)

```css
.lesson-sidebar {
  border-left: 1px solid var(--border);
  overflow-y: auto;
  background: var(--card);
}

@media (max-width: 1023px) {
  .lesson-sidebar {
    border-left: none;
    border-top: 1px solid var(--border);
  }
}

.lesson-sidebar-header {
  padding: 16px;
  border-bottom: 1px solid var(--border);
  font-family: 'Michroma';
  font-size: 13px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--muted-foreground);
  position: sticky;
  top: 0;
  background: var(--card);
  z-index: 1;
}

/* Reutiliza module-header e lesson-item do detalhe do curso */
.lesson-sidebar .lesson-item.active {
  background: oklch(0.715 0.120 84.2 / 0.08);
  border-left: 3px solid var(--primary);
}
.lesson-sidebar .lesson-item.active .lesson-title {
  color: var(--primary);
  font-weight: 600;
}
```

---

## LAYOUT — WIZARD DE CRIAÇÃO (Empresa Pro / Marketing)

Rota: `/cursos/criar`

Similar ao wizard de assembleias (4 steps):

```
[1. Info do Curso] ─── [2. Módulos] ─── [3. Aulas/Upload] ─── [4. Revisão]
      ●═══════════════○═══════════════○═══════════════○
```

### Step 1 — Info do Curso
- Título, Descrição, Categoria (select das 31 categorias), Thumbnail (upload)

### Step 2 — Módulos
- Adicionar módulos (título, ordem)
- Drag & drop para reordenar

### Step 3 — Aulas por Módulo
- Para cada módulo: adicionar aulas (título, vídeo upload, descrição, anexo PDF)
- Upload de vídeo com progress bar

### Step 4 — Revisão e Publicar
- Preview do curso completo
- [Publicar Curso →] ou [Salvar como Rascunho]

```css
/* Reutiliza wizard-steps do doc 09 (Assembleias) */
/* Adiciona drag handle para módulos */

.module-drag-handle {
  width: 24px;
  height: 24px;
  color: var(--muted-foreground);
  cursor: grab;
}
.module-drag-handle:active { cursor: grabbing; }

.module-editable-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 16px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 10px;
  margin-bottom: 8px;
}
.module-editable-item:hover { border-color: var(--primary); }

.module-editable-title {
  flex: 1;
  font-family: 'Manrope';
  font-size: 15px;
  font-weight: 500;
  color: var(--foreground);
}

.module-editable-actions {
  display: flex;
  gap: 8px;
}

.module-editable-btn {
  padding: 4px 8px;
  border: none;
  background: transparent;
  color: var(--muted-foreground);
  cursor: pointer;
  border-radius: 4px;
}
.module-editable-btn:hover { color: var(--foreground); background: var(--muted); }
.module-editable-btn.delete:hover { color: var(--destructive); }
```

---

## CERTIFICADO (100%)

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
  border-radius: 16px;
  padding: 24px;
  text-align: center;
}

.certificate-icon {
  font-size: 40px;
  margin-bottom: 8px;
}

.certificate-title {
  font-family: 'Michroma';
  font-size: 16px;
  color: var(--primary);
  letter-spacing: 0.05em;
  margin-bottom: 16px;
}

.certificate-preview {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 24px;
  margin-bottom: 16px;
}

.certificate-preview-brand {
  font-family: 'Michroma';
  font-size: 12px;
  color: var(--primary);
  letter-spacing: 0.1em;
  margin-bottom: 4px;
}

.certificate-preview-label {
  font-family: 'Manrope';
  font-size: 11px;
  color: var(--muted-foreground);
  text-transform: uppercase;
  margin-bottom: 16px;
}

.certificate-preview-name {
  font-family: 'Michroma';
  font-size: 20px;
  color: var(--foreground);
  margin-bottom: 8px;
}

.certificate-preview-course {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  font-style: italic;
}

.certificate-download-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 200ms;
}
.certificate-download-btn:hover { background: oklch(0.65 0.120 84.2); }
```

---

## COMPONENTES

| Componente | Tipo | Dependência |
|---|---|---|
| `CourseCard` | Card na grid do catálogo | — |
| `CourseContinueCard` | Card destaque "continuar estudando" | — |
| `CourseDetail` | Página de detalhe com módulos | Kobalte Accordion |
| `ModuleAccordion` | Módulo expandível com aulas | Kobalte Accordion |
| `LessonItem` | Item de aula dentro do módulo | — |
| `LessonPlayer` | Layout player + sidebar | Doc 05 (VideoPlayer) |
| `LessonSidebar` | Sidebar sticky com índice | — |
| `CourseProgress` | Barra de progresso | — |
| `CertificateCard` | Card de certificado disponível | — |
| `CourseWizard` | Wizard de criação (4 steps) | TanStack Form + Zod |
| `ModuleDragList` | Lista de módulos drag & drop | — |
| `CategoryPills` | Pills de categorias | — |

---

## ESTADOS

| Estado | Comportamento |
|---|---|
| Loading (catálogo) | Skeleton: 6 cards com shimmer (thumb + 3 linhas) |
| Loading (detalhe) | Skeleton: banner + 3 módulos shimmer |
| Empty (sem cursos disponíveis) | Ilustração + "Novos cursos em breve!" |
| Sem permissão | Rota retorna 403, tela nem carrega |
| Curso não iniciado | Botão "Iniciar Curso" ao invés de "Continuar" |
| Curso 100% | Badge verde "Concluído" + seção certificado |
| Aula concluída | Ícone ✅ verde no item |
| Aula atual | Highlight com borda esquerda gold |

---

## CHECKLIST

- [ ] Grid catálogo 3→2→1 colunas
- [ ] Card com thumbnail 16:9, progress overlay, meta info
- [ ] Seção "Continuar Estudando" com card destaque (só se tem curso em andamento)
- [ ] Detalhe do curso: accordion de módulos com aulas internas
- [ ] Status por módulo (Concluído/Em andamento/Bloqueado)
- [ ] Status por aula (✅/⬜) com ícones de cor
- [ ] Player layout: vídeo 16:9 + sidebar sticky 320px
- [ ] Mobile: sidebar vira accordion colapsável
- [ ] Aula ativa highlighted na sidebar (border-left gold)
- [ ] Navegação anterior/próxima aula
- [ ] Barra de progresso 8px gold
- [ ] Certificado ao 100%: card com preview + download PDF
- [ ] Wizard criação: 4 steps com drag & drop de módulos
- [ ] Ebooks/PDFs como material complementar downloadable
- [ ] Dark mode tokens oklch
