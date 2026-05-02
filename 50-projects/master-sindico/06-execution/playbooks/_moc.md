---
title: "MOC — 06-execution/playbooks"
type: moc
tags: [moc, master-sindico, v2, execution, playbooks]
created: 2026-04-23
updated: 2026-04-24
---

# MOC — 06-execution/playbooks

Playbooks do projeto Master Síndico v2. Cada playbook estende o correspondente em `vault/30-agents/playbooks/` com especialização local.

## Playbooks de processo

- [[research-loop]] — research-loop antes de decidir (5-stage; vault base)
- [[dual-check]] — revisão cruzada obrigatória em decisões não-triviais
- [[dep-audit]] — auditoria de deps (semanal + pre-release)
- [[cve-scan]] — varredura CVE (CI + semanal + emergency)
- [[plan-and-execute]] — SDD 5-fase aplicado a task
- [[rollback]] — rollback tático de deploy (Railway)
- [[onboarding-novo-projeto]] — fluxo dev entrando no Master Síndico v2

## Playbooks operacionais (incident response — 2026-04-24)

- [[stripe-down]] — Stripe webhook failure / API 5xx; idempotency replay; reconciliação subs (P0/P1, billing)
- [[zitadel-down]] — auth outage; estender TTL session; cache stale fallback (P0, identity — produto inteiro)
- [[mux-down]] — VOD stuck > 10min; saga compensation manual; reconciliação assets (P1, content)
- [[livekit-down-assembly]] — Livekit caindo durante assembleia ao vivo; decisão jurídica recesso/fallback/cancelar; preservar gravação (P0, assembly + legal)
- [[cross-tenant-leak]] — vazamento multi-tenant; contenção sem comms premature; LGPD T+1h DPO / T+72h ANPD (P0 absoluto, blocker)

## Playbooks operacionais (stubs — D-vault-025 2026-04-27)

- [[incident-response]] — resposta canônica a incidentes (segurança, indisponibilidade, vazamento) — fases detecção → triage → contenção → comunicação → resolução → post-mortem
- [[post-restore-reconcile]] — reconciliação pós-restore de backup (DR drill, restore real) — Stripe/Zitadel/Mux/saga state/outbox

## Vizinhos

- [[../_moc]]
- [[../incident]] — fluxo geral incident response + severidade SLA
- [[../rollback]] — procedimento rollback Railway
- [[../dr-drill]] — DR mensal (PITR, RPO/RTO)
- [[../../CLAUDE]]
- [[../../STATE]]
- [[../../11-audit/AUDIT]]
- [[../../11-audit/postmortems/template]]
- [[../../../vault/30-agents/playbooks/_moc]] — playbooks base genéricos
