---
title: ADR-0029 — Tenant ID type-driven (VOs distintos por tipo de tenant)
type: adr
tags:
  - adr
  - multi-tenant
  - type-safety
  - ddd
  - domain
  - master-sindico
  - v2
status: accepted (revised 2026-04-25 D-126)
date: 2026-04-23T00:00:00.000Z
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
aliases:
  - tenant-id-typed
  - typed-tenant-ids
---

# ADR-0029 — Tenant ID type-driven (VOs distintos)

## Status

**accepted (revised 2026-04-25 via D-126)** — 2026-04-23, revisado em 2026-04-25

> ⚠️ **Atualização 2026-04-25 — D-126**: tenant kinds reduzidos de **4 → 2**. `KindUser` e `KindNeighborhood` **REMOVIDOS** como TenantKinds. `UserID` e `PartnerID` continuam existindo como `EntityID` (não-tenant). User é entidade global (CPF/CNPJ único — STEERING §2 / I-002); Neighborhood/Partner é filtro geográfico, não isolamento de dados. Dual-check formal aprovou em 2026-04-25. Ver [[../../STATE#D-126]].

## Contexto

Findings convergentes [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-005]] (tenant_id type confusion) + [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-011]] (`X-Tenant-Id` header controlled by client) identificaram que a plataforma usa `tenant_id` como **string genérica** em múltiplos contextos:

- [[../topology-multitenant]] §1.1 permite "tenant corporativo" (empresa) **e** tenant condomínio usando o mesmo tipo lógico `tenant_id`.
- [[../../04-requirements/functional/identity#IDN-013]] aceita `X-Tenant-Id` **string** do cliente.
- Cache key ABAC `abac:v{N}:{user}:{tenant}` ([[../../08-security/BEYOND_CORP]] §3.3) concatena string — vulnerável a log injection + type confusion.
- Em [[../../01-domain/invariants#INV-089]] "Tenant isolation" não distingue `condominium_id` de `company_id`.

**Cenários de ataque** (pentester):

1. **Cross-type confusion**: query `WHERE tenant_id = $1` recebe `company_id` onde esperava `condominium_id` — se membership matrix não distingue, atacante com membership em empresa acessa condomínio com ID coincidente.
2. **String concat injection**: header `X-Tenant-Hint: "uuid1,uuid2"` (string única com vírgula) pode passar como `IN` slice em query construída dinamicamente — cross-tenant via IN expansion.
3. **Log injection**: cache key usa tenant raw → log poisoning.

Spec atual **não enforça tipo** — é "convenção" em vez de barreira de compilação.

## Decisão

**Tenant ID passa a ser type-driven** com **Value Objects distintos** em Go por tipo de tenant:

```go
// internal/shared/types/tenant.go

// TenantID é a interface vazia + método selador para evitar conversion cross-type.
type TenantID interface {
    UUID() uuid.UUID
    TenantKind() TenantKind
    String() string
    isTenantID()   // selador — impede que tipos externos implementem
}

type TenantKind string

// D-126 (2026-04-25): apenas 2 kinds canônicos.
// User e Neighborhood NÃO são tenants (User é entidade global; Neighborhood é filtro geográfico).
const (
    KindCondominium TenantKind = "condominium"
    KindCompany     TenantKind = "company"
)

// INV-TEN-001: TenantKind ∈ {condominium, company} enforced em DB CHECK + Go sealed interface.

// ---- CondominiumID ----

type CondominiumID struct{ v uuid.UUID }

func NewCondominiumID(u uuid.UUID) CondominiumID { return CondominiumID{v: u} }
func ParseCondominiumID(s string) (CondominiumID, error) {
    u, err := uuid.Parse(s)
    if err != nil { return CondominiumID{}, err }
    return CondominiumID{v: u}, nil
}
func (c CondominiumID) UUID() uuid.UUID     { return c.v }
func (c CondominiumID) TenantKind() TenantKind { return KindCondominium }
func (c CondominiumID) String() string      { return c.v.String() }
func (c CondominiumID) isTenantID()         {}

// ---- CompanyID ----

type CompanyID struct{ v uuid.UUID }
// ... equivalente

// ---- UserID (EntityID, NÃO TenantID) — D-126 ----
//
// User é entidade global (1 CPF/CNPJ = 1 User). NÃO implementa isTenantID().
// Continua existindo como tipo selado para evitar type confusion com outros UUIDs.

type UserID struct{ v uuid.UUID }
// ... equivalente, MAS sem método isTenantID() — não é tenant

// ---- PartnerID (ex-NeighborhoodID; EntityID, NÃO TenantID) — D-126 ----
//
// Partner (Vizinhança/parceiro local) é entidade escoped por filtro geográfico (CEP/raio).
// Dados do Partner pertencem ao tenant Company que o cadastrou; geographic filter aplica em queries.
// Renomeado de NeighborhoodID para clareza semântica.

type PartnerID struct{ v uuid.UUID }
// ... equivalente, MAS sem método isTenantID() — não é tenant
```

### Regras derivadas

1. **Assinaturas de repo** usam o tipo específico:
   ```go
   type TimelineRepo interface {
       ListByCondominium(ctx context.Context, id CondominiumID, cursor Cursor) (...)
   }
   ```
   O compilador rejeita `CompanyID` sendo passado.

2. **Middleware TenantContext** tipa o contexto:
   ```go
   type TenantCtx struct {
       ID   TenantID     // interface — polimórfico
       Kind TenantKind
   }
   ```

3. **`X-Tenant-Id` HINT only**: o header é **sugestão de contexto**, nunca autoridade. Resolução real:
   - path param (`/condominiums/:id`) → `CondominiumID` derivado
   - body param proibido (cobre A-DC-PEN-012)
   - token claims + membership lookup = autoridade
   - `X-Tenant-Id` ajuda o frontend em UI mas middleware valida membership ativo contra `TenantID` derivado do path.

4. **ABAC cache key** canônico: `abac:v{N}:{user_id.String()}:{tenant_kind}:{tenant_id.String()}` — kind explicita no prefixo → zero ambiguidade.

5. **Logs** emitem `{ "tenant_kind": "condominium", "tenant_id": "<uuid>" }` (estruturado) — nunca concat raw.

6. **SQL layer**: sqlc `@tenant_id` params tipados via custom type mapping:
   ```yaml
   # sqlc.yaml
   overrides:
     - go_type: "github.com/mastersindico/backend/internal/shared/types.CondominiumID"
       column: "timeline_entries.condominium_id"
   ```

## Consequências

### Positivas

- Fecha A-DC-SEC-005 (type confusion).
- Fecha A-DC-PEN-011 (X-Tenant-Id header spoofing parcial — + resolução path-based reforça).
- Compilador enforça isolamento — erro de type system antes de runtime.
- Log injection impossível (cache key / log key estruturado).
- Audit trail granular por `tenant_kind`.
- Alinha com [[../../STEERING]] §14 (no cross-BC imports — types compartilhados em `internal/shared/types`).

### Negativas

- Refactor abrangente em todas as queries sqlc + repos existentes no legado (custo M1 absorvido).
- Boilerplate inicial de 4 tipos × helpers (Parse, Marshal JSON, Scan, Value para SQL).
- Documentação: cada dev precisa entender que `CondominiumID` e `CompanyID` não são conversíveis (por design).

## Regras

1. **CI grep bloqueia**: `context.Value("tenant_id")` string bare, `tenant_id string` em assinaturas fora de `internal/shared/types`.
2. **Toda migration** que cria coluna de tenant_id deve usar `BYTEA NOT NULL` + coluna auxiliar `tenant_kind TEXT NOT NULL CHECK (tenant_kind IN ('condominium','company'))` — D-126 reduziu de 4 → 2 kinds.
3. **Cross-tenant query** precisa path explícito + ABAC `action=cross_tenant_read` (raro — só admin).
4. **JSON Marshal/Unmarshal** de tenant ID prefixa kind: `{"kind":"condominium","id":"<uuid>"}` — nunca string bare na API pública.

## Alternativas consideradas

1. **`type TenantID string`** (single alias) — rejeitado: não previne confusion, é apenas newtype semântico.
2. **`context.Value("condominium_id") / context.Value("company_id")`** com strings separadas — funciona, mas requer disciplina de get/set por site; erro fácil. Type-driven é uma barreira.
3. **UUID puro + tabela `tenants(id, kind)` consultada em toda query** — overhead constante; consulta tabela adicional em cada ABAC check; preferível enforce em compile-time.
4. **Adotar `TenantID` com enum `kind` dentro do struct único** (sem types distintos) — permite type confusion porque o campo `kind` é apenas dado (pode ser mutado por mistake ou attacker com pointer). Types distintos Go (closed sum via `isTenantID()`) impedem instanciação fora do pacote.

## Referências

- Finding: [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-005]] (M1 high).
- Finding: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-011]] (X-Tenant-Id controlled).
- [[../topology-multitenant]] — topologia multi-tenant atualizada.
- [[../../01-domain/invariants#INV-089]] — invariante existente (refinado).
- [[../../01-domain/invariants#INV-108]] — invariante novo (type-driven enforce).
- [[../../01-domain/invariants#INV-TEN-001]] — D-126: `TenantKind ∈ {condominium, company}` (criar).
- [[../../STATE#D-126]] — revisão 2026-04-25 (4→2 kinds).
- [[../../11-audit/audit-log/2026-04-24-multitenant-consolidation-map]] — mapa de consolidação multi-tenant.
- [[../../04-requirements/functional/identity#IDN-013]] — multi-context membership, atualizar para referenciar tipo.
- [[0021-multi-tenant-row-based]] — decisão multi-tenant row-based que este ADR refina.
- [[../../08-security/BEYOND_CORP#3.3]] — ABAC cache key (atualizar para prefix `tenant_kind`).
- Go closed sum types via unexported selador: https://www.reddit.com/r/golang/comments/12ke2mb/sum_types_through_sealed_interfaces/
- OWASP Type Juggling: https://owasp.org/www-community/vulnerabilities/Type_Juggling
- CWE-843 (Type Confusion): https://cwe.mitre.org/data/definitions/843.html
