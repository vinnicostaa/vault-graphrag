---
title: Identity Provider (IdP) — componente de autenticação
type: concept
tags:
  - security
  - idp
  - identity-provider
  - authentication
  - federation
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - IdP
  - Identity Provider
---

# Identity Provider (IdP)

**O que é**: serviço que autentica usuários e emite **provas de identidade** (tokens, assertions) que aplicações consumem. Componente concreto dentro da disciplina [[iam|IAM]]. Implementa [[oidc|OIDC]] e/ou [[saml-2|SAML]].

> IdP ≠ IAM. IdP é o **software/serviço** que faz autenticação. IAM é a **disciplina inteira** (authn + authz + lifecycle + audit).

## Papel do IdP

```
┌──────────┐          ┌──────────────┐          ┌──────────┐
│ User     │ login    │ Identity     │  token   │ App      │
│          │─────────▶│ Provider     │─────────▶│ (SP/RP)  │
│          │          │ (IdP)        │          │          │
└──────────┘          └──────────────┘          └──────────┘
   credencial              valida              consome prova
   (pw/MFA/             • autenticação         (id_token/SAML
    passkey)            • issues token          assertion)
                        • mantém sessão
```

**Responsabilidades**:
- Armazenar credenciais (senhas hash, public keys de WebAuthn).
- Verificar fatores de autenticação (password, MFA, passkey).
- Emitir tokens/assertions assinadas.
- Manter sessão SSO central.
- Federar com outros IdPs quando aplicável.
- Log de eventos de auth para audit.

## Categorias de IdP

### 1. Enterprise / workforce IdP

Para funcionários, parceiros B2B.

| Produto | Tipo | Protocolos |
|---|---|---|
| **Microsoft Entra ID** | SaaS | OIDC + SAML |
| **Okta** | SaaS | OIDC + SAML |
| **Google Workspace** | SaaS | OIDC + SAML |
| **Ping Identity** | SaaS / on-prem | OIDC + SAML |
| **ADFS** (Active Directory Federation Services) | Microsoft on-prem legado | SAML + OIDC básico |
| **OneLogin** | SaaS | OIDC + SAML |

### 2. Open-source / self-hosted

| Produto | Stack | Protocolos |
|---|---|---|
| **[[../providers/zitadel/_moc\|Zitadel]]** | Go | OIDC + SAML |
| **Keycloak** | Java (JBoss/Wildfly) | OIDC + SAML |
| **Authentik** | Python/Django | OIDC + SAML + LDAP |
| **Ory Kratos + Hydra** | Go | OIDC (Hydra) + identity (Kratos) |
| **FusionAuth** | Java | OIDC + SAML |

### 3. Consumer (CIAM)

End-users do produto.

| Produto | Foco |
|---|---|
| **Auth0** | CIAM leader; rules + hooks |
| **Firebase Auth** | Google stack; ótimo para mobile |
| **AWS Cognito** | AWS stack; user pools + federation |
| **Supabase Auth** (GoTrue) | Open-source; Postgres-backed |
| **Clerk** | Developer-first; React-first UI |
| **Stytch** | Passwordless-first |
| **WorkOS** | B2B SaaS (SSO/SCIM-as-a-service) |

### 4. Social IdP (federation)

Público "Sign in with X" — geralmente usado via federation em IdP próprio.

- Google, Apple, Microsoft, Facebook, GitHub, GitLab, Twitter/X, LinkedIn, Discord.

## Federation — quando combinam

```
User bate em App
        │
        ▼
┌────────────────────┐
│ Meu IdP (primary)  │     ┌──────────────────┐
│ "Escolha método:"  │─────▶│ Google IdP      │
│  • Senha local     │     │ (federated)      │
│  • Google          │     └──────────────────┘
│  • Microsoft       │     ┌──────────────────┐
│  • Apple           │─────▶│ Apple IdP       │
└────────────────────┘     └──────────────────┘
```

**Meu IdP** (Zitadel, Keycloak, Auth0) federeia com **social IdPs**. App vê só meu IdP; não precisa integrar com N SDKs diferentes.

Claims normalizados: meu IdP reescreve claims vindos do Google/Apple no formato esperado pela app.

## Identity brokering

Quando meu IdP age como **broker** entre SPs e múltiplos IdPs downstream (SAML ou OIDC), dando UX unificado. Keycloak chama isso "Identity Brokering"; Auth0 chama "Connections".

## Padrão canônico (como integrar IdP na aplicação)

Usando [[oidc|OIDC]] + [[pkce|PKCE]]:

```ts
// 1. Descoberta (uma vez, cached)
const issuer = await Issuer.discover('https://idp.example.com')

// 2. Client setup
const client = new issuer.Client({
  client_id: process.env.CLIENT_ID,
  redirect_uris: ['https://app.example.com/callback'],
  response_types: ['code'],
})

// 3. Redirect a login
app.get('/login', (req, res) => {
  const { url, state, nonce, codeVerifier } = buildAuthUrl(client)
  req.session.pkce = { state, nonce, codeVerifier }
  res.redirect(url)
})

// 4. Callback
app.get('/callback', async (req, res) => {
  const tokens = await client.callback(
    'https://app.example.com/callback',
    client.callbackParams(req),
    req.session.pkce
  )
  // Valida id_token, cria sessão local (cookie httpOnly)
  req.session.user = { sub: tokens.claims().sub }
  res.redirect('/')
})

// 5. Logout
app.get('/logout', (req, res) => {
  const url = client.endSessionUrl({ id_token_hint: req.session.idToken })
  req.session.destroy()
  res.redirect(url)
})
```

## IdP sem IdP — "faço eu mesmo"

Tentador, mas **difícil** fazer bem:

- Hash de senha correto (argon2id, bcrypt), rate limit, account lockout.
- Reset de senha seguro (token curto, 1-time).
- MFA (TOTP, WebAuthn).
- Session management server-side.
- Credential stuffing detection.
- Account enumeration defense.
- LGPD/GDPR compliance (data rights).

**Regra**: em 2026, não implementar IdP próprio em projeto novo. Use Zitadel (self-host OSS) ou SaaS (Auth0/Clerk/Supabase). Reserve energia para o domínio do produto.

## Anti-patterns

1. **Implementar auth próprio em projeto novo** — sunk cost; use IdP.
2. **Um IdP sem DR / backup** — IdP cai = todo mundo fora. Multi-region ou fallback plan.
3. **Senha no `users` table da app** — delegue ao IdP.
4. **Compartilhar client_secret** entre apps — cada app tem seu próprio.
5. **Não federar com social IdP** quando user quer Sign-in-with-X — UX pobre em CIAM.
6. **IdP + caching de claims sem invalidation** — [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]].
7. **Login UI custom em vez de IdP** — perde maturidade de UX (error handling, MFA prompts, passkey flow).
8. **Misturar CIAM e workforce em 1 IdP** — scopes diferentes, UX diferentes, compliance diferentes. Separar.

## Como grandes empresas escolhem

- **Startups B2B / SaaS**: Auth0, Clerk, WorkOS, Stytch.
- **Enterprise workforce**: Okta, Entra ID, Ping.
- **Privacy-focused / self-host**: Zitadel, Keycloak, Authentik.
- **AWS-shop**: Cognito (OK até escalar complexidade).
- **Google-shop**: Firebase Auth + Google Identity Platform.
- **Open-source puro**: Ory stack (Kratos + Hydra + Keto).

## Relações

- **Disciplina**: [[iam]]
- **Protocolos emitidos**: [[oidc]], [[saml-2]], [[oauth-2]]
- **Token format**: [[jwt]]
- **Federation**: [[sso]]
- **Factors**: [[mfa-step-up]], [[webauthn-passkeys]], [[otp-hotp-totp]]
- **Passwordless**: [[magic-links]]
- **Provisioning**: [[scim-provisioning]]
- **Provider concreto**: [[../providers/zitadel/_moc]]
- **Zero Trust**: [[beyond-corp]]

## Fontes

- [NIST SP 800-63B — Digital Identity — Authentication and Lifecycle](https://pages.nist.gov/800-63-3/sp800-63b.html) (consultada 2026-04-24)
- [OpenID Foundation](https://openid.net/) (consultada 2026-04-24)
- [Okta — What is an Identity Provider?](https://www.okta.com/identity-101/what-is-an-identity-provider/) (consultada 2026-04-24)
- [Auth0 — IdP](https://auth0.com/docs/authenticate/identity-providers) (consultada 2026-04-24)
- [Zitadel docs](https://zitadel.com/docs) (consultada 2026-04-24)
- [Keycloak docs](https://www.keycloak.org/documentation) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[iam]]
- [[sso]]
- [[oidc]]
- [[saml-2]]
- [[oauth-2]]
- [[mfa-step-up]]
- [[owasp-top-10]] — A07 Identification/Authentication Failures
- [[../runtime-antipatterns/op-024-rate-limit-ausente]] — IdP endpoints são alvo prioritário de brute force
- [[../providers/zitadel/_moc]]
- [[../providers/cloudflare/access]] — CF Access consome identidade do IdP configurado (OIDC)
