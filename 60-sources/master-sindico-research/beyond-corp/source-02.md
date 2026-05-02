---
title: "NIST SP 800-207 — Zero Trust Architecture (2020)"
type: source
tags: [research, zero-trust, nist, canonical, standard]
source: https://csrc.nist.gov/pubs/sp/800/207/final
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 02 — NIST SP 800-207 Zero Trust Architecture

## Metadata

- **Autores**: Scott Rose, Oliver Borchert, Stu Mitchell, Sean Connelly
- **Instituição**: National Institute of Standards and Technology (NIST), U.S. Department of Commerce
- **Publicação**: NIST Special Publication 800-207, publicação final em **11 de agosto de 2020**
- **URL oficial**: https://csrc.nist.gov/pubs/sp/800/207/final
- **URL PDF**: https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.800-207.pdf
- **URL release NIST**: https://www.nist.gov/news-events/news/2020/08/zero-trust-architecture-nist-publishes-sp-800-207
- **Data de acesso**: 2026-04-23
- **Tipo**: standard / publicação governamental normativa

## Resumo

Publicação de referência internacional sobre Zero Trust. Define conceitualmente o modelo, os princípios operacionais e os componentes arquiteturais. É citada em praticamente toda documentação comercial posterior (AWS, Google, Cloudflare, Microsoft) como fonte canônica.

Define **Zero Trust (ZT)** como um conjunto evolutivo de paradigmas de cibersegurança que move defesas de perímetros de rede estáticos para foco em usuários, ativos e recursos. Uma **Zero Trust Architecture (ZTA)** usa princípios ZT pra planejar infraestrutura e workflows empresariais/industriais.

Premissa central: **não há confiança implícita concedida a ativos ou contas de usuário com base apenas em sua localização física ou de rede** (LAN vs internet) ou na propriedade do ativo (empresarial ou pessoal). Autenticação e autorização (tanto do sujeito quanto do dispositivo) são funções discretas executadas **antes** de cada sessão a um recurso empresarial.

O documento:
1. Define os **7 tenets** do Zero Trust (core).
2. Descreve os **componentes lógicos**: Policy Engine (PE), Policy Administrator (PA), Policy Enforcement Point (PEP), e data sources (CDM, threat intel, SIEM, PKI, ID management, activity logs, data access policy, industry compliance).
3. Apresenta **3 abordagens de deployment**: enhanced identity governance, micro-segmentation, network infrastructure / SDP.
4. Lista casos de uso, ameaças à ZTA, considerações de migração.

## Trechos-chave

Os 7 tenets (seção 2.1):

> **Tenet 1**: "All data sources and computing services are considered resources."

> **Tenet 2**: "All communication is secured regardless of network location."

> **Tenet 3**: "Access to individual enterprise resources is granted on a per-session basis."

> **Tenet 4**: "Access to resources is determined by dynamic policy — including the observable state of client identity, application/service, and the requesting asset — and may include other behavioral and environmental attributes."

> **Tenet 5**: "The enterprise monitors and measures the integrity and security posture of all owned and associated assets."

> **Tenet 6**: "All resource authentication and authorization are dynamic and strictly enforced before access is allowed."

> **Tenet 7**: "The enterprise collects as much information as possible about the current state of assets, network infrastructure and communications and uses it to improve its security posture."

Componentes lógicos (seção 3.1):

> "The **Policy Engine (PE)** is responsible for the ultimate decision to grant access to a resource for a given subject. (...) The PE is paired with the policy administrator component."

> "The **Policy Administrator (PA)** is responsible for establishing and/or shutting down the communication path between a subject and a resource."

> "The **Policy Enforcement Point (PEP)** is responsible for enabling, monitoring, and eventually terminating connections between a subject and an enterprise resource."

## Aplicabilidade ao Master Síndico

**Sim, canônico e normativo.**

Justificativa: como SaaS multi-tenant brasileiro potencialmente sujeito a certificações (ISO 27001, SOC 2) e compliance LGPD, o master-sindico se beneficia de mapear sua arquitetura **explicitamente** aos 7 tenets do NIST pra facilitar:
- Auditoria externa (auditor pede "onde está seu PE/PA/PEP?").
- Due diligence de clientes corporate (administradoras grandes).
- Comunicação de roadmap de segurança.

Mapeamento proposto (detalhado em `_destilado.md` §2.2):
- **PE**: OPA ou Cedar (AWS Verified Permissions), com políticas ABAC versionadas em Git.
- **PA**: serviço de sessão (`BC-identity`) que emite/renova/revoga tokens.
- **PEP**: middleware em cada BC + API Gateway central + sidecar Envoy (quando houver mesh).
- **Data sources alimentando PE**: audit log (comportamento), device inventory, threat intel (rate limit / WAF), SIEM (Datadog/Grafana).

## Tags/tópicos

- zero-trust
- nist
- standard
- policy-engine
- policy-enforcement-point
- tenets
- governmental
