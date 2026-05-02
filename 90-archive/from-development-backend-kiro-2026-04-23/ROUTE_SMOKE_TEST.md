# Route Smoke Test — 2026-04-22

Teste manual de rotas contra o backend rodando local (`go run ./cmd/api` em 8000).
**Atenção**: build ativa é do dia 21/04 (PID 1450851, `/tmp/go-build*/b001/exe/api`). Sem restart, as rotas novas do Sprint 9 (Mux webhook público, content routes regravadas) não vão aparecer como o código-fonte atual dita. Este snapshot documenta **o que o processo já deployado expõe**.

## Rotas confirmadas ✅

| Método | Rota                     | HTTP | Observação                                  |
|--------|--------------------------|------|---------------------------------------------|
| GET    | `/health`                | 200  | Retorna `{status:ok,time,request_id}`       |
| GET    | `/health/ready`          | 200  | Readiness check                             |
| GET    | `/auth/login`            | 302  | PKCE redirect para Zitadel                  |
| GET    | `/auth/logout`           | 302  | Redirect para Zitadel end_session           |
| GET    | `/auth/callback`         | 401  | Sem `code`/`code_verifier` cookie → erro esperado |
| POST   | `/webhooks/stripe`       | 400  | `"invalid signature"` — HMAC HMAC-SHA256 validado corretamente |

## Rotas com 404 (não existem na build atual)

| Método | Rota                        | Observação                                                   |
|--------|-----------------------------|--------------------------------------------------------------|
| GET    | `/metrics`                  | Prometheus handler — pode exigir `telemetryProviders.PrometheusEnabled` |
| GET    | `/docs`                     | Scalar UI — path pode ter mudado                             |
| GET    | `/docs/openapi.yaml`        |                                                              |
| GET    | `/webhooks/mux`             | Esperado — GET bloqueado; POST deveria funcionar pós Sprint 9 |
| POST   | `/webhooks/mux`             | 404 — rota do A-022 ainda não está nesta build               |
| GET    | `/api/v1/plans`             | Authenticate pode estar retornando 404 em vez de 401 (não-standard) |
| GET    | `/api/v1/subscriptions/me`  | ditto                                                        |
| GET    | `/api/v1/condominiums`      | ditto                                                        |

## Próximos passos para teste end-to-end completo

Precisa:
1. `pkill -f "exe/api" && go run ./cmd/api` — restart com build atual.
2. Em 2º terminal: `stripe listen --forward-to localhost:8000/webhooks/stripe`
   — copiar `whsec_...` para `.env` (`STRIPE_WEBHOOK_SECRET`).
3. `stripe trigger customer.subscription.created` — força webhook → backend cria subscription local.
4. Verificar em DB: `psql -h localhost -U postgres -d mastersindico -c "SELECT * FROM billing.subscriptions"`.

## Observação sobre 404 em /api/v1/*

Rotas retornando 404 sem mensagem JSON indicam que não estão sendo registradas. Hipóteses:
- Gin strict routing + prefixo mal construído (e.g., module registrando "`/plans`" em vez de "`/api/v1/plans`");
- `RegisterModules` recebendo grupo errado;
- Require restart da API com build atual.

Não houve output `404 not found` com request_id header — é o 404 default do httprouter Gin sem passar pelo ErrorHandler. Quando pass, mensagem viria em `contracts.Context.JSON`.

## Gates verificados manualmente

- ✅ Health endpoint responde < 10ms.
- ✅ Stripe webhook valida HMAC (rejeitou body vazio com 400 "invalid signature").
- ✅ `/auth/login` redireciona para Zitadel via HTTP 302.

## O que NÃO foi testado (precisa user)

- Fluxo end-to-end Zitadel: `/auth/login` → Zitadel UI → `/auth/callback` com code → sessão criada → requests autenticados.
- Stripe checkout saga: POST `/subscriptions/checkout` → `customer.subscription.created` webhook → DB row criada.
- Mux webhook: `video.asset.ready` → video table updated.
- Livekit: Assembly start live → token gerado → room criada.

Tudo isso é **integração externa** que exige credenciais reais (Zitadel machine user, Stripe test keys ativos, Mux dev account, Livekit dev). O ambiente local Docker já tem Zitadel v4 rodando em `:8080` via Traefik, então o login PKCE deve funcionar end-to-end quando o backend for restartado.
