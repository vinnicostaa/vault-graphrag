---
title: Global Requirements — Master Síndico v2
type: requirement
tags: [requirements, global, cross-cutting, master-sindico, v2, ears]
source: legacy 04-requirements/global + specs/requirements/cross-domain + STEERING v2
created: 2026-04-23
updated: 2026-04-23
---

# Global Requirements — Master Síndico v2

Requisitos **transversais** que atravessam todos os bounded contexts. Formato **EARS**: "Quando/Enquanto `<trigger/state>`, o sistema deve `<comportamento>`".

Escopo: GR-001 a GR-035. Prioridade MoSCoW (M1 = Must, M2 = Should, M3+ = Could, backlog = Won't).

> Derivado do legado vivo em `../90-ingested/from-vault-50-projects-master-sindico/04-requirements/global.md` + `specs/requirements/cross-domain.md` + [[STEERING]] §15-24 + [[00-product/portfolio-de-produtos]].

---

## Legenda

- **M** = Must (M1 — MVP contratual 2026-05-08)
- **S** = Should (M2 — Connect Me + Conteúdo 2026-06-20)
- **C** = Could (M3+ — Assembly + LMS + polish)
- **W** = Won't (backlog pós-M3)

---

## GR-001 — Autenticação OIDC com PKCE

**Prioridade**: M  
**Descrição**: Toda request autenticada **deve** ser validada via Zitadel OIDC Authorization Code Flow + PKCE.

**EARS**: *Quando* um cliente emite request contra rota sob `/api/v1/*` ou WebSocket autenticado, *o sistema deve* exigir um access token válido (JWT via introspection Zitadel) propagado via cookie `access_token` (httpOnly, Secure, SameSite=Lax, domínio `.mastersindico.com.br`) ou header `Authorization: Bearer <token>` (RFC 6750).

**Dependências**: Zitadel tenant provisionado; cookie `gorilla/securecookie` pra `code_verifier`.

**Critério de aceite**:
- Request sem token → `401 unauthorized`.
- Token expirado → middleware tenta refresh (ou 401 se refresh falha).
- Cache Redis `ms:auth:{zitadelUserID}` TTL 5 min (evitar introspection em hot path).
- Introspection trouxe `role=""` → `403 NO_ROLE_ASSIGNED` **antes** do UPSERT local.

**Fonte**: legado `global.md` GR-001, `identity.md` Req 1, [[STEERING]] §15, §18.

---

## GR-002 — Autorização ABAC (matriz ordenada)

**Prioridade**: M  
**Descrição**: Autorização por atributos com avaliação em ordem fixa antes do handler.

**EARS**: *Quando* middleware Authenticate libera a request, *o sistema deve* avaliar a policy ABAC na ordem: (1) `admin_bypass` → (2) `role_check` → (3) `action_check` → (4) `tenant_check` → (5) `scope_check`. Se qualquer passo negar, responde `403 forbidden` com `code` canônico (`NO_ROLE_ASSIGNED`, `ACTION_NOT_ALLOWED`, `TENANT_MISMATCH`, `OUT_OF_SCOPE`).

**Dependências**: GR-001, cache Redis, webhook Stripe/Zitadel pra invalidação.

**Critério de aceite**:
- `RequirePermission(action, resource)` intercepta todas as rotas protegidas.
- Cache `ms:abac:{user_id}:{tenant_id}` TTL 5min; invalidação via webhook `customer.subscription.*`.
- Cross-tenant access negado por default.

**Fonte**: legado `cross-domain.md` Req 2, `identity.md` Req 2, [[00-product/personas]], matriz em [[matrix-functional]].

---

## GR-003 — Multi-tenant isolation

**Prioridade**: M  
**Descrição**: Nenhuma query pode acessar dados de outro tenant sem grant explícito.

**EARS**: *Quando* uma query atinge recurso escopado a condomínio/empresa, *o sistema deve* incluir `WHERE tenant_id = $current_tenant_id` no SQL. *Quando* lista-se recursos cross-tenant, *o sistema deve* aplicar filtro ABAC antes do retorno.

**Critério de aceite**:
- 100+ testes de cross-tenant isolation em ABAC matrix.
- Property-based test: qualquer user não pode obter `200` em recurso de `tenant != membership`.
- SQLc `@tenant_id` obrigatório em queries escopadas.

**Fonte**: legado `global.md` GR-003, [[STEERING]] §11.

---

## GR-004 — 1-device policy por conta

**Prioridade**: M  
**Descrição**: User tem no máximo 1 sessão ativa (configurável via `MAX_ACTIVE_SESSIONS`, default 1).

**EARS**: *Quando* user faz login em novo device (`device_fingerprint = IP + UA + canvas hash` não existente), *o sistema deve* (a) invalidar a sessão anterior via `DELETE sessions WHERE user_id = $1 AND id != $new`, (b) gravar audit trail `identity.device.switched`, (c) emitir evento `session.invalidated` para invalidar cache.

**Critério de aceite**:
- Login subsequente derruba token/sessão antigos (próxima request do device antigo → `401 SESSION_REVOKED`).
- Audit trail registra IP, UA, timestamp de ambas as sessões.
- PBT: nunca há 2 `sessions.revoked_at IS NULL` pro mesmo `user_id`.

**Fonte**: legado `identity.md` Req 1 rule 8, [[STEERING]] §17, matriz legada item final ("Limitar o uso do app a 1 dispositivo").

---

## GR-005 — Request ID propagation

**Prioridade**: M  
**Descrição**: Toda request **deve** carregar `X-Request-ID` propagado em logs, traces e response.

**EARS**: *Quando* uma request chega sem `X-Request-ID`, *o sistema deve* gerar UUIDv7 e injetar no context. *Quando* a response retorna, *o sistema deve* ecoar `X-Request-ID` no header. *Quando* logs são emitidos no ciclo da request, *o sistema deve* incluir `request_id` estruturado.

**Critério de aceite**:
- Middleware `RequestID` antes de `Logger`.
- Trace OpenTelemetry usa `request_id` como `trace_id` quando compatível.
- Webhooks (Stripe/Mux/Livekit) têm `request_id` derivado de `event.id`.

**Fonte**: legado `global.md` GR-005, [[STEERING]] §11.

---

## GR-006 — Rate limiting estruturado

**Prioridade**: M  
**Descrição**: Rate limits por rota e por tipo de identidade.

**EARS**: *Quando* múltiplas requests são emitidas por `(ip, user_id)` dentro da janela, *o sistema deve* aplicar limites:
- `/auth/*` → 20 rpm, burst 5.
- `/api/v1/*` → 60 rpm, burst 10.
- `/webhooks/*` → sem rate limit interno (HMAC + idempotência cobrem).
*Quando* limite estourado → `429 rate_limited` com header `Retry-After`.

**Critério de aceite**:
- Token bucket via `sync.Map` + cleanup goroutine (A-006, A-032 do legado).
- Testes: 21 requests em `/auth/login` em 1 min → 429 na 21ª.
- Brute-force: após 3 logins falhos em 5 min → CAPTCHA reCAPTCHA v3 exigido (Req 102 legado).

**Fonte**: legado `global.md` GR-006, [[STEERING]] §19.

---

## GR-007 — PII nunca em logs

**Prioridade**: M  
**Descrição**: Dados pessoais sensíveis proibidos em logs estruturados ou em texto livre.

**EARS**: *Quando* código emite log estruturado ou texto, *o sistema deve* mascarar CPF (`***.***.***-XX`), CNPJ (`**.***.***/XXXX-XX`), email (`u***@dom***.com`), telefone, access token, password, PAN, via helper `Masked()`. *Quando* CI roda, *o sistema deve* bloquear PR com grep regex detectando PII cru em `log.*`, `fmt.*` e `zerolog.*`.

**Critério de aceite**:
- Grep regex no pipeline CI; 0 falsos-positivos no baseline.
- Audit trail usa `user_id` e nunca `email` em tail logs.
- Helper `Masked()` em `pkg/security/masking`.

**Fonte**: legado `global.md` GR-007, [[STEERING]] §20, A-001..A-008 legado.

---

## GR-008 — Webhook HMAC antes de parse

**Prioridade**: M  
**Descrição**: Webhooks de Stripe, Mux, Livekit, Zitadel (futuro) **devem** ter signature verificada antes de `json.Unmarshal`.

**EARS**: *Quando* request chega em `/webhooks/<provider>`, *o sistema deve* (1) ler `raw body` como bytes, (2) verificar HMAC usando secret do provider, (3) só então fazer `Unmarshal`. *Quando* signature inválida → `400 invalid_signature` sem vazar detalhes.

**Critério de aceite**:
- Testes: payload sem signature → 400; signature incorreta → 400; signature expirada → 400.
- `body_reader := io.TeeReader(r.Body, &hashBuf)` pra evitar double-consume.

**Fonte**: legado `billing.md` Req 4 nota, [[STEERING]] §21.

---

## GR-009 — Idempotência de webhook

**Prioridade**: M  
**Descrição**: `event.id` do provider é único; reentrega não gera efeito colateral duplicado.

**EARS**: *Quando* webhook chega com `event.id` já processado, *o sistema deve* retornar `200 OK` sem reprocessar. *Quando* processamento parcial falhou antes, *o sistema deve* retomar de onde parou (ledger `webhook_events` com status).

**Critério de aceite**:
- Tabela `webhook_events` (`provider`, `event_id`, `status`, `payload_hash`, `received_at`, `processed_at`).
- Processamento é transacional: ledger UPSERT + handler na mesma UoW.
- Testes: reentrega 3x → 1 execução efetiva.

**Fonte**: legado `billing.md` Req 4, [[STEERING]] §9.

---

## GR-010 — Internacionalização (i18n) pt-BR primário

**Prioridade**: M  
**Descrição**: pt-BR é língua primária; EN é secundário no frontend; identifiers sempre em EN.

**EARS**: *Quando* usuário interage com frontend (web/mobile), *o sistema deve* exibir textos em pt-BR. *Quando* log é emitido no backend, *o sistema deve* usar EN técnico. *Quando* erro JSON é retornado ao cliente, *o sistema deve* usar pt-BR em `message` user-facing e EN em `code`.

**Critério de aceite**:
- Frontend usa chave i18n com fallback pt-BR.
- OpenAPI descriptions em EN; `summary` em EN; mensagens de erro user-facing em pt-BR.
- M3+: estrutura preparada pra EN (keys extraídos, tradução default `{}`).

**Fonte**: legado `global.md` GR-010, [[STEERING]] §3.

---

## GR-011 — Observability (OTel + Grafana + Sentry)

**Prioridade**: M  
**Descrição**: Logs estruturados + traces + metrics + error tracking.

**EARS**: *Quando* request atravessa a API, *o sistema deve* emitir (1) log estruturado com `request_id`, `user_id`, `tenant_id`, `route`, `status`, `duration_ms`; (2) OpenTelemetry span por use case e por query SQL; (3) counters de domain events + histogram de latência por endpoint; (4) errors não-operacionais em Sentry com `request_id` como tag.

**Critério de aceite**:
- Grafana dashboards: latência P50/P95/P99 por rota, taxa de erro, throughput.
- Sentry: backend + frontend + mobile conectados ao mesmo projeto ou projetos agrupados.
- OTel collector gRPC com batcher.
- PII nunca enviado a Sentry (scrubber regex).

**Fonte**: legado `global.md` GR-011, sprint 7 legado.

---

## GR-012 — LGPD compliance (high-level)

**Prioridade**: M  
**Descrição**: Plataforma cumpre LGPD end-to-end. Detalhes específicos em [[compliance-lgpd]].

**EARS**: *Quando* usuário cria conta, *o sistema deve* coletar consentimento explícito versionado. *Quando* usuário solicita exclusão, *o sistema deve* honrar SLA de 15 dias. *Quando* breach é detectado, *o sistema deve* notificar ANPD em até 72h.

**Critério de aceite**:
- Ver GR-023, GR-024, GR-025, [[compliance-lgpd]].

**Fonte**: legado `global.md` GR-012, [[STEERING]] §24.

---

## GR-013 — Acessibilidade WCAG 2.1 AA

**Prioridade**: S  
**Descrição**: Frontend web e mobile devem atingir WCAG 2.1 AA no mínimo.

**EARS**: *Quando* designer/frontend adiciona componente, *o sistema deve* validar via axe-core/lighthouse que: contraste ≥ 4.5:1 (texto normal), keyboard navigation completa, labels ARIA em inputs, foco visível, escaláveis a 200% sem perda.

**Critério de aceite**:
- Lighthouse a11y score ≥ 90 em páginas críticas (login, timeline, assembleia, checkout).
- Pipeline CI: axe-core via Playwright em rotas chave.

**Fonte**: legado `global.md` GR-013.

---

## GR-014 — Versionamento API `/api/v1`

**Prioridade**: M  
**Descrição**: Contrato canônico é `/api/v1`. Breaking changes requerem `/api/v2`.

**EARS**: *Quando* time propõe mudança breaking em endpoint existente, *o sistema deve* abrir `/api/v2/<endpoint>` em paralelo, manter `/api/v1` por 6 meses com header `Deprecation: true` e `Sunset: <date>`. *Quando* mudança non-breaking (campo opcional novo), *o sistema pode* adicionar direto em v1.

**Critério de aceite**:
- OpenAPI 3.1 spec versionado (`openapi.v1.yaml`, `openapi.v2.yaml`).
- Test contract: suite que valida v1 continua passando durante janela de deprecation.

**Fonte**: legado `global.md` GR-014, [[STEERING]] §13.

---

## GR-015 — Error envelope JSON canônico

**Prioridade**: M  
**Descrição**: Toda resposta de erro segue envelope consistente.

**EARS**: *Quando* erro ocorre em qualquer rota, *o sistema deve* responder:

```json
{
  "code": "validation_error",
  "message": "starts_at obrigatório",
  "fields": [{"path": "starts_at", "rule": "required"}],
  "request_id": "018f..."
}
```

Códigos canônicos: `validation_error` (422), `not_found` (404), `conflict` (409), `forbidden` (403), `unauthorized` (401), `payment_required` (402), `quota_exhausted` (403), `rate_limited` (429), `internal_error` (500), `service_unavailable` (503).

**Critério de aceite**:
- Middleware `ErrorHandler` sempre formata errors neste envelope.
- Nenhum handler escreve JSON de erro manualmente.
- Docs OpenAPI referenciam schema `ErrorResponse`.

**Fonte**: legado `global.md` GR-015.

---

## GR-016 — Paginação cursor-based

**Prioridade**: M  
**Descrição**: Listagens suportam `?limit` + `?cursor` opaco, não offset.

**EARS**: *Quando* cliente solicita listagem, *o sistema deve* aceitar `?limit=<N>` (default 20, max 100) e `?cursor=<opaque>`. *Quando* há mais resultados, *o sistema deve* retornar `{items: [...], next_cursor: "<opaque>"}`. *Quando* fim da lista → `next_cursor: null`.

**Critério de aceite**:
- Cursor é base64 de `{created_at, id}` (tie-breaker).
- PBT: varredura total via next_cursor retorna cada item 1x, sem lacunas.
- Listas ordenadas por `created_at DESC, id DESC` por default.

**Fonte**: legado `global.md` GR-016.

---

## GR-017 — Timezone UTC storage, `America/Sao_Paulo` display

**Prioridade**: M  
**Descrição**: Backend armazena UTC; frontend exibe em SP; API retorna ISO 8601 com offset.

**EARS**: *Quando* dado temporal é persistido, *o sistema deve* salvar em `TIMESTAMPTZ` convertido pra UTC. *Quando* API retorna campo temporal, *o sistema deve* serializar como ISO 8601 com `Z` ou offset. *Quando* frontend exibe, *o sistema deve* converter pra `America/Sao_Paulo` (ou timezone do usuário se feature flag M4+).

**Critério de aceite**:
- SQL CHECK: `at_time_zone('UTC') = timestamp_column` em constraint apropriada.
- Tests: entrada `2026-05-08T10:00:00-03:00` → armazenado `2026-05-08T13:00:00Z`.

**Fonte**: legado `global.md` GR-017.

---

## GR-018 — Upload size limits

**Prioridade**: M  
**Descrição**: Limites de body/upload por rota.

**EARS**: *Quando* client faz POST, *o sistema deve* aplicar limites:
- Body request geral: 10 MB.
- Avatar (morador/síndico/parceiro): 5 MB.
- Logo (empresa/parceiro): 5 MB.
- Galeria parceiro (cada foto): 5 MB.
- Foto execução (registro de serviço): 10 MB.
- PDF edital/ata/dossiê: 20 MB.
- Upload de vídeo: **NÃO passa pelo backend** (ver GR-019).

**Critério de aceite**:
- Middleware `MaxBytesReader` por rota.
- 413 `payload_too_large` com mensagem user-facing pt-BR.

**Fonte**: legado `global.md` GR-018.

---

## GR-019 — Upload direto via presigned URL

**Prioridade**: M  
**Descrição**: Uploads grandes nunca passam pelo backend.

**EARS**: *Quando* client precisa subir vídeo (Mux) ou PDF grande (S3), *o sistema deve* (1) emitir presigned URL com TTL 15 min, (2) client PUT direto pro provider, (3) provider dispara webhook quando pronto, (4) backend consome webhook e finaliza aggregate.

**Critério de aceite**:
- `POST /videos` → retorna `{upload_url, mux_upload_id}`, não aceita bytes.
- Webhook Mux `video.asset.ready` → backend atualiza `Video` aggregate pra `status=ready`.
- Testes: request com body multipart > 10 MB em `/videos` → 413.

**Fonte**: legado `global.md` GR-019, [[00-product/portfolio-de-produtos]] Produto 4.

---

## GR-020 — CORS allowlist estrita (sem wildcard subdomain)

**Prioridade**: M  
**Descrição**: Wildcard `*` proibido em staging e prod. **Wildcard subdomain** (`*.mastersindico.com.br`) também proibido — allowlist é **específica** por origin.

**EARS**: *Quando* app inicia, *o sistema deve* ler `CORS_ALLOWED_ORIGINS` de env e validar via `Config.Validate()`. *Quando* env contém `*` OU qualquer padrão `*.<domain>` em staging/prod → **app não inicia** (fail-fast). *Quando* request CORS vem de origin fora da allowlist → resposta sem headers CORS (browser bloqueia).

**Critério de aceite**:
- `Config.Validate()` retorna erro se `ENV in {staging,prod} AND (CORS contains "*" OR CORS matches "*.*")`.
- Dev permite wildcard explícito.
- Allowlist canônica M1/M2: `https://app.mastersindico.com.br`, `https://admin.mastersindico.com.br` (e staging equivalentes).
- Cookie `Domain=app.mastersindico.com.br` (sem ponto-inicial) — **não compartilha com siblings**.
- Pre-flight `OPTIONS` responde corretamente com `Access-Control-Allow-Origin: <origin exato>`.
- Teste: `Origin: evil.mastersindico.com.br` → pre-flight rejeitado (origin não está na allowlist).
- INV-103 é a invariante.

**Fonte**: legado `global.md` GR-020, A-005 legado, [[STEERING]] §22, reforçado pós-[[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-003]].

---

## GR-021 — Secrets gitignored

**Prioridade**: M  
**Descrição**: Nenhum secret em código ou repo.

**EARS**: *Quando* dev commita mudança, *o sistema deve* (via pre-commit hook + CI) bloquear commit que contenha: `.env*`, `zitadel-key*.json`, `*.pem`, `*.key`, `STRIPE_SECRET_KEY=sk_live_*`, padrões AWS AK. *Quando* env é carregado em runtime, *o sistema deve* exigir variáveis críticas via `Config.Require()`.

**Critério de aceite**:
- `.gitignore` inclui `.env`, `.env.*`, `zitadel-key*.json`, `*.pem`.
- Pre-commit: `gitleaks` scan.
- CI: `trufflehog filesystem .` bloqueante.

**Fonte**: legado A-018, [[STEERING]] §23.

---

## GR-022 — Coverage thresholds

**Prioridade**: M  
**Descrição**: Cobertura mínima por camada (gate CI).

**EARS**: *Quando* pipeline CI roda `go test -cover`, *o sistema deve* falhar se:
- `domain/` < 95%
- `application/` < 90%
- `infrastructure/database/` < 85%
- `infrastructure/http/` < 75%
- `shared/middleware/` < 90%
- `core/`, `pkg/` < 95%
- Global < 85%.

**Critério de aceite**:
- Arquivo `.coverage.yml` versionado.
- GitHub Action bloqueante.
- Domain-level PBT (property-based testing) obrigatório em: fração ideal, quotas, ABAC matrix, voto único, paginação (ver [[STEERING]] §28).

**Fonte**: legado `CLAUDE.md §8`, [[STEERING]] §25.

---

## GR-023 — LGPD: consentimento versionado

**Prioridade**: M  
**Descrição**: Cada versão de termos tem ID; aceite do usuário é rastreado.

**EARS**: *Quando* usuário aceita termos no onboarding, *o sistema deve* registrar `{user_id, terms_version, accepted_at, ip, user_agent}`. *Quando* termos mudam, *o sistema deve* exigir re-aceite no próximo login.

**Critério de aceite**:
- Tabela `user_term_acceptances` append-only.
- Middleware: se usuário logado não aceitou versão atual → modal bloqueante.
- Versões armazenadas em `terms_versions` com `content_md`, `published_at`.

**Fonte**: legado `cross-domain.md` Req 35, [[STEERING]] §24.

---

## GR-024 — LGPD: direito de exclusão (SLA 15d)

**Prioridade**: M  
**Descrição**: Usuário pode solicitar exclusão; processamento em ≤ 15 dias úteis.

**EARS**: *Quando* usuário emite `POST /me/deletion-request`, *o sistema deve* (1) criar ticket `account_deletion_request`, (2) notificar DPO, (3) em ≤ 15 dias executar hard-delete de dados pessoais + soft-anonymize de histórico institucional (timeline/ata mantém `user_id` como `"morador_anon_xxxx"`).

**Critério de aceite**:
- Audit trail registra request e completion.
- Dados críticos institucionais (timeline, ata, audit) nunca removidos — só anonimizados.
- Teste: após purge, `GET /users/{id}` retorna 404.

**Fonte**: legado `cross-domain.md` Req 39, Lei 13.709/2018, [[STEERING]] §24.

---

## GR-025 — LGPD: portabilidade (export JSON)

**Prioridade**: S  
**Descrição**: Usuário solicita export dos próprios dados.

**EARS**: *Quando* usuário emite `GET /me/export`, *o sistema deve* gerar ZIP com JSON (perfil, memberships, timeline entries ligadas, votos, propostas emitidas/recebidas). *Quando* tamanho > 50MB, *o sistema deve* gerar async + email com link presigned (TTL 48h).

**Critério de aceite**:
- Formato documentado (schema JSON publicado).
- Testes: export de user com 100 entries → < 5s.

**Fonte**: legado `global.md` GR-012, LGPD Art. 18.

---

## GR-026 — Audit trail append-only (5 anos)

**Prioridade**: M  
**Descrição**: Trilha de auditoria imutável com retenção 5 anos.

**EARS**: *Quando* evento crítico acontece (login, criação/modificação de pauta, voto, publicação de comunicado, validação de execução, mudança de permissão, soft-delete), *o sistema deve* gravar `audit_trail` com `{user_id, tenant_id, action, resource_type, resource_id, timestamp, ip, user_agent, changes_before, changes_after}`. *Quando* tentativa de UPDATE/DELETE em `audit_trail` → DB CHECK/trigger bloqueia.

**Critério de aceite**:
- Tabela `audit_trail` + tabela `lgpd_logs` separada (LGPD-specific).
- DB trigger `prevent_mutation_audit_trail`.
- Retenção: após 1 ano, archive em S3 Glacier; após 5 anos, purge por job job mensal.

**Fonte**: legado `cross-domain.md` Req 37, [[STEERING]] §5, DT-010 legado.

---

## GR-027 — Trial persona-aware

**Prioridade**: M  
**Descrição**: Trial tem duração variável por persona, com soft-block sem grace period.

**EARS**: *Quando* usuário conclui onboarding, *o sistema deve* setar subscription com `trial_started_at=now()`, `trial_days = {sindico:15, empresa:7, parceiro:30, morador_pagante: 0}`. *Quando* trial expira, *o sistema deve* aplicar soft-block (dados intactos, features premium bloqueadas com `402 PLAN_REQUIRED`).

**Critério de aceite**:
- Testes: síndico dia 16 → criar comunicado → 402.
- Banner "Trial expira em 24h" em T-24h.

**Fonte**: legado `billing.md` Req 4.1, Decision 4, [[STEERING]] §45.

---

## GR-028 — Reputação determinística

**Prioridade**: S  
**Descrição**: Função pura de métricas observáveis; sem input humano direto.

**EARS**: *Quando* evento que afeta reputação ocorre (contrato fechado, avaliação submetida, vídeo aprovado, harassment confirmado), *o sistema deve* chamar `RecalculateReputation(companyId)` que é função pura retornando `(tier, score_components)`. *Quando* operador Master Síndico tenta `PATCH` direto em `companies.reputation_tier` → 403.

**Critério de aceite**:
- Ver [[functional/commercial]] COM-reputation-*.
- PBT: mesmos inputs → mesmo tier (idempotência).

**Fonte**: [[STEERING]] §39, [[00-product/portfolio-de-produtos]] Produto 3.

---

## GR-029 — Lock 90 dias em vídeo institucional e vídeo-currículo

**Prioridade**: M (feature) / S (enforcement M2)  
**Descrição**: Vídeo não pode ser substituído antes de 90 dias pós-publish.

**EARS**: *Quando* usuário tenta publicar novo vídeo institucional (empresa/síndico) ou vídeo-currículo (morador) *enquanto* existe vídeo do mesmo tipo publicado há < 90 dias, *o sistema deve* retornar `409 conflict` com `code=video_locked` + `unlock_at=<ts>`.

**Critério de aceite**:
- Validação no use case `PublishVideo`.
- Teste: publish dia 89 → 409; dia 91 → ok (arquiva anterior).

**Fonte**: legado `content.md` Req 29 rule 7, [[STEERING]] §40.

---

## GR-030 — Timeline imutável

**Prioridade**: M  
**Descrição**: Timeline entry é INSERT-only; correção vira nova entry vinculada.

**EARS**: *Quando* alguém tenta `UPDATE timeline_entries SET ...` ou `DELETE` → DB trigger rejeita. *Quando* correção é necessária, *o sistema deve* exigir criação de `timeline_entries` novo com `type=adendo` + `previous_entry_id`.

**Critério de aceite**:
- DB trigger `prevent_mutation_timeline_entries`.
- Tabela sem coluna `deleted_at`.
- Teste: UPDATE/DELETE raise exception.

**Fonte**: legado `institutional.md` Req 11, [[STEERING]] §41.

---

## GR-031 — UUIDv7 como primary key default

**Prioridade**: S  
**Descrição**: IDs são UUIDv7 ordenáveis (time-sorted).

**EARS**: *Quando* novo recurso é criado, *o sistema deve* gerar PK UUIDv7 via função `uuid_generate_v7()` no PostgreSQL (extensão ou função SQL). *Quando* ordenação cronológica é necessária → usar ID diretamente (sem `ORDER BY created_at`).

**Critério de aceite**:
- Extensão instalada (ou função pura SQL).
- Benchmark: insert 10k rows → order by id estável.

**Fonte**: [[STEERING]] §12.

---

## GR-032 — Money em centavos (int64)

**Prioridade**: M  
**Descrição**: Valores monetários são `int64` centavos, nunca float.

**EARS**: *Quando* sistema persiste preço/fatura/orçamento, *o sistema deve* armazenar em `BIGINT` representando centavos (ex: `990` = R$ 9,90). *Quando* API retorna valor → inteiro centavos + `currency: "BRL"`. *Quando* frontend exibe → formata `R$ 9,90`.

**Critério de aceite**:
- Grep: `DECIMAL.*NOT NULL` em colunas `*price*`, `*amount*` só permitido via ADR.
- Stripe: usa `amount: 990` (já em centavos natively).

**Fonte**: legado T10, [[STEERING]] §12.

---

## GR-033 — Fração ideal NUMERIC(5,4) imutável pós-publish

**Prioridade**: M  
**Descrição**: Fração ideal de unidade é `NUMERIC(5,4)`; soma por condomínio ≤ 1.0000.

**EARS**: *Quando* unidade é criada/editada, *o sistema deve* validar `0 < fracao_ideal ≤ 1.0000` e re-validar invariante Σ ≤ 1.0000 do condomínio. *Quando* assembleia publica pauta com votação ponderada, *o sistema deve* "congelar" fração ideal em snapshot pra votação (proteção contra alteração mid-flight).

**Critério de aceite**:
- DB CHECK: `fracao_ideal > 0 AND fracao_ideal <= 1.0000`.
- Trigger/constraint de soma.
- PBT: qualquer sequência de inserts respeita Σ ≤ 1.
- Snapshot `assembly_fractions` populado quando pauta publicada.

**Fonte**: legado `institutional.md` Req 9, `assembly.md` Req 49, [[STEERING]] §44.

---

## GR-034 — OpenAPI 3.1 contract-first

**Prioridade**: M  
**Descrição**: Spec OpenAPI escrita antes do handler.

**EARS**: *Quando* nova rota é planejada, *o sistema deve* ter spec OpenAPI 3.1 mergeada **antes** do PR do handler. *Quando* handler é implementado, *o sistema deve* rodar contract test (`oapi-codegen` server types + `kin-openapi` validator) em CI.

**Critério de aceite**:
- `openapi.v1.yaml` versionado.
- CI: `npx @redocly/cli lint openapi.v1.yaml` passa.
- Handler usa types gerados (sem drift).

**Fonte**: [[STEERING]] §13.

---

## GR-035 — Event-driven entre bounded contexts

**Prioridade**: M  
**Descrição**: Nenhum BC importa `domain/` ou `application/` de outro BC. Comunicação via domain events.

**EARS**: *Quando* BC-A precisa notificar BC-B (ex: `execution.validated` → atualiza `master_plan`), *o sistema deve* publicar evento via NATS JetStream (ou UoW in-process intra-módulo) e BC-B consome via listener. *Quando* build detecta import cruzado → compila falha (via `go vet` custom ou linter ArchUnit-like).

**Critério de aceite**:
- Lint regra em CI: no cross-BC imports.
- Teste: evento `deal.signed_complete` → timeline entry criada em `institutional` via listener.
- Outbox pattern para eventos críticos.

**Fonte**: [[STEERING]] §14, legado `institutional.md` Req 11.

---

## GR-036 — Webhook body size limit (DoS hardening)

**Prioridade**: M (M1)  
**Descrição**: Todo endpoint webhook inbound tem `http.MaxBytesReader(r.Body, 64*1024)` **antes** de qualquer leitura ou parse.

**EARS**: *Quando* request chega em `/webhooks/<provider>` (Stripe, Mux, Livekit, Zitadel), *o sistema deve* envolver `r.Body` com `MaxBytesReader(w, r.Body, 64*1024)` **antes** de `io.ReadAll` / HMAC verify / parse. *Quando* body excede 64KB → `413 payload_too_large` + audit log + drop sem processar.

**Justificativa do limite**:
- Stripe event payloads médio: 2-5KB (max observado: ~30KB em invoices detalhadas).
- Mux: 1-3KB.
- Livekit JWT (body inteiro): ~1-2KB.
- **64KB** é 2x o max observado, suficiente para margem; bloqueia trivialmente payload 100MB de DoS.

**Critério de aceite**:
- Middleware `WebhookBodyLimit(64KB)` obrigatório em rotas `/webhooks/*`.
- CI grep bloqueia `io.ReadAll(c.Request.Body)` em handler webhook sem reader limited.
- Teste: payload 100KB → 413.
- INV-102 é a invariante.

**Dependências**: GR-008 (HMAC antes de parse), [[../02-architecture/adr/0027-webhook-idempotency-db]].

**Fonte**: [[../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-002]].

---

## Matriz de rastreabilidade (GR → BC)

| GR | Identity | Billing | Institutional | Commercial | Content | Assembly | Cross-Domain |
|---|---|---|---|---|---|---|---|
| 001 OIDC+PKCE | ✓ primary | — | — | — | — | — | — |
| 002 ABAC | ✓ primary | — | ✓ | ✓ | ✓ | ✓ | ✓ engine |
| 003 Tenant isolation | ✓ | ✓ | ✓ primary | ✓ | ✓ | ✓ | — |
| 004 1-device | ✓ primary | — | — | — | — | — | — |
| 005 Request-ID | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 006 Rate limit | ✓ | — | — | — | — | — | — |
| 007 PII masking | ✓ | ✓ | ✓ | ✓ | — | — | ✓ |
| 008 Webhook HMAC | — | ✓ primary | — | — | ✓ Mux | ✓ Livekit | — |
| 009 Idempotência | — | ✓ primary | — | — | ✓ | ✓ | — |
| 010 i18n | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 011 Observability | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 012 LGPD high-level | ✓ | ✓ | ✓ | ✓ primary | ✓ | ✓ | ✓ |
| 013 WCAG | — | — | ✓ | ✓ | ✓ | ✓ | — |
| 014 /api/v1 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 015 Error envelope | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 016 Pagination | — | — | ✓ | ✓ | ✓ | ✓ | — |
| 017 Timezone | — | ✓ | ✓ | ✓ | — | ✓ | ✓ |
| 018 Upload limits | — | — | ✓ | ✓ | ✓ | ✓ | — |
| 019 Presigned URL | — | — | — | — | ✓ primary | ✓ | — |
| 020 CORS | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 021 Secrets | ✓ | ✓ | — | — | ✓ | ✓ | — |
| 022 Coverage | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 023 Consent | ✓ primary | — | ✓ | ✓ | — | — | ✓ |
| 024 Exclusão | ✓ primary | — | — | — | — | — | ✓ |
| 025 Portabilidade | ✓ primary | — | ✓ | ✓ | — | — | — |
| 026 Audit trail | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ primary |
| 027 Trial | — | ✓ primary | — | — | — | — | — |
| 028 Reputação | — | — | — | ✓ primary | ✓ input | — | — |
| 029 Lock 90d | — | — | — | — | ✓ primary | — | — |
| 030 Timeline imutável | — | — | ✓ primary | — | — | — | — |
| 031 UUIDv7 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 032 Money centavos | — | ✓ primary | — | ✓ | — | — | — |
| 033 Fração ideal | — | — | ✓ primary | — | — | ✓ | — |
| 034 OpenAPI 3.1 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 035 Event-driven | — | — | ✓ | ✓ | ✓ | ✓ | ✓ primary |

---

## Pendências (dual-check Fase 3)

- ⚠️ **PENDÊNCIA GR-010**: decidir estrutura exata de i18n (library: `go-i18n`? `messages.yaml` dir?) — M3+.
- ⚠️ **PENDÊNCIA GR-013**: definir toolchain WCAG (axe-core só? Pa11y? Lighthouse CI com quais rotas?) — dual-check antes de M2.
- ⚠️ **PENDÊNCIA GR-020**: quem mantém `CORS_ALLOWED_ORIGINS` allowlist? Fluxo operacional (PR + review security?) — dual-check.
- ⚠️ **PENDÊNCIA GR-026**: compressão de audit trail após 1 ano vs particionamento — ADR pendente na Fase 5.
- ⚠️ **PENDÊNCIA GR-031**: UUIDv7 via extensão `pg_uuidv7` vs função SQL pura — ADR pendente (trade-off: extensão requer build custom).
- ⚠️ **PENDÊNCIA GR-028**: parâmetros exatos da fórmula Bronze→Diamante em [[STATE]] D-010 — pendente dual-check com stakeholder.

---

## Referências

- Legado: `90-ingested/from-vault-50-projects-master-sindico/04-requirements/global.md` + `specs/requirements/cross-domain.md`
- [[STEERING]] §15-24 (BeyondCorp + qualidade + compliance)
- [[00-product/portfolio-de-produtos]] — sub-produtos que consomem GR-*
- [[01-domain/bounded-contexts]] — mapeamento BC
- [[functional/_moc]] — requirements funcionais por BC
- [[non-functional]] — NFRs
- [[compliance-lgpd]] — LGPD detalhado

## Links

- [[_moc]]
- [[functional/_moc]]
- [[non-functional]]
- [[compliance-lgpd]]
- [[matrix-functional]]
- [[registration-data]]
- [[../STEERING]]
