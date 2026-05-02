---
title: Zitadel — Integration Guide (setup, SDK, OIDC, M2M)
type: note
tags: [provider, zitadel, identity, oidc, integration-guide]
source: https://zitadel.com/docs
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Zitadel Integration]
---

# Zitadel — Integration Guide

Guia operacional para ligar uma aplicação ao Zitadel. Pressupõe que o modelo de domínio já foi lido em [[overview]].

## 1. Decisão: cloud ou self-hosted

- **Cloud** — criar conta em `zitadel.cloud`, escolher região e slug de tenant → obtém `https://<tenant>.zitadel.cloud` como `issuer`.
- **Self-hosted** — Docker Compose ou Helm chart oficial (`zitadel/zitadel`). Pré-requisitos: Postgres 14+, TLS terminado na borda, domínio custom com DNS verificado. Primeira subida usa o `zitadel admin` CLI para setar o `IAM_OWNER`.

Em ambos os casos, o `issuer` que aparece em `/.well-known/openid-configuration` é a **única** fonte de verdade para a aplicação consumidora.

## 2. Criar Project e Applications

1. Entrar no console como `IAM_OWNER` (cloud) ou via `zitadel admin` (self).
2. Em uma `Organization`, criar `Project` (ex: `master-platform`).
3. Dentro do project, criar `Application`:
   - **Web** — para backend que conversa com o IdP usando `client_secret` ou `private_key_jwt`.
   - **Native/User-Agent** — para SPA/mobile; ativa **PKCE obrigatório**.
   - **API** — para resource server; gera um `client_id` consumido pela introspection call.
4. Configurar:
   - Redirect URIs (exatos, sem wildcard em prod).
   - Post-logout URIs.
   - `authMethodType`: `none` (PKCE), `basic`/`post` (secret) ou `private_key_jwt` (chave).
   - Grant types permitidos: `authorization_code`, `refresh_token`, `urn:ietf:params:oauth:grant-type:jwt-bearer` para M2M.

## 3. Variáveis de ambiente

Canônicas, usadas pelos SDKs oficiais:

```bash
ZITADEL_DOMAIN=<tenant>.zitadel.cloud        # sem protocolo; SDK prepende https://
ZITADEL_ISSUER=https://<tenant>.zitadel.cloud
ZITADEL_CLIENT_ID=<uuid-ou-numeric-id>
ZITADEL_CLIENT_SECRET=<secret>               # só em Web com secret; nunca em SPA/mobile
ZITADEL_KEY_ID=<keyId>                       # JWT Profile / machine user
ZITADEL_PRIVATE_KEY_PATH=/secrets/zitadel-key.pem
ZITADEL_PROJECT_ID=<projectId>               # usado pra compor audience scope
ZITADEL_REDIRECT_URI=https://app.example.com/auth/callback
```

Secrets sempre em vault (1Password, Doppler, AWS Secrets Manager, Railway secrets), nunca em `.env` commitado.

## 4. SDK — Go (`zitadel-go/v3`)

```go
import (
    "context"

    "github.com/zitadel/zitadel-go/v3/pkg/client"
    "github.com/zitadel/zitadel-go/v3/pkg/client/zitadel"
)

func newZitadelClient(ctx context.Context) (*client.Client, error) {
    return client.New(
        ctx,
        zitadel.New(cfg.ZitadelDomain),
        client.WithAuth(client.DefaultServiceUserAuthentication(
            cfg.ZitadelKeyPath,                    // JSON key do machine user
            client.ScopeZitadelAPI(),              // API management
        )),
    )
}
```

Para verificação de token em middleware:

```go
import "github.com/zitadel/oidc/v3/pkg/op"

verifier := op.NewIntrospectionResourceServer(
    ctx,
    cfg.ZitadelIssuer,
    cfg.ZitadelClientID,
    cfg.ZitadelClientSecret,
)

claims, err := verifier.Introspect(ctx, bearerToken)
if err != nil {
    return nil, err
}
if !claims.IsActive() {
    return nil, ErrTokenRevoked
}
```

## 5. Fluxo OIDC end-to-end (Authorization Code + PKCE)

### 5.1 Redireciona usuário para autorizar

```go
codeVerifier := oauth2.GenerateVerifier()          // 43-128 chars aleatórios
state := random32()
nonce := random32()

// guarda verifier/state/nonce em cookie httpOnly assinado
http.SetCookie(w, &http.Cookie{
    Name: "auth_state", Value: sign(verifier, state, nonce),
    HttpOnly: true, Secure: true, SameSite: http.SameSiteLaxMode,
    MaxAge: 600,
})

authURL := oauth2Config.AuthCodeURL(
    state,
    oauth2.AccessTypeOffline,                      // pede refresh_token
    oauth2.S256ChallengeOption(codeVerifier),
    oidc.Nonce(nonce),
)
http.Redirect(w, r, authURL, http.StatusFound)
```

### 5.2 Callback

```go
// 1. valida state (CSRF)
// 2. troca code por token usando o code_verifier guardado
token, err := oauth2Config.Exchange(
    ctx, r.URL.Query().Get("code"),
    oauth2.VerifierOption(codeVerifier),
)

// 3. valida id_token + nonce
idToken, err := idTokenVerifier.Verify(ctx, token.Extra("id_token").(string))
if idToken.Nonce != nonce { return ErrNonceMismatch }

// 4. cria sessão local; guarda APENAS refresh_token encriptado (ver antipatterns)
```

### 5.3 Refresh

```go
src := oauth2Config.TokenSource(ctx, &oauth2.Token{RefreshToken: stored})
newToken, err := src.Token()
// IMPORTANTE: rotacionar — sobrescrever o stored com newToken.RefreshToken
// (Zitadel invalida o refresh antigo após uso; RFC 6749 §6)
```

## 6. M2M — JWT Profile

```go
import (
    "github.com/golang-jwt/jwt/v5"
)

func buildAssertion(userID, keyID string, priv *rsa.PrivateKey, issuer string) (string, error) {
    now := time.Now()
    claims := jwt.MapClaims{
        "iss": userID,
        "sub": userID,
        "aud": issuer,
        "iat": now.Unix(),
        "exp": now.Add(1 * time.Hour).Unix(),
    }
    t := jwt.NewWithClaims(jwt.SigningMethodRS256, claims)
    t.Header["kid"] = keyID
    return t.SignedString(priv)
}

// POST form: grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
//            assertion=<signed>
//            scope=openid urn:zitadel:iam:org:project:id:<projectId>:aud
```

Cachear o `access_token` resultante pelo `exp - 60s` dentro do processo — não repete assertion a cada chamada.

## 7. Introspection no resource server

Resource server valida cada bearer token. Para tokens opacos, chamar `/oauth/v2/introspect`:

```http
POST /oauth/v2/introspect
Authorization: Basic base64(clientId:clientSecret)
Content-Type: application/x-www-form-urlencoded

token=<bearer>&token_type_hint=access_token
```

Resposta:

```json
{
  "active": true,
  "sub": "22648...",
  "aud": ["<projectId>"],
  "exp": 1714138200,
  "client_id": "22649...",
  "urn:zitadel:iam:user:metadata": { "...": "..." },
  "urn:zitadel:iam:org:project:roles": {
    "syndic": { "<orgId>": "<orgDomain>" }
  }
}
```

Chave de mapeamento local é **sempre** `sub` (ver [[patterns]] e [[antipatterns]]).

## 8. Checklist de integração

- [ ] Issuer idêntico ao `/.well-known/openid-configuration`.
- [ ] Redirect URIs exatos (sem trailing slash divergente).
- [ ] PKCE habilitado em SPA/mobile.
- [ ] `state` + `nonce` validados no callback.
- [ ] Só `refresh_token` é persistido, encriptado em repouso.
- [ ] Refresh rotation ativa (guarda novo token, descarta antigo).
- [ ] Introspection com cache de 5 min (ver [[patterns]]).
- [ ] `sub` como chave local; nunca `email`/`preferred_username`.
- [ ] Machine user com chave em vault; rotacionada periodicamente.
- [ ] `client_secret` (quando existe) nunca em repositório.

## Vizinhos no grafo

- [[_moc]]
- [[overview]]
- [[patterns]]
- [[antipatterns]]
- [[../_moc]]
- [[../../security/beyond-corp]]
- [[../../security/_moc]]
