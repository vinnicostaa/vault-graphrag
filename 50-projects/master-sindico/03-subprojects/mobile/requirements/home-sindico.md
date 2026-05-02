---
title: Home Síndico — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, institutional, feature-req]
project: master-sindico
stack: mobile
priority: m1
status: pending
bounded_context: institutional
created: 2026-04-24
---

# Home Síndico — Mobile

Central de governança do síndico com **até 13 cards**. No mobile M1, **subset enxuto** (home + condomínio default + timeline read + compliance view). S7+ ficam exclusivos web (planilhas, convocação de assembleia, relatórios complexos).

## Escopo M1/M2/M3

- **M1**: cards `S1 entrada / S2 seletor condomínio / S6 central governança (visão read) / S7 timeline read / S8 compliance view`.
- **M2**: swipe actions em "validações pendentes" + card Connect Me recebido + voice input em anotações.
- **M3**: modo apresentação (leitura em assembleia) + procuração + assinatura digital.

## Telas envolvidas

- `S1 S2 S6 S7 S8` (ver `[[../../web/ui-catalog/sindico/S1]]` ... `S8`)
- Telas S9-S31 **não presentes em mobile M1** (ficam web-only).

## Funcionais (FR-MOB-HOMES-N)

- **FR-MOB-HOMES-1** Primeira entrada no módulo governança → verificar flag `first_governance_access` via `GET /api/v1/syndic/governance/first-access`; se true → `S1`, senão → `S2`.
- **FR-MOB-HOMES-2** `S2` lista condomínios onde o usuário tem role `syndic` ativo; tap → persiste default + navega `S6`.
- **FR-MOB-HOMES-3** `S6` exibe 6-8 cards (depende de plano ativo, ABAC): `timeline`, `plano-diretor`, `eventos`, `comunicados`, `compliance`, `connect-me` (M2), `indicadores` (M3).
- **FR-MOB-HOMES-4** Cards são read-only em M1 (dados vêm de `/syndic/dashboard/{condominium_id}`); criação/edição = web.
- **FR-MOB-HOMES-5** Card "Compliance" exibe score atual + alertas críticos (C1..C12); tap → view detail.
- **FR-MOB-HOMES-6** Pull-to-refresh recarrega tudo.

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PERF-1** Cards renderizados em < 300ms com cache.
- **NFR-MOB-A11Y-1** Cards navegáveis via TalkBack/VoiceOver com rótulos semânticos.
- **NFR-MOB-OFFLINE-1** Read-only → cache 10min TTL OK; escrita = online obrigatório (M1 não tem escrita).

## Dados locais

- **Hive `syndic_dashboard_{condominium_id}`** (TTL 10min).
- **hydrated_bloc `SindicoHomeBloc`** persiste último condomínio + dashboard.
- **Invalidação**: evento `CondominiumChanged` limpa cache.

## Integração backend

- `GET /api/v1/syndic/governance/first-access`
- `POST /api/v1/syndic/governance/acceptance` (registro aceite termo).
- `GET /api/v1/institutional/memberships/me?role=syndic`
- `GET /api/v1/syndic/dashboard/{condominium_id}`
- `GET /api/v1/compliance/{condominium_id}/summary`

## Padrões Flutter aplicáveis

- `SindicoHomeBloc` + states `SindicoHomeLoading / SindicoHomeReady(dashboard, compliance) / Error`.
- `freezed` em states + `SindicoDashboard` entity.
- `hydrated_bloc` para offline.
- Rota aninhada `/syndic/:condominium_id` via `go_router` shell.

## Gaps/decisões abertas

- **Q-MOB-HOMES-01** Quais cards aparecem baseado em ABAC ainda não formalizado em policy; assumido 6 cards default + compliance.
- **Q-MOB-HOMES-02** Sindico pode ter múltiplos condomínios ativos simultâneos em M1? Assumido sim (lista em S2).
- **Q-MOB-HOMES-03** Modo apresentação (M3) exige zoom + alto contraste — a definir.

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/sindico/S1]] · [[../../web/ui-catalog/sindico/S2]] · [[../../web/ui-catalog/sindico/S6]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[timeline-read]]
