---
title: Tasks — BC commercial
type: spec
tags: [tasks, by-module, commercial, master-sindico, v2]
source: 04-requirements/functional/commercial.md + AUDIT A-021/A-029/A-030/A-031 + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Tasks — BC `commercial`

Tasks do bounded context **commercial** (Companies, Connect Me 4 fluxos, Reputação Bronze→Diamante, Vizinhança cupons, Banco de Talentos, Supplier Vote).

Alinhamento: [[../../04-requirements/functional/commercial]] (COM-001..COM-025).

---

## M1

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-005 | A-021 — Doc anti-pattern `SELECT *` em `07-quality/PATTERNS.md` | 10 | ⏳ |
| T-005a | A-021 — Substituir 12 `SELECT *` por colunas explícitas | 10 | ⏳ |

## M2 — Connect Me + Reputação + Vizinhança

### Connect Me

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-100 | UI síndico→empresa RFP + lista propostas + escolha | 11 | ⏳ |
| T-100a | UI empresa — receber RFP + enviar proposta estruturada | 11 | ⏳ |
| T-101 | UI morador→síndico form estruturado | 11 | ⏳ |
| T-102 | UI empresa→empresa Plus/Pro | 12 | ⏳ |
| T-103 | UI síndico→parceiro Vizinhança | 12 | ⏳ |
| T-104 | Fix A-029 — Company Evaluation TOCTOU (`FOR UPDATE` + PBT) | 11 | ⏳ |
| T-105 | Fix A-030 — Supplier Vote state machine race (voto pós-close) | 11 | ⏳ |
| T-110 | Denúncia assédio UI + admin action (COM-005) | 12 | ⏳ |
| T-111 | Auto-close 48h cron (COM-004) | 12 | ⏳ |

### Reputação

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-150 | Dashboard reputação pública empresa | 11 | ⏳ |
| T-151 | Recálculo determinístico pós-evento (contract closed, eval, video) | 11 | ⏳ |
| T-152 | PBT degradação + promoção tier | 11 | ⏳ |
| T-155 | ADR D-010 parametrização vs congelada (fechar) | 11 | ⏳ |
| T-160 | Dashboard reputação V2 — breakdown + trending | 13 | ⏳ |

### Vizinhança

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-500 | Mapa + cupons lock 4h | 12 | ⏳ |
| T-501 | Seal "Síndico-aprovado" | 12 | ⏳ |
| T-510 | Geo search (PostGIS ou lat/long + raio 2km) | 12 | ⏳ |

## M3 — Banco de Talentos

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-400 | Vídeo-currículo 90s + profile profissional | 14 | ⏳ |
| T-400a | Busca empresa skill + CEP + tier | 14 | ⏳ |

---

## Invariantes cobertos

I-021..I-030 de [[../../01-domain/bounded-contexts#commercial]]:
- I-021 Reputação = função determinística
- I-022 Connect Me unidirecional
- I-023 Proposta única por (ConnectMeRequest, empresa)
- I-024 Proposal imutável após `sent` (Amendment)
- I-025 ClosedDeal imutável após dupla assinatura
- I-026 SupplierVoteCast único por (session, voter)
- I-027 CompanyEvaluation única por (company, evaluator, condominium)
- I-028 Avaliação anônima ao avaliado
- I-029 Cupom Vizinhança lock 4h
- I-030 Quota ConnectMe decrementa no `publish`

---

## Links

- [[_moc]]
- [[../../04-requirements/functional/commercial]]
- [[../../01-domain/bounded-contexts#commercial]]
- [[../backlog]]
