---
title: Source 12 — Firebase Authentication vs Identity Platform
type: source
tags: [research, google, identity, firebase, identity-platform, ciam, auth, zitadel-compare]
source: https://docs.cloud.google.com/identity-platform/docs/product-comparison
created: 2026-04-23
updated: 2026-04-23
aliases: [identity-platform-vs-firebase]
---

# Source 12 — Firebase Authentication vs Identity Platform

- **URL primária**: https://docs.cloud.google.com/identity-platform/docs/product-comparison
- **Relacionado**: https://firebase.google.com/docs/auth, https://docs.cloud.google.com/identity-platform/docs/concepts-admin-auth-api
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Exclusivos do Identity Platform (Firebase Auth não tem)
- **MFA**: Identity Platform inclui built-in MFA; Firebase Auth não.
- **OIDC + SAML**: "OpenID Connect (OIDC) and Security Assertion Markup Language (SAML) authentication" only in Identity Platform.
- **Multi-tenancy**: native multi-tenancy no Identity Platform.
- **Blocking functions**: customização de auth flow no Identity Platform.
- **IAP integration**: somente Identity Platform integra com IAP.

### Compliance & SLA
- Identity Platform: BAA coverage (HIPAA), PCI-DSS scope, TISAX, "99.95% uptime SLA" (Enterprise).
- Firebase Auth: não tem essas certificações.

### Shared foundation
- Ambos suportam email/password, OAuth, phone sign-in, custom auth.
- Mesmos SDKs (Web, iOS, Android, Admin).

### CIAM readiness
- Identity Platform é purpose-built pra CIAM via multi-tenancy + SAML/OIDC + compliance.

## Aplicabilidade ao Master Síndico

**Stack atual**: Zitadel (self-host). Este source é benchmark defensivo para justificar a escolha.

### Matrix comparativa

| Feature | Identity Platform (Google) | Zitadel (nosso) |
|---|---|---|
| OIDC + SAML | Sim | Sim |
| MFA nativo | Sim | Sim |
| Multi-tenancy (organizations) | Sim | Sim |
| Hooks pre/post-auth | Blocking functions | Actions |
| Self-host | **Não** (GCP-only) | **Sim** |
| License | Proprietária | Apache 2.0 |
| SLA 99.95% enterprise | Sim (pago) | Depende do deployment |
| Custo | Per MAU | Infra own-cost |

### Decisão

- **M1/M2/M3**: **manter Zitadel**.
- **Motivos**:
  1. Self-host → sem vendor lock + compliance LGPD sob nosso controle.
  2. Paridade de features com Identity Platform nas que importam (MFA, SAML, OIDC, multi-tenancy).
  3. Quando cliente enterprise perguntar "usa Google?", mostrar esta matrix.
- **Risco residual**: 99.95% SLA enterprise do Identity Platform é forte argumento de venda — compensar com **status page própria + SLA interno documentado** (ligado a source-04 SLOs).

## Links relacionados

- [[source-05]] IAP (integra com Identity Platform).
- Destilado: [[_destilado#3.1 Identity Platform vs Zitadel]].
