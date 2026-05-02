---
title: MOC — 13-research/beyond-corp
type: moc
tags: [moc, master-sindico, v2, research, beyond-corp, zero-trust]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/beyond-corp

Google BeyondCorp whitepapers + BeyondProd + NIST SP 800-207 + CISA ZTMM v2 + Google IAP + AWS Zero Trust + Microsoft + Cloudflare One + Tailscale + Okta Identity-Centric.

> Pasta da v2 (remontagem do Master Síndico). Paridade com scaffold canônico em `vault/40-templates/project-scaffold/`. Stub criado na Fase 0 (2026-04-23). Conteúdo (destilado + 12 fontes) populado em 2026-04-23 via pesquisa web em fontes primárias oficiais.

## Arquivos

- [[_destilado]] — síntese aplicável ao master-sindico (PT-BR); mapeamento de princípios Beyond-Corp/Zero-Trust por BC; gaps identificados no modelo atual; recomendações priorizadas M1/M2/M3.
- [[source-01]] — **BeyondCorp: A New Approach to Enterprise Security** (Ward & Beyer, Google, 2014). Paper inaugural; identity+device como perímetro.
- [[source-02]] — **NIST SP 800-207 Zero Trust Architecture** (Rose et al., 2020). Standard normativo; 7 tenets; PE/PA/PEP.
- [[source-03]] — **BeyondCorp: Design to Deployment at Google** (Osborn et al., 2016). Device Inventory + Trust Tiers + ACE.
- [[source-04]] — **BeyondCorp Part III: The Access Proxy** (Cittadini et al., 2016). Access Proxy coarse-grained; GFE; divisão coarse/fine.
- [[source-05]] — **BeyondProd** (Google Cloud, 2019). Zero-trust leste-oeste; mTLS/ALTS; Binary Authorization; sandbox.
- [[source-06]] — **CISA Zero Trust Maturity Model v2** (CISA, 2023). 5 pilares + 3 transversais + 4 estágios; vocabulário de auditoria.
- [[source-07]] — **Google Cloud Identity-Aware Proxy (IAP)**. Materialização comercial do BeyondCorp; context-aware access.
- [[source-08]] — **AWS Zero Trust — Prescriptive Guidance**. 3 princípios + catálogo de produtos (Verified Access, Verified Permissions/Cedar, VPC Lattice).
- [[source-09]] — **Microsoft Zero Trust Overview**. 3 princípios didáticos (verify explicitly, least privilege, assume breach).
- [[source-10]] — **Cloudflare One / Zero Trust**. SASE cloud-agnóstico; Access, Tunnel, Gateway; reference arch Zero Trust for SaaS.
- [[source-11]] — **Tailscale — Zero Trust Networking**. Mesh VPN WireGuard; ACLs identity-based; SSH CA; foco em admin access.
- [[source-12]] — **Okta Identity-centric Zero Trust**. Identity como perímetro moderno; Adaptive MFA; Device Assurance; 5 níveis maturidade identity-powered.

## Decisões pendentes (⚠️) registradas no destilado

- IdP (Cognito vs Keycloak vs Okta vs Auth0 vs Entra ID) — ADR a abrir.
- PDP (OPA vs Cedar) — ADR a abrir, recomendação provisória OPA.
- Service mesh em M1 — recomendação provisória: adiar pra M2; M1 com mTLS manual nos 3 caminhos críticos.
- Edge CDN/WAF (Cloudflare vs CloudFront+AWS WAF) — ADR a abrir.
- Audit storage (S3 Object Lock vs QLDB vs TimescaleDB+hash-chain) — ADR a abrir.

## Próximos passos de research (opcional)

- Papers BeyondCorp IV e V (Maintaining Productivity, The User Experience) — fonte não-crítica.
- NIST SP 800-207A (Zero Trust em ambientes multi-cloud / cloud-native) — publicação posterior de 2023, aplicável se master-sindico mirar multi-cloud.
- Google SRE Book (relacionado: defense in depth, blameless postmortem culture).

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
- [[../../08-security/_moc]] — aqui os princípios viram specs de security
- [[../../02-architecture/_moc]] — aqui viram ADRs
