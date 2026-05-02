---
title: ABAC — Attribute-Based Access Control
type: concept
tags:
  - security
  - abac
  - authorization
  - attributes
  - nist
doc-oficial: https://csrc.nist.gov/publications/detail/sp/800-162/final
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Attribute-Based Access Control
  - ABAC
---

# ABAC — Attribute-Based Access Control

**O que é**: modelo de autorização onde decisão deriva de **atributos** do sujeito, recurso, ação e contexto (NIST SP 800-162). Mais expressivo que [[rbac|RBAC]] — permite regras como "user pode editar posts do seu próprio tenant, durante business hours, em device com MFA recente".

> **Esta nota é magra por design** — implementação concreta (Go engine, matriz de regras, denial reasons estruturados) vive em [[security-principles#ABAC-Engine]]. Aqui fica apenas o conceito + comparação + quando adotar.

## 4 categorias de atributos (NIST SP 800-162)

```
Decision = f(
  Subject attributes,       ← role, department, clearance, tenant_id, MFA_level
  Object attributes,        ← owner_id, classification, tenant_id, tags
  Action attributes,        ← verb (read/write/delete), sensitivity
  Environment attributes    ← time, IP, geolocation, device_trust
)
```

Exemplos:
- **Subject**: `{ role: "manager", plan_tier: "premium", tenant_id: "acme", mfa_verified: true }`
- **Object**: `{ type: "invoice", owner_id: "u-123", tenant_id: "acme", classification: "internal" }`
- **Action**: `{ verb: "approve", severity: "high" }`
- **Environment**: `{ ip: "10.0.0.5", hour: 14, device_trust: "managed", country: "BR" }`

## Policy Decision Point (PDP) + Policy Enforcement Point (PEP)

Padrão NIST XACML:

```
Client Request
    │
    ▼
┌──────────────────────────────────┐
│ PEP (Policy Enforcement Point)    │   ← handler / middleware
│ coleta atributos + query PDP      │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ PDP (Policy Decision Point)       │   ← engine (OPA, Cedar, matriz Go)
│ avalia policies                   │
└────────────┬─────────────────────┘
             │
             ▼
       Permit / Deny / NotApplicable
```

PEP aplica; PDP decide. Separação crítica para auditabilidade (PDP tem log de "por que"?).

## ABAC vs RBAC (quando migrar)

| Sintoma RBAC | Por que ABAC ajuda |
|---|---|
| Role sprawl (500+ roles) | Colapsar em roles-base + atributos |
| Role por tenant/região/produto | Atributo `tenant_id`/`region` |
| Mesma permissão em vários roles com pequenas variações | Atributos eliminam repetição |
| "User pode X **se** Y" hardcoded em handler | Policy expressa condição |
| Não consegue expressar "ownership" | Atributo `object.owner_id == subject.id` |

## Padrão canônico — ABAC Engine (matriz declarativa)

**Código de referência**: ver [[security-principles#ABAC-Engine]] — struct `ABACContext`, matriz de regras, denial reasons (`role_not_allowed`, `plan_below_required`, `tenant_mismatch`).

Aqui o conceito resumido:

```go
// Regra = combinação de atributos que permite ou nega
type Rule struct {
    Action          string
    Resource        string
    RoleAllowed     []Role
    PlanMin         PlanTier
    RequireTenant   bool
    RequireMFA      bool
    DenyReason      string
}

// Engine avalia em ordem
func (e *Engine) Decide(ctx ABACContext) Decision {
    for _, rule := range e.rules {
        if rule.matches(ctx) {
            return rule.decision()
        }
    }
    return DenyDefault   // deny by default
}
```

### Denial reasons estruturados

Em vez de boolean `granted`, retornar categoria:
```
{ "granted": false, "reason": "plan_below_required" }
```
Cliente reage com UX apropriado; auditor rastreia padrões de negação.

## Decisões de design

### Atributos estáticos vs dinâmicos

- **Estáticos** (role, tenant_id): vêm do token/sessão. Cache aceitável com cautela — ver [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]].
- **Dinâmicos** (`object.owner_id`): query no banco em cada check. Latência extra; inevitável.

### Performance

ABAC puro com 100 regras pode rodar em µs se policy é simples; ms se envolve queries de atributos. Alternativas:
- **Policy compilation** (OPA pré-compila Rego para Go) — execução sub-ms.
- **Cached decisions** com key derivada de atributos estáveis + TTL curto + invalidação ativa.
- **Batch decisions** para listar recursos (return `visible: bool` por item).

## ABAC + RBAC (híbrido — mais comum)

Padrão observado em SaaS modernos:
```
1. Role dá base (ex: "manager" → permissões-base)
2. ABAC refina:
   - tenant_id do recurso = tenant_id do user (sempre)
   - plan_tier >= required_plan (quotas)
   - mfa_verified (operações sensíveis)
   - business_hours (operações financeiras)
```

AWS IAM = exemplo clássico: Role/Policy (RBAC) + Condition keys (ABAC).

## Anti-patterns específicos

1. **ABAC inline em handler** (`if user.tenant_id == resource.tenant_id { ... }`) — espalha regras. Centralize em engine. Ver [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]].
2. **Explosão de atributos** — 50+ atributos por regra vira "spaghetti policy". Mantenha regras atômicas e componha.
3. **Sem denial reason estruturado** — user vê "403 forbidden"; não sabe se é plano, MFA, tenant, horário. Retornar categoria (sem detalhes técnicos).
4. **Atributo `is_admin` binário** — vai virar role sprawl. Prefira role + atributos adicionais.
5. **Avaliar ABAC só no cache hit** — ver [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]].
6. **Não auditar decisões ABAC** — ver [[../runtime-antipatterns/op-021-admin-sem-audit|OP-021]].

## Implementações e engines

| Engine | Modelo | Linguagem de policy |
|---|---|---|
| **AWS IAM** | RBAC + ABAC via conditions | JSON (`Condition`) |
| **Azure RBAC + conditions** | RBAC + ABAC | JSON |
| **Google IAM Conditions** | ABAC | CEL (Common Expression Language) |
| **AWS Cedar / Verified Permissions** | ABAC/ReBAC-friendly | Cedar DSL |
| **[[opa-rego\|Open Policy Agent]]** | Multi-model | Rego |
| **Casbin** | Multi-model | CONF-based policy |
| **Oso / Polar** | Multi-model | Polar DSL |

## Como grandes empresas usam

- **AWS**: policy JSON com `Condition` block = ABAC. `aws:PrincipalTag/Department=Finance` + `aws:ResourceTag/Department` → role único para N departamentos.
- **Google Cloud IAM**: `iam.conditions` em CEL — `request.time < timestamp("...")`, `resource.name.startsWith("...")`.
- **Microsoft Azure**: RBAC + conditions em scopes (preview) — expression language.
- **Netflix, Airbnb**: OPA interno para policy de microservices.
- **Master Síndico (projeto exemplo)**: ABAC Engine Go custom — ver [[security-principles#ABAC-Engine]] para código.

## Relações

- **Decision tree**: [[authorization-models]]
- **Comparação**: [[rbac]], [[acl]], [[rebac]]
- **Princípio raiz**: [[least-privilege]]
- **Engine**: [[opa-rego]], [[zanzibar-openfga]] (ReBAC alternativa)
- **Implementação concreta** (código Go, matriz, denial reasons): [[security-principles#ABAC-Engine]]
- **Zero Trust**: [[beyond-corp]] ("Autorização granular" §3)
- **Runtime antipatterns**: [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]], [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]], [[../runtime-antipatterns/op-021-admin-sem-audit|OP-021]]

## Fontes

- [NIST SP 800-162 — Guide to ABAC](https://csrc.nist.gov/publications/detail/sp/800-162/final) (consultada 2026-04-24)
- [XACML 3.0 — OASIS](https://docs.oasis-open.org/xacml/3.0/xacml-3.0-core-spec-os-en.html) (consultada 2026-04-24)
- [AWS ABAC whitepaper](https://aws.amazon.com/identity/attribute-based-access-control/) (consultada 2026-04-24)
- [Google Cloud IAM Conditions (CEL)](https://cloud.google.com/iam/docs/conditions-overview) (consultada 2026-04-24)
- [Cedar Language — AWS](https://docs.cedarpolicy.com/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[authorization-models]]
- [[rbac]]
- [[rebac]]
- [[opa-rego]]
- [[least-privilege]]
- [[security-principles]]
- [[beyond-corp]]
