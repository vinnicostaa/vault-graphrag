---
title: "Gap Analysis — Domínio (telas ui-catalog vs 01-domain + enums canônicos)"
type: audit
tags: [audit, ddd, gaps, domain, ui-catalog, invariants, aggregates, enums, events, vos, master-sindico, v2]
source: "03-subprojects/web/ui-catalog/ (192) + 03-subprojects/mobile/ui-catalog/ (59); 01-domain/{invariants,aggregates,domain-events,ubiquitous-language}.md; 90-archive/from-development-content-2026-04-23/04-Listas-Mestre/Enums-Consolidados.md"
created: 2026-04-24
updated: 2026-04-24
milestone: M1 (2026-05-08)
---

# Gap Analysis — Domínio (telas vs 01-domain + enums canônicos)

**Escopo auditado**: 192 MDs em `03-subprojects/web/ui-catalog/` + 59 MDs em `03-subprojects/mobile/ui-catalog/` = **251 telas**.
**Fontes canônicas**: `01-domain/invariants.md` (116 INVs), `01-domain/aggregates/` (40 arquivos), `01-domain/domain-events.md` (82 E-###, 72 PascalCase names), `01-domain/ubiquitous-language.md` (25 VOs, 23 Enums, 47 Aggregate Roots declarados), `90-archive/…/Enums-Consolidados.md` (19 categorias, 90+ valores).

**Convenção severidade** (RACI M1 = 2026-05-08; BCs core = identity + billing + institutional):

- 🔴 **CRITICAL**: INV/aggregate/enum/evento é consumido em tela M1 e a ausência gera bug funcional ou inconsistência jurídica/LGPD.
- 🟠 **HIGH**: M2 (assembleia ao vivo, connect-me, conteúdo).
- 🟡 **MEDIUM**: M3 (LMS, banco de talentos, vizinhança, marketing).
- 🟢 **LOW**: documentação incompleta sem impacto funcional imediato.

---

## Sumário

| Tipo | Citados em telas | Canônicos | Órfãos reais | 🔴 | 🟠 | 🟡 | 🟢 |
|---|---|---|---|---|---|---|---|
| **Invariantes** (`INV-###`, `INV-<BC>-X`) | 18 distintos | 116 | **10 órfãos** (INV-BT-* + INV-ASM-* + INV-LMS-E, não formalizados em `invariants.md`) | 0 | 1 | 9 | 0 |
| **Aggregates** | ~14 referenciados por nome | 40 (arquivo); 47 declarados no UL | **13 órfãos** (ValidationItem, Curriculum, VideoVisibilityGrant, NeighborhoodBenefit, ForumTopic/Post, LibraryMaterial, Live (LMS), LGPDLog, ComplianceDeclaration, TermAcceptance, Amendment, SpeechQueue, PreliminarySurvey) | 2 | 4 | 6 | 1 |
| **Enums** | 4 nominais + 2 quantificados | ~90 valores em 19 categorias + 13 tipos evento + 12 vídeo + 4 live + 6 material biblioteca + 7 fórum + 5 promoção | **1 divergência real** (18+ tipos evento no briefing vs 13 canônicos) + S24 "consolidar enum" pendente | 0 | 0 | 1 | 1 |
| **Eventos de domínio** | 3 E-### canônicos + 4 nomes PascalCase novos | 82 E-### / 72 nomes | **4 órfãos reais** (`ForumTopicCreated`, `ForumReplyPosted`, `LibraryMaterialPublished`, `TrainingCompleted`) — todos LMS | 0 | 0 | 4 | 0 |
| **Value Objects** | CPF(58), CNPJ(42), Email(10) + impl `mux/playback/locked_until/device_fingerprint` | 25 VOs | **0 órfãos** — VOs citados existem todos no UL; lacuna é sub-citação (FracaoIdeal/Money/CouponCode ausentes da UI) | 0 | 0 | 0 | 1 |

**Nota metodológica**: INV-BT-01/02/04/05/09/10, INV-ASM-events, INV-ASM-live, INV-ASM-modalidade-mvp e INV-LMS-E são **nomes simbólicos** usados em telas mas sem entrada em `01-domain/invariants.md` (que usa apenas numeração sequencial `INV-001..123`). Não há gap de regra — há gap de **registro canônico + mapeamento**. Mesma coerência existe entre os 7 aggregates declarados no `ubiquitous-language.md` como "Aggregate Root" sem arquivo correspondente em `01-domain/aggregates/` (dívida de templating, não de modelo).

---

## A. Invariantes órfãos

Órfão em **`01-domain/invariants.md`** = INV citado na tela com um nome **específico** que NÃO aparece em `01-domain/invariants.md`. **Descoberta**: vários desses INVs existem em `00-product/sub-produtos/**.md` com a nomenclatura derivada (`INV-<BC>-X`), mas **não foram promovidos** para a lista canônica de invariantes formalizados (que usa numeração sequencial `INV-001..INV-123`). Isso quebra a promessa de que `invariants.md` é a fonte única de regras testáveis.

### A.1. Série INV-BT-* (Banco de Talentos) — **EXISTEM em `00-product/sub-produtos/07-banco-de-talentos.md`** (linhas 645-654, INV-BT-01..10) mas NÃO promovidos para `01-domain/invariants.md`

| INV citado em tela | Telas | Fonte canônica atual | Gap | Severidade | Ação |
|---|---|---|---|---|---|
| **INV-BT-01** | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT02]] | 00-product/sub-produtos/07-banco-de-talentos.md:645 | Não promovido a invariants.md | 🟡 M3 | Promover como `INV-124` em seção **banco-talentos** de invariants.md preservando o alias `INV-BT-01` no campo "legado". |
| **INV-BT-02** | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT02]] | 00-product/.../07:646 | Não promovido | 🟡 M3 | Promover como `INV-125`. Mesma semântica de INV-056 (lock 90d vídeo institucional) — avaliar unificação na promoção. |
| **INV-BT-04** | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT10]] | 00-product/.../07:648 | Não promovido | 🟡 M3 (LGPD crítico) | Promover como `INV-126`. |
| **INV-BT-05** | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT01]], [[../../03-subprojects/web/ui-catalog/banco-talentos/_moc]] | 00-product/.../07:649 | Não promovido | 🟡 M3 | Promover como `INV-127`. |
| **INV-BT-09** | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT04]] | 00-product/.../07:653 | Não promovido | 🟡 M3 | Promover como `INV-128`. |
| **INV-BT-10** | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT07]] | 00-product/.../07:654 | Não promovido | 🟡 M3 | Promover como `INV-129`. |

**Nota**: `INV-BT-03`, `INV-BT-06`, `INV-BT-07`, `INV-BT-08` existem em 00-product mas NÃO são citados nas telas — ainda assim devem subir pra invariants.md ao promover a série.

### A.2. Série INV-LMS-* — **EXISTEM em `00-product/sub-produtos/06-lms-academia.md`** (linhas 560-565, INV-LMS-A..F) mas NÃO promovidos

| INV citado em tela | Telas | Fonte canônica atual | Gap | Severidade | Ação |
|---|---|---|---|---|---|
| **INV-LMS-E** | [[../../03-subprojects/web/ui-catalog/lms/LMS11]], [[../../03-subprojects/web/ui-catalog/lms/LMS12]], [[../../03-subprojects/web/ui-catalog/lms/LMS13]] | 00-product/sub-produtos/06-lms-academia.md:564 (marcado "derivada"; é regra ABAC, não invariante DDD puro) | Não promovido | 🟡 M3 | **Reclassificar como ABAC policy** em `02-architecture/abac-matrix.md` (melhor fit: ação `forum.topic.create` exige `role in {syndic, enterprise, admin}`) **OU** promover como `INV-131` se decidir manter dupla notação. Preferência: reclassificar — economiza 1 INV. |

**Série completa `INV-LMS-A..F`** também não promovida; só `INV-LMS-E` aparece em telas M3.

### A.3. Série INV-ASM-* — **NÃO** encontrada em `00-product/sub-produtos/` (busca grep negativa)

Esses são **novos** inventados pelas telas de assembleia — sem fonte canônica em lugar nenhum:

| INV citado | Telas | Fonte canônica | Classificação | Severidade | Ação |
|---|---|---|---|---|---|
| **INV-ASM-events** / **INV-ASM-events-append-only** | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-PAINEL-SIND]] (linha 85), [[../../03-subprojects/web/ui-catalog/assembleia/_moc]] (linha 69) | **Nenhuma** | Invariante de domínio **novo** | 🟠 M2 | Formalizar como `INV-130`. Semanticamente sobrepõe com INV-033 (audit append-only), INV-090 (audit-trail), INV-119 (DB-level immutability timeline+minutes), INV-120 (audit DB-level). **Decisão necessária**: é promoção de INV-119/120 estendendo pra `assembly_events` OU regra nova com enforcement próprio? Default: novo INV-130 com REVOKE+trigger análogo. |
| **INV-ASM-live** / **INV-ASM-live-<200ms** | [[../../03-subprojects/web/ui-catalog/assembleia/_moc]] (linha 68) | **Nenhuma** | SLO, NÃO invariante de domínio | 🟠 M2 | **Não promover**. Mover pra `07-quality/slo.md` OU `02-architecture/nfr.md` como SLO P95=200ms em WS assembleia. Remover citação como "INV" das telas para evitar falsa categorização. |
| **INV-ASM-modalidade-mvp** | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CFG]] (linha 70) | **Nenhuma** | Restrição de escopo M1, NÃO invariante permanente | 🟠 M2 | **Não promover**. Registrar em `10-schedule/milestones.md` seção "M1 exclusions" (modalidades `híbrida`/`online` travadas no backend M1, liberadas M2). Remover citação como "INV" de ASM-CFG. |

### A.4. INVs canônicos citados corretamente (confirmações — não são gaps)

| INV citado | Telas | Presente em invariants.md? |
|---|---|---|
| INV-067 (parceiro vizinhança sem LMS) | LMS1, LMS5, LMS11, vizinhanca/_moc, lms/_moc | ✅ linha 118 |
| INV-068 (Certificate auto-gen ao 100%) | LMS5 | ✅ linha 119 |
| INV-069 (Ebook access por plan_tier) | LMS14, LMS15 | ✅ linha 120 |
| INV-070 (signed playback TTL ≤24h) | LMS4, BT02 | ✅ linha 121 |
| INV-088 (Admin bypass audit obrigatório) | ASM-REL, ASM-CIEN, ASM-PROC-VAL | ✅ linha 153 |
| INV-091 (LGPD retenção 5 anos) | ASM-TERM, ONB-M4, ONB-S6 | ✅ linha 156 |
| INV-092 (PII nunca em logs) | ASM-REL | ✅ linha 157 |

### A.5. Resumo seção A

| Métrica | Valor |
|---|---|
| INVs citados únicos nas telas | 18 |
| INVs com entrada em invariants.md | 7 (39%) |
| INVs em 00-product/sub-produtos/ não promovidos | 7 (INV-BT-01/02/04/05/09/10 + INV-LMS-E) |
| INVs inéditos (sem fonte em nenhum lugar canônico) | 3 (INV-ASM-events, INV-ASM-live, INV-ASM-modalidade-mvp) |
| Dos inéditos: reais invariantes de domínio | 1 (INV-ASM-events) |
| Dos inéditos: devem virar SLO/escopo/ABAC | 2 (INV-ASM-live, INV-ASM-modalidade-mvp) |

**Ações totais A**: 7 promoções + 1 formalização nova + 2 reclassificações (SLO + escopo) + 1 reclassificação ABAC (LMS-E).

---

## B. Aggregates órfãos

Órfão = aggregate referenciado por nome em tela ou em `ubiquitous-language.md` como "Aggregate Root" SEM arquivo correspondente em `01-domain/aggregates/`.

| Aggregate citado | Telas que referem | BC esperado | Declarado em UL? | Arquivo existe? | Severidade | Ação |
|---|---|---|---|---|---|---|
| **ValidationItem** | [[../../03-subprojects/web/ui-catalog/sindico/S2]] (badge), [[../../03-subprojects/web/ui-catalog/sindico/S21]], [[../../03-subprojects/web/ui-catalog/sindico/S22]], [[../../03-subprojects/web/ui-catalog/sindico/S26]] | institutional / compliance | ✅ (Aggregate Root) | ❌ ausente | 🔴 **CRITICAL M1** (é tela M1 S2/S21/S22/S26) | Criar `01-domain/aggregates/ValidationItem.md` URGENTE. S21+S22+S26 são fluxo síndico validação de execução — core institutional M1. |
| **Curriculum** | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT01]]..BT11, `_moc` | commercial / banco-talentos | ✅ (Aggregate Root) | ❌ ausente | 🟡 M3 | Criar `01-domain/aggregates/Curriculum.md`. BT só ativa M3. |
| **VideoVisibilityGrant** | [[../../03-subprojects/web/ui-catalog/assembleia/_moc]] (regra 4), citado em INV-062/INV-063 | content / assembly | ✅ (Aggregate Root) | ❌ ausente | 🟠 M2 (votação de fornecedor) | Criar `01-domain/aggregates/VideoVisibilityGrant.md`. Pressuposto por INV-062/063 — coerência interna quebrada. |
| **NeighborhoodBenefit** (Cupom) | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS7]], [[../../03-subprojects/web/ui-catalog/vizinhanca/VZ3]], etc — lock 4h INV-051 | commercial / vizinhança | ✅ (Aggregate Root) | ❌ ausente | 🟡 M3 | Criar `01-domain/aggregates/NeighborhoodBenefit.md`. Pressuposto por INV-051..053. |
| **ForumTopic** + **ForumPost** | [[../../03-subprojects/web/ui-catalog/lms/LMS11]], [[../../03-subprojects/web/ui-catalog/lms/LMS12]], [[../../03-subprojects/web/ui-catalog/lms/LMS13]] — explicitamente citado como gap G-01 pela tela | content / LMS | ✅ (ambos, ForumTopic AR + ForumPost Entity) | ❌ ausente | 🟡 M3 | Criar `01-domain/aggregates/ForumTopic.md`. LMS11/13 já marcam como gap G-01. |
| **LibraryMaterial** | [[../../03-subprojects/web/ui-catalog/lms/LMS14]], [[../../03-subprojects/web/ui-catalog/lms/LMS15]] — explicitamente gap G-02 | content / LMS | ❌ (só `Ebook` no UL) | ❌ ausente | 🟡 M3 | Decidir: criar `LibraryMaterial.md` (superconjunto: PDF/video/ebook/template) OU reutilizar `Ebook` + `Video`. LMS14/15 assumem agregado único. |
| **Live** (LMS, distinto de LiveSession de Assembly) | [[../../03-subprojects/web/ui-catalog/lms/LMS8]], LMS9, LMS10, `_moc` — explicitamente gap G-03 | content / LMS | ❌ | ❌ ausente | 🟡 M3 | Criar `01-domain/aggregates/LMSLive.md` (nome sugerido para desambiguar do `LiveSession.md` de assembly). ADR CNT-022 (Mux Live vs Cloudflare) pendente. |
| **LGPDLog** | [[../../03-subprojects/web/ui-catalog/sindico/S19]] ("Connect Me lgpd_logs"), INV-048, INV-091 | cross-domain / commercial | ✅ (Aggregate Root append-only) | ❌ ausente (existe `AuditEntry.md` mas genérico) | 🟠 M2 (Connect Me M2) | Criar `01-domain/aggregates/LGPDLog.md` OU documentar especialização explícita de AuditEntry via `purpose='lgpd_reveal'`. |
| **ComplianceDeclaration** | [[../../03-subprojects/web/ui-catalog/compliance/C2]], C4, C5, C6, INV-034, INV-035 | institutional / compliance | ✅ (Aggregate Root) | ❌ ausente | 🔴 **CRITICAL M1** (compliance = M1 core) | Criar `01-domain/aggregates/ComplianceDeclaration.md`. C2 é tela M1 de assinatura; INV-034/035 garantem existência. |
| **TermAcceptance** | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-TERM]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-M4]], [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S6]], Req 35 | identity / LGPD | ✅ (Aggregate Root) | ❌ ausente | 🟠 M2 (onboarding M1 usa) | Criar `01-domain/aggregates/TermAcceptance.md`. Onboarding M1 registra aceite de termos — sem aggregate formal = risco jurídico. |
| **Amendment** (Adendo) | INV-050, UL explícito como "único mecanismo de modificação de registro imutável" (Req 25) | cross-domain (commercial/institutional/assembly) | ✅ (Aggregate Root) | ❌ ausente | 🟠 M2 | Criar `01-domain/aggregates/Amendment.md`. Sem aggregate, INV-050 fica conceitual. |
| **SpeechQueue** | [[../../03-subprojects/web/ui-catalog/assembleia/_moc]] (fila de fala cronometrada) | assembly | ✅ (Aggregate Root) | ❌ ausente | 🟠 M2 (assembleia ao vivo M2) | Criar `01-domain/aggregates/SpeechQueue.md`. |
| **PreliminarySurvey** (Enquete Preliminar Req 51) | derivado de UL (sem tela explícita, mas referido em Req 51) | assembly | ✅ (Aggregate Root) | ❌ ausente | 🟡 M3 | Criar `01-domain/aggregates/PreliminarySurvey.md` ou decidir se especialização de `Poll`. |
| **ComplianceAssessment**, **ConflictOfInterestDeclaration**, **RiskMap**, **CompanyAssessment**, **SearchIndex**, **Ebook**, **GovernanceScoreSnapshot** | declarados em UL, referenciados em INVs/Req mas sem tela direta hoje | diversos | ✅ todos | ❌ ausentes | 🟢 LOW | Templating tardio — criar stubs nos aggregates/ com apenas frontmatter + link pra UL. |

**Aggregates canônicos citados em tela E presentes em `01-domain/aggregates/`** (ok): User, Subscription, Condominium, Unit, Membership, TimelineEntry, MasterPlan (+MasterPlanActivity — ver G), Announcement, Event, Assembly, AgendaItem, Vote, Minutes, LiveSession, Proxy, AttendanceRecord, Company, ConnectMeRequest, Proposal, ClosedDeal, SupplierVoteSession, Partner, Video, VideoModeration, Course, Enrollment, Certificate.

**Total órfãos B**: 13 aggregates (+ 7 em LOW). **2 🔴 M1**, **4 🟠 M2**, **6 🟡 M3**, **1 🟢 LOW**.

---

## C. Enums órfãos

Órfão = enum citado em tela e NÃO presente em `Enums-Consolidados.md` + `05-enums-regras-quebras.md` + `01-domain/enums/`.

| Enum citado (ou referência numérica) | Telas | BC | Canônico existe? | Divergência? | Severidade | Ação |
|---|---|---|---|---|---|---|
| "18+ tipos de evento" | [[../../03-subprojects/web/ui-catalog/sindico/S16]] (título + descrição + linha 82), [[../../03-subprojects/web/ui-catalog/sindico/_moc]] | institutional | ✅ **13 tipos** em `Enums-Consolidados §6 Tipos de Evento (13)` | 🔶 **DIVERGÊNCIA** — briefing .ini lista 18+, backend tem 13 | 🟡 MEDIUM | S16 já registra "divergência; consolidar enum". **Decisão de produto**: aceitar os 13 canônicos e extender pra 18+? Ou os 5 a mais são **tipos de comunicado** (8) que estão sendo confundidos com eventos? Ver S24 (tipos comunicado vs evento sobrepostos). Resolver em sprint de domínio pré-M2. |
| "tipos de comunicado vs eventos sobrepostos" | [[../../03-subprojects/web/ui-catalog/sindico/S24]] (linha 82: "Lista de tipos de comunicado vs eventos sobrepostos — consolidar enum") | institutional | ✅ `Tipos (8 — canônico requirements.md)` + `Tipos pós-deal (5)` + `Tipos de Evento (13)` | 🔶 **AMBIGUIDADE** — mesmo tipo pode aparecer como Announcement e Event | 🟢 LOW | Em `01-domain/ubiquitous-language.md` clarificar matriz Announcement vs Event. Não é gap de enum; é gap de limites de agregado. |
| `LiveType` (4 valores LMS) | LMS8, LMS9, LMS10 | content / LMS | ✅ `Tipos de Live (4)` em Enums-Consolidados §LMS | ✅ coerente | 🟢 OK | — |
| `LibraryMaterialType` (6 valores) | LMS14 | content / LMS | ✅ `Tipos de Material da Biblioteca (6)` | ✅ coerente | 🟢 OK | — |
| `ForumCategory` (7 valores) | LMS11, LMS12 | content / LMS | ✅ `Categorias de Fórum (7)` | ✅ coerente | 🟢 OK | — |
| `PromotionType` (5 tipos) | VS7, VZ3 | commercial / vizinhança | ✅ `Tipos de Promoção S→Parceiro (5)` | ✅ coerente | 🟢 OK | — |
| `12 tipos de vídeo` | MK4 (com enumeração explícita) | content | ✅ `Tipos de Vídeo Institucional (12)` | ✅ coerente (rótulos batem 1-para-1) | 🟢 OK | — |

**Total órfãos C**: **0 órfãos reais**. 1 divergência numérica (18+ vs 13 tipos evento) + 1 ambiguidade de limite (Announcement vs Event).

---

## D. Eventos de domínio não declarados

Órfão = evento citado pela tela (por nome PascalCase no formato passado: `XxxYyyed`) mas AUSENTE de `01-domain/domain-events.md`.

| Evento citado | Publicador esperado | Consumidor (tela) | Canônico? | Severidade | Ação |
|---|---|---|---|---|---|
| **`ForumTopicCreated`** | ForumTopic (aggregate também ausente) | [[../../03-subprojects/web/ui-catalog/lms/LMS11]] (linha 84 marca como G-04), LMS13 | ❌ ausente em `domain-events.md` | 🟡 M3 | Adicionar como `E-083` quando aggregate ForumTopic for criado (ver B). |
| **`ForumReplyPosted`** | ForumTopic | LMS11, LMS13 | ❌ ausente | 🟡 M3 | Adicionar como `E-084`. |
| **`LibraryMaterialPublished`** | LibraryMaterial (aggregate também ausente) | [[../../03-subprojects/web/ui-catalog/lms/LMS14]] (linha 92 G-04) | ❌ ausente | 🟡 M3 | Adicionar como `E-085`. |
| **`TrainingCompleted`** | Enrollment/Training | [[../../03-subprojects/web/ui-catalog/lms/LMS7]] (linhas 61+69: "sugerido — não formalizado em domain-events; gap G-04") | ❌ ausente | 🟡 M3 | Decidir: é mesmo evento que `LessonCompleted`? Ou é evento agregado de conclusão de **trilha** de treinamentos? Se distinto, `E-086`. |

**Eventos citados que EXISTEM no canônico** (ok — não são gaps): `CertificateGenerated` (E-061), `LessonCompleted`, `CourseCompleted`, `E-059`, `E-060`. Eventos com sufixos `-ed` detectados nos TS/Dart são `assembly.closed` (via NATS), `InterestExpressed` (handler INV-048), `VotingClosed`, `ReputationTierChanged`, `ClosedDealSigned`, `ExecutionValidated` — todos em `domain-events.md`.

**Falsos positivos filtrados** (não são eventos de domínio): `RefreshRequested` (BLoC event UI), `AlreadySubmitted` (estado UI), `LocationDenied`/`PermanentlyDenied`/`PermissionDenied` (enums de permissão do device).

**Total órfãos D**: **4 eventos** — todos em telas LMS (M3), todos já marcados pela tela como G-04. Zero crítico M1.

---

## E. Value Objects não catalogados

| VO/conceito citado | Telas | Canônico (`ubiquitous-language.md`)? | Ação |
|---|---|---|---|
| CPF (58 refs) | 58 telas web/mobile | ✅ VO `CPF` (com `Masked()`) | 🟢 OK |
| CNPJ (42 refs) | 42 telas | ✅ VO `CNPJ` (com `Masked()`) | 🟢 OK |
| Email (10 refs) | 10 telas | ✅ VO `Email` | 🟢 OK |
| `device_fingerprint` (2 refs contextuais) | [[../../03-subprojects/web/ui-catalog/onboarding/ONB-S2]] etc | ✅ VO `DeviceFingerprint` | 🟢 OK |
| `locked_until` (4 refs) | BT02, Video refs | ✅ VO `LockedUntil` | 🟢 OK |
| `coupon_code` (1 ref) | VZ/VS | ✅ VO `CouponCode` | 🟢 OK |
| `mux`/`playback` refs (2+3) | Video/LMS | ✅ VOs `MuxAssetID`, `PlaybackID` | 🟢 OK |

**Sub-uso detectado (não gap de catálogo — gap de consistência UI)**: `Money`, `FracaoIdeal`, `DealNumber`, `PublicID`, `GovernanceScore`, `QuorumThreshold`, `TrialDuration`, `Quota` — **0 ocorrências** nas UI catalogs. As telas falam em "R$ X", "fração ideal 0.0234", "DL-20240912-001" etc em prosa mas não referenciam o VO explicitamente. Não é erro de domínio — é nota editorial.

**Total órfãos E**: **0**. Um 🟢 LOW editorial (UIs poderiam referenciar VOs canônicos explicitamente nos data-models).

---

## F. Casos ambíguos (conceito vs INV específico)

Padrões onde a tela cita um **conceito** ("Timeline é imutável", "ata imutável", "append-only") em prosa, sem vincular o INV-### específico que o enforce. Não é bug — é dívida de rastreabilidade.

| Conceito citado | Telas | INV canônico que cobre | Severidade | Ação |
|---|---|---|---|---|
| "Timeline imutável / append-only" | [[../../03-subprojects/web/ui-catalog/sindico/S11]], S12, [[../../03-subprojects/web/ui-catalog/morador/_moc]], `ASM-PAINEL-SIND` | INV-024 + INV-032 + INV-119 (hardening Fase 9 DB-level) | 🟢 | Adicionar `INV-024/INV-119` explicit link nas telas. |
| "Ata (minutes) imutável pós-publish/homologated" | assembleia/* | INV-072 + INV-119 | 🟢 | Mesma recomendação. |
| "1 device policy / sessão ativa única" | onboarding/ONB-*, morador auth | INV-005 | 🟢 | — |
| "Webhook HMAC antes do parse" | compliance/C5 (auditoria) | INV-018/064/093/105 | 🟢 | — |
| "Audit trail append-only" | [[../../03-subprojects/web/ui-catalog/assembleia/ASM-REL]], ASM-CIEN, C5 | INV-090 + INV-120 | 🟢 | — |
| "Vote único por unidade" | assembleia/ASM-VOT, ASM-TELAO | INV-071 | 🟢 | — |
| "Quórum por fração ideal" | ASM-* | INV-073 | 🟢 | — |
| "Presidente síndico não vota" | ASM-CFG, ASM-VOT | INV-081 | 🟢 | — |
| "Lock 90 dias no vídeo" | MK4, BT02 | INV-056, INV-BT-02 (a formalizar) | 🟡 | Ver A (INV-BT-02). |

**Regras ABAC-only citadas sem virar INV**:

- "Parceiro Vizinhança sem acesso a LMS" → INV-067 ✅
- "Morador não cria tópico (LMS-E)" → **órfã** — classificar como ABAC policy, não INV. Ver A.
- "Admin Marketing zero acesso a dados de negócio" → INV-049 ✅
- "Vídeos empresa bloqueados pra morador por default" → INV-061 ✅

---

## G. Aggregates vazios/ausentes (citados indiretamente por telas)

Além dos órfãos da seção B (aggregates declarados no UL sem arquivo), existem aggregates **implícitos** pressupostos por telas mas que nem sequer aparecem no UL:

| Pressuposto | Evidência (tela) | Decisão necessária | Severidade |
|---|---|---|---|
| **MasterPlanActivity** (vs MasterPlan) | [[../../03-subprojects/web/ui-catalog/sindico/S7]] ("master_plan_activity aggregate"), frequência 3 em catálogo | MasterPlanActivity é Entity dentro de MasterPlan (AR) ou AR próprio? Enforcement INV-031 pressupõe **Entity interna** (update só via handler). Documentar no arquivo `MasterPlan.md` (✅ existe). | 🟡 (M3; não bloqueia M1) |
| **SyndicMandate** | [[../../03-subprojects/web/ui-catalog/sindico/S4]] ("syndic_mandate aggregate, migr. 00201"); UL lista `SyndicMandate` como **Entity** | Tela chama de "aggregate"; UL de "Entity". Alinhar. Provável: Entity dentro de Membership ou Condominium. | 🟡 |
| **ServiceRecord** (alias de ValidationItem?) | [[../../03-subprojects/web/ui-catalog/sindico/S21]], S22 ("service_record / validation_item aggregate"); `_moc.md` linha 95: "validation_items vs service-records" | Decidir se são o mesmo aggregate (ValidationItem especializado) ou 2 aggregates. Impacta M1 (S21/S22 são telas síndico M1). | 🔴 **CRITICAL M1** — mesma severidade de B.ValidationItem. |
| **AssemblyEvent** (canais `assembly.checkin`, `assembly.vote.cast`, `assembly.homologation`) | ASM-PAINEL-SIND, INV-ASM-events-append-only | São Domain Events (via DomainEvent aggregate) ou aggregate próprio de stream? Decisão arquitetural — ver G-01 assembly live. | 🟠 M2 |
| **CurriculumVideo** (entidade dentro de Curriculum) | BT02 linha 70: `curriculum_videos(curriculum_id, video_id, duration_seconds, locked_until)` | Entity dentro de Curriculum. Documentar quando criar Curriculum.md. | 🟡 M3 |
| **Dashboard** agregador | S6 ("único dashboard cross-módulo"), S27, S2, MK1, MK7 | **Não** é aggregate de domínio — é read-model/projection. Documentar em `02-architecture/read-models.md` (se existir) ou explicitar "view-model, não AR". | 🟢 editorial |

---

## H. Gaps acionáveis top 20

Ordenado por impacto M1 + esforço de fix (menor → maior):

| # | Gap | Tipo | Severidade | Fix estimado | Owner sugerido |
|---|---|---|---|---|---|
| 1 | **Criar `01-domain/aggregates/ValidationItem.md`** (= service_record) | B + G | 🔴 M1 | 2h | domain-lead |
| 2 | **Criar `01-domain/aggregates/ComplianceDeclaration.md`** | B | 🔴 M1 | 2h | domain-lead |
| 3 | **Reconciliar ValidationItem vs ServiceRecord** (mesmo AR?) | G | 🔴 M1 | 1h decisão + 1h doc | domain-lead + S21/S22 author |
| 4 | **Resolver divergência "18+ tipos evento" S16** (13 canônicos vs briefing) | C | 🟡 MEDIUM | 2h (decisão produto) | product + domain-lead |
| 5 | **Criar `01-domain/aggregates/TermAcceptance.md`** (onboarding M1 precisa) | B | 🟠 M2 (mas onboarding é M1) | 2h | domain-lead |
| 6 | **Criar `01-domain/aggregates/Amendment.md`** (INV-050 pressupõe) | B | 🟠 M2 | 2h | domain-lead |
| 7 | **Mover INV-ASM-live (P95 <200ms) para SLO**, remover de telas como "invariante" | A + F | 🟠 M2 | 1h | arch-lead |
| 8 | **Reclassificar INV-ASM-modalidade-mvp como restrição escopo M1**, não invariante | A | 🟠 M2 | 30min | schedule-lead |
| 9 | **Formalizar INV-ASM-events-append-only** — decidir se é promoção de INV-033/090/119 ou novo INV-130 | A | 🟠 M2 | 1h | domain-lead |
| 10 | **Criar `01-domain/aggregates/LGPDLog.md`** OU especializar AuditEntry com `purpose=lgpd_reveal` | B | 🟠 M2 | 2h | LGPD/DPO |
| 11 | **Criar `01-domain/aggregates/VideoVisibilityGrant.md`** (INV-062/063 pressupõem) | B | 🟠 M2 | 2h | domain-lead |
| 12 | **Criar `01-domain/aggregates/SpeechQueue.md`** (assembleia live) | B | 🟠 M2 | 2h | domain-lead |
| 13 | **Formalizar INV-BT-01..10 como INV-124..129** em invariants.md | A | 🟡 M3 | 3h | domain-lead |
| 14 | **Criar `01-domain/aggregates/Curriculum.md`** (+ CurriculumVideo entity) | B | 🟡 M3 | 3h | domain-lead |
| 15 | **Criar `01-domain/aggregates/ForumTopic.md`** (+ ForumPost entity) + eventos `ForumTopicCreated`/`ForumReplyPosted` (E-083/084) | B + D | 🟡 M3 | 4h | domain-lead |
| 16 | **Criar `01-domain/aggregates/LibraryMaterial.md`** + evento `LibraryMaterialPublished` (E-085) | B + D | 🟡 M3 | 3h | domain-lead |
| 17 | **Criar `01-domain/aggregates/LMSLive.md`** (desambiguar de LiveSession assembly) + ADR CNT-022 (Mux Live vs Cloudflare Stream) | B | 🟡 M3 | 4h + ADR | arch-lead |
| 18 | **Criar `01-domain/aggregates/NeighborhoodBenefit.md`** (cupom — INV-051..053 pressupõe) | B | 🟡 M3 | 2h | domain-lead |
| 19 | **Classificar INV-LMS-E como ABAC policy** em abac-matrix.md (não é invariante de domínio) | A | 🟡 M3 | 1h | security-lead |
| 20 | **Criar stubs para aggregates declarados mas sem arquivo** (ComplianceAssessment, ConflictOfInterestDeclaration, RiskMap, CompanyAssessment, SearchIndex, Ebook, GovernanceScoreSnapshot, PreliminarySurvey) | B | 🟢 LOW | 4h templating | domain-lead |

**Bloqueadores absolutos de M1 (Gates 🔴)**: gaps **#1, #2, #3**. Esforço total M1 = ~6h.

---

## Links

### Fontes canônicas consultadas

- [[../../01-domain/invariants]] — 116 INVs formalizados
- [[../../01-domain/aggregates/_moc]] — 40 aggregates com arquivo
- [[../../01-domain/ubiquitous-language]] — UL com 47 AR declarados (divergência de 7 sem arquivo)
- [[../../01-domain/domain-events]] — 82 E-### canônicos
- [[../../01-domain/enums/vizinhanca-categorias-subcategorias]] — 32+202 categorias
- [[../../../../90-archive/from-development-content-2026-04-23/04-Listas-Mestre/Enums-Consolidados]] — 19 categorias, 90+ valores
- [[../../../../90-archive/from-master-sindico-v2-ingested/_reconciliacao-dev/05-enums-regras-quebras]] — reconciliação v2

### Auditorias correlatas

- [[2026-04-23-analise-development-vs-vault-sistematica]]
- [[2026-04-23-CONSOLIDADO-quebras-e-ops]]
- [[2026-04-24-gap-analysis-endpoints]] — gap análogo dimensão endpoints

### Catálogos UI auditados

- [[../../03-subprojects/web/ui-catalog/_moc]] (implícito via subdirs)
- [[../../03-subprojects/mobile/ui-catalog/_moc]]

---

## Notas metodológicas

1. **Extração de INVs**: regex `INV-[A-Za-z0-9]+(-[A-Za-z0-9]+)*`, 38 ocorrências brutas → 18 únicos após dedup.
2. **Extração de eventos**: regex PascalCase com sufixos conhecidos de eventos em passado (`Created|Published|Closed|Activated|...` × 55 sufixos) — 11 únicos → 7 após filtro de falsos positivos (BLoC events UI).
3. **Extração de aggregates**: combinação de PascalCase wikilinks + frase `XXX aggregate` + cross-check contra UL.
4. **Extração de enums**: busca por padrão `N tipos de Y`, `TipoEnum`/`TipoName`, + backrefs a `Enums-Consolidados.md`.
5. **Severidade M1/M2/M3**: referência a `10-schedule/milestones.md` (M1=2026-05-08, BCs identity+billing+institutional+Compliance+Onboarding).
6. **Falsos positivos filtrados**: `AlreadySubmitted`, `RefreshRequested`, `LocationDenied`, `PermanentlyDenied`, `PermissionDenied`, widgets UI Flutter/Solid (PrimaryButton, StatusBadge, etc).

---

**Relatório gerado**: 2026-04-24 | **Agente**: opus-4-7 1M (análise DDD) | **Raiz**: `50-projects/master-sindico`

---

## Apêndice I — Escopo auditado por BC

### Telas web (192 arquivos `.md` em `03-subprojects/web/ui-catalog/`)

| BC | Telas | M1? | INVs citados (telas) | Aggregates órfãos relevantes |
|---|---|---|---|---|
| `admin` | 1 (+ 1 GAP-admin-master-sindico.md) | 🔴 parcial | 0 | Gap explícito: `GAP-admin-master-sindico.md` já documenta ausência de telas admin. |
| `assembleia` | 21 | 🟠 M2 | 7 telas (INV-ASM-*, INV-088, INV-091, INV-092) | VideoVisibilityGrant, SpeechQueue, Amendment, AssemblyEvent. |
| `banco-talentos` | 11 | 🟡 M3 | 6 telas (INV-BT-01/02/04/05/09/10, INV-070) | Curriculum, CurriculumVideo. |
| `compliance` | 11 | 🔴 **M1** | 0 explícitas | ComplianceDeclaration (**M1 🔴**), ComplianceAssessment, ConflictOfInterestDeclaration. |
| `empresa` | 16 | 🟠 M2 | 0 | — |
| `lms` | 15 | 🟡 M3 | 9 telas (INV-067/068/069/070, INV-LMS-E) | ForumTopic/ForumPost, LibraryMaterial, LMSLive. |
| `marketing` | 8 | 🟡 M3 | 0 | — (todas coberta por Video/Company). |
| `morador` | 15 | 🔴 M1 parcial | 0 | — (home-shell usa User+Condominium+TimelineEntry — tudo canônico). |
| `onboarding` | 23 | 🔴 **M1** | 2 telas (INV-091) | TermAcceptance (**M1 🟠**). |
| `sindico` | 31 | 🔴 **M1** (dashboard + timeline + PD) | 0 explícitas | ValidationItem (**M1 🔴** — S2/S21/S22/S26), MasterPlanActivity (entity de MasterPlan), SyndicMandate (entity). |
| `vizinhanca` | 29 | 🟡 M3 | 1 tela (INV-067) | NeighborhoodBenefit (cupom), PromotionType ok. |

### Telas mobile (59 arquivos em `03-subprojects/mobile/ui-catalog/`)

| BC | Telas |
|---|---|
| `assembly` | 5 |
| `morador` | 15 |
| `onboarding` | 21 |
| `sindico` | 5 |
| `vizinhanca` | 7 |

### Observações sobre o escopo audit

- O enunciado falava em "181 web + 53 mobile"; contagem real encontrada: **192 + 59 = 251**. Possível que o número do enunciado exclua `_moc.md`/`_toc.md` (web tem 11 `_moc.md`, mobile tem 6 `_moc.md` — subtrair dá 181 + 53 exatos). Auditoria considera **todos os MDs** para extração de citações (incluindo `_moc.md` que tem insights consolidados de BC).

---

## Apêndice II — Lista canônica de aggregates vs. citações

### AR declarados em `01-domain/ubiquitous-language.md`: **47**

**Identity (2)**: User, Session (ambígua entre Entity/AR — P-UL-001)
**Billing (3)**: Subscription, Plan (**não está UL explicitamente como AR** — existe `Plan.md` agregado via MOC), FeatureUsage
**Institutional (11)**: Condominium, Unit (implícito), Membership, TimelineEntry, MasterPlan, Announcement, Event, Poll, ValidationItem, ComplianceAssessment, ConflictOfInterestDeclaration, RiskMap
**Commercial (12)**: Company, ConnectMeRequest, Proposal (P-UL-002), ClosedDeal, CompanyEvaluation, CompanyAssessment, SupplierVoteSession, HarassmentReport, Amendment, Partner, NeighborhoodBenefit, Curriculum
**Content (8)**: Video, VideoModeration, VideoVisibilityGrant, SearchIndex, Course, Enrollment, Certificate, ForumTopic, Ebook
**Assembly (7)**: Assembly, Vote, Minutes, LiveSession, ProxyGrant (= Proxy), SpeechQueue, PreliminarySurvey
**Cross-domain (5)**: AuditEntry, LGPDLog, ComplianceDeclaration, TermAcceptance, GovernanceScoreSnapshot

### AR com arquivo em `01-domain/aggregates/` (excluindo `_moc.md`): **40**

AbacDecision, AgendaItem, Announcement, Assembly, AttendanceRecord, AuditEntry, Certificate, ClosedDeal, CompanyEvaluation, Company, Condominium, ConnectMeRequest, Course, DomainEvent, Enrollment, Event, FeatureUsage, HarassmentReport, LiveSession, MasterPlan, Membership, Minutes, OutboxEntry, Partner, Plan, Poll, Proposal, Proxy, RateLimitEntry, Session, SessionState, Subscription, SupplierVoteSession, TimelineEntry, Unit, User, Video, VideoModeration, Vote, WebhookEvent.

### Divergência matemática

47 AR declarados UL + 6 AR técnicos (DomainEvent, AbacDecision, AgendaItem, OutboxEntry, RateLimitEntry, SessionState, WebhookEvent — **7 novos 2026-04-23**, já existem como arquivo) = ~53 AR intenção.

Arquivos existentes: 40. **Gap de arquivos = ~13 AR** (batendo com seção B).

Aggregates declarados UL SEM arquivo em `01-domain/aggregates/`:

1. ValidationItem (institutional) 🔴 M1
2. ComplianceAssessment (institutional) 🟢
3. ConflictOfInterestDeclaration (institutional) 🟢
4. RiskMap (institutional) 🟢
5. CompanyAssessment (commercial) 🟢
6. Amendment (cross-domain) 🟠
7. NeighborhoodBenefit (commercial) 🟡
8. Curriculum (commercial) 🟡
9. VideoVisibilityGrant (content) 🟠
10. SearchIndex (content) 🟢
11. ForumTopic (content) 🟡
12. Ebook (content) 🟢
13. SpeechQueue (assembly) 🟠
14. PreliminarySurvey (assembly) 🟡
15. LGPDLog (cross-domain) 🟠
16. ComplianceDeclaration (cross-domain) 🔴 M1
17. TermAcceptance (cross-domain) 🟠
18. GovernanceScoreSnapshot (cross-domain) 🟢

**Total = 18** (superconjunto dos 13 reportados em B; divergência resolvida = incluir os 5 🟢 como LOW).

---

## Apêndice III — Catálogo completo de eventos canônicos vs citados

### 82 E-### canônicos em `01-domain/domain-events.md` (IDs E-001..E-082)

Distribuição por BC (inferida de prefixos no name):
- identity: UserCreated, UserUpdated, UserDeleted, SessionCreated, DeviceChanged, etc.
- billing: SubscriptionCreated/Activated/Expired, PlanChanged, TrialStarted/Expired, WebhookEventReceived, TermAccepted (?)
- institutional: CondominiumCreated, UnitRegistered, UnitUpdated, MembershipCreated/Ended, MandateStarted/Ended, AnnouncementPublished, ExecutionRecordSubmitted, ExecutionValidated, ComplianceDeclarationSubmitted
- commercial: CompanyCreated/Approved/Suspended, ConnectMeRequestPublished, ConnectMeAutoClosed, ProposalSubmitted/Validated/Accepted/Rejected, CompanyEvaluationSubmitted, ReputationTierChanged, HarassmentReported, SupplierVoteSessionOpened, SupplierVoteCastRegistered, SupplierVoteClosed, NeighborhoodBenefitPublished
- content: VideoModerationRequested, VideoApproved/Rejected/Published/Locked, VideoLockExpired, VideoVisibilityRevoked, CertificateGenerated (E-061), LessonCompleted, CourseCompleted, CoursePublished, EnrollmentCreated
- assembly: EditalPublished, AgendaPublished, AssemblyStarted, CheckInRegistered, VotingOpened/Closed, SpeechTurnStarted/Ended, ProxyRevoked, MinutesGenerated, MinutesPublished, HomologationOpened, HomologationApproved, AssemblyClosed
- governance: GovernanceScoreUpdated, AcknowledgmentRegistered, TurnEnded, PollOpened, PollClosed, AntiFraudTriggered

### Eventos citados em tela (ID ou nome)

E-059, E-060, E-061 (citados explicitamente) — presentes.
PascalCase sem E-###: CertificateGenerated, CourseCompleted, LessonCompleted — presentes.

### Órfãos D (repetidos em D)

- `ForumTopicCreated`, `ForumReplyPosted` (LMS11, LMS13) — G-04
- `LibraryMaterialPublished` (LMS14) — G-04
- `TrainingCompleted` (LMS7) — G-04 ou redundante com `LessonCompleted`

---

## Apêndice IV — Detalhamento da divergência "18+ tipos de evento"

### Evidência

- **`03-subprojects/web/ui-catalog/sindico/S16.md`** linhas 2, 16, 19, 82 — "Criar Evento (18+ tipos)"
- **`03-subprojects/web/ui-catalog/sindico/_moc.md`** linha 34 — "Criar Evento (18+ tipos)"
- S16 linha 82 explicita: "Cliente lista 13 tipos no backend `event.go` mas o .ini lista 18+ tipos — divergência registrada; consolidar enum."

### Canônico

`90-archive/from-development-content-2026-04-23/04-Listas-Mestre/Enums-Consolidados.md:178` — **13 tipos**:

```
assembleia_ordinaria · assembleia_extraordinaria · manutencao_programada
inspecao_tecnica · obra_programada · interrupcao_programada_servicos
treinamento_equipe · campanha_institucional · evento_comunitario
reuniao_administrativa · atualizacao_normativa · simulado_emergencia · outros
```

### Hipótese dos "5 extras" (18 - 13)

Possivelmente os 5 estão confundindo **Announcement types** (`Enums-Consolidados §7 Comunicados — Tipos (8 — canônico requirements.md)`) com Event types. Conforme S24 linha 82: "Lista de tipos de comunicado vs eventos sobrepostos — consolidar enum". O briefing .ini trataria comunicados emergenciais + institucionais + pós-deal como subtipos de Event — mas o modelo DDD separa como 2 aggregates distintos (`Announcement` vs `Event`).

### Resolução sugerida

1. Manter `EventType` com 13 valores canônicos.
2. Documentar em UL que `Announcement` cobre comunicação não-calendarizada (8 tipos + 5 pós-deal) e `Event` cobre calendarizável (13).
3. Atualizar S16 e S24 para referenciar 13 tipos + nota "demais formas de comunicação via Announcement".

---

## Apêndice V — Referência cruzada gaps × INVs × aggregates × eventos

| Gap | INV afetado | Aggregate afetado | Evento afetado |
|---|---|---|---|
| A-DDD-001: ValidationItem sem arquivo | INV-031 (?), INV-086 | **ValidationItem** ausente | — |
| A-DDD-002: ComplianceDeclaration sem arquivo | INV-034, INV-035 | **ComplianceDeclaration** ausente | `ComplianceDeclarationSubmitted` (canônico) |
| A-DDD-003: TermAcceptance sem arquivo | INV-091 (LGPD) | **TermAcceptance** ausente | `TermAccepted` (canônico) |
| A-DDD-004: Amendment sem arquivo | **INV-050** | **Amendment** ausente | — (evento `AmendmentCreated`?) |
| A-DDD-005: VideoVisibilityGrant sem arquivo | **INV-062, INV-063** | **VideoVisibilityGrant** ausente | `VideoVisibilityRevoked` (canônico) + falta `VideoVisibilityGranted` |
| A-DDD-006: LGPDLog sem arquivo | INV-048, INV-091, INV-109 | **LGPDLog** ausente | — |
| A-DDD-007: SpeechQueue sem arquivo | (novo INV) | **SpeechQueue** ausente | `SpeechTurnStarted`/`Ended` (canônico) |
| A-DDD-008: Curriculum sem arquivo | INV-BT-01..10 | **Curriculum** ausente | `CurriculumPublished`/`CurriculumUpdated` não formalizados |
| A-DDD-009: ForumTopic sem arquivo | INV-LMS-E (a reclassificar) | **ForumTopic** ausente | **ForumTopicCreated**, **ForumReplyPosted** ausentes (D) |
| A-DDD-010: LibraryMaterial sem arquivo | (novo INV lock 90d?) | **LibraryMaterial** ausente | **LibraryMaterialPublished** ausente (D) |
| A-DDD-011: LMSLive sem arquivo | (novo INV) | **LMSLive** ausente | `LMSLiveScheduled`/`Started`/`Ended` não formalizados |
| A-DDD-012: NeighborhoodBenefit sem arquivo | INV-051, INV-052, INV-053 | **NeighborhoodBenefit** ausente | `NeighborhoodBenefitPublished` (canônico) |
| A-DDD-013: INV-BT-01..10 não promovidos | **INV-BT-01..10** (6 citados) | Curriculum (bloqueado por DDD-008) | — |
| A-DDD-014: INV-ASM-events sem formalização | **INV-ASM-events-append-only** | AssemblyEvent/DomainEvent | — |
| A-DDD-015: INV-LMS-E é ABAC, não DDD | **INV-LMS-E** | ForumTopic (bloqueado por DDD-009) | — |
| A-DDD-016: Divergência 18+ vs 13 tipos de evento | — | Event (canônico) | EventCreated (implícito) |
| A-DDD-017: Ambiguidade Announcement vs Event | — | Announcement ↔ Event | — |

---

## Apêndice VI — Scorecard final

| Dimensão | Score | Comentário |
|---|---|---|
| **Coerência INV** | 7/18 = 39% | 7 dos 18 INVs citados estão formalizados; 6 precisam ser promovidos de 00-product; 3 são novos (1 real invariante, 2 reclassificações) |
| **Coerência aggregates** | 40/47 ou 40/53 = **75-85%** | 13 ARs declarados sem arquivo (ou 18 incluindo LOW) |
| **Coerência enums** | 7/7 = **100%** (citados com canônico) | 1 divergência numérica (18+ vs 13) a resolver |
| **Coerência eventos** | 7/11 reais = **64%** | 4 eventos LMS órfãos (todos já marcados G-04 pelas telas) |
| **Coerência VOs** | 7/7 citados = **100%** | Sub-uso editorial mas zero órfão |
| **Score global DDD (ponderado)** | ~77% | **Produção-pronto com lacunas pontuais M1 (ValidationItem + ComplianceDeclaration) e dívida M2/M3 tratável** |

**Severidade agregada M1**: 2 gaps 🔴 bloqueadores (ValidationItem.md, ComplianceDeclaration.md); 0 🔴 em enums/eventos. Esforço estimado pra destravar M1 = **~6h** (2h + 2h + 1h decisão + 1h mapeamento ServiceRecord vs ValidationItem).

**Severidade agregada M2**: 6 gaps 🟠 (TermAcceptance, Amendment, VideoVisibilityGrant, SpeechQueue, LGPDLog, INV-ASM-events) — **~12h** de domain templating + 1 ADR (promoção INV-ASM-events).

**Severidade agregada M3**: Resto — absorvível na sprint de ativação BC correspondente (LMS, BT, Vizinhança).

---

## Apêndice VII — Contratos de fix (templates por gap M1/M2)

### VII.1. `01-domain/aggregates/ValidationItem.md` (Gap A-DDD-001, 🔴 M1)

**Template mínimo obrigatório**:

```markdown
---
title: ValidationItem (Aggregate)
type: aggregate
bc: institutional
tags: [domain, ddd, aggregate, institutional, master-sindico, v2]
created: 2026-04-24
updated: 2026-04-24
---

# ValidationItem

Checklist legal consumido por síndico durante mandato (fundo reserva, laudos, certificações, inspeções).

## Identidade / Raiz
Aggregate Root: `ValidationItem { id, condominium_id, type, title, status, assigned_to, due_date, validated_at, validated_by, evidence_uri, ... }`

## VOs
- `ValidationItemType` (Enum: `reserva_fundo | laudo_anual | ppci | aterramento | para_raios | cftv_lgpd | elevadores | alvara | ...`)
- `ValidationItemStatus` (Enum: `pending | in_review | approved | rejected | expired`)
- `DueDate` (VO com validação antecedência)

## Factory / Repo
- `New(condominium_id, type, due_date) -> (*ValidationItem, error)` — valida type ∈ enum + due_date > now.
- `Reconstruct(...)` — sem validação.
- Repo interface `IValidationItemRepo` em `domain/institutional/validation_item/repo.go`.

## Eventos emitidos
- `ValidationItemCreated` (E-TBD)
- `ValidationItemValidated` (E-TBD)
- `ValidationItemRejected` (E-TBD)
- `ValidationItemExpired` (E-TBD, via job)

## Invariantes
- Vínculo com `Condominium` (INV-089 tenant isolation).
- Append-only trilha de estados via AuditEntry (INV-033, INV-090).
- Related: INV-086 (ABAC `service_record.validate`).

## Notas de reconciliação
- `ValidationItem` ≡ `ServiceRecord` (alias do código)? **Decisão P-A-DDD-003**: S21 usa "service_record", S22 usa ambos. Adotar **ValidationItem** como canônico DDD e documentar alias `service_record` como nome legado backend (tabela `validation_items`, endpoint `/api/v1/condominiums/{id}/validation-items`).

## Links
- [[../invariants]] — INV-086 (ABAC), INV-089 (tenant)
- [[../ubiquitous-language]]
- [[../../03-subprojects/web/ui-catalog/sindico/S21]] — lista pendentes
- [[../../03-subprojects/web/ui-catalog/sindico/S22]] — validar + avaliar
- [[../../03-subprojects/web/ui-catalog/sindico/S26]] — fluxo central
```

### VII.2. `01-domain/aggregates/ComplianceDeclaration.md` (Gap A-DDD-002, 🔴 M1)

**Pressupostos canônicos pré-existentes**:
- INV-034 (compliance_declarations anual, UNIQUE parcial por `(condominium_id, year)`)
- INV-035 (obrigatória no onboarding síndico)
- Req 34 referenciado no UL
- Evento `ComplianceDeclarationSubmitted` já existe em domain-events.md

**Template mínimo** (mesma estrutura V.1 ajustando):
- Identidade: `ComplianceDeclaration { id, condominium_id, year, items[], signed_by, signed_at, status, version }`
- Estados: `draft | signed | amended | superseded`
- Eventos: `ComplianceDeclarationSubmitted` (canônico), `ComplianceDeclarationAmended` (novo)
- Enforcement de imutabilidade pós-`signed` via Amendment (INV-050)
- INV-034, INV-035 são os invariantes core

### VII.3. `01-domain/aggregates/TermAcceptance.md` (Gap A-DDD-003, 🟠 M2 mas onboarding M1 usa)

**Pressupostos**:
- UL declara `TermAcceptance { user_id, term_version, accepted_at, ip }` (Req 35)
- Onboarding M1 registra aceite LGPD + termos de uso — sem agregado, implementação ad-hoc

**Template mínimo**:
- Identidade: `TermAcceptance { id, user_id, term_type, term_version, accepted_at, ip_address, user_agent }`
- `TermType` enum: `terms_of_use | privacy_policy | lgpd_consent | pd_consent | cookies`
- Append-only — nunca UPDATE/DELETE (INV-033 estende-se)
- Evento: `TermAccepted` (canônico? — verificar E-### series)

### VII.4. Formalização `INV-130: AssemblyEvent append-only` (Gap A-DDD-014, 🟠 M2)

**Proposta de entry em `01-domain/invariants.md` seção hardening Fase 10**:

| ID | Invariante | Owner | Enforcement | Teste | Finding |
|---|---|---|---|---|---|
| **INV-130** | `assembly_events` (check-in, voto, homologação, fila de fala) têm `REVOKE UPDATE, DELETE` para `app_user` + trigger `forbid_mutation()` análogo a INV-119/INV-120. INSERT-only. Correção via nova entry com `corrects_event_id` e `correction_reason` (padrão INV-032 timeline) | assembly | Migration REVOKE + trigger + stored proc admin `assembly.admin_event_correction(...)` 4-eyes | Integration (UPDATE como app_user → ERROR P0001) | INV-ASM-events-append-only citado em ASM-PAINEL-SIND, _moc |

### VII.5. Decisão necessária — ValidationItem ↔ ServiceRecord (Gap A-DDD-001 + A-DDD-018)

**Contexto**:
- `01-domain/ubiquitous-language.md` declara `ValidationItem` como Aggregate Root oficial.
- `03-subprojects/web/ui-catalog/sindico/S21.md` linha 28 usa ação ABAC `service_record.list_pending` + linha 47 widget `ServiceRecordCard`.
- `03-subprojects/web/ui-catalog/sindico/S22.md` linha 28 usa `service_record.validate` + linha 81 "service_record + company_evaluation".
- `03-subprojects/web/ui-catalog/sindico/_moc.md` linha 95 explicita "validation_items vs service-records" como divergência de nomenclatura.

**Hipóteses**:
1. **Alias puro**: `service_record` é nome legado da API/backend; `ValidationItem` é nome canônico DDD. Ação: documentar alias no aggregate + padronizar ações ABAC para `validation_item.*`.
2. **Dois aggregates**: `ServiceRecord` = registro de execução de serviço (feito por empresa); `ValidationItem` = item de checklist legal (síndico valida). Ação: criar **AMBOS** os arquivos de aggregate.

**Recomendação**: hipótese 2. S21/S22 combinam as duas perspectivas — síndico aprova service_record (saída de execução empresa) **E** valida validation_item (checklist legal próprio). São dimensões distintas: ServiceRecord é AR em `commercial` (registro de execução após ClosedDeal, relacionado a Req 22); ValidationItem é AR em `institutional` (checklist síndico). Cruzam em workflows (validar execução VS validar item de mandato) mas são agregados distintos.

**Decisão necessária**: domain-lead + product + author de S21/S22/S26 sentam 30min e decidem. **Bloqueador M1 se não decidido até sprint que inicia 2026-04-28**.

---

## Apêndice VIII — Comandos grep reproduzíveis

Para re-auditoria ou verificação dos achados:

```bash
ROOT="/home/vinni/Documentos/Obsidian Vault/automation-agents/vault/50-projects/master-sindico"

# 1. INVs citados (regex full pattern)
find "$ROOT/03-subprojects/web/ui-catalog/" "$ROOT/03-subprojects/mobile/ui-catalog/" \
  -type f -name "*.md" -exec grep -oHE "INV-[A-Za-z0-9]+(-[A-Za-z0-9]+)*" {} \; \
  | cut -d: -f2- | sort -u

# 2. INVs canônicos formalizados
grep -oE "INV-[A-Z0-9-]+" "$ROOT/01-domain/invariants.md" | sort -u

# 3. Aggregate Roots declarados no UL
grep "Aggregate Root" "$ROOT/01-domain/ubiquitous-language.md" \
  | grep -oE "\`[A-Z][a-zA-Z]+\`" | sort -u

# 4. Aggregates com arquivo
ls "$ROOT/01-domain/aggregates/" | grep -v "_moc" | sed 's/\.md$//' | sort

# 5. Diff: declarados UL vs arquivos existentes
comm -23 <(grep "Aggregate Root" "$ROOT/01-domain/ubiquitous-language.md" \
  | grep -oE "\`[A-Z][a-zA-Z]+\`" | tr -d '`' | sort -u) \
         <(ls "$ROOT/01-domain/aggregates/" | grep -v _moc | sed 's/\.md$//' | sort -u)

# 6. Eventos PascalCase canônicos
grep -oE "\b[A-Z][a-zA-Z]+(Created|Published|Closed|Activated|Completed|Started|Ended|Voted|Homologated|Submitted|Approved|Rejected|Revoked|Expired|Opened|Deleted|Generated|Received|Updated|Changed|Accepted|Declined)\b" \
  "$ROOT/01-domain/domain-events.md" | sort -u

# 7. Gaps explícitos em telas (self-reported)
grep -rnE "G-0[0-9]|gap G-|não formalizad|ausente|não existe" \
  "$ROOT/03-subprojects/web/ui-catalog/" "$ROOT/03-subprojects/mobile/ui-catalog/"
```

---

## Apêndice IX — Estado das dependências canônicas

| Artefato canônico | Path | Linhas | Atualizado |
|---|---|---|---|
| `invariants.md` | `01-domain/invariants.md` | 297 | 2026-04-23 |
| `aggregates/_moc.md` | `01-domain/aggregates/_moc.md` | 125 | 2026-04-23 |
| `ubiquitous-language.md` | `01-domain/ubiquitous-language.md` | 313 | 2026-04-23 |
| `domain-events.md` | `01-domain/domain-events.md` | 696 | 2026-04-23 |
| `Enums-Consolidados.md` | `90-archive/from-development-content-2026-04-23/04-Listas-Mestre/` | 686 | 2026-04-23 |
| `05-enums-regras-quebras.md` | `90-archive/from-master-sindico-v2-ingested/_reconciliacao-dev/` | 399 | 2026-04-23 |
| UI catalogs web | `03-subprojects/web/ui-catalog/` | 192 arquivos | 2026-04-22..24 |
| UI catalogs mobile | `03-subprojects/mobile/ui-catalog/` | 59 arquivos | 2026-04-22..24 |

---

## Apêndice X — Risco e recomendações finais para o domain-lead

### Risco de NÃO tratar os gaps M1 antes de 2026-05-08

1. **ValidationItem ausente**: S21/S22/S26 são telas síndico M1 no fluxo de validação de execução. Sem o AR formalizado, o backend implementa "como der" — race conditions em validação concorrente, ausência de INV de unicidade por (condominium, type, year), bypass de ABAC por confusão service_record vs validation_item. **Impacto**: bug em telas core + possível inconsistência jurídica no registro imutável de aprovação.

2. **ComplianceDeclaration ausente**: Onboarding síndico M1 exige declaração inicial (INV-035). Sem AR, o código de `FinishOnboardingUseCase` não tem onde plugar — possível "hack" temporário que precisa ser refeito M2 ou lançamento sem o requisito 34/35 satisfeito.

3. **ServiceRecord vs ValidationItem**: Decisão não-tomada = dois times implementam interpretações diferentes — 2 semanas antes M1 é tarde para reconciliar.

### Recomendação prioritária

Executar os **3 gaps 🔴 CRITICAL** (itens #1, #2, #3 do top 20) em **sprint de domain hardening 2026-04-25 a 2026-04-27** (3 dias), com owner domain-lead + revisor arch-lead + author S21/S22/S26 (sindico BC). Custo ≤ 1 dia-pessoa útil. Sem isso, M1 começa com dívida técnica crítica de domínio.

### Próximos passos automáticos sugeridos

1. Abrir PR em `01-domain/aggregates/` criando `ValidationItem.md` + `ComplianceDeclaration.md` com o template VII.
2. Abrir decisão em `STATE.md` formalizando ValidationItem ≠ ServiceRecord (hipótese 2) com trigger sprint de domain reconciliation.
3. Criar task em `05-tasks/backlog.md` para promoção INV-BT-01..10 (M3) e formalização INV-130 (M2) + reclassificação INV-LMS-E + INV-ASM-live + INV-ASM-modalidade-mvp.
4. Atualizar `invariants.md` com seção "Fase 10 — reconciliação UI catalog" registrando INVs promovidos.
5. Atualizar `03-subprojects/web/ui-catalog/sindico/S16.md` e `S24.md` com resolução da divergência "18+ tipos evento" após decisão de produto.
