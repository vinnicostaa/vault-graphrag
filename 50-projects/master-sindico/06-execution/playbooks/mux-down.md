---
title: "Playbook — Mux Down / VOD Outage"
type: playbook
tags: [playbook, incident, master-sindico, ops, content, mux, video]
severity: P1
trigger: "Asset stuck > 10min, playback 4xx/5xx > 5%, Mux webhook failure"
owner: oncall-rotation
created: 2026-04-24
updated: 2026-04-24
---

# Playbook — Mux Down / VOD Outage

> Bounded context: `content`. Provider `IVideoProvider` → `MuxProvider`. Webhook público `POST /webhooks/mux` (HMAC + anti-replay 5min). Saga A-027 (`UploadVideo` com `CancelUpload` compensation).

---

## 1. Trigger

Alertas (Sentry / Grafana):

- **`mux.asset.stuck_in_processing > 10min`** (DLQ check) — vídeos com `status=processing` há mais de 10min
- **`mux.playback.failure_rate > 5%`** janela 5min (4xx/5xx em playback URL signed)
- **`mux.webhook.failure_rate > 5%`** janela 5min (rota `/webhooks/mux`)
- **`mux.upload_url.creation_5xx > 5%`** janela 5min (chamadas `CreateDirectUpload`)

Customer report: "vídeo não termina de processar", "não consigo assistir", "upload trava em 99%".

Dashboards:

- Grafana: `Per-BC dashboard / content` → painel "Mux upload pipeline" + "Playback errors"
- Sentry: tag `bc:content` + `provider:mux`
- Mux Status: <https://status.mux.com>

---

## 2. Severidade + impacto cliente

**P1** — degradação UX importante mas **não bloqueia core do produto** (assembleias e cobrança seguem funcionando):

| Cenário | Sev | Impacto |
|---|---|---|
| Upload Connect Me / Banco Talentos não conclui | **P1** | Empresas não conseguem submeter portfólio; síndicos não veem novos parceiros |
| Playback intermitente em vídeos institucionais | **P1** | UX degradada; dependência baixa em vídeos no MVP |
| Asset stuck > 1h em massa (> 50 assets) | **P0** | Saga A-027 não compensou; quotas presas em `90-day lock` |
| Webhook Mux falha | **P1** | Estado dos vídeos no nosso DB diverge do Mux real |

SLO afetado: `content` 99.5% availability + 100% vídeos moderados antes do publish (observability §3.1).

---

## 3. Triagem em < 5min

### 3.1 Verificar status Mux (paralelo)

```bash
# ✅ Status oficial Mux
curl -s https://status.mux.com/api/v2/status.json | jq '.status.indicator'
# expected: "none" | "minor" | "major" | "critical"

# ✅ Verificar API direta — listar últimos assets
curl -s -u "$MUX_TOKEN_ID:$MUX_TOKEN_SECRET" \
  https://api.mux.com/video/v1/assets?limit=10 | jq '.data[] | {id, status, created_at}'
```

### 3.2 Métrica do nosso endpoint

```bash
# ✅ Logs Railway do API — webhook Mux + provider
railway logs --service api-prod --tail 500 \
  | grep -E '/webhooks/mux|mux_provider|mux_signature' | head -50

# ✅ Sentry — eventos content últimos 30min
# Filtro: tags.bc=content AND timestamp:-30m
```

### 3.3 Estado do DB — assets stuck

```bash
railway connect postgres-prod
```

```sql
-- ✅ Vídeos stuck em "processing" há mais de 10min (suspeitos)
SELECT
  id,
  user_id,
  mux_asset_id,
  mux_upload_id,
  status,
  type,
  EXTRACT(EPOCH FROM (NOW() - created_at))/60 AS age_minutes,
  created_at
FROM content.videos
WHERE status IN ('processing', 'uploading')
  AND created_at < NOW() - INTERVAL '10 minutes'
ORDER BY created_at ASC
LIMIT 100;

-- ✅ Distribuição por status (visão macro)
SELECT status, COUNT(*) FROM content.videos GROUP BY status;

-- ✅ Marketing agency links pendentes (uploads quebrados afetam eles também)
SELECT status, COUNT(*) FROM content.marketing_agency_links GROUP BY status;
```

### 3.4 Webhook events recentes

```sql
-- ✅ Eventos Mux processados última hora
-- (assumindo tabela mux_webhook_events similar à stripe; se não existir, usa logs)
-- Em M1 não há tabela dedicada — usar logs Railway + audit_log_entries
SELECT entry_type, COUNT(*)
FROM cross_domain.audit_log_entries
WHERE entry_type LIKE 'mux.%'
  AND created_at > NOW() - INTERVAL '1 hour'
GROUP BY entry_type;
```

### 3.5 Latência endpoint

```bash
# ✅ Ping no /webhooks/mux com signature inválida — 400 esperado < 200ms
curl -i -w "\nTotal: %{time_total}s\n" \
  -X POST https://api.mastersindico.com.br/webhooks/mux \
  -H 'mux-signature: t=1700000000,v1=invalid' \
  -d '{}'
# expected: HTTP 400; tempo < 500ms
```

---

## 4. Mitigação imediata (ordem de menor risco)

### 4.1 Mux está down (status.mux.com indica outage)

Não há ação técnica imediata além de:

1. **Comms imediato** (< 15min) — status page Master Síndico: "uploads de vídeo temporariamente indisponíveis"
2. **Aguardar Mux retentar webhooks** — Mux retenta com backoff por até 24h
3. **Garantir que Saga A-027 vai compensar** uploads órfãos (state `failed` automaticamente após TTL)

### 4.2 Mux verde mas nosso endpoint falha

| Causa | Sintoma | Mitigação |
|---|---|---|
| `MUX_WEBHOOK_SECRET` rotacionado e não atualizado | 100% webhooks retornam 400 (`invalid signature`) | Re-set `MUX_WEBHOOK_SECRET` em Railway → restart |
| Anti-replay rejeitando legítimos | Logs `timestamp out of window` em massa | Verificar clock drift do servidor; `timedatectl` → ntp sync |
| Bug em `mux_provider.go` (deploy recente) | Erros específicos a tipos de event | **Rollback** (`rollback.md`) |
| Saga A-027 não compensa | Assets órfãos > 50 acumulados | Compensação manual (§4.4) |

### 4.3 Comunicar usuários afetados

Banner no app (feature flag a implementar Sprint 11; em M1 = via cliente web manualmente):

```text
"Upload de vídeo temporariamente indisponível. Suas tentativas serão
reprocessadas automaticamente. Tente novamente em alguns minutos."
```

### 4.4 Saga compensation manual (assets stuck > 1h)

Quando Saga A-027 não compensou automaticamente — script ops:

```sql
-- ✅ Listar assets candidatos a compensação manual
SELECT
  id,
  user_id,
  mux_upload_id,
  mux_asset_id,
  type,
  created_at
FROM content.videos
WHERE status IN ('processing', 'uploading')
  AND created_at < NOW() - INTERVAL '1 hour'
ORDER BY created_at ASC;
```

Para cada video stuck, executar via worker `cmd/saga-compensator` (a implementar — ver §5.2 abaixo) ou comando manual:

```bash
# ✅ Cancelar upload no Mux (idempotente)
curl -X DELETE -u "$MUX_TOKEN_ID:$MUX_TOKEN_SECRET" \
  https://api.mux.com/video/v1/uploads/<MUX_UPLOAD_ID>

# ✅ Cancelar asset no Mux (se já foi promovido a asset)
curl -X DELETE -u "$MUX_TOKEN_ID:$MUX_TOKEN_SECRET" \
  https://api.mux.com/video/v1/assets/<MUX_ASSET_ID>
```

```sql
-- ✅ Marcar video.status=failed + reverter quota lock 90d
BEGIN;

UPDATE content.videos
SET status = 'failed',
    failure_reason = 'mux outage 2026-MM-DD — manual compensation',
    updated_at = NOW()
WHERE id = '<VIDEO_ID>';

-- Reverter quota consumida (90d lock release)
UPDATE billing.feature_usages
SET counter = counter - 1
WHERE user_id = '<USER_ID>'
  AND feature_key = 'video.upload'
  AND window_start <= NOW()
  AND window_end > NOW()
  AND counter > 0;

-- Audit
INSERT INTO cross_domain.audit_log_entries (id, actor_user_id, entry_type, target_id, payload, created_at)
VALUES (gen_random_uuid(), '<OPS_USER_ID>', 'mux.saga_compensation_manual',
        '<VIDEO_ID>', jsonb_build_object('reason','outage','duration_min',60), NOW());

COMMIT;
```

---

## 5. Recovery

### 5.1 Verificar volta ao baseline

```bash
# ✅ Smoke test upload completo
# 1) Backend cria upload URL
SMOKE_TOKEN="<smoke session token>"
curl -X POST https://api.mastersindico.com.br/api/v1/content/videos \
  -H "Authorization: Bearer $SMOKE_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"type":"empresa_apresentacao","title":"smoke-prod"}'
# expected: 201 + {upload_url, video_id}

# 2) Métricas Grafana voltam
# - mux.asset.stuck_in_processing → 0
# - playback.failure_rate < 1%
# - webhook.success_rate > 99%
```

### 5.2 Drain de assets stuck — re-upload script

Pseudocódigo do worker `cmd/saga-reconciler/main.go` (proposta Sprint 10):

```go
// cmd/saga-reconciler/main.go (proposta Sprint 10)
func main() {
    ctx := context.Background()
    pool := mustConnectPG(ctx)
    mux := mustNewMuxProvider()
    repo := content.NewVideoRepo(pool)
    quotas := billing.NewQuotaConsumerAdapter(pool)

    stuck, err := repo.ListStuckSince(ctx, 1*time.Hour, 200)
    must(err)

    for _, v := range stuck {
        // 1) Tenta GetAsset no Mux pra ver estado real
        asset, err := mux.GetAsset(ctx, v.MuxAssetID)
        switch {
        case errors.Is(err, mux_errors.ErrNotFound):
            // Asset realmente sumiu — compensar
            _ = mux.CancelUpload(ctx, v.MuxUploadID)
            _ = repo.MarkFailed(ctx, v.ID, "mux outage compensation")
            _ = quotas.RevertConsumption(ctx, v.UserID, "video.upload", 1)
        case err == nil && asset.Status == "ready":
            // Mux finalizou — sincronizar nosso estado
            _ = repo.MarkReady(ctx, v.ID, asset.PlaybackID, asset.DurationSec)
        case err == nil && asset.Status == "errored":
            _ = repo.MarkFailed(ctx, v.ID, asset.ErrorMessage)
            _ = quotas.RevertConsumption(ctx, v.UserID, "video.upload", 1)
        default:
            // ainda processando — deixar pra próxima rodada
        }
    }
}
```

Comando de invocação:

```bash
# ✅ Roda 1x manual pós-recovery
railway run --service api-prod -- /app/saga-reconciler --since=1h
```

### 5.3 Reconciliação Mux ↔ DB

```bash
# ✅ Exporta assets ativos no Mux
curl -s -u "$MUX_TOKEN_ID:$MUX_TOKEN_SECRET" \
  'https://api.mux.com/video/v1/assets?limit=100' \
  | jq -c '.data[] | {id, status, playback_ids}' \
  > /tmp/mux-assets.jsonl
```

```sql
-- ✅ Assets no nosso DB
COPY (
  SELECT mux_asset_id, status FROM content.videos
  WHERE mux_asset_id IS NOT NULL
) TO STDOUT WITH CSV HEADER;
```

Diff via script — flag órfãos em qualquer lado.

### 5.4 Migration rollback?

Tipicamente não aplicável a outage Mux. Se deploy alterou schema `content.*`, ver `rollback.md` §5.

---

## 6. Comms templates

### 6.1 Slack `#ms-ops`

```text
[INCIDENT] Sev P1 — Mux VOD outage
Detectado: <HH:MM UTC> via <PagerDuty|Sentry|customer report>
Sintoma: <% uploads stuck> | <% playback failures> | webhook fail
Status Mux: <https://status.mux.com — indicator>
Impacto: Connect Me + Banco Talentos uploads bloqueados
       : ~<N> assets stuck > 10min
Mitigação: <aguardando Mux | saga compensation manual em curso>
Investigando. Updates a cada 20min.
```

### 6.2 Status page (público)

```text
Investigando — Upload de vídeos
Estamos resolvendo um problema com nosso provedor de vídeo. Funcionalidades
de assembleia, cobrança e demais serviços continuam normais.
Vídeos que falharam serão reprocessados automaticamente.
```

### 6.3 E-mail cliente (P1 prolongado > 4h)

```text
Assunto: [Master Síndico] Upload de vídeos — atualização

Olá,

Detectamos uma indisponibilidade em nosso provedor de vídeo (Mux) entre
<HH:MM> e <HH:MM> de <DD/MM>.

O que isso significou:
- Tentativas de upload nesse período não foram concluídas.
- Vídeos parcialmente enviados serão reprocessados automaticamente, ou
  você pode tentar enviar novamente.
- Sua quota de uploads não foi consumida pelas tentativas falhadas.

Outras funcionalidades continuaram normais.

Equipe Master Síndico
```

### 6.4 ANPD (LGPD)

**Aplicável SE Mux declarar breach próprio**:

- Vídeos podem conter PII (rosto, voz, ambiente residencial visível)
- Vídeos privados de assembleia gravada (M3) contêm votos e opiniões
- Se Mux teve incident de segurança → seguir `cross-tenant-leak.md` §6
- T+1h DPO interno; T+72h ANPD via Res. 15/2024

Outage operacional sem leak: postmortem interno, sem ANPD.

---

## 7. Pós-incidente

### 7.1 Postmortem (≤ 48h se P1 prolongado > 1h, blameless)

### 7.2 Métricas a coletar

- Assets stuck count (snapshot durante incident)
- Quotas comprometidas (lock 90d revertido por compensação)
- Uploads perdidos (failed permanentemente)
- TTR vs target
- Error budget consumido (% trimestral content SLO)

### 7.3 Action items típicos

- Implementar `cmd/saga-reconciler` worker recorrente (Sprint 10) — auto-compensar stuck > 1h
- Tabela dedicada `content.mux_webhook_events` (paridade com Stripe) para idempotency + replay
- Alert `mux.asset.stuck > 30 count` para detectar saturação antes de impactar massa
- Alert pre-expiry de quota lock 90d (se vídeo falhou, lock está consumido)

---

## 8. Escalação

| Quando | Quem | Como |
|---|---|---|
| Sev P1 > 2h | Tech Lead | PagerDuty escalation |
| > 100 assets stuck simultaneamente | Tech Lead + DevOps | considerar pause de novos uploads via banner UI |
| Mux API 5xx persiste > 1h | Mux Support | <https://www.mux.com/support> ou support@mux.com — incluir `environment_id` + asset_ids exemplo |
| Suspeita de leak vídeo privado | Tech Lead + DPO | escalar `cross-tenant-leak.md` §6 |
| Compensação manual > 100 records | Ops + Tech Lead | revisar script antes de executar; sempre BEGIN/ROLLBACK em staging primeiro |

Contatos:

- Mux environment ID: ver `1Password / Master Síndico / Mux`
- Mux webhook secret: `MUX_WEBHOOK_SECRET` em Railway prod
- Mux signing key: `MUX_SIGNING_KEY` (playback URLs assinadas)
- DPO interno: `dpo@mastersindico.com.br`

---

## 9. LGPD checklist

- [ ] Mux declarou breach? (verificar e-mail support@mux.com / security)
- [ ] Logs nossos expuseram URL signed playback (que tem TTL mas é PII se vazar)? (`grep mux.com/playback sentry/exports/`)
- [ ] Vídeos privados (assembleia gravada — futuro M3) estavam acessíveis publicamente durante incident?
- [ ] Asset stuck conteve PII visual? (rosto, voz) — relevante se vazou
- [ ] Se sim → escalar `cross-tenant-leak.md` §6 (T+1h DPO; T+72h ANPD)

Mux é processador (Art. 5º, VIII LGPD); nós controladores. Em breach lateral, **somos** responsáveis pela notificação ANPD + titulares.

---

## 10. Vizinhos

- [[../incident]] — fluxo geral incident response
- [[../rollback]] — procedimento rollback Railway
- [[../../11-audit/postmortems/template]]
- [[../../03-subprojects/backend/security]] §6.2 (HMAC Mux), §12.3 (anti-replay)
- [[../../02-architecture/observability]] §3.1 (SLO content 99.5%)
- [[../../08-security/threat-model]] §6.2 (Mux key leak), §4.5 (upload Mux STRIDE), §9 (moderation bypass)
- [[../../01-domain/bounded-contexts|bounded-contexts]] — content context + Saga A-027
- [[livekit-down-assembly]] — Mux estava em estudo para live (cancelado em favor de Livekit)
- [[cross-tenant-leak]] — caso vídeo privado vaze cross-tenant
