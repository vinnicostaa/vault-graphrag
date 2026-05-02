---
title: JWT — JSON Web Tokens (RFC 7519 + BCP RFC 8725)
type: concept
tags:
  - security
  - jwt
  - jws
  - jwe
  - token
  - authentication
doc-oficial: https://datatracker.ietf.org/doc/rfc7519/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - JSON Web Token
  - JWS
  - JWE
---

# JWT — JSON Web Tokens

**O que é**: formato de **token compacto, URL-safe, auto-contido** (RFC 7519). Codifica claims JSON em 3 segmentos Base64URL separados por `.`. JWT é **formato**, não protocolo — usado em OAuth 2.0 (access tokens/refresh tokens opcionais), OIDC (id_token obrigatório), Cloudflare Access, auth M2M, cookies de sessão curtos.

> JWT é formato de token. **Não** é OAuth. **Não** é OIDC. Aparece dentro deles. Tratar como formato agnóstico.

## Estrutura

```
<header>.<payload>.<signature>
       ▲                 ▲
       └─ Base64URL      └─ Base64URL de HMAC/RSA/ECDSA
          JSON              sobre "<header>.<payload>"
```

**Header** (JOSE):
```json
{"alg":"RS256","typ":"JWT","kid":"key-id-abc"}
```

**Payload** (claims):
```json
{
  "iss":"https://idp.example.com",
  "sub":"user-123",
  "aud":"https://api.example.com",
  "exp":1735690000,
  "iat":1735686400,
  "nbf":1735686400,
  "jti":"unique-token-id",
  "scope":"read:user write:posts"
}
```

**Signature**:
```
HMAC-SHA256( "<header>.<payload>", secret )   // HS256
  OR
RSASSA-PKCS1-v1_5( "<header>.<payload>", private_key )   // RS256
  OR
ECDSA( "<header>.<payload>", private_key )   // ES256
```

## JWS vs JWE

- **JWS** (RFC 7515): signed — confidencialidade **nula**, integridade garantida. Claims são **legíveis** por qualquer um que decode base64. Nunca colocar senha/PII no payload.
- **JWE** (RFC 7516): encrypted — confidencialidade + integridade. 5 segmentos (header.encrypted_key.iv.ciphertext.tag). Raro em OAuth; mais comum em troca M2M sensível.

**Regra 99% dos casos**: JWT = JWS assinado com RS256/ES256. Claims públicos; segurança vem da validação da assinatura.

## Claims registrados (RFC 7519)

| Claim | Nome | Obrigatório? | Significado |
|---|---|---|---|
| `iss` | Issuer | OIDC: sim | Quem emitiu (URL do IdP) |
| `sub` | Subject | OIDC: sim | User ID único |
| `aud` | Audience | OIDC: sim | Quem pode consumir (API URL) |
| `exp` | Expiration | Sempre | Unix timestamp; rejeitar após |
| `iat` | Issued At | Recomendado | Unix timestamp emissão |
| `nbf` | Not Before | Opcional | Unix timestamp; rejeitar antes |
| `jti` | JWT ID | Recomendado | Identificador único (revogação, replay defense) |

## Best Current Practices (RFC 8725)

Regras **não-negociáveis** para uso correto:

### 1. **Sempre verificar `alg`** — nunca aceitar `alg=none`

```ts
// ❌ ATAQUE: remover assinatura + alterar alg para "none"
// Token: eyJhbGciOiJub25lIn0.<payload>.
// Biblioteca vulnerável aceita e dá acesso.

// ✅ Explicitar algoritmos aceitos
const payload = jwt.verify(token, publicKey, { algorithms: ['RS256'] })
```

### 2. **Algorithm confusion attack** — verificar `alg` contra tipo de chave

Atacante pega RS256 (asimétrico) e muda para HS256 (simétrico) usando a **chave pública** como "secret". Se lib não checa tipo da chave vs alg, valida e dá acesso.

Mitigação: **never** aceitar `alg` que não bate com o tipo da chave configurada.

### 3. **Sempre validar `iss`, `aud`, `exp`** (e opcionalmente `nbf`)

```ts
const payload = await jose.jwtVerify(token, JWKS, {
  issuer: 'https://idp.example.com',
  audience: 'https://api.example.com',
  // exp é validado por default
})
```

Validar **audience** previne token emitido para API-A sendo usado em API-B (scope confusion).

### 4. **Clock skew tolerance ≤ 60s**

```ts
await jose.jwtVerify(token, JWKS, {
  clockTolerance: '30s',   // máximo
})
```

Mais que 60s = janela de abuso de refresh tokens stale.

### 5. **`kid` no header + JWKS rotation**

Header declara `kid` (key ID). Lib busca no JWKS endpoint do IdP; suporta rotação sem downtime.

```ts
const JWKS = jose.createRemoteJWKSet(
  new URL('https://idp.example.com/.well-known/jwks.json')
)
// Automaticamente pega chave certa via kid no header
```

Cache JWKS (10-15min típico); refresh em `kid` não encontrado.

### 6. **Rejeitar `alg` fracos**

- `none` — nunca.
- `HS256` com secret curto — fácil de crackear.
- `RS256` com chave < 2048 bits — proibir.
- Preferir `ES256`/`EdDSA` (assinaturas menores, mais rápido).

### 7. **Payload pequeno**

JWTs vão em header Authorization ou cookie. Header HTTP tem limite ~8KB. Cookie limite 4KB. Não incluir todos os dados do usuário; use apenas claims essenciais (sub, role, scope).

## Stateless vs stateful (revogação)

**Problema**: JWT é auto-contido e **válido até `exp`**. Revogar antes do `exp` exige estado server-side.

**Soluções**:
- **Short-lived access tokens** (5-15min) + refresh token rotativo — revogação efetiva ao limitar janela.
- **Deny list / revocation list** (server-side set de `jti` revogados até TTL) — verificar em cada request. Volta a ser "stateful" — use cache rápido.
- **Token introspection** (RFC 7662) — endpoint `/introspect` do IdP retorna ativo/inativo. Latência extra; usar cache 30s-5min com invalidação proativa.

[[op-008-autorizacao-apenas-cache-hit|OP-008]] aplica aqui: cache de JWT valida mas role mudou no banco → stale authz.

## Sender-constrained tokens (RFC 9449 DPoP, RFC 8705 mTLS)

Problema: JWT roubado (MitM, XSS, log leak) é usável por qualquer holder. "Bearer token" = quem carrega, usa.

**Solução**: amarrar token a **chave do cliente**:

- **DPoP** (RFC 9449): header `DPoP: <jwt-assinado-por-key-do-client>` junto com Authorization. Server valida que hash da chave está no claim `cnf` do access token.
- **mTLS-bound tokens** (RFC 8705): client autentica TLS com cert; token contém hash do cert (`cnf.x5t#S256`).

Stripe, GitHub, Cloudflare Access suportam DPoP em fluxos modernos. Use em APIs de alto valor (financial, admin).

## Padrões de uso

### 1. JWT como access token (OAuth 2.0 "self-encoded")

Access token = JWT assinado. API valida offline (sem chamar IdP). Rápido; revogação difícil.

### 2. JWT como id_token (OIDC)

Emitido pelo IdP junto com access token. Claims `sub`, `name`, `email`, `picture`. **Client** usa (não API) — representa quem logou.

Ver [[oidc]].

### 3. JWT em cookie httpOnly de sessão curta

Alternativa a session store server-side. Cookie com JWT curto (15min) + refresh em httpOnly cookie separado. Rotação silenciosa em cada request.

Trade-off: não dá revogação imediata; mas sem servidor de sessão.

### 4. JWT para comunicação M2M (service-to-service)

Worker A emite JWT assinado com chave privada; Worker B valida com chave pública (JWKS compartilhado via config map). Identidade do caller sem OAuth completo.

### 5. JWT em webhook (CF Access + Worker)

Cloudflare Access adiciona header `Cf-Access-Jwt-Assertion` em cada request. Worker valida via JWKS do team. Ver [[../providers/cloudflare/access#4a-Worker-atrás-de-Access]].

## Anti-patterns

- **`alg=none` aceito**.
- **Algorithm confusion** (RS vs HS com chave pública).
- **Payload com PII completa** (email, CPF, endereço) — JWT em log = PII leak.
- **Token longo (dias+)** sem refresh — roubo persiste.
- **Ignorar `aud`** — token de serviço A aceito em serviço B.
- **Assinar HS256 com secret fraco** — crackeável.
- **JWT grande** (>1KB) em cookie — estoura limite; alguns proxies truncam.
- **Refresh token eterno** — cada refresh deve rotacionar (refresh atual invalida anterior); rotação obrigatória em OAuth 2.1 + RFC 9700 §4.14.

## SDKs canônicos

| Linguagem | Lib |
|---|---|
| JS/TS | `jose` (recomendado) ou `jsonwebtoken` (evitar novos projetos) |
| Go | `github.com/golang-jwt/jwt/v5` ou `github.com/lestrrat-go/jwx/v2` (JWKS cache) |
| Python | `python-jose` ou `authlib` |
| Java | `nimbus-jose-jwt` |
| Rust | `jsonwebtoken` crate |

**Escolha**: preferir lib que suporta **JWKS remoto + rotation automática** (evita reinventar cache).

## Grandes empresas (como fazem)

- **Auth0 / Okta / Zitadel / Entra ID / Google**: RS256/ES256 assinado; JWKS rotacionado; access token curto (60min) + refresh rotativo.
- **AWS Cognito**: RS256; regions separadas.
- **Cloudflare Access**: RS256 via JWKS em `https://<team>.cloudflareaccess.com/cdn-cgi/access/certs`.
- **Stripe**: webhook signatures usam **HMAC-SHA256 cru** (header `Stripe-Signature`, formato `t=<timestamp>,v1=<hex>`) — não é JWT/JOSE, é HMAC raw; algumas APIs internas usam tokens JWT-like.

## Relações

- **Usado em**: [[oauth-2]], [[oidc]], [[sso]], [[session-management]]
- **Complementa**: [[pkce]] (fluxo de emissão seguro)
- **Aplicado em**: [[../providers/cloudflare/access|CF Access]], [[../providers/zitadel/_moc|Zitadel]]
- **Antipatterns ligados**: [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]] (cache stale de claims), [[../runtime-antipatterns/op-022-log-com-pii|OP-022]] (JWT em log)
- **Regra operacional**: [[security-principles#OIDC]] (mantém a regra; aqui está o formato)

## Fontes

- [RFC 7519 — JSON Web Token](https://datatracker.ietf.org/doc/rfc7519/) (consultada 2026-04-24)
- [RFC 8725 — JWT Best Current Practices](https://datatracker.ietf.org/doc/rfc8725/) (consultada 2026-04-24)
- [RFC 7515 — JWS](https://datatracker.ietf.org/doc/rfc7515/) (consultada 2026-04-24)
- [RFC 7516 — JWE](https://datatracker.ietf.org/doc/rfc7516/) (consultada 2026-04-24)
- [RFC 9449 — DPoP](https://datatracker.ietf.org/doc/rfc9449/) (consultada 2026-04-24)
- [RFC 8705 — mTLS client authentication and sender-constrained tokens](https://datatracker.ietf.org/doc/rfc8705/) (consultada 2026-04-24)
- [jose (npm)](https://github.com/panva/jose) (consultada 2026-04-24)
- [jwt.io cheatsheet](https://jwt.io/introduction) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[oauth-2]]
- [[oidc]]
- [[session-management]]
- [[security-principles]]
- [[owasp-top-10]] — A02/A07 (cryptographic failures + auth failures)
- [[cve-response-workflow]] — CVEs históricos em libs JWT (ex.: CVE-2015-9235 alg=none)
- [[../providers/cloudflare/access]]
- [[../providers/zitadel/_moc]]
