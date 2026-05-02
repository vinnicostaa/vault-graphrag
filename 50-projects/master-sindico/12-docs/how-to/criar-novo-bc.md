---
title: How-to — Criar novo bounded context
type: guide
tags: [how-to, bc, bounded-context, ddd, master-sindico, v2]
source: 01-domain/bounded-contexts + 02-architecture/clean-arch-mapping + padrões do legado
created: 2026-04-23
updated: 2026-04-23
---

# How-to — Criar novo bounded context

Quando criar novo BC (raro, mas acontece em M4+): use este guia. Exemplo usado: adicionar `loyalty` (BC hipotético pra programa de fidelidade).

---

## 1. Justificar criação

Novo BC é **custoso** (tabelas, events, tests). Justifique:

- [ ] Responsabilidade primária diferente dos 6+cross atuais
- [ ] Linguagem ubíqua **distinta** (termos que não existem em outros BCs)
- [ ] Aggregates com invariantes próprios
- [ ] Mais de 1 provider externo exclusivo ou ciclo de vida diferente
- [ ] Time diferente vai operar (raro; mas impacta autonomia)

Se não bate ≥ 3 critérios: **integre no BC existente** (sub-domínio em `commercial/` ou `content/`).

---

## 2. Research-loop + dual-check

Passos obrigatórios:
1. [[../../06-execution/playbooks/research-loop]] — verificar se big tech faz algo similar, padrão aplicável
2. [[../../06-execution/playbooks/dual-check]] — revisor em contexto limpo confirma
3. Registrar D-### em [[../../STATE]]
4. ADR em [[../../02-architecture/adr/_moc]] (ADR-####)

---

## 3. Escrever spec

Criar arquivos na ordem:

### 3.1 Portfolio impacto

- Atualizar [[../../00-product/portfolio-de-produtos]] — adicionar sub-produto se aplicável, ou mapear novo BC → sub-produto existente

### 3.2 Domain layer

- [[../../01-domain/bounded-contexts]] — nova seção com:
  - Responsabilidade primária
  - Linguagem ubíqua
  - Aggregates principais
  - VOs
  - Invariantes (I-###)
  - Dependências (eventos consumidos)
  - Eventos publicados
  - Provider externo (se houver)
  - Status + marco de entrega
- [[../../01-domain/ubiquitous-language]] — termos novos
- [[../../01-domain/invariants]] — I-### novos
- [[../../01-domain/domain-events]] — eventos pub/sub novos
- [[../../01-domain/business-rules]] — BRs específicos
- [[../../01-domain/context-map]] — relacionamentos (Partnership / Customer-Supplier / Conformist / SharedKernel)
- `01-domain/aggregates/<Aggregate>.md` por aggregate novo

### 3.3 Architecture layer

- [[../../02-architecture/c4-components]] — adicionar componentes do BC novo
- [[../../02-architecture/clean-arch-mapping]] — mapear diretórios do novo módulo
- [[../../02-architecture/patterns]] — patterns específicos (se inventar algo que não tá lá)
- [[../../02-architecture/event-driven]] — se BC é event-heavy

### 3.4 Requirements

- `[[../../04-requirements/functional/<novo-bc>]]` — requirements com IDs estáveis (ex: LOY-001..LOY-N)

### 3.5 Security

- Impacto em [[../../08-security/threat-model]] — STRIDE por feature
- Impacto em [[../../08-security/BEYOND_CORP]] — ABAC matrix extension
- Impacto em [[../../08-security/lgpd]] — data-flow, retenção, consent

### 3.6 Testing

- Coverage target: domain ≥ 95%
- PBT em invariants críticos
- Integration tests com testcontainers-go

### 3.7 Tasks

- `[[../../05-tasks/by-module/<novo-bc>]]` — novo arquivo
- [[../../05-tasks/backlog]] — tasks T-### no sub-produto correspondente
- Mapear marcos (M1/M2/M3/M4+)

---

## 4. Criar código (fora desta pasta; no `backend/`)

Ordem canônica (herdado de [[../../06-execution/dev#23-execute--ordem-canônica]]):

```
1. Migration SQL
   backend/internal/shared/providers/database/migrations/
   NN_<module>_create_<table>.sql

2. sqlc query
   backend/internal/modules/<novo-bc>/infrastructure/database/queries.sql
   make sqlc

3. Domain entity / VO
   backend/internal/modules/<novo-bc>/domain/entities/<entity>.go
   backend/internal/modules/<novo-bc>/domain/value_objects/<vo>.go

4. Repo interface (domain)
   backend/internal/modules/<novo-bc>/domain/repositories/<repo>.go

5. Domain event
   backend/internal/modules/<novo-bc>/domain/events/<event>.go

6. Use case CQRS
   backend/internal/modules/<novo-bc>/application/use_cases/<action>_command.go
   backend/internal/modules/<novo-bc>/application/use_cases/<action>_query.go

7. Mapper
   backend/internal/modules/<novo-bc>/infrastructure/database/mappers/<entity>_mapper.go

8. Repo impl
   backend/internal/modules/<novo-bc>/infrastructure/database/<repo>_repo.go

9. Provider externo (se houver)
   backend/internal/shared/providers/<kind>/interface.go (interface)
   backend/internal/modules/<novo-bc>/infrastructure/providers/<provider>/ (impl)

10. HTTP handler + rota
    backend/internal/modules/<novo-bc>/infrastructure/http/routes/<resource>.go
    OpenAPI spec atualizada (backend/openapi/spec.yml)

11. Wire-up
    backend/internal/modules/<novo-bc>/module.go (exporta)
    backend/cmd/api/main.go (wire DI)

12. Testes
    backend/internal/modules/<novo-bc>/domain/entities/<entity>_test.go (unit)
    backend/internal/modules/<novo-bc>/application/use_cases/<action>_command_integration_test.go
```

---

## 5. Frontend + Mobile (se aplicável)

- Web: `web/src/apps/<app-afetado>/` — telas + componentes
- Mobile: `app/lib/features/<feature>/` — seguir Clean Arch per feature

---

## 6. Smoke test

End-to-end manual:
- Criar entity via POST handler
- Verificar evento publicado
- Verificar consumer em outro BC (se aplicável)
- Smoke test via curl ou Postman

---

## 7. PR + review

- PR title: `feat(<novo-bc>): add <novo-bc> bounded context`
- Body:
  - Summary + AC
  - ADR link
  - Screenshots se afeta UI
  - Gates verdes
- Review ≥ 1 humano + CI
- Squash merge

---

## 8. Docs finais

- [ ] Atualizar [[../README]] se afeta stack
- [ ] Atualizar [[../adr-index]] com ADR nova
- [ ] Atualizar [[../changelog]] no próximo release
- [ ] Atualizar MOCs das pastas tocadas

---

## Exemplo real (do legado)

Adicionar BC `assembly` (Sprint 7):
1. Research: livekit vs alternativas → ADR-0008
2. Dual-check: OK (propositor dev + revisor agente contexto limpo)
3. D-009 legado em STATE
4. Spec em `01-domain/bounded-contexts.md#6-assembly`
5. 15 requirements ASM-001..ASM-015 em `04-requirements/functional/assembly.md`
6. Invariantes I-039..I-050
7. 7 aggregates (Assembly, AgendaItem, Vote, Minutes, LiveSession, Proxy, AttendanceRecord)
8. Provider Livekit via `ILiveProvider`
9. Código Sprint 7 backend + Sprint 14-16 frontend
10. AUDIT findings A-033, A-034 abertos na construção

---

## Anti-padrões

- ❌ Criar BC pra "agrupar coisas parecidas" — se conceitos são diferentes mas dentro do mesmo contexto de negócio, fica em um BC só com sub-pasta
- ❌ BC importa `domain/` ou `application/` de outro BC — comunicação só via events ou `shared/types/`
- ❌ BC sem invariants claros — se não dá pra listar 5+ invariants, é sub-domínio, não BC
- ❌ Skip research-loop — decisão sem pesquisa vira legacy rápido

---

## Links

- [[_moc]]
- [[criar-adr]]
- [[../../01-domain/bounded-contexts]]
- [[../../02-architecture/clean-arch-mapping]]
- [[../../06-execution/dev]]
- [[../../06-execution/playbooks/research-loop]]
