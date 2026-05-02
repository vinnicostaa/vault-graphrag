---
title: State Machines — Master Síndico (entidades dinâmicas)
type: spec
tags: [domain, ddd, state-machines, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/01-domain/* + specs/requirements/*
created: 2026-04-23
updated: 2026-04-23
---

# State Machines — Master Síndico v2

Máquinas de estado das entidades mais dinâmicas do domínio. Toda transição é validada na factory/método do aggregate; transições ilegais retornam `ErrValidation` ou `ErrConflict`.

**Convenção**: estados em `snake_case` (EN) ou pt-BR quando semântica jurídica BR única. Transições = arestas; gatilho entre parênteses.

---

## 1. Subscription (billing)

**Aggregate**: `Subscription`
**BC**: billing
**Fonte**: requirements/billing.md Req 4, 4.1; regras-canonicas §4

```
                  UserCreated (evento identity)
                          │
                          ▼
                    ┌───────────┐
                    │  trial    │◀─────────── (novo User; trial persona-aware)
                    └─────┬─────┘
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
       │ payment_ok       │ trial_ended      │ canceled
       ▼                  ▼                  ▼
  ┌─────────┐        ┌──────────┐       ┌──────────┐
  │ active  │───────▶│ expired  │       │ canceled │
  └────┬────┘        └──────────┘       └──────────┘
       │                  ▲
       │ payment_failed   │ (webhook Stripe)
       ▼                  │
  ┌──────────┐            │
  │ past_due │────────────┘
  └──────────┘
```

### Estados

| Estado | Descrição | Acesso premium |
|---|---|---|
| `trial` | Primeira janela gratuita persona-aware (síndico 15d, empresa 7d, parceiro 30d) | Tudo visível; ações estratégicas bloqueadas conforme persona |
| `active` | Pagamento confirmado; features premium liberadas | ✅ |
| `past_due` | Pagamento falhou; Stripe tenta retry | 🟡 Tolerância curta (default Stripe) |
| `expired` | Trial ended ou past_due > retry_max | ❌ Soft-block sem grace (Decision 4) |
| `canceled` | User cancelou via Customer Portal | ❌ Soft-block |
| `paused` | Admin pausou (raro; legal) | ❌ |

### Transições válidas

| De → Para | Gatilho | Ação |
|---|---|---|
| — → `trial` | evento `UserCreated` | Create Subscription; `trial_ends_at = NOW() + duration(persona)` |
| `trial` → `active` | webhook `payment_intent.succeeded` | Ativar features; publish `SubscriptionActivated` |
| `trial` → `expired` | job: `NOW() >= trial_ends_at` e sem pagamento | Publish `TrialExpired`; soft-block |
| `active` → `past_due` | webhook `invoice.payment_failed` | Manter features curta janela (retry Stripe) |
| `past_due` → `active` | webhook `payment_intent.succeeded` (retry ok) | Restaura |
| `past_due` → `expired` | retry exhausted | Publish `SubscriptionExpired`; soft-block |
| `active` → `canceled` | webhook `customer.subscription.deleted` | Publish `SubscriptionCanceled`; soft-block |
| `expired` → `active` | webhook `payment_intent.succeeded` (reativação) | Reativa |
| `canceled` → `active` | webhook (recreate subscription) | Reativa (Stripe gerencia pro-rata) |
| qualquer → `paused` | admin decision + audit log | Pausa |

### Transições proibidas

- ❌ `active → trial` (INV-013 — BR-018: trial nunca volta)
- ❌ `canceled → trial` (mesma razão)
- ❌ `expired → trial`

### Invariantes

- INV-007: append-only (novo registro a cada mudança, não UPDATE)
- INV-013: trial nunca volta pós `active` / `canceled`
- INV-019: soft-block sem grace period

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `trial` | `UserCreated` | `T-BIL-sm-none-trial` | `/tests/integration/billing/sm_subscription_test.go` |
| `trial` → `active` | webhook `payment_intent.succeeded` | `T-BIL-sm-trial-active` | idem |
| `trial` → `expired` | job `trial_ends_at <= NOW()` | `T-BIL-sm-trial-expired` | idem |
| `active` → `past_due` | webhook `invoice.payment_failed` | `T-BIL-sm-active-past_due` | idem |
| `past_due` → `active` | webhook `payment_intent.succeeded` (retry ok) | `T-BIL-sm-past_due-active` | idem |
| `past_due` → `expired` | retry exhausted (Stripe Smart Retries esgotado) | `T-BIL-sm-past_due-expired` | idem |
| `active` → `canceled` | webhook `customer.subscription.deleted` | `T-BIL-sm-active-canceled` | idem |
| `expired` → `active` | reativação com `payment_intent.succeeded` | `T-BIL-sm-expired-active` | idem |
| `canceled` → `active` | recreate subscription Stripe | `T-BIL-sm-canceled-active` | idem |
| * → `paused` | admin decision | `T-BIL-sm-any-paused` | idem |
| **Proibida** `active → trial` | — | `T-BIL-sm-no-trial-revival` (expect `ErrValidation`) | idem |

Ver linha consolidada em [[../09-testing/test-traceability#11-state-machines-transições]].

---

## 2. VideoModeration (content)

**Aggregate**: `Video` + `VideoModeration`
**BC**: content
**Fonte**: requirements/content.md Req 29

```
       upload (presigned Mux)
            │
            ▼
     ┌────────────┐
     │ uploading  │
     └─────┬──────┘
           │ webhook video.ready
           ▼
     ┌────────────┐
     │ processing │
     └─────┬──────┘
           │ auto
           ▼
     ┌──────────────┐          rejected
     │  moderation  │────────────────────▶ ┌──────────┐
     └──────┬───────┘                      │ rejected │
            │ admin approve                 └──────────┘
            ▼
     ┌──────────┐
     │ approved │
     └─────┬────┘
           │ owner publish
           ▼
     ┌───────────┐   locked_until (= published_at + 90d)
     │ published │─────────────────────────────────────▶ ┌────────┐
     └─────┬─────┘                                       │ locked │
           │                                             └────┬───┘
           │                                                  │ NOW >= locked_until
           │  admin remove (bypass com motivo)                ▼
           │◀─────────────────────────────────           ┌──────────┐
           ▼                                             │ unlocked │
     ┌──────────┐                                        └────┬─────┘
     │ removed  │◀──────────────────────────────────────      │
     └──────────┘         owner remove pós-lock               ▼
                                                         ┌──────────┐
                                                         │ removed  │
                                                         └──────────┘
```

### Estados (VideoState)

| Estado | Descrição |
|---|---|
| `uploading` | Upload em progresso (presigned URL Mux) |
| `processing` | Mux processing (HLS transcoding) |
| `moderation` | Aguardando aprovação admin (48h SLA) |
| `approved` | Admin aprovou; owner pode publicar |
| `rejected` | Admin rejeitou (motivo em `rejection_reason`) |
| `published` | Público/restrito conforme ABAC |
| `locked` | Bloqueado por 90d pós-publish (não pode remover nem substituir) |
| `unlocked` | `locked_until` passou; pode substituir |
| `removed` | Soft-remove (owner ou admin) |

### Transições válidas

| De → Para | Gatilho | Invariante |
|---|---|---|
| — → `uploading` | `UploadVideoUseCase` | presigned URL Mux |
| `uploading` → `processing` | webhook Mux `video.ready` | — |
| `uploading` → `rejected` (implícito via removal) | erro Mux; saga compensa (A-027) | INV-065 |
| `processing` → `moderation` | auto (fila admin) | — |
| `moderation` → `approved` | admin aprova | INV-058 |
| `moderation` → `rejected` | admin rejeita com motivo | — |
| `approved` → `published` | owner publica | seta `locked_until` INV-056 |
| `published` → `locked` | auto (sempre locked após publish) | INV-056 |
| `locked` → `unlocked` | job: `NOW() >= locked_until` | INV-056 |
| `unlocked` → `published` (nova versão) | owner substitui (trava trimestral INV-059) | — |
| `published`/`locked`/`unlocked` → `removed` | admin remove + motivo + audit (override 90d só admin) | INV-057 |

### Transições proibidas

- ❌ `uploading` → `published` (tem que passar por moderação)
- ❌ `moderation` → `published` (precisa aprovar antes)
- ❌ `locked` → `removed` sem `user.is_admin = true`
- ❌ `rejected` → `published` (sem re-moderação)

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `uploading` | `UploadVideoUseCase` | `T-CNT-sm-none-uploading` | `/tests/integration/content/sm_video_test.go` |
| `uploading` → `processing` | webhook Mux `video.ready` | `T-CNT-sm-upload-processing` | idem |
| `processing` → `moderation` | auto (fila admin) | `T-CNT-sm-proc-moderation` | idem |
| `moderation` → `approved` | admin aprova | `T-CNT-sm-mod-approved` | idem |
| `moderation` → `rejected` | admin rejeita | `T-CNT-sm-mod-rejected` | idem |
| `approved` → `published` | owner publica | `T-CNT-sm-approved-published` | idem |
| `published` → `locked` | auto (seta `locked_until`) | `T-CNT-sm-published-locked` | idem |
| `locked` → `unlocked` | job `NOW() >= locked_until` | `T-CNT-sm-locked-unlocked` | idem |
| `unlocked` → `published` | nova versão (trava trimestral INV-059) | `T-CNT-sm-unlocked-replace` | idem |
| `published`/`locked`/`unlocked` → `removed` | admin bypass + audit | `T-CNT-sm-remove-admin-bypass` | idem |
| **Proibida** `moderation → published` direto | — | `T-CNT-sm-no-mod-published-direct` | idem |
| **Proibida** `rejected → published` | — | `T-CNT-sm-rejected-no-republish` | idem |

---

## 3. Assembly (assembly)

**Aggregate**: `Assembly`
**BC**: assembly
**Fonte**: requirements/assembly.md Req 48-60

```
     (síndico configura)
              │
              ▼
       ┌───────────┐
       │ scheduled │
       └──────┬────┘
              │ edital publicado + pauta publicada
              ▼
       ┌───────────┐
       │ published │──── cancelamento (até 3 dias antes) ──▶ ┌──────────┐
       └──────┬────┘                                         │ canceled │
              │ síndico clica "Iniciar Assembleia"           └──────────┘
              ▼
       ┌─────────────┐
       │ in_progress │
       └──────┬──────┘
              │ síndico encerra (todas votações fechadas)
              ▼
       ┌────────────┐      ata gerada + publicada
       │  closed    │──────────────────────────────▶ (minutes published)
       └────────────┘
              │ homologação votada e aprovada
              ▼
       ┌──────────────┐
       │ homologated  │  (imutável absoluto — INV-085)
       └──────────────┘
```

### Estados (AssemblyStatus)

| Estado | Descrição |
|---|---|
| `scheduled` | Configurada; pauta pode estar em draft |
| `published` | Pauta + edital publicados; notificações T-5/T-3/T-1 disparadas |
| `in_progress` | Síndico iniciou; check-ins, votações, fala rolando |
| `closed` | Síndico encerrou; ata gerada async |
| `homologated` | Ata aprovada em votação de homologação; imutável |
| `canceled` | Cancelada até 3 dias antes |

### Transições válidas

| De → Para | Gatilho | Invariante |
|---|---|---|
| — → `scheduled` | `ScheduleAssemblyUseCase` | — |
| `scheduled` → `published` | `PublishAgenda + PublishEdital` | INV-074 (não pode pular) |
| `published` → `in_progress` | `StartAssemblyUseCase` | INV-043 (≤ 1 ativa) |
| `in_progress` → `closed` | `CloseAssemblyUseCase` (todas votações fechadas) | — |
| `closed` → `homologated` | `HomologateMinutesUseCase` (voto homologação aprovado) | INV-085 |
| `scheduled`/`published` → `canceled` | admin/síndico (até 3d antes) | — |

### Transições proibidas

- ❌ `scheduled` → `in_progress` (sem publish)
- ❌ `in_progress` → `scheduled` (nunca recua)
- ❌ `homologated` → qualquer outro estado (imutável absoluto)
- ❌ `closed` → `in_progress` (nunca reabre)
- ❌ `canceled` → `in_progress` (cancela é terminal)

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `scheduled` | `ScheduleAssemblyUseCase` | `T-ASM-sm-none-scheduled` | `/tests/integration/assembly/sm_assembly_test.go` |
| `scheduled` → `published` | `PublishAgenda + PublishEdital` | `T-ASM-sm-scheduled-published` | idem |
| `published` → `in_progress` | `StartAssemblyUseCase` (INV-043) | `T-ASM-sm-published-inProgress` | idem |
| `in_progress` → `closed` | `CloseAssemblyUseCase` | `T-ASM-sm-inProgress-closed` | idem |
| `closed` → `homologated` | `HomologateMinutesUseCase` (INV-085) | `T-ASM-sm-closed-homologated` | idem |
| `scheduled`/`published` → `canceled` | admin/síndico (≥ 3d antes) | `T-ASM-sm-cancel-preStart` | idem |
| **Proibida** `scheduled → in_progress` direto | — | `T-ASM-sm-no-skip-publish` | idem |
| **Proibida** `homologated → *` | — | `T-ASM-sm-homologated-immutable` | idem |

---

## 4. ConnectMeRequest (commercial)

**Aggregate**: `ConnectMeRequest`
**BC**: commercial
**Fonte**: requirements/commercial.md Req 19-19.3

```
              (síndico/morador/empresa cria)
                           │
                           ▼
                    ┌─────────┐
                    │  draft  │
                    └────┬────┘
                         │ publish (consome quota BR-021)
                         ▼
                    ┌───────────┐
                    │ published │──── 48h sem interesse ──▶ ┌─────────────┐
                    └─────┬─────┘                           │ auto_closed │
                          │ empresa expressa interesse      └─────────────┘
                          ▼   (reveal + LGPD log)
               ┌─────────────────────┐
               │ interest_expressed  │
               └──────────┬──────────┘
                          │ empresa submete proposta
                          ▼
               ┌─────────────────────┐
               │ proposals_received  │
               └──────────┬──────────┘
                          │ síndico seleciona proposta
                          ▼
               ┌─────────────┐
               │  selected   │
               └──────┬──────┘
                      │ dupla assinatura (empresa + síndico)
                      ▼
               ┌──────────────┐
               │ closed_deal  │  (ClosedDeal aggregate criado; TimelineEntry auto-publish)
               └──────┬───────┘
                      │ execução concluída + validada
                      ▼
               ┌───────────┐
               │ evaluated │  (CompanyEvaluation; alimenta Reputação)
               └───────────┘
```

### Estados

| Estado | Descrição |
|---|---|
| `draft` | Rascunho (edição livre); quota **não** decrementada |
| `published` | RFP visível às empresas qualificadas; quota decrementada INV-045 |
| `interest_expressed` | Empresa clicou "Tenho Interesse"; dados revelados INV-048 |
| `proposals_received` | ≥ 1 proposta `sent` recebida |
| `auto_closed` | 48h sem interesse → job fecha |
| `selected` | Síndico escolheu proposta (status proposal: `validated → accepted`) |
| `closed_deal` | Contrato assinado (dupla); ClosedDeal criado |
| `evaluated` | Avaliação pós-contrato submetida |
| `canceled` | Síndico cancelou pré-assinatura |

### Transições válidas

| De → Para | Gatilho | Invariante |
|---|---|---|
| — → `draft` | Create | — |
| `draft` → `published` | `PublishConnectMeUseCase` (UoW: decrementa quota + insere + notifica) | INV-045 |
| `draft` → (delete) | user descarta rascunho | sem impacto quota |
| `published` → `interest_expressed` | empresa expressa interesse (LGPD log) | INV-048 |
| `published` → `auto_closed` | job: NOW >= published_at + 48h | INV-047 |
| `interest_expressed` → `proposals_received` | empresa envia Proposal `sent` | INV-039 (proposta imutável pós sent) |
| `proposals_received` → `selected` | síndico valida + aceita uma proposta | INV-039 |
| `selected` → `closed_deal` | dupla assinatura | INV-040, INV-025 |
| `closed_deal` → `evaluated` | `CompanyEvaluation` submetida pós-execução | BR-027 |
| qualquer antes de `closed_deal` → `canceled` | síndico cancela | — |

### Transições proibidas

- ❌ `published` → `draft` (imutável após publish — quota já consumida)
- ❌ `closed_deal` → `canceled` (usa Amendment pra alterar INV-050)
- ❌ `evaluated` → qualquer (terminal)

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `draft` | Create | `T-COM-sm-none-draft` | `/tests/integration/commercial/sm_connect_me_test.go` |
| `draft` → `published` | `PublishConnectMeUseCase` (UoW consome quota) | `T-COM-sm-draft-published` | idem |
| `published` → `interest_expressed` | empresa expressa interesse + LGPD log | `T-COM-sm-published-interest` | idem |
| `published` → `auto_closed` | job 48h (INV-047) | `T-COM-sm-published-autoclose` | idem |
| `interest_expressed` → `proposals_received` | empresa envia Proposal `sent` (INV-039) | `T-COM-sm-interest-proposals` | idem |
| `proposals_received` → `selected` | síndico valida + aceita | `T-COM-sm-proposals-selected` | idem |
| `selected` → `closed_deal` | dupla assinatura (INV-040) | `T-COM-sm-selected-closedDeal` | idem |
| `closed_deal` → `evaluated` | `CompanyEvaluation` submetida | `T-COM-sm-closedDeal-evaluated` | idem |
| * pré-closed_deal → `canceled` | síndico cancela | `T-COM-sm-cancel-preSign` | idem |
| **Proibida** `published → draft` | — | `T-COM-sm-no-unpublish` | idem |
| **Proibida** `closed_deal → canceled` | — | `T-COM-sm-no-undo-deal` | idem |

---

## 5. VoteSession (assembly)

**Aggregate**: sub-aggregate `AgendaItem + Vote` dentro de `Assembly`
**BC**: assembly
**Fonte**: requirements/assembly.md Req 57

```
               (síndico abre no Painel)
                        │
                        ▼
                  ┌──────────┐
                  │   open   │◀──── votos chegam (INSERT único por voter)
                  └─────┬────┘
                        │ síndico encerra ou timeout
                        ▼
                  ┌────────────┐
                  │ vote_count │  (count em progresso via WebSocket broadcast)
                  └──────┬─────┘
                        │ count finalizado
                        ▼
                  ┌──────────┐
                  │  closed  │  (resultado imutável — INV-071)
                  └─────┬────┘
                        │ incluído em ata
                        ▼
                  ┌─────────┐
                  │ tallied │ (resultado reportado; quórum calculado)
                  └─────────┘
```

### Estados (VotingStatus)

| Estado | Descrição |
|---|---|
| `open` | Votação ativa; moradores podem votar |
| `vote_count` | Contagem em progresso (síndico fechou; job agregando) |
| `closed` | Resultado final consolidado |
| `tallied` | Resultado reportado no painel + incluído na ata |

Nota: na implementação comum, `vote_count` + `tallied` podem colapsar em `closed` (transição atômica). Explicitamos aqui pra observabilidade.

### Transições válidas

| De → Para | Gatilho |
|---|---|
| — → `open` | `OpenVotingUseCase` (síndico via painel) |
| `open` → `vote_count` | síndico fecha OU timeout automático |
| `vote_count` → `closed` | aggregation complete |
| `closed` → `tallied` | resultado publicado (WebSocket broadcast) |

### Invariantes durante transições

- INV-071: Vote único por (agenda_item, voter) — UNIQUE DB
- INV-076: Timeout → `Absent` vote idempotente
- INV-081: Presidente não vota
- INV-083: `fracao_ideal_weight` snapshot no momento do voto

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `open` | `OpenVotingUseCase` | `T-ASM-sm-vote-open` | `/tests/integration/assembly/sm_vote_session_test.go` |
| `open` → `vote_count` | síndico fecha OR timeout | `T-ASM-sm-vote-count` | idem |
| `vote_count` → `closed` | aggregation complete | `T-ASM-sm-vote-closed` | idem |
| `closed` → `tallied` | resultado publicado via WS | `T-ASM-sm-vote-tallied` | idem |
| **Proibida** voto em `closed`/`tallied` | — | `T-ASM-sm-no-vote-post-close` | idem |
| **Proibida** `tallied → open` (reabertura) | — | `T-ASM-sm-no-reopen` | idem |

**Transições proibidas adicionadas (fechamento A-DC-QA-061)**:
- ❌ Voto após `closed` / `tallied` (INV-071)
- ❌ Reabertura pós-`tallied`

---

## 6. Company Reputation Tier (commercial)

**Aggregate**: `Company` (campo `reputation_tier`)
**BC**: commercial
**Fonte**: regras-canonicas + business-rules BR-028..BR-031 + portfolio

```
          (CompanyCreated, CNPJ verificado)
                     │
                     ▼
               ┌──────────┐
               │  Bronze  │ (default)
               └────┬─────┘
                    │ ≥ 3 contratos + avg ≥ 3/5
                    ▼
               ┌──────────┐
               │  Prata   │
               └────┬─────┘
                    │ ≥ 10 contratos + avg ≥ 3.5 + ≥ 3 condos
                    ▼
               ┌──────────┐
               │   Ouro   │
               └────┬─────┘
                    │ ≥ 30 + avg ≥ 4 + ≥ 10 condos + ≥ 1 vídeo aprovado + case
                    ▼
               ┌──────────┐
               │ Platina  │
               └────┬─────┘
                    │ ≥ 100 + avg ≥ 4.5 + ≥ 30 condos + ≥ 3 vídeos + ≥ 12m
                    ▼
               ┌──────────┐
               │ Diamante │
               └──────────┘
```

### Transições **reversíveis** (função pura recalcula)

- Cada `CompanyEvaluationSubmitted` ou novo `ClosedDeal` ou `VideoPublished/Removed` → `RecalculateReputation(company_id)`.
- Se inputs caem abaixo do threshold → tier **desce** (degradação permitida BR-030).
- Recálculo é idempotente: mesmo input → mesmo output (INV-036).

### Flag paralela: `suspended`

- `HarassmentReported` confirmado → `company.suspended = true` durante 72h cooldown moderação (BR-031).
- **Não** rebaixa tier; marca flag **sobre** o tier atual. Visualmente o badge mostra "Prata - Suspensa".

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `Bronze` | `CompanyCreated` (default) | `T-COM-sm-rep-default-bronze` | `/tests/integration/commercial/sm_reputation_test.go` |
| `Bronze` → `Prata` | ≥ 3 contratos + avg ≥ 3/5 | `T-COM-sm-rep-bronze-prata` | idem |
| `Prata` → `Ouro` | ≥ 10 + avg ≥ 3.5 + ≥ 3 condos | `T-COM-sm-rep-prata-ouro` | idem |
| `Ouro` → `Platina` | ≥ 30 + avg ≥ 4 + ≥ 10 condos + 1 vídeo + case | `T-COM-sm-rep-ouro-platina` | idem |
| `Platina` → `Diamante` | ≥ 100 + avg ≥ 4.5 + ≥ 30 condos + 3 vídeos + 12m | `T-COM-sm-rep-platina-diamante` | idem |
| Qualquer tier → tier inferior | inputs caem abaixo do threshold (BR-030) | `T-COM-sm-rep-degradation-reversible` | idem |
| Flag `suspended` toggle | `HarassmentReportConfirmed` (72h) | `T-COM-sm-rep-suspended-flag` | idem |

**Transições proibidas adicionadas (fechamento A-DC-QA-061)**:
- ❌ Pular tiers (ex: Bronze → Ouro sem Prata intermediário) — mesmo que inputs bateriam, recálculo é sequencial tier a tier
- ❌ `Bronze` → `canceled` ou estado fora da enum (tier é determinístico, função pura)

---

## 7. User Session (identity)

**Aggregate**: `Session`
**BC**: identity
**Fonte**: requirements/identity.md Req 1 + BR-004

```
               (login Zitadel + PKCE)
                       │
                       ▼
                ┌─────────┐
                │ active  │◀──── request válido (cookie/Bearer)
                └────┬────┘
                     │
        ┌────────────┼─────────────────────────────────┐
        │            │                                 │
        │ idle>TTL   │ login novo device (1-device)    │ user logout
        ▼            ▼                                 ▼
  ┌──────────┐  ┌──────────────────────┐         ┌──────────┐
  │   idle   │  │ revoked_other_device │         │logged_out│
  └────┬─────┘  └──────────────────────┘         └──────────┘
       │                                              ▲
       │ request chega                                │
       ▼                                              │
  ┌─────────┐                                         │
  │ active  │◀────────────────────────────────────────┘
  └─────────┘
       │ admin revoga / zitadel revoga / ban
       ▼
  ┌──────────┐
  │ revoked  │
  └──────────┘
```

### Estados

| Estado | Descrição |
|---|---|
| `active` | Cookie httpOnly / Bearer válido; cache Redis TTL 5min |
| `idle` | Sem request recente; ainda válida até `expires_at` |
| `revoked_other_device` | Login em 2º device invalidou essa (1-device) |
| `logged_out` | User fez logout |
| `revoked` | Admin/Zitadel/ban invalidou |
| `expired` | `expires_at` passou |

### Transições válidas

| De → Para | Gatilho |
|---|---|
| — → `active` | `CreateSessionUseCase` pós Zitadel OIDC callback |
| `active` → `idle` | sem request em X min (inferido) |
| `idle` → `active` | request chega |
| `active`/`idle` → `revoked_other_device` | novo login em device diferente → `InvalidatePreviousByUserID` dentro UoW (INV-005) |
| `active`/`idle` → `logged_out` | user faz logout |
| `active`/`idle` → `revoked` | admin/Zitadel revoga |
| qualquer → `expired` | `NOW() >= expires_at` |

### Transições proibidas

- ❌ Qualquer terminal (`revoked_other_device`, `logged_out`, `revoked`, `expired`) → `active` (cria nova session, não reaproveita — INV-007)

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `active` | `CreateSessionUseCase` pós OIDC callback | `T-IDN-sm-login-active` | `/tests/integration/identity/sm_session_test.go` |
| `active` → `idle` | sem request em X min | `T-IDN-sm-active-idle` | idem |
| `idle` → `active` | request chega | `T-IDN-sm-idle-active` | idem |
| `active`/`idle` → `revoked_other_device` | novo login device ≠ (INV-005) | `T-IDN-sm-1device-revoke` | idem |
| `active`/`idle` → `logged_out` | user logout | `T-IDN-sm-logout` | idem |
| `active`/`idle` → `revoked` | admin/Zitadel revoga | `T-IDN-sm-admin-revoke` | idem |
| * → `expired` | `NOW() >= expires_at` | `T-IDN-sm-expired-terminal` | idem |
| **Proibida** terminal → `active` | — | `T-IDN-sm-no-reuse-terminal` | idem |

---

## 8. MasterPlan Activity (institutional)

**Aggregate**: `MasterPlan + MasterPlanActivity`
**BC**: institutional
**Fonte**: legado modelagem + requirements/institutional.md

```
                 (síndico cria atividade no plano)
                              │
                              ▼
                        ┌──────────┐
                        │  planned │
                        └─────┬────┘
                              │ contrato fechado (ClosedDealSigned)
                              ▼
                        ┌───────────┐
                        │ contracted│
                        └─────┬─────┘
                              │ execução iniciada (ExecutionRecordSubmitted)
                              ▼
                        ┌───────────┐
                        │in_progress│
                        └─────┬─────┘
                              │
            ┌─────────────────┼──────────────────┐
            │                 │                  │
            │ delayed         │ validated        │ rejected
            ▼                 ▼                  ▼
     ┌──────────┐       ┌───────────┐     ┌──────────┐
     │ delayed  │       │ completed │     │ rejected │
     └──────────┘       └───────────┘     └──────────┘
                              │
                              │ adendo (correção)
                              ▼
                        ┌───────────┐
                        │ amended   │
                        └───────────┘
```

### Estados (7)

`planned | contracted | in_progress | delayed | completed | rejected | amended`

### Regra-chave

- Mudança de status **apenas** via evento da Timeline (INV-031, BR-010).
- `in_progress → completed` só via `ExecutionValidated` (handler institutional consome evento commercial).

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `planned` | síndico cria atividade no plano | `T-INS-sm-mpa-none-planned` | `/tests/integration/institutional/sm_master_plan_test.go` |
| `planned` → `contracted` | `ClosedDealSigned` evento commercial | `T-INS-sm-mpa-planned-contracted` | idem |
| `contracted` → `in_progress` | `ExecutionRecordSubmitted` | `T-INS-sm-mpa-contracted-inProgress` | idem |
| `in_progress` → `completed` | `ExecutionValidated` | `T-INS-sm-mpa-inProgress-completed` | idem |
| `in_progress` → `delayed` | prazo SLO vencido sem validação | `T-INS-sm-mpa-inProgress-delayed` | idem |
| `in_progress` → `rejected` | síndico rejeita execução | `T-INS-sm-mpa-inProgress-rejected` | idem |
| qualquer → `amended` | adendo (INV-050) | `T-INS-sm-mpa-amended` | idem |
| **Proibida** edição direta (sem evento) | — | `T-INS-sm-mpa-no-direct-update` | idem |

**Transições proibidas adicionadas (fechamento A-DC-QA-061)**:
- ❌ Edição de `status` sem evento da Timeline (INV-031)
- ❌ `completed` → `in_progress` (terminal virtuoso; correção via adendo)
- ❌ `rejected` → `completed` sem novo `ExecutionValidated`

---

## 9. Proposal (commercial)

**Aggregate**: `Proposal`
**BC**: commercial
**Fonte**: requirements/commercial.md Req 20

```
                  (empresa cria)
                        │
                        ▼
                  ┌──────────┐
                  │  draft   │
                  └────┬─────┘
                       │ empresa envia
                       ▼
                  ┌──────────┐           síndico devolve pra revisão
                  │   sent   │────────────────────────────▶ (via Amendment)
                  └────┬─────┘
                       │ síndico abre período análise
                       ▼
                  ┌─────────────────────┐
                  │ awaiting_validation │
                  └──────────┬──────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          │ síndico valida   │ síndico rejeita  │ timeout (30d)
          ▼                  ▼                  ▼
     ┌───────────┐      ┌──────────┐       ┌──────────┐
     │ validated │      │ rejected │       │ expired  │
     └─────┬─────┘      └──────────┘       └──────────┘
           │ aceita (pode ir pra votação fornecedor)
           ▼
     ┌──────────┐
     │ accepted │  (imutável; vira ClosedDeal)
     └──────────┘
```

### Estados (ProposalStatus)

`draft | sent | awaiting_validation | validated | accepted | rejected | expired`

### Imutabilidade

- Após `sent` → imutável (INV-039). Correção via Amendment (`previous_proposal_id`).
- `accepted` e `rejected` são terminais (INV-039).

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `draft` | empresa cria | `T-COM-sm-prop-none-draft` | `/tests/integration/commercial/sm_proposal_test.go` |
| `draft` → `sent` | empresa envia | `T-COM-sm-prop-draft-sent` | idem |
| `sent` → `awaiting_validation` | síndico abre período análise | `T-COM-sm-prop-sent-awaiting` | idem |
| `awaiting_validation` → `validated` | síndico valida | `T-COM-sm-prop-awaiting-validated` | idem |
| `awaiting_validation` → `rejected` | síndico rejeita | `T-COM-sm-prop-awaiting-rejected` | idem |
| `awaiting_validation` → `expired` | timeout 30d | `T-COM-sm-prop-awaiting-expired` | idem |
| `validated` → `accepted` | síndico aceita (vira ClosedDeal) | `T-COM-sm-prop-validated-accepted` | idem |
| **Proibida** edição após `sent` | — | `T-COM-sm-prop-accepted-immutable` | idem |
| **Proibida** `rejected → *` | — | `T-COM-sm-prop-rejected-terminal` | idem |

---

## 10. HarassmentReport (commercial, cross-cutting)

**Aggregate**: `HarassmentReport`
**BC**: commercial
**Fonte**: requirements/commercial.md Req 19 anti-assédio

```
        (usuário denuncia em Connect Me)
                       │
                       ▼
                ┌──────────┐
                │ reported │
                └────┬─────┘
                     │ admin inicia análise (48h SLA)
                     ▼
                ┌───────────┐
                │ in_review │
                └─────┬─────┘
                      │
         ┌────────────┼─────────────────────┐
         │            │                     │
         │ confirmed  │ dismissed           │ escalated
         ▼            ▼                     ▼
    ┌──────────┐  ┌──────────┐       ┌────────────┐
    │confirmed │  │dismissed │       │ escalated  │
    └────┬─────┘  └──────────┘       └──────┬─────┘
         │ ação: company.suspended=true
         │      + reputação flag 72h
         ▼
    ┌──────────┐
    │  closed  │
    └──────────┘
```

### Estados

`reported | in_review | confirmed | dismissed | escalated | closed`

### Cobertura de transições (fechamento A-DC-QA-060)

| Transição | Gatilho | Test ID | Arquivo previsto |
|---|---|---|---|
| — → `reported` | usuário denuncia em Connect Me | `T-COM-sm-hr-none-reported` | `/tests/integration/commercial/sm_harassment_test.go` |
| `reported` → `in_review` | admin inicia análise (SLA 48h) | `T-COM-sm-hr-reported-review` | idem |
| `in_review` → `confirmed` | admin confirma assédio | `T-COM-sm-hr-review-confirmed` | idem |
| `in_review` → `dismissed` | admin arquiva (sem fundamento) | `T-COM-sm-hr-review-dismissed` | idem |
| `in_review` → `escalated` | admin escala (DPO/legal) | `T-COM-sm-hr-review-escalated` | idem |
| `confirmed` → `closed` | após 72h cooldown + ação executada | `T-COM-sm-hr-confirmed-closed` | idem |
| `dismissed` → `closed` | terminal direto | `T-COM-sm-hr-dismissed-closed` | idem |
| `escalated` → `closed` | após resolução externa | `T-COM-sm-hr-escalated-closed` | idem |
| **Proibida** `closed → *` | — | `T-COM-sm-hr-closed-terminal` | idem |
| **Proibida** reabertura pós-`closed` | — | `T-COM-sm-hr-no-reopen` | idem |

**Transições proibidas adicionadas (fechamento A-DC-QA-061)**:
- ❌ Reabertura de `closed` (só via nova denúncia)
- ❌ `reported → confirmed/dismissed/escalated` direto (sem passar por `in_review`)

---

## Resumo — máquinas implementadas (10)

| # | State Machine | BC | Estados | Transições proibidas |
|---|---|---|---|---|
| 1 | Subscription | billing | 6 | `→ trial` pós-active/canceled |
| 2 | VideoModeration | content | 9 | `moderation → published` direto; `locked → removed` sem admin |
| 3 | Assembly | assembly | 6 | `scheduled → in_progress` sem publish; `homologated → *` |
| 4 | ConnectMeRequest | commercial | 8 | `published → draft`; `closed_deal → canceled` |
| 5 | VoteSession | assembly | 4 | vote após `closed` |
| 6 | Company Reputation | commercial | 5 (tiers) | não-determinístico (inputs → tier) |
| 7 | User Session | identity | 6 | terminal → `active` |
| 8 | MasterPlan Activity | institutional | 7 | edição direta (só via evento Timeline) |
| 9 | Proposal | commercial | 7 | edição após `sent`; `accepted/rejected → *` |
| 10 | HarassmentReport | commercial | 6 | reabertura pós `closed` |

---

## ⚠️ Pendências de dual-check (Fase 5)

- **P-SM-001**: Subscription — legado menciona `past_due` mas não define retry_max. Stripe default é 4 tentativas em 1 semana. Confirmar.
- **P-SM-002**: VideoModeration — 48h SLA de moderação. O que acontece se admin não atua em 48h? Auto-approve? Auto-reject? Escalar?
- **P-SM-003**: Assembly — `canceled` pode gerar rescheduling (nova Assembly linkada)? Hoje tratamos como terminal; confirmar.
- **P-SM-004**: ConnectMeRequest — `evaluated` é terminal ou a empresa pode contestar avaliação? Legado: avaliação é final.
- **P-SM-005**: VoteSession — `vote_count` e `tallied` fundem? Para observabilidade mantemos separados; pra implementação simples podem colapsar.
- **P-SM-006**: Company Reputation — recálculo é **on-demand** (reactive em cada evento) ou **job noturno** (batch)? Legado sugere reactive dentro da UoW de `CompanyEvaluationSubmitted`. Confirmar custo.
- **P-SM-007**: Session — transição explícita `active → idle` é útil ou implícita por TTL do cache Redis?
- **P-SM-008**: MasterPlan `amended` — após adendo, estado "amended" substitui anterior ou vira estado histórico paralelo?
- **P-SM-009**: Proposal — `validated → rejected` após validação é possível? Legado fala de `sindico rejeita antes ou após validação`?
- **P-SM-010**: HarassmentReport — `escalated` leva aonde? Legal? DPO? Admin sênior?

---

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[invariants]]
- [[business-rules]]
- [[domain-events]]
- [[aggregates/_moc]]
