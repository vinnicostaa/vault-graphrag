---
title: Calendar — Eventos Recorrentes
type: guide
tags: [schedule, calendar, recurring, master-sindico, v2]
source: cronograma-macro + 06-execution/qa + dep-audit/cve-scan
created: 2026-04-23
updated: 2026-04-23
---

# Calendar — Eventos Recorrentes

Cadência fixa do projeto. Nada aqui é negociável — é ritmo de produto saudável.

---

## Ciclo semanal

| Dia | Hora UTC | Evento | Dono | Duração |
|---|---|---|---|---|
| **Seg** | 09:00 | Sprint planning | Dev + Arquiteto | 1h |
| **Seg** | 10:00 | Standup async no canal #dev | Time | 15min (async) |
| Ter | — | Execução | — | — |
| Qua | — | Execução | — | — |
| **Qui** | 15:00 | Mid-sprint check-in | Dev | 30min |
| Qui | — | Execução | — | — |
| **Sex** | 14:00 | Grooming backlog (próxima sprint) | Dev + PO | 1h |
| Sex | 16:00 | Polish + staging deploy | Dev | — |
| Sex | 17:00 | Review (PRs abertas) | Time | — |
| Sáb | — | Buffer (folga ou crítico) | — | — |
| **Dom** | 20:00 | Sprint retrospectiva (self-review) | Dev | 30min |
| **Dom** | 22:00 | **Sprint-auditor** roda (skill) | Agente | 1-2h autônomo |
| **Dom** | 23:00 | AUDIT atualizado; SESSION_CHARTER próxima sprint | Dev | 15min |

---

## Ciclo mensal

| Dia | Evento | Dono |
|---|---|---|
| **1º de cada mês** | Reset quotas vídeos (cron BIL-008/009) | Backend (auto) |
| **1ª seg do mês** | Review SLO + error budget | Dev + Ops |
| **1ª sex do mês** | Dep-audit completo (todos sub-projetos) | Sprint-auditor |
| **Última sex** | Runbooks review (atualizar se stale) | Ops + Dev |

---

## Ciclo trimestral

| Trimestre | Evento | Dono |
|---|---|---|
| **Q1 fim** (31/Mar) | Backup/restore test | Ops |
| **Q2 fim** (30/Jun) | Rotação de secrets (Stripe, Zitadel, Twilio, Mux, Livekit, Resend) | Ops + Dev |
| **Q3 fim** (30/Set) | LGPD compliance review | DPO + Dev |
| **Q4 fim** (31/Dez) | Pen-test interno | Dev + agente |
| **Todo Q** | Runbooks review trimestral | Ops |

---

## Ciclo anual

| Data | Evento | Dono |
|---|---|---|
| **01/Jan** | Reset quotas anuais Connect Me (cron BIL-009) | Backend (auto) |
| **Q2** | Pen-test externo (contratado) | Dev + fornecedor |
| **Q3** | Auditoria LGPD formal (externa) | DPO + auditor |
| **Q4** | SOC 2 prep (pós escalar) | Ops + Compliance |

---

## Marcos de deploy (2026)

| Data | Marco | Janela |
|---|---|---|
| **08/Mai 2026** | 🟢 M1 — MVP Contratual | 00:00-06:00 UTC (freeze 48h antes) |
| **20/Jun 2026** | 🟡 M2 — Connect Me + Conteúdo | 00:00-06:00 UTC |
| **20/Jul 2026** | 🔵 M3 — Assembleia + LMS + Polish | 00:00-06:00 UTC |

---

## Janelas de manutenção

- **Deploy normal** (não-marco): qualquer hora, zero-downtime rolling Railway
- **Migration pesada**: sáb 03:00-05:00 UTC (menor tráfego BR)
- **Maintenance programada**: anunciar 72h antes via status page
- **Incident response**: sem janela — mitigation imediata

---

## Sprint windows (2026)

Ver [[cronograma-macro]] pra detalhe sprint-a-sprint.

- Sprint 10: 2026-04-22 → 2026-05-04 (finaliza M1)
- Sprint 11: 2026-05-06 → 2026-05-18 (M2 kickoff)
- Sprint 12: 2026-05-13 → 2026-05-25
- Sprint 13: 2026-05-20 → 2026-06-01
- Sprint 14: 2026-06-24 → 2026-07-06 (M3 kickoff)
- Sprint 15: 2026-07-01 → 2026-07-13
- Sprint 16: 2026-07-08 → 2026-07-20 (finaliza M3)

---

## Feriados BR relevantes

| Data | Nome | Impacto |
|---|---|---|
| 01/Mai | Dia Trabalho | Qui — buffer Sprint 10 |
| 06/Set (2026) | Independência (dom) | Sprint-auditor normal |
| 07/Set (2026) | Independência (seg) | Planning pode adiar 1 dia |
| 12/Out | N. Sra. Aparecida | Seg — planning adiar |
| 02/Nov | Finados | Seg — planning adiar |
| 15/Nov | Proclamação República | Dom — sprint-auditor normal |
| 25/Dez | Natal | Dom-Sex 21-26/Dez freeze opcional |

---

## CI/CD janelas

### Deploy windows (Railway)

- **Staging**: deploy automático em merge `main` (qualquer hora)
- **Produção**: 
  - Deploy normal (não-marco): horário comercial BR 09:00-17:00 BRT
  - Deploy marco (M1/M2/M3): 00:00-06:00 UTC (= 21:00-03:00 BRT dia anterior)
  - **Freeze** 48h antes de deploy marco

### Cron jobs (backend)

| Cron | Schedule | O que faz |
|---|---|---|
| `reset-quotas-monthly` | `0 0 1 * *` (1º do mês 00:00 UTC) | Reset `feature_usages.videos` |
| `reset-quotas-yearly` | `0 0 1 1 *` (1º Jan 00:00 UTC) | Reset `feature_usages.connect_me` |
| `auto-close-connect-me` | `*/15 * * * *` (a cada 15min) | Fecha RFPs com 48h sem interesse |
| `governance-score` | `0 3 * * *` (diário 03:00 UTC) | Calcula score diário condomínio |
| `mandate-end-check` | `0 0 * * *` (diário 00:00) | Alerta mandatos terminando 30d |
| `trial-expiring-soon` | `0 */6 * * *` (4x/dia) | Notifica 3d antes de trial expirar |
| `lgpd-retention-purge` | `0 4 * * *` (diário 04:00) | Remove dados com retenção expirada |
| `webhook-dlq-retry` | `*/5 * * * *` (a cada 5min) | Retry webhooks em DLQ |
| `video-lock-unlock` | `0 1 * * *` (diário 01:00) | Desbloqueia vídeos locked após 90d |

Referência: [[../90-ingested/_consolidado-providers-fluxos]] (22 crons catalogados).

---

## Links

- [[_moc]]
- [[cronograma-macro]]
- [[milestones]]
- [[../06-execution/qa]]
- [[../06-execution/playbooks/dep-audit]]
- [[../06-execution/playbooks/cve-scan]]
