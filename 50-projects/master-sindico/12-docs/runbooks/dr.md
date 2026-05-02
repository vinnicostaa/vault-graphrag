---
title: Runbook — Disaster Recovery (DR/BCP)
type: guide
tags: [runbook, dr, bcp, backup, postgres, redis, opensearch, lgpd, master-sindico, v2]
source: 08-security/lgpd.md + 02-architecture/topology-multitenant.md + ADR-0017 (Railway) + ADR-0009 (PG 18) + ADR-0018 (OpenSearch) + ADR-0010 (Redis)
created: 2026-04-23
updated: 2026-04-23
---

# Runbook — Disaster Recovery (DR/BCP)

Recuperação de incident **catastrófico** — perda de banco de dados, corrupção de cluster, região indisponível, ransomware, ou incident que exija **restore de backup completo**.

Este runbook cobre **recuperação técnica**. Incidents LGPD (exposição PII) usam [[incident-lgpd-breach]] em paralelo. Rollback de deploy simples usa [[rollback-deploy]].

> **Regra**: DR é mais severo que rollback. Se não sabe qual usar — chame DPO + arquiteto antes de qualquer comando destrutivo.

---

## 1. Objetivos (RTO/RPO)

| Tier | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) | Exemplos |
|---|---|---|---|
| **Crítico** (tier 1) | **4 horas** | **1 hora** | API backend, Postgres, Zitadel, Stripe webhook receiver, assemblies live em curso |
| **Não-crítico** (tier 2) | **24 horas** | **24 horas** | OpenSearch (busca degradada via PG tsvector), Grafana, Sentry dashboards, backoffice de relatórios |

**Racional RTO 4h crítico**: M1 tem SLO 99.5% (ADR-0025) = 3.6h/mês error budget; 4h single-event já queima budget mensal mas é realista pra early-stage solo-dev (não-multi-AZ Railway; compra ordenada de restore + teste smoke).

**Racional RPO 1h crítico**: WAL archive incremental hourly + full daily. Perder < 1h de writes é aceitável pra governança condominial (não é trading). Exige notificação LGPD Art. 48 se PII foi escrita na janela de perda e não replicada (ver §5).

---

## 2. Gatilhos

### Técnicos
- Cluster Postgres Railway **INDISPONÍVEL** > 30min (sem ETA de provider)
- **Corrupção** detectada (`pg_amcheck` reporta dano em tabela crítica)
- **Ransomware** — criptografia externa de backup S3 ou host Railway
- **Região Railway** down com SLA esgotado (status.railway.app)
- Incident de **delete destrutivo** (DROP TABLE/DATABASE executado em prod por engano) sem outro caminho de restore

### Organizacionais
- Decisão CEO/DPO: ativar DR em incident público que exige evidência forense isolada
- Ordem judicial de isolamento do ambiente vivo (preservar estado para perícia)

---

## 3. Estratégia de backup

### 3.1 Postgres (crítico)

| Tipo | Cadência | Retenção | Localização | Verificação |
|---|---|---|---|---|
| **WAL archive** (incremental) | contínuo via `pg_receivewal` → S3 | 30 dias | S3 `s3://mastersindico-backups/wal/` (AWS `us-east-1` fora de Railway) | checksum por arquivo WAL |
| **pg_dump full** | diário 03:00 BRT | 90 dias (últimos 30 + semanal × 8 + mensal × 12) | S3 `s3://mastersindico-backups/full/<YYYY-MM-DD>.sql.gz` | restore smoke test semanal automático em container efêmero |
| **Snapshot Railway** | diário (managed) | 7 dias | Railway managed | — (complementar, não substitui S3 próprio) |

**Invariante**: todo backup S3 usa **Object Lock / Versioning** (WORM) — ransomware não consegue apagar/sobrescrever. Chave KMS separada, não compartilhada com infra Railway.

**Alerta**: Grafana monitora `backup_age_seconds` — se > 3600s (WAL) ou > 30h (full), dispara PagerDuty.

### 3.2 Redis (crítico degradado)

| Tipo | Cadência | Retenção | Localização |
|---|---|---|---|
| **RDB snapshot** | a cada 6h + before shutdown | 48h | S3 `s3://mastersindico-backups/redis/` |

**Escopo**: Redis armazena cache ABAC (TTL 5min) + rate limit counters + webhook idempotency cache (ADR-0027: DB é source of truth). **Perda total = degradação aceitável** — cache rehydrates, rate limit reseta (ok), idempotency já está em DB.

**Consequência**: RPO de Redis é "best-effort 6h" mas não bloqueia DR — Redis é recuperável via warm-up.

### 3.3 OpenSearch (não-crítico)

| Tipo | Cadência | Retenção | Localização |
|---|---|---|---|
| **Snapshot** index-based | diário 04:00 BRT | 14 dias | S3 `s3://mastersindico-backups/opensearch/` |

**Escopo**: busca de empresas / vídeos / conteúdo. Fallback em DR: **PG tsvector** (ADR-0018) cobre busca degradada durante restore OpenSearch.

### 3.4 Objetos (S3/Mux)

- **Mux**: provider-side. Mux mantém original + renditions; em DR do master-sindico, assets permanecem em Mux. Não precisa restore nosso.
- **S3 proprietário** (export LGPD ZIPs, attachments): versioning ativo + Lifecycle `GLACIER_INSTANT_RETRIEVAL` após 30d. Recovery via `aws s3 cp --version-id`.

### 3.5 Zitadel (crítico — auth)

Zitadel self-host em Postgres dedicado (ADR-0003). Backup inclui:
- PG Zitadel via `pg_dump` separado (mesmo pipeline do §3.1 mas DB distinto).
- Config file (`zitadel.yaml`) versionado em git (sem secrets — secrets em Railway env).

---

## 4. Procedure de recovery (tier crítico, Postgres-centric)

### Pré-condições

- [ ] Oncall **senior** + arquiteto + DPO notificados via PagerDuty + Slack `#incident`
- [ ] Decisão de acionar DR tomada **explicitamente** (quem, quando, por quê — registrado em audit)
- [ ] Status page atualizada: `MAJOR OUTAGE — recovery em andamento, ETA <X>h`
- [ ] Comms inicial aos usuários afetados (§5)
- [ ] Incident bridge aberto (Meet/Zoom)

### Passo 1 — Freeze (T+0 → T+30min)

1. Feature flag **global kill switch** ativado (backend mantém `/health/live` 200 mas `/health/ready` 503 — webhooks retornam 503 pra Stripe/Mux/Livekit reterem e reenviarem).
2. Parar workers (saga orchestrator, NATS consumers se ativo, cron jobs) — evita escrita nova.
3. Snapshot do estado atual (mesmo que corrompido) para **forense**:
   ```bash
   aws s3 cp <current-state-dump> s3://mastersindico-backups/forensic/<YYYY-MM-DD-HH>/
   ```
4. Comms `#incident`: "DR iniciado. Escrita congelada. Restore começando de backup de <timestamp>."

### Passo 2 — Identificar ponto de restore (T+30min → T+1h)

Decisão: qual backup usar?

| Cenário | Estratégia |
|---|---|
| Corrupção detectada hoje 14:00 | Restore **full** do último `pg_dump` < 14:00 + replay **WAL** até `00:59:59` anterior à corrupção (PITR — Point-In-Time Recovery) |
| Ransomware total | Restore **full** do backup S3 (WORM) mais recente + replay WAL até `ransomware_detected_at - 5min` |
| DROP TABLE por engano | Restore em **banco de staging** primeiro, extrair tabela, re-importar em prod (não restore completo) |
| Região Railway down | Subir instância Railway em região alternativa OU Fly.io com PG — restore mais recente |

Documentar em `#incident`:
```
[DR DECISION]
Estratégia: PITR
Full base: 2026-04-23-03:00.sql.gz (S3)
WAL replay até: 2026-04-23 13:59:00 BRT
Janela de dados perdidos: ~1min (acceptable, dentro de RPO 1h)
```

### Passo 3 — Provisionar instância de restore (T+1h → T+2h)

**Não restaurar por cima da instância corrompida.** Sempre subir instância nova e apontar DNS quando pronta.

```bash
# Provisionar Postgres novo (Railway ou Fly.io failover)
railway postgres create --region us-east-1 --name mastersindico-prod-restore

# Obter DATABASE_URL_RESTORE (nova)
export DATABASE_URL_RESTORE=$(railway variables --service mastersindico-prod-restore --raw | grep DATABASE_URL)
```

### Passo 4 — Restore base (T+2h → T+3h)

```bash
# 1. Baixar full backup
aws s3 cp s3://mastersindico-backups/full/2026-04-23.sql.gz . --version-id <VERSION>

# Verificar checksum
sha256sum 2026-04-23.sql.gz  # comparar com manifest.json ao lado no S3

# 2. Restore base
gunzip -c 2026-04-23.sql.gz | psql $DATABASE_URL_RESTORE

# 3. Replay WAL até ponto desejado (PITR)
#    Preparar recovery.conf:
cat <<EOF > recovery.signal
# PG 18 usa cluster_name + pg_wal_replay_resume
EOF

# Config em postgresql.conf (ou variables Railway):
#   restore_command = 'aws s3 cp s3://mastersindico-backups/wal/%f %p'
#   recovery_target_time = '2026-04-23 13:59:00 BRT'
#   recovery_target_action = 'promote'

# Iniciar Postgres em recovery mode
# Railway: usar SERVICE_RESTORE_COMMAND env var

# Aguardar log "archive recovery complete" + "database system is ready to accept connections"
```

### Passo 5 — Validação pré-switchover (T+3h → T+3h30min)

Antes de apontar tráfego:

```sql
-- Sanity check: últimos registros batem com timestamp esperado?
SELECT MAX(created_at) FROM identity.users;
SELECT MAX(created_at) FROM assembly.minutes;
SELECT MAX(created_at) FROM billing.invoices;

-- Integridade: hash-chain timeline (INV-024) consistente?
SELECT institutional.verify_timeline_hash_chain('<condominium_id_sample>');

-- Row counts batem com métricas do último dashboard?
SELECT COUNT(*) FROM identity.users;        -- comparar com último export Grafana
SELECT COUNT(*) FROM content.videos;
SELECT COUNT(*) FROM commercial.contracts;
```

Smoke test mínimo (via scripts/smoke-restore.sh):
- [ ] Login síndico_test@mastersindico.com.br OK (Zitadel + session via backend restored)
- [ ] GET /timeline/<condo-id> retorna dados
- [ ] POST webhook Stripe de teste é aceito com 200 (idempotency via DB ADR-0027)
- [ ] Query ABAC matrix cross-tenant retorna 404 (não 403) para user sem membership

### Passo 6 — Switchover (T+3h30min → T+4h)

1. Atualizar `DATABASE_URL` no service `api` (Railway) para apontar à instância restaurada.
2. Deploy do backend (force restart) → nova pool de conexões.
3. Desativar kill switch gradualmente:
   - Health check `/health/ready` retorna 200
   - Webhooks voltam a processar (Stripe reenviará eventos em queue da janela de freeze)
   - Workers reiniciam
4. Monitorar em tempo real (Grafana + Sentry):
   - Error rate < 1% por 15min
   - Nenhum erro `pg_authentication_failed` (conexões velhas cacheadas)
   - Stripe webhook success rate ≥ 99%

### Passo 7 — Pós-switchover (T+4h → T+24h)

- [ ] Status page: `RESOLVED` — anexar `incident_url`
- [ ] Notificação **aos usuários afetados** (§5) — obrigatória se PII perdida ou sessões invalidadas
- [ ] Documentar em audit:
  ```sql
  INSERT INTO audit.dr_events (action, base_backup, wal_target, duration_sec, admin_id, reason)
    VALUES ('restore_completed', '2026-04-23.sql.gz', '2026-04-23 13:59:00 BRT', 14400, <admin_id>, '<reason>');
  ```
- [ ] Reconciliação **Stripe**: webhooks reenviados podem duplicar se idempotency falhou → validar `webhook_events.stripe_event_id` UNIQUE (ADR-0027) + `payments.stripe_charge_id` UNIQUE
- [ ] Postmortem **crítico** (template [[../../11-audit/postmortems/template]]) em 72h

### Passo 8 — Redis warm-up (T+4h → T+5h)

Redis vazio pós-DR. Não bloqueia restore, mas cache miss rate 100% inicialmente — latência p95 sobe temporariamente. Estratégias:

1. **Cold start aceitável**: deixar warm-up natural (tráfego preenche). OK se SLO p95 já projeta essa janela.
2. **Warm-up ativo**: script pré-aquece ABAC cache para top-N usuários ativos (scripts/redis-warmup.sh).
3. Rate limit reset é **aceitável** — usuários levítimos não percebem; atacantes reiniciam budget (trade-off OK).

### Passo 9 — OpenSearch rebuild (T+24h → T+48h)

Se OpenSearch snapshot não foi restaurado no mesmo window:
- Fallback: busca degradada via PG tsvector (ADR-0018) — já é o default dev.
- Rebuild: replay domain events de `outbox` (INV-019) para reconstruir índices via consumers.

---

## 5. Comms + LGPD Art. 48

### Interno
- **Slack `#incident`**: updates a cada 30min
- **Email stakeholders internos** (CEO, PO, DPO, jurídico): T+1h, T+4h (resolved), T+24h (postmortem draft)
- **Bridge incident**: permanece ativo até switchover + 1h estabilização

### Externo

**Regra LGPD Art. 48**: se DR implica **perda efetiva de dados pessoais** (RPO janela perdida) OU **indisponibilidade > 24h de serviço que processa PII**, notificação **ANPD + titulares em ≤ 72h** é obrigatória.

| Cenário DR | Notificação LGPD obrigatória? |
|---|---|
| Restore com PITR cobrindo 100% dos writes (zero perda) | **Não** (incident de disponibilidade, não de dados) |
| Perda de writes na janela RPO (< 1h crítico) afetando PII (ex: novo cadastro perdido) | **Sim** — notificar titulares cujo dado foi perdido (re-cadastro) + ANPD |
| Indisponibilidade > 24h | **Sim** se PII foi inacessível em janela relevante (ex: titular não conseguiu exercer direito LGPD dentro de prazo) |

Procedure LGPD: acionar [[incident-lgpd-breach]] em paralelo — DPO toca esse fluxo.

### Canais titulares (se LGPD aplicável)
- **Email** (Resend, fallback SMTP backup) — template "DR notice — dados podem ter sido afetados"
- **In-app banner** (web + mobile) — obrigatório dar ciência
- **Status page** público — evento `incident-dr-<ID>` com timeline

### Template status page (pré-aprovado jurídico)

```
[MAJOR OUTAGE] <timestamp início>
Estamos executando recovery de dados de backup após incident em <sistema>.
Estimated recovery: <ETA>.
Atualizaremos a cada 30min.

Alguns dados criados entre <início_janela> e <fim_janela> podem precisar ser re-inseridos.
Se você foi afetado, enviaremos notificação direta por email.

Contato: dpo@mastersindico.com.br
```

---

## 6. Test schedule

### Quarterly DR drill (obrigatório)

Toda Q (março, junho, setembro, dezembro):

1. **Staging** — executar §4 completo em ambiente staging a partir de backup de prod (cópia sanitizada — sem PII real, ou PII pseudonimizada via ADR-0028).
2. **Metas**:
   - RTO real < 4h (meta) — medir.
   - RPO real < 1h — verificar último WAL aplicado.
   - Smoke test 100% OK.
3. **Postmortem do drill** — mesmo template; identificar gaps.
4. **Update runbook** — se algum passo desatualizou (ex: Railway CLI mudou sintaxe).

### Weekly automated restore smoke test

CronJob em container efêmero:
1. Pull do último `pg_dump` full.
2. `createdb`, restore em container Postgres.
3. Rodar subset de queries-chave (últimos IDs por tabela; sanity).
4. Report em Grafana: `dr_smoke_test_last_success_seconds`.
5. Se falhar 2× consecutivas → PagerDuty P1.

### Annual region-failover drill

Exercício completo de mudança de provider/região (Railway → Fly.io OU us-east-1 → sa-east-1). M3+ quando multi-region virar requisito (SOC 2 ou crescimento justificar).

---

## 7. Checklists

### Pré-DR (todo trimestre, fora de incident)

- [ ] Backup `pg_dump` full existe em S3 < 30h (Grafana green)
- [ ] WAL archive lag < 5min (Grafana green)
- [ ] Checksums dos backups recentes batem com manifest
- [ ] Object Lock ativo em buckets S3 de backup
- [ ] `DATABASE_URL_RESTORE_TEMPLATE` disponível em 1Password DPO + arquiteto
- [ ] Último quarterly drill < 90d

### Durante DR

- [ ] Oncall senior + arquiteto + DPO na bridge
- [ ] Kill switch ativado
- [ ] Comms `#incident` a cada 30min
- [ ] Status page atualizada
- [ ] Backup target identificado e checksum validado
- [ ] Nova instância provisionada (não restore in-place)
- [ ] Smoke test pré-switchover OK
- [ ] Audit `dr_events` registrado pós-switchover

### Pós-DR

- [ ] Status page RESOLVED
- [ ] Notificação titulares enviada (se LGPD aplicável)
- [ ] ANPD notificada (se LGPD aplicável)
- [ ] Redis warm-up OK
- [ ] OpenSearch rebuild OK (ou degradação aceita documentada)
- [ ] Stripe reconciliation OK (sem duplicidade)
- [ ] Postmortem agendado 72h

---

## 8. Anti-padrões

- ❌ **Restore in-place** sobre banco corrompido — sempre nova instância, sempre.
- ❌ **Ignorar checksum** do backup antes de restore.
- ❌ **Skippar quarterly drill** — "funciona" vira "não testamos".
- ❌ **Notificar titulares sem coordenação DPO** — LGPD exige procedure formal.
- ❌ **Restore sem freeze prévio** — nova escrita durante restore = dados pós-DR perdidos.
- ❌ **Assumir Stripe/Mux perderam estado** — providers mantêm ledger próprio; reconciliation pós-restore é obrigatória.

---

## 9. Dependências externas em DR

| Provider | Comportamento em DR master-sindico down | Ação nossa |
|---|---|---|
| **Stripe** | Webhooks retornam 503 → Stripe retry exponential (72h) | Nada — aceitar retry; reconciliar pós-switchover via idempotency DB (ADR-0027) |
| **Mux** | Webhooks mesma lógica; uploads presigned podem expirar | Renovar URLs presigned para uploads em curso pós-restore |
| **Livekit** | Sessões live ativas quebram irremediavelmente durante freeze | Comms usuário: "sessão encerrada por incident; re-agendar" |
| **Zitadel** | Sessões podem ser revalidadas; refresh tokens válidos | Usuários logados permanecem; novo login re-autentica via Zitadel normal |
| **Resend** | Queue retém emails até reconexão | Flush natural pós-restore; monitor bounces |
| **Twilio SMS** | Best-effort queue; pode perder SMS < 4h | Aceitar; re-enviar em batch pós-restore se crítico |

---

## 10. Contatos DR

- **Oncall senior**: PagerDuty rotation
- **Arquiteto**: <contato>
- **DPO**: `dpo@mastersindico.com.br`
- **CEO** (decisão pública): <contato>
- **Railway support**: https://help.railway.com/
- **AWS S3 escalation**: AWS Business Support
- **ANPD** (se LGPD aplicável): https://www.gov.br/anpd/ + `anpd@anpd.gov.br`

---

## Links

- [[_moc]]
- [[rollback-deploy]]
- [[incident-lgpd-breach]]
- [[secret-rotation]]
- [[webhook-reprocessing]]
- [[../../02-architecture/topology-multitenant]]
- [[../../08-security/lgpd]]
- [[../../11-audit/postmortems/template]]
- LGPD Art. 48: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm
- PostgreSQL 18 PITR docs: https://www.postgresql.org/docs/18/continuous-archiving.html
- Railway backup docs: https://docs.railway.app/reference/backups
