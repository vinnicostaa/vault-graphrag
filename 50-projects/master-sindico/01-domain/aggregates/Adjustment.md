---
title: Adjustment — aggregate commercial (adendo formal)
type: aggregate
tags:
  - master-sindico
  - domain
  - aggregate
  - cross-domain
  - validatable
  - imutabilidade
  - adendo
project: master-sindico
bounded_context: cross-domain
implements: IValidatable
status: confirmed-2026-04-25-D-131
created: 2026-04-24T00:00:00.000Z
updated: '2026-04-25'
---

# Adjustment — aggregate cross-domain (adendo formal)

**Confirmed (2026-04-25 D-131)** — criado em 2026-04-24 junto com [[../../STATE#D-105]]; **movido de `commercial` → `cross-domain` BC** em 2026-04-25 via [[../../STATE#D-131]] após dual-check formal.

> ⚠️ **Atualização 2026-04-25**: Adjustment é cross-BC. Pode aplicar-se a items de timeline (institutional), closed deals (commercial), master plan actions (institutional), atas de assembleia (assembly), comunicados (institutional), agenda items (assembly). BC owner: `cross-domain`.

## Visão geral

`Adjustment` é o **único mecanismo** de correção a um item já publicado na timeline imutável ([[../../02-architecture/adr/0033-ata-timeline-db-immutability]]). Quando síndico ou empresa precisam corrigir/complementar uma atividade já publicada, emitem um `Adjustment` que referencia o item original e **adiciona** informação — **nunca edita** o original.

## Campos principais

- `target_item_id` + `target_item_type` — referência ao item corrigido
- `adjustment_type` — enum (correção/complemento/retratação/esclarecimento)
- `parent_adjustment_id` — opcional; referência ao adjustment-pai em cadeia (INV-ADJ-006)
- `chain_depth` — int derivado; profundidade na cadeia (0 = raiz; máx 5)
- `lgpd_redacted` — bool; true se este adjustment é retratação por direito-esquecimento

## Enum `target_item_type` (D-131)

```go
type AdjustmentTargetType string
const (
    TargetTimelineEntry      AdjustmentTargetType = "timeline_entry"       // institutional
    TargetClosedDeal         AdjustmentTargetType = "closed_deal"          // commercial
    TargetMasterPlanAction   AdjustmentTargetType = "master_plan_action"   // institutional
    TargetAtaMinutes         AdjustmentTargetType = "ata_minutes"          // assembly
    TargetAnnouncement       AdjustmentTargetType = "announcement"         // institutional
    TargetAgendaItem         AdjustmentTargetType = "agenda_item"          // assembly
)
```
- `description` — texto do adendo
- `evidence[]` — mídia justificativa
- `requested_by_user_id` — quem solicita (pode ser síndico ou empresa)
- `status` — IValidatable
- `validated_by_user_id`, `validated_at`, `validation_reason`
- `legal_basis?` — base jurídica quando aplicável (Compliance C6)

## Implementa IValidatable

Adjustment precisa validação cruzada: se empresa emite, síndico valida; se síndico emite, empresa é notificada mas pode contestar em 7 dias (backlog M2). Aprovação publica `AdjustmentPublished` na timeline como **novo item apontando para o original**, mantendo integridade imutabilidade.

## Invariantes críticos

- **INV-ADJ-001**: `target_item_id` **deve** existir e estar em estado `approved`/`published` (não se ajusta item rejeitado)
- **INV-ADJ-002**: adjustment **nunca** altera campos do item original (append-only)
- **INV-ADJ-003**: na timeline, adjustment aparece **em nova linha** com link visual "adendo ao item X"
- **INV-ADJ-004**: cadeia de adjustments — adjustment pode ser alvo de outro adjustment (raro mas suportado)
- **INV-ADJ-005**: Compliance C6 depende de `adjustments pendentes > 30 dias == 0` para somar 15 pts (ver INV-GOV-002)
- **INV-ADJ-006** (D-131 2026-04-25): profundidade máxima da cadeia adendo-de-adendo = **5**. Após 5, novo adendo deve referenciar o adendo-raiz (não recursivo infinito). DB CHECK constraint `chain_depth <= 5`.

## LGPD Interaction (D-131 2026-04-25)

Adendo de retratação por direito-esquecimento (LGPD Art. 18 V):
- `adjustment_type = retratação`
- `lgpd_redacted = true` no adjustment
- `target_item.lgpd_redacted = true` propagado (UI mascara conteúdo do target original com placeholder "Conteúdo retratado por solicitação LGPD")
- `audit_entries` mantido 5 anos imutável (ADR-0028 HMAC keyed por adjustment_id)
- Operação requer step-up MFA do síndico ([[../../02-architecture/adr/0026-admin-mfa-m1]])

## Gaps

- Fluxo de **contestação** do síndico quando empresa solicita adjustment ambíguo — definir
- Quando ajuste é **legal** (ex: determinação judicial), bypass de aprovação — estudar casos
- Relação com C6 (tela de adendos) — cross-link explícito

## Links

- [[_moc]]
- [[../patterns/validatable-interface]]
- [[../../02-architecture/adr/0033-ata-timeline-db-immutability]]
- [[TimelineEntry]]
- [[../../03-subprojects/web/ui-catalog/compliance/C6]] — tela síndico (imutabilidade e adendos)
- [[../../03-subprojects/web/ui-catalog/sindico/S21]]
- [[../../STATE#D-105]]
