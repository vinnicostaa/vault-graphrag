---
title: Aggregate — Proxy (Procuração)
type: spec
tags: [domain, ddd, aggregate, assembly, master-sindico, v2]
bc: assembly
source: 90-ingested/.../specs/requirements/assembly.md Req 52
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `Proxy` (Procuração)

**BC**: assembly · **Raiz**: ✅

Outorga de voto de morador (titular) para outro morador. Pode ser específica (item da pauta) ou geral (toda assembleia). Assinatura digital Sprint 3+.

## Entidade raiz

```go
type Proxy struct {
    id                  ProxyID
    assemblyID          AssemblyID
    authorizedByUserID  UserID                  // titular
    proxyUserID         UserID                  // quem recebe procuração
    scope               ProxyScope              // general | specific (agenda_item_ids)
    agendaItemIDs       []AgendaItemID          // se scope=specific
    state               ProxyState              // active | revoked | expired | used
    validUntil          time.Time               // assembly.ended_at + 24h
    revokedAt           *time.Time
    createdAt           time.Time
}
```

## Value Objects

- `ProxyID`, `ProxyScope`, `ProxyState` (enums)

## Invariantes

- **INV-077**: proxy não vota diferente do titular; se titular votou, proxy é impedido
- **INV-078**: válida até 24h pós-encerramento assembleia
- **INV-079**: revogável até 1h pré-assembleia
- Procurador deve ser morador do mesmo condomínio

## Factories

```go
func NewProxy(assemblyID AssemblyID, authorizedBy, proxyUID UserID, scope ProxyScope, agendaItemIDs []AgendaItemID, assemblyEndDate time.Time) (*Proxy, error) {
    // state=active; validUntil = assemblyEndDate + 24h
}

func ReconstructProxy(...) *Proxy
```

## Métodos de domínio

- `Revoke(now time.Time, assemblyScheduledDate time.Time) error` — valida `now < assemblyScheduledDate - 1h`
- `MarkUsed()` — após proxy votar
- `MarkExpired()` — se `NOW() > validUntil`

## Repository interface

```go
type IProxyRepository interface {
    Save(ctx context.Context, p *Proxy) error
    FindByID(ctx context.Context, id ProxyID) (*Proxy, error)
    FindByAuthorizedUserAndAssembly(ctx context.Context, uid UserID, aid AssemblyID) (*Proxy, error)
    FindByProxyUserAndAssembly(ctx context.Context, uid UserID, aid AssemblyID) ([]*Proxy, error)
    ListActiveByAssembly(ctx context.Context, aid AssemblyID) ([]*Proxy, error)
}
```

## Eventos emitidos

- `ProxyGranted` / `ProxyRevoked` (E-068)

## Links

- [[../bounded-contexts#6-assembly]]
- [[../invariants#assembly-inv-071-a-inv-085]]
- [[../business-rules]] BR-047
- [[Assembly]] · [[Vote]]
