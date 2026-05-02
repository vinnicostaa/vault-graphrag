---
title: SSO — Single Sign-On
type: concept
tags:
  - security
  - sso
  - single-sign-on
  - federation
  - authentication
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Single Sign-On
  - SSO
---

# SSO — Single Sign-On

**O que é**: padrão de UX/arquitetura onde **um login** no IdP central concede acesso a múltiplas aplicações sem re-autenticar. Operacionalizado via [[oidc|OIDC]] ou [[saml-2|SAML]]. Hoje, SSO moderno quase sempre usa OIDC + WebAuthn/passkeys + MFA.

> SSO ≠ protocolo. SSO é **resultado**; OIDC/SAML são os protocolos que implementam. "Sign in with Google" no seu app → é SSO via OIDC federation.

## Como funciona (SP-initiated, OIDC)

```
1. User → App: acessa /dashboard
2. App detecta: sem sessão → redirect ao IdP
3. User → IdP: login (com MFA se necessário)
4. IdP mantém sessão própria (cookie do domínio do IdP)
5. IdP → App: id_token + access_token (OIDC) via redirect callback
6. App cria sessão local (cookie httpOnly do domínio da app)

Próximas apps federadas:
7. User → App2: acessa App2
8. App2 redirect ao IdP
9. IdP vê cookie de sessão dele → **sem re-login** → token direto para App2
10. App2 cria sessão local
```

**Chave**: IdP mantém **sessão central** (próprio domínio) visível em cada redirect. Apps federadas detectam "user já logou no IdP recentemente" e entregam token sem pedir credencial de novo.

## Tipos de SSO

### 1. Enterprise SSO (workforce)

- Funcionários logam uma vez no IdP corporativo (Okta, Entra ID, Ping, ADFS).
- Acessam 50-500 apps SaaS internas (Gmail, Slack, Jira, Salesforce, etc.) sem re-login.
- Protocolos: SAML dominante legacy, OIDC crescendo.
- UX comum: "Login with Okta", "Login with Microsoft".

### 2. Consumer SSO (social login)

- User loga com conta Google/Apple/Facebook/GitHub em app de terceiro.
- App federa com social IdP.
- Protocolo: OIDC (todos os grandes suportam) ou legacy OAuth 1.0 (Twitter, Flickr).

### 3. Identity Federation B2B

- Empresa A tem usuários; App SaaS B quer aceitar login de A sem gerenciar senhas de A.
- A publica IdP (ex: Okta); B faz `trust` de A.
- Protocolos: SAML (histórico), OIDC, SCIM (provisioning).
- Serviços como [WorkOS](https://workos.com) abstraem a complexidade.

## Vantagens do SSO

- **UX**: 1 senha (ou melhor, 1 passkey) em vez de N por app.
- **Segurança**: senha forte + MFA aplicados uma vez; apps downstream herdam.
- **Centralização**: offboarding em 1 lugar → revoga em todos os apps.
- **Compliance**: audit centralizado; access review em ferramenta única.
- **MFA/Passkey** consistente: aplicação complexa cara por app; IdP centraliza.

## Desvantagens / riscos

- **Single Point of Failure**: IdP cai → todo mundo cai. Mitigação: IdP com DR, BYO-backup-flow em apps críticas.
- **Compromisso catastrófico**: se atacante obtém credencial do IdP admin, tem acesso a tudo. Mitigação: FIDO2 hardware keys no IdP; access reviews; anomaly detection.
- **Token theft = massive blast radius**: session cookie roubado no IdP → atacante loga em todos os apps federados.
- **Phishing de IdP**: página falsa "Microsoft login" captura credencial. Mitigação: passkeys (phishing-resistant).
- **Latência extra**: 1 redirect extra por login (mitigado pela frequência — login raro).

## SSO + [[mfa-step-up|Step-up authentication]]

User loga com password (acr=1). Operação sensível (wire transfer) exige MFA recente → app faz `/authorize?acr_values=urn:mace:incommon:iap:silver prompt=login`. IdP força re-auth com MFA. Token novo tem acr=2.

Ver [[oidc]] §acr e [[mfa-step-up]].

## SLO (Single Logout) — o lado difícil

SSO entrega 1 login; logout é mais complexo:

- **RP-Initiated Logout** (OIDC): user clica "sair" em App A → A redireciona ao `/end_session` do IdP → IdP termina sessão central e redireciona de volta.
- **Back-channel logout**: IdP avisa server-side todos os RPs ativos. Mais robusto; requer RPs expor endpoint `logout_token`.
- **Front-channel logout** (OIDC): IdP carrega iframes escondidos para cada RP. Browsers 3rd-party bloqueiam.

**Anti-pattern**: user faz "logout" em 1 app, continua logado em outras. Sempre integrar back-channel logout em IdP moderno.

## Protocolos por contexto

| Contexto | Protocolo preferido |
|---|---|
| B2C (user loga com Google) | OIDC |
| B2C (novo produto) | OIDC + passkeys |
| B2B SaaS (cliente corporativo SAML-only) | SAML 2.0 |
| Workforce interno novo | OIDC |
| Workforce interno legado (ADFS) | SAML |
| API-to-API | OAuth 2.0 Client Credentials (não é SSO propriamente) |
| CLI / Device sem browser | OIDC Device Authorization Grant |

## Padrão "Mesh of SSO"

Realidade corporativa: muitas empresas têm **vários IdPs**:
- Workforce: Entra ID + Okta.
- CIAM: Auth0 para produtos B2C.
- Legacy: ADFS para apps antigas.

Estratégia: um **IdP master** que brokera outros (Keycloak fazendo broker, ou IdP primary + federation com secundários).

## Anti-patterns

1. **"SSO" que é só cookie compartilhado** entre subdomínios — não é SSO; é sessão cross-subdomain. Frágil e não compliance-ready.
2. **Self-hosted password-only SSO em projeto novo** — use passkeys.
3. **Sem SLO** — logout em 1 app não limpa outros.
4. **Sessão IdP muito longa** (30d+) — user fica logado em muito tempo; token stolen tem janela ampla.
5. **Cross-subdomain sem precaution**: `app.example.com` pode ler cookie de `admin.example.com` se cookie tem `domain=.example.com` — isolamento importa.
6. **Mixing IAM e CIAM no mesmo IdP** — [[identity-provider]] §anti-pattern.
7. **Social login sem verificar `email_verified`** — alguns IdPs sociais não marcam como verified.
8. **Aceitar qualquer IdP B2B externo sem pinning** — atacante cria IdP fake e convence admin.

## Como grandes empresas fazem

- **Google Workspace**: Google Sign-In como IdP corporativo + SAML para apps externos.
- **Microsoft 365**: Entra ID + Conditional Access + MFA everywhere + PIM para admin roles.
- **Stripe Connect**: pode usar IdP próprio ou delegar ao IdP do client (Stripe Dashboard SSO).
- **GitHub Enterprise**: SAML/OIDC para enterprise; SCIM provisioning.
- **Slack Enterprise Grid**: SSO + SCIM obrigatórios; Enterprise Key Management para data at rest.

## Relações

- **Protocolos**: [[oidc]] (moderno), [[saml-2]] (legacy enterprise), [[oauth-2]] (base)
- **Componente**: [[identity-provider]]
- **Disciplina**: [[iam]]
- **Fortalecimento**: [[mfa-step-up]], [[webauthn-passkeys]]
- **Provisioning**: [[scim-provisioning]]
- **Tokens**: [[jwt]]
- **Sessão**: [[session-management]], [[cookies]]
- **Zero Trust**: [[beyond-corp]]

## Fontes

- [OIDC SSO flows](https://openid.net/specs/openid-connect-core-1_0.html) (consultada 2026-04-24)
- [SAML Web Browser SSO Profile](https://docs.oasis-open.org/security/saml/v2.0/saml-profiles-2.0-os.pdf) (consultada 2026-04-24)
- [Okta — SSO explained](https://www.okta.com/sso/) (consultada 2026-04-24)
- [OneLogin — SSO guide](https://www.onelogin.com/learn/how-single-sign-on-works) (consultada 2026-04-24)
- [Auth0 — Identity Federation](https://auth0.com/docs/authenticate/identity-providers/enterprise-identity-providers) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[identity-provider]]
- [[oidc]]
- [[saml-2]]
- [[mfa-step-up]]
- [[iam]]
- [[owasp-top-10]] — A07 (SSO é superfície-crítica de auth failure)
- [[../runtime-antipatterns/op-024-rate-limit-ausente]] — proteção de endpoints de login SSO
- [[../providers/zitadel/_moc]]
- [[../providers/cloudflare/access]]
