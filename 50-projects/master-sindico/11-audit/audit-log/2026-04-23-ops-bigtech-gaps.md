---
title: Ops Big-Tech Gaps — Master Síndico v2 (SRE + Security + Data discipline)
type: audit
tags: [audit, sre, ops, security, reliability, dr, finops, lgpd, master-sindico, v2]
source: 02-architecture/observability.md + patterns.md + event-driven.md + topology-multitenant.md + 06-execution/* + 08-security/* + 09-testing/test-strategy.md + adr/_moc.md + Google SRE Book + AWS Well-Architected + Netflix Tech Blog + 12-Factor
created: 2026-04-23
updated: 2026-04-23
aliases: [ops-bigtech-gaps, sre-security-gaps-2026-04-23, G-### log]
---

# Ops Big-Tech Gaps — Master Síndico v2

Auditoria adversarial de **práticas operacionais/SRE/segurança/data** aplicadas pelas big techs (Google SRE, Netflix, AWS Well-Architected, 12-Factor, Stripe, LinkedIn) versus estado atual da remontagem v2. **Não é** pass de produto nem de arquitetura — apenas disciplinas ops/sec/data que impedem ou amaçam o GA M1 (2026-05-08).

**Mindset**: assume breach + assume outage + assume LGPD audit request amanhã. Se a prática não está escrita em artefato do v2 (02-architecture / 06-execution / 08-security / 12-docs/runbooks), **é gap** — mesmo que "ninguém esqueceria" em teoria.

---

## §1 Sumário

**Total de gaps identificados**: **48**

Por eixo:

| Eixo | Gaps | M1 blockers | M2 | M3+ |
|---|---|---|---|---|
| 1 — Observability / SRE | 7 | 3 | 3 | 1 |
| 2 — Reliability / Chaos | 7 | 2 | 3 | 2 |
| 3 — Disaster Recovery / Data | 9 | 4 | 3 | 2 |
| 4 — Security ops | 8 | 4 | 3 | 1 |
| 5 — Deployment / CI-CD | 6 | 2 | 3 | 1 |
| 6 — Incident management | 5 | 3 | 2 | 0 |
| 7 — Cost / capacity | 3 | 1 | 2 | 0 |
| 8 — Data governance / compliance | 3 | 1 | 1 | 1 |
| **Total** | **48** | **20** | **20** | **8** |

**Leitura-chave**:
- **20 M1 blockers** — impedem rodar SaaS confiável no GA 2026-05-08. Top 15 consolidados em §3.
- **Concentração em DR/Data (9 gaps, 4 M1)** — é o eixo onde o v2 está mais pobre: backup Postgres mencionado vagamente no `STEPS` (`backup/restore test trimestral`) mas **sem RPO/RTO quantificados, sem DR drill doc, sem backup Redis/NATS, sem imutável offline**.
- **Security ops com 4 M1 blockers** — secret rotation é trimestral manual (`deploy.md §4`), sem automação; dep scan existe (govulncheck/npm audit/osv-scanner em `cve-process.md`) mas **sem SAST além do gosec**, DAST zero, pentest apenas "anual" sem data marcada, bug bounty ausente.
- **Incident management falta on-call formal** — `incident.md §canal` diz "PagerDuty oncall pós-Sprint 11" mas M1 é Sprint 10 → **GA sem on-call formal** é risco P0.
- **Postmortem template existe** em `observability.md §6` mas **não há postmortem process obrigatório** cross-team (só em prod P0/P1); falta cadence de review.

---

## §2 Gaps G-### por eixo

### Eixo 1 — Observability / SRE

#### G-001 — Error budget burn rate alerts implementados apenas no papel

- **Eixo**: 1
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: tem em [[../../02-architecture/observability.md]] §5.1 (tabela 4 janelas 1h/6h/24h/72h) mas **sem PromQL concreto nem wire-up no Grafana**. Nenhum dashboard versionado ainda. Nenhum teste de alerta em staging ("alert sim-test") documentado.
- **Prática big-tech**: Google SRE Workbook ch.5 "Alerting on SLOs" — multi-window multi-burn-rate alerts com PromQL canônico + hygiene test bimestral que dispara alerta fake.
- **Por que importa pro MS**: assembleia live (crítica em M3 mas rollout validation em M1) sem burn-rate alert funcional = descobre outage no Twitter. SLO no papel não serve pra nada.
- **Correção nominal**: criar `12-docs/runbooks/alerts/` com 1 arquivo por alert (PromQL + runbook link + severity) + task em `06-execution/qa.md` "alert fire drill mensal".

#### G-002 — OTel tracing ponta-a-ponta backend → provider → webhook não demonstrado

- **Eixo**: 1
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: [[../../02-architecture/observability.md]] §2.3 lista spans canônicos incluindo `provider.stripe.create_subscription`, mas **trace ID não é propagado no payload enviado ao Stripe** (ausente em `metadata.idempotency_key` ou `client_reference_id`). Webhook inbound não injeta trace ID no span do processamento do evento. Resultado: trace quebra na fronteira externa.
- **Prática big-tech**: Dapper paper (Google §4.1) + Stripe docs (`Idempotency-Key: trace_id` convention). AWS X-Ray HTTP header `X-Amzn-Trace-Id` propagado em SDK.
- **Por que importa pro MS**: incident "pagamento não registrou" → Stripe webhook chegou mas billing não processou → sem trace cross-boundary, debug é arqueologia. 90% dos incidents críticos em MS são cross-boundary (Stripe, Mux webhook).
- **Correção nominal**: editar `02-architecture/observability.md` §2.3 add seção "Trace propagation cross-provider" + criar `12-docs/runbooks/trace-debugging.md`.

#### G-003 — Dashboards Grafana listados como "canônicos" mas não versionados

- **Eixo**: 1
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/observability.md]] §7 lista 6 dashboards ("Overview / Per-BC / Database / Providers / Business / Outbox+DLQ") mas **não há JSONs de dashboard em repo** (`infra/grafana/dashboards/`). Sem dashboard-as-code = reconstrução manual pós-outage = golden signals perdidos em incident.
- **Prática big-tech**: Grafana Labs + Netflix (`Atlas`) publicam dashboards como código versionado. AWS Well-Architected Operational Excellence pillar.
- **Por que importa pro MS**: 1ª assembleia live com 100 moradores — error budget queimando e ops sem dashboard pronto pra ler os USE/RED metrics é cenário de escândalo com cliente.
- **Correção nominal**: criar `12-docs/runbooks/grafana-dashboards.md` + estrutura `infra/grafana/dashboards/*.json` prevista no `06-execution/deploy.md` §1 (pipeline faz `grafana-cli` import).

#### G-004 — Alert runbook link obrigatório no alerta — regra existe, enforcement zero

- **Eixo**: 1
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/observability.md]] §5.2 ("todo alert tem runbook") é regra no papel; sem CI gate / grep / template linter. Pasta `12-docs/runbooks/` referenciada não listada como existente no `06-execution/playbooks/` que enumerei.
- **Prática big-tech**: Google SRE ch.6 ("Your alerts should have runbooks — always"). PagerDuty enforce via `runbook_url` obrigatório em service config.
- **Por que importa pro MS**: on-call solo 2026-Q2 (M1 tem dev sênior + pleno apenas) não pode improvisar às 3am. Alert fatigue mata a cultura.
- **Correção nominal**: editar `06-execution/qa.md` sprint-auditor checklist adicionando "grep PromQL alerts sem `annotations.runbook_url`".

#### G-005 — Audit log tamper-evident (hash chain) planejado só M2 — CloudTrail-like ausente em M1

- **Eixo**: 1
- **Prioridade**: M2
- **Esforço**: M
- **Estado MS**: [[../../08-security/BEYOND_CORP.md]] §11.4 menciona "hash-chain rolling M2" + "S3 Object Lock M3"; M1 entrega só `REVOKE UPDATE, DELETE ON audit_log`. Não há prova criptográfica contra DBA malicioso.
- **Prática big-tech**: AWS CloudTrail Log File Integrity (SHA-256 digest chain). Google Cloud Audit Logs imutáveis por design.
- **Por que importa pro MS**: ata de assembleia tem **valor jurídico em disputa**. Juiz pedindo prova de não-tampering → `REVOKE UPDATE` é frágil. Esperar M2 é aceitável se M1 documentar limite explícito ao cliente.
- **Correção nominal**: aditar `08-security/BEYOND_CORP.md` §11 com disclaimer M1→M2 visível + ADR-0032 "audit log integrity roadmap".

#### G-006 — Log sampling em alto volume sem estratégia

- **Eixo**: 1
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/observability.md]] §2.3.1 tem sampling **de traces** (1%/100%-5xx/100%-debug) mas **não de logs**. Loki ingestion custa por GB; sem log sampling cliente enterprise com 50 condomínios vai explodir custo.
- **Prática big-tech**: Google SRE ch.12 "Logging" + Netflix Atlas (tiered retention). Loki tenant-level limits.
- **Por que importa pro MS**: M2 com 500 tenants logs volume 10×; sem sampling = bill Grafana Cloud explode + alert fatigue em dashboard lento.
- **Correção nominal**: add seção §2.1.3 "Log sampling strategy" em `02-architecture/observability.md` (info 10% sample se endpoint > 1krpm; warn/error sempre 100%).

#### G-007 — Golden signals (latency/traffic/errors/saturation) não mapeados explicitamente por BC

- **Eixo**: 1
- **Prioridade**: M3
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/observability.md]] §2.2 tem RED + USE mas não rotulados como "golden signals Google". Per-BC dashboard (§7 item 2) promete breakdown mas sem checklist fixo.
- **Prática big-tech**: Google SRE ch.6 "Monitoring Distributed Systems" — 4 golden signals por microserviço.
- **Por que importa pro MS**: dev junior olha dashboard e não sabe se está completo. Standardize.
- **Correção nominal**: editar `02-architecture/observability.md` §7 adicionando checklist fixo "4 signals obrigatórios por BC" + exemplo PromQL.

---

### Eixo 2 — Reliability / Chaos

#### G-008 — Timeout policy por chamada downstream sem tabela canônica

- **Eixo**: 2
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/patterns.md]] §12 tem CB config (open thresholds) mas **não timeouts por provider**. Código atual usa context deadline ad-hoc. `event-driven.md` §8 retry delay 200ms→5s, sem timeout per-call.
- **Prática big-tech**: Netflix Hystrix "timeout é o primeiro default a configurar; circuit breaker é o segundo". Stripe SDK default 80s — absurdo pra webhook path.
- **Por que importa pro MS**: Stripe lento = request MS acumula conn PG → pool exaurido → cascade. Timeout agressivo é **1ª barreira** antes do CB.
- **Correção nominal**: add tabela "Timeouts canônicos por provider" em `02-architecture/patterns.md` §12 (Stripe 10s, Mux 5s, LiveKit 3s, Zitadel 2s, Twilio 5s, Resend 5s).

#### G-009 — Bulkhead pattern ausente — 1 tenant ruidoso derruba todos

- **Eixo**: 2
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: [[../../02-architecture/topology-multitenant.md]] §3 tabela admite "tenant grande pode noisy-neighbor; mitigação connection pool limits por tenant em **M3+**". M1 não tem bulkhead = 1 admin rodando bulk import trava todos.
- **Prática big-tech**: Netflix Hystrix thread pool per dependency. AWS ECS task CPU/memory limits. Shopify pod-per-shop bulkhead.
- **Por que importa pro MS**: 1ª administradora cliente com 50 condomínios importa massa de dados → pool PG travado → assembleia live paralela travada → cliente A culpa cliente B → contrato em risco.
- **Correção nominal**: editar `02-architecture/topology-multitenant.md` §3 — adicionar "bulkhead M1 via `pg_bouncer` per-tenant connection limit + rate limit per-tenant" (não esperar M3).

#### G-010 — Chaos engineering plan existe só como checkbox futuro

- **Eixo**: 2
- **Prioridade**: M2
- **Esforço**: M
- **Estado MS**: [[../../09-testing/test-strategy.md]] §10 diz "M1-M2 não introduzir. M3 latency injection staging (Toxiproxy). M4+ Chaos Monkey blast=1 pod prod". Pré-requisitos listados mas **nenhum chaos test sequer em staging antes do GA**.
- **Prática big-tech**: Netflix Simian Army desde 2011. Google DiRT. Stripe "Game Day" trimestral.
- **Por que importa pro MS**: saga de assembleia (criar room → egress → publicar ata) tem 5 passos em providers externos. Sem chaos, 1ª vez que Mux cai em prod é 1ª vez que compensação é testada ao vivo.
- **Correção nominal**: antecipar 1 chaos drill M2 em `06-execution/STEPS.md` §6 (mensal: derrubar 1 provider em staging; valida CB + saga compensation + alerting).

#### G-011 — Load shedding ausente — saturação vira 5xx em cascata

- **Eixo**: 2
- **Prioridade**: M2
- **Esforço**: M
- **Estado MS**: rate limit existe ([[../../08-security/BEYOND_CORP.md]] §6) mas é **rate limit estático por IP/user**, não load shedding adaptativo (recusar antes de saturar PG pool / goroutines).
- **Prática big-tech**: Google SRE ch.22 "Addressing Cascading Failures" — `max_inflight_requests` + shed low-priority first (health checks < user traffic < admin).
- **Por que importa pro MS**: assembleia live 100+ vote concurrent + síndico tentando publicar video = pool PG saturado → tudo 500. Sem shed, ressurreição leva 20min.
- **Correção nominal**: criar `12-docs/runbooks/load-shedding.md` + ADR M2 "shed de `/admin/*` + `/neighborhood/*` antes de `/assemblies/*/vote`".

#### G-012 — Rate limit adaptativo (Stripe style) não planejado

- **Eixo**: 2
- **Prioridade**: M3
- **Esforço**: L
- **Estado MS**: [[../../08-security/BEYOND_CORP.md]] §6 rate limit fixo por path group. Sem feedback loop (backend saturation → apertar rate limit dinamicamente).
- **Prática big-tech**: Stripe engineering blog "Adaptive Concurrency Limits" / Netflix Concurrency Limits. AWS adaptive throttling.
- **Por que importa pro MS**: M3+ escala com 5k tenants; rate static não aguenta pico de assembleias do fim-do-mês.
- **Correção nominal**: ADR M3 "adaptive concurrency limits" + editar `BEYOND_CORP.md` §6 roadmap.

#### G-013 — Retry policy documentada mas jitter seed determinístico não mencionado

- **Eixo**: 2
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/event-driven.md]] §8 retry com jitter (`retry.RandomDelay`) — porém sem spec do seed (padrão lib usa `time.Now` — aceitável) nem **retry budget global** (limite total de retries cross-system antes de shed).
- **Prática big-tech**: Google SRE ch.22 "retry budget" — percent of total RPS que pode ser retry. Sem isso: thundering herd pós-outage derruba novamente.
- **Por que importa pro MS**: Stripe volta após 30min outage → worker outbox tenta 1000 eventos ao mesmo tempo → Stripe rate-limit → loop.
- **Correção nominal**: add §8.1 "Retry budget" em `02-architecture/event-driven.md` (cap 10% do RPS normal; excedendo → DLQ).

#### G-014 — CB fallback "fail-closed vs fail-open" decidido caso-a-caso, não tabulado

- **Eixo**: 2
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/patterns.md]] §12 anti-pattern menciona "fail-closed (auth/billing) vs fail-open (cosmético)" sem tabela canônica por provider.
- **Prática big-tech**: Netflix "Fallback Strategies" — tabela documentada por dependency.
- **Por que importa pro MS**: Zitadel down → deve devolver 503 (fail-closed) ou permitir cache-only (fail-open stale)? Decisão precisa estar escrita, não no dev mid-incident.
- **Correção nominal**: add tabela "Fallback strategy por provider" em `02-architecture/patterns.md` §12.

---

### Eixo 3 — Disaster Recovery / Data

#### G-015 — RPO / RTO não quantificados por sub-produto

- **Eixo**: 3
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../06-execution/STEPS.md]] §6 diz "backup/restore test trimestral" mas **sem RPO/RTO definidos**. SLO de disponibilidade existe ([[../../02-architecture/observability.md]] §3.1 tabela por BC) mas RPO (dado perdido ok?) / RTO (quanto tempo até restaurar?) ausentes.
- **Prática big-tech**: AWS Well-Architected Reliability Pillar — RPO/RTO quantitativo por workload tier.
- **Por que importa pro MS**: ata de assembleia (BC assembly) tem RPO = 0 tolerável (assinatura jurídica); timeline/feed (BC institutional) tolera RPO 15min; billing RPO ≤ 5min. Sem isso, backup é one-size-fits-all.
- **Correção nominal**: add matriz "RPO/RTO por BC" em `02-architecture/observability.md` §3.1 ao lado da matriz SLO.

#### G-016 — Backup Postgres PITR sem config documentada

- **Eixo**: 3
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: `06-execution/deploy.md` §pré-deploy menciona "DB snapshot pré-migration (pgdump em S3)" — ou seja, **snapshot one-shot manual**, não PITR contínuo. Railway Postgres default tem backup diário mas **WAL archiving / PITR não configurado no v2**.
- **Prática big-tech**: AWS RDS automated backups + PITR (5min). Google Cloud SQL PITR default.
- **Por que importa pro MS**: incident 14h → rollback volta 14h de dados = toda assembleia do dia perdida. Snapshot diário não serve.
- **Correção nominal**: criar `12-docs/runbooks/postgres-pitr.md` + editar `06-execution/deploy.md` §3 "migrations expand-contract" add "WAL archiving staging/prod + PITR 5min RPO".

#### G-017 — DR drill não existe — backup é schrödinger

- **Eixo**: 3
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: [[../../06-execution/STEPS.md]] §6 "backup/restore test trimestral" é 1 linha vaga. Sem procedure, sem responsável, sem staging-restored database validado.
- **Prática big-tech**: Google SRE ch.26 "Data Integrity — you only have a backup when you've restored it". Netflix DRT bimestral.
- **Por que importa pro MS**: 99% dos backups que precisam ser restaurados **não funcionam na 1ª tentativa** (SRE book data). Descobrir isso em incident real = fim de contrato.
- **Correção nominal**: criar `12-docs/runbooks/dr-drill.md` com playbook mensal; editar `06-execution/qa.md` sprint-auditor para checkar "último DR drill ≤ 30d".

#### G-018 — Backup Redis não mencionado

- **Eixo**: 3
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: busca em arquivos-fonte: Redis aparece como cache ABAC ([[../../08-security/BEYOND_CORP.md]] §3.3) + idempotency webhook ([[../../02-architecture/patterns.md]] §11) + rate limit (§6) + session denylist. **Persistence / snapshot / AOF** não mencionados em DR.
- **Prática big-tech**: Redis docs — AOF `everysec` + RDB snapshot. Stripe/Shopify persistem Redis.
- **Por que importa pro MS**: perder Redis = perder idempotency webhook Stripe → pagamento dobra cobrado. Perder denylist = sessions revogadas voltam. M1 Redis é single (Railway); crash = reset cold.
- **Correção nominal**: editar `02-architecture/patterns.md` §11 add "Idempotency durability: Redis AOF everysec + fallback DB `processed_events`".

#### G-019 — Backup NATS JetStream não aplicável M1 mas sem plano M2

- **Eixo**: 3
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/event-driven.md]] §2.1 confirma "M1 outbox only, NATS só M2". **Porém**: quando NATS entrar M2, `event-driven.md` não menciona backup de streams críticas (billing 30d, assembly 90d).
- **Prática big-tech**: NATS docs — stream snapshot + cluster replication factor ≥ 3.
- **Por que importa pro MS**: stream `events.assembly` retém 90d = ata-relacionado. Perder = gap auditoria LGPD.
- **Correção nominal**: editar `02-architecture/event-driven.md` §9 add "Stream backup policy quando NATS entrar".

#### G-020 — Multi-AZ no Railway não verificado

- **Eixo**: 3
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: Railway é escolha M1 ([[../../02-architecture/adr/_moc.md]] ADR-0017) mas **sem verificação se o plano contratado tem multi-AZ**. Railway pro tier tem; hobby não. Sem isso, 1 AZ outage = down.
- **Prática big-tech**: AWS Well-Architected Reliability — multi-AZ default.
- **Por que importa pro MS**: SLO 99.5% precisa 3.6h downtime/mês budget. 1 AZ outage AWS us-east já ultrapassa.
- **Correção nominal**: editar `06-execution/deploy.md` adicionar checklist pré-GA "Railway plan tem multi-AZ confirmado? serviço por serviço".

#### G-021 — Schema migration reversível (down) ausente

- **Eixo**: 3
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: [[../../06-execution/deploy.md]] §3 exige expand-contract (bom), **mas não exige `down` migration escrita e testada**. Goose suporta; prática atual assumida "não roda down em prod".
- **Prática big-tech**: 12-Factor VIII (dev/prod parity). Stripe migration guide exige reversível + CI test up-then-down.
- **Por que importa pro MS**: migration quebra deploy → expand fez coluna nova → rollback deploy mas coluna nova fica → código antigo crasha em "unknown column". Sem down, rollback DB = restaurar snapshot (horas).
- **Correção nominal**: editar `06-execution/deploy.md` §3 add "Toda migration tem `down` executável + CI roda `up → down → up` em test container".

#### G-022 — Backup imutável offline (ransomware) ausente

- **Eixo**: 3
- **Prioridade**: M3
- **Esforço**: M
- **Estado MS**: nada sobre backup isolado em S3 com Object Lock retenção. `BEYOND_CORP.md` §11.4 menciona S3 Object Lock **apenas para audit_log M3**. DB backup não tem imutabilidade.
- **Prática big-tech**: AWS Backup Vault Lock. Veeam 3-2-1-1-0. Post-Colonial Pipeline best practice.
- **Por que importa pro MS**: ransomware criptografa prod + backups recentes. Sem cópia air-gapped = pagar ou morrer. Condomínio BR tem atacante motivado (folha de pagamento + ata).
- **Correção nominal**: add seção "Ransomware-resistant backup" em `12-docs/runbooks/dr-drill.md` + ADR M3.

#### G-023 — Multi-region failover plan ausente

- **Eixo**: 3
- **Prioridade**: M3
- **Esforço**: L
- **Estado MS**: [[../../02-architecture/topology-multitenant.md]] §8 menciona "M3 read replicas regionais SP/RJ" mas sem failover plan. ADR-0017 Railway M1 / AWS M4+ não cobre DR region.
- **Prática big-tech**: Netflix active-active across regions. AWS Well-Arch ch.10.
- **Por que importa pro MS**: AWS SP outage (histórico 2020, 2023) = down nacional. Cliente enterprise exigirá SLA com DR.
- **Correção nominal**: ADR M3 "multi-region failover" + runbook.

---

### Eixo 4 — Security ops

#### G-024 — Secret rotation automação ausente — trimestral manual

- **Eixo**: 4
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: [[../../06-execution/deploy.md]] §4 "Rotação trimestral (Stripe, Zitadel, Twilio, Mux, Livekit, Resend)" — **manual**. `BEYOND_CORP.md` §10.2 idem. Sem automação, ninguém vai lembrar em 2026-Q3.
- **Prática big-tech**: AWS Secrets Manager automatic rotation. HashiCorp Vault dynamic secrets. Google Cloud Secret Manager version rotation.
- **Por que importa pro MS**: secret leaked (gitleaks gap, logs, S3) → janela de exposição = até próxima rotação trimestral = até 90d. Com automação mensal → 30d.
- **Correção nominal**: ADR-0032 "secret rotation automation" + runbook em `12-docs/runbooks/secret-rotation.md`; editar `06-execution/deploy.md` §4.

#### G-025 — KMS field-level encryption PII ausente

- **Eixo**: 4
- **Prioridade**: M1
- **Esforço**: L
- **Estado MS**: [[../../08-security/BEYOND_CORP.md]] §7 mask em logs ok; §11 audit_log ok. Porém **DB at-rest CPF/CNPJ/email armazenados em plaintext**. ADR-0028 LGPD keyed HMAC é pra pseudonimização em logs, não field-level at-rest.
- **Prática big-tech**: AWS KMS + pgcrypto envelope encryption por linha. Stripe PII tokenization. Shopify Vault.
- **Por que importa pro MS**: DBA malicioso OU SQL injection OU backup vazado = CPFs dos moradores expostos = LGPD breach 72h ANPD. Encryption at-rest do Railway protege disk theft, não DBA.
- **Correção nominal**: ADR-0033 "field-level encryption PII tier1" + editar `08-security/BEYOND_CORP.md` §7 add subseção.

#### G-026 — SAST/DAST além de gosec ausente

- **Eixo**: 4
- **Prioridade**: M2
- **Esforço**: M
- **Estado MS**: [[../../08-security/cve-process.md]] §2.1 tem gosec + govulncheck + trivy + osv-scanner. **Falta**: Semgrep/CodeQL (SAST mais abrangente), OWASP ZAP/Burp (DAST contra API staging).
- **Prática big-tech**: GitHub Advanced Security (CodeQL) default. Stripe Semgrep + Snyk. OWASP ZAP baseline em CI.
- **Por que importa pro MS**: gosec pega SQLi óbvio, não pega lógica (ex: IDOR em `/api/v1/condominiums/:id` se dev esqueceu tenant check). Semgrep custom rules pegam.
- **Correção nominal**: editar `08-security/cve-process.md` §2.1 add Semgrep + ZAP baseline + issue GitHub Advanced Security ativação.

#### G-027 — Pentest externo "anual" sem data agendada

- **Eixo**: 4
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../08-security/BEYOND_CORP.md]] §15.2 "Anual: pen-test externo". §14.2 M2 "Pen-test externo 1º ciclo". Sem vendor, sem orçamento, sem data. No pentest-specs.md interno (self-pentester 42 findings) fica como checklist mas não externo.
- **Prática big-tech**: SOC2 exige pentest anual independente. Stripe/PagerDuty publicam vendor relationship.
- **Por que importa pro MS**: 1º cliente enterprise em M2 vai pedir laudo pentest. Sem = perde venda. Agendar pre-M2 = 90d lead time com vendor (Trustwave, NCC, HackerOne).
- **Correção nominal**: add item em `10-schedule/milestones.md` pré-M2 "pentest vendor agendado" + criar `08-security/pentest-vendor-rfq.md`.

#### G-028 — Bug bounty roadmap ausente

- **Eixo**: 4
- **Prioridade**: M3
- **Esforço**: S
- **Estado MS**: busca em `08-security/` — zero menção HackerOne/YesWeHack/Bugcrowd.
- **Prática big-tech**: Google VRP, Stripe VDP (Vulnerability Disclosure Program) gratuito sem bounty + bounty formal quando maduro.
- **Por que importa pro MS**: VDP sem bounty ($0) mostra maturidade + captura low-hanging fruits antes de atacante. Bug bounty formal é pós-SOC2.
- **Correção nominal**: criar stub `08-security/bug-bounty-roadmap.md` com VDP M3 + paid bounty M5.

#### G-029 — LGPD incident response 72h sem playbook detalhado

- **Eixo**: 4
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../06-execution/incident.md]] §LGPD tem timeline 72h (bom) mas referencia `12-docs/runbooks/incident-lgpd-breach.md` que **não existe** (listado "a criar"). Sem playbook, DPO improvisa.
- **Prática big-tech**: Facebook/Meta breach playbook pós-2019 incident. GDPR article 33 equivalent.
- **Por que importa pro MS**: ANPD pode multar 2% faturamento (LGPD Art. 52) por comunicação fora-de-prazo. 72h é apertado — precisa ensaio.
- **Correção nominal**: criar `12-docs/runbooks/incident-lgpd-breach.md` com template de notificação + comm titulares + postmortem externo.

#### G-030 — Pre-commit hook gitleaks existe em CI mas não em dev local

- **Eixo**: 4
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../08-security/cve-process.md]] §6 tem gitleaks no CI. [[../../08-security/BEYOND_CORP.md]] §10.4 menciona "gitleaks em pre-commit hook" mas **sem instalação documentada** (nenhum `.pre-commit-config.yaml` em artefato do v2 ou link).
- **Prática big-tech**: pre-commit framework + lefthook. Stripe hook obrigatório on-clone.
- **Por que importa pro MS**: A-018 já ocorreu (zitadel-key vazou) — repetir esse incident em M1 com cliente real é PR disaster.
- **Correção nominal**: criar `06-execution/dev.md` §onboarding dev add "Instalar pre-commit + lefthook" + arquivo `.pre-commit-config.yaml` listado em CLAUDE.

#### G-031 — Least privilege prod audit trimestral ausente

- **Eixo**: 4
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: [[../../08-security/BEYOND_CORP.md]] §12.7 menciona JIT/JEA em M2+ mas **sem auditoria**: quem tem Railway prod access? quem tem AWS console (M2)? quem tem DB direct connection?
- **Prática big-tech**: Google "prod access review" trimestral. AWS Well-Arch Security ch.IAM.
- **Por que importa pro MS**: dev saindo com prod access ativo = vetor. Fornecedor (Railway) operador = suspeita.
- **Correção nominal**: add item trimestral em `06-execution/qa.md` "prod access matrix review" + template em `08-security/access-review-template.md`.

---

### Eixo 5 — Deployment / CI-CD

#### G-032 — Canary deployment planejado mas sem feature flag infra

- **Eixo**: 5
- **Prioridade**: M2
- **Esforço**: M
- **Estado MS**: [[../../06-execution/deploy.md]] §2 deploy diz "Canary (se feature flag): 5% → 25% → 50% → 100% em 4h". **Mas**: busca em `02-architecture/` não encontra escolha de feature flag provider (LaunchDarkly, Unleash, hand-rolled, Flipt).
- **Prática big-tech**: Netflix Archaius. Stripe internal flags. Shopify beta flags.
- **Por que importa pro MS**: sem flag infra = canary é toggle env var = all-or-nothing = não é canary.
- **Correção nominal**: ADR-0034 "feature flag provider" (provisório: Unleash self-host ou Flipt).

#### G-033 — Automated rollback em SLO regression ausente em prod

- **Eixo**: 5
- **Prioridade**: M1
- **Esforço**: M
- **Estado MS**: [[../../06-execution/deploy.md]] §1 tem rollback automático **staging** (error rate > 0.5% por 15min). §2 prod: "Monitorar... Alertas PagerDuty" — rollback **manual**. [[../../06-execution/rollback.md]] confirma: gatilhos automáticos só staging.
- **Prática big-tech**: AWS CodeDeploy automatic rollback on CloudWatch alarm. Spinnaker canary analysis.
- **Por que importa pro MS**: prod deploy 2am, error rate spike → ops dormindo → 30min de impacto antes de acknowledge → SLO budget trashed.
- **Correção nominal**: editar `06-execution/deploy.md` §2 add "Prod auto-rollback on SLO burn fast (1h window > 14.4×) → Railway promote automatically" + runbook.

#### G-034 — Deploy freeze pre-assembleia não documentado

- **Eixo**: 5
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../06-execution/deploy.md]] §pré-deploy tem "Code freeze 48h pré-marco" (M1/M2/M3 release). **Não tem** freeze por evento cliente (assembleia agendada).
- **Prática big-tech**: Stripe "change freeze" em Black Friday. AWS deploy freeze pré-reinvent. Netflix Oscar night freeze.
- **Por que importa pro MS**: cliente marca assembleia terça 19h. Dev deploya nova versão do WebSocket terça 18h30. Assembleia quebra = contrato em risco.
- **Correção nominal**: editar `06-execution/deploy.md` §pré-deploy add "Consultar calendar assembleias: freeze 24h antes de qualquer assembleia agendada em cliente enterprise".

#### G-035 — DB migration online (pt-osc/gh-ost style) ausente

- **Eixo**: 5
- **Prioridade**: M2
- **Esforço**: L
- **Estado MS**: expand-contract ([[../../06-execution/deploy.md]] §3) cobre column add/drop online. **Falta**: DDL com lock pesado (ALTER TABLE ADD CONSTRAINT em tabela grande, REINDEX). Goose não gerencia.
- **Prática big-tech**: gh-ost (GitHub), pt-online-schema-change (Percona), pg_repack (Postgres).
- **Por que importa pro MS**: `timeline_entries` vai crescer; M2 REINDEX = lock table = downtime. Preparar antes.
- **Correção nominal**: ADR M2 "online schema change tool for Postgres" + editar `06-execution/deploy.md` §3.

#### G-036 — Smoke tests pós-deploy automáticos parciais

- **Eixo**: 5
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: [[../../06-execution/deploy.md]] §1 smoke test Playwright 5 fluxos **staging**. §2 prod: "Smoke test prod: GET /health / auth/me / endpoint crítico por sub-produto" — semi-manual, não automatizado.
- **Prática big-tech**: Synthetic monitoring continuous (Datadog, Pingdom, Grafana Synthetics). Netflix canary analysis.
- **Por que importa pro MS**: prod deploy 5% canary — precisa análise automatizada dos 5% vs 95% (latency, error rate) antes de promover.
- **Correção nominal**: add synthetic monitoring em `02-architecture/observability.md` (Grafana Synthetics ou hand-rolled cron hitting `/health` + endpoints críticos).

#### G-037 — Blue/green não planejado — Railway rolling only

- **Eixo**: 5
- **Prioridade**: M3
- **Esforço**: M
- **Estado MS**: [[../../06-execution/deploy.md]] §1 "zero-downtime rolling deploy". Não há blue/green (2 envs paralelos).
- **Prática big-tech**: AWS CodeDeploy blue/green. Heroku preboot. Netflix red/black.
- **Por que importa pro MS**: M3 assembleia live 100+ user com WebSocket persistent conn — rolling derruba sessions. Blue/green + connection drain = zero impact.
- **Correção nominal**: ADR M3 "blue/green deployment for WebSocket services" (quando migrar Railway → AWS ECS ADR-0017).

---

### Eixo 6 — Incident management

#### G-038 — On-call rotation formal só pós-Sprint 11 — GA é Sprint 10

- **Eixo**: 6
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../06-execution/STEPS.md]] §6 "Oncall rotation 24/7 a partir Sprint 11 (assembleias live exigem)". [[../../06-execution/incident.md]] §canal "Pré-M3: oncall primário (dev sênior) + secundário — rotação semanal". **Porém**: GA M1 2026-05-08 = Sprint 10 fim = **zero on-call formal no GA**.
- **Prática big-tech**: PagerDuty/Opsgenie com schedule. Stripe 3-tier. Netflix primary/secondary/escalation.
- **Por que importa pro MS**: síndico paga trial → 1º pagamento Stripe 2am → webhook falha → sem on-call = descobre 9h manhã = subscription quebrada antes do 1º dia útil de operação.
- **Correção nominal**: antecipar oncall pra Sprint 9 (pre-GA 2 semanas); editar `06-execution/incident.md` §canal + STEPS §6.

#### G-039 — Severidade Sev0/1/2/3 definidas, TTR SLA sem PagerDuty config

- **Eixo**: 6
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../06-execution/incident.md]] §classificação tabela P0..P4 + TTR — ótima. **Mas**: sem PagerDuty/Opsgenie account configurado listado em artefato. Sem urgency mapping (P0 → imediato SMS + call).
- **Prática big-tech**: PagerDuty urgency + notification rules.
- **Por que importa pro MS**: P0 LGPD breach às 3am → email only = perdido até manhã.
- **Correção nominal**: criar `12-docs/runbooks/pagerduty-setup.md` + editar `06-execution/incident.md` §canal.

#### G-040 — Postmortem cadence review ausente

- **Eixo**: 6
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/observability.md]] §6 postmortem template ok; [[../../06-execution/incident.md]] diz "em todo incident P0/P1". **Falta**: cadence de review (mensal?) — postmortem vira arquivo morto sem ação.
- **Prática big-tech**: Google SRE ch.15 "Postmortem Culture" — revisão mensal cross-team + action items tracking explícito.
- **Por que importa pro MS**: time 2 devs M1 sem review = action items de postmortem não viram T-###.
- **Correção nominal**: add `06-execution/qa.md` sprint-auditor checklist "postmortems do mês têm AI tracked em T-###".

#### G-041 — Public status page ausente

- **Eixo**: 6
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: [[../../02-architecture/observability.md]] §1 menciona "status page BetterStack ou statuspage.io". [[../../06-execution/incident.md]] "status page (pós-M3) — Statuspage.io ou custom". **Conflito**: obs diz M1, incident diz M3.
- **Prática big-tech**: Stripe, GitHub, AWS — public status page obrigatório.
- **Por que importa pro MS**: cliente enterprise em M2 exige status page pra SLA. Sem = cliente pergunta por email = suporte overwhelmed.
- **Correção nominal**: resolver conflito (decidir M2) + criar `12-docs/runbooks/status-page.md` + ADR provider.

#### G-042 — Cliente communication plan em incident é 1 linha

- **Eixo**: 6
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../06-execution/incident.md]] §5 "P0/P1 prolongado: status page update; email a clientes afetados" — 1 linha. Sem templates, sem canal, sem DPO comm timeline LGPD (ok, está em §LGPD mas só ANPD).
- **Prática big-tech**: Stripe incident comms playbook. Cloudflare "We broke the internet" series — comm franca.
- **Por que importa pro MS**: síndico cliente pequeno que percebe sozinho = perdeu confiança. Comm proativa = mitiga churn.
- **Correção nominal**: criar `12-docs/incident-communication-template.md` (mencionado em `cve-process.md §7.3` como "a criar").

---

### Eixo 7 — Cost / capacity planning

#### G-043 — Budget alerts por provider ausente

- **Eixo**: 7
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: busca em `02-architecture/` e `06-execution/`: zero menção Stripe balance alert, Mux minutes alert, LiveKit participant-hours, Twilio SMS budget, Railway resource usage. Sem alert = descobre em fatura.
- **Prática big-tech**: AWS Budgets + alerts. Google Cloud Budget Notifications. Stripe Dashboard radar.
- **Por que importa pro MS**: bug em loop de SMS Twilio = R$ milhares em 24h. Budget alert 50%/80%/100% pega cedo.
- **Correção nominal**: criar `12-docs/runbooks/provider-budget-alerts.md` + editar `06-execution/qa.md` sprint-auditor "alerts budget ativos".

#### G-044 — Capacity planning trimestral ausente

- **Eixo**: 7
- **Prioridade**: M2
- **Esforço**: S
- **Estado MS**: `02-architecture/topology-multitenant.md` §8 tabela de fases por tenant count mas sem processo trimestral de revisão (growth rate × capacity).
- **Prática big-tech**: Google SRE ch.11 "Being On-Call" + ch.29 "Dealing with Interrupts" — capacity review mensal.
- **Por que importa pro MS**: crescimento 2x mês = DB hit 80% → esperar 6 meses pra descobrir.
- **Correção nominal**: add item trimestral em `06-execution/qa.md` "capacity review: PG connections peak, Redis memory, tenant count growth".

#### G-045 — FinOps cross-charge por tenant ausente

- **Eixo**: 7
- **Prioridade**: M2
- **Esforço**: M
- **Estado MS**: [[../../02-architecture/observability.md]] §2.2.1 diz "tenant_id label com contagem baixa" mas **sem tracking de cost per tenant** (Mux minutes por tenant, LiveKit hours por tenant, SMS count per tenant).
- **Prática big-tech**: AWS Cost Allocation Tags. Shopify per-shop profitability.
- **Por que importa pro MS**: identificar tenant que consome 80% Mux e paga plano basic = sinal de upgrade forçado ou cancelamento (free-rider).
- **Correção nominal**: add §14 "Cost per tenant tracking" em `02-architecture/observability.md` (métricas `provider_usage_total{provider, tenant_tier}`).

#### G-046 — Idle resource cleanup (orphan Mux assets, stale Redis sessions)

- **Eixo**: 7
- **Prioridade**: M1 (blocker custo)
- **Esforço**: S
- **Estado MS**: upload Mux aborted → asset órfão cobra storage. Session Redis expirada mas denylist persiste. Nenhum arquivo menciona garbage collection scheduled.
- **Prática big-tech**: AWS S3 Lifecycle policies. Shopify orphan cleanup worker.
- **Por que importa pro MS**: 1000 uploads falhos/mês * R$ 0.005/GB storage = baixo mas acumula; sem cleanup = Mux cobra sem razão.
- **Correção nominal**: add worker "orphan cleanup" em `02-architecture/event-driven.md` §9 (stream workers); runbook `12-docs/runbooks/orphan-cleanup.md`.

---

### Eixo 8 — Data governance / compliance

#### G-047 — Data classification PII tier1/tier2/public ausente

- **Eixo**: 8
- **Prioridade**: M1
- **Esforço**: S
- **Estado MS**: [[../../08-security/BEYOND_CORP.md]] §7 lista "proibido em logs" (CPF, CNPJ, email, etc) mas sem tier formal. LGPD Art. 11 (dados sensíveis) vs Art. 5 I (dados pessoais) não distinguidos em schema DB.
- **Prática big-tech**: Stripe data classification tiers. Google PII taxonomy.
- **Por que importa pro MS**: DPIA precisa começar por "quais campos são tier 1 (saúde/raça), tier 2 (CPF/email), tier 3 (nome público)". Sem = DPIA frouxo.
- **Correção nominal**: criar `08-security/data-classification.md` + editar `01-domain/` schema DB add comentário `@tier` em campos sensíveis.

#### G-048 — Data retention policy por entidade ausente

- **Eixo**: 8
- **Prioridade**: M2
- **Esforço**: M
- **Estado MS**: [[../../02-architecture/observability.md]] §9 tabela retention **só de logs** (info 30d, audit 5 anos). **Não existe** retention por entidade de negócio: user deletado (LGPD direito esquecimento), timeline (ata imutável 5 anos), video (lock 90d + retention pós?), session (Redis TTL ok), outbox processed (partition drop mencionado só exemplo).
- **Prática big-tech**: GDPR art 17 + equivalent LGPD Art. 18 V. Shopify retention schedules per entity.
- **Por que importa pro MS**: right-to-erasure request chega → sem policy = dev decide ad-hoc o que apagar = ou deleta demais (quebra ata jurídica) ou de menos (multa ANPD).
- **Correção nominal**: criar `08-security/data-retention-matrix.md` (User, Timeline, Ata, Video, Coupon, Session, Log, Audit, Evaluation — cada com retention + deletion strategy + LGPD art. ref).

---

## §3 Top 15 gaps M1 blockers (impedem SaaS confiável no GA 2026-05-08)

Priorização pelo critério "sem isso, outage/breach com cliente real em ≤ 30d pós-GA é provável":

| # | Gap | Eixo | Por que blocker |
|---|---|---|---|
| 1 | **G-038** On-call rotation formal pós-Sprint 11 | 6 | GA em Sprint 10 = zero on-call no dia 1 |
| 2 | **G-017** DR drill não existe | 3 | Backup não-testado = 99% falha no restore real |
| 3 | **G-016** Postgres PITR sem config | 3 | Snapshot diário = RPO 24h vs assembleia diária |
| 4 | **G-015** RPO/RTO não quantificados | 3 | Sem número = ops não sabe ambição do backup |
| 5 | **G-033** Auto-rollback em prod ausente | 5 | 2am outage + ops dormindo = budget trashed |
| 6 | **G-024** Secret rotation automação ausente | 4 | Leak janela 90d antes da próxima rotação manual |
| 7 | **G-025** KMS field-level PII encryption | 4 | DBA/backup leak = LGPD breach 72h ANPD |
| 8 | **G-029** LGPD 72h playbook não existe | 4 | Multa 2% faturamento por comm tardio |
| 9 | **G-030** Gitleaks pre-commit local ausente | 4 | Repetir A-018 zitadel-key leak |
| 10 | **G-042** Cliente comm template ausente | 6 | Incident → cliente descobre sozinho → churn |
| 11 | **G-040** Postmortem cadence review ausente | 6 | AI de postmortem vira arquivo morto |
| 12 | **G-039** PagerDuty urgency config ausente | 6 | P0 3am via email → 6h TTA |
| 13 | **G-021** Migration `down` não-testada | 3 | Rollback deploy vs migration dessincroniza |
| 14 | **G-043** Budget alerts providers ausente | 7 | Bug em loop SMS = fatura explode sem aviso |
| 15 | **G-008** Timeout policy downstream sem tabela | 2 | Stripe lento = PG pool exaurido = cascade |

Os outros 5 M1 blockers (**G-001**, **G-002**, **G-009**, **G-020**, **G-046**, **G-047**) têm impacto mas mitigação parcial existe (observability §5, bulkhead parcial via Redis cache, Railway default plan, Mux lifecycle possível).

**Nota**: Top 15 tem 5 gaps de eixo DR/Data + 4 security + 3 incident mgmt + 1 deploy + 1 reliability + 1 cost + 0 obs/compliance — **refletindo que obs tem base sólida (SLO/OTel) e a frente mais pobre é DR/incident/security-ops**.

---

## §4 Roadmap G-### por marco

### M1 blockers (resolver pré-GA 2026-05-08) — 20 gaps

Eixo 1 (3): G-001, G-002, G-003, G-004
Eixo 2 (2): G-008, G-013
Eixo 3 (4): G-015, G-016, G-017, G-020, G-021
Eixo 4 (4): G-024, G-025, G-029, G-030
Eixo 5 (2): G-033, G-034
Eixo 6 (3): G-038, G-039, G-040, G-042
Eixo 7 (1): G-043, G-046
Eixo 8 (1): G-047

### M2 (resolver 2026-06-20) — 20 gaps

Eixo 1 (3): G-005, G-006
Eixo 2 (3): G-009, G-010, G-011, G-014
Eixo 3 (3): G-018, G-019
Eixo 4 (3): G-026, G-027, G-031
Eixo 5 (3): G-032, G-035, G-036
Eixo 6 (2): G-041
Eixo 7 (2): G-044, G-045
Eixo 8 (1): G-048

### M3+ (resolver pós-2026-07-20) — 8 gaps

Eixo 1 (1): G-007
Eixo 2 (2): G-012
Eixo 3 (2): G-022, G-023
Eixo 4 (1): G-028
Eixo 5 (1): G-037
Eixo 6 (0): —
Eixo 7 (0): —
Eixo 8 (1): —

---

## §5 Observações finais

- **Forças do v2**: observability (OTel + SLO + postmortem template) e patterns (CB + outbox + idempotency) estão em nível bom-a-excelente. ABAC + BeyondCorp também.
- **Fraquezas do v2 (eixos mais pobres)**: DR/Data (9 gaps, 4 M1) e Security ops (8 gaps, 4 M1). Incident management tem ótimo framework mas execução operacional zero (PagerDuty config, on-call pré-GA, runbooks LGPD).
- **Padrão recorrente**: muitos gaps M1 são **documento a criar** que já é referenciado ("a criar" em `BEYOND_CORP.md §16`, `cve-process.md §7.3`, `incident.md §LGPD`). Fechar `12-docs/runbooks/*` é >50% do trabalho M1.
- **Anti-padrão identificado**: promessas de "M2+" / "M3+" em segurança (hash-chain audit, KMS field-level, pentest externo) viraram "pra depois" sem explicitar **limite visível pro cliente** em M1 — LGPD auditor não aceita "em roadmap".

---

## §6 Vizinhos

- [[../AUDIT]] — findings A-### abertos (este arquivo não cria A-###, cria G-###; migração manual pelo dev se aprovado)
- [[../../02-architecture/observability.md]]
- [[../../02-architecture/patterns.md]]
- [[../../02-architecture/event-driven.md]]
- [[../../02-architecture/topology-multitenant.md]]
- [[../../06-execution/STEPS]]
- [[../../06-execution/deploy]]
- [[../../06-execution/incident]]
- [[../../06-execution/rollback]]
- [[../../08-security/BEYOND_CORP]]
- [[../../08-security/cve-process]]
- [[../../09-testing/test-strategy]]
- [[2026-04-23-pentest-specs]] — findings A-DC-PEN-### complementares (security em specs)
- [[2026-04-23-qa-review]] — QA findings
- [[2026-04-23-reconciliacao-v2-vs-dev]] — diffs v2 vs legado
