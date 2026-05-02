---
title: MOC — Runtime Antipatterns (OP-###)
type: moc
tags:
  - moc
  - antipattern
  - runtime
  - operational
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Operational Antipatterns
  - OP Catalog
---

# Runtime Antipatterns — catálogo OP-###

Antipatterns **observáveis em runtime/produção**: concorrência, idempotência, cache, acoplamento, observabilidade, segurança operacional, performance.

> **Distinção vs code smells clássicos**: [[../antipatterns/_moc]] cataloga **code smells Fowler/refactoring-guru** (AP-001..AP-023, design/structure). Este catálogo (`OP-###`) é **operacional** — comportamento em produção sob carga. Reconciliação 2026-04-24 ([[../../00-meta/STATE|D-vault-012]]) resolveu [[../../00-meta/AUDIT|A-vault-041]] migrando operacional de AP→OP para eliminar colisão de IDs.

**Fonte autoritativa tabela resumida** (sintoma + fix + severidade): [[../../40-templates/AGENTS_SPEC]] §4.

**Status destilação atômica**: ✅ completa (D-vault-013 executado 2026-04-24). Cada OP-### tem nota própria com exemplo de código, fix preferido, alternativas, quando tolerar, e cross-links para patterns que resolvem.

---

## Catálogo OP-001..OP-026 (26 entradas)

### Concorrência

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-001-race-condition-contador\|OP-001]] | Race condition em contador / quota | Critical |
| [[op-002-check-then-act\|OP-002]] | Check-then-act não atômico (TOCTOU) | High |

### Idempotência

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-003-webhook-sem-idempotencia\|OP-003]] | Webhook sem idempotência | Critical |
| [[op-004-sem-retry-strategy\|OP-004]] | Operação crítica sem retry strategy | High |

### Cache

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-005-cache-sem-versionamento\|OP-005]] | Cache sem versionamento quando regra muda | Critical |
| [[op-006-cache-falha-quebra-fluxo\|OP-006]] | Cache falha quebra fluxo principal | High |
| [[op-007-cache-key-json-gigante\|OP-007]] | Cache key com JSON serializado gigante | Medium |
| [[op-008-autorizacao-apenas-cache-hit\|OP-008]] | Autorização avaliada apenas no cache hit | Critical |

### Acoplamento / design

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-009-vazamento-dominio-modulos\|OP-009]] | Vazamento de domínio entre módulos | High |
| [[op-010-feature-importa-internals\|OP-010]] | Feature importa internals de outra feature | High |
| [[op-011-logica-negocio-infra\|OP-011]] | Lógica de negócio em serviço de infra | Medium |
| [[op-012-fan-out-desnecessario\|OP-012]] | Fan-out desnecessário (N queries quando 1 basta) | Medium |

### Duplicação

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-013-switch-replicado\|OP-013]] | Switch/case gigante replicado em N lugares | High |
| [[op-014-construtor-gigante\|OP-014]] | Construtor com 10+ parâmetros posicionais | Medium |
| [[op-015-validacao-repetida\|OP-015]] | Validação repetida em N métodos | Medium |

### Null-safety / validação

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-016-force-unwrap\|OP-016]] | Force-unwrap / non-null assertion em mapeamento | Critical |
| [[op-017-mapeamento-parcial\|OP-017]] | Mapeamento parcial falha silenciosamente | High |

### Estado / invariantes

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-018-state-machine-sem-guards\|OP-018]] | Transição de state machine sem guards | High |
| [[op-019-mutacao-sem-validacao\|OP-019]] | Mutação de campo derivado sem validação | Medium |

### Observabilidade

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-020-erro-engolido\|OP-020]] | Erro engolido silenciosamente | High |
| [[op-021-admin-sem-audit\|OP-021]] | Ação de admin sem audit trail | High |
| [[op-022-log-com-pii\|OP-022]] | Log com PII / payload sensível sem sanitização | Critical |

### Segurança operacional

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-023-webhook-sem-hmac\|OP-023]] | Webhook sem validação de assinatura HMAC | Critical |
| [[op-024-rate-limit-ausente\|OP-024]] | Rate limit ausente em endpoint caro ou exposto | High |

### Performance

| ID | Antipattern | Severidade |
|---|---|---|
| [[op-025-coluna-sem-indice\|OP-025]] | Coluna com query frequente sem índice | Medium |
| [[op-026-lista-sem-paginacao\|OP-026]] | Endpoint de lista sem paginação default | Medium |

---

## Severidade agregada

| Severity | Count | IDs |
|---|---:|---|
| **Critical** | 7 | OP-001, OP-003, OP-005, OP-008, OP-016, OP-022, OP-023 |
| **High** | 11 | OP-002, OP-004, OP-006, OP-009, OP-010, OP-013, OP-017, OP-018, OP-020, OP-021, OP-024 |
| **Medium** | 8 | OP-007, OP-011, OP-012, OP-014, OP-015, OP-019, OP-025, OP-026 |

---

## Equivalência histórica (2026-04-24, D-vault-012)

Todos os IDs deste catálogo **eram `AP-###`** antes da reconciliação — mudaram 1:1 (`AP-001..AP-026` antigo → `OP-001..OP-026` novo). Os `AP-001..AP-023` do [[../antipatterns/_moc]] **permanecem clássicos** (Fowler/GoF code smells) e não colidem mais.

Notas anteriormente citando `AP-0XX` operacional foram patched cross-vault (ver D-vault-012 em STATE para lista completa).

## Como contribuir

1. Antipattern runtime novo identificado em produção: verificar se cabe num OP-001..OP-026 existente.
2. Cabe → enriquecer a nota com caso concreto na seção "Relações" ou "Quando tolerar".
3. Não cabe → propor OP-027+ como nota atômica nova em kebab-case aqui (`op-027-<slug>.md`), linkar de volta no MOC + `AGENTS_SPEC §4`.
4. Reviewer cita o ID (`OP-XXX`) em review de PR.

## Vizinhos no grafo

- [[../_moc]]
- [[../antipatterns/_moc]] — code smells clássicos (distinto deste catálogo)
- [[../../40-templates/AGENTS_SPEC]] — tabela resumida §4
- [[../../00-meta/STATE]] — D-vault-012 (reconciliação) + D-vault-013 (extração atômica 2026-04-24)
- [[../../00-meta/AUDIT]] — A-vault-041 fechado por D-vault-012
- [[../security/security-principles]] — referências diretas a OP-008/022/023/024
- [[../database/database-patterns]] — referências a OP-001/012/025
