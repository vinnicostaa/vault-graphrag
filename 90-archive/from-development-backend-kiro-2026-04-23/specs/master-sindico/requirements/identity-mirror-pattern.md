---
title: Identity Mirror Pattern — User/Session ↔ Zitadel
version: 1.0
status: stable
type: requirement
domain: identity
tags: [identity, zitadel, bc, design-decision]
---

# Identity Mirror Pattern — User/Session espelhados do Zitadel

## Resumo

O Master Síndico mantém **mirror parcial local** das entidades `User` e `Session` que o Zitadel também tem. Não é redundância — é padrão arquitetural deliberado chamado **"External Identity with Local Projection"**, usado por GitHub Enterprise, Slack+Okta, Shopify e outros.

Esta nota documenta a decisão, o que é duplicado, o que **não** é, e por que cada parte existe.

## Source of Truth (o que manda em cada dado)

| Dado | Source of truth | Motivo |
|---|---|---|
| Password hash, MFA, reset tokens, email verification | **Zitadel** | Função primária do IdP — nunca replicar |
| Identidade (email, username, phone, `email_verified`) | **Zitadel** | IdP é autoritativo; API consome via claims + introspection |
| Atributos de domínio (`role`, `role_type`, `plan_id`, `visibility_ready`, `banned`, `ban_expires_at`) | **DB Master Síndico** | Semântica do produto, não do IdP |
| Session do IdP (login único, SSO) | **Zitadel** | `zitadel_session_id` vem no introspection |
| Session da **aplicação** (device_fingerprint, 1-device policy, audit) | **DB Master Síndico** | BeyondCorp enforcement que Zitadel não faz |

## Por que `users` local tem que existir

1. **Foreign keys** — `subscriptions`, `condominiums`, `timeline_entries`, `assemblies` precisam de FK para `users.id`. Referenciar ID externo do Zitadel em toda tabela = sem integridade referencial, sem JOIN.
2. **ABAC no request path** — cada request precisa de `{role, role_type, plan_tier, tenant_id}` para autorização. Buscar no Zitadel = 1 round-trip extra por endpoint. Inaceitável.
3. **Atributos que Zitadel não tem** — `role_type` (principal/auxiliary/titular/dependent), `visibility_ready`, `banned`, `ban_expires_at`.
4. **Read models de domínio** — JOIN `users → subscriptions → plans` é query comum. Impossível sem mirror.

## Por que `sessions.Create` tem que existir

Existem **dois tipos de sessão** que não se sobrepõem:

- **Zitadel session** → login do usuário no IdP (SSO, "você está logado no Zitadel")
- **Application session (MS)** → vínculo `(user, device)` *nesta aplicação* — o coração do BeyondCorp

O fluxo de login faz **5 passos** após o callback OIDC:

```
1. Zitadel valida JWT (introspection) → retorna claims + zitadel_session_id
2. UPSERT users (sync mirror)                                  ← sem isto, sem FK
3. DELETE sessions WHERE user_id=X AND device_fingerprint ≠ Y   ← 1-device policy
4. INSERT sessions(user_id, zitadel_session_id, device_fp, ...) ← o Create
5. Segue para handler
```

Sem o `Create` da session local:
- **Nenhum enforcement de 1-device** (Zitadel não faz)
- **Nenhum audit trail** (quem acessou de onde, quando)
- **Nenhum device fingerprint** (anti-fraude, detecção de roubo de token)
- **Nenhuma invalidação cruzada** (login no celular invalida desktop)

## Padrão de mercado

É o padrão **"External Identity with Local Projection"**:

- **GitHub Enterprise** com SAML/OIDC — mantém `users` local mirrored
- **Stripe** — `customer_id` (deles) + `user_id` (cliente)
- **Slack + Okta** — sessions locais com device binding
- **Shopify** — profile local + identity externo

## O que **não** duplicar (regra dura)

- ❌ Password hash — Zitadel-only
- ❌ MFA secrets — Zitadel-only
- ❌ Reset tokens — Zitadel-only
- ❌ Email verification state como fonte autoritativa — consumir via claim `email_verified`, não persistir como source of truth
- ❌ Refresh tokens — Zitadel gerencia ciclo de vida; backend apenas valida access token via introspection

## Schema de referência

Tabela `users` (mirror):

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    zitadel_id TEXT NOT NULL UNIQUE,                 -- link para Zitadel (source of truth de identidade)
    email TEXT NOT NULL,
    username TEXT,
    phone TEXT,

    role user_role NOT NULL,                         -- semântica do produto
    role_type user_role_type,                        -- sub-contexto operacional
    plan_id UUID REFERENCES plans(id),               -- cache ABAC
    status user_status NOT NULL DEFAULT 'active',
    visibility_ready BOOLEAN NOT NULL DEFAULT false,

    device_fingerprint TEXT,                         -- BeyondCorp
    last_ip TEXT,

    banned BOOLEAN NOT NULL DEFAULT false,
    ban_reason TEXT,
    ban_expires_at TIMESTAMPTZ,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMPTZ
);

CREATE INDEX idx_users_zitadel_id ON users(zitadel_id);
CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_role_plan ON users(role, plan_id) WHERE deleted_at IS NULL;
```

Tabela `sessions` (enforcement local):

```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    zitadel_session_id TEXT NOT NULL,                -- link para Zitadel session

    device_fingerprint TEXT NOT NULL,
    ip_address INET NOT NULL,
    user_agent TEXT NOT NULL,

    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ,                          -- 1-device: outra session revogou esta

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_sessions_user_id_active ON sessions(user_id)
    WHERE revoked_at IS NULL AND expires_at > NOW();
CREATE INDEX idx_sessions_zitadel_id ON sessions(zitadel_session_id);
```

## Fluxo de sincronização

### Primeiro login (signup implícito)

```
1. User faz login OIDC → Zitadel autentica
2. Zitadel redirect callback com code
3. Backend troca code por access_token
4. Backend introspection → claims (email, role, zitadel_user_id, zitadel_session_id)
5. Backend: UPSERT users (role vem dos claims; outros campos defaults)
6. Backend: cria session local (device fingerprint do header X-Device-Fingerprint)
7. Backend: set cookie access_token httpOnly no domínio raiz
```

### Login subsequente

```
1. Request vem com cookie access_token
2. Middleware Authenticate: introspection (cache Redis 5min)
3. Verifica user local — se não existe, UPSERT (edge case: user deletado no DB mas ativo no Zitadel)
4. Verifica session local por device_fingerprint:
   - Match: ok, segue
   - Mismatch: invalida session anterior (1-device), cria nova
   - Nenhuma: cria nova
5. Segue para ABAC
```

### Mudança de role no Zitadel

```
Admin muda role de user-X no Zitadel.
Próxima request de user-X:
 - Introspection retorna claims com nova role
 - Cache Redis tem claims velhos (até 5min de TTL)
 - Opções:
   (a) Aceitar staleness de 5min (default atual)
   (b) Webhook Zitadel → purge `ms:auth:{zitadel_user_id}` no Redis (instantâneo)
```

**Decisão em aberto**: ver D-001 em `.kiro/STATE.md`. Opção (b) será implementada no Sprint 4 junto com o primeiro cenário admin real.

### User deletado no Zitadel

```
Admin deleta user no Zitadel.
 - Introspection falha → middleware rejeita request com 401
 - Job noturno sincroniza `users.deleted_at = now()` para zitadel_id que retornam "not found"
 - FKs com ON DELETE RESTRICT protegem dados históricos (subscriptions, timeline, etc)
```

## Edge cases

1. **Email mudou no Zitadel**: próxima introspection traz novo email; UPSERT atualiza. Não afeta FKs (que usam `id` interno, não email).
2. **Role removida de user**: `role = ''` nos claims. Middleware rejeita com `403 NO_ROLE_ASSIGNED` **antes** de UPSERT — evita salvar user com role vazia (T15 em requirements.md).
3. **Tentativa de login com user banido**: `banned = true` + `ban_expires_at` no passado ou NULL → 403. Banimento local não afeta Zitadel (user ainda pode logar no IdP, mas não consegue operar no MS).

## Invariantes protegidas

- `users.zitadel_id` é UNIQUE — 1 user MS = 1 user Zitadel
- `users.role` nunca é string vazia — middleware rejeita antes do UPSERT
- Toda `session` tem `user_id` que existe em `users` (FK)
- Um user tem no máximo 1 session ativa (1-device enforcement via `DELETE` antes do `INSERT`)
- `sessions.zitadel_session_id` nunca é re-usado após revogação — session local nova é `INSERT`, não `UPDATE`

## Referências

- `steering/security.md` seção "Zitadel — Introspection via JWT Profile"
- `steering/go-patterns.md` seção "BeyondCorp Middleware Stack"
- Requirements Req 1 (Autenticação), Req 2 (ABAC), Req 3 (Registro)
- Decisões Técnicas T1–T15 em `requirements.md` (ou `requirements/product-overview.md`)
