---
title: Runbook — Rollback de Deploy (operacional)
type: guide
tags: [runbook, rollback, deploy, railway, master-sindico, v2]
source: 06-execution/rollback.md + playbook rollback + Railway docs
created: 2026-04-23
updated: 2026-04-23
---

# Runbook — Rollback de Deploy

> ⚠️ **ATUALIZAÇÃO 2026-04-27**: este runbook foi escrito para Railway (descontinuado por D-138/ADR-0045). Stack atual é **Cloudflare Pages + Workers + AWS ECS Fargate sa-east-1**. Comandos Railway abaixo são **HISTÓRICOS**. Para procedimento atual, ver [[../../06-execution/deploy-strategy]] §8 (rollback por camada).
>
> **TL;DR rollback novo**:
> - **CF Pages**: dashboard "Rollback to" OU `wrangler pages deployment list`
> - **CF Workers (BFF)**: `wrangler rollback --message "incident X"`
> - **AWS ECS**: `aws ecs update-service --cluster ms-prod --service backend --task-definition ms-backend:N-1`
> - **RDS migration destrutiva**: PITR restore (proibido sem aviso prévio + snapshot)
>
> Reescrita completa pendente — ver [[deploy-cicd-github-actions]] §7 (auto-rollback).

Procedimento operacional pra reverter deploy em staging ou produção (Railway — HISTÓRICO). Versão curta pra quem acordou às 3h da manhã com alerta.

Especificação estratégica: [[../../06-execution/rollback]].

---

## Quando rodar

### Gatilhos automáticos (staging)

- Error rate > 0.5% por 15min
- p99 > 1s por 15min
- Webhook fail > 5% por 1h
- `/health/ready` 503 por 2min

### Gatilhos manuais (produção)

Decisão humana. Exemplo:
- Incident P0/P1 logo após deploy
- Reclamação em massa
- Regressão funcional (ex: pagamento parou)
- LGPD breach detectado

---

## 10 passos (salvar pro celular)

1. **Acknowledge** PagerDuty
2. **Confirmar gatilho** via Grafana + Sentry (1 min)
3. Abrir `#incident` — status inicial
4. Identificar deploy ruim:
   ```
   railway runs list --environment prod --limit 10
   ```
5. Decidir escopo: backend / web / ambos
6. Executar rollback:
   ```
   railway runs promote <previous-run-id> --environment prod --service api
   ```
7. Esperar 2min, verificar `/health/ready`:
   ```
   curl https://api.mastersindico.com.br/health/ready
   ```
8. Smoke test crítico (login + dashboard)
9. Métricas voltaram? `#incident` UPDATE "Rollback executado"
10. Postmortem em 48h ([[../../11-audit/postmortems/template]])

---

## Comandos Railway

### Listar deploys

```bash
# últimos 10 deploys
railway runs list --environment prod --limit 10

# filter por service
railway runs list --environment prod --service api
```

Saída:
```
ID          TIMESTAMP           STATUS     COMMIT   TRIGGERED_BY
abc123...   2026-04-22 14:32    SUCCESS    a1b2c3   github-actions
xyz789...   2026-04-22 12:15    SUCCESS    d4e5f6   github-actions  ← PREVIOUS
```

### Promote versão anterior

```bash
# backend
railway runs promote xyz789 --environment prod --service api

# web
railway runs promote <run-id> --environment prod --service web

# verificar em tempo real
railway logs --environment prod --service api --tail
```

### Cancelar deploy em andamento

Se rollback for chamado durante deploy em curso:
```bash
railway runs cancel <running-run-id>
# então promote versão estável anterior
```

---

## Migration rollback

### Se migration foi **expand** (nova coluna/tabela, backward-compat)

Rollback seguro — código antigo ignora coluna nova:

```bash
railway runs promote <previous> --environment prod
# sem ação no DB
```

### Se migration foi **destrutiva** (DROP COLUMN/TABLE)

**PARE**. Rollback de código **não** restaura dados. Precisa:

1. Comms cliente pra downtime (se ainda não estão vendo):
   ```
   [MAINTENANCE] Serviço em manutenção de emergência. Voltamos em ~30min.
   ```
2. Restore pgdump pré-deploy:
   ```bash
   aws s3 ls s3://mastersindico-backups/prod/ | head -5
   aws s3 cp s3://mastersindico-backups/prod/<YYYY-MM-DD-pre-deploy>.sql.gz .
   
   # ler DATABASE_URL prod
   source .env.prod  # cuidado: só oncall senior
   
   # restaurar
   gunzip -c <YYYY-MM-DD-pre-deploy>.sql.gz | psql $DATABASE_URL_PROD
   ```
3. Rollback código:
   ```bash
   railway runs promote <previous> --environment prod
   ```
4. Reabrir serviço — status page UPDATE
5. Postmortem **crítico** — migration destrutiva violou expand-contract. T-### no backlog pra educação + processo.

---

## Frontend web

Web não tem schema DB; rollback = promover run anterior:
```bash
railway runs promote <run-id> --environment prod --service web
```

Cache browser: Vite gera hash por asset, staleness mínimo. Usuários veem versão nova ao refresh.

---

## Mobile

Rollback store é lento:
- **iOS TestFlight**: revogar build promovido (horas)
- **App Store**: expedited review (horas-dias)
- **Google Play**: rollback via staged rollout (se < 100% rollout)
- **Android Play (100% rollout)**: novo release com versão antiga

**Workaround imediato**: feature flag OFF via backend. Apps antigos com flag OFF comportam-se como versão N-1.

---

## Pós-rollback — validação

- [ ] `/health/ready` 200
- [ ] Error rate < 0.5% (Grafana, 5min sustained)
- [ ] p99 < 1s
- [ ] Sentry — errors param de aparecer
- [ ] Smoke test manual OK
- [ ] Webhooks voltando a processar (DLQ drenando)
- [ ] Stripe payments success rate ≥ 99%

Se métricas não voltarem em 15min pós-rollback: **escalate** — dev sênior chama arquiteto + CEO (se prod) + considerar outra estratégia.

---

## Comms template

### Início

```
[INCIDENT] Sev P1 — <título>
Detectado: <HH:MM UTC> via PagerDuty
Sintoma: Error rate 3% em /api/v1/payments/checkout
Impacto estimado: 5% usuários tentando pagar
Ação: Rollback em andamento (promove run xyz789)
ETA: ~5min
Oncall: @<nome>
```

### Em andamento

```
[UPDATE 14:40 UTC]
Rollback executado. Error rate baixando: 3% → 1% → 0.4%.
Smoke test OK. Monitorando.
```

### Resolved

```
[RESOLVED 14:45 UTC]
Rollback concluído. Sistema estável.
Postmortem agendado para 2026-04-24 (48h).
```

---

## Checklist pós-incident

- [ ] `#incident` marcado RESOLVED
- [ ] Status page: "Operational"
- [ ] Postmortem agendado (48h)
- [ ] Branch de debug criada (investigar fora de prod)
- [ ] Hotfix testado em staging antes de re-deploy
- [ ] Comms cliente se pública (email + status page update)
- [ ] DPO notificado se LGPD afetado
- [ ] Stakeholders internos (PO, CEO se M1+) cientes

---

## Anti-padrões

- ❌ Hotfix direto em prod sem rollback primeiro ("conserto rápido")
- ❌ Promote versão muito antiga (1+ semanas; arrisca incompat migrations)
- ❌ Rollback sem comms — deixa time + cliente no escuro
- ❌ Skippar postmortem pra "economizar tempo"
- ❌ Culpar dev específico — foco em sistema/processo

---

## Links

- [[_moc]]
- [[webhook-reprocessing]]
- [[incident-lgpd-breach]]
- [[../../06-execution/rollback]]
- [[../../06-execution/playbooks/rollback]]
- [[../../06-execution/incident]]
- [[../../11-audit/postmortems/template]]
