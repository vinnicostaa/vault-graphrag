---
title: Zitadel — Overview (modelo de domínio + flows OIDC/M2M)
type: note
tags: [provider, zitadel, identity, oidc, overview]
source: https://zitadel.com/docs
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Zitadel Overview, Zitadel Modelo]
---

# Zitadel — Overview

Zitadel é um **identity provider** open-source, OIDC-compliant, com multi-tenancy nativa via `Organization` e permissões granulares. Esta nota destila o modelo de domínio e os flows que todo agente precisa entender antes de integrar.

## Modelo de domínio

### Organization

Raiz da hierarquia; representa um **tenant**. Cada organization tem seu próprio conjunto de users, projects, policies (login, password, branding) e domínios verificados. Toda entidade em Zitadel pertence a **uma** organization.

### Project

Agrupa aplicações relacionadas sob uma mesma fronteira de autorização. Um `Project` define **roles** que podem ser concedidas a users (diretamente) ou a outras organizations (via `Project Grant`). É o limite lógico onde vivem os `scopes` que o token carrega.

### Application

Client OIDC concreto dentro de um `Project`. Três tipos relevantes:

| Tipo | Uso | Auth method |
|---|---|---|
| `Web` | Backend server-side com secret | `client_secret_basic` / `client_secret_post` / `private_key_jwt` |
| `Native` / SPA | Mobile, desktop, browser-only | **PKCE obrigatório** (sem secret) |
| `API` | Recurso protegido (resource server) | Só valida tokens; não emite |

Cada `Application` possui `client_id`, redirect URIs, post-logout URIs, grant types permitidos e response types.

### User

Dois subtipos:

- **Human User** — identidade humana; tem email, senha, MFA (TOTP/WebAuthn/OTP SMS), sessão interativa via login UI.
- **Machine User** — service account; autentica via **JWT Profile (RFC 7523)** usando chave privada (JSON `Key`), **não** por senha. Ideal para integrações server-to-server.

### Role e Grant

- **Role** — definida no `Project`, é um rótulo (ex: `admin`, `manager`, `viewer`).
- **Grant** — liga `User ↔ Project ↔ Role[]` com possível escopo adicional. `User Grant` em runtime vira claim `urn:zitadel:iam:org:project:roles` no token.
- **Project Grant** — cede um `Project` para outra `Organization` consumir (multi-tenant B2B).

### Action

Código **JavaScript** executado em hooks (`pre-userinfo`, `pre-access-token`, `post-authentication`, `pre-creation`). Usado para:

- Enriquecer claims custom (ex: `plan_tier`, `tenant_id`, `device_id`).
- Invalidar cache externo ao revogar sessão.
- Disparar webhook em eventos de identity.

Actions são a forma oficial de injetar regra de negócio do produto **dentro do token**, sem acoplar o app ao IdP.

## OIDC — Authorization Code + PKCE

Fluxo padrão para aplicações com usuário humano. Clients públicos (SPA/mobile) **precisam** de PKCE; backends também devem usar para defesa em profundidade.

```
Browser/App                Zitadel                    Backend/API
    │                         │                           │
    │ 1. code_verifier (random)                           │
    │    code_challenge = S256(verifier)                  │
    │                                                     │
    │ 2. GET /oauth/v2/authorize                          │
    │    ?client_id&redirect_uri&scope=openid profile     │
    │    &code_challenge&code_challenge_method=S256       │
    │    &state&nonce                                     │
    ├────────────────────────▶│                           │
    │ 3. Login + consent                                  │
    │◀────────────────────────┤                           │
    │ 4. 302 → redirect_uri?code=AUTH_CODE&state          │
    │                                                     │
    │ 5. POST /oauth/v2/token                             │
    │    grant_type=authorization_code                    │
    │    code, code_verifier, client_id                   │
    ├────────────────────────▶│                           │
    │ 6. { access_token, id_token, refresh_token }        │
    │◀────────────────────────┤                           │
    │                                                     │
    │ 7. GET /api/resource  (Bearer access_token)         │
    ├────────────────────────────────────────────────────▶│
    │                          8. introspect / verify JWT │
    │                         ◀─────────────────────────── │
    │ 9. 200 OK                                           │
    │◀────────────────────────────────────────────────────┤
```

- `state` previne CSRF no callback; validar no retorno.
- `nonce` previne replay do `id_token`; comparar com valor guardado no browser.
- `code_verifier` nunca viaja até o passo 5 — roubo do `code` sozinho não rende token.

## M2M — JWT Profile (RFC 7523)

Para serviço-a-serviço. Superior a `client_credentials` com secret porque a chave privada **nunca** sai do chamador.

1. Gerar `Machine User` no Zitadel, baixar `Key` JSON (`keyId`, privateKey RSA).
2. Chamador monta um **assertion JWT** assinado com a privada:
   - `iss` = `sub` = `userId` do machine user.
   - `aud` = issuer do Zitadel (`https://<tenant>.zitadel.cloud`).
   - `exp` curto (≤ 1h); `iat` e `nbf` consistentes.
   - Header: `alg=RS256`, `kid=keyId`.
3. POST em `/oauth/v2/token` com:
   - `grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer`
   - `assertion=<JWT assinado>`
   - `scope=openid urn:zitadel:iam:org:project:id:<projectId>:aud`
4. Recebe `access_token` (opaco ou JWT, conforme config do project).

## Introspection (RFC 7662)

Endpoint `/oauth/v2/introspect` valida `access_token` server-side e devolve claims atuais, incluindo `active: false` se o token foi revogado. É o caminho canônico para tokens **opacos** e para detectar revogação em tokens JWT (que não morrem antes de expirar).

Ver [[patterns]] para a estratégia de introspection + cache curto.

## Cloud vs Self-hosted

| Critério | Cloud (`<tenant>.zitadel.cloud`) | Self-hosted |
|---|---|---|
| Setup | Minutos; TLS e DNS prontos | Horas; cuida de Postgres, TLS, backup |
| Custo | Free até 25k MAU; pay-per-user acima | Infra apenas; sem licença |
| SLA | Gerenciado pelo vendor | Operador responsável |
| Customização profunda | Limitada | Total (tema, binário custom) |
| Residência de dados | Regiões ofertadas pelo vendor | Qualquer lugar |
| Compliance estrito (on-prem) | Não atende | Atende |

Regra prática: começar em **cloud** e migrar para self-hosted só quando compliance ou custo exigirem — o modelo de dados é compatível (export/import via `v2` API).

## Vizinhos no grafo

- [[_moc]]
- [[../_moc]]
- [[integration-guide]]
- [[patterns]]
- [[antipatterns]]
- [[../../security/_moc]]
- [[../../security/beyond-corp]]
