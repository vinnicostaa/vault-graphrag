---
title: Zitadel — Patterns (introspection cache, rotation, ABAC, multi-tenant)
type: note
tags: [provider, zitadel, identity, oidc, patterns]
source: https://zitadel.com/docs
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Zitadel Patterns]
---

# Zitadel — Patterns

Padrões aplicáveis a qualquer aplicação que use Zitadel como IdP. Cada padrão tem o problema que resolve, a forma canônica e o trade-off.

## 1. Introspection + cache TTL curto

**Problema**: introspection é uma chamada de rede a cada request; sem cache, latência soma. Sem revogação, JWT vive até `exp`.

**Forma**: cachear o resultado da introspection por **5 minutos** (ou menos em rotas sensíveis), chaveado pelo hash do token. Entrada do cache guarda `claims` + `expiresAt`.

```go
type entry struct {
    claims    Claims
    cachedAt  time.Time
}

func (s *Store) Introspect(ctx context.Context, token string) (Claims, error) {
    key := sha256hex(token)
    if e, ok := s.cache.Get(key); ok && time.Since(e.cachedAt) < 5*time.Minute {
        return e.claims, nil
    }
    c, err := s.remote.Introspect(ctx, token)
    if err != nil || !c.Active {
        return Claims{}, ErrInactive
    }
    s.cache.Set(key, entry{claims: c, cachedAt: time.Now()})
    return c, nil
}
```

**Trade-off**: janela de até 5 min entre revogação real e efeito local. Aceitável para a maioria dos produtos; para rotas críticas (pagamento, delete de conta), passar `noCache=true` e chamar direto.

**Invalidação ativa**: configurar um `Action` em `post-logout` / `post-revocation` que dispara webhook para o app; app invalida as entradas do user. Combina cache com freshness perto de real-time.

## 2. Refresh token rotation (RFC 6749 §6)

**Problema**: refresh token é credencial longa. Se vazar, atacante mantém sessão indefinidamente.

**Forma**: sempre que usar um refresh token, Zitadel devolve um **novo** refresh no mesmo response; o antigo é invalidado imediatamente. App **precisa** persistir o novo.

```go
src := oauth2Config.TokenSource(ctx, &oauth2.Token{RefreshToken: current})
next, err := src.Token()
if err != nil { return err }

// atomically swap — perder essa linha = session morta no próximo refresh
if err := store.SaveRefresh(userID, next.RefreshToken); err != nil {
    return err
}
```

**Detecção de roubo**: se Zitadel receber o refresh antigo depois do uso, retorna `invalid_grant`. App que recebe esse erro deve **revogar toda a sessão** do user (não só o token) — é sinal claro de roubo.

## 3. ABAC via custom claims (Action `pre-access-token`)

**Problema**: permissões do produto (`plan_tier`, `tenant_id`, `device_id`) não moram no IdP. Buscá-las a cada request duplica round-trips.

**Forma**: escrever uma `Action` em `pre-access-token` que consulta o produto (ou usa metadata guardada no user Zitadel) e injeta claims custom no token.

```javascript
// Action: pre-access-token
function addCustomClaims(ctx, api) {
  const meta = ctx.v1.user.metadata || []
  const planTier = meta.find(m => m.key === 'plan_tier')?.value || 'base'
  api.v1.claims.setClaim('plan_tier', planTier)

  const tenantId = ctx.v1.user.human.profile.preferredLanguage  // exemplo
  api.v1.claims.setClaim('tenant_id', tenantId)
}
```

Middleware no app lê `plan_tier`, `tenant_id`, `roles` direto do token — ABAC engine aplica em [[../../security/beyond-corp]] estilo.

**Regra**: claims custom **complementam**, não substituem, o matrix ABAC server-side. Token pode ser antigo; política canônica fica no app.

## 4. `sub` como chave de mapeamento local

**Problema**: `email` e `preferred_username` são **mutáveis** em Zitadel. Usar como FK no DB significa renomeação de email quebra histórico e permissões.

**Forma**: toda linha que aponta para um user carrega `external_sub` (`TEXT NOT NULL UNIQUE` em Postgres), espelhando o `sub` do token. `email` vira atributo de exibição sincronizado periodicamente, nunca chave.

```sql
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_sub    TEXT NOT NULL UNIQUE,    -- claim `sub` do Zitadel
    email           CITEXT,                   -- display only
    display_name    TEXT,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_seen_at    TIMESTAMPTZ
);
CREATE INDEX idx_users_email ON users (email);
```

Toda autenticação faz `UPSERT ... ON CONFLICT (external_sub)` sincronizando atributos mutáveis.

## 5. Action `pre-creation` / `post-authentication` para side-effects

- **`pre-creation`**: bloquear signup por política (ex: domínio de email não permitido).
- **`post-authentication`**: disparar webhook para o app criar linha em `users` já no primeiro login, em vez de fazer lazy-provision no middleware.

Lazy-provision no middleware funciona, mas introduz uma classe de bugs de concorrência (dois requests do mesmo user novo criando duas linhas). Action + idempotency key no app elimina isso.

## 6. Multi-tenant via Organization (não inventar)

**Problema**: tentação de criar `tenant_id` no app e manter todos users numa única `Organization` do Zitadel.

**Forma**: cada tenant do produto = uma `Organization` no Zitadel. Benefícios:

- Branding e políticas de login por tenant (domínio, logo, cor).
- Audit trail isolado por organization.
- Domain verification separado (cada tenant pode ter `@tenant.example.com`).
- Admin delegation natural — um admin do tenant só vê users dele.

`Project` do produto é compartilhado; `Project Grant` cede acesso a cada `Organization`. `tenant_id` no app passa a ser derivado do `urn:zitadel:iam:org:id` claim.

## 7. Device fingerprint + 1-device-per-account

Para produtos com requisito forte de BeyondCorp (ver [[../../security/beyond-corp]]):

1. Cliente calcula fingerprint determinístico (UA + platform + screen + canvas hash).
2. Primeiro login registra `(user_sub, fingerprint_hash)` em `user_devices`.
3. Login subsequente com outro fingerprint invalida a sessão anterior (revogar refresh no Zitadel via Management API) e registra o novo.
4. `Action` custom claim pode carregar `device_id` ativo para o app validar.

**Trade-off**: UX mais atrita (user que limpa cookies perde sessão). Combinar com re-auth transparente se MFA estiver ativo e o device_id é validado por device signature.

## 8. Private Key JWT no `Web` application

**Problema**: `client_secret` é uma senha compartilhada; leak = comprometimento total.

**Forma**: configurar `authMethodType=private_key_jwt` na `Application`. Zitadel armazena a chave **pública**; app assina assertion igual ao M2M para autenticar no token endpoint. Rotação de chave = gerar nova, subir pública, derrubar antiga — sem janela de secret exposto.

## 9. Observabilidade mínima

Todo middleware de auth deve expor:

- `auth.introspect.duration_ms` (histograma) — alerta P95 > 150ms.
- `auth.introspect.cache_hit_ratio` — alerta < 0.8 fora de deploys.
- `auth.refresh.rotation_failures` — qualquer > 0 em 5 min é incidente (roubo possível).
- `auth.token.invalid_count` por motivo (`expired`, `revoked`, `audience_mismatch`).

Métricas ficam em contexto de tenant + user **hash**, nunca do `sub` em claro.

## Vizinhos no grafo

- [[_moc]]
- [[overview]]
- [[integration-guide]]
- [[antipatterns]]
- [[../_moc]]
- [[../../security/beyond-corp]]
- [[../../security/_moc]]
