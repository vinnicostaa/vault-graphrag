# 18 — LIVES INSTITUCIONAIS (STREAMING AO VIVO)

> Sprint 4 · Rotas: /lives, /lives/:id, /admin/lives/nova (MasterSíndico only)
> APENAS o Administrador MasterSíndico pode iniciar transmissões ao vivo.

---

## REGRAS DE NEGÓCIO

### Restrição Crítica
- **APENAS** MasterSíndico (dono da plataforma) pode criar/iniciar lives
- Síndicos e Empresas NÃO fazem live
- Integração: Mux Live Streaming ou Cloudflare Stream
- Ingest RTMP: chave RTMP para OBS/software de transmissão
- Gravação automática: live disponível como vídeo após término

### Permissões (Matriz Funcional)
| Ação | Base | Morador Pg | Síndico N1+ | Emp. Plus/Pro | Marketing |
|------|------|-----------|-------------|---------------|-----------|
| Assistir live ao vivo | ❌ | ✅ | ✅ | ✅ | ✅ |
| Assistir live gravada | ✅ | ✅ | ✅ | ✅ | ✅ |
| Criar/gerenciar live | ❌ | ❌ | ❌ | ❌ | ❌ (só MasterSíndico) |
| Chat durante live | ❌ | ✅ | ✅ | ✅ | ✅ |

### Chat
- Opcional: MasterSíndico pode ativar/desativar chat por live
- Chat em tempo real durante a transmissão
- Sem chat na versão gravada

### Notificação
- Ao iniciar live: notificação Push/Email "Live ao vivo agora!" (para planos com acesso)

---

## LAYOUT — LISTA DE LIVES

Rota: `/lives`

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  LIVES MASTER SÍNDICO                                  │
│          │  h2 Michroma 20px                                      │
│          │                                                        │
│          │  ┌─── AO VIVO AGORA 🔴 ──────────────────────────┐   │
│          │  │                                                 │   │
│          │  │  [THUMBNAIL LIVE 16:9]                          │   │
│          │  │  🔴 AO VIVO • 234 assistindo                    │   │
│          │  │                                                 │   │
│          │  │  Gestão de Fachadas: o que todo síndico         │   │
│          │  │  precisa saber                                  │   │
│          │  │                                                 │   │
│          │  │  [🔴 Assistir Agora →]                          │   │
│          │  └─────────────────────────────────────────────────┘   │
│          │                                                        │
│          │  ─── PRÓXIMAS LIVES ───────────────────────────────   │
│          │  ┌──────────────────┐ ┌──────────────────┐            │
│          │  │ [thumb]          │ │ [thumb]          │            │
│          │  │ ⏰ 25/02 20h     │ │ ⏰ 04/03 20h     │            │
│          │  │ AVCB Simplif.    │ │ Seguro Condo     │            │
│          │  │ [🔔 Lembrar]     │ │ [🔔 Lembrar]     │            │
│          │  └──────────────────┘ └──────────────────┘            │
│          │                                                        │
│          │  ─── LIVES GRAVADAS ───────────────────────────────   │
│          │  ┌────────────┐ ┌────────────┐ ┌────────────┐        │
│          │  │ [thumb]    │ │ [thumb]    │ │ [thumb]    │        │
│          │  │ Hidráulica │ │ Seguros    │ │ Elétrica   │        │
│          │  │ 1h23min    │ │ 55min      │ │ 1h05min    │        │
│          │  │ 15/01/2026 │ │ 08/01/2026 │ │ 20/12/2025 │        │
│          │  └────────────┘ └────────────┘ └────────────┘        │
└──────────────────────────────────────────────────────────────────┘
```

### CSS — Card Live Ao Vivo (destaque)

```css
.live-now-card {
  background: var(--card);
  border: 2px solid var(--destructive);
  border-radius: 16px;
  overflow: hidden;
  margin-bottom: 28px;
  animation: livePulse 2s ease-in-out infinite;
}

@keyframes livePulse {
  0%, 100% { border-color: var(--destructive); }
  50% { border-color: oklch(0.568 0.200 26.4 / 0.5); }
}

.live-now-thumb {
  width: 100%;
  aspect-ratio: 16/9;
  object-fit: cover;
  background: var(--surface);
  position: relative;
}

.live-now-badge {
  position: absolute;
  top: 12px;
  left: 12px;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 4px 10px;
  background: var(--destructive);
  color: var(--destructive-foreground);
  border-radius: 6px;
  font-family: 'Manrope';
  font-size: 12px;
  font-weight: 700;
  text-transform: uppercase;
}

.live-now-dot {
  width: 8px;
  height: 8px;
  background: white;
  border-radius: 50%;
  animation: blink 1s step-start infinite;
}

.live-now-viewers {
  position: absolute;
  top: 12px;
  right: 12px;
  padding: 4px 10px;
  background: oklch(0 0 0 / 0.6);
  color: white;
  border-radius: 6px;
  font-family: 'Manrope';
  font-size: 12px;
  font-weight: 500;
}

.live-now-body { padding: 20px; }

.live-now-title {
  font-family: 'Manrope';
  font-size: 18px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 12px;
  line-height: 1.3;
}

.live-now-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: var(--destructive);
  color: var(--destructive-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 200ms;
}
.live-now-btn:hover { opacity: 0.9; }
```

### CSS — Card Próxima Live

```css
.live-upcoming-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
  transition: all 200ms;
}
.live-upcoming-card:hover { border-color: var(--primary); }

.live-upcoming-thumb {
  width: 100%;
  aspect-ratio: 16/9;
  object-fit: cover;
  background: var(--muted);
  position: relative;
}

.live-upcoming-date {
  position: absolute;
  bottom: 8px;
  left: 8px;
  padding: 4px 8px;
  background: oklch(0 0 0 / 0.7);
  color: white;
  border-radius: 4px;
  font-family: 'Manrope';
  font-size: 12px;
}

.live-upcoming-body { padding: 14px; }

.live-upcoming-title {
  font-family: 'Manrope';
  font-size: 15px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 10px;
}

.live-remind-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 14px;
  background: transparent;
  border: 1px solid var(--primary);
  color: var(--primary);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 200ms;
}
.live-remind-btn:hover { background: oklch(0.715 0.120 84.2 / 0.1); }
.live-remind-btn.active {
  background: var(--primary);
  color: var(--primary-foreground);
}
```

### CSS — Card Live Gravada (reutiliza padrão video card doc 05)

```css
.live-recorded-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 200ms;
}
.live-recorded-card:hover {
  border-color: var(--primary);
  transform: translateY(-2px);
}

.live-recorded-thumb {
  width: 100%;
  aspect-ratio: 16/9;
  object-fit: cover;
  background: var(--muted);
  position: relative;
}

.live-recorded-duration {
  position: absolute;
  bottom: 8px;
  right: 8px;
  padding: 2px 6px;
  background: oklch(0 0 0 / 0.7);
  color: white;
  border-radius: 4px;
  font-family: 'Manrope';
  font-size: 11px;
}

.live-recorded-body { padding: 12px; }

.live-recorded-title {
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 4px;
}

.live-recorded-date {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
}
```

---

## LAYOUT — PLAYER DE LIVE (Assistindo)

Rota: `/lives/:id`

```
┌──────────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                            │
│          │  ┌────────────────────────────────────┐ ┌──────────────┐  │
│          │  │                                    │ │  CHAT 💬     │  │
│          │  │                                    │ │              │  │
│          │  │         [LIVE PLAYER]               │ │  João: Boa! │  │
│          │  │          16:9 ratio                  │ │  Maria: Top │  │
│          │  │          max-h: 70vh                 │ │  Carlos: 👏│  │
│          │  │                                    │ │  Ana: Dúvida│  │
│          │  │  🔴 AO VIVO  •  234 assistindo      │ │  ...        │  │
│          │  │                                    │ │              │  │
│          │  └────────────────────────────────────┘ │  ┌──────────┐│  │
│          │                                         │  │ Msg...   ││  │
│          │  Gestão de Fachadas                     │  │  [Enviar]││  │
│          │  h2 Manrope 20px bold                   │  └──────────┘│  │
│          │                                         └──────────────┘  │
│          │  🏢 Master Síndico                                        │
│          │  Iniciada há 45 minutos                                   │
└──────────────────────────────────────────────────────────────────────┘
```

### Mobile: chat vira overlay togglável

```
┌──────────────────────────────┐
│  [← Voltar]  🔴 AO VIVO     │
│                              │
│  ┌──────────────────────────┐│
│  │                          ││
│  │    [LIVE PLAYER]         ││
│  │    full width            ││
│  │                          ││
│  └──────────────────────────┘│
│                              │
│  Gestão de Fachadas          │
│  234 assistindo              │
│                              │
│  [💬 Chat]  ← toggle         │
│                              │
│  ┌── CHAT OVERLAY ──────────┐│
│  │ João: Boa!               ││
│  │ Maria: Top               ││
│  │ ┌────────┐ [Enviar]      ││
│  │ │ Msg... │               ││
│  │ └────────┘               ││
│  └──────────────────────────┘│
└──────────────────────────────┘
```

### CSS — Live Player Layout

```css
.live-layout {
  display: grid;
  grid-template-columns: 1fr 340px;
  gap: 0;
  height: calc(100vh - 64px);
}

@media (max-width: 1023px) {
  .live-layout {
    grid-template-columns: 1fr;
    height: auto;
  }
}

.live-main { overflow-y: auto; }

.live-player-container {
  width: 100%;
  aspect-ratio: 16/9;
  max-height: 70vh;
  background: #000;
  position: relative;
}

.live-badge-overlay {
  position: absolute;
  top: 16px;
  left: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.live-info {
  padding: 20px 24px;
}

.live-title {
  font-family: 'Manrope';
  font-size: 20px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 8px;
}

.live-meta {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--muted-foreground);
}
```

### CSS — Chat Sidebar

```css
.live-chat {
  border-left: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  background: var(--card);
}

@media (max-width: 1023px) {
  .live-chat {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    height: 50vh;
    border-left: none;
    border-top: 1px solid var(--border);
    border-radius: 16px 16px 0 0;
    z-index: 40;
    transform: translateY(100%);
    transition: transform 300ms ease;
  }
  .live-chat.open { transform: translateY(0); }
}

.live-chat-header {
  padding: 14px 16px;
  border-bottom: 1px solid var(--border);
  font-family: 'Michroma';
  font-size: 13px;
  letter-spacing: 0.1em;
  color: var(--foreground);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.live-chat-close {
  display: none;
  background: none;
  border: none;
  color: var(--muted-foreground);
  cursor: pointer;
  font-size: 18px;
}
@media (max-width: 1023px) {
  .live-chat-close { display: block; }
}

.live-chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 12px 16px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.live-chat-message {
  font-family: 'Manrope';
  font-size: 13px;
  line-height: 1.4;
  padding: 4px 0;
}

.live-chat-author {
  font-weight: 600;
  color: var(--primary);
  margin-right: 6px;
}

.live-chat-text {
  color: var(--foreground);
}

.live-chat-input-area {
  padding: 12px 16px;
  border-top: 1px solid var(--border);
  display: flex;
  gap: 8px;
}

.live-chat-input {
  flex: 1;
  padding: 8px 12px;
  background: var(--background);
  border: 1px solid var(--input);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  outline: none;
}
.live-chat-input:focus {
  border-color: var(--ring);
}

.live-chat-send {
  padding: 8px 14px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}

/* Botão toggle chat mobile */
.live-chat-toggle {
  display: none;
  padding: 10px 20px;
  background: var(--surface);
  color: var(--surface-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  margin: 16px;
}
@media (max-width: 1023px) {
  .live-chat-toggle { display: flex; align-items: center; gap: 6px; }
}
```

---

## PAINEL ADMIN — GERENCIAR LIVE (MasterSíndico only)

Rota: `/admin/lives/nova` — exclusivo web, painel admin

```
┌──────────────────────────────────────────┐
│  NOVA TRANSMISSÃO                        │
│                                          │
│  Título *                                │
│  [Gestão de Fachadas: o que...     ]    │
│                                          │
│  Descrição                               │
│  [Nesta live, vamos abordar...     ]    │
│                                          │
│  Thumbnail                               │
│  [📷 Upload]                             │
│                                          │
│  Chat: [✅ Habilitado] / [❌ Desabilitado]│
│                                          │
│  ─── CHAVE RTMP ──────────────────────  │
│  ┌──────────────────────────────────┐    │
│  │ URL: rtmp://live.mastersindico...│    │
│  │ Key: sk-live-abc123xyz...        │    │
│  └──────────────────────────────────┘    │
│  [📋 Copiar URL]  [📋 Copiar Key]       │
│                                          │
│  [Iniciar Live 🔴]  [Encerrar ⏹️]       │
└──────────────────────────────────────────┘
```

```css
.rtmp-keys-box {
  background: var(--surface);
  border-radius: 8px;
  padding: 16px;
  margin: 16px 0;
}

.rtmp-key-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.rtmp-key-label {
  font-family: 'Manrope';
  font-size: 12px;
  font-weight: 600;
  color: var(--surface-foreground);
  opacity: 0.7;
  min-width: 40px;
}

.rtmp-key-value {
  flex: 1;
  font-family: monospace;
  font-size: 13px;
  color: var(--surface-foreground);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.rtmp-copy-btn {
  padding: 4px 8px;
  background: oklch(1 0 0 / 0.1);
  border: none;
  border-radius: 4px;
  color: var(--surface-foreground);
  cursor: pointer;
  font-size: 12px;
}
```

---

## COMPONENTES

| Componente | Tipo | Dependência |
|---|---|---|
| `LiveNowCard` | Card destaque para live ativa | — |
| `LiveUpcomingCard` | Card de próxima live | — |
| `LiveRecordedCard` | Card de live gravada (video card) | — |
| `LivePlayer` | Player de live streaming | Mux/Cloudflare |
| `LiveChat` | Sidebar/overlay de chat | WebSocket |
| `LiveChatMessage` | Mensagem individual | — |
| `LiveChatInput` | Input de mensagem | — |
| `LiveChatToggle` | Botão toggle mobile | — |
| `LiveAdminPanel` | Painel RTMP (MasterSíndico) | — |

---

## CHECKLIST

- [ ] Card "Ao Vivo Agora" com borda pulsante vermelha
- [ ] Badge 🔴 AO VIVO com dot piscando
- [ ] Contagem de espectadores em tempo real
- [ ] Cards de próximas lives com data/hora + [Lembrar]
- [ ] Cards de lives gravadas (reutiliza video card pattern)
- [ ] Player live 16:9 + chat sidebar 340px
- [ ] Chat mobile: overlay 50vh com toggle
- [ ] Chat em tempo real (WebSocket)
- [ ] Painel admin: chaves RTMP + controle iniciar/encerrar
- [ ] Chat toggle ativo/desativo pelo MasterSíndico
- [ ] Base só vê gravadas, pagantes veem ao vivo
- [ ] Dark mode tokens oklch
