---
title: "Microsoft Zero Trust — Overview & Principles"
type: source
tags: [research, zero-trust, microsoft, principles, framework]
source: https://learn.microsoft.com/en-us/security/zero-trust/zero-trust-overview
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 09 — Microsoft Zero Trust Overview

## Metadata

- **Instituição**: Microsoft Corporation
- **URL overview**: https://learn.microsoft.com/en-us/security/zero-trust/zero-trust-overview
- **URL business landing**: https://www.microsoft.com/en-us/security/business/zero-trust
- **URL Azure ZT**: https://learn.microsoft.com/en-us/azure/security/fundamentals/zero-trust
- **URL ZT architecture**: https://www.microsoft.com/en-us/security/business/security-101/what-is-zero-trust-architecture
- **URL develop-ZT**: https://learn.microsoft.com/en-us/security/zero-trust/develop/overview
- **Data de acesso**: 2026-04-23
- **Última atualização pela Microsoft**: 2025-02-27 (conforme metadado do doc)
- **Tipo**: documentação oficial / framework

## Resumo

Microsoft estrutura Zero Trust em **3 princípios** (didáticos, memoráveis) e **6 pilares** (identidades, endpoints, apps, dados, infraestrutura, redes).

**3 princípios**:

1. **Verify explicitly** — sempre autenticar e autorizar com base em todos os pontos de dados disponíveis (identidade do usuário, localização, saúde do device, serviço/workload, classificação do dado, anomalias).
2. **Use least privilege access** — limitar acesso com JIT (just-in-time) + JEA (just-enough-access), políticas adaptativas baseadas em risco, proteção de dados pra produtividade.
3. **Assume breach** — minimizar blast radius, segmentar acesso, verificar criptografia end-to-end, usar analytics pra detectar ameaças.

**Filosofia-chave**: "never trust, always verify" — Zero Trust assume que cada requisição vem de uma rede não-controlada, mesmo que venha de dentro do firewall corporativo.

Alinhamento com governos:
- US Executive Order 14028 (Cybersecurity), OMB memo 22-09 (Federal Zero Trust Strategy).
- Microsoft alinha Entra ID com requisitos de identidade do memo 22-09.

Microsoft Secure Future Initiative (SFI) — desde Nov 2023, multi-year commitment pra implementar ZT rígido na própria Microsoft.

## Trechos-chave

> "Zero Trust is a security strategy. It isn't a product or a service, but an approach in designing and implementing the following set of security principles."
> — Microsoft Learn, Zero Trust overview

> "Verify explicitly — Always authenticate and authorize based on all available data points."
> — Microsoft Learn

> "Use least privilege access — Limit user access with Just-In-Time and Just-Enough-Access (JIT/JEA), risk-based adaptive policies, and data protection."
> — Microsoft Learn

> "Assume breach — Minimize blast radius and segment access. Verify end-to-end encryption and use analytics to get visibility, drive threat detection, and improve defenses."
> — Microsoft Learn

> "Instead of believing everything behind the corporate firewall is safe, the Zero Trust model assumes breach and verifies each request as though it originated from an uncontrolled network."
> — Microsoft Learn

> "Regardless of where the request originates or what resource it accesses, the Zero Trust model teaches us to 'never trust, always verify.'"
> — Microsoft Learn

## Aplicabilidade ao Master Síndico

**Sim, médio — útil como framing didático e referência de arquitetura de identidade.**

Justificativa:

1. **Os 3 princípios são o melhor jingle pra comunicar Zero Trust aos stakeholders não-técnicos** (advogados do condomínio, síndicos, board). Mais fácil que explicar os 7 tenets do NIST numa reunião:
   - "Sempre verificamos quem está pedindo."
   - "Damos o mínimo necessário pra cada ação."
   - "Assumimos que já fomos invadidos e projetamos pra limitar o dano."

2. **Entra ID como referência de IdP enterprise** — se master-sindico tiver clientes corporate que já usam Microsoft 365, oferecer **federation via Entra ID (SAML/OIDC)** é diferencial comercial.

3. **Conceito de JIT/JEA** é aplicável: ações administrativas sensíveis (ex.: "impersonate síndico pra dar suporte") devem exigir elevação de privilégio por tempo curto + audit log + aprovação (dual-control).

4. **"Assume breach" no multi-tenant**: se um atacante comprometer credencial de um síndico, **projetar pra que ele não tenha acesso automático a outros condomínios**. Separação forte de `condo_id` em queries + RLS Postgres + policy ABAC.

⚠️ O overview da Microsoft é menos técnico que NIST e BeyondCorp; usar principalmente como referência pra **linguagem de comunicação** e como âncora caso master-sindico integre com Azure/Entra ID.

## Tags/tópicos

- microsoft
- zero-trust
- principles
- verify-explicitly
- least-privilege
- assume-breach
- communication-framing
