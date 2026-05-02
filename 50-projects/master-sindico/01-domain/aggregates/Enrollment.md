---
title: Aggregate — Enrollment
type: spec
tags: [domain, ddd, aggregate, content, lms, master-sindico, v2]
bc: content
source: 90-ingested/.../specs/requirements/content.md Req 98.1
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Enrollment`

**BC**: content · **Raiz**: ✅

Matrícula `User ↔ Course`. Rastreia progresso (%). Ao atingir 100% → dispara `Certificate`.

## Entidade raiz

```go
type Enrollment struct {
    id                  EnrollmentID
    userID              UserID
    courseID            CourseID
    progressPercent     int                     // 0-100
    completedLessonIDs  []LessonID
    quizScores          map[ModuleID]int        // score 0-100
    startedAt           time.Time
    completedAt         *time.Time
    lastAccessAt        time.Time
}
```

## Value Objects

- `EnrollmentID`
- tipos primitivos com bounds

## Invariantes

- **INV-037**: `progressPercent = 100` → dispara certificate (idempotente)
- UNIQUE(user_id, course_id) — 1 matrícula única por par
- Parceiro Vizinhança: sem enrollment (validação caller; INV-067)

## Factories

```go
func NewEnrollment(userID UserID, courseID CourseID) (*Enrollment, error) {
    // progressPercent=0
}

func ReconstructEnrollment(...) *Enrollment
```

## Métodos de domínio

- `CompleteLesson(lessonID LessonID, totalLessons int) error` — adiciona em `completedLessonIDs`; recalcula `progressPercent`
- `SubmitQuizScore(moduleID ModuleID, score int) error`
- `IsComplete() bool` — `progressPercent == 100`
- `MarkComplete()` — seta `completedAt`; dispara `CourseCompleted`

## Repository interface

```go
type IEnrollmentRepository interface {
    Save(ctx context.Context, e *Enrollment) error
    FindByID(ctx context.Context, id EnrollmentID) (*Enrollment, error)
    FindByUserAndCourse(ctx context.Context, uid UserID, cid CourseID) (*Enrollment, error)
    ListByUser(ctx context.Context, uid UserID) ([]*Enrollment, error)
    ListCompletedByUser(ctx context.Context, uid UserID) ([]*Enrollment, error)
}
```

## Eventos emitidos

- `EnrollmentCreated` (E-058)
- `LessonCompleted` (E-059)
- `CourseCompleted` (E-060)

## Links

- [[../bounded-contexts#5-content]]
- [[../invariants#content-inv-056-a-inv-070]]
- [[Course]] · [[Certificate]]
