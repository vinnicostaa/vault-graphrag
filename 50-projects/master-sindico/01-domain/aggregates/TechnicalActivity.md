---
title: TechnicalActivity — aggregate commercial
type: aggregate
tags: [master-sindico, domain, aggregate, commercial, validatable]
project: master-sindico
bounded_context: commercial
implements: IValidatable
status: stub
created: 2026-04-24
---

# TechnicalActivity — aggregate commercial

**Stub** — criado em 2026-04-24 junto com [[../../STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)|D-105]].

## Visão geral

`TechnicalActivity` representa uma **atividade técnica avulsa** registrada pela empresa (sem necessariamente estar ligada a um ClosedDeal). Inclui classificação de risco e natureza técnica. Diferente de `ServiceRecord` porque pode ser independente de Connect Me (ex: visita técnica preventiva agendada). Submete-se ao hub de validações ([[../patterns/validatable-interface]]).

## Campos principais

- `title`, `description`, `executed_at`
- `risk_level` (enum — igual ServiceRecord)
- `activity_nature` (enum — preventiva/corretiva/rotina)
- `company_id`, `condominium_id`, `submitted_by_user_id`
- `related_service_record_id?` — opcional, quando é continuação de um ServiceRecord
- `status` — IValidatable
- `media[]` via IStorageProvider

## Implementa IValidatable

Aprovação adiciona atividade ao **histórico técnico da empresa** (não gera `TimelineEntry` no condomínio, diferente de ServiceRecord). Útil para construir trilha de serviço preventivo da empresa no condomínio sem poluir a timeline pública.

## Diferença vs ServiceRecord

| Aspecto | ServiceRecord | TechnicalActivity |
|---|---|---|
| Origem | ClosedDeal (Connect Me fechado) | Independente OU continuação ServiceRecord |
| Publicação pós-aprovação | TimelineEntry pública | Histórico interno empresa |
| Visibilidade morador | Sim (quando timeline publica) | Não (interno) |
| Validação obrigatória | Sim | Sim |

## Gaps

Expandir em Sprint 10 com invariantes detalhadas.

## Links

- [[_moc]]
- [[../patterns/validatable-interface]]
- [[ServiceRecord]]
- [[Company]]
- [[../../03-subprojects/web/ui-catalog/empresa/E6]] — tela empresa registra atividade técnica
- [[../../03-subprojects/web/ui-catalog/sindico/S21]]
- [[../../STATE#D-105]]
