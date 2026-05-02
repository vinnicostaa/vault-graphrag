---
title: MOC — Security
type: moc
tags:
  - moc
  - security
created: 2026-04-22
updated: 2026-04-24
aliases:
  - Security MOC
---

# Security

Cluster de knowledge atemporal sobre segurança — autorização, identidade, protocolos, sessões, workload identity, compliance. Ancora [[owasp-top-10|OWASP Top 10]] nas notas específicas. Aplicações concretas em `50-projects/<slug>/08-security/` + providers em [[../providers/zitadel/_moc]] e [[../providers/cloudflare/access|CF Access]].

> Expansão 2026-04-24 ([[../../00-meta/STATE|D-vault-021]]): +28 notas cobrindo RBAC/ABAC/ACL/ReBAC, IAM/IdP/SSO/MFA, OAuth/OIDC/JWT/SAML/PKCE, WebAuthn/passkeys/OTP/magic links, OPA/Zanzibar/SPIFFE/SCIM, session management, delegation, secrets, client-side storage, cookies, CSRF.

---

## 🧭 Por onde começar?

| Pergunta | Vai para |
|---|---|
| "Qual modelo de autorização usar?" | [[authorization-models]] (decision tree) |
| "Como user loga?" | [[identity-provider]] + [[oidc]] |
| "Token roubado — como mitigar?" | [[session-management]] + [[jwt]] §sender-constrained |
| "Cookie httpOnly ou localStorage?" | [[credential-storage]] |
| "Webhook seguro?" | [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]] + [[security-principles#Webhook-Signature]] |
| "Secrets no CI?" | [[secrets-management]] |
| "Passkey ou TOTP?" | [[webauthn-passkeys]] vs [[otp-hotp-totp]] |
| "Quem acessou recurso X?" | [[authorization-models]] + audit em [[security-principles]] |

---

## 📚 Notas por categoria

### 🏛️ Fundamentos / operação

- [[security-principles]] — **regra operacional** (middleware stack, ABAC engine, rate limit, HMAC, LGPD, 1-device policy, idempotency, anti-fraude)
- [[beyond-corp]] — Zero Trust (Google BeyondCorp, NIST 800-207); modelo conceitual
- [[least-privilege]] — **princípio raiz** (POLP; aplicação cross-domain)

### 🔐 Authorization models (C1 — D-vault-021)

- [[authorization-models]] — **decision tree** RBAC vs ABAC vs ACL vs ReBAC vs PBAC
- [[rbac]] — Role-Based (NIST ANSI/INCITS 359)
- [[abac]] — Attribute-Based (magra; linka [[security-principles#ABAC-Engine]])
- [[acl]] — Access Control List (POSIX, S3 legacy, NTFS)
- [[rebac]] — Relationship-Based (Zanzibar)
- [[opa-rego]] — PBAC / policy-as-code (OPA, Rego)
- [[zanzibar-openfga]] — ReBAC engines (OpenFGA, SpiceDB)

### 🪪 Identity (C2 — D-vault-021)

- [[iam]] — Identity and Access Management (guarda-chuva)
- [[identity-provider]] — IdP (componente concreto)
- [[sso]] — Single Sign-On
- [[mfa-step-up]] — MFA + Step-up authentication
- [[webauthn-passkeys]] — FIDO2 phishing-resistant (W3C WebAuthn L3)
- [[otp-hotp-totp]] — OTP / HOTP (RFC 4226) / TOTP (RFC 6238) + desambiguação Erlang/OTP
- [[magic-links]] — passwordless via email

### 🔑 Protocolos (C3 — D-vault-021)

- [[oauth-2]] — OAuth 2.0 / 2.1 (RFC 9700 BCP Set/2024 baseline)
- [[oidc]] — OpenID Connect (authn em cima de OAuth)
- [[pkce]] — Proof Key for Code Exchange (RFC 7636)
- [[jwt]] — JSON Web Tokens + BCP (RFC 8725) + DPoP (RFC 9449) + mTLS-bound (RFC 8705)
- [[saml-2]] — SAML 2.0 (enterprise legado)

### 👷 Workload & provisioning (C4 — D-vault-021)

- [[spiffe-spire]] — SPIFFE/SPIRE workload identity (CNCF)
- [[scim-provisioning]] — SCIM 2.0 (RFC 7643/7644) — auto-provisioning
- [[delegation-impersonation]] — act-as, on-behalf-of, OAuth Token Exchange (RFC 8693)

### 🍪 Sessions & credentials (C5 — D-vault-021)

- [[session-management]] — server-side session, rotation, revocation, fixation
- [[cookies]] — RFC 6265bis, CHIPS, `__Host-` / `__Secure-` prefixes
- [[credential-storage]] — client-side (browser + mobile + desktop)
- [[csrf-defense]] — SameSite + double-submit + Origin + Sec-Fetch-*

### 🔐 Secrets (infra)

- [[secrets-management]] — Vault, SOPS, KMS, age, PGP (infra/dev side)

### 🛡️ Application Security & Compliance

- [[owasp-top-10]] — OWASP Top 10 2021 (mapping para notas deste cluster em A01/A07)
- [[cve-response-workflow]] — CVE/CVSS, triage, SLA (D-vault-009)

---

## 🔗 Mapa de cross-references (alta densidade)

### Por OWASP Top 10

| OWASP | Notas-chave |
|---|---|
| **A01 Broken Access Control** | [[authorization-models]], [[rbac]], [[abac]], [[rebac]], [[opa-rego]], [[security-principles]] |
| **A02 Cryptographic Failures** | [[credential-storage]], [[secrets-management]], [[jwt]] |
| **A03 Injection** | [[security-principles#Validação-de-input]] |
| **A05 Security Misconfiguration** | [[cookies]], [[csrf-defense]], [[secrets-management]] |
| **A06 Vulnerable Components** | [[cve-response-workflow]] |
| **A07 Auth Failures** | [[identity-provider]], [[oidc]], [[jwt]], [[session-management]], [[mfa-step-up]], [[webauthn-passkeys]], [[otp-hotp-totp]] |
| **A08 Data Integrity** | [[security-principles#Webhook-Signature]] |
| **A09 Logging/Monitoring** | [[../runtime-antipatterns/op-021-admin-sem-audit\|OP-021]], [[../runtime-antipatterns/op-022-log-com-pii\|OP-022]] |
| **A10 SSRF** | [[security-principles]] §SSRF |

### Por providers (implementação concreta)

- **[[../providers/zitadel/_moc]]** — implementa [[oidc]], [[oauth-2]], [[saml-2]], [[scim-provisioning]]
- **[[../providers/cloudflare/access]]** — BeyondCorp implementation via [[oidc]]; JWT validation (ver [[jwt]] §CF Access)
- **[[../providers/cloudflare/tunnel]]** — ZTNA substitui VPN ([[beyond-corp]])
- **[[../providers/stripe/_moc]]** — webhook HMAC ([[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]])
- **[[../providers/aws-docs/_moc]]** — AWS IAM (RBAC+ABAC), SCIM, KMS

### Por runtime antipatterns

- [[../runtime-antipatterns/op-003-webhook-sem-idempotencia|OP-003]], [[../runtime-antipatterns/op-004-sem-retry-strategy|OP-004]] — webhooks
- [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]] — authz stale
- [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]] — policy espalhada
- [[../runtime-antipatterns/op-021-admin-sem-audit|OP-021]] — bypass sem audit
- [[../runtime-antipatterns/op-022-log-com-pii|OP-022]] — PII em log
- [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]] — webhook sem HMAC
- [[../runtime-antipatterns/op-024-rate-limit-ausente|OP-024]] — rate limit ausente

---

## 🎓 Leitura recomendada por perfil

### Dev novo no projeto
1. [[security-principles]] (overview prático)
2. [[beyond-corp]] (mental model)
3. [[authorization-models]] (decision tree)
4. [[cookies]] + [[session-management]] + [[credential-storage]]

### Implementando SSO
1. [[sso]] (modelo)
2. [[oidc]] (protocolo)
3. [[pkce]] (segurança do code flow)
4. [[jwt]] (formato)
5. [[identity-provider]] (escolha de IdP)
6. [[mfa-step-up]] + [[webauthn-passkeys]] (factors)

### Implementando autorização
1. [[authorization-models]]
2. [[least-privilege]]
3. [[abac]] ou [[rbac]] ou [[rebac]] conforme decision tree
4. [[opa-rego]] (se PBAC)

### Compliance / governance
1. [[iam]] (lifecycle)
2. [[scim-provisioning]]
3. [[delegation-impersonation]]
4. [[cve-response-workflow]]

### Zero Trust / infra moderna
1. [[beyond-corp]]
2. [[spiffe-spire]] (workload)
3. [[secrets-management]]
4. [[../providers/cloudflare/access]] + [[../providers/cloudflare/tunnel]]

---

## 📜 Regras duras do cluster

- **Evitar duplicação**: notas satélites (como [[abac]]) linkam [[security-principles]] em vez de repetir código. Decision tree só em [[authorization-models]].
- **Data de consulta obrigatória**: URLs externas têm `(consultada YYYY-MM-DD)`. Frontmatter `doc-consulted` canônico.
- **Cross-ref denso**: cada nota tem ≥ 3 wikilinks; mínimo inclui [[_moc]], [[security-principles]] quando aplicável, e nota-pare conceitual.
- **Escopo**: knowledge atemporal. Aplicação a projeto específico vai em `50-projects/<slug>/08-security/`.
- **Nunca**: hardcoded secrets (ver [[secrets-management]]); token em localStorage ([[credential-storage]]); webhook sem HMAC ([[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]]); auth em frontend ([[security-principles]] §1).

---

## 🗃️ Histórico de expansão

- **2026-04-22**: `security-principles`, `beyond-corp` (ingestão inicial).
- **2026-04-24**: `owasp-top-10`, `cve-response-workflow` (D-vault-009).
- **2026-04-24**: +28 notas expansão cluster (D-vault-021). Ver [[../../00-meta/STATE|STATE]] para detalhes.

---

## 🧭 Vizinhos no grafo

- [[../_moc]] — 10-knowledge raiz
- [[../../00-meta/STATE]] — D-vault-009, D-vault-021
- [[../../00-meta/AUDIT]] — compliance findings
- [[../runtime-antipatterns/_moc]] — OPs relacionados a segurança
- [[../antipatterns/_moc]] — code smells (complementar)
- [[../providers/_moc]] — implementações concretas
- [[../../30-agents/playbooks/cve-scan]] — playbook operacional
- [[../architecture/clean-architecture-layers]] — onde authz vive na arquitetura

## 📖 Fontes-chave do cluster

Todas as notas linkam fontes específicas. Principais:

- [NIST SP 800-63B Digital Identity](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [NIST SP 800-162 ABAC](https://csrc.nist.gov/publications/detail/sp/800-162/final)
- [NIST SP 800-207 Zero Trust](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [OWASP Top 10 2021](https://owasp.org/Top10/)
- [OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)
- [IETF OAuth WG — RFCs](https://datatracker.ietf.org/wg/oauth/documents/)
- [OpenID Foundation](https://openid.net/)
- [FIDO Alliance / Passkeys](https://passkeys.dev/)
- [Google Zanzibar paper](https://research.google/pubs/pub48190/)
