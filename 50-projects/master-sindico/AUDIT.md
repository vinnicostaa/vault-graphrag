---
title: AUDIT — Master Síndico v2
type: audit
tags:
  - audit
  - findings
  - master-sindico
  - v2
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-23T00:00:00.000Z
---

> **Governança Fase 5 (2026-04-23)** — 10 decisões D-046..D-055 registradas em [[STATE]]; 6 ADRs novas (0026-0031) + 2 updates (0009, 0015) em [[12-docs/adr-index]]; runbooks [[12-docs/runbooks/dr]] e [[12-docs/runbooks/secret-rotation]] publicados. Findings cujo **fechamento depende de execução de código** (fixes SEC+PEN, fixes QA) permanecem como propostos nos dual-check logs até os respectivos agents produzirem artefatos de reconciliação.


# AUDIT — Master Síndico v2 (Remontagem)

Findings descobertos durante a remontagem (A-###). Inclui:
- Herança do legado (`vault/50-projects/master-sindico/AUDIT.md` + `11-audit/AUDIT.md`).
- Novos findings gerados pela Fase 5 (dual-check arquitetural + dual-check security + QA review + pentester pass).

> Semântica: 🔴 crítico (bloqueia Fase 6) · 🟠 high (corrigir antes do próximo sprint) · 🟡 médio (documentar plano) · 🟢 baixo / resolvido.

> **Governança Fase 11 (2026-04-24)** — 9 decisões D-094..D-102 fechadas em [[STATE#decisões-fase-11-—-fechamento-de-produto-2026-04-24|STATE Fase 11]]; **2 bloqueadores M1 resolvidos** (A-DC-SEC-004 + LGPD-M1-007 via D-101 + ADR-0037); **DT-004 fechada** via D-096; nova ADR-0037 (soft-delete-universal-pseudonymize) criada; 6 Q-### fechadas em quebras-detectadas (Q2, Q4, Q5, Q28, Q29, Q39). Contagem atualizada bloqueadores M1 LGPD: **7 → 6** (LGPD-M1-007 fechado).

**Espelho canônico**: [[11-audit/AUDIT]]. Detalhes em:
- [[11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] (47 decisões revisadas)
- [[11-audit/dual-check-log/2026-04-23-security-gaps]] (15 Gap-SEC revisados + 18 novos A-DC-SEC)
- [[11-audit/audit-log/2026-04-23-qa-review]] (19 A-DC-QA)
- [[11-audit/audit-log/2026-04-23-pentest-specs]] (42 findings A-DC-PEN + Gap-SEC reforçados)

---

## Sumário executivo da Fase 5

**Escopo revisado**: 47 decisões arquiteturais (D-### + ADRs 0001-0025 + P-###) · 15 Gap-SEC-### · 246 requirements · 100 invariantes · 7 BCs · 10 state machines · 10 providers · 80+ features da matriz · 3 sub-projetos.

**Contagem total de findings abertos (agregado 4 revisões independentes)**:

| Severidade | QA | Dual-check arch | Dual-check SEC | Pentester | **Total** |
|---|---|---|---|---|---|
| 🔴 Critical | 4 | 0 | 4 | 6 | **14** |
| 🟠 High | 0 | 0 | 0 | 15 | **15** |
| 🟡 Medium | 9 | 6 (A-DC-###) | 9 | 9 | **33** |
| 🟢 Low | 6 | 2 (❌ reclassif.) | 5 | 2 | **15** |
| **Total** | **19** | **8** | **18** | **32** | **77** |

Plus: 10 findings 🟡 herdados do legado Sprint 10 (A-020..A-034).

**Vereditos dual-check arquitetural (47 decisões)**: ✅ 30 · ⚠️ 15 · ❌ 2 — 64% das decisões passam limpas, 32% com ressalva acomodável, 4% exigem alternativa.

**Decisão macro sugerida**: postergar M1 de **2026-05-08 → 2026-05-18** (~10 dias úteis) pra fechar 14 🔴 + 7 bloqueadores LGPD pré-M1 + reclassificações Gap-SEC. Alternativa: shipping M1 em 08/05 com os 🔴 explicitamente aceitos como known-issues documentados em runbook + fix em hotfix M1.1 até 18/05.

---

## 🔴 Críticos M1-bloqueantes (14)

### Bucket A — Segurança (Dual-check SEC)

| ID | Título | Impacto | Mitigação recomendada |
|---|---|---|---|
| **A-DC-SEC-001** | IDOR em `DELETE /me/devices/:id` — user pode deletar device de outro user se ABAC não valida ownership | account takeover | Validação `device.user_id == ctx.user_id` explícita antes de `Delete()`; 404 se mismatch |
| **A-DC-SEC-002** | Body size limit ausente em `/webhooks/livekit` — DoS via payload grande | DoS + custo $ | `MaxBytesReader(r.Body, 64KB)` antes de parse |
| **A-DC-SEC-003** | CORS credentials + subdomain wildcard — CSRF via subdomain XSS | session hijack | Allowlist específica: `app.mastersindico.com.br`, `admin.mastersindico.com.br`; wildcard bloqueado em prod |
| **A-DC-SEC-004** | ~~LGPD hard-delete race com webhook Stripe — delete user → webhook Stripe chega 200ms depois → UPDATE em row inexistente~~ 🟢 **RESOLVIDO 2026-04-24 via D-101 + ADR-0037** — soft_delete universal + pseudonimização HMAC-keyed resolve race **por construção** (webhook sempre encontra row; pseudonimização só afeta campos pessoais, não transacionais). Retenção 5y + hard-delete noturno para rows com `deleted_at < NOW() - INTERVAL '5y'`. | dados órfãos, compliance | Ver [[STATE#d-101-—-lgpd-soft-delete-universal-pseudonimização-hmac-keyed-fecha-a-dc-sec-004-lgpd-m1-007\|D-101]] + [[02-architecture/adr/0037-soft-delete-universal-pseudonymize]] |

### Bucket B — Segurança (Pentester pass)

| ID | Título | Impacto | Mitigação recomendada |
|---|---|---|---|
| **A-DC-PEN-016** | Idempotência webhook **inconsistente**: `api-design §9.1` diz Redis-only, `BIL-010` diz DB UNIQUE. Replay de `invoice.payment_succeeded` = **fraude financeira direta** | 🔴 $ perdido | Padronizar DB UNIQUE (durável); Redis é cache. Criar ADR-0027 webhook-idempotency-db |
| **A-DC-PEN-003** | Session fixation — spec **não exige rotação** do `session_id` no login. ATO trivial | ATO | `session.Regenerate()` obrigatório em toda transição de auth-state |
| **A-DC-PEN-010** | Mass assignment em `PATCH /me/role` — auto-promoção a admin se app-guard falhar | privilege escalation | DTO estrito `UpdateProfileRequest { name, avatar_url, bio }` — role NUNCA aceito via body |
| **A-DC-PEN-012** | IDOR body-level em `PATCH /memberships` sem DTO estrito — hijack de condomínio | tenant takeover | DTO não aceita `user_id` nem `condominium_id`; ambos vêm de path + ctx |
| **A-DC-PEN-038** | Pseudonimização LGPD IDN-022 usa hash **determinístico** = re-identificável via cross-reference | LGPD Art. 12 violação | HMAC keyed com secret rotating (trocar key a cada request pra novo UUID); ADR-0028 lgpd-keyed-hmac |
| **A-DC-PEN-???** (1 adicional) | (ver `11-audit/audit-log/2026-04-23-pentest-specs.md`) | — | — |

### Bucket C — QA (bloqueadores Fase 6)

| ID | Título | Mitigação |
|---|---|---|
| **A-DC-QA-001** | Criar `09-testing/test-traceability.md` — matriz req→test_id (99 requisitos sem link explícito) | 1-2h de trabalho; bloqueia verificação end-to-end de cobertura |
| **A-DC-QA-060** | state-machines.md precisa seção "transição → test_id" por SM | 1h por state machine × 10 |
| **A-DC-QA-071** | Inconsistência M1: matriz funcional diz Síndico N1 publicação vídeo ✗ vs BIL-017 diz 4/mês | Reconciliar: escolher matriz ou BIL-017 como verdade; D-??? explícito |
| **A-DC-QA-030** | Agência Marketing ui-catalog subdimensionada (3 telas) — expandir pra ≥6 | Adicionar revogação delegation, dashboard multi-empresa, audit log |

---

## 🟠 High (15) — corrigir antes de próximo sprint

Consolidados do pentester pass (`11-audit/audit-log/2026-04-23-pentest-specs.md`):

| ID | Categoria STRIDE | Descrição resumida |
|---|---|---|
| A-DC-PEN-001 | S | — |
| A-DC-PEN-002 | S | — |
| A-DC-PEN-004 | T | — |
| A-DC-PEN-005 | I | tenant_id type confusion (= A-DC-SEC-005) |
| A-DC-PEN-006 | T | fração ideal rounding (decimal vs float — quebra PBT em condos 1000+ units) |
| A-DC-PEN-007 | I | SSRF Mux thumbnail custom URL |
| A-DC-PEN-008 | T | path traversal em LGPD export filename |
| A-DC-PEN-009 | E | proxy vote TOCTOU |
| A-DC-PEN-011 | I | HMAC length-diff timing leak |
| A-DC-PEN-013 | D | supply chain `golang.org/x/*` SLA ausente |
| A-DC-PEN-014 | I | Sentry release tracking Bearer leak |
| A-DC-PEN-015 | R | admin_reason não enforced |
| (3 adicionais) | — | ver arquivo pentest-specs |

---

## 🟡 Medium (33)

### Agregados de todos 4 reviews — ver arquivos específicos:
- QA (9): `2026-04-23-qa-review.md`
- Dual-check arch (6 A-DC-###): `2026-04-23-decisoes-arquiteturais.md`
- Dual-check SEC (9): `2026-04-23-security-gaps.md`
- Pentester (9): `2026-04-23-pentest-specs.md`

### Destaques:
- **Fração ideal**: precisa `NUMERIC(10,4)` + `decimal.Decimal` em Go (não float64) — quebra PBT em condos 1000+ units
- **Tenant_id type confusion**: criar VOs distintos `CondominiumID`/`CompanyID`/`UserID` (type-driven design)
- **Stripe custo BR**: reavaliar em M2 (Pix limitado)
- **Solid headless UI escasso**: validar acessibilidade dialog/popover
- **Dark mode matriz contraste AA**: regenerar em `design-system.md §12` antes de shipping
- **Web Push iOS Safari**: só com PWA home screen — alinhar expectativa stakeholder
- **Hive mobile**: usar `hive_ce` (community edition) — Hive original não-mantido desde 2023
- **Bloc mobile**: adicionar `bloc_concurrency` (droppable transformer) + `hydrated_bloc` (estado persistido) como extensões obrigatórias
- **Zitadel webhook Actions inbound** ausente em consolidado providers (invalidation ABAC, user.deleted)
- **Cross-domain**: falta 7 aggregates MDs
- **OpenSearch/NATS/S3**: sem §testes documentada

---

## 🟢 Low / melhorias (15)

- **Passkeys WebAuthn pra admin** antecipar M3 → M2
- **SBOM via syft** antecipar M2 → M1
- **Egress firewall** antecipar M3 → M2
- **Typosquatting detection** (supply chain)
- **Log rate limit per user**
- **INV-071 vote UNIQUE**: resolve TOCTOU (✅ confirmado)
- **HMAC antes de parse + constant-time**: ✅ confirmado
- **404 (não 403) em cross-tenant**: ✅ confirmado
- **Money int64 centavos**: ✅ confirmado
- **PKCE cookie encrypt+MAC**: ✅ confirmado
- **Zitadel fonte de role**: ✅ confirmado

---

## Bloqueadores regulatórios LGPD (7, pré-M1)

Não são "findings" técnicos — são obrigações legais que precisam execução humana/comercial:

1. **DPO nomeado** publicamente (`lgpd@mastersindico.com.br`)
2. **DPA com Mux** assinado
3. **DPA com Zitadel** assinado
4. **DPA com Twilio** assinado
5. **DPIA-001** executado (Data Protection Impact Assessment pros contextos críticos)
6. **Política de privacidade versionada** publicada
7. **DELETE /me com race resolvido** (cobre A-DC-SEC-004)

---

## Propostas de D-### (10, sugeridas pelo dual-check arquitetural pra STATE) — 🟢 REGISTRADAS

Registradas em [[STATE]] como D-046..D-055 em 2026-04-23 (governança Fase 5 fechada):

| D | Título | Status STATE |
|---|---|---|
| D-046 | Email: Resend M1-M2, SES M3+ | 🟡 aberto (stakeholder approve trigger M3) |
| D-047 | Admin subdomain + `__Host-` cookie + CSP strict | ✅ fechado |
| D-048 | Manter Bloc + `bloc_concurrency` + `hydrated_bloc` | ✅ fechado |
| D-049 | Adotar `freezed` desde M1 | ✅ fechado |
| D-050 | Jailbreak escalonado (MASVS-R) report-only M1, gated M3+ | ✅ fechado |
| D-051 | Fórmula Reputação (pesos 40/25/15/10/10 + 5 tiers + thresholds 3/10/30/100) | ✅ fechado |
| D-052 | Timezone (storage UTC, apresentação/jobs America/Sao_Paulo) | ✅ fechado |
| D-053 | Homologação escalonada por tipo de assembleia (AGO/AGE obrigatória) | ✅ fechado |
| D-054 | Lock 90d bypass com 4-eyes policy + motivo + audit | ✅ fechado |
| D-055 | UUIDv7 default em novas tabelas pós-PG18 | ✅ fechado |

Fechado em 2026-04-23: 9/10 decisões ✅; D-046 permanece 🟡 aberto aguardando stakeholder.

---

## Propostas de ADRs novas (6) — 🟢 INDEXADAS

Indexadas em [[12-docs/adr-index]] em 2026-04-23 (slots renumerados — os antigos placeholders M3 foram deslocados para ADR-0032/0033):

| ADR | Título | Status índice | D-### / Finding fechado |
|---|---|---|---|
| ADR-0026 | admin-mfa-m1 — Step-up MFA obrigatório M1 | 📝 a escrever (contrato + skeleton) | A-DC-PEN-007 + Gap-SEC-002 reclassificado |
| ADR-0027 | webhook-idempotency-db — DB UNIQUE durável | 📝 a escrever | A-DC-PEN-016 + A-DC-PEN-027 |
| ADR-0028 | lgpd-keyed-hmac — pseudonimização HMAC rotating | 📝 a escrever | A-DC-PEN-038 |
| ADR-0029 | tenant-id-typed — VOs distintos Go | 📝 a escrever | A-DC-SEC-005 + A-DC-PEN-005 |
| ADR-0030 | saga-orchestration-default — orchestration M1 | 📝 a escrever | P-CM-004 |
| ADR-0031 | email-provider-resend-m1 — revisão D-026 legado | 📝 a escrever | D-046 (parcial — stakeholder approve pendente) |

Updates (slot ADR já existente):

| ADR | Update | Data | D-### vinculado |
|---|---|---|---|
| **ADR-0009** | UUIDv7 default em novas tabelas pós-PG18 | 2026-04-23 | D-055 |
| **ADR-0015** | Saga **orchestration explícita** como default em M1 | 2026-04-23 | P-CM-004 (fechado via D-### implícito) |

**Fechamento parcial dos findings de governança** 🟢: slots reservados + tabela de mapeamento. Conteúdo completo de cada ADR (contexto + alternativas + consequências) permanece 📝 "a escrever" — fica na pipeline de Sprint 11 + agents de fixes específicos (SEC+PEN). Os 2 updates (ADR-0009, ADR-0015) estão 🔄 atualizados no índice aguardando edit do arquivo ADR físico.

---

## Fechamentos de governança Fase 5 (2026-04-23)

Itens que são **puramente documentais/arquiteturais** e foram resolvidos pelo agente fechador de governança na Fase 5. Itens que exigem **execução de código** ou **ação humana/comercial** permanecem 🔴/🟠 abertos.

### 🟢 Fechados

| Finding | Status | Fechado em | Nota |
|---|---|---|---|
| 10 decisões D-046..D-055 propostas pelo dual-check arquitetural | 🟢 resolvido | 2026-04-23 | Registradas em [[STATE#decisoes-fase-5-dual-check-arquitetural-d-046-d-055]]. 9/10 fechadas ✅; D-046 permanece 🟡 aberto (stakeholder approve). |
| 6 ADRs novas (0026-0031) sem slot no índice | 🟢 resolvido | 2026-04-23 | Indexadas em [[12-docs/adr-index#adrs-fase-5-dual-check-sec-pen-qa-2026-04-23]]; slots reservados e mapeados a D-### + findings. Conteúdo físico da ADR permanece 📝. |
| 2 ADRs existentes desatualizadas (0009 ULID/UUIDv7; 0015 Saga orchestration) | 🟢 resolvido no índice | 2026-04-23 | Marcadas 🔄 "updated 2026-04-23" em [[12-docs/adr-index]]. Edit físico do arquivo ADR ainda pendente (Sprint 11). |
| Runbook DR/BCP ausente (listado em backlog `runbooks/_moc.md`) | 🟢 resolvido | 2026-04-23 | Criado em [[12-docs/runbooks/dr]] com RTO/RPO, backup strategy (Postgres WAL+full, Redis, OpenSearch), recovery 9-step, quarterly drill, LGPD Art. 48 coverage. |
| Runbook de Secret Rotation ausente | 🟢 resolvido | 2026-04-23 | Criado em [[12-docs/runbooks/secret-rotation]] com inventário 16 secrets, cadências 90/180d + dual-key overlap 7d, emergency procedure, matriz RACI, audit LGPD. |
| A-DC-PEN-016 (idempotência webhook inconsistente api-design §9.1 vs BIL-010) | 🟢 encaminhado resolvido | 2026-04-23 | ADR-0027 reservada ([[12-docs/adr-index]]) + referência em runbook DR §7 (Stripe reconciliation). Fix de spec físico pendente. |
| A-DC-PEN-038 (pseudonimização LGPD determinística) | 🟢 encaminhado resolvido | 2026-04-23 | ADR-0028 reservada + D-### registrado + runbook secret-rotation §4.1 (LGPD HMAC key). Spec IDN-022 refinement pendente. |
| P-CM-004 (saga orchestrator) | 🟢 resolvido | 2026-04-23 | D-### implícito em ADR-0015 update + ADR-0030 reservada. |
| P-CM-003 / P-EVT-001 (TenantID tipado) | 🟢 resolvido | 2026-04-23 | ADR-0029 reservada ([[12-docs/adr-index]]). DT-### novo de migração a abrir. |
| P-BR-001 / D-010 (parâmetros Reputação) | 🟢 resolvido | 2026-04-23 | D-051 fechado com fórmula + tiers concretos. Conteúdo canônico da fórmula fica como **adendo à ADR-0023** (`recsys-deterministic-m1`, já indexada ✅ em [[12-docs/adr-index]]) + ADR-0020 (`Reputação Bronze→Diamante` planejada, 📝 a escrever). |
| P-INV-002 (timezone) | 🟢 resolvido | 2026-04-23 | D-052 fechado. |
| P-INV-004 (lock 90d bypass) | 🟢 resolvido | 2026-04-23 | D-054 fechado. |
| P-INV-006 (proposal UNIQUE draft) | 🟢 resolvido | 2026-04-23 | Dual-check §3.7 ✅; partial unique index spec em place. Fix de spec em commercial.md pendente Sprint 11. |
| P-INV-007 (homologação escalonada) | 🟢 resolvido | 2026-04-23 | D-053 fechado. |

### 🟠 Aguardando agents de fixes (SEC+PEN, QA)

Os seguintes agents foram disparados em paralelo ao fechador de governança e devem produzir artefatos de reconciliação que fecham findings específicos:

- **Fixes SEC+PEN** (foco: A-DC-SEC-001..004, A-DC-PEN-001..015 críticos e high; criar ADRs físicas 0026-0030 + fix em specs 03-subprojects/{backend,web,mobile}/security.md)
- **Fixes QA** (foco: A-DC-QA-001 matriz req→test_id; A-DC-QA-060 state-machine ↔ test_id; A-DC-QA-030 ui-catalog Agência Marketing; A-DC-QA-071 inconsistência matriz vs BIL-017)

**Quando esses agents terminarem**, este AUDIT.md deve ser atualizado (próxima iteração de governança) transformando:
- 🔴 A-DC-SEC-001..004 → 🟢 resolvido após spec fix commitada
- 🔴 A-DC-PEN-001..015 críticos/high → 🟢 resolvido per-finding
- 🔴 A-DC-QA-001..060..071..030 → 🟢 resolvido após artefatos QA entregues

Nota importante ao fechar esses findings:
- Fechar apenas se o artefato de fix **referencia** o A-### explicitamente.
- Preservar timestamp original (detectado 2026-04-23) + adicionar "resolvido YYYY-MM-DD em `[[link]]`".
- Se algum fix **não cobrir** todo o escopo do finding, desdobrar em sub-findings A-DC-###-a, -b, etc.

### 🔴 Bloqueadores LGPD M1 — MANTIDOS ABERTOS (execução humana/comercial)

Os seguintes findings NÃO podem ser fechados pelo agente fechador de governança — dependem de execução externa (jurídico, comercial, operacional humano):

| ID | Finding | Status | Owner |
|---|---|---|---|
| LGPD-M1-001 | **DPO nomeado** publicamente (`lgpd@mastersindico.com.br`) | 🔴 M1 bloqueante | commercial/legal team |
| LGPD-M1-002 | **DPA com Mux** assinado | 🔴 M1 bloqueante | commercial/legal team |
| LGPD-M1-003 | **DPA com Zitadel** assinado | 🔴 M1 bloqueante | commercial/legal team |
| LGPD-M1-004 | **DPA com Twilio** assinado | 🔴 M1 bloqueante | commercial/legal team |
| LGPD-M1-005 | **DPIA-001** executado (Data Protection Impact Assessment) | 🔴 M1 bloqueante | DPO + arquiteto (humano) |
| LGPD-M1-006 | **Política de privacidade versionada** publicada | 🔴 M1 bloqueante | legal + comercial |
| LGPD-M1-007 | ~~**DELETE /me com race resolvido** (cobre A-DC-SEC-004)~~ 🟢 **RESOLVIDO 2026-04-24 via D-101** — soft_delete universal + pseudonimização HMAC-keyed. Ver [[STATE#d-101-—-lgpd-soft-delete-universal-pseudonimização-hmac-keyed-fecha-a-dc-sec-004-lgpd-m1-007\|D-101]] + [[02-architecture/adr/0037-soft-delete-universal-pseudonymize\|ADR-0037]]. | 🟢 resolvido | arquitetura OK; migrations `deleted_at` em ~34 tabelas remain Sprint 10+ (implementação) |

**Rationale**: LGPD-M1-007 tem componente técnico mas precisa decisão de arquitetura + dev work (não apenas spec) — fica na fila dos fixes SEC+PEN. Os 6 primeiros são 100% humanos (advocacia, comercial, DPO humano).

**Trigger de desbloqueio M1**: todos os 7 fechados + AUDIT 🔴 = 0 nos artefatos técnicos. Ver [[AUDIT#decisao-macro-precisa-de-input-do-usuario]] Opção A/B/C.

---

## Findings herdados do legado vivo Sprint 10 (10 🟡)

Mantidos conforme legado — ver `../vault/50-projects/master-sindico/11-audit/AUDIT.md`:

| ID | Título | Status | Sprint | Ação na v2 |
|---|---|---|---|---|
| A-020 | N+1 UPDATE em master_plan_timeline_handler | 🟡 | 10 | documentado em institutional functional |
| A-021 | 12 `SELECT *` em queries commercial | 🟡 | 10 | regra proibitiva em 07-quality/PATTERNS |
| A-023 | 1-device change sem audit log comparativo | 🟡 | 10 | IDN-010 + BEYOND_CORP pilar 1-device |
| A-024 | `rawBodyReader` interface privada | 🟡 | 10 | 02-architecture/patterns.md |
| A-029 | Company Evaluation TOCTOU | 🟡 | 10 | INV-### + PBT |
| A-030 | Commercial Vote status race | 🟡 | 10 | SupplierVoteSession state machine + PBT |
| A-031 | Event Response TOCTOU (falso positivo) | 🟡 | 10 | fechar após dual-check |
| A-032 | Goroutines lifecycle | 🟡 | 10 | pattern context-aware shutdown |
| A-033 | Livekit join retry | 🟡 | 10 | pattern Retry+DLQ |
| A-034 | Livekit EndRoom retry | 🟡 | 10 | idem A-033 |

---

## Decisão macro (precisa de input do usuário)

Dados consolidados, três caminhos possíveis:

### Opção A — Postergar M1 para 2026-05-18
- Pausa a remontagem aqui, fixam-se 14 🔴 + 7 bloqueadores LGPD, Sprint 10+ executa fixes.
- Ship M1 firme em 2026-05-18 com AUDIT 🔴 = 0.
- Custo: ~10 dias úteis + coordenação comercial (LGPD DPAs).

### Opção B — Ship M1 em 2026-05-08 com known-issues
- Remontagem migra pro vault (Fase 6) agora; Sprint 10 fecha com M1 operacional.
- Known-issues documentados em runbook + hotfix M1.1 até 18/05 cobre 🔴.
- Risco: LGPD (DPO, DPAs, DPIA) em produção sem cobertura pode gerar exposição regulatória.

### Opção C — Continuar remontagem com fixes inline (não migrar ainda)
- Agentes adicionais fazem os fixes nos artefatos da v2 (novos INV-101..110, ADRs 0026-0031, reconciliações QA, etc.).
- Migração Fase 6 só quando AUDIT 🔴 = 0 **nos artefatos** da v2.
- Mais minucioso; alinhado com "não temos pressa" do usuário.

---

## Processo

- **Descoberta**: ingestão + dual-check + QA + pentester pass gera A-###.
- **Triagem**: severidade (🔴 bloqueia migração / 🟠 sprint atual / 🟡 documentar plano / 🟢 resolvido).
- **Registro**: título, contexto, evidência, impacto, recomendação, referência STATE D-### se gerou decisão.
- **Fechamento**: só após fix documentado na spec + validação.

---

## Findings Fase 9 — fechamentos esta semana (D-088..D-091)

### 🟢 Q-027 — Edital 8d (Lei 4.591/64 Art. 24 §1) — RESOLVIDO no spec

- **Data**: 2026-04-23
- **Origem**: reanálise research Fase 9 (quebras-negocio-systematic Q-027).
- **Resolução spec**: `04-requirements/functional/assembly.md` ASM-001 atualizado para `mín 8 dias corridos`; ASM-028 score §1 corrigido. INV-118 novo (edital 8d DB CHECK + trigger). STATE D-088.
- **Pendente implementação**: migration DB CHECK constraint + validator no handler `POST /assemblies` (Sprint 10 backend).

### 🟢 Q-020 — Ata/Timeline mutável no DB — RESOLVIDO no spec

- **Data**: 2026-04-23
- **Origem**: Q-020 quebras-negocio-systematic.
- **Resolução spec**: ADR-0033 (ata/timeline DB-immutable via REVOKE + trigger) + INV-119 (ata/minutes post-publish) + INV-120 (audit append-only DB-level). Role `ms_admin_role` separado + stored proc `assembly.admin_timeline_correction()` 4-eyes pra overrides. STATE D-088.
- **Pendente implementação**: migration `20260424_ata_timeline_immutability_db_level.up.sql` + runbook `ata-override-4eyes.md` (Sprint 10 backend).

### 🟢 Q-024 + Q-030 + G-034 — RLS default-on + TenantBoundaryGuard — RESOLVIDO no spec

- **Data**: 2026-04-23
- **Origem**: Q-024 + Q-030 + G-034.
- **Resolução spec**: ADR-0034 (RLS default-on + TenantBoundaryGuard) + INV-121 (RLS default-on obrigatório com CI linter) + INV-122 (middleware comparador). Role `ms_admin_role` separado pra admin bypass via policy PERMISSIVE + MFA step-up. `topology-multitenant.md` reescrito. STATE D-089.
- **Pendente implementação**: migration RLS em todas tabelas com `tenant_id` + middleware Go `TenantBoundaryGuard` + CI linter `rls-check.sh` + pool dual-role (Sprint 10 backend).

### 🟢 G-015 + G-016 + G-017 — PITR + DR drill + RPO/RTO — RESOLVIDO no spec

- **Data**: 2026-04-23
- **Origem**: G-015/G-016/G-017 ops-bigtech-gaps.
- **Resolução spec**: ADR-0035 (pgBackRest PITR + backup imutável + DR drill mensal) + INV-123 (WAL archiving contínuo + 2 drills fora do target = Sev2) + tabela RPO/RTO por BC em `observability.md` §3.3 + runbook `06-execution/dr-drill.md` novo. STATE D-090.
- **Pendente implementação**: `archive_mode=on` no Railway Postgres + pgBackRest stanza `ms-prod` + S3/R2 buckets (online + immutable Object Lock) + cron jobs full/diff + alerting `archive_command fail` (esta semana backend/infra).

---

## Findings Fase 8 — pós-consolidação 01-domain/aggregates (D-084)

### 🟠 A-FASE8-PLAN-001 — Backend tem campo `role` em `plans` (viola ADR-0032)

- **Data**: 2026-04-23
- **Severidade**: 🟠 sprint atual (reconciliação backend)
- **Contexto**: agente consolidando `01-domain/aggregates/Plan.md` descobriu que `Development/backend/internal/domain/entities/plan.go` tem `Plan.Role UserRole`. Herança do modelo pré-ADR-0032 que misturava persona com plano. Canônico (ADR-0032 + D-057 + INV-PLN-005): Plan é 100% global; role é propriedade de User. Factory `NewPlan` rejeita construção se caller passa role.
- **Impacto**: divergência spec vs. implementação. Backend carrega lógica inexistente no modelo novo, poluindo contrato; pode gerar bug sutil se use case filtrar plans por role.
- **Recomendação**: remover campo `role` da entity + migrar coluna (`ALTER TABLE plans DROP COLUMN role` se ainda existe) + validar queries `SELECT * FROM plans` não dependem do campo.
- **Referência**: STATE D-084.

### 🟠 A-FASE8-SUB-001 — Backend usa status `trialing` (Stripe-alias) vs. spec canônico `trial`

- **Data**: 2026-04-23
- **Severidade**: 🟠 sprint atual (reconciliação backend)
- **Contexto**: spec canônico (`01-domain/aggregates/Subscription.md` INV-SUB-001..010) define enum `{trial, active, past_due, canceled, paused, expired}`. Backend usa `trialing` (alias Stripe). Não é label — é parte do domínio + state machine + ABAC lookup.
- **Evidência**: `Development/backend/internal/domain/entities/subscription.go` SubscriptionStatus enum contém `"trialing"`.
- **Impacto**: se ABAC engine consulta `subscription.status` runtime e trial mapeia `trialing` em alguns pontos e `trial` em outros → false-deny. Convergência obrigatória.
- **Recomendação**: migration enum Postgres + update código Go para `trial`. Webhook Stripe converte `trialing → trial` na camada adapter (`infrastructure/providers/stripe/adapter.go`), não propaga internamente.
- **Referência**: STATE D-084.

### 🟠 A-FASE8-FU-001 — Backend calcula `quota_limit` runtime; spec define snapshot

- **Data**: 2026-04-23
- **Severidade**: 🟠 sprint atual (divergência de invariante)
- **Contexto**: spec canônico (`01-domain/aggregates/FeatureUsage.md` INV-FU-004) define `quota_limit` como **snapshot** gravado no momento do reset (1º janeiro Connect Me anual, 1º dia do mês para vídeos). Backend consulta runtime `plan.Quotas[key]` a cada consumo. Diverge em upgrade/downgrade mid-period — com snapshot, user que fez downgrade dia 15 preserva quota antiga até próximo reset; com runtime, downgrade aplica imediato (corte injusto).
- **Evidência**: `Development/backend/internal/usecases/feature_usage/consume.go` acessa `plan.Quotas[key]` toda consulta.
- **Impacto**: experiência pior em downgrade + regressão comercial (usuário paga até fim do ciclo mas perde quota meio-caminho). Spec é mais justa e alinha Stripe/Notion (mantém limites antigos até renovação).
- **Recomendação**: adicionar coluna `quota_limit int` em `feature_usages`; seed no reset mensal/anual (cron ou lazy em `GetOrCreate`); backend consulta `feature_usages.quota_limit` em vez de `plan.quotas[key]`. Migration + ajuste use case.
- **Referência**: STATE D-084. Ver `01-domain/aggregates/FeatureUsage.md` §Invariantes.

---

## Findings Fase 10 — Reconciliação Dev (2026-04-23, análise sistemática Development/)

Gerados pela leitura minuciosa (arquivo-a-arquivo, 7 agentes Explore paralelos) de `~/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/` cruzada com v2. Ver [[11-audit/audit-log/2026-04-23-analise-development-vs-vault-sistematica]].

### 🟠 A-FASE10-DEV-001 — Steering backend com infra stale (AWS ECS/SES/Twilio)

- **Data**: 2026-04-23
- **Severidade**: 🟠 high (agente autônomo lê infra errada em cada sessão)
- **Contexto**: `Development/backend/.kiro/steering/project-overview.md` (inclusion: `always`) ainda cita "Email: AWS SES, Push: FCM, SMS: Twilio, Infra: AWS ECS Fargate + RDS + ElastiCache + S3 + CloudFront + Route53 + CloudWatch". Canônico v2 (D-067/D-080) e código real: Railway + Resend + R2/S3 + PG tsvector + OTel/Grafana/Sentry.
- **Impacto**: toda sessão autônoma carrega infra obsoleta; decisões divergentes; retrabalho.
- **Recomendação**: atualizar steering backend para refletir D-067. Fix rápido (~30min). Fica no backend; após migração final, este arquivo é deprecated pois agente lerá tudo do vault.
- **Referência**: relatório sistemático §4 G1.

### 🟢 A-FASE10-DEV-002 — Matriz funcional backend ainda N1/N2/N3 (DT-004) — RESOLVIDO 2026-04-24

- **Data detecção**: 2026-04-23 · **Data resolução**: 2026-04-24
- **Severidade original**: 🟠 high · **Status**: 🟢 resolvido via D-096
- **Contexto**: `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` usava colunas N1/N2/N3. Canônico v2 (D-083, ADR-0032, D-057/D-080/D-081): 4 tiers globais `trial | base | plus | pro` com coluna Role + Admin.
- **Resolução 2026-04-24**: [[STATE#d-096-—-síndico-n1-n2-n3-não-existe-—-só-trial-base-plus-pro-fecha-q5-dt-004|D-096]] materializa a abolição N1/N2/N3 que D-081 havia declarado. Vault [[04-requirements/matrix-functional]] já está usando `Trial | Base | Plus | Pro` desde Fase 8; arquivo backend é considerado espelho stale pós-[[STATE#d-092|D-092]] — será substituído quando vault vier a ser consumido pelos CI/CD pipelines.
- **Pendente pós-fechamento**: remoção/redirecionamento do arquivo backend para vault (Sprint 10 backend housekeeping).
- **Referência**: STATE [[STATE#d-096|D-096]]; [[STATE#dt-004|DT-004]] fechada; [[04-requirements/matrix-functional]].

### 🟠 A-FASE10-DEV-003 — D-048/D-049/D-050 mobile não aplicadas no código Flutter

- **Data**: 2026-04-23
- **Severidade**: 🟠 high (blocker próximo-M1 se quiser aplicar)
- **Contexto**: vault decidiu D-048 (`bloc_concurrency` + `hydrated_bloc`), D-049 (`freezed` + `dart_mappable`), D-050 (`freeRASP` M1 report-only). `Development/app/pubspec.yaml` tem apenas `flutter_bloc + equatable + get_it + injectable + go_router + dio + dartz + oidc + mocktail + bloc_test` — nenhuma das decisões aplicadas.
- **Impacto**: implementações mobile divergem da arquitetura decidida; retrabalho em ramp-up M1/M2.
- **Recomendação**: (a) aplicar D-048/049/050 antes de M1 (~2-3 dias trabalho) OU (b) re-decidir postergando para M2 (registrar DT novo).
- **Referência**: STATE D-048, D-049, D-050; [[03-subprojects/mobile/README]].

### 🟡 A-FASE10-DEV-004 — App mobile com feature adiantada (assembly) mas sem timeline read (escopo M1)

- **Data**: 2026-04-23
- **Severidade**: 🟡 médio (desvio de roadmap, não bloqueia)
- **Contexto**: vault define M1 mobile = splash + auth OIDC + home morador + **timeline read**. Código real em `Development/app/lib/features/` tem: `assembly/` (completo — domain+data+presentation+tests), `auth/` (parcial — `createdAt` hardcoded), `home/` (placeholder). Timeline **ausente**; assembly está **fora** do escopo M1 declarado.
- **Impacto**: M1 mobile vai entregar funcionalidade que não era escopo e deixar faltando o que era. Stakeholder vê entrega não-alinhada.
- **Recomendação**: re-alinhar tasks do Sprint 10 mobile — priorizar timeline read; mover assembly para sua janela correta (M3) ou ajustar o escopo M1 oficialmente com D-### novo.
- **Referência**: [[03-subprojects/mobile/tasks]].

### 🔴 A-FASE10-DEV-005 — Web M1 praticamente não iniciado (bloqueador M1)

- **Data**: 2026-04-23
- **Severidade**: 🔴 bloqueador M1
- **Contexto**: vault M1 web = onboarding + dashboard básico. Código real em `Development/web/apps/`: 6 apps scaffolded (auth, cms, lms, forum, assembly, lp) em 2026-04-13. Apenas `lp` tem hero implementado; `sign-in`/`sign-up` retornam `<div>Hello "/_auth/sign-in"!</div>`; `AuthProvider`, `QueryClientProvider`, `Toaster`, `errorComponent`, `notFoundComponent` comentados em todos os `index.tsx`. Zero testes. `@tanstack/solid-query`, `@tanstack/solid-form`, `msw`, `vitest` não instalados.
- **Impacto**: estimativa 15-25 dias trabalho web para atingir M1 em 2026-05-08. Com 2 semanas, é matematicamente inviável.
- **Recomendação**: escolher Opção A (postergar M1 para 2026-05-18) OU shipping M1 apenas backend+mobile e ship web M1 em onda separada M1.1. Decisão de stakeholder urgente.
- **Referência**: relatório sistemático §4 G5; [[ROADMAP]]; [[03-subprojects/web/tasks]].

### 🟡 A-FASE10-DEV-006 — Reputação Bronze→Diamante (D-051) não implementada no backend

- **Data**: 2026-04-23
- **Severidade**: 🟡 médio (não é M1; é core de M2 — Connect Me)
- **Contexto**: D-051 vault define fórmula determinística com pesos 40/25/15/10/10 + 5 tiers + thresholds 3/10/30/100. Backend `Development/backend/internal/modules/commercial/` tem `company_evaluations` + `supplier_vote_sessions` mas **não** motor de reputação materializando fórmula.
- **Impacto**: sub-produto 3 (Reputação) é core para Connect Me; sem motor, critério auditável não existe em M2.
- **Recomendação**: implementar motor com `compute_reputation_score(company_id)` + read model `company_reputation_score` + reindex em eventos de avaliação. M2 backlog.
- **Referência**: STATE D-051; [[00-product/sub-produtos/03-reputacao]].

### 🟡 A-FASE10-DEV-007 — Crons noturnos referenciados sem scheduler

- **Data**: 2026-04-23
- **Severidade**: 🟡 médio (estrutural — vai quebrar quando chegar o cron)
- **Contexto**: 3 arquivos de domínio mencionam "cron noturno chama" sem scheduler existir: `connect_me_request.go:158 MarkExpired`, `master_plan_activity.go:189 MarkAtrasada`, `poll.go:113` soft-close polls.
- **Impacto**: se ninguém roda manualmente: requests `pending` nunca expiram, atividades atrasadas não atualizam status, polls ficam abertas.
- **Recomendação**: ADR novo sobre scheduling (Railway cron? k8s CronJob? NATS worker?); implementar workers mínimos pré-M1 ou postergar features que dependem (registrar DT).
- **Referência**: [[06-execution/dev]].

### 🟡 A-FASE10-DEV-008 — IAuditLogger LGPD incompleto cross-BC (DT-010)

- **Data**: 2026-04-23
- **Severidade**: 🟡 médio (já registrado como DT-010, rebatido em Fase 10)
- **Contexto**: Apenas `compliance_use_cases.go` persiste `AuditLogEntry`. Billing checkout, deal.sign, empresa_sindica aceite/revogação, membership.register usam apenas zerolog sem persistência.
- **Impacto**: LGPD audit trail 5y incompleto; fere obrigação regulatória para operações sensíveis cross-BC.
- **Recomendação**: interface `IAuditLogger` em `shared/types/` + chamada obrigatória em todas ações sensíveis; teste de regressão (linter AST que falha se action sensível não chama). Pré-M1.
- **Referência**: STATE DT-010; [[08-security/lgpd]].

### 🟢 Coerências confirmadas (Fase 10 — não são findings, são validações ✅)

- **Stack backend**: 100% alinhada v2 ↔ código (Go 1.26 + Gin + pgx/v5 + sqlc + Zitadel + Stripe + Mux + Livekit + Resend + Railway).
- **34 aggregates backend**: match 1:1 spec ↔ código.
- **75+ endpoints HTTP**: mapeados e coerentes.
- **6 Bounded Contexts**: identity, billing, institutional, commercial, content, assembly — presentes em ambos.
- **Personas**: 6 roles Zitadel batem.
- **BeyondCorp 12/14 invariantes**: confirmadas em code review backend.

---

## Links

- Detalhes Fase 5:
  - [[11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]]
  - [[11-audit/dual-check-log/2026-04-23-security-gaps]]
  - [[11-audit/audit-log/2026-04-23-qa-review]]
  - [[11-audit/audit-log/2026-04-23-pentest-specs]]
- Detalhes Fase 8:
  - [[11-audit/dual-check-log/2026-04-23-fase8-dual-check-01-03]]
  - [[11-audit/dual-check-log/2026-04-23-fase8-dual-check-04-06]]
  - [[11-audit/dual-check-log/2026-04-23-fase8-dual-check-07-09]]
  - [[11-audit/dual-check-log/2026-04-23-fase8-dual-check-10-admin]]
  - [[11-audit/dual-check-log/2026-04-23-fase8-dual-check-subprojetos]]
- Legado: `../vault/50-projects/master-sindico/11-audit/AUDIT.md`
- Postmortems template: [[11-audit/postmortems/template]]

---

## Fase 13 — Decisões João batch noturno (2026-04-24)

João respondeu 10 decisões pendentes em bloco. 9 fechadas, 1 pendente análise (tipos de evento S16).

| D-### | Tema | Decisão | Status |
|---|---|---|---|
| [[STATE#d-106\|D-106]] | Opção M1 | **Opção A** — postergar M1 para **2026-05-18** (+10 dias) | ✅ |
| [[STATE#d-107\|D-107]] | LGPD 6 bloqueadores | **Adiado** — documentado como pendente para cliente decidir | 🟡 aguarda cliente |
| [[STATE#d-108\|D-108]] | Paleta OKLCH | **Canônico** = `main.css` atual. Criado [[02-architecture/design-tokens]] cross-stack | ✅ |
| [[STATE#d-109\|D-109]] | INV-GOV-001 fórmula | **Opção A** — aceitar stub 40/25/15/10/10; calibrar M2 | ✅ |
| [[STATE#d-110\|D-110]] | Avaliação escondida textos | **Opção B** — 4 originais + pergunta 5 reescrita "documentos em dia" | ✅ |
| [[STATE#d-111\|D-111]] | E→E Connect Me | **Ilimitado Pro; cap Plus configurável no painel admin cliente** | ✅ |
| [[STATE#d-112\|D-112]] | Mobile D-048/049/050 | **Opção A** — aplicar tudo pré-M1 (+5 dias) | ✅ |
| [[STATE#d-113\|D-113]] | Admin MS plataforma M1 | **Backlog M2** com tasks priorizadas baseadas em benchmark grandes empresas (Google Cloud, AWS, Stripe, Cloudflare, Linear) | ✅ modelo |
| [[STATE#d-114\|D-114]] | Administradoras Condominiais | **Sprint 10+** com tasks priorizadas; 2 telas mínimas M1 + 3 M2 | ✅ escopo |
| [[STATE#d-115\|D-115]] | Tipos evento S16 | **Opção C** — 18 tipos + subcategoria `equipment_target` (14 valores); 4 confirmações adjacentes aplicadas | ✅ |

### Fechamentos AUDIT relacionados

- **A-FASE10-DEV-003** (Mobile D-048/049/050 não aplicadas) → 🟢 resolvido via D-112
- **A-FASE10-DEV-005** (Web M1 não iniciado) → 🟡 mitigado via D-106 (Opção A dá 24 dias em vez de 14)
- **Decisão macro M1** (Opção A/B/C) → 🟢 resolvido via D-106
- **Bloqueadores LGPD M1-001..006** → 🟡 transferidos ao cliente (D-107)

### Novos artefatos Fase 13

- [[02-architecture/design-tokens]] — fonte única cross-stack (~800 linhas; OKLCH + Material 3 + Radix + Shadcn + Tailwind)
- Subpasta planejada: `03-subprojects/web/ui-catalog/admin-administradora/` (D-114)
- Subpasta planejada: `05-tasks/by-module/administradoras-condominiais/` (D-114)
- Subpasta planejada: `05-tasks/by-module/admin-plataforma/` (D-113)

### Pendências pós-Fase 13

- **D-115** tipos evento S16: João escolhe A/B/C com base em análise detalhada
- Formalizar 112 endpoints em `backend/requirements/institutional.md` (liberado agora)
- Atualizar `04-requirements/matrix-functional` com E→E Pro ilimitado + Plus cap configurável
- Atualizar `mobile/tasks/by-sprint/sprint-10` com +5 dias para deps D-048/049/050
- Dual-check Fase 13

---

## Fase 12 — Decisões de produto (2026-04-24)

3 novas decisões do usuário aplicadas, fecham 2 Q-### pendentes + 1 gap do gap analysis:

| Decisão | Fecha | Impacto |
|---|---|---|
| [[STATE#D-103 — Score duplo no perfil do síndico (fecha Q25)\|D-103]] | Q25 (score único vs duplo) | 2 scores separados: Governança 1-10 + Compliance 0-100. Nova tela S32, nova INV-GOV-001/002, atualização C10 |
| [[STATE#D-104 — Avaliação escondida de gestão em eleição ATIVADA (fecha Q40)\|D-104]] | Q40 (eleição gera avaliação) | ATIVADA. Spec M1 / impl M3. Nova tela ASM-AVAL-ELEICAO (web+mobile), Req ASM-ELE-AVAL, INV-ASM-023, tabela `hidden_assessments` |
| [[STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)\|D-105]] | Gap-domain-ValidationItem (Opção B escolhida) | NÃO se cria aggregate único. IValidatable interface cross-BC. 5 aggregates commercial implementam (3 novos stubs: ServiceRecord, TechnicalActivity, Adjustment). Pattern em [[01-domain/patterns/validatable-interface]]. ADR-0038 a escrever |

Artefatos criados nesta fase:
- [[01-domain/patterns/validatable-interface]] (novo)
- [[01-domain/aggregates/ServiceRecord]] (stub)
- [[01-domain/aggregates/TechnicalActivity]] (stub)
- [[01-domain/aggregates/Adjustment]] (stub)
- [[03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO]] (novo)
- [[03-subprojects/mobile/ui-catalog/assembly/ASM-AVAL-ELEICAO]] (novo)
- [[03-subprojects/web/ui-catalog/sindico/S32-score-governanca]] (novo)
- [[01-domain/invariants#INV-GOV-001]] (novo)
- [[01-domain/invariants#INV-GOV-002]] (novo)
- [[01-domain/invariants#INV-ASM-023]] (novo)

Spec Fase 12 **canônica**; implementação distribuída (score duplo M1; avaliação escondida M3 junto com Livekit; ValidationItem pattern M1).


---

## Fase 14 — IAM Cloudflare-style + provider migration (2026-04-23)

3 decisões cliente aplicadas com dual-check obrigatório:

| Decisão | Fecha | Impacto |
|---|---|---|
| [[STATE#D-116 — IAM Cloudflare-style (Member + Group + Membership + Role + PermissionGroup + Quota + Plan)\|D-116]] | Insight cliente "(group_role)_*_(role_type)" + role/group/quota/plan | Substitui personas pré-estabelecidas por modelo Cloudflare-style. 7 entidades (Member/Group/Membership/Role/PermissionGroup/Quota/Plan). Único role primitivo = `platform_admin`. Patterns 1+2+4+6 do benchmark IAM aplicados. ADR-0039 conditional accepted (3 artefatos pré-M1 pendentes). |
| [[STATE#D-117 — Cloudflare como edge primário (CDN/WAF/Pages/R2/Workers/Access/Tunnel) + Railway permanece origin\|D-117]] | Cliente quer "migrar tudo para Cloudflare" | Cloudflare = edge layer canônico (CDN/WAF/Pages/R2/Workers/Access/Tunnel/Email Routing inbound/Turnstile). Railway origin permanece (Backend Go + Postgres + Redis). ADR-0017 (AWS ECS M4+) mantida — Cloudflare é ortogonal. ADR-0040 accepted. |
| [[STATE#D-118 — SMS Twilio em prod, NoopSmsProvider em dev/staging, ship M1 com SMS desligado se Twilio não liberado\|D-118]] | Cliente: "em dev nao vamos usar SMS ate liberar Twilio" | Twilio prod / Noop dev. Ship M1 com `sms.enabled=false` em prod até cliente liberar Twilio BR + DPA. MFA TOTP-only inicialmente. ADR-0041 accepted. |

Artefatos criados nesta fase:
- [[02-architecture/adr/0039-iam-cloudflare-style-master-sindico]] (novo, conditional)
- [[02-architecture/adr/0040-cloudflare-edge-primary-railway-origin]] (novo)
- [[02-architecture/adr/0041-sms-twilio-prod-noop-dev]] (novo)
- [[11-audit/audit-log/2026-04-23-fase14-iam-cloudflare-sms]] (novo, com dual-check ADR-0039)
- [[02-architecture/adr/_moc]] (atualizado: 0037, 0038, 0039, 0040, 0041 + tabela síntese + decisões pendentes)

Dual-check ADR-0039 (Agent Opus contexto limpo): APROVADO COM AJUSTES — 3 críticos + 5 médios + 2 cosméticos, todos aplicados. Críticos: (1) Membership como entidade explícita resolvendo multi-Group por Member; (2) `Role.is_default` removido em favor de `Group.default_role_id` canônico; (3) status `accepted (conditional)` com 3 artefatos pré-M1 pendentes (catálogo PG seed YAML + migration idempotente + endpoints platform-side admin).

ADR-0040 e ADR-0041 sem dual-check separado nesta sessão (debt registrado para sessão futura).

Pré-requisitos pré-M1 (Sprint 10) explicitados em [[11-audit/audit-log/2026-04-23-fase14-iam-cloudflare-sms#Pré-requisitos pré-M1 (Sprint 10)]].


---

## Fase 14.1 — Refinamento IAM pós-análise GPT externo (2026-04-24)

Cliente João compartilhou 4 rodadas de ChatGPT externo sobre IAM. Cross-check contra benchmark F14.2 + ADR-0039 atual resultou em **6 refinamentos aplicados (R1-R6)** e **2 sugestões rejeitadas** (anti-patterns).

| Refinamento | Onde entrou no ADR-0039 | Fonte real |
|---|---|---|
| R1 Permission Boundary formal | §2 `Group.boundary_permission_group_ids` + §2.a dedicada + INV-IAM-005 | AWS Permissions Boundary + benchmark §1.7 |
| R2 Princípios fundamentais | §0 nova (default=zero, aditiva, deny>allow, least privilege, boundary, creator⊇target) | AWS Policy Evaluation Logic |
| R3 Regra da interseção | INV-IAM-006 em §2.5 + validação em §Painel admin | AWS iam:PassRole + Boundary |
| R4 Categoria `pg.iam.*` | §4 catálogo expandido (role.*, membership.*, group.edit_boundary, policy.dry_run, policy.simulate_as, audit.read) | AWS `iam:*` actions |
| R5 Quota refatorada | §5 completo (scope_type precedence + soft/hard + on_hard_exceeded) | AWS Service Quotas |
| R6 Anti-escalada | Tabela §Painel admin + audit reason `IAM_ESCALATION_ATTEMPT` | INV-IAM-006 combinada |

**Rejeitadas**: flag boolean `can_grant_permissions` (resolvida por R4); Policies JSON AWS-style M1 (Anti-pattern 7 — mantido vocabulário fechado).

**Debts novos**: D-IAM-USER-DIRECT-PG (M3+), D-IAM-BOUNDARY-UI (M2+).

**Conditional Acceptance**: 3 → 4 artefatos pré-M1 (adicionado boundary templates per group_kind).

Artefatos atualizados:
- [[02-architecture/adr/0039-iam-cloudflare-style-master-sindico]] (§0 novo; §2 boundary attr; §2.a dedicada; §2.5 INV-IAM-005/006; §4 categoria `pg.iam.*`; §5 quota refatorada; §Painel admin com tabela endpoints + PG requerido + regras validação; §Pontos abertos com D-IAM-USER-DIRECT-PG + D-IAM-BOUNDARY-UI; §Aplicação progressiva atualizada)
- [[STATE#D-116 refinamento — Análise GPT externo + R1-R6 aplicados (2026-04-24)|STATE §D-116 refinamento]]
- [[11-audit/audit-log/2026-04-24-iam-refinamento-pos-analise-gpt]] (análise completa)

Dual-check dos refinamentos R1-R6: opcional (polimento operacional, não arquitetural). Decisão: não bloquear; registrar como debt não-crítico.

---

## Sessão 2026-04-24 — Systematic review + consolidation

> Análise sistemática completa via 6 subagentes Opus em contextos limpos (dual-check natural). Decisões D-119..D-125 abertas em STATE. Agentes de execução escreveram 4 stubs `07-quality/`, 5 playbooks operacionais, 1 ADR (0042 search-provider), reescreveram `analise-crua.md`, e mapearam 23+ docs multi-tenant com 6 conflitos + 9 D-MT-### pendentes.

### Findings novos abertos (P0)

- **A-035** (P0 / domain) — `Unit` sem `unit_type` e `fracao_ideal`. Quórum por fração ideal (CC art. 1.352/1.353) impossível. Task: T-UNIT-01 (BE-001). Fix Sprint 10.
- **A-036** (P0 / arch) — Outbox + NATS declarados pillar M1 mas backend usa `InMemoryPublisher`. Eventos perdidos em crash pós-commit. Task: T-OUTBOX-01.
- **A-037** (P0 / arch) — `TenantBoundaryGuard` middleware (ADR-0034) ausente do código. Task: T-GUARD-01.
- **A-038** (P0 / sec) — `webhook_events` UNIQUE table não existe; idempotency só via HMAC + estabilidade `evt.ID`. Replay pós-crash gera dupla cobrança Stripe. Task: T-WEBHOOK-01.
- **A-039** (P0 / sec) — Anti-replay HMAC: backend 5min vs spec/pentest 2min (regressão pós A-DC-PEN-017). Task: T-WEBHOOK-02.
- **A-040** (P0 / sec) — Cookie session `Domain=.mastersindico.com.br` (cross-subdomain) vs spec `Domain=app.mastersindico.com.br` (sem ponto). Task: T-COOKIE-01.
- **A-041** (P0 / lgpd) — DPO não nomeado (LGPD Art. 41). Task: T-LGPD-M1-001.
- **A-042** (P0 / lgpd) — 9 DPAs pendentes (Mux, Zitadel, Twilio, Resend, Railway, Cloudflare, OpenSearch/Meili). Tasks: T-LGPD-M1-002..009.
- **A-043** (P0 / lgpd) — DT-010 IAuditLogger LGPD cross-BC incompleto (Billing/deal/empresa-sindica/membership só zerolog). Fere LGPD Art. 46. Task: T-AUDIT-01.
- **A-044** (P0 / sec) — MFA admin reclassificado para M1 (ADR-0026) mas SEM task em Sprint 10. Idem step-up MFA (IDN-036). Tasks: T-MFA-01, T-MFA-02.
- **A-045** (P0 / arch) — `ISearchProvider` real M1 (D-120 revoga D-067 PG tsvector); backend usa `ILIKE`. ADR-0042 proposed pending dual-check OpenSearch vs Meili. Task: T-SEARCH-01.

### Findings novos abertos (P1)

- **A-046** (P1 / domain) — BC `financial`/cobrança ausente; Assembly depende de inadimplência (Lei 4.591/64 art. 24 §3). Recomendação: criar BC thin OU explicitar ACL com Superlógica.
- **A-047** (P1 / domain) — BC `notifications` implícito; sem aggregate `Notification`/consent por canal LGPD.
- **A-048** (P1 / domain) — Conselho Fiscal e Subsíndico sem aggregate (P-BC-002 aberto).
- **A-049** (P1 / proc) — Velocidade decisional alta sem dual-check: 13 D-### na Fase 13/14 (24/04) sem registro em `dual-check-log/`. Apenas D-116 documenta. STEERING §32 violado. Task: T-DC-03 dual-check retroativo.
- **A-050** (P1 / multi-tenant) — 6 conflitos confirmados em multi-tenant — ver `[[audit-log/2026-04-24-multitenant-consolidation-map]]` para mapa completo. **D-MT-2 (kinds 4 vs 2 vs ADR-0039) é crítica antes de Sprint 10**.
- **A-051** (P1 / proc) — D-103 (governance score 1-10 + compliance 0-100) sem dual-check; 3 docs em conflito (INS-032, ASM-028, cross-domain.md Fase 12). Task: T-DC-02.
- **A-052** (P1 / spec) — BIL-035 ainda diz `live_admin_only` e cross-domain.md Fase 12 não menciona D-097. Task: T-SPEC-01.

### Resoluções 2026-04-24

- ✅ **A-FASE10-DEV-005** parcial mitigado por D-119 + sincronização CLAUDE.md/ROADMAP.md M1 = 2026-05-18 firme + criação de 27 tasks Sprint 10 explicitando escopo.
- ✅ Stubs `07-quality/CLEAN_CODE.md`, `CLEAN_ARCH.md`, `SOLID.md`, `CODE_CALISTHENICS.md` reescritos com 54 regras Go concretas + checklist PR + detecção CI. Endereçam AP-001..AP-025.
- ✅ 5 playbooks operacionais criados: `stripe-down.md`, `zitadel-down.md`, `mux-down.md`, `livekit-down-assembly.md`, `cross-tenant-leak.md`. Comandos executáveis (psql, livekit-cli, railway, curl), templates comms, hook LGPD T+72h ANPD.
- ✅ ADR-0042 (search provider) criado como `proposed (pending dual-check)`.
- ✅ Decisões D-058/D-060/D-063/D-064/D-066/D-075/D-076/D-079 marcadas REVOGADAS no corpo. D-006/D-011/D-067 marcadas SUPERADAS/PARCIAL-REVOGADA.
- ✅ DR drill pré-M1 agendado 2026-04-30 com log template em `audit-log/2026-04-30-dr-drill-pre-m1.md` — gate de aceite M1.
- ✅ Mapa multi-tenant consolidado em `audit-log/2026-04-24-multitenant-consolidation-map.md` (23+ docs, 6 conflitos, 9 D-MT-### propostas).
- ✅ Sprint 10 atualizado com janela `2026-04-22 → 2026-05-15` + 27 tasks novas (T-OUTBOX/WEBHOOK/UNIT/RLS/GUARD/MFA/SEARCH/MIGRATE/SPEC + T-DC-01..05 + T-MT-01..07 + T-QUALITY/PLAYBOOK/DR/LOAD/ANALISE/AUDIT + T-LGPD-M1-001..009).

### Bloqueadores M1 ainda em aberto (gate humano)

| ID | Owner | Decisão pendente |
|---|---|---|
| **D-MT-2** | João + arquiteto + product | Tenant kinds: 4 (ADR-0029) vs 2 (legado) vs ADR-0039 redefine — **crítico antes de Sprint 10** |
| **D-MT-8** | João + arquiteto | Tenant resolution flow oficial (header → JWT → path → membership) |
| **D-103** | João + product | Dual-check governance score 1-10 + compliance 0-100 (T-DC-02) |
| **ADR-0042** | João + arquiteto | OpenSearch vs Meilisearch (T-DC-01) |
| **D-094..D-118** | arquiteto | Dual-check retroativo 13 decisões Fase 13/14 (T-DC-03) |
| **D-107** | João | 9 bloqueadores LGPD com SLA (DPO + 8 DPAs + DPIA + política privacidade) |

### Métricas de cobertura

- Decisões registradas: D-001..D-125 (125 decisões)
- Findings históricos: A-001..A-052 (+ AUDIT existente A-FASE8/9/10)
- Multi-tenant docs: 23+ vivos
- Tasks Sprint 10 (pós-update): ~50 itens (vs 14 originais)
- Coverage Go (snapshot 2026-04-22): domain ≥95%, application ≥90%, global ≥85%

### Vizinhos

- [[audit-log/2026-04-24-multitenant-consolidation-map]]
- [[audit-log/2026-04-30-dr-drill-pre-m1]]
- [[../STATE#D-119]]
- [[../05-tasks/by-sprint/sprint-10]]
- [[../02-architecture/adr/0042-search-provider]]

---

## Sessão 2026-04-27 — Consistency Pass (38 patches em 30 arquivos)

> Sessão autônoma a pedido do João. Cobertura: 7 buckets de decisões revogadas (D-094 Morador Base, D-097 Lives, D-099 vínculos, D-103 score, D-120 tsvector, D-134 admin, D-135 IP-block) + dupla checagem 4 rounds. **38 patches finais aplicados em 30 arquivos distintos.**

### Achados resolvidos

- **A-052** (BIL-035 stale + cross-domain D-097) → 🟢 totalmente resolvido (BIL-035 EARS reescrito, INV-BIL-015 atualizado, FR-BE-BIL-025 expandido)
- **DT-012** vocabulary cleanup → 🟡 PROGRESSO MAIOR (de ~15 stale + 7 buckets para ~10 menções residuais N2+/N3+/Morador Pagante apenas)
- **BR-063** antifraude (regra "Bloqueio múltiplos cadastros mesmo IP < 1h" stale) → 🟢 reescrita conforme D-135 (device fingerprint progressivo) com link para IDN-026

### Achados mantidos abertos

- **A-077** admin-privilegios.md (54KB) reescrita completa pendente — mantido (banner suficiente; reescrita = sprint dedicado)
- **A-FASE10-DEV-005** Web M1 não iniciado (🔴 bloqueador) — mantido sem mudança nesta sessão
- Stripe leak no domain backend (BLOCKER de migração Mercado Pago) — auditado mas NÃO corrigido nesta sessão (refactor de código, não documentação). João decide se abre D-### nova de migração.

### Métricas

- **Patches**: 38 em 30 arquivos
- **Decisões fechadas/atualizadas**: 7 buckets cobertos (D-094, D-097, D-099, D-103, D-120, D-134, D-135)
- **Subagentes Opus paralelos**: 4 (chaos audit + 3 mapeamentos D-### stale)
- **Rounds dupla checagem**: 4 (subagentes + 3 grep manuais)
- **Stale capturadas: round 1=17, round 2=21, round 3=14, round 4=0** ✅
- **Chaos pasta**: confirmado independentemente que `_archive/2026-04-24-chaos-processed/` está 100% canonizado (Mercado Pago era histórico, decisão atual é nova)

### Vizinhos

- [[audit-log/2026-04-27-consistency-pass]] — log detalhado com 38 diffs catalogados
- [[../STATE#DT-012]] — débito atualizado
- [[../01-domain/business-rules#BR-063]] — regra reescrita

---

## Sessão 2026-04-25 — Consolidação `_chaos/` 8 fases (fechamento)

> Sessão iniciada 2026-04-24 (analise sistemática + dual-check + Fases A-B) e concluída 2026-04-25 (Fases C-H). 61 arquivos absorvidos do `_chaos/` em overlay canônico. Total: 12 decisões D-126..D-137 + 9 ajustes do dual-check + 4 agentes Opus em paralelo.

### Findings novos (resolvidos nesta sessão)

- **A-053** ✅ (P0/structural) — `_chaos/` 61 arquivos não-canonizados → todos absorvidos em 8 fases (A-H). `_chaos/` agora vazio. Fechado via D-137.
- **A-054** ✅ (P0/multi-tenant) — Tenant kinds 4 (ADR-0029) vs 2 (cliente) divergência → ADR-0029 atualizada 4→2 kinds. Fechado via D-126.
- **A-055** ✅ (P0/identity) — D-058 referenciada erroneamente em 5+ arquivos como "admin role transversal" (D-058 real é Connect Me Morador) → corrigido em web/README.md, architecture.md, security.md, admin-privilegios.md (banner superseded), ui-catalog.md macro. Fechado via D-134 com correção crítica de citação.
- **A-056** ✅ (P0/proc) — 13 decisões Fase 13/14 (D-094..D-118) sem dual-check registrado (STEERING §32 violado) → dual-check formal das 10 decisões estruturais consolidadas (D-126..D-135) executado 2026-04-25; 7 aprovadas + 3 com ajustes incorporados. Pendente dual-check retroativo D-094..D-118 (T-DC-03 Sprint 10).
- **A-057** ✅ (P0/quality) — `07-quality/{CLEAN_CODE,CLEAN_ARCH,SOLID,CODE_CALISTHENICS}.md` eram stubs → reescritos com 54 regras Go concretas (Fase pré-consolidação). Confirmado nesta sessão.
- **A-058** 🟡 (P1/quality) — 5 playbooks operacionais (`stripe-down,zitadel-down,mux-down,livekit-down-assembly,cross-tenant-leak`) inexistentes → criados (Fase pré-consolidação).
- **A-059** ✅ (P0/domain) — Adjustment aggregate em `commercial` BC vs cross-BC reality → movido pra `cross-domain` BC + adicionado enum target_type + INV-ADJ-006 + LGPD interaction. Fechado via D-131.
- **A-060** ✅ (P0/domain) — Vote como aggregate root vs entity (DDD purista) → rebatizado entity de AgendaItem mantendo IVoteRepository. Fechado via D-130.
- **A-061** ✅ (P0/domain) — Mandate aggregate referenciado mas não existia → regularizado como flags em Membership. Fechado via D-129.
- **A-062** ✅ (P0/domain) — Block como `Unit.block *string` insuficiente vs cliente blocos polimórficos → Block aggregate root opcional (Unit FK). Fechado via D-128.
- **A-063** ✅ (P0/assembly) — Administradora persona ambígua (sub-tipo Empresa? actor delegado?) → Entity de Assembly `AssemblyAdministratorLink` + auto-invalida em End(). Fechado via D-127.
- **A-064** ✅ (P0/assembly) — Auditor de Assembly persona vs role efêmera → role temporária ABAC via Membership efêmera. Fechado via D-132.
- **A-065** ✅ (P0/conceptual) — Sub-produto 5 "Assembleias Live" confundia Lives broadcast vs Assembly conferência → renomeado "Assembleias & Votações"; Lives broadcast novo aggregate `BroadcastLive` no `content` BC + ADR-0044 broadcast provider. Fechado via D-133.
- **A-066** ✅ (P0/UX) — Admin role transversal embutida em cms (D-072) inviável com Cloudflare Access → revogado D-072; criada app dedicada `apps/admin` (D-117 viabiliza SSO MFA + D-113 benchmark big tech). Fechado via D-134.
- **A-067** ✅ (P0/security/UX) — Bloqueio múltiplas contas IP (Req 1 AC11) quebraria famílias condomínio → rejeitado; adotado device fingerprint progressivo (rate-limit progressivo + Turnstile). Fechado via D-135.
- **A-068** ✅ (P1/proc) — "Morador Pagante" persistiu como label vivo apesar de D-067/D-096 abolirem slugs → D-136 oficializa abolição como label. Fechado.
- **A-069** ✅ (P1/audit) — `_chaos/` em estado caótico sem inventário → archive `_archive/2026-04-24-chaos-processed/` com 7 sub-pastas + _README.md + 16 PDFs em `60-sources/`. Fechado.
- **A-070** ✅ (P1/req-coverage) — `04-requirements/functional/*` cobertura de Reqs cliente desconhecida → Fase C cruzou requirements(6).md (206KB) + mastersindico-requirements.pdf (4256l) → 20 Reqs novos absorvidos + 15 enriquecidos + ~70 confirmados no-op + ~140 telas delegadas Fase D. Total ~301 Reqs canônicos pós-absorção (de ~260). Fechado.
- **A-071** ✅ (P1/ui) — 27 MDs feature-specs do cliente em `_chaos/` não absorvidos → Fase B 21 specs + 1 pattern + 1 delta merge + 1 monolito 398KB dividido + 1 referência externa. Fechado.
- **A-072** ✅ (P1/ui) — 9 PDFs jornadas+módulos não destilados pra frontend → Fase D 8 arquivos `ui-catalog/jornadas/` (sindico/morador/empresa/agencia-marketing/parceiro-vizinhanca/onboarding/curriculo-cadastro/curriculo-empresa-view). Fechado.
- **A-073** ✅ (P1/assembly) — 3 PDFs Assembleia (1281+289+661l) não destilados → Fase E 7 telas frontend + 11 Reqs novos ASM-034..044 + 3 aggregates patched. Fechado.
- **A-074** ✅ (P1/billing) — 2 Matrizes funcionais cliente com N1/N2/N3 stale → Fase F absorvida com 22+ traduções + Matriz Trial detalhada nova + 23 deltas. Fechado.

### Findings ainda 🟡 abertos (registrados como débitos)

- **DT-012** 🟡 — Débito vocabulário N2+/N3+/Morador Pagante em ~15 arquivos não-críticos (descritivos/secundários). Críticos corrigidos. Resolução rolling M2.
- **A-075** 🟡 (P1/lgpd) — D-107 6 bloqueadores LGPD M1 (DPO + 5 DPAs Mux/Zitadel/Twilio/Resend/Railway/Cloudflare/OpenSearch) ainda transferidos ao cliente sem owner formal. Não-bloqueador estrutural.
- **A-076** 🟡 (P1/proc) — Dual-check retroativo D-094..D-118 (13 decisões Fase 13/14 sem registro formal). Task Sprint 10.
- **A-077** 🟡 (P1/admin) — `admin-privilegios.md` (54KB) tem banner "superseded em parte" mas **reescrita completa** (apps/admin dedicada D-134) ainda pendente. Risco baixo (banner suficientemente claro).
- **A-078** 🟢 (P2/ui) — Sub-roles em rotas internas dos produtos (administradora condominial D-114, agência marketing D-102, empresa-síndica D-073, auditor assembly D-132) — telas + ABAC ainda a especificar. Fase futura.
- **A-079** 🟢 (P2/ops) — ADR-0044 broadcast provider (Cloudflare Stream vs Mux Live) — dual-check pendente. M2.
- **A-080** 🟢 (P2/security) — Pendência 1.1 BeyondCorp 4-fases roadmap a destilar de `ARCHITECTURE_ANALYSIS_REPORT.md`. M2.
- **A-081** 🟢 (P2/perf) — Pendência 1.2 retry budget + jitter pattern a destilar. M2.
- **A-082** 🟢 (P2/quality) — Pendência 1.3 PBT custom shrinkers + PROPERTY_TEST_SEED. M2-M3.

### Resoluções dos findings da sessão 2026-04-24 (intermediários)

- A-035 (P0 / Unit `unit_type` + `fracao_ideal`) → 🟢 **foundation entregue 2026-04-27 (Agente 2)**: VOs `unit_type.go`+`fracao_ideal.go`+tests, migration `00211_add_unit_type_and_fracao_ideal.sql` com trigger Σ ≤ 100 por condomínio (ERRCODE 23514 → 422 fraction_sum_exceeded), entity `Unit` extendida (NewUnitWithDetails, ReconstructUnitFull, ChangeUnitType, SetFracaoIdeal, HasFracaoIdeal, Deactivate) preservando backward-compat para 4 call sites. `go build/vet` ✅, 17 tests novos ✅. **Pendente mecânico** (~1-2h): sqlc generate + mapper + use case + handler. Detalhes [[11-audit/audit-log/2026-04-27-agente-2-systematic-pass]] §Lote 5.
- A-036 (P0 / outbox + NATS declarados mas não existem) → ainda 🟡 em backlog Sprint 10 (T-OUTBOX-01).
- A-037 (P0 / TenantBoundaryGuard middleware) → ainda 🟡 em backlog Sprint 10 (T-GUARD-01).
- A-038 (P0 / webhook_events UNIQUE table) → ainda 🟡 em backlog Sprint 10 (T-WEBHOOK-01).
- A-039 (P0 / anti-replay HMAC 5min vs 2min) → ainda 🟡 em backlog Sprint 10 (T-WEBHOOK-02).
- A-040 (P0 / Cookie session domain) → ainda 🟡 em backlog Sprint 10 (T-COOKIE-01).
- **A-FASE10-DEV-005** (Web M1 não iniciado, 🔴) → 🟡 **mitigação parcial 2026-04-27 (Agente 2)**: 11 quick wins web aplicados, `bun run check` 0/0/1 (zero errors/warnings), apps/admin scaffolded port 3010 (D-134), 81 tests verdes. Telas funcionais ainda a implementar (auth completo, M1-M15 morador, S3-S6 + S11-S14 síndico, AD1-AD2). Detalhes [[11-audit/audit-log/2026-04-27-agente-2-systematic-pass]] §Lotes 1-2-4.
- **A-FASE10-DEV-003** (Mobile D-048/049/050) → 🟢 **D-048 + D-049 deps aplicadas 2026-04-27 (Agente 2)** em pubspec.yaml; `flutter analyze` zero issues; 27 tests passed. Adoção incremental (refactor entities/blocs para `@freezed` + transformers `concurrent`/`droppable`) por feature. **D-050** (jailbreak/root) ainda não aplicada — backlog (não bloqueia M1).
- A-041 (P0 / DPO não nomeado) → 🟡 em D-107 cliente.
- A-042 (P0 / 9 DPAs LGPD) → 🟡 em D-107 cliente.
- A-043 (P0 / DT-010 IAuditLogger) → ainda 🟡 em backlog Sprint 10 (T-AUDIT-01).
- A-044 (P0 / MFA admin sem task Sprint 10) → tasks T-MFA-01/02 criadas.
- A-045 (P0 / ISearchProvider real M1) → ADR-0042 proposed; Sprint 10 task T-SEARCH-01.
- A-046 (P1 / BC `financial`/cobrança ausente) → ainda aberto (backlog M2+).
- A-047 (P1 / BC `notifications` implícito) → ainda aberto (backlog M2+).
- A-048 (P1 / Conselho Fiscal/Subsíndico aggregate) → ainda aberto (backlog M2+).
- A-049 (P1 / 13 D-### Fase 13/14 sem dual-check) → renumerado A-076 acima.
- A-050 (P1 / 6 conflitos multi-tenant) → resolvido via D-126/127/128/129 + ADR-0029 update.
- A-051 (P1 / D-103 sem dual-check) → resolvido via dual-check formal Fase H 2026-04-25.
- A-052 (P1 / BIL-035 stale + cross-domain Fase 12 D-097) → patches feitos em Fase F + **revalidados e completados em Consistency Pass 2026-04-27** (FR-BE-BIL-025 + INV-BIL-015 + EARS reescrita matriz expandida admin/Empresa Pro/agência delegada). 🟢 **Resolvido**.

### Métricas finais sessão 2026-04-25

- Decisões registradas: **12** (D-126..D-137)
- ADRs: 1 atualizada (0029) + 1 criada (0044 proposed)
- Aggregates: **6 afetados** (Block novo, AssemblyAdministratorLink novo entity, Vote rebatizado entity, Adjustment movido cross-domain, Unit FK Block, Membership flags mandato)
- Reqs canônicos: **+20 novos + 15 enriquecidos + 11 ASM novos + 5 COM novos + 4 INS novos + 3 CNT novos + 3 XD novos**
- Telas frontend ui-catalog: **27 + 8 jornadas + 9 assembly = 44+ specs novas**
- PDFs preservados em `60-sources/`: **16**
- Findings A-053..A-082: **30 resolvidos/registrados** (14 fechados na sessão)
- Débitos: **DT-012** (vocabulary cleanup rolling)
- `_chaos/` final: **0 arquivos** ✅
- 4 subagentes Opus paralelos (D, E, F + dual-check)
- ETA real: ~6h elapsed (paralelismo aplicado)

### Vizinhos

- [[STATE#D-126]] .. [[STATE#D-137]] — decisões da sessão
- [[_archive/2026-04-24-chaos-processed/_README]] — inventário 8 fases
- [[60-sources/master-sindico-research/client-material/_pendencias-fase-h]] — patterns a destilar
