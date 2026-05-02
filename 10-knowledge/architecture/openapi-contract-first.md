---
title: OpenAPI Contract-First
type: concept
tags: [architecture, openapi, contract-first, api-design]
source: https://spec.openapis.org/oas/v3.1.0
created: 2026-04-24
updated: 2026-04-24
aliases: [OpenAPI 3.1, API contract-first, spec-first]
---

# OpenAPI Contract-First

Abordagem de design onde o **contrato da API é escrito antes da implementação**. O arquivo OpenAPI 3.1 é a fonte única de verdade — handlers, clients, testes e documentação derivam dele, não o contrário.

## Por que contrato-first (vs code-first)

**Code-first** (gerar spec a partir de anotações no handler) amarra o contrato à implementação: qualquer refatoração vaza pro consumidor. O contrato vira efeito colateral do código.

**Contract-first** inverte: o contrato é negociado **antes** de alguém codar. Benefícios concretos:

- **Paralelismo**: front e back trabalham em cima do mesmo contrato, sem bloqueio
- **Mock server** (Prism, Microcks) gera stubs pro front antes do back existir
- **Validação em CI** — request/response checados contra schema, quebra falha PR
- **Breaking change é explícito** — diff no YAML é auditável
- **SDK generation** pra múltiplos consumidores (web, mobile, parceiros)

Code-first ainda serve pra proto interno de um time só, onde consumidor e provedor são a mesma pessoa.

## Estrutura do arquivo

```yaml
openapi: 3.1.0
info:
  title: Payments API
  version: 1.4.0
  description: API de pagamentos e reconciliação
servers:
  - url: https://api.example.com/v1
paths:
  /charges:
    post:
      operationId: createCharge
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateChargeRequest'
      responses:
        '201':
          description: Charge criado
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Charge'
        '4XX':
          $ref: '#/components/responses/ClientError'
components:
  schemas:
    Charge:
      type: object
      required: [id, amount, currency, status]
      properties:
        id: { type: string, format: uuid }
        amount: { type: integer, minimum: 1 }
        currency: { type: string, pattern: '^[A-Z]{3}$' }
        status:
          type: string
          enum: [pending, succeeded, failed]
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

Chaves canônicas: `openapi` (versão da spec), `info` (metadata), `servers` (base URLs por ambiente), `paths` (endpoints), `components` (reutilizáveis), `security` (default auth).

## Versionamento

Regra: **path-based versioning** (`/api/v1/...`). Query string e header pra versão são frágeis em cache e difíceis de documentar.

- Breaking change → nova major (`/v2`). Deprecar `/v1` com header `Deprecation: true` + `Sunset: <rfc1123-date>` no mínimo **6 meses antes** do desligamento
- Non-breaking (novo endpoint, novo campo opcional) → minor bump do `info.version`, mesmo path
- Patch → correção de doc/typo, sem mudança contratual

## Componentes reutilizáveis

Centralizar em `components/` reduz divergência e facilita diff:

- `schemas/` — DTOs e value objects. Um por arquivo se monorepo, agrupado se simples
- `parameters/` — query/header/path params usados em 2+ endpoints
- `responses/` — envelopes de erro (`400 BadRequest`, `429 RateLimited`) padronizados
- `securitySchemes/` — `bearerAuth`, `oauth2`, `apiKey`
- `examples/` — payloads de exemplo para docs e testes de contrato

Usar `$ref` agressivamente. Schema inline só se usado uma vez.

## Geração de client + server stubs

Toolchain recomendada por linguagem:

| Stack | Ferramenta | Nota |
|---|---|---|
| Go | `ogen` ou `oapi-codegen` | `ogen` gera código mais idiomático; `oapi-codegen` é mais maduro |
| TypeScript | `openapi-typescript` (types) + `openapi-fetch` (client) | Leve, type-safe, zero runtime extra |
| Rust | `progenitor` | Gera client tipado para APIs `application/json` |
| Multi-lang | `openapi-generator` | Suporte amplo, output verbose |

Server stubs viram **interface** que o handler precisa implementar. Compilador garante que toda operação do spec tem handler. Nova operação no YAML → build quebra até implementar.

## Linting

**Spectral** (Stoplight) é o de facto. Regras padrão (`spectral:oas`) cobrem operationId único, tags obrigatórias, response schemas, descrições não-vazias. Custom rulesets pra convenções internas (ex: "todo endpoint precisa de `x-rate-limit`").

**Redocly CLI** complementa com checks de consistência e bundling de multi-file spec.

Ambos rodam em CI como gate bloqueante.

## Contract testing

Duas escolas:

- **Pact** (consumer-driven) — consumidor declara expectativas, broker publica, provider verifica. Bom pra microserviços internos onde o contrato evolui junto
- **OpenAPI + Dredd / Schemathesis** (provider-driven) — spec é a verdade, ferramenta dispara requests e valida response contra schema. Schemathesis faz property-based testing em cima do spec (fuzzing controlado)

Em API pública/SDK-oriented: OpenAPI+Schemathesis. Em mesh interno: Pact.

## Quando usar

- APIs **públicas** ou expostas a parceiros
- **Múltiplos consumidores** (web + mobile + integrações)
- Projeto que **gera SDK**
- Times separados pra front e back (paralelismo)
- Compliance/audit precisa de contrato versionado

## Quando ignorar

- Proto interno do **mesmo time**, mesmo repo, único consumidor
- Prototipação descartável (dias, não meses)
- gRPC/Protobuf já cobrem o caso (aí o `.proto` é o contrato-first equivalente)

## Links

- [[_moc]]
- [[../_moc]]
- [[../methodology/sdd-workflow]]
