---
title: MOC — 04-requirements/functional
type: moc
tags: [moc, master-sindico, v2, requirements, functional]
created: 2026-04-23
updated: 2026-04-25
---

# MOC — 04-requirements/functional

Requirements funcionais por **bounded context** (7): identity, billing, institutional, commercial, content, assembly, cross-domain.

Format: EARS ("Quando/Enquanto `<trigger/state>`, o sistema deve `<behavior>`").  
Prioridade: M (M1 2026-05-18 pós-D-106), S (M2 2026-06-20), C (M3 2026-07-20), W (backlog).

## Arquivos (pós-Fase C 2026-04-25)

| BC | Arquivo | IDs canônicos | Count atual | Marco primário | Deltas Fase C |
|---|---|---|---|---|---|
| **Identity** | [[identity]] | IDN-001..IDN-039 | 39 | M1 | IDN-026 reescrito (D-135) |
| **Billing** | [[billing]] | BIL-001..BIL-055 | 55 | M1 | banner Stripe ratificado |
| **Institutional** | [[institutional]] | INS-001..INS-047 | 47 | M1-M2 | +4 deltas (R12, R36, R46, R47) |
| **Commercial** | [[commercial]] | COM-001..COM-050 | 50 | M2 | +5 deltas (R22, R23, R26, R27, R53, R-_chaos curriculum) |
| **Content** | [[content]] | CNT-001..CNT-038 | 38 | M2-M3 | +3 deltas (R29 12 tipos, R96 QR, R95 voice, R98.1 LMS enums) |
| **Assembly** | [[assembly]] | ASM-001..ASM-033 + ASM-ELE-AVAL | 34 | M3 | R48 enriquecido, R52 100% digital, R57 presencial assistido |
| **Cross-Domain** | [[cross-domain]] | XD-001..XD-038 | 38 | M1-M3 | +3 deltas (R94 Context Confirmer, D-131 Adjustment, R39 Compliance) |

**Total Fase C**: ~301 requirements funcionais (de ~260).

## Sub-domínios stubs (2026-04-27 — D-vault-025)

- [[governance]] — sub-domínio do BC `institutional` cobrindo Plano Diretor + Timeline + Eventos + Comunicados + Avaliação Gestão + Perguntas (16 REQ-GOV-* anchors)
- [[compliance]] — sub-domínio do BC `cross-domain` cobrindo LGPD + NR-1 + BeyondCorp + audit trail + dossier export gate

## Convenções

- **EARS**: "Quando X [acontece], [enquanto Y], o sistema deve Z."
- **Prioridade**: M (Must-M1), S (Should-M2), C (Could-M3+), W (Won't/backlog).
- **Critério de aceite**: ligável a teste automatizado ou manual checklist.
- **Dependências**: GRs globais ([[../global]]) ou outros requirements.
- **Fonte**: traceability ao legado (arquivo + Req# quando aplicável).
- **Pendências**: `⚠️ PENDÊNCIA` explícito quando não validado.

## Pendências abertas (para dual-check Fase 3)

- IDN-007 (MFA policy por persona), IDN-013 (estrutura X-Tenant-Id), IDN-028 (CAPTCHA vendor).
- BIL-001 (preços exatos), BIL-011 (lista Stripe events), BIL-033 (Mercado Pago vs Stripe-only).
- INS-017 (26 áreas master plan), INS-025 (13 tipos evento), INS-040 (estrutura certificações).
- COM-005 (Receita adapter), COM-025 (thresholds Bronze→Diamante / D-010), COM-017/035 (assinatura digital, threshold seal).
- CNT-003/005 (moderação SLA, OpenSearch migration), CNT-022 (Mux Live vs CF Stream).
- ASM-020 (Livekit self-host vs Cloud), ASM-024/028 (template ata, fórmula fala equilibrada), ASM-032 (assíncrona vs live).
- XD-001/015/017/020/033 (providers e migração infra).

## Vizinhos

- [[../_moc]] — MOC da pasta 04-requirements
- [[../global]] — GRs transversais
- [[../non-functional]] — NFRs
- [[../compliance-lgpd]] — LGPD
- [[../matrix-functional]] — matriz feature × persona × plano
- [[../registration-data]] — dados cadastro
- [[../../01-domain/bounded-contexts]] — modelagem BC (a destilar)
