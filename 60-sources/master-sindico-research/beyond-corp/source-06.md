---
title: "CISA Zero Trust Maturity Model v2.0 (April 2023)"
type: source
tags: [research, zero-trust, cisa, maturity-model, canonical, compliance]
source: https://www.cisa.gov/zero-trust-maturity-model
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 06 — CISA Zero Trust Maturity Model v2.0

## Metadata

- **Instituição**: Cybersecurity and Infrastructure Security Agency (CISA), U.S. Department of Homeland Security
- **Publicação**: Versão 2.0, Abril 2023 (versão 1.0 publicada em 2021 em resposta ao US Executive Order 14028)
- **URL oficial (landing)**: https://www.cisa.gov/zero-trust-maturity-model
- **URL PDF**: https://www.cisa.gov/sites/default/files/2023-04/zero_trust_maturity_model_v2_508.pdf
- **Release note**: https://www.cisa.gov/news-events/news/cisa-releases-updated-zero-trust-maturity-model
- **Data de acesso**: 2026-04-23
- **Tipo**: framework / maturity model governamental

## Resumo

Modelo de maturidade Zero Trust criado pela CISA (agência de cibersegurança do governo americano) como guia prático pra agências federais e, na prática, adotado globalmente como checklist de implementação de ZT. A v2 (2023) ampliou a v1 adicionando estágio "Initial" entre Traditional e Advanced.

**Estrutura**:

### 5 pilares fundamentais

1. **Identity** — verificação e autenticação contínua de usuários e dispositivos associados.
2. **Devices** — garantir que dispositivos são seguros e gerenciados, independente de propriedade (corp ou BYOD).
3. **Networks** — gerenciar tráfego interno e externo, sem depender de perímetros.
4. **Applications & Workloads** — controles granulares e políticas de proteção pra aplicações on-prem e cloud.
5. **Data** — monitoramento contínuo e cifra de dados, em qualquer estado (at-rest, in-transit, in-use).

### 3 capacidades transversais (cross-cutting)

- **Visibility & Analytics** — logs, telemetria, SIEM.
- **Automation & Orchestration** — resposta automatizada, SOAR.
- **Governance** — políticas, auditoria, compliance.

### 4 estágios de maturidade

1. **Traditional** — perímetro clássico, auth estática, poucos sinais de contexto.
2. **Initial** — primeiras integrações (MFA parcial, alguma automação).
3. **Advanced** — ABAC integrado, automação ampla, telemetria correlacionada.
4. **Optimal** — políticas 100% dinâmicas, JIT access, resposta automatizada em tempo real.

## Trechos-chave

> "The Zero Trust Maturity Model (ZTMM) represents a gradient of implementation across five distinct pillars (...). Each pillar includes general details regarding the following cross-cutting capabilities: Visibility and Analytics, Automation and Orchestration, and Governance."
> — CISA ZTMM v2.0, 2023

> "Zero Trust provides a collection of concepts and ideas designed to minimize uncertainty in enforcing accurate, least privilege per-request access decisions in information systems and services in the face of a network viewed as compromised."
> — CISA ZTMM v2.0 (citando NIST)

> Pilares definidos como: **Identity | Devices | Networks | Applications & Workloads | Data**
> — CISA ZTMM v2.0, estrutura canônica

> Estágios: **Traditional → Initial → Advanced → Optimal**
> — CISA ZTMM v2.0

## Aplicabilidade ao Master Síndico

**Sim — o ZTMM é o melhor instrumento de auto-avaliação e planejamento de roadmap Zero Trust do master-sindico.**

Justificativa: os 5 pilares + 4 estágios permitem construir uma **matriz 5x4** onde cada célula descreve onde master-sindico está hoje e onde precisa chegar em M1, M2, M3.

Exemplo de aplicação (rascunho — refinar em STEERING):

| Pilar | Hoje (herdado) | M1 | M2 | M3 (Optimal) |
|---|---|---|---|---|
| Identity | Initial (OAuth, MFA opcional) | Advanced (OAuth 2.1+PKCE, MFA obrigatório, device-bound tokens) | Advanced+ (passkeys, step-up) | Optimal (continuous risk scoring) |
| Devices | Traditional (implícito) | Initial (fingerprint + inventory) | Advanced (Play Integrity, DeviceCheck) | Optimal (atestação contínua) |
| Networks | Initial (TLS edge) | Advanced (mTLS crítico, segmentação VPC) | Advanced+ (service mesh) | Optimal (micro-segmentação dinâmica) |
| Apps/Workloads | Traditional | Initial (WAF, API Gateway, PDP) | Advanced (OPA/Cedar, Binary Auth) | Optimal (sandbox, policy-as-code universal) |
| Data | Initial (TLS, criptografia at-rest parcial) | Advanced (criptografia de campo sensível, LGPD) | Advanced+ (DLP, audit WORM) | Optimal (classificação automática) |

**Uso prático**: montar este quadro em `08-security/ztmm-assessment.md` e revalidar a cada release.

**Por que isso importa pra master-sindico especificamente**: clientes corporate (administradoras com 100+ prédios) vão **pedir evidência de maturidade de segurança** em due diligence. ZTMM é um vocabulário compartilhado com auditores e compradores técnicos.

## Tags/tópicos

- cisa
- zero-trust
- maturity-model
- compliance
- assessment
- pillars
- government-framework
