[FFV Academy](../index.html)/API Design & Contratos

🔌

Blog

# API Design & Contratos

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

9artigos

485XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/rest-maduro/index.html)

01

## 🏛️ REST maduro: Richardson levels, idempotência e HATEOAS (raramente)

Os 4 níveis do Richardson Maturity Model, verbos HTTP corretamente (idempotência de PUT/DELETE vs segurança de GET), status codes que comunicam (201 vs 204 vs 200), HATEOAS na prática (raramente vale a pena). O REST sem jargão de livro de 2010.

⏱ 12 min·+50 XP

→[](../aprenda/versionamento-sem-dor/index.html)

02

## 🔀 Versionamento sem dor: URL, header, sunset e estratégia de migração

URL versioning (/v1/) vs header (Accept: application/vnd.api+json;version=1) vs parameter. Quando cada um cabe. Headers Sunset e Deprecation para comunicar descontinuação. Backward compat, feature flags, e o padrão "nunca remover endpoint — só deprecar".

⏱ 10 min·+45 XP

→[](../aprenda/graphql-quando-faz-sentido/index.html)

03

## ◈ GraphQL quando faz sentido: N+1, DataLoader e federation

GraphQL resolve problemas reais (over-fetching, agregação de APIs) mas cria outros (N+1, caching complexo). DataLoader pra batchar resolvers, persisted queries pra cache. Apollo Federation vs schema stitching. Quando REST + BFF é resposta melhor.

⏱ 14 min·+60 XP

→[](../aprenda/grpc-e-protobuf/index.html)

04

## ⚡ gRPC + Protobuf: RPC tipado e streaming bidirecional

Protobuf como IDL compacto (binário, schema evolution, field tags), gRPC com HTTP/2 multiplexing, 4 modos (unary, server streaming, client streaming, bidi). Quando gRPC bate REST (interno de microserviços de baixa latência) e quando não (APIs públicas browser-facing).

⏱ 12 min·+55 XP

→[](../aprenda/openapi-como-contrato-vivo/index.html)

05

## 📜 OpenAPI como contrato vivo: codegen, mock server e contract testing

Escrever OpenAPI primeiro (spec-driven dev), gerar client + server skeleton + docs automáticos, mock server (Prism/Wiremock) pra desenvolver frontend sem backend pronto, contract testing (Pact) entre times. Por que YAML à mão é melhor que gerar do código.

⏱ 11 min·+50 XP

→[](../aprenda/paginacao-filtros-ordenacao/index.html)

06

## 📑 Paginação, filtros e ordenação profissionais

Offset pagination (LIMIT/OFFSET) vs cursor-based (opaque token) — por que cursor vence em datasets grandes. Keyset pagination com index composto. Filter DSL sem explodir (RSQL, Google AIP filter syntax), ordenação estável (tiebreaker por id).

⏱ 10 min·+45 XP

→[](../aprenda/idempotency-keys-e-webhooks/index.html)

07

## 🔁 Idempotency keys e webhooks: exactly-once na prática

Idempotency-Key header (modelo Stripe) pra POST seguro contra retry, store com TTL e response cacheada. Webhooks com assinatura HMAC, timestamp pra evitar replay, retry exponencial, DLQ quando consumer falha. Design de payload mínimo + event fetching.

⏱ 12 min·+55 XP

→[](../aprenda/rate-limiting-e-quotas-em-api/index.html)

08

## 🚦 Rate limiting e quotas: token bucket, leaky bucket e fairness

Algoritmos: fixed window (problemático), sliding window, token bucket (mais usado), leaky bucket. Headers padrão (RateLimit-Limit, Remaining, Reset), 429 com Retry-After. Quota diária vs burst limit, por-tenant, distribuído com Redis atomicamente.

⏱ 11 min·+50 XP

→[](../aprenda/capstone-api-rest-produto-completo/index.html)

09

## 🏁 Capstone: API REST completa de um produto real

Projeto consolidando a trilha: API de gestão de tarefas com OpenAPI spec-driven, versionamento /v1, auth com JWT, paginação cursor, idempotency em POST, webhook em mudanças, rate limit por tenant, testes de contrato com Pact. Deploy + docs.

⏱ 16 min·+75 XP

→

[← Voltar à home](../index.html)
