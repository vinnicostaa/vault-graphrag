---
title: Cloudflare SASE / Cloudflare One — Zero Trust umbrella
type: note
tags:
  - provider
  - cloudflare
  - sase
  - zero-trust
  - cloudflare-one
  - beyondcorp
  - umbrella
category: zero-trust
doc-oficial: https://developers.cloudflare.com/reference-architecture/architectures/sase/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - SASE
  - Cloudflare One
  - Cloudflare SASE
  - Zero Trust Stack
---

# Cloudflare SASE — Cloudflare One

**O que é**: **SASE** (Secure Access Service Edge) é o modelo arquitetural que unifica networking e security em **uma plataforma cloud + plano de controle único**. **Cloudflare One** é a implementação commercial do SASE pela Cloudflare — combina 6+ produtos (Access, Tunnel, WARP, Gateway, CASB, DEX, Email Security, DLP, Magic WAN) num único stack Zero Trust.

> **Esta é a umbrella nota**. Produtos individuais ([[access]], [[tunnel]], [[warp]]) têm notas próprias. Aqui explica **como eles compõem SASE** e **como a pilha implementa [[../../security/beyond-corp|BeyondCorp]]** concretamente.

## O que SASE resolve

Problema que o modelo tradicional não resolve bem:
- Workforce remota/móvel (não há mais "perímetro").
- Apps migrando pra SaaS + cloud (não cabem atrás de VPN corporativa).
- Threats mais sofisticadas (malware, phishing, insider, ransomware).
- VPNs concentram tráfego em poucas regiões → bottleneck + latência.

SASE resposta: **inspecionar tráfego na edge**, perto do usuário, com policies unificadas por identity + device posture + destination.

## Componentes (Cloudflare One stack)

### 🔑 Acesso
- **[[access|Cloudflare Access]]** — ZTNA para apps privados (HTTP/SSH/RDP/VNC/Kubernetes/generic TCP/UDP). Policy engine por app.
- **[[tunnel|Cloudflare Tunnel]]** — conector `cloudflared` que expõe apps internos sem porta inbound. Suporta replicas HA.
- **Secure Web Gateway (SWG)** — inspeção de tráfego outbound (HTTP/DNS). Bloqueia categorias, malware, phishing; DLP inline.

### 💻 Device / Client
- **[[warp|WARP / Cloudflare One Client]]** — agent em device (Win/macOS/Linux/iOS/Android/ChromeOS). Roteia traffic via WireGuard/MASQUE. Coleta **device posture**.
- **Device posture integrations** — CrowdStrike, SentinelOne, Microsoft Intune, JAMF, Kolide, etc.

### 🛡️ Data / App Security
- **CASB (Cloud Access Security Broker)** — escaneia SaaS configs (Google Workspace, Microsoft 365, Salesforce, GitHub, etc.) em busca de misconfigurations + shadow IT.
- **DLP (Data Loss Prevention)** — detecta PII, credenciais, IP em trânsito. Custom profiles.
- **Remote Browser Isolation (RBI)** — renderiza site suspeito em container Cloudflare; cliente recebe só stream.
- **Email Security** (antigo Area 1) — inline + API mode; phishing + BEC + malware.

### 🌐 Network
- **Magic WAN** — IPsec/GRE tunnels de branches/datacenters.
- **Magic Transit** — DDoS + L3/L4 para IPs inteiros (BGP anycast).
- **Network Interconnect (CNI)** — peering direto com Cloudflare (AWS Direct Connect, Megaport, Equinix).

### 📊 Observability
- **DEX (Digital Experience Monitoring)** — latência/saúde por usuário/device; testes sintéticos periódicos.
- **Audit Logs** — trilha completa de decisões de policy.
- **Analytics + GraphQL API** — queries ad-hoc.

## User flows canônicos

### 1. Remote worker → app interno

```
┌──────────────┐           ┌──────────────────────────┐           ┌──────────────┐
│ User device  │           │ Cloudflare edge (PoP)     │           │ Corp network │
│ (WARP client)│──QUIC/WG──▶│ • IdP authenticate        │           │              │
│              │           │ • Device posture check     │           │ ┌──────────┐ │
│              │           │ • Access policy eval       │──Tunnel──▶│ │App       │ │
│              │           │ • DLP inline inspection    │           │ │(:3000)   │ │
│              │           │ • Log audit                │           │ └──────────┘ │
└──────────────┘           └──────────────────────────┘           └──────────────┘
```

**Zero** porta inbound corporativa. Tunnel outbound-only do cloudflared. Cada request re-avalia policy.

### 2. Internet browsing via Gateway

```
┌──────────────┐           ┌──────────────────────────┐
│ User device  │           │ Cloudflare edge            │
│ (WARP client)│──────────▶│ • DNS filter (category)    │
│              │           │ • HTTP inspection (MITM     │
│              │           │   com cert managed)         │
│              │           │ • Malware/phishing block    │
│              │           │ • DLP inspection outbound   │
│              │           │ • RBI para suspicious       │
│              │           │ • Audit log                 │
└──────────────┘           └───────────┬────────────────┘
                                       │
                                       ▼ (se aprovado)
                                  Internet
```

Cliente opera em **home wifi, hotel, airport** com mesma policy do escritório.

### 3. SaaS misconfig detection (CASB)

CASB scanneia tenants SaaS conectados (OAuth/token) periodicamente. Alerta admin de: share públicos, permissions excessivas, MFA desligado, contas inativas. **Não é proxy** — é análise out-of-band via APIs oficiais.

## BeyondCorp → Cloudflare One mapping

Ver também [[../../security/beyond-corp|beyond-corp]] para princípios conceituais.

| [[../../security/beyond-corp|BeyondCorp princípio]] | Implementação Cloudflare One |
|---|---|
| **Identidade forte em toda request** | IdP integration (OIDC/SAML via [[access]]); [[../../security/mfa-step-up]] enforcement |
| **Postura do device como sinal** | [[warp]] client + posture integrations (CrowdStrike, Intune, etc.) |
| **Autorização granular (ABAC)** | Access policies por app com user + device + network + time + risk |
| **Access Proxy (GAP equivalente)** | Cloudflare edge + Tunnel — cada request passa por policy engine |
| **Trust tiers (managed/BYOD/external)** | Device posture signals + WARP enrollment status |
| **Device inventory** | WARP enrollment + device registry + integration MDM |
| **Acesso sem VPN** | ZTNA via [[access]] + [[tunnel]] substitui VPN |
| **Assume breach** | DLP inline, audit trail completo, RBI, CASB monitoring |
| **Single control plane** | Cloudflare Zero Trust dashboard unifica todos os produtos |

## SASE vs VPN tradicional

| Dimensão | VPN tradicional | SASE (Cloudflare One) |
|---|---|---|
| **Post-auth access** | Broad — user entra na rede inteira | Per-app, per-request |
| **Bottleneck** | Concentrator central | Edge anycast 330+ PoPs |
| **Device posture** | Geralmente não | First-class signal |
| **Internet traffic** | Split tunnel ou full routing | SWG filter + DLP |
| **Operation** | Open port UDP/500 IKE | Outbound-only tunnels |
| **Revocation** | Lenta (TLS session) | Imediata (policy update) |
| **Audit** | Limitado (kernel logs) | Completo por request |
| **Latência** | Depende da concentrator | Sub-50ms PoP próximo |

## Quando adotar Cloudflare One (vs alternativas)

**Prefira CF One quando**:
- Workforce distribuída globalmente.
- Apps mix de privados (atrás de firewall) + SaaS.
- Quer consolidar 3+ produtos (VPN + antivirus gateway + CASB + email security).
- Já usa CDN Cloudflare (network footprint já existe).

**Alternativas / concorrentes**:
- **Zscaler ZIA/ZPA** — enterprise-grade, UX mais madura em grandes corps.
- **Netskope** — foco em CASB + DLP.
- **Palo Alto Prisma Access** — stack completo, mais pesado operacionalmente.
- **Cisco Duo + Umbrella** — se já Cisco-shop.
- **Twingate, Tailscale** — ZTNA leve, foco P2P; menos funcionalidades SWG/DLP.

## Padrões operacionais

### Enrollment gradual (IT-led)

1. Pilot team (10-20 users) instalar WARP.
2. Conectar IdP (OIDC/SAML).
3. Migrar 1 app legacy de VPN → Access + Tunnel.
4. Rodar em paralelo 30d (comparar experience).
5. Ampliar 1 dept/semana.
6. Desativar VPN concentrator quando coverage 100%.

### Policy design (ZTNA)

Ver [[access]] §Policies. Regra: **deny-by-default**, policies explícitas por app.

### Device posture minimum baseline

- OS version ≤ N-2 (ou N-1 em high-risk).
- Disk encryption enabled.
- Endpoint protection installed + updated.
- WARP enrolled + active.
- Screen lock policy.

### Split-DNS + tunnel routes

Decide: o que vai pelo túnel vs direto pra internet. "Split tunnel" padrão em apps corp; full tunnel em high-compliance.

## Anti-patterns

1. **Habilitar tudo "de cara"** — Cloudflare One tem 10+ produtos; começar simples (Access + Tunnel) e expandir. Rollout gradual.
2. **Confiar só em WARP** sem device posture — WARP enrollment sem posture check é apenas túnel. Use posture checks para policy.
3. **Policies muito frouxas** — "allow all employees" esvazia Zero Trust. Grupos granulares.
4. **Sem fallback quando IdP indisponível** — IdP down = ninguém loga. Multi-IdP ou break-glass.
5. **Bypass policy via IP whitelist sem audit** — cross-check com WARP. Whitelist por IP isolado é perigoso.
6. **Não integrar com endpoint security** — perder o "device posture" signal do CrowdStrike/Intune.
7. **Mesmo policy para managed e BYOD** — separar trust tiers.

## Referências oficiais (Cloudflare Reference Architecture)

| Documento | URL |
|---|---|
| SASE Reference Architecture | `https://developers.cloudflare.com/reference-architecture/architectures/sase/` |
| CF SASE + CrowdStrike | `/reference-architecture/architectures/cloudflare-sase-with-crowdstrike/` |
| CF SASE + SentinelOne | `/reference-architecture/architectures/cloudflare-sase-with-sentinelone/` |
| CF SASE + Microsoft | `/reference-architecture/architectures/cloudflare-sase-with-microsoft/` |
| ZTNA Access Policies design guide | `/reference-architecture/design-guides/designing-ztna-access-policies/` |
| VPN → ZTNA migration | `/reference-architecture/design-guides/network-vpn-migration/` |
| Zero Trust for SaaS | `/reference-architecture/design-guides/zero-trust-for-saas/` |
| Zero Trust for startups | `/reference-architecture/design-guides/zero-trust-for-startups/` |

Todos consultada 2026-04-24. Ver hub completo em [[_moc#Reference-Architectures]].

## Relações

- **Princípio conceitual**: [[../../security/beyond-corp|BeyondCorp]] (Google papers 2014-2016) — este produto operacionaliza
- **Componentes destilados**: [[access]], [[tunnel]], [[warp]]
- **Componentes lazy**: `gateway.md`, `casb.md`, `dlp.md`, `email-security.md`, `magic-wan.md`, `magic-transit.md`, `dex.md` (destilar quando projeto demandar)
- **Cross-security**: [[../../security/iam]], [[../../security/identity-provider]], [[../../security/mfa-step-up]]
- **Alternativas**: [[../zitadel/_moc]] (IdP apenas; não SASE completo)

## Fontes

- [Cloudflare SASE Reference Architecture](https://developers.cloudflare.com/reference-architecture/architectures/sase/) (consultada 2026-04-24)
- [Cloudflare One docs](https://developers.cloudflare.com/cloudflare-one/) (consultada 2026-04-24)
- [Gartner SASE definition (originou o termo 2019)](https://www.gartner.com/en/information-technology/glossary/sase) (consultada 2026-04-24)
- [NIST SP 800-207 Zero Trust Architecture](https://csrc.nist.gov/publications/detail/sp/800-207/final) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[access]]
- [[tunnel]]
- [[warp]]
- [[overview]]
- [[../../security/beyond-corp]]
- [[../../security/iam]]
- [[../../security/identity-provider]]
- [[../../security/mfa-step-up]]
