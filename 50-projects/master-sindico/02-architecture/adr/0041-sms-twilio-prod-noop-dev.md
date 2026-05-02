---
title: ADR-0041 — SMS provider Twilio em prod, NoopProvider em dev (cliente ainda não liberou conta Twilio)
type: adr
tags: [adr, sms, provider, twilio, master-sindico, v2, fase-14]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0041 — SMS Twilio prod / Noop dev

## Status

`accepted` — 2026-04-23 (Fase 14, derivado de D-118).

## Contexto

Master Síndico precisa de SMS para:

1. **MFA backup channel** (TOTP é primário — [[0026-admin-mfa-m1]]; SMS para fallback OTP em caso de perda de device).
2. **OTP de verificação** em fluxos de signup mobile (alternativa a email para personas com baixa literacia digital — moradores idosos).
3. **Alertas críticos opcionais** opt-in: assembleia começa em 30min, voto encerra em X.
4. **Recovery de conta** quando email não disponível.

**Restrições conhecidas**:

- **Custo**: SMS Brasil via Twilio ~R$0,15-0,40 por mensagem. Volume M1 (5k SMS/mês cap) ≈ R$1.000/mês max.
- **Compliance**: LGPD (telefone é PII; consentimento explícito em opt-in).
- **Anti-abuse**: rate limit indispensável (cada SMS custa dinheiro real).
- **Cliente ainda não liberou conta Twilio**: João explicitou na sessão 2026-04-23 que o ambiente Twilio brasileiro não está habilitado pelo cliente — provedor real só em prod **quando** ele liberar.

Cliente disse:
> *"em dev nao vamos usar provider de SMS ate o cliente liberar a conta na Twilio"*

## Decisão

**Provider único Twilio em prod, NoopProvider em dev e staging**, com interface abstrata `ISmsProvider` (espelha [[0031-email-provider-resend-m1]] `IEmailProvider`).

### Implementações

```go
// application/ports/sms_provider.go
type ISmsProvider interface {
    Send(ctx context.Context, msg SmsMessage) (messageID string, err error)
    Webhook(ctx context.Context, event SmsWebhookEvent) error // delivery, failed
}

type SmsMessage struct {
    To       PhoneE164  // formato E.164 (+5511999999999)
    Template SmsTemplate
    Vars     map[string]string
    UserID   UserID     // para rate-limit + audit
    Reason   SmsReason  // mfa_otp | signup_otp | assembly_alert | account_recovery
}
```

| Implementação | Quando | Comportamento |
|---|---|---|
| `TwilioSmsProvider` | env `prod` apenas | Envia via Twilio Programmable SMS (BR region); webhook delivery → atualiza `users.phone_status`. Bloqueio de envio se `phone_status = invalid` (number lookup) ou `phone_consent = false`. |
| `NoopSmsProvider` | env `dev` e `staging` | **Não envia**. Loga `[SMS-NOOP] To=+55... Template=mfa_otp Vars={code:123456}` em logs estruturados (zerolog [[0008-zerolog-structured-logs]]). UI dev tem painel `/dev/sms-log` (apenas em build dev) para inspecionar OTPs. |
| `TestSmsProvider` | testes unitários | Captura mensagens em slice em-memória para assertions. |

### Configuração

```yaml
# config.yaml
sms:
  enabled: true        # gates the whole feature
  provider: noop       # dev/staging — overridden em prod=twilio
  twilio:
    account_sid: ${TWILIO_ACCOUNT_SID}     # apenas prod
    auth_token:  ${TWILIO_AUTH_TOKEN}       # apenas prod
    from_number: ${TWILIO_FROM_NUMBER}      # E.164 BR
    region:      sa1                        # São Paulo POP
  rate_limits:
    per_user_per_day: 10
    global_per_day:   500     # escalável por gatilho de evento
  templates:
    mfa_otp:           "Master Sindico: codigo {{code}}. Valido 5min. Nao compartilhe."
    signup_otp:        "Master Sindico: ative seu cadastro com {{code}}."
    assembly_alert:    "Master Sindico: assembleia comeca em 30min. Acesse {{short_url}}."
    account_recovery:  "Master Sindico: clique para recuperar conta {{short_url}}."
```

**Bootstrap rule**: em prod, se `provider != "twilio"` ou credenciais ausentes → **falha imediata**. Em dev/staging, default `noop` é seguro.

### Templates

- Curtos (BR Twilio long code = 160 chars antes de chunking).
- **Sem PII além do necessário** (não citar nome unidade, condomínio em SMS).
- Idioma único pt-BR M1 (M3+ avaliar i18n).
- Versionados em git; renderização server-side.

### Rate limits & quota

- **Por user**: 10 SMS/dia (anti-abuse + custo).
- **Global**: 500 SMS/dia (cap inicial; ajustável conforme volume real).
- **Por reason**: `mfa_otp` permite até 3 retry/15min/user; `assembly_alert` 1×/assembleia/user; `account_recovery` 3×/dia/user.
- Excedido: response `429 Too Many Requests`; UI sugere email fallback.

### Webhook handling

- Twilio dispara `delivered`, `failed`, `undelivered` para `webhooks.mastersindico.com.br/sms/twilio` (Workers BFF — [[0040-cloudflare-edge-primary-railway-origin]]).
- Idempotency: `MessageSid` é PK (UNIQUE em DB) — [[0027-webhook-idempotency-db]].
- Status atualiza `users.phone_status` ∈ {`verified`, `unverified`, `invalid`, `bounced`}.
- `bounced` (3 falhas seguidas) → desativa SMS para esse número; UI mostra "número inválido — atualize cadastro".

### Consentimento LGPD

- Telefone capturado no signup (opcional ou obrigatório dependendo do fluxo).
- **Consentimento explícito** para uso em SMS marketing/alerta opt-in: `users.sms_consent_given_at`, `users.sms_consent_revoked_at`.
- MFA OTP / signup OTP / account recovery: legítimo interesse (segurança) — não exige consent extra.
- Audit trail de todo SMS enviado (não conteúdo, mas: user_id, reason, status, timestamp) — [[0028-lgpd-keyed-hmac]].

## Consequências

### Positivas

- **Zero custo SMS em dev/staging**: NoopProvider + log = UX dev limpa, OTPs visíveis no painel `/dev/sms-log`.
- **Switch trivial dev→prod**: configuração; sem mudança de código.
- **Test coverage real**: TestSmsProvider valida fluxos OTP em CI sem hit em provider externo.
- **Anti-abuse forte**: rate-limit per-user + global + per-reason = cap previsível de spend.
- **Webhook idempotente**: bounce/delivered handled corretamente, atualiza estado.
- **LGPD compliance**: consentimento separado de OTP de segurança (legítimo interesse).

### Negativas

- **Dev pode esquecer que SMS não é enviado**: mitigação via banner UI dev "SMS NoopProvider ativo" em `/dev/sms-log`.
- **Dependência de Twilio aprovação BR**: cliente bloqueia ship M1 com SMS real. Mitigação: M1 ship com SMS-prod desligado (`sms.enabled=false`) → MFA via TOTP only; ativar SMS prod quando Twilio liberado.
- **DPA Twilio** precisa assinatura (LGPD) — bloqueio comercial pré-ativação prod.
- **Custo escalável**: monitorar em [[0020-opentelemetry-grafana-sentry]] dashboard `sms_sent_per_day`; alarme se cap diário 80%.

### Neutras

- Não há provider redundante M1 (ex: AWS SNS como fallback). Simplicidade > redundância nesse volume. Debt M3+ se Twilio incidente cause queda longa.

## Trade-offs explícitos

- **Sem fallback automático Twilio→outro provider** M1: aceitar SLO Twilio (~99.9%); em incidente, MFA via TOTP cobre — SMS é canal complementar.
- **Sem RCS / WhatsApp Business** M1: foco Twilio SMS only. WhatsApp Business via Twilio Conversations API debt M2+ se cliente pedir (canal mais barato e maior engagement em BR).

## Alternativas consideradas

1. **AWS SNS** — recusada M1: custo similar Twilio em BR; DX inferior; deliverability semelhante; vantagem (entrar em ecossistema AWS) só relevante M4+ ([[0017-railway-initial-aws-m4]]).
2. **MessageBird** — recusada: BR coverage menor que Twilio; preço comparável.
3. **Zenvia / Movile (BR-native)** — viável; recusada por DX inferior e aprendizado time vs Twilio que já é mainstream.
4. **WhatsApp Business como canal único** — recusada M1: aprovação Meta para template e número leva semanas; OTP em SMS é mais universal e instantâneo.
5. **Sem SMS em M1** — recusada: MFA backup channel é requisito de segurança ([[0026-admin-mfa-m1]]); senha-reset via apenas email é vulnerabilidade conhecida (compromise email = takeover).

## Regras

1. **DPA Twilio assinado pré-ativação prod** — sem isso, ativação bloqueada.
2. **Bootstrap CI**: em prod, `provider=twilio` + credenciais presentes obrigatório (`Config.Validate()` falha caso contrário).
3. **OTP nunca em log estruturado prod** — só hash + reason + status.
4. **Consent LGPD verificado antes de envio non-security** — `users.sms_consent_given_at != null AND sms_consent_revoked_at == null`.
5. **Rate limit imune a bypass por admin** — exceção apenas via flag explícita `bypass_rate_limit_reason` em audit log.
6. **Phone E.164** validado em entrada (regex + Twilio Lookup API opcional para PJ).

## Aplicação progressiva

| Marco | Entrega |
|---|---|
| **pré-M1 (Sprint 10)** | `ISmsProvider` interface + `NoopSmsProvider` + `TestSmsProvider` + painel `/dev/sms-log`; templates + rate-limit middleware; webhook handler com idempotency |
| **M1 (2026-05-18)** | Ship com `sms.enabled=true` + `provider=noop` em dev/staging; **`sms.enabled=false` em prod** até cliente liberar Twilio; MFA TOTP-only |
| **Quando cliente liberar Twilio** | DPA assinado; credenciais em secrets manager; flip `sms.enabled=true` + `provider=twilio` em prod; smoke test OTP |
| **M2** | Dashboard `sms_sent_per_day` + alarmes; WhatsApp Business API avaliação (Twilio Conversations) |
| **M3+** | RCS evaluation; multi-provider failover (SNS fallback) se incidente Twilio significativo |

## Referências

- [[../../../../10-knowledge/security/mfa-step-up]] — MFA pattern reference
- [[../../../../10-knowledge/security/otp-hotp-totp]] — OTP fundamentals (TOTP primário; SMS-OTP fallback)
- [[0026-admin-mfa-m1]] — MFA admin obrigatório (este ADR é o canal SMS dele)
- [[0027-webhook-idempotency-db]] — webhook DB UNIQUE handling (Twilio segue mesmo padrão)
- [[0028-lgpd-keyed-hmac]] — pseudonimização (telefone é PII)
- [[0040-cloudflare-edge-primary-railway-origin]] — Workers BFF recebe webhooks Twilio
- [[../../STATE]] §D-118
- Twilio Programmable SMS Brasil: https://www.twilio.com/pt-br/sms (consultar quando ativar)
