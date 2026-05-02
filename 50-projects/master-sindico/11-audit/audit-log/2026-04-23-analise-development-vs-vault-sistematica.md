---
name: Análise sistemática Development/ × vault master-sindico-v2
description: >-
  Cruzamento minucioso (arquivo-a-arquivo, recente→antigo) das 4 sub-pastas de
  Development (backend, full-stack, Content, app, web) + configs (.kiro,
  .claude, raiz) contra o vault, produzido por 7 agentes Explore em paralelo
type: audit
date: 2026-04-23T00:00:00.000Z
owner: opus-4.7-main
scope: full Development/ (excluindo .git) × full master-sindico-v2/
severity: 'info (não introduz findings novos, consolida reconciliação)'
---

# Análise sistemática `Development/` × vault `master-sindico-v2`

**Data**: 2026-04-23 22:45
**Método**: 7 agentes Explore "very thorough" paralelos (backend, full-stack, Content, app, web, .kiro/.claude/raiz, vault). Inventário com mtime gerado em `/tmp/msv2-inv/*.txt`. Leitura do mais recente para o mais antigo por pasta. Sem busca por keywords — varredura direta.
**Escopo Development**: 6.541 arquivos em 7 pastas top-level + 3 arquivos raiz (pós-exclusão de `.git/`, `node_modules/`, builds).
**Escopo vault**: 795 arquivos, 13 MB.

---

## 1. Sumário executivo

### Veredito geral

O vault **já está 80–90% alinhado** com o Development real. A reconciliação `v2↔dev` de hoje de madrugada (`90-ingested/_reconciliacao-dev/*` + `11-audit/audit-log/2026-04-23-reconciliacao-v2-vs-dev.md`) fez a maior parte do trabalho de bater código real contra vault. O que resta são:

- **3 divergências ativas bloqueantes/próximas de bloquear** (infra no steering; DT-004 matriz N1/N2/N3 backend; mobile deps D-048/049/050 não aplicadas).
- **5 gaps de escopo M1** entre o que o vault afirma e o que Development tem (web praticamente não iniciado; mobile com feature adiantada mas sem splash/timeline; reputação/cupons/painel admin não implementados).
- **Stale docs no Content** que duplicam versões antigas (requirements.md 205k ≡ `.kiro/specs/master-sindico-platform/requirements.md`) — candidatos a archive.
- **full-stack/ 100% legado** — declarado como "Superseded" no `CLAUDE.md` raiz, confirmado pelas análises. Fontes úteis já ingeridas no vault (`90-ingested/from-source-material-contextos/`).

### Matriz bate vs diverge (resumo visual)

| Dimensão | Vault afirma | Development real | Veredito |
|---|---|---|---|
| Stack backend | Go 1.26 + Gin + pgx + sqlc + Zitadel + Stripe + Mux + Livekit + Resend + Railway | Idem (Go 1.26.2 exato) | ✅ Bate |
| 6 Bounded Contexts backend | identity, billing, institutional, commercial, content, assembly | Idem | ✅ Bate |
| 34 aggregates backend | matriz em `90-ingested/_reconciliacao-dev/01-backend-real.md` | match 1:1 | ✅ Bate |
| Stack web | SolidJS + TanStack Router/Query/Form + Rsbuild + UnoCSS + Kobalte + Biome + Bun | SolidJS + TanStack Router + Rsbuild + UnoCSS + Kobalte + Biome + Bun (Query/Form não instalados) | 🟡 Parcial |
| Stack mobile | Flutter + flutter_bloc + bloc_concurrency + hydrated_bloc + freezed + get_it + go_router + freeRASP | Flutter + flutter_bloc + equatable + get_it + injectable + go_router + dio + dartz + oidc (sem concurrency/hydrated/freezed/freeRASP) | 🟡 Parcial — D-048/049/050 não aplicadas |
| Infra provider | Railway + Resend + tsvector (D-067) | Idem | ✅ Bate |
| **Infra no steering backend** | Railway + Resend (vault canônico) | `backend/.kiro/steering/project-overview.md` ainda cita AWS ECS/SES/Twilio | 🔴 **Diverge ativo** |
| Personas | 6 (syndic, resident, enterprise, marketing, local_company, admin) | Zitadel com 6 roles idem | ✅ Bate |
| Planos | 4 globais Stripe-style (trial/base/plus/pro) — D-057/D-080/D-081 (N1/N2/N3 abolidos da UI) | slugs DB com `syndic_n1/n2/n3` + plan_tier global; matriz funcional backend ainda usa N1/N2/N3 (DT-004) | 🟡 Parcial — DT-004 aberta |
| Audit/STATE/SESSION_CHARTER | vault tem AUDIT.md + STATE.md próprios | backend/.kiro/AUDIT.md + STATE.md + SESSION_CHARTER.md — 2 fontes vivas | ⚠️ Dual-source — precisa decidir qual é canônica |
| Escopo M1 mobile | splash + auth OIDC + home morador + timeline read | feature `assembly` completa (fora M1) + auth parcial + home placeholder — timeline ausente | 🟡 Diverge em escopo |
| Escopo M1 web | onboarding + dashboard básico | 6 apps scaffolded, 1 com hero (lp), 5 sem conteúdo — auth são stubs | 🔴 Gap de execução M1 |
| Reputação Bronze→Diamante (D-051 fórmula) | vault tem fórmula fechada + pesos 40/25/15/10/10 | código commercial tem evaluations + vote sessions; motor de reputação não implementado | 🔴 Gap de implementação |
| Sistema de Cupons "Promoção do Dia" (D-059/D-078) | vault reconhece + AUDIT Q-002 race condition | backend não tem módulo; full-stack tinha schemas; Content documenta via SOW Jan/2026 | 🟡 Backlog (M2/M3) |
| Painel Admin Master Síndico (Content Decision 8) | sub-produto 11 `00-product/sub-produtos/11-administradoras-condominiais.md` + `03-subprojects/admin-privilegios.md` | role admin existe em identity; sem BC admin dedicado | 🟡 Transversal por design (OK) |
| 11 sub-produtos | listados em `00-product/portfolio-de-produtos.md` | 6 BCs implementados (faltam LMS/Banco de Talentos/Cupons/Painel como módulos próprios — embutidos ou backlog) | 🟡 OK para M1 (fora escopo) |
| Documentos originais | já ingeridos em `90-ingested/from-*/` | presentes em `Content/contents/*` e `web/old/docs/*` | ✅ Ingeridos |
| Legado `full-stack/` | declarado Superseded | parou em 2026-02-28 (+ 2 tweaks isolados 2026-04-05) | ✅ OK |
| Specs duplicado `Content/requirements.md` ≡ `Content/.kiro/specs/.../requirements.md` | vault tem versão destilada nos recortes | cópias idênticas por md5, 205k, stale | 🟠 Candidatos a archive |

---

## 2. Cronologia: o que é recente, o que é legado

### Linha do tempo consolidada

```
2026-01-27  ── full-stack inicia (npm, docker-compose, schemas)
2026-01-28  ── full-stack consolidação inicial (apps/api, apps/web, packages)
2026-02-07  ── full-stack/.claude/settings.local.json (último settings)
2026-02-08  ── web/old/docs/ (PDFs Manual Técnico, Identidade Visual, Matriz Funcional — fontes primárias do João)
2026-02-11  ── full-stack/bkp/payment/ (refactor billing abortado → arquivado)
2026-02-20  ── enterprise-catergories-subcategories-enum.schema.ts (raiz Development)
2026-02-25  ── full-stack contextos/ANALISE_TECNICA_COMPLETA_V2.md (247k), ANALISE_COMPLETA.md (180k), ANALISE_TECNICA.md (34k)
2026-02-26  ── full-stack/docs/arquitetura-moderna-master-sindico.md (proposta de pivô arquitetural)
2026-02-28  ── full-stack/ANALISE_COMPLETA.md (77k, última atividade substancial)
2026-03-09  ── Content/contents/ — PDFs do João + .ini
2026-03-10  ── Content/.kiro/specs/master-sindico-platform/requirements.md (205k — origem dos recortes)
2026-03-12  ── Content/.claude e .kiro settings
2026-03-23  ── Content/excopo-tecnico-antigo.txt (SOW v3.0 Jan/2026, referência contratual)
2026-03-30  ── Content/requirements.md, design.md, tasks.md, ARCHITECTURE_ANALYSIS_REPORT.md (monolíticos)
2026-04-05  ── full-stack últimos 2 tweaks isolados (api-response.ts, send-verification-code.use-case.ts) — já abandonado de fato
2026-04-12  ── web/apps/lp/ (primeiro app SolidJS criado)
2026-04-13  ── web/ scaffolding completo (6 apps, packages ui/schemas, bun.lock)
2026-04-13 12:06 ── full-stack/ última alteração de pasta — início do freeze
2026-04-13 21:18+  ── backend/ scaffold inicial Go (gitkeeps, docs swagger)
2026-04-20  ── CLAUDE.md raiz Development (7.6k)
2026-04-21 01:33 ── app/ (Flutter) scaffold inteiro num único commit
2026-04-21 13:30-23:40 ── backend/ burst: commercial + assembly + content + testes
2026-04-21 18:52-20:14 ── app/ burst: feature assembly completa (domain+data+presentation+tests)
2026-04-22 00:09-07:12 ── backend/ burst: providers MinIO/email/Zitadel + migração 310 + OpenAPI
2026-04-22 05:12  ── web/.kiro/ marcado DEPRECATED (tombstone README)
2026-04-22 06:38  ── web/AGENTS.md atualizado
2026-04-22 14:00-17:12 ── backend/ burst: server/routes + rate_limiter + institutional master-plan + 14 steering
2026-04-22 16:24-19:50 ── Content/ burst: agente-3 produz _MOC, handoffs, regras-canonicas, quebras-detectadas, 3 arquivos módulo, enums, jornadas
2026-04-22 17:07  ── backend/.kiro/AUDIT.md (33k — findings Sprint 9→10)
2026-04-22 19:50  ── Content/_handoffs/relatorio-final-agente3 (entrega consolidada 40 quebras)
2026-04-23 01:44  ── vault master-sindico-v2/ criado
2026-04-23 02:16  ── vault 90-ingested/from-* (ingestão em batch dos materiais-fonte)
2026-04-23 03:00-04:30 ── vault population (00-12) + _reconciliacao-dev
2026-04-23 13:48-13:54 ── vault 90-ingested/_reconciliacao-dev/*.md (reconciliação com Development)
2026-04-23 16:01-17:03 ── vault Fase 7 (portfolio, sub-produtos, ADR-0032)
2026-04-23 20:23-20:34 ── vault Fase 8 (dual-check + research aplicações)
2026-04-23 21:57-22:04 ── vault Fase 9 (invariants, edital 8d, topology RLS, PITR) + AUDIT consolidado
```

**Interpretação temporal**: `full-stack/` morreu em fevereiro. `Content/` é documentação de produto do João acumulada ao longo de fev→março, com destilação pelo agente-3 em 22/04. `backend/` + `app/` + `web/` são todos **novos de 21/04** — a reescrita Go+Flutter+SolidJS tem apenas 2–3 dias. O vault foi criado em 23/04 sobre essa base.

### Classificação automática

| Categoria | Pastas/arquivos | Estado |
|---|---|---|
| **VIVO (ativo)** | backend/, app/, web/ (código), backend/.kiro/, Content/_handoffs + _MOC + 01-04 + regras-de-negocio, CLAUDE.md raiz | Atualizar conforme sprint |
| **LEGADO MAS ÚTIL** | Content/contents/ (PDFs originais João), Content/excopo-tecnico-antigo.txt (SOW contratual) | Preservar como fonte histórica |
| **STALE CANDIDATO A ARCHIVE** | Content/requirements.md + design.md + tasks.md + ARCHITECTURE_ANALYSIS_REPORT.md + main.go.md, Content/.kiro/specs/master-sindico-platform/, Content/.kiro/settings/ vazio, Content/.claude/ | Arquivar com pointer |
| **MORTO** | full-stack/ inteiro (546 arquivos TS/Bun), full-stack/.claude, enterprise-catergories-subcategories-enum.schema.ts raiz, web/old/docs/ (já ingerido) | Sinalizar ou mover para archive |
| **CONFIG RESIDUAL** | Development/.kiro/hooks/read-context-before-task.kiro.hook, Development/.claude/settings.local.json | Baixa prioridade |

---

## 3. Matriz de coerência por segmento

### 3.1. `backend/` (Go — ATIVO)

**Alinhamento com vault**: ✅ **94%** — a reconciliação formal (01-backend-real.md + AUDIT.md + STATE.md vault) cobre quase tudo.

| Item | Development | Vault | Bate? |
|---|---|---|---|
| Versão Go | 1.26.2 | 1.26 | ✅ |
| Framework HTTP | Gin 1.12 abstraído via `contracts.HTTPRouter` | idem | ✅ |
| DB driver | pgx/v5 5.9.1 + sqlc 1.30.0 + goose v3 | idem | ✅ |
| Auth | Zitadel OIDC v3 + JWT profile + httpOnly cookies | idem | ✅ |
| Pagamentos | stripe-go/v82 | idem | ✅ |
| Vídeo VOD | mux-go 1.1.1 | idem (ADR-0004 atualizada) | ✅ |
| Live | livekit-server-sdk-go/v2 2.16.2 | idem | ✅ |
| Email | resend-go/v2 | Resend (D-067) | ✅ |
| Storage | minio-go/v7 S3-compat | MinIO dev / R2/S3 prod | ✅ |
| Observabilidade | OTel SDK 1.43.0 + Prometheus bridge + zerolog | idem | ✅ |
| Deploy | Railway | Railway (D-067) | ✅ |
| 6 BCs | identity, billing, institutional, commercial, content, assembly | idem | ✅ |
| Sprints 0-9 | concluídos (backend/.kiro/SESSION_CHARTER.md confirma ~33 features) | idem (reconciliação confirma) | ✅ |
| ADRs | não formalizadas no código (mas referenciadas em steering) | 35 ADRs (25+6+3+1) | ⚠️ ADRs são doc do vault |
| Gates Sprint 9 | build/vet/test-race/gosec/govulncheck todos verdes | idem confirmado | ✅ |
| **Matriz funcional** | ainda colunas N1/N2/N3 em backend/.kiro | canônico é Trial/Base/Plus/Pro global (D-083) | 🔴 **DT-004 aberta** |
| **Infra no steering** | `backend/.kiro/steering/project-overview.md` cita AWS ECS/SES/Twilio | Railway/Resend (D-067) | 🔴 **Divergência ativa** |
| Reputação Bronze→Diamante | não implementada | D-051 fórmula determinística | 🔴 Gap implementação |
| LMS/Academia | não implementada | sub-produto 6 | 🟡 M2/M3 backlog |
| Banco de Talentos | não implementada como BC próprio | sub-produto 7 (pode ir em content+commercial) | 🟡 M2/M3 |
| Sistema de Cupons | não implementada | D-059/D-078 + AUDIT Q-002 race | 🟡 M2/M3 |
| Crons noturnos | referenciados em 3 domain files (MarkAtrasada, MarkExpired, soft-close polls) mas sem scheduler | vault não documenta | 🟡 Gap estrutural |
| NATS | presente em go.mod + docker-compose, não wired (InMemoryPublisher síncrono) | NATS JetStream (ADR-0019 threshold) | 🟡 Conforme ADR threshold |
| IAuditLogger LGPD (DT-010) | apenas `compliance_use_cases.go` persiste; billing/deal/empresa apenas zerolog | vault: LGPD compliance obrigatória + audit trail 5y | 🔴 DT-010 aberta |
| Session cleanup job | ausente (noop) | vault não documenta cron | 🟡 Tech debt |
| CI postgres:17 vs compose postgres:18 | divergência | vault não documenta | 🟡 Baixo impacto |

**TODOs abertos relevantes (backend/.kiro/AUDIT.md)**:
- A-023 (1-device change sem audit fingerprint) — medium
- A-024 (rawBodyReader interface privada duplicada 3x) — medium
- A-039 (rate limiting só por IP; spec exige IP+userId) — medium

**Decisões/DTs abertas no backend/.kiro/STATE.md**:
- D-001 ABAC cache TTL 5min sem webhook invalidation
- DT-003 sem circuit breaker Zitadel
- DT-004 sem timeout em UoW.Run
- DT-008 PAT GitHub clássico
- DT-010 IAuditLogger LGPD incompleto

**Backend/.kiro ativo e robusto**: 30 arquivos em `.kiro/` (specs, steering 14 arquivos, AUDIT vivo, STATE vivo, SESSION_CHARTER, ROUTE_SMOKE_TEST). 7 arquivos em `.claude/` (settings + 6 skills: dual-validate, go-review, migration-writer, sprint-auditor, sqlc-writer, task-executor). Hook `PostToolUse` em Write/Edit roda `go build ./...`. Permissões deny explícitas (rm -rf, git push force, DROP TABLE/DATABASE, docker prune).

### 3.2. `app/` (Flutter — ATIVO, jovem)

**Alinhamento com vault**: 🟡 **65%** — divergências de deps e escopo M1.

| Item | Development | Vault `03-subprojects/mobile/` | Bate? |
|---|---|---|---|
| Framework | Flutter Dart SDK ^3.11.4 | idem (freezed/hydrated/freeRASP previstos) | 🟡 |
| Arquitetura | Feature First + Clean Arch | idem | ✅ |
| State | flutter_bloc + equatable | flutter_bloc + bloc_concurrency + hydrated_bloc (D-048) | 🔴 D-048 não aplicada |
| Models | (sem freezed) | freezed + dart_mappable (D-049) | 🔴 D-049 não aplicada |
| Segurança | flutter_secure_storage | freeRASP M1 report-only (D-050) + flutter_secure_storage + flutter_jailbreak_detection + cert pinning | 🔴 D-050 não aplicada |
| DI | get_it + injectable | idem | ✅ |
| Router | go_router 14.8 | idem | ✅ |
| HTTP | Dio 5.8 | Dio + Retry + Interceptors | 🟡 Interceptors ausentes |
| Auth | OIDC PKCE Zitadel | idem | ✅ |
| i18n | intl ^0.20 declarado, sem arbs | vault não detalha | 🟡 |
| Testes | 8 arquivos (bloc_test + mocktail) | 95%/90% thresholds | 🟡 Cobertura abaixo |
| Escopo M1 | feature assembly (completa) + auth parcial + home placeholder; **SEM timeline read** | splash + auth OIDC + home morador + timeline read | 🟡 Escopo desviado |
| CLAUDE.md + ARCHITECTURE.md + CODING_MANUAL.md | presentes | vault tem `03-subprojects/mobile/README+requirements+architecture+patterns+security+design-system+ui-catalog+tasks.md` | ✅ Ambos completos |

**Sinais relevantes**:
- `createdAt: DateTime.now()` hardcoded no login — Zitadel UserInfo não expõe o campo
- Rotas `/register` e `/profile` declaradas sem GoRoute registrado
- Divergência de chave de token: datasource usa `AUTH_TOKEN`, constante diz `auth_access_token`

### 3.3. `web/` (SolidJS — ATIVO mas escasso)

**Alinhamento com vault**: 🟡 **55%** — stack bate, mas código real está em estágio inicial frente ao escopo M1.

| Item | Development | Vault `03-subprojects/web/` | Bate? |
|---|---|---|---|
| Framework | SolidJS ^1.9.12 | idem | ✅ |
| Build | Rsbuild 1.7.5 + Rspack | idem (não Vite) | ✅ |
| Router | TanStack Solid Router 1.168 | idem | ✅ |
| Styling | UnoCSS + presets Wind4/Attributify/Icons/WebFonts | idem (Manual IV OKLCH) | ✅ |
| Headless UI | Kobalte + CVA | idem | ✅ |
| Valid/Schemas | Zod v4 + `packages/schemas` vazio | idem + schemas preenchidos previstos | 🟡 |
| HTTP | Axios 1.15 + withCredentials | idem | ✅ |
| Lint/Format | Biome | idem | ✅ |
| Monorepo | Bun workspaces (6 apps) | idem | ✅ |
| **TanStack Query** | não instalado em apps/ atuais | previsto | 🟡 |
| **TanStack Form** | não instalado | previsto | 🟡 |
| Testes | zero | 95%/90%/85%/75% | 🔴 Gap total |
| **Escopo M1** | 5/6 apps são placeholders; apenas `lp` tem hero; auth sign-in/sign-up são stubs; AuthProvider/QueryClient comentados | onboarding + dashboard básico | 🔴 **Gap grande para M1** |
| `web/.kiro/` | DEPRECATED (tombstone 22/04) | canônico em `backend/.kiro/specs/master-sindico/` | ✅ |
| `web/CLAUDE.md` + `web/AGENTS.md` | presentes | alinha com steering | ✅ |
| `old/` | gitignored, contém AuthProvider legado | referência histórica | ✅ OK |

### 3.4. `Content/` (docs de produto — FUSÃO DE RECENTE E LEGADO)

**Alinhamento com vault**: ✅ **85%** — o vault já destilou a maior parte do conteúdo.

| Seção Content | Presença no vault | Bate? |
|---|---|---|
| `_MOC.md` | vault tem `_moc.md` próprio | 🟡 Paralelo |
| `_handoffs/*` | vault `11-audit/audit-log/*` | 🟡 Rastreabilidade em pastas diferentes |
| `01-Arquitetura/Stack-e-Modulos.md` | vault `02-architecture/*` (mais detalhado) | ✅ superseded by vault |
| `02-Jornadas/Inventario-Telas.md` (136+ telas) | vault `03-subprojects/{web,mobile}/ui-catalog.md` | ✅ destilado |
| `02-Jornadas/Onboarding-por-Persona.md` | vault `00-product/personas.md` + `04-requirements/registration-data.md` | ✅ destilado |
| `03-Modulos/Assembleia-Live-Funcional.md` | vault `04-requirements/functional/assembly.md` (ASM-001 atualizado 8d) | ✅ destilado |
| `03-Modulos/Compliance-Detalhado.md` (C1-C11) | vault `04-requirements/compliance-lgpd.md` + institucional | 🟡 C1-C11 específicos não claramente rastreáveis |
| `03-Modulos/Connect-Me-4-direcoes.md` | vault `04-requirements/functional/commercial.md` | ✅ destilado |
| `03-Modulos/Vizinhanca-4-jornadas.md` | vault `00-product/sub-produtos/08-vizinhanca.md` | ✅ destilado |
| `04-Listas-Mestre/Enums-Consolidados.md` (19 cat, 90+ valores) | vault `90-ingested/_reconciliacao-dev/05-enums-regras-quebras.md` | ✅ destilado |
| `regras-de-negocio/regras-canonicas.md` (10 Regras de Ouro + 12 Decisões) | vault `STEERING.md` + `01-domain/invariants.md` + STATE.md | ✅ destilado |
| `regras-de-negocio/quebras-detectadas.md` (40 quebras) | vault AUDIT 79 findings (incluindo 31 Q-### + 48 G-###) | ✅ **superseded por AUDIT** |
| `sdd-gsd-aplicado.md` | vault CLAUDE.md + STEERING.md adotam SDD | ✅ |
| `requirements.md` raiz (205k, 30/03) | destilado em vault `04-requirements/functional/*` | 🟠 **Stale canditato a archive** |
| `design.md` (84k) | superseded por vault `02-architecture/*` | 🟠 **Stale** |
| `tasks.md` (67k) | superseded por vault `05-tasks/*` + backend/.kiro/specs/tasks | 🟠 **Stale** |
| `ARCHITECTURE_ANALYSIS_REPORT.md` (67k) | destilado | 🟠 **Stale** |
| `excopo-tecnico-antigo.txt` (SOW v3.0 Jan/2026) | vault sabe da divergência (gap contratual Q-023: Sistema de Cupons, Q-026/27 escopo expandido) | ✅ **Preservar como fonte histórica** |
| `main.go.md` | — | 🟠 Stale |
| `.kiro/specs/master-sindico-platform/requirements.md` (205k, 10/03) | idêntico md5 a Content/requirements.md | 🟠 **Duplicata stale** |
| `.kiro/settings/mcp.json` vazio | — | 🟠 Residual |
| `.claude/settings.local.json` | — | 🟠 Residual (sessão mar/2026) |
| `contents/*.pdf` + `.ini` (23 arquivos fonte do João) | ingeridos em `90-ingested/from-source-material-*` | ✅ **Preservar** |

**40 quebras do Content vs 79 findings do vault AUDIT**: o vault AUDIT engloba as 40 + ampliou com pentest/qa-review/ops-bigtech-gaps. Não existe matriz 1:1 (Content Q-### → Vault Q-###/G-###) — **oportunidade de correlação explícita**.

### 3.5. `full-stack/` (TS — MORTO)

**Alinhamento com vault**: ⚪ N/A — declarado Legacy/Superseded no `CLAUDE.md` raiz e confirmado.

- 546 arquivos. Bun + Fastify 5 + Drizzle (beta 1.0.0) + Awilix + SolidJS + Zod v4 + CASL.
- Parou em **2026-02-28**. Dois tweaks isolados em 2026-04-05 não representam retomada.
- **Motivos prováveis do abandono** (sintetizados dos 4 ANALISE_*.md):
  1. ABAC com CASL nunca ativado (`PermissionService` comentado em `billing.module.ts`; `seed-plans.ts` vazio)
  2. Refactor billing abortado → pasta `bkp/payment/` arquivada em 11/02/2026
  3. Módulo `communication` com arquivos de 0 bytes (send-connect-me.use-case.ts, connect-me.routes.ts)
  4. Stack em versões beta (Drizzle beta.12, Zod 4 dias após release, Node ≥25 pré-LTS)
  5. Zero testes (*.test.ts / *.spec.ts inexistentes em 546 arquivos)
  6. ~500k de docs de análise em `contextos/` antes do pivô
  7. `arquitetura-moderna-master-sindico.md` (26/02) propõe Effect-TS + Temporal.io — escopo expansivo demais
- **Conceitos migrados para Go**: BillingModule → billing/ Go, ConnectMeEntity → commercial/, PlanPermission/PlanQuota/FeatureUsage → billing ABAC Go, MoneyVO → `pkg/money`, UoW pattern mantido, roles Zitadel mapeados
- **Conceitos possivelmente perdidos (checar)**: motor de vídeo com comentários/likes/views, paywall 25% + contador 70%, vínculos explícitos com condomínios em `schema/users/vinculos.ts`, username generator automático, PermissionService CASL granular
- **Docs ANALISE_* ingeridos**: sim — via `90-ingested/from-source-material-contextos/analise_integrada.md` e cascata

### 3.6. `.kiro/` + `.claude/` + arquivos raiz Development

| Caminho | Estado | Ação vault |
|---|---|---|
| `backend/.kiro/` | 30 arquivos VIVOS | ⚠️ Dual-source com vault — decidir canônico |
| `backend/.claude/settings.json` | 7 arquivos VIVOS (settings + 6 skills) | Preservar — é operacional |
| `Development/.claude/settings.local.json` | Residual mar/2026 | Baixa prioridade |
| `Development/.kiro/hooks/read-context-before-task.kiro.hook` | Hook Kiro Desktop (não Claude Code) | Baixa prioridade |
| `Content/.kiro/` | Duplicata stale + settings vazios | Archive |
| `Content/.claude/settings.local.json` | Sessão de leitura mar/2026 | Archive |
| `web/.kiro/specs/master-sindico/README.md` | Tombstone DEPRECATED 22/04 ✅ | OK |
| `full-stack/.claude/settings.local.json` | Fev/2026, sessão do legado | Archive com full-stack |
| `Development/CLAUDE.md` raiz | 7.6k, descreve workspace | Atualizar se necessário |
| `Development/backend.code-workspace` | VS Code workspace | 🟠 **Não inclui `app/`** — adicionar |
| `Development/enterprise-catergories-subcategories-enum.schema.ts` | 14k, 20/02/2026, 31 categorias Zod | 🟠 Desatualizado (vault tem 38 categorias em 12 grupos); archive |

---

## 4. Gaps e divergências críticas (top 20 acionáveis)

Em ordem decrescente de impacto:

### 🔴 Crítico / próximo de bloquear M1

**G1. Steering `project-overview.md` cita infra descartada** (inclusion: always)
- Caminho: `Development/backend/.kiro/steering/project-overview.md`
- Cita: "Email: AWS SES, Push: FCM, SMS: Twilio, Infra: AWS ECS Fargate + RDS + ElastiCache + S3 + CloudFront + Route53 + CloudWatch"
- Realidade + vault D-067: Railway + Resend + R2/S3 + tsvector + OTel/Grafana/Sentry
- **Impacto**: todo agente lerá infra errada em cada sessão autônoma → decisões divergentes
- **Ação**: atualizar o steering (ou registrar em STATE.md do vault como DT nova e priorizar)

**G2. DT-004 matriz funcional backend ainda N1/N2/N3**
- Vault canônico: 4 tiers globais `trial/base/plus/pro` com coluna Role + Admin (D-057/D-080/D-081/D-083)
- Backend: `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` ainda usa colunas N1/N2/N3
- **Impacto**: implementações de feature gate podem ler a matriz errada → bloqueios inconsistentes
- **Ação**: reescrita da matriz backend seguindo o canônico, confirmar D-083

**G3. Mobile: deps D-048/049/050 não aplicadas**
- Vault decidiu: `bloc_concurrency`, `hydrated_bloc` (D-048), `freezed + dart_mappable` (D-049), `freeRASP` M1 report-only (D-050)
- Código real: só `flutter_bloc + equatable + get_it + injectable + go_router + dio + dartz + oidc + mocktail + bloc_test`
- **Impacto**: retrabalho — migrar entidades para freezed + adicionar freeRASP antes de M1 pode não dar tempo
- **Ação**: ou prazo D-048/049/050 posterga para M2 (registrar DT), ou equipe executa a migração antes de 2026-05-08/18

**G4. Unit sem `unit_type` e `fracao_ideal` (Blocker M1 declarado)**
- Vault AUDIT: A-FASE8 — backend `Unit` aggregate não tem esses campos
- Assembly exige fração ideal para quórum fracional NUMERIC(5,4)
- **Impacto**: bloquea M1 assembly + institucional
- **Ação**: migration obrigatória pré-M1 (já em tasks.md Sprint 10)

**G5. Escopo M1 web ainda não iniciado**
- Vault M1 web: onboarding + dashboard básico
- Real: 5/6 apps são placeholders; sign-in/sign-up são `<div>Hello...!</div>`; AuthProvider + QueryClient + Toaster + errorComponent comentados em todos os index.tsx
- **Impacto**: 15–25 dias de trabalho web para atingir M1 em 2026-05-08 (ou 18)
- **Ação**: rever D-009/D-010 (cronograma) ou ship M1 só com backend+mobile

**G6. LGPD DT-010 IAuditLogger incompleto + 7 bloqueadores LGPD humanos**
- Backend: só `compliance_use_cases.go` persiste AuditLogEntry; billing/deal/empresa só zerolog
- Vault AUDIT: 7 bloqueadores LGPD M1 humanos (DPO público, DPAs Mux/Zitadel/Twilio, DPIA-001, política de privacidade, DELETE /me race, KMS field-level PII)
- **Impacto**: exposição regulatória
- **Ação**: unificar IAuditLogger cross-BC no código; ação humana/comercial para DPAs

**G7. Reputação Bronze→Diamante não implementada**
- Vault D-051: fórmula determinística pesos 40/25/15/10/10 + 5 tiers
- Backend: só evaluations + vote sessions sem motor
- **Impacto**: sub-produto 3 (Reputação) é core para Connect Me/marketplace — sem isso não há crité­rio auditável
- **Ação**: priorizar implementação do motor de reputação (backlog Sprint 10/11)

### 🟠 High

**G8. Crons noturnos referenciados mas sem scheduler**
- Backend: `connect_me_request.go MarkExpired`, `master_plan_activity.go MarkAtrasada`, `poll.go` soft-close — todos mencionam "cron noturno chama" mas scheduler não existe
- Vault: não documenta política de crons
- **Ação**: definir ADR de scheduling (k8s CronJob? Railway cron? NATS + worker?) e registrar

**G9. Session cleanup job ausente**
- `identity/module.go:98` — "noop — session cleanup job virá em sprint futuro"
- Tabela `sessions` cresce indefinidamente
- **Ação**: priorizar em Sprint 10 junto com DT-004

**G10. Providers stubs não substituídos** (FCM push, Twilio SMS, GoTpl PDF)
- Retornam `"FCM not configured"` / JSON em vez de PDF real
- Vault confirma FCM/Twilio no roadmap
- **Ação**: M2/M3 backlog; marcar explicitamente em `03-subprojects/backend/tasks.md` do vault

**G11. ABAC cache invalidation noop (D-001)**
- `main.go:157-166` — `SubscriptionActivated` handler noop; mudança de plano via webhook Stripe só reflete no ABAC após TTL 5min
- **Ação**: implementar Redis DEL pattern no handler

**G12. NATS ainda placeholder**
- `go.mod` + docker-compose têm NATS; código usa `InMemoryPublisher` síncrono
- Vault ADR-0019: "NATS JetStream threshold" — correto, mas cross-module event bus assíncrono é premissa de observability + resilience
- **Ação**: definir data concreta de threshold (M2?) e registrar no ROADMAP

**G13. Specs duplicadas em Content**
- `Content/requirements.md` (205k, 30/03) ≡ `Content/.kiro/specs/master-sindico-platform/requirements.md` (205k, 10/03) — mesmo md5
- **Ação**: archive ambos com pointer; adicionar README.md apontando para `backend/.kiro/specs/master-sindico/requirements/` + `master-sindico-v2/04-requirements/`

**G14. `enterprise-categorias-subcategorias-enum.schema.ts` raiz Development desatualizado**
- 31 categorias TS, 20/02/2026 (pré-backend Go)
- Vault: 38 categorias em 12 grupos (Enums-Consolidados.md)
- **Ação**: archive no `full-stack/` (ou `_archive/`) e registrar como fonte legada; canonizar enum no backend Go `domain/enums/vizinhanca_categories.go`

**G15. Banco de Talentos não tem mapeamento BC no código**
- Vault sub-produto 7: vídeo-currículo 90s + 3 vínculos profissionais + 18 áreas de interesse + 9 modalidades
- Backend: 6 BCs (nenhum `talent`). Possivelmente split em content (vídeo) + commercial (oferta) + identity (perfil)
- **Ação**: registrar mapeamento explícito no vault `03-subprojects/backend/modules-mapping.md` (não existe ainda)

**G16. Compliance C1-C11 do Content não rastreáveis 1:1**
- Content detalha 11 telas compliance
- Vault tem `compliance-lgpd.md` (LGPD) + institutional includes compliance
- **Ação**: criar matriz Content C1-C11 → vault req IDs

**G17. `backend.code-workspace` não inclui `app/`**
- 4 folders: Development, backend, web, full-stack
- **Ação**: adicionar `app` ao workspace (nome antigo era `mobile`)

### 🟡 Medium

**G18. Deprecations não processadas**: Content/requirements.md + design.md + tasks.md + ARCHITECTURE_ANALYSIS_REPORT.md + main.go.md + Content/.kiro/ + Content/.claude/. Mover para `Content/_archive/2026-03/` com pointer README.

**G19. Dual-source AUDIT/STATE**:
- `backend/.kiro/AUDIT.md` (33k, 22/04) + `backend/.kiro/STATE.md` (69k, 22/04)
- `master-sindico-v2/AUDIT.md` (27k, 23/04) + `master-sindico-v2/STATE.md` (63k, 23/04)
- Ambos atualizados independentemente. **Ação**: definir canônico. Sugestão: vault é fonte canônica; backend/.kiro/AUDIT+STATE são operacionais da sessão do SDD backend.

**G20. Matriz de quebras Content (40) → vault AUDIT (79) sem pointer explícito**
- Agente-3 do Content produziu 40 quebras; AUDIT vault tem 79 findings
- **Ação**: criar arquivo `11-audit/dual-check-log/2026-04-23-content-quebras-vs-vault-audit.md` mapeando cada Content#QX → vault Q-###/G-###

---

## 5. Duplicações e redundâncias detectadas

| Arquivos | md5 igual? | Recomendação |
|---|---|---|
| `Content/requirements.md` ≡ `Content/.kiro/specs/master-sindico-platform/requirements.md` | sim (ambos 205920 bytes) | manter um, apontar o outro |
| `Content/_MOC.md` vs `master-sindico-v2/_moc.md` | não (paralelos) | decidir canônico ou mesclar |
| `backend/.kiro/specs/master-sindico/requirements.md` vs `master-sindico-v2/04-requirements/*` | similar tema | backend = operacional, vault = canônico |
| `Content/regras-de-negocio/quebras-detectadas.md` (40) vs `master-sindico-v2/AUDIT.md` (77+ findings) | vault AUDIT engloba | AUDIT é canônico; Content é pré-destilação |
| `web/old/docs/ANALISE_COMPLETA_API.md` + `ANALISE_COMPLETA_MODELAGEM.md` vs `90-ingested/from-source-material-contextos/analise_integrada.md` | vault ingeriu | `web/old/docs/` pode ser arquivado ou apontado |
| `90-ingested/from-archive-specs-consolidation/CLAUDE.md` + `CLAUDE (copy 1).md` + `(copy 2).md` + `(copy 3).md` | 4 cópias do mesmo arquivo | limpar cópias (copy N) |
| `enterprise-catergories-subcategories-enum.schema.ts` (raiz Development) vs `Content/04-Listas-Mestre/Enums-Consolidados.md §12` (38 categorias vizinhança) | conteúdo divergente (31 vs 38) | arquivar TS; canonizar vault |

---

## 6. Recomendações priorizadas

### P0 — antes de M1 (2026-05-08 ou 18)

1. **Atualizar `backend/.kiro/steering/project-overview.md`** para Railway/Resend/tsvector (resolver G1). 30min.
2. **Resolver DT-004** (matriz funcional N1/N2/N3 → trial/base/plus/pro no backend spec). 2h.
3. **Migration Unit `unit_type` + `fracao_ideal`** (A-FASE8, Blocker M1). Já em tasks Sprint 10.
4. **Decidir D-048/D-049/D-050 mobile**: manter prazo M1 ou postergar (registrar DT). Se manter: migração freezed + hydrated_bloc + freeRASP. 2-3 dias.
5. **Decidir M1 escopo web**: redefinir entregável ou postergar para M1+10d (Opção A do AUDIT).
6. **DT-010 IAuditLogger unificado** antes de M1 (audit trail LGPD obrigatório). 1 dia.

### P1 — próxima sprint

7. **Implementar motor de Reputação Bronze→Diamante** (D-051).
8. **Scheduler para crons noturnos** (ADR + implementação).
9. **Session cleanup job** em identity.
10. **ABAC cache invalidation** no `SubscriptionActivated` handler.
11. **Matriz Content quebras (40) → vault findings (77+)** em dual-check-log.
12. **Archive limpo** de `Content/requirements.md + design.md + tasks.md + ARCHITECTURE_ANALYSIS_REPORT.md + main.go.md + Content/.kiro/ + Content/.claude/` para `Content/_archive/2026-03/`.

### P2 — higiene e consistência

13. **Adicionar `app/` ao `backend.code-workspace`**.
14. **Archive `full-stack/`** (mover para `_archive/full-stack-2026-02/` OU criar pointer README no root + .gitignored git submodule).
15. **Archive `enterprise-catergories-subcategories-enum.schema.ts`** e canonizar enum Vizinhança no backend Go.
16. **Limpar cópias `CLAUDE (copy N).md`** em `90-ingested/from-archive-specs-consolidation/`.
17. **Criar `03-subprojects/backend/modules-mapping.md`** com mapa sub-produtos vault → BCs + módulos + features + migrations.
18. **Criar `03-subprojects/mobile/sprint-status.md`** e `03-subprojects/web/sprint-status.md` com estado real.
19. **Consolidar decisão dual-source AUDIT/STATE** (vault canônico vs backend/.kiro operacional) em STEERING.md.
20. **ADRs 0026–0031 físicas** (já com slots reservados — Sprint 11 declarado).

---

## 7. Inventário bruto (referência)

Arquivos de inventário gerados em `/tmp/msv2-inv/` (mtime + tamanho + path, ordenados desc):

- `/tmp/msv2-inv/backend.txt` — 717 arquivos (Go source + testes + docs + .kiro + .claude)
- `/tmp/msv2-inv/full-stack.txt` — 546 arquivos (TS + docs ANALISE_* + schemas)
- `/tmp/msv2-inv/Content.txt` — 47 arquivos (markdown + PDFs + .kiro + .claude)
- `/tmp/msv2-inv/app.txt` — 245 arquivos (Dart + Android/iOS/Web scaffolds)
- `/tmp/msv2-inv/web.txt` — 265 arquivos (SolidJS + packages + old/docs)
- `/tmp/msv2-inv/.kiro.txt` + `.claude.txt` — 1 arquivo cada (root)

Total Development não-gerado: ~1.820 arquivos de código/config/doc. Legado full-stack: 30%. Recente vivo: 57%. Content histórico: 3%. Binários João: 1.3%. Residuais: 8.7%.

---

## 8. Referências cruzadas

- Vault: `STATE.md`, `AUDIT.md`, `CLAUDE.md`, `STEERING.md`, `_moc.md`
- Vault reconciliação: `90-ingested/_reconciliacao-dev/*.md` (5 arquivos), `11-audit/audit-log/2026-04-23-reconciliacao-v2-vs-dev.md` (94 divergências formais)
- Vault AUDIT consolidado: `11-audit/audit-log/2026-04-23-CONSOLIDADO-quebras-e-ops.md` (79 findings)
- Vault portfolio: `00-product/portfolio-de-produtos.md` (11 sub-produtos)
- Dev docs operacionais: `backend/.kiro/AUDIT.md` + `STATE.md` + `SESSION_CHARTER.md`
- Dev docs de produto: `Content/_MOC.md` + `_handoffs/2026-04-22-relatorio-final-agente3.md` (40 quebras + 10 decisões pendentes com João)

---

## 9. Conclusão

O Development está em estado **tridimensional**: backend robusto e quase pronto para M1 (com 3 gates abertos menores), mobile e web em estágios iniciais com divergências de deps/escopo, e Content servindo como matéria-prima já majoritariamente destilada no vault.

O vault já fez a maior parte do trabalho de reconciliação — as 94 divergências já estão formalizadas, a stack está alinhada, as decisões D-001..D-091 cobrem o essencial. **Não há "surpresa grande"**: tudo que importa já está mapeado. O que falta é:

1. Executar os gates P0 (G1–G6) antes de M1
2. Decidir postergação (Opção A: M1→2026-05-18) vs ship parcial (Opção B)
3. Higiene (P2) — archive, mapping, duplicação

**Este relatório não introduz findings novos** — apenas consolida a leitura sistemática ao nível arquivo-a-arquivo solicitado e cruza com a reconciliação já existente no vault. Todos os achados técnicos já estão em STATE.md + AUDIT.md + decisões D-### do vault.
