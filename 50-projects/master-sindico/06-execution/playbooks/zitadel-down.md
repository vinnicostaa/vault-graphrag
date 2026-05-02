---
title: "Playbook — Zitadel Down / Auth Outage"
type: playbook
tags: [playbook, incident, master-sindico, ops, identity, zitadel, auth]
severity: P0
trigger: "/auth/login 5xx > 5%, OAuth callback fail, Zitadel introspection 401/timeout"
owner: oncall-rotation
created: 2026-04-24
updated: 2026-04-24
---

# Playbook — Zitadel Down / Auth Outage

> Bounded context: `identity`. Zitadel é OIDC provider externo. Middleware `Authenticate` chama `Introspect(token)` em todo `/api/v1/*`. Sem Zitadel = produto down.

---

## 1. Trigger

Alertas (Sentry / Grafana):

- **`auth.login.5xx_rate > 5%`** janela 2min (rota `/auth/login`)
- **`auth.callback.failure_rate > 5%`** (rota `/auth/callback`)
- **`zitadel.introspection.timeout_rate > 5%`** (qualquer chamada saliente)
- **`zitadel.healthcheck != 200`** por 1min consecutivo
- **DT-003 ativo**: sem circuit breaker → cada request `/api/v1/*` cascateia 503 quando introspection falha

Customer report: "não consigo logar", "fica em loop no callback", "fica desconectando sozinho".

Dashboards:

- Grafana: `Per-BC dashboard / identity` → painel "OIDC discovery latency", "Introspection p95"
- Sentry: tag `bc:identity` + `provider:zitadel`
- Zitadel Cloud Status (se cloud): <https://status.zitadel.com>
- Zitadel self-host (Sprint 9.19 pendente — Railway managed): `railway logs --service zitadel-prod`

---

## 2. Severidade + impacto cliente

**P0 absoluto** em qualquer cenário:

| Cenário | Impacto |
|---|---|
| Zitadel introspection 100% falha | **Login down + sessões existentes morrem ao expirar token (~5min cache TTL)** |
| `/auth/login` 5xx | Novos logins bloqueados; sessões já válidas continuam funcionando até `ms_session` cookie expirar |
| `/auth/callback` falha | Usuários no meio do fluxo OIDC ficam presos; precisam recomeçar |
| OIDC discovery 401 | Cert do issuer inválido ou rotação Zitadel — discovery `.well-known/openid-configuration` quebra tudo |

SLO afetado: `identity` 99.5% availability + p95 `/auth/login` < 500ms (observability §3.1).

**Não é só "feature degradada"** — produto inteiro fica inacessível pra novos logins.

---

## 3. Triagem em < 5min

### 3.1 Verificar status Zitadel (paralelo)

```bash
# ✅ OIDC discovery — tem que retornar 200 + JSON com endpoints
curl -s -o /dev/null -w "%{http_code} %{time_total}s\n" \
  https://auth.mastersindico.com.br/.well-known/openid-configuration

# ✅ Introspection direta com token de teste (cache miss forçado)
TOKEN="<token de smoke-test do 1Password>"
curl -s -X POST https://auth.mastersindico.com.br/oauth/v2/introspect \
  -u "$ZITADEL_CLIENT_ID:$ZITADEL_CLIENT_SECRET" \
  -d "token=$TOKEN" | jq '.active'
# expected: true (se token válido) | false (se inválido) | erro 5xx (Zitadel down)

# ✅ Cert do issuer válido?
echo | openssl s_client -showcerts -servername auth.mastersindico.com.br \
  -connect auth.mastersindico.com.br:443 2>/dev/null | openssl x509 -noout -dates
```

### 3.2 Logs nossos

```bash
# ✅ Logs do API — erros recentes em Authenticate middleware
railway logs --service api-prod --tail 500 \
  | grep -E 'authenticate|zitadel|introspect|oidc' | head -50

# ✅ Sentry — eventos identity últimos 15min
# Filtro: tags.bc=identity AND timestamp:-15m
```

### 3.3 Estado das sessões nossas

```bash
railway connect postgres-prod
```

```sql
-- ✅ Sessões ativas (não invalidadas)
SELECT COUNT(*)
FROM identity.sessions
WHERE invalidated_at IS NULL
  AND expires_at > NOW();

-- ✅ Distribuição de expires_at — quanto tempo até maioria cair?
SELECT
  width_bucket(EXTRACT(EPOCH FROM (expires_at - NOW()))/60, 0, 240, 8) AS bucket_min,
  COUNT(*)
FROM identity.sessions
WHERE invalidated_at IS NULL AND expires_at > NOW()
GROUP BY bucket_min
ORDER BY bucket_min;
-- bucket 0 = sessões expirando próximos 30min — esses usuários vão precisar relogar

-- ✅ Logs de sync user recente — caiu pra zero?
SELECT date_trunc('minute', updated_at) AS minute, COUNT(*)
FROM identity.users
WHERE updated_at > NOW() - INTERVAL '30 minutes'
GROUP BY minute
ORDER BY minute DESC;
-- queda abrupta = ninguém logando = confirmação Zitadel down
```

### 3.4 Cache Redis (introspection cache 5min)

```bash
# ✅ Conectar Redis prod
railway connect redis-prod

# Dentro do redis-cli
KEYS 'ms:auth:*' | head -20
DBSIZE
```

Cache populado = introspection ainda funcionou em algum momento → degradação parcial.
Cache vazio + DBSIZE caindo → tudo expirou e Zitadel está down há > 5min.

---

## 4. Mitigação imediata (ordem de menor risco)

### 4.1 Zitadel está down (cloud ou self-host)

**Estratégia base: não desabilitar autenticação como bypass — viola compliance LGPD + BeyondCorp invariante #2 (Zero Trust).**

#### Opção A — Estender TTL da sessão (mitigar churn de sessões válidas)

Aumenta tempo de vida do `ms_session` cookie de modo que usuários já logados não precisem re-introspect imediatamente:

```bash
# ✅ Aumentar SESSION_TTL_SECONDS de 3600 (1h) → 86400 (24h)
railway variables set --service api-prod SESSION_TTL_SECONDS=86400

# ✅ Estender cache introspection (Redis) de 5min → 30min
railway variables set --service api-prod AUTH_CACHE_TTL_SECONDS=1800

railway service restart api-prod
```

**Trade-off**: usuários revogados (banidos / desligados) podem manter sessão até 30min após revoke. Aceitável durante incident.

#### Opção B — Stale cache fallback (DT-003 — Sprint 10)

Não disponível em M1. Quando implementado: circuit breaker abre → serve `AuthUser` do Redis (mesmo TTL expirado, marker `stale=true`) por até 2x TTL adicional. Loga warning. Quando Zitadel volta, fecha breaker e refresca.

#### Opção C — Comunicar (NUNCA desabilitar `Authenticate`)

Mensagem clara para usuários:

```text
"Login temporariamente indisponível. Sessões ativas continuam funcionando.
Tente novamente em alguns minutos."
```

### 4.2 Zitadel verde mas nosso endpoint falha

| Causa | Sintoma | Mitigação |
|---|---|---|
| Cert TLS issuer expirou | `x509: certificate has expired` em logs | Rotacionar cert (Let's Encrypt renew) — ver §5.3 |
| `ZITADEL_KEY_PATH` não-acessível | Service account JSON faltando após deploy | Re-injetar via `ZITADEL_KEY_JSON` env var (`docker-entrypoint.sh` recria arquivo) |
| Network policy Railway broken | Timeouts 100% em introspection | Restart pods + verificar Railway service connectivity |
| Bug nosso em `oidc_handler.go` (deploy recente) | Apenas `/auth/callback` quebra | **Rollback** (`rollback.md`) |
| Rate limit `/auth/*` muito agressivo | Logs `429 rate_limited` em massa | Bump temporário `AUTH_RATE_LIMIT_RPM=60` (de 20) + restart |

### 4.3 Restart Zitadel pod (self-host Railway)

```bash
# ✅ Sprint 9.19 ainda pendente — Zitadel managed em Railway
# Quando estiver: PostgreSQL próprio do Zitadel é separado do nosso DB
railway service restart zitadel-prod

# Verificar healthcheck pós-restart
curl -s https://auth.mastersindico.com.br/debug/healthz
# expected: 200 OK
```

Se Zitadel cloud → não temos botão; abrir ticket support imediatamente (§8).

### 4.4 PostgreSQL do Zitadel saturado (self-host)

Zitadel self-host usa **DB próprio** (não compartilhado com nosso `master-sindico` DB):

```bash
# ✅ Verificar status do PG do Zitadel
railway connect postgres-zitadel-prod
```

```sql
-- Connections ativas
SELECT count(*), state FROM pg_stat_activity GROUP BY state;
-- > 80% do max_connections = saturação
```

Se saturado: bump connections + investigar query lenta (`pg_stat_statements`).

---

## 5. Recovery

### 5.1 Verificar volta ao baseline

```bash
# ✅ Smoke test login completo
SMOKE_USER="smoke-prod@mastersindico.com.br"
# fluxo manual via browser ou Playwright;
# expected: redirect Zitadel → callback → cookie ms_session set → /api/v1/auth/me 200

# ✅ Métricas Grafana voltam ao baseline
# - /auth/login 2xx > 99% por 10min
# - introspection p95 < 200ms
# - cache hit rate > 70%
```

### 5.2 Reverter mitigações (TTL estendido)

Após confirmação 30min de baseline:

```bash
railway variables set --service api-prod SESSION_TTL_SECONDS=3600
railway variables set --service api-prod AUTH_CACHE_TTL_SECONDS=300
railway service restart api-prod
```

### 5.3 Re-emitir certificado (se cert expirou)

Self-host com Let's Encrypt automatizado:

```bash
# ✅ Forçar renovação cert (Railway gerencia automaticamente; este é fallback)
railway run --service zitadel-prod -- certbot renew --force-renewal

# ✅ Validar novo cert
echo | openssl s_client -servername auth.mastersindico.com.br \
  -connect auth.mastersindico.com.br:443 2>/dev/null \
  | openssl x509 -noout -dates
# notAfter > 30d à frente
```

### 5.4 Invalidar cache Redis (se introspection retornou dados ruins durante incident)

```bash
railway connect redis-prod
```

```redis
# ✅ Limpar cache auth — usuários re-introspect na próxima request
EVAL "for i,k in ipairs(redis.call('keys', ARGV[1])) do redis.call('del', k) end" 0 'ms:auth:*'
```

Custo: pico de introspection requests assim que cache esvaziar. Aceitável pós-recovery.

---

## 6. Comms templates

### 6.1 Slack `#ms-ops`

```text
[INCIDENT] Sev P0 — Zitadel auth down
Detectado: <HH:MM UTC> via <PagerDuty|customer report>
Sintoma: <% logins falhando> | <introspection timeout>
Status Zitadel: <https://status.zitadel.com — indicator | self-host: down/up>
Impacto:
  - Novos logins: bloqueados
  - Sessões ativas: ~<N> ainda válidas (TTL ~<X>min)
Mitigação: <TTL estendido | restart pod | aguardando provider>
Investigando. Updates a cada 10min.
```

### 6.2 Status page (público)

```text
Investigando — Login temporariamente indisponível
Estamos resolvendo um problema com nosso provedor de autenticação.
Se você já está logado, sua sessão continua funcionando.
Novos logins serão restabelecidos em breve.
```

### 6.3 E-mail cliente (se outage > 1h)

```text
Assunto: [Master Síndico] Login temporariamente indisponível

Olá,

Detectamos uma indisponibilidade em nosso sistema de login entre <HH:MM> e
<HH:MM> de <DD/MM>.

O que isso significou:
- Quem estava logado continuou usando o app sem interrupções.
- Quem tentou logar nesse intervalo recebeu erro temporário.

Nenhum dado foi exposto. O sistema já voltou ao normal.

Se ainda estiver com dificuldade, responda este e-mail.

Equipe Master Síndico
```

### 6.4 ANPD (LGPD)

**Aplicável SE Zitadel reportar breach** (cloud ou self-host):

- Zitadel processa autenticação → expõe email + nome + claim de role
- Se vazou → é **PII breach P0** → seguir `cross-tenant-leak.md` §6
- T+1h DPO interno; T+72h ANPD via Res. 15/2024

Não-LGPD: outage simples (Zitadel down sem leak) **não** dispara notificação ANPD; apenas postmortem interno.

---

## 7. Pós-incidente

### 7.1 Postmortem (≤ 48h, blameless)

Trigger: qualquer outage Zitadel > 5min é **postmortem obrigatório** (§6.1 observability — auth = single point of failure).

### 7.2 Métricas a coletar

- Duração total (T_detect → T_resolved)
- Logins falhados (de Sentry + Grafana)
- Sessões ativas durante incident (mantiveram acesso)
- Sessões que expiraram durante (forçaram re-login pós-recovery)
- TTR vs target (P0 = ≤ 30min)
- Error budget consumido (% trimestral identity SLO)

### 7.3 Action items típicos

- **DT-003** (Sprint 10): circuit breaker `sony/gobreaker` em `zitadel_provider.go` + stale cache fallback
- **A-024** (Sprint 10): elevar `RawBody()` a método público em `contracts.Context` (caso debug seja necessário)
- Documentar runbook se variante nova (cert, network, DB do Zitadel)
- Migrar para Zitadel Cloud se self-host se mostrar instável (decisão Tech Lead)
- Aumentar redundância: 2 issuers (primary/fallback) — investigar viabilidade Zitadel

---

## 8. Escalação

| Quando | Quem | Como |
|---|---|---|
| Sev P0 > 15min | Tech Lead + DPO (sempre, por precaução LGPD) | PagerDuty escalation |
| Self-host Zitadel down > 30min | DevOps + Tech Lead | decisão restart vs migrar para Zitadel Cloud emergencial |
| Zitadel Cloud down (se cloud) | Zitadel Support | <https://zitadel.com/contact> + status page; planos enterprise têm SLA |
| Cert TLS expirou em prod | DevOps | renovar imediatamente; documentar gap em prevenção (alert pre-expiry 30d) |
| DB Zitadel corrompido | Tech Lead + DPO | DR drill aplicável (`dr-drill.md`) — restore PITR |

Contatos:

- Zitadel admin master: ver `1Password / Master Síndico / Zitadel`
- Zitadel Cloud Support (se aplicável): support@zitadel.com
- DPO interno: `dpo@mastersindico.com.br`
- DevOps oncall: PagerDuty rotation

---

## 9. LGPD checklist

- [ ] Zitadel declarou breach? (verificar e-mail security@zitadel.com)
- [ ] Logs da nossa API expuseram tokens em claro? (`grep -E 'Bearer [A-Za-z0-9]' sentry/exports/`)
- [ ] Logs expuseram email/CPF do `IDToken` claims? (não devemos logar; verificar)
- [ ] Sessões antigas (pré-incident) ficaram válidas mais tempo que TTL configurado? (extensão de TTL durante recovery)
- [ ] Se PII vazou → escalar `cross-tenant-leak.md` §6 (T+1h DPO; T+72h ANPD)

Zitadel é processador (Art. 5º, VIII LGPD); nós controladores. Em breach lateral, **somos** responsáveis pela notificação ANPD + titulares.

---

## 10. Vizinhos

- [[../incident]] — fluxo geral incident response
- [[../rollback]] — procedimento rollback Railway
- [[../../11-audit/postmortems/template]]
- [[../../03-subprojects/backend/security]] §4 (autenticação Zitadel OIDC), §16 (DT-003 circuit breaker)
- [[../../02-architecture/observability]] §3.1 (SLO identity 99.5%)
- [[../../08-security/threat-model]] §6.3 (Zitadel compromise), §4.1 (auth surface STRIDE)
- [[../../01-domain/bounded-contexts|bounded-contexts]] — identity context
- [[cross-tenant-leak]] — caso PII vaze
- [[stripe-down]] — billing depende de auth upstream
