---
title: Deploy — Staging contínuo + Produção por marco
type: guide
tags: [execution, deploy, staging, prod, railway, master-sindico, v2]
source: legado 06-execution + STEPS.md + runbooks-rollback
created: 2026-04-23
updated: 2026-04-23
---

# Deploy — Staging contínuo + Produção por marco

Estratégia: **staging contínuo** (toda PR merged → deploy staging automático) + **produção por marco** (M1/M2/M3, com freeze 48h e promote manual).

---

## 1. Staging (contínuo)

### Trigger

```
PR merged em `main` → GitHub Actions → Railway staging (auto)
```

### Pipeline

1. **Test**: `go test -race` + `go test -tags=integration` + `gosec` + `govulncheck`
2. **Build**: multi-stage Dockerfile (backend, web, mobile CI)
3. **Deploy**: Railway API push; zero-downtime rolling deploy
4. **Migration**: goose `up` (backend) — expand-contract mandatory (sem DROP COLUMN/TABLE on deploy)
5. **Smoke test**: Playwright em 5 fluxos críticos
   - Login síndico → dashboard
   - Cria condomínio → adiciona unidade
   - Publica timeline entry → aparece feed
   - Publica RFP Connect Me → empresa recebe (M2+)
   - Abre assembleia + check-in → vota (M3+)
6. **Monitoring window**: 24h OTel + Sentry

### Monitoramento (24h pós-deploy staging)

- Error rate: **< 0.5%** (threshold)
- p95 API: **< 500ms**
- p99 API: **< 2s**
- Webhook Stripe/Mux/Livekit: recebidos sem erro
- Sentry: sem errors novos em rotas críticas

### Rollback automático

Trigger:
- Error rate > **0.5%** por 15min consecutivos
- p99 > **1s** por 15min consecutivos
- Webhook Stripe/Mux/Livekit falhando sistematicamente

Ação: Railway `promote <versão anterior>`; alerta time via PagerDuty.

Ver [[rollback]].

---

## 2. Produção (por marco)

### Pré-deploy (48h antes)

**Checklist go/no-go**:

- [ ] [[../11-audit/AUDIT]] sem 🔴; 🟡 com plano documentado
- [ ] [[../STATE]] sem bloqueador (B-###)
- [ ] Coverage global ≥ 85%
- [ ] Gates CI verdes (build / vet / test -race / integration / gosec -sev high / govulncheck)
- [ ] Staging 48h estável (todas métricas dentro)
- [ ] Smoke test Playwright passing
- [ ] k6 load test passing (1000+ concurrent em rotas críticas)
- [ ] Chaos test passing (Saga Mux/Livekit)
- [ ] CHANGELOG atualizado com todas tasks do marco
- [ ] Release Notes redigidas
- [ ] DB snapshot pré-migration (pgdump em S3)
- [ ] Feature flags criadas (rollout progressivo se necessário)
- [ ] Runbooks revisados (rollback, incident, webhook-reprocessing)
- [ ] Oncall notificado (canal time + PagerDuty schedule)
- [ ] DPO ciente se mudança afeta LGPD
- [ ] **Code freeze**: 48h pré-deploy (ninguém merge em `main`)

### Deploy

```
1. Merge última PR de release em `main` + tag `v<marco>.0.0` (ex: v1.0.0 pra M1)
2. GitHub Actions deploy pipeline promote staging → prod
   - Railway CLI: `railway up --environment prod`
3. /health/ready verifica PG + Redis + Zitadel + Stripe + Mux + Livekit reachable
4. Smoke test prod:
   - GET /health → 200
   - GET /api/v1/auth/me (auth'd) → 200
   - Endpoint crítico por sub-produto
5. Canary (se feature flag): 5% → 25% → 50% → 100% em 4h
```

Backend, web, mobile CI publicam simultaneamente (ou em ordem que respeita backward-compat: backend → web → mobile).

### Janela de monitoramento pós-deploy (48h)

Métricas críticas (dashboards Grafana):
- **Uptime** SLO: 99.5% (M1-M3), 99.9% (pós-M3)
- **Error rate** global: < 0.1%
- **p95 API**: < 500ms
- **p99 API**: < 2s
- **Webhook delivery rate**: ≥ 99% (Stripe, Mux, Livekit)
- **Sentry**: < 10 errors novos/5min
- **Stripe payment success rate**: ≥ 99%
- **Mux asset processing**: no stuck assets > 10min
- **Livekit room health**: participants join rate > 95%

Alertas PagerDuty:
- Error rate > 0.5% (15min) → P1
- p99 > 1s (15min) → P2
- Stripe webhook fail > 5% (1h) → P1
- LGPD: 72h countdown de breach notification se incident → P0

### Critérios de sucesso por marco

**M1 (2026-05-08)**:
- Síndico cria conta → escolhe plano → trial ativo → paga → recorrência Stripe real
- Condomínio criado + unidades + timeline funcionando
- Mobile auth + home

**M2 (2026-06-20)**:
- Connect Me 4 fluxos end-to-end
- Reputação dashboard empresa
- Vídeos upload + moderação 48h + lock 90d
- Vizinhança cupons 4h
- OpenSearch prod respondendo ≤ 200ms

**M3 (2026-07-20)**:
- Assembleia live com 100+ participantes simultâneos
- Votação real-time WebSocket < 200ms
- Ata imutável + homologação
- LMS thin (5 áreas, 3 trilhas, certificados)
- a11y WCAG 2.1 AA em fluxos críticos
- SLO 99.5% últimos 7 dias

---

## 3. Migrations — expand-contract

Toda mudança de schema segue expand-contract pra evitar downtime:

1. **Expand**: adicionar nova coluna/tabela, deploy, código usa ambas (antiga + nova)
2. **Migrate data**: backfill se necessário (job ou migration)
3. **Code cutover**: atualizar código pra usar só nova
4. **Contract**: remover coluna antiga (em deploy seguinte)

**Proibido**: `DROP TABLE` / `DROP COLUMN` on primeiro deploy da mudança. Pausa modo autônomo se tentar.

---

## 4. Secrets — nunca no código

- `.env` gitignored
- Segredos em Railway env vars ou AWS Secrets Manager
- Rotação trimestral (Stripe, Zitadel, Twilio, Mux, Livekit, Resend)
- CI usa `secrets.*` do GitHub Actions

---

## 5. DNS + domínios

- `app.mastersindico.com.br` → web SolidJS (Railway)
- `api.mastersindico.com.br` → backend Go (Railway)
- `auth.mastersindico.com.br` → Zitadel (Railway, 9.19)
- `cdn.mastersindico.com.br` → Mux playback
- Cookie domain: `.mastersindico.com.br` (wildcard; todos subdomínios herdam sessão)

---

## Links

- [[STEPS]]
- [[dev]]
- [[qa]]
- [[rollback]]
- [[incident]]
- [[../12-docs/runbooks/rollback-deploy]]
- [[../12-docs/runbooks/webhook-reprocessing]]
- [[../10-schedule/cronograma-macro]]
- [[../10-schedule/milestones]]
