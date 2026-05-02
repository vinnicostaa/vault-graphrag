---
title: Sub-produto 12 — Reservas de Espaços Comuns
type: concept
tags:
  - master-sindico
  - sub-produto
  - reservas
  - m3-backlog
project: master-sindico
milestone_target: m3+
status: backlog
created: 2026-04-24T00:00:00.000Z
---

# Sub-produto 12 — Reservas de Espaços Comuns

**Status**: 📋 Backlog M3+ · **Origem**: [[../../STATE#d-115|D-115]] (2026-04-24, confirmação 4 da Opção C)

## Contexto

Durante análise de tipos de evento S16 (D-115), detectou-se que o cliente incluía no Bloco 9 da lista de eventos: churrasqueira (reserva), salão de festas (reserva), piscina (reserva individual de morador). Essas **não são Eventos** no sentido de `Event` aggregate — são **reservas de espaço comum**, um sub-produto distinto com regras próprias (disponibilidade, conflito de horário, taxa de uso, limpeza pós-uso, caução, etc.).

## Escopo (quando for especificado)

### Entidades prováveis
- `CommonSpace` (espaço comum: churrasqueira, salão de festas, quadra esportiva, piscina, academia, co-working)
- `Reservation` (reserva individual com data+hora, morador, espaço, status, taxa?)
- `SpaceAvailabilityWindow` (janelas de disponibilidade por espaço: ex. churrasqueira disponível sáb/dom 10h-22h)
- `ReservationRule` (regras específicas: antecedência mínima/máxima, limite por morador/mês, caução, lotação máxima)

### Jornadas antecipadas
1. **Morador reserva espaço** (VM/M view): lista espaços → calendário disponibilidade → seleciona data+hora → confirma pagamento de taxa (se houver) → recebe ticket
2. **Síndico gerencia regras** (S view): define janelas, taxas, limites, caução, antecedência
3. **Síndico aprova reserva** (se regra exige): algumas reservas (ex: salão festas) precisam aprovação manual do síndico
4. **Cancelamento**: morador cancela até X horas antes; reembolso proporcional; libera espaço no calendário

### Integrações
- **Billing**: cobrança de taxa via Stripe (microtransação)
- **Timeline (institutional)**: publicar "Reserva aprovada: Salão de Festas 15/06 por [morador]" se regras definirem visibilidade
- **Calendário** (Event shadow?): reservas aprovadas podem virar Event shadow type `reserva_espaco` para aparecer no calendário geral do condomínio

## Regras de negócio previstas

- **1 reserva ativa por morador** em certos espaços (salão) — INV a definir
- **Conflito de horário**: `UNIQUE (space_id, datetime_range)` com exclusion constraint Postgres
- **Caução retornável**: não é taxa; fica em escrow até conclusão sem dano → devolve
- **LGPD**: histórico de reservas é dado pessoal — retenção 5 anos + pseudonimização pós-exclusão (aplica [[../../02-architecture/adr/0037-soft-delete-universal-pseudonymize]])

## Escopo M3+ (não MVP)

Este sub-produto **NÃO é MVP**. Foi colocado em backlog explícito para não poluir o enum `event_type` (D-115). Início esperado: M3 pós-assembleia live.

### Tasks a abrir quando entrar no roadmap

- T-RES-01: modelagem DDD (aggregates, invariantes, state-machines)
- T-RES-02: ADR conflito de horário (Postgres exclusion vs app-level)
- T-RES-03: telas M-RES-* + S-RES-*
- T-RES-04: integração Billing (Stripe checkout para taxa)
- T-RES-05: integração Timeline (decisão: shadow Event ou entrada LT própria?)
- T-RES-06: regras de cancelamento/caução

## Referências cruzadas

- **Não deve** entrar em [[../../01-domain/aggregates/Event]] (D-115 confirmação 4)
- **Possível relação com Event** via shadow pattern (mesmo mecanismo que Assembly → Event shadow)
- **Diferente de** [[../../01-domain/aggregates/Poll]] (enquetes)

## Links

- [[_moc]]
- [[../../STATE#d-115|D-115]] (origem desta decomposição)
- [[../../01-domain/aggregates/Event]] (sub-produto relacionado mas distinto)
- [[../../04-requirements/functional/institutional]] (context)
- [[../../10-schedule/milestones]] (roadmap M3+)
