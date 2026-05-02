---
title: OP-024 — Rate limit ausente em endpoint caro ou exposto
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - security
  - rate-limit
  - dos
id: OP-024
severity: High
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Missing rate limit
  - DoS via unrestricted endpoint
---

# OP-024 — Rate limit ausente em endpoint caro ou exposto

**Severidade**: High
**Categoria**: Segurança / Disponibilidade

## Sintoma

Endpoints públicos (`/auth/login`, `/auth/register`, `/search`, `/webhooks/*`, qualquer GET pesado) sem rate limiter. Alvo fácil de: brute force credencial, scraping, DoS acidental por integração mal feita, exaustão de pool DB, custo explosivo em serviços pagos (OpenAI API, SMS, etc.).

## Por que é problema

- **Brute force credential**: sem throttle, atacante testa 1000 req/s por conta.
- **Cost runaway**: endpoint que dispara LLM/SMS custa direto no provider.
- **DoS auto-inflingido**: cliente JS com bug → loop infinito → tira API do ar.
- **Amplification**: web scraping industrial drena banco sem throttle.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — /auth/login sem rate limit
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body
  const user = await authService.authenticate(email, password)
  // ...
})
```

## Fix preferido — middleware rate limit por escopo

```ts
import { rateLimit } from 'express-rate-limit'

// Auth — restritivo (prevenir brute force)
const authLimiter = rateLimit({
  windowMs: 60_000,        // 1 min
  limit: 60,               // 60 req/IP/min
  standardHeaders: 'draft-7',
  legacyHeaders: false,
  keyGenerator: (req) => ipKeyFromRequest(req),
})

// Protegido — permissivo (UX)
const apiLimiter = rateLimit({
  windowMs: 60_000,
  limit: 300,              // 300 req/userID/min
  keyGenerator: (req) => req.user?.id ?? ipKeyFromRequest(req),
})

// Webhook — **não** aplicar rate limit padrão (provider retry é legítimo)
// key dedicada por signature quando aplicar
app.post('/auth/login', authLimiter, loginHandler)
app.use('/api/v1', apiLimiter)
```

## Defaults canônicos (AGENTS_SPEC §3.20)

| Endpoint | Limit | Key |
|---|---|---|
| `/auth/login`, `/auth/register` | 60 req/min + CAPTCHA após 3 falhas | IP |
| `/api/v1/*` protegido | 300 req/min | userID (fallback IP) |
| Webhooks | sem rate limit global | — |
| `/search` (autenticado) | 60 req/min | userID |
| Endpoints de AI/SMS/outbound pago | conforme custo; alertar em 80% quota | userID + tenant |

## Alternativa

- **Token bucket** (Redis `INCR` + TTL) com burst permitido.
- **API Gateway rate limit** (AWS API Gateway, Cloudflare Rate Limiting) em vez de app.
- **Adaptive rate limiting**: apertar quando sinais de abuse (user-agent anômalo, geolocalização divergente).

## Complementos

- **CAPTCHA após N falhas** de login.
- **Exponential backoff** no cliente legítimo (retorno 429 com `Retry-After`).
- **Fail closed** em endpoint crítico (se rate limiter indisponível, recusar — melhor que brecha).

## Quando tolerar

Endpoint interno não-exposto (bind em `127.0.0.1`), volume naturalmente limitado pelo caller único. Mesmo aí, limit generoso previne loop acidental.

## Relações

- **Patterns**: Token Bucket, Circuit Breaker (par complementar)
- **OPs relacionados**: [[op-023-webhook-sem-hmac]] (webhooks precisam de HMAC, não rate limit padrão), [[op-003-webhook-sem-idempotencia]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-024, §3.20
- OWASP — [API4:2023 Unrestricted Resource Consumption](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/) (consultada 2026-04-24)
- Cloudflare — [Rate limiting best practices](https://developers.cloudflare.com/waf/rate-limiting-rules/) (consultada 2026-04-24)
- RFC 6585 — [Additional HTTP Status Codes (429 Too Many Requests)](https://datatracker.ietf.org/doc/html/rfc6585) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../security/security-principles]]
- [[op-023-webhook-sem-hmac]]
- [[../security/owasp-top-10]]
