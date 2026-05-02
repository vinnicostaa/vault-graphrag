---
title: How-to — Adicionar provider externo
type: guide
tags: [how-to, provider, integration, master-sindico, v2]
source: 02-architecture/patterns + STEERING + exemplos Stripe/Mux/Livekit/Resend do legado
created: 2026-04-23
updated: 2026-04-23
---

# How-to — Adicionar provider externo

Como adicionar novo provider (ex: SMS alternativo, storage alternativo, novo SDK) seguindo Clean Architecture estrita + DIP.

**Princípio**: SDK externo **nunca** importado no domain/. Sempre via interface em `shared/providers/<kind>/` + impl em `infrastructure/providers/<provider>/`.

---

## 1. Research-loop

Antes de escolher provider, rodar [[../../06-execution/playbooks/research-loop]]:

1. Doc oficial do candidato
2. Alternativas (≥ 2)
3. Critérios:
   - License + modelo de pricing
   - Manutenção ativa (último release ≤ 6m)
   - Suporte BR / LGPD compliance
   - Latência + SLA
   - Webhooks (se aplicável)
   - Rate limits + quotas
   - Custo estimado em escala M3+ (1000+ condomínios)
4. [[../../06-execution/playbooks/dual-check]] antes de fechar

Registrar em [[../../STATE]] como D-### + ADR em [[../../02-architecture/adr/_moc]].

---

## 2. Definir interface (DIP)

Arquivo: `backend/internal/shared/providers/<kind>/interface.go`

Exemplo (SMS):

```go
// Package sms defines provider-agnostic SMS contracts.
package sms

import "context"

// Message é o payload agnóstico ao provider.
type Message struct {
    To      string // E.164
    Body    string
    TraceID string // correlação com request
}

// Response is the provider-agnostic send outcome.
type Response struct {
    MessageID string
    Status    Status
}

type Status string

const (
    StatusSent    Status = "sent"
    StatusQueued  Status = "queued"
    StatusFailed  Status = "failed"
)

// ISMSProvider é o contrato implementado por adapters.
type ISMSProvider interface {
    Send(ctx context.Context, msg Message) (Response, error)
}
```

**Regras**:
- Interface **não menciona** SDK-specific types (nada de `twilio.Message` aqui)
- Tipos próprios (`Message`, `Response`) são value objects próprios
- Context sempre primeiro param (OTel + cancelamento)
- Error wrappable (caller decide `errors.Is`)

---

## 3. Implementar adapter

Arquivo: `backend/internal/modules/<consumer-bc>/infrastructure/providers/<provider>/<provider>_adapter.go`

Ou se é provider cross-BC: `backend/internal/shared/providers/<kind>/<provider>/<provider>_adapter.go`

Exemplo (Twilio SMS):

```go
package twilio

import (
    "context"
    "fmt"

    "github.com/twilio/twilio-go"
    twiapi "github.com/twilio/twilio-go/rest/api/v2010"

    "mastersindico/internal/shared/providers/sms"
)

// Provider implementa sms.ISMSProvider usando Twilio SDK.
type Provider struct {
    client     *twilio.RestClient
    fromNumber string
    logger     *zerolog.Logger
}

func NewProvider(accountSID, authToken, fromNumber string, logger *zerolog.Logger) *Provider {
    client := twilio.NewRestClientWithParams(twilio.ClientParams{
        Username: accountSID,
        Password: authToken,
    })
    return &Provider{client: client, fromNumber: fromNumber, logger: logger}
}

func (p *Provider) Send(ctx context.Context, msg sms.Message) (sms.Response, error) {
    params := &twiapi.CreateMessageParams{}
    params.SetTo(msg.To)
    params.SetFrom(p.fromNumber)
    params.SetBody(msg.Body)

    resp, err := p.client.Api.CreateMessage(params)
    if err != nil {
        p.logger.Error().
            Err(err).
            Str("trace_id", msg.TraceID).
            Msg("twilio sms send failed")
        return sms.Response{}, fmt.Errorf("twilio send: %w", err)
    }

    return sms.Response{
        MessageID: *resp.Sid,
        Status:    sms.Status(*resp.Status),
    }, nil
}
```

**Regras**:
- Adapter **isola** SDK (`github.com/twilio/twilio-go` não aparece em nenhum outro arquivo além desse)
- Log estruturado (sem PII — `msg.To` mask se for número de usuário real)
- Error wrapping com contexto
- No try/catch defensivo — captura só se tem ação útil (log + retornar error wrapped)

---

## 4. Webhook handler (se aplicável)

Se provider envia webhooks (Stripe, Mux, Livekit, Twilio inbound):

Arquivo: `backend/internal/modules/<bc>/infrastructure/http/webhooks/<provider>_webhook.go`

```go
func (h *Handler) HandleProviderWebhook(c *gin.Context) {
    // 1. Ler raw body (antes de parse — HMAC verify first)
    rawBody, err := io.ReadAll(c.Request.Body)
    if err != nil {
        c.AbortWithStatus(400)
        return
    }

    // 2. HMAC verify ANTES de parse (I-053, DR-020)
    signature := c.GetHeader("X-Provider-Signature")
    if !hmac.Verify(rawBody, signature, h.webhookSecret) {
        c.AbortWithStatus(403)
        return
    }

    // 3. Parse
    event, err := parse(rawBody)
    if err != nil {
        c.AbortWithStatus(400)
        return
    }

    // 4. Idempotency check (event.id já processado?)
    if processed, _ := h.repo.WebhookProcessed(c, event.ID); processed {
        c.Status(200) // ack; não reprocessa
        return
    }

    // 5. Execute use case
    if err := h.useCase.Execute(c, event); err != nil {
        c.AbortWithStatus(500) // provider retry
        return
    }

    // 6. Mark processed
    _ = h.repo.MarkWebhookProcessed(c, event.ID)

    c.Status(200)
}
```

**Regras duras** (I-053, I-010):
- HMAC **antes** de parse — prevenir timing attacks + DoS
- Idempotency (event.id unique) — prevenir replay + duplicate process
- Retry-friendly: retornar 5xx se use case falha (provider retry); 200 se processado
- DLQ (Dead Letter Queue) pra events que falham repetidamente — job reprocessa

Runbook: [[../runbooks/webhook-reprocessing]].

---

## 5. Wire-up

Arquivo: `backend/cmd/api/main.go` (DI explícito manual)

```go
func main() {
    cfg := loadConfig()
    logger := initLogger()

    // Provider
    smsProvider := twilio.NewProvider(
        cfg.TwilioAccountSID,
        cfg.TwilioAuthToken,
        cfg.TwilioFromNumber,
        logger,
    )

    // Use case consome via interface
    sendCodeUC := auth.NewSendCodeUseCase(repo, smsProvider, logger)

    // Handler usa use case
    // ...
}
```

---

## 6. Config + secrets

- Config struct em `internal/shared/config/config.go` com campos novos
- `Config.Validate()` valida não-vazio em staging/prod
- Secret em Railway env (`TWILIO_AUTH_TOKEN`)
- `.env.example` atualizado (sem valor real)
- `.gitignore` protege `.env`

---

## 7. Testes

### Unit (adapter)

Mock HTTP client (testcontainers-go se provider oferece emulator):

```go
func TestTwilio_Send(t *testing.T) {
    // mock HTTP com httptest.NewServer
    // verify request body + auth header
    // retorna fixture response
}
```

### Integration (com emulator se existe)

- Mux: `muxinc/mux-node` tem emulator? Usar dev mode
- Stripe: Stripe CLI `stripe listen` + fixtures
- Livekit: docker compose local

### E2E (staging)

Smoke test mínimo:
- Enviar SMS pra número teste
- Verificar log estruturado OK
- Verificar audit trail entry

---

## 8. Observabilidade

- OTel span `sms.send` com attrs:
  - `provider`: twilio
  - `to.masked`: `+55119****1234`
  - `trace_id`: request ID
- Sentry: errors auto-capturados
- Grafana dashboard: SMS send rate + error rate + p95

---

## 9. Rate limits + circuit breaker

Se provider tem rate limits (Stripe 100 req/s, Twilio 1 msg/s):

- Implementar client-side rate limit (`golang.org/x/time/rate`)
- Circuit breaker (sony/gobreaker) se provider instável
- Retry com backoff exponencial (ex: 1s, 2s, 4s, 8s)
- DLQ se falha permanente

---

## 10. Docs

- [ ] Atualizar [[../../02-architecture/c4-components]] (novo provider box)
- [ ] Atualizar [[../../02-architecture/patterns]] se pattern novo
- [ ] Atualizar [[../README#stack]] se stack muda
- [ ] ADR em [[../../02-architecture/adr/_moc]]
- [ ] Runbook se operacionalmente relevante em [[../runbooks/_moc]]
- [ ] [[../changelog]] no próximo release

---

## Exemplos canônicos do legado

Deve seguir o padrão de:
- **Stripe** (`internal/shared/providers/payment/stripe/`) — webhook idempotent + HMAC
- **Mux** (`internal/shared/providers/video/mux/`) — presigned URL + webhooks
- **Livekit** (`internal/shared/providers/live/livekit/`) — SFU + saga compensation
- **Resend** (`internal/shared/providers/email/resend/`) — templates versionados
- **MinIO** (`internal/shared/providers/storage/minio/`) — S3-compatible PresignedURLs
- **Zitadel** (`internal/shared/providers/auth/zitadel/`) — OIDC introspection + cache Redis

---

## Anti-padrões

- ❌ Importar SDK externo fora de `infrastructure/providers/<provider>/`
- ❌ Parse body antes de HMAC verify em webhook
- ❌ Sem idempotency em webhook (double-process risk)
- ❌ Provider sem rate limit / circuit breaker (cascade failure)
- ❌ Secrets em código ou commitados
- ❌ Log body do webhook com PII sem masking
- ❌ Abstração prematura — se é único provider + não troca mesmo, pode pular interface (mas ~documentar~)

---

## Links

- [[_moc]]
- [[criar-novo-bc]]
- [[../../02-architecture/patterns]]
- [[../../02-architecture/clean-arch-mapping]]
- [[../../06-execution/playbooks/research-loop]]
- [[../runbooks/webhook-reprocessing]]
- [[../../90-ingested/_consolidado-providers-fluxos]] — catálogo 22 crons + 27 webhooks herdado
