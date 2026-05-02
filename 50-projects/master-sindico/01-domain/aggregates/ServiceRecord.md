---
title: ServiceRecord — aggregate commercial
type: aggregate
tags: [master-sindico, domain, aggregate, commercial, validatable]
project: master-sindico
bounded_context: commercial
implements: IValidatable
status: stub
created: 2026-04-24
---

# ServiceRecord — aggregate commercial

**Stub** — criado em 2026-04-24 junto com [[../../STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)|D-105]]. Aggregate registrado no UL mas sem arquivo MD completo; este documento é o pontapé mínimo. Expandir em Sprint 10 com invariantes detalhadas.

## Visão geral

`ServiceRecord` representa o **registro de execução de serviço** que a empresa submete após fechar um negócio (ClosedDeal) via Connect Me. Contém descrição da execução, datas, fotos/vídeos, classificação de risco e natureza técnica. Entra no **hub de validações do síndico** ([[../patterns/validatable-interface]]) aguardando aprovação.

## Identidade

- **ID**: UUIDv7 (`service_record_id`)
- **Tenant**: `condominium_id`
- **Relacionamentos**: `closed_deal_id` (origem), `company_id`, `submitted_by_user_id`

## Campos principais

- `title` — resumo do serviço
- `description` — detalhes de execução
- `risk_level` — enum (baixo/médio/alto/crítico)
- `activity_nature` — enum (preventiva/corretiva/emergencial/melhoria)
- `executed_at` — timestamp de execução
- `media[]` — fotos, vídeos, PDFs (via IStorageProvider)
- `status` — IValidatable ValidationStatus (pending/approved/rejected/adjustment_requested)
- `submitted_at`, `validated_at`, `validated_by_user_id`, `validation_reason`

## Implementa IValidatable

Ver [[../patterns/validatable-interface]] para contrato completo. Invariantes específicos:

- **Aprovação gera `TimelineEntry` imutável** via `ITimelinePublisher` (evento `ServiceRecordApproved`)
- **Ajuste solicitado** permite empresa submeter nova versão (referenciando item original via `original_service_record_id`) — não edita o original
- **Rejeição** encerra o registro; empresa pode submeter novo para o mesmo ClosedDeal

## Eventos publicados

- `ServiceRecordSubmitted(id, condominium_id, company_id, submitted_by, submitted_at)`
- `ServiceRecordApproved(id, validated_by, validated_at)` → trigger publicação timeline
- `ServiceRecordRejected(id, validated_by, validated_at, reason)`
- `ServiceRecordAdjustmentRequested(id, validated_by, validated_at, reason)`

## Invariantes (a expandir)

- INV-SR-001: append-only (versões novas em caso de ajuste; nunca UPDATE do original)
- INV-SR-002: só síndico do condomínio pode validar (+ admin MS)
- INV-SR-003: aprovação é irreversível (após `approved`, não volta a `pending`)

## Gaps a expandir

- Campos detalhados (ex: assinatura digital técnica responsável)
- Regras de expiração (quanto tempo aguarda validação antes de auto-approve?)
- Relação com `compliance_score` (C10) — aprovação tempestiva soma pontos

## Links

- [[_moc]]
- [[../patterns/validatable-interface]]
- [[ClosedDeal]]
- [[Company]]
- [[TimelineEntry]]
- [[../../03-subprojects/web/ui-catalog/empresa/E9]] — tela empresa submete
- [[../../03-subprojects/web/ui-catalog/sindico/S21]] — tela síndico valida
- [[../../03-subprojects/web/ui-catalog/sindico/S22]] — detalhe validação
- [[../../STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)|D-105]]
