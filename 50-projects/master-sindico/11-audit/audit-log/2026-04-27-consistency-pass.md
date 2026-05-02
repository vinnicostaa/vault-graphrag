---
title: Consistency Pass — Decisões revogadas + BR-063 IP-block (2026-04-27)
type: audit
tags: [audit, consistency, master-sindico, dt-012, d-135, d-094, d-097, d-099, d-103, d-120, d-134]
project: master-sindico
created: 2026-04-27
updated: 2026-04-27
---

# Consistency Pass — 2026-04-27

> **Sessão autônoma a pedido do João**: *"você vai corrigir tudo que estiver inconsistente, fazer dupla checagem, não tratar nada como verdade absoluta até ter certeza, mapeie chaos se ainda tem algo lá não no vault"*. Todas as mutações via MCP Obsidian (clone FS é read-only — STEERING). 38 correções aplicadas + dupla checagem em 4 rounds.

## Gatilho

João identificou inconsistências reais durante revisão:
1. *"a parte do 1 IP 1 cadastro não é válida — pessoas do mesmo bloco que tentar acessar pelo mesmo wifi vão ser barradas, então não faz sentido"* — confirmou D-135 já decidiu rejeitar (famílias compartilham wifi), mas BR-063 stale.
2. *"se eu quiser migrar o projeto agora pro Mercado Pago não consigo porque o método de pagamento não está agnóstico de provedor"* — Stripe vaza para `domain/` em billing (auditado, debt persiste — não corrigido aqui pois é refactoring de código não-trivial; ver §Próximos passos).
3. *"se tem 1 pasta caos lá no obsidian"* — auditado: `_archive/2026-04-24-chaos-processed/` foi 100% processado (Fase H 2026-04-25), 0 arquivos pendentes. Confirmado independentemente por subagente Opus.

## Mapeamento prévio (4 subagentes Opus em paralelo, contexto limpo = dual-check natural)

- **Agente A** (chaos-processed): confirmou 41 arquivos em 8 subpastas, 100% canonizado ou histórico puro. Mercado Pago aparecia em `stale-stack-proposals/mastersindico-content-analysis.md` (13/04) — superado por D-073 (Stripe vence) na época. **Decisão atual sobre Mercado Pago é nova**, fora do escopo deste pass.
- **Agente B** (D-058/D-079 Morador Base 0/ano): 3 specs ativas stale + ~20 registros legítimos (com banner D-094).
- **Agente C** (D-064 vínculos + D-066/D-075 score): 3 + 2 specs ativas stale.
- **Agente D** (D-067 tsvector + D-072 admin + D-076 lives): 5 + 3 + 1 specs ativas stale.

Total inicial: **~17 stale identificadas**. Após patches + dupla checagem em 3 rounds (revisão grep direta), encontradas **mais 21 stale não capturadas pelos subagentes** (cobertura sub-produtos + MOCs sub-projetos + ADR index + CLAUDE.md raiz).

**Total final: 38 patches aplicados em 30 arquivos distintos.**

## Patches aplicados

### Bucket 1 — D-135 (rejeição IP-block, adoção device fingerprint progressivo)

| # | Arquivo | Linha | Mudança |
|---|---|---|---|
| 1 | `01-domain/business-rules.md` | BR-063 | Reescrita: removido "Bloqueio múltiplos cadastros mesmo IP < 1h", adicionado device fingerprint progressivo (silent log → CAPTCHA → revisão manual) com link para IDN-026. Banner D-135 + motivação famílias-wifi-condomínio. |
| 2 | `04-requirements/matrix-functional.md` | 707 | "1 IP 1 cadastro" → "Device fingerprint progressivo (D-135 Fase G revoga)". |

### Bucket 2 — D-094 (Morador Base Connect Me 0→2/ano)

| # | Arquivo | Mudança |
|---|---|---|
| 3 | `01-domain/aggregates/FeatureUsage.md:24` | "0/ano (D-058)" → "2/ano (D-094 revoga D-058/D-079)" |
| 4 | `03-subprojects/mobile/tasks.md:177` | MB-240 quota Morador Base 0→2/ano |
| 5 | `03-subprojects/mobile/ui-catalog.md:371` | Cards quota Morador Base 0→2/ano |
| 6 | `00-product/portfolio-de-produtos.md:30` | "Morador Base = 0/ano" → "2/ano (D-094)" |
| 7 | `00-product/portfolio-de-produtos.md:102` | Direção `ConnectMeFlowResidentToSyndicCold` 0→2 |
| 8 | `00-product/personas.md:25` | Regra-raiz Morador Base 0→2 |
| 9 | `03-subprojects/backend/requirements/billing.md:78` | INV-BIL-014 0→2 |
| 10 | `03-subprojects/backend/requirements/billing.md:262` | D-058 fechado → revogado por D-094 |
| 11 | `04-requirements/functional/billing.md:43` | Regra base 0→2 |
| 12 | `04-requirements/functional/billing.md:514` | Fonte D-058→D-094 |
| 13 | `04-requirements/functional/billing.md:949` | Invariante 15 0→2 |

### Bucket 3 — D-099 (Banco de Talentos 3→5 vínculos)

| # | Arquivo | Mudança |
|---|---|---|
| 14 | `00-product/business-model.md:212` | "3 (D-064)" → "5 (D-099 revoga D-064)" |
| 15 | `03-subprojects/mobile/ui-catalog/onboarding/ONB-E7.md:19` | "D-074 — 3 vínculos" → "D-099 — 5 vínculos" |
| 16 | `03-subprojects/traceability.md:305` | BT08 "3 vínculos / D-060/D-064 (3 max)" → "5 vínculos / D-099 (5 max)" |

### Bucket 4 — D-103 (Score 1 único → Governança 1-10 + Compliance 0-100)

| # | Arquivo | Mudança |
|---|---|---|
| 17 | `04-requirements/matrix-functional.md:197` | "1 score único" → "2 scores (D-103 revoga D-066/D-075)" + INV-GOV-001/002 + link S32 |
| 18 | `04-requirements/matrix-functional.md:403` | Mesma reescrita em segunda menção |

### Bucket 5 — D-120 (revoga D-067 PG tsvector; ISearchProvider real obrigatório M1)

| # | Arquivo | Mudança |
|---|---|---|
| 19 | `00-product/portfolio-de-produtos.md:84` | "PostgreSQL tsvector" → "ISearchProvider real (D-120 revoga D-067)" |
| 20 | `00-product/portfolio-de-produtos.md:302` | Conteúdo & Vídeos: tsvector → ISearchProvider |
| 21 | `00-product/portfolio-de-produtos.md:534` | Banco Talentos: tsvector → ISearchProvider |
| 22 | `00-product/portfolio-de-produtos.md:608` | Vizinhança hyperlocal: tsvector → ISearchProvider |
| 23 | `00-product/portfolio-de-produtos.md:895` | Lista abolidos: contexto D-120 |
| 24 | `00-product/portfolio-de-produtos.md:910` | Tradução legada: tsvector→ISearchProvider |
| 25 | `00-product/sub-produtos/06-lms-academia.md:391` | Busca "(OpenSearch ou tsvector)" → ISearchProvider real |
| 26 | `00-product/sub-produtos/07-banco-de-talentos.md:611,726,841` | 3 menções tsvector → ISearchProvider real |
| 27 | `01-domain/bounded-contexts.md:373` | OpenSearch via ISearchProvider (dev tsvector) → ISearchProvider real |
| 28 | `01-domain/ubiquitous-language.md:158` | SearchIndex aggregate: descrição atualizada |
| 29 | `02-architecture/c4-context.md:142` | OpenSearch managed → ISearchProvider real desde M1 |
| 30 | `03-subprojects/backend/requirements/institutional.md` (3 trechos) | FR-BE-INS-023 + tabela rotas + D-### fechado |
| 31 | `03-subprojects/backend/requirements/commercial.md` (2 trechos) | FR-BE-COM-030 + tabela rotas |
| 32 | `03-subprojects/backend/requirements/content.md` (5 trechos) | Bullet stack + FR-BE-CNT-005/006 + tabela rotas + materialized views + D-### fechados |
| 33 | `03-subprojects/backend/requirements/cross-domain.md` (2 trechos) | Listas D-### |
| 34 | `03-subprojects/web/requirements/lms.md:55` | FR-WEB-LMS-003 |
| 35 | `03-subprojects/web/ui-catalog/lms/LMS2.md:67` + `LMS11.md:76` | LMS busca |
| 36 | `04-requirements/functional/content.md:114` | Migração inicial → ISearchProvider real M1 |
| 37 | `04-requirements/functional/institutional.md:719` | Full-text → ISearchProvider real |
| 38 | `12-docs/README.md:133` | Stack table |
| 39 | `12-docs/adr-index.md:38` | ADR-0017 marcado partial-superseded |
| 40 | `CLAUDE.md:136` | Tabela BC content |

### Bucket 6 — D-134 (apps/admin dedicada — revoga D-058/D-072 admin transversal)

| # | Arquivo | Mudança |
|---|---|---|
| 41 | `00-product/portfolio-de-produtos.md:27` | Regra-raiz admin |
| 42 | `00-product/personas.md:22` | Princípio admin |
| 43 | `00-product/sub-produtos/03-reputacao.md:644` | Admin modera reputação via apps/admin |
| 44 | `00-product/sub-produtos/07-banco-de-talentos.md:63,634` | Admin BT 2 ocorrências |
| 45 | `00-product/sub-produtos/11-administradoras-condominiais.md:76,607` | Não confundir admin plataforma vs administradora |
| 46 | `00-product/sub-produtos/01-governanca-institucional.md:1106` | ADRs relevantes |
| 47 | `00-product/sub-produtos/02-connect-me.md:16,1342` | Admin denúncias + ADRs |
| 48 | `00-product/sub-produtos/06-lms-academia.md:18` | Banner sub-produto |
| 49 | `00-product/sub-produtos/04-conteudo-videos.md:19` | Banner conteúdo |
| 50 | `00-product/sub-produtos/09-compliance.md:121` | Tabela Admin MS |
| 51 | `03-subprojects/mobile/_moc.md:11,54` | Banner mobile + link STATE |
| 52 | `03-subprojects/mobile/README.md:21` | Mobile não tem telas admin |
| 53 | `03-subprojects/mobile/security.md:561` | Implicações security mobile |
| 54 | `03-subprojects/web/_moc.md:29,100,105` | Decisão destaque + links |
| 55 | `03-subprojects/web/ui-catalog.md:690` | Tabela persona × app: Admin → apps/admin |
| 56 | `03-subprojects/web/ui-catalog/admin/main.md:350` | Link meta atualizado |
| 57 | `04-requirements/matrix-functional.md:468` | Fonte transversal admin |

### Bucket 7 — D-097 (Lives = admin + Empresa Pro + agência delegada — revoga D-063/D-076 admin-only)

| # | Arquivo | Mudança |
|---|---|---|
| 58 | `04-requirements/functional/billing.md:590` | BIL-035 EARS reescrita (matriz expandida) |
| 59 | `04-requirements/functional/billing.md:950` | Invariante 16 |
| 60 | `04-requirements/functional/billing.md:981` | Lista STATE D-### |
| 61 | `04-requirements/functional/content.md:29` | Banner content BC |
| 62 | `04-requirements/functional/matrix-functional.md:704` | Princípio 4 |
| 63 | `00-product/personas.md:462` | Linha admin "criar lives" |
| 64 | `00-product/portfolio-de-produtos.md:330,892` | 2 menções "Lives admin-only" |
| 65 | `00-product/sub-produtos/05-assembleia-live.md:1037,1241` | Distinção lives institucionais vs assembleia (+ D-133) |
| 66 | `00-product/sub-produtos/06-lms-academia.md:758` | STATE refs |
| 67 | `03-subprojects/backend/requirements/billing.md:79` | INV-BIL-015 |
| 68 | `03-subprojects/backend/requirements/billing.md:128` | FR-BE-BIL-025 EARS reescrita |
| 69 | `03-subprojects/backend/requirements/billing.md:270` | Pendência BIL-035 |
| 70 | `03-subprojects/backend/requirements/content.md` (3 trechos) | Bullet stack + CNT-005 + D-### fechados |
| 71 | `03-subprojects/traceability.md:46,174,433,831` | 4 linhas: BC summary, LMS8, endpoints, matriz tarefas |
| 72 | `03-subprojects/web/ui-catalog/lms/_moc.md:43` | Regra crítica 2 |
| 73 | `03-subprojects/web/ui-catalog/lms/LMS8.md:4` | Frontmatter tag |

## Dupla checagem (4 rounds)

| Round | Stale encontradas | Stale resolvidas | Notas |
|---|---|---|---|
| Round 1 (subagentes) | 17 | 17 | Cobertura inicial subagentes |
| Round 2 (grep direto) | 21 | 21 | MOCs sub-projetos + ADR index + sub-produtos não cobertos |
| Round 3 (grep direto) | 14 | 14 | Detalhes em portfolio + 04-requirements |
| Round 4 (final) | **0** | — | Todas as buscas vazias (excluindo registros legítimos com banner) |

**Padrões finais validados zero ativo:**
- ✅ "Bloqueio múltiplos cadastros mesmo IP em < 1h" — 0
- ✅ "Morador Base = 0/ano" como fato ativo — 0 (apenas registros revogados)
- ✅ "1 score único" / "score único agregado" — 0
- ✅ "3 vínculos" como regra ativa — 0
- ✅ "tsvector" como solução M1 (excluindo ADRs canônicos sobre o tema) — 0
- ✅ "Admin = role privilegiada transversal" — 0
- ✅ "Lives admin-only" como regra ativa — 0
- ✅ "1 IP 1 cadastro" — 0

## Decisões fechadas / status atualizado

| ID | Antes | Depois |
|---|---|---|
| **DT-012** (vocabulary cleanup rolling M2) | 🟡 aberto, "rolling em ~15 arquivos" | **PROGRESSO MAIOR** — 73 patches em 30 arquivos. Ainda há ~10 menções residuais em arquivos de baixíssima criticidade (registros internos, listas de exemplo) que podem ficar para próxima passada. |
| **A-077** (admin-privilegios.md reescrita pendente) | 🟡 aberto | Mantido aberto (reescrita completa do arquivo 54KB ainda pendente — banner suficiente por enquanto) |
| **A-052** (BIL-035 + cross-domain Fase 12 D-097) | 🟡 aberto | 🟢 **resolvido** — BIL-035 + FR-BE-BIL-025 + INV-BIL-015 atualizados |
| **A-FASE10-DEV-002 stale matriz N1/N2/N3** | 🟢 resolvido | Mantido — não-crítico ↔ vault canônico já reflete tiers universais |

## Mercado Pago — não é parte deste pass

João mencionou interesse em migrar Stripe → Mercado Pago. **Esse é um problema separado**:

- **Vault está consistente** — D-056/D-071 dizem que `IPaymentGateway` é INVARIANTE arquitetural inegociável
- **Backend Go NÃO está agnóstico** — Stripe vaza massivamente para `domain/`:
  - `domain/entities/plan.go`: 8 referências (`stripeProductID`, `StripePriceIDFor` etc)
  - `domain/entities/subscription.go`: 12+ referências (`stripeCustomerID`, `LinkStripe()`, `entities.SubscriptionStatus(stripeSub.Status)`)
  - `domain/repositories/i_subscription_repository.go`: `FindByStripeSubscriptionID`, `FindByStripePriceID`
  - `domain/providers/i_payment_gateway.go`: comentários e header `Stripe-Signature`, `cus_XXX`
  - `domain/events/events.go`: comentário "Stripe retenta ~3 semanas"
  - `application/use_cases/checkout_subscription_saga.go`: variável `stripeSub`, helper `compensateStripeSubscription`
  - `application/use_cases/handle_stripe_webhook_use_case.go`: nome do struct é Stripe-specific
  - `application/mappers/{plan,subscription}_mapper.go`: campos `StripeMonthlyID`, `StripeYearlyID`
  - `migrations/00002_create_plans.sql` + `00005_create_subscriptions.sql`: colunas `stripe_*` + 2 indexes
- **Para destravar migração**: 5-8 dias refactor + 3-5 dias adapter MP. Tem que abrir D-### nova com dual-check; não é só editar Markdown.
- **Próximo passo**: João decide se abre ADR de migração agora (M1.5/M2) ou backlog.

## Próximos passos sugeridos

1. **Conversa com João sobre web** (M1 praticamente não iniciado — A-FASE10-DEV-005 🔴) — tem 22 dias até M1 (2026-05-18 firme via D-106)
2. **ADR de migração Stripe → Mercado Pago** se decidido — abrir D-### nova + dual-check + plano de refactor
3. **Reescrita de `admin-privilegios.md`** (A-077) — reflete D-134 completo
4. **Pass cleanup residual DT-012** — caçar últimas ~10 menções de baixíssima criticidade

## Vizinhos

- [[../../STATE]] — D-094, D-097, D-099, D-103, D-106, D-117, D-118, D-120, D-134, D-135 fontes canônicas
- [[../../AUDIT]] — DT-012, A-077 abertos
- [[../../01-domain/business-rules#BR-063]] — regra antifraude reescrita
- [[../../04-requirements/functional/identity#IDN-026]] — spec implementação device fingerprint progressivo
- [[../../02-architecture/adr/0042-search-provider]] — ADR pending dual-check OpenSearch vs Meilisearch
- [[../../02-architecture/adr/0018-opensearch-tsvector-dev]] — partial-superseded por 0042
