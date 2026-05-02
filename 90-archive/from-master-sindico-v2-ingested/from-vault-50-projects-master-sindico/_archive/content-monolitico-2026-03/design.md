# Design Document — Master Síndico Platform

## Overview

Master Síndico é uma plataforma complexa de governança condominial multi-tenant que implementa Clean Architecture, Domain-Driven Design (DDD) e princípios SOLID. A plataforma opera em Go, migrando de uma arquitetura TypeScript/Node.js, com foco em escalabilidade, isolamento de dados e compliance LGPD.

### Visão Geral do Sistema

A plataforma serve múltiplos tipos de usuários (síndicos, moradores, empresas, agências, parceiros) através de uma arquitetura event-driven com 8 bounded contexts principais. O sistema implementa multi-tenancy estrito com isolamento por tenant_id em todas as operações de banco de dados.

### Objetivos Principais

- **Governança Condominial**: Timeline imutável, Plano Diretor reativo, validações centralizadas
- **Marketplace Unidirecional**: Connect Me com LGPD by design
- **Compliance Robusto**: Audit trail imutável, retenção de 5 anos, declarações anuais
- **Assembleias Digitais**: Módulo presencial para MVP, preparado para híbrido/online
- **Multi-tenancy Seguro**: Isolamento estrito de dados por tenant
- **Event-Driven**: Comunicação assíncrona entre domínios via NATS

### Princípios Arquiteturais

1. **Clean Architecture**: Separação clara entre Domain, Application, Infrastructure e Presentation
2. **Domain-Driven Design**: Bounded contexts bem definidos, agregados, entidades e value objects
3. **SOLID**: Princípios aplicados em todos os níveis
4. **Multi-tenancy First**: tenant_id em todas as queries, middleware de injeção
5. **Event-Driven**: 80+ domain events para comunicação cross-domain
6. **LGPD Compliance**: Audit logs imutáveis, consentimento explícito, rastreabilidade completa

## Architecture

### High-Level Architecture


```
┌─────────────────────────────────────────────────────────────────┐
│                     PRESENTATION LAYER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  REST API    │  │  WebSocket   │  │  GraphQL     │          │
│  │  (Gin)       │  │  (Assembly)  │  │  (Future)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     APPLICATION LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Use Cases   │  │  DTOs        │  │  Mappers     │          │
│  │  Commands    │  │  Validators  │  │  Services    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                       DOMAIN LAYER                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  8 BOUNDED CONTEXTS                                        │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │ │
│  │  │ Identity │ │Instituti │ │Commercial│ │ Content  │    │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │ │
│  │  │ Assembly │ │Compliance│ │Analytics │ │Neighborhood│  │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │ │
│  │                                                            │ │
│  │  Aggregates, Entities, Value Objects, Domain Events      │ │
│  │  Repository Interfaces, Domain Services                   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   INFRASTRUCTURE LAYER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  PostgreSQL  │  │  Redis       │  │  NATS        │          │
│  │  (GORM)      │  │  (Cache)     │  │  (Events)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Zitadel     │  │  OpenSearch  │  │  S3/Mux      │          │
│  │  (Auth)      │  │  (Search)    │  │  (Storage)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### Bounded Contexts

#### 1. Identity Context
**Responsabilidade**: Autenticação, autorização, gestão de usuários e planos

**Agregados**:
- User (root)
- Subscription
- Session
- DeviceFingerprint

**Domain Events**:
- UserRegistered
- UserAuthenticated
- SubscriptionCreated
- SubscriptionExpired
- TrialExpired
- SessionInvalidated

#### 2. Institutional Context
**Responsabilidade**: Gestão de condomínios, síndicos, moradores, empresas

**Agregados**:
- Condominium (root)
- Mandate
- Unit
- SindicoProfile
- CompanyProfile
- PartnerProfile

**Domain Events**:
- CondominiumCreated
- MandateStarted
- MandateEnded
- UnitRegistered
- ProfileCreated
- ProfileUpdated

#### 3. Commercial Context
**Responsabilidade**: Connect Me, propostas, negócios fechados, avaliações

**Agregados**:
- ConnectMeRequest (root)
- Proposal
- ClosedDeal
- Evaluation

**Domain Events**:
- ConnectMeRequestCreated
- ConnectMeInterestShown
- ProposalSent
- ProposalValidated
- DealConfirmed
- EvaluationSubmitted

#### 4. Content Context
**Responsabilidade**: Timeline, Plano Diretor, eventos, comunicados

**Agregados**:
- TimelineEntry (root)
- MasterPlanAction
- Event
- Comunicado
- Adendo

**Domain Events**:
- TimelinePublished
- MasterPlanActionCreated
- MasterPlanStatusUpdated
- EventCreated
- ComunicadoPublished
- AdendoCreated


#### 5. Assembly Context
**Responsabilidade**: Assembleias presenciais, votações, quórum, atas

**Agregados**:
- Assembly (root)
- AgendaItem
- Vote
- Proxy
- AttendanceRecord

**Domain Events**:
- AssemblyCreated
- AssemblyStarted
- MoradorCheckedIn
- VoteCast
- VoteHomologated
- AssemblyFinalized

#### 6. Compliance Context
**Responsabilidade**: Audit trail, declarações, LGPD, validações

**Agregados**:
- AuditLog (root)
- Declaration
- LGPDConsent
- ValidationRequest

**Domain Events**:
- AuditLogCreated
- DeclarationSubmitted
- ConsentGiven
- ConsentRevoked
- ValidationRequested
- ValidationCompleted

#### 7. Analytics Context
**Responsabilidade**: Métricas, dashboards, relatórios

**Agregados**:
- Metric (root)
- Report
- Dashboard

**Domain Events**:
- MetricCalculated
- ReportGenerated
- DashboardUpdated

#### 8. Neighborhood Context
**Responsabilidade**: Parceiros da vizinhança, promoções, benefícios

**Agregados**:
- Partner (root)
- Promotion
- Benefit
- Activation

**Domain Events**:
- PartnerRegistered
- PromotionCreated
- BenefitActivated
- PartnerInvited

### Layer Responsibilities

#### Domain Layer
- **Entities**: Objetos com identidade única (User, Condominium, Assembly)
- **Value Objects**: Objetos imutáveis sem identidade (Address, Money, DateRange)
- **Aggregates**: Clusters de entidades tratadas como unidade (Condominium + Units + Mandate)
- **Domain Events**: Eventos que representam mudanças de estado
- **Repository Interfaces**: Contratos para persistência
- **Domain Services**: Lógica de negócio que não pertence a uma entidade específica

#### Application Layer
- **Use Cases**: Orquestração de lógica de negócio (CreateCondominium, PublishTimeline)
- **Commands**: Intenções de mudança de estado (CreateUserCommand)
- **Queries**: Leitura de dados (GetCondominiumQuery)
- **DTOs**: Objetos de transferência de dados
- **Mappers**: Conversão entre DTOs e entidades de domínio
- **Application Services**: Coordenação de use cases

#### Infrastructure Layer
- **Repository Implementations**: Persistência com GORM
- **External Service Adapters**: Anti-corruption layers para serviços externos
- **Event Bus**: Publicação e subscrição de eventos via NATS
- **Cache**: Redis para dados frequentemente acessados
- **Search**: OpenSearch para busca full-text

#### Presentation Layer
- **REST Controllers**: Endpoints HTTP com Gin
- **WebSocket Handlers**: Comunicação real-time para Assembly
- **Middleware**: Autenticação, autorização, multi-tenancy, rate limiting
- **Request Validators**: Validação de entrada com Zod-like library

### Multi-Tenancy Strategy

#### Tenant Isolation

```go
// Middleware injeta tenant_id no contexto
func TenantMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        userID := c.GetString("user_id")
        tenantID := getTenantIDFromUser(userID)
        
        c.Set("tenant_id", tenantID)
        c.Next()
    }
}

// Repository sempre filtra por tenant_id
func (r *CondominiumRepository) FindByID(ctx context.Context, id string) (*Condominium, error) {
    tenantID := ctx.Value("tenant_id").(string)
    
    var condo Condominium
    err := r.db.Where("id = ? AND tenant_id = ?", id, tenantID).First(&condo).Error
    return &condo, err
}
```

#### Tenant Types

```go
type TenantType string

const (
    TenantTypeCondominium TenantType = "condominium"
    TenantTypeCompany     TenantType = "company"
)

type Tenant struct {
    ID        string
    Type      TenantType
    Name      string
    Status    TenantStatus
    CreatedAt time.Time
}
```

#### Cross-Tenant Operations

Para operações que cruzam tenants (Connect Me, validações), usamos grants explícitos:

```go
type CrossTenantGrant struct {
    ID              string
    SourceTenantID  string
    TargetTenantID  string
    GrantType       GrantType // connect_me, validation, etc
    ExpiresAt       time.Time
    ConsentGiven    bool
    ConsentAt       *time.Time
}
```


### Event-Driven Architecture

#### Event Bus (NATS)

```go
type EventBus interface {
    Publish(ctx context.Context, event DomainEvent) error
    Subscribe(topic string, handler EventHandler) error
    SubscribeAsync(topic string, handler EventHandler) error
}

type DomainEvent interface {
    EventID() string
    EventType() string
    AggregateID() string
    OccurredAt() time.Time
    Payload() interface{}
}
```

#### Event Flow Example

```
1. Síndico publica Timeline Entry
   ↓
2. TimelinePublished event emitido
   ↓
3. Event Bus (NATS) distribui para subscribers
   ↓
4. MasterPlanService escuta e atualiza status de ação relacionada
   ↓
5. NotificationService escuta e envia notificações para moradores
   ↓
6. AuditService escuta e registra no audit log
   ↓
7. AnalyticsService escuta e atualiza métricas
```

#### Event Handlers

```go
// Domain event
type TimelinePublished struct {
    EventID       string
    TimelineID    string
    TenantID      string
    EntryType     string
    AuthorID      string
    RelatedAction *string // MasterPlan action ID
    OccurredAt    time.Time
}

// Event handler
type MasterPlanEventHandler struct {
    masterPlanService *MasterPlanService
}

func (h *MasterPlanEventHandler) Handle(ctx context.Context, event DomainEvent) error {
    timelineEvent := event.(*TimelinePublished)
    
    if timelineEvent.RelatedAction != nil {
        return h.masterPlanService.UpdateActionStatus(
            ctx,
            *timelineEvent.RelatedAction,
            StatusCompleted,
        )
    }
    
    return nil
}
```

#### Event Sourcing (Audit Trail)

Para compliance LGPD, mantemos event sourcing parcial no audit trail:

```go
type AuditLog struct {
    ID          string
    TenantID    string
    EventType   string
    AggregateID string
    UserID      string
    IPAddress   string
    UserAgent   string
    Payload     json.RawMessage
    CreatedAt   time.Time
}

// Append-only, nunca deletado ou modificado
func (r *AuditLogRepository) Append(ctx context.Context, log *AuditLog) error {
    return r.db.Create(log).Error
}
```

## Components and Interfaces

### Core Domain Models

#### Identity Context

```go
// User aggregate root
type User struct {
    ID                string
    Email             string
    PasswordHash      string
    Name              string
    Phone             string
    UserType          UserType
    Status            UserStatus
    EmailVerified     bool
    EmailVerifiedAt   *time.Time
    DeviceFingerprint *DeviceFingerprint
    CreatedAt         time.Time
    UpdatedAt         time.Time
}

type UserType string

const (
    UserTypeSindico           UserType = "sindico"
    UserTypeMorador           UserType = "morador"
    UserTypeEmpresa           UserType = "empresa"
    UserTypeAgencia           UserType = "agencia"
    UserTypeParceiro          UserType = "parceiro"
)

type DeviceFingerprint struct {
    DeviceID      string
    IPAddress     string
    UserAgent     string
    LastSeenAt    time.Time
}

// Subscription aggregate
type Subscription struct {
    ID              string
    UserID          string
    TenantID        string
    Plan            Plan
    Status          SubscriptionStatus
    StartDate       time.Time
    EndDate         *time.Time
    TrialEndDate    *time.Time
    PaymentMethod   *PaymentMethod
    BillingHistory  []BillingRecord
}

type Plan string

const (
    PlanTrial       Plan = "trial"
    PlanBase        Plan = "base"
    PlanPlus        Plan = "plus"
    PlanPro         Plan = "pro"
)
```

#### Institutional Context

```go
// Condominium aggregate root
type Condominium struct {
    ID              string
    TenantID        string
    Name            string
    Address         Address
    CondominiumType CondominiumType
    TotalUnits      int
    BlocksOrTowers  int
    AlphanumericID  string // 8 caracteres
    QRCode          string
    Status          CondominiumStatus
    CurrentMandate  *Mandate
    Units           []Unit
    CreatedAt       time.Time
    UpdatedAt       time.Time
}

type Address struct {
    Street       string
    Number       string
    Complement   string
    Neighborhood string
    City         string
    State        string
    CEP          string
    Country      string
}

type Mandate struct {
    ID              string
    CondominiumID   string
    SindicoID       string
    MandateType     MandateType
    StartDate       time.Time
    EndDate         time.Time
    Status          MandateStatus
    EmpresaSindica  *EmpresaSindica
}

type EmpresaSindica struct {
    CompanyID   string
    Permissions EmpresaSindicaPermissions
    LinkedAt    time.Time
}

type EmpresaSindicaPermissions struct {
    ViewPublications       bool
    EditPublications       bool
    HidePublications       bool
    ValidateExecutions     bool
    ValidateComunicados    bool
}

// Unit entity
type Unit struct {
    ID            string
    CondominiumID string
    Block         string
    Number        string
    OwnerName     string
    OwnerCPF      string
    Status        UnitStatus
    Titular       *Morador
    Dependentes   []Morador
}

type Morador struct {
    UserID       string
    Role         MoradorRole
    JoinedAt     time.Time
    LeftAt       *time.Time
}

type MoradorRole string

const (
    MoradorRoleTitular    MoradorRole = "titular"
    MoradorRoleDependente MoradorRole = "dependente"
)
```


#### Content Context

```go
// Timeline aggregate root (immutable)
type TimelineEntry struct {
    ID              string
    TenantID        string
    EntryType       TimelineEntryType
    Title           string
    Description     string
    Date            time.Time
    AuthorID        string
    Attachments     []Attachment
    RelatedActionID *string // MasterPlan action
    Amended         bool
    Adendos         []Adendo
    PublishedAt     time.Time
    CreatedAt       time.Time
}

type TimelineEntryType string

const (
    EntryTypeMasterPlanAction   TimelineEntryType = "atividade_da_gestao"
    EntryTypeServiceExecution   TimelineEntryType = "registro_de_execucao"
    EntryTypeComunicado         TimelineEntryType = "comunicado"
    EntryTypeEvent              TimelineEntryType = "evento"
    EntryTypeAdendo             TimelineEntryType = "adendo"
    EntryTypeQuestionResult     TimelineEntryType = "resultado_consulta_moradores"
    EntryTypeAssemblyResult     TimelineEntryType = "assembly_result"
)

// MasterPlan aggregate root
type MasterPlanAction struct {
    ID              string
    TenantID        string
    Title           string
    Description     string
    AreaImpactada   AreaImpactada
    TipoAtividade   TipoAtividade
    RiscoAssociado  RiscoAssociado
    ProximaAcao     ProximaAcao
    EstimatedDate   time.Time
    EstimatedCost   *Money
    Status          MasterPlanStatus // Calculado automaticamente
    RiskScore       int
    TimelineEntryID *string
    CreatedAt       time.Time
    UpdatedAt       time.Time
}

type MasterPlanStatus string

const (
    StatusPlanejada      MasterPlanStatus = "planejada"
    StatusEmContratacao  MasterPlanStatus = "em_contratacao"
    StatusEmExecucao     MasterPlanStatus = "em_execucao"
    StatusConcluida      MasterPlanStatus = "concluida"
    StatusSuspensa       MasterPlanStatus = "suspensa"
    StatusReprogramada   MasterPlanStatus = "reprogramada"
    StatusAtrasada       MasterPlanStatus = "atrasada"
)

// Event aggregate
type Event struct {
    ID              string
    TenantID        string
    Title           string
    Description     string
    EventType       EventType
    StartDate       time.Time
    EndDate         time.Time
    Location        string
    NotifyMoradores bool
    Attachments     []Attachment
    Status          EventStatus
    Participants    []Participant
    CreatedAt       time.Time
}

// Comunicado aggregate
type Comunicado struct {
    ID              string
    TenantID        string
    Title           string
    Message         string
    ComunicadoType  ComunicadoType
    Priority        Priority
    NotifyMoradores bool
    Attachments     []Attachment
    ReadStatus      map[string]time.Time // morador_id -> read_at
    CreatedAt       time.Time
}
```

#### Commercial Context

```go
// ConnectMe aggregate root
type ConnectMeRequest struct {
    ID              string
    SourceTenantID  string
    TargetTenantID  string
    RequestType     ConnectMeType
    ServiceCategory string
    Description     string
    Urgency         Urgency
    BudgetRange     *MoneyRange
    Status          ConnectMeStatus
    InterestShownAt *time.Time
    DataRevealed    bool
    DataRevealedAt  *time.Time
    LGPDLog         *LGPDLog
    CreatedAt       time.Time
    ExpiresAt       time.Time
}

type ConnectMeType string

const (
    ConnectMeSindicoToCompany   ConnectMeType = "sindico_to_company"
    ConnectMeMoradorToSindico   ConnectMeType = "morador_to_sindico"
    ConnectMeCompanyToCompany   ConnectMeType = "company_to_company"
    ConnectMeSindicoToPartner   ConnectMeType = "sindico_to_partner"
)

type LGPDLog struct {
    UserID       string
    IPAddress    string
    Timestamp    time.Time
    TermsVersion string
}

// Proposal aggregate
type Proposal struct {
    ID              string
    CompanyID       string
    CondominiumID   string
    ConnectMeID     *string
    Title           string
    Description     string
    Scope           string
    EstimatedCost   Money
    EstimatedDuration Duration
    Attachments     []Attachment
    Status          ProposalStatus
    ValidationBy    *string // sindico_id
    ValidatedAt     *time.Time
    RejectionReason *string
    CreatedAt       time.Time
    UpdatedAt       time.Time
}

// ClosedDeal aggregate
type ClosedDeal struct {
    ID              string
    CompanyID       string
    CondominiumID   string
    ProposalID      *string
    ServiceDescription string
    AgreedCost      Money
    StartDate       time.Time
    EndDate         time.Time
    PaymentTerms    string
    CompanySigned   bool
    CompanySignedAt *time.Time
    SindicoSigned   bool
    SindicoSignedAt *time.Time
    Status          DealStatus
    Executions      []ServiceExecution
    Garantia        *Garantia
    CreatedAt       time.Time
}

type ServiceExecution struct {
    ID              string
    DealID          string
    Description     string
    Date            time.Time
    Photos          []string
    Videos          []string
    RiskLevel       RiskLevel
    Nature          ServiceNature
    Status          ExecutionStatus
    ValidatedBy     *string
    ValidatedAt     *time.Time
    TimelineEntryID *string
}
```


#### Assembly Context

```go
// Assembly aggregate root
type Assembly struct {
    ID                  string
    TenantID            string
    AssemblyType        AssemblyType
    ConvocationDate     time.Time
    ScheduledDate       time.Time
    StartTime           *time.Time
    EndTime             *time.Time
    Location            string
    Agenda              []AgendaItem
    Quorum              QuorumConfig
    SpeechTime          SpeechTime
    Participants        InstitutionalParticipants
    Attendance          []AttendanceRecord
    Proxies             []Proxy
    Status              AssemblyStatus
    TransparencyScore   *TransparencyIndicator
    CreatedAt           time.Time
}

type AgendaItem struct {
    ID              string
    Order           int
    Title           string
    Description     string
    ItemType        AgendaItemType
    VotingRequired  bool
    VoteType        *VoteType
    Documents       []Attachment
    Status          ItemStatus
    OpenedAt        *time.Time
    ClosedAt        *time.Time
    Votes           []Vote
    Result          *VotingResult
    Homologated     bool
    HomologatedBy   *string
    HomologatedAt   *time.Time
}

type Vote struct {
    ID          string
    AgendaItemID string
    UnitID      string
    MoradorID   string
    VoteOption  VoteOption
    Weight      float64 // fração ideal
    VoteOrigin  VoteOrigin
    CastAt      time.Time
}

type VoteOrigin string

const (
    VoteOriginApp               VoteOrigin = "app"
    VoteOriginPresencialAssistido VoteOrigin = "presencial_assistido"
    VoteOriginProxy             VoteOrigin = "proxy"
)

type Proxy struct {
    ID              string
    AssemblyID      string
    DelegatorUnitID string
    DelegateUnitID  string
    Document        string
    ValidatedBy     string
    ValidatedAt     time.Time
}

type AttendanceRecord struct {
    ID          string
    AssemblyID  string
    MoradorID   string
    UnitID      string
    CheckInMethod CheckInMethod
    CheckInAt   time.Time
    Status      AttendanceStatus
}

type AttendanceStatus string

const (
    AttendancePresent    AttendanceStatus = "presente"
    AttendanceAbsent     AttendanceStatus = "ausente_momentaneo"
    AttendanceLeft       AttendanceStatus = "desconectado_saiu"
)
```

#### Compliance Context

```go
// AuditLog (append-only, immutable)
type AuditLog struct {
    ID          string
    TenantID    string
    EventType   string
    AggregateID string
    UserID      string
    IPAddress   string
    UserAgent   string
    Action      string
    Payload     json.RawMessage
    CreatedAt   time.Time
}

// Declaration aggregate
type Declaration struct {
    ID              string
    TenantID        string
    DeclarationType DeclarationType
    Year            int
    Status          DeclarationStatus
    SubmittedBy     string
    SubmittedAt     time.Time
    Content         json.RawMessage
    Attachments     []Attachment
}

type DeclarationType string

const (
    DeclarationFiscal      DeclarationType = "fiscal"
    DeclarationTechnical   DeclarationType = "technical"
    DeclarationLabor       DeclarationType = "labor"
    DeclarationConflict    DeclarationType = "conflict_of_interest"
)

// LGPD Consent
type LGPDConsent struct {
    ID              string
    UserID          string
    ConsentType     ConsentType
    Purpose         string
    Granted         bool
    GrantedAt       *time.Time
    RevokedAt       *time.Time
    TermsVersion    string
    IPAddress       string
}

type ConsentType string

const (
    ConsentDataProcessing   ConsentType = "data_processing"
    ConsentDataSharing      ConsentType = "data_sharing"
    ConsentMarketing        ConsentType = "marketing"
    ConsentAssemblyRecording ConsentType = "assembly_recording"
)
```

### Repository Interfaces

```go
// Identity Context
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id string) (*User, error)
    FindByEmail(ctx context.Context, email string) (*User, error)
    Update(ctx context.Context, user *User) error
    Delete(ctx context.Context, id string) error
}

type SubscriptionRepository interface {
    Create(ctx context.Context, sub *Subscription) error
    FindByUserID(ctx context.Context, userID string) (*Subscription, error)
    FindByTenantID(ctx context.Context, tenantID string) (*Subscription, error)
    Update(ctx context.Context, sub *Subscription) error
}

// Institutional Context
type CondominiumRepository interface {
    Create(ctx context.Context, condo *Condominium) error
    FindByID(ctx context.Context, id string) (*Condominium, error)
    FindBySindicoID(ctx context.Context, sindicoID string) ([]*Condominium, error)
    Update(ctx context.Context, condo *Condominium) error
    Delete(ctx context.Context, id string) error
}

type UnitRepository interface {
    Create(ctx context.Context, unit *Unit) error
    FindByID(ctx context.Context, id string) (*Unit, error)
    FindByCondominiumID(ctx context.Context, condoID string) ([]*Unit, error)
    Update(ctx context.Context, unit *Unit) error
}

// Content Context
type TimelineRepository interface {
    Create(ctx context.Context, entry *TimelineEntry) error
    FindByID(ctx context.Context, id string) (*TimelineEntry, error)
    FindByTenantID(ctx context.Context, tenantID string, filters TimelineFilters) ([]*TimelineEntry, error)
    // Não há Update ou Delete - append-only
}

type MasterPlanRepository interface {
    Create(ctx context.Context, action *MasterPlanAction) error
    FindByID(ctx context.Context, id string) (*MasterPlanAction, error)
    FindByTenantID(ctx context.Context, tenantID string, filters MasterPlanFilters) ([]*MasterPlanAction, error)
    Update(ctx context.Context, action *MasterPlanAction) error
}

// Commercial Context
type ConnectMeRepository interface {
    Create(ctx context.Context, request *ConnectMeRequest) error
    FindByID(ctx context.Context, id string) (*ConnectMeRequest, error)
    FindBySourceTenant(ctx context.Context, tenantID string) ([]*ConnectMeRequest, error)
    FindByTargetTenant(ctx context.Context, tenantID string) ([]*ConnectMeRequest, error)
    Update(ctx context.Context, request *ConnectMeRequest) error
}

type ProposalRepository interface {
    Create(ctx context.Context, proposal *Proposal) error
    FindByID(ctx context.Context, id string) (*Proposal, error)
    FindByCompanyID(ctx context.Context, companyID string) ([]*Proposal, error)
    FindByCondominiumID(ctx context.Context, condoID string) ([]*Proposal, error)
    Update(ctx context.Context, proposal *Proposal) error
}

// Assembly Context
type AssemblyRepository interface {
    Create(ctx context.Context, assembly *Assembly) error
    FindByID(ctx context.Context, id string) (*Assembly, error)
    FindByTenantID(ctx context.Context, tenantID string) ([]*Assembly, error)
    Update(ctx context.Context, assembly *Assembly) error
}

// Compliance Context
type AuditLogRepository interface {
    Append(ctx context.Context, log *AuditLog) error
    FindByTenantID(ctx context.Context, tenantID string, filters AuditFilters) ([]*AuditLog, error)
    FindByAggregateID(ctx context.Context, aggregateID string) ([]*AuditLog, error)
    // Não há Update ou Delete - append-only
}
```


### Domain Services

```go
// Identity Context
type AuthenticationService interface {
    Authenticate(ctx context.Context, email, password string) (*AuthToken, error)
    ValidateToken(ctx context.Context, token string) (*TokenClaims, error)
    RefreshToken(ctx context.Context, refreshToken string) (*AuthToken, error)
    Logout(ctx context.Context, userID string) error
    ValidateDeviceFingerprint(ctx context.Context, userID, deviceID, ipAddress string) error
}

type AuthorizationService interface {
    CheckPermission(ctx context.Context, userID, tenantID, action string) (bool, error)
    GetUserPermissions(ctx context.Context, userID, tenantID string) (*Permissions, error)
    GrantCrossTenantAccess(ctx context.Context, grant *CrossTenantGrant) error
}

// Content Context
type MasterPlanService interface {
    UpdateActionStatus(ctx context.Context, actionID string, status MasterPlanStatus) error
    CalculateRiskScore(action *MasterPlanAction) int
    GetOverdueActions(ctx context.Context, tenantID string) ([]*MasterPlanAction, error)
}

type TimelineService interface {
    PublishEntry(ctx context.Context, entry *TimelineEntry) error
    CreateAdendo(ctx context.Context, originalID string, adendo *Adendo) error
    GetTimelineHistory(ctx context.Context, tenantID string, filters TimelineFilters) ([]*TimelineEntry, error)
}

// Commercial Context
type ConnectMeService interface {
    CreateRequest(ctx context.Context, request *ConnectMeRequest) error
    ShowInterest(ctx context.Context, requestID, companyID string) error
    RevealContactData(ctx context.Context, requestID string) (*ContactData, error)
    AutoCloseExpiredRequests(ctx context.Context) error
}

type ValidationService interface {
    ValidateProposal(ctx context.Context, proposalID, sindicoID string, approved bool, reason *string) error
    ValidateExecution(ctx context.Context, executionID, sindicoID string, approved bool, reason *string) error
    GetPendingValidations(ctx context.Context, tenantID string) (*PendingValidations, error)
}

// Assembly Context
type AssemblyService interface {
    StartAssembly(ctx context.Context, assemblyID string) error
    CheckInMorador(ctx context.Context, assemblyID, moradorID string, method CheckInMethod) error
    CastVote(ctx context.Context, vote *Vote) error
    HomologateVoting(ctx context.Context, agendaItemID, homologatorID string) error
    FinalizeAssembly(ctx context.Context, assemblyID string) error
    CalculateTransparencyScore(assembly *Assembly) (*TransparencyIndicator, error)
}

type QuorumService interface {
    CalculateQuorum(assembly *Assembly) (*QuorumStatus, error)
    ValidateQuorum(assembly *Assembly, item *AgendaItem) (bool, error)
    SimulateQuorum(assembly *Assembly, projectedAttendance int) (*QuorumSimulation, error)
}

// Compliance Context
type AuditService interface {
    LogAction(ctx context.Context, log *AuditLog) error
    GetAuditTrail(ctx context.Context, aggregateID string) ([]*AuditLog, error)
    ExportAuditReport(ctx context.Context, tenantID string, filters AuditFilters) ([]byte, error)
}

type LGPDService interface {
    RecordConsent(ctx context.Context, consent *LGPDConsent) error
    RevokeConsent(ctx context.Context, consentID string) error
    GetUserConsents(ctx context.Context, userID string) ([]*LGPDConsent, error)
    ExportUserData(ctx context.Context, userID string) ([]byte, error)
    DeleteUserData(ctx context.Context, userID string) error
}
```

## Data Models

### Database Schema Design

#### Multi-Tenancy Pattern

Todas as tabelas principais incluem `tenant_id` com índice composto:

```sql
CREATE TABLE condominiums (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,
    name VARCHAR(255) NOT NULL,
    address JSONB NOT NULL,
    condominium_type VARCHAR(50) NOT NULL,
    total_units INTEGER NOT NULL,
    alphanumeric_id VARCHAR(8) UNIQUE NOT NULL,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id)
);

CREATE INDEX idx_condominiums_tenant ON condominiums(tenant_id);
CREATE INDEX idx_condominiums_tenant_status ON condominiums(tenant_id, status);
```

#### Core Tables

**Tenants**
```sql
CREATE TABLE tenants (
    id UUID PRIMARY KEY,
    type VARCHAR(50) NOT NULL, -- 'condominium' | 'company'
    name VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_tenants_type ON tenants(type);
CREATE INDEX idx_tenants_status ON tenants(status);
```

**Users**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    user_type VARCHAR(50) NOT NULL,
    status VARCHAR(50) NOT NULL,
    email_verified BOOLEAN DEFAULT FALSE,
    email_verified_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_type ON users(user_type);
CREATE INDEX idx_users_status ON users(status);
```

**Device Fingerprints**
```sql
CREATE TABLE device_fingerprints (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    device_id VARCHAR(255) NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    user_agent TEXT,
    last_seen_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id),
    CONSTRAINT unique_user_device UNIQUE (user_id, device_id)
);

CREATE INDEX idx_device_fingerprints_user ON device_fingerprints(user_id);
CREATE INDEX idx_device_fingerprints_ip ON device_fingerprints(ip_address);
```

**Subscriptions**
```sql
CREATE TABLE subscriptions (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    tenant_id UUID NOT NULL,
    plan VARCHAR(50) NOT NULL,
    status VARCHAR(50) NOT NULL,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP,
    trial_end_date TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id),
    CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id)
);

CREATE INDEX idx_subscriptions_user ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_tenant ON subscriptions(tenant_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status);
```


**Timeline (Append-Only)**
```sql
CREATE TABLE timeline_entries (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,
    entry_type VARCHAR(50) NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    date TIMESTAMP NOT NULL,
    author_id UUID NOT NULL,
    related_action_id UUID, -- MasterPlan action
    amended BOOLEAN DEFAULT FALSE,
    published_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id),
    CONSTRAINT fk_author FOREIGN KEY (author_id) REFERENCES users(id)
);

CREATE INDEX idx_timeline_tenant ON timeline_entries(tenant_id);
CREATE INDEX idx_timeline_tenant_date ON timeline_entries(tenant_id, date DESC);
CREATE INDEX idx_timeline_type ON timeline_entries(entry_type);
CREATE INDEX idx_timeline_author ON timeline_entries(author_id);

-- Attachments separados para normalização
CREATE TABLE timeline_attachments (
    id UUID PRIMARY KEY,
    timeline_entry_id UUID NOT NULL,
    file_type VARCHAR(50) NOT NULL,
    file_url TEXT NOT NULL,
    file_size BIGINT,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_timeline_entry FOREIGN KEY (timeline_entry_id) REFERENCES timeline_entries(id)
);
```

**Master Plan**
```sql
CREATE TABLE master_plan_actions (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    area_impactada VARCHAR(100) NOT NULL,
    tipo_atividade VARCHAR(100) NOT NULL,
    risco_associado VARCHAR(50) NOT NULL,
    proxima_acao VARCHAR(100) NOT NULL,
    estimated_date TIMESTAMP NOT NULL,
    estimated_cost_amount DECIMAL(15,2),
    estimated_cost_currency VARCHAR(3),
    status VARCHAR(50) NOT NULL,
    risk_score INTEGER NOT NULL,
    timeline_entry_id UUID,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id),
    CONSTRAINT fk_timeline_entry FOREIGN KEY (timeline_entry_id) REFERENCES timeline_entries(id)
);

CREATE INDEX idx_master_plan_tenant ON master_plan_actions(tenant_id);
CREATE INDEX idx_master_plan_status ON master_plan_actions(tenant_id, status);
CREATE INDEX idx_master_plan_risk ON master_plan_actions(tenant_id, risk_score DESC);
CREATE INDEX idx_master_plan_date ON master_plan_actions(tenant_id, estimated_date);
```

**Connect Me Requests**
```sql
CREATE TABLE connect_me_requests (
    id UUID PRIMARY KEY,
    source_tenant_id UUID NOT NULL,
    target_tenant_id UUID NOT NULL,
    request_type VARCHAR(50) NOT NULL,
    service_category VARCHAR(100),
    description TEXT NOT NULL,
    urgency VARCHAR(50) NOT NULL,
    budget_min DECIMAL(15,2),
    budget_max DECIMAL(15,2),
    status VARCHAR(50) NOT NULL,
    interest_shown_at TIMESTAMP,
    data_revealed BOOLEAN DEFAULT FALSE,
    data_revealed_at TIMESTAMP,
    lgpd_log JSONB,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMP NOT NULL,
    
    CONSTRAINT fk_source_tenant FOREIGN KEY (source_tenant_id) REFERENCES tenants(id),
    CONSTRAINT fk_target_tenant FOREIGN KEY (target_tenant_id) REFERENCES tenants(id)
);

CREATE INDEX idx_connect_me_source ON connect_me_requests(source_tenant_id);
CREATE INDEX idx_connect_me_target ON connect_me_requests(target_tenant_id);
CREATE INDEX idx_connect_me_status ON connect_me_requests(status);
CREATE INDEX idx_connect_me_expires ON connect_me_requests(expires_at);
```

**Assemblies**
```sql
CREATE TABLE assemblies (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,
    assembly_type VARCHAR(50) NOT NULL,
    convocation_date TIMESTAMP NOT NULL,
    scheduled_date TIMESTAMP NOT NULL,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    location VARCHAR(255),
    quorum_config JSONB NOT NULL,
    speech_time VARCHAR(20) NOT NULL,
    participants JSONB NOT NULL,
    status VARCHAR(50) NOT NULL,
    transparency_score JSONB,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id)
);

CREATE INDEX idx_assemblies_tenant ON assemblies(tenant_id);
CREATE INDEX idx_assemblies_status ON assemblies(status);
CREATE INDEX idx_assemblies_date ON assemblies(scheduled_date);

CREATE TABLE agenda_items (
    id UUID PRIMARY KEY,
    assembly_id UUID NOT NULL,
    order_num INTEGER NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    item_type VARCHAR(50) NOT NULL,
    voting_required BOOLEAN DEFAULT FALSE,
    vote_type VARCHAR(50),
    status VARCHAR(50) NOT NULL,
    opened_at TIMESTAMP,
    closed_at TIMESTAMP,
    homologated BOOLEAN DEFAULT FALSE,
    homologated_by UUID,
    homologated_at TIMESTAMP,
    
    CONSTRAINT fk_assembly FOREIGN KEY (assembly_id) REFERENCES assemblies(id),
    CONSTRAINT unique_assembly_order UNIQUE (assembly_id, order_num)
);

CREATE TABLE votes (
    id UUID PRIMARY KEY,
    agenda_item_id UUID NOT NULL,
    unit_id UUID NOT NULL,
    morador_id UUID NOT NULL,
    vote_option VARCHAR(50) NOT NULL,
    weight DECIMAL(10,4) DEFAULT 1.0,
    vote_origin VARCHAR(50) NOT NULL,
    cast_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_agenda_item FOREIGN KEY (agenda_item_id) REFERENCES agenda_items(id),
    CONSTRAINT unique_unit_vote UNIQUE (agenda_item_id, unit_id)
);

CREATE INDEX idx_votes_agenda_item ON votes(agenda_item_id);
CREATE INDEX idx_votes_morador ON votes(morador_id);
```

**Audit Logs (Append-Only)**
```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    aggregate_id UUID NOT NULL,
    user_id UUID NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    user_agent TEXT,
    action VARCHAR(100) NOT NULL,
    payload JSONB NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    
    CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id),
    CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_audit_logs_tenant ON audit_logs(tenant_id, created_at DESC);
CREATE INDEX idx_audit_logs_aggregate ON audit_logs(aggregate_id, created_at DESC);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id, created_at DESC);
CREATE INDEX idx_audit_logs_event_type ON audit_logs(event_type);

-- Particionamento por data para performance (5 anos de retenção)
CREATE TABLE audit_logs_2024 PARTITION OF audit_logs
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

### Value Objects

```go
type Money struct {
    Amount   decimal.Decimal
    Currency string // BRL, USD, etc
}

type MoneyRange struct {
    Min Money
    Max Money
}

type Duration struct {
    Value int
    Unit  DurationUnit // days, weeks, months
}

type DateRange struct {
    Start time.Time
    End   time.Time
}

type Attachment struct {
    ID       string
    FileType string
    FileURL  string
    FileSize int64
    UploadedAt time.Time
}

type QuorumConfig struct {
    Type             QuorumType
    RequiredPercent  float64
    FracaoIdealBased bool
}

type TransparencyIndicator struct {
    Overall         float64
    Convocacao      float64
    Participacao    float64
    Votacao         float64
    Documentacao    float64
    Components      map[string]float64
}
```


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system-essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

Após análise dos acceptance criteria, identifiquei as seguintes redundâncias que foram eliminadas:

- **Propriedades 1.8 e 1.10**: Ambas testam invalidação de sessão ao login de segundo dispositivo → Consolidadas em Property 1
- **Propriedades 11.1 e 11.7**: Ambas testam imutabilidade da Timeline → Consolidadas em Property 5
- **Propriedades de validação de campos obrigatórios**: Múltiplos requirements testam validação de campos → Consolidadas em Property 3

### Authentication and Authorization Properties

#### Property 1: Single Device Session Enforcement
*For any* user account, when authentication occurs from a new device, the previous device session should be invalidated immediately, ensuring only one active session exists per user.

**Validates: Requirements 1.8, 1.10**

#### Property 2: Expired Token Rejection
*For any* JWT token with expiration timestamp in the past, authentication attempts using that token should be rejected with an authentication error.

**Validates: Requirements 1.6**

#### Property 3: Authentication Attempt Logging
*For any* authentication attempt (successful or failed), an audit log entry should be created containing timestamp, IP address, user agent, and attempt result.

**Validates: Requirements 1.7, 1.9**

#### Property 4: Rate Limiting Enforcement
*For any* user account, after N failed authentication attempts within a time window, subsequent authentication attempts should be blocked until the time window expires.

**Validates: Requirements 1.5**

#### Property 5: IP-Based Fraud Prevention
*For any* IP address, attempting to create more than M user accounts should be blocked to prevent plan circumvention.

**Validates: Requirements 1.11**

#### Property 6: Tenant Isolation
*For any* user and any data query, the query results should only include data where tenant_id matches the user's tenant, ensuring complete tenant isolation.

**Validates: Requirements 2.5**

#### Property 7: Unauthorized Access Logging
*For any* unauthorized access attempt, the system should both deny the access and create an audit log entry with user_id, attempted action, and timestamp.

**Validates: Requirements 2.6**

#### Property 8: Cross-Tenant Grant Validation
*For any* cross-tenant operation, access should only be granted if an explicit CrossTenantGrant exists with valid expiration and consent_given=true.

**Validates: Requirements 2.7**

### Timeline and Content Properties

#### Property 9: Timeline Immutability
*For any* published Timeline entry, attempts to modify or delete the entry should be rejected, maintaining append-only immutability.

**Validates: Requirements 11.1, 11.7**

#### Property 10: Timeline Entry Uniqueness
*For any* two Timeline entries, their entry_id values should be unique, ensuring no ID collisions.

**Validates: Requirements 11.5**

#### Property 11: Timeline Publication Timestamp
*For any* Timeline entry, when published, a published_at timestamp should be set and should never be null for published entries.

**Validates: Requirements 11.6**

#### Property 12: Adendo Linkage
*For any* Adendo created, it should be linked to exactly one original Timeline entry, and the original entry should be marked as amended=true.

**Validates: Requirements 11.8, 11.9**

#### Property 13: Timeline Event Emission
*For any* Timeline entry publication, a TimelinePublished event should be emitted containing the entry_id, tenant_id, and entry_type.

**Validates: Requirements 11.10**

#### Property 14: Master Plan Status Automation
*For any* Timeline entry with a related_action_id, publishing the entry should automatically update the corresponding MasterPlanAction status to completed.

**Validates: Requirements 11.11**

#### Property 15: Morador Read-Only Access
*For any* morador user, Timeline queries should succeed but Timeline write operations (create, update, delete) should be denied.

**Validates: Requirements 11.12**

### Connect Me and LGPD Properties

#### Property 16: Contact Data Privacy
*For any* ConnectMe request, sindico contact data (name, phone, email) should not be visible to the company until the company explicitly shows interest.

**Validates: Requirements 19.3**

#### Property 17: LGPD Data Revelation Logging
*For any* ConnectMe request where a company shows interest, a LGPD log entry should be created containing timestamp, company_id, sindico_id, ip_address, and terms_version.

**Validates: Requirements 19.6**

#### Property 18: Connect Me Quota Enforcement
*For any* user with plan-based quotas, creating a ConnectMe request that would exceed the quota limit should be rejected with a quota error.

**Validates: Requirements 19.7, 19.8**

#### Property 19: Connect Me Auto-Expiration
*For any* ConnectMe request with no interest shown, if 48 hours have elapsed since creation, the request status should automatically change to expired and a notification should be sent to the requester.

**Validates: Requirements 19.11, 19.12**

### Assembly Properties

#### Property 20: Assembly Convocation Lead Time
*For any* assembly of type ordinaria, the convocation_date should be at least 8 days before scheduled_date; for extraordinaria, at least 3 days before.

**Validates: Requirements 48.3, 48.4**

#### Property 21: Vote Uniqueness Per Unit
*For any* agenda item with voting enabled, each unit should be able to cast exactly one vote, preventing duplicate votes from the same unit.

**Validates: Requirements 57.3**

#### Property 22: Proxy Vote Replication
*For any* morador with an active proxy delegation, when the delegate casts a vote, the vote should automatically count for both the delegate's unit and the delegator's unit.

**Validates: Requirements 57.4**

#### Property 23: Vote Immutability After Closure
*For any* agenda item, once voting is closed, all votes for that item should become immutable and attempts to modify or delete votes should be rejected.

**Validates: Requirements 57.6, 57.7**

#### Property 24: Assembly Auto-Save
*For any* vote cast during an assembly, the vote should be persisted to the database immediately, ensuring no data loss even if the system crashes.

**Validates: Requirements 57.8**

### Serialization Properties

#### Property 25: JSON Round-Trip Preservation
*For any* valid data structure, serializing to JSON then parsing back to a data structure should produce a structure equivalent to the original (round-trip property).

**Validates: Requirements 68.4**

#### Property 26: JSON Parse Error Reporting
*For any* invalid JSON string, the parser should return an error containing line number and column number information indicating where the parse failure occurred.

**Validates: Requirements 68.2**

### Encryption and Security Properties

#### Property 27: Sensitive Field Encryption
*For any* record containing sensitive fields (passwords, cpf, cnpj, financial_data), the fields should be encrypted at rest in the database.

**Validates: Requirements 71.1**

#### Property 28: Decryption Authorization
*For any* decryption request, the operation should only succeed if the requesting user has authorization to access the encrypted data.

**Validates: Requirements 71.4**

#### Property 29: Decryption Audit Trail
*For any* successful decryption operation, an audit log entry should be created containing user_id, data_type, timestamp, and ip_address.

**Validates: Requirements 71.5**


## Error Handling

### Error Types

```go
type ErrorType string

const (
    ErrorTypeValidation      ErrorType = "validation_error"
    ErrorTypeAuthentication  ErrorType = "authentication_error"
    ErrorTypeAuthorization   ErrorType = "authorization_error"
    ErrorTypeNotFound        ErrorType = "not_found"
    ErrorTypeConflict        ErrorType = "conflict"
    ErrorTypeRateLimit       ErrorType = "rate_limit_exceeded"
    ErrorTypeQuotaExceeded   ErrorType = "quota_exceeded"
    ErrorTypeInternal        ErrorType = "internal_error"
    ErrorTypeExternal        ErrorType = "external_service_error"
)

type AppError struct {
    Type       ErrorType
    Message    string
    Details    map[string]interface{}
    StatusCode int
    Err        error
}

func (e *AppError) Error() string {
    return e.Message
}
```

### Error Handling Strategy

#### Domain Layer
- Domain errors são retornados como valores, não panic
- Erros de validação de regras de negócio são explícitos
- Erros de invariantes violadas são críticos

```go
func (c *Condominium) AddUnit(unit *Unit) error {
    if c.Status != CondominiumStatusActive {
        return &AppError{
            Type:    ErrorTypeValidation,
            Message: "cannot add unit to inactive condominium",
            Details: map[string]interface{}{
                "condominium_id": c.ID,
                "status":         c.Status,
            },
        }
    }
    
    if len(c.Units) >= c.TotalUnits {
        return &AppError{
            Type:    ErrorTypeConflict,
            Message: "condominium has reached maximum unit capacity",
            Details: map[string]interface{}{
                "condominium_id": c.ID,
                "total_units":    c.TotalUnits,
                "current_units":  len(c.Units),
            },
        }
    }
    
    c.Units = append(c.Units, *unit)
    return nil
}
```

#### Application Layer
- Use cases capturam erros de domínio e infraestrutura
- Traduzem erros técnicos em erros de aplicação
- Adicionam contexto relevante

```go
func (uc *CreateCondominiumUseCase) Execute(ctx context.Context, cmd CreateCondominiumCommand) (*CondominiumDTO, error) {
    // Validação de entrada
    if err := cmd.Validate(); err != nil {
        return nil, &AppError{
            Type:       ErrorTypeValidation,
            Message:    "invalid command",
            Details:    map[string]interface{}{"validation_errors": err},
            StatusCode: http.StatusBadRequest,
        }
    }
    
    // Verificar quota de condomínios
    count, err := uc.condoRepo.CountBySindicoID(ctx, cmd.SindicoID)
    if err != nil {
        return nil, &AppError{
            Type:       ErrorTypeInternal,
            Message:    "failed to check condominium quota",
            StatusCode: http.StatusInternalServerError,
            Err:        err,
        }
    }
    
    if count >= 15 {
        return nil, &AppError{
            Type:       ErrorTypeQuotaExceeded,
            Message:    "maximum condominium limit reached",
            Details:    map[string]interface{}{"limit": 15, "current": count},
            StatusCode: http.StatusForbidden,
        }
    }
    
    // Criar condomínio
    condo := NewCondominium(cmd)
    if err := uc.condoRepo.Create(ctx, condo); err != nil {
        return nil, &AppError{
            Type:       ErrorTypeInternal,
            Message:    "failed to create condominium",
            StatusCode: http.StatusInternalServerError,
            Err:        err,
        }
    }
    
    return MapCondominiumToDTO(condo), nil
}
```

#### Infrastructure Layer
- Erros de banco de dados são wrapeados
- Erros de serviços externos são tratados com retry e circuit breaker
- Timeouts são configuráveis

```go
func (r *CondominiumRepository) Create(ctx context.Context, condo *Condominium) error {
    result := r.db.WithContext(ctx).Create(condo)
    if result.Error != nil {
        if errors.Is(result.Error, gorm.ErrDuplicatedKey) {
            return &AppError{
                Type:    ErrorTypeConflict,
                Message: "condominium with this ID already exists",
                Err:     result.Error,
            }
        }
        return &AppError{
            Type:    ErrorTypeInternal,
            Message: "database error",
            Err:     result.Error,
        }
    }
    return nil
}
```

#### Presentation Layer
- Erros são convertidos em respostas HTTP apropriadas
- Detalhes técnicos são ocultados em produção
- Logs estruturados para debugging

```go
func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next()
        
        if len(c.Errors) > 0 {
            err := c.Errors.Last().Err
            
            var appErr *AppError
            if errors.As(err, &appErr) {
                c.JSON(appErr.StatusCode, gin.H{
                    "error": gin.H{
                        "type":    appErr.Type,
                        "message": appErr.Message,
                        "details": appErr.Details,
                    },
                })
                return
            }
            
            // Erro não tratado
            c.JSON(http.StatusInternalServerError, gin.H{
                "error": gin.H{
                    "type":    ErrorTypeInternal,
                    "message": "internal server error",
                },
            })
        }
    }
}
```

### Retry Strategy

Para operações com serviços externos, implementamos retry com exponential backoff:

```go
type RetryConfig struct {
    MaxAttempts int
    InitialDelay time.Duration
    MaxDelay     time.Duration
    Multiplier   float64
}

func RetryWithBackoff(ctx context.Context, config RetryConfig, operation func() error) error {
    delay := config.InitialDelay
    
    for attempt := 1; attempt <= config.MaxAttempts; attempt++ {
        err := operation()
        if err == nil {
            return nil
        }
        
        // Não fazer retry para erros não-retriáveis
        var appErr *AppError
        if errors.As(err, &appErr) {
            if appErr.Type == ErrorTypeValidation || appErr.Type == ErrorTypeAuthorization {
                return err
            }
        }
        
        if attempt == config.MaxAttempts {
            return err
        }
        
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-time.After(delay):
            delay = time.Duration(float64(delay) * config.Multiplier)
            if delay > config.MaxDelay {
                delay = config.MaxDelay
            }
        }
    }
    
    return nil
}
```

### Circuit Breaker

Para proteger contra falhas em cascata:

```go
type CircuitBreaker struct {
    maxFailures  int
    timeout      time.Duration
    state        CircuitState
    failures     int
    lastFailTime time.Time
    mu           sync.RWMutex
}

type CircuitState string

const (
    CircuitStateClosed   CircuitState = "closed"
    CircuitStateOpen     CircuitState = "open"
    CircuitStateHalfOpen CircuitState = "half_open"
)

func (cb *CircuitBreaker) Execute(operation func() error) error {
    cb.mu.RLock()
    state := cb.state
    cb.mu.RUnlock()
    
    if state == CircuitStateOpen {
        if time.Since(cb.lastFailTime) > cb.timeout {
            cb.mu.Lock()
            cb.state = CircuitStateHalfOpen
            cb.mu.Unlock()
        } else {
            return &AppError{
                Type:    ErrorTypeExternal,
                Message: "circuit breaker is open",
            }
        }
    }
    
    err := operation()
    
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    if err != nil {
        cb.failures++
        cb.lastFailTime = time.Now()
        
        if cb.failures >= cb.maxFailures {
            cb.state = CircuitStateOpen
        }
        
        return err
    }
    
    cb.failures = 0
    cb.state = CircuitStateClosed
    return nil
}
```


## Testing Strategy

### Dual Testing Approach

A estratégia de testes combina unit tests e property-based tests de forma complementar:

- **Unit Tests**: Verificam exemplos específicos, casos extremos e condições de erro
- **Property Tests**: Verificam propriedades universais através de geração aleatória de inputs
- Ambos são necessários para cobertura abrangente

### Unit Testing

#### Foco dos Unit Tests

- Exemplos concretos que demonstram comportamento correto
- Casos extremos (edge cases) identificados durante análise
- Condições de erro específicas
- Integração entre componentes

#### Estrutura de Unit Tests

```go
// Domain layer tests
func TestCondominium_AddUnit_Success(t *testing.T) {
    // Arrange
    condo := &Condominium{
        ID:         "condo-1",
        TotalUnits: 10,
        Status:     CondominiumStatusActive,
        Units:      []Unit{},
    }
    
    unit := &Unit{
        ID:     "unit-1",
        Number: "101",
    }
    
    // Act
    err := condo.AddUnit(unit)
    
    // Assert
    assert.NoError(t, err)
    assert.Len(t, condo.Units, 1)
    assert.Equal(t, "unit-1", condo.Units[0].ID)
}

func TestCondominium_AddUnit_ExceedsCapacity(t *testing.T) {
    // Arrange
    condo := &Condominium{
        ID:         "condo-1",
        TotalUnits: 1,
        Status:     CondominiumStatusActive,
        Units:      []Unit{{ID: "existing-unit"}},
    }
    
    unit := &Unit{ID: "unit-2", Number: "102"}
    
    // Act
    err := condo.AddUnit(unit)
    
    // Assert
    assert.Error(t, err)
    assert.Contains(t, err.Error(), "maximum unit capacity")
}

// Application layer tests
func TestCreateCondominiumUseCase_Execute_Success(t *testing.T) {
    // Arrange
    mockRepo := new(MockCondominiumRepository)
    mockEventBus := new(MockEventBus)
    
    uc := NewCreateCondominiumUseCase(mockRepo, mockEventBus)
    
    cmd := CreateCondominiumCommand{
        SindicoID: "sindico-1",
        Name:      "Test Condo",
        Address:   Address{Street: "Main St"},
    }
    
    mockRepo.On("CountBySindicoID", mock.Anything, "sindico-1").Return(5, nil)
    mockRepo.On("Create", mock.Anything, mock.AnythingOfType("*Condominium")).Return(nil)
    mockEventBus.On("Publish", mock.Anything, mock.AnythingOfType("CondominiumCreated")).Return(nil)
    
    // Act
    result, err := uc.Execute(context.Background(), cmd)
    
    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, result)
    mockRepo.AssertExpectations(t)
    mockEventBus.AssertExpectations(t)
}

// Infrastructure layer tests
func TestCondominiumRepository_Create_DuplicateID(t *testing.T) {
    // Arrange
    db := setupTestDB(t)
    repo := NewCondominiumRepository(db)
    
    condo1 := &Condominium{ID: "condo-1", Name: "First"}
    condo2 := &Condominium{ID: "condo-1", Name: "Second"}
    
    // Act
    err1 := repo.Create(context.Background(), condo1)
    err2 := repo.Create(context.Background(), condo2)
    
    // Assert
    assert.NoError(t, err1)
    assert.Error(t, err2)
    
    var appErr *AppError
    assert.True(t, errors.As(err2, &appErr))
    assert.Equal(t, ErrorTypeConflict, appErr.Type)
}
```

### Property-Based Testing

#### Biblioteca Recomendada

Para Go, recomendamos **gopter** (https://github.com/leanovate/gopter) para property-based testing.

```go
import (
    "testing"
    "github.com/leanovate/gopter"
    "github.com/leanovate/gopter/gen"
    "github.com/leanovate/gopter/prop"
)
```

#### Configuração de Property Tests

Cada property test deve executar no mínimo 100 iterações:

```go
func TestProperties(t *testing.T) {
    parameters := gopter.DefaultTestParameters()
    parameters.MinSuccessfulTests = 100
    properties := gopter.NewProperties(parameters)
    
    // Adicionar properties aqui
    
    properties.TestingRun(t)
}
```

#### Property Test Examples

```go
// Property 1: Single Device Session Enforcement
// Feature: master-sindico-platform, Property 1: For any user account, when authentication occurs from a new device, the previous device session should be invalidated immediately
func TestProperty_SingleDeviceSessionEnforcement(t *testing.T) {
    parameters := gopter.DefaultTestParameters()
    parameters.MinSuccessfulTests = 100
    properties := gopter.NewProperties(parameters)
    
    properties.Property("single device session enforcement", prop.ForAll(
        func(userID string, device1 string, device2 string) bool {
            // Arrange
            authService := setupAuthService()
            
            // Act: Login from device 1
            session1, _ := authService.Authenticate(context.Background(), userID, "password", device1)
            
            // Act: Login from device 2
            session2, _ := authService.Authenticate(context.Background(), userID, "password", device2)
            
            // Assert: Session 1 should be invalidated
            isValid1 := authService.ValidateSession(context.Background(), session1.Token)
            isValid2 := authService.ValidateSession(context.Background(), session2.Token)
            
            return !isValid1 && isValid2
        },
        gen.Identifier(),      // userID
        gen.Identifier(),      // device1
        gen.Identifier(),      // device2
    ))
    
    properties.TestingRun(t)
}

// Property 6: Tenant Isolation
// Feature: master-sindico-platform, Property 6: For any user and any data query, the query results should only include data where tenant_id matches the user's tenant
func TestProperty_TenantIsolation(t *testing.T) {
    parameters := gopter.DefaultTestParameters()
    parameters.MinSuccessfulTests = 100
    properties := gopter.NewProperties(parameters)
    
    properties.Property("tenant isolation", prop.ForAll(
        func(tenantID1 string, tenantID2 string, dataCount int) bool {
            if tenantID1 == tenantID2 {
                return true // Skip if same tenant
            }
            
            // Arrange
            db := setupTestDB()
            repo := NewCondominiumRepository(db)
            
            // Create data for tenant 1
            for i := 0; i < dataCount; i++ {
                condo := &Condominium{
                    ID:       fmt.Sprintf("condo-%d", i),
                    TenantID: tenantID1,
                }
                repo.Create(context.Background(), condo)
            }
            
            // Act: Query as tenant 2
            ctx := context.WithValue(context.Background(), "tenant_id", tenantID2)
            results, _ := repo.FindByTenantID(ctx, tenantID2)
            
            // Assert: Should return 0 results
            return len(results) == 0
        },
        gen.Identifier(),           // tenantID1
        gen.Identifier(),           // tenantID2
        gen.IntRange(1, 10),        // dataCount
    ))
    
    properties.TestingRun(t)
}

// Property 9: Timeline Immutability
// Feature: master-sindico-platform, Property 9: For any published Timeline entry, attempts to modify or delete the entry should be rejected
func TestProperty_TimelineImmutability(t *testing.T) {
    parameters := gopter.DefaultTestParameters()
    parameters.MinSuccessfulTests = 100
    properties := gopter.NewProperties(parameters)
    
    properties.Property("timeline immutability", prop.ForAll(
        func(title string, description string, newTitle string) bool {
            // Arrange
            repo := NewTimelineRepository(setupTestDB())
            entry := &TimelineEntry{
                ID:          uuid.New().String(),
                TenantID:    "tenant-1",
                Title:       title,
                Description: description,
                PublishedAt: time.Now(),
            }
            
            // Act: Create entry
            repo.Create(context.Background(), entry)
            
            // Act: Try to update
            entry.Title = newTitle
            updateErr := repo.Update(context.Background(), entry)
            
            // Act: Try to delete
            deleteErr := repo.Delete(context.Background(), entry.ID)
            
            // Assert: Both operations should fail
            return updateErr != nil && deleteErr != nil
        },
        gen.AlphaString(),  // title
        gen.AlphaString(),  // description
        gen.AlphaString(),  // newTitle
    ))
    
    properties.TestingRun(t)
}

// Property 25: JSON Round-Trip Preservation
// Feature: master-sindico-platform, Property 25: For any valid data structure, serializing to JSON then parsing back should produce equivalent structure
func TestProperty_JSONRoundTrip(t *testing.T) {
    parameters := gopter.DefaultTestParameters()
    parameters.MinSuccessfulTests = 100
    properties := gopter.NewProperties(parameters)
    
    properties.Property("json round-trip preservation", prop.ForAll(
        func(condo Condominium) bool {
            // Act: Serialize
            jsonBytes, err := json.Marshal(condo)
            if err != nil {
                return false
            }
            
            // Act: Deserialize
            var decoded Condominium
            err = json.Unmarshal(jsonBytes, &decoded)
            if err != nil {
                return false
            }
            
            // Assert: Should be equivalent
            return reflect.DeepEqual(condo, decoded)
        },
        genCondominium(), // Custom generator
    ))
    
    properties.TestingRun(t)
}

// Custom generator for Condominium
func genCondominium() gopter.Gen {
    return gopter.CombineGens(
        gen.Identifier(),
        gen.Identifier(),
        gen.AlphaString(),
        gen.IntRange(10, 500),
    ).Map(func(values []interface{}) Condominium {
        return Condominium{
            ID:         values[0].(string),
            TenantID:   values[1].(string),
            Name:       values[2].(string),
            TotalUnits: values[3].(int),
            Status:     CondominiumStatusActive,
        }
    })
}
```


### Integration Testing

#### Database Integration Tests

```go
func setupTestDB(t *testing.T) *gorm.DB {
    dsn := "host=localhost user=test password=test dbname=test_db port=5432 sslmode=disable"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    require.NoError(t, err)
    
    // Run migrations
    db.AutoMigrate(&Condominium{}, &Unit{}, &TimelineEntry{})
    
    // Clean up after test
    t.Cleanup(func() {
        db.Exec("TRUNCATE TABLE condominiums, units, timeline_entries CASCADE")
    })
    
    return db
}
```

#### External Service Integration Tests

```go
func TestMercadoPagoAdapter_CreatePayment(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }
    
    // Arrange
    adapter := NewMercadoPagoAdapter(testConfig)
    payment := &PaymentIntent{
        Amount:      100.00,
        Currency:    "BRL",
        Description: "Test payment",
    }
    
    // Act
    result, err := adapter.CreatePayment(context.Background(), payment)
    
    // Assert
    assert.NoError(t, err)
    assert.NotEmpty(t, result.TransactionID)
    assert.Equal(t, PaymentStatusPending, result.Status)
}
```

### End-to-End Testing

#### Critical User Flows

```go
func TestE2E_SindicoPublishesTimeline(t *testing.T) {
    // Setup
    app := setupTestApp(t)
    sindico := createTestSindico(t, app)
    condo := createTestCondominium(t, app, sindico.ID)
    
    // Login
    token := loginUser(t, app, sindico.Email, "password")
    
    // Create Timeline Entry
    entry := TimelineEntryRequest{
        Title:       "Test Entry",
        Description: "Test Description",
        EntryType:   "atividade_da_gestao",
        Date:        time.Now(),
    }
    
    resp := app.POST("/api/timeline").
        WithHeader("Authorization", "Bearer "+token).
        WithJSON(entry).
        Expect().
        Status(http.StatusCreated).
        JSON().Object()
    
    entryID := resp.Value("id").String().Raw()
    
    // Verify Timeline Entry is visible to moradores
    morador := createTestMorador(t, app, condo.ID)
    moradorToken := loginUser(t, app, morador.Email, "password")
    
    app.GET("/api/timeline/"+entryID).
        WithHeader("Authorization", "Bearer "+moradorToken).
        Expect().
        Status(http.StatusOK).
        JSON().Object().
        ValueEqual("title", "Test Entry")
}
```

### Test Coverage Goals

- **Unit Tests**: 80%+ code coverage
- **Integration Tests**: All repository implementations, all external adapters
- **Property Tests**: All 29 correctness properties
- **E2E Tests**: Top 10 critical user flows

### Test Organization

```
/tests
  /unit
    /domain
      condominium_test.go
      timeline_test.go
      assembly_test.go
    /application
      create_condominium_usecase_test.go
      publish_timeline_usecase_test.go
    /infrastructure
      condominium_repository_test.go
      timeline_repository_test.go
  /integration
    /repositories
      condominium_repository_integration_test.go
    /adapters
      mercadopago_adapter_integration_test.go
      mux_adapter_integration_test.go
  /properties
    authentication_properties_test.go
    timeline_properties_test.go
    assembly_properties_test.go
    serialization_properties_test.go
  /e2e
    sindico_journey_test.go
    morador_journey_test.go
    company_journey_test.go
    assembly_flow_test.go
```

### Continuous Integration

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - name: Run unit tests
        run: go test -v -short ./tests/unit/...
      
  property-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - name: Run property tests
        run: go test -v ./tests/properties/...
  
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: '1.21'
      - name: Run integration tests
        run: go test -v ./tests/integration/...
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/test_db
```

## Implementation Roadmap

### Phase 1: Foundation (Sprints 1-3)

**Sprint 1: Core Infrastructure**
- Setup Go project structure (Clean Architecture)
- Configure GORM with PostgreSQL
- Implement multi-tenancy middleware
- Setup Zitadel integration for authentication
- Implement basic ABAC authorization
- Setup NATS event bus
- Configure Redis cache

**Sprint 2: Identity Context**
- User registration and authentication
- Device fingerprinting
- Session management
- Subscription management
- Plan-based feature access
- Trial module implementation

**Sprint 3: Institutional Context - Part 1**
- Condominium registry
- Mandate management
- Unit registry
- Síndico profile
- Company profile
- Partner profile

### Phase 2: Core Features (Sprints 4-7)

**Sprint 4: Institutional Context - Part 2**
- User management (condominium)
- Morador vinculo management
- Empresa síndica delegation
- Context confirmation (anti-error)

**Sprint 5: Content Context**
- Timeline (append-only, immutable)
- Master Plan (reactive status)
- Events
- Comunicados
- Adendos

**Sprint 6: Commercial Context - Part 1**
- Connect Me (Síndico → Company)
- Connect Me (Morador → Síndico)
- Connect Me (Company → Company)
- Connect Me (Síndico → Partner)
- LGPD logging

**Sprint 7: Commercial Context - Part 2**
- Proposal management
- Supplier voting
- Closed deals
- Service execution registry
- Post-deal comunicados

### Phase 3: Advanced Features (Sprints 8-11)

**Sprint 8: Compliance Context**
- Audit trail (append-only)
- Declarations (annual)
- LGPD consent management
- Validation hub
- Compliance dashboard

**Sprint 9: Neighborhood Context**
- Partner management
- Promotion creation
- Benefit activation
- Metrics and history
- Multi-journey support (Síndico, Empresa, Morador, Parceiro)

**Sprint 10: Analytics Context**
- Dashboard indicators
- Report generation
- Metrics calculation
- Transparency indicators

**Sprint 11: Content Context - Advanced**
- Morador questions
- Morador evaluations
- Company evaluations
- Voice input support
- QR code generation

### Phase 4: Assembly Module (Sprints 12-15)

**Sprint 12: Assembly Creation and Setup**
- Assembly creation
- Agenda management
- Convocation
- Institutional participants
- Quorum configuration

**Sprint 13: Assembly Execution**
- Check-in (app, QR, terminal)
- Proxy management
- Speech queue
- Telão (big screen)
- President controls

**Sprint 14: Assembly Voting**
- Vote casting (app, presencial assistido)
- Proxy vote replication
- Fração ideal weighting
- Real-time results
- Vote homologation

**Sprint 15: Assembly Finalization**
- Assembly finalization
- Report generation
- Transparency indicator
- Minutes (ata) generation
- Timeline publication

### Phase 5: External Integrations (Sprint 16)

**Sprint 16: External Services**
- Mercado Pago payment integration
- AWS SES email service
- Twilio/Zenvia SMS service
- Mux video streaming
- AWS S3 storage
- ViaCEP lookup
- Firebase push notifications

### Phase 6: Polish and Optimization (Sprints 17-18)

**Sprint 17: Performance Optimization**
- Database query optimization
- Caching strategy implementation
- Asynchronous job processing
- Rate limiting
- Circuit breakers

**Sprint 18: Security Hardening**
- Data encryption at rest
- Data encryption in transit
- Input validation and sanitization
- Security audit
- Penetration testing

### Phase 7: Testing and Documentation (Sprint 19-20)

**Sprint 19: Comprehensive Testing**
- Complete unit test suite
- Property-based tests for all 29 properties
- Integration tests
- E2E tests for critical flows

**Sprint 20: Documentation and Deployment**
- API documentation (OpenAPI/Swagger)
- Developer documentation
- Deployment automation
- Monitoring and observability setup
- Production launch

## Deployment Architecture

### Infrastructure Components

```
┌─────────────────────────────────────────────────────────────────┐
│                         Load Balancer                            │
│                         (AWS ALB / Nginx)                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      API Servers (Go)                            │
│                      (Auto-scaling group)                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Server 1 │  │ Server 2 │  │ Server 3 │  │ Server N │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  PostgreSQL  │  │    Redis     │  │     NATS     │         │
│  │  (Primary +  │  │   (Cache)    │  │ (Event Bus)  │         │
│  │   Replicas)  │  │              │  │              │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   External Services                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Zitadel  │  │   Mux    │  │   S3     │  │Mercado   │       │
│  │  (Auth)  │  │ (Video)  │  │(Storage) │  │  Pago    │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

### Deployment Strategy

- **Container Orchestration**: Kubernetes or Docker Swarm
- **CI/CD**: GitHub Actions or GitLab CI
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Tracing**: Jaeger or Zipkin
- **Secrets Management**: HashiCorp Vault or AWS Secrets Manager

### Scalability Considerations

- **Horizontal Scaling**: API servers can scale horizontally behind load balancer
- **Database**: PostgreSQL with read replicas for read-heavy operations
- **Cache**: Redis cluster for distributed caching
- **Event Bus**: NATS cluster for high availability
- **Storage**: S3 for unlimited file storage
- **CDN**: CloudFront or Cloudflare for static assets and video streaming

## Conclusion

Este design document fornece um blueprint técnico completo para a implementação da Master Síndico Platform em Go. A arquitetura proposta:

- Implementa Clean Architecture com separação clara de responsabilidades
- Utiliza Domain-Driven Design com 8 bounded contexts bem definidos
- Garante multi-tenancy seguro com isolamento estrito de dados
- Implementa event-driven architecture para comunicação assíncrona
- Assegura compliance LGPD com audit trail imutável
- Fornece 29 correctness properties para validação via property-based testing
- Define estratégia de testes abrangente (unit, integration, property, e2e)
- Estabelece roadmap de implementação em 20 sprints

O sistema está preparado para escalar horizontalmente, suportar 1000+ usuários concorrentes no módulo de Assembly, e manter 5 anos de retenção de dados para compliance.

