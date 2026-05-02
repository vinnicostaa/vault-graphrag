---
title: "Playbook — Stripe Down / Webhook Failure"
type: playbook
tags: [playbook, incident, master-sindico, ops, billing, stripe]
severity: P0|P1
trigger: "Sentry alert: Stripe webhook failure rate > 5% (5min) OU Stripe API 5xx > 1% (5min)"
owner: oncall-rotation
created: 2026-04-24
updated: 2026-04-24
---

# Playbook — Stripe Down / Webhook Failure

> Bounded context: `billing`. Webhook público `POST /webhooks/stripe` (HMAC, sem `Authenticate`). Idempotência via `event.id` UNIQUE. Provider `IPaymentGateway` → `StripeGateway`.

---

## 1. Trigger

Alertas (Sentry / Grafana):

- **`stripe.webhook.failure_rate > 5%`** janela 5min (rota `/webhooks/stripe`)
- **`stripe.api.5xx_rate > 1%`** janela 5min (chamadas saintes — checkout, customer-portal)
- **DLQ webhook_events** com `status='failed' AND retry_count<5` cardinalidade > 10
- Customer report: "checkout falhou", "minha assinatura sumiu", "fui cobrado 2x"

Dashboards:

- Grafana: `Per-BC dashboard / billing` → painel "Stripe webhook" + "Stripe API latency"
- Sentry: tag `bc:billing` + `provider:stripe`
- Stripe Dashboard: <https://dashboard.stripe.com/events>

---

## 2. Severidade + impacto cliente

| Cenário | Sev | Impacto |
|---|---|---|
| Webhook não processa eventos de `customer.subscription.*` ou `invoice.*` em prod | **P0** | Cobrança ativa fora do estado real; trial pode expirar mal; subscription pode ficar `past_due` sem aviso |
| Stripe API 5xx em `/billing/checkout` | **P1** | Novos signups bloqueados; usuários existentes não afetados |
| Stripe API 5xx em `/billing/customer-portal` | **P2** | Cliente não consegue cancelar/atualizar cartão; degradação |
| Status page Stripe verde mas nosso endpoint falha | **P1** | Bug nosso (HMAC, parsing, DB) — investigar antes de culpar Stripe |

SLO afetado: `billing` 99.9% webhooks Stripe processados (observability §3.1).

---

## 3. Triagem em < 5min

### 3.1 Verificar status Stripe (paralelo)

```bash
# ✅ Status oficial
curl -s https://status.stripe.com/api/v2/status.json | jq '.status.indicator'
# expected: "none" | "minor" | "major" | "critical"

# ✅ Eventos recentes no Stripe (paginar)
stripe events list --limit 20 --api-key "$STRIPE_SECRET_KEY"
```

### 3.2 Métrica do nosso endpoint

```bash
# ✅ Logs Railway últimos 15min do serviço API
railway logs --service api-prod --tail 500 | grep -E '/webhooks/stripe|stripe_webhook' | head -50

# ✅ Sentry — eventos billing últimos 30min
# Filtro: tags.bc=billing AND timestamp:-30m
```

### 3.3 Estado do DB (queries prontas)

```bash
# Conectar via Railway proxy
railway connect postgres-prod
```

```sql
-- ✅ Webhooks falhados últimas 2h (idempotency table)
SELECT
  id,
  event_id,
  event_type,
  status,
  retry_count,
  last_error,
  received_at
FROM billing.stripe_webhook_events
WHERE received_at > NOW() - INTERVAL '2 hours'
  AND status IN ('failed', 'pending')
ORDER BY received_at DESC
LIMIT 50;

-- ✅ Subscriptions em estado anômalo
SELECT
  status,
  COUNT(*)
FROM billing.subscriptions
GROUP BY status;
-- esperado em prod estável: trialing < active; past_due < 1% do total;
-- incomplete/incomplete_expired devem decair em < 24h

-- ✅ Subscriptions presas em "incomplete" > 1h (signup quebrou)
SELECT id, user_id, stripe_subscription_id, created_at
FROM billing.subscriptions
WHERE status = 'incomplete'
  AND created_at < NOW() - INTERVAL '1 hour'
ORDER BY created_at DESC;
```

### 3.4 Latência endpoint

```bash
# ✅ Pulse no /webhooks/stripe (sem signature válida — 400 esperado < 200ms)
curl -i -w "\nTotal: %{time_total}s\n" \
  -X POST https://api.mastersindico.com.br/webhooks/stripe \
  -H 'Stripe-Signature: t=0,v1=invalid' \
  -d '{}'
# expected: HTTP 400 + tempo < 500ms; se demora 5s+ → DB lento ou pgxpool saturado
```

---

## 4. Mitigação imediata (ordem de menor risco)

### 4.1 Stripe está down (status.stripe.com indicador != "none")

Não há o que fazer no nosso lado além de:

1. **Comms imediato** (< 10min) — status page Master Síndico + Slack `#ms-ops`
2. **Garantir que DLQ está acumulando** corretamente (eventos não processados ficam em `stripe_webhook_events` status=`pending`)
3. **NÃO** desabilitar verificação HMAC (manter `VerifyWebhookSignature` ativo) — Stripe vai retentar em [`exponential backoff`](https://stripe.com/docs/webhooks/best-practices)
4. Aguardar Stripe restaurar; **drain DLQ pós-recovery** (§5.2)

### 4.2 Stripe verde mas nosso endpoint falha (bug nosso)

Possíveis causas + mitigação:

| Causa | Sintoma | Mitigação |
|---|---|---|
| HMAC secret rotacionado e não atualizado | 100% 400 com `ErrInvalidWebhookSignature` | Verificar `STRIPE_WEBHOOK_SECRET` em Railway → comparar com Stripe Dashboard → rotacionar |
| pgxpool saturado | Timeouts intermitentes; logs `connection pool exhausted` | Bump temporário `DATABASE_MAX_CONNS` via Railway env + restart; investigar query lenta com `pg_stat_activity` |
| Bug em handler (deploy recente) | 5xx em event types específicos | **Rollback** (ver `rollback.md`) — não tente hotfix sob pressão |
| Idempotency bug (UNIQUE 23505 não tratado) | `failed` com erro `duplicate key violation` | Hotfix curto: garantir `ON CONFLICT DO NOTHING` em `stripe_webhook_events` insert |

### 4.3 Pausar novas cobranças (decisão Tech Lead — só se P0 prolongado > 1h)

Se houver feature flag (não há em M1; flag será adicionada Sprint 11):

```bash
# ⚠️ futuro — flag não existe em M1
railway variables set --service api-prod BILLING_CHECKOUT_ENABLED=false
railway service restart api-prod
```

Em M1: alternativa é remover endpoint `/billing/checkout` da rota (deploy hotfix). **Evite** — prefira comunicar e esperar.

---

## 5. Recovery

### 5.1 Verificar volta ao baseline

```bash
# ✅ Endpoint responde
curl -i https://api.mastersindico.com.br/health/ready

# ✅ Métrica Grafana: webhook success rate > 99% por 10min consecutivos

# ✅ Sentry: parou de receber novos errors em billing por 10min
```

### 5.2 Drain DLQ — replay webhooks falhados

Pseudocódigo do worker `cmd/webhook-replayer/main.go` (a implementar Sprint 10 — DT-010 audit + retry policy):

```go
// cmd/webhook-replayer/main.go (proposta Sprint 10)
func main() {
    ctx := context.Background()
    pool := mustConnectPG(ctx)
    sm := mustNewStripeGateway()
    repo := billing.NewWebhookEventRepo(pool)

    failed, err := repo.ListFailedSince(ctx, time.Now().Add(-6*time.Hour), 200)
    must(err)

    for _, evt := range failed {
        if evt.RetryCount >= 5 { continue }            // dead-letter, requer ação manual
        rawBody := evt.RawBody                          // armazenado no insert original
        // Replay = re-executar o handler IDEMPOTENTE; não re-verifica HMAC
        if err := billing.HandleStripeEventReplay(ctx, evt.EventID, evt.EventType, rawBody); err != nil {
            repo.MarkRetryFailed(ctx, evt.ID, err)
            continue
        }
        repo.MarkProcessed(ctx, evt.ID)
    }
}
```

Comando manual (até worker existir):

```bash
# ✅ Replay manual via Stripe CLI (re-envia webhook do Stripe pra nosso endpoint;
# stripe assina novamente, idempotency UNIQUE no DB protege duplicação)
stripe events resend evt_xxxxxxxxxxxx --webhook-endpoint we_xxxxxxxxxxxx
```

```sql
-- ✅ Listar event_ids candidatos a resend
SELECT event_id, event_type, retry_count, last_error
FROM billing.stripe_webhook_events
WHERE status = 'failed'
  AND retry_count < 5
  AND received_at > NOW() - INTERVAL '6 hours'
ORDER BY received_at;
```

### 5.3 Reconciliação Stripe ↔ DB

Pós-incident prolongado, comparar estado Stripe vs nosso DB:

```bash
# ✅ Exporta subscriptions ativas no Stripe (paginar)
stripe subscriptions list --status active --limit 100 \
  --api-key "$STRIPE_SECRET_KEY" \
  | jq -c '.data[] | {id, customer, status, current_period_end}' \
  > /tmp/stripe-subs.jsonl
```

```sql
-- ✅ Subscriptions ativas no nosso DB
COPY (
  SELECT stripe_subscription_id, stripe_customer_id, status, current_period_end
  FROM billing.subscriptions
  WHERE status = 'active'
) TO STDOUT WITH CSV HEADER;
```

Diff via `jq` + `awk` ou script Python; **flag** qualquer ID em só um dos dois lados → ticket reconciliação.

### 5.4 Migration rollback?

Não aplicável tipicamente — incidents Stripe raramente envolvem migrations. Se deploy recente fez migration que afeta `billing.*`, ver `rollback.md` §5 (expand-contract; nunca DROP COLUMN no primeiro deploy).

---

## 6. Comms templates

### 6.1 Slack `#ms-ops` (síntese inicial)

```text
[INCIDENT] Sev <P0|P1> — Stripe webhook failure
Detectado: <HH:MM UTC> via <PagerDuty|Sentry|customer report>
Sintoma: <% webhooks failing> | <novos signups bloqueados>
Status Stripe: <https://status.stripe.com — indicator>
Impacto estimado: <N usuários | feature: cobrança | feature: signup>
Mitigação: <DLQ acumulando OK | rollback iniciado | aguardando Stripe>
Investigando. Updates a cada 15min.
```

### 6.2 Status page (público)

```text
Investigando — Cobranças e novos signups
Estamos investigando intermitências no processamento de pagamentos.
Assinaturas ativas continuam válidas. Eventos pendentes serão reprocessados
automaticamente.
Próxima atualização em 15min.
```

### 6.3 E-mail cliente (P0 prolongado > 1h)

```text
Assunto: [Master Síndico] Status do processamento de pagamentos

Olá,

Detectamos um incidente no nosso provedor de pagamentos (Stripe) entre <HH:MM>
e <HH:MM> de <DD/MM>. Durante esse período, alguns eventos de cobrança não
foram processados imediatamente.

O que isso significa para você:
- Sua assinatura continua ativa.
- Cobranças pendentes serão reprocessadas em até <X horas>.
- Você não será cobrado em duplicata — temos proteção de idempotência.

Se notar algo incorreto na sua fatura ou pagamento, responda este e-mail.

Equipe Master Síndico
```

### 6.4 ANPD (LGPD)

**Geralmente não aplicável** — Stripe não nos repassa PAN (somos SAQ-A). Verificar:

- Houve exposição de PII nos logs de erro? (CPF, email, telefone)
- Stripe declarou breach do lado deles? (verificar e-mails security@stripe.com)

Se sim → seguir `cross-tenant-leak.md` §6 (ANPD Res. 15/2024).

---

## 7. Pós-incidente

### 7.1 Postmortem (≤ 48h, blameless)

Trigger obrigatório se:

- Duração > 5min com user-visible impact
- > 20% error budget trimestral consumido
- LGPD signal (raro neste playbook)

Template: [[../../11-audit/postmortems/template]].

### 7.2 Métricas a coletar

- Duração total do incidente (T_detect → T_resolved)
- Webhooks failed count (de `stripe_webhook_events`)
- Subscriptions afetadas (estado anômalo)
- Receita em risco (Σ MRR de `past_due` durante janela)
- TTR (time to recovery) vs target
- Error budget consumido (% trimestral)

### 7.3 Action items típicos

- Adicionar alert mais cedo (ex: error rate > 1% em 2min, não 5%)
- Implementar `cmd/webhook-replayer` (Sprint 10)
- Circuit breaker em `StripeGateway` (DT-003 — sony/gobreaker)
- Documentar runbook se variante nova

---

## 8. Escalação

| Quando | Quem | Como |
|---|---|---|
| Sev P0 > 30min | Tech Lead + DPO (se LGPD) | PagerDuty escalation policy |
| Stripe API 5xx persiste > 1h | Stripe Support | <https://support.stripe.com> — incluir `account_id` + `event_id`s falhados |
| Stripe Support não responde > 2h em P0 | Stripe Phone (US 24/7) | +1-888-963-8398 (Premium support); precisa account ID + último charge ID |
| Bug nosso (rollback necessário) | Dev sênior + Tech Lead | Aprovar rollback (`rollback.md`) |
| Reconciliação manual > 100 subs | Finance/Ops | Time financeiro processa diff em planilha |

Contatos:

- Stripe account ID: ver `1Password / Master Síndico / Stripe Live`
- Stripe webhook secret: `STRIPE_WEBHOOK_SECRET` em Railway prod env
- DPO interno: `dpo@mastersindico.com.br`

---

## 9. LGPD checklist

- [ ] Houve exposição de PII em logs de erro? (`grep -E 'cpf|email|telefone' sentry/exports/`)
- [ ] Stripe declarou breach próprio? (verificar e-mails security@stripe.com)
- [ ] Se sim → escalar para `cross-tenant-leak.md` §6 (T+1h DPO; T+72h ANPD)
- [ ] Documentar no postmortem mesmo que não-LGPD (audit trail)

Stripe é processador (Art. 5º, VIII LGPD); nós controladores. Em breach lateral, **somos** responsáveis pela notificação ANPD + titulares.

---

## 10. Vizinhos

- [[../incident]] — fluxo geral incident response + severidade SLA
- [[../rollback]] — procedimento rollback Railway
- [[../../11-audit/postmortems/template]]
- [[../../03-subprojects/backend/security]] §6.1 (HMAC Stripe), §12.1 (idempotency)
- [[../../02-architecture/observability]] §3.1 (SLO billing 99.9%)
- [[../../08-security/threat-model]] §6.1 (Stripe key leak), §4.6 (Stripe surface STRIDE)
- [[../../01-domain/bounded-contexts|bounded-contexts]] — billing context
- [[cross-tenant-leak]] — caso PII vaze nos logs
- [[zitadel-down]] — se auth também cair, billing/checkout fica indisponível upstream
