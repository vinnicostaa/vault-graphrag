---
title: Aggregate — Membership
type: spec
tags:
  - domain
  - ddd
  - aggregate
  - institutional
  - master-sindico
  - v2
bc: institutional
source: 90-ingested/.../specs/requirements/institutional.md Req 7-9
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
---

# Aggregate — `Membership`

**BC**: institutional · **Raiz**: ✅ (vínculo operacional ABAC)

Vínculo `User ↔ Condomínio ↔ (Unit) ↔ Role` com período. Chave primária de autorização ABAC.

## Entidade raiz

```go
type Membership struct {
    id              MembershipID
    userID          UserID
    condominiumID   CondominiumID
    unitID          *UnitID                 // nil pra síndico profissional sem unidade
    role            Role                    // sindico | morador | subsindico | conselho | assembly_auditor
    roleType        RoleType                // principal | titular | dependent | auxiliary | ...
    status          MembershipStatus        // active | suspended | ended
    startDate       time.Time
    endDate         *time.Time
    
    // Mandato (D-129 2026-04-25) — relevante quando role ∈ {sindico, subsindico}
    isActiveMandate         bool
    mandateStartedAt        *time.Time
    mandateEndedAt          *time.Time           // null = ativo
    mandateEndReason        *MandateEndReason    // enum
    mandateEndDocumentURL   *string              // URL ata homologada ou descrição
    transferredToUserID     *UserID              // pra rotação cross-síndico
    
    // Role efêmera (D-132 — auditor de assembly)
    expiresAt       *time.Time              // role temporária; null = permanente
    
    createdAt       time.Time
    updatedAt       time.Time
}

// D-129: enum de motivo de fim de mandato
type MandateEndReason string
const (
    TermEnded   MandateEndReason = "term_ended"   // mandato concluído
    Removed     MandateEndReason = "removed"      // destituído
    Resigned    MandateEndReason = "resigned"     // renunciou
    Transferred MandateEndReason = "transferred"  // rotação
    Other       MandateEndReason = "other"
)
```

## Mandato (D-129 — regularização 2026-04-25)

> [[../../STATE#D-129]] consolidou mandato de síndico/subsíndico como **flags temporais de Membership**, não aggregate separado. `Mandate` aggregate **não existe** no canônico — só projeção via query: `SELECT * FROM memberships WHERE role IN ('sindico','subsindico') AND condominium_id=X ORDER BY mandate_started_at DESC`.

Histórico: cada vínculo de síndico vira uma row de Membership com `mandate_started_at`/`mandate_ended_at`. Lista cronológica = histórico de mandatos.

Termo "Mandato" continua existindo no glossário e linguagem ubíqua (negócio); tecnicamente é Membership com role institucional + `is_active_mandate=true`.

## Role efêmera — Auditor (D-132)

> [[../../STATE#D-132]] introduziu `role=assembly_auditor` como Membership efêmera com `expires_at = assembly.ended_at`. Auto-revoga ao `Assembly.End()`. Permite User externo (não morador) ter Membership temporária para 1 assembleia (exceção INV-026 — ver invariantes abaixo).

## Value Objects

- `MembershipID`, `MembershipStatus` (enum)
- `Role` (compartilhado via `shared/types/`)

## Invariantes

- **INV-026** (revisado 2026-04-25 D-132): Membership ativa única por `(user_id, condominium_id) WHERE status='active'` (UNIQUE parcial). **Exceção**: `role='assembly_auditor'` permite múltiplas memberships ativas com a mesma `(user_id, condominium_id)` — auditor pode coexistir com role permanente.
- **INV-### (D-129)**: `mandate_ended_at` deve ser NULL OR `> mandate_started_at` (DB CHECK)
- **INV-### (D-129)**: `is_active_mandate=true` requer `mandate_started_at IS NOT NULL` E `mandate_ended_at IS NULL`
- **INV-### (D-129)**: `mandate_end_reason` é obrigatório quando `mandate_ended_at IS NOT NULL`
- **INV-### (D-132)**: `expires_at` requer `role='assembly_auditor'` (role efêmera só pra auditor por enquanto)
- Mandato de síndico/subsíndico é um Membership com `role ∈ {sindico, subsindico}` e flags de mandato preenchidas

## Factories

```go
func NewMembership(userID UserID, condoID CondominiumID, unitID *UnitID, role Role, roleType RoleType, startDate time.Time) (*Membership, error) {
    // valida: role coerente com unit (ex: morador precisa unit)
}

func ReconstructMembership(...) *Membership
```

## Métodos de domínio

- `Activate()` — `suspended → active`
- `Suspend(reason string)` — `active → suspended`
- `End(endDate time.Time, reason string)` — terminal; preserva todos os campos no histórico
- `TransferRole(newRole Role) error` — raro; preserve audit
- **D-129**: `EndMandate(endDate, reason MandateEndReason, documentURL *string)` — encerra mandato; emite `MembershipMandateEnded` (timeline imutável)
- **D-129**: `TransferMandate(toUserID UserID, documentURL *string)` — rotação síndico; cria nova Membership pro novo síndico + encerra a atual com `transferredToUserID`
- **D-132**: `ExpireAuditorRole()` — chamado por `Assembly.End()` em cascata para todas Memberships com `role=assembly_auditor` da assembleia

## Repository interface

```go
type IMembershipRepository interface {
    Save(ctx context.Context, m *Membership) error
    FindByID(ctx context.Context, id MembershipID) (*Membership, error)
    FindActiveByUserAndCondo(ctx context.Context, uid UserID, cid CondominiumID) (*Membership, error)
    ListActiveByCondominium(ctx context.Context, cid CondominiumID) ([]*Membership, error)
    ListActiveByUser(ctx context.Context, uid UserID) ([]*Membership, error)
    CountCondosBySyndic(ctx context.Context, syndicID UserID) (int, error)  // INV-027
}
```

## Eventos emitidos

- `MembershipCreated` (E-017)
- `MembershipEnded` (E-018)

## Links

- [[../bounded-contexts#3-institutional]]
- [[../invariants#institutional-inv-021-a-inv-035]]
- [[Condominium]] · [[Unit]] · [[../aggregates/User]]
