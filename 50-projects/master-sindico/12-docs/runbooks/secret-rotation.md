---
title: Runbook — Rotação de Secrets
type: guide
tags: [runbook, secret-rotation, security, vault, stripe, zitadel, mux, livekit, twilio, lgpd, master-sindico, v2]
source: 08-security/BEYOND_CORP.md §secrets + Gap-SEC-015 Secrets Manager + ADR-0026 admin-mfa + ADR-0027 webhook idempotency + ADR-0028 lgpd-keyed-hmac
created: 2026-04-23
updated: 2026-04-23
---

# Runbook — Rotação de Secrets

> ⚠️ **ATUALIZAÇÃO 2026-04-27**: este runbook menciona Railway env vars (descontinuado por D-138/ADR-0045). Stack atual usa **AWS Secrets Manager** (origin) + **Cloudflare Workers Secrets** (`wrangler secret put`). Princípios de **dual-key overlap 7d** + audit `lgpd_logs` permanecem válidos.
>
> **Substituições de comandos**:
> - `railway variables set X=Y --service api --environment prod` → `aws secretsmanager put-secret-value --secret-id ms/prod/X --secret-string Y --region sa-east-1` + `aws ecs update-service --force-new-deployment` (para ECS pegar nova versão)
> - `railway variables get X --raw` → `aws secretsmanager get-secret-value --secret-id ms/prod/X --query SecretString --output text`
> - Workers secrets: `npx wrangler secret put X --env production`
>
> Reescrita completa pendente. Inventário §1 (cadências, dual-key) e procedure §3 (7-day overlap) permanecem válidos com substituição de comandos acima. Audit §6 mantém-se idêntico (lgpd_logs).

Rotação operacional de secrets (API keys, signing keys, service accounts) com **dual-config overlap** para ship sem downtime.

**Regra de ouro**: toda rotação é **dual-key (old + new ativos)** por uma janela mínima antes de revogar a chave antiga. Rotação "hard cutover" só em incident de comprometimento (ver §6).

> Toda rotação gera entry em `lgpd_logs` + release tag em Sentry + anotação em Grafana. Sem evidência audit = rotação inválida.

---

## 1. Inventário de secrets e cadência

Cadência alvo (SLA organizacional) para cada secret. Rotação antecipada é sempre permitida; atrasada dispara PagerDuty P2.

| Secret | Cadência | Dual-key overlap | Onde vive | Dono |
|---|---|---|---|---|
| **Stripe live keys** (`sk_live_*`, `pk_live_*`, webhook signing secret) | **90d** | 7d (webhook secret — keys normais podem rotacionar imediato) | Stripe dashboard → Railway env var | Oncall + CFO (billing) |
| **Stripe webhook endpoint secret** (`whsec_*`) | 90d | 7d (obrigatório — payloads em trânsito precisam validar com ambas) | Stripe dashboard → backend `STRIPE_WEBHOOK_SECRET_PRIMARY` + `STRIPE_WEBHOOK_SECRET_SECONDARY` | Oncall |
| **Zitadel service account key** (OIDC backend → Zitadel admin) | **180d** | 7d | Zitadel admin console → Railway env var `ZITADEL_SERVICE_ACCOUNT_KEY` | Arquiteto |
| **Cookie signing key** (`gorilla/securecookie` hashKey + blockKey) | **180d** | **7d obrigatório** (cookies em circulação precisam validar com old OR new) | Railway env var `COOKIE_HASH_KEY_PRIMARY` + `COOKIE_HASH_KEY_SECONDARY` + `COOKIE_BLOCK_KEY_PRIMARY` + `COOKIE_BLOCK_KEY_SECONDARY` | Arquiteto |
| **LGPD HMAC key** (pseudonimização — ADR-0028) | **180d** | N/A — rotação gera novos hashes; entries antigos permanecem | Railway env var `LGPD_HMAC_KEY_CURRENT` + `LGPD_HMAC_KEY_VERSION` | DPO + arquiteto |
| **Mux API access token + secret** | **180d** | 7d | Mux dashboard → Railway env `MUX_TOKEN_ID` + `MUX_TOKEN_SECRET` | Oncall |
| **Mux signing key** (playback policy signed) | 180d | 7d | Mux dashboard → Railway env `MUX_SIGNING_KEY_ID` + `MUX_SIGNING_KEY_PRIVATE` | Oncall |
| **Livekit API key + secret** | **180d** | 7d | Livekit cloud dashboard → Railway env `LIVEKIT_API_KEY` + `LIVEKIT_API_SECRET` | Oncall |
| **Twilio API key + secret** (SMS) | **180d** | 7d | Twilio console → Railway env `TWILIO_ACCOUNT_SID` + `TWILIO_AUTH_TOKEN` | Oncall |
| **Resend API key** | 180d | 7d | Resend dashboard → Railway env `RESEND_API_KEY` | Oncall |
| **OpenSearch admin password** | 180d | 7d | OpenSearch managed → Railway env | Oncall |
| **NATS JetStream credentials** (se ativo — ADR-0019) | 180d | 7d | NATS self-host cred file → Railway env | Arquiteto |
| **JWT signing key** (step-up MFA tokens — ADR-0026) | 90d | **7d obrigatório** (tokens curtos 5min mas em flight) | Railway env `JWT_SIGN_KEY_CURRENT` + `JWT_SIGN_KEY_PREVIOUS` | Arquiteto |
| **AWS access keys** (S3 backup — DR runbook) | 90d | 7d | AWS IAM → Railway env | Arquiteto |
| **Database password** (Postgres Railway) | 180d | não aplica (Railway gerencia — reload pool 1×) | Railway managed | Arquiteto |
| **Redis password** | 180d | não aplica | Railway managed | Arquiteto |
| **Master encryption key** (at-rest — atributos sensíveis em DB, crypto box) | **anual** (ou comprometimento) | **30d overlap** obrigatório (re-encrypt em background) | AWS KMS (não Railway env) | DPO + arquiteto |

**Nota**: secrets gerenciados pelo próprio Railway (DB password, Redis password) rotacionam via `railway service rotate-credentials` com reload de pool — não exigem dual-key porque Railway orquestra.

---

## 2. Pré-requisitos operacionais

### Ferramentas
- Railway CLI autenticada (arquiteto/oncall)
- 1Password vault `mastersindico/prod-secrets` (armazena chaves antigas durante overlap — 7d)
- Sentry CLI para release tagging
- Grafana API token para annotation
- Provider dashboards (Stripe, Zitadel, Mux, Livekit, Twilio, Resend, Mux)

### Comunicação
- Announce em `#engineering` 24h antes da rotação planejada
- Janela preferencial: **terça-feira 14:00-16:00 BRT** (fora de horário crítico BR; antes do deploy semanal quinta)
- PagerDuty em standby durante janela

### Audit trail pré-rotação
- [ ] Entry em `lgpd_logs` (categoria `secret_rotation_planned`) com secret_name + owner + planned_at
- [ ] Snapshot do estado atual em 1Password (comentário "pré-rotação")

---

## 3. Procedure canônica (7-day dual-key overlap)

Fluxo padrão aplicável à maioria dos secrets (Stripe webhook secret, cookie keys, JWT sign keys, provider API keys). Exceções específicas em §4.

### Step 0 — Planejamento (D-1)

1. Abrir PR/ticket com checklist desta procedure (template `.github/ISSUE_TEMPLATE/secret-rotation.md`).
2. Announce `#engineering`.
3. Validar backup de config atual em 1Password.

### Step 1 — Gerar nova key (D0, T+0)

No provider dashboard (Stripe / Mux / Livekit / Twilio / etc.):
- Criar **nova** credential (não revogar antiga ainda).
- Copiar para 1Password com label `<provider>-<date>-NEW`.

Para secrets internos (cookie keys, JWT sign keys, LGPD HMAC):
```bash
# Cookie signing key (hash + block)
openssl rand -hex 32 > new_cookie_hash.key    # 64 bytes hex → 32 bytes
openssl rand -hex 32 > new_cookie_block.key

# JWT signing key (HS256)
openssl rand -hex 64 > new_jwt_sign.key

# LGPD HMAC key (ADR-0028)
openssl rand -hex 32 > new_lgpd_hmac.key
```

### Step 2 — Deploy dual-config (D0, T+1h)

Configurar ambas as keys no backend (código já deve suportar — invariante: lê `*_PRIMARY` E `*_SECONDARY`/`*_PREVIOUS`, aceita ambas).

```bash
# Definir secondary = key atual (legada); primary = new
railway variables set \
  COOKIE_HASH_KEY_SECONDARY="$(railway variables get COOKIE_HASH_KEY_PRIMARY --raw)" \
  --service api --environment prod

railway variables set \
  COOKIE_HASH_KEY_PRIMARY="$(cat new_cookie_hash.key)" \
  --service api --environment prod

# Deploy (rolling)
railway up --service api --environment prod
```

**Validação pós-deploy**:
- [ ] `/health/ready` 200
- [ ] Tests em container canary: validar cookie emitido com PRIMARY é lido de volta OK
- [ ] Tests em container canary: cookie antigo (emitido com SECONDARY) ainda valida

### Step 3 — Traffic shift (D0, T+1h → D0, T+2h)

**Emissão de novos artefatos (cookies, tokens, HMACs) usa PRIMARY. Validação aceita PRIMARY ∪ SECONDARY.**

- Novas sessões → cookie assinado com PRIMARY
- Sessões antigas → cookie assinado com SECONDARY; validação passa
- Webhooks Stripe novos → HMAC validado contra `STRIPE_WEBHOOK_SECRET_PRIMARY` primeiro, fallback `_SECONDARY`
- JWT step-up novo → assinado com PRIMARY; verificação aceita ambas

**Exceção — API keys outbound** (Mux, Livekit, Twilio, Resend): backend usa **apenas** PRIMARY em chamadas saintes. SECONDARY existe só para rollback rápido (§5).

### Step 4 — Monitor 48h (D0 → D+2)

**Métricas a observar** (Grafana panels):
- Error rate `cookie_decode_failed` — deve permanecer próximo de zero
- `jwt_verify_failed_total` — idem
- `webhook_stripe_signature_invalid` — deve ser zero
- `stripe_charge_success_rate` ≥ 99%
- `mux_upload_success_rate`, `livekit_room_create_success_rate`, etc.
- Sentry: errors tagados `release=post-rotation-<date>` aparecendo?

Se **qualquer** métrica degradar > baseline + 2σ por > 15min → invocar §5 rollback.

### Step 5 — Revoke old key (D+7)

Pré-condição: 7 dias de dual-key rodando sem erro acima de baseline.

1. **Provider**: revogar chave antiga no dashboard.
   - Stripe: `Developers → API keys → Revoke sk_live_OLD`
   - Mux: `Settings → Access tokens → Delete`
   - Livekit: `API Keys → Revoke`
   - etc.
2. **Railway**: remover SECONDARY var OU atribuir valor sentinel `__revoked__` (backend valida e loga tentativas de uso — detecta código legado).
3. **1Password**: mover entry `<provider>-<date>-NEW` → apenas `<provider>-<date>-PRIMARY`; old key fica em "revoked archive" por 90d para forense.

### Step 6 — Audit (D+7)

```sql
-- lgpd_logs
INSERT INTO compliance.lgpd_logs (category, action, actor_id, metadata, logged_at)
  VALUES (
    'secret_rotation',
    'rotation_completed',
    '<admin_id>',
    jsonb_build_object(
      'secret_name', 'stripe_webhook_secret',
      'old_key_fingerprint', '<sha256 first 8 chars>',
      'new_key_fingerprint', '<sha256 first 8 chars>',
      'dual_key_window_days', 7,
      'provider', 'stripe'
    ),
    now()
  );
```

Sentry release tag:
```bash
sentry-cli releases new "rotation-stripe-webhook-$(date +%Y-%m-%d)"
sentry-cli releases set-commits --auto "rotation-stripe-webhook-$(date +%Y-%m-%d)"
sentry-cli releases finalize "rotation-stripe-webhook-$(date +%Y-%m-%d)"
```

Grafana annotation:
```bash
curl -X POST https://grafana.mastersindico.internal/api/annotations \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Secret rotation completed: stripe_webhook_secret",
    "tags": ["secret-rotation", "security", "stripe"],
    "time": '$(date +%s000)'
  }'
```

---

## 4. Exceções por tipo de secret

### 4.1 LGPD HMAC key (ADR-0028) — sem dual-key no MESMO sentido

HMAC de pseudonimização LGPD **não** pode ter overlap: o hash gerado depende da key; mudar key = hashes antigos não re-derivam.

**Procedure**:
1. Gerar nova key + incrementar versão (`LGPD_HMAC_KEY_CURRENT_VERSION`).
2. Deploy: backend assina **novos** pseudonimos com `(current_version, current_key)`; valida com `(entry.version → lookup key)` em `lgpd_key_registry` table.
3. Keys antigas permanecem em `lgpd_key_registry` (encrypted by KMS) por **5 anos** (LGPD art. 16 retenção de logs).
4. Revogação: key antiga sai de produção env vars mas permanece em `lgpd_key_registry` (consulta só em caso de auditoria ANPD).

### 4.2 Master encryption key (AWS KMS) — 30d overlap obrigatório

Chave usada para encrypt at-rest de atributos sensíveis (tokens OAuth armazenados, secrets em DB):
1. Criar nova key version em KMS (alias `alias/mastersindico-prod` bumped).
2. Re-encrypt em background (jobs noturnos) — ler linha, decrypt com key old, encrypt com key new.
3. Após 30d + todos rows re-encrypted → disable key old (ainda recuperável 7d) → schedule deletion 30d depois.
4. LGPD evidência: job log + count de rows re-encrypted + checksum.

### 4.3 Cookie signing key — atenção especial

Cookies de sessão com TTL configurado (ex: 7 dias). **Overlap mínimo = TTL cookie + margem**. Com TTL 7d → overlap **14d recomendado**, nunca menos que 7d.

**Validação**:
```go
// backend tenta PRIMARY primeiro, SECONDARY depois; nunca SECONDARY-only.
if err := decodeCookie(cookie, cookieKeyPrimary); err != nil {
    if err2 := decodeCookie(cookie, cookieKeySecondary); err2 != nil {
        return ErrCookieInvalid
    }
    // cookie decoded com SECONDARY — re-emit com PRIMARY no próximo response
    reissueCookieWithPrimary(...)
}
```

### 4.4 Stripe webhook secret — dual validation

Até 7d overlap (Stripe permite até 24h oficialmente, mas internamente mantemos 7d):
```go
// backend tenta verify com PRIMARY; fallback SECONDARY.
if !verifyStripeSignature(rawBody, sigHeader, STRIPE_WEBHOOK_SECRET_PRIMARY) {
    if !verifyStripeSignature(rawBody, sigHeader, STRIPE_WEBHOOK_SECRET_SECONDARY) {
        return 401 // Stripe retry em 5min
    }
    audit("webhook_verified_with_secondary_key")  // sinal de que rotation overlap ainda ativo
}
```

### 4.5 JWT sign key (step-up MFA — ADR-0026)

Tokens step-up têm TTL 5min (NIST AAL2 §5.2.9). Overlap 7d é excessivo vs TTL; mas mantemos 7d como SLA uniforme — custo zero de manter duas keys + simplificação operacional.

---

## 5. Rollback — se rotação deu errado

### Gatilhos
- Error rate > baseline + 2σ por > 15min pós-deploy (Step 2)
- Taxa de `*_verify_failed` subindo (Sentry/Grafana)
- Provider API respondendo 401/403 (Stripe/Mux/Livekit)

### Procedure rollback

```bash
# 1. Revert: PRIMARY volta a ser o valor antigo (ainda em 1Password/vault)
railway variables set \
  COOKIE_HASH_KEY_PRIMARY="$(get_from_1password 'cookie-hash-key-PRE-rotation')" \
  COOKIE_HASH_KEY_SECONDARY="$(cat new_cookie_hash.key)" \
  --service api --environment prod

# 2. Deploy force
railway up --service api --environment prod --detach

# 3. Monitor métricas voltarem ao baseline
```

**Chave antiga permanece ativa** (porque é agora a PRIMARY de novo). Chave nova fica como SECONDARY por 7d em modo "monitor" até decidir:
- Investigar root cause do erro
- Re-planejar rotação

### Audit do rollback

```sql
INSERT INTO compliance.lgpd_logs (category, action, actor_id, metadata)
  VALUES (
    'secret_rotation',
    'rotation_rolled_back',
    '<admin_id>',
    jsonb_build_object(
      'secret_name', '...',
      'reason', '<descrição>',
      'error_spike_metric', '<métrica específica>',
      'rollback_duration_sec', <seconds>
    )
  );
```

Sentry release: `rotation-ROLLBACK-<secret>-<date>`.

---

## 6. Rotação emergencial (compromise detected)

Se há **evidência de comprometimento** da chave (leak em log público, GitGuardian alert, provider notification, IOC em threat intel):

**Pula o dual-key overlap. Hard cutover imediato.**

### Procedure

1. **T+0**: announce `#incident` — Sev P0.
2. **T+5min**: gerar nova key.
3. **T+10min**: revogar old key **no provider IMEDIATAMENTE** (Stripe dashboard, Mux, etc.). Isto quebra tráfego que usa old key — é intencional.
4. **T+15min**: deploy nova key como PRIMARY; SECONDARY vazio ou sentinel.
5. **T+20min**: monitorar — tráfego legítimo migra; tráfego malicioso com old key começa a retornar 401/403 (sucesso da mitigation).
6. **T+1h**: iniciar forense — logs, audit trail, determinar se PII foi acessada.
7. **T+24h**: decidir se é incident LGPD breach → escalar pra [[incident-lgpd-breach]] (72h ANPD).

### Trade-off

Hard cutover = **usuários válidos também podem ser impactados** (cookies antigos invalidos → re-login forçado; webhooks de providers externos entram em retry queue; jobs assíncronos podem falhar 1-2 ciclos). Aceitável em face do risco de compromisso contínuo.

### Audit

```sql
INSERT INTO compliance.lgpd_logs (category, action, severity, actor_id, metadata)
  VALUES (
    'secret_rotation',
    'emergency_rotation',
    'P0',
    '<admin_id>',
    jsonb_build_object(
      'secret_name', '...',
      'trigger', '<IOC, leak URL, provider notification>',
      'hard_cutover', true,
      'elapsed_sec_compromise_to_revoke', <seconds>,
      'breach_notification_triggered', <bool>
    )
  );
```

---

## 7. Checklist consolidado

### Rotação planejada (99% dos casos)

- [ ] PR/ticket aberto com template
- [ ] Announce `#engineering` 24h antes
- [ ] Janela agendada (ter 14-16 BRT idealmente)
- [ ] Backup chave atual em 1Password
- [ ] Gerar nova key
- [ ] Deploy dual-config (PRIMARY = new; SECONDARY = old)
- [ ] Validar canary pós-deploy (cookie emitido OK, antigo ainda valida, etc.)
- [ ] Monitor 48h Grafana + Sentry
- [ ] D+7: revogar old key no provider
- [ ] D+7: remover SECONDARY env var
- [ ] D+7: audit em `lgpd_logs`
- [ ] D+7: Sentry release tag
- [ ] D+7: Grafana annotation
- [ ] Entry calendário próxima rotação (cadência do inventário §1)

### Rotação emergencial

- [ ] `#incident` P0 aberto
- [ ] DPO + arquiteto + CEO notificados
- [ ] Nova key gerada
- [ ] Old key revogada no provider
- [ ] Deploy hard cutover
- [ ] Forense iniciada
- [ ] Decisão breach notification LGPD em 24h
- [ ] Postmortem em 72h

---

## 8. Anti-padrões

- ❌ **Rotacionar todas as secrets ao mesmo tempo** — se der errado, rollback é caótico.
- ❌ **Skippar dual-key overlap** em rotação planejada — hard cutover sem necessidade invalida sessões de usuários válidos.
- ❌ **Revogar old key** no provider **antes** do deploy new key — deploy window = 100% de erro.
- ❌ **Não auditar** — rotação sem `lgpd_logs` entry viola trilha organizacional (LGPD Art. 50).
- ❌ **Manter SECONDARY além de 7d** "por via das dúvidas" — expande superfície de ataque.
- ❌ **Rotação fora de janela combinada** — oncall sobrecarregado se der errado.
- ❌ **Usar `git commit` de `.env` com nova key** — secrets jamais em git; sempre Railway vars + 1Password.
- ❌ **Copiar key em canal público** (Slack `#general`, email aberto) — 1Password compartilhamento controlado.

---

## 9. Matriz de responsabilidade

| Atividade | Oncall | Arquiteto | DPO | CFO |
|---|---|---|---|---|
| Rotação planejada (provider API keys) | **R** | C | I | — |
| Rotação Stripe keys (billing) | R | C | I | **A** |
| Rotação cookie/JWT signing keys | C | **R** | I | — |
| Rotação LGPD HMAC key | C | R | **A** | — |
| Rotação KMS master key | C | R | **A** | I |
| Rotação emergencial | **R** | R | **A** (se PII) | I |
| Audit/LGPD evidence | — | — | **R** | — |
| Postmortem pós-rotação com erro | R | R | I | — |

(R = Responsible, A = Accountable, C = Consulted, I = Informed — RACI)

---

## 10. Monitoring contínuo

### Métricas Grafana (painéis dedicados)

- `secret_age_days{secret_name="..."}` — idade de cada secret; alerta quando ultrapassa cadência §1 + 5d (buffer)
- `secret_dual_key_overlap_active{secret_name}` — boolean; se > 7d ligado, alerta (rotação travou na Step 5?)
- `cookie_decode_failed_total` — spike = rotação cookie key problemática
- `webhook_signature_invalid_total{provider}` — spike = rotação provider problemática ou atacante

### Alertas PagerDuty

- **P2**: secret não rotacionado dentro do SLA (§1 + 5d buffer)
- **P1**: taxa `*_decode_failed` / `*_signature_invalid` > baseline + 3σ
- **P0**: leak detectado (GitGuardian, provider notification) → §6 emergency

---

## Links

- [[_moc]]
- [[dr]]
- [[rollback-deploy]]
- [[incident-lgpd-breach]]
- [[webhook-reprocessing]]
- [[../../08-security/BEYOND_CORP]]
- [[../../08-security/lgpd]]
- [[../../08-security/threat-model]]
- [[../../02-architecture/adr/0027-webhook-idempotency-db]] (a criar)
- [[../../02-architecture/adr/0028-lgpd-keyed-hmac]] (a criar)
- Stripe key rotation docs: https://stripe.com/docs/keys#rotating
- NIST SP 800-57 Part 1 Rev 5 (Key Management): https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final
- OWASP Cryptographic Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
