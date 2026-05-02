---
title: OAuth 2.0 / 2.1 — Authorization framework (RFC 9700 BCP)
type: concept
tags:
  - security
  - oauth
  - oauth2
  - authorization
  - delegation
  - protocol
doc-oficial: https://datatracker.ietf.org/doc/rfc6749/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - OAuth
  - OAuth 2
  - OAuth 2.0
  - OAuth 2.1
---

# OAuth 2.0 / 2.1

**O que é**: framework de **autorização delegada** (RFC 6749, 2012). Permite que uma aplicação (client) acesse recursos em nome de um usuário (resource owner) sem ver a senha dele. **Não é protocolo de autenticação** — para autenticar usuário, use [[oidc|OpenID Connect]] por cima.

> OAuth delega **autorização de acesso a recurso**. OIDC empacota **identidade do usuário**. Confundir os dois é erro #1 em sistemas novos.

**Baseline atual** (consultada 2026-04-24):
- **OAuth 2.1** (`draft-ietf-oauth-v2-1`) — **ainda é draft** (não-RFC), ~draft-15 em 2026; consolida 2.0 + BCP + segurança moderna, mas timeline de publicação final incerta.
- **RFC 9700** (Set/2024) — *Best Current Practice for OAuth 2.0 Security* — **este é o autoritativo publicado**; referência primária.
- Tratar OAuth 2.0 puro (RFC 6749) como **histórico**; RFC 9700 é o estado-da-arte; OAuth 2.1 draft serve como direção.

**Sobre Implicit/ROPC**: RFC 9700 §2.1.2 diz **"SHOULD NOT use Implicit grant"** e §2.4 diz **"MUST NOT use Resource Owner Password Credentials"**. OAuth 2.1 draft remove ambos formalmente. Em projeto novo: nunca.

## 4 papéis (RFC 6749 §1.1)

```
         ┌────────────────┐           ┌────────────────┐
         │ Resource Owner │           │ Authorization  │
         │ (User)         │◀──────────┤ Server         │
         │                │           │ (IdP / AS)     │
         └────────────────┘           └────────────────┘
                  │                            │
                  │                            │ Access Token
                  ▼                            ▼
         ┌────────────────┐           ┌────────────────┐
         │ Client         │──────────▶│ Resource Server│
         │ (App, SPA,     │           │ (API)          │
         │  Mobile)       │           │                │
         └────────────────┘           └────────────────┘
```

- **Resource Owner**: usuário que possui o recurso.
- **Client**: aplicação que quer acessar (SPA, mobile, backend, CLI).
- **Authorization Server (AS)**: emite tokens — IdP (Zitadel, Okta, Auth0, Keycloak, Entra ID).
- **Resource Server (RS)**: API que consome o token — seu backend.

## Tipos de client

- **Confidential client**: pode manter segredo (backend com secret). Usa `client_secret`.
- **Public client**: não pode manter segredo (SPA, mobile, CLI). **Não** usa `client_secret`; usa **PKCE** obrigatório.

## Grants (fluxos)

Em OAuth 2.1, ficam apenas **4 grants ativos**:

### 1. Authorization Code + PKCE (fluxo canônico)

Para SPAs, mobile apps e **qualquer cliente moderno**. Substitui Implicit.

```
Client → AS: GET /authorize?response_type=code
             &client_id=...&redirect_uri=...&scope=...
             &state=<csrf-defense>
             &code_challenge=<hash(verifier)>
             &code_challenge_method=S256

  ↓ User autentica no AS

AS → Client: redirect redirect_uri?code=<auth-code>&state=...

Client → AS: POST /token
             grant_type=authorization_code
             &code=<auth-code>
             &redirect_uri=...
             &client_id=...
             &code_verifier=<verifier>

AS → Client: { access_token, refresh_token, id_token (se OIDC), expires_in }
```

PKCE obrigatório (`code_challenge_method=S256`). Ver [[pkce]].
`state` obrigatório — defende CSRF em redirect. Ver [[csrf-defense]].

### 2. Client Credentials

Para **M2M** (service-to-service sem usuário).

```
Client → AS: POST /token
             grant_type=client_credentials
             &client_id=... &client_secret=...
             &scope=api:read

AS → Client: { access_token, expires_in }
```

### 3. Device Authorization Grant (RFC 8628)

Para dispositivos sem browser (CLI, smart TV, IoT).

```
Device → AS: POST /device_authorization
             client_id=...&scope=...

AS → Device: { device_code, user_code, verification_uri, expires_in, interval }

Device mostra user_code na tela + URL
User abre URL em outro device, loga, digita user_code

Device polls AS/token: grant_type=urn:ietf:params:oauth:grant-type:device_code
AS responde pending até user aprovar; depois entrega access_token.
```

GitHub CLI (`gh auth login`), AWS CLI (`aws sso login`), gcloud, Vercel CLI usam este fluxo.

### 4. Refresh Token

Sobe novo access token sem re-autenticar usuário.

```
Client → AS: POST /token
             grant_type=refresh_token
             &refresh_token=<rt>
             &client_id=...

AS → Client: { access_token, refresh_token (novo — rotação!), expires_in }
```

**Refresh rotation obrigatória** em OAuth 2.1 (RFC 9700 §4.14): cada refresh emite novo RT e **invalida o anterior**. Se client usa RT velho após rotação, AS detecta reuso → revoga sessão inteira (sinal de token roubado).

### Grants deprecated/removidos em OAuth 2.1

- **Implicit** (response_type=token): vulnerável (access token em URL, sem PKCE). **Deprecado**.
- **Resource Owner Password Credentials** (grant_type=password): client coleta senha diretamente; viola a premissa de delegação. **Deprecado**.

**Nunca implementar** em sistemas novos.

## Extensões críticas (2024)

### PAR — Pushed Authorization Requests (RFC 9126)

Client envia params `/authorize` via POST em `/par` primeiro; recebe `request_uri`. Depois redirect com apenas `request_uri`. Previne leak de scope/state em URL.

### JAR — JWT-Secured Authorization Request (RFC 9101)

Params da request `/authorize` viram JWT assinado. Integridade do request flow.

### RAR — Rich Authorization Requests (RFC 9396)

`authorization_details` em lugar de `scope` — objetos estruturados com recursos específicos (ex: "transferência R$ 500 para conta X"). Open Banking usa.

### DPoP / mTLS-bound tokens

Ver [[jwt#Sender-constrained-tokens]].

## Endpoints canônicos

| Endpoint | Descrição |
|---|---|
| `/authorize` | UI para user autenticar + consent |
| `/token` | Troca code → access token; refresh; M2M |
| `/userinfo` (OIDC) | Claims do usuário (OIDC-only) |
| `/introspect` (RFC 7662) | Valida token ativo (para RS) |
| `/revoke` (RFC 7009) | Revoga token |
| `/jwks.json` | Chaves públicas para validar JWT |
| `/.well-known/openid-configuration` | Discovery (auto-config de clients) |

## Como validar tokens (Resource Server)

Duas estratégias:

### A. **Self-contained** (JWT + offline)

- Access token é JWT assinado por AS.
- RS valida assinatura via JWKS (cached).
- Rápido (sem round-trip); revogação requer deny list ou TTL curto.

### B. **Introspection** (RFC 7662)

- RS faz POST `/introspect` em cada request (ou cache curto).
- AS retorna `{"active": true, "sub": "...", "exp": ..., "scope": "..."}`.
- Latência extra; revogação imediata.

**Prática comum**: JWT + cache introspection (ex.: 30s-5min) com invalidação proativa em eventos `TokenRevoked`. Ver [[session-management]].

## Anti-patterns (OWASP A07 + RFC 9700)

1. **Implicit ou ROPC em projeto novo** — usar Authorization Code + PKCE.
2. **`client_secret` em SPA/mobile** — public client não mantém segredo; use PKCE.
3. **redirect_uri match laxo** (`https://app.example.com/*`) — string match exato obrigatório (RFC 9700 §4.1.1 "Authorization Code Injection"); pattern matching é vetor histórico.
4. **`state` ausente** em Authorization Code — CSRF via redirect.
5. **Access token em URL** (fragment, query) — cai em logs, referrer. Use Authorization header.
6. **Refresh token sem rotação** — token vaza + usado indefinidamente.
7. **Scope overly broad** (`scope=*`) — use mínimo necessário (princípio POLP — ver [[least-privilege]]).
8. **Cross-client token sharing** — token emitido para client A usado por B. `aud` check obrigatório.
9. **OAuth para autenticar login** (sem OIDC) — antipattern histórico; access token não prova que usuário logou agora.
10. **Expor `/introspect` sem autenticação** — resource servers legítimos autenticam (Basic, mTLS, private_key_jwt).

## Providers / SDKs

| Provider | Tipo | MCP/SDK |
|---|---|---|
| [[../providers/zitadel/_moc|Zitadel]] | IdP open-source self-host | Go/JS SDKs |
| Auth0 / Okta | SaaS IdP | SDKs oficiais todas linguagens |
| Keycloak | IdP open-source Java | `keycloak-js`, admin API |
| Entra ID (Microsoft) | Enterprise SSO | MSAL |
| Google Identity | Social + workspace | Google Sign-In SDKs |
| [[../providers/cloudflare/access|Cloudflare Access]] | Zero Trust gateway | JWKS via `<team>.cloudflareaccess.com` |

**Libs**:
- JS: `oauth4webapi`, `openid-client` (Node), `@auth/core` (unified)
- Go: `golang.org/x/oauth2`, `zitadel/oidc/v3`
- Python: `authlib`
- Java: Spring Security OAuth Client

## Grandes empresas — padrões observados

- **GitHub OAuth**: Authorization Code + PKCE para GitHub Apps / OAuth Apps modernos. Fine-grained PATs substituindo legacy tokens. Rotação obrigatória.
- **Google**: Accounts-wide (Sign in with Google) via OIDC. `access_token` de ~1h, refresh rotativo. Sensitive scopes exigem verification.
- **Stripe Connect**: OAuth Code flow para link de Connect accounts (platform acessa conta conectada). State obrigatório.
- **Slack**: OAuth 2.0 + state + rotation. Bot tokens separados de user tokens.
- **Cloudflare**: Cloudflare Access usa OIDC com IdP configurável; emite JWT curto ao app protegido.

## Relações

- **Baseline**: [[security-principles#OIDC]] (regras operacionais)
- **Protocolos relacionados**: [[oidc]] (auth em cima de OAuth), [[saml-2]] (alternativa enterprise), [[pkce]] (segurança do grant code), [[jwt]] (formato de token comum)
- **Aplicado em**: [[../providers/zitadel/_moc]], [[../providers/cloudflare/access]]
- **Defende contra**: A07 OWASP
- **Antipatterns runtime**: [[../runtime-antipatterns/op-003-webhook-sem-idempotencia|OP-003]] (webhooks de OAuth eventos), [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]]

## Fontes

- [RFC 6749 — OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/rfc6749/) (consultada 2026-04-24)
- [RFC 9700 — Best Current Practice for OAuth 2.0 Security (Sep/2024)](https://datatracker.ietf.org/doc/rfc9700/) (consultada 2026-04-24)
- [OAuth 2.1 draft](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) (consultada 2026-04-24)
- [RFC 8628 — Device Authorization Grant](https://datatracker.ietf.org/doc/rfc8628/) (consultada 2026-04-24)
- [RFC 9126 — PAR](https://datatracker.ietf.org/doc/rfc9126/) (consultada 2026-04-24)
- [RFC 9101 — JAR](https://datatracker.ietf.org/doc/rfc9101/) (consultada 2026-04-24)
- [RFC 9396 — RAR](https://datatracker.ietf.org/doc/rfc9396/) (consultada 2026-04-24)
- [RFC 7009 — Token Revocation](https://datatracker.ietf.org/doc/rfc7009/) (consultada 2026-04-24)
- [RFC 7662 — Token Introspection](https://datatracker.ietf.org/doc/rfc7662/) (consultada 2026-04-24)
- [OAuth 2 Simplified — aaronparecki.com](https://www.oauth.com) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[oidc]]
- [[pkce]]
- [[jwt]]
- [[least-privilege]]
- [[csrf-defense]]
- [[owasp-top-10]] — A07 Identification/Authentication Failures
- [[../providers/zitadel/_moc]]
- [[../providers/cloudflare/access]]
