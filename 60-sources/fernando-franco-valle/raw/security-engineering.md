[FFV Academy](../index.html)/Security Engineering

🛡️

Blog

# Security Engineering

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

10artigos

595XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/threat-modeling-stride/index.html)

01

## 🎯 Threat modeling com STRIDE: de onde vêm os ataques

STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege) como checklist pragmático; DFD (data flow diagram) pra visualizar surface; priorizar por DREAD ou Risk Matrix; integrar no design review.

⏱ 13 min·+55 XP

→[](../aprenda/authn-vs-authz/index.html)

02

## 🔑 Authn vs Authz: a diferença e as armadilhas

Autenticação (quem é você) vs Autorização (o que pode fazer). Por que misturar no mesmo middleware é bug comum. RBAC vs ABAC vs ReBAC (Zanzibar/SpiceDB). Session vs stateless (JWT). Como TIPAR o Request\<User\> no TS pra nunca acessar endpoint sem authn.

⏱ 11 min·+50 XP

→[](../aprenda/oauth2-oidc-do-zero/index.html)

03

## 🔐 OAuth2 e OIDC do zero: fluxos e PKCE

Authorization Code + PKCE (único fluxo aceitável em 2026), Client Credentials (machine-to-machine), Device Code (CLI/TV). OIDC sobre OAuth2: id_token, scopes, claims. Erros fatais: implicit flow, ROPC. Como debugar token inválido sem expor secret.

⏱ 15 min·+65 XP

→[](../aprenda/jwt-paseto-sessions/index.html)

04

## 🎫 JWT, Paseto ou sessions: quando cada um

JWT: quando é bom (stateless, edge), quando é péssimo (logout, revoke, claim bloat). Paseto v4 como alternativa moderna (sem algorithm confusion). Sessions com Redis pra apps clássicos. Refresh token rotation, sliding expiration, revocation lists.

⏱ 12 min·+55 XP

→[](../aprenda/password-hashing-moderno/index.html)

05

## 🧂 Password hashing moderno: argon2, bcrypt, peppers

Por que SHA-256 puro é crime em 2026. Argon2id (OWASP \#1), bcrypt (aceitável), scrypt. Salt sempre, pepper opcional (server-side secret extra). Timing-safe compare. Migrar hash antigo sem quebrar login (re-hash no login).

⏱ 10 min·+45 XP

→[](../aprenda/owasp-top-10-com-exemplo-em-codigo/index.html)

06

## 📋 OWASP Top 10 (2024) com exemplo em código

A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection, A04 Insecure Design, A05 Security Misconfig, A06 Vulnerable Components, A07 Identification & Authn Failures, A08 Software & Data Integrity Failures, A09 Logging Failures, A10 SSRF. Cada um com código ruim → código seguro em TS.

⏱ 16 min·+70 XP

→[](../aprenda/secrets-management/index.html)

07

## 🗝️ Secrets management: Vault, SOPS e AWS Secrets Manager

Secret em env var é pior que sem segredo. HashiCorp Vault (dynamic secrets, leases), SOPS (git-friendly com KMS/age), AWS Secrets Manager (rotation automática), Doppler pra times pequenos. Padrão: secret never leaves vault; aplicação pede short-lived.

⏱ 12 min·+55 XP

→[](../aprenda/supply-chain-security/index.html)

08

## 📦 Supply chain: SBOM, sigstore e dependency confusion

SolarWinds, XZ backdoor: atacante entra via deps. SBOM (CycloneDX, SPDX) pra inventário. Sigstore (cosign, rekor) pra assinatura verificável. Scanning com Trivy, Snyk, Dependabot, Socket.dev. Dependency confusion: como namespace privado vaza pra npm público.

⏱ 13 min·+60 XP

→[](../aprenda/zero-trust-e-mtls/index.html)

09

## 🚪 Zero Trust e mTLS: verificar sempre, nunca confiar na rede

Princípios Zero Trust (verify explicitly, least privilege, assume breach). mTLS pra autenticação mútua entre serviços. SPIFFE/SPIRE pra identidade de workload. Service mesh (Istio, Linkerd) como camada. Alternativa simples: Tailscale/Cloudflare Zero Trust em dev.

⏱ 12 min·+55 XP

→[](../aprenda/capstone-pentest-em-app-proprio/index.html)

10

## 🏁 Capstone: pentest em app próprio (ético)

Pegar app que você construiu (ex: Trail 21 capstone) e atacar: Burp Suite pra MITM, ffuf pra fuzzing, nuclei pra scan automatizado, sqlmap pra testar injection. Documentar cada vuln encontrada, severity (CVSS), fix aplicado, regression test. Entregável: relatório de pentest + PRs corrigindo.

⏱ 18 min·+85 XP

→

[← Voltar à home](../index.html)
