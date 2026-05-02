---
title: Cloudflare — Usage in projects
type: note
tags:
  - provider
  - cloudflare
  - usage-tracker
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - CF usage
---

# Cloudflare — Usage in projects

Tabela viva de quais projetos do vault (`50-projects/`) consomem Cloudflare e em quais produtos. Atualizada a cada adoção.

## Status atual

**🔴 Não instrumentado** — zero projetos no vault usam Cloudflare ainda (2026-04-24, pós D-vault-020).

O cluster `providers/cloudflare/` foi criado como **knowledge global reutilizável**, antecipando adoção em projetos futuros. A decisão de destilar veio do operador (usuário) explicitamente, não de necessidade de projeto em curso.

## Projetos

*Nenhum ainda.*

Conforme projetos adotem Cloudflare, registrar aqui seguindo formato:

```markdown
### <slug-do-projeto>

- **Link principal**: [[../../../50-projects/<slug>/CLAUDE|<Nome>]]
- **Produtos usados**: `workers` · `r2` · `pages` (ex.)
- **Ambiente**: `<dev|staging|prod>` · conta CF: `<account-id-parcial>`
- **Caso de uso**: <1-2 linhas descrevendo por que CF foi escolhido aqui>
- **Adoção em**: 2026-XX-XX
- **Padrões aplicados**: links pra seções relevantes de [[patterns]]
- **Gotchas descobertos**: links pra seções de [[antipatterns]] ou OPs criados
```

## Critérios de promoção

Conforme [[../../../00-meta/STATE|D-vault-017]] (política lazy) adaptado a provider:

- **1º projeto adota CF** → atualizar esta tabela + deixar provider como `🟡 1 projeto`.
- **2º projeto adota CF** → promover para `🟢 ativo` + reavaliar se algum dos 11 produtos em backlog lazy deve sair do backlog (ex: se ambos projetos usam `queues`, destilar `queues.md`).
- **Patterns emergentes cross-projeto** → promover da nota atômica do produto para `patterns.md` cross-produto quando 2+ projetos descobrem o mesmo.

## Gatilhos para revisitar

- Algum dos 11 produtos em backlog lazy é requerido por projeto real → destilar (eleva o backlog para ativo).
- `doc-consulted` do `_moc.md` > 90 dias → revalidar via [[../../../30-agents/mcp-integration|MCP docs.mcp.cloudflare.com]] + atualizar.
- Cloudflare anuncia produto novo relevante (ex: framework Remix oficial) ou deprecation de feature usada por algum projeto → atualizar patterns/antipatterns.

## Links

- [[_moc]]
- [[patterns]]
- [[antipatterns]]
- [[../../../50-projects/_moc]] — inventário de projetos ativos
- [[../../../00-meta/STATE]] — D-vault-020 (decisão de adoção)
