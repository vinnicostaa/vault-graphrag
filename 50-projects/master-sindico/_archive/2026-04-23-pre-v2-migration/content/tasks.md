# Implementation Plan: Master Síndico Platform

## Overview

This implementation plan breaks down the Master Síndico Platform into actionable coding tasks following Clean Architecture, DDD, and SOLID principles. The platform is a multi-tenant condominium management system with 8 bounded contexts, implementing BeyondCorp security model and event-driven architecture.

**Implementation Priority**: Core request flow first (authentication → context evaluation → policy enforcement → use case execution), then bounded contexts incrementally.

**Tech Stack**: Go 1.22+, PostgreSQL, GORM, Zitadel, NATS, Redis, OpenSearch, Gin

**Architecture**: Modular monolith with 9 independent products, Clean Architecture layers, strict tenant isolation

## Tasks

- [ ] 1. Project foundation and core infrastructure
  - [ ] 1.1 Initialize Go project with Clean Architecture structure
    - Create directory structure: `internal/{shared,identity,auth,policy,trust,governance,assembly,connectme,lms,neighborhood,billing,search,compliance}`
    - Create `cmd/api/main.go` entry point
    - Initialize Go modules with required dependencies (Gin, GORM, Zitadel SDK, NATS, Redis, gopter)
    - Setup `.env` configuration management
    - _Requirements: Architecture foundation_

  - [ ] 1.2 Setup database infrastructure with multi-tenancy support
    - Configure GORM with PostgreSQL connection
    - Create `tenants` table migration
    - Create `users` table migration with indexes
    - Implement tenant isolation at database level with composite indexes
    - Add Row-Level Security (RLS) policies for tenant isolation
    - _Requirements: 2.5_

  - [ ] 1.3 Implement shared kernel domain primitives
    - Create value objects: `Money`, `Address`, `DateRange`, `Duration`, `Attachment`
    - Create common types: `TenantID`, `UserID`, `EntityID`
    - Implement error types: `AppError` with `ErrorType` enum
    - Create domain event interface: `DomainEvent`
    - _Requirements: Shared domain concepts_


  - [ ]* 1.4 Write property tests for shared kernel
    - **Property 25: JSON Round-Trip Preservation**
    - **Validates: Requirements 68.4**
    - Test that Money, Address, DateRange serialize/deserialize correctly
    - _Requirements: 68.4_

- [ ] 2. Core request flow - Authentication layer (internal/auth/)
  - [ ] 2.1 Implement Zitadel integration as Anti-Corruption Layer
    - Create `internal/identity/` domain models: `User`, `Session`, `DeviceFingerprint`
    - Implement `ZitadelAdapter` in infrastructure layer
    - Create `UserRepository` interface and GORM implementation
    - Implement JWT token validation middleware
    - _Requirements: 1.1, 1.2, 1.3, 1.4_

  - [ ] 2.2 Implement authentication middleware with device fingerprinting
    - Create `AuthMiddleware` that extracts JWT from Authorization header
    - Validate token expiration and signature
    - Extract user claims (user_id, email, roles)
    - Implement device fingerprinting (device_id, IP, user_agent)
    - Store validated user context in Gin context
    - _Requirements: 1.6, 1.7, 1.9_

  - [ ]* 2.3 Write property tests for authentication
    - **Property 2: Expired Token Rejection**
    - **Validates: Requirements 1.6**
    - **Property 3: Authentication Attempt Logging**
    - **Validates: Requirements 1.7, 1.9**

  - [ ] 2.4 Implement session management with single-device enforcement
    - Create `SessionRepository` interface and implementation
    - Implement `AuthenticationService.Authenticate()` method
    - On new device login, invalidate previous device sessions
    - Store active session in Redis with TTL
    - _Requirements: 1.8, 1.10_

  - [ ]* 2.5 Write property test for single-device session
    - **Property 1: Single Device Session Enforcement**
    - **Validates: Requirements 1.8, 1.10**

  - [ ] 2.6 Implement rate limiting and fraud prevention
    - Create rate limiter middleware using Redis
    - Track failed login attempts per user (5 attempts, 15 min window)
    - Track account creation attempts per IP (3 accounts, 24h window)
    - Return 429 Too Many Requests when limits exceeded
    - _Requirements: 1.5, 1.11_

  - [ ]* 2.7 Write property tests for rate limiting
    - **Property 4: Rate Limiting Enforcement**
    - **Validates: Requirements 1.5**
    - **Property 5: IP-Based Fraud Prevention**
    - **Validates: Requirements 1.11**

- [ ] 3. Core request flow - Trust evaluation layer (internal/trust/)
  - [ ] 3.1 Implement trust context evaluator
    - Create `TrustContext` struct with fields: `user_id`, `tenant_id`, `role`, `plan`, `device_fingerprint`, `ip_address`
    - Implement `TrustEvaluator.Evaluate()` method
    - Extract user subscription and plan from database
    - Determine user role within tenant (sindico, morador, empresa, etc)
    - Store trust context in request context
    - _Requirements: 2.1, 2.2, 2.3_

  - [ ] 3.2 Implement tenant context injection middleware
    - Create `TenantMiddleware` that runs after authentication
    - Extract tenant_id from user's active subscription
    - Inject tenant_id into request context
    - Validate user has active subscription for tenant
    - _Requirements: 2.5_

  - [ ]* 3.3 Write property test for tenant isolation
    - **Property 6: Tenant Isolation**
    - **Validates: Requirements 2.5**


- [ ] 4. Core request flow - Policy engine (internal/policy/)
  - [ ] 4.1 Implement ABAC policy engine foundation
    - Create `Policy` struct with fields: `resource`, `action`, `conditions`
    - Create `PolicyEngine` interface with `Evaluate()` method
    - Implement basic policy matching logic
    - Define policy conditions: `role`, `plan`, `tenant_ownership`
    - _Requirements: 2.4_

  - [ ] 4.2 Define policies for core resources
    - Create policy definitions file (YAML or Go structs)
    - Define policies for Timeline: sindico can write, morador can read
    - Define policies for Condominium: sindico can manage, morador can view
    - Define policies for Assembly: sindico can create, morador can participate
    - Define policies for ConnectMe: plan-based quotas
    - _Requirements: 2.4, 11.12, 19.7, 19.8_

  - [ ] 4.3 Implement authorization middleware
    - Create `AuthorizationMiddleware` that runs after trust evaluation
    - Extract resource and action from request (path + method)
    - Call `PolicyEngine.Evaluate()` with trust context and policy
    - Return 403 Forbidden if policy evaluation fails
    - Log unauthorized access attempts to audit log
    - _Requirements: 2.6_

  - [ ]* 4.4 Write property tests for authorization
    - **Property 7: Unauthorized Access Logging**
    - **Validates: Requirements 2.6**
    - **Property 15: Morador Read-Only Access**
    - **Validates: Requirements 11.12**

  - [ ] 4.5 Implement cross-tenant grant mechanism
    - Create `CrossTenantGrant` domain model
    - Create `CrossTenantGrantRepository` interface and implementation
    - Implement grant validation in policy engine
    - Check grant expiration and consent status
    - _Requirements: 2.7_

  - [ ]* 4.6 Write property test for cross-tenant grants
    - **Property 8: Cross-Tenant Grant Validation**
    - **Validates: Requirements 2.7**

- [ ] 5. Checkpoint - Core request flow validation
  - Ensure all tests pass for authentication, trust evaluation, and policy enforcement
  - Manually test complete request flow: JWT → auth → trust → policy → 403/200
  - Ask the user if questions arise about the core flow

- [ ] 6. Identity bounded context - Subscription management
  - [ ] 6.1 Implement Subscription domain model
    - Create `Subscription` aggregate with fields: `user_id`, `tenant_id`, `plan`, `status`, `trial_end_date`
    - Define `Plan` enum: `trial`, `base`, `plus`, `pro`
    - Define `SubscriptionStatus` enum: `active`, `expired`, `cancelled`
    - Implement domain methods: `IsActive()`, `IsTrialExpired()`, `GetFeatureAccess()`
    - _Requirements: 3.1, 3.2, 3.3_

  - [ ] 6.2 Create subscription database schema and repository
    - Create `subscriptions` table migration
    - Implement `SubscriptionRepository` with GORM
    - Add methods: `Create()`, `FindByUserID()`, `FindByTenantID()`, `Update()`
    - Add indexes on `user_id`, `tenant_id`, `status`
    - _Requirements: 3.1_

  - [ ] 6.3 Implement subscription use cases
    - Create `CreateSubscriptionUseCase` (trial by default)
    - Create `UpgradeSubscriptionUseCase` (trial → paid plan)
    - Create `CheckTrialExpirationUseCase` (background job)
    - Emit domain events: `SubscriptionCreated`, `SubscriptionExpired`, `TrialExpired`
    - _Requirements: 3.4, 3.5, 3.6_

  - [ ] 6.4 Implement plan-based feature access
    - Create `FeatureAccessService` that checks plan limits
    - Define feature limits per plan (condominium count, ConnectMe quota, etc)
    - Integrate with policy engine for quota enforcement
    - _Requirements: 3.7, 3.8, 3.9_

  - [ ]* 6.5 Write unit tests for subscription logic
    - Test trial expiration calculation
    - Test plan upgrade flows
    - Test feature access checks per plan
    - _Requirements: 3.1-3.9_


- [ ] 7. Institutional bounded context - Condominium management
  - [ ] 7.1 Implement Condominium aggregate root
    - Create `Condominium` entity with fields: `id`, `tenant_id`, `name`, `address`, `total_units`, `alphanumeric_id`, `status`
    - Create `Mandate` entity with fields: `sindico_id`, `start_date`, `end_date`, `mandate_type`
    - Create `Unit` entity with fields: `block`, `number`, `owner_name`, `owner_cpf`
    - Implement domain methods: `AddUnit()`, `StartMandate()`, `EndMandate()`
    - _Requirements: 4.1, 4.2, 4.3, 4.4_

  - [ ] 7.2 Create condominium database schema
    - Create `condominiums` table with tenant_id composite indexes
    - Create `mandates` table with foreign key to condominiums
    - Create `units` table with foreign key to condominiums
    - Add unique constraint on `alphanumeric_id`
    - _Requirements: 4.1, 4.2_

  - [ ] 7.3 Implement CondominiumRepository
    - Create repository interface with methods: `Create()`, `FindByID()`, `FindBySindicoID()`, `Update()`
    - Implement GORM-based repository with tenant isolation
    - Ensure all queries include `WHERE tenant_id = ?`
    - _Requirements: 2.5, 4.1_

  - [ ] 7.4 Implement condominium creation use case
    - Create `CreateCondominiumUseCase`
    - Validate sindico has not exceeded condominium quota (15 for trial/base)
    - Generate unique 8-character alphanumeric_id
    - Generate QR code for condominium
    - Create initial mandate for sindico
    - Emit `CondominiumCreated` event
    - _Requirements: 4.5, 4.6, 4.7, 4.8_

  - [ ] 7.5 Implement unit management use cases
    - Create `RegisterUnitUseCase`
    - Validate unit count does not exceed `total_units`
    - Create `LinkMoradorToUnitUseCase` (titular or dependente)
    - Implement unit ownership validation
    - _Requirements: 5.1, 5.2, 5.3, 5.4_

  - [ ]* 7.6 Write unit tests for condominium domain logic
    - Test unit capacity validation
    - Test mandate date validation
    - Test alphanumeric_id uniqueness
    - _Requirements: 4.1-5.4_

- [ ] 8. Institutional bounded context - User profiles and roles
  - [ ] 8.1 Implement user profile domain models
    - Create `SindicoProfile` with fields: `user_id`, `condominium_ids`, `mandate_history`
    - Create `MoradorProfile` with fields: `user_id`, `unit_id`, `role` (titular/dependente)
    - Create `CompanyProfile` with fields: `user_id`, `cnpj`, `service_categories`, `service_area`
    - Create `PartnerProfile` with fields: `user_id`, `cnpj`, `partner_type`, `service_area`
    - _Requirements: 6.1, 6.2, 6.3, 6.4_

  - [ ] 8.2 Create profile database schemas
    - Create `sindico_profiles` table
    - Create `morador_profiles` table with foreign key to units
    - Create `company_profiles` table
    - Create `partner_profiles` table
    - Add indexes on `user_id` for all profile tables
    - _Requirements: 6.1-6.4_

  - [ ] 8.3 Implement profile creation use cases
    - Create `CreateSindicoProfileUseCase` (triggered on first condominium creation)
    - Create `CreateMoradorProfileUseCase` (triggered on unit linkage)
    - Create `CreateCompanyProfileUseCase` (self-registration)
    - Create `CreatePartnerProfileUseCase` (invitation-based)
    - _Requirements: 6.5, 6.6, 6.7_

  - [ ] 8.4 Implement empresa síndica delegation
    - Create `EmpresaSindica` value object with permissions
    - Implement `DelegateToEmpresaSindicaUseCase`
    - Define permission flags: `view_publications`, `edit_publications`, `validate_executions`
    - Store delegation in mandate entity
    - _Requirements: 7.1, 7.2, 7.3_

  - [ ]* 8.5 Write unit tests for profile management
    - Test profile creation flows
    - Test empresa síndica permission validation
    - Test morador role assignment (titular vs dependente)
    - _Requirements: 6.1-7.3_


- [ ] 9. Content bounded context - Timeline (immutable, append-only)
  - [ ] 9.1 Implement Timeline aggregate root
    - Create `TimelineEntry` entity with fields: `id`, `tenant_id`, `entry_type`, `title`, `description`, `date`, `author_id`, `published_at`
    - Define `TimelineEntryType` enum: `atividade_da_gestao`, `registro_de_execucao`, `comunicado`, `evento`, `adendo`
    - Create `Adendo` entity with fields: `original_entry_id`, `amendment_text`, `created_at`
    - Mark Timeline as append-only (no Update or Delete methods)
    - _Requirements: 11.1, 11.2, 11.3, 11.4_

  - [ ] 9.2 Create timeline database schema
    - Create `timeline_entries` table with `tenant_id` index
    - Create `timeline_attachments` table (normalized)
    - Add index on `(tenant_id, date DESC)` for efficient queries
    - Add index on `entry_type` for filtering
    - Ensure no UPDATE or DELETE triggers exist (append-only enforcement)
    - _Requirements: 11.1, 11.5_

  - [ ] 9.3 Implement TimelineRepository (append-only)
    - Create repository interface with methods: `Create()`, `FindByID()`, `FindByTenantID()`
    - Explicitly omit `Update()` and `Delete()` methods
    - Implement GORM repository with tenant isolation
    - Add pagination support for timeline queries
    - _Requirements: 11.1, 11.7_

  - [ ]* 9.4 Write property test for timeline immutability
    - **Property 9: Timeline Immutability**
    - **Validates: Requirements 11.1, 11.7**
    - **Property 10: Timeline Entry Uniqueness**
    - **Validates: Requirements 11.5**
    - **Property 11: Timeline Publication Timestamp**
    - **Validates: Requirements 11.6**

  - [ ] 9.5 Implement timeline publication use case
    - Create `PublishTimelineEntryUseCase`
    - Validate author is sindico or empresa síndica with permissions
    - Set `published_at` timestamp automatically
    - Upload attachments to S3 and store URLs
    - Emit `TimelinePublished` event
    - _Requirements: 11.6, 11.10_

  - [ ] 9.6 Implement adendo creation use case
    - Create `CreateAdendoUseCase`
    - Validate original entry exists and belongs to same tenant
    - Create new Timeline entry with type `adendo`
    - Mark original entry as `amended = true`
    - Link adendo to original entry
    - _Requirements: 11.8, 11.9_

  - [ ]* 9.7 Write property test for adendo linkage
    - **Property 12: Adendo Linkage**
    - **Validates: Requirements 11.8, 11.9**

  - [ ] 9.8 Implement timeline query use cases
    - Create `GetTimelineUseCase` with filters (entry_type, date_range)
    - Implement pagination (50 entries per page)
    - Ensure morador users can only read, not write
    - Return entries sorted by date descending
    - _Requirements: 11.12_

- [ ] 10. Content bounded context - Master Plan (reactive status)
  - [ ] 10.1 Implement MasterPlan aggregate root
    - Create `MasterPlanAction` entity with fields: `id`, `tenant_id`, `title`, `area_impactada`, `tipo_atividade`, `risco_associado`, `estimated_date`, `status`
    - Define `MasterPlanStatus` enum: `planejada`, `em_contratacao`, `em_execucao`, `concluida`, `atrasada`
    - Implement `CalculateRiskScore()` method based on risk factors
    - Implement `UpdateStatusBasedOnTimeline()` method (reactive)
    - _Requirements: 12.1, 12.2, 12.3, 12.4_

  - [ ] 10.2 Create master plan database schema
    - Create `master_plan_actions` table with `tenant_id` index
    - Add index on `(tenant_id, status)` for filtering
    - Add index on `(tenant_id, estimated_date)` for date queries
    - Add foreign key to `timeline_entries` (optional, for linkage)
    - _Requirements: 12.1_

  - [ ] 10.3 Implement MasterPlanRepository
    - Create repository interface with CRUD methods
    - Implement GORM repository with tenant isolation
    - Add query methods: `FindByStatus()`, `FindOverdue()`, `FindByRiskScore()`
    - _Requirements: 12.1_

  - [ ] 10.4 Implement master plan creation use case
    - Create `CreateMasterPlanActionUseCase`
    - Calculate initial risk score based on inputs
    - Set initial status to `planejada`
    - Validate estimated_date is in the future
    - _Requirements: 12.5, 12.6_


  - [ ] 10.5 Implement reactive status update via event handler
    - Create `MasterPlanEventHandler` that listens to `TimelinePublished` events
    - When Timeline entry has `related_action_id`, update MasterPlan status to `concluida`
    - Implement automatic status calculation: if `estimated_date` passed and status != `concluida`, set to `atrasada`
    - _Requirements: 11.11, 12.7_

  - [ ]* 10.6 Write property test for master plan status automation
    - **Property 14: Master Plan Status Automation**
    - **Validates: Requirements 11.11**

  - [ ]* 10.7 Write unit tests for master plan logic
    - Test risk score calculation
    - Test overdue detection
    - Test status transitions
    - _Requirements: 12.1-12.7_

- [ ] 11. Checkpoint - Content contexts validation
  - Ensure Timeline is truly immutable (no update/delete operations work)
  - Verify MasterPlan status updates automatically when Timeline is published
  - Test morador read-only access to Timeline
  - Ask the user if questions arise about content management

- [ ] 12. Commercial bounded context - Connect Me (LGPD-compliant marketplace)
  - [ ] 12.1 Implement ConnectMe aggregate root
    - Create `ConnectMeRequest` entity with fields: `id`, `source_tenant_id`, `target_tenant_id`, `request_type`, `service_category`, `urgency`, `status`
    - Define `ConnectMeType` enum: `sindico_to_company`, `morador_to_sindico`, `company_to_company`, `sindico_to_partner`
    - Create `LGPDLog` value object with fields: `user_id`, `ip_address`, `timestamp`, `terms_version`
    - Implement domain methods: `ShowInterest()`, `RevealContactData()`, `Expire()`
    - _Requirements: 19.1, 19.2, 19.3, 19.4_

  - [ ] 12.2 Create connect_me database schema
    - Create `connect_me_requests` table with indexes on `source_tenant_id` and `target_tenant_id`
    - Add `data_revealed` boolean flag (default false)
    - Add `interest_shown_at` timestamp
    - Add `expires_at` timestamp (48h from creation)
    - Add `lgpd_log` JSONB field
    - _Requirements: 19.1, 19.6_

  - [ ] 12.3 Implement ConnectMeRepository
    - Create repository interface with methods: `Create()`, `FindBySourceTenant()`, `FindByTargetTenant()`, `Update()`
    - Implement GORM repository
    - Add method `FindExpiredRequests()` for background job
    - _Requirements: 19.1_

  - [ ] 12.4 Implement ConnectMe creation use case
    - Create `CreateConnectMeRequestUseCase`
    - Check user's plan quota (trial: 3, base: 10, plus: 30, pro: unlimited)
    - Validate service category exists
    - Set `expires_at` to 48 hours from now
    - Initially hide sindico contact data (name, phone, email)
    - Emit `ConnectMeRequestCreated` event
    - _Requirements: 19.5, 19.7, 19.8_

  - [ ]* 12.5 Write property test for ConnectMe quota enforcement
    - **Property 18: Connect Me Quota Enforcement**
    - **Validates: Requirements 19.7, 19.8**

  - [ ] 12.6 Implement interest and data revelation use case
    - Create `ShowInterestUseCase` (company clicks "Tenho interesse")
    - Set `interest_shown_at` timestamp
    - Create LGPD log entry with company_id, ip_address, timestamp
    - Reveal sindico contact data (name, phone, email)
    - Set `data_revealed = true`
    - Emit `ConnectMeInterestShown` event
    - _Requirements: 19.3, 19.6, 19.9_

  - [ ]* 12.7 Write property tests for ConnectMe LGPD compliance
    - **Property 16: Contact Data Privacy**
    - **Validates: Requirements 19.3**
    - **Property 17: LGPD Data Revelation Logging**
    - **Validates: Requirements 19.6**

  - [ ] 12.8 Implement ConnectMe auto-expiration background job
    - Create `ExpireConnectMeRequestsJob` (runs every hour)
    - Find requests where `expires_at < now()` and `interest_shown_at IS NULL`
    - Update status to `expired`
    - Send notification to requester
    - _Requirements: 19.11, 19.12_

  - [ ]* 12.9 Write property test for auto-expiration
    - **Property 19: Connect Me Auto-Expiration**
    - **Validates: Requirements 19.11, 19.12**


- [ ] 13. Commercial bounded context - Proposals and closed deals
  - [ ] 13.1 Implement Proposal aggregate root
    - Create `Proposal` entity with fields: `id`, `company_id`, `condominium_id`, `connect_me_id`, `title`, `scope`, `estimated_cost`, `status`
    - Define `ProposalStatus` enum: `pending`, `validated`, `rejected`, `accepted`
    - Implement domain methods: `Validate()`, `Reject()`, `Accept()`
    - _Requirements: 20.1, 20.2, 20.3_

  - [ ] 13.2 Create proposals database schema
    - Create `proposals` table with indexes on `company_id` and `condominium_id`
    - Add foreign key to `connect_me_requests` (optional)
    - Add `validation_by` field (sindico_id)
    - Add `validated_at` timestamp
    - _Requirements: 20.1_

  - [ ] 13.3 Implement proposal creation and validation use cases
    - Create `CreateProposalUseCase` (company submits proposal)
    - Create `ValidateProposalUseCase` (sindico approves/rejects)
    - Emit `ProposalSent` and `ProposalValidated` events
    - _Requirements: 20.4, 20.5, 20.6_

  - [ ] 13.4 Implement ClosedDeal aggregate root
    - Create `ClosedDeal` entity with fields: `id`, `company_id`, `condominium_id`, `proposal_id`, `agreed_cost`, `start_date`, `end_date`, `status`
    - Create `ServiceExecution` entity with fields: `deal_id`, `description`, `date`, `photos`, `videos`, `risk_level`, `status`
    - Implement domain methods: `SignByCompany()`, `SignBySindico()`, `RegisterExecution()`
    - _Requirements: 21.1, 21.2, 21.3_

  - [ ] 13.5 Create closed deals database schema
    - Create `closed_deals` table with dual signature tracking
    - Create `service_executions` table with foreign key to `closed_deals`
    - Add indexes on `company_id`, `condominium_id`, `status`
    - _Requirements: 21.1_

  - [ ] 13.6 Implement closed deal use cases
    - Create `CreateClosedDealUseCase` (from accepted proposal)
    - Create `SignDealUseCase` (company and sindico signatures)
    - Create `RegisterServiceExecutionUseCase` (company registers work done)
    - Create `ValidateServiceExecutionUseCase` (sindico validates)
    - Emit `DealConfirmed` event
    - _Requirements: 21.4, 21.5, 21.6, 21.7_

  - [ ] 13.7 Implement service execution to timeline integration
    - When service execution is validated, create Timeline entry automatically
    - Set entry type to `registro_de_execucao`
    - Include photos, videos, and description
    - Link to MasterPlan action if applicable
    - _Requirements: 21.8, 21.9_

  - [ ]* 13.8 Write unit tests for proposal and deal workflows
    - Test proposal validation flows
    - Test dual signature requirement
    - Test execution validation
    - _Requirements: 20.1-21.9_

- [ ] 14. Assembly bounded context - Foundation and creation
  - [ ] 14.1 Implement Assembly aggregate root
    - Create `Assembly` entity with fields: `id`, `tenant_id`, `assembly_type`, `convocation_date`, `scheduled_date`, `location`, `status`
    - Define `AssemblyType` enum: `ordinaria`, `extraordinaria`
    - Define `AssemblyStatus` enum: `convocada`, `em_andamento`, `finalizada`, `cancelada`
    - Create `QuorumConfig` value object with fields: `type`, `required_percent`, `fracao_ideal_based`
    - Implement domain methods: `Start()`, `Finalize()`, `Cancel()`
    - _Requirements: 48.1, 48.2_

  - [ ] 14.2 Implement AgendaItem entity
    - Create `AgendaItem` entity with fields: `id`, `assembly_id`, `order`, `title`, `description`, `item_type`, `voting_required`, `vote_type`
    - Define `AgendaItemType` enum: `informativo`, `deliberativo`, `votacao`
    - Define `VoteType` enum: `maioria_simples`, `maioria_qualificada`, `unanimidade`
    - Implement domain methods: `OpenVoting()`, `CloseVoting()`, `Homologate()`
    - _Requirements: 49.1, 49.2_

  - [ ] 14.3 Create assembly database schema
    - Create `assemblies` table with `tenant_id` index
    - Create `agenda_items` table with foreign key to assemblies
    - Add unique constraint on `(assembly_id, order)` for agenda items
    - Add indexes on `scheduled_date` and `status`
    - _Requirements: 48.1, 49.1_

  - [ ] 14.4 Implement assembly creation use case
    - Create `CreateAssemblyUseCase`
    - Validate convocation lead time (ordinaria: 8 days, extraordinaria: 3 days)
    - Create agenda items from input
    - Set initial status to `convocada`
    - Send convocation notifications to all moradores
    - Emit `AssemblyCreated` event
    - _Requirements: 48.3, 48.4, 48.5_

  - [ ]* 14.5 Write property test for convocation lead time
    - **Property 20: Assembly Convocation Lead Time**
    - **Validates: Requirements 48.3, 48.4**


- [ ] 15. Assembly bounded context - Attendance and check-in
  - [ ] 15.1 Implement AttendanceRecord entity
    - Create `AttendanceRecord` entity with fields: `id`, `assembly_id`, `morador_id`, `unit_id`, `check_in_method`, `check_in_at`, `status`
    - Define `CheckInMethod` enum: `app`, `qr_code`, `terminal_presencial`
    - Define `AttendanceStatus` enum: `presente`, `ausente_momentaneo`, `desconectado_saiu`
    - _Requirements: 52.1, 52.2_

  - [ ] 15.2 Create attendance database schema
    - Create `attendance_records` table with foreign key to assemblies
    - Add indexes on `assembly_id` and `morador_id`
    - Add unique constraint on `(assembly_id, unit_id)` to prevent duplicate check-ins
    - _Requirements: 52.1_

  - [ ] 15.3 Implement check-in use cases
    - Create `CheckInMoradorUseCase` (app-based check-in)
    - Create `CheckInViaQRCodeUseCase` (QR code scan)
    - Create `CheckInViaTerminalUseCase` (presencial terminal)
    - Validate morador belongs to condominium
    - Validate assembly is in `em_andamento` status
    - Emit `MoradorCheckedIn` event
    - _Requirements: 52.3, 52.4, 52.5_

  - [ ] 15.4 Implement Proxy entity and use cases
    - Create `Proxy` entity with fields: `id`, `assembly_id`, `delegator_unit_id`, `delegate_unit_id`, `document`, `validated_by`
    - Create `RegisterProxyUseCase` (morador delegates vote)
    - Create `ValidateProxyUseCase` (sindico validates proxy document)
    - Store proxy document URL in S3
    - _Requirements: 53.1, 53.2, 53.3_

  - [ ]* 15.5 Write unit tests for attendance and proxy
    - Test duplicate check-in prevention
    - Test proxy validation
    - Test check-in method variations
    - _Requirements: 52.1-53.3_

- [ ] 16. Assembly bounded context - Voting system
  - [ ] 16.1 Implement Vote entity
    - Create `Vote` entity with fields: `id`, `agenda_item_id`, `unit_id`, `morador_id`, `vote_option`, `weight`, `vote_origin`, `cast_at`
    - Define `VoteOption` enum: `a_favor`, `contra`, `abstencao`
    - Define `VoteOrigin` enum: `app`, `presencial_assistido`, `proxy`
    - Add unique constraint on `(agenda_item_id, unit_id)` to prevent duplicate votes
    - _Requirements: 57.1, 57.2_

  - [ ] 16.2 Create votes database schema
    - Create `votes` table with foreign key to agenda_items
    - Add indexes on `agenda_item_id` and `morador_id`
    - Add `weight` field for fração ideal (default 1.0)
    - _Requirements: 57.1_

  - [ ] 16.3 Implement vote casting use case
    - Create `CastVoteUseCase`
    - Validate agenda item voting is open
    - Validate morador is checked in or has proxy
    - Prevent duplicate votes from same unit
    - Calculate vote weight based on fração ideal
    - Persist vote immediately (auto-save)
    - Emit `VoteCast` event
    - _Requirements: 57.3, 57.8_

  - [ ]* 16.4 Write property test for vote uniqueness
    - **Property 21: Vote Uniqueness Per Unit**
    - **Validates: Requirements 57.3**

  - [ ] 16.5 Implement proxy vote replication
    - When delegate casts vote, automatically create vote for delegator's unit
    - Set `vote_origin = proxy` for delegator's vote
    - Use same `vote_option` as delegate
    - Apply delegator's fração ideal weight
    - _Requirements: 57.4_

  - [ ]* 16.6 Write property test for proxy vote replication
    - **Property 22: Proxy Vote Replication**
    - **Validates: Requirements 57.4**

  - [ ] 16.7 Implement vote closure and immutability
    - Create `CloseVotingUseCase` (sindico closes voting for agenda item)
    - Mark agenda item as `closed`
    - Make all votes for that item immutable (reject updates/deletes)
    - Calculate voting results
    - _Requirements: 57.6, 57.7_

  - [ ]* 16.8 Write property test for vote immutability
    - **Property 23: Vote Immutability After Closure**
    - **Validates: Requirements 57.6, 57.7**
    - **Property 24: Assembly Auto-Save**
    - **Validates: Requirements 57.8**

  - [ ] 16.9 Implement vote homologation use case
    - Create `HomologateVotingUseCase` (sindico or secretary homologates results)
    - Validate quorum was met
    - Mark agenda item as `homologated`
    - Store homologator_id and timestamp
    - Emit `VoteHomologated` event
    - _Requirements: 57.9, 57.10_


- [ ] 17. Assembly bounded context - Quorum and finalization
  - [ ] 17.1 Implement QuorumService
    - Create `QuorumService` with method `CalculateQuorum(assembly)`
    - Calculate attendance percentage based on checked-in units
    - Support fração ideal-based calculation
    - Return quorum status: `atingido`, `nao_atingido`, `segunda_convocacao`
    - _Requirements: 54.1, 54.2_

  - [ ] 17.2 Implement quorum validation for voting
    - Before opening voting on agenda item, validate quorum is met
    - If quorum not met, prevent voting and notify president
    - Support different quorum requirements per agenda item type
    - _Requirements: 54.3_

  - [ ] 17.3 Implement assembly finalization use case
    - Create `FinalizeAssemblyUseCase`
    - Validate all agenda items are closed and homologated
    - Generate assembly report (ata)
    - Calculate transparency indicator
    - Update assembly status to `finalizada`
    - Publish assembly results to Timeline
    - Emit `AssemblyFinalized` event
    - _Requirements: 58.1, 58.2, 58.3, 58.4_

  - [ ] 17.4 Implement transparency indicator calculation
    - Create `TransparencyIndicator` value object with scores for: convocacao, participacao, votacao, documentacao
    - Calculate overall transparency score (0-100)
    - Store indicator in assembly record
    - _Requirements: 58.5_

  - [ ]* 17.5 Write unit tests for quorum and finalization
    - Test quorum calculation with different attendance levels
    - Test fração ideal weighting
    - Test transparency score calculation
    - _Requirements: 54.1-58.5_

- [ ] 18. Checkpoint - Assembly module validation
  - Ensure complete assembly flow works: creation → convocation → check-in → voting → homologation → finalization
  - Test proxy vote replication
  - Verify vote immutability after closure
  - Ask the user if questions arise about assembly implementation

- [ ] 19. Compliance bounded context - Audit trail
  - [ ] 19.1 Implement AuditLog entity (append-only)
    - Create `AuditLog` entity with fields: `id`, `tenant_id`, `event_type`, `aggregate_id`, `user_id`, `ip_address`, `action`, `payload`, `created_at`
    - Mark as append-only (no Update or Delete methods)
    - _Requirements: 68.1_

  - [ ] 19.2 Create audit_logs database schema
    - Create `audit_logs` table with partitioning by date (5-year retention)
    - Add indexes on `tenant_id`, `aggregate_id`, `user_id`, `event_type`
    - Add index on `created_at DESC` for time-based queries
    - Configure table partitioning: one partition per year
    - _Requirements: 68.1_

  - [ ] 19.3 Implement AuditLogRepository (append-only)
    - Create repository interface with methods: `Append()`, `FindByTenantID()`, `FindByAggregateID()`
    - Explicitly omit `Update()` and `Delete()` methods
    - Implement GORM repository with tenant isolation
    - _Requirements: 68.1_

  - [ ] 19.4 Implement audit logging middleware
    - Create `AuditMiddleware` that logs all API requests
    - Capture: user_id, ip_address, user_agent, request_path, request_method, response_status
    - Log after request completes (defer)
    - Store request/response payloads for sensitive operations
    - _Requirements: 68.2, 68.3_

  - [ ] 19.5 Implement domain event audit handler
    - Create `AuditEventHandler` that listens to all domain events
    - For each event, create audit log entry
    - Store event payload as JSON
    - Ensure audit log is created even if event handler fails
    - _Requirements: 68.4_

  - [ ]* 19.6 Write unit tests for audit logging
    - Test audit log creation for various events
    - Test append-only enforcement
    - Test 5-year retention policy
    - _Requirements: 68.1-68.4_

- [ ] 20. Compliance bounded context - LGPD consent management
  - [ ] 20.1 Implement LGPDConsent entity
    - Create `LGPDConsent` entity with fields: `id`, `user_id`, `consent_type`, `purpose`, `granted`, `granted_at`, `revoked_at`, `terms_version`
    - Define `ConsentType` enum: `data_processing`, `data_sharing`, `marketing`, `assembly_recording`
    - Implement domain methods: `Grant()`, `Revoke()`
    - _Requirements: 69.1, 69.2_

  - [ ] 20.2 Create lgpd_consents database schema
    - Create `lgpd_consents` table with indexes on `user_id` and `consent_type`
    - Add `terms_version` field to track consent version
    - Add `ip_address` field for audit trail
    - _Requirements: 69.1_

  - [ ] 20.3 Implement LGPD consent use cases
    - Create `RecordConsentUseCase` (user grants consent)
    - Create `RevokeConsentUseCase` (user revokes consent)
    - Create `GetUserConsentsUseCase` (retrieve all consents for user)
    - Emit `ConsentGiven` and `ConsentRevoked` events
    - _Requirements: 69.3, 69.4_


  - [ ] 20.4 Implement data export and deletion (LGPD rights)
    - Create `ExportUserDataUseCase` (right to data portability)
    - Create `DeleteUserDataUseCase` (right to be forgotten)
    - Export all user data as JSON
    - Anonymize user data instead of hard delete (preserve audit trail)
    - _Requirements: 69.5, 69.6_

  - [ ]* 20.5 Write unit tests for LGPD compliance
    - Test consent grant and revoke flows
    - Test data export completeness
    - Test data anonymization
    - _Requirements: 69.1-69.6_

- [ ] 21. Compliance bounded context - Declarations and validations
  - [ ] 21.1 Implement Declaration entity
    - Create `Declaration` entity with fields: `id`, `tenant_id`, `declaration_type`, `year`, `status`, `submitted_by`, `content`
    - Define `DeclarationType` enum: `fiscal`, `technical`, `labor`, `conflict_of_interest`
    - Define `DeclarationStatus` enum: `draft`, `submitted`, `approved`, `rejected`
    - _Requirements: 70.1, 70.2_

  - [ ] 21.2 Create declarations database schema
    - Create `declarations` table with indexes on `tenant_id` and `year`
    - Add `content` JSONB field for flexible declaration data
    - Add `attachments` array for supporting documents
    - _Requirements: 70.1_

  - [ ] 21.3 Implement declaration submission use case
    - Create `SubmitDeclarationUseCase`
    - Validate declaration is for current or previous year
    - Validate required fields based on declaration type
    - Store attachments in S3
    - Emit `DeclarationSubmitted` event
    - _Requirements: 70.3, 70.4_

  - [ ]* 21.4 Write unit tests for declarations
    - Test declaration validation rules
    - Test annual submission requirement
    - Test attachment handling
    - _Requirements: 70.1-70.4_

- [ ] 22. Event infrastructure - NATS event bus
  - [ ] 22.1 Implement EventBus interface and NATS adapter
    - Create `EventBus` interface with methods: `Publish()`, `Subscribe()`, `SubscribeAsync()`
    - Implement `NATSEventBus` adapter
    - Configure NATS connection with retry and reconnect logic
    - Implement event serialization/deserialization (JSON)
    - _Requirements: Event-driven architecture_

  - [ ] 22.2 Implement domain event base types
    - Create `BaseDomainEvent` struct with common fields: `event_id`, `event_type`, `aggregate_id`, `occurred_at`
    - Implement concrete event types for all 80+ domain events
    - Define event topics/subjects for NATS
    - _Requirements: Event-driven architecture_

  - [ ] 22.3 Implement event handlers for cross-context communication
    - Create `MasterPlanEventHandler` (listens to `TimelinePublished`)
    - Create `NotificationEventHandler` (listens to all user-facing events)
    - Create `AuditEventHandler` (listens to all events)
    - Create `AnalyticsEventHandler` (listens to metric-relevant events)
    - Register all handlers with event bus
    - _Requirements: Event-driven architecture_

  - [ ] 22.4 Implement event publishing in use cases
    - Add event publishing to all use cases that modify state
    - Ensure events are published after successful database commit
    - Implement transactional outbox pattern for reliability
    - _Requirements: Event-driven architecture_

  - [ ]* 22.5 Write integration tests for event bus
    - Test event publishing and subscription
    - Test event handler execution
    - Test event serialization round-trip
    - _Requirements: Event-driven architecture_

- [ ] 23. Checkpoint - Event-driven architecture validation
  - Verify events are published for all state changes
  - Test cross-context communication via events (Timeline → MasterPlan)
  - Ensure audit logs are created for all events
  - Ask the user if questions arise about event infrastructure

- [ ] 24. External integrations - Storage and media
  - [ ] 24.1 Implement S3 storage adapter
    - Create `StorageService` interface with methods: `Upload()`, `Download()`, `Delete()`, `GetPresignedURL()`
    - Implement `S3StorageAdapter` using AWS SDK
    - Configure bucket structure: `/{tenant_id}/{resource_type}/{file_id}`
    - Implement file type validation and size limits
    - _Requirements: File storage_

  - [ ] 24.2 Implement Mux video streaming adapter
    - Create `VideoService` interface with methods: `UploadVideo()`, `GetStreamURL()`, `DeleteVideo()`
    - Implement `MuxVideoAdapter` using Mux API
    - Store Mux asset IDs in database
    - Generate time-limited playback URLs
    - _Requirements: Video streaming for assembly recordings_

  - [ ]* 24.3 Write integration tests for storage and video
    - Test file upload and download
    - Test video upload and streaming
    - Test presigned URL generation
    - _Requirements: File storage, video streaming_


- [ ] 25. External integrations - Notifications and payments
  - [ ] 25.1 Implement email notification adapter
    - Create `EmailService` interface with methods: `SendEmail()`, `SendBulkEmail()`
    - Implement `SESEmailAdapter` using AWS SES
    - Create email templates for: convocation, ConnectMe notifications, assembly reminders
    - Implement email queuing for bulk sends
    - _Requirements: Email notifications_

  - [ ] 25.2 Implement SMS notification adapter
    - Create `SMSService` interface with methods: `SendSMS()`, `SendBulkSMS()`
    - Implement `TwilioSMSAdapter` or `ZenviaSMSAdapter`
    - Implement SMS rate limiting
    - _Requirements: SMS notifications_

  - [ ] 25.3 Implement push notification adapter
    - Create `PushNotificationService` interface with methods: `SendPush()`, `SendBulkPush()`
    - Implement `FirebasePushAdapter` using Firebase Cloud Messaging
    - Store device tokens in database
    - Implement notification preferences per user
    - _Requirements: Push notifications_

  - [ ] 25.4 Implement payment gateway adapter
    - Create `PaymentService` interface with methods: `CreatePayment()`, `GetPaymentStatus()`, `RefundPayment()`
    - Implement `MercadoPagoAdapter` using Mercado Pago SDK
    - Handle payment webhooks for status updates
    - Store payment transactions in database
    - _Requirements: Payment processing for subscriptions_

  - [ ]* 25.5 Write integration tests for notifications and payments
    - Test email sending
    - Test SMS sending
    - Test push notification delivery
    - Test payment creation and status retrieval
    - _Requirements: Notifications, payments_

- [ ] 26. Search infrastructure - OpenSearch integration
  - [ ] 26.1 Implement OpenSearch adapter
    - Create `SearchService` interface with methods: `Index()`, `Search()`, `Delete()`
    - Implement `OpenSearchAdapter` using OpenSearch Go client
    - Configure index mappings for: condominiums, timeline, proposals, assemblies
    - Implement tenant-aware search (filter by tenant_id)
    - _Requirements: Full-text search_

  - [ ] 26.2 Implement search indexing event handlers
    - Create `SearchIndexHandler` that listens to domain events
    - Index documents when created/updated: `CondominiumCreated`, `TimelinePublished`, `ProposalSent`
    - Remove documents when deleted
    - Implement bulk indexing for initial data load
    - _Requirements: Full-text search_

  - [ ] 26.3 Implement search query use cases
    - Create `SearchCondominiumsUseCase`
    - Create `SearchTimelineUseCase`
    - Create `SearchProposalsUseCase`
    - Implement faceted search (filters by type, date, status)
    - Implement pagination and sorting
    - _Requirements: Full-text search_

  - [ ]* 26.4 Write integration tests for search
    - Test document indexing
    - Test search queries with filters
    - Test tenant isolation in search results
    - _Requirements: Full-text search_

- [ ] 27. Caching layer - Redis integration
  - [ ] 27.1 Implement Redis cache adapter
    - Create `CacheService` interface with methods: `Get()`, `Set()`, `Delete()`, `Exists()`
    - Implement `RedisCacheAdapter` using go-redis client
    - Configure cache TTLs per data type
    - Implement cache key namespacing by tenant
    - _Requirements: Performance optimization_

  - [ ] 27.2 Implement caching in repositories
    - Add caching to frequently accessed data: user profiles, subscriptions, condominium details
    - Implement cache-aside pattern (read-through cache)
    - Invalidate cache on updates
    - Use cache for session storage
    - _Requirements: Performance optimization_

  - [ ] 27.3 Implement distributed locking for critical operations
    - Use Redis for distributed locks
    - Implement locking for: vote casting, payment processing, quota checks
    - Set lock TTLs to prevent deadlocks
    - _Requirements: Concurrency control_

  - [ ]* 27.4 Write integration tests for caching
    - Test cache hit/miss scenarios
    - Test cache invalidation
    - Test distributed locking
    - _Requirements: Performance optimization_

- [ ] 28. Checkpoint - Infrastructure integrations validation
  - Verify all external services are properly integrated
  - Test error handling and retry logic for external services
  - Verify caching improves performance
  - Ask the user if questions arise about integrations


- [ ] 29. REST API layer - HTTP handlers and routing
  - [ ] 29.1 Setup Gin HTTP framework and middleware stack
    - Initialize Gin router with custom configuration
    - Register middleware in order: CORS → Logger → Recovery → Auth → Tenant → Authorization → Audit
    - Configure request timeout (30s default)
    - Configure request size limits
    - _Requirements: API foundation_

  - [ ] 29.2 Implement authentication endpoints
    - POST `/api/auth/register` - User registration
    - POST `/api/auth/login` - User login (returns JWT)
    - POST `/api/auth/refresh` - Refresh JWT token
    - POST `/api/auth/logout` - Logout (invalidate session)
    - GET `/api/auth/me` - Get current user profile
    - _Requirements: 1.1-1.11_

  - [ ] 29.3 Implement condominium management endpoints
    - POST `/api/condominiums` - Create condominium
    - GET `/api/condominiums/:id` - Get condominium details
    - GET `/api/condominiums` - List condominiums (for sindico)
    - PUT `/api/condominiums/:id` - Update condominium
    - POST `/api/condominiums/:id/units` - Register unit
    - POST `/api/condominiums/:id/units/:unit_id/moradores` - Link morador to unit
    - _Requirements: 4.1-5.4_

  - [ ] 29.4 Implement timeline endpoints
    - POST `/api/timeline` - Publish timeline entry (sindico only)
    - GET `/api/timeline` - Get timeline (with filters and pagination)
    - GET `/api/timeline/:id` - Get timeline entry details
    - POST `/api/timeline/:id/adendos` - Create adendo
    - _Requirements: 11.1-11.12_

  - [ ] 29.5 Implement master plan endpoints
    - POST `/api/master-plan` - Create master plan action
    - GET `/api/master-plan` - List master plan actions (with filters)
    - GET `/api/master-plan/:id` - Get action details
    - PUT `/api/master-plan/:id` - Update action
    - GET `/api/master-plan/overdue` - Get overdue actions
    - _Requirements: 12.1-12.7_

  - [ ] 29.6 Implement ConnectMe endpoints
    - POST `/api/connect-me` - Create ConnectMe request
    - GET `/api/connect-me` - List requests (source or target tenant)
    - GET `/api/connect-me/:id` - Get request details
    - POST `/api/connect-me/:id/interest` - Show interest (company)
    - POST `/api/connect-me/:id/reveal` - Reveal contact data
    - _Requirements: 19.1-19.12_

  - [ ] 29.7 Implement proposal and deal endpoints
    - POST `/api/proposals` - Create proposal (company)
    - GET `/api/proposals` - List proposals
    - GET `/api/proposals/:id` - Get proposal details
    - POST `/api/proposals/:id/validate` - Validate proposal (sindico)
    - POST `/api/deals` - Create closed deal
    - POST `/api/deals/:id/sign` - Sign deal
    - POST `/api/deals/:id/executions` - Register service execution
    - POST `/api/deals/:id/executions/:exec_id/validate` - Validate execution
    - _Requirements: 20.1-21.9_

  - [ ] 29.8 Implement assembly endpoints
    - POST `/api/assemblies` - Create assembly
    - GET `/api/assemblies` - List assemblies
    - GET `/api/assemblies/:id` - Get assembly details
    - POST `/api/assemblies/:id/start` - Start assembly
    - POST `/api/assemblies/:id/check-in` - Check-in morador
    - POST `/api/assemblies/:id/proxies` - Register proxy
    - POST `/api/assemblies/:id/agenda/:item_id/votes` - Cast vote
    - POST `/api/assemblies/:id/agenda/:item_id/close` - Close voting
    - POST `/api/assemblies/:id/agenda/:item_id/homologate` - Homologate voting
    - POST `/api/assemblies/:id/finalize` - Finalize assembly
    - _Requirements: 48.1-58.5_

  - [ ] 29.9 Implement compliance endpoints
    - GET `/api/audit-logs` - Get audit logs (with filters)
    - GET `/api/audit-logs/:aggregate_id` - Get audit trail for aggregate
    - POST `/api/consents` - Record LGPD consent
    - DELETE `/api/consents/:id` - Revoke consent
    - GET `/api/consents` - Get user consents
    - GET `/api/export/user-data` - Export user data (LGPD)
    - DELETE `/api/user-data` - Delete user data (LGPD)
    - POST `/api/declarations` - Submit declaration
    - GET `/api/declarations` - List declarations
    - _Requirements: 68.1-70.4_

  - [ ] 29.10 Implement search endpoints
    - GET `/api/search/condominiums` - Search condominiums
    - GET `/api/search/timeline` - Search timeline entries
    - GET `/api/search/proposals` - Search proposals
    - _Requirements: Full-text search_

  - [ ]* 29.11 Write integration tests for API endpoints
    - Test authentication flows
    - Test authorization (403 for unauthorized access)
    - Test tenant isolation (users can't access other tenants' data)
    - Test CRUD operations for all resources
    - _Requirements: All API endpoints_


- [ ] 30. Data encryption and security hardening
  - [ ] 30.1 Implement encryption service
    - Create `EncryptionService` interface with methods: `Encrypt()`, `Decrypt()`
    - Implement AES-256-GCM encryption
    - Store encryption keys in environment variables or secrets manager
    - Implement key rotation mechanism
    - _Requirements: 71.1_

  - [ ] 30.2 Implement field-level encryption for sensitive data
    - Encrypt password hashes (additional layer beyond bcrypt)
    - Encrypt CPF/CNPJ fields
    - Encrypt financial data (bank accounts, payment info)
    - Implement transparent encryption/decryption in repositories
    - _Requirements: 71.1, 71.2_

  - [ ]* 30.3 Write property tests for encryption
    - **Property 27: Sensitive Field Encryption**
    - **Validates: Requirements 71.1**
    - **Property 28: Decryption Authorization**
    - **Validates: Requirements 71.4**
    - **Property 29: Decryption Audit Trail**
    - **Validates: Requirements 71.5**

  - [ ] 30.4 Implement security headers middleware
    - Add security headers: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, Strict-Transport-Security
    - Implement CORS with whitelist
    - Add Content-Security-Policy header
    - _Requirements: Security best practices_

  - [ ] 30.5 Implement input validation and sanitization
    - Create validation middleware using go-playground/validator
    - Sanitize all string inputs (prevent XSS)
    - Validate all numeric inputs (prevent overflow)
    - Validate all date inputs (prevent invalid dates)
    - _Requirements: Security best practices_

  - [ ]* 30.6 Write security tests
    - Test XSS prevention
    - Test SQL injection prevention (GORM should handle this)
    - Test CSRF protection
    - Test rate limiting
    - _Requirements: Security best practices_

- [ ] 31. Background jobs and scheduled tasks
  - [ ] 31.1 Setup background job infrastructure
    - Choose job queue library (e.g., asynq, machinery)
    - Configure Redis as job queue backend
    - Implement job worker pool
    - Implement job retry logic with exponential backoff
    - _Requirements: Background processing_

  - [ ] 31.2 Implement scheduled jobs
    - Create `ExpireTrialSubscriptionsJob` (runs daily)
    - Create `ExpireConnectMeRequestsJob` (runs hourly)
    - Create `UpdateOverdueMasterPlanActionsJob` (runs daily)
    - Create `SendAssemblyRemindersJob` (runs daily)
    - Create `CleanupExpiredSessionsJob` (runs hourly)
    - _Requirements: 3.6, 19.11, 12.7_

  - [ ] 31.3 Implement async event processing jobs
    - Create `SendNotificationJob` (triggered by events)
    - Create `IndexSearchDocumentJob` (triggered by events)
    - Create `GenerateReportJob` (triggered by user request)
    - _Requirements: Async processing_

  - [ ]* 31.4 Write tests for background jobs
    - Test job execution
    - Test job retry on failure
    - Test scheduled job timing
    - _Requirements: Background processing_

- [ ] 32. Monitoring, logging, and observability
  - [ ] 32.1 Implement structured logging
    - Use zerolog or zap for structured logging
    - Log all errors with context (user_id, tenant_id, request_id)
    - Log all slow queries (>100ms)
    - Log all external service calls
    - Configure log levels per environment (debug, info, warn, error)
    - _Requirements: Observability_

  - [ ] 32.2 Implement metrics collection
    - Use Prometheus client library
    - Expose `/metrics` endpoint
    - Collect metrics: request count, request duration, error rate, database query duration
    - Collect business metrics: active users, assemblies created, votes cast
    - _Requirements: Observability_

  - [ ] 32.3 Implement distributed tracing
    - Use OpenTelemetry for tracing
    - Trace all HTTP requests
    - Trace all database queries
    - Trace all external service calls
    - Export traces to Jaeger or Zipkin
    - _Requirements: Observability_

  - [ ] 32.4 Implement health check endpoints
    - GET `/health` - Basic health check (returns 200 OK)
    - GET `/health/ready` - Readiness check (checks database, Redis, NATS)
    - GET `/health/live` - Liveness check (checks if app is running)
    - _Requirements: Kubernetes deployment_

  - [ ]* 32.5 Write tests for observability
    - Test metrics are collected
    - Test traces are created
    - Test health check endpoints
    - _Requirements: Observability_


- [ ] 33. Database migrations and seeding
  - [ ] 33.1 Setup migration framework
    - Use golang-migrate or GORM AutoMigrate
    - Create migration files for all tables
    - Implement up and down migrations
    - Add migration versioning
    - _Requirements: Database management_

  - [ ] 33.2 Create seed data for development
    - Create seed script for test tenants
    - Create seed script for test users (sindico, morador, empresa)
    - Create seed script for test condominiums
    - Create seed script for test timeline entries
    - _Requirements: Development environment_

  - [ ] 33.3 Implement database backup strategy
    - Configure automated PostgreSQL backups
    - Implement point-in-time recovery
    - Test backup restoration process
    - _Requirements: Data protection_

- [ ] 34. Dependency injection with Wire
  - [ ] 34.1 Setup Wire for dependency injection
    - Install Wire code generator
    - Create provider functions for all services
    - Create provider functions for all repositories
    - Create provider functions for all adapters
    - _Requirements: Clean Architecture_

  - [ ] 34.2 Generate Wire dependency graph
    - Create `wire.go` files for each module
    - Run Wire code generator
    - Verify generated code compiles
    - _Requirements: Clean Architecture_

  - [ ] 34.3 Refactor main.go to use Wire
    - Replace manual dependency construction with Wire
    - Initialize all dependencies via Wire
    - Verify application starts correctly
    - _Requirements: Clean Architecture_

- [ ] 35. Configuration management
  - [ ] 35.1 Implement configuration loading
    - Use viper for configuration management
    - Support multiple config sources: env vars, config files, command-line flags
    - Define configuration struct with all settings
    - Validate configuration on startup
    - _Requirements: Configuration management_

  - [ ] 35.2 Create environment-specific configs
    - Create `config.dev.yaml` for development
    - Create `config.staging.yaml` for staging
    - Create `config.prod.yaml` for production
    - Document all configuration options
    - _Requirements: Configuration management_

  - [ ] 35.3 Implement secrets management
    - Use AWS Secrets Manager or HashiCorp Vault
    - Load secrets at startup
    - Rotate secrets periodically
    - Never commit secrets to version control
    - _Requirements: Security_

- [ ] 36. Error handling and circuit breakers
  - [ ] 36.1 Implement circuit breaker for external services
    - Use gobreaker library
    - Configure circuit breakers for: Zitadel, Mercado Pago, Mux, SES, Twilio
    - Set failure thresholds and timeout durations
    - Implement fallback behavior when circuit is open
    - _Requirements: Resilience_

  - [ ] 36.2 Implement retry logic with exponential backoff
    - Create `RetryWithBackoff` utility function
    - Apply to all external service calls
    - Configure max attempts and backoff multiplier
    - Skip retry for non-retriable errors (4xx)
    - _Requirements: Resilience_

  - [ ] 36.3 Implement graceful degradation
    - When external service fails, return cached data if available
    - When search fails, fall back to database query
    - When notification fails, queue for retry
    - _Requirements: Resilience_

  - [ ]* 36.4 Write tests for error handling
    - Test circuit breaker opens after failures
    - Test retry logic with exponential backoff
    - Test graceful degradation
    - _Requirements: Resilience_

- [ ] 37. Checkpoint - Infrastructure and resilience validation
  - Verify all background jobs run on schedule
  - Test circuit breakers open and close correctly
  - Verify metrics and traces are collected
  - Test graceful degradation when external services fail
  - Ask the user if questions arise about infrastructure


- [ ] 38. Additional bounded contexts - Neighborhood and LMS
  - [ ] 38.1 Implement Neighborhood bounded context
    - Create `Partner` aggregate with fields: `id`, `tenant_id`, `name`, `cnpj`, `partner_type`, `service_area`
    - Create `Promotion` entity with fields: `partner_id`, `title`, `description`, `discount`, `valid_until`
    - Create `Benefit` entity with fields: `partner_id`, `benefit_type`, `description`
    - Implement use cases: `RegisterPartnerUseCase`, `CreatePromotionUseCase`, `ActivateBenefitUseCase`
    - _Requirements: Neighborhood context_

  - [ ] 38.2 Create neighborhood database schemas
    - Create `partners` table with tenant_id
    - Create `promotions` table with foreign key to partners
    - Create `benefits` table with foreign key to partners
    - Create `benefit_activations` table (many-to-many: moradores and benefits)
    - _Requirements: Neighborhood context_

  - [ ] 38.3 Implement LMS (Learning Management System) bounded context
    - Create `Course` aggregate with fields: `id`, `title`, `description`, `content_type`, `duration`
    - Create `Enrollment` entity with fields: `user_id`, `course_id`, `progress`, `completed_at`
    - Create `Certificate` entity with fields: `user_id`, `course_id`, `issued_at`, `certificate_url`
    - Implement use cases: `CreateCourseUseCase`, `EnrollUserUseCase`, `TrackProgressUseCase`, `IssueCertificateUseCase`
    - _Requirements: LMS context_

  - [ ] 38.4 Create LMS database schemas
    - Create `courses` table
    - Create `enrollments` table with foreign keys to users and courses
    - Create `certificates` table
    - Create `course_modules` and `course_lessons` tables
    - _Requirements: LMS context_

  - [ ]* 38.5 Write unit tests for Neighborhood and LMS
    - Test partner registration
    - Test promotion creation and expiration
    - Test course enrollment and progress tracking
    - _Requirements: Neighborhood and LMS contexts_

- [ ] 39. Additional bounded contexts - Billing and Analytics
  - [ ] 39.1 Implement Billing bounded context
    - Create `Invoice` aggregate with fields: `id`, `tenant_id`, `subscription_id`, `amount`, `due_date`, `status`
    - Create `Payment` entity with fields: `invoice_id`, `payment_method`, `paid_at`, `transaction_id`
    - Create `BillingHistory` entity for audit trail
    - Implement use cases: `GenerateInvoiceUseCase`, `ProcessPaymentUseCase`, `SendPaymentReminderUseCase`
    - _Requirements: Billing context_

  - [ ] 39.2 Create billing database schemas
    - Create `invoices` table with tenant_id
    - Create `payments` table with foreign key to invoices
    - Create `billing_history` table (append-only)
    - _Requirements: Billing context_

  - [ ] 39.3 Implement Analytics bounded context
    - Create `Metric` aggregate with fields: `id`, `tenant_id`, `metric_type`, `value`, `timestamp`
    - Create `Report` entity with fields: `id`, `tenant_id`, `report_type`, `generated_at`, `data`
    - Create `Dashboard` entity with fields: `id`, `tenant_id`, `widgets`, `layout`
    - Implement use cases: `CalculateMetricUseCase`, `GenerateReportUseCase`, `UpdateDashboardUseCase`
    - _Requirements: Analytics context_

  - [ ] 39.4 Create analytics database schemas
    - Create `metrics` table with time-series optimization
    - Create `reports` table
    - Create `dashboards` table with JSONB for widget configuration
    - _Requirements: Analytics context_

  - [ ] 39.5 Implement analytics event handlers
    - Create `MetricsEventHandler` that listens to all domain events
    - Calculate metrics: active users, assemblies per month, votes cast, ConnectMe requests
    - Store metrics in time-series format
    - _Requirements: Analytics context_

  - [ ]* 39.6 Write unit tests for Billing and Analytics
    - Test invoice generation
    - Test payment processing
    - Test metric calculation
    - Test report generation
    - _Requirements: Billing and Analytics contexts_

- [ ] 40. WebSocket support for real-time assembly
  - [ ] 40.1 Implement WebSocket server
    - Setup WebSocket handler using gorilla/websocket
    - Implement connection management (connect, disconnect, heartbeat)
    - Implement room-based broadcasting (one room per assembly)
    - Authenticate WebSocket connections via JWT
    - _Requirements: Real-time assembly_

  - [ ] 40.2 Implement real-time assembly events
    - Broadcast when morador checks in
    - Broadcast when vote is cast
    - Broadcast when voting is closed
    - Broadcast when quorum status changes
    - Broadcast when agenda item is opened/closed
    - _Requirements: Real-time assembly_

  - [ ] 40.3 Implement telão (big screen) support
    - Create dedicated WebSocket endpoint for telão
    - Broadcast assembly state: current agenda item, vote counts, quorum status
    - Implement speech queue display
    - Implement real-time vote visualization
    - _Requirements: 56.1-56.5_

  - [ ]* 40.4 Write integration tests for WebSocket
    - Test connection and disconnection
    - Test room-based broadcasting
    - Test authentication
    - Test message delivery
    - _Requirements: Real-time assembly_


- [ ] 41. Performance optimization
  - [ ] 41.1 Implement database query optimization
    - Add missing indexes based on query patterns
    - Optimize N+1 queries using GORM preloading
    - Implement database connection pooling
    - Add query timeout configuration
    - _Requirements: Performance_

  - [ ] 41.2 Implement API response caching
    - Cache frequently accessed endpoints (timeline, condominium details)
    - Implement cache invalidation on updates
    - Use ETags for conditional requests
    - Implement HTTP cache headers
    - _Requirements: Performance_

  - [ ] 41.3 Implement pagination for all list endpoints
    - Add cursor-based pagination for timeline
    - Add offset-based pagination for other resources
    - Limit page size to 50 items max
    - Return pagination metadata (total, page, per_page)
    - _Requirements: Performance_

  - [ ] 41.4 Implement database read replicas
    - Configure PostgreSQL read replicas
    - Route read queries to replicas
    - Route write queries to primary
    - Implement replica lag monitoring
    - _Requirements: Scalability_

  - [ ]* 41.5 Write performance tests
    - Test API response times under load
    - Test database query performance
    - Test cache hit rates
    - _Requirements: Performance_

- [ ] 42. API documentation
  - [ ] 42.1 Generate OpenAPI/Swagger documentation
    - Use swaggo/swag to generate OpenAPI spec from code comments
    - Document all endpoints with request/response schemas
    - Document authentication requirements
    - Document error responses
    - _Requirements: API documentation_

  - [ ] 42.2 Setup Swagger UI
    - Serve Swagger UI at `/api/docs`
    - Enable "Try it out" functionality
    - Add API examples for common use cases
    - _Requirements: API documentation_

  - [ ] 42.3 Write API usage guide
    - Document authentication flow
    - Document common workflows (create condominium, publish timeline, create assembly)
    - Document rate limits and quotas
    - Document error codes and meanings
    - _Requirements: API documentation_

- [ ] 43. Deployment preparation
  - [ ] 43.1 Create Dockerfile
    - Use multi-stage build for smaller image
    - Use Alpine Linux as base image
    - Copy only necessary files
    - Set non-root user
    - _Requirements: Containerization_

  - [ ] 43.2 Create docker-compose.yml for local development
    - Define services: api, postgres, redis, nats, opensearch
    - Configure service dependencies
    - Mount volumes for data persistence
    - Expose necessary ports
    - _Requirements: Local development_

  - [ ] 43.3 Create Kubernetes manifests
    - Create Deployment for API servers
    - Create Service for load balancing
    - Create ConfigMap for configuration
    - Create Secret for sensitive data
    - Create Ingress for external access
    - Create HorizontalPodAutoscaler for auto-scaling
    - _Requirements: Kubernetes deployment_

  - [ ] 43.4 Create CI/CD pipeline
    - Setup GitHub Actions or GitLab CI
    - Run tests on every commit
    - Build Docker image on merge to main
    - Push image to container registry
    - Deploy to staging automatically
    - Deploy to production manually
    - _Requirements: CI/CD_

  - [ ]* 43.5 Write deployment tests
    - Test Docker image builds successfully
    - Test Kubernetes manifests are valid
    - Test health checks work in deployed environment
    - _Requirements: Deployment_

- [ ] 44. Final checkpoint - End-to-end validation
  - Run complete test suite (unit, integration, property, e2e)
  - Verify all 29 correctness properties pass
  - Test complete user journeys: sindico creates condominium → publishes timeline → creates assembly → moradores vote → assembly finalized
  - Load test with 1000+ concurrent users
  - Security audit and penetration testing
  - Ask the user if questions arise before production deployment


## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP delivery
- Each task references specific requirements for traceability
- Property tests validate universal correctness properties from the design document
- Unit tests validate specific examples and edge cases
- Checkpoints ensure incremental validation at key milestones
- The implementation follows Clean Architecture with strict layer separation
- Multi-tenancy is enforced at every layer (middleware, repository, policy)
- Event-driven architecture enables loose coupling between bounded contexts
- All sensitive data is encrypted at rest and in transit
- LGPD compliance is built into the core architecture
- The system is designed to scale horizontally with read replicas and caching

## Implementation Priority

1. **Core Request Flow** (Tasks 1-5): Foundation for all features
2. **Identity & Institutional** (Tasks 6-8): User and condominium management
3. **Content Contexts** (Tasks 9-11): Timeline and Master Plan
4. **Commercial Context** (Tasks 12-13): ConnectMe and proposals
5. **Assembly Module** (Tasks 14-18): Complete assembly workflow
6. **Compliance** (Tasks 19-21): Audit trail and LGPD
7. **Infrastructure** (Tasks 22-28): Events, integrations, caching
8. **API Layer** (Tasks 29-30): REST endpoints and security
9. **Background Jobs** (Tasks 31-32): Scheduled tasks and observability
10. **Additional Contexts** (Tasks 38-39): Neighborhood, LMS, Billing, Analytics
11. **Real-time** (Task 40): WebSocket for assembly
12. **Optimization** (Tasks 41-42): Performance and documentation
13. **Deployment** (Tasks 43-44): Containerization and CI/CD

## Testing Strategy

- **Unit Tests**: 80%+ code coverage, focus on domain logic
- **Property Tests**: All 29 correctness properties must pass
- **Integration Tests**: All repositories, all external adapters
- **E2E Tests**: Critical user journeys (sindico, morador, company)
- **Load Tests**: 1000+ concurrent users for assembly module
- **Security Tests**: XSS, SQL injection, CSRF, rate limiting

## Success Criteria

- All 104 requirements implemented and tested
- All 29 correctness properties pass
- API response time < 200ms (p95)
- Database query time < 100ms (p95)
- 99.9% uptime SLA
- Zero data leaks between tenants
- LGPD compliance verified
- Security audit passed
- Load test passed (1000+ concurrent users)
