---
title: Railway — Antipatterns
type: note
tags: [provider, railway, antipatterns, deploy, ops]
source: https://docs.railway.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Railway Antipatterns]
---

# Railway — Antipatterns

Erros recorrentes ao operar Railway — o que **não fazer**.

## ❌ Secrets em `.env` commitado no git

Secrets (API keys, DB passwords, JWT secrets) **nunca** no git. Use Railway Variables (sealed). `.env.example` sem valores é OK.

## ❌ `PORT` hardcoded

Railway injeta `$PORT` dinâmico. Hardcode `:8080` falha em runtime. Sempre:

```go
port := os.Getenv("PORT")
if port == "" { port = "8080" }   // fallback só para dev local
http.ListenAndServe(":"+port, mux)
```

## ❌ Containers sem `HEALTHCHECK` nem endpoint `/health`

Deployment fica "unhealthy" e Railway pode considerar rollback. Sempre expor `/health` retornando 200 em < 5s.

## ❌ Não drenar conexões em shutdown (SIGTERM ignorado)

SIGTERM sem handler = Railway mata forçadamente 10s depois → connections drop → 502 para clientes. Sempre implementar graceful shutdown ([[patterns#2 Graceful shutdown]]).

## ❌ Log stdout plain text sem JSON estruturado

Railway parse básico de texto; downstream (Datadog/Loki) parse JSON. Logs plain text perdem campos (trace_id, user_id) na agregação. Use zerolog/pino/slog com JSON output.

## ❌ Migrations no startup hook da app

Race entre múltiplas instâncias:
- 2 pods sobem paralelo → ambos tentam `ALTER TABLE` → lock conflict ou estado inconsistente.
- Rodar como **pre-deploy command** via Railway (ou job separado `railway run migrate`).

## ❌ Usar `0.0.0.0:<PORT fixo>` ignorando port injetado

Já coberto acima, mas vale reforçar: Railway orquestra port mapping. Hardcode = crash.

## ❌ Skip preview deploys em PRs

Perde gate de CI visual. Preview deploys custam pouco (ephemeral; destroi no merge) e evitam bug em main. Sempre habilitar para projetos com frontend.

## ❌ Não configurar backup do plugin Postgres

Settings → Postgres plugin → Backups. Default **off** (!). Habilitar com frequência ≥ diária. Retenção mínima 7 dias.

## ❌ Egress público quando private networking bastaria

Service A chama Service B via URL pública externa quando ambos estão no mesmo project:

```
# antipattern
API_URL=https://api.mystartup.com

# correto (zero egress billing)
API_URL=http://api.railway.internal:3000
```

Economia real em arquiteturas chatty (microservices).

## ❌ Um único environment para tudo

Usar só `production` sem `staging` = deploys sem teste. Criar `staging` sempre; deploy flow: PR → preview → merge → staging → manual promote → production.

## ❌ Cron jobs sem idempotência

Railway pode re-executar cron em retry cenários edge. Jobs precisam ser idempotentes (advisory lock, flag de execução em DB, ou `ON CONFLICT DO NOTHING` em INSERTs).

## ❌ Alto RAM/CPU settings sem justificativa

Pay-per-resource: RAM/CPU ociosos queimam billing. Começar com 512MB + 0.5 vCPU e ajustar via métricas reais. Nunca "por segurança" 4GB + 2 vCPU sem evidência.

## ❌ Depender de filesystem local persistente

Containers em Railway são ephemeral — disk zera em restart. Para persistência, usar plugin Postgres/Redis ou volume externo (S3/R2). Nunca gravar dados críticos em `/tmp` achando que fica.

## ❌ Logs com PII em claro

CPF, email, telefone, tokens em logs ferem LGPD. Sempre mascarar (`util.Masked`) — Railway logs aparecem em dashboard visível a qualquer colaborador do project.

## Links

- [[_moc]]
- [[../_moc]]
- [[patterns]]
- [[../../antipatterns/_moc]]
- [[../../security/beyond-corp]] — PII em logs é violação LGPD
