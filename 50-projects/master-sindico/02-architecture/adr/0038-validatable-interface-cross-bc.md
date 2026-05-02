---
title: ADR-0038 — IValidatable interface cross-BC (Opção B DDD puro)
type: adr
tags: [master-sindico, adr, architecture, ddd, validation, cross-bc]
project: master-sindico
status: accepted
date: 2026-04-24
deciders: João (stakeholder), agente principal (Claude Opus)
related: [D-105, Gap Analysis 2026-04-24 §4]
---

# ADR-0038 — IValidatable interface cross-BC (Opção B DDD puro)

**Status**: ✅ Accepted (2026-04-24)
**Fonte**: [[../../STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)|D-105]]
**Supersede**: nenhum (complementa [[0015-uow-intra-saga-inter]])

## Contexto

O hub "Validações Pendentes" (telas [[../../03-subprojects/web/ui-catalog/sindico/S21]], [[../../03-subprojects/web/ui-catalog/sindico/S22]], [[../../03-subprojects/web/ui-catalog/sindico/S26]]) agrega **5 tipos** de itens que o síndico valida:

1. **ServiceRecord** (commercial)
2. **TechnicalActivity** (commercial)
3. **PostDealAnnouncement** (commercial — a avaliar se Announcement cobre)
4. **Proposal** (commercial)
5. **Adjustment** (commercial)

Todos precisam da **mesma ação** (validar, aprovar, rejeitar, solicitar ajuste) feita pelo **mesmo ator** (síndico), com **mesmo audit trail** e **mesma notificação**, mas cada um tem **invariantes distintos** no domínio.

Gap Analysis 2026-04-24 §4 identificou ausência de aggregate `ValidationItem.md` como gap crítico M1. Dilema: criar aggregate único polimórfico (Opção A) ou aggregates distintos + interface (Opção B)?

## Decisão

**Opção B — DDD puro.**

- `ValidationItem` é **conceito/superset semântico**, **não aggregate**.
- Cada item validável é aggregate próprio no seu BC.
- Cada aggregate implementa interface cross-BC `IValidatable` (pacote `internal/shared/types/validatable.go`).
- Hub consome via view materializada `pending_validations` (UNION ALL) OU endpoint agregador `GET /api/v1/institutional/validation-items`.
- Aggregates retornam DTOs uniformes no hub (sem vazar tipos internos).

## Motivação

**Justificativa do usuário (João, 2026-04-24)**: "facilita manutenção e estabilidade".

**Justificativa técnica**:
- Aggregate único polimórfico com campo `type` e N campos opcionais é anti-pattern (aggregate inchado, invariantes ambíguos, teste complexo).
- Aggregates distintos mantêm **Single Responsibility Principle** e encapsulamento.
- Cada aggregate evolui independente sem risco de quebrar outros.
- Teste unitário isolado; mocks focados.
- Índices de DB específicos por tabela.

## Consequências

### Positivas

- **DDD alinhado**: aggregates com invariantes próprios, SRP preservado
- **Evolução independente**: adicionar 6º tipo não toca os 5 existentes
- **Teste focado**: cada aggregate testável isoladamente
- **Performance**: queries scoped por tipo, indexes específicos
- **Manutenção**: mudança em `ServiceRecord` não tem risco de regressão em `Proposal`

### Negativas

- **5 implementações** da interface (vs 1 no caso polimórfico)
- **Hub precisa view materializada OU 5 queries** (vs 1 query simples)
- **Refresh da view** requer trigger em cada tabela OU cron (1min)
- **DTOs uniformes** exigem mapeamento consistente em cada implementação

### Trade-offs com Opção A (rejeitada)

| Aspecto | Opção B (accepted) | Opção A (rejected) |
|---|---|---|
| Aggregate único | ❌ | ✅ (com `type` discriminator) |
| SRP / invariantes encapsulados | ✅ | ❌ |
| Código implementação | 5 impls | 1 impl |
| UI hub | View materializada ou agregador | 1 query simples |
| Evolução (+1 tipo novo) | Novo aggregate isolado | Nova coluna na tabela única |
| Performance queries | Por tipo, focadas | Tabela única maior |
| Testabilidade | Isolada | Precisa cobrir N variantes |

## Alternativas consideradas

### Opção A — Aggregate único polimórfico
**Rejeitada** pelo usuário + DDD anti-pattern. Tabela `validation_items` com `type ∈ {...}` e campos opcionais nulos por tipo.

### Opção C — Event Sourcing com projeções por tipo
**Rejeitada** — over-engineering para M1/M2. Implicaria refatorar estratégia de persistência de 5 aggregates existentes para event-sourcing; risco alto, benefício baixo em prazo curto.

### Opção D — Table inheritance Postgres
**Rejeitada** — complexidade operacional (migrations, índices, reindex) não compensa. Funciona bem em Postgres mas é feature específica; perde portabilidade e legibilidade do schema.

## Spec da interface

Ver [[../../01-domain/patterns/validatable-interface]] para contrato completo (método `Validate(ctx, actor, action, reason)`, preconditions, postconditions, eventos publicados, view SQL de `pending_validations`, endpoint hub).

## Implementadores (aggregates)

| Aggregate | Status | BC | Arquivo |
|---|---|---|---|
| `Proposal` | ✅ existente | commercial | [[../../01-domain/aggregates/Proposal]] |
| `ServiceRecord` | 🟡 stub criado 2026-04-24 | commercial | [[../../01-domain/aggregates/ServiceRecord]] |
| `TechnicalActivity` | 🟡 stub criado 2026-04-24 | commercial | [[../../01-domain/aggregates/TechnicalActivity]] |
| `Adjustment` | 🟡 stub criado 2026-04-24 | commercial | [[../../01-domain/aggregates/Adjustment]] |
| `PostDealAnnouncement` | 🟡 a avaliar: [[../../01-domain/aggregates/Announcement]] cobre ou separar | commercial | ver Announcement |

Expandir stubs em Sprint 10.

## Quando reconsiderar

**Gatilho de migração para Opção A**: se aparecerem **2+ novos tipos validáveis em 1 sprint** (ex: validação de voto de mesa, validação de adendo assembleia, validação de promoção Vizinhança) E o padrão de validação estabilizar (mesmos 4 actions: approve/reject/adjustment/reject), reconsiderar migração.

Sinal: se interface tem **mais de 10 implementadores** com mesma lógica boilerplate, aggregate polimórfico com strategy pattern pode compensar. Abrir ADR-## nova nessa hora.

## Rastreabilidade

- **Decision**: [[../../STATE#D-105]]
- **Pattern spec**: [[../../01-domain/patterns/validatable-interface]]
- **Gap origin**: [[../../11-audit/audit-log/2026-04-24-gap-analysis-dominio]] §4
- **Hub UI**: [[../../03-subprojects/web/ui-catalog/sindico/S21]], [[../../03-subprojects/web/ui-catalog/sindico/S22]], [[../../03-subprojects/web/ui-catalog/sindico/S26]]
- **Backend endpoint**: a formalizar em [[../../03-subprojects/backend/requirements/institutional]]
- **Aggregates related**: Proposal, ServiceRecord, TechnicalActivity, Adjustment, Announcement

## Links

- [[_moc]]
- [[0015-uow-intra-saga-inter]] — UoW intra / Saga inter
- [[0033-ata-timeline-db-immutability]] — imutabilidade timeline (Adjustment é único mecanismo de correção)
- [[../../01-domain/patterns/validatable-interface]]
- [[../../STATE#D-105]]
