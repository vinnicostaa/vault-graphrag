# 📡 Identity Structs & Contracts (Go)

> **Módulo:** Identity
> **Foco:** Definição de dados para Go e eventos NATS.

---

## 🆔 Core Entities

```go
// User representa a identidade central (espelhada do Zitadel)
type User struct {
    ID        string    `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    Type      UserType  `json:"user_type"` // sindico, morador, empresa, parceiro
    TenantID  string    `json:"tenant_id,omitempty"`
    CreatedAt time.Time `json:"created_at"`
}

// Tenant representa a unidade de isolamento (Condomínio ou Empresa)
type Tenant struct {
    ID         string     `json:"id"`
    Type       TenantType `json:"tenant_id"` // condominium, company
    PlanID     string     `json:"plan_id"`
    Status     string     `json:"status"` // trial, active, suspended
    ValidUntil time.Time  `json:"valid_until"`
}
```

---

## ⚡ Domain Events (NATS JetStream)

**Stream:** `IDENTITY`
**Subject Pattern:** `identity.v1.>`

| Evento | Subject | Payload |
| :--- | :--- | :--- |
| `UserCreated` | `identity.v1.user.created` | `{ "user_id": "uuid", "email": "..." }` |
| `TenantActivated` | `identity.v1.tenant.activated` | `{ "tenant_id": "uuid", "plan": "base" }` |
| `PlanChanged` | `identity.v1.billing.plan_changed` | `{ "tenant_id": "uuid", "new_plan": "pro" }` |

---

## 🛡️ ABAC Claims (Casbin Context)

O middleware extrairá estas informações do JWT do Zitadel para alimentar o Casbin:

```go
type AuthContext struct {
    UserID   string
    TenantID string
    Role     string
    Plan     string
    Action   string
}
```

**Voltar para:** [[Sprint 1 Identity]]
