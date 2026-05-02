---
title: IAM — Identity and Access Management (guarda-chuva)
type: concept
tags:
  - security
  - iam
  - identity
  - umbrella
  - lifecycle
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Identity and Access Management
  - IAM
---

# IAM — Identity and Access Management

**O que é**: disciplina e conjunto de serviços para **gerenciar quem é quem** (identidade) + **quem pode fazer o quê** (acesso) + **lifecycle** completo (onboarding, change, offboarding) + **audit**. IAM é o **guarda-chuva** — engloba authn, authz, provisioning, governance, compliance.

> **Não confundir com**: [[identity-provider|IdP]] (componente) ou [[authorization-models|Authorization]] (domínio específico). IAM é a disciplina; IdP é o software; authz é a dimensão.

## Os 4 pilares de IAM

```
┌──────────────────────────────────────────────────────┐
│                     IAM                              │
├──────────────────┬───────────────────────────────────┤
│ Authentication   │ "Quem é você?"                    │
│                  │ — Senha + MFA + WebAuthn          │
│                  │ — SSO (OIDC/SAML)                 │
│                  │ — Federation entre IdPs           │
├──────────────────┼───────────────────────────────────┤
│ Authorization    │ "O que você pode fazer?"          │
│                  │ — RBAC / ABAC / ReBAC / PBAC      │
│                  │ — Scopes, policies, permissions   │
├──────────────────┼───────────────────────────────────┤
│ Lifecycle /      │ "Onboarding / offboarding"        │
│ Governance       │ — Provisioning (manual, SCIM, JIT)│
│                  │ — Access reviews (quarterly)      │
│                  │ — Role mining / certification     │
├──────────────────┼───────────────────────────────────┤
│ Audit /          │ "Como detecto abuso e comprovo    │
│ Compliance       │  compliance?"                     │
│                  │ — Logs, trilhas, reports          │
│                  │ — SIEM integration                │
│                  │ — SOC2/ISO/PCI audit trails       │
└──────────────────┴───────────────────────────────────┘
```

## Decision tree — onde ir com cada dúvida

```
Você quer saber sobre:
│
├─ "Como user prova que é ele?" → [[identity-provider]], [[oidc]], [[oauth-2]]
├─ "Segundo fator / biometria?" → [[mfa-step-up]], [[webauthn-passkeys]], [[otp-hotp-totp]]
├─ "Magic link / passwordless?" → [[magic-links]]
├─ "Single Sign-On entre apps?" → [[sso]]
├─ "O que user pode fazer?" → [[authorization-models]], [[rbac]], [[abac]], [[rebac]]
├─ "Provisionamento automático em SaaS?" → [[scim-provisioning]]
├─ "Delegação, act-as, impersonation?" → [[delegation-impersonation]]
├─ "Workload-to-workload auth?" → [[spiffe-spire]]
├─ "Armazenar credentials/secrets?" → [[credential-storage]] (client), [[secrets-management]] (infra)
├─ "Sessão e revogação?" → [[session-management]]
├─ "Policy engine?" → [[opa-rego]]
└─ "Princípio base?" → [[least-privilege]]
```

## Identity lifecycle (Joiner-Mover-Leaver)

| Fase | Atividades | Tooling |
|---|---|---|
| **Joiner** | Onboard: criar conta, atribuir roles iniciais, enviar credencial | SCIM auto-provision; HRIS → IdP sync (Workday, BambooHR) |
| **Mover** | Promoção/transferência: roles novos + **remover roles antigos** (risco de acumulação) | Access reviews; role mining |
| **Leaver** | Offboard: disable/delete account, revoke tokens, recovery de dados | SCIM deprovision; automated termination playbook |

**Risco clássico**: "access creep" (mover) — user muda de função mas mantém acessos do cargo antigo → privilegios acumulam.

## Access reviews / Certification

Trimestral ou semestral:
- Managers revisam roles dos subordinados.
- Owners de aplicação revisam quem acessa.
- Certificação → aprovação ou revogação em ferramenta (Okta, SailPoint, Entra ID Access Reviews).

Compliance (SOC2, ISO 27001) exige evidência.

## Identity types

| Tipo | Exemplos | Auth típica |
|---|---|---|
| **Human user** | Funcionários, clientes finais | [[oidc]] + [[mfa-step-up]] |
| **Service account** | M2M dentro da org | Client Credentials OAuth |
| **Workload identity** | Pods, containers, Lambda | [[spiffe-spire]], AWS IAM Role for Service Accounts |
| **External partner** | B2B federation | [[saml-2]] ou [[oidc]] federated |
| **Device** | Laptops corporativos, IoT | Client certificates, WARP enrollment, mTLS |

## IAM em cloud (AWS / GCP / Azure)

### AWS IAM + IAM Identity Center (antes SSO)

- **IAM Users** (legado): credentials permanentes; evitar em favor de IAM Identity Center.
- **IAM Roles**: assumíveis via STS; short-lived; preferido para human + workload.
- **IAM Identity Center**: SSO enterprise para múltiplas contas AWS.
- **SCP (Service Control Policies)** no Organizations: guardrail top-down.

### Google Cloud IAM

- **IAM Policies** no resource (binding): `role → principal`.
- **Workload Identity Federation**: identidade externa (GitHub Actions, Okta) → GCP sem static keys.
- **Access Context Manager**: ABAC context (device, IP, geo).

### Microsoft Entra ID (antes Azure AD)

- **Users / Groups / Service Principals** (managed identities).
- **Conditional Access**: ABAC-like policies (device compliance, sign-in risk).
- **PIM**: privileged role elevation temporária.

## IAM Governance (IGA) — separado de IAM operacional

Ferramentas IGA (Identity Governance & Administration):
- **SailPoint**, **Saviynt**, **Omada**: enterprise.
- **Okta Identity Governance**, **Entra ID Governance**: integrado a IdPs modernos.

Responsável por:
- Access reviews
- Role mining (discover roles from usage patterns)
- Separation of Duties enforcement
- Regulatory reporting

## IAM vs CIAM

- **IAM** (workforce): funcionários, parceiros. Ênfase em governance + compliance.
- **CIAM** (Customer IAM): end-users do produto. Ênfase em UX, self-service, escala (milhões de accounts), privacy (LGPD/GDPR).

Produtos como Auth0, Zitadel, Firebase Auth, Cognito são CIAM-friendly.

## Padrões canônicos

### Strong authn-authz separation

```
┌────────────┐    authn     ┌──────────┐
│  Client    │─────────────▶│  IdP     │   (OIDC, SAML)
└────────────┘              └──────────┘
       │ token                     │
       ▼                            │
┌────────────┐    authz     ┌──────────┐
│  Resource  │─────────────▶│  PDP     │   (OPA, ABAC engine)
│  Server    │              └──────────┘
└────────────┘
```

### Identity as a Service (IDaaS)

Externalize autenticação/governance. Core do produto **não** implementa auth; delega a IdP.

Pros: segurança madura, compliance incluso, MFA/WebAuthn de graça.
Cons: dependência externa; custos escalam com seats; lock-in potencial.

## Anti-patterns

1. **IAM como "database de users"** — mistura responsabilidades. IAM é serviço.
2. **Não rodar access reviews** — permissions drift por anos.
3. **Não automatizar offboarding** — termination demora dias → janela de ataque.
4. **Service accounts como usuários humanos** — sem owner, sem rotation, sem audit focado.
5. **Roles compartilhados** ("admin" global em planilha compartilhada) — audit impossível.
6. **Confiar em perímetro de rede** em vez de identidade — ver [[beyond-corp]].
7. **Um IdP master sem DR** — se IdP cai, todo mundo fica fora.
8. **CIAM sem rate limit / bot defense** — credential stuffing.
9. **IAM próprio (build) em vez de comprado** — security debt acumulada; use Zitadel/Keycloak se open-source é requisito.

## Como grandes empresas fazem

- **Google**: BeyondCorp (ver [[beyond-corp]]) + workforce via internal IAM + GSuite. Todo request é autenticado + autorizado + auditado.
- **Microsoft**: Entra ID é produto e também usado internamente. Conditional Access + PIM para elevação.
- **Netflix**: "Bless" SSH CA + internal BAM (Broad Authorization/Identity Management).
- **Airbnb**: "Himeji" (Zanzibar-like) para authz; IdP próprio para authn.
- **Auth0 / Okta**: são CIAM/IAM SaaS; milhares de empresas delegam.

## Relações

- **Authentication**: [[identity-provider]], [[oidc]], [[oauth-2]], [[saml-2]], [[sso]], [[mfa-step-up]]
- **Authorization**: [[authorization-models]], [[rbac]], [[abac]], [[rebac]]
- **Lifecycle**: [[scim-provisioning]]
- **Delegation**: [[delegation-impersonation]]
- **Workload**: [[spiffe-spire]]
- **Zero Trust**: [[beyond-corp]]
- **Princípio**: [[least-privilege]]
- **Providers concretos**: [[../providers/zitadel/_moc]], [[../providers/cloudflare/access]]

## Fontes

- [NIST SP 800-63 — Digital Identity Guidelines](https://pages.nist.gov/800-63-3/) (consultada 2026-04-24)
- [Gartner IAM category](https://www.gartner.com/reviews/market/access-management) (consultada 2026-04-24)
- [AWS IAM docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/) (consultada 2026-04-24)
- [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/) (consultada 2026-04-24)
- [Google Cloud IAM](https://cloud.google.com/iam/docs) (consultada 2026-04-24)
- [Okta — IAM vs CIAM](https://www.okta.com/identity-101/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[identity-provider]]
- [[sso]]
- [[oidc]]
- [[authorization-models]]
- [[least-privilege]]
- [[beyond-corp]]
