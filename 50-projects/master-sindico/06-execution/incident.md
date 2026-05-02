---
title: Incident Response
type: guide
tags: [execution, incident, response, lgpd, master-sindico, v2]
source: legado 06-execution + runbooks incident-response + LGPD spec
created: 2026-04-23
updated: 2026-04-23
---

# Incident Response

Resposta a incidentes em produção. **Objetivo**: detectar rápido, mitigar rápido, aprender sempre.

Cultura: **blameless postmortem** — foco em sistema, não pessoa. Ninguém é demitido por incident bem-investigado.

---

## Classificação de severidade (SLA TTR)

| Sev | Descrição | TTR alvo | Exemplo |
|---|---|---|---|
| **P0** | Platform down, LGPD breach, data loss | **≤ 30min** | API 5xx > 50%; DB corrupto; PII exfiltrated |
| **P1** | Feature crítica indisponível (pagamento, auth, assembleia live) | **≤ 1h** | Stripe webhook não processa; auth Zitadel down; Livekit rooms não abrem |
| **P2** | Degradação parcial (feature não-crítica, lentidão) | **≤ 4h** | Vizinhança cupons lentos; busca OpenSearch lenta; Mux upload demora |
| **P3** | Bug não-bloqueante | **≤ 24h** | UI broken em edge case; relatório errado; email atrasado |
| **P4** | Cosmético / nice-to-have | **próxima sprint** | Typo; cor errada; texto PT incorreto |

---

## Detecção

### Fontes de sinal

1. **Sentry** — alertas automáticos (threshold: > 10 errors novos / 5min em rota crítica)
2. **Grafana / OTel** — dashboards SLI/SLO com alertas PagerDuty
3. **PagerDuty** — rotação oncall 24/7 pós-Sprint 11
4. **Suporte cliente** — canal dedicado + form web
5. **Stripe/Mux/Livekit webhooks** — DLQ monitor
6. **Status page** (pós-M3) — Statuspage.io ou custom

### Alertas principais

| Alerta | Severidade | Trigger |
|---|---|---|
| API error rate > 0.5% | P1 | 15min sustained |
| API p99 > 1s | P2 | 15min sustained |
| DB connection pool exhausted | P0 | imediato |
| Stripe webhook failure > 5% | P1 | 1h sustained |
| Zitadel auth down | P0 | `/auth/login` 5xx > 50% |
| Mux asset stuck > 10min | P2 | DLQ check |
| Livekit room reject rate > 10% | P1 | durante assembleia live |
| LGPD: breach signals (exfil) | P0 | imediato |

---

## Runbook — primeiros 15 min

### 1. Acknowledge (< 1min)

Oncall PagerDuty: acknowledge o alerta. Pra time saber que alguém tá olhando.

### 2. Triage (< 5min)

- Abrir dashboards Grafana (SLI/SLO)
- Abrir Sentry (errors recentes)
- Abrir canal #incident — postar síntese inicial:
  ```
  [INCIDENT] Sev P1 — <título>
  Detectado: 14:32 UTC via PagerDuty
  Sintoma: <o que tá quebrado>
  Impacto estimado: <% users / feature>
  Investigando.
  ```

### 3. Decide (< 10min)

- **Rollback?** ver [[rollback]] — se gatilho bate, rollback primeiro, investiga depois
- **Feature flag OFF?** se feature recente (M2/M3) quebrou
- **Escalate?** P0 chama dev sênior + DPO (se LGPD); P1 chama dev sênior

### 4. Mitigate (< 30min)

Opções:
- Rollback deploy (mais comum)
- Feature flag OFF
- Hotfix em `main` + deploy rápido (se trivial; cuidado com regressão)
- Rate limit agressivo (se DDoS ou bot)
- Cache warmup (se cache miss cascata)
- DB connection pool bump (se esgotou; tactical fix)

### 5. Comms (contínuo)

- #incident atualizar a cada 15min até resolução
- Se P0/P1 prolongado: status page update; email a clientes afetados
- LGPD: ANPD + titulares em 72h (se aplicável)

### 6. Resolve + verify

- Métricas voltam ao baseline?
- Smoke test passa?
- Sentry para de receber errors?
- Atualizar #incident: "RESOLVED"

---

## LGPD — fluxo de breach notification

Se incident envolve **exposição de dados pessoais** (CPF, CNPJ, email, dados sensíveis):

```
T+0    → Detecção
T+1h   → DPO notificado (canal dedicado)
T+4h   → Análise: quais dados, quantos titulares, qual exfil vetor
T+24h  → Plano de notificação + mitigação
T+72h  → ANPD comunicada (obrigação legal LGPD Art. 48)
T+72h  → Titulares notificados (LGPD Art. 48)
T+7d   → Postmortem externo (se público/reputação)
T+30d  → Plano de prevenção recorrência documentado
```

Runbook detalhado: [[../12-docs/runbooks/incident-lgpd-breach]].

Herança: [[../08-security/lgpd]].

---

## Postmortem blameless

Em **todo incident P0/P1** (e P2 se aprendizado relevante):

### Quando

48h pós-resolução (antes do sprint-auditor).

### Template

[[../11-audit/postmortems/template]]:

1. **TL;DR** (2-3 linhas): o quê, quem foi afetado, quanto durou
2. **Timeline**:
   - T+0: detecção
   - T+Xmin: acknowledge
   - T+Ymin: mitigation
   - T+Zmin: resolved
3. **Impacto**:
   - Usuários afetados (count + %)
   - Features bloqueadas
   - Tempo de indisponibilidade
   - SLO impact (error budget consumido)
   - Financeiro (se relevante — Stripe não processou X transações)
   - LGPD (se aplicável)
4. **Root cause** (5 whys ou similar):
   - O que de fato causou? (não "erro humano")
   - Por que passou pelos gates?
   - Qual sistema/processo falhou?
5. **Lições aprendidas**:
   - O que foi bem na resposta?
   - O que foi ruim na resposta?
   - O que podemos mudar no sistema?
6. **Ações de prevenção** (T-### tracking):
   - Alerta novo pra detectar mais cedo
   - Teste novo pra pegar no CI
   - Runbook atualizado
   - Processo mudado
7. **Comms**:
   - Interno (time)
   - Externo (cliente, status page, email)
   - ANPD (se LGPD)

### Cultura blameless

- Nunca: "por que o dev X fez Y" → sempre "por que o sistema permitiu Y"
- Nunca demissão por incident bem-investigado
- Celebrar quem detecta/mitiga rápido
- Aprendizado > punição

---

## Canal + rotação oncall

### Pré-M3

- #incident canal Slack/Discord
- PagerDuty oncall primário (dev sênior) + secundário (dev pleno)
- Rotação semanal (domingo 00:00 UTC handoff)

### Pós-M3

- PagerDuty 24/7 com 3 tiers
- DPO no rotation pra incidents LGPD
- Status page pública

---

## Links

- [[STEPS]]
- [[deploy]]
- [[rollback]]
- [[../08-security/lgpd]]
- [[../11-audit/postmortems/_moc]]
- [[../11-audit/postmortems/template]]
- [[../12-docs/runbooks/incident-lgpd-breach]]
- [[../12-docs/runbooks/rollback-deploy]]
- [[../12-docs/runbooks/webhook-reprocessing]]
