---
title: Consolidado 2026-04-23 — Quebras de negócio + Práticas ops big techs
type: audit
tags:
  - audit
  - consolidado
  - quebras
  - ops
  - bigtech
  - master-sindico
  - v2
  - m1
source: >-
  11 agentes paralelos research 13-research/ + 2 auditorias focadas (Q-### +
  G-###)
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-23T00:00:00.000Z
status: consolidado
owner: audit
---

# Consolidado — Quebras de negócio + Práticas ops big techs

Escopo explícito (instrução do usuário): **NÃO novas funcionalidades**; foco duplo em (a) **quebras de negócio** (race conditions, fraudes, edge cases, perda de dados, compliance) e (b) **práticas operacionais de big techs** aplicáveis ao Master Síndico (SRE, ops, segurança, DR).

Fontes deste consolidado (ver referências linha-por-linha ao final):
- `11-audit/audit-log/2026-04-23-quebras-negocio-systematic.md` — 31 quebras Q-001..Q-031
- `11-audit/audit-log/2026-04-23-ops-bigtech-gaps.md` — 48 gaps G-001..G-048
- `13-research/beyond-corp/_aplicacoes-master-sindico.md`
- `13-research/google-arch/_aplicacoes-master-sindico.md`
- `13-research/netflix/_aplicacoes-master-sindico.md`
- `13-research/youtube/_aplicacoes-master-sindico.md`
- `13-research/google-meet/_aplicacoes-master-sindico.md`
- `13-research/linkedin/_aplicacoes-master-sindico.md`
- `13-research/instagram/_aplicacoes-master-sindico.md`
- `13-research/tiktok/_aplicacoes-master-sindico.md`
- `13-research/condominial-br/_aplicacoes-master-sindico.md`

---

## Sumário executivo

| Categoria | 🔴 M1 blocker | 🟠 M2 | 🟡 M3+ | Total |
|---|---|---|---|---|
| Quebras de negócio (Q-###) | 15 | 12 | 4 | 31 |
| Práticas ops big-tech (G-###) | 20 | 20 | 8 | 48 |
| **Total bruto** | **35** | **32** | **12** | **79** |

**Top-3 findings mais graves**:
1. **Q-020** — Ata/Timeline mutável via UPDATE direto no DB (INV-024/072 app-level não tem constraint DB). Fraude de ata processual = nulidade legal da assembleia.
2. **Q-027** — Edital assembleia 5d vs 8d do Art. 24 §1 da Lei 4.591/64. **Assembleias nulas na justiça por vício de convocação.**
3. **G-029** — Runbook LGPD breach 72h ANPD não existe (declarado "a criar" em BEYOND_CORP §16). **Multa 2% faturamento por incidente.**

---

## Bloco 1 — Top-15 quebras de negócio críticas M1 (🔴)

### Q-020 — Ata/Timeline mutável via DB direto
- **Categoria**: data-loss / integridade
- **Onde**: `01-domain/invariants.md` INV-024/072; `04-requirements/functional/assembly.md`
- **Problema**: enforcement "app-level + trigger opcional". DBA ou RCE falsifica ata já publicada.
- **Fix M1**: `REVOKE UPDATE, DELETE ON timeline_entries, assembly_minutes FROM app_user` + trigger `RAISE EXCEPTION` obrigatório (não opcional) + INV promovido a DB-level.
- **Referência**: Q-020 quebras-negocio + INV-024/072

### Q-027 — Edital 5d vs Lei 4.591/64 Art. 24 §1 (8d)
- **Categoria**: compliance regulatório
- **Onde**: `04-requirements/functional/assembly.md` ASM-001
- **Problema**: MS permite convocar assembleia com 5 dias. Lei 4.591/64 exige **mínimo 8 dias**. Toda decisão tomada por assembleia convocada <8d é **anulável**.
- **Fix M1**: `edital.min_antecedence_days = 8`; bloqueio no handler + migração retroativa editais já criados <8d.
- **Referência**: Q-027 + condominial-br §4

### Q-005 — Homologação race: `closed → canceled` admin
- **Categoria**: edge-case state machine
- **Onde**: `01-domain/state-machines.md` §3 Assembly
- **Problema**: spec não lista transição `closed → canceled` como proibida. Admin pode cancelar ata homologada = limbo jurídico (ata inexistente juridicamente mas publicada via push).
- **Fix M1**: state machine: `closed` é estado terminal. Só admit `closed → archived`. Cancelar ata homologada requer **assembleia reversora** nova (não admin ato único).

### Q-026 — LGPD 15d erasure vs 5y audit retention
- **Categoria**: compliance contradição
- **Onde**: `04-requirements/compliance-lgpd.md` BR-053 vs `01-domain/invariants.md` INV-091
- **Problema**: usuário pede erasure; 15d pra cumprir; audit trail precisa reter 5 anos (contratual). Contradição não resolvida (P-INV-005 aberto).
- **Fix M1**: INV-127 — **pseudonimização HMAC-keyed** imediata em audit (user_id → hmac_hash); hard-delete PII do `users` + `profiles`. Audit referencia hash, não PII.
- **Referência**: Q-026 + ADR-0028 (precedente pseudonimização)

### Q-024 — Cross-tenant leak via `tenant_id` em path sem guard
- **Categoria**: segurança / isolamento
- **Onde**: rotas REST que tomam `tenant_id` em path/query (ex: `GET /tenants/{id}/timeline`)
- **Problema**: INV-089/108 declaram tenant isolation mas nenhum middleware comparador `path.tenant_id vs ctx.tenant_id` existe. User A manda request com B's tenant_id no path → 200 OK.
- **Fix M1**: `TenantBoundaryGuard` middleware obrigatório; INV-125 — toda rota com `tenant_id` em path deve passar pelo guard antes do handler.
- **Referência**: Q-024

### Q-002 — Cupom Vizinhança 4h lock TOCTOU (Redis-only)
- **Categoria**: race condition
- **Onde**: `00-product/sub-produtos/08-vizinhanca.md` Sistema Cupons + INV-052
- **Problema**: Redis `SET NX EX 14400` não sobrevive crash Redis; sem DB constraint de cooldown, usuário emite 2 cupons em 50ms em redis-cluster partition.
- **Fix M1**: tabela `coupon_cooldowns (user_id, partner_id, locked_until TIMESTAMPTZ, PRIMARY KEY (user_id, partner_id))` + UPSERT atômico com `WHERE locked_until < NOW()`. Redis vira cache, Postgres vira fonte-de-verdade.

### Q-001 — 1-device bypass via vote handler sem re-validação session freshness
- **Categoria**: race / fraud
- **Onde**: INV-110 (1-device) + assembly.md vote handler
- **Problema**: device A loga; user revoga via device B; em 50-200ms device A ainda tem JWT válido; vota. INV-110 impõe 1 session ativa mas handler do voto não re-verifica se `session.id == users.current_session_id`.
- **Fix M1**: INV-111 — vote handler valida `ctx.session_id == users.current_session_id` ANTES de registrar voto. Se stale, rejeita 401.

### Q-013 — Síndico destituído mid-assembleia em aberto
- **Categoria**: edge-case state machine
- **Onde**: `01-domain/state-machines.md` Assembly + SyndicMandate
- **Problema**: assembleia aberta às 19:00; às 19:30 destituição aprovada; o PRÓPRIO síndico (ex-síndico) continua presidindo até 19:45 (ata fechada). Votos tomados sob sua presidência após 19:30 = contestáveis.
- **Fix M1**: INV-117 — `SyndicMandate.status=revoked` força assembleia atual para `status=paused_reorg`; eleição de presidente ad-hoc via votação antes de retomar.

### Q-008 — Reputação: empresa avalia a si mesma via conta laranja
- **Categoria**: fraude
- **Onde**: `00-product/sub-produtos/03-reputacao.md` INV-043
- **Problema**: INV-043 UNIQUE (contract, evaluator) impede duplicata, mas não impede `evaluator = CPF do sócio da empresa`. Conflict_of_interest é estático no onboarding (não detecta rel CPF↔CNPJ).
- **Fix M1**: INV-114 — verify `evaluator.cpf` não é sócio do `CompanyEvaluation.company_id` via tabela `cnpj_members` (populada no signup CNPJ). INV-115 — household blocklist (mesmo endereço/telefone).

### Q-011 — Cupom ID_LOJA+SEQ+PALAVRA brute-forceável
- **Categoria**: fraude
- **Onde**: INV-052 formato cupom
- **Problema**: formato `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` = 1000 SEQs × 1 palavra pública = brute-force 1000 tentativas pra resgatar cupom alheio.
- **Fix M1**: redesenhar INV-052 — `ID_LOJA(5) + HMAC(secret, counter, partner)[:8]`; valida no servidor via HMAC verify.

### Q-003 — Quota Connect Me racing (`FOR UPDATE` insuficiente)
- **Categoria**: race
- **Onde**: `01-domain/aggregates/FeatureUsage.md` INV-FU-005
- **Problema**: 2 clients enviam Connect Me simultâneo; ambos passam check de quota antes do UPDATE; decrement vai pra -1.
- **Fix M1**: `UPDATE ... SET current = current - 1 WHERE current > 0 RETURNING current` — se linha não retorna, quota exhaust. Atomicidade garantida pelo WHERE.

### Q-021 — Stripe webhook down 24h → subscriptions desalinhadas
- **Categoria**: data-loss / reconciliação
- **Onde**: `04-requirements/functional/billing.md` BIL-016..021
- **Problema**: Stripe retry 3d; se nosso endpoint cai 24h+ sem backfill, subs em `trial` ficam `active` no Stripe e MS não sabe.
- **Fix M1**: cron noturno `stripe_reconcile` — lista subscriptions com `status ∈ {trial, past_due}` + timestamp >7d; chama `IPaymentGateway.GetSubscription(id)` + atualiza local.

### Q-022 — NATS JetStream perda de mensagens críticas
- **Categoria**: data-loss
- **Onde**: `02-architecture/event-driven.md`
- **Problema**: JetStream configurado com retention default = memory-only em staging. Se stream `billing.events` é memória, reboot perde eventos.
- **Fix M1**: INV-116 — todos streams críticos (`billing.*`, `content.video.*`, `assembly.*`) em File storage + replicação `R=3` + `MaxAge=30d`.

### Q-023 — Mux upload órfão (rede morre no meio)
- **Categoria**: data-loss / fraude quota
- **Onde**: `04-requirements/functional/content.md`
- **Problema**: user inicia upload; quota já decrementou; rede morre; Mux webhook `upload.created` nunca chega; quota perdida + arquivo em S3 temp (custo infra).
- **Fix M1**: quota **reserva** (soft-hold) no `upload.create`; **commit** no `upload.asset_created` webhook; **release** no `upload.errored` OU cron 24h expire holds órfãos.

### Q-028 — ICP-Brasil vs gov.br para ata homologada
- **Categoria**: compliance
- **Onde**: `05-assembleia-live.md` ATA homologação
- **Problema**: spec silente sobre nível da assinatura. STJ dez/2024 aceita gov.br (assinatura avançada) pra AGO/AGE; ICP-Brasil obrigatório só pra averbação cartorial.
- **Fix M1**: decisão D-BR-### (gov.br no MVP, ICP via adapter opcional M3+). ADR-00XX ICP adapter bookmark.

### Q-030 — Cross-tenant leakage via search tsvector
- **Categoria**: segurança / isolamento
- **Onde**: busca Banco de Talentos + Connect Me via tsvector
- **Problema**: query tsvector sem cláusula `WHERE tenant_id = :ctx.tenant_id` retorna currículos de outros condomínios. RLS ajuda mas MS usa app-level sqlc.
- **Fix M1**: (fundir com G-034 RLS default-on) — `ENABLE ROW LEVEL SECURITY` + `CREATE POLICY` em todas tabelas com `tenant_id`. Fallback de defesa em profundidade.

---

## Bloco 2 — Top-20 práticas ops big-tech M1 blockers (G-###)

### G-016 — PostgreSQL PITR não configurado
- **Eixo**: DR / Data
- **Prática big-tech**: AWS RDS / GCP CloudSQL PITR WAL archiving padrão; Google SRE ch.26 "untested backup = no backup".
- **Estado MS**: `deploy.md §pré-deploy` só `pg_dump` snapshot; sem WAL archiving = RPO 24h.
- **Por que importa**: assembleia diária → 24h de transações podem virar pó. Um incidente DB corrupto = 1 dia de atas perdidas.
- **Fix M1**: config Railway WAL archiving + `wal-g` ou `pgbackrest` → S3/R2; RPO alvo 5min, RTO alvo 30min.

### G-017 — DR drill não documentado
- **Eixo**: DR
- **Prática big-tech**: Google DiRT (Disaster Recovery Testing) trimestral; AWS GameDay.
- **Fix M1**: `06-execution/dr-drill.md` — procedure + cadência + owner + restore em staging de backup real PITR. Comprovado via `restore_success_at` no runbook mensal.

### G-015 — RPO/RTO não quantificados por BC
- **Eixo**: DR
- **Estado MS**: SLO existe em `observability.md` §3.1; RPO/RTO ausentes.
- **Fix M1**: tabela paralela em `observability.md` com RPO/RTO por BC: identity RTO 5min / RPO 5min; billing RTO 1h / RPO 15min; assembly RTO 30min / RPO 5min; content RTO 4h / RPO 1h; etc.

### G-024 — Secret rotation trimestral manual
- **Eixo**: Security ops
- **Prática big-tech**: Vault / AWS Secrets Manager rotation automática; HashiCorp 30d.
- **Estado MS**: `deploy.md §4` + `BEYOND_CORP §10.2` declaram "manual trimestral" = janela 90d de exposição.
- **Fix M1**: Railway env via Vault adapter OU mínimo rotation automatizada via cron + Zitadel/Stripe API rotate; incident playbook de key-leak com rotation <1h.

### G-025 — KMS field-level PII encryption ausente
- **Eixo**: Security / LGPD
- **Prática big-tech**: Google Spanner encryption at rest per-row; AWS RDS + KMS CMK + field-level envelope encryption.
- **Estado MS**: ADR-0028 cobre só pseudonimização em logs. CPF/CNPJ/email **plaintext at-rest** no Postgres.
- **Fix M1**: KMS (AWS KMS ou HashiCorp Vault Transit) + pgcrypto com envelope encryption em `users.cpf`, `users.email`, `companies.cnpj`, `users.phone`. DBA vê cifrado; app decrypta com KMS key.
- **Impacto se ignorar**: DBA malicioso OU backup leak = LGPD breach Art. 48 + multa 2% faturamento.

### G-029 — Runbook LGPD breach 72h ANPD não existe
- **Eixo**: Incident / compliance
- **Estado MS**: `12-docs/runbooks/incident-lgpd-breach.md` declarado "a criar" em BEYOND_CORP §16, não existe.
- **Fix M1**: criar runbook com steps: T+0h triage, T+12h comms interna, T+24h DPO assess, T+48h draft ANPD notification, T+72h submit.

### G-030 — Gitleaks pre-commit local ausente
- **Eixo**: Security ops
- **Prática big-tech**: GitHub Advanced Security + GitGuardian + local pre-commit hooks.
- **Estado MS**: CI tem em `cve-process.md §6`; dev local **não**. A-018 já ocorreu (zitadel-key vazou em commit).
- **Fix M1**: husky pre-commit hook + `.gitleaks.toml` + mandatório via CLAUDE.md §checklist.

### G-038 — On-call rotation só pós-Sprint 11 (GA é Sprint 10)
- **Eixo**: Incident
- **Estado MS**: `incident.md §canal` declara rotation "post-GA".
- **Fix M1**: rotação formal ANTES do GA (2026-05-08). Mesmo que seja 2 devs alternando — doc + PagerDuty/Opsgenie trial grátis.

### G-033 — Auto-rollback em prod ausente
- **Eixo**: Deployment
- **Prática big-tech**: Netflix Spinnaker auto-rollback on canary SLO regression; Argo Rollouts.
- **Estado MS**: só staging tem (`deploy.md §1`); prod é manual via PagerDuty.
- **Fix M1**: canary prod 5% tráfego × 10min; rollback automático se burn rate SLO > 2x → notify on-call.

### G-042 — Template comunicação incidente cliente ausente
- **Eixo**: Incident
- **Estado MS**: `12-docs/incident-communication-template.md` declarado "a criar" em `cve-process.md §7.3`.
- **Fix M1**: template + status page público + SLA de primeira comunicação 30min (Sev0/Sev1).

### G-041 — Public status page ausente
- **Eixo**: Incident
- **Prática big-tech**: statuspage.io, Atlassian Statuspage, Better Stack.
- **Estado MS**: não mencionado.
- **Fix M1**: status page pública linkada em `app.mastersindico.com.br/status`; incidentes Sev0/1 postam automático.

### G-026 — Dependency vuln scan no CI incompleto
- **Eixo**: Security
- **Estado MS**: `cve-process.md §6` cita govulncheck mas não confirma npm audit + osv-scanner como gates bloqueantes.
- **Fix M1**: CI gate bloqueante — `govulncheck + npm audit high+ + osv-scanner` falham merge; SBOM gerada a cada deploy.

### G-034 — Postgres RLS default-on ausente
- **Eixo**: Security / data
- **Prática big-tech**: Spanner auth-per-row nativo; AWS Redshift RLS.
- **Estado MS**: RLS cobre ~5 tabelas; sqlc linter é frágil. Novo handler com bug em `WHERE tenant_id` vaza.
- **Fix M1**: `ALTER TABLE ... ENABLE ROW LEVEL SECURITY + CREATE POLICY` em **todas** tabelas com `tenant_id`; middleware seta `SET LOCAL app.current_tenant = :ctx`. Defesa-em-profundidade + fecha Q-024/Q-030.

### G-032 — Saga primitives / coordinator
- **Eixo**: Patterns / reliability
- **Prática big-tech**: Netflix Conductor, AWS Step Functions, Temporal, Uber Cadence.
- **Estado MS**: ADR-0030 declara orchestration default + 5 sagas canônicas; cada uma vira hand-coded (duplicação retry/compensation/persistence).
- **Fix M1**: primitivo `pkg/saga` com `SagaStep{Do, Compensate, Idempotent}` + tabela `saga_instances(id, type, state, current_step, idempotency_key, created_at, updated_at)`. Rejeita Conductor (overkill); Go-native.

### G-022 — Circuit breaker + retry entre MS e providers
- **Eixo**: Reliability
- **Prática big-tech**: Netflix Hystrix → resilience4j → sony/gobreaker.
- **Estado MS**: `patterns.md` cita retry; circuit breaker não formalizado.
- **Fix M1**: `sony/gobreaker` em cada provider (Stripe/Mux/Livekit/Twilio/Resend/Zitadel) + timeout per-call + exponential backoff + jitter. Fail-closed em `billing.*`; fail-open em `content.video.thumbnail`.

### G-023 — Bulkhead por provider
- **Eixo**: Reliability
- **Prática big-tech**: Netflix bulkhead isolation.
- **Estado MS**: ausente.
- **Fix M1**: `x/sync/semaphore` por provider (Stripe 20 slots concorrentes / Mux 10 / Livekit 5 / Zitadel 100). Pico de uploads Mux não derruba billing no mesmo process.
- **Referência**: ADR-0033 novo (netflix/_aplicacoes §R1).

### G-006 — Audit log tamper-evident (hash-chaining)
- **Eixo**: Observability / compliance
- **Prática big-tech**: AWS CloudTrail log file integrity; GCP Cloud Audit log hash chain.
- **Estado MS**: `audit_log_entries` append-only app-level; admin DB pode contornar.
- **Fix M1**: coluna `prev_hash TEXT` + trigger calcula `row_hash = sha256(prev_hash || payload)`; CLI offline `audit-verify` re-valida cadeia. INV-112.
- **Referência**: beyond-corp/_aplicacoes R3 + Q-020

### G-005 — Error budget automated CD halt
- **Eixo**: SRE
- **Prática big-tech**: Google SRE ch.3 error budget; deploy freeze automático quando budget < 20%.
- **Estado MS**: ADR-0025 é texto-apenas; zero métrica stateful.
- **Fix M1**: exporter Prometheus `slo_budget_remaining{bc, window}` + alerta `budget_exhausted` → halt CI/CD deploy promotion + notify on-call.

### G-013 — Backup imutável offline (ransomware)
- **Eixo**: DR / Security
- **Prática big-tech**: AWS S3 Object Lock + Glacier Deep Archive + separação de conta.
- **Estado MS**: ausente.
- **Fix M1**: S3 bucket separado com Object Lock compliance mode + retenção 90d; backup incremental nightly. Ransomware encrypta prod, backup sobrevive.

### G-002 — Tracing distribuído OTel ponta-a-ponta incompleto
- **Eixo**: Observability
- **Prática big-tech**: Jaeger + Tempo + Honeycomb.
- **Estado MS**: mencionado em `observability.md`; não comprovado ponta-a-ponta (backend → Stripe webhook → DB).
- **Fix M1**: `otelgrpc` + `otelhttp` em todos entry/exit points; Stripe webhook handler com trace context propagado; trace ID em log zerolog.

---

## Bloco 3 — Práticas ops importantes M2 (top-10)

1. **G-012** Chaos engineering plan mesmo em staging (LitmusChaos/ChaosMesh) — testar failover NATS/Redis/PG.
2. **G-021** Rate limit adaptativo (não só static) — Stripe-style aumenta threshold se histórico limpo.
3. **G-007** Log sampling em alto volume — reduzir custo observability.
4. **G-014** Multi-AZ Railway confirmado + documentado.
5. **G-043** DPIA por sub-produto — checklist LGPD Art. 32.
6. **G-008** Golden signals dashboards (latency, traffic, errors, saturation) per BC.
7. **G-036** Database migration online (sem downtime) pg-osc / pt-online-schema-change adapter.
8. **G-020** Timeout per-call downstream explícito (contract test).
9. **G-028** Pentest externo agendado (annual) orçado.
10. **G-039** Blameless postmortem obrigatório + template em `07-quality/postmortem-process.md`.

---

## Bloco 4 — Roadmap prescritivo (priorizado pro usuário)

### Esta semana (pré-Sprint 10 kickoff)
1. Q-020 Ata/Timeline DB-level constraint
2. Q-027 Edital 8d (Lei 4.591/64) — **urgência regulatória**
3. Q-024 + G-034 RLS default-on + TenantBoundaryGuard
4. G-016 + G-015 PITR Postgres + RPO/RTO por BC

### Sprint 10 (pré-GA 2026-05-08)
5. Q-026 + G-025 LGPD pseudonimização HMAC + KMS field-level PII
6. G-029 + G-042 runbooks LGPD breach + incident comms template
7. G-024 secret rotation automatizada
8. Q-002 + Q-003 + Q-008 race fixes (cupom, quota, reputação)
9. G-022 + G-023 circuit breaker + bulkhead per provider
10. G-038 on-call rotation formal
11. G-041 + G-042 status page + comms template
12. G-026 CI gates bloqueantes (govulncheck/osv/npm audit)
13. G-030 gitleaks pre-commit local

### Sprint 11-12 (pós-GA, hardening)
14. Q-021 Stripe reconciliation cron
15. Q-022 NATS JetStream file storage + R=3
16. Q-023 Mux upload orphan reaper
17. Q-001 + Q-013 INV vote/session freshness + paused_reorg
18. Q-011 HMAC cupom redesign
19. G-005 error budget automated
20. G-006 audit hash-chain
21. G-013 backup imutável offline
22. G-032 saga primitives `pkg/saga`
23. G-033 auto-rollback prod
24. G-002 OTel ponta-a-ponta

### M3+ (backlog)
- Cell-based architecture (failure domain por grupo tenant)
- Multi-region failover
- Chaos engineering formal
- Bug bounty program
- DPIA por sub-produto
- ANOREG-BR averbação cartorial

---

## Bloco 5 — Padrões-chave descobertos (meta)

1. **Enforcement app-level sem DB constraint** é o padrão de falha #1 (Q-020, Q-002, Q-003). Big techs defendem em profundidade: app + DB trigger + RLS.
2. **State machines incompletas** para estados conflitantes em paralelo (Q-005, Q-013, Q-014, Q-015). Spec só cobre "happy transitions" — faltam transições proibidas e forced-pauses em eventos externos.
3. **Reconciliação proativa ausente** (Q-021, Q-022, Q-023). Webhooks/eventos externos podem perder; precisa cron reconciler + outbox pattern com retries + orphan reaper.
4. **Documentos declarados mas não criados** (G-029, G-042, 12-docs/runbooks/*). `grep -rn "a criar\|TODO\|pendente"` no v2 revelaria > 20 stubs.
5. **Compliance regulatório BR ignorado** em specs MVP (Q-027 Lei 4.591, Q-028 ICP-Brasil, Q-029 voto secreto, G-029 ANPD 72h, G-043 DPIA). **Risco legal > risco técnico** pro negócio.
6. **Isolamento tenant depende só de convenção WHERE em sqlc** (Q-024, Q-030). Single handler com bug = cross-tenant leak. RLS default-on é barreira barata.
7. **Observabilidade madura em spec, operacional imatura** (G-002, G-005, G-006, G-007). Boa definição SLI/SLO, mas zero burn-rate automation + zero tracing end-to-end + audit não tamper-evident.
8. **DR é o eixo mais fraco** (G-013..017, 4 M1 blockers). Backup mencionado, nunca testado; PITR não configurado; ransomware-proof ausente.

---

## Referências linha-por-linha

- Quebras Q-### detalhes e exploração: [[11-audit/audit-log/2026-04-23-quebras-negocio-systematic]]
- Gaps G-### detalhes: [[11-audit/audit-log/2026-04-23-ops-bigtech-gaps]]
- Research verticals: [[13-research/_moc]]
- STATE decisões vivas: [[STATE]]
- AUDIT principais findings: [[AUDIT]]
- Sub-produtos impactados: `00-product/sub-produtos/01..11`
- Invariantes canônicos: [[01-domain/invariants]]
- State machines atuais: [[01-domain/state-machines]]

## Vizinhos

- [[2026-04-23-quebras-negocio-systematic]] — fonte 1 (31 Q-###)
- [[2026-04-23-ops-bigtech-gaps]] — fonte 2 (48 G-###)
- [[../../AUDIT]]
- [[../../STATE]]
- [[../../_moc]]
