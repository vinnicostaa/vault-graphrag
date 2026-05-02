---
title: Playbook — Incident Response
type: playbook
tags:
  - playbook
  - master-sindico
  - incident
  - ops
  - stub
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# Playbook — Incident Response

> 🟡 **Stub** criado em 2026-04-27. Playbook canônico de resposta a incidentes (segurança, indisponibilidade, vazamento). Conteúdo a expandir com experiência de DR drill.

## Fases

### 1. Detecção
- Sentry alert + Grafana SLO breach
- User report
- Vendor notification (Stripe, Mux, Cloudflare)

### 2. Triage (within 15min)
- Severidade: SEV-1 (catastrófico/vazamento) → SEV-4 (cosmético)
- War room via Slack `#incident-<YYYY-MM-DD>` 
- Incident commander designado

### 3. Contenção
- Rollback (ver [[rollback]] e [[../../30-agents/playbooks/rollback]])
- Feature flag kill-switch
- Isolamento de dados (revoke tokens, kick sessions)

### 4. Comunicação
- Status page atualizada **em até 15min** após confirmação
- Email afetados se LGPD ([[../../12-docs/incident-communication-template]])
- ANPD em até 2 dias úteis se vazamento

### 5. Resolução
- Fix + deploy + verify
- Atualizar status page

### 6. Post-mortem (within 5 dias úteis)
- Root cause analysis
- Action items (tracked em [[../../11-audit/postmortems/_moc]])
- Atualização deste playbook se gap identificado

## Quando escalar

| Severidade | Tempo até escalar | Escalonamento |
|---|---|---|
| SEV-1 | imediato | CEO + DPO + jurídico |
| SEV-2 | 1h | Tech lead + Product |
| SEV-3 | 4h | Tech lead |
| SEV-4 | 24h | — |

## Links

- [[_moc]]
- [[rollback]]
- [[post-restore-reconcile]]
- [[../../08-security/cve-process]]
- [[../../12-docs/incident-communication-template]]
- [[../../11-audit/postmortems/_moc]]
