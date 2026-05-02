---
title: ADR-0001 — Clean Architecture + DDD tático + CQRS
type: adr
tags: [adr, architecture, clean-arch, ddd, cqrs, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0001 — Clean Architecture + DDD tático + CQRS

## Status

accepted — 2026-04-23

## Contexto

Master Síndico tem 7 sub-produtos mapeados em 6 BCs principais + cross-domain. Precisamos de:

- Separação rígida entre núcleo de negócio e infraestrutura (framework, SDKs externos, persistência trocáveis sem reescrever domínio).
- Invariantes de negócio não-triviais (fração ideal, vote único, quota persona-aware, timeline imutável).
- Testabilidade de domínio independente de DB/framework.
- Coverage thresholds duros (domain 95%, application 90%).
- CQRS para escalar leitura (Redis, OpenSearch, views materializadas) sem quebrar write.

## Decisão

**Clean Architecture estrita** (`domain → application → infrastructure`) com dependências apontando **para dentro**. **DDD tático** (aggregates privados, factories `NewX/ReconstructX`, repo interface em domain, VOs imutáveis, domain events, domain services). **CQRS leve** (command ≠ query, arquivos separados; sem event sourcing completo em M1/M2).

### Convenções

- Estrutura canônica `internal/modules/<bc>/{domain,application,infrastructure}/`.
- Domain zero import de framework (gin, pgx, SDK externo).
- Linter `go-arch-lint` no CI para detectar violações.

## Consequências

### Positivas

- Domain testável sem infra (mocks raros; usa VOs e aggregates reais).
- Trocar Gin por Fiber / Echo / net/http não toca domínio.
- CQRS permite query otimizada em OpenSearch sem contaminar write side.
- DDD tático dá nomes ubíquos ([[../../01-domain/ubiquitous-language]]).

### Negativas

- Overhead cognitivo inicial: devs novos demoram ~1-2 semanas para internalizar regra de dependência.
- Mais arquivos (mapper explícito, VOs separados) — não é "quick hack" território.
- Necessita disciplina em PR review (framework vazando é erro comum).

## Alternativas consideradas

1. **Hexagonal (Ports & Adapters) puro sem CQRS** — funciona mas abre porta para handlers mistos (read+write).
2. **Transaction Script simples** (service + repo + controller) — mais rápido no início; dívida técnica explode em M2+ com invariantes complexas.
3. **Event Sourcing completo** — overkill para M1/M2; timeline e minutes já são "event sourced" por natureza imutável; reavaliar em M3+.

## Referências

- [[../clean-arch-mapping]]
- [[../patterns]] §1-5
- [[../../13-research/instagram/_destilado]] §9 (monolito modular Django)
- [[../../13-research/netflix/_destilado]] §1
- [[../../CLAUDE]] §3
