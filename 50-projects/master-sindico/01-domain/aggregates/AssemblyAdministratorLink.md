---
title: Entity — AssemblyAdministratorLink
type: spec
tags: [domain, ddd, entity, assembly, master-sindico, v2, new-2026-04-25]
bc: assembly
source: STATE#D-127 (2026-04-25)
created: 2026-04-25
updated: 2026-04-25
status: proposed (M3 implementation)
---

# Entity — `AssemblyAdministratorLink`

**BC**: assembly · **Raiz**: ❌ (entity de `Assembly` aggregate)

Cargo cedido pelo síndico durante a assembleia para uma Empresa (administradora) ou User externo. **Auto-invalida** ao `Assembly.End()` — lifecycle 100% derivado de Assembly.

> Criado em 2026-04-25 via [[../../STATE#D-127]]. Originalmente proposto como aggregate root, **reclassificado como entity** após dual-check formal (coerência DDD com [[Vote]] entity de [[AgendaItem]]). Repositório dedicado por escala/query, mas invariantes delegadas a [[Assembly]].

## Entity (sub de Assembly aggregate)

```go
type AssemblyAdministratorLink struct {
    id                  LinkID
    assemblyID          AssemblyID         // FK ao aggregate raiz
    grantedToUserID     *UserID            // delegação a user externo (opcional)
    grantedToCompanyID  *CompanyID         // delegação a empresa (opcional — XOR com user)
    grantedByUserID     UserID             // o síndico que cedeu
    status              LinkStatus
    grantedAt           time.Time
    acceptedAt          *time.Time
    revokedAt           *time.Time
    revokeReason        *string
    documentURL         *string            // link do documento de delegação
    tokenHash           string             // HMAC keyed (ADR-0028)
    permissionGroupID   string             // canônico: pg.assembly.administrator_delegation
}

type LinkStatus string
const (
    PendingInvite LinkStatus = "pending_invite"
    Accepted      LinkStatus = "accepted"
    Active        LinkStatus = "active"
    Revoked       LinkStatus = "revoked"
    Expired       LinkStatus = "expired"
)
```

## State machine

```
pending_invite → accepted → active ──┬──→ revoked (síndico ou administradora revoga manualmente)
                                     └──→ expired (Assembly.End() ou TTL atingido)
```

(Documentar em `01-domain/state-machines.md`.)

## Permissões (PermissionGroup canônico — D-116 IAM)

`pg.assembly.administrator_delegation` (ADR-0039 D-116):

- `assembly.fracao_ideal.import` — importar fração ideal (planilha)
- `assembly.inadimplencia.register` — registrar inadimplência (até 1h antes)
- `assembly.proxy.validate` — validar procurações (digitais, Lei 14.063)
- `assembly.assembly.homologate` — homologar votação

## Invariantes

- **INV-ASM-### (nova)**: `grantedToUserID XOR grantedToCompanyID` — só uma delegação por link
- **INV-ASM-### (nova)**: `Assembly.End()` cascateia `status=expired` em todos os links da assembleia (domain event, sem TTL/job)
- **INV-ASM-### (nova)**: tokenHash gerado via HMAC keyed (ADR-0028) com payload `{assembly_id, user_id|company_id, expires_at}`; TTL ≤ duração da assembleia
- **INV-ASM-### (nova)**: 1 link ativo por (assembly_id, granted_to) — UNIQUE constraint

## Eventos emitidos

- `AssemblyAdministratorLinkGranted` — síndico convidou
- `AssemblyAdministratorLinkAccepted` — administradora aceitou
- `AssemblyAdministratorLinkUsed` — auditoria de cada ação executada (audit append-only)
- `AssemblyAdministratorLinkRevoked` — manual ou automático (`Assembly.End()`)

## Repository

```go
type IAssemblyAdministratorLinkRepository interface {
    Save(ctx context.Context, link *AssemblyAdministratorLink) error
    FindByID(ctx context.Context, id LinkID) (*AssemblyAdministratorLink, error)
    ListByAssembly(ctx context.Context, aid AssemblyID) ([]*AssemblyAdministratorLink, error)
    ListActiveByGrantee(ctx context.Context, granteeID string) ([]*AssemblyAdministratorLink, error)
    InvalidateAllForAssembly(ctx context.Context, aid AssemblyID) error  // chamado em Assembly.End()
}
```

> Repositório dedicado é prática consagrada para entity quando há razão técnica (escala, query patterns). **Invariantes continuam delegadas a `Assembly` aggregate** — métodos críticos sempre invocados via `Assembly.GrantAdministrator()`, `Assembly.RevokeAdministrator()`, `Assembly.End()`.

## Edge cases

- **Convite não aceito antes de `Assembly.Start()`**: permanece `pending_invite`; vence em `Assembly.End()` sem efeito
- **Síndico transfere mandato durante assembleia ativa** (raro, [[../../STATE#D-129]]): link **NÃO herda** automaticamente; novo síndico precisa convidar de novo
- **Assembleia cancelada antes de iniciar**: link expira sem efeito
- **User externo (não morador, não membro do condomínio)**: aceito; persona Empresa terceirizada típica de administradoras condominiais profissionais

## Links

- [[../bounded-contexts#6-assembly]]
- [[Assembly]] (aggregate raiz)
- [[AgendaItem]] · [[Vote]] (outras entities/sub-aggregates de Assembly)
- [[../../02-architecture/adr/0028-lgpd-keyed-hmac]] (token HMAC)
- [[../../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]] (PermissionGroup catalog D-116)
- [[../../STATE#D-127]] (decisão original)
- [[../../STATE#D-132]] (Auditor — padrão similar de role efêmera)
