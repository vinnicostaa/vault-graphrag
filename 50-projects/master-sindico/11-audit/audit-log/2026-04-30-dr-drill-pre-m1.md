---
title: DR Drill pré-M1 — 2026-04-30
type: audit
tags:
  - drill
  - dr
  - postgres
  - pitr
  - pre-m1
  - master-sindico
created: 2026-04-30T00:00:00.000Z
date_planned: 2026-04-30T00:00:00.000Z
owner: SRE on-call (rotação Sprint 10)
status: planned
blocks_milestone: M1 (2026-05-18)
references:
  - adr-0035
  - dr-drill-playbook
  - T-DR-01
---

# DR Drill pré-M1 — 2026-04-30

> ⚠️ **Drill especial obrigatório antes do M1 (2026-05-18)**. Cadência regular cairia em 2026-05-05 (1ª terça de maio), dentro da janela de freeze e tarde demais para corrigir falhas. Este é o **gate de aceite M1** para infraestrutura de backup. Se falhar, repete-se até passar — ainda há buffer 7-12 dias.

## Plano

- **Data planejada**: 2026-04-30 (quinta), 14:00 UTC-3
- **Janela**: 14:00 → 16:00 UTC-3 (com tear-down)
- **Owner**: SRE on-call (rotação Sprint 10) — atribuir 24h antes em `#ms-ops`
- **Aprovação**: Tech Lead (João)
- **Procedimento**: ver [[../../06-execution/dr-drill]] §Procedure (8 etapas)
- **Critérios de aceite (gate M1)**:
  - [ ] RTO observado ≤ 30min para identity/billing/assembly (target ADR-0035)
  - [ ] RTO observado ≤ 4h global
  - [ ] RPO observado ≤ 5min
  - [ ] 7/7 smoke tests passam (login, billing, timeline, assembly, ata, audit, RLS)
  - [ ] Reconcile Stripe dry-run < 10 mismatches
  - [ ] Reconcile Mux dry-run < 10 orphans
  - [ ] RLS confirmadamente ativa (query sem `SET app.tenant_id` retorna 0 rows)

## Particularidades pré-M1

Diferente de drill rotineiro:

1. **Volume de dados production-like**: staging deve ser populado **antes** do drill com:
   - 15 condomínios × 100 unidades = 1500 units
   - 6 meses de timeline (~50 entries por condomínio = 750 entries)
   - 50 vídeos Mux (com asset IDs reais via test mode)
   - 5 assembleias Livekit completas (com atas homologadas)
   - 200 users (mix de personas) + memberships
   - 100 ConnectMeRequests (4 fluxos misturados)
   - 30 ClosedDeals com dupla assinatura
   - **Não é dado real de cliente** — gerado via `internal/shared/testutil/seed_realistic.go` (a criar)
2. **Testar adapter de search D-120**: validar que reindexação OpenSearch/Meili pós-restore funciona via outbox replay
3. **Testar `domain_events` outbox**: confirmar que eventos pós-target_time são re-emitidos sem dupla execução (idempotência)
4. **Validar ADR-0034 RLS default-on**: confirmar que TODAS as tabelas têm RLS ativa (não só sensíveis)
5. **Validar ADR-0027 webhook idempotency**: re-injetar webhooks Stripe pós-restore — não devem duplicar charges
6. **Validar `TenantBoundaryGuard`**: chamar endpoint cross-tenant — esperado 403

## Contingência

Se drill falhar em qualquer critério:

- **1ª falha**: investigar root cause (24h), aplicar fix, re-rodar até 2026-05-04
- **2ª falha consecutiva**: incident Sev2 → escalation → considerar adiar M1 (D-### no STATE)
- **Falha em RLS / TenantBoundaryGuard / outbox / search**: bloqueia M1 absolutamente; correção + re-drill obrigatórios

## Resultado (preencher no dia)

- [ ] Pré-voo concluído (15min)
- [ ] Provisioning staging-dr concluído (10min)
- [ ] Restore concluído — RTO_obs: ___ min (target: 30min identity/billing/assembly · 4h global)  ✅|❌
- [ ] RPO_obs: ___ min (target: 5min)  ✅|❌
- [ ] Smoke tests: ___/7 passed
- [ ] Reconcile dry-run — Stripe: ___ mismatches | Mux: ___ orphans (target: < 10 cada)
- [ ] RLS validation: ✅|❌
- [ ] Tear-down concluído

## Incidentes detectados

(preencher pós-drill)

- [ ] Nenhum
- [ ] _Listar findings + severidade + plano de fix_

## Ações abertas

(preencher pós-drill)

- [ ] Nenhuma
- [ ] _Listar com owner + prazo_

## Veredicto final

- [ ] ✅ M1 LIBERADO — backup+restore validados production-grade
- [ ] ❌ M1 BLOQUEADO — repete drill até passar; ver "Contingência"

## Vizinhos

- [[../../06-execution/dr-drill]]
- [[../../02-architecture/adr/0035-postgres-pitr-wal-archiving]]
- [[../../05-tasks/by-sprint/sprint-10#T-DR-01]]
- [[../AUDIT]]
- [[../../STATE#D-106]]
