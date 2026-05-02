---
title: Assembly — Backend Requirements (Go)
type: requirement
tags: [master-sindico, backend, go, assembly, voting, quorum, livekit, feature-req]
project: master-sindico
stack: backend
bounded_context: assembly
milestone: m3
status: in-progress
created: 2026-04-24
---

# Assembly — Backend Requirements (Go)

## Escopo do bounded context

Módulo Go `internal/modules/assembly/` responsável por (MVP M3):

- **Assembly** state machine `configure → publish → start → end → homologate` (imutável pós-homologação).
- **AgendaItem** (5 tipos): informativo, consulta_moradores, votacao_simples, votacao_fracao_ideal, votacao_fornecedor.
- **Vote** UNIQUE DB `(agenda_item_id, voter_id)` — A-025 fix herdado.
- **AttendanceRecord** (check-in 3 modalidades: app QR, terminal kiosk, manual síndico).
- **Proxy** (procurações) — autorização até 24h pós-assembleia; cancelamento até T-1h.
- **ScienceRecord** — ciência obrigatória pauta (INS-027 relacionado; assembly-specific via `assembly_acknowledgments`).
- **Minutes** (ata) — auto-gen pós-encerramento; imutável pós-homologação (ASM-025).
- **LiveSession** via `ILiveProvider` Livekit — WebRTC M4+; **MVP M3 é presencial + votação assíncrona** (ASM-032).
- **EndRoomSaga** (A-028 + A-033 + A-034) — livekit end → recording S3 → cleanup.
- **Edital 8d** (INV-ASM-001) — mín 8 dias corridos Lei 4.591/64 Art. 24 §1; hard block (D-088 corrigiu bug 5d).
- **Fração ideal NUMERIC(5,4)** snapshot congelado ao publish (ASM-007).
- **Homologação escalonada** (D-053) — ordem + dependências.
- **Simulador de quorum** função pura.

**Não é responsabilidade**: reputação empresa (commercial consome `supplier_vote.session_closed`); timeline entry da ata (institutional via `ITimelinePublisher`).

## Status atual (vault canônico)

- **Sprint M3 — in progress**: aggregates delineados; domain events + state machine em spec; **código real parcial** (main classes stubs).
- **A-###** fechados (spec):
  - A-025 vote UNIQUE DB — padrão aplicado (herdado de commercial.supplier_vote).
  - A-028 EndRoom saga base delineada.
  - A-033 ILiveProvider + parceiro wire-up futuro.
  - A-034 Livekit recording → S3 sink.
- **A-### abertos**:
  - **ASM-010 procuração assinatura digital** — ADR M4+ provider.
  - **ASM-028 score transparência "fala equilibrada"** — formulação (Gini vs desvio padrão) dual-check.
  - **ASM-032 MVP M3 presencial + assíncrona** — reconciliação client vs legado.
- **D-###**: D-053 (homologação escalonada), D-088 (edital 8d Lei 4.591/64 fix), INV-118 (no override admin para edital 8d), Q-027 Fase 9 (bug 5d corrigido).

## Agregados (domain)

- [[../../../01-domain/aggregates/Assembly]] — state machine; `quorum_type`, `speech_time_minutes`, `scheduled_date`, `modality`.
- [[../../../01-domain/aggregates/AgendaItem]] — 5 tipos; `order`, `title`, `attachment_url`, `votacao_required`.
- [[../../../01-domain/aggregates/Vote]] — UNIQUE; imutável; device fingerprint + timestamp + IP.
- [[../../../01-domain/aggregates/AttendanceRecord]] — 3 modalidades; FIFO para painel president.
- [[../../../01-domain/aggregates/Proxy]] — validação procurador = morador mesmo condomínio.
- [[../../../01-domain/aggregates/Minutes]] — template HTML → PDF; assinatura M4+.
- [[../../../01-domain/aggregates/LiveSession]] — Livekit room; moderador=síndico; recording.
- `AssemblyFractionSnapshot` — congelado ao publish (ASM-007); imutável.
- `ScienceRecord` (a.k.a. `assembly_acknowledgments`) — modal bloqueante.
- `SpeechQueue` entry (FIFO cronometrado).
- `AssemblyHomologation` — pós-ata; maioria simples.
- `AssemblyLegalAcceptance` — termos 1ª assembleia.
- `TransparencyScoreSnapshot` — 10 componentes pós-homologação.
- `AssemblyReport` (6 tipos) — gen async.

## Entidades e Value Objects

| Tipo | Elemento | Invariante chave |
|---|---|---|
| Entity | `Assembly` | State machine; 1 ativa por condomínio; mín 8d antecedência (INV-ASM-001) |
| Entity | `AgendaItem` | Imutável pós publish; adendo via `previous_item_id` |
| Entity | `Vote` | UNIQUE `(agenda_item_id, voter_id)`; imutável pós envio |
| Entity | `Minutes` | Imutável pós `homologated_at NOT NULL` |
| VO | `QuorumType` | enum `{simple_majority, qualified_majority, unanimous}` |
| VO | `Modality` | enum `{presencial (M3), online (M4+), hibrida (M4+)}` |
| VO | `AssemblyType` | enum `{ordinaria, extraordinaria, ratificacao, outra}` |
| VO | `FracaoIdealSnapshot` | NUMERIC(5,4); congelado ao publish; imutável |
| VO | `SpeechDuration` | 2min | 3min configurável; extensão +1min 1x |
| VO | `CheckInMethod` | `{app, terminal, manual}` |

## Invariantes (INV-ASM-###)

1. **INV-ASM-001** (INV-118 global): Assembleia convocada **mín 8 dias corridos** antecedência (Lei 4.591/64 Art. 24 §1); **sem override admin** (D-088 Fase 9).
2. **INV-ASM-002**: 1 assembleia ativa por condomínio.
3. **INV-ASM-003**: Pauta imutável após publicação (versionamento via adendo).
4. **INV-ASM-004**: Voto único por `(agenda_item, voter)` — UNIQUE DB (A-025).
5. **INV-ASM-005**: Voto imutável pós-envio (timestamp + IP + device).
6. **INV-ASM-006**: Fração ideal NUMERIC(5,4) snapshot congelado ao publish.
7. **INV-ASM-007**: Ausência momentânea: 15 min window antes de abstenção automática.
8. **INV-ASM-008**: Ata auto-publicada na Timeline (imutável após homologação).
9. **INV-ASM-009**: Procurador não pode votar diferente do titular (titular prevalece).
10. **INV-ASM-010**: Síndico não vota (imparcialidade).
11. **INV-ASM-011**: WebSocket latência < 200ms (NFR).
12. **INV-ASM-012**: Homologação é maioria simples + imutável ata pós.

## Eventos de domínio

Publica:

- `assembly.scheduled|updated|cancelled`.
- `assembly.agenda.published` → congela fraction snapshot + trigger notificações T-3d/T-1d/T-3h.
- `assembly.acknowledgment.recorded {user_id, assembly_id}`.
- `assembly.checkin.recorded {user_id, method}` → painel president WebSocket broadcast.
- `assembly.voting.opened {session_id, item_id}` → content BC grant video visibility se supplier vote.
- `assembly.voting.closed {session_id, result}` → content revoga grants; commercial atualiza supplier vote result.
- `assembly.vote.cast {voter_id, item_id, value_hash}` — broadcast WS resultado.
- `assembly.speech.requested|approved|ended`.
- `assembly.proxy.created|revoked|used`.
- `assembly.closed {assembly_id}` → `ITimelinePublisher.Publish(type=assembly_result)` + trigger homologação.
- `assembly.minutes.generated` → signed URL.
- `assembly.homologated` → lock minutes + emit transparency score.
- `assembly.transparency_score.computed {score, components_breakdown}`.
- `assembly.live.room.created|ended` → EndRoomSaga.

Consome:

- `commercial.proposal.validated` → pode incluir em supplier vote pauta.
- `content.video.approved {company_id}` → elegível para attach to agenda item.
- `institutional.condominium.mandate_transferred` → fecha assembleias ativas.
- `identity.session.revoked` → WebSocket close 1008 no painel/participant.

## Requirements funcionais (FR-BE-ASM-###)

- **FR-BE-ASM-001** (ASM-001 + INV-ASM-001 + D-088) — *Quando* `POST /assemblies`, *o handler deve* validar `scheduled_date ≥ NOW() + 8d` **hard block** (Lei 4.591/64 Art. 24 §1); sem override admin; `< 8d` → `422 validation_error code=edital_too_short`.
- **FR-BE-ASM-002** (ASM-001) — *Quando* criar assembly, *o handler deve* validar 1 ativa por condomínio; cancelamento permitido até T-3 dias.
- **FR-BE-ASM-003** (ASM-002) — *Quando* `POST /assemblies/:id/edital`, *o handler deve* aceitar upload OU trigger job async `GenerateEditalPDF` via template HTML → wkhtmltopdf/Puppeteer → `IStorageProvider` S3.
- **FR-BE-ASM-004** (ASM-003) — *Quando* `POST /assemblies/:id/agenda-items`, *o handler deve* aceitar enum 5 tipos; mín 1, máx 20 itens; order sequential.
- **FR-BE-ASM-005** (ASM-004 + ASM-007) — *Quando* `POST /assemblies/:id/agenda/publish` (UoW), *o handler deve*: (a) set `published_at=NOW()`, lock, (b) criar `assembly_fraction_snapshots` (congelar fração ideal de todas unidades ativas), (c) publish `assembly.agenda.published`, (d) trigger notificações T-3d/T-1d/T-3h via scheduler.
- **FR-BE-ASM-006** (ASM-005) — *Quando* agenda.published, *o scheduler deve* agendar via cron horário: T-3d, T-1d, T-3h; canais `IPushProvider` + `IEmailProvider` + `ISMSProvider` (opt-in); tracking clicks.
- **FR-BE-ASM-007** (ASM-006) — *Enquanto* morador não acknowledged, *o middleware deve* responder `403 acknowledgment_required` em rotas autenticadas (exceto `/assemblies/:id/acknowledge`); tracking 6 meses.
- **FR-BE-ASM-008** (ASM-010) — *Quando* `POST /proxies`, *o handler deve* validar: (a) procurador = morador mesmo condomínio, (b) `valid_until ≤ assembly.scheduled_date + 24h`, (c) cancelamento permitido até T-1h (`revoked_at=NOW()`); durante vote `INV-ASM-009` garante titular prevalece.
- **FR-BE-ASM-009** (ASM-011) — *Quando* `GET /assemblies/:id/simulator?present_fraction=<x>`, *o handler deve* função pura sem persistência retornar `{quorum_reached, votes_needed_by_type, by_quorum_type_breakdown}`.
- **FR-BE-ASM-010** (ASM-012) — *Quando* `POST /assemblies/:id/check-in`, *o handler deve* aceitar `method ∈ {app, terminal, manual}`: app → QR scan + session token; terminal → CPF + bloco/unit + confirm; manual → só principal pode registrar (ABAC); broadcast WebSocket < 200ms (INV-ASM-011).
- **FR-BE-ASM-011** (ASM-013) — *Quando* 1ª assembly do user, *o handler deve* exigir aceite de `assembly_legal_acceptances` (versioned); modal bloqueante via interceptor.
- **FR-BE-ASM-012** (ASM-014 + ASM-015 + A-025) — *Quando* `POST /assemblies/:id/agenda-items/:item/votes`, *o handler deve* INSERT `votes` UNIQUE `(agenda_item_id, voter_id)`; conflito → `409 already_voted`; suporta 4 tipos (Simples, Fração Ideal, Fornecedor, Eleição); peso via snapshot `assembly_fraction_snapshots`.
- **FR-BE-ASM-013** (ASM-016) — *Quando* voto registrado, *o handler deve* persistir `{timestamp, ip, device_fingerprint, signature_assisted}` imutável; ausência momentânea 15min window → abstenção automática (cron fine-grained por assembly).
- **FR-BE-ASM-014** (ASM-017) — *Quando* `GET /assemblies/:id/president` (WS + HTTP), *o handler deve* validar ABAC `role_type=principal OR designated`; retornar dashboard: pauta, presentes real-time, votações abertas, fila fala, alertas (quorum crítico, tempo excedido); síndico **não vota** (INV-ASM-010).
- **FR-BE-ASM-015** (ASM-018) — *Quando* telão acessa, *o WebSocket `/ws/assemblies/:id/projection` deve* prover 2 canais: presentation (pauta atual, gráficos, vídeos empresa durante vote) e operational (quorum %, fila, alertas); mesmo connection múltiplos listeners.
- **FR-BE-ASM-016** (ASM-019) — *Quando* morador "Quero falar" (`POST /assemblies/:id/speech-queue`), *o handler deve* adicionar FIFO; síndico `POST /assemblies/:id/speech-queue/:id/approve` → timer inicia (2/3 min); alerta 30s antes; +1 min 1x; timeout → `ILiveProvider.MuteParticipant` (Livekit API).
- **FR-BE-ASM-017** (ASM-020 + A-033 + A-034) — *Quando* assembleia híbrida/online (M4+), *o handler `POST /assemblies/:id/live-room` deve* `ILiveProvider.CreateRoom` (Livekit), gerenciar participants (síndico=moderator, moradores=participant), recording → `IStorageProvider` S3; EndRoomSaga (A-028) no encerramento.
- **FR-BE-ASM-018** (ASM-021 + CNT-009) — *Quando* item `votacao_fornecedor` abre, *o handler deve* (mesma UoW): (a) criar `SupplierVoteSession` (commercial-side via evento), (b) publish `assembly.voting.opened {autorizar_exibicao_votacao=true}` → content BC emite grants temporários, (c) resultado imutável + revogação grants ao encerrar.
- **FR-BE-ASM-019** (ASM-022) — *Quando* WebSocket cai, *o client-side deve* salvar votos em Redis local via `POST /assemblies/:id/offline-votes-sync` quando reconecta; handler reprocessa idempotent via UNIQUE `(voter_id, item_id)`; conflito → último voto válido.
- **FR-BE-ASM-020** (ASM-023) — *Quando* síndico `POST /assemblies/:id/close`, *o handler deve* (UoW): (a) validar todas votações encerradas, (b) trigger job async `GenerateMinutes`, (c) status → closed, (d) publish `assembly.closed` → `ITimelinePublisher.Publish(type=assembly_result)`.
- **FR-BE-ASM-021** (ASM-024) — *Quando* `GenerateMinutes` job, *o backend deve* consolidar (data/hora/local + presentes anonimizada + pauta+resultado + falas + procurações) → template HTML → PDF via wkhtmltopdf → S3 signed URL; assinatura digital M4+.
- **FR-BE-ASM-022** (ASM-025) — *Quando* `homologated_at NOT NULL`, *o lock deve* rejeitar UPDATE em `minutes`; correção apenas via adendo em timeline.
- **FR-BE-ASM-023** (ASM-026) — *Quando* `GET /assemblies/:id/reports/:type?format=csv|pdf`, *o handler deve* gerar 6 tipos: presença (anon sem nomes), votação, procurações, fala, consolidado, transparência; acesso `syndic + admin`.
- **FR-BE-ASM-024** (ASM-027 + D-053) — *Quando* ata gerada, *o handler `POST /assemblies/:id/homologate-vote` deve* abrir votação "Aprova a ata?" maioria simples; aprovada → `homologated_at=NOW()` lock; contestação via adendo INS-016; escalonamento via ordem homologação (D-053).
- **FR-BE-ASM-025** (ASM-028 + ASM-030) — *Quando* homologada, *o job `CalcTransparencyScore` deve* calcular 0-100 com 10 componentes: edital ≥8d (10pt), edital distribuído (10pt), ciência pauta (10pt), quorum (10pt), online disponível (10pt), ata Timeline (10pt), relatórios (10pt), fala equilibrada (10pt — ASM-028 formulação pendente), homologação (10pt), sem atraso (10pt); persist `assembly_transparency_score_history`; badge via widget.
- **FR-BE-ASM-026** (ASM-031) — *Quando* estado muda (vote aberta, presença), *o backend deve* auto-save Redis `assembly:{id}:state:*`; recovery < 1min (NFR sprint 7); failover com dump periódico snapshot.

## Endpoints REST expostos

| Método | Path | Auth | Descrição | Status |
|---|---|---|---|---|
| POST | /api/v1/assemblies | Bearer + ABAC(syndic) | Criar (8d validation) | ⏳ M3 |
| GET | /api/v1/assemblies/:id | Bearer + ABAC(membership) | Detalhe | ⏳ M3 |
| POST | /api/v1/assemblies/:id/edital | Bearer + ABAC | Upload/gen PDF | ⏳ M3 |
| POST | /api/v1/assemblies/:id/agenda-items | Bearer + ABAC | Add item (5 tipos) | ⏳ M3 |
| POST | /api/v1/assemblies/:id/agenda/publish | Bearer + ABAC + UoW | Lock + snapshot fração | ⏳ M3 |
| POST | /api/v1/assemblies/:id/acknowledge | Bearer | Ciência obrigatória | ⏳ M3 |
| POST | /api/v1/assemblies/:id/check-in | Bearer | 3 modalidades | ⏳ M3 |
| POST | /api/v1/proxies | Bearer | Procuração | ⏳ M3 |
| DELETE | /api/v1/proxies/:id | Bearer | Revoke até T-1h | ⏳ M3 |
| GET | /api/v1/assemblies/:id/simulator | Bearer + ABAC | Função pura | ⏳ M3 |
| POST | /api/v1/assemblies/:id/agenda-items/:item/open-voting | Bearer + ABAC(principal) | Abre sessão | ⏳ M3 |
| POST | /api/v1/assemblies/:id/agenda-items/:item/votes | Bearer + UNIQUE | Voto | ⏳ M3 (A-025 pattern) |
| POST | /api/v1/assemblies/:id/agenda-items/:item/close-voting | Bearer + ABAC | Fecha + result | ⏳ M3 |
| POST | /api/v1/assemblies/:id/speech-queue | Bearer | FIFO | ⏳ M3 |
| POST | /api/v1/assemblies/:id/speech-queue/:id/approve | Bearer + ABAC | Timer start | ⏳ M3 |
| GET | /api/v1/assemblies/:id/president | Bearer + ABAC + WS | Painel | ⏳ M3 |
| POST | /api/v1/assemblies/:id/live-room | Bearer + ABAC | Livekit (M4+) | ⏳ M4+ |
| POST | /api/v1/assemblies/:id/close | Bearer + ABAC | Encerramento | ⏳ M3 |
| GET | /api/v1/assemblies/:id/minutes | Bearer | Ata signed URL | ⏳ M3 |
| POST | /api/v1/assemblies/:id/homologate-vote | Bearer | Aprova ata | ⏳ M3 |
| GET | /api/v1/assemblies/:id/reports/:type | Bearer + ABAC | 6 tipos | ⏳ M3 |
| GET | /api/v1/assemblies/:id/transparency-score | Bearer | Score 0-100 | ⏳ M3 |
| WS | /ws/assemblies/:id/voting | session cookie | Real-time vote broadcast | ⏳ M3 |
| WS | /ws/assemblies/:id/speech | session cookie | Fila de fala | ⏳ M3 |
| WS | /ws/assemblies/:id/projection | session cookie | Telão 2 áreas | ⏳ M3 |
| POST | /api/v1/assemblies/:id/offline-votes-sync | Bearer + idempotent | Contingência | ⏳ M3 |

## Integração com outros BCs

**Consome**:

- `ITimelinePublisher` (institutional) — `assembly.closed` → timeline entry.
- `ICondominiumIDsProvider` (institutional) — scope validation.
- `IVideoProvider` (content) — attach vídeos a agenda items.
- `ILiveProvider` (XD-014) — Livekit (M4+).
- `IQuotaCheck` (billing) — admin override lives.
- `IStorageProvider` — edital, ata, recordings.
- `IEmailProvider`, `IPushProvider`, `ISMSProvider` — notificações T-3d/T-1d/T-3h.
- `IAuditLogger`, `IEventBus`.
- `IUserModerator` (identity) — session close no painel.

**Expõe**:

- `IAssemblyReader` — `GetActive(ctx, cond_id) Assembly?`, `GetLive(ctx, assembly_id) LiveSession?`.
- `IVotingSessionCoordinator` — commercial supplier vote hooks in via events (sem interface direta).

**Publica eventos**: `assembly.scheduled|updated|cancelled`, `assembly.agenda.*`, `assembly.acknowledgment.*`, `assembly.checkin.*`, `assembly.voting.*`, `assembly.vote.*`, `assembly.speech.*`, `assembly.proxy.*`, `assembly.closed`, `assembly.minutes.*`, `assembly.homologated`, `assembly.transparency_score.*`, `assembly.live.room.*`.

**Consome eventos**: `commercial.proposal.validated`, `content.video.approved`, `institutional.condominium.mandate_transferred`, `identity.session.revoked`.

## Persistência

**Tabelas principais**:

- `assemblies` — state machine; `scheduled_date`, `modality`, `quorum_type`.
- `agenda_items` — FK `assembly_id`, `item_type` enum, `attachment_url`.
- `assembly_fraction_snapshots` — imutável; congelado ao publish.
- `votes` — UNIQUE `(agenda_item_id, voter_id)`; imutável.
- `attendance_records` — 3 methods.
- `proxies` — `authorized_by`, `proxy_user_id`, `valid_until`, `revoked_at`, `signature`.
- `assembly_acknowledgments` — ciência 6m.
- `assembly_legal_acceptances` — versioned.
- `speech_queue`, `speeches` — FIFO cronometrado.
- `minutes` — imutável pós homologação.
- `assembly_homologations`.
- `assembly_transparency_scores` — append-only histórico.
- `live_sessions` — Livekit room metadata.
- `assembly_reports` — gen async log.

**Migrations**: `00900..01099` assembly.

**RLS default-on**: `assemblies`, `votes`, `attendance_records`, `proxies`, `minutes` — policies por `condominium_id`.

## Providers externos

- **Livekit Cloud** (`ILiveProvider`) — TURN/STUN, WebRTC rooms, recording, mute API (M4+); ADR pendente self-hosted vs Cloud (custo).
- **Storage S3/R2** — edital, ata, recording, relatórios PDF.
- **wkhtmltopdf/Puppeteer** — ata, edital, reports gen.
- **FCM/Email/SMS** — notificações T-3d/T-1d/T-3h.
- **Mux** (via content BC) — attachments videos.

**Sagas**:

- **EndRoomSaga** (A-028 + A-033 + A-034) — síndico encerra → `ILiveProvider.EndRoom` → aguarda recording webhook → S3 sink → metadata persist → publish `assembly.live.room.ended`.
- **GenerateMinutesSaga** — assembly.closed → consolidar dados → template render → PDF job → S3 upload → signed URL persist → trigger homologation vote.
- **NotificationFanoutSaga** — agenda.published → schedule T-3d/T-1d/T-3h por persona; compensação: cancela jobs pendentes se assembly cancelled.

## Testes

**Coverage target**: Domain 95% (voting crítico), Application 85%, Infra 60%.

**PBT**:

- Edital 8d: `∀ scheduled_date < NOW()+8d → 422`; `∀ scheduled_date ≥ NOW()+8d → OK` (INV-ASM-001 regression).
- Vote UNIQUE: `∀ (voter, item) req ≥ 2 → 1 vote persist` (A-025 regression).
- Fração ideal snapshot: `∀ unit changes pós-publish → snapshot imutável`.
- Proxy: `∀ titular vote → proxy vote ignorado` (INV-ASM-009).
- Síndico não vota: `∀ principal syndic → vote → 403` (INV-ASM-010).
- Offline sync idempotent: `∀ re-submit offline votes → UNIQUE persiste 1`.
- Quorum simulator: `∀ present_fraction ∈ [0,1] → função pura determinística`.

**Integration** (testcontainers + Livekit mock):

- Concurrent vote race (100 req) → 1 vote persist.
- Fraction snapshot lock após publish.
- EndRoomSaga com recording webhook.
- WebSocket latency p95 < 200ms (NFR).
- Homologation vote flow end-to-end.

## Segurança específica do BC

**LGPD**:

- Presença em minutes anonimizada (só unidade/fração, não nomes).
- `lgpd_logs` em: minutes gen (purpose=legal_obligation, basis=legal), recording download (purpose=assembly_record), procuração (purpose=authorization).
- Retenção minutes: longa (legal obligation) — política 10 anos.
- Audit trail completo em todo voto, procuração, fala aprovada.

**ABAC actions canônicos**:

- `assembly.create`, `assembly.update`, `assembly.cancel`.
- `assembly.agenda.publish`.
- `assembly.preside`, `assembly.president.dashboard.read`.
- `assembly.vote.cast`, `assembly.vote.read.self`.
- `assembly.speech.request`, `assembly.speech.approve`.
- `assembly.proxy.create`, `assembly.proxy.revoke`.
- `assembly.minutes.publish`.
- `assembly.homologation.vote.cast`.
- `assembly.live.start` (M4+).

**Rate limits**:

- `POST /assemblies/:id/check-in` — 3 tentativas/min por user.
- `POST /votes` — 5/min por user (prevenção abuso).
- `POST /speech-queue` — 10/assembleia por user.
- WebSocket connections — limite 5 por user simultâneas.

**Step-up MFA** (IDN-036):

- `POST /assemblies/:id/minutes/publish` (Lei 4.591/64 compliance).

## Decisões e dívidas

**D-###**:

- D-053 (homologação escalonada com ordem/dependências) — fechado conceito; impl parcial.
- D-088 (edital 8d Lei 4.591/64 Art. 24 §1; fix bug 5d) — fechado Fase 9.
- INV-118 (no override admin para edital 8d) — fechado.
- Q-027 Fase 9 (bug 5d corrigido para 8d) — fechado.
- ADR-0030 (saga orchestration default) — aplicado EndRoomSaga + GenerateMinutesSaga.

**Pendências**:

- ASM-010 (procuração assinatura digital provider) — ADR M4+.
- ASM-020 (Livekit Cloud vs self-hosted) — ADR M4+ custo.
- ASM-024 (template ata jurídico BR) — dual-check advogado.
- ASM-028 (fala equilibrada formulação Gini vs desvio padrão) — dual-check.
- ASM-032 (MVP M3 presencial + assíncrona vs live) — reconciliação cliente vs legado.

## Gaps e bloqueadores Sprint 10 (M1)

M3 é o marco principal de assembly; M1 **não bloqueia** este BC. Entretanto, fundação deve estar pronta:

1. **Schema aggregates** — aggregates com todos os campos e constraints (inclui snapshot fraction).
2. **UNIQUE vote constraint** — padrão A-025 reutilizado.
3. **`ITimelinePublisher` consumer** — assembly.closed → timeline entry.
4. **`ICondominiumIDsProvider` consumer** — scope resolution.
5. **Edital 8d validation** — INV-118 hard block implementado em domain layer.
6. **WebSocket scaffolding** — ASM-017, ASM-018, ASM-019 canais delineados.
7. **Provider interface `ILiveProvider`** — wired pra Livekit SDK (M4+).

Sprint M3 lockdown inclui: voting tipos 4, painel síndico, speech queue, minutes gen, homologation, transparency score.

## Ligações

- [[../../../04-requirements/functional/assembly]] — requirement canônico global
- [[../../../01-domain/aggregates/Assembly]]
- [[../../../01-domain/aggregates/AgendaItem]]
- [[../../../01-domain/aggregates/Vote]]
- [[../../../01-domain/aggregates/Proxy]]
- [[../../../01-domain/aggregates/Minutes]]
- [[../../../01-domain/aggregates/LiveSession]]
- [[../../../01-domain/aggregates/_moc]]
- [[../../../02-architecture/adr/0030-saga-orchestration-default]]
- [[../../../11-audit/AUDIT]]
- [[../architecture]]
- [[../patterns]] — Saga, UoW, Repository, WebSocket handler abstraction
- [[../security]]
- [[../tasks]] — Sprint M3 lockdown
- Correspondente web: [[../../web/requirements/assembly]]
- Correspondente mobile: a definir

## Links
- [[../../../_moc]]
- [[../../../CLAUDE]]
