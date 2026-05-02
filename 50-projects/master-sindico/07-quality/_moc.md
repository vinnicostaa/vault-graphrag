---
title: MOC — 07-quality
type: moc
tags: [moc, master-sindico, v2, quality]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 07-quality

CLEAN_CODE, CLEAN_ARCH, CODE_CALISTHENICS, PATTERNS e catálogo de antipatterns aplicados ao master-sindico. Especializa as fontes canônicas de [[../../vault/10-knowledge/principles/clean-code]], [[../../vault/10-knowledge/architecture/clean-architecture-layers]], [[../../vault/10-knowledge/principles/code-calisthenics]] e [[../../vault/10-knowledge/antipatterns/_moc]] com exemplos Go concretos + detecção em CI.

---

## Arquivos

- [[CLEAN_CODE]] — princípios Clean Code (nomes, funções pequenas, error handling, sem if/else hell, checklist PR) aplicados ao projeto.
- [[CLEAN_ARCH]] — Clean Architecture estrita: regra de dependência, violações proibidas, como refatorar, mapping Go → layers, testes por camada.
- [[CODE_CALISTHENICS]] — 9 regras Go-flavor (indent, sem else, VOs, coleções tipadas, Demeter, sem abreviação, entidades pequenas, sem setter, sem side-effect oculto) com exemplos antes/depois.
- [[PATTERNS]] — 15+ patterns do projeto (DDD tático, Repository, Factory, VO, CQRS, UoW, Saga, Strategy, Specification, ACL, Idempotency, Circuit Breaker, Retry+Backoff+DLQ, Outbox, Observer) com snippets Go e anti-pattern relacionado.
- [[antipatterns]] — Catálogo AP-001..AP-026 específicos do master-sindico (God Object 450L/550L, vazamento ORM, N+1, cache sem versioning, rate limit sem cleanup, PII em log, secrets em `.env`, saga sem compensação, TOCTOU, etc.) com mapeamento AP → fix pattern.

---

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
- [[../STEERING]] — não-negociáveis
- [[../01-domain/invariants]] — INV-### testáveis
- [[../01-domain/business-rules]] — BR-###
- [[../02-architecture/patterns]] — mapa canônico de patterns
- [[../02-architecture/clean-arch-mapping]] — camadas e regras
- [[../09-testing/_moc]] — testing strategy + thresholds
- [[../11-audit/AUDIT]] — findings A-### legados
