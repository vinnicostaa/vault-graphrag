---
title: Cloudflare Access — Zero Trust auth para apps internos
type: note
tags:
  - provider
  - cloudflare
  - access
  - zero-trust
  - beyondcorp
  - authorization
  - sso
category: zero-trust
doc-oficial: https://developers.cloudflare.com/cloudflare-one/policies/access/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Access
  - CF Access
  - Cloudflare One Access
---

# Cloudflare Access

**Categoria**: Zero Trust auth gateway — autenticação/autorização de aplicações internas ou públicas restritas, sem VPN.

**Intenção**: implementar BeyondCorp real. Cada request a app protegido passa por policy engine antes de chegar ao origem; suporta OIDC, SAML, OAuth, certs e service tokens.

Ver também [[../../security/beyond-corp]] para o modelo conceitual e [[tunnel]] para o complemento de exposição.

## Quando usar

1. **Apps internos corporativos** — admin dashboards, Grafana, Jira self-hosted, GitLab interno.
2. **Preview deploys protegidos** — `*.pages.dev` só acessível a time.
3. **SSH / RDP / VNC via browser** — render interativo com auth centralizada.
4. **API gateway com auth federado** — Workers atrás de Access conhecem identidade via `Cf-Access-Jwt-Assertion`.
5. **Zero Trust Network Access (ZTNA)** substituindo VPN.

## Quando **NÃO** usar

- **Auth end-user do produto** (cliente final) — Access é para **equipe/parceiros**, não para auth de consumidores. Use [[../zitadel/_moc|Zitadel]] ou similar para consumer auth.
- **Casos onde provider OIDC terceiro já cobre**: se o app integra nativamente SSO via Zitadel/Okta/Auth0, Access é camada adicional — avaliar se necessário.
- **Compute que precisa credentials permanentes** — use service tokens Access (tem TTL limitado).

## Padrão canônico

### 1. Configurar Identity Provider

Dashboard: `Zero Trust → Settings → Authentication → Login methods`.

Opções: Google Workspace, Microsoft Entra ID, Okta, GitHub, GitLab, OIDC genérico, SAML, One-time PIN (email).

### 2. Criar Application

Dashboard: `Zero Trust → Access → Applications → Add an application`.

Tipos:
- **Self-hosted**: hostname (ex: `admin.example.com`). Requer Cloudflare gerenciando DNS + Tunnel (para apps internos) ou proxy (para apps públicos).
- **SaaS**: SSO para SaaS externo (Salesforce, Workday).
- **Private Network**: TCP/UDP não-HTTP.
- **Infrastructure**: SSH / RDP / Kubernetes.

### 3. Criar Policies

```
Application: admin.example.com
│
├── Policy 1: "Allow team"
│   Action: Allow
│   Include: Emails ending in @example.com
│   Require: MFA (hardware key OR TOTP)
│   Session duration: 24h
│
└── Policy 2: "Block known bad actors"
    Action: Block
    Include: Country is CN, RU
```

**Rules engine**: Include (OR entre regras) + Require (AND de condições adicionais) + Exclude.

**Sinais de decisão**:
- User (email, group, IdP claim)
- Device (WARP enrolled, posture checks — disk encryption, OS version)
- Network (IP, country, ASN)
- Context (time of day, session age)

### 4. Integração com origin

#### 4a. Worker atrás de Access

Access adiciona header `Cf-Access-Jwt-Assertion` com JWT assinado. Worker verifica:

```ts
import * as jose from 'jose'

const JWKS = jose.createRemoteJWKSet(
  new URL(`https://${TEAM}.cloudflareaccess.com/cdn-cgi/access/certs`)
)

app.use(async (c, next) => {
  const token = c.req.header('Cf-Access-Jwt-Assertion')
  if (!token) return c.text('unauthorized', 401)

  try {
    const { payload } = await jose.jwtVerify(token, JWKS, {
      issuer: `https://${TEAM}.cloudflareaccess.com`,
      audience: AUD_TAG,    // obtido no dashboard da app
    })
    c.set('user', payload.email as string)
    c.set('groups', payload.groups as string[])
  } catch (err) {
    return c.text('invalid token', 401)
  }
  await next()
})
```

#### 4b. App HTTP genérico atrás de Tunnel

Tunnel conecta origem interno → Cloudflare. Access camada em cima: `admin.example.com` → Access policy → Tunnel → `localhost:3000`.

Zero código no app; auth é totalmente externalizada.

## Service Tokens

Para M2M (CI, worker que chama app Access-protected):

```bash
# Dashboard: Zero Trust → Access → Service Auth → Service Tokens
CF-Access-Client-Id: <id>
CF-Access-Client-Secret: <secret>
```

Request ao app inclui headers; Access valida antes de rotear.

## MCPs relacionados

- **`casb.mcp.cloudflare.com`** — detect SaaS misconfigurations.
- **`dex.mcp.cloudflare.com`** — Digital Experience Monitoring (latência/saúde para usuários enroladas no WARP).

## Limites / Pricing (consultada 2026-04-24)

- **Free tier**: 50 users, todas features core.
- **Paid (Zero Trust Standard)**: $7/user/mo; Enterprise variável.
- **Session duration**: 30 min a 1 mês (policy-level).
- **Applications**: ilimitado.

## Antipatterns específicos

- **Só Access sem MFA** — auth factor único. Sempre Require MFA ou posture check.
- **Access policy por IP apenas** — IP spoofável (via VPN/proxy); é sinal, não prova. Combinar com user/device.
- **Não verificar JWT no origin (Worker)** — atacante pode remover header e atingir origin se Cloudflare proxy for bypassado. Sempre validar JWT.
- **Session duration muito longa** (24h+) em app crítico — revogação lenta. 1-4h é bom default para admin.
- **Service token em commit público** — rotação imediata via dashboard; use secret manager.
- **Misturar Access com auth próprio de app** — dupla camada confunde; prefira delegar inteiramente para Access quando possível.

## Relações

- **Combina com**: [[tunnel]] (ZTNA para app interno); [[workers]] (Worker verificando JWT Access); [[pages]] (proteger preview deploys)
- **Complementa**: [[../../security/beyond-corp]] (modelo conceitual); [[warp]] (lazy — client device enrollment para posture checks)
- **Alternativas**:
  - Zscaler, Netskope ZTNA (enterprise com foco compliance).
  - VPN tradicional (OpenVPN, WireGuard) — mais antigo, menos granular.
  - [[../zitadel/_moc]] para auth end-user (consumer) — complementar, não alternativa.

## Reference Architectures & Design Guides

- [Designing ZTNA Access Policies](https://developers.cloudflare.com/reference-architecture/design-guides/designing-ztna-access-policies/) — framework para policy design (consultada 2026-04-24)
- [Network VPN → ZTNA migration](https://developers.cloudflare.com/reference-architecture/design-guides/network-vpn-migration/) — fases de migração concentrator→ZTNA (consultada 2026-04-24)
- [Zero Trust for SaaS](https://developers.cloudflare.com/reference-architecture/design-guides/zero-trust-for-saas/) — Access sobre SaaS apps (consultada 2026-04-24)
- [Zero Trust for startups](https://developers.cloudflare.com/reference-architecture/design-guides/zero-trust-for-startups/) — adoption gradual (consultada 2026-04-24)

Ver [[sase]] para arquitetura completa e [[_moc#Reference-Architectures]] para hub.

## Fontes

- [Cloudflare Access docs](https://developers.cloudflare.com/cloudflare-one/policies/access/) (consultada 2026-04-24)
- [Access Applications](https://developers.cloudflare.com/cloudflare-one/applications/) (consultada 2026-04-24)
- [Access — Service tokens](https://developers.cloudflare.com/cloudflare-one/identity/service-tokens/) (consultada 2026-04-24)
- [Verify JWT — Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/identity/authorization-cookie/validating-json/) (consultada 2026-04-24)
- [Cloudflare One pricing](https://www.cloudflare.com/plans/zero-trust-services/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[sase]] — umbrella Cloudflare One
- [[tunnel]]
- [[warp]] — client-side complement
- [[../../security/beyond-corp]]
- [[../../security/security-principles]]
- [[../zitadel/_moc]]
