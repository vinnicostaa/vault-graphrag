---
title: Aggregate — Course
type: spec
tags: [domain, ddd, aggregate, content, lms, master-sindico, v2]
bc: content
source: 90-ingested/.../specs/requirements/content.md Req 98.1
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Course` (LMS)

**BC**: content · **Raiz**: ✅

Curso da Academia Master Síndico. Contém modules → lessons → quiz. M3 thin: 5 áreas, 3 trilhas piloto. M4+ pleno.

## Entidade raiz

```go
type Course struct {
    id              CourseID
    title           string
    description     string
    area            CourseArea              // cursos | treinamentos | lives | forum | biblioteca (5)
    category        CourseCategory          // governança | legislação | finanças | RH | manutenção | compliance | segurança | relações | empreendedorismo | tecnologia (10)
    trilha          *CourseTrilha           // sindico | empresa | morador (3)
    status          CourseStatus            // draft | published | archived
    modules         []CourseModule          // ordenadas
    accessLevel     AccessLevel             // free | plus | pro
    estimatedHours  int
    publishedAt     *time.Time
    createdAt       time.Time
    updatedAt       time.Time
}

type CourseModule struct {
    id              ModuleID
    courseID        CourseID
    order           int
    title           string
    lessons         []CourseLesson
    quiz            *Quiz
}

type CourseLesson struct {
    id              LessonID
    moduleID        ModuleID
    order           int
    title           string
    videoID         *VideoID
    textContent     *string
    durationMinutes int
}

type Quiz struct {
    id              QuizID
    moduleID        ModuleID
    questions       []QuizQuestion
    passThreshold   int                     // padrão 70%
}
```

## Value Objects

- `CourseID`, `ModuleID`, `LessonID`, `QuizID`
- `CourseArea`, `CourseCategory`, `CourseTrilha`, `CourseStatus`, `AccessLevel` (enums)
- `QuizQuestion` (entity: id, question, options, correct_index, weight)

## Invariantes

- Parceiro Vizinhança: **sem** acesso (INV-067)
- `passThreshold ∈ [1, 100]` default 70
- `publishedAt != NULL` ⇒ `status=published`

## Factories

```go
func NewCourse(title, description string, area CourseArea, category CourseCategory, trilha *CourseTrilha, accessLevel AccessLevel) (*Course, error) {
    // status=draft
}

func ReconstructCourse(...) *Course
```

## Métodos de domínio

- `AddModule(m CourseModule) error`
- `RemoveModule(moduleID ModuleID) error` — só se `status=draft`
- `Publish() error` — `draft → published`
- `Archive() error` — `published → archived`

## Repository interface

```go
type ICourseRepository interface {
    Save(ctx context.Context, c *Course) error
    FindByID(ctx context.Context, id CourseID) (*Course, error)
    ListByTrilha(ctx context.Context, t CourseTrilha) ([]*Course, error)
    ListByArea(ctx context.Context, a CourseArea) ([]*Course, error)
    ListPublished(ctx context.Context) ([]*Course, error)
    RecommendByExecutionRecord(ctx context.Context, category string) ([]*Course, error) // Req 23 integration
}
```

## Eventos emitidos

- `CoursePublished` (E-057)

## Links

- [[../bounded-contexts#5-content]]
- [[../invariants#content-inv-056-a-inv-070]]
- [[../business-rules]] BR-039, BR-040
- [[Enrollment]] · [[Certificate]]
