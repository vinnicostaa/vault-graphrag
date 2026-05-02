---
title: "AWS Zero Trust — Prescriptive Guidance & product landing"
type: source
tags: [research, zero-trust, aws, cedar, verified-access, verified-permissions]
source: https://aws.amazon.com/security/zero-trust/
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 08 — AWS Zero Trust

## Metadata

- **Instituição**: Amazon Web Services (AWS)
- **URL landing**: https://aws.amazon.com/security/zero-trust/
- **URL Prescriptive Guidance — princípios**: https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-zero-trust-architecture/zero-trust-principles.html
- **URL Prescriptive Guidance — componentes**: https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-zero-trust-architecture/components.html
- **URL Prescriptive Guidance — best practices**: https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-zero-trust-architecture/best-practices.html
- **URL whitepaper**: https://pages.awscloud.com/AWSMP-Whitepaper-SEC-IAM-ZeroTrust-Whitepaper.html
- **URL blog announce**: https://aws.amazon.com/blogs/security/new-whitepaper-available-charting-a-path-to-stronger-security-with-zero-trust/
- **Data de acesso**: 2026-04-23
- **Tipo**: documentação de produto + prescriptive guidance

## Resumo

AWS consolida sua abordagem Zero Trust em torno de 3 princípios + catálogo de produtos nativos:

**Princípios**:
1. **Verify and authenticate** — identificação e autenticação fortes de todos os principais (users, machines, devices), verificação contínua ao longo da sessão, idealmente por request.
2. **Least-privilege access** — JIT provisioning, RBAC, revisões regulares.
3. **Micro-segmentation** — dividir rede em segmentos isolados (Security Groups, VPC Lattice, PrivateLink).

Adicionado pela AWS nas best practices:
4. **Continuous monitoring and analytics** — SIEM, UEBA, threat intel.

**Produtos AWS relevantes**:
- **AWS Verified Access** — ZTNA gerenciado, equivalente AWS de Cloudflare Access / Google IAP. Acesso a aplicações sem VPN, com policies baseadas em identidade e contexto (Cedar policies).
- **AWS Verified Permissions** — serviço de autorização externa (PDP gerenciado) baseado em **Cedar** (linguagem de política aberta criada pela AWS).
- **Amazon VPC Lattice** — service-to-service connectivity com authN/authZ embutido (IAM auth em nível de service).
- **Amazon Cognito** — IdP para aplicações (OAuth/OIDC).
- **AWS IAM Identity Center** (antigo SSO) — pra workforce.
- **AWS WAF + Shield** — edge protection.
- **Amazon GuardDuty / Security Hub** — analytics/detecção.

## Trechos-chave

> "The verify and authenticate principle emphasizes the importance of strong identification and authentication for principals of all types, including users, machines, and devices. ZTA requires continuous verification of identities and authentication status throughout a session, ideally on each request."
> — AWS Prescriptive Guidance, Zero Trust principles

> "The principle of least privilege involves granting principals the minimum level of access required to perform their tasks."
> — AWS Prescriptive Guidance

> "Micro-segmentation is a network security strategy that divides a network into smaller, isolated segments for authorizing specific traffic flows."
> — AWS Prescriptive Guidance

> "Access to data should not be solely made based on network location (...) Systems must strongly prove their identities and trustworthiness."
> — AWS Zero Trust landing page

> "Continuous monitoring and analytics involve the collection, analysis, and correlation of security-related events and data across your organization's environment."
> — AWS Prescriptive Guidance

## Aplicabilidade ao Master Síndico

**Sim, alto — especialmente se master-sindico confirmar AWS como provedor principal.**

Justificativa e mapeamento:

| Necessidade master-sindico | Produto AWS | Alternativa agnóstica |
|---|---|---|
| IdP end-user | Cognito | Keycloak, Auth0 |
| IdP workforce (equipe interna) | IAM Identity Center + SSO | Okta, Azure AD |
| PDP (ABAC) | **Verified Permissions (Cedar)** | OPA (Rego) |
| ZTNA pra admin console | Verified Access | Cloudflare Access, Tailscale |
| Service-to-service authN | VPC Lattice + IAM auth | Istio/Linkerd + SPIFFE |
| Edge WAF | AWS WAF + Shield + CloudFront | Cloudflare |
| Detecção | GuardDuty + Security Hub | Datadog Security, Wazuh |
| Secrets | Secrets Manager, Parameter Store | Vault |

**Decisão crítica sobre PDP**: Cedar (AWS-managed) vs OPA (portável).

Trade-off:
- **Cedar pros**: linguagem desenhada pra autorização (mais restritiva que Rego, menos pegadinhas), gerenciado, integra com API Gateway + Verified Access nativamente.
- **Cedar cons**: lock-in AWS (embora Cedar em si seja open source, Verified Permissions é AWS-only); comunidade menor que OPA.
- **OPA pros**: ecossistema maduro, portável (Kubernetes admission, gateways, DBs), Rego é flexível demais mas poderoso.
- **OPA cons**: self-hosted (você opera o PDP); Rego tem curva de aprendizado íngreme.

Recomendação provisória: **OPA em M1** (portabilidade, ADR reversível em M2 se migrarmos pra Cedar).

⚠️ PENDÊNCIA: ADR formal "PDP: OPA vs Cedar" a abrir.

## Tags/tópicos

- aws
- zero-trust
- cedar
- verified-access
- verified-permissions
- vpc-lattice
- cognito
- pdp
