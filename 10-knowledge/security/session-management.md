---
title: Session Management — server-side sessions, rotation, revocation
type: concept
tags:
  - security
  - session
  - session-management
  - authentication
  - cookies
  - token
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Session Management
  - Sessions
---

# Session Management

**O que é**: como a aplicação mantém **estado de autenticação** entre requests HTTP (stateless). Sessões são a ponte entre login (authn) e cada request subsequente. Design errado → XSS rouba tudo, CSRF age como user, session fixation pega conta.

> Nota operacional aplicada (1-device policy, HMAC checks, idempotency) vive em [[security-principles#Sessão-1-Device-Policy]]. Aqui fica o **modelo conceitual** + padrões de rotação/revogação/fixation.

## Três modelos de session

### 1. Server-side session (session store)

```
Cookie: session=<random-id>   (httpOnly, secure, sameSite)
        │
        ▼
Server-side store (Redis, DB)
  key: session-id
  value: { userId, createdAt, lastSeenAt, fingerprint, ... }
```

- **Prós**: revogação trivial (DELETE key); dados sensíveis ficam no server.
- **Contras**: round-trip ao store em cada request; precisa HA no store.

### 2. Stateless token (JWT em cookie)

```
Cookie: session=<JWT>   (httpOnly, secure, sameSite)
        │
        ▼ (validação offline via assinatura)
Server decoda claims (sub, exp, ...)
```

- **Prós**: zero round-trip; simples de escalar horizontal.
- **Contras**: revogação difícil (até exp); dados no token (payload leak).

### 3. Híbrido (token + short session)

```
Cookie: session=<JWT-curto-15min>
Cookie: refresh=<opaque-refresh-token>  (diferente path)
```

- JWT curto (15min) valida offline → rápido.
- Refresh token de longa duração (dias/semanas) rotaciona via endpoint server-side (pode revogar).
- Middle ground entre os dois extremos.

**Regra 2026**: híbrido ou server-side. JWT puro com TTL de dias é antipattern (sem revogação).

## Rotação

### Session fixation defense

Ataque: atacante planta session ID conhecido na vítima (via link, XSS parcial); quando vítima loga, atacante usa mesmo ID.

**Mitigação**: **regenerate session ID ao login** (e em qualquer elevação de privilégio).

```ts
app.post('/login', async (c) => {
  const { email, password } = await c.req.json()
  const user = await authenticate(email, password)

  // Destruir sessão pré-login (se houver)
  await sessionStore.destroy(c.get('sessionId'))
  // Criar nova com ID novo
  const newId = await sessionStore.create({ userId: user.id, ... })
  c.cookie('session', newId, cookieOpts)

  return c.json({ ok: true })
})
```

### Rotação periódica

- Sessão de 30d sem rotação = vetor de token stolen.
- **Sliding window**: cada request reseta `lastSeenAt`; expira após X min de inatividade.
- **Hard max**: sessão máxima 30d mesmo com atividade (re-login obrigatório).
- **Rotação silenciosa**: cada N requests, gera novo session ID (migra estado).

### Refresh token rotation (JWT híbrido)

Cada refresh → novo RT + invalida anterior (OAuth 2.1 §6.2). Se RT antigo for usado após, **família** é revogada (sinal de token stolen).

## Revogação

### Server-side session

```ts
await redis.del(`session:${id}`)
```

Revogação imediata. Next request → 401.

### JWT

Precisa **deny list** (blacklist) consultada em cada request:
```ts
const denied = await redis.get(`deny:${jti}`)
if (denied) return c.text('revoked', 401)
```
Cache server-side; sync entre instâncias. Latência adicional.

### Revogar **todas** as sessões do user (mudança de senha, breach)

```ts
await redis.del(`sessions:${userId}:*`)      // server-side
await tokenVersion.increment(userId)          // JWT: claim tokenVersion no JWT; check
```

JWT com `claim: tokenVersion` no banco; middleware compara `jwt.tokenVersion == user.tokenVersion`.

## Fingerprint + 1-device policy

Ver [[security-principles#Sessão-1-Device-Policy]] para implementação concreta.

- Request trás `X-Device-Fingerprint` header.
- Server compara com fingerprint na session.
- Mismatch → 401 + audit log.
- 1-device policy: segundo login **invalida** primeiro.

## Idle timeout + absolute timeout

| Timeout | Semântica |
|---|---|
| **Idle timeout** | Inatividade por X min → expira. 15-30min padrão apps normais; 5min apps sensíveis. |
| **Absolute timeout** | Independente de atividade → re-login após X horas/dias. 8-24h apps normais; 30d streaming/consumer. |

Ambos recomendados (OWASP Session Management Cheat Sheet).

## Session storage scaling

### Redis (mais comum)

- TTL nativo; rápido.
- Cluster para HA.
- Persist para durabilidade (não essencial; se perder, users re-logam).

### DB (Postgres / MongoDB)

- Consistent com resto do data; slower.
- Boa para apps pequenas ou quando já tem DB.

### Cookie-only (Iron Session / Next.js iron-session / encrypted JWT)

- Estado serializado + criptografado no cookie.
- Zero server state.
- Limite ~4KB por cookie.
- Bom pra CDN/edge workers sem store central.

## Multi-region / distribuído

- **Server-side session**: store regional + replication (Redis Cluster, DynamoDB Global Tables).
- **JWT**: zero state → nativamente distribuído (cuidado com deny list sync).
- **Edge** (Cloudflare Workers): use KV + Cache API tier ([[../providers/cloudflare/patterns|CF patterns]]).

## Padrão canônico completo

```ts
interface Session {
  id: string            // random 256-bit
  userId: string
  createdAt: number
  lastSeenAt: number
  fingerprint: string
  ipHash: string
  mfaVerified: boolean
  mfaVerifiedAt: number
}

const COOKIE_OPTS = {
  httpOnly: true,
  secure: true,
  sameSite: 'lax' as const,
  path: '/',
  maxAge: 30 * 24 * 60 * 60, // 30d
}

async function createSession(userId: string, req: Request): Promise<string> {
  const id = randomBytes(32).toString('base64url')
  const session: Session = {
    id, userId,
    createdAt: Date.now(),
    lastSeenAt: Date.now(),
    fingerprint: req.headers.get('X-Device-Fingerprint') ?? '',
    ipHash: hashIP(req.headers.get('CF-Connecting-IP') ?? ''),
    mfaVerified: false,
    mfaVerifiedAt: 0,
  }
  await redis.setex(`session:${id}`, 30 * 24 * 60 * 60, JSON.stringify(session))
  return id
}

async function validateSession(sessionId: string, req: Request): Promise<Session | null> {
  const raw = await redis.get(`session:${sessionId}`)
  if (!raw) return null
  const s: Session = JSON.parse(raw)

  // Idle timeout
  if (Date.now() - s.lastSeenAt > 30 * 60 * 1000) { await redis.del(`session:${sessionId}`); return null }

  // Fingerprint check
  const currentFp = req.headers.get('X-Device-Fingerprint') ?? ''
  if (s.fingerprint !== currentFp) {
    await redis.del(`session:${sessionId}`)
    // audit log (fingerprint mismatch)
    return null
  }

  // Sliding window
  s.lastSeenAt = Date.now()
  await redis.setex(`session:${sessionId}`, 30 * 24 * 60 * 60, JSON.stringify(s))
  return s
}
```

## Anti-patterns

1. **Session ID previsível** (sequential IDs, MD5 of timestamp) — use `crypto.randomBytes(32)`.
2. **Cookie sem httpOnly** — XSS rouba.
3. **Cookie sem Secure em HTTPS** — MITM.
4. **SameSite=None** sem necessidade — CSRF. Padrão `Lax`. Ver [[cookies]].
5. **Session sem rotação** — 30d sem mudança = backdoor.
6. **Expor session ID em URL** (`?session=...`) — referrer leak, logs. **Sempre** cookie.
7. **Sem fingerprint check** — token stolen → logou em outro device sem sinal.
8. **Sem logout server-side** — só remover cookie não mata sessão.
9. **1 session ID para N subdomínios sem isolation** — comprometido em `www.example.com` vaza para `admin.example.com`.
10. **Password reset sem invalidar sessões** — user reseta por suspeita de roubo; atacante continua com sessão ativa.
11. **Session ID como bearer em API** — use Authorization header com JWT; sessão via cookie é para browser.

## OWASP A07 ligação

Session management é central em OWASP A07 Authentication Failures:
- ID previsível → captura.
- Sem rotação → token stolen dura.
- Sem absolute timeout → perpetual sessions.
- Cross-subdomain leak → privilege escalation.

## Como grandes empresas fazem

- **Google**: sessão no cookie `SID/SSID/HSID`; sub-cookies por serviço; absolute 60d + step-up periódico.
- **Microsoft Entra ID**: sessão central + Conditional Access re-avalia em cada request via risk signals.
- **Stripe**: Dashboard session 12h absolute; dashboard sensíveis re-auth a cada 5min (step-up).
- **GitHub**: session revogação via "Log out everywhere"; GitHub Enterprise integra SCIM offboarding.
- **Slack**: session per-workspace; enterprise → SSO session piloted.

## Relações

- **Cookie técnica**: [[cookies]] (atributos, CHIPS, prefixes)
- **Storage de credential client**: [[credential-storage]]
- **CSRF defense**: [[csrf-defense]]
- **Token format**: [[jwt]] (se JWT em cookie)
- **Revogação + logout SSO**: [[sso]] §SLO, [[oidc]] §logout
- **MFA state**: [[mfa-step-up]]
- **Regra operacional aplicada**: [[security-principles#Sessão-1-Device-Policy]]
- **OWASP**: [[owasp-top-10]] A07
- **Runtime antipattern**: [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]]

## Fontes

- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) (consultada 2026-04-24)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) (consultada 2026-04-24)
- [NIST SP 800-63B §7 Session Management](https://pages.nist.gov/800-63-3/sp800-63b.html) (consultada 2026-04-24)
- [iron-session (Next.js cookie-based)](https://github.com/vvo/iron-session) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[cookies]]
- [[credential-storage]]
- [[csrf-defense]]
- [[jwt]]
- [[mfa-step-up]]
- [[security-principles]]
- [[owasp-top-10]]
