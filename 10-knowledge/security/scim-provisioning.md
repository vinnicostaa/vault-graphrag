---
title: SCIM — System for Cross-domain Identity Management
type: concept
tags:
  - security
  - scim
  - provisioning
  - identity
  - lifecycle
  - iam
doc-oficial: https://datatracker.ietf.org/doc/rfc7644/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - SCIM
  - System for Cross-domain Identity Management
  - SCIM 2.0
---

# SCIM — System for Cross-domain Identity Management

**O que é**: standard REST API para **provisionamento automatizado** de usuários e grupos entre IdP e aplicações (RFC 7644, RFC 7643, 2015). Elimina onboarding/offboarding manual em SaaS. Cliente (IdP) ↔ Servidor (app).

> SSO resolve login. SCIM resolve **o resto**: criar conta quando user entra, atualizar quando muda cargo, **desativar quando sai**. Offboarding em 1 clique na ferramenta central.

## Por que SCIM existe

Sem SCIM:
- Admin cria user em Okta.
- Admin cria user em Slack (manual).
- Admin cria user em Jira (manual).
- Admin cria user em Figma (manual).
- ...repete N vezes.
- User sai → admin esquece de desativar em 2 das 30 apps.

**Consequência**: 30% das enterprises deixam contas ativas de ex-funcionários (Ponemon research). Vetor de ataque.

Com SCIM:
- Admin cria user em Okta → Okta faz POST SCIM para cada app → contas criadas automaticamente.
- Admin desativa user → PATCH para todas as apps → desativadas automaticamente.

## Fluxo canônico

```
┌────────────────────┐       SCIM API         ┌────────────────────┐
│ SCIM Client (IdP)  │ ──────POST /Users─────▶│ SCIM Server (App)  │
│ Okta, Entra ID,    │                        │ Slack, Jira, etc.  │
│ OneLogin, ...      │ ──────PATCH /Users/X──▶│                    │
│                    │ ──────DELETE /Users/X─▶│                    │
└────────────────────┘                        └────────────────────┘
```

IdP é **source of truth** de identidade. Apps são downstream — refletem estado vindo do IdP.

## Recursos SCIM

Schema padrão:
- `/Users` — usuários individuais.
- `/Groups` — grupos / roles.
- `/Me` — user corrente (query).
- `/ServiceProviderConfig` — metadata de capabilities da app.
- `/Schemas` — schema dos recursos.
- `/ResourceTypes` — tipos de recursos disponíveis.

## Operações

- `GET /Users` — lista (paginada, filtros).
- `GET /Users/{id}` — busca por ID.
- `POST /Users` — criar.
- `PUT /Users/{id}` — replace inteiro.
- `PATCH /Users/{id}` — alteração parcial (RFC 6902 JSON Patch).
- `DELETE /Users/{id}` — hard delete (ou soft, via `active: false` em PATCH).

## Payload de User SCIM 2.0

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id": "2819c223-7f76-453a-919d-413861904646",
  "externalId": "bjensen",
  "userName": "bjensen@example.com",
  "name": {
    "formatted": "Ms. Barbara J Jensen, III",
    "familyName": "Jensen",
    "givenName": "Barbara",
    "middleName": "Jane",
    "honorificPrefix": "Ms.",
    "honorificSuffix": "III"
  },
  "displayName": "Babs Jensen",
  "emails": [{ "value": "bjensen@example.com", "type": "work", "primary": true }],
  "active": true,
  "groups": [{ "value": "group-id-123", "display": "Engineering" }],
  "meta": {
    "resourceType": "User",
    "created": "2026-01-15T10:00:00Z",
    "lastModified": "2026-04-24T12:00:00Z",
    "version": "W/\"a330bc54f0671c9\"",
    "location": "https://example.com/scim/v2/Users/2819c223-..."
  }
}
```

Extensões:
- `urn:ietf:params:scim:schemas:extension:enterprise:2.0:User` — manager, department, employeeNumber, costCenter.
- Custom via `urn:<vendor>:...`.

## Auth no SCIM API

- **OAuth 2.0 Bearer token** (mais comum).
- **Basic Auth** (legado; algumas apps).
- **Client Certificate mTLS** (enterprise).

Token deve ter scope restrito (provisioning only) + rotação.

## JIT — Just-In-Time provisioning

Alternativa/complemento ao SCIM:
- User primeira vez loga via SSO → app cria automaticamente a conta com claims do JWT/SAML.
- Sem fricção inicial; mas **não gerencia updates/offboarding**.

**Best practice**: JIT para **provisionamento inicial** + SCIM para **lifecycle** (update/delete).

## Webhook-based alternativas

Alguns apps expõem webhook em vez de SCIM:
- IdP dispara `user.deactivated` → webhook acoplado.
- Funciona para cases onde RESTful SCIM é overkill.
- Menos padronizado; lock-in por app.

Para compliance enterprise SOC2, **SCIM é preferido** (auditável).

## Padrão canônico — implementar SCIM server na app

```ts
// GET /scim/v2/Users (list)
app.get('/scim/v2/Users', async (c) => {
  const filter = c.req.query('filter')       // `userName eq "alice"`
  const startIndex = Number(c.req.query('startIndex') ?? 1)
  const count = Math.min(Number(c.req.query('count') ?? 100), 200)

  const users = await userRepo.queryScim({ filter, startIndex, count })
  return c.json({
    schemas: ['urn:ietf:params:scim:api:messages:2.0:ListResponse'],
    totalResults: users.total,
    startIndex,
    itemsPerPage: users.items.length,
    Resources: users.items.map(toScimUser)
  })
})

// POST /scim/v2/Users (create)
app.post('/scim/v2/Users', async (c) => {
  const body = await c.req.json<ScimUser>()
  const user = await userRepo.createFromScim(body)
  c.status(201)
  return c.json(toScimUser(user))
})

// PATCH /scim/v2/Users/:id
app.patch('/scim/v2/Users/:id', async (c) => {
  const id = c.req.param('id')
  const { Operations } = await c.req.json<{ Operations: PatchOp[] }>()
  const user = await userRepo.applyPatch(id, Operations)
  return c.json(toScimUser(user))
})
```

**Bibliotecas**: `scim2-parse-filter`, `scim-patch` (Node); `scim-for-sdkless-apps` (patterns).

## Anti-patterns

1. **Hard delete** sem audit — SOC2 exige trilha. Prefira `active: false` (soft) + archive depois.
2. **SCIM sem auth** — API de user management exposta é catástrofe. Token + audit.
3. **Aceitar `id` do IdP como PK** — usar `externalId` SCIM, PK interna separada. IdP pode reemitir com outro id.
4. **Não implementar PATCH** — forçar PUT perde optimistic concurrency, exige replay completo.
5. **Ignore `etag` / `meta.version`** — race conditions.
6. **Rate limit fraco em SCIM endpoint** — IdP legitimamente dispara centenas de requests em sync batch; distinguir de abuse.
7. **Sync bidirecional** — default é IdP → App. Fazer App → IdP é risco (quem é source of truth?).

## Como grandes empresas implementam

- **Slack**: SCIM Enterprise Grid obrigatório; integra com Okta, Entra ID, Google Workspace.
- **Zoom**: SCIM para enterprise tier; grupos sincronizados.
- **GitHub Enterprise**: SCIM para user sync + team provisioning.
- **Figma Enterprise**: SCIM obrigatório para sync de workspace.
- **1Password Business**: SCIM bridge.
- **WorkOS** (SaaS): **SCIM-as-a-Service** — startup implementa via WorkOS API em vez de SCIM direto (abstrai N IdPs).

## IdPs que suportam SCIM client

| IdP | Custom apps? |
|---|---|
| **Okta** | Sim; também catalogo SCIM integrations |
| **Microsoft Entra ID** | Sim; "Enterprise Applications" com SCIM config |
| **Google Workspace** | Sim via App Directory |
| **OneLogin / Ping** | Sim |
| **Auth0** | Sim; menos comum no fluxo B2C |
| **Zitadel** | Suporta SCIM 2.0 v2 |

## Relações

- **Componente de**: [[iam]] (lifecycle automation)
- **Combina com**: [[oidc]] / [[saml-2]] (authn); SSO só resolve login, SCIM resolve account lifecycle
- **Complementa**: JIT provisioning (primeira criação via login)
- **Cross-ref**: [[identity-provider]] (IdP é SCIM client); [[sso]]

## Fontes

- [RFC 7643 — SCIM Core Schema](https://datatracker.ietf.org/doc/rfc7643/) (consultada 2026-04-24)
- [RFC 7644 — SCIM Protocol](https://datatracker.ietf.org/doc/rfc7644/) (consultada 2026-04-24)
- [SCIM homepage](https://www.simplecloud.info/) (consultada 2026-04-24)
- [Okta SCIM Guide](https://developer.okta.com/docs/concepts/scim/) (consultada 2026-04-24)
- [Microsoft Entra ID — Provisioning with SCIM](https://learn.microsoft.com/en-us/entra/identity/app-provisioning/use-scim-to-provision-users-and-groups) (consultada 2026-04-24)
- [WorkOS Directory Sync](https://workos.com/docs/directory-sync) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[iam]]
- [[identity-provider]]
- [[sso]]
- [[oidc]]
- [[saml-2]]
