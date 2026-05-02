---
title: Cookies — RFC 6265bis, CHIPS, prefixes (__Host-, __Secure-)
type: concept
tags:
  - security
  - cookie
  - http
  - rfc6265
  - chips
  - samesite
  - csrf
doc-oficial: https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - HTTP Cookies
  - Cookie attributes
---

# Cookies — técnica profunda

**O que é**: mecanismo HTTP clássico para estado stateful em protocolo stateless. RFC 6265 (2011); RFC 6265bis (draft ativo 2024-2026) consolida mudanças de browser e adiciona `SameSite`, `Partitioned` (CHIPS), name prefixes, cookie size limits.

> Operações de regras ("cookie httpOnly sempre", "SameSite=Lax") vivem em [[security-principles]] e [[beyond-corp]]. **Esta nota** cobre **os atributos em profundidade** + edge cases (CHIPS, prefixes, size, public suffix list).

## Anatomia de um cookie

```
Set-Cookie: session=<value>; Path=/; Domain=app.example.com; Max-Age=86400;
            Expires=Thu, 01 Jan 2026 00:00:00 GMT; HttpOnly; Secure;
            SameSite=Lax; Priority=High; Partitioned
```

| Atributo | Significado |
|---|---|
| `<name>=<value>` | Dados (1 par por cookie) |
| `Path` | Prefixo path onde cookie é enviado |
| `Domain` | Hostname (+ subdomínios se prefix-matched) |
| `Max-Age=<s>` | TTL em segundos (preferido a Expires) |
| `Expires=<date>` | Timestamp absoluto (legacy) |
| `HttpOnly` | JS **não acessa** (document.cookie) |
| `Secure` | Só HTTPS (omitir em http://) |
| `SameSite=Strict\|Lax\|None` | CSRF defense |
| `Priority=Low\|Medium\|High` | Browser eviction policy |
| `Partitioned` | CHIPS — partition por top-level site |

## Atributos críticos de segurança

### HttpOnly

```
Set-Cookie: session=abc; HttpOnly
```

JS não consegue ler `document.cookie`; XSS rouba **não** funciona. **Obrigatório** para session cookies.

Contra: dificulta debug JS; logs JS não veem. Aceitável.

### Secure

```
Set-Cookie: session=abc; Secure
```

Browser só envia em conexão HTTPS. Protege MITM. **Obrigatório** em produção.

### SameSite

```
Set-Cookie: session=abc; SameSite=Lax
```

| Valor | Comportamento | CSRF? |
|---|---|---|
| `Strict` | Cookie só em mesma origem. Clicar link externo → **não** envia cookie. | Bloqueia CSRF; UX ruim em alguns casos |
| `Lax` (padrão moderno) | Cookie em same-origin + top-level navegação GET (link clicado). | Bloqueia CSRF de POST/PUT/DELETE; permite login flow |
| `None` | Cookie enviado em cross-site requests. **Exige `Secure`**. | Sem defesa CSRF nativa; precisa token |

**Padrão recomendado**: `Lax` para session, `Strict` para action tokens (logout, MFA).

### Browser defaults (2024+)

- **Chrome/Edge** (desde Chrome 80, fev/2020): cookies sem `SameSite` explícito são tratados como `Lax`.
- **Firefox**: Lax-by-default implementado parcialmente; depende de versão/canal — não assumir.
- **Safari**: comportamento próprio via ITP (Intelligent Tracking Prevention); cross-site tracking restringido de forma distinta; não é Lax-by-default puro.
- Em **todos os browsers top-tier**: cookie com `SameSite=None` **sem** `Secure` é **rejeitado**.

**Regra prática**: sempre declare `SameSite` explicitamente — não confie em default.

## Name prefixes — `__Host-` e `__Secure-`

Prefixos canônicos para forçar browser a validar atributos:

### `__Host-`

```
Set-Cookie: __Host-session=abc; Path=/; Secure; HttpOnly; SameSite=Lax
```

Browser **exige** (senão rejeita):
- `Secure`
- `Path=/`
- **Sem** `Domain` attribute (cookie fica no exact hostname; subdomínios **não** vêem)

Uso: **prefira para session cookie crítico**. Impede subdomain cookie tossing.

### `__Secure-`

```
Set-Cookie: __Secure-remember=xyz; Secure; Domain=.example.com
```

Browser exige `Secure`. Permite `Domain` attribute (cross-subdomain).

Uso: cookies compartilhados entre subdomínios.

## CHIPS — Cookies Having Independent Partitioned State

Nova feature (Chrome 114+ stable 2024): cookie **particionado por top-level site** para iframes.

```
Set-Cookie: _cf_embedded=abc; Secure; SameSite=None; HttpOnly; Partitioned
```

Cenário: seu iframe de comentários (embed.example.com) é incluído em N sites diferentes (blog-a.com, blog-b.com). Sem CHIPS, cookie é global; com CHIPS, cada top-level site tem partição isolada.

**Motivação**: 3rd-party cookie phaseout. CHIPS permite funcionalidade cross-site legítima sem tracking.

## Size limits

| Limite | Valor |
|---|---|
| **Max cookie size** (name + value + attrs) | **Mínimo garantido 4096 bytes** por cookie (RFC 6265 §6.1 — browsers devem suportar **pelo menos** isto); na prática, Chrome/Firefox/Safari respeitam o mínimo, muitos aceitam mais. Headers completos limitados por `Max header size` do servidor. |
| **Max cookies por domain** | 50-180 (varia por browser) |
| **Max cookies totais** | ~6000 Chrome, ~3000 Firefox |
| **Max header size total** (servidor) | Geralmente 8KB default |

**Consequência**: JWT grande (> 2KB) em cookie vira problema. Alternativas:
- Sessão server-side com session ID curto no cookie.
- JWT minimal (apenas `sub` + `exp`).
- Dividir dados em múltiplos cookies (complexa).

## Domain + Path matching

### Domain

- `Domain=example.com` → enviado para `example.com` **e** subdomínios (`a.example.com`, `b.example.com`).
- Sem `Domain` attribute → enviado **só** para o hostname exato (mais seguro).
- **Nunca** setar `Domain=` com top-level domain (`Domain=com`) — browser bloqueia via Public Suffix List.

### Path

- `Path=/` → toda a URL do domain.
- `Path=/admin` → apenas `/admin/*`.
- **Prefixo match**; `Path=/api` matches `/api/v1`.

### Public Suffix List (PSL)

Lista mantida pela Mozilla com eTLDs (`.co.uk`, `.com.br`, `*.github.io`). Browser não permite cookies cross-`eTLD+1`.

- `app.example.co.uk` e `other.example.co.uk` → compartilham (`example.co.uk` = eTLD+1).
- `app.example.com.br` + `other.example.com.br` → compartilham.
- `user.github.io` + `other.github.io` → **não** compartilham (`*.github.io` é PSL).

Isso afeta `Domain=.example.com.br` — válido; `Domain=.com.br` — bloqueado.

## Priority

```
Set-Cookie: session=abc; Priority=High
```

Quando browser precisa evictar cookies (passou do limite), prioriza evictar `Low` antes de `High`. **Priority** não é segurança; é resiliência.

## Padrão canônico

### Session cookie (http-only, secure, samesite)

```ts
response.headers.set('Set-Cookie',
  `__Host-session=${sessionId}; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=86400; Priority=High`
)
```

### Remember-me (cross-subdomain)

```ts
response.headers.set('Set-Cookie',
  `__Secure-rme=${rememberToken}; Domain=.example.com; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=2592000`  // 30d
)
```

### CSRF token (accessible to JS via header)

```ts
// CSRF token: cookie acessível a JS (não httpOnly) + header no request
response.headers.set('Set-Cookie',
  `csrf_token=${csrfToken}; Path=/; Secure; SameSite=Strict`   // sem HttpOnly
)
```

Ver [[csrf-defense]] para padrão completo.

### Partitioned 3rd-party cookie (iframe embed)

```ts
response.headers.set('Set-Cookie',
  `_embed=abc; Secure; HttpOnly; SameSite=None; Partitioned`
)
```

## Logout: apagando cookie

```
Set-Cookie: session=; Path=/; Max-Age=0; HttpOnly; Secure; SameSite=Lax
```

Name + Path + Domain precisam **bater** com o Set-Cookie original.

## Anti-patterns

1. **Cookie sem `HttpOnly`** para session — XSS rouba.
2. **Cookie sem `Secure`** em produção HTTPS — MITM em hotspot.
3. **`SameSite=None` sem motivo real** — abre CSRF; use apenas para 3rd-party legítimo + Partitioned.
4. **JWT > 4KB em cookie** — rejeitado/truncado; use sessão server-side ou split.
5. **Session cookie com `Domain=.example.com`** — qualquer subdomínio (inclusive comprometido) lê/escreve. Use `__Host-` prefix.
6. **Cookie usado como state transient** (PKCE verifier em cookie persistente) — prefira sessionStorage ou server-side.
7. **Apagar cookie sem `Path` correto** — original tinha `Path=/admin`, delete com `Path=/` não remove.
8. **Fingerprint/CSRF token em cookie httpOnly** — JS precisa acessar para enviar em header. httpOnly **não**.
9. **Reutilizar session ID após login** — [[session-management|session fixation]]. Regenerate ao login.
10. **Cookie tossing em subdomínio** — `evil.example.com` seta cookie `Domain=.example.com` → afeta `app.example.com`. Defesa: `__Host-` prefix.

## Debug

### Browser DevTools
- Application → Storage → Cookies: vê todos com atributos.
- Network tab: cada request mostra cookies em `Cookie` header.

### Testar cookie em CLI
```bash
curl -v -b "session=abc" https://app.example.com/api
curl -v -c cookies.txt -X POST https://app.example.com/login \
  -d "email=..." -d "password=..."
```

## Como grandes empresas implementam

- **Google**: `__Host-GAPS`, `__Secure-3PSID`, prefixes canônicos; partitioned cookies para Ads; TTL curto com rotation.
- **Stripe**: `__Host-session` para dashboard; 1h absolute; step-up cookie para ops sensíveis.
- **GitHub**: `_gh_sess` server-side session; `__Host-user_session` variant; cross-device tracking via cookie diferente.
- **Cloudflare Access**: `CF_Authorization` (JWT); valida via JWKS em worker.
- **Shopify**: `_secure_session_id` com HttpOnly + Secure; 1h TTL.

## Relações

- **Context maior**: [[session-management]], [[credential-storage]]
- **Defesa**: [[csrf-defense]]
- **Regras operacionais**: [[security-principles#Princípios-inegociáveis]], [[beyond-corp#Anti-patterns-BeyondCorp]]
- **Runtime antipattern**: [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]] (cookies não substituem HMAC em webhook)
- **Storage alternativo**: [[credential-storage]] (comparação com localStorage/etc.)

## Fontes

- [RFC 6265 — HTTP State Management Mechanism](https://datatracker.ietf.org/doc/rfc6265/) (consultada 2026-04-24)
- [RFC 6265bis draft](https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis/) (consultada 2026-04-24)
- [CHIPS (Partitioned cookies) — Chrome docs](https://developers.google.com/privacy-sandbox/cookies/chips) (consultada 2026-04-24)
- [MDN — Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) (consultada 2026-04-24)
- [Public Suffix List](https://publicsuffix.org/) (consultada 2026-04-24)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[session-management]]
- [[csrf-defense]]
- [[credential-storage]]
- [[security-principles]]
- [[beyond-corp]]
