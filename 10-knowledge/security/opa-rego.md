---
title: OPA & Rego — policy-as-code engine (CNCF)
type: concept
tags:
  - security
  - opa
  - rego
  - policy-as-code
  - authorization
  - cncf
doc-oficial: https://www.openpolicyagent.org/docs/latest/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Open Policy Agent
  - OPA
  - Rego
---

# OPA & Rego — Open Policy Agent

**O que é**: policy engine general-purpose (CNCF graduated 2021) que decouple **policies** da aplicação. Escreve regras em **Rego** (linguagem declarativa) que avaliam JSON input e retornam decisão. Implementa **PBAC** (policy-based) e pode expressar RBAC/ABAC/ReBAC como casos.

> OPA não é IdP, não faz authn. É um **PDP (Policy Decision Point)** — recebe context + retorna allow/deny/obligation. App continua autenticando; OPA decide o quê.

## Quando OPA brilha

1. **Kubernetes admission control** (Gatekeeper, Kyverno alternative).
2. **API authorization** em microservices com policies compartilhadas.
3. **Terraform / IaC policy** (OPA como gate pré-apply).
4. **Data filtering** (ABAC row-level).
5. **Múltiplos modelos de autorização numa org** (algumas apps RBAC, outras ABAC, outras ReBAC) — 1 engine, N policies.

## Quando **não**

- App simples com 1 policy RBAC — bib local (Casbin, cedar-policy) resolve sem rodar binary.
- App com authz que muda toda semana — Rego tem curva; overhead pode não compensar.
- Hyper-performance sub-ms estrito — OPA é rápido, mas sidecar adiciona 1-5ms por check.

## Padrão canônico — decisão

**Input** (contextual JSON):
```json
{
  "user": {
    "id": "alice",
    "role": "manager",
    "tenant_id": "acme"
  },
  "action": "read",
  "resource": {
    "type": "invoice",
    "owner_id": "bob",
    "tenant_id": "acme"
  },
  "context": {
    "time": "2026-04-24T14:30:00Z",
    "mfa_verified": true
  }
}
```

**Policy** em Rego (`policies/authz.rego`):
```rego
package app.authz

import future.keywords.if
import future.keywords.in

default allow := false

# Manager pode ler qualquer invoice do seu tenant
allow if {
  input.user.role == "manager"
  input.action == "read"
  input.resource.type == "invoice"
  input.resource.tenant_id == input.user.tenant_id
}

# Owner pode gerenciar seu próprio invoice
allow if {
  input.action in ["read", "update", "delete"]
  input.resource.owner_id == input.user.id
}

# Operações sensíveis exigem MFA recente
deny contains msg if {
  input.action in ["delete", "transfer"]
  not input.context.mfa_verified
  msg := "mfa_required"
}

# Decisão final
final_decision := {
  "allowed": allow,
  "denials": deny,
}
```

**Query** (via SDK ou HTTP):
```ts
const resp = await fetch('http://opa:8181/v1/data/app/authz/allow', {
  method: 'POST',
  body: JSON.stringify({ input }),
})
const { result } = await resp.json()   // true | false
```

## Deployment modes

### 1. **Sidecar** (mais comum em k8s)

Cada pod tem container OPA escutando em localhost:8181. App consulta via HTTP local. Policies atualizadas via Bundle API ou reload.

### 2. **Library mode** (embedded)

OPA linkado como lib Go em processo da app. Sem network overhead; build mais pesado.

### 3. **Centralized server**

1 cluster OPA HA; apps consultam via rede. Menor manutenção; + latência.

### 4. **WASM compiled**

Rego compilado para WASM; executado em Workers/edge. Cloudflare Workers, client-side evaluation.

## Bundle API — policy distribution

Policies + data viram `bundle.tar.gz`. OPA busca periodicamente de:
- HTTP endpoint (S3, GCS, server interno).
- OCI registry (policies como imagens OCI).
- Git repo (via operator).

Versionamento, rollback, signed bundles (cosign) para integridade.

## Rego — lições importantes

- **Linguagem declarativa** (Datalog-based). Regras "allow" são disjunção — **qualquer** regra matching → allow.
- **`default allow := false`** obrigatório — sem isso, `allow` é `undefined` se nenhuma regra matcha.
- **Sem loops imperativos** — usa comprehensions.
- **Tested**: `rego test` com `_test.rego` files.
- **Formatter**: `opa fmt` (análogo a `gofmt`).
- **VS Code extension** oficial com syntax + autocompletion.

## Anti-patterns

1. **Sem `default allow := false`** — policy com bug permite tudo.
2. **Rego com lógica de negócio profunda** — Rego é para decisão de authz, não para cálculo de preço. Mantém pequeno.
3. **Passar todos os dados do DB como input** — explode. Use `http.send` no Rego pra busca lazy ou `data` no bundle.
4. **Policies sem teste** — Rego aceita spaghetti; bugs silenciosos. `opa test policies/`.
5. **Sidecar sem health check / fallback** — se OPA down, app degrada. Circuit breaker + deny-by-default ou graceful degrade.
6. **Bundle não assinado** — supply chain attack via policy maliciosa. Sign com cosign.
7. **Policy com performance ruim** (`some x` em loop enorme) — profile com `opa eval --profile`.
8. **Misturar authn e authz** — OPA **não** valida JWT; app valida e passa claims como input.

## Como grandes empresas usam

- **Netflix**: OPA para microservices authz; "Secure Continuous Deployment" com policy gates.
- **Pinterest, Intuit, Capital One**: OPA em microservices internos.
- **Chef Software**: InSpec + Chef Automate usam Rego.
- **Styra** (criadora de OPA, agora gerenciada commercial via Styra DAS): enterprise OPA.
- **Microsoft**: AKS com Azure Policy + Gatekeeper (OPA-based).
- **AWS**: Lambda@Edge + OPA via WASM em alguns cases; Cedar competindo como AWS-native.

## Alternativas

| Engine | Linguagem policy | Foco |
|---|---|---|
| **[[zanzibar-openfga\|OpenFGA / SpiceDB]]** | Zanzibar DSL | ReBAC / grafo |
| **AWS Cedar** | Cedar DSL | Simpler syntax; boa para ABAC/ReBAC |
| **Casbin** | CONF-based | Multi-model (RBAC/ABAC/ACL); lib embedded |
| **Oso / Polar** | Polar DSL | App-first; Python/Node/Go libs |
| **Kyverno** | YAML-based | Kubernetes-specific (competidor Gatekeeper) |
| **PyCasbin, Vakt** | Python | Framework-integrated |

OPA é general-purpose + CNCF graduated + ecosistema maduro. Outros são mais especializados.

## Relações

- **Implementa**: [[authorization-models]] (PBAC; pode expressar RBAC/ABAC/ReBAC)
- **Integra com**: [[iam]] (PDP no arquitetura IAM), [[oidc]] (input vem de JWT claims validado)
- **Alternativas**: [[zanzibar-openfga]] (ReBAC nativo), Cedar, Casbin
- **Evita**: [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]] (policies fora do app), [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]] (decisão sempre via PDP, não cache cego)

## Fontes

- [OPA docs](https://www.openpolicyagent.org/docs/latest/) (consultada 2026-04-24)
- [Rego Playground](https://play.openpolicyagent.org/) (consultada 2026-04-24)
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/) (consultada 2026-04-24)
- [CNCF OPA project](https://www.cncf.io/projects/open-policy-agent/) (consultada 2026-04-24)
- [AWS Cedar](https://www.cedarpolicy.com/) (consultada 2026-04-24)
- [Styra DAS (commercial OPA)](https://www.styra.com/styra-das/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[authorization-models]]
- [[abac]]
- [[rbac]]
- [[rebac]]
- [[zanzibar-openfga]]
- [[least-privilege]]
