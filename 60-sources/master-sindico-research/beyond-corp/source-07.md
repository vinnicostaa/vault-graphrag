---
title: "Google Cloud Identity-Aware Proxy (IAP)"
type: source
tags: [research, zero-trust, google-cloud, iap, context-aware-access, access-proxy]
source: https://cloud.google.com/security/products/iap
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 07 — Google Cloud Identity-Aware Proxy (IAP)

## Metadata

- **Instituição**: Google Cloud
- **URL principal**: https://cloud.google.com/security/products/iap
- **URL docs**: https://cloud.google.com/iap/docs (redirecionada pra https://docs.cloud.google.com/iap/docs)
- **URL conceitos**: https://docs.cloud.google.com/iap/docs/concepts-overview
- **URL context-aware access**: https://docs.cloud.google.com/iap/docs/cloud-iap-context-aware-access-howto
- **Data de acesso**: 2026-04-23
- **Tipo**: documentação de produto / product reference

## Resumo

IAP é a materialização comercial do modelo BeyondCorp na Google Cloud. É uma **Access Proxy gerenciada** que faz authN + authZ antes de a requisição chegar ao backend (App Engine, Cloud Run, GCE, GKE, on-prem via conector).

Fluxo:
1. Requisição chega ao Load Balancer HTTPS.
2. IAP intercepta; se não autenticado, redireciona pra Google Sign-In / Workforce Identity Federation / Identity Platform.
3. Após authN, IAP consulta IAM policy (role `IAP-secured Web App User`) + **Access Levels** (regras context-aware).
4. Se passa, IAP assina JWT e encaminha ao backend; backend valida assinatura (cabeçalho `X-Goog-IAP-JWT-Assertion`).

**Context-Aware Access** permite políticas que consideram:
- IP de origem (ex.: "bloquear fora do Brasil").
- Atributos do device (postura, certificado, OS version) via **Endpoint Verification**.
- Horário.
- Custom attributes via Access Context Manager.

Protege 3 tipos de recurso:
- **Google Cloud console e APIs** (primeira linha de defesa do admin access).
- **VMs** (SSH/RDP sem VPN).
- **Web apps** (HTTPS).

## Trechos-chave

> "IAP lets you establish a central authorization layer for applications accessed by HTTPS, so you can use an application-level access control model instead of relying on network-level firewalls."
> — docs.cloud.google.com/iap/docs/concepts-overview

> "By adding an IAM conditional binding to the IAM policy, access to your resources is further restricted based on request attributes."
> — docs.cloud.google.com/iap/docs

> "IAP enables you to implement robust context-aware controls to restrict access to only designated administrators."
> — docs.cloud.google.com/iap/docs/cloud-iap-context-aware-access-howto

> "To limit access based on IP address or end-user device attributes, you can create an access level."
> — docs.cloud.google.com/iap/docs

> "IAP operates based on a context-aware access model, incorporating factors such as user identity, device status, and location to make access decisions."
> — Google Cloud docs (síntese)

## Aplicabilidade ao Master Síndico

**Parcial — conceitualmente sim, produto específico não se master-sindico não for GCP-first.**

Justificativa:

- **Se master-sindico rodar em GCP**: IAP é caminho rápido pra BeyondCorp-as-a-service, especialmente pra **portal admin interno** (painel de operação da plataforma). Zero código de auth, zero VPN pra devs. Recomendado pra **console admin do SaaS** (equipe interna).
- **Se master-sindico rodar em AWS** (provável): equivalente seria **AWS Verified Access** (feature-similar, menos maduro) ou **Cloudflare Access** (cloud-agnóstico, melhor UX) — ver `source-10.md`.
- **Para end-users do SaaS** (síndicos, moradores): IAP não encaixa diretamente; end-users não têm contas Google corporativas. Mas os **padrões** (JWT assinado por proxy, context-aware rules, IAM conditional) são reproduzíveis.

**Lições aplicáveis independentes de produto**:

1. **Separar authN (proxy/gateway) de authZ fine-grained (backend)** — IAP faz authN+authZ coarse; backend checa IAM condicional.
2. **JWT assinado com assinatura verificável** entre proxy e backend (IAP usa ES256 com JWK rotativo; master-sindico deve usar o mesmo padrão).
3. **Access Levels = ABAC**: "IP BR + device verificado + horário comercial + user role síndico".
4. **Separar acesso admin (equipe interna) de acesso produto (usuários finais)**: use IAP / Cloudflare Access / Tailscale pra admin; gateway customizado pra produto.

⚠️ PENDÊNCIA: decisão de cloud (AWS vs GCP) impacta se adotamos IAP diretamente ou replicamos o padrão.

## Tags/tópicos

- iap
- google-cloud
- context-aware-access
- access-proxy
- zero-trust
- managed-proxy
