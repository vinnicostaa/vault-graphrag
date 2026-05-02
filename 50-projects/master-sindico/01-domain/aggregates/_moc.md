---
title: MOC — 01-domain/aggregates
type: moc
tags:
  - moc
  - master-sindico
  - v2
  - domain
  - aggregate
  - ddd
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
---

# MOC — 01-domain/aggregates

Um arquivo curto (1 página) por aggregate raiz do domínio. Cada arquivo contém: entidade raiz, VOs, invariantes, factory `New/Reconstruct`, repository interface (Go signature), eventos emitidos.

**Convenção** (STEERING §7):
- Aggregates com estado **privado** (campos não-exportados em Go) + getters explícitos
- Factories: `NewX(...)` valida; `ReconstructX(...)` sem validação (vem do repo)
- Repository interface em `domain/`; impl em `infrastructure/`
- Mapper explícito entre row/aggregate

---

## identity (2 aggregates)

- [[User]] — identidade única por CPF/CNPJ; mirror local do Zitadel
- [[Session]] — sessão ativa com device fingerprint + IP

---

## billing (3 aggregates)

- [[Subscription]] — assinatura Stripe mapeada
- [[Plan]] — definição de plano (tier, features, quotas)
- [[FeatureUsage]] — contador consumo vs quota

---

## institutional (9 aggregates)

- [[Condominium]] — tenant primário (kind=`condominium`, D-126)
- [[Block]] — **NOVO 2026-04-25** (D-128); bloco do condomínio (opcional); aggregate root próprio; identifier polimórfico (letter/number/named); arquivamento independente
- [[Unit]] — unidade com fração ideal; FK opcional pra Block (D-128)
- [[Membership]] — vínculo user↔condo↔(unit)↔role; **flags de mandato** consolidadas aqui (D-129); suporta role efêmera `assembly_auditor` com `expires_at` (D-132)

## institutional — aggregates extras (institutional BC)

> Estes aggregates pertencem a `institutional` (não commercial — correção D-105/D-131). [[Adjustment]] foi movido para `cross-domain` em D-131.

- [[ServiceRecord]] (stub) — registro execução serviço via Connect Me; aprovação gera TimelineEntry (BC: institutional via validation hub)
- [[TechnicalActivity]] (stub) — atividade técnica avulsa; histórico interno empresa
- [[TimelineEntry]] — append-only imutável
- [[MasterPlan]] — Plano Diretor plurianual
- [[Announcement]] — comunicado oficial imutável pós-publish
- [[Event]] — evento condominial (reunião, obra, festa)
- [[Poll]] — enquete informal

---

## commercial (9 aggregates)

- [[Company]] — empresa com CNPJ
- [[ConnectMeRequest]] — RFP unidirecional (4 fluxos)
- [[Proposal]] — resposta de empresa; imutável pós-sent
- [[ClosedDeal]] — contrato fechado; imutável pós-dupla-assinatura
- [[CompanyEvaluation]] — avaliação pós-contrato; UNIQUE (company, evaluator, condo)
- [[SupplierVoteSession]] — votação colegiada
- [[HarassmentReport]] — denúncia anti-assédio
- [[Partner]] — Vizinhança (comércio local)

---

## content (5 aggregates)

- [[Video]] — Mux-backed; lock 90d pós-publish
- [[VideoModeration]] — estado de aprovação/rejeição
- [[Course]] — LMS curso com modules/lessons/quiz
- [[Enrollment]] — matrícula user↔course
- [[Certificate]] — emitido ao 100% conclusão

---

## assembly (6 aggregates + 2 entities)

> [[../../STATE#D-130]] (2026-04-25): Vote rebatizado de aggregate root → entity de AgendaItem. [[../../STATE#D-127]]: AssemblyAdministratorLink novo como entity de Assembly. [[../../STATE#D-133]]: LiveSession permanece em assembly (Livekit conferência); Lives broadcast vão pra novo aggregate `BroadcastLive` no `content` BC.

- [[Assembly]] — assembleia com agenda + attendance + votes + minutes (raiz coordena entities filhas)
- [[AgendaItem]] — item de pauta (aggregate root); coordena Vote entity via `CastVote()`/`Homologate()`
- [[Minutes]] — ata imutável pós-publish
- [[LiveSession]] — sessão Livekit (conferência N↔N — Assembly online/híbrida)
- [[Proxy]] — procuração 100% digital (D-104 + Lei 14.063); validade 24h pós-encerramento
- [[AttendanceRecord]] — check-in (app/terminal/manual)

### Entities de Assembly (não aggregate roots)

- [[Vote]] — entity de [[AgendaItem]] (D-130 2026-04-25); UNIQUE `(agenda_item_id, voter_id)`; mantém `IVoteRepository` por escala (10k+ votos por assembleia grande); invariantes via AgendaItem
- [[AssemblyAdministratorLink]] — entity de [[Assembly]] (D-127 2026-04-25); cargo cedido por-assembleia; auto-invalida em `Assembly.End()`

---

## cross-domain (8 aggregates) — D-131 2026-04-25 adicionou Adjustment

Agregados transversais, não exclusivos de um BC — usados por infra + observability + security + LGPD.

- [[DomainEvent]] — interface canônica de evento de domínio + metadata (outbox pattern, UUIDv7, correlation)
- [[AbacDecision]] — cache entry + outcome auditável da engine ABAC (INV-086..INV-089)
- [[AuditEntry]] — trilha LGPD/governança append-only; retenção 5 anos; hard-delete pós-expiry
- [[OutboxEntry]] — estado de publicação outbox (pending/published/failed/dlq); at-least-once via NATS JetStream
- [[WebhookEvent]] — ledger de idempotência de webhooks inbound (Stripe/Mux/Livekit/Zitadel) UNIQUE (provider, event_id)
- [[RateLimitEntry]] — estado de token bucket por (subject, scope); Redis atomic + fallback memory
- [[SessionState]] — view-model transversal do lifecycle de sessão (device, risk, MFA, ABAC/ratelimit keys)
- [[Adjustment]] — **MOVIDO 2026-04-25** (D-131); adendo formal cross-BC (timeline_entry/closed_deal/master_plan_action/ata_minutes/announcement/agenda_item); único mecanismo de modificação sobre conteúdo imutável; cadeia profundidade máxima 5 (INV-ADJ-006)

---

## Total: **42 aggregates** canônicos + **2 entities filhas** explicitadas

Distribuição (atualizada 2026-04-25 D-126..D-135):

| BC | Count | Δ |
|---|---|---|
| identity | 2 | — |
| billing | 3 | — |
| institutional | 9 (+ 7 extras institucional) | **+1 Block (D-128)** |
| commercial | 9 (Adjustment movido) | **−1 Adjustment** |
| content | 5 (BroadcastLive M2 — D-133) | (+ 1 futuro BroadcastLive) |
| assembly | 6 + 2 entities (Vote, AssemblyAdministratorLink) | **−1 Vote** (rebatizado entity) |
| cross-domain | 8 | **+1 Adjustment** |
| **total** | **42 aggregates + 2 entities** | netto **+1** |

> **Não-aggregates (2026-04-25)**:
> - **Mandate** — projeção via flags em [[Membership]] (D-129); não é aggregate
> - **Vote** — entity de [[AgendaItem]] (D-130); não é aggregate root mais
> - **AssemblyAdministratorLink** — entity de [[Assembly]] (D-127); não é aggregate root

---

## Stubs em destilação (2026-04-27 — D-vault-025)

20 stubs criados pra fechar wikilinks órfãos em ui-catalog/jornadas. Frontmatter `status: stub`. Promoção a destilado completo quando BC respectivo entrar em sprint dedicado.

| Stub | BC | Origem | Inbound | Disputa |
|---|---|---|---:|---|
| [[Curriculum]] | commercial | Banco de Talentos (sub-produto 07) | 16 | — |
| [[MasterPlanAction]] | institutional | sub-aggregate de [[MasterPlan]] | 9 | — |
| [[Promotion]] | commercial | Vizinhança (sub-produto 08) — D-064/D-078 | 11 | — |
| [[LegalAcceptance]] | cross-domain | aceite legal append-only | 12 | — |
| [[EmpresaProfile]] | commercial | apresentação de [[Company]] | 4 | dispute: pode ser merged em Company |
| [[ExecutionRecord]] | commercial | registro execução pós-ClosedDeal | 3 | dispute: similar a [[ServiceRecord]] |
| [[AgenciaLink]] | commercial | delegação Company → agência marketing | 8 | — |
| [[Profile]] | identity | perfil público de [[User]] | 3 | dispute: pode virar VO de User |
| [[MoradorQuestion]] | institutional | pergunta não-vinculante | 2 | — |
| [[MarketingLink]] | commercial | delegação publicação marketing | 2 | dispute: merge em [[AgenciaLink]] (ambos delegação Company→agência) |
| [[EmpresaVideo]] | content | vídeo institucional Company | 2 | dispute: kind especial de [[Video]] |
| [[PromotionActivation]] | commercial | ativação cupom por morador | 5 | — |
| [[GovernanceMarkers]] | institutional | marcadores declaratórios condomínio | 2 | dispute: pode virar VO de [[Condominium]] |
| [[ResponsibilityTerm]] | cross-domain | termo responsabilidade técnica síndico | 1 | dispute: variante de [[LegalAcceptance]] kind=tech_responsibility |
| [[ManagementEvaluation]] | institutional | avaliação anônima da gestão | 1 | — |
| [[EmpresaUser]] | commercial | user vinculado a Company | 1 | dispute: provável [[Membership]] kind=empresa_user |
| [[Account]] | identity | conta de operação | 1 | dispute: clarificar vs [[User]] |
| [[CompanyTag]] | commercial | tag/categoria de Company | 2 | dispute: provável VO, não aggregate |
| [[ContactRequest]] | commercial | revelação contato Banco de Talentos | 1 | dispute: variante de [[ConnectMeRequest]] |
| [[BroadcastLive]] | content | Lives broadcast 1→N (D-133) | 2 | — |

**Total stubs**: 20 (12 sem disputa + 8 com `dispute:` para resolução em sprint).

## Vizinhos

- [[../_moc]] — MOC de 01-domain
- [[../bounded-contexts]]
- [[../ubiquitous-language]]
- [[../invariants]]
- [[../domain-events]]
- [[../business-rules]]
- [[../state-machines]]
- [[../../STATE]] — decisões vivas
- [[../../CLAUDE]] — contrato do agente
