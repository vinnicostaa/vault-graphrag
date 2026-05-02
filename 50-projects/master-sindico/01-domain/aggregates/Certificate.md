---
title: Aggregate — Certificate
type: spec
tags: [domain, ddd, aggregate, content, lms, master-sindico, v2]
bc: content
source: 90-ingested/.../specs/requirements/content.md Req 98.1
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Certificate`

**BC**: content · **Raiz**: ✅

Certificado emitido automaticamente ao completar 100% de curso. Serial verificável via URL pública. Assinatura digital BR plena em Sprint 3+ (MP 2.200-2).

## Entidade raiz

```go
type Certificate struct {
    id              CertificateID
    serialNumber    CertificateSerial       // UNIQUE; URL pública `/certificates/verify/{serial}`
    userID          UserID
    courseID        CourseID
    enrollmentID    EnrollmentID
    pdfURL          string                  // MinIO / R2
    issuedAt        time.Time
    digitalSignedAt *time.Time              // Sprint 3+
    validUntil      *time.Time              // opcional; default: indefinida (evergreen)
}
```

## Value Objects

- `CertificateID`, `CertificateSerial` (VO com algoritmo de serialização + checksum)

## Invariantes

- **INV-037, INV-068**: emitido automaticamente ao `Enrollment.progressPercent = 100%` (idempotente)
- `serialNumber` UNIQUE; verificável via URL pública sem auth
- Imutável pós-issued (sem edição)

## Factories

```go
func NewCertificate(userID UserID, courseID CourseID, enrollmentID EnrollmentID, serialGen SerialGenerator, pdfURL string) (*Certificate, error) {
    // issuedAt = NOW()
}

func ReconstructCertificate(...) *Certificate
```

## Métodos de domínio

- `Sign(at time.Time)` — seta `digitalSignedAt` (Sprint 3+)
- `Verify() bool` — valida serial + (futuro) assinatura

## Repository interface

```go
type ICertificateRepository interface {
    Save(ctx context.Context, c *Certificate) error
    FindByID(ctx context.Context, id CertificateID) (*Certificate, error)
    FindBySerial(ctx context.Context, s CertificateSerial) (*Certificate, error)       // verificação pública
    ListByUser(ctx context.Context, uid UserID) ([]*Certificate, error)
}
```

## Eventos emitidos

- `CertificateGenerated` (E-061)

## Links

- [[../bounded-contexts#5-content]]
- [[../invariants#content-inv-056-a-inv-070]]
- [[Enrollment]] · [[Course]]
