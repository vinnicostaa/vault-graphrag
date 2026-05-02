---
title: Runbook — Incident LGPD Breach (72h ANPD)
type: guide
tags: [runbook, lgpd, incident, anpd, master-sindico, v2]
source: 08-security/lgpd.md + 06-execution/incident.md + LGPD Art. 48
created: 2026-04-23
updated: 2026-04-23
---

# Runbook — Incident LGPD Breach

Resposta a incident que envolve **exposição de dados pessoais** (CPF, CNPJ, email, telefone, endereço, fotos, etc.).

**Prazo legal**: ANPD + titulares notificados em **≤ 72h** (LGPD Art. 48).

**Severidade default**: P0.

**Dono**: DPO (Data Protection Officer) + oncall dev sênior.

---

## Gatilhos

- Alert Sentry: "PII leak detected in logs" (grep regex automático)
- Alert Grafana: query anômala (dump massivo sem auth, cross-tenant leak)
- Denúncia interna (dev detectou em review)
- Denúncia externa (pesquisador segurança, jornalista, titular)
- Incident terceirizado (breach em provider: Stripe, Mux, Zitadel, Livekit, Resend)

Classificar **sempre como P0** até triagem concluir.

---

## Timeline obrigatória

```
T+0    → Detecção
T+15m  → Oncall acknowledge + abrir #incident
T+1h   → DPO notificado (canal dedicado + email)
T+1h   → Isolar o vetor (feature flag OFF / revoke tokens / rollback)
T+4h   → Análise inicial: quais dados, quantos titulares, qual vetor
T+8h   → Contenção confirmada (exfil parada)
T+24h  → Plano de notificação + mitigação aprovado
T+48h  → Auditoria interna: reproduzir root cause; documentar
T+72h  → ANPD comunicada (obrigação legal)
T+72h  → Titulares notificados (email + in-app + SMS)
T+7d   → Comms externa (blog/status page se público)
T+30d  → Plano de prevenção recorrência + postmortem final
```

---

## Step 1 — Acknowledge + isolar (T+0 → T+1h)

### 1.1 Oncall acknowledge

PagerDuty acknowledge dentro de 15min. Postar em `#incident`:

```
[INCIDENT LGPD] Sev P0 — <título curto>
Detectado: HH:MM UTC via <source>
Sintoma: <breve>
Status: INVESTIGANDO. DPO notificado.
Oncall: @<nome>
```

### 1.2 DPO notificado

Canal + email ao DPO dentro de 1h.

### 1.3 Isolar o vetor

Ações possíveis (uma ou várias):
- Feature flag OFF da feature afetada
- Revogar tokens Zitadel comprometidos
- Bloquear IPs maliciosos via rate limit
- Rollback do deploy (se root cause é código recém-deployado)
- Shut down endpoint afetado temporariamente
- Invalidar sessões ativas dos usuários afetados

**Objetivo**: parar exfil em curso antes de qualquer outra coisa.

---

## Step 2 — Análise (T+1h → T+8h)

### 2.1 Quais dados

Checar `audit_trail` + `lgpd_logs`:

```sql
SELECT * FROM lgpd_logs 
WHERE logged_at >= <incident_start>
ORDER BY logged_at;

SELECT * FROM audit_trail
WHERE action ~ 'SELECT|EXPORT|LOGIN'
  AND occurred_at >= <incident_start>
ORDER BY occurred_at;
```

Classificar dados expostos:
- [ ] Dados de identificação (CPF, CNPJ, nome, email)
- [ ] Dados sensíveis (LGPD Art. 5 II): saúde, biometria, orientação, filiação
- [ ] Dados financeiros (Stripe card last4 — tokenizado, menos grave)
- [ ] Conteúdo gerado (vídeos, comunicados)

### 2.2 Quantos titulares

```sql
SELECT COUNT(DISTINCT user_id) FROM audit_trail
WHERE action = '<vetor>'
  AND occurred_at BETWEEN <start> AND <end>;
```

Escalation:
- < 100 titulares: notificação individual
- 100-10k: batch notification + status page
- > 10k: notificação pública + comms imprensa

### 2.3 Root cause

- Bug em código?
- Exploit de vulnerabilidade conhecida?
- Acesso indevido (credencial comprometida)?
- Provider terceiro (Stripe breach, Zitadel issue)?

### 2.4 Vetor de exfil

- Via API (endpoint mal-autenticado)?
- Via DB direct (credential leak)?
- Via logs (PII não-mascarada)?
- Via export (usuário malicioso com privilégio)?

Documentar **tudo** em draft de notificação ANPD.

---

## Step 3 — Contenção (T+8h)

- Fix aplicado (hotfix ou rollback)
- Vulnerabilidade fechada
- Credentials rotacionadas se comprometidas
- Monitor exfil signals — para Sentry alert novo
- Status page atualizada: "Issue mitigated, investigation ongoing"

---

## Step 4 — Notificação ANPD (T+24h → T+72h)

### 4.1 Draft de notificação

Campos obrigatórios (Resolução CD/ANPD nº 15/2024):
- Descrição do incident
- Natureza dos dados afetados
- Quantidade de titulares
- Riscos envolvidos
- Medidas adotadas
- Período de exposição
- Contato DPO

Template: **solicitar ao DPO** (canal dedicado). DPO mantém template + canal ANPD.

### 4.2 Submit ANPD

- Portal ANPD (https://www.gov.br/anpd/)
- DPO submete
- Número de protocolo anexado ao postmortem

### 4.3 Notificação titulares

Canais (todos):
- **Email** (Resend) — template LGPD breach notification pt-BR
- **In-app banner** (web + mobile) — obrigatório dar ciência
- **SMS** (Twilio) — se telefone disponível + incidente alto risco

Conteúdo:
- O que aconteceu (linguagem simples)
- Dados afetados
- O que fizemos pra conter
- O que o titular pode fazer (mudar senha, monitorar cartão, etc.)
- Canal de contato: `dpo@mastersindico.com.br`
- Direitos LGPD (acesso, correção, exclusão, portabilidade)

---

## Step 5 — Comms externa (T+7d)

Se incident é público (jornalista, pesquisador, dump publicado):

- Blog/status page: "Security incident disclosure"
- Redes sociais (opcional)
- Imprensa (opcional; DPO + CEO decidem)
- Investor notification (se aplicável)

---

## Step 6 — Postmortem (T+30d)

Ver [[../../11-audit/postmortems/template]]:

Campos obrigatórios pra incident LGPD:
- ANPD protocol number
- Titulares notificados (count)
- Notificações retornadas (bounces, undeliverable)
- Data de plena remediação
- Ações de prevenção (T-### no backlog)
- Auditoria externa (se incident público, pode ser exigida)

---

## Preveneção contínua

### Práticas obrigatórias

- [ ] PII mask em logs (CI grep bloqueador)
- [ ] Tenant isolation integration tests (100+)
- [ ] ABAC matrix cobertura pentest
- [ ] `govulncheck` + `gosec` verdes em CI
- [ ] Dep-audit + CVE-scan semanal
- [ ] Pentest externo anual (Q2)
- [ ] Treinamento LGPD dev semestral

### Alertas

- Sentry regex: `CPF|CNPJ|\w+@\w+\.\w+` em logs → P1
- Grafana: query anômala (dump grande sem auth) → P1
- Grafana: export endpoint volume anômalo → P2

---

## Contatos

- DPO: `dpo@mastersindico.com.br` (canal dedicado)
- Jurídico: <email>
- ANPD: https://www.gov.br/anpd/ + `anpd@anpd.gov.br`
- Status page: <link>

---

## Links

- [[_moc]]
- [[rollback-deploy]]
- [[webhook-reprocessing]]
- [[../../08-security/lgpd]]
- [[../../08-security/threat-model]]
- [[../../06-execution/incident]]
- [[../../11-audit/postmortems/template]]
- LGPD Art. 48: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm
- ANPD Resolução 15/2024: https://www.gov.br/anpd/pt-br/documentos-e-publicacoes/regulamentos
