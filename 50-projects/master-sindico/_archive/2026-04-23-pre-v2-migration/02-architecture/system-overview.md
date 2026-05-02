---
title: System Overview — Master Síndico
type: project
tags: [master-sindico, architecture, system-overview]
project: master-sindico
created: 2026-04-22
---

# System Overview — Master Síndico

Visão macro da arquitetura. Backend Go monolítico modular (sem microserviços no momento) + web SolidJS + mobile Flutter + infra Railway.

---

## Diagrama (alto nível)

```
             ┌─────────────────────────────────┐
             │       Clientes                  │
             │ ┌─────────┐ ┌──────┐ ┌───────┐  │
             │ │ Browser │ │  PWA │ │ Mobile│  │
             │ │ (Solid) │ │      │ │(Flutter) │
             │ └────┬────┘ └──┬───┘ └───┬───┘  │
             └──────┼─────────┼─────────┼──────┘
                    │         │         │
                    └────────┬┴─────────┘
                             │ HTTPS (TLS 1.3)
                             ▼
             ┌───────────────────────────────┐
             │    Railway Edge / CDN         │
             └───────────────┬───────────────┘
                             │
                             ▼
             ┌───────────────────────────────┐
             │     API Go (monolito modular) │
             │   cmd/api + internal/modules  │
             │                               │
             │  ┌──────────────┐             │
             │  │  Middleware  │             │
             │  │ (BeyondCorp) │             │
             │  └──────┬───────┘             │
             │         ▼                     │
             │  ┌──────────────┐             │
             │  │   Handlers   │             │
             │  │ (abstração)  │             │
             │  └──────┬───────┘             │
             │         ▼                     │
             │  ┌──────────────┐             │
             │  │  Use Cases   │             │
             │  │    CQRS      │             │
             │  └──┬────────┬──┘             │
             │     ▼        ▼                │
             │  ┌─────┐ ┌───────────┐        │
             │  │Domain│ │ Mappers  │        │
             │  └─────┘ └───────────┘        │
             └─────────┬───────────────────┬─┘
                       │                   │
         ┌─────────────┼───────────────┐   │
         ▼             ▼               ▼   ▼
    ┌─────────┐  ┌─────────┐   ┌──────────────┐
    │  PG 18  │  │ Redis 7 │   │  Providers   │
    │ (sqlc)  │  │ (cache) │   │  externos    │
    └─────────┘  └─────────┘   ├──────────────┤
                               │ Stripe       │
                               │ Mux (vídeo)  │
                               │ Livekit      │
                               │ Zitadel OIDC │
                               │ Twilio SMS   │
                               │ SES / Resend │
                               │ FCM push     │
                               │ OpenSearch   │
                               └──────────────┘
```

---

## Estilo arquitetural

- **Monolito modular** (não microserviços) — simplicidade operacional
- **Clean Architecture** em camadas
- **DDD + CQRS** em cada módulo
- **Vertical slices**: feature nova entra como slice completo (domain → app → infra → http)
- **BeyondCorp adaptado**: zero-trust end-to-end

---

## Fluxo de uma request (típico)

```
1. Cliente faz POST /api/v1/condominiums/:id/units + JWT/cookie + X-Device-Fingerprint
2. Railway edge → API Go (:8000)
3. GinRouter captura → GinAdapter envolve em contracts.Context
4. Middleware chain (ordem rígida):
   a. RequestID (gera se ausente)
   b. Recovery (panic → 500)
   c. CORS (allowlist)
   d. RateLimit (token bucket)
   e. Authenticate (Zitadel introspection)
   f. ABAC (cache Redis 5min)
   g. TenantIsolation
   h. DeviceFingerprint (1-device check)
   i. AuditLog (futuro DT-010)
5. Handler:
   a. DecodeJSON → struct request
   b. Extrai ctx.User()
   c. Monta Command
   d. Chama UseCase.Execute(ctx, cmd)
6. UseCase:
   a. Valida via VO constructors
   b. Orquestra Repository + Provider
   c. UoW ou Saga conforme fluxo
   d. Publica DomainEvent se necessário
7. Repository: sqlc query tipada → mapper → entidade
8. Handler: entidade → DTO (mapper) → JSON response
9. Response: status + headers + body
10. Log estruturado (request_id, tenant_id, user_id, duration)
```

---

## Camadas (Clean Arch)

Ver [[clean-arch-mapping]].

```
┌─────────────────────────────────────┐
│ cmd/api/main.go (DI explícito)      │
├─────────────────────────────────────┤
│ internal/server (GinAdapter)        │
│ internal/shared/middleware          │
├─────────────────────────────────────┤
│ internal/modules/<ctx>/             │
│  ├── domain/  ◄── núcleo            │
│  ├── application/                   │
│  └── infrastructure/                │
├─────────────────────────────────────┤
│ internal/core/contracts             │
│ internal/core/errors                │
├─────────────────────────────────────┤
│ pkg/ (logger, utils, money,         │
│       pagination, safecast)         │
└─────────────────────────────────────┘
```

---

## Módulos e domínios

Ver [[../01-domain/bounded-contexts]].

- `identity` — auth, sessão, 1-device
- `billing` — Stripe, plano, trial, quotas
- `institutional` — condomínio, unidades, timeline, anúncios, eventos, compliance
- `commercial` — Connect Me, empresas, reputação, contratos
- `content` — vídeos Mux, busca, trava 90d
- `assembly` — Livekit, votação, atas

Comunicação entre módulos: eventos de domínio in-process hoje (NATS quando escalar).

---

## Infra (Railway atual)

- 1 instância backend (scalable)
- Postgres 18 managed
- Redis 7 managed
- Buckets S3-compatible pra assets
- Livekit self-hosted (container)
- Zitadel self-hosted (container) — plano migrar pra managed
- NATS JetStream container (quando necessário)

Plano migração AWS (M4+): ECS Fargate + RDS PG18 + ElastiCache + S3 + CloudFront + Route53 + CloudWatch + WAF. Decisão em D-0XX.

---

## Observability

- **Logs**: zerolog structured → stdout → Railway logging → (futuro: CloudWatch ou Loki)
- **Traces**: OpenTelemetry → OTLP → Grafana Tempo ou Jaeger (futuro)
- **Metrics**: OpenTelemetry metrics → Prometheus (futuro Grafana)
- **Errors**: Sentry SDK Go + web + mobile
- **Uptime**: status page externa (BetterStack ou similar)

---

## Segurança

Ver [[../08-security/BEYOND_CORP]] e [[../08-security/threat-model]].

---

## Deploy

- Push to `main` → GitHub Actions CI → Railway staging
- Manual promote → prod
- Smoke test automatizado em cada deploy

Ver [[../06-execution/deploy-cycle]].

---

## Links

- [[_moc]]
- [[clean-arch-mapping]]
- [[deployment-topology]]
- [[data-flow]]
- [[../01-domain/bounded-contexts]]
- [[../03-subprojects/backend/README]]
- [[../08-security/BEYOND_CORP]]
- [[../06-execution/STEPS]]
