---
title: OP-011 — Lógica de negócio em serviço de infra
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - coupling
  - clean-architecture
id: OP-011
severity: Medium
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Business logic in infra service
  - Hardcoded rules in infrastructure
---

# OP-011 — Lógica de negócio em serviço de infra

**Severidade**: Medium
**Categoria**: Acoplamento

## Sintoma

`PermissionService` (camada infra) tem regras hardcoded de ownership: `if userRole === 'admin' { grantAll() }`. Domínio (a lei de negócio) está escrito dentro da infra (execução). Quebra a Dependency Rule de Clean Architecture.

## Por que é problema

- **Testabilidade**: pra testar regra de ownership, precisa subir toda a infra.
- **Reuso**: outra camada (CLI, worker) que precisa da mesma regra duplica ou importa infra.
- **Change preventer**: mudança de política exige deploy de infra inteira.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — infra decide regra de negócio
func (s *PermissionService) CanEdit(ctx context.Context, userID, postID string) bool {
    user := s.repo.FindUser(userID)
    if user.Role == "admin" { return true }  // ← regra de domínio na infra
    if user.Role == "moderator" && time.Since(user.CreatedAt) > 30*24*time.Hour {
        return true
    }
    return false
}
```

## Fix preferido — regras em domain/application; infra só executa

```go
// domain/services/ownership_rules.go — puro, testável, sem I/O
type OwnershipRules struct{}

func (r OwnershipRules) CanEdit(user User, post Post, now time.Time) bool {
    if user.IsAdmin() { return true }
    if user.IsModerator() && user.Tenure(now) > 30*24*time.Hour { return true }
    return post.Author().ID == user.ID
}

// infrastructure/permission_service.go — orquestra, não decide
func (s *PermissionService) CanEdit(ctx context.Context, userID, postID string) bool {
    user, _ := s.userRepo.Find(ctx, userID)
    post, _ := s.postRepo.Find(ctx, postID)
    return s.rules.CanEdit(user, post, time.Now())
}
```

## Alternativa

- **Regras em config/seed**: tabela `plan_permissions` com matriz `(role, feature, action)` lida pela app (ver AGENTS_SPEC §6.3 Functional Matrix).
- **Policy engine externo** (OPA, Casbin) quando a matriz fica complexa — infra só consulta o engine; engine tem as regras.

## Quando tolerar

Regra trivial, estável, restrita a um único flow (ex.: `if isSystemUser { skipCaptcha }`). Acima de 3-4 branches, extrair.

## Relações

- **Patterns**: [[../patterns/strategy]] (regras plugáveis), [[../architecture/clean-architecture-layers]] (Dependency Rule), ABAC Functional Matrix (AGENTS_SPEC §6.3)
- **OPs relacionados**: [[op-009-vazamento-dominio-modulos]] (vazamento no sentido inverso), [[op-013-switch-replicado]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-011, §6.2 (Authorization como módulo central)
- Uncle Bob — [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) (consultada 2026-04-24)
- [[../architecture/a-little-architecture]] — ISP + DIP aplicados

## Links

- [[_moc]]
- [[../architecture/clean-architecture-layers]]
- [[../patterns/strategy]]
- [[op-009-vazamento-dominio-modulos]]
