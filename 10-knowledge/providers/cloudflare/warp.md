---
title: Cloudflare WARP — Client-side device agent (Cloudflare One Client)
type: note
tags:
  - provider
  - cloudflare
  - warp
  - cloudflare-one-client
  - zero-trust
  - device-posture
  - wireguard
category: zero-trust
doc-oficial: https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - WARP
  - Cloudflare One Client
  - 1.1.1.1 app
---

# Cloudflare WARP (Cloudflare One Client)

**O que é**: client-side agent que roteia tráfego do device até a rede Cloudflare via **WireGuard** ou **MASQUE**, + DNS-over-HTTPS. Oficialmente renomeado para **Cloudflare One Client** (mantém codinome WARP). Peça essencial da arquitetura [[sase|Cloudflare One / SASE]] — leva o perímetro até o device.

> Sem WARP, Cloudflare One é **server-side only** (Access + Tunnel). WARP traz o device para dentro da policy — posture signals, DNS filter, SWG, DLP client-side.

## Dois papéis do WARP

### 1. Consumer "1.1.1.1 app"
- Download gratuito na Apple Store / Google Play / microsoft store / brew cask.
- Roteia via Cloudflare DNS (`1.1.1.1`, `1.0.0.1`) + WARP-encrypted tunnel.
- Finalidade: privacy + performance. **Não** participa de policies corp.

### 2. WARP enrolled (Cloudflare Zero Trust)
- Device enrolled em team Cloudflare Zero Trust.
- Identity + device posture signals alimentam [[access|Access]] policies.
- Tráfego inspecionado por Gateway (SWG + DLP).
- Split-tunnel ou full-tunnel configurável pelo admin.

**Esta nota foca em #2** (enterprise use).

## Deployment

### Via MDM (recomendado enterprise)

| MDM | Suporte |
|---|---|
| **Microsoft Intune** | Configurator + device registration |
| **JAMF** (macOS/iOS) | Configuration profiles + login hooks |
| **Kandji** (macOS/iOS) | Cloudflare Zero Trust library |
| **JumpCloud** | Custom deployment via policies |
| **Workspace ONE / Hexnode / Fleet** | Generic MDM payloads |

Config via MDM:
- `organization` (team name)
- `auth_client_id` / `auth_client_secret` (OAuth service token)
- `enable_always_on` (true — previne bypass)
- `switch_locked` (true — user não consegue desligar)
- `support_url` (onboarding link)

### Manual install

```bash
# macOS
brew install --cask cloudflare-warp

# Linux (Debian/Ubuntu)
curl https://pkg.cloudflareclient.com/install | sudo bash
sudo warp-cli registration new <team-name>
sudo warp-cli connect

# Windows
winget install --id Cloudflare.Warp
```

Team name + login OIDC → device enrolled.

### Platforms suportadas (2026)

| OS | Client | Nota |
|---|---|---|
| **macOS** | Swift (native) | Full features + Keychain integration |
| **Windows** | .NET | Full features |
| **Linux** | CLI (`warp-cli`) | Server/dev-focused; menos UI |
| **iOS** | Swift | App Store; supervised via MDM |
| **Android** | Kotlin | Play Store; work profile suportado |
| **ChromeOS** | Android app | Managed via Google Workspace |

## Protocolos

- **WireGuard** — padrão (UDP, sub-kernel). Melhor performance.
- **MASQUE** (HTTP/3 + QUIC + CONNECT-IP) — fallback para redes que bloqueiam WireGuard (hotel WiFi, corporate proxy). Experimental → production desde 2024.

## Operating modes

1. **Traffic and DNS** (default) — todo tráfego passa pela Cloudflare: HTTP(S), DNS, tudo. Permite SWG + DLP inline.
2. **DNS-only** — apenas DNS queries via Cloudflare (sem tunnel de tráfego). Útil para DNS filtering sem overhead.
3. **Traffic-only** — tráfego via tunnel; DNS resolve localmente. Caso de uso específico.
4. **Local proxy** — client fica num network namespace; útil para browsers específicos sem afetar OS.
5. **Posture-only** — coleta posture signals mas não roteia tráfego. Useful para inventário.

## Device posture signals

Ampla lista; os principais:

### Built-in (sem integrations)
- OS version + build.
- Disk encryption state (FileVault, BitLocker, LUKS).
- Firewall status.
- Antivirus signatures fresh.
- OS serial number.
- WARP active + version.
- User session status.
- Client certificate presence.

### Via integration (Service-to-Service)
- **CrowdStrike Falcon** — ZTA score, detections.
- **SentinelOne** — agent health, threats.
- **Microsoft Intune** — compliance state.
- **JAMF** — pro config status.
- **Kolide** — custom checks.
- **Tanium** — threat signal.

Policies [[access|Access]] consomem esses signals:

```
ALLOW se:
  user IN group:engineering
  AND device_posture.os_version >= macOS 14.0
  AND device_posture.disk_encryption = true
  AND device_posture.crowdstrike_zta_score >= 70
```

## Integração com Access & Tunnel

```
User device (WARP)
      │
      │ authenticated + posture check
      ▼
┌──────────────────────────┐
│ Cloudflare edge           │
│ ┌──────────────────────┐  │
│ │ Access policy eval    │──▶ [[tunnel]] para app interno
│ │ (identity + posture) │  │   ou SWG para internet
│ └──────────────────────┘  │
└──────────────────────────┘
```

### Access for Infrastructure (SSH short-lived certs)

Requer WARP. User conecta via `ssh` normal; WARP intercepta + injeta cert de curta duração emitido pelo CF Access. Audit completo de sessão (comandos digitados). Alternativa: BastionHost clássico.

## Gateway policies (quando WARP está ligado)

- **DNS policies** — bloquear categorias (malware, adult, etc.); `block`, `allow`, `warn`.
- **HTTP policies** — deep inspection (MITM com cert CF instalado via MDM).
- **Network policies** — L4 (IP/porta) para non-HTTP.
- **Egress** — force traffic via IP fixo Cloudflare (SaaS allowlist).

## Limites e considerações

- **Bandwidth**: sem cap explícito em enterprise; consumer free tier é unmetered mas priority reduced.
- **Device count**: por license/seat em enterprise plans.
- **IPv6**: suportado; dual-stack config.
- **MTU**: 1280 default (WireGuard); adjustable.
- **Battery (mobile)**: WARP é leve; impacto < 5% bateria típica.
- **Certificate trust**: inspeção HTTPS exige CA CF instalada (via MDM). Sem isso, muitos sites HTTPS quebram.

## Anti-patterns

1. **User consegue desligar WARP** em device corp — configurar `switch_locked=true` via MDM.
2. **Sem policy de posture** — WARP só tunel; Zero Trust exige policy usando os signals.
3. **Forçar full-tunnel em SaaS que funciona melhor direto** — latência extra sem benefício. Split tunnel com SaaS allowlist.
4. **Sem cert CF instalado** — HTTPS inspection quebra; fallback silencioso perde DLP/SWG.
5. **MASQUE como padrão em todas redes** — WireGuard é melhor; MASQUE é fallback quando bloqueado.
6. **Posture check **sem** integration com endpoint security** — built-in signals são limitados; vale adicionar CrowdStrike/Intune.
7. **Consumer 1.1.1.1 em device corp** — não fornece policies; confunde user. Enroll enterprise.
8. **Sem DEX configurado** — sem observability, incidentes são "o VPN ta lento" sem dados.

## Reference Architectures

- ZTNA Access Policies design — https://developers.cloudflare.com/reference-architecture/design-guides/designing-ztna-access-policies/ (consultada 2026-04-24)
- VPN concentrator → ZTNA migration — https://developers.cloudflare.com/reference-architecture/design-guides/network-vpn-migration/ (consultada 2026-04-24)

Ver [[sase]] para visão completa da pilha.

## Relações

- **Complementa**: [[access]] (ZTNA server-side), [[tunnel]] (cloudflared server-side) — tríade Cloudflare One
- **Umbrella**: [[sase]]
- **Princípio**: [[../../security/beyond-corp]] (WARP é "Device Trust Agent" do modelo BeyondCorp)
- **Posture**: [[../../security/mfa-step-up]] (device posture é signal para step-up)
- **Network proto**: WireGuard, MASQUE (QUIC)

## Fontes

- [Cloudflare One Client docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/) (consultada 2026-04-24)
- [WARP deployment via MDM](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/mdm-deployment/) (consultada 2026-04-24)
- [WARP device posture](https://developers.cloudflare.com/cloudflare-one/identity/devices/) (consultada 2026-04-24)
- [WireGuard protocol](https://www.wireguard.com/) (consultada 2026-04-24)
- [MASQUE IETF](https://datatracker.ietf.org/wg/masque/about/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[sase]]
- [[access]]
- [[tunnel]]
- [[../../security/beyond-corp]]
- [[../../security/mfa-step-up]]
