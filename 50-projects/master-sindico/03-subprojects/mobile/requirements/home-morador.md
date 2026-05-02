---
title: Home Morador — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, institutional, feature-req]
project: master-sindico
stack: mobile
priority: m1
status: pending
bounded_context: institutional
created: 2026-04-24
---

# Home Morador — Mobile

Painel do morador autenticado vinculado a condomínio. Shell com **bottom nav 4 abas** (Home / Timeline / Eventos / Perfil) + grade de cards.

## Escopo M1/M2/M3

- **M1**: 4 cards (Linha do Tempo / Plano Diretor / Eventos / Comunicados) + header condomínio + troca de condomínio.
- **M2**: badges de pendência (avaliação bimestral vencida, evento hoje, comunicado não lido).
- **M3**: card Assembleia Live (quando há sessão ativa).

## Telas envolvidas

- `M1` (equivalente web `[[../../web/ui-catalog/morador/M1]]`)
- `M2` (trocar condomínio — modal bottom sheet mobile)

## Funcionais (FR-MOB-HOMEM-N)

- **FR-MOB-HOMEM-1** No primeiro render, carregar `GET /api/v1/institutional/memberships/me/default`.
- **FR-MOB-HOMEM-2** Header fixo: nome do condomínio + ID + ícone "trocar" (se > 1 vínculo).
- **FR-MOB-HOMEM-3** Grade 2x2 de cards (Linha do Tempo / Plano Diretor / Eventos / Comunicados).
- **FR-MOB-HOMEM-4** Card exibe badge numérico quando `GET /api/v1/institutional/dashboard/resident` retorna `{ events_unconfirmed, timeline_unread, assessment_pending }`.
- **FR-MOB-HOMEM-5** Tap no card → `go_router` navega para feature correspondente (preservando tab bottom nav).
- **FR-MOB-HOMEM-6** Pull-to-refresh recarrega memberships + dashboard.
- **FR-MOB-HOMEM-7** Se morador sem vínculo ativo → empty state com CTA "Adicionar acesso" (vai para ONB-M2).
- **FR-MOB-HOMEM-8** Offline: exibir dados do último sync (hydrated_bloc) com banner "Você está offline — dados podem estar desatualizados".

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PERF-1** Primeiro frame com dados do cache < 200ms; fetch remoto em background.
- **NFR-MOB-A11Y-1** Cards com `Semantics(label: '<título>, <badge> pendências', button: true)`.
- **NFR-MOB-OFFLINE-1** Sempre renderizar algo quando offline (last known state).

## Dados locais

- **Hive `memberships`** (TTL 24h) — lista completa do usuário.
- **Hive `dashboard_resident`** (TTL 5min) — counters.
- **hydrated_bloc `HomeBloc`** persiste default_membership_id + last_dashboard.
- **Invalidação**: eventos Bloc `MembershipChanged` ou `DashboardStale` invalidam.

## Integração backend

- `GET /api/v1/institutional/memberships/me` — lista.
- `GET /api/v1/institutional/memberships/me/default` — ativo default.
- `GET /api/v1/institutional/dashboard/resident` — counters.
- `PATCH /api/v1/institutional/memberships/me/default { membership_id }` — trocar.

Sem WebSocket.

## Padrões Flutter aplicáveis

- `HomeBloc` com states freezed `HomeLoading / HomeReady(membership, dashboard) / HomeEmpty / HomeOffline(stale)`.
- `hydrated_bloc` para survivir cold starts.
- `RefreshIndicator` nativo para pull-to-refresh.
- `GridView.count(crossAxisCount: 2)` com `AspectRatio(1.0)` para cards quadrados.
- Deep-link card → `context.push('/timeline')` via `go_router`.

## Gaps/decisões abertas

- **Q-MOB-HOMEM-01** "Votação de Fornecedor" aparece no PDF cliente mas inventário canônico não tem rota própria; assumido = card "Eventos" expandido (ver web M1 §Gaps).
- **Q-MOB-HOMEM-02** Badge "avaliação bimestral pendente" bloqueia navegação ou só avisa? Assumido apenas aviso em M1 (bloqueio em M2 via avaliar-gestao).

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/morador/M1]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[timeline-read]] · [[eventos]] · [[comunicados]] · [[avaliar-gestao]]
