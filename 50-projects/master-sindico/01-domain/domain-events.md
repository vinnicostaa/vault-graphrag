---
title: Domain Events — Master Síndico (catálogo cross-BC)
type: spec
tags: [domain, ddd, domain-events, event-bus, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/01-domain/bounded-contexts.md + specs/requirements/*
created: 2026-04-23
updated: 2026-04-23
---

# Domain Events — catálogo cross-BC

Catálogo canônico dos eventos de domínio que cruzam bounded contexts. **Regra**: BCs não importam `domain/` de outros BCs; comunicação acontece via eventos com payload estável (Published Language — ver [[context-map]]).

**Invariantes do event bus** (herdados):

- **INV-094**: `event.id` é UUIDv7 ordenável (DR-017)
- **INV-095**: Events publicados dentro da mesma tx via outbox pattern (`domain_events` table)
- Payload imutável — alteração breaking = novo evento `v2` (deprecation 6m)
- Consumidores idempotentes — reentrega não duplica side-effect
- **Transport**: in-process event bus hoje; NATS JetStream quando escalar (Sprint 10+)

**Naming**: `<Aggregate><PastTense>` — ex: `UserCreated`, `SubscriptionActivated`, `VoteCast`. pt-BR só quando semântica jurídica BR é única (`EditalPublished`).

**Schema comum (todo evento)**:

```go
type EventEnvelope struct {
    ID          UUIDv7
    Type        string    // "identity.user.created" (namespace.aggregate.action)
    Version     string    // "v1"
    AggregateID string
    TenantID    string    // quando aplicável
    OccurredAt  time.Time // UTC
    ActorID     string    // user que causou (se aplicável)
    Payload     json.RawMessage
}
```

---

## identity

### E-001. `UserCreated`

- **Publisher**: `identity`
- **Quando**: primeiro login via Zitadel; UPSERT em `users` com `zitadel_id`
- **Payload**:
  ```json
  { "user_id": "uuid", "zitadel_id": "...", "email_masked": "a***@domain.com", "role": "syndic|resident|enterprise|marketing|local_company|admin", "persona_detected": "sindico|morador|empresa|parceiro", "user_type": "PF|PJ", "created_at": "..." }
  ```
- **Subscribers**:
  - `billing` → cria `Subscription` em trial persona-aware
  - `institutional` → habilita onboarding (`pending` Membership, Profile)
  - `cross-domain` → audit entry + rate limit reset
- **Side-effects**: cria `Subscription.status=trial`, `trial_ends_at = NOW() + trial_duration(persona)`; fan-out notification boas-vindas via Resend.

### E-002. `UserUpdated`

- **Publisher**: `identity`
- **Quando**: dados do user mudaram (email, phone, name) via Zitadel ou app
- **Payload**: `{ user_id, fields_changed: [...], changed_at }`
- **Subscribers**: `cross-domain` (invalida cache ABAC); `institutional` (sync Profile se aplicável).

### E-003. `UserDeleted`

- **Publisher**: `identity`
- **Quando**: hard delete pós-LGPD request ou job de sync com Zitadel
- **Payload**: `{ user_id, deleted_at, reason }`
- **Subscribers**: todos (soft archive dados históricos, preserva votos/avaliações/minutes com pseudonimização).

### E-004. `SessionCreated`

- **Publisher**: `identity`
- **Quando**: novo login com device fingerprint
- **Payload**: `{ session_id, user_id, device_fingerprint_hash, ip, user_agent, created_at }`
- **Subscribers**: `cross-domain` (audit log).

### E-005. `SessionInvalidated`

- **Publisher**: `identity`
- **Quando**: logout, troca de device (1-device policy), admin revoga
- **Payload**: `{ session_id, user_id, reason: "logout|other_device|admin_revoke|expired|zitadel_revoked", invalidated_at }`
- **Subscribers**: `cross-domain` (audit); frontend (force logout via WebSocket).

### E-006. `DeviceChanged` (Sprint 10+)

- **Publisher**: `identity`
- **Quando**: login em device diferente do anterior (A-023 fix)
- **Payload**: `{ user_id, previous_device_hash, new_device_hash, changed_at, ip }`
- **Subscribers**: `cross-domain` (audit + antifraude); frontend (notification opt-in).

---

## billing

### E-007. `SubscriptionCreated` / `TrialStarted`

- **Publisher**: `billing`
- **Quando**: UPSERT de Subscription no primeiro login (via handler de `UserCreated`)
- **Payload**: `{ subscription_id, user_id, plan_id, plan_slug, trial_started_at, trial_ends_at, status: "trial" }`
- **Subscribers**: `institutional`, `commercial`, `content`, `assembly` (cache ABAC + feature-gate); `cross-domain` (audit, métricas produto).

### E-008. `SubscriptionActivated`

- **Publisher**: `billing`
- **Quando**: Stripe webhook `payment_intent.succeeded` / `subscription.created` com pagamento ok
- **Payload**: `{ subscription_id, user_id, plan_id, plan_tier, period_start, period_end, activated_at }`
- **Subscribers**: `institutional`, `commercial`, `content`, `assembly` (desbloqueia features); `cross-domain` (invalida cache ABAC).

### E-009. `PlanChanged`

- **Publisher**: `billing`
- **Quando**: upgrade/downgrade via Stripe Customer Portal
- **Payload**: `{ subscription_id, user_id, previous_plan_id, new_plan_id, effective_at, prorated: bool }`
- **Subscribers**: todos os BCs downstream (quota reset, feature gate refresh); `cross-domain` (invalida cache ABAC).

### E-010. `TrialExpiringSoon`

- **Publisher**: `billing`
- **Quando**: job diário detecta trial com `trial_ends_at - NOW() < 24h`
- **Payload**: `{ subscription_id, user_id, trial_ends_at, hours_remaining }`
- **Subscribers**: `cross-domain` (notification via Resend/FCM).

### E-011. `TrialExpired` / `SubscriptionExpired`

- **Publisher**: `billing`
- **Quando**: `NOW() >= trial_ends_at` ou `subscription.deleted` do Stripe
- **Payload**: `{ subscription_id, user_id, expired_at, reason: "trial_ended|canceled|past_due" }`
- **Subscribers**: `institutional`, `commercial`, `content`, `assembly` (soft-block features premium); `cross-domain` (invalida cache ABAC, notification).

### E-012. `SubscriptionCanceled`

- **Publisher**: `billing`
- **Quando**: webhook `customer.subscription.deleted` ou cancel direto
- **Payload**: `{ subscription_id, user_id, canceled_at, reason }`
- **Subscribers**: igual ao E-011.

### E-013. `QuotaExhausted`

- **Publisher**: `billing`
- **Quando**: tentativa de consumir quota com saldo 0 (ou near-zero)
- **Payload**: `{ user_id, feature_key: "connect_me|video_upload_institutional|...", period: "annual|monthly", limit, used, reset_at }`
- **Subscribers**: `commercial`/`content` (bloqueia ação); frontend (sugere upgrade).

---

## institutional

### E-014. `CondominiumCreated`

- **Publisher**: `institutional`
- **Quando**: síndico conclui onboarding condomínio
- **Payload**: `{ condominium_id, public_id, name, cnpj_masked, total_units, created_by, mandate_type, created_at }`
- **Subscribers**: `commercial` (permite abrir Connect Me do síndico), `assembly` (habilita agendar assembleia), `cross-domain` (audit).

### E-015. `UnitRegistered`

- **Publisher**: `institutional`
- **Quando**: unidade adicionada ao condomínio (carga inicial ou retroativa)
- **Payload**: `{ unit_id, condominium_id, unit_identifier, unit_type, fracao_ideal, titular_id, registered_at }`
- **Subscribers**: `assembly` (atualiza total de frações elegíveis).

### E-016. `UnitUpdated`

- **Publisher**: `institutional`
- **Quando**: fração ideal mudou (raro; via adendo), titular trocou, dependente add/remove
- **Payload**: `{ unit_id, fields_changed, updated_at }`
- **Subscribers**: `assembly` (se houver assembleia ativa, triggers alerta).

### E-017. `MembershipCreated`

- **Publisher**: `institutional`
- **Quando**: user vinculado a condomínio+unidade com role
- **Payload**: `{ membership_id, user_id, condominium_id, unit_id, role: "sindico|morador|subsindico|conselho", role_type, start_date, created_at }`
- **Subscribers**: `cross-domain` (invalida ABAC cache); `commercial` (permite operar Connect Me); `assembly` (quem pode participar).

### E-018. `MembershipEnded`

- **Publisher**: `institutional`
- **Quando**: saída de morador, fim de mandato, revogação
- **Payload**: `{ membership_id, user_id, condominium_id, ended_at, reason }`
- **Subscribers**: `cross-domain` (invalida ABAC cache).

### E-019. `TimelineEntryAdded`

- **Publisher**: `institutional`
- **Quando**: nova entry append-only (comunicado, deal fechado, assembleia, etc)
- **Payload**: `{ entry_id, condominium_id, entry_type: "comunicado|evento|assembleia_result|registro_execucao|mandato_iniciado|mandato_encerrado|adendo|outro", author_id, title, published_at, corrects_entry_id? }`
- **Subscribers**: `content` (indexa em search), `cross-domain` (audit).

### E-020. `AnnouncementPublished`

- **Publisher**: `institutional`
- **Quando**: comunicado oficial publicado (transição `draft → published`; imutável pós)
- **Payload**: `{ announcement_id, condominium_id, type, title, author_id, published_at }`
- **Subscribers**: `cross-domain` (notification fan-out FCM/Resend).

### E-021. `EventScheduled`

- **Publisher**: `institutional`
- **Quando**: evento condominial criado (reunião, obra, festa)
- **Payload**: `{ event_id, condominium_id, type, title, scheduled_at, created_at }`
- **Subscribers**: `cross-domain` (notification).

### E-022. `PollOpened` / `PollClosed`

- **Publisher**: `institutional`
- **Quando**: enquete informal abre/fecha
- **Payload**: `{ poll_id, condominium_id, type, opened_at|closed_at, results? }`
- **Subscribers**: `cross-domain` (notification).

### E-023. `MandateStarted`

- **Publisher**: `institutional`
- **Quando**: novo mandato de síndico começa
- **Payload**: `{ mandate_id, condominium_id, sindico_id, mandate_type, start_date, end_date }`
- **Subscribers**: `commercial` (permite Connect Me; reset governance score); `cross-domain` (audit).

### E-024. `MandateTransferred` / `MandateEnded`

- **Publisher**: `institutional`
- **Quando**: mandato muda dono ou termina
- **Payload**: `{ mandate_id, previous_sindico_id, new_sindico_id?, ended_at, reason }`
- **Subscribers**: `cross-domain` (audit, dossiê PDF Req 40).

### E-025. `MasterPlanActivityAdvanced`

- **Publisher**: `institutional`
- **Quando**: atividade do Plano Diretor muda status (via evento `ExecutionValidated` de commercial)
- **Payload**: `{ activity_id, master_plan_id, previous_status, new_status, changed_at }`
- **Subscribers**: `cross-domain` (governance score).

### E-026. `ComplianceDeclarationSubmitted`

- **Publisher**: `institutional`
- **Quando**: síndico submete declaração anual
- **Payload**: `{ declaration_id, condominium_id, year, submitted_by, submitted_at }`
- **Subscribers**: `cross-domain` (governance score + audit).

### E-027. `GovernanceScoreUpdated`

- **Publisher**: `institutional`/`cross-domain`
- **Quando**: job diário recalcula score
- **Payload**: `{ condominium_id, score, components, calculated_at }`
- **Subscribers**: frontend (dashboard); `cross-domain` (alert se < 60).

---

## commercial

### E-028. `CompanyCreated`

- **Publisher**: `commercial`
- **Quando**: empresa conclui cadastro CNPJ válido (pending_verification)
- **Payload**: `{ company_id, cnpj_masked, razao_social, categoria_principal, created_at }`
- **Subscribers**: `billing` (cria Subscription empresa); `cross-domain` (audit, moderação fila).

### E-029. `CompanyApproved`

- **Publisher**: `commercial`
- **Quando**: admin aprova empresa (status `pending_verification → active`)
- **Payload**: `{ company_id, approved_by, approved_at }`
- **Subscribers**: `content` (permite upload vídeo); `cross-domain` (notification).

### E-030. `CompanySuspended`

- **Publisher**: `commercial`
- **Quando**: admin suspende ou harassment confirmado
- **Payload**: `{ company_id, reason: "harassment|fraud|admin_decision", suspended_at, suspended_by }`
- **Subscribers**: `content` (desativa vídeos temporariamente); `cross-domain` (notification, audit).

### E-031. `ConnectMeRequestPublished`

- **Publisher**: `commercial`
- **Quando**: síndico/morador/empresa publica RFP (status `draft → published`)
- **Payload**: `{ request_id, fluxo: "SindicoEmpresa|MoradorSindico|EmpresaEmpresa|SindicoParceiro", publisher_user_id, target_scope, urgency, quota_consumed: true, published_at }`
- **Subscribers**: `billing` (decrementa quota dentro UoW); `cross-domain` (notification fan-out empresas qualificadas).

### E-032. `ConnectMeInterestExpressed`

- **Publisher**: `commercial`
- **Quando**: empresa clica "Tenho Interesse" → dados do síndico revelados
- **Payload**: `{ request_id, company_id, data_revealed: bool, lgpd_log_id, expressed_at, ip }`
- **Subscribers**: `cross-domain` (LGPD log obrigatório); frontend (revelação).

### E-033. `ConnectMeAutoClosed`

- **Publisher**: `commercial`
- **Quando**: job detecta 48h sem interesse
- **Payload**: `{ request_id, closed_at, reason: "no_interest_48h" }`
- **Subscribers**: `cross-domain` (notification publisher).

### E-034. `ProposalSubmitted`

- **Publisher**: `commercial`
- **Quando**: empresa envia proposta (status `draft → sent`; imutável pós)
- **Payload**: `{ proposal_id, request_id, company_id, title, estimated_cost_cents, sent_at }`
- **Subscribers**: `cross-domain` (notification síndico).

### E-035. `ProposalValidated`

- **Publisher**: `commercial`
- **Quando**: síndico valida proposta (pré-requisito pra votação fornecedor Req 21)
- **Payload**: `{ proposal_id, request_id, validated_by, validated_at }`
- **Subscribers**: `assembly` (habilita `votacao_fornecedor` em assembleia).

### E-036. `ProposalAccepted` / `ProposalRejected`

- **Publisher**: `commercial`
- **Quando**: resultado final (imutável); aceito → vira ClosedDeal
- **Payload**: `{ proposal_id, request_id, accepted: bool, reason?, decided_at }`
- **Subscribers**: `cross-domain` (notification).

### E-037. `ClosedDealSigned`

- **Publisher**: `commercial`
- **Quando**: dupla assinatura completa (empresa + síndico)
- **Payload**: `{ deal_id, deal_number, request_id, company_id, condominium_id, agreed_cost_cents, start_date, end_date, signed_at }`
- **Subscribers**: `institutional` (cria `TimelineEntry` tipo `registro_execucao` dentro de UoW); `cross-domain` (notification, audit).

### E-038. `ExecutionRecordSubmitted`

- **Publisher**: `commercial`
- **Quando**: empresa registra execução (fotos, datas, descrição)
- **Payload**: `{ record_id, deal_id, company_id, condominium_id, submitted_at }`
- **Subscribers**: `cross-domain` (notification síndico para validar).

### E-039. `ExecutionValidated`

- **Publisher**: `commercial`
- **Quando**: síndico valida execução (obrigatório — Req 23)
- **Payload**: `{ record_id, deal_id, validated_by, validated_at, approved: bool, rejection_reason? }`
- **Subscribers**: `institutional` (atualiza status atividade em `MasterPlan`); `content` (recomenda cursos LMS relevantes).

### E-040. `CompanyEvaluationSubmitted`

- **Publisher**: `commercial`
- **Quando**: síndico submete avaliação pós-contrato (Req 26)
- **Payload**: `{ evaluation_id, company_id, evaluator_user_id_hashed, condominium_id, criteria: {prazos, qualidade, comunicacao, profissionalismo, custos}, submitted_at }`
- **Subscribers**: `commercial` (handler interno: recalcula reputação); `cross-domain` (audit).

### E-041. `ReputationTierChanged`

- **Publisher**: `commercial`
- **Quando**: `ReputationCalculator` roda e tier muda (Bronze → Prata, etc)
- **Payload**: `{ company_id, previous_tier, new_tier, calculated_at, inputs: {contracts, avg_rating, condos_count, videos} }`
- **Subscribers**: `cross-domain` (audit, notification empresa); frontend (atualiza badge).

### E-042. `SupplierVoteSessionOpened` / `SupplierVoteClosed`

- **Publisher**: `commercial` (ou `assembly` quando é parte de assembleia)
- **Quando**: votação colegiada do conselho abre/fecha
- **Payload**: `{ session_id, request_id, proposals: [...], quorum_type, opened_at|closed_at, result? }`
- **Subscribers**: `assembly` (se está dentro), `cross-domain` (audit).

### E-043. `SupplierVoteCastRegistered`

- **Publisher**: `commercial`
- **Quando**: voto individual em SupplierVoteSession
- **Payload**: `{ cast_id, session_id, voter_id, choice: proposal_id, voted_at }`
- **Subscribers**: frontend (contagem real-time).

### E-044. `HarassmentReported`

- **Publisher**: `commercial`
- **Quando**: usuário denuncia abuso em Connect Me
- **Payload**: `{ report_id, reported_user_id, reporter_user_id_hashed, context: "connect_me", reason, reported_at }`
- **Subscribers**: `cross-domain` (fila moderação 48h; audit).

### E-045. `AmendmentSigned`

- **Publisher**: `commercial`
- **Quando**: adendo assinado por ambas as partes (Req 25)
- **Payload**: `{ amendment_id, amendment_type: "deal|execution_record|proposal|timeline|minutes", original_record_id, reason, signed_at }`
- **Subscribers**: `institutional` (cria TimelineEntry tipo `adendo`).

### E-046. `NeighborhoodBenefitPublished`

- **Publisher**: `commercial`
- **Quando**: parceiro publica cupom/promoção
- **Payload**: `{ benefit_id, partner_id, benefit_type, scope: "aberta|exclusiva", condominium_id?, valid_from, valid_until, published_at }`
- **Subscribers**: `cross-domain` (notification moradores no bairro via FCM).

### E-047. `CouponRedeemed`

- **Publisher**: `commercial`
- **Quando**: morador gera/redime cupom (lock 4h aplica)
- **Payload**: `{ coupon_id, code, benefit_id, user_id, partner_id, generated_at, expires_at }`
- **Subscribers**: `cross-domain` (audit, analytics).

---

## content

### E-048. `VideoUploaded`

- **Publisher**: `content`
- **Quando**: upload presigned Mux completo (webhook `video.ready`)
- **Payload**: `{ video_id, mux_asset_id, owner_id, tenant_type: "company|syndic|resident", video_type, duration_seconds, uploaded_at }`
- **Subscribers**: `cross-domain` (fila moderação 48h); `content` (internal: transição `processing → moderation`).

### E-049. `VideoModerationRequested`

- **Publisher**: `content`
- **Quando**: transição de estado pra `moderation`
- **Payload**: `{ moderation_id, video_id, requested_at }`
- **Subscribers**: `cross-domain` (notification admin pra moderar em 48h).

### E-050. `VideoApproved`

- **Publisher**: `content`
- **Quando**: admin aprova moderação
- **Payload**: `{ video_id, moderation_id, approved_by, approved_at }`
- **Subscribers**: `content` (internal: permite publish); `cross-domain` (notification owner).

### E-051. `VideoRejected`

- **Publisher**: `content`
- **Quando**: admin rejeita moderação
- **Payload**: `{ video_id, moderation_id, rejected_by, rejection_reason, rejected_at }`
- **Subscribers**: `cross-domain` (notification owner).

### E-052. `VideoPublished`

- **Publisher**: `content`
- **Quando**: owner publica pós-aprovação; seta `locked_until = NOW() + 90d`
- **Payload**: `{ video_id, owner_id, tenant_id, published_at, locked_until }`
- **Subscribers**: `commercial` (se vídeo institucional empresa → alimenta reputação tier Platina+); busca (reindex).

### E-053. `VideoLocked` / `VideoLockExpired`

- **Publisher**: `content`
- **Quando**: locked_until passa ou fica ativo
- **Payload**: `{ video_id, locked: bool, locked_until, changed_at }`
- **Subscribers**: frontend (atualiza UI de substituição).

### E-054. `VideoRemoved`

- **Publisher**: `content`
- **Quando**: admin remove (legal) ou owner remove pós-lock
- **Payload**: `{ video_id, removed_by, reason, removed_at }`
- **Subscribers**: Mux (`CancelMuxAsset` via saga); `commercial` (se alimentava reputação, recalcula).

### E-055. `VideoVisibilityGranted`

- **Publisher**: `content` (gatilho de `assembly.VotingOpened`)
- **Quando**: votação fornecedor abre + empresa autorizou → moradores ganham acesso temporário
- **Payload**: `{ grant_id, video_id, voting_session_id, granted_users: [...], granted_at, expires_at }`
- **Subscribers**: `cross-domain` (audit LGPD).

### E-056. `VideoVisibilityRevoked`

- **Publisher**: `content` (gatilho de `assembly.VotingClosed`)
- **Quando**: votação fecha → auto-revoga
- **Payload**: `{ grant_id, video_id, voting_session_id, revoked_at }`
- **Subscribers**: `cross-domain` (audit).

### E-057. `CoursePublished`

- **Publisher**: `content`
- **Quando**: admin publica curso LMS (status `draft → published`)
- **Payload**: `{ course_id, category, area, trilha?, published_at }`
- **Subscribers**: `cross-domain` (notification alunos interessados).

### E-058. `EnrollmentCreated`

- **Publisher**: `content`
- **Quando**: user matricula em curso
- **Payload**: `{ enrollment_id, user_id, course_id, created_at }`
- **Subscribers**: `cross-domain` (analytics produto).

### E-059. `LessonCompleted`

- **Publisher**: `content`
- **Quando**: user completa lição
- **Payload**: `{ enrollment_id, lesson_id, completed_at, progress_percent }`
- **Subscribers**: `content` (internal: se `progress=100%` → gera certificate).

### E-060. `CourseCompleted`

- **Publisher**: `content`
- **Quando**: user atinge 100% de curso
- **Payload**: `{ enrollment_id, course_id, completed_at }`
- **Subscribers**: `content` (handler `GenerateCertificate`); `cross-domain` (badge gamification Sprint 4+).

### E-061. `CertificateGenerated`

- **Publisher**: `content`
- **Quando**: certificate emitido + PDF gerado
- **Payload**: `{ certificate_id, serial_number, user_id, course_id, issued_at }`
- **Subscribers**: `cross-domain` (notification email com PDF).

---

## assembly

### E-062. `AssemblyScheduled`

- **Publisher**: `assembly`
- **Quando**: síndico configura assembleia (Req 48)
- **Payload**: `{ assembly_id, condominium_id, type, modality, scheduled_date, location, quorum_type, created_at }`
- **Subscribers**: `cross-domain` (audit).

### E-063. `EditalPublished`

- **Publisher**: `assembly`
- **Quando**: edital PDF publicado (desbloqueia `StartAssembly`)
- **Payload**: `{ edital_id, assembly_id, pdf_url, published_at }`
- **Subscribers**: `cross-domain` (notification T-5 dias, T-3 dias, T-1 dia, T-3 horas — Req 50).

### E-064. `AgendaPublished`

- **Publisher**: `assembly`
- **Quando**: pauta transição `draft → published` (lock)
- **Payload**: `{ assembly_id, agenda_items: [...], published_at }`
- **Subscribers**: `cross-domain` (notification ciência obrigatória Req 50.2).

### E-065. `AcknowledgmentRegistered`

- **Publisher**: `assembly`
- **Quando**: morador clica "Li e entendi a pauta" (Req 50.2)
- **Payload**: `{ ack_id, user_id, assembly_id, acknowledged_at, ip }`
- **Subscribers**: middleware (unlock app pra aquele user).

### E-066. `AssemblyStarted`

- **Publisher**: `assembly`
- **Quando**: síndico abre assembleia (pré-req: edital publicado)
- **Payload**: `{ assembly_id, started_by, started_at }`
- **Subscribers**: `cross-domain` (audit, notification).

### E-067. `CheckInRegistered`

- **Publisher**: `assembly`
- **Quando**: morador registra presença (app/terminal/manual)
- **Payload**: `{ attendance_id, assembly_id, user_id, unit_id, mode: "app|terminal|manual", registered_at }`
- **Subscribers**: WebSocket painel presidente (Req 60.2); `cross-domain` (audit).

### E-068. `ProxyGranted` / `ProxyRevoked`

- **Publisher**: `assembly`
- **Quando**: procuração outorgada/revogada pré-assembleia
- **Payload**: `{ proxy_id, authorized_by_user_id, proxy_user_id, assembly_id, valid_until, granted_at|revoked_at }`
- **Subscribers**: `cross-domain` (audit).

### E-069. `VotingOpened`

- **Publisher**: `assembly`
- **Quando**: síndico abre votação de item (via Painel do Presidente)
- **Payload**: `{ voting_id, assembly_id, agenda_item_id, item_type: "votacao_simples|votacao_fracao_ideal|votacao_fornecedor", opened_at, expires_at? }`
- **Subscribers**: `content` (se `votacao_fornecedor` com autorização → cria `VideoVisibilityGrant`); WebSocket morador (notification "vote agora"); `cross-domain` (audit).

### E-070. `VoteCast`

- **Publisher**: `assembly`
- **Quando**: voto individual registrado (UNIQUE INV-071)
- **Payload**: `{ vote_id, voting_id, agenda_item_id, voter_id, choice: "Aprovar|Rejeitar|Abster|Absent", is_proxy: bool, fracao_ideal_weight, voted_at }`
- **Subscribers**: WebSocket painel (contagem real-time); `cross-domain` (audit).

### E-071. `VotingClosed`

- **Publisher**: `assembly`
- **Quando**: síndico encerra votação ou timeout automático
- **Payload**: `{ voting_id, agenda_item_id, closed_at, result: {aprovar, rejeitar, abster, absent, quorum_reached}, passed: bool }`
- **Subscribers**: `content` (revoga `VideoVisibilityGrant`); `commercial` (se `votacao_fornecedor` → consolida); WebSocket.

### E-072. `SpeechQueueJoined` / `SpeechTurnStarted` / `SpeechTurnEnded`

- **Publisher**: `assembly`
- **Quando**: morador entra/inicia/encerra fala (Req 60.5)
- **Payload**: `{ queue_entry_id|turn_id, user_id, assembly_id, timer_start/end, duration_seconds }`
- **Subscribers**: WebSocket painel; Livekit (corte microfone no `TurnEnded` via API).

### E-073. `AssemblyClosed`

- **Publisher**: `assembly`
- **Quando**: síndico encerra assembleia (todas votações fechadas)
- **Payload**: `{ assembly_id, closed_at, closed_by }`
- **Subscribers**: `assembly` (internal: dispara geração de ata); `cross-domain` (audit).

### E-074. `MinutesGenerated`

- **Publisher**: `assembly`
- **Quando**: job async gera PDF ata
- **Payload**: `{ minutes_id, assembly_id, pdf_url, generated_at }`
- **Subscribers**: `cross-domain` (notification).

### E-075. `MinutesPublished`

- **Publisher**: `assembly`
- **Quando**: síndico publica ata (imutável pós); auto-cria TimelineEntry
- **Payload**: `{ minutes_id, assembly_id, condominium_id, published_at, published_by }`
- **Subscribers**: `institutional` (cria `TimelineEntry` tipo `assembly_result` dentro UoW); `cross-domain` (audit, notification).

### E-076. `HomologationOpened` / `HomologationApproved`

- **Publisher**: `assembly`
- **Quando**: votação de aprovação da ata (Req 60.6)
- **Payload**: `{ homologation_id, assembly_id, opened_at|approved_at, approved: bool, quorum_reached: bool }`
- **Subscribers**: `cross-domain` (audit, notification).

### E-077. `TransparencyScoreCalculated`

- **Publisher**: `assembly`
- **Quando**: pós-homologação (Req 60.7) — 10 componentes
- **Payload**: `{ assembly_id, condominium_id, score (0-100), components: [...], calculated_at }`
- **Subscribers**: frontend (badge público); `cross-domain` (métrica).

---

## cross-domain

### E-078. `AuditEntryAppended`

- **Publisher**: `cross-domain`
- **Quando**: ação sensível em qualquer BC (IAuditLogger — DT-010)
- **Payload**: `{ entry_id, actor_id, action, resource_type, resource_id, tenant_id, ip, user_agent, metadata, created_at }`
- **Subscribers**: frontend admin (dashboard audit); monitoring.

### E-079. `LGPDLogAppended`

- **Publisher**: `cross-domain`
- **Quando**: revelação de dados pessoais (Connect Me InterestExpressed, exportação `/me/export`, etc)
- **Payload**: `{ log_id, subject_user_id_hashed, consumer_user_id, data_revealed: [...], purpose, terms_version, created_at, ip }`
- **Subscribers**: monitoring; job de retenção 5 anos.

### E-080. `TermAccepted`

- **Publisher**: `cross-domain`
- **Quando**: user aceita termos versionados (Req 35)
- **Payload**: `{ acceptance_id, user_id, term_type: "tos|privacy|conduct", term_version, accepted_at, ip }`
- **Subscribers**: `cross-domain` (audit); `identity` (unlock se bloqueado por termos novos).

### E-081. `AntiFraudTriggered`

- **Publisher**: `cross-domain`
- **Quando**: CAPTCHA após 3 falhas login, múltiplos cadastros mesmo IP, anomalia IP (Req 102)
- **Payload**: `{ signal_type, target_ip|user_id, details, triggered_at }`
- **Subscribers**: `identity` (bloqueia temporariamente); monitoring.

### E-082. `WebhookEventReceived`

- **Publisher**: `cross-domain` (middleware infra)
- **Quando**: qualquer webhook Stripe/Mux/Livekit chega (pós HMAC verify)
- **Payload**: `{ provider, event_type, event_id, received_at }`
- **Subscribers**: BC correspondente (`billing`/`content`/`assembly`) consome idempotente.

---

## Total: **82 eventos canônicos** (15-30 esperados — expandimos pra minúcia cross-BC)

Distribuição:

| BC Publisher | Count |
|---|---|
| identity | 6 |
| billing | 7 |
| institutional | 14 |
| commercial | 20 |
| content | 14 |
| assembly | 16 |
| cross-domain | 5 |
| **total** | **82** |

---

## Regras de versionamento

- Evento tem `version: "v1"` default.
- Alteração breaking (remover campo, mudar tipo, renomear) → novo evento `UserCreatedV2` com 6m de deprecation.
- Alteração aditiva (novo campo opcional) → ok no mesmo version.
- Schema evolui via `domain_events_schema.json` (registry); validação no publish + no subscribe.

---

## ⚠️ Pendências de dual-check (Fase 5)

- **P-EVT-001**: `tenant_id` em payload é ambíguo entre `condominium_id` e `company_id`. Confirmar se cada evento declara explicitamente qual tipo (sugerido).
- **P-EVT-002**: outbox pattern implementado? Legado tem `domain_events` table? Confirmar pra M1. Decisão: implementar outbox antes de sair pra NATS.
- **P-EVT-003**: `ExecutionValidated` em `commercial` altera `MasterPlan` em `institutional`. Decisão: evento + handler em institutional. Legado pode estar chamando service direto — confirmar.
- **P-EVT-004**: `VideoVisibilityGranted` / `VideoVisibilityRevoked` — hoje é tabela `video_visibility_grants` em content. Confirmar se há mesmo eventos publicados ou é efeito direto.
- **P-EVT-005**: `SupplierVoteSession` vive em `commercial` ou é parte de `Assembly`? Legado tem tabelas em ambos. Confirmar ownership.
- **P-EVT-006**: `ReputationTierChanged` recalcula on-demand síncrono ou job async? Legado parece síncrono dentro da UoW de `CompanyEvaluationSubmitted`. Confirmar.
- **P-EVT-007**: `DeviceChanged` (E-006) entra em Sprint 10+? Confirmar timing — é um antipattern legado A-023 pendente.

---

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[context-map]] (Published Language)
- [[invariants#cross-domain]]
- [[state-machines]]
- [[aggregates/_moc]]
- [[../02-architecture/_moc]] (event bus detalhado — a destilar)
