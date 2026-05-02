---
title: RBAC — Role-Based Access Control
type: concept
tags:
  - security
  - rbac
  - authorization
  - roles
  - nist
doc-oficial: https://csrc.nist.gov/projects/role-based-access-control
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Role-Based Access Control
  - RBAC
---

# RBAC — Role-Based Access Control

**O que é**: modelo onde permissões são atribuídas a **papéis** (roles), não a usuários; usuários recebem papéis. Formalizado por NIST (ANSI/INCITS 359-2004). Dominante em enterprise IAM.

**Quando usar**: ver decision tree em [[authorization-models#Como-escolher]].

## Modelo NIST (níveis)

### Core RBAC (flat)

- **Users** — entidades que se autenticam.
- **Roles** — agregam permissões (ex: `admin`, `editor`, `viewer`).
- **Permissions** — (operação, objeto). Ex: `(read, posts)`, `(delete, users)`.
- **Assignment** — user ↔ role (N:N); role ↔ permission (N:N).

```
User Alice  ─┬─▶ Role Admin      ─┬─▶ Permission (read, posts)
             │                     └─▶ Permission (delete, users)
             └─▶ Role Editor       ─▶ Permission (write, posts)
```

### Hierarchical RBAC

Roles herdam de outros roles:
```
Admin (inherits Editor)
  Editor (inherits Viewer)
    Viewer → (read, posts)
```
`Admin` automaticamente tem (read, posts) via herança. **Cuidado**: ciclos proibidos.

### Constrained RBAC — Separation of Duties (SoD)

- **SSD (Static Separation of Duty)** — user não pode ter roles mutuamente exclusivos (ex: `approver` e `requester` na mesma transação).
- **DSD (Dynamic Separation of Duty)** — user tem ambos, mas só um ativo por sessão.

Operações críticas (financeiras, compliance SOX) exigem SoD.

## Padrão canônico (implementação)

```go
type Permission struct {
    Action   string   // "read", "write", "delete"
    Resource string   // "posts", "users"
}

type Role struct {
    Name        string
    Permissions []Permission
    Inherits    []string   // parent roles
}

type RBAC struct {
    roles    map[string]*Role
    userRoles map[string][]string
}

func (r *RBAC) Can(userID, action, resource string) bool {
    for _, roleName := range r.userRoles[userID] {
        if r.roleHasPermission(roleName, action, resource) {
            return true
        }
    }
    return false
}

func (r *RBAC) roleHasPermission(roleName, action, resource string) bool {
    role := r.roles[roleName]
    if role == nil { return false }
    for _, perm := range role.Permissions {
        if perm.Action == action && perm.Resource == resource { return true }
    }
    // Herança
    for _, parent := range role.Inherits {
        if r.roleHasPermission(parent, action, resource) { return true }
    }
    return false
}
```

## Quando RBAC é suficiente

- **Catálogo estável de papéis** (ex: organização tradicional: admin/manager/employee/guest).
- **Permissões agregam bem por papel** (todos os "editors" têm mesma capacidade).
- **Sem regras contextuais** (independe de horário, device, tenant, atributos do recurso).

## Quando RBAC **não basta**

- **Regras que dependem de atributos do recurso**: "user pode editar **seu próprio** post" — inclui ownership, não cabe em role puro.
- **Regras contextuais**: business hours, device trust, MFA recente.
- **Multi-tenant**: "admin" de um tenant não pode ver outro — precisa **ABAC + RBAC** (ver [[abac]]).
- **Compartilhamento ad-hoc** (Google Docs-like): "compartilhar com user X" vira grafo, não role. Ver [[rebac]].

**Regra prática**: RBAC puro aguenta aplicações simples; SaaS multi-tenant quase sempre combina **RBAC + ABAC**.

## Anti-patterns

### 1. **Role sprawl**

500+ roles como `admin_finance_eu_read_2024`. Sinal de que regras são dimensionais — migrar para ABAC ou RBAC+ABAC.

### 2. **Role explosion por tenant**

`admin_tenant_123` para cada tenant. Pool de roles cresce com N tenants. Use role genérico + `tenant_id` como atributo (ABAC).

### 3. **"Admin" sem granularidade**

Papel "admin" com tudo gera bypass fácil e ausência de audit. Fatiar em `admin:users`, `admin:billing`, `admin:content`.

### 4. **Herança circular**

`A inherits B`, `B inherits A` → loop infinito. Validar no DAG.

### 5. **Permissão hardcoded em handler**

```go
if user.Role == "admin" { allow }   // ❌
if rbac.Can(user.ID, "delete", "post") { allow }   // ✅
```

Ver [[../runtime-antipatterns/op-011-logica-negocio-infra|OP-011]].

### 6. **Sem separation of duties em operações críticas**

Mesmo user cria + aprova + executa transação financeira → risco de fraude/erro.

## Implementações reais

| Sistema | RBAC flavor |
|---|---|
| **AWS IAM** | RBAC + conditions (ABAC-flavored) via policies JSON |
| **Kubernetes** | RBAC nativo: Roles + RoleBindings + ClusterRoles |
| **Linux POSIX** | Grupos (role-like) + ACL discricionária |
| **Jira, GitLab, GitHub (básico)** | Roles por projeto + herança |
| **Keycloak, Zitadel** | RBAC + mappers ABAC |
| **Casbin** | RBAC/ABAC/ACL unificado por policy |

## Como grandes empresas fazem

- **AWS**: IAM Roles + Policies (JSON); conditions (`aws:PrincipalTag/...`) adiciona ABAC.
- **GitHub**: org → teams → repositories; roles por repo (admin/maintain/write/triage/read) + grants individuais.
- **Google Workspace**: admin roles pré-definidos + custom roles; SoD via specific admin roles (user mgmt vs billing).
- **Microsoft Entra ID**: roles built-in + custom; PIM (Privileged Identity Management) habilita roles temporariamente.

## Relações

- **Decision tree**: [[authorization-models]] (onde escolher entre modelos)
- **Complemento natural**: [[abac]] (RBAC + ABAC híbrido)
- **Alternativa**: [[rebac]] para grafos, [[acl]] para casos simples
- **Princípio**: [[least-privilege]]
- **Aplicado**: [[security-principles#ABAC-Engine]] — engine Go que combina RBAC base + ABAC refinamentos (tenant isolation, plan tier, MFA recente) em matriz declarativa
- **Providers**: [[../providers/zitadel/_moc]] (roles + project permissions), [[../providers/cloudflare/access]]

## Fontes

- [ANSI/INCITS 359-2004 — NIST RBAC model](https://csrc.nist.gov/publications/detail/conference-paper/2000/07/26/the-nist-model-for-role-based-access-control-towards-a-unified-s) (consultada 2026-04-24)
- [NIST RBAC FAQ](https://csrc.nist.gov/projects/role-based-access-control/faqs) (consultada 2026-04-24)
- [AWS IAM Role docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) (consultada 2026-04-24)
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) (consultada 2026-04-24)
- [Casbin — multiple models (RBAC+ABAC+ACL)](https://casbin.org/docs/overview) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[authorization-models]]
- [[abac]]
- [[acl]]
- [[least-privilege]]
- [[iam]]
- [[security-principles]]
