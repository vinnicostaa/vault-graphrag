---
title: Beyond-Corp References — Ponte com 13-research
type: spec
tags: [security, beyond-corp, references, research-bridge, master-sindico, v2]
source: 13-research/beyond-corp/source-01 a source-12
created: 2026-04-23
updated: 2026-04-23
aliases: [beyond-corp-references, bc-references]
---

# Beyond-Corp References — Ponte com 13-research

Catálogo de fontes canônicas do research destilado em [[../13-research/beyond-corp/_destilado]]. Serve de:

1. **Dual-check futuro**: quando alguém questionar um controle em [[BEYOND_CORP]], aponta pra fonte oficial.
2. **Bibliografia** pra auditor externo / due-diligence enterprise.
3. **Referência pros 7 tenets NIST literal** (evitar alucinação em citações).
4. **Reprodutibilidade**: URLs + datas de acesso permitem revalidar.

---

## Índice rápido

| # | Fonte | Ano | Relevância | Link research |
|---|---|---|---|---|
| 01 | BeyondCorp: A New Approach to Enterprise Security (Ward & Beyer) | 2014 | 🔴 Canônico | [[../13-research/beyond-corp/source-01]] |
| 02 | **NIST SP 800-207 Zero Trust Architecture** | 2020 | 🔴 Canônico (normativo) | [[../13-research/beyond-corp/source-02]] |
| 03 | BeyondCorp: Design to Deployment at Google | 2016 | 🔴 Canônico | [[../13-research/beyond-corp/source-03]] |
| 04 | BeyondCorp Part III: The Access Proxy | 2016 | 🔴 Canônico | [[../13-research/beyond-corp/source-04]] |
| 05 | **BeyondProd** — Cloud-Native Security at Google | 2019 | 🔴 Canônico (leste-oeste) | [[../13-research/beyond-corp/source-05]] |
| 06 | **CISA Zero Trust Maturity Model v2** | 2023 | 🔴 Canônico (auditoria) | [[../13-research/beyond-corp/source-06]] |
| 07 | Google Cloud Identity-Aware Proxy (IAP) | docs | 🟡 Alto | [[../13-research/beyond-corp/source-07]] |
| 08 | AWS Zero Trust — Prescriptive Guidance | docs | 🟡 Alto | [[../13-research/beyond-corp/source-08]] |
| 09 | Microsoft Zero Trust Overview | docs | 🟢 Médio | [[../13-research/beyond-corp/source-09]] |
| 10 | Cloudflare One / Zero Trust | docs | 🟡 Alto | [[../13-research/beyond-corp/source-10]] |
| 11 | Tailscale — Zero Trust Networking | docs | 🟢 Médio | [[../13-research/beyond-corp/source-11]] |
| 12 | Okta Identity-centric Zero Trust | docs | 🟢 Médio | [[../13-research/beyond-corp/source-12]] |

---

## Fonte 01 — BeyondCorp: A New Approach to Enterprise Security (Ward & Beyer, 2014)

- **URL oficial**: https://research.google/pubs/beyondcorp-a-new-approach-to-enterprise-security/
- **URL USENIX**: https://www.usenix.org/publications/login/dec14/ward
- **PDF**: https://www.usenix.org/system/files/login/articles/login_dec14_02_ward.pdf
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-01]]

### Trechos literais aplicados no master-sindico

> "Virtually every company today uses firewalls to enforce perimeter security. However, this security model is problematic because, when that perimeter is breached, an attacker has relatively easy access to a company's privileged intranet."
> — Ward & Beyer, 2014, p. 6

Aplicado em [[BEYOND_CORP]] §1 (premissa de zero rede confiável) + [[threat-model]] §3 (trust boundaries).

> "Access depends on the device and the user rather than the network from which the device connects."
> — síntese do modelo

Aplicado em [[BEYOND_CORP]] §4 (1-device + fingerprint + trust_tier) + [[pentest-checklist]] §1.3.

---

## Fonte 02 — NIST SP 800-207 Zero Trust Architecture (2020)

- **URL oficial**: https://csrc.nist.gov/pubs/sp/800/207/final
- **PDF**: https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-207.pdf
- **Release note**: https://www.nist.gov/news-events/news/2020/08/zero-trust-architecture-nist-publishes-sp-800-207
- **Autores**: Scott Rose, Oliver Borchert, Stu Mitchell, Sean Connelly (NIST)
- **Publicação**: 11 de agosto de 2020
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-02]]

### Os 7 tenets literal (NIST §2.1)

Referência oficial pros tenets traduzidos em [[BEYOND_CORP]] §1:

> **Tenet 1**: "All data sources and computing services are considered resources."

> **Tenet 2**: "All communication is secured regardless of network location."

> **Tenet 3**: "Access to individual enterprise resources is granted on a per-session basis."

> **Tenet 4**: "Access to resources is determined by dynamic policy — including the observable state of client identity, application/service, and the requesting asset — and may include other behavioral and environmental attributes."

> **Tenet 5**: "The enterprise monitors and measures the integrity and security posture of all owned and associated assets."

> **Tenet 6**: "All resource authentication and authorization are dynamic and strictly enforced before access is allowed."

> **Tenet 7**: "The enterprise collects as much information as possible about the current state of assets, network infrastructure and communications and uses it to improve its security posture."

### Componentes lógicos (NIST §3.1)

Referência oficial pros PE/PA/PEP em [[BEYOND_CORP]] §12.2:

> "The **Policy Engine (PE)** is responsible for the ultimate decision to grant access to a resource for a given subject. (...) The PE is paired with the policy administrator component."

> "The **Policy Administrator (PA)** is responsible for establishing and/or shutting down the communication path between a subject and a resource."

> "The **Policy Enforcement Point (PEP)** is responsible for enabling, monitoring, and eventually terminating connections between a subject and an enterprise resource."

### Aplicação no master-sindico

- **PE**: OPA (sidecar M2) ou Cedar — ⚠️ ADR pendente.
- **PA**: BC-identity (serviço de sessão emite/renova/revoga tokens).
- **PEP**: middleware ABAC em cada BC + API Gateway.
- Ver [[BEYOND_CORP]] §2 (stack) + §12.2 (PE/PA/PEP).

---

## Fonte 03 — BeyondCorp: Design to Deployment at Google (Osborn et al., 2016)

- **URL oficial**: https://research.google/pubs/beyondcorp-design-to-deployment-at-google/
- **PDF**: https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44860.pdf
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-03]]

### Aplicação

- **Device Inventory + Trust Tiers**: aplicado em [[BEYOND_CORP]] §4 (tabela `devices` + trust_tier).
- **ACE (Access Control Engine)**: abstração do nosso ABAC engine.

---

## Fonte 04 — BeyondCorp Part III: The Access Proxy (Cittadini et al., 2016)

- **URL oficial**: https://research.google/pubs/beyondcorp-part-iii-the-access-proxy/
- **PDF**: https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45728.pdf
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-04]]

### Aplicação

- **Access Proxy coarse-grained**: API Gateway (Gin abstraído) + middleware stack em [[BEYOND_CORP]] §2.
- **Divisão coarse (AP) vs fine (backend)**: middleware authN/rate limit + ABAC fine no handler.

---

## Fonte 05 — BeyondProd (Google Cloud, 2019)

- **URL oficial**: https://cloud.google.com/docs/security/beyondprod
- **Anúncio blog**: https://cloud.google.com/blog/products/identity-security/beyondprod-whitepaper-discusses-cloud-native-security-at-google
- **Mirror PDF**: https://price2meet.com/gcp/docs/security_beyondprod.pdf
- **Autores**: Maya Kaczorowski + time security Google Cloud
- **Publicação**: Dezembro 2019
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-05]]

### Trechos literais

> "Service trust should depend on characteristics like code provenance, trusted hardware, and service identity, rather than the location in the production network."
> — BeyondProd whitepaper

Aplicado em [[BEYOND_CORP]] §12.5-6 (SPIFFE/SPIRE + mTLS leste-oeste).

> "Only authenticated, trusted, and specifically authorized callers or services can access any other service."

Aplicado em [[threat-model]] §3 (TZ 3 → TZ 4 requires mTLS M2+).

> "ALTS is a mutual authentication and transport encryption system for services in Google infrastructure."

ALTS ≈ **SPIFFE/SPIRE + Istio/Linkerd mTLS** no ecossistema open. Roadmap M3 em [[BEYOND_CORP]] §14.3.

> "BAB is a deploy-time enforcement check that helps ensure that code meets internal security requirements before the code is deployed."

Binary Authorization = cosign + Kyverno / OPA Gatekeeper. Roadmap M3 em [[BEYOND_CORP]] §14.3.

### Mapeamento Google → Open Source

| Google | Open source equivalent | Aplicação master-sindico |
|---|---|---|
| ALTS | SPIFFE/SPIRE + Istio/Linkerd | M3+ |
| BAB | Sigstore + cosign + Kyverno/OPA Gatekeeper | M3+ |
| Borg | Kubernetes | M3 (quando sair Railway) |
| gVisor | gVisor OSS / Kata Containers / Firecracker | M4+ |
| Google Security Ticket | Propagate user identity via signed headers | M2+ |

---

## Fonte 06 — CISA Zero Trust Maturity Model v2 (2023)

- **URL oficial (landing)**: https://www.cisa.gov/zero-trust-maturity-model
- **PDF**: https://www.cisa.gov/sites/default/files/2023-04/zero_trust_maturity_model_v2_508.pdf
- **Release note**: https://www.cisa.gov/news-events/news/cisa-releases-updated-zero-trust-maturity-model
- **Publicação**: v2.0 em Abril 2023 (v1.0 em 2021 em resposta ao US Executive Order 14028)
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-06]]

### Estrutura (canônica)

**5 pilares**:
1. Identity
2. Devices
3. Networks
4. Applications & Workloads
5. Data

**3 capacidades transversais**:
- Visibility & Analytics
- Automation & Orchestration
- Governance

**4 estágios**: Traditional → Initial → Advanced → Optimal

### Aplicação master-sindico

Matriz 5×4 em [[BEYOND_CORP]] §13 (mapping pilar × BC) + roadmap por estágio §14.

**Por que importa**: clientes corporate (administradoras 100+ prédios) pedem evidência de maturidade em due-diligence. ZTMM é vocabulário compartilhado com auditor e comprador técnico.

---

## Fonte 07 — Google Cloud Identity-Aware Proxy (IAP)

- **URL principal**: https://cloud.google.com/security/products/iap
- **URL docs**: https://cloud.google.com/iap/docs
- **URL context-aware**: https://docs.cloud.google.com/iap/docs/cloud-iap-context-aware-access-howto
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-07]]

### Trechos-chave

> "IAP lets you establish a central authorization layer for applications accessed by HTTPS, so you can use an application-level access control model instead of relying on network-level firewalls."

> "To limit access based on IP address or end-user device attributes, you can create an access level."

### Aplicação master-sindico

**Parcial**: IAP seria caminho rápido **se rodássemos em GCP**. Hoje Railway (M1) + decisão aberta AWS/GCP (M2+).

Lições independentes de produto aplicadas:
1. Separar authN (proxy) de authZ fine-grained (backend) — ver [[BEYOND_CORP]] §2.
2. JWT assinado entre proxy e backend — nosso pattern interno.
3. Access Levels = ABAC matrix — ver [[BEYOND_CORP]] §3.
4. Separar acesso admin (equipe interna) de acesso produto (end-user) — roadmap M2+.

⚠️ PENDÊNCIA: decisão cloud final (AWS vs GCP) impacta adoção direta do IAP.

---

## Fonte 08 — AWS Zero Trust — Prescriptive Guidance

- **URL principal**: https://docs.aws.amazon.com/prescriptive-guidance/latest/strategy-zero-trust-architecture/welcome.html
- **URL Verified Access**: https://aws.amazon.com/verified-access/
- **URL Verified Permissions (Cedar)**: https://aws.amazon.com/verified-permissions/
- **URL VPC Lattice**: https://aws.amazon.com/vpc/lattice/
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-08]]

### Princípios (síntese AWS)

1. Authenticate and authorize every request (identity-first).
2. Shrink implicit trust zones (micro-segmentation).
3. Integrate network, identity, and application policy.

### Produtos mapeáveis (se AWS-first)

| AWS | Função | Equivalente master-sindico |
|---|---|---|
| AWS Verified Access | BeyondCorp-as-a-service (browser access) | Para admin console M2+ |
| AWS Verified Permissions | PDP gerenciado (Cedar) | ⚠️ ADR OPA vs Cedar |
| VPC Lattice | Service-to-service policy (L7) | Alternativa a Istio/Linkerd |
| IAM + STS | Identity + tokens | Zitadel (end-user); IAM pra ops |
| AWS WAF | Edge protection | ⚠️ ADR Cloudflare vs AWS WAF |
| AWS Secrets Manager | Secret storage | M2+ migration |

---

## Fonte 09 — Microsoft Zero Trust Overview

- **URL principal**: https://www.microsoft.com/en-us/security/business/zero-trust
- **URL docs**: https://learn.microsoft.com/en-us/security/zero-trust/zero-trust-overview
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-09]]

### 3 princípios didáticos

1. **Verify explicitly** — autenticar/autorizar com todos os sinais (identity, location, device health, data classification, anomalies).
2. **Use least privilege access** — JIT/JEA, políticas adaptativas por risco.
3. **Assume breach** — minimize blast radius, segment, encrypt end-to-end, analyze.

### Aplicação

Framing de 1 linha por princípio ajuda comunicação com stakeholder não-técnico. Ver [[BEYOND_CORP]] §1 (princípios) + §12.7 (JIT/JEA).

---

## Fonte 10 — Cloudflare One / Zero Trust

- **URL principal**: https://www.cloudflare.com/zero-trust/
- **URL docs**: https://developers.cloudflare.com/cloudflare-one/
- **URL Access**: https://www.cloudflare.com/plans/zero-trust-services/
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-10]]

### Componentes

- **Access**: BeyondCorp-style para apps internas (SSO + policy + MFA).
- **Tunnel**: acesso seguro sem abrir portas públicas.
- **Gateway**: DNS/HTTP filtering outbound.
- **WAF**: edge protection com OWASP CRS.
- **Browser Isolation**: sandbox browser.
- **DLP**: data loss prevention.

### Aplicação

**Prováveis pra master-sindico** (ADR aberto):
- **Cloudflare WAF + DDoS**: edge protection M2.
- **Cloudflare Access**: admin console pra equipe interna (vs VPN).
- **Cloudflare Tunnel**: dev/staging sem expor portas.

Alternativas: AWS WAF + Verified Access.

---

## Fonte 11 — Tailscale Zero Trust Networking

- **URL principal**: https://tailscale.com/
- **URL kb**: https://tailscale.com/kb/
- **URL ACLs**: https://tailscale.com/kb/1018/acls/
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-11]]

### Aplicação

**Foco em admin access**:
- Mesh VPN WireGuard entre máquinas dev + servers staging/prod.
- ACLs identity-based (não IP-based).
- SSH CA (cert curto) em vez de SSH keys permanentes.

Provável pra **acesso de ops/devs a servers** (JIT/JEA em [[BEYOND_CORP]] §12.7). Não aplicável a end-user do SaaS.

---

## Fonte 12 — Okta Identity-centric Zero Trust

- **URL principal**: https://www.okta.com/resources/whitepaper/the-state-of-zero-trust-security/
- **URL Device Assurance**: https://www.okta.com/products/device-assurance/
- **URL Adaptive MFA**: https://www.okta.com/products/adaptive-multi-factor-authentication/
- **Data de acesso**: 2026-04-23
- **Detalhe**: [[../13-research/beyond-corp/source-12]]

### 5 níveis maturidade identity-powered ZT

1. Fragmented identity
2. Unified identity
3. Context-based access
4. Adaptive workforce
5. Identity-powered ZT

### Aplicação

- **Adaptive MFA**: step-up baseado em risco — ver [[BEYOND_CORP]] §12.3.
- **Device Assurance**: atesta posture do device antes de permitir — ver [[BEYOND_CORP]] §4.3 (M2 Play Integrity / DeviceCheck).
- **Passwordless (passkeys)**: WebAuthn prioritário — roadmap M3 em [[BEYOND_CORP]] §14.3.

Alternativa direta ao Zitadel se ADR IdP reabrir.

---

## Fontes secundárias (não-destiladas mas consultadas)

### NIST SP 800-207A (2023) — ZT em ambientes cloud/multi-cloud

- URL: https://csrc.nist.gov/pubs/sp/800/207/a/final
- Relevante se **master-sindico mirar multi-cloud** (M4+).
- **⚠️ PENDÊNCIA**: ler + destilar se decisão cloud multi-region for tomada.

### OWASP Top 10 2021

- URL: https://owasp.org/Top10/
- Mapeado integralmente em [[owasp-top10]].
- Próxima atualização prevista 2025.

### OWASP ASVS (Application Security Verification Standard)

- URL: https://owasp.org/www-project-application-security-verification-standard/
- Checklist mais detalhada que Top 10.
- **⚠️ PENDÊNCIA M2**: mapear Level 2 completo.

### CIS Benchmarks

- URL: https://www.cisecurity.org/cis-benchmarks
- Hardening específico (Docker, K8s, Postgres).
- **⚠️ PENDÊNCIA M3**: aplicar CIS Docker + K8s quando migrar ECS/EKS.

### Google BeyondCorp papers IV e V (Maintaining Productivity / User Experience)

- IV: https://research.google/pubs/beyondcorp-the-user-experience/
- Relevante pra UX de MFA / step-up / device onboarding.
- **⚠️ PENDÊNCIA**: ler pré-M2 pra informar UX de MFA obrigatório.

### SOC 2 Type II requirements

- URL: https://www.aicpa-cima.com/resources/landing/system-and-organization-controls-soc-suite-of-services
- **⚠️ PENDÊNCIA M3+**: kickoff com auditor se cliente enterprise pedir.

### ISO/IEC 27001:2022

- **⚠️ PENDÊNCIA M3+**: avaliar custo-benefício vs SOC 2.

### Lei 13.709/2018 (LGPD)

- URL: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm
- Mapeada em [[lgpd]].

### Lei 14.063/2020 (assinaturas eletrônicas)

- URL: https://www.planalto.gov.br/ccivil_03/_ato2019-2022/2020/lei/l14063.htm
- Relevante pra ata + contratos Connect Me.
- Aplicação em [[../04-requirements/functional/assembly]] (a escrever) + [[../04-requirements/functional/commercial]] (a escrever).

---

## Dual-check discipline

Sempre que alguém (dual-check reviewer, QA, pentester externo) questionar um controle em BEYOND_CORP / threat-model / owasp-top10 / pentest-checklist / lgpd / cve-process, a resposta deve:

1. Apontar fonte canônica aqui.
2. Linkar pro source-NN correspondente em `13-research/beyond-corp/`.
3. Citar trecho literal quando relevante (evitar paráfrase desencarnada).
4. Se não há fonte, marcar **⚠️ PENDÊNCIA** + abrir item pra research-loop.

---

## ⚠️ PENDÊNCIAS (decisões de research ainda em aberto)

- **ADR IdP**: Cognito vs Keycloak vs Auth0 vs Entra ID vs Zitadel (continua). Base: Fontes 07-12.
- **ADR PDP**: OPA vs Cedar. Base: Fontes 02 (NIST) + 08 (AWS).
- **ADR Service Mesh**: em M2? Qual (Linkerd vs Istio)? Base: Fonte 05 (BeyondProd).
- **ADR Edge/WAF**: Cloudflare vs AWS WAF. Base: Fontes 08, 10.
- **ADR Audit Storage**: S3 Object Lock vs TimescaleDB hash-chain. Base: LGPD + Fonte 02 tenet 7.
- **ADR Cloud**: AWS (Railway → ECS) vs GCP vs multi-cloud. Base: Fontes 07, 08.
- **ADR Admin Access**: Cloudflare Access vs Tailscale vs AWS Verified Access. Base: Fontes 07, 08, 10, 11.

Registrar em [[../STATE]] quando decididas.

---

## Links

- [[_moc]]
- [[BEYOND_CORP]]
- [[threat-model]]
- [[owasp-top10]]
- [[cve-process]]
- [[lgpd]]
- [[pentest-checklist]]
- [[../13-research/beyond-corp/_destilado]]
- [[../13-research/beyond-corp/_moc|research MOC]]
- [[../STATE]] — decisões ADR-###
