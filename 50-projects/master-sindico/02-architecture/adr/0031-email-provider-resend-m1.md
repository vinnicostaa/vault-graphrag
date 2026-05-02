---
title: ADR-0031 — Provider de email transacional Resend em M1-M2, migração para AWS SES em M3+
type: adr
tags: [adr, email, provider, resend, ses, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
aliases: [email-provider-resend-m1, resend-m1-ses-m3]
---

# ADR-0031 — Resend em M1-M2, SES em M3+

## Status

accepted — 2026-04-23 (formaliza D-026 legado + reavaliação dual-check Fase 5)

## Contexto

D-026 legado (`STATE.md` v1) estava registrado como "SES vs Resend — decidir antes M1" mas sem ADR formal. [[../../11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] (dual-check arch Fase 5) listou como decisão ⚠️ pendente a revisar.

**Requisitos M1** ([[../../04-requirements/global]] GR-002, GR-015, GR-024):

- Verificação de email (Zitadel OIDC callback handoff).
- Password reset.
- LGPD: breach notification (Art. 48), deletion confirmation.
- Billing: confirmação de pagamento Stripe, trial expirando.
- Assembly: edital publicado, convocação.
- Connect Me: interesse recebido, proposta fechada.

**Volume estimado M1**: ~5k emails/mês (1 cliente, 1000 usuários ativos, ~5 touchpoints cada).
**Volume estimado M3+**: ~200k emails/mês (escala multi-cliente + LMS).

Comparação:

| Critério | Resend | AWS SES |
|---|---|---|
| DX (SDK, docs, templates) | 🟢 excelente (React Email nativo) | 🟡 funcional (SESv2 SDK) |
| Setup M1 | 🟢 minutes (API key, domain verify) | 🟡 hours (sandbox → production, DKIM, dedicated IP) |
| LGPD DPA | 🟡 a assinar (Resend Cloud) | 🟢 AWS DPA (public, assinado automaticamente) |
| Localização | 🟡 US-only (Cloudflare R2 storage) | 🟢 sa-east-1 disponível |
| Preço M1 (5k/mês) | 🟢 $0 (free tier 3k/mês) + $20/mês primeiro paid | 🟢 $0.10/1k = $0.50/mês |
| Preço M3+ (200k/mês) | 🟠 $90/mês (100k tier) | 🟢 $20/mês |
| Deliverability BR | 🟡 reputação Resend crescendo | 🟢 SES reputação estabelecida |
| Templates mgmt | 🟢 React Email + versionamento | 🟡 SES templates (simples) |

## Decisão

**Resend em M1-M2. Migração para AWS SES a partir de M3+** quando (a) volume passar de 50k emails/mês, (b) AWS ECS migration estiver em curso ([[0017-railway-initial-aws-m4]]), OU (c) custo Resend exceder $50/mês recorrente.

### M1 (2026-05-08)

- **Provider**: Resend Cloud (free tier 3k/mês + paid tier $20/mês).
- **Domínio verificado**: `mail.mastersindico.com.br` (subdomain dedicado para isolar reputação).
- **SPF + DKIM + DMARC**: configurados via DNS Cloudflare na verificação do domain.
- **DPA Resend**: ⚠️ solicitar assinado **pré-ship M1** — cobre [[../../04-requirements/compliance-lgpd#LGPD-014]].
- **Interface abstrata**: `IEmailProvider` em `application/ports/` — implementação `Resend` com fallback para AWS SES (para reduzir coupling e facilitar migração).

```go
type IEmailProvider interface {
    Send(ctx context.Context, msg EmailMessage) (messageID string, err error)
    Webhook(ctx context.Context, event WebhookEvent) error // bounces, complaints
}
```

### M2 (2026-06-20)

- Continua Resend. Adiciona:
  - **Templates** via React Email ([docs](https://react.email)) — separação frontend team owns templates.
  - **Webhook handlers** para bounce / complaint — atualizar `users.email_status ∈ {valid, bounced, complained}` e pausar sends.
  - **DMARC policy** `p=reject` (stricter).

### M3+ (2026-07-20 onward)

**Gatilhos de migração** (qualquer um dispara):

1. Volume > 50k emails/mês (Resend paid tier $90/mês vs SES ~$5).
2. AWS ECS migration em andamento ([[0017-railway-initial-aws-m4]]) + time de infra tem bandwidth.
3. Resend incident affecting > 1h deliverability.

**Plano de migração**:

1. Provision SES production access (fora do sandbox) — 24-48h AWS aprovação.
2. Verify domain + DKIM keys.
3. Dual-send 1 semana (envia via ambos; compara deliverability via webhook).
4. Flip flag `email_provider = "ses"` em config → traffic 100% SES.
5. Retire Resend API key após 30 dias (mantém webhooks para eventos em-trânsito).

## Consequências

### Positivas

- M1 velocity: Resend setup é trivial (API key + verify domain), libera foco para outros itens críticos.
- React Email templates = owner-shippable pelo frontend team.
- Abstração `IEmailProvider` já prepara para migração (swap implementation, mesmo port).
- LGPD DPA assinado é bloqueio comercial — cobrir pré-M1.

### Negativas

- Custo M2-M3 se volume explodir inesperadamente (R$50→$500/mês em cenário pessimista).
- Resend bounce/complaint handling menos maduro que SES (2026-04-23 estado da arte).
- **Dependência extra**: +1 provider externo para auditar SLA, status page, DPA.

## Regras

1. **DPA Resend assinado pré-M1** — sem isso, ship bloqueado ([[../../04-requirements/compliance-lgpd#LGPD-014]]).
2. **SPF/DKIM/DMARC** validados em CI boot (`Config.Validate()` falha se DNS records não batem).
3. **Nenhum PII em subject line** — GR-007 aplicável a email também.
4. **Template versionado** em git — renderização server-side com `template/html` (auto-escape).
5. **Rate limit** por `user_id` emissor: 10 emails/hora (anti-abuse) + quota global 5k/dia em M1.
6. **Webhook bounce** desativa emails futuros para endereço bounced (status `users.email_status='bounced'`).

## Alternativas consideradas

1. **SES desde M1** — rejeitado: custo setup (sandbox → prod aprovação AWS, DKIM config, dedicated IP se volume) não compensa economia em volume baixo.
2. **Mailgun** — opção viável, mas não oferece templates React modernos e DX pior que Resend (2026-04-23).
3. **Postmark** — excelente deliverability, mas preço > Resend em baixo volume.
4. **Brevo (Sendinbloo)** — BR legal entity boa para LGPD, mas DX e templates modernos inferior.
5. **Self-hosted Postfix + DKIM** — operacional impossível M1 (reputação IP = mês+ para warm-up).

## Referências

- Finding/decisão: [[../../11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] (D-026 legado).
- [[../../04-requirements/compliance-lgpd#LGPD-014]] — DPA obrigatório.
- [[../../04-requirements/functional/cross-domain]] — eventos que disparam emails.
- [[0017-railway-initial-aws-m4]] — migração AWS que impacta M3 trigger.
- [[0015-uow-intra-saga-inter]] — email invocado em sagas (TrialExpirySaga, AccountDeletionSaga).
- Resend docs: https://resend.com/docs
- AWS SES pricing: https://aws.amazon.com/ses/pricing/
- React Email: https://react.email
