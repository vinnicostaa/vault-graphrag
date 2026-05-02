---
title: CLAUDE.md — Master Síndico
type: agent-contract
tags: [claude, agent, project-contract, master-sindico]
project: master-sindico
created: 2026-04-22
updated: 2026-04-23
---

# CLAUDE.md — Master Síndico

Contrato do agente para o projeto **Master Síndico** — plataforma SaaS de **governança condominial** (Brasil) multi-tenant, multi-persona. Complementa [[../../CLAUDE]] (raiz do vault).

> **Ordem de leitura toda sessão**: `vault/CLAUDE.md` → este arquivo → [[STATE]] → [[11-audit/AUDIT]] → [[SESSION_CHARTER]] → task do sprint atual.

> **Regra dura**: antes de afirmar qualquer coisa técnica, completar o [[../../30-agents/playbooks/research-loop]] + [[../../30-agents/playbooks/dual-check]]. Nunca inventar. Sempre linkar fonte.

---

## 1. Visão do produto

**O que é**: plataforma SaaS de governança condominial conectando síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local.

**O que NÃO é**:
- ❌ ERP condominial (cobrança, emissão fiscal)
- ❌ App de comunicação interna (WhatsApp substituto)
- ❌ Sistema financeiro

**É**: governança aplicada — critério na contratação (reputação **Bronze → Diamante**), memória institucional auditável (timeline imutável), ecossistema unificado (perfis, vídeos, cursos, contato direto).

**Invariante-chave**: identidade é única (CPF/CNPJ); contextos de operação são múltiplos. O mesmo usuário pode ser síndico no Condomínio X, morador no Y, e ter empresa cadastrada.

Ver [[00-product/portfolio-de-produtos]], [[00-product/personas]], [[00-product/business-model]].

---

## 2. Sub-produtos (11 consolidados Fase 8)

1. Governança Institucional (núcleo M1)
2. Connect Me — marketplace unidirecional (4 fluxos S→E, M→S, E→E Pro, S→Parceiro)
3. Reputação Bronze→Diamante — fórmula determinística (D-051)
4. Conteúdo & Vídeos (Mux)
5. Assembleias Live & Votações (Livekit)
6. LMS & Academia Thin
7. Banco de Talentos
8. Vizinhança / Parceiro Local (com Sistema de Cupons — D-059/D-078)
9. Compliance (C1-C11)
10. Agência de Marketing (role Zitadel própria — D-060/D-073)
11. Empresa-Síndica

Detalhes em [[00-product/sub-produtos/_moc]].

---

## 3. Sub-projetos

| Sub-projeto | Responsabilidade | Stack | Canônico |
|---|---|---|---|
| **Backend** | API REST + WebSocket, domínio, persistência, integrações | Go 1.26 + Gin abstraído + pgx/v5 + sqlc + goose + zerolog + Redis 7 + NATS | [[03-subprojects/backend/README]] |
| **Web** | Portal síndico + cms + auth + lms + forum + assembly + lp | SolidJS + TS + Rsbuild + UnoCSS + Kobalte + Biome + Bun | [[03-subprojects/web/README]] |
| **Mobile** | App morador/síndico iOS + Android | Flutter + Dart + flutter_bloc + get_it + go_router + oidc | [[03-subprojects/mobile/README]] |

---

## 4. Stack confirmada (reconciliada D-067/D-080)

| Camada | Tech | Status |
|---|---|---|
| Backend | Go 1.26 + Gin (abstraído via `contracts.HTTPRouter`) | ✅ |
| Query | pgx/v5 + sqlc + goose | ✅ |
| Auth | Zitadel (OIDC Authorization Code + PKCE) | ✅ D-001 |
| Cookie domain | `.mastersindico.com.br` httpOnly + fallback Bearer mobile | ✅ |
| Billing | Stripe via `IPaymentGateway` | ✅ |
| Vídeo VOD | Mux via `IVideoProvider` | ✅ |
| Live (assembleias) | Livekit | ✅ |
| Busca | **ISearchProvider** com adapter OpenSearch ou Meilisearch (D-120 revoga D-067 — PG tsvector descartado; busca é vital pro produto desde M1, não acessório) | 🟡 adapter inicial a decidir Sprint 10 (ADR-0042 pending dual-check) |
| SMS | Twilio via `ISMSProvider` (stub — provider real M2) | ✅ |
| Email | **Resend** via `IEmailProvider` (D-067 substituiu SES) | ✅ |
| Push | FCM via `IPushProvider` (stub — provider real M2) | ✅ |
| Mensageria | NATS JetStream (threshold — ADR-0019) | 🟡 |
| DB | PostgreSQL 18 | ✅ |
| Cache | Redis 7 | ✅ |
| Infra | **Cloudflare-only** (após [[STATE#D-138]] / [[02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]] em 2026-04-27 — Pages + Workers + Containers + R2 + KV/DO + Hyperdrive→Postgres externo Neon/Supabase). Histórico: D-067 substituiu AWS ECS por Railway; D-138 substitui Railway por Cloudflare. | ✅ |
| Observability | OpenTelemetry + Grafana + Sentry | ✅ Sprint 7 |
| Frontend | SolidJS + TS + UnoCSS + Kobalte (D-042) | ✅ |
| Mobile | Flutter + Dart + Bloc + freezed + hydrated_bloc (D-048/049) | 🟡 deps pendentes (A-FASE10-DEV-003) |

Decisões em aberto: ver [[STATE]] — D-046 (email M3+), D-050 (freeRASP gated), A-FASE10-DEV-001..005 reconciliações.

---

## 5. Padrões de engenharia (referências ao knowledge global)

Herda [[../../CLAUDE#8-padrões-de-engenharia-inegociáveis-constituição-técnica]]. **Todos os padrões abaixo vêm do knowledge atemporal**; aqui é **aplicação MS**:

| Padrão | Referência canônica | Aplicação MS |
|---|---|---|
| **Clean Architecture** | [[../../10-knowledge/architecture/clean-architecture-layers]] | 4 camadas rígidas por BC; domínio zero SDK externo |
| **DDD** | [[../../10-knowledge/architecture/ddd-strategic-tactical]] | 6 BCs explícitos; ubiquitous language em [[01-domain/ubiquitous-language]] |
| **CQRS** | [[../../10-knowledge/architecture/cqrs]] | Command ≠ Query; arquivo separado por use case |
| **SOLID** | [[../../10-knowledge/principles/solid]] | Especialmente DIP (domain define interface, infra implementa) |
| **Clean Code** | [[../../10-knowledge/principles/clean-code]] | 17 princípios aplicados; nomes em pt-BR no copy/logs |
| **Code Calisthenics** | [[../../10-knowledge/principles/code-calisthenics]] | 9 regras Go-flavor; sem bloco `else`; early return |
| **Do/Don't** | [[../../10-knowledge/principles/do-dont-checklist]] | Checklist consolidado cross-linguagem |
| **SDD** | [[../../10-knowledge/methodology/sdd-workflow]] | Ciclo 5-fase (Discuss→Plan→Execute→Verify→Ship) + Fase 6 Sprint Audit |
| **GSD** | [[../../10-knowledge/methodology/gsd-get-shit-done]] | Ship small + DoD explícito + `<verify>` declarativo + STATE vivo |
| **TDD** | [[../../10-knowledge/testing/testing-strategy]] | Coverage thresholds em [[09-testing/coverage-thresholds]] |
| **BDD** | [[../../10-knowledge/testing/bdd-behavior-driven-development]] | Para features críticas (assembleia, votação, cupom) |
| **ACID + SAGA** | [[../../10-knowledge/patterns/transaction-patterns]] | UoW intra-BC; Saga inter-BC (billing checkout, video upload, live session) |
| **BeyondCorp / Zero Trust** | [[../../10-knowledge/security/beyond-corp]] | Detalhado em [[08-security/BEYOND_CORP]] |
| **Legacy-as-reference** | [[../../10-knowledge/methodology/legacy-as-reference]] | Ex full-stack TS morto em 2026-02-28 — referência, não template |
| **Graph-RAG** | [[../../10-knowledge/methodology/graph-rag]] | Este vault inteiro é um grafo — linkar, não duplicar |

### Padrões proibidos neste projeto

- ❌ Bloco `else` — usar early return. Memória: [[../../10-knowledge/antipatterns/_moc]]
- ❌ `_ = fn()` em I/O crítica — logar erro sempre
- ❌ SDK externo no domínio (Stripe/Mux/Livekit/Zitadel SDK só em `infrastructure/providers/`)
- ❌ Tipo do ORM vazando do repo (mapper explícito obrigatório)
- ❌ `SELECT *` em queries sqlc (colunas explícitas — A-021)
- ❌ CORS wildcard em staging/prod (A-005, `Config.Validate()`)
- ❌ PII em logs (CPF/CNPJ/email/token — usar `Masked()` ou `user_id`)
- ❌ Abstração sem 2+ implementações reais (YAGNI)
- ❌ Features "enquanto estou aqui" — só o que a task pede

---

## 6. Bounded contexts (6)

| Módulo | Responsabilidade | Status | Canônico |
|---|---|---|---|
| `identity` | Usuários, sessões, autenticação Zitadel, 1-device policy | ✅ Produção | [[04-requirements/functional/identity]] |
| `billing` | Planos globais (trial/base/plus/pro — ADR-0032), assinaturas Stripe, quotas, trial | ✅ Produção | [[04-requirements/functional/billing]] |
| `institutional` | Condomínio, unidades, timeline imutável (ADR-0033), plano diretor | ✅ Produção | [[04-requirements/functional/institutional]] |
| `commercial` | Connect Me (4 fluxos), contratos, reputação, marketplace, banco de talentos, empresas-sindicas | ✅ Produção | [[04-requirements/functional/commercial]] |
| `content` | Vídeos Mux (trava 90d), busca via `ISearchProvider` real (OpenSearch/Meilisearch — ADR-0042 pending; D-120 revoga D-067 tsvector — stub M1), privacidade de métricas | ✅ Produção | [[04-requirements/functional/content]] |
| `assembly` | Assembleias live (Livekit), votações, atas imutáveis, quórum fracional | ✅ Produção | [[04-requirements/functional/assembly]] |
| `cross-domain` | Integrações entre módulos, eventos de domínio, IAuditLogger | 🟡 Em evolução (DT-010) | [[04-requirements/functional/cross-domain]] |

Context map em [[01-domain/invariants]]. Ubiquitous language em [[01-domain/ubiquitous-language]].

---

## 7. Marcos de entrega

| Marco | Data | Escopo |
|---|---|---|
| **M1 — MVP contratual** | **2026-05-18** (firme — fechado em D-106 após avaliação Web 0% + LGPD bloqueadores) | Identity + Billing + Institutional base + Web onboarding + Mobile auth shell + ISearchProvider real |
| **M2 — Connect Me + Conteúdo** | 2026-06-20 | Commercial + Content + Reputação + Cupons |
| **M3 — Assembleia + LMS + Polish** | 2026-07-20 | Assembly + LMS + Banco de Talentos + a11y WCAG 2.1 AA |

Priorização **sempre** protege M1. Ver [[ROADMAP]], [[10-schedule/cronograma-macro]].

---

## 8. Fluxo canônico de execução

Playbook master: [[../../30-agents/playbooks/plan-and-execute]].

**Gate 0** toda sessão: abrir [[11-audit/AUDIT]]. Itens 🔴/🟠 **bloqueiam** task nova.

**Ordem canônica de implementação (backend)**:
```
1. Migration SQL + CHECK constraints (numeração particionada por módulo)
2. sqlc query tipada
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

Detalhamento em [[../../10-knowledge/architecture/clean-architecture-layers#ordem-de-implementação-por-feature]].

---

## 9. Coverage thresholds

Fonte canônica: [[09-testing/coverage-thresholds]].

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

## 10. Security posture

Canônico: [[../../10-knowledge/security/beyond-corp]]. Detalhamento específico MS: [[08-security/BEYOND_CORP]], [[08-security/lgpd]].

Resumo:
- **Zero-trust** — sem "rede interna confiável"
- **ABAC engine** matriz declarativa role × plan_tier × action × tenant
- **Tenant isolation** via RLS default-on (ADR-0034) + TenantBoundaryGuard middleware (INV-122)
- **1 device per account** — `device_fingerprint` registrado
- **PKCE obrigatório** — code_verifier em cookie gorilla/securecookie
- **Ata/Timeline DB-immutable** — REVOKE + trigger (ADR-0033)
- **pgBackRest PITR** + DR drill mensal (ADR-0035)
- **LGPD** — audit trail 5y + pseudonimização HMAC keyed (ADR-0028)
- **CORS** — bloqueio de `*` em staging/prod via `Config.Validate()`

---

## 11. MCPs e plugins habilitados

Ver [[plugins]]. Obrigatórios dia 1:

- `context7@claude-plugins-official` — doc oficial de libs
- `gopls-lsp@claude-plugins-official`
- `typescript-lsp@claude-plugins-official`
- `security-guidance@claude-plugins-official`
- `railway@claude-plugins-official`

MCPs ativos: `obsidian` (este vault), `github`, `postgres@crystaldba` (read-only), `aws-docs@awslabs`.

---

## 12. Modo autônomo — limites

Herda [[../../30-agents/autonomous-execution]]. **Dual-check obrigatório** em toda decisão não-trivial (ver [[../../30-agents/playbooks/dual-check]]).

Categorias destrutivas que **pausam** modo autônomo:

1. `DROP TABLE / DROP COLUMN` sem expand-contract
2. Deploy em produção
3. `git push --force` em main/develop
4. Remover dependência crítica
5. Operação em dado pessoal real (LGPD)

**Em dúvida técnica não-destrutiva**: research-loop + dual-check + registrar D-###. **Nunca** pausar por "não sei decidir X".

---

## 13. Fontes de verdade (hierarquia)

Ordem de consulta quando há divergência:

1. **Código real** em `Development/backend/internal/` + `Development/app/lib/` + `Development/web/apps/` — estado atual
2. **Este CLAUDE.md** + [[STATE]] + [[11-audit/AUDIT]] — decisões e findings
3. [[04-requirements/_moc]] + [[05-tasks/_moc]] — canônico para requisitos e tasks
4. [[02-architecture/_moc]] + [[01-domain/_moc]] — como e por quê
5. `_archive/2026-04-23-pre-v2-migration/` — legado arquivado (referência histórica)
6. `../../60-sources/master-sindico-research/` — research destilado (big techs, condominial BR)
7. `../../90-archive/from-master-sindico-v2-ingested/` — material histórico ingerido

**Quando divergir**: código vence sobre doc; atualizar doc **antes** de prosseguir.

---

## 14. Idioma

- Corpo de notas: **pt-BR**
- Frontmatter, identifiers Go/TS/Dart, títulos de API: **inglês**
- Docstrings de API pública bilíngue (`// EN: ... / // PT: ...`)
- Commit messages: pt-BR ou inglês (consistente por PR)

---

## 15. Checklist de fim de sessão

- [ ] [[STATE]] atualizado (D-### novos / dívidas)
- [ ] [[11-audit/AUDIT]] com findings novos/fechados
- [ ] [[SESSION_CHARTER]] registra ordens e progresso
- [ ] Notas novas com frontmatter + ≥ 2 `[[links]]`
- [ ] MOCs das pastas impactadas atualizados
- [ ] Gates verdes: `go build / vet / test -race / integration / gosec -sev high / govulncheck`
- [ ] `tasks.md` reflete progresso
- [ ] Docs sincronizadas (README, OpenAPI, CHANGELOG)
- [ ] Dual-check aplicado em toda decisão não-trivial da sessão

---

## 16. Links essenciais do projeto

### Governança viva
- [[_moc]] — MOC do projeto
- [[notation-conventions]] — **glossário de siglas** (D-###/A-###/DT-###/ADR/INV/BR/GR/FR/M1-M3/severidades) — onboarding obrigatório para qualquer pessoa entrando, especialmente não-técnica
- [[STATE]] — decisões (D-###) e dívidas (DT-###)
- [[11-audit/AUDIT]] — findings (A-###)
- [[SESSION_CHARTER]] — ordens da sessão atual
- [[STEERING]] — não-negociáveis
- [[ROADMAP]] — marcos M1/M2/M3
- [[DoR-DoD]] — ready / done

### Contexto
- [[analise-crua]] — análise crua do código legado TS (matéria-prima histórica)
- [[_ingestion-log]] — trilha de auditoria do material ingerido
- [[_archive/2026-04-23-pre-v2-migration/_moc]] — legado arquivado
- [[client-canvas/_moc]] — 23 diagramas `.canvas` originais do João (fluxos + arquitetura + modelos), absorvidos 2026-04-24 do vault `Clients/MasterSindico/`. **Exceção formal kebab-case (D-vault-026)**: nomes preservam formato original do cliente para rastreabilidade ao vault `Clients/MasterSindico/`. Novos arquivos destilados nessa pasta DEVEM seguir kebab-case.

### Canônico (zero → prod)
- [[00-product/_moc]]
- [[01-domain/_moc]]
- [[02-architecture/_moc]]
- [[03-subprojects/_moc]]
- [[04-requirements/_moc]]
- [[05-tasks/_moc]]
- [[06-execution/_moc]]
- [[07-quality/_moc]]
- [[08-security/_moc]]
- [[09-testing/_moc]]
- [[10-schedule/_moc]]
- [[11-audit/_moc]]
- [[12-docs/_moc]]

### Vault geral (atemporal)
- [[../../CLAUDE]] — contrato raiz do vault
- [[../../10-knowledge/_moc]] — knowledge atemporal (architecture, patterns, principles, providers, security, testing, methodology, database, observability)
- [[../../20-stacks/_moc]] — stacks (go, typescript, postgres; flutter a criar)
- [[../../30-agents/playbooks/_moc]] — playbooks operacionais
- [[../../40-templates/_moc]] — templates
- [[../../60-sources/master-sindico-research/_toc]] — research destilado deste projeto
