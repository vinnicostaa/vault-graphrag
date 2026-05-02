---
title: Video Upload & Trava Temporal — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, content, video, upload, mux]
bc: content
source: _chaos/06-VIDEO-UPLOAD.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 06 — Upload de Vídeo & Trava Temporal

> **Origem**: `_chaos/06-VIDEO-UPLOAD.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).

> Sprint 1 · App `cms` · Rota: `/upload`
> Integração: **Mux** via `IVideoProvider`.

---

## Regras de negócio

- **Upload de arquivo pronto** (sem editor na plataforma).
- **Trava trimestral 90 dias** (GR-029): vídeos "Institucional" (Empresa) e "Banco de Talentos / Currículo" (Morador) → 1 atualização a cada 90 dias.
- **Limites mensais por `plan_tier`**:

  | Role × tier | Vídeos / mês |
  |---|---|
  | Síndico `plus` | 4 |
  | Síndico `pro` | 12 |
  | Empresa `plus` | 8 (max 2/sem) |
  | Empresa `pro` | 12 (max 3/sem) |

- **Banco de Talentos** (D-099): até 90 segundos, com 5 vínculos profissionais visíveis (M3).
- **Formatos**: MP4, MOV, WebM. Tamanho máx. definido pelo backend (ver [[../../../../04-requirements/global|requirements/global]]).

> Quotas detalhadas: [[../../../../00-product/business-model#41-matriz-consolidada]].

## Layout — Upload Page

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Upload de Vídeo          [Cota: 3/8 este mês]│
│          │  h2 Michroma                                   │
│          │                                                │
│          │  TIPO DE VÍDEO (Radio)                         │
│          │  ○ Instrucional ← conteúdo técnico             │
│          │  ○ Institucional ← trava 90 dias               │
│          │  ○ Banco de Talentos ← morador, max 90s       │
│          │                                                │
│          │  DRAG & DROP ZONE                              │
│          │  ┌──────────────────────────────────────────┐ │
│          │  │          ☁️ (cloud-upload icon 48px)      │ │
│          │  │                                          │ │
│          │  │   Arraste seu vídeo ou clique para       │ │
│          │  │   selecionar arquivo                     │ │
│          │  │                                          │ │
│          │  │   MP4, MOV, WebM • Máx 500MB             │ │
│          │  └──────────────────────────────────────────┘ │
│          │                                                │
│          │  (Após selecionar:)                             │
│          │  ┌──────────────────────────────────────────┐ │
│          │  │ [thumb] video-exemplo.mp4                │ │
│          │  │         125MB • 03:42                    │ │
│          │  │         ████████████░░ 78%               │ │
│          │  │         [✕ Remover]                      │ │
│          │  └──────────────────────────────────────────┘ │
│          │                                                │
│          │  METADADOS                                     │
│          │  [Título do vídeo _______________________]    │
│          │  [Descrição _____________________________ ]    │
│          │  │                                    380/500│ │
│          │                                                │
│          │  Categoria: [▼ Selecione]                      │
│          │  Tags: [+Adicionar tag]                        │
│          │                                                │
│          │  Thumbnail: [Upload custom] ou [Auto-gerado]  │
│          │                                                │
│          │  [Publicar Vídeo →] primary                    │
│          │  [Salvar Rascunho] outline                     │
└──────────────────────────────────────────────────────────┘
```

## CSS

```css
.upload-zone {
  border: 2px dashed var(--border); border-radius: var(--radius-xl);
  padding: 48px 24px; text-align: center;
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
  background: var(--card);
}
.upload-zone:hover, .upload-zone.dragover {
  border-color: var(--primary);
  background: oklch(0.715 0.120 84.2 / 0.05);
}
.upload-zone-icon { width: 48px; height: 48px; color: var(--muted-foreground); margin: 0 auto 16px; }
.upload-zone-text { font-family: 'Manrope'; font-size: 14px; color: var(--muted-foreground); }
.upload-zone-formats { font-size: 12px; color: var(--muted-foreground); margin-top: 8px; }

.upload-progress {
  display: flex; align-items: center; gap: 12px;
  padding: 16px; background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md);
}
.upload-progress-bar { flex: 1; height: 6px; background: var(--muted); border-radius: 3px; }
.upload-progress-fill {
  height: 100%; background: var(--primary); border-radius: 3px;
  transition: width var(--duration-slow);
}
.upload-progress-percent {
  font-family: 'Manrope'; font-size: 14px; font-weight: 600;
  color: var(--foreground); min-width: 40px;
}
```

## Trava de 90 dias

Se o usuário tenta upload de tipo "Institucional" ou "Banco de Talentos" e existe vídeo nos últimos 90 dias:

```
┌──────────────────────────────────────┐
│  ⚠️ Upload bloqueado                 │
│                                      │
│  Você já atualizou seu vídeo         │
│  institucional em 15/01/2026.        │
│                                      │
│  Próxima atualização permitida em:   │
│  15/04/2026 (45 dias restantes)      │
│                                      │
│  ████████████████░░░░ 50%            │
│  45 dias restantes                   │
│                                      │
│  [← Voltar]                         │
└──────────────────────────────────────┘
```

```css
.upload-blocked {
  text-align: center; padding: 48px 24px;
  background: oklch(0.769 0.165 70.1 / 0.05);
  border: 1px solid oklch(0.769 0.165 70.1 / 0.3);
  border-radius: var(--radius-xl);
}
.upload-blocked-icon { width: 48px; height: 48px; color: var(--warning); margin: 0 auto 16px; }
.upload-blocked-title {
  font-family: 'Michroma'; font-size: 18px;
  color: var(--foreground); margin-bottom: 8px;
}
.upload-blocked-text { font-family: 'Manrope'; font-size: 14px; color: var(--muted-foreground); }
.upload-blocked-countdown {
  font-family: 'Manrope'; font-size: 24px; font-weight: 700;
  color: var(--warning); margin: 16px 0;
}
```

## Componentes

`UploadPage`, `DragDropZone`, `UploadProgress`, `UploadMetadataForm`, `UploadTypeSelector`, `UploadBlockedState`, `ThumbnailSelector`, `TagInput`, `QuotaIndicator`.

## Links

- [[./video-player|video-player]] — consumo do vídeo upado
- [[../../../../00-product/business-model#41-matriz-consolidada]] — quotas mensais por tier
- [[../../patterns/ui-states|patterns/ui-states]] — estados obrigatórios
- [[../../../../02-architecture/design-tokens|design-tokens]] — tokens
