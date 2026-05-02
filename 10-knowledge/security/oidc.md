---
title: OpenID Connect (OIDC) — Authentication em cima de OAuth 2.0
type: concept
tags:
  - security
  - oidc
  - openid-connect
  - authentication
  - sso
  - identity
doc-oficial: https://openid.net/specs/openid-connect-core-1_0.html
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - OpenID Connect
  - OIDC
---

# OpenID Connect (OIDC)

**O que é**: camada de **autenticação** construída em cima de [[oauth-2|OAuth 2.0]]. OIDC resolve o que OAuth não: *"quem é o usuário que acabou de logar"*. Spec **OpenID Connect Core 1.0** (publicada fev/2014) — **versão vigente = 1.0 incorporating errata set 2** (15 dez 2023).

> OAuth entrega **access_token** (para chamar API). OIDC adiciona **id_token** (prova quem logou) e endpoint **`/userinfo`** (claims do usuário).

## O que OIDC adiciona sobre OAuth

| Camada | Contribuição |
|---|---|
| **id_token** | JWT com claims de identidade (`sub`, `email`, `name`, `picture`, `auth_time`, `nonce`, `iss`, `aud`, `exp`) — **assinado pelo AS** |
| **`/userinfo` endpoint** | Consulta claims com access_token (útil quando id_token é leve) |
| **Discovery** `/.well-known/openid-configuration` | Auto-config de clients — endpoints, algoritmos suportados, JWKS URI |
| **Standard claims** | Formato padronizado (`given_name`, `email_verified`, `locale`, `address`, etc.) |
| **`nonce`** | Previne replay de id_token (obrigatório no client quando usa id_token) |
| **`acr` / `amr`** | Authentication Context Class + Methods (MFA, hardware key, biometria) |

## Fluxo canônico (Authorization Code + PKCE + OIDC)

```
Client → AS: GET /authorize?
             response_type=code
             &scope=openid profile email              ← `openid` OBRIGATÓRIO
             &client_id=...&redirect_uri=...
             &state=<csrf>&nonce=<replay-defense>
             &code_challenge=<hash>&code_challenge_method=S256

 ↓ User autentica

AS → Client: redirect ?code=...&state=...

Client → AS: POST /token
             grant_type=authorization_code
             code=... & code_verifier=...

AS → Client: {
  "access_token": "...",       ← para chamar API (OAuth)
  "id_token": "<JWT>",         ← prova de identidade (OIDC)
  "refresh_token": "...",
  "token_type": "Bearer",
  "expires_in": 3600
}

Client decoda id_token, valida:
  • Assinatura via JWKS
  • iss = AS esperado
  • aud = client_id esperado
  • exp > agora
  • nonce = nonce enviado
  • azp = client_id (quando há múltiplos audiences)
```

## Scopes OIDC padrão

| Scope | Retorna |
|---|---|
| `openid` | **Obrigatório** — ativa OIDC |
| `profile` | name, given_name, family_name, picture, locale, preferred_username |
| `email` | email, email_verified |
| `address` | address |
| `phone` | phone_number, phone_number_verified |
| `offline_access` | refresh_token (alguns IdPs exigem este scope) |

## id_token vs access_token — nunca confundir

| Característica | id_token | access_token |
|---|---|---|
| Consumido por | **Client** (app) | **Resource Server** (API) |
| Formato | JWT **sempre** | JWT ou opaque (depende do IdP) |
| Propósito | Prova de autenticação | Autorização de acesso |
| Validação | Client valida ao login | RS valida em cada request |
| TTL típico | 5-60min (variável por IdP; Google ~1h, Entra ID configurável) | 5-60min |
| Refresh | Não é renovado; re-login | Via refresh_token |

**Erro #1**: enviar `id_token` no `Authorization: Bearer` para API. **Errado**. API espera access_token.

## Discovery + auto-config

```bash
curl https://idp.example.com/.well-known/openid-configuration
```

Retorna:
```json
{
  "issuer": "https://idp.example.com",
  "authorization_endpoint": "https://idp.example.com/oauth/authorize",
  "token_endpoint": "https://idp.example.com/oauth/token",
  "userinfo_endpoint": "https://idp.example.com/oauth/userinfo",
  "jwks_uri": "https://idp.example.com/.well-known/jwks.json",
  "response_types_supported": ["code", "id_token"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256", "ES256"],
  "scopes_supported": ["openid", "profile", "email", "offline_access"],
  "code_challenge_methods_supported": ["S256"]
}
```

Libs modernas (`openid-client`, `oidc-client-ts`) consomem discovery → zero config manual.

## Logout — OIDC Session Management + Back-channel/Front-channel logout

OIDC tem 3 specs de logout:

### 1. **RP-Initiated Logout** (mais comum)

Client redireciona para `/end_session`:
```
GET /end_session?id_token_hint=<id_token>&post_logout_redirect_uri=...
```
AS invalida sessão e redireciona de volta.

### 2. **Front-channel logout**

AS insere iframes escondidos para cada RP logar fora. Pouco confiável (browsers bloqueando 3rd party frames).

### 3. **Back-channel logout**

AS chama `logout_token` endpoint de cada RP server-side quando user faz logout global. Mais robusto; RP precisa expor endpoint.

## Authentication Context Class Reference (acr)

Claim `acr` indica **força da autenticação**:
- `0` — guest / weak
- `1` — password
- `2` — MFA
- Custom URIs do IdP

**Step-up authentication**: API pede request com `acr >= 2`. Se user só tem `acr=1`, IdP força re-auth com 2FA. Ver [[mfa-step-up]].

## Grandes empresas — como implementam

- **Google Sign-In**: OIDC RP-initiated via `accounts.google.com`. Device flow para Smart TV/CLI. SDK oficial handle PKCE/nonce.
- **Microsoft Entra ID**: OIDC primary; SAML para legacy. MSAL.js/MSAL.NET abstraem tudo.
- **Zitadel**: OIDC + PKCE obrigatório em public clients; introspection + JWT profile; suporta step-up.
- **Okta / Auth0**: Discovery padrão; tenants isolados; hooks customizáveis (pre-token).
- **Keycloak**: OIDC + SAML; realms por organização; brokering com outros IdPs.
- **Apple Sign-In**: OIDC-compatível; privacy-focused (relay email); obrigatório em apps iOS com login social.

## Padrão canônico (Next.js / Node)

```ts
// Usando openid-client
import { Issuer } from 'openid-client'

const issuer = await Issuer.discover('https://idp.example.com')
const client = new issuer.Client({
  client_id: process.env.CLIENT_ID,
  client_secret: process.env.CLIENT_SECRET,
  redirect_uris: ['https://app.example.com/callback'],
  response_types: ['code'],
})

// 1. Redirect para AS
const { authUrl, codeVerifier, state, nonce } = generateAuthUrl(client)
// save codeVerifier, state, nonce in session

// 2. Callback
const params = client.callbackParams(req)
const tokenSet = await client.callback(
  'https://app.example.com/callback',
  params,
  { code_verifier: sessionCodeVerifier, state: sessionState, nonce: sessionNonce }
)

// tokenSet.access_token / .id_token / .refresh_token
const userInfo = await client.userinfo(tokenSet.access_token)
```

## Anti-patterns

- **Esquecer `nonce`** — id_token pode ser replay.
- **Validar id_token sem checar `iss` + `aud` + `exp` + `nonce`**.
- **Confiar em `email_verified` sem entender o IdP** — alguns IdPs sociais nunca marcam como verified.
- **Usar id_token como access_token** (enviar em Bearer para API).
- **Esquecer de invalidar sessão local no logout** — user "logado" mesmo após logout no IdP.
- **Assumir todo OIDC IdP suporta discovery** — alguns legados precisam config manual.
- **Não suportar back-channel logout** em ambiente SSO corporativo — user com sessão zombie.

## Relações

- **Base**: [[oauth-2]] (OIDC estende)
- **Formato de token**: [[jwt]] (id_token é JWT)
- **Fluxo seguro**: [[pkce]]
- **Aplicações**: [[sso]], [[iam]], [[mfa-step-up]] (acr/amr)
- **Regra operacional**: [[security-principles#OIDC-introspection-via-JWT-Profile]] (mantém detalhe operacional)
- **Providers concretos**: [[../providers/zitadel/_moc]] (detalhes implementação), [[../providers/cloudflare/access]] (OIDC gateway)

## Fontes

- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html) (consultada 2026-04-24)
- [OIDC Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html) (consultada 2026-04-24)
- [OIDC Session Management 1.0](https://openid.net/specs/openid-connect-session-1_0.html) (consultada 2026-04-24)
- [OIDC RP-Initiated Logout 1.0](https://openid.net/specs/openid-connect-rpinitiated-1_0.html) (consultada 2026-04-24)
- [OIDC Back-Channel Logout 1.0](https://openid.net/specs/openid-connect-backchannel-1_0.html) (consultada 2026-04-24)
- [openid-client (Node lib)](https://github.com/panva/openid-client) (consultada 2026-04-24)
- [Zitadel OIDC docs](https://zitadel.com/docs/guides/integrate/login/oidc) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[oauth-2]]
- [[jwt]]
- [[pkce]]
- [[sso]]
- [[mfa-step-up]]
- [[owasp-top-10]] — A07 Identification/Authentication Failures
- [[../providers/zitadel/_moc]]
- [[../providers/cloudflare/access]]
