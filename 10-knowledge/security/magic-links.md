---
title: Magic Links — passwordless via email link assinado
type: concept
tags:
  - security
  - magic-link
  - passwordless
  - email-otp
  - authentication
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Magic Link
  - Email Magic Link
  - Passwordless email
---

# Magic Links

**O que é**: autenticação passwordless via **link assinado enviado por email**. User digita email → recebe link com token curto e TTL → clica → entra logado. Padrão crescente em B2B SaaS (Slack, Notion, Linear, Vercel, PostHog) pela simplicidade de UX.

## Fluxo canônico

```
1. User → App: submete email em /login
2. App gera token curto (random 128-256 bit) + armazena (token, email, created_at, used=false) com TTL 10-15min
3. App → User (email): link https://app.example.com/login/verify?token=<token>
4. User clica no link (mesmo browser ou outro)
5. App → DB: busca token; se válido, não-expirado, não-usado → marca used=true
6. App cria sessão (cookie httpOnly) → redirect /dashboard
```

**Security props**:
- Token **uso único** (marcado `used=true` após primeira verificação).
- TTL curto (10-15min típico).
- Entropia: mínimo 128 bits random.
- Emitir **novo** token a cada request (não reutilizar).

## Padrão de armazenamento

```sql
CREATE TABLE magic_links (
    token       TEXT PRIMARY KEY,                  -- hash do token (não o token direto)
    email       TEXT NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL,
    expires_at  TIMESTAMPTZ NOT NULL,
    used_at     TIMESTAMPTZ,
    ip_address  INET,
    user_agent  TEXT
);

CREATE INDEX idx_magic_links_email_created ON magic_links(email, created_at DESC);
```

**Crítico**: armazenar **hash do token**, não o token. Se banco vaza, token não é diretamente utilizável.

```ts
// Geração
import { randomBytes, createHash } from 'crypto'

const token = randomBytes(32).toString('base64url')        // enviado no email
const tokenHash = createHash('sha256').update(token).digest('hex')  // armazenado

await db.magic_links.insert({
  token: tokenHash,
  email,
  expiresAt: new Date(Date.now() + 15 * 60 * 1000),
})

// Link enviado
const link = `https://app.example.com/login/verify?token=${token}`
```

## Email OTP (variante)

Em vez de link, enviar **código numérico** (6-8 dígitos) que user digita no app. Mesmo mecanismo de token + TTL, mas UX adaptada para mobile (onde trocar de app para email + clicar link volta ao navegador é fricção).

```
Email: "Your code is: 349812 (valid for 10 minutes)"
```

## Vantagens vs desvantagens

### Vantagens
- Zero senha — zero enumeração de usuários, zero credential stuffing, zero reset flow.
- UX simples: 1 campo (email), 1 clique.
- Ótimo B2B (user já confia no email corp).
- Fácil de integrar (no SDK externo necessário além de sender de email).

### Desvantagens
- **Email account é o fator único**. Se comprometido → todas as apps dependentes comprometidas.
- Latência de email (segundos a minutos em serviços lentos).
- Spam/greylisting bloqueiam email de verification.
- Problema em apps offline / com baixa conectividade.
- Phishing possível (link falso via email spoofed — mitigar com SPF/DKIM/DMARC + copy do email).
- Desktop → mobile flow: user no mobile quer link abrir no mobile; link abre no desktop se email cliente é desktop.

## Defesas adicionais

### Device binding

Armazenar fingerprint (user-agent + IP hash) na request inicial; só validar token se abre no mesmo device. Impede: atacante copia link do email → abre em outro device.

Trade-off: quebra flow "pedir link no desktop, abrir no mobile". Algumas implementações aceitam downgrade: "abrir em novo device requer 2º fator".

### Link + OTP combo

Email contém **ambos**: link curto + código. Desktop clica link; mobile copia código. Resolve desktop/mobile mismatch.

### Rate limit agressivo

- 1 magic link por email por 60s.
- Máximo 3 pendings ativos por email.
- Por IP: 5 requests/hour (previne enumeration de emails + spam).

### Bloquear abertura em webview

Impede phishing via in-app browsers (Facebook, Instagram) que podem logar interação. Verificar `navigator.userAgent`; sugerir abrir em browser real.

## Anti-patterns

1. **Token longo (> 1h TTL)** — janela de abuso se email vaza. 15min max.
2. **Token reutilizável** — atacante usa antes, usuário usa depois acha logado. `used_at` obrigatório.
3. **Token em URL sem TLS** — GET parameter em log de proxy. HTTPS obrigatório.
4. **Sem rate limit** — atacante enche inbox de email + spam server sender.
5. **Link sem expiração** — token eterno = backdoor.
6. **Token em plain text no banco** — DB leak = todas contas comprometidas até TTL.
7. **Fallback silencioso para senha** quando user prefer — confusa.
8. **Sem opção de MFA after magic link** — conta com só magic link é 1-factor; exigir MFA para ops sensíveis ([[mfa-step-up|step-up]]).
9. **Enviar dados sensíveis no email** — mesmo "last 4 of card" em email é PII. Mantém email minimal.
10. **Rate-limit só por email (não por IP)** — enumeração: "enter each email from leaked list; see which don't bounce".

## Como grandes empresas usam

- **Slack**: magic link por email para primeira vez; depois senha + MFA ou SSO.
- **Notion**: magic link como padrão (passwordless primary).
- **Vercel**: magic link + GitHub OAuth.
- **Linear**: magic link + Google OAuth.
- **PostHog**: magic link é o primary.
- **Medium**: magic link clássico.
- **Figma**: magic link + SSO enterprise.

Observação: maioria combina com SSO quando cliente é enterprise.

## Padrão canônico (Node)

```ts
import { Hono } from 'hono'
import { randomBytes, createHash } from 'crypto'

const app = new Hono()

app.post('/login', async (c) => {
  const { email } = await c.req.json<{ email: string }>()

  // Rate limit (por email + IP) — ver session-management
  if (await rateLimiter.check(email, c.req.header('CF-Connecting-IP'))) {
    return c.text('too many requests', 429)
  }

  const token = randomBytes(32).toString('base64url')
  const tokenHash = createHash('sha256').update(token).digest('hex')

  await db.magicLinks.insert({
    tokenHash,
    email,
    expiresAt: new Date(Date.now() + 15 * 60 * 1000),
    ipAddress: c.req.header('CF-Connecting-IP'),
    userAgent: c.req.header('User-Agent'),
  })

  await emailProvider.send({
    to: email,
    subject: 'Your login link',
    html: `<p>Click to login: <a href="https://app.example.com/login/verify?token=${token}">Login</a></p>
           <p>This link expires in 15 minutes.</p>
           <p>If you didn't request this, ignore this email.</p>`,
  })

  return c.json({ ok: true })   // nunca confirmar "email existe" — previne enumeration
})

app.get('/login/verify', async (c) => {
  const token = c.req.query('token')
  if (!token) return c.redirect('/login?error=invalid')

  const tokenHash = createHash('sha256').update(token).digest('hex')
  const row = await db.magicLinks.findOne({ tokenHash })

  if (!row || row.usedAt || row.expiresAt < new Date()) {
    return c.redirect('/login?error=expired')
  }

  await db.magicLinks.update({ tokenHash }, { usedAt: new Date() })

  // Cria sessão — ver session-management.md
  const user = await db.users.upsert({ email: row.email })
  const sessionId = await createSession(user.id)
  c.cookie('session', sessionId, { httpOnly: true, secure: true, sameSite: 'lax' })

  return c.redirect('/dashboard')
})
```

## Relações

- **Alternativa a**: senha, [[otp-hotp-totp]]
- **Combina com**: [[mfa-step-up]] (step-up após magic link para ops sensíveis), [[session-management]]
- **Armazenamento**: [[credential-storage]] (cookie httpOnly na sessão criada)
- **Rate limit**: [[../runtime-antipatterns/op-024-rate-limit-ausente|OP-024]]
- **Não é substituto de**: [[webauthn-passkeys]] (phishing-resistant; magic link não é)

## Fontes

- [OWASP — Passwordless auth cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) (consultada 2026-04-24)
- [Auth0 — Magic Links best practices](https://auth0.com/docs/authenticate/passwordless/authentication-methods/email-magic-link) (consultada 2026-04-24)
- [Vercel Auth — magic link docs](https://authjs.dev/getting-started/providers/email) (consultada 2026-04-24)
- [Supabase Auth — magic link](https://supabase.com/docs/guides/auth/auth-email-passwordless) (consultada 2026-04-24)
- [Stytch — magic link docs](https://stytch.com/docs/guides/magic-links) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[mfa-step-up]]
- [[otp-hotp-totp]]
- [[webauthn-passkeys]]
- [[session-management]]
- [[identity-provider]]
- [[owasp-top-10]] — A07 (passwordless ainda é fator único; tratar com cuidado)
- [[../runtime-antipatterns/op-024-rate-limit-ausente]] — rate limit agressivo no endpoint `/login` é obrigatório
