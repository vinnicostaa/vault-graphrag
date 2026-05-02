---
title: Check-in — UI Spec
type: ui-spec
tags: [ui-catalog, assembly, master-sindico, frontend, check-in, qr]
bc: assembly
source: _chaos/ARQUITETURA-ASSEMBLEIA.pdf §5.1-5.2 + DIAGRAMA-VISUAL.pdf §3 + MÓDULO-TELAS.pdf TELA "CHECK-IN"
absorbed_in: 2026-04-25 — Fase E
status: absorbed
---

# Check-in — 3 Modalidades de Entrada na Assembleia

> **Origem**: 3 PDFs Master Síndico — destilados em Fase E (2026-04-25).
> **Backend**: ASM-012 (3 modalidades + WebSocket < 200ms latência).

> Sprint 3 · App `assembly` · Rotas: `/assembleias/:id/checkin`, `/terminal/:terminalId`, `/qr/:assemblyId/checkin`
> Perfis: Morador (app/QR), Operador terminal (sindico/administradora).

---

## Visão geral — 3 modalidades

| Modalidade   | Como funciona                                  | Voto consequente                |
|--------------|------------------------------------------------|----------------------------------|
| **App**      | Morador autenticado clica "Check-in"           | Voto direto pelo app             |
| **QR Code**  | Escaneia QR exclusivo da assembleia            | Voto pelo app pós-redirect       |
| **Terminal** | Operador captura CPF + bloco + unidade no kiosk| **Voto presencial assistido** (ASM-016) |

---

## Tela 1 — Check-in pelo App (Morador)

**Rota**: `/assembleias/:id/checkin` (Morador autenticado)

```
┌────────────────────────────────────────────┐
│  CHECK-IN NA ASSEMBLEIA                    │
│                                            │
│  Assembleia Ordinária 2026/1              │
│  📅 15/03/2026 • 🕐 19:00                 │
│  📍 Salão de festas — Bloco A             │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │   📍 LOCALIZAÇÃO ATUAL                │ │
│  │   ✅ Você está no condomínio         │ │
│  │   (geofence opcional — M4+)          │ │
│  └──────────────────────────────────────┘ │
│                                            │
│  Sua unidade:                              │
│  Bloco A — Apt 302                         │
│  Titular: João Silva                       │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │     ✓ FAZER CHECK-IN                  │ │
│  │     (toque para registrar presença)  │ │
│  └──────────────────────────────────────┘ │
│                                            │
│  ⚠️ Após check-in, você ficará registrado │
│  como presente. Saída registra ausência   │
│  momentânea.                               │
└────────────────────────────────────────────┘
```

### Após check-in

```
┌────────────────────────────────────────┐
│  ✅ PRESENÇA REGISTRADA               │
│                                        │
│  19:02:34 • 15/03/2026                │
│  Bloco A — Apt 302                    │
│                                        │
│  Sua presença foi sincronizada com o  │
│  painel do síndico.                   │
│                                        │
│  [Ir para sessão ao vivo →]           │
└────────────────────────────────────────┘
```

### Regras

- Step-up MFA opcional (M4+) — biometria/PIN para confirmar identidade.
- Geofence opcional (M4+ — só permite check-in se dentro do condomínio).
- WebSocket broadcast `/ws/assemblies/:id/state` → painel síndico recebe < 200ms.
- Idempotente: re-clique não duplica registro.

---

## Tela 2 — Check-in via QR Code

**Rota acessada**: `/qr/:assemblyId/checkin` (token assinado, redireciona pro app autenticado)

```
┌──────────────────────────────────────┐
│  📱 ESCANEIE O QR CODE               │
│                                      │
│  ┌──────────────────────────┐       │
│  │                          │       │
│  │    [QR CODE IMG]         │       │
│  │                          │       │
│  │   Tamanho exibido:       │       │
│  │   no telão de entrada    │       │
│  │                          │       │
│  └──────────────────────────┘       │
│                                      │
│  Acesso direto pelo app Master       │
│  Síndico — toque no link após scan   │
│                                      │
│  Não tem o app? [Baixar agora]       │
└──────────────────────────────────────┘
```

### Flow técnico

1. QR code gerado ao publicar assembleia (ASM-001 + R48 §11).
2. URL assinada: `https://app.mastersindico.com.br/qr/:assemblyId/checkin?token=:hmac`.
3. Token contém: `{assembly_id, expires_at=startTime+4h}`.
4. App abre com deep-link → autentica → checa apto → registra check-in.
5. Se não autenticado → redireciona pra login → retoma fluxo.

### Onde exibir o QR

- Telão de entrada (sala de assembleia).
- Email de notificação T-3h.
- Push notification.
- Cartaz físico (impressão exportável).

---

## Tela 3 — Terminal de Apoio (Kiosk)

**Rota**: `/terminal/:terminalId` (acesso público no terminal — autenticação via token de sessão do operador)

```
┌──────────────────────────────────────────────┐
│  🏢 TERMINAL DE APOIO — ENTRADA              │
│  Master Síndico • Operador: Maria (admin)   │
│                                              │
│  CPF do morador *                            │
│  ┌────────────────────────────────────────┐ │
│  │ [___.___.___-__]                       │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  Bloco *           Unidade *                 │
│  [A    ▼]          [302   ]                 │
│                                              │
│  [🔍 Buscar morador]                        │
│                                              │
│  ┌─Resultado da busca────────────────────┐ │
│  │ ✓ João Silva (CPF *.789-00)           │ │
│  │ ✓ Bloco A — Apt 302                   │ │
│  │ ✓ Apto a votar                         │ │
│  │ ⓘ Será voto presencial assistido      │ │
│  │   (operador lança voto pelo painel)   │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  [✓ Registrar presença]                     │
└──────────────────────────────────────────────┘
```

### Após check-in via terminal

```
┌──────────────────────────────────────┐
│  ✅ PRESENÇA REGISTRADA              │
│                                      │
│  João Silva • Bloco A Apt 302       │
│  Modalidade: Terminal                │
│  Operador: Maria (admin)             │
│  19:05:12 • TERM-001-LOBBY          │
│                                      │
│  ⚠️ Voto será presencial assistido   │
│  (lançado pelo síndico/admin no      │
│  painel da assembleia)               │
│                                      │
│  [Próximo morador →]                 │
└──────────────────────────────────────┘
```

### Regras (ASM-012 + R57 §6)

- Terminal: tablet/notebook fixo, autenticado pelo operador (síndico ou administradora).
- Sessão `terminal_session_id` única por hardware.
- CPF + bloco + unidade obrigatórios.
- Sistema valida: morador existe, é da unidade, está apto a votar (não inadimplente).
- Marca `origin=terminal` no `assembly_check_ins`.
- Voto consequente é **presencial assistido** (ASM-016 — termo + operator_id obrigatórios).

---

## Tela 4 — Painel de Presença (Síndico/Administradora — real-time)

**Rota**: `/assembleias/:id/live/presenca`

Reaproveita widget `PresenceCard` de [[./live-session|live-session]]. Detalha lista nominal:

```
┌──────────────────────────────────────────────────┐
│  PRESENÇA REAL-TIME                              │
│  72/120 unidades presentes (60%)                 │
│  Frações presentes: 68%                          │
│                                                  │
│  Filtros: [Todas ▼] [Presentes] [Ausentes]      │
│           [Online] [Modalidade ▼]                │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │ ✅ A-101  João Costa     App      19:01    │ │
│  │ ✅ A-102  Maria Lima     QR       19:03    │ │
│  │ 🟡 A-103  Pedro Souza    Terminal 19:05    │ │
│  │ ⏸  A-104  Ana Paula     App      19:30→saiu│ │
│  │ ❌ A-105  (não presente)                    │ │
│  └────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

### Estados de presença (ASM-### Fase E)

- `presente` (verde) — fez check-in e está conectado/presencial.
- `ausente_momentaneo` (amarelo) — saiu da sessão; janela de 15min antes de virar abstenção (ASM-016).
- `desconectado` (cinza) — saiu definitivamente.
- `nao_presente` (vermelho/X) — nunca fez check-in.

---

## Componentes

`CheckInAppButton`, `QRCodeDisplay`, `QRScanRedirect`, `TerminalKioskForm`, `TerminalCheckInResult`, `PresenceCard`, `PresenceList`, `PresenceStatusBadge`.

## Cross-links

- [[./pre-assembly|pre-assembly]] — preparação prévia
- [[./live-session|live-session]] — painel ao vivo
- [[./voting|voting]] — votação consequente
- [[../../../../04-requirements/functional/assembly|functional/assembly]] — ASM-012 (check-in 3 modalidades)
- [[../../../../01-domain/aggregates/Assembly|Assembly]]
- [[../../../../STATE]] — D-133

## Pendências

- ⚠️ Geofence M4+ (precisa permissão localização — UX privacy + ADR).
- ⚠️ Biometria opcional no terminal (TouchID nativo iPad / Windows Hello).
- ⚠️ Detecção de ausência momentânea: heartbeat WebSocket vs polling — definir spec técnica.
