---
title: Delegation & Impersonation — act-as, on-behalf-of (RFC 8693)
type: concept
tags:
  - security
  - delegation
  - impersonation
  - token-exchange
  - oauth
  - authorization
doc-oficial: https://datatracker.ietf.org/doc/rfc8693/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Token Exchange
  - Act-As
  - On-Behalf-Of
  - Impersonation
---

# Delegation & Impersonation

**O que é**: cenários onde um principal (user, service) executa ação **em nome de** outro. Duas semânticas:

- **Delegation** (act-as): principal recebe autorização **limitada** para agir como outro, sendo audit trail sabe que é A agindo em B.
- **Impersonation**: principal adota identidade completa de outro — audit vê como se fosse o outro. **Mais arriscado**; exige compliance rigoroso.

OAuth 2.0 **Token Exchange** (RFC 8693) é o padrão técnico. Cedar/IAM/Zanzibar implementam modelos próprios.

## Casos de uso

### Admin "view as user" (support)

Admin do Stripe precisa debugar conta do cliente. Clica "view as" → vê dashboard como user. Audit registra: "admin X impersonou user Y em data Z para propósito W".

### Service calling API on behalf of user

Backend usa user access token para chamar terceiro (ex: app chama GitHub API com token OAuth do user).

### Cross-service M2M com contexto de user

Service A processa request de user X → chama service B. B precisa saber que request veio de user X (para ABAC) sem A ter credencial M2M genérica.

### Parent resource → child resource (delegation chain)

User autoriza app → app autoriza sub-processor → sub-processor não pode re-delegar.

## OAuth 2.0 Token Exchange (RFC 8693)

Padrão oficial. Formato:

```
POST /token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&subject_token=<token-representando-user>
&subject_token_type=urn:ietf:params:oauth:token-type:access_token
&actor_token=<token-representando-admin-ou-service>     (opcional)
&actor_token_type=urn:ietf:params:oauth:token-type:access_token
&requested_token_type=urn:ietf:params:oauth:token-type:access_token
&audience=https://api.example.com
&scope=read:users
```

**Resposta**:
```json
{
  "access_token": "<new-token>",
  "token_type": "Bearer",
  "expires_in": 300,
  "issued_token_type": "urn:ietf:params:oauth:token-type:access_token"
}
```

Novo token tem claims:
- `sub` = user (subject).
- `act` = admin/service (actor) — pode ser aninhado para delegation chain.
- `aud` = API que vai consumir.
- Scope limitado.

## Claim `act` (actor)

RFC 8693 §4.1 — claim opcional que **preserva chain**:

```json
{
  "sub": "user-alice",
  "act": {
    "sub": "service-etl",
    "act": {
      "sub": "admin-bob",
      "purpose": "data-export",
      "ticket_id": "TICKET-1234"
    }
  },
  "aud": "api",
  "exp": 1735690000,
  "scope": "read:users"
}
```

Audit trail vê: admin-bob → service-etl → acting as user-alice. Compliance-ready.

## Impersonation vs Delegation — quando usar qual

| Critério | Delegation (act-as) | Impersonation |
|---|---|---|
| Audit | Actor + subject separados | Actor invisível (aparece como subject) |
| Compliance | Melhor (cadeia clara) | Arriscado (aparenta ser user) |
| UX downstream | API vê diferente | API vê normal |
| Caso | Support view-as, mostly | Reservado para emergência + compliance approval |

**Regra**: impersonation é **raro** em 2026; prefira delegation com `act` claim. Support "view as" pode ser delegation com flag + audit.

## Padrões operacionais

### Admin view-as (delegation)

```
1. Admin: clica "view as user:alice" no painel.
2. App UI → Backend: POST /impersonate/alice (com admin access token)
3. Backend valida: admin tem permissão `support:impersonate`?
   Sim → emite token novo via Token Exchange:
         sub=alice, act=admin, scope=read only, exp=15min, ticket_id required
4. UI armazena token novo em cookie separado (não substitui admin session).
5. API Requests com novo token → vê sub=alice, ABAC avalia user; audit registra act=admin.
6. "Stop viewing as" → deletar cookie novo; volta ao original.
```

**Constraints operacionais**:
- TTL **muito curto** (10-15min).
- Scope **limitado** (read-only típico; write exige re-auth e approval explícito).
- `purpose` ou `ticket_id` obrigatório.
- Audit log detalhado.
- Notificação opcional ao user impersonado (algumas reguladas).

### Service on-behalf-of user (OAuth standard)

Frontend app delega ao backend:
```
1. Frontend: tem user access token.
2. Backend chama AS: token-exchange subject_token=<user-token>
                     actor_token=<service-token>
                     audience=downstream-api
3. AS valida que service tem trust para on-behalf-of.
4. Backend usa token novo para chamar downstream-api.
5. downstream-api vê sub=user, act=service.
```

## Compliance — quando impersonation é legítima

### LGPD/GDPR perspectiva

Tratamento de dados pessoais por operador terceiro requer **base legal** (consent, legitimate interest, contract). Impersonation passa por "processor" — documentar em DPIA.

### SOX

Impersonation de admin em sistema financeiro **sempre** requer:
- Approval (2-person rule).
- Audit trail imutável.
- Retention 7 anos.
- Purpose declarado.

### SOC 2

Controles CC6.3: acesso privilegiado documentado + reviewed. Impersonation é sub-categoria.

## Implementations

### Zitadel

Tem "Personal Access Token" + "Machine Users" para delegation. Future support de RFC 8693 in progress (verify doc atualizada).

### Auth0

Actions/Rules permitem implementar Token Exchange customizado.

### Keycloak

Built-in Token Exchange (desde 12.x). Configurável via admin UI.

### Okta

Token Exchange via Inline Hooks + custom claims.

### AWS IAM

**STS AssumeRole** com `source_identity` parameter (RFC 8693-aligned). Session policy limita permissões.

### GCP IAM

**Service Account Token Creator** — account A impersonates B com audit no Cloud Audit Logs.

## Anti-patterns

1. **Impersonation sem audit** — bypass compliance. Sempre audit.
2. **Token com `sub` substituído sem `act`** — legítimo é Token Exchange com `act`; só trocar `sub` é manipulação.
3. **TTL longo** (> 1h) em token de impersonation — janela de abuso.
4. **Scope amplo** (`scope=*`) em token delegado — mínimo necessário.
5. **Delegation chain sem limite** — atacante pode encadear infinitamente. Limite ≤ 3 níveis.
6. **Sem `purpose`/`ticket_id`** — audit não sabe por que.
7. **User não notificado** (em casos onde compliance exige) — LGPD/GDPR risk.
8. **Impersonation usável em operações destructivas** (DELETE /users) sem 2-person rule — nunca.
9. **Service-to-service como "impersonate every user"** — anti-delegation; use service credential com escopo amplo próprio.

## Como grandes empresas fazem

- **Stripe**: admin "view as" com TTL curto + ticket requerido + notificação user opcional (configurável).
- **GitHub**: Enterprise admin não pode impersonar usuários padrão (audit/compliance); usa Organization ownership claims.
- **AWS**: STS AssumeRole + SSO Session Tag + CloudTrail registra source_identity.
- **GCP**: Service Account Impersonation com `source_account` log obrigatório.
- **Intercom, Zendesk**: "impersonate mode" com banner visível; audit log; auto-expire.

## Relações

- **Extensão de**: [[oauth-2]] (Token Exchange é grant type)
- **Token format**: [[jwt]] (act claim)
- **Audit requirement**: [[../runtime-antipatterns/op-021-admin-sem-audit|OP-021]]
- **Princípio**: [[least-privilege]] (scope limitado, TTL curto)
- **Compliance**: [[../security/_moc]] (LGPD, SOC2)

## Fontes

- [RFC 8693 — OAuth 2.0 Token Exchange](https://datatracker.ietf.org/doc/rfc8693/) (consultada 2026-04-24)
- [Auth0 — Token Exchange](https://auth0.com/docs/authenticate/custom-token-exchange) (consultada 2026-04-24)
- [Keycloak — Token Exchange](https://www.keycloak.org/docs/latest/securing_apps/#_token-exchange) (consultada 2026-04-24)
- [AWS STS AssumeRole with source_identity](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_control-access_monitor.html) (consultada 2026-04-24)
- [GCP Service Account Impersonation](https://cloud.google.com/iam/docs/service-account-impersonation) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[oauth-2]]
- [[jwt]]
- [[least-privilege]]
- [[iam]]
- [[security-principles]]
