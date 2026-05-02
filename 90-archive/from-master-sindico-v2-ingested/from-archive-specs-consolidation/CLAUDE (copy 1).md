# CLAUDE.md

Guia **enxuto** de orientação para agentes Claude Code trabalhando neste repositório.
Detalhes vivem em `.kiro/steering/` (carregados sob demanda) e `AGENTS.md`
(contrato completo do agente). Este arquivo só lista ponteiros e o essencial
para a primeira leitura — maximiza prompt caching entre sessões.

## Projeto

**Master Síndico** — SaaS de governança condominial para Brasil. Este repositório é o
**backend Go** (Clean Architecture + DDD + CQRS), parte do monorepo em `../`
(ver `../CLAUDE.md`).

Stack: Go 1.26 + Gin (abstraído) + PostgreSQL 18 (pgx+sqlc) + Redis 7 + Zitadel OIDC +
Stripe + Mux + Livekit + Resend + MinIO + NATS + goose + zerolog + OTel.
BeyondCorp adaptado (Zero Trust, ABAC, 1-device fingerprint).

## Fontes de verdade (ordem de consulta)

1. **`.kiro/SESSION_CHARTER.md`** — ordens ativas da sessão corrente (se houver). Leia sempre antes de tudo.
2. **`.kiro/specs/master-sindico/`** — requirements, design, tasks. Primeira parada para qualquer implementação.
3. **`.kiro/steering/*.md`** — regras duras por tópico. Carregue o relevante antes de codar:
   - `sdd-workflow.md`, `autonomous-execution.md`, `mcp-integration.md`
   - `go-patterns.md`, `database.md`, `security.md`, `testing.md`
   - `do-dont.md` (checklist consolidado), `sdd-layers.md`, `code-calisthenics.md`, `transaction-patterns.md`
4. **`.kiro/STATE.md`** — bloqueadores vivos e decisões em aberto (D-0XX).
5. **`.kiro/AUDIT.md`** — findings pendentes (A-0XX). **Gate 0** de qualquer sessão: se houver `🔴 Aberto` ou `🟡 Em andamento`, resolver antes da próxima task.
6. **`AGENTS.md`** — contrato completo do agente (papel, princípios, quando recusar, estrutura de pastas).
7. **Código existente** — quando spec e código divergem, ler o código e atualizar spec.

Se spec não existe ou está ambígua: **pare e atualize spec antes de implementar**.

## Ciclo obrigatório por task

```
Discuss → Plan (com <verify>) → Execute → Verify (build+vet+test+coverage) → Ship (PR)
```

Regras duras (detalhes em `.kiro/steering/sdd-workflow.md`):

- **Consulte doc oficial via Context7 MCP** antes de usar qualquer SDK externo.
- **Coverage thresholds são gate duro**: domain 95%, application 90%, handlers 75%, middleware 90%, global ≥85%.
- **Security gates**: `gosec -severity high` + `govulncheck` passam antes de PR.
- **Never trust the frontend**: validação server-side sempre.
- **README.md do sub-projeto impactado** é atualizado na fase Ship (feature-level; bug-fix não atualiza).
- **Sprint Audit (Fase 6)** obrigatória ao fim de cada sprint: `/sprint-auditor`.

## Comandos (quick reference)

```bash
docker compose up -d                              # Postgres 18 + Redis 7 + NATS + Zitadel + MinIO + Livekit
go build ./... && go vet ./...
go test -race -count=1 ./...
go test -tags=integration -count=1 ./...
go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
sqlc generate && git diff --exit-code internal/**/generated
go run ./cmd/migrate up                           # goose via embed.FS
gosec -severity high -exclude-dir=generated -exclude-dir=cmd/zitadel-setup ./...
govulncheck ./...
go run ./cmd/api
stripe listen --forward-to localhost:8000/webhooks/stripe
```

## Estrutura do código

Ver `AGENTS.md` — seção "Estrutura do projeto". Este arquivo **não** duplica para evitar drift.

## Regras absolutas

Ver `.kiro/steering/do-dont.md` (checklist consolidado) e `AGENTS.md` (princípios).
Este arquivo lista apenas a **ordem de leitura**; evita duplicar texto que vive em steering.

## Skills disponíveis (`.claude/skills/`)

- `task-executor` — executa task do `tasks.md` seguindo protocolo SDD
- `sprint-auditor` — revisa sprint concluído, materializa findings em `.kiro/AUDIT.md`
- `dual-validate` — pattern Opus+Opus para decisões de alto impacto em modo autônomo
- `go-review` — revisa código Go por prioridade
- `migration-writer` — cria migration goose com convenções do projeto
- `sqlc-writer` — escreve query sqlc com tipos corretos

Invoque com `/{nome}` no Claude Code.

## Sprint atual

**Sprint 9 — Integrations** (iniciado 2026-04-22). Sprints 0–8 backend completos. Status:

- ✅ Providers: Mux (vídeo VOD + live + webhook HMAC público), Livekit (WebRTC assembly),
  Resend (email templates pt-BR parciais), MinIO (storage S3-compatible), Stripe (checkout + webhook).
- ✅ Dockerfile multi-stage + railway.toml + GitHub Actions CI.
- ✅ Zitadel v4 em docker-compose com Traefik + Login UI.
- ✅ OpenAPI 3.0 em `/docs` (Scalar UI).
- ✅ OTel completo (traces + metrics + Prometheus bridge).
- ⏳ Pendente: Zitadel managed em Railway (9.19); email templates boas-vindas, assembly, billing-alerts (9.10); IAuditLogger cross-módulo (DT-010).

**Marco 1**: 2026-05-08 (MVP contratual). Ver `.kiro/specs/master-sindico/tasks.md` + `.kiro/STATE.md`.
