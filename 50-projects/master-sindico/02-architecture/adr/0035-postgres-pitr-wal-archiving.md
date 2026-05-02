---
title: ADR-0035 — PostgreSQL PITR via WAL archiving + DR drill mensal
type: adr
tags: [adr, master-sindico, v2, postgres, backup, dr, pitr, observability, sre]
source: G-015 + G-016 + G-017 consolidado 2026-04-23 + google-arch research (Google SRE ch.26)
created: 2026-04-23
updated: 2026-04-23
status: accepted
deciders: arquiteto, SRE, DPO
consulted: operations
---

# ADR-0035 — PostgreSQL PITR via WAL archiving + DR drill mensal

## Contexto

Estado atual (pré-Fase 9):

- `06-execution/deploy.md §pré-deploy` menciona "pg_dump em S3 pré-migration" — snapshot one-shot antes de migration, **não é PITR**.
- `06-execution/STEPS.md §6` tem 1 linha "backup Postgres trimestral" sem procedure/responsável.
- **RPO (Recovery Point Objective) não configurado**: em incidente de corruption/delete/ransomware, última janela de perda = desde último snapshot = **24h ou mais**.
- **RTO (Recovery Time Objective) não quantificado**: tempo de restore + validação desconhecido. Pode ser 4h ou 12h em produção real; não medido.
- **DR drill nunca realizado**. Google SRE ch.26: *"you only have a backup when you've restored it"*.

Para SaaS que roda assembleias diárias + billing recurrente + assinaturas legais (ata homologada) + timeline imutável regulatória:

- Perder 24h de atas = síndicos precisam refazer decisões; assembleias podem virar nulas.
- Perder 24h de Stripe subscriptions = reconciliação manual com Stripe API custosa.
- Perder 24h de audit log = violação Art. 37 LGPD (rastreabilidade).

Pesquisa big-tech (google-arch, netflix, AWS Well-Architected):
- RDS/CloudSQL fazem PITR nativo com WAL archiving contínuo + retenção 7-35 dias.
- Railway não tem PITR nativo; exige tooling externo (wal-g, pgBackRest, Barman).
- Netflix SRE impõe DR drill trimestral; Google DiRT = testes semi-anuais.
- Backup imutável offline (Object Lock) protege de ransomware em prod + backup online comprometidos.

## Decisão

### Parte A — WAL archiving contínuo com pgBackRest para S3/R2

Adotar **pgBackRest** como tooling de backup (vs wal-g: mais feature-rich, incremental delta, CLI mais madura; vs Barman: mais "on-prem", difícil em Railway).

Config mínima:

```
# postgresql.conf
archive_mode = on
archive_command = 'pgbackrest --stanza=ms-prod archive-push %p'
archive_timeout = 60  # força segment switch pelo menos a cada 60s

# pgBackRest stanza ms-prod
repo1-path=/var/lib/pgbackrest
repo1-type=s3
repo1-s3-endpoint=<cloudflare-r2-endpoint>
repo1-s3-bucket=ms-pg-backup
repo1-s3-region=auto
repo1-retention-full=14           # retém 14 backups full (2 semanas)
repo1-retention-diff=28           # retém 28 incrementais diff
repo1-cipher-type=aes-256-cbc
repo1-cipher-pass=<env>
```

Ritmo de backup:
- **Full backup**: 1× semana (domingo 03:00 UTC, off-peak BR).
- **Diff backup**: 1× dia (00:00 UTC).
- **WAL continuous**: auto via `archive_command`; ~60s de commits por segment.

### Parte B — RPO/RTO quantificados por BC

Tabela canônica paralela à tabela SLO em `observability.md`:

| BC | RPO | RTO | Justificativa |
|---|---|---|---|
| identity | 5 min | 15 min | Login quebrado = usuário fora; priorização máxima |
| billing | 5 min | 30 min | Perder evento Stripe = reconciliação manual complexa |
| assembly | 5 min | 30 min | Assembleia em curso = dado ao vivo; aceita re-sync de votos |
| institutional | 5 min | 1 h | Timeline/comunicados; perda tolerável pelo síndico |
| content (vídeos) | 1 h | 4 h | Upload pode ser retomado pelo usuário |
| commercial | 5 min | 1 h | Connect Me / Banco Talentos |
| cross-domain (audit) | 0 min | 15 min | Audit log **nunca pode perder** — streaming contínuo |
| global (plans) | 0 min | 5 min | Catálogo; mudança rara |

RPO de referência geral = **5 minutos** (alinhado com `archive_timeout=60`). Audit log tem RPO 0 via outbox pattern replicado.

### Parte C — Backup imutável offline (defense contra ransomware)

Segunda cópia dos backups full semanais em **bucket separado** com **Object Lock compliance mode + retenção 90 dias**:

- Bucket: `ms-pg-backup-immutable` (conta AWS separada ou R2 locker).
- IAM: role de escrita apenas; sem delete mesmo para root.
- Ransomware encripta prod + backup online; backup imutável offline sobrevive.
- Copy via cron `pgbackrest --stanza=ms-prod backup --type=full` + `aws s3 cp ... s3://ms-pg-backup-immutable/ --object-lock-mode COMPLIANCE --object-lock-retain-until-date +90d`.

### Parte D — DR drill mensal obrigatório

Runbook `06-execution/dr-drill.md` — procedure:

```
Cadência: 1× mês, toda 1ª terça-feira, 14:00 UTC-3.
Owner: on-call SRE (rotação).
Duração alvo: <2h.

Steps:
1. Escolher backup full mais recente + WAL window (24h até T-1h).
2. Spin up ms-staging-dr (ambiente isolado, Railway staging tier).
3. pgBackRest restore --type=time --target="YYYY-MM-DD HH:MM:SS".
4. Rodar smoke tests: login + billing + timeline + assembly.
5. Medir: tempo total (RTO observado) + último WAL restaurado (RPO observado).
6. Registrar em `11-audit/audit-log/drill-YYYY-MM.md` com tempo medido vs target.
7. Alerta se RPO > target ou RTO > target; postmortem.
8. Destruir ambiente staging-dr.
```

Se RPO ou RTO observado > target por 2 drills consecutivos → incident Sev2 + revisão ADR.

### Parte E — Reconciliação de providers pós-restore

Se restore resultou em perda de WAL (ex: backup de 5min atrás; WAL archive perdeu últimos 2min):

- Stripe: cron `stripe_reconcile` (Q-021) lista subscriptions com timestamp desalinhado + chama `IPaymentGateway.GetSubscription()` + update local.
- Mux: lista assets em Mux sem row em `videos` table; mark órfãos + retry webhook.
- Livekit: sessões em curso são invalidadas; assembleia reabre se estiver em andamento.

Runbook `06-execution/playbooks/post-restore-reconcile.md`.

## Alternativas consideradas

1. **wal-g em vez de pgBackRest** — wal-g é mais simples, single binary Go, integra direto com S3. Mas pgBackRest suporta diff incremental (vs wal-g só full) = backup 10x menor. pgBackRest vence por eficiência; wal-g = fallback se Railway complicar.
2. **Railway Postgres add-on PITR nativo** — se Railway anunciar até Sprint 10, adotar. Até lá: tooling externo.
3. **RDS managed backup** — mudaria de Railway pra AWS RDS; trade-off custo/ops. Mantida Railway em M1; re-avaliar M3+.
4. **Logical replication streaming pra segundo PG** — cobre HA mas não PITR (não restaura para ponto no tempo arbitrário). Complementar, não substituto.
5. **Só backup full diário, sem WAL** — RPO 24h. Rejeitado: inaceitável para assembleia / billing.

## Consequências

### Positivas

- RPO 5min vs 24h atual (288× melhor).
- RTO mensurado e provado via drill, não fictício.
- Ransomware em prod + backup online = backup imutável offline sobrevive (aceita perda 7d máx).
- Compliance LGPD Art. 37 + 48 (rastreabilidade + security incidents): restore de audit log íntegro.
- Alinhamento Google SRE ch.26 "tested backup = real backup".

### Negativas

- Custo: S3/R2 storage ~R$50/mês para ~100GB full + 2 semanas diff. Marginal.
- Overhead WAL archiving: ~3% aumento em IOPS write. Tolerável em Railway standard.
- DR drill mensal = ~2h engenheiro. Aceitável vs custo de perda de dados.
- pgBackRest primeira config: ~4h setup.

## Implementação

### Esta semana (pré-Sprint 10)

- Config `archive_mode`, `archive_command` no Postgres Railway.
- pgBackRest stanza configurada + primeiro full backup manual.
- Cron job: full semanal + diff diário + immutable copy.
- Tabela RPO/RTO em `observability.md`.
- Runbook `06-execution/dr-drill.md` criado.
- INV-123 (PITR + RPO/RTO).

### Sprint 10 (pré-M1)

- **Primeiro DR drill** executado e documentado.
- Runbook `06-execution/playbooks/post-restore-reconcile.md`.
- Alerting: falha `archive_command` por >5min = Sev1 PagerDuty.
- Smoke test suite pós-restore documentada.

### Pós-M1

- 2º drill validando (cadência estabelecida).
- Dashboard Grafana "DR health": último full backup, último WAL, tamanho stanza, expected RPO.

## Referências

- G-015, G-016, G-017 — [[../../11-audit/audit-log/2026-04-23-ops-bigtech-gaps]]
- Google SRE Book ch.26 — "Data Integrity: What You Really Want"
- AWS Well-Architected Pillar 5 — Reliability
- pgBackRest documentation — https://pgbackrest.org
- [[../observability]] — SLI/SLO por BC (paralela à tabela RPO/RTO)
- INV-123 (PITR + RPO/RTO) novo
- D-090 — STATE

## Vizinhos

- [[0020-opentelemetry-grafana-sentry]]
- [[0025-sli-slo-error-budget]]
- [[0033-ata-timeline-db-immutability]]
- [[../observability]]
- [[../../06-execution/STEPS]]
