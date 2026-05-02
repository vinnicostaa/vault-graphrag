---
title: System Architecture — API Design
parent: System Architecture
domain: ALL
status: Draft
tags:
  - mastersindico
  - architecture
  - api
  - endpoints
  - rest
---

# API Design — Endpoints por Módulo

> **Base URL:** `/v1/`
> **Auth:** Cookie-based (JWT session)
> **Formato:** JSON
> **Docs:** Swagger + Scalar (já configurado)
> **Validação:** Zod v4 (compartilhado via packages/schemas)
> **Total estimado:** 150+ endpoints

---

## Convenções

- Rotas versionadas: `/v1/<module>/...`
- RESTful: GET (lista/detalhe), POST (criar), PUT (update completo), PATCH (update parcial), DELETE (soft delete)
- Tenant scoping automático via middleware
- Paginação: `?page=1&limit=20`
- Filtros: query params
- Sorting: `?sort=created_at&order=desc`
- Erros: `{ statusCode, error, message, details? }`

---

## Identity (Existente ✅)

### Auth
```
POST   /v1/auth/register
POST   /v1/auth/login
POST   /v1/auth/logout
POST   /v1/auth/refresh
POST   /v1/auth/verify-email
POST   /v1/auth/forgot-password
POST   /v1/auth/reset-password
GET    /v1/auth/me
GET    /v1/auth/sessions
DELETE /v1/auth/sessions/:id
```

### Users
```
GET    /v1/users/:id
PUT    /v1/users/:id
GET    /v1/users/:id/vinculos
POST   /v1/users/:id/vinculos
DELETE /v1/users/:id/vinculos/:vinculoId
```

### Profiles
```
GET    /v1/profiles/:id
PUT    /v1/profiles/:id
GET    /v1/profiles/syndic/:userId
GET    /v1/profiles/enterprise/:userId
GET    /v1/profiles/resident/:userId
```

### Billing
```
GET    /v1/billing/plans
GET    /v1/billing/subscription
POST   /v1/billing/subscribe
POST   /v1/billing/cancel
POST   /v1/billing/change-plan
POST   /v1/billing/webhooks/stripe
GET    /v1/billing/invoices
GET    /v1/billing/feature-usage
```

### Onboarding
```
GET    /v1/onboarding/progress
POST   /v1/onboarding/start
PATCH  /v1/onboarding/progress
POST   /v1/onboarding/complete
```

### Search
```
GET    /v1/search/companies?q=&category=&city=
GET    /v1/search/sindicos?q=&city=&type=
GET    /v1/search/videos?q=&type=
GET    /v1/search/partners?q=&category=&neighborhood=
GET    /v1/search/curricula?q=&area=&cnh=
GET    /v1/search/courses?q=&category=&level=
GET    /v1/search/forum?q=&category=
```

---

## Institutional (Sprint 2+)

### Condominiums
```
POST   /v1/condominiums
GET    /v1/condominiums
GET    /v1/condominiums/:id
PUT    /v1/condominiums/:id
DELETE /v1/condominiums/:id

POST   /v1/condominiums/:id/units
GET    /v1/condominiums/:id/units
PUT    /v1/condominiums/:id/units/:unitId

GET    /v1/condominiums/:id/mandate
POST   /v1/condominiums/:id/mandate/transfer
POST   /v1/condominiums/:id/mandate/end

GET    /v1/condominiums/:id/moradores
POST   /v1/condominiums/:id/moradores/invite
```

### Governance — Timeline
```
GET    /v1/condominiums/:id/timeline
GET    /v1/condominiums/:id/timeline/:entryId
POST   /v1/condominiums/:id/timeline
POST   /v1/condominiums/:id/timeline/:entryId/adendo
```

### Governance — Master Plan
```
GET    /v1/condominiums/:id/master-plan
POST   /v1/condominiums/:id/master-plan
GET    /v1/condominiums/:id/master-plan/:actionId
PUT    /v1/condominiums/:id/master-plan/:actionId
GET    /v1/condominiums/:id/master-plan/dashboard
```

### Governance — Events
```
GET    /v1/condominiums/:id/events
POST   /v1/condominiums/:id/events
GET    /v1/condominiums/:id/events/:eventId
PUT    /v1/condominiums/:id/events/:eventId
POST   /v1/condominiums/:id/events/:eventId/participate
```

### Governance — Comunicados
```
GET    /v1/condominiums/:id/comunicados
POST   /v1/condominiums/:id/comunicados
GET    /v1/condominiums/:id/comunicados/:comunicadoId
POST   /v1/condominiums/:id/comunicados/:comunicadoId/read
```

### Governance — Consultas Moradores
```
GET    /v1/condominiums/:id/questions
POST   /v1/condominiums/:id/questions
POST   /v1/condominiums/:id/questions/:questionId/vote
GET    /v1/condominiums/:id/questions/:questionId/results
POST   /v1/condominiums/:id/questions/:questionId/close
```

### Validations
```
GET    /v1/condominiums/:id/validations
GET    /v1/condominiums/:id/validations/:itemId
POST   /v1/condominiums/:id/validations/:itemId/approve
POST   /v1/condominiums/:id/validations/:itemId/reject
POST   /v1/condominiums/:id/validations/:itemId/request-adjustment
```

### Compliance
```
GET    /v1/condominiums/:id/compliance/declarations
POST   /v1/condominiums/:id/compliance/declarations
GET    /v1/condominiums/:id/compliance/conflicts
POST   /v1/condominiums/:id/compliance/conflicts
GET    /v1/condominiums/:id/compliance/governance-score
GET    /v1/condominiums/:id/compliance/risk-map
GET    /v1/condominiums/:id/compliance/audit-trail
POST   /v1/condominiums/:id/compliance/dossier/export
```

### Evaluations
```
GET    /v1/condominiums/:id/evaluations/management
POST   /v1/condominiums/:id/evaluations/management
GET    /v1/condominiums/:id/evaluations/management/results
GET    /v1/companies/:companyId/evaluations
POST   /v1/companies/:companyId/evaluations
```

---

## Commercial (Sprint 3+)

### Connect Me
```
POST   /v1/connect-me
GET    /v1/connect-me
GET    /v1/connect-me/:id
POST   /v1/connect-me/:id/interest
POST   /v1/connect-me/:id/decline
POST   /v1/connect-me/:id/respond (Morador→Síndico)
GET    /v1/connect-me/quotas
```

### Proposals
```
POST   /v1/proposals
GET    /v1/proposals
GET    /v1/proposals/:id
PUT    /v1/proposals/:id
POST   /v1/proposals/:id/send
POST   /v1/condominiums/:id/proposals/:proposalId/validate
POST   /v1/condominiums/:id/proposals/:proposalId/reject
```

### Supplier Voting
```
POST   /v1/condominiums/:id/votings
GET    /v1/condominiums/:id/votings
GET    /v1/condominiums/:id/votings/:votingId
POST   /v1/condominiums/:id/votings/:votingId/start
POST   /v1/condominiums/:id/votings/:votingId/vote
POST   /v1/condominiums/:id/votings/:votingId/close
GET    /v1/condominiums/:id/votings/:votingId/results
```

### Closed Deals
```
POST   /v1/deals
GET    /v1/deals
GET    /v1/deals/:id
POST   /v1/deals/:id/sign/company
POST   /v1/deals/:id/sign/sindico
POST   /v1/deals/:id/cancel
```

### Service Execution
```
POST   /v1/companies/:companyId/executions
GET    /v1/companies/:companyId/executions
GET    /v1/companies/:companyId/executions/:id
PUT    /v1/companies/:companyId/executions/:id
POST   /v1/companies/:companyId/executions/:id/submit

POST   /v1/companies/:companyId/technical-activities
POST   /v1/companies/:companyId/comunicados
```

### Vizinhança
```
POST   /v1/partners
GET    /v1/partners
GET    /v1/partners/:id
PUT    /v1/partners/:id

POST   /v1/partners/:id/promotions
GET    /v1/partners/:id/promotions
PUT    /v1/partners/:id/promotions/:promoId
GET    /v1/partners/:id/promotions/:promoId/metrics

POST   /v1/condominiums/:id/partner-invitations
GET    /v1/condominiums/:id/nearby-partners
```

### Marketing Agency
```
POST   /v1/agencies/link
GET    /v1/agencies/linked-companies
POST   /v1/agencies/companies/:companyId/videos
```

---

## Content (Sprint 4+)

### Videos
```
POST   /v1/videos/upload
GET    /v1/videos
GET    /v1/videos/:id
PUT    /v1/videos/:id
DELETE /v1/videos/:id
POST   /v1/videos/:id/hide
GET    /v1/videos/:id/stream (Mux playback URL)
```

### Content Interaction
```
POST   /v1/content/:contentId/like
DELETE /v1/content/:contentId/like
POST   /v1/content/:contentId/rate
GET    /v1/forum/topics
POST   /v1/forum/topics
GET    /v1/forum/topics/:id
POST   /v1/forum/topics/:id/replies
```

### Academia
```
GET    /v1/academy/courses
GET    /v1/academy/courses/:id
POST   /v1/academy/courses (admin/empresa)
POST   /v1/academy/courses/:id/enroll
GET    /v1/academy/courses/:id/progress
POST   /v1/academy/courses/:id/lessons/:lessonId/complete
GET    /v1/academy/certificates
GET    /v1/academy/lives
```

### Talent Bank
```
POST   /v1/talent/curriculum
GET    /v1/talent/curriculum (meu currículo)
PUT    /v1/talent/curriculum
POST   /v1/talent/curriculum/video
GET    /v1/talent/search (empresas)
POST   /v1/talent/:curriculumId/favorite
DELETE /v1/talent/:curriculumId/favorite
POST   /v1/talent/:curriculumId/tags
GET    /v1/talent/:curriculumId/pdf
GET    /v1/talent/search-history
GET    /v1/talent/funnel-analytics
```

---

## Assembly (Sprint 5+)

### Configuration
```
POST   /v1/condominiums/:id/assemblies
GET    /v1/condominiums/:id/assemblies
GET    /v1/condominiums/:id/assemblies/:assemblyId
PUT    /v1/condominiums/:id/assemblies/:assemblyId
POST   /v1/condominiums/:id/assemblies/:assemblyId/publish (edital)
```

### Agenda
```
POST   /v1/assemblies/:id/agenda
GET    /v1/assemblies/:id/agenda
PUT    /v1/assemblies/:id/agenda/:itemId
DELETE /v1/assemblies/:id/agenda/:itemId
PATCH  /v1/assemblies/:id/agenda/reorder
```

### Pre-Assembly
```
POST   /v1/assemblies/:id/proxies
GET    /v1/assemblies/:id/proxies
POST   /v1/assemblies/:id/ciencia
GET    /v1/assemblies/:id/quorum-simulation
POST   /v1/assemblies/:id/polls
GET    /v1/assemblies/:id/polls/:pollId/results
```

### Live-Day
```
POST   /v1/assemblies/:id/start
POST   /v1/assemblies/:id/checkin
GET    /v1/assemblies/:id/attendance

POST   /v1/assemblies/:id/agenda/:itemId/open
POST   /v1/assemblies/:id/agenda/:itemId/close
PATCH  /v1/assemblies/:id/agenda/:itemId/status

POST   /v1/assemblies/:id/votes
GET    /v1/assemblies/:id/agenda/:itemId/votes/results
POST   /v1/assemblies/:id/agenda/:itemId/votes/homologate

POST   /v1/assemblies/:id/speech-queue/join
POST   /v1/assemblies/:id/speech-queue/start
POST   /v1/assemblies/:id/speech-queue/end
POST   /v1/assemblies/:id/speech-queue/extend
GET    /v1/assemblies/:id/speech-queue

POST   /v1/assemblies/:id/contingency/activate
POST   /v1/assemblies/:id/contingency/resolve

POST   /v1/assemblies/:id/finalize
```

### WebSocket
```
WSS    /v1/assemblies/:id/ws
       → Channels: telao, voting, speech, checkin, control
       → Auth: JWT query param
```

### Post-Assembly
```
GET    /v1/assemblies/:id/reports
GET    /v1/assemblies/:id/reports/:type (presence/voting/proxy/speech/consolidated/transparency)
GET    /v1/assemblies/:id/reports/:type/export?format=csv|pdf
GET    /v1/assemblies/:id/transparency-score
```

---

## Dashboards (Sprint 5+)

```
GET    /v1/dashboard/sindico
GET    /v1/dashboard/empresa
GET    /v1/dashboard/agencia
GET    /v1/dashboard/admin (admin panel)
```

---

## Navegação

← [[System Architecture - Data Model]] | [[System Architecture]]
