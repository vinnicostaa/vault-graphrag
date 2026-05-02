---
title: Source 05 — Google Cloud Identity-Aware Proxy IAP
type: source
tags: [research, google, iap, zero-trust, beyondcorp, auth, identity]
source: https://cloud.google.com/security/products/iap
created: 2026-04-23
updated: 2026-04-23
aliases: [gcp-iap]
---

# Source 05 — Google Cloud Identity-Aware Proxy (IAP)

- **URL primária**: https://cloud.google.com/security/products/iap
- **Docs**: https://docs.cloud.google.com/iap/docs/concepts-overview
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

- "Identity-Aware Proxy implements zero-trust access for Google Cloud resources".
- "Alternativa cloud-native ao VPN tradicional; gerencia acesso a apps em Cloud Run, App Engine, Compute Engine e GKE".
- "IAP verifies identity and enforces authorization at the application level, eliminating broad network access and perimeter-based security, with every request evaluated in real time".
- Policies context-aware baseadas em: **user identity, group membership, device security, contextual signals (location/IP)**.
- "End users point their web browser to an internet-accessible URL... no VPN client required".
- Integra com Access Context Manager pra device policy + access level.

## Aplicabilidade ao Master Síndico

- **Direta (literal IAP)**: NÃO. Seria vendor-lock GCP; além disso Master Síndico já opera no padrão BeyondCorp via CLAUDE.md §3.10 (zero-trust, auth+authz em toda request).
- **Direta (conceitual)**: ALTA. Padrão que já seguimos — IAP é concretização canônica do BeyondCorp.
- Patterns a reforçar:
  - **M1**: ABAC matrix `(role, tenant_id, resource_type, action)` como funções Go testáveis, chamadas antes de todo handler Gin.
  - **M2**: Device posture — MFA obrigatório em novo device, rate limit por device ID, binding session ↔ device.
  - **M2+**: Context-aware (geo + horário comercial).
- **Não-replicar**: IAP como proxy físico externo. Nossa versão = middleware `authz.Enforce` in-app.

## Links relacionados

- [[../beyond-corp/_moc]] — pesquisa BeyondCorp dedicada.
- [[source-12]] Identity Platform (backend de auth que IAP usa; comparamos com Zitadel).
- Destilado: [[_destilado#3.2 IAP]].
