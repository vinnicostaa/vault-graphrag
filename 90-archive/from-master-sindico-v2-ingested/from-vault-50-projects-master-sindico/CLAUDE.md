---
title: CLAUDE.md — Master Síndico
type: agent-contract
tags: [claude, agent, project-contract, master-sindico]
project: master-sindico
created: 2026-04-22
---

# CLAUDE.md — Master Síndico

Contrato do agente para o projeto **Master Síndico** — plataforma SaaS de **governança condominial** (Brasil) multi-tenant, multi-persona. Complementa [[../../CLAUDE]] (raiz do vault).

> Ordem de leitura toda sessão: `vault/CLAUDE.md` → este arquivo → `STATE.md` → `11-audit/AUDIT.md` → `SESSION_CHARTER.md` → task do sprint atual.

---

## 1. Visão do produto

**O que é**: plataforma SaaS de governança condominial conectando síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local.

**O que NÃO é**:
- ❌ ERP condominial (cobrança, emissão fiscal)
- ❌ App de comunicação interna (WhatsApp substituto)
- ❌ Sistema financeiro

**É**: governança aplicada — critério na contratação (reputação **Bronze → Diamante**), memória institucional auditável (timeline imutável), ecossistema unificado (perfis, vídeos, cursos, contato direto).

**Invariante-chave**: identidade é única (CPF/CNPJ); contextos de operação são múltiplos. O mesmo usuário pode ser síndico no Condomínio X, morador no Y, e ter empresa cadastrada.

Ver [[00-product/vision]], [[00-product/personas]], [[00-product/business-model]], [[00-product/glossary]].

---

## 2. Sub-projetos

| Sub-projeto | Responsabilidade | Stack | Pasta canônica |
|---|---|---|---|
| **Backend** | API REST + WebSocket, domínio, persistência, integrações | Go 1.26 + Gin abstraído + pgx/v5 + go-redis/v9 + zerolog + goose + sqlc | [[03-subprojects/backend/README]] |
| **Web** | Portal síndico + back-office + marketing + área morador | SolidJS + TS + Vite + Tailwind | [[03-subprojects/web/README]] |
| **Mobile** | App morador/síndico (iOS + Android) | Flutter + Dart | [[03-subprojects/mobile/README]] |

---

## 3. Stack confirmada

| Camada | Tech | Status |
|---|---|---|
| Backend | Go 1.26 + Gin (abstraído via `contracts.HTTPRouter`) | ✅ |
| Query | pgx/v5 + sqlc + goose | ✅ |
| Auth | Zitadel (OIDC Authorization Code + PKCE) | ✅ D-001 |
| Cookie domain | `.mastersindico.com.br` httpOnly + fallback Bearer mobile | ✅ |
| Billing | Stripe via `IPaymentGateway` | ✅ |
| Vídeo | Mux via `IVideoProvider` | ✅ |
| Live (assembleias) | Livekit | ✅ |
| Busca | OpenSearch (migrar pra PG tsvector em dev; OS em prod) | ✅ |
| SMS | Twilio via `ISMSProvider` | ✅ |
| Email | AWS SES (considerar Resend) via `IEmailProvider` | ✅ |
| Push | FCM via `IPushProvider` | ✅ |
| Mensageria | NATS JetStream (quando necessário) | ✅ |
| DB | PostgreSQL 18 | ✅ |
| Cache | Redis 7 | ✅ |
| Infra | Railway (migração AWS ECS Fargate em plano) | ✅ D-02X |
| Observability | OpenTelemetry + Grafana + Sentry | ✅ Sprint 7 |
| Frontend | SolidJS + TS + Tailwind | ✅ |
| Mobile | Flutter + Dart | ✅ |

Decisões em aberto: ver [[STATE]] — D-024 (Railway vs ECS), D-026 (SES vs Resend), etc.

---

## 4. Arquitetura e padrões

### Padrões obrigatórios neste projeto

Herda [[../../CLAUDE#8-padrões-de-engenharia-inegociáveis-constituição-técnica]]. Especializações Master Síndico:

- **DDD tático** — entidades privadas com estado encapsulado; factories `NewX`/`ReconstructX`; repo interface no domain, impl em infra. Ver [[01-domain/bounded-contexts]], [[07-quality/PATTERNS#ddd]].
- **Clean Architecture estrita** — dependências apontam para dentro (`infrastructure → application → domain`). Domínio **zero** import de Gin, pgx, Stripe SDK, Mux SDK. Ver [[02-architecture/clean-arch-mapping]].
- **CQRS** — commands (`*_command.go`) separados de queries (`*_query.go`); use case por comando. Ver [[07-quality/PATTERNS#cqrs]].
- **UoW vs Saga** — `UnitOfWork.Run(ctx, fn)` quando tudo cabe na mesma tx; Saga (com compensação explícita) quando atravessa provider externo (Mux, Livekit, Stripe). Ver [[07-quality/PATTERNS#saga]], [[11-audit/AUDIT]] A-027/A-028.
- **BeyondCorp adaptado** — zero-trust, ABAC com matriz (admin_bypass → role → action → tenant), 1 dispositivo por conta via `device_fingerprint`, PKCE obrigatório. Ver [[08-security/BEYOND_CORP]].
- **Multi-tenant estrito** — toda query com `WHERE condominium_id = $tenant_id` quando escopo condomínio; cross-tenant isolation testada sistematicamente. Ver [[08-security/threat-model#multi-tenant]].
- **Imutabilidade** — `timeline_entries` sem `deleted_at`, sem UPDATE; governança auditável.
- **Code Calisthenics Go-flavor** — sem bloco `else`; early return / guard clauses; sem `any`/`interface{}` onde tipo concreto serve. Ver [[07-quality/CODE_CALISTHENICS]].

### Padrões proibidos neste projeto

- ❌ Bloco `else` — usar early return. Memória: `feedback_no_else`
- ❌ `_ = fn()` em I/O crítica — logar erro sempre (histórico: A-001..A-008, A-015, A-019)
- ❌ SDK externo no domínio (Stripe/Mux/Livekit/Zitadel SDK só em `infrastructure/providers/`)
- ❌ Tipo do ORM vazando do repo (mapper explícito obrigatório)
- ❌ `SELECT *` em queries sqlc (colunas explícitas — A-021)
- ❌ CORS wildcard em staging/prod (A-005, `Config.Validate()`)
- ❌ PII em logs (CPF/CNPJ/email/token — usar `Masked()` ou `user_id`)
- ❌ Abstração sem 2+ implementações reais (YAGNI)
- ❌ Features "enquanto estou aqui" — só o que a task pede

---

## 5. Bounded contexts

| Módulo | Responsabilidade | Status | Pasta canônica |
|---|---|---|---|
| `identity` | Usuários, sessões, autenticação Zitadel, 1-device policy | ✅ Produção | [[01-domain/bounded-contexts#identity]], [[04-requirements/functional/identity]] |
| `billing` | Planos, assinaturas, Stripe, trial por persona, quotas | ✅ Produção | [[04-requirements/functional/billing]] |
| `institutional` | Condomínio, unidades, timeline imutável, plano diretor | ✅ Produção | [[04-requirements/functional/institutional]] |
| `commercial` | Connect Me (4 fluxos), contratos, marketplace, banco de talentos, empresas-sindicas | ✅ Produção | [[04-requirements/functional/commercial]] |
| `content` | Vídeos Mux (trava 90 dias), busca OpenSearch, privacidade de métricas | ✅ Produção | [[04-requirements/functional/content]] |
| `assembly` | Assembleias live (WebSocket/Livekit), votações, atas, fila de fala | ✅ Produção | [[04-requirements/functional/assembly]] |
| `cross-domain` | Integrações entre módulos, eventos de domínio, IAuditLogger (DT-010) | 🟡 Em evolução | [[04-requirements/functional/cross-domain]] |

Context map em [[01-domain/context-map]]. Ubiquitous language em [[01-domain/ubiquitous-language]].

---

## 6. Marcos de entrega

| Marco | Data | Escopo |
|---|---|---|
| **M1 — MVP contratual** | **2026-05-08** | Identity + Billing + Institutional base |
| **M2 — Connect Me + Conteúdo** | 2026-06-20 | Commercial + Content |
| **M3 — Assembleia + LMS + Polish** | 2026-07-20 | Assembly + LMS + melhorias finais |

Priorização **sempre** protege M1. Ver [[ROADMAP]], [[10-schedule/cronograma-macro]].

---

## 7. Fluxo canônico de execução

Ver [[../../10-knowledge/methodology/sdd-workflow]] e [[../../30-agents/playbooks/plan-and-execute]].

**SDD 5-fase**: Discuss → Plan → Execute → Verify → Ship → Audit (fim de sprint).

**Gate 0** toda sessão: abrir [[11-audit/AUDIT]]. Itens 🔴🟡 **bloqueiam** task nova.

**Ordem canônica de implementação** (adaptar):
```
1. Migration SQL + CHECK constraints (numeração particionada por módulo)
2. sqlc query
3. Domain entity / VO
4. Repo interface (domain)
5. Domain event (se cross-context)
6. Use case CQRS (command ≠ query, arquivo separado)
7. Mapper
8. Repo impl (infrastructure/database)
9. Provider externo (se houver SDK)
10. Handler + rota
11. Wire-up no module + main
12. Testes (unidade + integração)
```

---

## 8. Coverage thresholds (`.coverage.yml`)

| Camada | Mínimo |
|---|---|
| `domain/` (entities, VOs, services) | **95%** |
| `application/` (use cases) | **90%** |
| `infrastructure/database/` | **85%** |
| `infrastructure/http/` | **75%** |
| `shared/middleware/` | **90%** |
| `core/`, `pkg/` | **95%** |
| **Global** | **≥ 85%** |

Gates duros (task não é mergeada sem):

```bash
go build ./...                                # ✅
go vet ./...                                  # ✅
go test -race -count=1 ./...                  # ✅
go test -tags=integration -count=1 ./...      # ✅
gosec -severity=high ./...                    # ✅ 0 findings
govulncheck ./...                             # ✅ 0 CVEs em reachable
```

---

## 9. Security posture (BeyondCorp adaptado)

Detalhamento em [[08-security/BEYOND_CORP]]. Resumo:

- **Zero-trust** — sem "rede interna confiável"; toda request auth + autz
- **ABAC engine** — admin_bypass → role_not_allowed → action_not_configured → tenant_mismatch
- **Tenant isolation** — cross-tenant tests sistemáticos (100+ em ABAC matrix)
- **1 device per account** — `device_fingerprint` registrado; `InvalidateOtherSessions` ao criar nova
- **Rate limiting** — /auth (20rpm, burst 5); /api/v1 (60rpm, burst 10); sync.Map com cleanup (A-006, A-032)
- **PKCE obrigatório** — code_verifier em cookie gorilla/securecookie
- **Secrets** — nunca em código; `.env` gitignored (protege `zitadel-key*.json` — A-018)
- **PII em logs** — proibido: CPF/CNPJ/email/token; usar `Masked()`
- **CORS** — bloqueio de `*` em staging/prod via `Config.Validate()`

---

## 10. MCPs e plugins habilitados

Detalhamento em [[plugins]]. Obrigatórios dia 1:

- `context7@claude-plugins-official` — doc oficial de libs (obrigatório antes de usar SDK)
- `gopls-lsp@claude-plugins-official`
- `typescript-lsp@claude-plugins-official`
- `security-guidance@claude-plugins-official`
- `railway@claude-plugins-official`

MCPs ativos: `obsidian` (user-level), `github` (oficial), `postgres@crystaldba` (read-only), `aws-docs@awslabs`.

---

## 11. Modo autônomo — limites

Herda [[../../30-agents/autonomous-execution]]. Categorias destrutivas que **pausam** o modo autônomo:

1. `DROP TABLE / DROP COLUMN` ou migration destrutiva sem expand-contract
2. Deploy em produção
3. `git push --force` em main/develop
4. Remover dependência crítica (pgx, gin-adapter, go-redis)
5. Operação em dado pessoal real (LGPD)

**Em dúvida técnica não-destrutiva**: rodar [[../../30-agents/playbooks/research-loop]] + [[../../30-agents/playbooks/dual-check]] e registrar D-###. **Nunca** pausar por "não sei decidir X" — o playbook resolve.

---

## 12. Fontes de verdade (hierarquia)

Ordem de consulta quando há divergência:

1. **Código real** em `backend/internal/` — estado atual
2. `04-requirements/**` + `05-tasks/**` — canônico para requisitos e tasks
3. `02-architecture/**` + `01-domain/**` — como e por quê
4. `STATE.md` — decisões vivas (D-###)
5. `11-audit/AUDIT.md` — findings abertos (A-###)
6. `legacy-reference` — **referência de domínio**, não template de código. Ver [[legacy-reference]]
7. `client-vault/**` — visão original do cliente (System Architecture v4.0 Mar/2026 **parcialmente desatualizado** — menciona Ent/Casbin/Uber-FX não adotados)
8. `contextos/**` — docs-fonte destilados em `04-requirements/` (pasta `content/` sanitizada 2026-04-22; ver [[_archive/content-moved-2026-04-22]])

**Quando divergir**: código vence sobre doc; atualizar doc **antes** de prosseguir.

---

## 13. Idioma

- Corpo de notas: **pt-BR**
- Frontmatter, identifiers Go/TS/Dart, títulos de API: **inglês**
- Docstrings de API pública com `// EN: ... / // PT: ...` bilíngue

---

## 14. Checklist de fim de sessão

- [ ] `STATE.md` atualizado (D-### novos / dívidas)
- [ ] `11-audit/AUDIT.md` com findings novos/fechados
- [ ] `SESSION_CHARTER.md` registra ordens e progresso
- [ ] Notas novas com frontmatter + ≥ 2 `[[links]]`
- [ ] MOCs das pastas impactadas atualizados
- [ ] Gates verdes: `go build / vet / test -race / integration / gosec -sev high / govulncheck`
- [ ] `tasks.md` reflete progresso (cuidado com reincidência de A-012/A-017)
- [ ] Docs sincronizadas (README, OpenAPI, CHANGELOG)

---

## 15. Links essenciais do projeto

### Governança viva
- [[_moc]] — MOC do projeto
- [[STATE]] — decisões (D-###) e dívidas (DT-###)
- [[11-audit/AUDIT]] — findings (A-###)
- [[SESSION_CHARTER]] — ordens da sessão atual
- [[STEERING]] — não-negociáveis do projeto
- [[ROADMAP]] — marcos M1/M2/M3
- [[DoR-DoD]] — ready / done

### Contexto
- [[overview]] — visão geral (legado, mantido por compatibilidade)
- [[legacy-reference]] — TypeScript legacy como referência
- [[plugins]] — plugins Claude Code habilitados
- [[client-vault/00 - Visão Geral/00 - Índice Master Síndico]] — vault do cliente

### Canônico (zero → prod)
- [[00-product/_moc]]
- [[01-domain/_moc]]
- [[02-architecture/_moc]]
- [[03-subprojects/_moc]]
- [[04-requirements/_moc]]
- [[05-tasks/_moc]]
- [[06-execution/_moc]] — inclui STEPS.md
- [[07-quality/_moc]] — CLEAN_CODE / CLEAN_ARCH / CODE_CALISTHENICS / PATTERNS
- [[08-security/_moc]] — BEYOND_CORP / threat-model / CVE process
- [[09-testing/_moc]]
- [[10-schedule/_moc]]
- [[11-audit/_moc]]
- [[12-docs/_moc]]

### Vault geral
- [[../../CLAUDE]] — contrato raiz
- [[../../30-agents/playbooks/_moc]] — playbooks obrigatórios
- [[../../20-stacks/go/_moc]] — idiomatic Go
- [[../../10-knowledge/architecture/clean-architecture-layers]]
