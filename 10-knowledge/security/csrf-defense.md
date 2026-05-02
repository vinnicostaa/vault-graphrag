---
title: CSRF Defense — Cross-Site Request Forgery
type: concept
tags:
  - security
  - csrf
  - xss
  - cookies
  - web-security
  - owasp
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - CSRF
  - Cross-Site Request Forgery
  - XSRF
---

# CSRF Defense

**O que é**: proteção contra Cross-Site Request Forgery — ataque onde site malicioso induz browser do user (autenticado em outro site) a fazer requests em seu nome. Defesas modernas combinam `SameSite` cookies + tokens + verificação de origem.

## O ataque clássico

```
1. User autentica em bank.com → cookie session definido.
2. User, em outra aba, visita evil.com.
3. evil.com tem: <form action="https://bank.com/transfer" method="POST">
                   <input name="to" value="attacker">
                   <input name="amount" value="10000"></form>
                   <script>document.forms[0].submit()</script>
4. Browser envia o POST com cookies de bank.com (same-origin cookie attached).
5. bank.com processa transfer se não verificar origem / token.
```

CSRF é **clássico** — explorado desde ~2001. OWASP A08:2013; absorvido em A01 Broken Access Control em 2017/2021.

## Defesas modernas (2024+)

### 1. **SameSite cookie** (primeira linha)

```
Set-Cookie: session=abc; SameSite=Lax; HttpOnly; Secure
```

- `SameSite=Strict`: bloqueia **qualquer** request cross-site.
- `SameSite=Lax` (padrão browsers modernos): bloqueia POST/PUT/DELETE cross-site; permite top-level GET navigation.

**Desde 2020+**: browsers modernos aplicam `Lax` por default em cookies sem `SameSite` → CSRF em POST é largamente mitigado.

**Limites**: não ajuda se usuário legitimamente tem app aberto em subdomain comprometido (site-same).

### 2. **CSRF token** (double-submit cookie)

```
1. Servidor emite token: cookie "csrf_token=xyz" (não httpOnly) + renderiza em HTML/meta.
2. JS do cliente lê cookie e inclui em header "X-CSRF-Token" em cada mutação.
3. Servidor compara cookie vs header — devem bater.
```

**Funciona porque**:
- evil.com não pode ler cookie (same-origin policy).
- evil.com não consegue setar header custom em request cross-origin (CORS simple request restrictions).

```ts
// Servidor
import { randomBytes } from 'crypto'

app.use(async (c, next) => {
  if (['POST','PUT','PATCH','DELETE'].includes(c.req.method)) {
    const cookieToken = getCookie(c, 'csrf_token')
    const headerToken = c.req.header('X-CSRF-Token')
    if (!cookieToken || cookieToken !== headerToken) {
      return c.text('csrf token mismatch', 403)
    }
  }
  await next()
})

app.get('/csrf', (c) => {
  const token = randomBytes(32).toString('base64url')
  c.cookie('csrf_token', token, { secure: true, sameSite: 'Strict', path: '/' })
  return c.json({ csrfToken: token })   // SPA lê e armazena para enviar em header
})
```

### 3. **Origin / Referer header check**

```ts
app.use(async (c, next) => {
  if (['POST','PUT','PATCH','DELETE'].includes(c.req.method)) {
    const origin = c.req.header('Origin') ?? c.req.header('Referer')
    if (!origin || !origin.startsWith('https://app.example.com')) {
      return c.text('forbidden', 403)
    }
  }
  await next()
})
```

- `Origin` (introduzido ~2009): presente em cross-origin + mutation requests. Browser **não permite** JS customizar.
- `Referer`: legado mas ainda presente em maioria dos casos (pode ser stripped por privacy policies).

Complementar a SameSite; **defesa em profundidade**.

### 4. **Sec-Fetch-\* headers** (moderno)

Browsers modernos enviam automaticamente:
- `Sec-Fetch-Site: same-origin | same-site | cross-site | none`
- `Sec-Fetch-Mode: navigate | cors | no-cors | websocket`
- `Sec-Fetch-Dest: document | iframe | script | ...`

Servidor pode rejeitar `Sec-Fetch-Site: cross-site` em endpoints mutators:

```ts
if (c.req.header('Sec-Fetch-Site') === 'cross-site') {
  return c.text('forbidden', 403)
}
```

Vantagem: atacante **não** pode customizar esses headers (browser stripa). Baixo suporte em browsers antigos.

## Padrão canônico combinado

```
┌─────────────────────────────────────────────┐
│ Defense-in-depth CSRF                        │
├─────────────────────────────────────────────┤
│ 1. Cookie session: SameSite=Lax + Secure    │  ← primeira linha
│ 2. Cookie CSRF token: SameSite=Strict        │
│ 3. JS envia token em X-CSRF-Token header    │
│ 4. Server: compare cookie vs header          │
│ 5. Server: check Origin / Sec-Fetch-Site     │
│ 6. Step-up auth em ops críticas              │  ← última linha
└─────────────────────────────────────────────┘
```

Para SPAs modernas com cookie httpOnly: itens 1, 3, 4, 5.

## Apps com JWT em Authorization header (não-cookie)

**CSRF não se aplica**: Authorization header não é enviado automaticamente. Browser só inclui em request same-origin via fetch() explícito → evil.com não consegue adicionar.

**Mas**: `Access-Control-Allow-Origin: *` + `Access-Control-Allow-Credentials: true` é antipattern que pode abrir brecha. Sempre listar origins.

## Webview / mobile WebView

Apps mobile que renderizam web (in-app browser Instagram, Facebook) podem injetar JS → contorna CSRF em algumas situações. Recomendação: evitar auth via webview; use OAuth + redirect para browser externo + Universal Links (iOS) / App Links (Android).

## Anti-patterns

1. **Só SameSite sem token** — ok para 95% dos casos; mas subdomain comprometido ou `SameSite=None` reaparece CSRF. Token é defesa em profundidade.
2. **CSRF token em header Authorization** — mix com JWT confunde. Header dedicado (`X-CSRF-Token`).
3. **Token global único** por user — se vazou (via log, por exemplo), atacante usa até user logout. Prefira token por sessão, rotação periódica.
4. **Token previsível** (timestamp) — use `crypto.randomBytes`.
5. **Cookie CSRF com `SameSite=None`** — defeat propósito.
6. **Sem `Origin` check em websocket upgrade** — WS é CSRF-vulnerable no handshake; validar explicit.
7. **Trust em `Referer` sozinho** — legitimate users têm referrer policies que removem; use Origin primeiro.
8. **Nenhuma proteção** — "SameSite basta" — ok até SameSite=None ser necessário por outra feature.

## Mascaramento de token (BREACH defense)

BREACH é ataque side-channel em compressão HTTPS (2013). Mitigação: **mascarar token diferentemente em cada response**:

```ts
// Token base persistente (sessão)
const secretToken = 'xyz'
// Token enviado ao cliente: XOR com nonce random
const nonce = randomBytes(32)
const maskedToken = nonce + xor(secretToken, nonce)
// Cliente envia maskedToken; servidor descomprime + valida
```

Libs (double-submit-csrf em Rails, csrf-csrf em Node) fazem por default.

## Como grandes empresas fazem

- **GitHub**: CSRF token per form + Origin check + SameSite.
- **Stripe**: dashboard usa cookie httpOnly + token em header + step-up em operações sensíveis.
- **Django**: `@csrf_protect` decorator padrão; double-submit.
- **Rails**: `protect_from_forgery` com masked tokens.
- **ASP.NET**: `[ValidateAntiForgeryToken]` attribute.
- **Next.js / SvelteKit**: built-in CSRF protection para Actions/forms.
- **Shopify**: `X-Shopify-Shop-Api-Call-Limit` além de CSRF; triple-layer.

## Padrão para APIs públicas

APIs acessadas programaticamente (não browser) **não** precisam CSRF — não há cookie ambient. Use:
- Bearer token (Authorization header) — não enviado automaticamente.
- API keys com rotation.
- Rate limit + anomaly detection.

CSRF é **ataque específico de browser com cookie-based auth**.

## Relações

- **Ataque defendido contra**: combinação XSS (rouba cookie) + CSRF (age). Defesa conjunta: [[cookies]] + este + XSS prevention.
- **Cookie mechanics**: [[cookies]] (SameSite, __Host- prefix)
- **Session**: [[session-management]]
- **XSS defense** (complementar): [[security-principles]] §CSP + input validation
- **OWASP**: [[owasp-top-10]] A01 (access control; CSRF é sub-category)
- **Regras operacionais**: [[security-principles#Princípios-inegociáveis]]

## Fontes

- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) (consultada 2026-04-24)
- [MDN CSRF](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks#cross-site_request_forgery_csrf) (consultada 2026-04-24)
- [SameSite cookies explained — web.dev](https://web.dev/articles/samesite-cookies-explained) (consultada 2026-04-24)
- [Sec-Fetch-* — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Site) (consultada 2026-04-24)
- [BREACH attack (2013)](http://breachattack.com/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[cookies]]
- [[session-management]]
- [[credential-storage]]
- [[security-principles]]
- [[owasp-top-10]]
