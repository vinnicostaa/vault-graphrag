---
title: Rollback — Gatilhos + Procedimento
type: guide
tags: [execution, rollback, railway, master-sindico, v2]
source: vault/30-agents/playbooks/rollback + runbooks-rollback legado + STEPS
created: 2026-04-23
updated: 2026-04-23
---

# Rollback — Gatilhos + Procedimento

Rollback = reverter o ambiente (prod ou staging) pra versão anterior conhecida-boa. Preferível a hotfix precipitado.

---

## Gatilhos

### Automáticos (staging)

- Error rate > **0.5%** por 15 min consecutivos
- p99 > **1s** por 15 min consecutivos
- Webhook Stripe/Mux/Livekit falhando sistematicamente (> 5% fail rate por 1h)
- `/health/ready` respondendo 503 por > 2min

### Manuais (decisão humana)

- Incident em prod detectado (Sentry pico, reclamação em massa, SLO breach)
- Regressão funcional descoberta pós-deploy (teste de produção manual)
- Bug crítico bloqueando fluxo de negócio (ex: pagamento Stripe não processa)
- LGPD incident (breach) — pode forçar rollback se exfil ongoing
- Conflito de schema DB (se deploy aplicou migration incompatível)

---

## Decisão rollback vs hotfix

| Situação | Ação |
|---|---|
| Bug bloqueia > 10% usuários | **Rollback** imediato |
| Bug bloqueia < 10%, fix trivial | Hotfix em `main` + deploy rápido |
| Regressão em feature nova M2/M3 | Feature flag OFF; depois investigar |
| Incident de segurança (CVE reachable) | **Rollback** + patch emergencial |
| Migration DB incompatível | **Rollback** + recriar migration expand-contract |

---

## Procedimento — Railway

### 1. Avaliar escopo

```
1. Confirmar gatilho (métricas Grafana, Sentry, PagerDuty)
2. Time comms: canal #incident — alertar oncall + dev sênior + DPO (se LGPD)
3. Identificar deploy causador:
   railway runs list --environment prod
   git log --oneline main..origin/main
4. Decidir: rollback apenas backend? frontend? ambos?
```

### 2. Executar rollback

**Backend**:
```bash
railway runs list --environment prod
# identificar run anterior estável
railway runs promote <previous-run-id> --environment prod
```

**Frontend web**:
```bash
railway runs promote <previous-run-id> --environment prod --service web
```

**Mobile**: rollback é via app store (demora 4-24h)
- Não promover release-candidate TestFlight/Play pros canais externos
- Se já saiu: force-update banner via backend feature flag

### 3. Verificar

```bash
# /health/ready responde 200?
curl https://api.mastersindico.com.br/health/ready

# Smoke test crítico:
# Login síndico → dashboard → timeline
# (manual ou Playwright)
```

Métricas Grafana:
- Error rate voltou < 0.5%
- p99 < 1s

### 4. Comms + postmortem

- Canal #incident: atualizar status
- Time aviso: "Rollback executado pra <versão>; monitorando"
- **Postmortem blameless** em 48h ([[../11-audit/postmortems/template]])
- Se LGPD afetado: DPO ≤ 72h (ANPD notification)

### 5. Migration rollback

Se deploy aplicou migration **destrutiva** (DROP COLUMN/TABLE):
- **Não dá pra reverter simples** — restore do snapshot pgdump pré-deploy
- Decisão humana + comms cliente (downtime planejado)
- DB snapshot pré-deploy em S3 (checklist pré-deploy [[deploy#pré-deploy]])

Se migration foi **expand** apenas (nova coluna/tabela): rollback seguro (código antigo ignora).

---

## Prevenção

- Expand-contract em migrations (sem DROP no primeiro deploy)
- Feature flags pra features novas M2/M3
- Canary deploy (5% → 25% → 100%) em M3+
- Chaos tests em staging (Sprint 12+)
- 48h staging antes de promote prod (nunca furar)

---

## Métricas de rollback saudável

- TTR (time to rollback) alvo: **< 10 min**
- MTTR (mean time to recovery) alvo: **< 30 min**
- Rollback rate alvo: **< 1 por mês** (em regime de operação estável)

Acima disso: revisar processo (flake nas gates CI, deploy window, staging insuficiente).

---

## Links

- [[STEPS]]
- [[deploy]]
- [[incident]]
- [[../12-docs/runbooks/rollback-deploy]]
- [[../11-audit/postmortems/template]]
- [[../../vault/30-agents/playbooks/rollback]]
