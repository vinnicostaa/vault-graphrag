---
title: Aggregate — AttendanceRecord
type: spec
tags: [domain, ddd, aggregate, assembly, master-sindico, v2]
bc: assembly
source: 90-ingested/.../specs/requirements/assembly.md Req 56
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `AttendanceRecord`

**BC**: assembly · **Raiz**: ✅ (entidade com ciclo próprio; múltiplos check-ins/saídas)

Check-in de morador na assembleia. 3 modalidades: app (QR scan), terminal (kiosk), manual (síndico confirma). WebSocket broadcasta em < 200ms (BR-050).

## Entidade raiz

```go
type AttendanceRecord struct {
    id                  AttendanceID
    assemblyID          AssemblyID
    userID              UserID
    unitID              UnitID
    fracaoIdealAtCheckIn FracaoIdeal
    mode                CheckInMode             // app | terminal | manual
    checkedInAt         time.Time
    checkedOutAt        *time.Time              // opcional; ausência momentânea
    returnedAt          *time.Time              // se voltou pós saída
    confirmedBy         *UserID                 // se mode=manual
    ipAddress           string
    deviceFingerprint   *DeviceFingerprint      // se app
    createdAt           time.Time
}
```

## Value Objects

- `AttendanceID`, `CheckInMode` (enum pt-BR)
- `FracaoIdeal` (snapshot)

## Invariantes

- UNIQUE(assembly_id, user_id) — 1 record único por user por assembleia (adicionar/editar tempos, não duplicar)
- Ausência momentânea não apaga record (`checked_out_at` + eventual `returned_at`)
- Mode `manual` requer `confirmed_by != NULL`

## Factories

```go
func NewAttendanceRecord(assemblyID AssemblyID, userID UserID, unitID UnitID, fracao FracaoIdeal, mode CheckInMode, confirmedBy *UserID, ip string, fp *DeviceFingerprint) (*AttendanceRecord, error)
func ReconstructAttendanceRecord(...) *AttendanceRecord
```

## Métodos de domínio

- `CheckOut(now time.Time)` — seta `checkedOutAt`; timer ausência 15min inicia (BR-046)
- `Return(now time.Time)` — seta `returnedAt`

## Repository interface

```go
type IAttendanceRecordRepository interface {
    Save(ctx context.Context, r *AttendanceRecord) error
    FindByAssemblyAndUser(ctx context.Context, aid AssemblyID, uid UserID) (*AttendanceRecord, error)
    ListByAssembly(ctx context.Context, aid AssemblyID) ([]*AttendanceRecord, error)
    SumFracaoIdealPresentByAssembly(ctx context.Context, aid AssemblyID) (FracaoIdeal, error)     // quórum check
}
```

## Eventos emitidos

- `CheckInRegistered` (E-067) — broadcast WebSocket painel presidente

## Links

- [[../bounded-contexts#6-assembly]]
- [[../invariants#assembly-inv-071-a-inv-085]]
- [[../business-rules]] BR-042, BR-046, BR-050
- [[Assembly]] · [[Proxy]]
