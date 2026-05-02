---
title: Playbook — Post-Restore Reconcile
type: playbook
tags:
  - playbook
  - master-sindico
  - dr
  - ops
  - stub
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# Playbook — Post-Restore Reconcile

> 🟡 **Stub** criado em 2026-04-27. Procedimento pós-restauração de backup (DR drill, restore real após disaster). Conteúdo a expandir com primeiro DR drill.

## Quando usar

- Após restore de backup (DR drill mensal — [[../../02-architecture/adr/0035-postgres-pitr-wal-archiving]])
- Após restore real após disaster
- Após PITR para ponto específico no tempo

## Reconciliação obrigatória

### 1. Stripe (idempotência)
- Re-processar webhooks `invoice.payment_succeeded` que chegaram durante o gap
- Validar `subscriptions.status` consistente com Stripe

### 2. Zitadel (sessão)
- Invalidar sessões anteriores ao restore-point
- User precisa re-login

### 3. Mux (assets órfãos)
- Listar assets no Mux sem `Video` correspondente no DB
- Marcar para arquivamento ou re-importação

### 4. Audit trail
- Registrar evento `RestoreCompleted` em [[../../01-domain/aggregates/AuditEntry]]
- Notificar DPO + tech lead

### 5. Saga state
- Verificar sagas em execução no momento do snapshot
- Resumir ou compensar conforme [[../../02-architecture/adr/0030-saga-orchestration-default]]

### 6. Outbox
- Re-processar `outbox` table desde o restore-point para garantir delivery

## Validação

- [ ] Counts de tabelas críticas vs último snapshot pré-disaster
- [ ] Tests de smoke (login, billing webhook, vídeo upload, assembly)
- [ ] Audit trail consistente
- [ ] Status page atualizada

## Links

- [[_moc]]
- [[incident-response]]
- [[../../02-architecture/adr/0035-postgres-pitr-wal-archiving]]
- [[../dr-drill]]
