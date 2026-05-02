---
title: Backlog Global — Master Síndico v2
type: spec
tags: [tasks, backlog, moscow, master-sindico, v2]
source: 90-ingested/_consolidado-specs-legado.md + _consolidado-providers-fluxos.md + legado `05-tasks/**` + AUDIT A-020..A-034 + STATE D-006..D-015, DT-002..DT-003
created: 2026-04-23
updated: 2026-04-23
---

# Backlog Global — Master Síndico v2

Backlog **consolidado** de tasks (T-###) organizado por **sub-produto × BC** com priorização **MoSCoW** (MUST / SHOULD / COULD / WON'T-M1). Cobre:

- Herança do legado vivo (Sprint 10 em curso, AP-### pendentes, A-### abertos).
- Lacunas descobertas na remontagem (DT-002 ADRs formais, DT-003 destilar TS, novas ADRs pós-research).
- Novos findings A-### da Fase 5 (placeholders — preencher conforme dual-check avança).

**Princípios de priorização**:
- Priorizar M1 (2026-05-08) acima de tudo. Nada que não seja **MUST-M1** entra na Sprint 10.
- MUST-M2/M3 empilha em 11-16 respectivamente.
- SHOULD = entrega se prazo permitir dentro do marco; senão vai pra próximo.
- COULD = backlog pós-marco; não bloqueia.
- WON'T-M1 = explicitamente fora de escopo do MVP contratual (protege foco).

**IDs estáveis**: `T-###` (global). Task herdada do legado preserva referência (`Kiro 9.19` → `T-042 (ex-Kiro 9.19)`).

---

## Sumário

- [Priorização por marco (MoSCoW)](#priorização-por-marco-moscow)
- [Por sub-produto](#por-sub-produto)
- [Por bounded context (roll-up)](#por-bounded-context-roll-up)
- [Transversais (cross-cutting)](#transversais-cross-cutting)
- [Dívidas técnicas / remontagem](#dívidas-técnicas--remontagem)
- [AUDIT findings → tasks](#audit-findings--tasks)
- [WON'T-M1 (fora de escopo protegido)](#wont-m1-fora-de-escopo-protegido)

---

## Priorização por marco (MoSCoW)

| Marco | MUST (count) | SHOULD | COULD | WON'T |
|---|---|---|---|---|
| **M1** (2026-05-08) | 18 | 6 | — | 14 |
| **M2** (2026-06-20) | 22 | 9 | 4 | — |
| **M3** (2026-07-20) | 13 | 7 | 6 | — |
| **Pós-M3** | — | — | 12 | — |

---

## Por sub-produto

### Sub-produto 1 — Governança Institucional (BC `institutional` + `identity`)

#### MUST-M1

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-001 | Finalizar frontend web onboarding síndico + dashboard condomínio | 10 | IDN-001..004 ✅, INT-001 ✅ | Persona síndico logged-in, escolhe plano, vê timeline |
| T-002 | Finalizar mobile home síndico/morador (auth + home shell) | 10 | IDN-001 ✅, INT-007 ✅ | App abre, auth OIDC fluxo mobile Bearer, lê timeline |
| T-003 | Fix AUDIT A-020 — N+1 em `master_plan_timeline_handler` | 10 | A-020 aberto | Query otimizada, PBT cobrindo, coverage domain 95% mantida |
| T-004 | Fix AUDIT A-023 — 1-device change sem audit log comparativo | 10 | A-023 aberto | `DeviceChanged` event contém `old_fp` + `new_fp`; `audit_trail` append-only entry com diff |
| T-005 | Documentar anti-pattern A-021 `SELECT *` em `07-quality/PATTERNS.md` | 10 | A-021 aberto | Regra proibitiva + grep CI bloqueador |
| T-006 | LGPD base — consent registry + delete endpoint + DPO notificação | 10 | XDM-009 ✅ (audit trail), LGPD spec ✅ | `GET /me/export`, `DELETE /me` SLA 15d, audit log |

#### SHOULD-M1

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-007 | Seletor contexto síndico 2+ condos persistido em sessão | 10 | INT-005 | UI seleciona condo; sessão mantém |
| T-008 | Governance Score job noturno em prod | 11 | INT-024 ✅ backend | Job roda 03:00 UTC; snapshot diário em `governance_score_snapshots` |

#### MUST-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-020 | Poll + Announcement UI completa | 11 | INT-014, INT-015 | Síndico cria enquete, morador vota, resultado publicado no timeline |
| T-021 | Ciência morador com bloqueio app (antes assembleia) | 11 | INT-007 ✅, ASM-005 | Edital publicado → morador perde acesso app até `Acknowledge` |

#### MUST-M3

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-050 | Event (condomínio) 13 tipos UI | 14 | INT-013 ✅ | Síndico cria evento, morador confirma ciência |
| T-051 | Compliance anual UI + bloqueio se declaração pendente | 15 | INT-023 ✅ | Síndico não exporta dossiê se declaração vencida |

---

### Sub-produto 2 — Connect Me (BC `commercial`)

#### MUST-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-100 | Connect Me UI síndico→empresa (RFP + lista propostas + escolha) | 11 | COM-002 ✅ backend | Síndico publica RFP; empresa recebe notificação; volta proposta; síndico escolhe |
| T-101 | Connect Me UI morador→síndico (form estruturado) | 11 | COM-006 ✅ backend | Morador envia; síndico recebe; audit log |
| T-102 | Connect Me UI empresa→empresa (Plus/Pro) | 12 | COM-007 backend | Empresa Plus/Pro → outra Plus/Pro; permissão por tier |
| T-103 | Connect Me UI síndico→parceiro Vizinhança | 12 | COM-008 backend | Síndico convida parceiro; vira acordo leve |
| T-104 | Fix AUDIT A-029 — Company Evaluation TOCTOU | 11 | A-029 | PBT + `SELECT ... FOR UPDATE`; race test |
| T-105 | Fix AUDIT A-030 — Supplier Vote race (voto pós-close) | 11 | A-030 | State machine SupplierVoteSession aceita voto só em `open` |
| T-106 | Quotas Connect Me com reset anual job | 11 | BIL-007, BIL-009 ✅ | Job 1º-jan reset `feature_usages.connect_me`; PBT |

#### SHOULD-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-110 | Denúncia assédio UI + admin action | 12 | COM-005 ✅ | Síndico denuncia; admin suspende empresa |
| T-111 | Auto-close 48h Connect Me (cron) | 12 | COM-004 ✅ | Cron move RFP sem interesse em 48h → `auto-closed` |

---

### Sub-produto 3 — Reputação Bronze→Diamante (BC `commercial`)

#### MUST-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-150 | Dashboard reputação pública da empresa | 11 | COM-015 ✅, tier calc ✅ | Empresa vê tier + métricas geradoras; síndico vê na busca |
| T-151 | Recálculo deterministico pós-evento (contract closed, eval submitted, video approved) | 11 | STATE D-010 ⏳ | Função pura testada; fluxo síncrono via UoW |
| T-152 | Degradação permitida (test matrix) | 11 | D-010 | PBT: inputs pioram → tier cai |

#### SHOULD-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-155 | Fechar D-010 — parametrização vs congelada | 11 | research-loop + dual-check | ADR criada (ADR-00XX); thresholds congelados em MVP |

---

### Sub-produto 4 — Conteúdo & Vídeos (BC `content`)

#### MUST-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-200 | Player HLS + upload presigned UI | 11 | CTT-001 ✅, Mux SDK real Sprint 9 ✅ | Empresa sobe vídeo 2min; vê playback HLS; sem proxy de bytes |
| T-201 | Moderação manual UI (admin) | 12 | CTT-001 ✅ | Admin aprova/rejeita em 48h |
| T-202 | Lock 90d enforcement + removal guard | 11 | CTT-003 ✅ backend | Tentativa remove < 90d → 403 `LOCKED_UNTIL` |
| T-203 | Privacidade métricas cross-tenant (ABAC + test) | 12 | CTT-005 ✅ | Cross-tenant integration tests (10+) |
| T-204 | Visibilidade temporária durante votação fornecedor | 12 | CTT-004 ✅, ASM-010 (M3) | `VideoVisibilityGrant` criado/revogado |

#### SHOULD-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-210 | Busca full-text OpenSearch prod | 12 | CTT-008 | Queries "apresentação portaria" retornam ≤ 200ms |

#### MUST-M3

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-250 | LMS thin — 5 áreas + 3 trilhas + certificados | 14 | CTT-011 | Síndico faz trilha iniciante; 100% → certificado |
| T-251 | Ebooks biblioteca base (PDF S3) | 15 | CTT-010 | User Plus+ baixa ebook; audit log |

---

### Sub-produto 5 — Assembleias Live (BC `assembly`)

#### MUST-M3

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-300 | Assembleia configuração UI + edital PDF | 14 | ASM-001, ASM-004 ✅ backend | Síndico agenda, gera/upload edital |
| T-301 | Check-in UI (app + QR + terminal) | 14 | ASM-009 ✅ backend | Morador check-in via QR; terminal físico funciona |
| T-302 | Votação real-time WebSocket < 200ms | 14 | ASM-010 ✅ backend | 100 conexões simultâneas; voto reflete em <200ms |
| T-303 | Proxy (procuração) UI | 15 | ASM-006 | Morador outorga; proxy vota |
| T-304 | Fix AUDIT A-033 — Livekit Join/List retry + circuit breaker | 14 | A-033 | Retry exponential 3x; DLQ; PBT |
| T-305 | Fix AUDIT A-034 — Livekit EndRoom retry + NotFound tolerant | 14 | A-034 | EndRoom retorna success mesmo em `room_not_found` |
| T-306 | Ata automática + homologação | 15 | ASM-014 ✅ | Votação fim → ata gerada; homologação publica; imutável |
| T-307 | Painel presidente UI + telão 2 áreas | 14 | ASM-011, ASM-012 | Síndico vê quórum real-time, fila fala, controles |

#### SHOULD-M3

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-310 | Simulador quórum pré-assembleia | 15 | ASM-008 | Síndico simula com # presentes; sistema calcula |
| T-311 | Enquetes preliminares consultivas | 15 | ASM-007 | Síndico pré-assembleia; resultado informativo |
| T-312 | 6 relatórios + score transparência | 16 | ASM-015 | CSV + PDF; score 0-100 |
| T-313 | Fila fala cronometrada + alertas 30s | 16 | ASM-013 | Timer visível presidente + morador |

---

### Sub-produto 6 — LMS + Banco de Talentos (BCs `content` + `commercial`)

#### MUST-M3

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-400 | Banco Talentos — vídeo-currículo 90s + busca empresa | 14 | COM-018, COM-019 | Morador sobe 90s; empresa busca por skill+CEP |
| T-401 | LMS thin — 5 áreas (Gestão/Direito/Finanças/Comunicação/Técnica) + 3 trilhas | 14 | CTT-011 | Ver T-250 |

#### COULD (pós-M3)

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-450 | LMS pleno + parceria ABRACS | M4+ | — | — |
| T-451 | Certificados com assinatura BR (MP 2.200-2) | M4+ | — | — |

---

### Sub-produto 7 — Vizinhança (BC `commercial`)

#### MUST-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-500 | Vizinhança mapa + cupons (lock 4h) | 12 | COM-020..023 | Parceiro publica cupom; lock 4h; morador redime |
| T-501 | Seal "Síndico-aprovado" | 12 | COM-020 | Síndico N2+ emite seal; visível no perfil parceiro |

#### SHOULD-M2

| ID | Título | Sprint alvo | Dependências | AC |
|---|---|---|---|---|
| T-510 | Geo search PostGIS ou lat/long + raio | 12 | — | Morador de condomínio em raio 2km vê parceiros |

---

## Por bounded context (roll-up)

Ver [[by-module/_moc]] para detalhe.

- [[by-module/identity]] — 8 tasks (T-001..T-005, T-600..T-602)
- [[by-module/billing]] — 6 tasks (T-006, T-106, T-610..T-613)
- [[by-module/institutional]] — 11 tasks (T-001..T-008, T-020..T-021, T-050..T-051)
- [[by-module/commercial]] — 18 tasks (T-100..T-152, T-500..T-510)
- [[by-module/content]] — 9 tasks (T-200..T-251)
- [[by-module/assembly]] — 10 tasks (T-300..T-313)
- [[by-module/cross-domain]] — 9 tasks (T-700..T-708)

---

## Transversais (cross-cutting)

### Observabilidade / operação

| ID | Título | Marco | Dependências | AC |
|---|---|---|---|---|
| T-700 | OTel completo em 3 sub-projetos | M1 | Sprint 7 ✅ backend | Traces cobrem 80% endpoints; dashboards Grafana |
| T-701 | Sentry prod em backend + web + mobile | M1 | — | Errors capturados + alertas PagerDuty |
| T-702 | Runbooks críticos escritos (incident, rollback, webhook-reprocess) | M1 | — | Ver [[../12-docs/runbooks/_moc]] |
| T-703 | Load test k6 — 1000 concurrent | M2 | — | API p95 < 500ms; rotas críticas |
| T-704 | Chaos test (Mux/Livekit down) | M2 | — | Saga compensa; user vê erro amigável |
| T-705 | Postmortem template + primeiro dry-run | M2 | — | Ver [[../11-audit/postmortems/template]] |

### Segurança / LGPD

| ID | Título | Marco | Dependências | AC |
|---|---|---|---|---|
| T-706 | Security review BeyondCorp — pentest pass | M1 | Sprint 8 ✅ | 0 achados high/critical |
| T-707 | LGPD auditoria externa (consent + retention + export + delete) | M1 | T-006 | Relatório DPO assinado |
| T-708 | CORS wildcard bloqueado em staging/prod (`Config.Validate`) | M1 | A-005 ✅ legado | Startup falha se `*` em prod |

---

## Dívidas técnicas / remontagem

Ver [[../STATE#dívidas-técnicas-dt]] para contexto completo.

| ID | Título | Severidade | Marco |
|---|---|---|---|
| T-800 | DT-001 — Agregar gaps client-vault × vault ativo em `01-domain/_gaps-resolvidos.md` | Baixa | M1 |
| T-801 | DT-002 — Formalizar 20 ADRs planejadas (ADR-0001..ADR-0020) | Média | M1 |
| T-802 | DT-003 — Destilar `structured_dump.md` (grep dirigido por regra negócio/TODO) | Baixa | M2 |
| T-803 | DT-004 — Timeout em UoW.Run (herdado legado) | Média | M1 |
| T-804 | DT-010 — `IAuditLogger` cross-módulo | Média | M1 |
| T-805 | Novas ADRs pós-research (D-006..D-015) | Média | M1-M2 |

---

## AUDIT findings → tasks

Findings abertos em [[../11-audit/AUDIT]] que viram task:

| A-### | Título | Task derivada | Marco |
|---|---|---|---|
| A-020 | N+1 UPDATE `master_plan_timeline_handler` | T-003 | M1 |
| A-021 | 12 `SELECT *` em commercial | T-005 (doc) + T-810 (fix code) | M1 |
| A-023 | 1-device change sem audit log comparativo | T-004 | M1 |
| A-024 | `rawBodyReader` interface privada | T-811 (elevar ao contracts/core) | M2 |
| A-029 | Company Evaluation TOCTOU | T-104 | M2 |
| A-030 | Supplier Vote status race | T-105 | M2 |
| A-031 | Event Response TOCTOU (falso positivo) | T-812 (fechar via dual-check) | M2 |
| A-032 | Goroutines lifecycle explícito | T-813 (pattern "context-aware shutdown") | M2 |
| A-033 | Livekit join/List retry | T-304 | M3 |
| A-034 | Livekit EndRoom retry + NotFound | T-305 | M3 |

Placeholders para findings **novos** da Fase 5 (dual-check/QA/pentester pass):
- `T-900` — bucket findings Modelagem/DDD (cross-BC coupling, aggregate boundaries, ubiquitous ambiguity).
- `T-910` — bucket findings Segurança (ABAC matrix coverage, tenant isolation 10 queries, webhook HMAC, rate limit strategy, LGPD data-flow).
- `T-920` — bucket findings Multi-produto (feature flag consistency, quotas matrix, visibility cross-tenant).
- `T-930` — bucket findings Operação (SLI/SLO por sub-produto, backup/restore, DR/BCP).

---

## WON'T-M1 (fora de escopo protegido)

Para proteger foco do MVP contratual (ver [[../00-product/portfolio-de-produtos#anti-catálogo]]):

- Assembleia online/híbrida pleno → M4+
- Empresa-síndica → M5+
- Agência Marketing tenant próprio → M5+
- Moderação ML (OpenAI/Rekognition) → M4+
- Multi-região → M6+
- Marketplace financeiro (split Stripe Connect) → M5+ compliance BR
- Integração ERP (Superlógica, Igora) → M4+ parceria
- Live streaming assembleia parcial → M3 (thin); pleno M4+
- LMS pleno (+ ABRACS) → M4+
- AI copiloto síndico → M6+
- Validação Receita Federal CNPJ/CPF → M4+
- Compliance NR-1 → M4+
- Breach notification 72h automática → M4+

---

## Como criar task nova

1. Identificar **sub-produto** + **BC**.
2. Mapear marco (M1/M2/M3/pós-M3) e **MoSCoW**.
3. Se atravessa BCs: marcar `transversal` em [[by-module/cross-domain]].
4. Dependências: identificar requirement GR-###/IDN-###/etc. + tasks anteriores.
5. AC (Acceptance Criteria) com teste previsto.
6. DoR/DoD: ver [[../DoR-DoD]].
7. ID sequencial estável (T-###). Não reaproveitar.

---

## Links

- [[_moc]]
- [[by-sprint/_moc]]
- [[by-module/_moc]]
- [[../STATE]]
- [[../11-audit/AUDIT]]
- [[../ROADMAP]]
- [[../DoR-DoD]]
- [[../00-product/portfolio-de-produtos]]
- [[../01-domain/bounded-contexts]]
- [[../90-ingested/_consolidado-specs-legado]]
- [[../90-ingested/_consolidado-providers-fluxos]]
