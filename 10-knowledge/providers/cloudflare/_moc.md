---
title: MOC — Cloudflare (Edge Cloud Platform)
type: moc
tags:
  - moc
  - provider
  - cloudflare
  - edge
  - cdn
  - workers
  - zero-trust
category: edge-platform
mcp: 'https://*.mcp.cloudflare.com/mcp'
mcp-servers-count: 14
sdk-official-cli: wrangler v3+
sdk-official-js: '@cloudflare/workers-types (TS) + hono/itty-router (opcional)'
doc-oficial: 'https://developers.cloudflare.com'
doc-consulted: 2026-04-24T00:00:00.000Z
created: 2026-04-24T00:00:00.000Z
updated: '2026-04-24'
aliases:
  - CF
  - Cloudflare Platform
---

# Cloudflare — Edge Cloud Platform

**Categoria**: Edge cloud platform (CDN + edge compute + edge storage + Zero Trust + DNS + WAF + streaming + outros 40+ produtos).

**MCP disponível**: ✅ **14 MCP servers oficiais** em [`cloudflare/mcp-server-cloudflare`](https://github.com/cloudflare/mcp-server-cloudflare) (consultada 2026-04-24). Auth: Streamable HTTP + OAuth.

| MCP Server | Endpoint | Uso |
|---|---|---|
| **Documentation** | `https://docs.mcp.cloudflare.com/mcp` | Doc oficial Cloudflare (research-loop §4) |
| **Workers Bindings** | `https://bindings.mcp.cloudflare.com/mcp` | Storage/AI/compute primitives |
| **Workers Builds** | `https://builds.mcp.cloudflare.com/mcp` | Gerenciar builds |
| **Observability** | `https://observability.mcp.cloudflare.com/mcp` | Logs + analytics |
| **Radar** | `https://radar.mcp.cloudflare.com/mcp` | Internet traffic trends |
| **Container** | `https://containers.mcp.cloudflare.com/mcp` | Sandbox envs |
| **Browser Rendering** | `https://browser.mcp.cloudflare.com/mcp` | Fetch + screenshot + markdown |
| **Logpush** | `https://logs.mcp.cloudflare.com/mcp` | Job health summaries |
| **AI Gateway** | `https://ai-gateway.mcp.cloudflare.com/mcp` | Prompts/responses inspection |
| **Audit Logs** | `https://auditlogs.mcp.cloudflare.com/mcp` | Compliance reports |
| **DNS Analytics** | `https://dns-analytics.mcp.cloudflare.com/mcp` | DNS troubleshooting |
| **DEX** | `https://dex.mcp.cloudflare.com/mcp` | Digital Experience Monitoring |
| **CASB (Cloudflare One)** | `https://casb.mcp.cloudflare.com/mcp` | SaaS misconfig detection |
| **GraphQL** | `https://graphql.mcp.cloudflare.com/mcp` | Analytics via GraphQL API |

> **Regra**: priorizar `docs.mcp.cloudflare.com` sobre WebFetch em docs; priorizar `bindings.mcp.cloudflare.com` ao escrever Workers.

**SDKs / Tooling oficiais**:
- **CLI**: `wrangler` v3+ (deploy + local dev + secrets + bindings)
- **TS types**: `@cloudflare/workers-types` (definições para Workers/KV/R2/D1/DOs)
- **Framework fit**: Hono (preferido), itty-router, Remix on Workers, Next.js on Pages
- **Deploy config**: `wrangler.toml` (ou `wrangler.jsonc` v3+)

**Modelo de preço** (resumo; doc-consulted 2026-04-24):
- **Free tier amplo** em maioria dos produtos (Workers 100k req/dia, R2 10GB + zero egress, Pages ilimitado dentro de limites).
- **Workers Paid** $5/mo: 10M req + 30s CPU por req.
- **R2** $0.015/GB/mo + **zero egress** (principal diferenciador vs S3).
- **D1** pay-as-you-go: leitura/escrita + armazenamento.
- **DOs (Durable Objects)** $5/mo mínimo + compute por chamada.

---

## Quando usar Cloudflare

1. **Edge-first architecture** — latência baixa em múltiplas regiões sem orquestrar infra regional.
2. **Static + dynamic sites globais** (Pages + Workers) sem vendor lock em framework.
3. **Armazenamento de blobs sem egress** (R2) — backup, user uploads, CDN origin.
4. **SQLite distribuído próximo do usuário** (D1) para leitura-pesada com tolerância a sync.
5. **Coordenação stateful na edge** (Durable Objects) — WebSockets, collaborative editing, counters consistentes.
6. **Zero Trust / BeyondCorp** (Access + Tunnel + WARP) — VPN-alternativa para acesso a apps internos.
7. **DDoS protection + WAF + CDN** padrão-ouro para qualquer app público.

## Quando **NÃO** usar Cloudflare

- **Compute long-running** (>30s CPU) — usar Railway/AWS ECS/Fly.io.
- **GPU / ML training** — Workers AI é inference apenas; training vai em AWS/GCP/provedor dedicado.
- **Banco relacional OLTP complexo** (joins multi-tabela pesados, stored procedures) — D1 é SQLite, não Postgres. Usar Hyperdrive ↔ Postgres externo ou [[../railway/_moc|Railway Postgres]].
- **Region pinning compliance** (GDPR data residency estrita em 1 região) — Cloudflare é global-by-default; data locality existe mas com nuances.
- **Casos onde cold start variabilidade mata SLO** — Workers tem cold start sub-50ms mas não determinístico em free tier.

---

## Produtos destilados (piloto — Onda 1+2+3)

### Onda 1 — Fundação edge

- [[workers]] — V8 isolates, HTTP handler, bindings, 10ms free / 30s paid CPU
- [[pages]] — Static + edge functions via Git integration; Workers Pages unified em 2024+
- [[r2]] — S3-compatible, **zero egress**, signed URLs, lifecycle rules

### Onda 2 — Stateful edge

- [[d1]] — SQLite on Workers Platform; read replicas globais; limits 10GB/DB
- [[kv]] — Key-value eventual consistency (60s propagation), boa para config/feature flags
- [[durable-objects]] — Single-threaded coordinator por "location" — WebSockets, rate limiter, counter

### Onda 3 — Zero Trust (Cloudflare One)

- [[sase]] — **Umbrella** — SASE architecture; como os produtos compõem Zero Trust completo; mapping BeyondCorp → Cloudflare One
- [[access]] — Auth aplicações internas sem VPN; SSO + MFA + policies; BeyondCorp implementado
- [[tunnel]] — Cloudflared connector reverse proxy; expor app interno sem abrir porta no firewall
- [[warp]] — Client-side agent (Cloudflare One Client); device posture + WireGuard/MASQUE; fecha a tríade Access+Tunnel+WARP

---

## Produtos em backlog lazy (11)

Política de promoção conforme [[../../../00-meta/STATE|D-vault-017]] (análogo a `20-stacks/`): destilação sob demanda quando 2+ projetos reais usarem OU quando surgir padrão promovível.

| Produto | Categoria | Observação |
|---|---|---|
| `dns.md` | Edge | Autoritativo; integração com Workers |
| `cdn.md` | Edge | **Unificar 3 camadas**: default CDN + Cache Rules + Workers Cache API |
| `waf.md` | Edge | Overlap com [[../../security/owasp-top-10]]; linkar lá |
| `queues.md` | Storage/dados | Event-driven + DLQ nativo |
| `hyperdrive.md` | Storage/dados | Cache + connection pool para Postgres externo |
| `gateway.md` | Zero Trust | Secure Web Gateway (SWG) + DLP inline; complemento [[sase]] |
| `casb.md` | Zero Trust | Cloud Access Security Broker — scan SaaS configs |
| `dex.md` | Zero Trust | Digital Experience Monitoring |
| `magic-wan.md` | Zero Trust / Network | IPsec/GRE tunnels de branches |
| `magic-transit.md` | Network | L3/L4 DDoS + anycast BGP |
| `stream.md` | Mídia | Overlap com [[../mux/_moc]]; linkar quando destilar |
| `images.md` | Mídia | Transform + delivery + signed URLs |
| `workflows.md` | Outros | Durable Workflows (beta 2024+) |
| `email-routing.md` | Outros | Inbound email → Workers |
| `email-security.md` | Outros | Antigo Area 1; phishing + BEC + malware |
| `turnstile.md` | Outros | CAPTCHA; overlap com [[../../security/_moc]] |

**Workers AI / Vectorize / AI Gateway** — **excluído do escopo** por decisão do usuário (2026-04-24).

---

## Sub-docs nucleares

- [[overview]] — modelo mental (V8 isolates ≠ containers ≠ Lambda), eyeball network, global-by-default
- [[integration-guide]] — wrangler setup, bindings, deploy, CI/CD, secrets, envs
- [[patterns]] — **cross-produto apenas**: bindings injection, wrangler env promotion, edge cache + KV combo, DO + WebSocket pattern
- [[antipatterns]] — **cross-produto apenas**: vendor lock-in, cache-layer confusion, R2 egress misassumption, DO hotspot
- [[usage-in-projects]] — status inicial 🔴 não instrumentado

---

## Reference Architectures & Design Guides oficiais

Cloudflare publica documentação arquitetural extensa em `developers.cloudflare.com/reference-architecture/`. Em vez de duplicar, **hub de links curados** abaixo. Consultar direto na fonte; versões evoluem. Todos verificados 2026-04-24.

### Reference Architectures (visão fim-a-fim)

| Documento | Foco | Produtos | Linka em |
|---|---|---|---|
| [SASE](https://developers.cloudflare.com/reference-architecture/architectures/sase/) | Arquitetura Zero Trust completa | Access, Tunnel, WARP, Gateway, CASB, DEX | [[sase]] |
| [Security](https://developers.cloudflare.com/reference-architecture/architectures/security/) | Defesa em camadas | WAF, DDoS, Bot, Access | [[patterns]] |
| [CDN](https://developers.cloudflare.com/reference-architecture/architectures/cdn/) | Content delivery | CDN, Workers, Cache Rules | `cdn.md` (lazy) |
| [Load Balancing](https://developers.cloudflare.com/reference-architecture/architectures/load-balancing/) | Traffic distribution + failover | Load Balancing, Workers | backlog |
| [Magic Transit](https://developers.cloudflare.com/reference-architecture/architectures/magic-transit/) | L3/L4 DDoS + anycast BGP | Magic Transit | `magic-transit.md` (lazy) |
| [Multi-vendor Security](https://developers.cloudflare.com/reference-architecture/architectures/multi-vendor/) | Integração 3rd-party (Zscaler, Netskope, etc.) | Cross-platform | referência |
| [SASE + CrowdStrike](https://developers.cloudflare.com/reference-architecture/architectures/cloudflare-sase-with-crowdstrike/) | EDR integration | WARP + CrowdStrike | [[warp]] |
| [SASE + SentinelOne](https://developers.cloudflare.com/reference-architecture/architectures/cloudflare-sase-with-sentinelone/) | EDR integration | WARP + SentinelOne | [[warp]] |
| [SASE + Microsoft](https://developers.cloudflare.com/reference-architecture/architectures/cloudflare-sase-with-microsoft/) | Entra ID + Intune | Access + WARP + Entra | [[sase]] |
| [Email Security Deployments](https://developers.cloudflare.com/reference-architecture/architectures/email-security-deployments/) | Inline + API modes | Email Security (Area 1) | `email-security.md` (lazy) |
| [AI Security for Apps](https://developers.cloudflare.com/reference-architecture/architectures/ai-security-for-apps/) | LLM app protection | AI Gateway, Firewall for AI | **fora do escopo** (D-vault-020) |

### Design Guides (how-to específicos)

| Documento | Quando usar | Linka em |
|---|---|---|
| [Designing ZTNA Access Policies](https://developers.cloudflare.com/reference-architecture/design-guides/designing-ztna-access-policies/) | Antes de criar policy Access | [[access]] |
| [Network VPN → ZTNA migration](https://developers.cloudflare.com/reference-architecture/design-guides/network-vpn-migration/) | Migração de VPN concentrator | [[tunnel]], [[warp]], [[sase]] |
| [Extend CF benefits to SaaS end-customers](https://developers.cloudflare.com/reference-architecture/design-guides/extending-cloudflares-benefits-to-saas-providers-end-customers/) | SaaS for SaaS (custom domains) | [[pages]], [[workers]] |
| [Leveraging CF for SaaS apps](https://developers.cloudflare.com/reference-architecture/design-guides/leveraging-cloudflare-for-your-saas-applications/) | Build SaaS on CF | [[pages]], [[workers]], [[d1]] |
| [Securely deliver applications](https://developers.cloudflare.com/reference-architecture/design-guides/secure-application-delivery/) | HTTPS + WAF + Bot mitigation | `waf.md` (lazy) |
| [Securing Guest Wireless](https://developers.cloudflare.com/reference-architecture/design-guides/securing-guest-wireless-networks/) | Public WiFi protection | Magic WAN |
| [Streamlined WAF deployment](https://developers.cloudflare.com/reference-architecture/design-guides/streamlined-waf-deployment-across-zones-and-applications/) | Multi-zone WAF | `waf.md` (lazy) |
| [Zero Trust for SaaS](https://developers.cloudflare.com/reference-architecture/design-guides/zero-trust-for-saas/) | SaaS security via CF One | [[sase]] |
| [Zero Trust for startups](https://developers.cloudflare.com/reference-architecture/design-guides/zero-trust-for-startups/) | Startup-scale adoption | [[sase]] |

> **Regra**: citar estes na nota relevante quando aplicável; **não** duplicar conteúdo. CF mantém fresh — nosso vault aponta.

---

## Sobreposições explícitas (ajuste #5 dual-check)

- **WAF** ↔ [[../../security/owasp-top-10]] — mitigations atemporais lá; configuração Cloudflare aqui.
- **Pages/Workers** ↔ [[../railway/_moc]] — ambos são deploy platforms. Railway wins em compute long-running + Postgres; CF wins em edge + zero egress. Decisão por padrão de uso.
- **Access/Tunnel** ↔ [[../../security/beyond-corp]] — teoria e padrão BeyondCorp em security; implementação CF Zero Trust aqui.
- **Access** ↔ [[../zitadel/_moc]] — ambos fazem auth; Access para apps corporativos internos, Zitadel para auth no produto (end-user).
- **Stream** (lazy) ↔ [[../mux/_moc]] — Mux é padrão-ouro para VOD complexo; CF Stream é mais barato para casos simples.
- **Turnstile** (lazy) ↔ [[../../security/_moc]] — CAPTCHA/bot mitigation.

---

## Glossário interno (confusões comuns)

- **Workers** ≠ Lambda. Workers = V8 isolates (leves, compartilhados) com cold start <50ms; Lambda = containers/microVMs isolados com cold start 100ms-segundos.
- **Durable Objects** ≠ container stateful. DOs são **1 isolate por objeto por "location"** — coordenador único por chave, não réplicas.
- **D1** ≠ Postgres. D1 é SQLite gerenciado com read replicas; write goes to primary; sem joins complexos performáticos em escala.
- **KV** ≠ Redis. KV tem eventual consistency (~60s propagação global); **não** usar para session/cache que precisa consistência forte.
- **R2** ≠ S3 literalmente, mas **S3-compatible API**. Diferença-chave: zero egress fees.
- **Pages** — hoje é "Workers Pages" (convergência 2024+); `pages.dev` é o subdomínio de preview.
- **Cloudflare One** — brand umbrella para Zero Trust (Access + Tunnel + WARP + Gateway + CASB + DEX).

---

## Vizinhos no grafo

- [[../_moc]] — raiz providers
- [[../../security/_moc]] — WAF/Turnstile/Access overlap
- [[../../security/beyond-corp]] — Access + Tunnel implementam BeyondCorp
- [[../../security/owasp-top-10]] — WAF mitigations
- [[../railway/_moc]] — PaaS alternativo para compute long-running
- [[../mux/_moc]] — VOD alternativo a Stream
- [[../zitadel/_moc]] — auth end-user (complementar a Access)
- [[../../../00-meta/STATE]] — D-vault-020 (formalização)
- [[../../../30-agents/mcp-integration]] — 14 MCPs Cloudflare

## Fontes

- [Cloudflare Developer Platform](https://developers.cloudflare.com) (consultada 2026-04-24)
- [cloudflare/mcp-server-cloudflare](https://github.com/cloudflare/mcp-server-cloudflare) (consultada 2026-04-24)
- [Cloudflare Agents / MCP docs](https://developers.cloudflare.com/agents/model-context-protocol/) (consultada 2026-04-24)
- [Cloudflare pricing](https://www.cloudflare.com/plans/developer-platform/) (consultada 2026-04-24)

## Regra dura

- Nada aqui é específico de projeto. Se citar nome de projeto concreto, errou de arquivo — vai em `50-projects/<slug>/`.
- Data de consulta da doc (`doc-consulted`) é **obrigatória** por provider. Cloudflare muda release cycle mensal — re-validar antes de cada decisão.
- Produto em backlog lazy **não pode** ser referenciado em código de projeto como "canônico deste vault" até ser destilado.
