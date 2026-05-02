# 06 — UPLOAD DE VÍDEO & TRAVA TEMPORAL

> Sprint 1 · Rota: /upload
> Integração: Mux ou Cloudflare Stream

---

## REGRAS DE NEGÓCIO

- Upload de arquivo pronto (sem editor na plataforma)
- Trava trimestral (90 dias): vídeos "Institucional" (Empresa) e "Currículo" (Morador) → 1 atualização a cada 90 dias
- Limites mensais por plano: Síndico N2 4/mês, Emp Plus 8/mês (max 2/sem), Emp Pro 12/mês (max 3/sem)
- Vídeo-Currículo: até 90 segundos
- Formatos: MP4, MOV, WebM. Tamanho max definido pelo backend

## LAYOUT — UPLOAD PAGE

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Upload de Vídeo          [Cota: 3/8 este mês]│
│          │  h2 Michroma                                   │
│          │                                                │
│          │  TIPO DE VÍDEO (Radio)                         │
│          │  ○ Instrucional ← conteúdo técnico             │
│          │  ○ Institucional ← trava 90 dias               │
│          │  ○ Vídeo-Currículo ← morador, max 90s         │
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
  border: 2px dashed var(--border); border-radius: 12px;
  padding: 48px 24px; text-align: center;
  cursor: pointer; transition: all 200ms;
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
  border-radius: var(--radius);
}
.upload-progress-bar { flex: 1; height: 6px; background: var(--muted); border-radius: 3px; }
.upload-progress-fill { height: 100%; background: var(--primary); border-radius: 3px; transition: width 300ms; }
.upload-progress-percent { font-family: 'Manrope'; font-size: 14px; font-weight: 600; color: var(--foreground); min-width: 40px; }
```

## TRAVA DE 90 DIAS

Se o usuário tenta upload de tipo "Institucional" ou "Currículo" e existe vídeo nos últimos 90 dias:

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
  background: var(--warning) / 0.05; border: 1px solid var(--warning) / 0.3;
  border-radius: 12px;
}
.upload-blocked-icon { width: 48px; height: 48px; color: var(--warning); margin: 0 auto 16px; }
.upload-blocked-title { font-family: 'Michroma'; font-size: 18px; color: var(--foreground); margin-bottom: 8px; }
.upload-blocked-text { font-family: 'Manrope'; font-size: 14px; color: var(--muted-foreground); }
.upload-blocked-countdown { font-family: 'Manrope'; font-size: 24px; font-weight: 700; color: var(--warning); margin: 16px 0; }
```

## COMPONENTES

UploadPage, DragDropZone, UploadProgress, UploadMetadataForm, UploadTypeSelector, UploadBlockedState, ThumbnailSelector, TagInput, QuotaIndicator
