---
title: Avaliar Gestão — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, reputation, feature-req]
project: master-sindico
stack: mobile
priority: m2
status: pending
bounded_context: commercial
created: 2026-04-24
---

# Avaliar Gestão — Mobile

Avaliação **bimestral obrigatória** do morador sobre a gestão do síndico. **3 critérios fixos (Transparência / Eficácia / Comunicação)** com nota 1-10. Feeds score Reputação (D-051).

## Escopo M1/M2/M3

- **M1**: apenas aviso/badge (pendência visível no home-morador).
- **M2**: fluxo completo de submissão (3 sliders + comentário opcional).
- **M3**: histórico pessoal de avaliações + visualização de impacto no score.

## Telas envolvidas

- `M12` (avaliar gestão; `[[../../web/ui-catalog/morador/M12]]`)
- `M12-BLOCK` (bloqueio leve se avaliação vencida há > 30d — passo extra antes de home em M3)

## Funcionais (FR-MOB-AVAL-N)

- **FR-MOB-AVAL-1** `GET /api/v1/reputation/assessment/pending` retorna `{ due: bool, period: '2026-06', syndic: {...} }`.
- **FR-MOB-AVAL-2** Se `due=true` → badge no card home + notificação silent (1x por ciclo).
- **FR-MOB-AVAL-3** Tela exibe 3 sliders horizontais 1-10 (com valor atual, tooltip descritivo).
- **FR-MOB-AVAL-4** Campo opcional `comment` (max 500 chars).
- **FR-MOB-AVAL-5** `POST /api/v1/reputation/assessment { period, scores: {transparencia, eficacia, comunicacao}, comment }` — irreversível.
- **FR-MOB-AVAL-6** Após submit, tela de agradecimento + link "Ver impacto" (M3).
- **FR-MOB-AVAL-7** Avaliação anônima para síndico (só agregado); audit guarda user_id.
- **FR-MOB-AVAL-8** Se tentar avaliar fora do período → erro `422 OUT_OF_WINDOW`.

## Não-funcionais (NFR-MOB)

- **NFR-MOB-A11Y-1** Sliders com `Semantics(value: '<N> de 10')` e ajuste via volume-up/down (Android a11y).
- **NFR-MOB-UX-1** Touch target de slider thumb ≥ 48dp.
- **NFR-MOB-SEC-1** Payload não inclui fingerprint do device (somente user_id no Bearer).

## Dados locais

- **SharedPreferences** `last_assessment_period_submitted` (evita nag se já enviou).
- **Hive `assessment_pending`** (TTL 1h) — cache do status.
- **NÃO persistir scores não-submitidos** (avaliação rascunho = web; mobile = submit atomic).
- **Invalidação**: após submit 201 → limpar pendência local + refetch.

## Integração backend

- `GET /api/v1/reputation/assessment/pending`
- `POST /api/v1/reputation/assessment`
- `GET /api/v1/reputation/assessment/history/me` (M3)

Sem WebSocket. Push silent no início do ciclo.

## Padrões Flutter aplicáveis

- `AssessmentBloc` + states `AssessmentLoading / AssessmentIdle(pending) / AssessmentSubmitting / AssessmentDone / AssessmentOutOfWindow`.
- `bloc_concurrency.droppable()` no `SubmitRequested` (evita duplo envio).
- `freezed` em `AssessmentScores` + `AssessmentPayload`.
- `Slider` widget Material 3 customizado.

## Gaps/decisões abertas

- **Q-MOB-AVAL-01** Bloqueio de acesso ao home se > 30d pendente — implementar em M2 ou M3? Assumido M3 (M2 apenas badge).
- **Q-MOB-AVAL-02** Comment livre pode ter moderação? Assumido server-side em M3 (M2 sem moderação).
- **Q-MOB-AVAL-03** Avaliação multi-síndico (condomínio com 2 síndicos ativos)? Depende de decisão reputação D-051 estendida.

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/morador/M12]]
- [[../../../00-product/sub-produtos/03-reputacao]]
- [[../../../STATE#D-051|D-051 Fórmula Reputação]]
