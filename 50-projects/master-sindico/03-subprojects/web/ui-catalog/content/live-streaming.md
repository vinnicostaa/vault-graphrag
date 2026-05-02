---
title: Live Streaming Institucional (Broadcast) — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, content, live, streaming, broadcast]
bc: content
source: _chaos/18-LIVE-STREAMING.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 18 — Lives Institucionais (Streaming ao Vivo, broadcast)

> **Origem**: `_chaos/18-LIVE-STREAMING.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).
> **⚠️ D-133 (2026-04-25)**: Lives são **broadcast institucionais** (vídeos ao vivo de empresa / Master Síndico). **NÃO é Assembleia** (essa é conferência tipo Meet — ver [[../assembly/management|assembly/management]]). Backend Livekit é dedicado a Assembly. **Lives broadcast** podem usar provider diferente: **Mux Live** ou **Cloudflare Stream** (decisão de provider em ADR — pending).

> Sprint 4 · App `cms` / `lms` (player live) · Rotas: `/lives`, `/lives/:id`, `/admin/lives/nova` (apps/admin only)
> **APENAS o Administrador Master Síndico** pode iniciar transmissões broadcast.

---

## Regras de negócio

### Restrição crítica
- **APENAS** Admin Master Síndico cria / inicia lives institucionais (D-134 — em `apps/admin` separado).
- Síndicos e Empresas **NÃO** fazem live broadcast.
- **Integração**: Mux Live Streaming **ou** Cloudflare Stream (ADR pending — D-133).
- **Ingest RTMP**: chave RTMP para OBS / software de transmissão.
- **Gravação automática**: live disponível como vídeo após término (passa a ser asset Mux normal).

### Permissões

| Ação | Morador `base` | Morador `plus` (futuro) | Síndico (`base`+) | Empresa `plus`/`pro` | Marketing | Admin |
|---|---|---|---|---|---|---|
| Assistir live ao vivo | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Assistir live gravada | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Criar / gerenciar live | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Chat durante live | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |

### Chat
- Opcional: Admin pode ativar / desativar por live.
- Tempo real durante a transmissão (WebSocket).
- Sem chat na versão gravada (vira VOD comum).

### Notificação
- Ao iniciar live: notificação Push / Email "Live ao vivo agora!" (planos com acesso).

---

## Layout — Lista de Lives

Rota: `/lives`. Estrutura:

1. **Ao Vivo Agora** (destaque, borda pulsante vermelha)
2. **Próximas Lives** (cards com data / [Lembrar])
3. **Lives Gravadas** (grid reaproveita VideoCard de [[./video-player|video-player]])

### CSS — Card Live Ao Vivo

```css
.live-now-card {
  background: var(--card);
  border: 2px solid var(--destructive);
  border-radius: var(--radius-2xl);
  overflow: hidden;
  margin-bottom: 28px;
  animation: livePulse 2s ease-in-out infinite;
}
@keyframes livePulse {
  0%, 100% { border-color: var(--destructive); }
  50%      { border-color: oklch(0.568 0.200 26.4 / 0.5); }
}
.live-now-badge {
  position: absolute; top: 12px; left: 12px;
  display: flex; align-items: center; gap: 6px;
  padding: 4px 10px;
  background: var(--destructive); color: var(--destructive-foreground);
  border-radius: var(--radius-sm);
  font-family: 'Manrope'; font-size: 12px; font-weight: 700;
  text-transform: uppercase;
}
.live-now-dot {
  width: 8px; height: 8px;
  background: white; border-radius: 50%;
  animation: blink 1s step-start infinite;
}
.live-now-viewers {
  position: absolute; top: 12px; right: 12px;
  padding: 4px 10px;
  background: oklch(0 0 0 / 0.6); color: white;
  border-radius: var(--radius-sm);
  font-family: 'Manrope'; font-size: 12px; font-weight: 500;
}
```

---

## Layout — Player de Live (assistindo)

Rota: `/lives/:id`. Layout: vídeo 16:9 + chat sidebar 340 px.

```css
.live-layout {
  display: grid; grid-template-columns: 1fr 340px;
  gap: 0; height: calc(100vh - 64px);
}
@media (max-width: 1023px) {
  .live-layout { grid-template-columns: 1fr; height: auto; }
}
.live-player-container {
  width: 100%; aspect-ratio: 16/9; max-height: 70vh;
  background: #000; position: relative;
}
.live-chat {
  border-left: 1px solid var(--border);
  display: flex; flex-direction: column; background: var(--card);
}
@media (max-width: 1023px) {
  .live-chat {
    position: fixed; bottom: 0; left: 0; right: 0;
    height: 50vh; border-left: none; border-top: 1px solid var(--border);
    border-radius: var(--radius-2xl) var(--radius-2xl) 0 0;
    z-index: var(--z-banner);
    transform: translateY(100%); transition: transform var(--duration-slow) ease;
  }
  .live-chat.open { transform: translateY(0); }
}
.live-chat-message {
  font-family: 'Manrope'; font-size: 13px; line-height: 1.4; padding: 4px 0;
}
.live-chat-author {
  font-weight: 600; color: var(--primary); margin-right: 6px;
}
.live-chat-input {
  flex: 1; padding: 8px 12px;
  background: var(--background); border: 1px solid var(--input);
  border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 14px; color: var(--foreground);
  outline: none;
}
```

---

## Painel Admin — Gerenciar Live (em `apps/admin`)

Rota: `/admin/lives/nova` (D-134 — app admin separada).

```
┌──────────────────────────────────────────┐
│  NOVA TRANSMISSÃO                        │
│  Título *                                │
│  Descrição                               │
│  Thumbnail (upload)                      │
│  Chat: [✅ Habilitado] / [❌ Desabilitado]│
│  ─── CHAVE RTMP ──────                  │
│  URL: rtmp://live.mastersindico...       │
│  Key: sk-live-abc123xyz...               │
│  [Iniciar Live 🔴]  [Encerrar ⏹️]       │
└──────────────────────────────────────────┘
```

```css
.rtmp-keys-box {
  background: var(--surface);
  border-radius: var(--radius-md);
  padding: 16px; margin: 16px 0;
}
.rtmp-key-value {
  flex: 1; font-family: var(--font-mono); font-size: 13px;
  color: var(--surface-foreground); overflow: hidden;
  text-overflow: ellipsis; white-space: nowrap;
}
```

## Componentes

`LiveNowCard`, `LiveUpcomingCard`, `LiveRecordedCard`, `LivePlayer` (Mux / Cloudflare adapter), `LiveChat` (WebSocket), `LiveChatMessage`, `LiveChatInput`, `LiveChatToggle`, `LiveAdminPanel`.

## Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Links

- [[./video-player|video-player]] — VideoCard reaproveitado em lives gravadas
- [[../assembly/management|assembly/management]] — Assembleia (Livekit, NÃO é live broadcast — D-133)
- [[../admin/main|admin/main]] — apps/admin
- [[../../../../STATE]] — D-133 separação Live × Assembleia / D-134 admin separada
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
