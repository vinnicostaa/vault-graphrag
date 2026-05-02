---
title: Context Map — Master Síndico
type: spec
tags: [domain, ddd, context-map, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/01-domain/bounded-contexts.md + legacy-reference.md
created: 2026-04-23
updated: 2026-04-23
---

# Context Map — Master Síndico v2

Mapa de relações entre bounded contexts (BCs) no estilo DDD. Classifica cada relação por: **tipo** (Customer/Supplier, Shared Kernel, ACL — Anti-Corruption Layer, Published Language, Partnership, Conformist, Big Ball of Mud), **direção** (upstream U → downstream D) e **eventos/interfaces** cruzados.

**Fonte primária**: destilação do legado `vault/50-projects/master-sindico/01-domain/bounded-contexts.md` §"Mapa de Contextos" + specs de requirements.

**Regra-mãe** (STEERING §14): BCs **nunca** fazem import direto de `domain/` ou `application/` de outro BC. Todas as relações abaixo são realizadas via **domain events** (event bus) ou `shared/types/` (primitivos como `AuthUser`, `TenantID`).

---

## Legenda de tipos de relação

| Tipo | Significado | Sinal |
|---|---|---|
| **Customer/Supplier (U/D)** | Upstream tem obrigação de estabilidade; Downstream adapta ao upstream. | ↠ |
| **Shared Kernel** | Subconjunto de modelo compartilhado (zero-override). Restringe autonomia. Usar só se absolutamente necessário. | ═ |
| **Conformist** | Downstream se submete ao modelo upstream sem tradução. | ⤵ |
| **ACL (Anti-Corruption Layer)** | Downstream traduz o modelo upstream pra não corromper o seu. | ↹ |
| **Published Language** | Interface bem documentada e versionada (ex: OpenAPI, CloudEvents). | 📢 |
| **Partnership** | Ambos dependem mutuamente; evolução coordenada. | ↔ |
| **Separate Ways** | Sem integração (problema resolvido isoladamente). | — |

---

## Matriz de relações (7×7; `cross-domain` à parte)

Legenda na primeira linha. Ler: **linha → coluna** (linha é o BC de partida; coluna é o BC de destino).

| De ↓ / Para → | identity | billing | institutional | commercial | content | assembly | cross-domain |
|---|---|---|---|---|---|---|---|
| **identity** | — | ↠ U/D (events) | ↠ U/D (events) | ↠ U/D (events) | ↠ U/D (events) | ↠ U/D (events) | ═ Shared Kernel (AuthUser) |
| **billing** | ⤵ Conformist (UserCreated) | — | ↠ U/D (feature-gate) | ↠ U/D (feature-gate) | ↠ U/D (feature-gate) | ↠ U/D (feature-gate) | ═ Shared Kernel (Money) |
| **institutional** | — | ⤵ Conformist (Subscription) | — | ↔ Partnership | ↔ Partnership (via grants) | ↔ Partnership | ═ Shared Kernel (CondominiumID) |
| **commercial** | — | ⤵ Conformist (quota) | ↔ Partnership (Membership, Timeline) | — | ↠ U/D (publica vídeo → consome mod) | ⤵ Conformist (SupplierVote em Assembly) | 📢 Published Language (events) |
| **content** | — | ⤵ Conformist (quota) | — | ↔ Partnership (vídeo empresa ↔ reputação) | — | ↔ Partnership (VideoVisibilityGrant durante votação) | 📢 Published Language |
| **assembly** | — | ⤵ Conformist (plan live) | ↔ Partnership (Condo, Unit, Membership) | ⤵ Conformist (proposals validadas) | ↔ Partnership (visibility grant) | — | 📢 Published Language |
| **cross-domain** | ↹ ACL (ABAC cache) | ↹ ACL (Stripe webhook → invalidate ABAC) | — | — | — | — | — |

---

## Relações detalhadas

### R-01. `identity` ↠ `billing` (Customer/Supplier)

- **Direção**: `identity` é upstream; `billing` é downstream.
- **Gatilho**: `UserCreated` evento.
- **Ação**: `billing` assina; cria `Subscription` em trial persona-aware (BR-001, Req 4.1).
- **Interface**: Domain event `UserCreated { user_id, role, persona, created_at }`.
- **Tipo de comunicação**: assíncrona (in-process event bus; NATS quando escalar).
- **ACL necessário**: não — payload já em linguagem ubíqua.

### R-02. `identity` ↠ `institutional` (Customer/Supplier)

- **Direção**: `identity` upstream; `institutional` downstream.
- **Gatilho**: `UserCreated` → habilita `Membership`; `UserUpdated` → atualiza cache ABAC.
- **Interface**: eventos `UserCreated`, `UserUpdated`, `UserDeleted`.
- **Observação**: `institutional` referencia `user_id` por ID, nunca por ponteiro (Rule 1 legado).

### R-03. `identity` ↠ `commercial` (Customer/Supplier)

- **Direção**: `identity` upstream; `commercial` downstream.
- **Gatilho**: `UserCreated` → permite criar `Company` com `admin_user_id`; `MembershipCreated` (de institutional) → atualiza cache ABAC em commercial.
- **Interface**: eventos `UserCreated`, `MembershipCreated`.

### R-04. `identity` ↠ `content`, `assembly` (Customer/Supplier)

- **Direção**: `identity` upstream.
- **Gatilho**: eventos do User (sync role, invalidate session).
- **Interface**: `UserCreated`, `SessionInvalidated`.

### R-05. `billing` ↠ `institutional` (Customer/Supplier — feature gate)

- **Direção**: `billing` upstream; `institutional` downstream.
- **Gatilho**: `SubscriptionActivated` / `SubscriptionExpired` / `PlanChanged`.
- **Ação**: `institutional` libera/trava features premium (criar Timeline, comunicado, exportar relatório).
- **Interface**: event payload inclui `plan_tier`, `trial_ends_at`, `features: {...}`.
- **ACL**: não necessário — `plan_tier` é enum compartilhado via `shared/types/`.

### R-06. `billing` ↠ `commercial`, `content`, `assembly` (Customer/Supplier)

- **Direção**: `billing` upstream; demais downstream.
- **Gatilho**: mesmos eventos do R-05.
- **Ação**: libera/trava Connect Me, upload vídeo, sessão live por plan_tier.

### R-07. `institutional` ↔ `commercial` (Partnership)

- **Direção**: bilateral.
- **Gatilhos**:
  - **institutional → commercial**: `MembershipCreated` habilita ABAC cache; `CondominiumCreated` permite `ConnectMeRequest` do síndico
  - **commercial → institutional**: `ClosedDealSigned` → auto-publica `TimelineEntry` tipo `registro_execucao`; `ExecutionValidated` → atualiza status atividade em `MasterPlan`
- **Tipo**: Partnership — evolução coordenada porque Connect Me referencia `CondominiumID` + `MembershipID`; e `MasterPlan` reflete eventos de execução contratual.
- **Interface**: eventos `MembershipCreated`, `ClosedDealSigned`, `ExecutionValidated`, `TimelineEntryAdded`.

### R-08. `institutional` ↔ `assembly` (Partnership)

- **Direção**: bilateral.
- **Gatilhos**:
  - **institutional → assembly**: `CondominiumCreated`, `UnitRegistered`, `MembershipCreated` (habilitam agendar assembleia e ter participantes)
  - **assembly → institutional**: `MinutesPublished` → cria `TimelineEntry` tipo `assembly_result`
- **Tipo**: Partnership — assembleia não existe sem condomínio + unidades + moradores.
- **Interface**: eventos `CondominiumCreated`, `UnitRegistered`, `MembershipCreated`, `MinutesPublished`, `TimelineEntryAdded`.

### R-09. `commercial` ↔ `content` (Partnership)

- **Direção**: bilateral (cada lado tem obrigação).
- **Gatilhos**:
  - **commercial → content**: `CompanyApproved` → libera upload de vídeo empresa; `CompanyEvaluationSubmitted` → pode referenciar vídeo
  - **content → commercial**: `VideoPublished` com tipo institucional empresa → input pra fórmula de Reputação (Platina+ requer ≥ 1 vídeo aprovado); `VideoRemoved` → ajusta Reputação
- **Tipo**: Partnership.
- **Interface**: eventos `CompanyApproved`, `VideoPublished`, `VideoRemoved`.

### R-10. `content` ↔ `assembly` (Partnership — visibility grant)

- **Direção**: bilateral, crítica.
- **Gatilhos**:
  - **assembly → content**: `VotingOpened` (tipo `votacao_fornecedor`) com propostas de empresa e `autorizar_exibicao_votacao=true` → cria `VideoVisibilityGrant` (morador pode ver vídeo empresa apenas durante a votação — Rule 4, Req 103)
  - **assembly → content**: `VotingClosed` → revoga `VideoVisibilityGrant` (auto-revoke)
- **Tipo**: Partnership — obrigação mútua: `content` **deve** respeitar grants emitidos por `assembly`; `assembly` **deve** emitir grants no `VotingOpened`.
- **Interface**: eventos `VotingOpened`, `VotingClosed`; aggregate `VideoVisibilityGrant` em `content`.

### R-11. `commercial` ⤵ `assembly` (Conformist — SupplierVote dentro de Assembly)

- **Direção**: `assembly` upstream; `commercial` downstream (conformista).
- **Gatilho**: `SupplierVoteSession` criada no contexto de uma `Assembly` (Req 21). `VotingClosed` → consolida resultado pra `ClosedDeal`.
- **Tipo**: Conformist — `commercial` se submete à estrutura de votação do `assembly` (peso fração ideal, quórum, imutabilidade).
- **Interface**: eventos `VotingOpened`, `VoteCast`, `VotingClosed`; referência `agenda_item_id` → `supplier_vote_session_id`.

### R-12. `cross-domain` ═ Shared Kernel com todos

- **Elementos compartilhados**:
  - `shared/types/AuthUser` (user_id, role, tenant_id, plan_tier)
  - `shared/types/TenantID` (`condominium_id` ou `company_id`)
  - `shared/types/Money` (int64 centavos BRL — T10)
  - `shared/types/FracaoIdeal` (NUMERIC 4 casas — T10)
  - `core/contracts/IDomainEvent`, `core/contracts/IUseCase`, `core/contracts/IRepository`, `core/contracts/IUnitOfWork`, `core/contracts/HTTPRouter`, `core/contracts/HandlerFunc`, `core/contracts/Context`
  - `core/errors/AppError` + sentinels (`ErrValidation`, `ErrNotFound`, `ErrConflict`, `ErrForbidden`, `ErrUnauthorized`, `ErrInternal`)
- **Justificativa**: permitir que middlewares cross-BC (auth, rate limit, logger, error handler) operem sem importar módulos.
- **Regra**: `shared/middleware/` importa apenas `shared/types/` — **nunca** `modules/` (A2 antipattern legado).

### R-13. `cross-domain` ↹ ACL pra integrações externas

- **Stripe webhook → `billing`**: HMAC verify antes de parse (DR-020). Translator transforma payload Stripe em domain event `SubscriptionActivated`/etc.
- **Mux webhook → `content`**: idem (`video.ready`, `video.errored`).
- **Livekit webhook → `assembly`**: idem (`room_started`, `room_ended`).
- **Zitadel introspection → `identity`**: cache Redis `ms:auth:{zitadelUserID}` TTL 5min (T13).

### R-14. `cross-domain` ↹ ABAC cache (invalidation cross-BC)

- **Gatilho**: eventos de mudança de `plan_tier` (`SubscriptionActivated`, `PlanChanged`, `SubscriptionExpired`); mudança de Membership (`MembershipCreated`, `MembershipEnded`); mudança de role no Zitadel.
- **Ação**: `cross-domain/abac_cache.go` invalida key `abac:{user_id}:{tenant_id}`.
- **Tipo**: ACL — cross-domain traduz eventos de vários BCs em uma operação cache.

---

## 📢 Published Language — catálogo de eventos

Os BCs publicam eventos com **payload estável, versionado** (SemVer). Catálogo canônico: [[domain-events]].

**Regras do event bus**:

1. Evento tem UUIDv7 ordenável (DR-017, IR-007).
2. Payload imutável — alteração breaking = novo evento `v2` (deprecation 6m).
3. Consumidores **idempotentes** — reentrega não duplica side-effect (outbox pattern pra garantir atomicidade entre commit DB + publish event).
4. Transaction boundary: evento publicado **dentro** da mesma UoW/tx via tabela `domain_events` (outbox commit-ado junto). Publisher assíncrono lê outbox e envia ao bus.

---

## ═ Shared Kernel — lista exaustiva

### Tipos primitivos compartilhados

```
shared/types/AuthUser          {id, email, role, role_type, tenant_id, plan_tier}
shared/types/TenantID          (UUID) — ambíguo entre condominium e company; prefixar no código
shared/types/Money             (int64 cents, currency="BRL") — T10
shared/types/FracaoIdeal       (NUMERIC(5,4))
shared/types/UUID              UUIDv7
shared/types/Timestamp         time.Time UTC

core/contracts/IDomainEvent    {ID, Type, AggregateID, OccurredAt, Payload}
core/contracts/IUseCase[Cmd,R] Execute(ctx, cmd) (R, error)
core/contracts/IRepository[T]  Save(ctx,T) error; FindByID(ctx,id) (T,error); ...
core/contracts/IUnitOfWork     Run(ctx, fn func(ctx)error) error
core/contracts/HTTPRouter      abstração framework-agnostic (Gin/etc)

core/errors/AppError           {Code, Message, Err, Fields}
core/errors/sentinels          ErrValidation, ErrNotFound, ErrConflict, ErrForbidden, ErrUnauthorized, ErrInternal, ErrBusiness
```

### Enums compartilhados (via `shared/types/`)

```
Role: syndic | resident | enterprise | marketing | local_company | admin
RoleType: principal | auxiliary | assistant | titular | dependent | administrator | operator
PlanTier: trial | base | plus | pro
```

### Políticas ABAC (enforcement em `cross-domain/abac/`)

- `policies.yaml` — matriz `(resource_type, action) → allowed_roles, plan_tier_min`.
- `engine.go` — ordem: admin_bypass → action_configured → role_allowed → tenant_matches → plan_tier_sufficient.

---

## Proibições (não fazer)

- ❌ **identity** importar `billing/domain/*` para ler `PlanTier` — use `shared/types/PlanTier`.
- ❌ **commercial** chamar `institutional.GetCondominium(id)` diretamente — use evento `CondominiumCreated` pra construir projeção local.
- ❌ **content** consultar tabela `subscriptions` de `billing` — use cache ABAC decisão.
- ❌ **assembly** escrever em `timeline_entries` diretamente — emita `AssemblyClosed` + `MinutesPublished`; `institutional` assina e cria entry.
- ❌ **cross-domain** importar `modules/X/application/*` — trabalha só com `shared/types/` + event bus + providers.

---

## Grafo simplificado (texto)

```
identity (upstream absoluto)
   │
   ├─ UserCreated ──→ billing (cria Subscription trial)
   │                     │
   │                     ├─ SubscriptionActivated ──→ institutional
   │                     ├─ SubscriptionActivated ──→ commercial
   │                     ├─ SubscriptionActivated ──→ content
   │                     └─ SubscriptionActivated ──→ assembly
   │
   ├─ UserCreated ──→ institutional (habilita Membership)
   │                     │
   │                     ├─ MembershipCreated ───→ commercial (ABAC cache)
   │                     ├─ CondominiumCreated ──→ assembly (pré-req)
   │                     ├─ CondominiumCreated ──→ commercial (pré-req)
   │                     └─ TimelineEntryAdded ──→ content (indexação busca)
   │
   └─ UserCreated ──→ commercial/content/assembly (idem)

commercial ↔ content:
   ├─ commercial: CompanyApproved ────→ content: permite upload
   └─ content:    VideoPublished ─────→ commercial: alimenta Reputação

assembly ↔ content:
   ├─ assembly: VotingOpened(fornecedor) ──→ content: cria VideoVisibilityGrant
   └─ assembly: VotingClosed             ──→ content: revoga grant

commercial ↔ assembly:
   ├─ commercial: ProposalValidated ───→ assembly: habilita votação fornecedor (Req 21)
   └─ assembly:   SupplierVoteClosed ──→ commercial: consolida ClosedDeal

commercial → institutional:
   ├─ ClosedDealSigned      ──→ TimelineEntry tipo registro_execucao (auto-publish)
   ├─ ExecutionValidated    ──→ atualiza status atividade MasterPlan

assembly → institutional:
   └─ MinutesPublished ──→ TimelineEntry tipo assembly_result

cross-domain:
   ├─ Captura eventos de TODOS ──→ AuditEntry + LGPDLog append-only
   ├─ Stripe webhook          ──→ invalidate ABAC cache (Redis)
   └─ GovernanceScore job noturno lê eventos ──→ calcula score do dia
```

---

## ⚠️ Pendências de dual-check (Fase 5)

- **P-CM-001**: hoje event bus é in-process. Legado mantém NATS JetStream como opção (D-0XX). Pra M2+, confirmar topologia (in-process suficiente? particionar por BC? outbox pattern obrigatório?).
- **P-CM-002**: confirmar que **não** há shared DB schema cross-BC (cada BC owner do próprio schema; tabelas sem FKs cross-schema). Legado respeita; v2 manter.
- **P-CM-003**: `TenantID` é ambíguo entre `condominium_id` e `company_id`. Confirmar se vale criar tipos distintos `CondominiumID`, `CompanyID` pra evitar mismatch runtime.
- **P-CM-004**: Saga compensation (A-027, A-028 legado) hoje é síncrona. Confirmar se M2+ precisa saga orchestrator (ex: `saga.Orchestrator` em `cross-domain`).
- **P-CM-005**: R-11 `commercial ⤵ assembly` via SupplierVote: confirmar se `commercial` **consome** resultado (via event `SupplierVoteClosed`) ou **reifica** no próprio BC (event + own aggregate `SupplierVoteResult`).

---

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[ubiquitous-language]]
- [[invariants]]
- [[domain-events]]
- [[business-rules]]
- [[aggregates/_moc]]
- [[../STEERING#princípios-de-arquitetura]]
- [[../CLAUDE#3-padrões-herdados-inegociáveis]]
- [[../00-product/portfolio-de-produtos#dependências-inter-sub-produto]]
