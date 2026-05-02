---
title: ADR-0040 — Cloudflare como edge primário (CDN/WAF/Pages/R2/Workers/Access/Tunnel) + Railway permanece origin Go/Postgres/Redis
type: adr
tags: [adr, infra, cloudflare, railway, edge, master-sindico, v2, fase-14]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
amends: [0017-railway-initial-aws-m4]
---

# ADR-0040 — Cloudflare edge primário + Railway origin

## Status

`SUPERSEDED` — 2026-04-27 por [[0045-cloudflare-only-stack-deprecate-railway]] (Cloudflare-only, Railway descontinuado por decisão cliente em [[../../STATE#D-138]]).

> **⚠️ ADR HISTÓRICA — não usar como guia operacional.** Mantida para rastreabilidade do raciocínio anterior (Cloudflare edge + Railway origin). A nova arquitetura é **Cloudflare-only**.

### Status original (revogado)
`accepted` — 2026-04-23 (Fase 14, derivado de D-117). **Amenda** (não substitui) [[0017-railway-initial-aws-m4]]. **Complementa** [[0031-email-provider-resend-m1]].

## Contexto

[[0017-railway-initial-aws-m4]] cobre origin (compute Go + Postgres + Redis em Railway M1, AWS ECS M4+). Já citava **Cloudflare R2** para object storage. Não formalizava Cloudflare como camada edge nem prescrevia uso explícito de Pages/Workers/Access/Tunnel/WAF.

Cliente (João) trouxe na sessão de 2026-04-23 (Fase 14):

> *"vamos migrar tudo para Cloudflare + Resend para dev e Cloudflare + Twilio para prod… ou se porver podemos usar somente a cloudflare para o dev"*

Validação técnica realizada (consultando [[../../../../10-knowledge/providers/cloudflare/_moc]] — `doc-consulted: 2026-04-24`):

- **Cloudflare Workers** = V8 isolates → roda **JS/TS/WASM**, **não** Go nativamente. Backend Go atual precisa de container.
- **Cloudflare Containers (beta 2024+)** poderia rodar Go, mas é beta — não-elegível para M1 produtivo.
- **Cloudflare D1** = SQLite gerenciado, **não** Postgres. Migração de schema atual (RLS, partitioning, sqlc, goose) inviável M1.
- **Cloudflare Email Routing** = **apenas inbound** (forward/process via Workers). Outbound transacional **não existe** na CF — depende de Resend/SES (ADR-0031 permanece válido).
- **Cloudflare KV** = eventual consistency (~60s). Não substitui Redis ABAC cache (5min TTL determinístico em ADR-0012).
- **Cloudflare Hyperdrive** = pool de conexões + cache para Postgres externo (Railway/Neon/RDS). Útil para BFF Workers que consultam Postgres origin.

Conclusão: **substituir tudo por Cloudflare é inviável M1**. **Complementar** com Cloudflare na borda é benéfico (zero egress R2, DDoS/WAF padrão-ouro, edge cache, Pages para SolidJS, Access protege admin).

## Decisão

**Cloudflare como camada edge canônica do Master Síndico**. **Railway permanece como origin** (compute Go + Postgres + Redis) em M1. Migração de origin para AWS ECS em M4+ permanece válida ([[0017-railway-initial-aws-m4]] mantida). Cloudflare opera **independente** de quem é o origin (Railway hoje, AWS amanhã).

### Mapa de responsabilidades

```
                          ┌──────────────────────────────────────┐
                          │         CLOUDFLARE EDGE              │
                          │  (DDoS + WAF + CDN + Edge compute)   │
   browser/mobile  ───▶   │                                      │
                          │  Pages: SolidJS web (static + SSR)   │
                          │  Workers: BFF (rate-limit, cache)    │
                          │  Access: protege /admin/*            │
                          │  Email Routing: inbound suporte@..   │
                          │  R2: uploads, vídeos transientes     │
                          │  Turnstile: CAPTCHA (signup, form)   │
                          │  Tunnel: cloudflared expõe origin    │
                          └─────────────┬────────────────────────┘
                                        │ (Tunnel ou Hyperdrive)
                                        ▼
                          ┌──────────────────────────────────────┐
                          │          RAILWAY ORIGIN              │
                          │  Backend Go (Gin)                    │
                          │  Postgres 18 (RLS, partitioning)     │
                          │  Redis 7 (ABAC cache, sessions)      │
                          │  Worker (jobs, sagas)                │
                          └──────────────────────────────────────┘
```

### Cloudflare — produtos in-scope M1

| Produto | Uso M1 | Justificativa |
|---|---|---|
| **CDN** padrão | Cache de assets web/mobile, vídeos públicos (institucionais não-locked) | Zero-egress, latência global |
| **DDoS Protection** | Sempre-on (free tier ilimitado) | Padrão de mercado, sem opex |
| **WAF** | OWASP Top 10 ([[../../../../10-knowledge/providers/cloudflare/_moc#Sobreposições]]) + custom rules para /api/v1 críticos | INV-101..INV-123 (security hardening) |
| **Pages** | Web SolidJS deploy contínuo via Git | Edge functions + preview por branch |
| **R2** | Object storage: avatares, uploads de doc, vídeos curtos, exports LGPD | Zero-egress fees crítico (vs S3 ~$0.09/GB egress) |
| **Workers (BFF)** | Rate-limit por endpoint, edge cache de leituras públicas (timeline público), webhook receivers idempotentes (Stripe), Mux signed URL gen | Reduz carga origin em 40-60% (cache hits + rate shaping) |
| **Access (Zero Trust)** | Protege `/admin/*` (painéis platform_admin + group admin) com SSO + MFA antes de chegar ao origin | [[0026-admin-mfa-m1]] reforçado em camada extra |
| **Tunnel (cloudflared)** | Origin Railway só responde via Tunnel (sem expor IP público). Acessos diretos = drop. | Defense in depth + zero firewall config |
| **Email Routing (inbound)** | `suporte@mastersindico.com.br`, `billing@mastersindico.com.br`, `bounces+resend@`, `bounces+ses@` → Worker processa | Outbound continua via Resend (ADR-0031); inbound era gap |
| **Turnstile** | CAPTCHA em signup, password reset, form Connect Me | Anti-bot sem reCAPTCHA Google (LGPD-friendly) |
| **Logpush** | Workers logs + Access logs → Grafana Loki | Observability ([[0020-opentelemetry-grafana-sentry]]) |

### Cloudflare — produtos out-of-scope M1 (lazy ou nunca)

- **D1**: Postgres origin permanece em Railway. D1 entra somente se houver edge-side data com tolerância eventual (improvável).
- **Durable Objects**: assemblies live (WS) já decididas em [[0011-livekit-sfu]] (LiveKit SFU). DO seria revisão M3+ se quisermos coordenação leve para chats/threads.
- **Cloudflare Containers (beta)**: para Go origin, M3+ avaliação se Railway custo > 3× e Cloudflare GA.
- **Stream**: Mux já decidido em [[0010-mux-video-provider]]. Stream entra só se Mux falhar comercialmente.
- **Workers AI / Vectorize**: scope explicitamente excluído ([[../../../../10-knowledge/providers/cloudflare/_moc]] decisão 2026-04-24).

### Configuração de domínio M1

- **Domínio raiz**: `mastersindico.com.br` (zone Cloudflare).
- **Subdomínios**:
  - `app.mastersindico.com.br` → Pages (web SolidJS)
  - `api.mastersindico.com.br` → Workers BFF → (Tunnel) → Railway Backend
  - `admin.mastersindico.com.br` → Pages (admin panel) atrás de Cloudflare Access (SSO Zitadel)
  - `cdn.mastersindico.com.br` → R2 público (vídeos institucionais não-locked)
  - `mail.mastersindico.com.br` → outbound Resend (ADR-0031); MX = Cloudflare Email Routing para inbound
  - `webhooks.mastersindico.com.br` → Workers handlers idempotentes (Stripe, Resend, Mux)
- **DNS**: Cloudflare DNS (lazy entry no MOC global; uso direto aqui).
- **SSL**: Universal SSL (free) com edge cert Cloudflare; origin Railway com cert Tunnel.

### Estratégia dev/staging/prod

| Env | Edge Cloudflare | Origin |
|---|---|---|
| **dev (local)** | Wrangler local (`wrangler dev`) — Workers/Pages emulado | Railway dev DB OR docker-compose Postgres+Redis local |
| **staging** | Cloudflare zone `staging.mastersindico.com.br` | Railway staging plan |
| **prod** | Cloudflare zone `mastersindico.com.br` | Railway prod plan (M4+ AWS ECS) |

**Migration M4+ de origin**: Cloudflare não muda. Apenas o Tunnel re-conecta a ECS Fargate. Workers BFF não precisa redesplegar. Diferenciador-chave: **Cloudflare = stable layer; origin = movable**.

## Consequências

### Positivas

- **Zero egress fees** em R2 (uploads/exports/vídeos) — economia projetada $200-800/mês conforme volume vs S3 (D-100 backup plan).
- **DDoS/WAF padrão-ouro grátis** — cobertura compliance ([[0016-beyondcorp-security-posture]]).
- **Origin agnóstico** — migração Railway → AWS ECS em M4+ não afeta edge.
- **Defense in depth admin**: `/admin/*` passa por Access (MFA SSO) **antes** de chegar ao Backend Go (que também tem [[0026-admin-mfa-m1]]). Dupla barreira.
- **Workers BFF** absorve picos de leitura (timeline público, video discovery) sem onerar origin Go.
- **Email inbound (`suporte@`, `billing@`) ganha automação trivial** via Workers (cria ticket, processa bounce).
- **CAPTCHA LGPD-friendly** com Turnstile (substitui Google reCAPTCHA — evita transferência internacional desnecessária).
- **Latência global**: web/mobile users em qualquer ponto Brasil (e diáspora) batem em PoP CF próximo.
- **Custo M1 projetado adicional**: ~$5-20/mês Cloudflare (Workers Paid + Access Standard se necessário). Free tier cobre Pages, R2 até 10GB, DDoS, WAF rules básicas, DNS.

### Negativas

- **Aumenta superfície de provider**: + Cloudflare na lista de DPAs (LGPD), de status pages a auditar, de MCPs a manter (14 servers oficiais — [[../../../../10-knowledge/providers/cloudflare/_moc]]).
- **Wrangler/Workers exigem capacitação**: time backend é Go-first; precisa 1 dev fluente em TS/Workers (ou frontend assume BFF).
- **Cloudflare Access é pago** ($3/user/mês Plan Standard). Para 10-20 admins MS, ~$30-60/mês — registrar no orçamento.
- **Tunnel = ponto único de exposição origin**. Se Tunnel falhar, origin inalcançável. Mitigação: 2+ replicas cloudflared, fallback static maintenance page em Pages.
- **Vendor lock no edge**: Workers/Pages/R2 portáveis com refactor (Hono → outro framework, R2 → S3 trivial); Access/Tunnel/Email Routing menos portáveis.
- **DPA Cloudflare** precisa assinatura (LGPD): cobertura via [Cloudflare DPA](https://www.cloudflare.com/cloudflare-customer-dpa/) — **bloqueio pré-M1**.

### Neutras / observações

- ADR-0017 (Railway → AWS ECS M4+) permanece. Cloudflare cobre o **edge**, ortogonal ao plano de origin.
- Mobile Flutter usa mesmo `api.mastersindico.com.br` (Workers BFF → Tunnel → origin Go). Sem app-specific edge config.
- ABAC cache (ADR-0012, Redis 5min) **não migra para KV** (KV é eventual ~60s; ABAC precisa determinismo + invalidação webhook precisa).

## Trade-offs explícitos

- **Não usar D1**: economia de complexidade — Postgres é fonte canônica única; D1 introduz second source-of-truth + sync.
- **Não usar Workers AI / Vectorize**: scope-cut explícito; ML/RecSys determinístico em M1 ([[0023-recsys-deterministic-m1]]).
- **Não usar Stream para vídeos**: Mux mantido ([[0010-mux-video-provider]]) — locked por ADR.

## Alternativas consideradas

1. **Cloudflare full-stack (Workers Containers + D1 + R2)** — recusada: inviável M1 (Containers beta + D1 schema rewrite enorme).
2. **AWS CloudFront + WAF + S3 + Lambda@Edge** — recusada: opex alto sem time AWS-native em M1; perde zero-egress R2.
3. **Fastly + Compute@Edge (WASM)** — recusada: maturidade Brasil inferior; custo + DX similar a Cloudflare.
4. **Apenas Railway, sem edge** — recusada: DDoS/WAF caseiros + sem edge cache = origin sobrecarregado em pico.
5. **Vercel Edge Functions + Pages** — viável para web frontend; recusada como camada única porque Vercel não cobre R2-equivalent zero-egress nem Tunnel para origin não-Vercel.

## Pontos abertos / debts

- **D-INFRA-CONTAINERS**: re-avaliar Cloudflare Containers (GA?) M3+ para mover origin Go.
- **D-INFRA-D1**: avaliar D1 read replica para edge-side leituras (timeline público) — M3+.
- **D-INFRA-MULTI-REGION**: hoje origin é single-region Railway BR. Se cliente enterprise exigir multi-region (ex: PT), Cloudflare já está pronto, origin precisa replicar.
- **D-INFRA-ACCESS-COST**: monitorar custo Cloudflare Access conforme cresce admin team.

## Aplicação progressiva

| Marco | Entrega |
|---|---|
| **pré-M1 (Sprint 10)** | Setup zone CF + DNS + Universal SSL; Pages SolidJS; R2 buckets (uploads, exports, videos); Tunnel cloudflared para origin Railway; DPA assinado |
| **M1 (2026-05-18)** | Workers BFF (rate-limit, webhook receivers); Access protege `/admin/*`; Email Routing inbound `suporte@` + `billing@`; Turnstile em signup/reset |
| **M2** | Logpush → Grafana Loki; Hyperdrive avaliação (se BFF crescer queries Postgres direto); WAF custom rules contra padrões vistos em prod |
| **M3+** | Reavaliar D1 para edge-cached reads; Containers para origin Go; Multi-region |
| **M4+** | Origin migra para AWS ECS — Cloudflare permanece como edge sem mudança |

## Referências

- [[../../../../10-knowledge/providers/cloudflare/_moc]] — catálogo Cloudflare (consultado 2026-04-24)
- [[../../../../10-knowledge/providers/cloudflare/access]] — Zero Trust Access
- [[../../../../10-knowledge/providers/cloudflare/r2]] — object storage
- [[../../../../10-knowledge/providers/cloudflare/pages]] — Pages
- [[../../../../10-knowledge/providers/cloudflare/workers]] — Workers
- [[../../../../10-knowledge/providers/cloudflare/tunnel]] — Tunnel
- [[../../../../10-knowledge/providers/railway/_moc]] — Railway provider
- [[../../../../10-knowledge/security/beyond-corp]] — BeyondCorp posture (Access implementa)
- [[0017-railway-initial-aws-m4]] — origin (amenda esta)
- [[0026-admin-mfa-m1]] — MFA (reforçado por Access)
- [[0031-email-provider-resend-m1]] — outbound email (Cloudflare cobre só inbound)
- [[0039-iam-cloudflare-style-master-sindico]] — IAM espelha mental model Cloudflare; agora infra também
- [[../../STATE]] §D-117
