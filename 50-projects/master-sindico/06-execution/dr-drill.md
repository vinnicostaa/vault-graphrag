---
title: DR Drill — Disaster Recovery mensal Master Síndico
type: playbook
tags:
  - playbook
  - dr
  - backup
  - pitr
  - postgres
  - master-sindico
  - v2
  - sre
source: G-017 consolidado 2026-04-23 + ADR-0035 + Google SRE ch.26
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-23T00:00:00.000Z
---

# DR Drill — Disaster Recovery mensal

Runbook obrigatório para validar backup + restore + RPO/RTO contra os targets declarados em [[../02-architecture/observability]] §3.3. **Google SRE ch.26**: *"you only have a backup when you've restored it"*.

## Cadência

- **1× mês**, 1ª terça-feira, 14:00 UTC-3 (cadência regular).
- **Owner**: on-call SRE da semana (rotação); aprovação do Tech Lead.
- **Duração alvo**: < 2h do início ao tear-down.
- **Ambiente**: `ms-staging-dr` — isolado, tier staging Railway, destruído pós-drill.

## ⚠️ Drill especial pré-M1 (2026-04-30) — NÃO PULAR

> Cadência regular cairia em 2026-05-05 (1ª terça-feira de maio), **depois** de M1 firme em 2026-05-18 mas dentro da janela de freeze. Para validar backup+restore **antes do go-live com Stripe Live e dados reais**, agendar um drill extra:
>
> - **Data**: 2026-04-30 (quinta), 14:00 UTC-3
> - **Owner**: SRE on-call + Tech Lead
> - **Critério de aceite M1**: RTO observado ≤ target + RPO ≤ 5min + 7/7 smoke tests
> - **Bloqueia M1 se falhar**: sim — sem backup validado, deploy de prod com cobrança real é negligente (Google SRE ch.26 / AWS Well-Architected Pillar 5)
> - **Log**: `[[../11-audit/audit-log/2026-04-30-dr-drill-pre-m1]]` (template já criado)
> - **Particularidades**: testar restore de **production-like data** (snapshot do staging populado com dados de teste em volume realista — ~15 condomínios × 100 unidades × 6 meses de timeline + 50 vídeos Mux + 5 assembleias Livekit)
> - **Repetir** caso 1ª execução falhe — ainda há buffer 7-12 dias até M1

## Pré-requisitos

- `pgBackRest` stanza `ms-prod` configurada e funcional (ADR-0035 §A).
- Acesso S3/R2 bucket `ms-pg-backup` (IAM role `dr-readonly`).
- Ambiente `ms-staging-dr` provisionável via `infra/staging-dr.tf` (Terraform).
- Runbook `post-restore-reconcile.md` disponível (playbooks/).
- Calendário Google: "DR Drill — MS" bloqueia 14:00-16:00 UTC-3.

## Procedure

### 1. Pré-voo (15 min)

- [ ] Comunicar no Slack `#ms-ops` início do drill.
- [ ] Confirmar último full backup em `pgbackrest --stanza=ms-prod info` → último timestamp < 7d.
- [ ] Confirmar WAL archive contínuo: `ls -la s3://ms-pg-backup/archive/ms-prod/ | tail -5` — segments recentes < 2min.
- [ ] Escolher target time: **T = NOW() - 1h** (restaura até 1h atrás, simula incidente com window de 1h).

### 2. Provisioning staging-dr (10 min)

- [ ] `terraform -chdir=infra/staging-dr apply` → spin up Postgres 18 + app Go + Redis isolados.
- [ ] Validar health: `curl https://staging-dr.mastersindico.com.br/health` retorna 503 (DB ainda vazio) — esperado.
- [ ] Anotar start timestamp do drill em `audit-log/drill-YYYY-MM.md`.

### 3. Restore (T_restore — medir RTO)

- [ ] `START_RESTORE=$(date -u +%s)` em variável.
- [ ] `pgbackrest --stanza=ms-prod --target-time="YYYY-MM-DD HH:MM:SS" --type=time restore` no DB staging-dr.
- [ ] Aguardar conclusão: `pg_isready` + query `SELECT 1 FROM users LIMIT 1`.
- [ ] `END_RESTORE=$(date -u +%s)` + calcular `RTO_obs = END_RESTORE - START_RESTORE`.
- [ ] **Target**: RTO_obs ≤ maior RTO da tabela (4h global; 30min identity/billing/assembly).

### 4. Medir RPO observado

- [ ] Query no banco restaurado: `SELECT max(created_at) FROM cross_domain.audit_log_entries;` → último evento restaurado.
- [ ] `RPO_obs = T_target - max(created_at)` em minutos.
- [ ] **Target**: RPO_obs ≤ 5 min por padrão; 0 min para cross-domain (audit streaming contínuo).

### 5. Smoke tests (golden paths)

- [ ] **Login**: `POST /auth/login` com usuário de teste → 200 + JWT.
- [ ] **Billing**: `GET /me/subscription` → retorna plan_tier válido.
- [ ] **Timeline**: `GET /condominiums/{id}/timeline?limit=10` → 10 entries com `published_at`.
- [ ] **Assembly**: `GET /assemblies?status=scheduled` → lista não vazia.
- [ ] **Integridade ata**: `SELECT count(*) FROM assembly.assembly_minutes WHERE status = 'homologated'` → matches prod count (tolerância ±5 devido ao target-time).
- [ ] **Integridade audit**: `SELECT count(*) FROM cross_domain.audit_log_entries WHERE created_at > T_target - INTERVAL '1h'` → esperado 0 (WAL fora do target-time).
- [ ] **RLS**: conectar como `app_user`; query sem `SET app.tenant_id` → 0 rows (validar RLS ativa).

### 6. Reconciliação simulada

- [ ] Rodar `post-restore-reconcile.md` mas em dry-run:
  - Simular reconcile Stripe: `stripe_reconcile --dry-run --since="1h ago"` → log de quantas subscriptions precisariam update.
  - Simular reconcile Mux: listar orphan assets sem row em `videos`.
- [ ] Se numbers > 10× esperado em dia normal → flag incident.

### 7. Registrar drill (`11-audit/audit-log/drill-YYYY-MM.md`)

```markdown
---
title: DR Drill YYYY-MM
type: audit-log
tags: [drill, dr, postgres, pitr]
created: YYYY-MM-DD
owner: <on-call SRE>
---

# DR Drill YYYY-MM

- **Data**: YYYY-MM-DD HH:MM UTC-3
- **Target time**: YYYY-MM-DD HH:MM UTC (T - 1h)
- **Último full backup**: YYYY-MM-DD HH:MM
- **RTO observado**: X min (target: Y min)  ✅|❌
- **RPO observado**: X min (target: 5 min)  ✅|❌
- **Smoke tests**: 7/7 passed
- **Reconcile dry-run**: Stripe <N>, Mux <N> (normal: < 10)
- **Incidents detected**: nenhum | <descrever>
- **Ações abertas**: nenhuma | <listar>
```

### 8. Tear-down (10 min)

- [ ] `terraform -chdir=infra/staging-dr destroy` → destrói ambiente.
- [ ] Confirmar S3 staging-dr bucket limpo.
- [ ] Slack `#ms-ops`: "DR drill YYYY-MM ✅|❌ — RTO X / RPO Y".
- [ ] Se ❌ (target não atingido): abrir incident Sev2 + notificar Tech Lead.

## Escalation

Se RPO ou RTO observado exceder target em **2 drills consecutivos**:

1. Incident Sev2 em `#ms-ops`.
2. Postmortem blameless ([[../02-architecture/observability]] §6).
3. Revisão ADR-0035 — parâmetros (retention, archive_timeout, tier Railway).
4. Considerar alternativas: RDS managed PITR / Wal-G / HA streaming replication.

## Métricas continuadas

- Grafana dashboard `DR health` (pós-M1):
  - Último full backup (gauge)
  - Último WAL archived (gauge)
  - Tamanho stanza (gauge)
  - Expected RPO vs medido (histograma de drills)
- Alert `archive_command fail > 5min` → Sev1 PagerDuty.

## Referências

- [[../02-architecture/adr/0035-postgres-pitr-wal-archiving]]
- [[../02-architecture/observability]] §3.3 RPO/RTO
- [[playbooks/post-restore-reconcile]]
- [[../11-audit/audit-log/_moc]] (drill-YYYY-MM.md registros)
- Google SRE Book ch.26
- AWS Well-Architected Pillar 5 Reliability

## Vizinhos

- [[STEPS]]
- [[deploy]]
- [[incident]]
- [[rollback]]
