---
title: Non-Functional Requirements — Master Síndico v2
type: requirement
tags:
  - requirements
  - non-functional
  - nfr
  - performance
  - availability
  - security
  - master-sindico
  - v2
source: >-
  legado CLAUDE.md + STEERING + novo escopo-7 + _chaos/requirements(6).md
  (2026-04-13)
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
fase_c_absorvido: true
---

> **Banner Fase C (2026-04-25)**: cobertura R71-R92 do `_chaos/requirements(6).md`:
> - R71 (Encryption at Rest), R72 (Encryption in Transit) — cobertos NFR-020 (zero-trust) + STEERING §10.
> - R73 (Input validation), R74 (Rate limiting/DDoS) — cobertos NFR-021 + IDN-026.
> - R75 (Session mgmt), R76 (2FA) — cobertos IDN-007/IDN-009/IDN-038.
> - R77 (Caching), R78 (Query opt), R79 (Async), R80 (Event-driven) — cobertos NFR-001..004 + XD-001..002.
> - R81-R85 (Logging, errors, performance, health, metrics) — cobertos NFR-025..028.
> - R86-R87 (Testing, PBT) — cobertos NFR-013..014.
> - R88-R90 (CI/CD, env, migrations) — cobertos STEERING §12.
> - R91 (WCAG) — coberto NFR-029.
> - R92 (i18n) — coberto NFR-030.
> - **R95 Voice Input** (acessibilidade) → CNT-038 (M3+ backlog) — referência cruzada.
> - **D-135 device fingerprint progressivo** → IDN-026 (M1) + DPIA-001 (a criar).
> Cobertura no-op: 100% das NFRs já presentes. Únicos deltas absorvidos: cross-link a IDN-026 e CNT-038.

# Non-Functional Requirements — Master Síndico v2

Requisitos **não-funcionais** (qualidades do sistema). Cobrem performance, disponibilidade, escalabilidade, manutenibilidade, portabilidade, segurança, observabilidade, acessibilidade, i18n, compliance.

Formato: `NFR-###` + métrica + threshold + rationale + critério de teste.

> Destilado de [[STEERING]] §6-36 + `../90-ingested/from-vault-50-projects-master-sindico/CLAUDE.md` §8-9 + `novo escopo-7.md` cronograma.

---

## Performance

### NFR-001 — Latência API (REST)

**Métrica**: P95 end-to-end para rotas `/api/v1/*` (excluindo upload direto).  
**Threshold**:
- P50 ≤ 150ms.
- P95 ≤ 500ms.
- P99 ≤ 1500ms.

**Rationale**: UX de dashboard, Connect Me, timeline requer sensação imediata. Acima de 1s gera abandono.

**Teste**: k6/grafana benchmark em staging com 500 RPS sintético por 10 min.

**Fonte**: [[STEERING]] §26, practice big techs.

---

### NFR-002 — Latência WebSocket (assembleia real-time)

**Métrica**: P95 round-trip pra eventos de check-in, vote, speech queue.  
**Threshold**: P95 ≤ 200ms.

**Rationale**: Votação live exige percepção de sincronismo. Acima de 200ms degrada confiança.

**Teste**: simulação de 100 participantes concorrentes em staging.

**Fonte**: legado `assembly.md` Req 57, Req 60.3, NFR crítico.

---

### NFR-003 — Throughput mínimo

**Métrica**: Requisições por segundo sustentáveis sem degradação.  
**Threshold**:
- M1 (base contratual): 100 RPS sustained, 500 RPS burst.
- M3: 500 RPS sustained, 2000 RPS burst.
- M4+: ≥ 5000 RPS.

**Rationale**: M1 atende 1k condomínios × 100 moradores média × picos de assembleia.

**Teste**: load test antes de cada marco.

**Fonte**: [[00-product/portfolio-de-produtos]] escalabilidade.

---

### NFR-004 — Latência query DB

**Métrica**: P95 tempo de execução de query SQL.  
**Threshold**:
- Query simples (index lookup): P95 ≤ 10ms.
- Query agregada: P95 ≤ 100ms.
- Relatórios complexos: assíncronos (job background).

**Teste**: `pg_stat_statements` monitoramento contínuo + Grafana dashboard.

**Fonte**: boas práticas PostgreSQL 18.

---

## Disponibilidade e resiliência

### NFR-005 — SLO de disponibilidade

**Métrica**: Uptime mensal.  
**Threshold**:
- M1 (MVP): 99% (≈7h downtime/mês aceitável).
- M3: 99.5% (≈3.6h/mês).
- M4+: 99.9% (≈43 min/mês).

**Rationale**: Master Síndico é plataforma de governança — downtime durante assembleia é inaceitável mas fora dela é tolerável em MVP.

**Fonte**: [[STEERING]] §26.

---

### NFR-006 — Recuperação de crash (RTO)

**Métrica**: Tempo pra voltar operacional após crash.  
**Threshold**:
- Backend stateless: RTO ≤ 1 min (health check + restart).
- DB PostgreSQL: RTO ≤ 15 min (Railway managed); ≤ 5 min (ECS M4+).
- Assembleia live em progress: state Redis → recuperação < 1 min (ASM-031).

**Fonte**: legado `assembly.md` Req 60.10.

---

### NFR-007 — RPO (data loss)

**Métrica**: Janela máxima de perda de dados em catástrofe.  
**Threshold**: ≤ 1h (backup diário + WAL archiving).

**Rationale**: Transações financeiras (Stripe) não podem perder; WAL + replica fornece RPO < 1min.

**Fonte**: boas práticas.

---

### NFR-008 — Backup + restore test

**Métrica**: Frequência de teste de restore.  
**Threshold**: 1x/mês restore em staging a partir de backup produção.

**Fonte**: SRE practice.

---

### NFR-009 — Circuit breakers em providers externos

**Métrica**: Fallback graceful quando Stripe/Mux/Livekit falham.  
**Threshold**:
- Timeout: 10s em chamadas síncronas.
- Retry: 3x com backoff exponencial.
- Circuit break: abre após 5 falhas em 30s; reabre em 60s.

**Fonte**: [[STEERING]] §10.

---

## Escalabilidade

### NFR-010 — Multi-tenant até 10k condomínios (M3)

**Métrica**: Condomínios suportados.  
**Threshold**:
- M1: 100 condomínios.
- M2: 1.000.
- M3: 10.000.
- M4+: 100.000.

**Rationale**: cada condomínio ≈ 50 unidades × 2 pessoas = 100 users → M3 suporta 1M users.

**Fonte**: [[00-product/portfolio-de-produtos]].

---

### NFR-011 — Horizontal scaling

**Métrica**: Backend stateless, scalable horizontalmente.  
**Threshold**: Target 100% CPU → auto-scale +1 instância ≤ 2 min.

**Critério**: Railway/ECS auto-scaling configurado.

**Fonte**: Cloud-native practice.

---

### NFR-012 — Queue depth (NATS)

**Métrica**: Depth de eventos pendentes em NATS JetStream.  
**Threshold**: ≤ 1000 msgs/stream sustentável; alerta > 5000.

**Fonte**: [[STEERING]] §14.

---

## Manutenibilidade

### NFR-013 — Coverage thresholds

**Métrica**: Code coverage por camada.  
**Threshold** (por [[STEERING]] §25):
- `domain/`: ≥ 95%.
- `application/`: ≥ 90%.
- `infrastructure/database/`: ≥ 85%.
- `infrastructure/http/`: ≥ 75%.
- `shared/middleware/`: ≥ 90%.
- `core/`, `pkg/`: ≥ 95%.
- **Global**: ≥ 85%.

**Teste**: CI gate via `go test -coverprofile`.

**Fonte**: [[STEERING]] §25, GR-022.

---

### NFR-014 — Property-based testing em invariantes

**Métrica**: PBT cobre invariantes críticas.  
**Threshold**: Obrigatório em: fração ideal (INS-005, ASM-015), quotas (BIL-016/017), ABAC matrix (XD-004), vote único (ASM-015), pagination (GR-016).

**Critério**: Lib `gopter` ou `testing/quick`. 1k iterações.

**Fonte**: [[STEERING]] §28.

---

### NFR-015 — Documentação viva

**Métrica**: ADRs + OpenAPI + CHANGELOG sincronizados.  
**Threshold**:
- Toda decisão arquitetural → ADR em `02-architecture/adr/ADR-####-<slug>.md`.
- Toda rota → OpenAPI 3.1 spec antes do handler (GR-034).
- CHANGELOG atualizado em cada release.

**Fonte**: [[STEERING]] §34, §13.

---

### NFR-016 — Build time

**Métrica**: `go build ./...` de cold.  
**Threshold**: ≤ 90s em CI; ≤ 30s em dev (cache).

**Fonte**: dev experience.

---

### NFR-017 — Test time

**Métrica**: Unit tests suite completa.  
**Threshold**:
- Unit: ≤ 2 min.
- Integration (`-tags=integration`): ≤ 10 min.

**Fonte**: GSD.

---

## Portabilidade

### NFR-018 — Railway → AWS ECS migration readiness

**Prioridade**: M (planejamento) / C (execução M4+)  
**Métrica**: Infra portable via IaC.  
**Threshold**:
- Dockerfile multi-arch (amd64 + arm64).
- Env-driven config (12-factor).
- Health checks standard.
- Providers via interfaces (XD-012+).

**Fonte**: legado D-024.

---

### NFR-019 — Stack portabilidade

**Métrica**: Backend Go 1.26+ rodável em Linux/macOS dev, Linux prod.  
**Threshold**:
- Frontend: SolidJS + TS build estático (qualquer CDN).
- Mobile: Flutter build iOS + Android.

**Fonte**: [[../../03-subprojects/_moc]] (a destilar).

---

## Segurança

### NFR-020 — Zero-trust BeyondCorp

**Métrica**: Toda request auth + autz.  
**Threshold**: 0 rotas protegidas sem middleware Authenticate + RequirePermission (exceto `/auth/*`, `/webhooks/*`, health checks).

**Teste**: audit automático de routes.

**Fonte**: [[STEERING]] §15.

---

### NFR-021 — Gosec + govulncheck zero findings

**Métrica**: Pipeline security.  
**Threshold**:
- `gosec -severity=high ./...` → 0 findings.
- `govulncheck ./...` → 0 CVEs em reachable code.

**Fonte**: [[STEERING]] §26.

---

### NFR-022 — Penetration testing

**Métrica**: Pentest externo pré-M1 e pré-M3.  
**Threshold**: Pentest OWASP Top 10 por 3º partido; 0 críticos/high antes de production.

**Fonte**: [[STEERING]] §5 protocol pentester.

---

### NFR-023 — Secret rotation

**Métrica**: Secrets rotacionados regularmente.  
**Threshold**:
- Stripe keys: anual.
- Mux tokens: anual.
- JWT signing keys (futuro): trimestral.

**Fonte**: best practice security.

---

### NFR-024 — MFA para admin

**Prioridade**: M (M2)  
**Métrica**: Admin login sempre com MFA.  
**Threshold**: 100% admins com MFA ativo.

**Dependências**: IDN-007.

**Fonte**: [[00-product/personas]] Admin.

---

## Observabilidade

### NFR-025 — Logs estruturados 100% das requests

**Métrica**: Cobertura de logs.  
**Threshold**: 100% das requests (exceto health checks) geram log `{request_id, user_id, tenant_id, route, status, duration_ms}`.

**Fonte**: GR-011.

---

### NFR-026 — Traces OTel em hot paths

**Métrica**: Cobertura de traces.  
**Threshold**:
- Use cases: 100% tracados.
- Queries SQL: 100% tracadas.
- Chamadas externas (Stripe, Mux): 100% tracadas.

**Fonte**: GR-011.

---

### NFR-027 — Métricas Grafana/Prometheus

**Métrica**: Dashboards publicados.  
**Threshold**:
- Latência P50/P95/P99 por rota.
- Taxa erro por rota.
- Throughput.
- Domain events counters.
- Queue depth NATS.

**Fonte**: legado sprint 7.

---

### NFR-028 — Alerting

**Métrica**: SLO alerts.  
**Threshold**:
- P95 > 1s por 5 min → alert.
- Error rate > 1% por 5 min → alert.
- DB connection pool > 80% → alert.
- Webhook failure rate > 5% → alert.

**Fonte**: SRE practice.

---

## Acessibilidade

### NFR-029 — WCAG 2.1 AA

**Métrica**: A11y score.  
**Threshold**:
- Lighthouse a11y ≥ 90 em páginas chave (login, timeline, assembleia, checkout).
- axe-core 0 violations críticas.
- Contraste ≥ 4.5:1.
- Keyboard navigation completa.
- Screen reader testado (NVDA ou VoiceOver).

**Fonte**: GR-013, [[STEERING]] §26.

---

## Internacionalização

### NFR-030 — pt-BR primário

**Métrica**: Cobertura i18n.  
**Threshold**:
- 100% user-facing strings em pt-BR (backend + frontend).
- Keys extraídos; EN default `{}` pra M3+.
- Datas: ISO 8601 com offset; display `America/Sao_Paulo`.

**Fonte**: GR-010, [[STEERING]] §3.

---

## Compliance

### NFR-031 — LGPD compliance

**Métrica**: Controles LGPD implementados.  
**Threshold**: ver [[compliance-lgpd]] — consent versionado, exclusão SLA 15d, portabilidade, audit trail 5 anos, breach 72h, DPA com terceiros.

**Fonte**: [[compliance-lgpd]], GR-012.

---

### NFR-032 — DPA com terceiros

**Métrica**: Data Processing Agreement assinado.  
**Threshold**: 100% fornecedores críticos (Stripe, Mux, Zitadel, Livekit, AWS) com DPA.

**Fonte**: LGPD Art. 39.

---

### NFR-033 — Retention policies aplicadas

**Métrica**: Política de retenção + purge.  
**Threshold**:
- `audit_trail`: 5 anos.
- `lgpd_logs`: 5 anos.
- Backup: 1 ano.
- Vídeos arquivados: 7 anos após baixa.

**Fonte**: GR-026, LGPD.

---

## Experiência do desenvolvedor (DX)

### NFR-034 — Setup local ≤ 30min

**Métrica**: Tempo pra dev rodar projeto local.  
**Threshold**: Clone → `make dev` → aplicação no ar em ≤ 30 min (com Docker instalado).

**Fonte**: onboarding.

---

### NFR-035 — CI pipeline ≤ 15min

**Métrica**: Tempo total CI por PR.  
**Threshold**: ≤ 15 min (build + test unit + integration + lint + security).

**Fonte**: GSD, shipping rápido.

---

## Data classification

### NFR-036 — Data classification

**Métrica**: Classificação de dados por criticidade.  
**Threshold** (ver [[compliance-lgpd]]):
- **CRITICAL**: CPF, CNPJ, dados financeiros (Stripe), votos.
- **HIGH**: email, telefone, endereço, vídeos privados.
- **NORMAL**: timeline pública, vídeos institucionais aprovados, score governança.

Controles diferentes por classe (encryption at rest, acesso logado, retenção).

**Fonte**: [[compliance-lgpd]].

---

## Conformidade setor

### NFR-037 — Lei do Condomínio (BR)

**Métrica**: Aderência à Lei 4.591/1964 + Código Civil (arts. 1.331-1.358).  
**Threshold**: Quórum fracional (GR-033), assinatura digital em ata (Lei 14.063), edital.

**Fonte**: legislação BR.

---

### NFR-038 — Assinatura digital BR (MP 2.200-2 / Lei 14.063)

**Prioridade**: C (M3+)  
**Métrica**: Valor jurídico de documentos.  
**Threshold**: ICP-Brasil ou assinatura avançada (Lei 14.063/2020) em ata, edital, certificados LMS.

**Fonte**: [[00-product/portfolio-de-produtos]] Produto 5.

---

## Pendências

- ⚠️ **PENDÊNCIA NFR-002**: teste de latência WebSocket em cenário real (250 participantes) — validar Livekit cap M3.
- ⚠️ **PENDÊNCIA NFR-010**: cálculo exato de user × condo × assembleia concorrente na infra Railway (vs ECS) — ADR M3.
- ⚠️ **PENDÊNCIA NFR-022**: escolher fornecedor de pentest — cotar M2/M3.
- ⚠️ **PENDÊNCIA NFR-038**: provider BR de assinatura digital (Certisign? Serasa? Clicksign?) — dual-check.

---

## Referências

- Legado: `CLAUDE.md` §8-9, `novo escopo-7.md` cronograma.
- [[STEERING]] §6-36.
- [[global]] — GRs relacionados.
- [[compliance-lgpd]] — detalhamento LGPD.
- [[../00-product/portfolio-de-produtos]] — escala por marco.
- [[_moc]]
