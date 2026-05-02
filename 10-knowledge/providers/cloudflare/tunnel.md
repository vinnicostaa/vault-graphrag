---
title: Cloudflare Tunnel — Reverse proxy persistent sem abrir porta
type: note
tags:
  - provider
  - cloudflare
  - tunnel
  - cloudflared
  - zero-trust
  - reverse-proxy
  - ztna
category: zero-trust
doc-oficial: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Tunnel
  - Cloudflared
  - CF Tunnel
  - Cloudflare Tunnel
---

# Cloudflare Tunnel

**Categoria**: Outbound-only reverse proxy via daemon `cloudflared`.

**Intenção**: expor app interno (running em laptop, VM, container, Kubernetes pod) através de hostname Cloudflare, **sem abrir porta inbound** no firewall. Tunnel abre conexão TLS outbound para Cloudflare edge e recebe tráfego dali.

## Modelo mental

```
User request: admin.example.com
        │
        ▼
┌──────────────────────────────────┐
│  Cloudflare edge (330+ PoPs)     │
│  ┌─────────────────────────┐     │
│  │ [Opcional] Access policy │     │
│  │ [Opcional] WAF / rate lim│     │
│  │ [Opcional] Cache         │     │
│  └────────────┬────────────┘     │
└───────────────┼──────────────────┘
                │ Tunnel (QUIC/HTTP2 outbound persistent)
                │
                ▼
┌──────────────────────────────────┐
│  Your infra (atrás de firewall)  │
│  ┌─────────────────────────┐     │
│  │ cloudflared daemon       │     │
│  │  ↓                       │     │
│  │ localhost:3000           │     │
│  │ (or 192.168.x.x:8080,    │     │
│  │  k8s service, etc.)      │     │
│  └─────────────────────────┘     │
└──────────────────────────────────┘
```

Nenhuma porta inbound aberta. Firewall só precisa permitir outbound 443.

## Quando usar

1. **Expor app interno sem VPN nem port forwarding** — Grafana, Jira, admin dashboards, dev servers.
2. **Staging/preview em infra privada** — team acessa via `staging.example.com` sem configurar VPN.
3. **Home lab / remote work** — expor serviços locais com hostname estável.
4. **ZTNA (Zero Trust Network Access) para servidores on-prem** — combinado com [[access]].
5. **Conexão Kubernetes sem Ingress público** — `cloudflared` como sidecar ou DaemonSet.
6. **SSH / RDP via browser** — Tunnel + Access Infrastructure (beta).

## Quando **NÃO** usar

- **Latência ultra-baixa em protocolo único** (real-time gaming, HFT) — Tunnel adiciona hop; latência é normalmente boa mas não zero.
- **Bandwidth massivo** (vídeo streaming pesado) — considerar R2 + CDN nativo.
- **Apenas HTTP público sem policy** — setup direto CNAME → app é mais simples.

## Padrão canônico

### 1. Instalar cloudflared

```bash
# macOS
brew install cloudflared

# Linux
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb

# Docker
docker run -d --name cloudflared \
  -v /etc/cloudflared:/etc/cloudflared \
  cloudflare/cloudflared:latest tunnel --config /etc/cloudflared/config.yml run

# Kubernetes — Helm chart oficial ou DaemonSet manual
```

### 2. Criar Tunnel (modo "remotely-managed" recomendado)

Dashboard: `Zero Trust → Networks → Tunnels → Create tunnel`.

```bash
# O dashboard gera token — usar no daemon
cloudflared service install <TOKEN>
# Serviço systemd em Linux, launchd em macOS, service em Windows
```

### 3. Configurar rotas (via dashboard remotely-managed)

```
Tunnel: my-admin-tunnel
│
├── Public Hostname
│   admin.example.com → http://localhost:3000
│
├── Public Hostname
│   grafana.example.com → http://10.0.0.5:3000
│
└── Private Network (TCP/UDP não-HTTP)
    CIDR: 10.0.0.0/24 (acessível via WARP client)
```

DNS automaticamente apontado para Cloudflare (CNAME oculto para tunnel endpoint).

### 4. Plugar Access em cima (ZTNA)

Em `Zero Trust → Access → Applications`, criar application com hostname `admin.example.com`. Policies + MFA + posture checks + session duration.

Request flow final:
```
User → admin.example.com → CF edge → Access policy (OK)
                                    → Tunnel → localhost:3000
```

## Modo "locally-managed" (config.yml)

Alternativa para setups avançados:

```yaml
# /etc/cloudflared/config.yml
tunnel: <tunnel-uuid>
credentials-file: /etc/cloudflared/<tunnel-uuid>.json

ingress:
  - hostname: admin.example.com
    service: http://localhost:3000
    originRequest:
      connectTimeout: 30s
      noTLSVerify: false
      originServerName: admin.local   # SNI override
  - hostname: api.example.com
    service: http://localhost:8080
  - service: http_status:404           # fallback obrigatório
```

```bash
cloudflared tunnel route dns <tunnel-name> admin.example.com
cloudflared tunnel run <tunnel-name>
```

## Limites (consultada 2026-04-24)

- **Tunnels por conta**: 1000 (free tier).
- **Hostnames por tunnel**: 1000.
- **Replicas (HA)**: múltiplos `cloudflared` em mesma tunnel para failover/load-balancing automático.
- **Protocolos**: HTTP/HTTPS, TCP, UDP, SSH, RDP, WSS.
- **Bandwidth**: sem limite explícito em free; enterprise tem SLA.

## Pricing (consultada 2026-04-24)

- **Tunnel por si só**: grátis (incluso em Zero Trust free).
- **Access sobre tunnel**: $7/user/mo após 50 users free.

## Antipatterns específicos

- **1 tunnel para N apps não-relacionados** — operacionalmente ruim; 1 falha derruba todos. Prefira 1 tunnel por app crítico, ou grupos lógicos.
- **Sem replica em prod** — single point of failure. Rode 2+ `cloudflared` na mesma tunnel (Cloudflare faz round-robin).
- **Tunnel exposto sem Access** — funciona, mas perde a premissa de ZTNA. Sempre compor com Access para apps internos.
- **TLS origem quebrado** (`noTLSVerify: true` em prod) — atalho que vira dívida. Use cert próprio ou TLS origem via Cloudflare CA.
- **Deixar `cloudflared` rodando em laptop pessoal para app de time** — laptop desliga → tunnel cai. Use VM dedicada, container, ou K8s.
- **Expor `metrics` do cloudflared sem proteção** — porta de observabilidade vaza infos; bind em loopback only.

## Casos especiais

### Kubernetes (DaemonSet)

Pod `cloudflared` em cada node; expõe services internos:

```yaml
ingress:
  - hostname: admin.example.com
    service: http://admin.svc.cluster.local:3000
```

Helm: `helm install tunnel cloudflare/cloudflared --set-file config=./config.yml`.

### Docker Compose (dev)

```yaml
services:
  app:
    image: my-app:dev
    ports: ["3000:3000"]
  tunnel:
    image: cloudflare/cloudflared:latest
    command: tunnel --no-autoupdate run
    environment:
      TUNNEL_TOKEN: ${TUNNEL_TOKEN}
    depends_on: [app]
```

## Relações

- **Combina com**: [[access]] (ZTNA); [[workers]] (Worker como origem via `service: http://worker.example.com`); [[../../security/beyond-corp]]
- **Alternativas**:
  - ngrok (dev/prototype).
  - Tailscale Funnel (P2P + SSO).
  - AWS PrivateLink / VPC peering (para dentro de AWS).
  - Nginx reverse proxy em VPS pública (auto-gerenciada).

## Reference Architectures & Design Guides

- [Network VPN → ZTNA migration](https://developers.cloudflare.com/reference-architecture/design-guides/network-vpn-migration/) — cloudflared como substituto de VPN concentrator (consultada 2026-04-24)
- [Securely deliver applications](https://developers.cloudflare.com/reference-architecture/design-guides/secure-application-delivery/) — padrões de exposição segura (consultada 2026-04-24)

Ver [[sase]] para arquitetura completa.

## Fontes

- [Tunnel docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) (consultada 2026-04-24)
- [cloudflared GitHub](https://github.com/cloudflare/cloudflared) (consultada 2026-04-24)
- [Remote-managed vs locally-managed](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/configure-tunnels/) (consultada 2026-04-24)
- [HA / Replicas](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/deploy-tunnels/deployment-guides/) (consultada 2026-04-24)
- [Kubernetes Helm chart](https://github.com/cloudflare/helm-charts) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[sase]]
- [[access]]
- [[warp]]
- [[../../security/beyond-corp]]
- [[../../security/security-principles]]
