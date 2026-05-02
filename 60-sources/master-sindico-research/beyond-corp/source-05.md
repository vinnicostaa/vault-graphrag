---
title: "BeyondProd — Google whitepaper on cloud-native security (2019)"
type: source
tags: [research, beyond-prod, zero-trust, google, microservices, canonical, mtls]
source: https://cloud.google.com/docs/security/beyondprod
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 05 — BeyondProd: A new approach to cloud-native security

## Metadata

- **Autores**: Maya Kaczorowski + time de segurança do Google Cloud
- **Instituição**: Google Cloud
- **Publicação**: Whitepaper oficial, Dezembro 2019
- **URL oficial (docs)**: https://cloud.google.com/docs/security/beyondprod (também em https://docs.cloud.google.com/docs/security/beyondprod)
- **URL blog de anúncio**: https://cloud.google.com/blog/products/identity-security/beyondprod-whitepaper-discusses-cloud-native-security-at-google
- **URL mirror PDF**: https://price2meet.com/gcp/docs/security_beyondprod.pdf
- **Data de acesso**: 2026-04-23
- **Tipo**: whitepaper oficial

## Resumo

Enquanto BeyondCorp trata de **usuários acessando recursos corporativos** (north-south), BeyondProd trata de **microserviços acessando microserviços em produção** (east-west, server-to-server).

Princípios centrais:

1. **Protection of the network at the edge** — workloads isoladas de ataques de rede por default.
2. **No inherent mutual trust between services** — serviços não confiam uns nos outros pelo fato de estarem na mesma rede.
3. **Trusted machines running code of known provenance** — só máquinas auditadas rodam código cuja origem é verificável.
4. **Choke points for consistent policy enforcement across services** — políticas consistentes via mTLS universal, service mesh, ingress controllers.
5. **Simple, automated, and standardized change rollout** — mudanças via pipelines automatizados (Borg + BAB).
6. **Isolation between workloads sharing an operating system** — sandboxing com gVisor.

Implementação Google:
- **ALTS** (Application Layer Transport Security) = mTLS universal entre serviços (Google's answer to SPIFFE).
- **BAB** (Binary Authorization for Borg) = só deploys assinados + revisados rodam em prod.
- **Borg** = orchestrator (progenitor do Kubernetes).
- **gVisor** = sandbox userspace kernel pra isolar workloads.
- **Google Security Ticket**: token ephemeral carregando identidade do usuário end-user entre serviços (propaga auth context).

Mapeamento pra ecossistema open:
- ALTS ≈ **SPIFFE/SPIRE + mTLS via Istio/Linkerd**.
- BAB ≈ **Sigstore + cosign + Kyverno / OPA Gatekeeper**.
- Borg ≈ **Kubernetes**.
- gVisor ≈ **gVisor open source** ou **Kata Containers** ou **Firecracker**.

## Trechos-chave

> "Service trust should depend on characteristics like code provenance, trusted hardware, and service identity, rather than the location in the production network."
> — BeyondProd whitepaper, Google Cloud

> "Only authenticated, trusted, and specifically authorized callers or services can access any other service."
> — BeyondProd whitepaper

> "Each machine has an ALTS credential that is provisioned using the host integrity system (...). Identities are bound to services instead of to a specific server name or host."
> — BeyondProd whitepaper (trecho citado em docs.cloud.google.com/docs/security/beyondprod)

> "ALTS is a mutual authentication and transport encryption system for services in Google infrastructure."
> — BeyondProd whitepaper

> "BAB is a deploy-time enforcement check that helps ensure that code meets internal security requirements before the code is deployed."
> — BeyondProd whitepaper

> "gVisor uses a user space kernel to intercept and handle syscalls, reducing the interaction with the host and the potential attack surface."
> — BeyondProd whitepaper

## Aplicabilidade ao Master Síndico

**Sim, canônico para a camada east-west.**

Justificativa: master-sindico com 7 BCs vai ter comunicação interna significativa (ex.: BC-assembly chama BC-identity pra validar elegibilidade; BC-billing chama BC-commercial pra processar pagamento de serviço). Essa comunicação leste-oeste é **o ponto cego mais comum** em SaaS brasileiros — TLS simples ou HTTP interno com "a VPC é confiável".

Princípios aplicáveis em ordem de prioridade:

1. **M1 — identidade de serviço explícita**: cada BC tem um `service_id` + credencial mTLS (mesmo que rotação manual no M1).
2. **M1 — mTLS nos 3 caminhos críticos**: `BC-identity ↔ BC-billing`, `BC-identity ↔ BC-assembly`, `API Gateway ↔ todos os BCs`. Sem mesh ainda; TLS mútuo manual com certs curtos.
3. **M2 — service mesh**: Linkerd (mais leve que Istio) habilita mTLS automático em todos os caminhos + observabilidade + retry/circuit break.
4. **M2 — Binary Authorization**: pipeline CI assina imagens com cosign; cluster só roda imagens assinadas (Kyverno ou OPA Gatekeeper como admission controller).
5. **M3 — SPIFFE/SPIRE** pra identidade de workload uniforme (se multi-cloud ou workloads híbridos on-prem).
6. **Futuro — sandboxing** (gVisor/Kata) para workloads que processam input não-confiável (ex.: parse de PDFs de atas, thumbnails de vídeos).

## Tags/tópicos

- beyond-prod
- microservices-security
- mtls
- alts
- binary-authorization
- service-identity
- spiffe
- east-west-traffic
