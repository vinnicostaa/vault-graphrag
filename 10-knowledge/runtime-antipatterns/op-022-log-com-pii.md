---
title: OP-022 — Log com PII / payload sensível sem sanitização
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - observability
  - security
  - pii
  - lgpd
id: OP-022
severity: Critical
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - PII in logs
  - Sensitive payload logging
---

# OP-022 — Log com PII / payload sensível sem sanitização

**Severidade**: Critical (vazamento de dados, LGPD/GDPR)
**Categoria**: Observabilidade / Segurança

## Sintoma

`logger.info({ webhook: event.data })`, `console.log(user)`, `debug(request.body)` — logs contêm cartão de crédito, CPF/CNPJ, tokens, senha em claro, email, chaves API. Sistema de logs (Datadog, ELK, Cloudwatch, Splunk) agora tem dataset regulatório em escopo de compliance/retenção.

## Por que é problema

- **Vazamento**: se log vaza (breach, acesso indevido, export), PII vaza junto.
- **Compliance**: LGPD art. 46-49 exige proteção adequada; logs PII em plain text falham.
- **Retenção indevida**: logs tipicamente guardados 30-90d — além do necessário para PII.
- **Terceiros**: SaaS de logs pode estar em outro país/jurisdição.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — payload cru no log
logger.Info("webhook received",
    "event_type", event.Type,
    "payload", event.Data,   // contém card.last4, CPF, email, etc.
)
```

## Fix preferido — função `sanitize` com allowlist

```go
// Lista allow — só campos seguros logáveis
var safeFields = map[string]bool{
    "event_type": true, "event_id": true, "source": true,
    "amount":     true, "currency": true, "object_type": true,
    // email, cpf, card.*, token — NÃO estão aqui
}

func sanitize(payload map[string]any) map[string]any {
    out := make(map[string]any)
    for k, v := range payload {
        if safeFields[k] {
            out[k] = v
        } else {
            out[k] = "[redacted]"
        }
    }
    return out
}

logger.Info("webhook received",
    "event_type", event.Type,
    "payload", sanitize(event.Data),
)
```

**Regra**: usar **allow-list**, não block-list. Novo campo adicionado ao payload → `redacted` por padrão → sem vazamento acidental.

## Casos especiais

- **CPF/CNPJ**: mascarar preservando algarismos úteis para suporte (`***.***.123-45`).
- **Email**: mascarar em debug (`j***@example.com`); inteiro só em audit trail autorizado com retenção diferenciada.
- **Tokens**: logar **prefixo** (`sk_test_...`, `eyJhbGc...`) — suficiente para correlação, insuficiente pra uso.
- **PII em mensagens de erro retornadas ao cliente**: **nunca**; frontend tem estado local suficiente.

## Alternativa

- **Structured logging middleware** que aplica sanitização automaticamente por campo (ex.: zap com `zap.Field` customizado, pino com redact).
- **Tagged loggers** que marcam contexto sensível e força sanitização.
- **Lint pre-commit**: detecta `logger.*(<object raw>)` e falha.

## Nunca

- Logar senha plain text, mesmo acidentalmente (esqueceu de remover `console.log(req.body)`).
- Logar chaves privadas, secret keys, JWT completos.
- Retornar detalhes internos (stack trace, SQL error, campo de erro) para o cliente.

## Relações

- **Patterns**: Structured logging, [[../patterns/decorator]] (sanitize decorator)
- **OPs relacionados**: [[op-021-admin-sem-audit]] (mesmo cluster observabilidade), [[op-020-erro-engolido]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-022, §3.20 (Segurança)
- LGPD — [ANPD Guia sobre tratamento de dados em log](https://www.gov.br/anpd/pt-br) (consultada 2026-04-24)
- Pino — [Redaction](https://getpino.io/#/docs/redaction) (consultada 2026-04-24)
- OWASP — [Logging cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../observability/structured-logging]]
- [[../security/security-principles]]
- [[op-021-admin-sem-audit]]
