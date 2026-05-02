---
title: Gap Analysis Consolidado — UI catalog vs resto do vault
type: audit
tags:
  - master-sindico
  - gap-analysis
  - consolidado
  - audit
  - cross-vault
project: master-sindico
date: 2026-04-24T00:00:00.000Z
scope: 234 telas (181 web + 53 mobile) × backend/domínio/produto/design-system
method: 4 agentes Opus paralelos + dual-check
---

# Gap Analysis Consolidado — UI catalog vs resto do vault

**Data**: 2026-04-24
**Escopo**: 234 telas UI catalog × fontes canônicas (backend requirements, 01-domain, 00-product, STATE/AUDIT, 04-requirements/matrix, 12-docs, 03-subprojects/design-system)
**Método**: 4 agentes Opus paralelos, cada um com foco em um eixo; dual-check independente valida consolidação
**Relatórios-filhos** (linhas):
- [[2026-04-24-gap-analysis-endpoints]] (803)
- [[2026-04-24-gap-analysis-dominio]] (656)
- [[2026-04-24-gap-analysis-produto-processos]] (732)
- [[2026-04-24-gap-analysis-design-system]] (308)

---

## Sumário executivo

O vault passou de estrutura superficial para **muito detalhada** em UI catalog (234 telas × endpoints × ABAC × invariantes). Cruzamento sistemático revelou **4 classes** de gaps, ordenadas por bloqueador M1:

| # | Classe | Quantidade | 🔴 M1-bloqueador | Relatório |
|---|---|---|---|---|
| 1 | **Endpoints fantasma** (telas citam, backend não formaliza) | 333 | 112 | [[2026-04-24-gap-analysis-endpoints#a]] |
| 2 | **ABAC actions órfãs** (telas usam, matriz canônica não lista) | 221 | ~76 | [[2026-04-24-gap-analysis-produto-processos#gap-j-03]] |
| 3 | **Componentes DS órfãos** (telas citam, design-system não cataloga) | 500 (436 web + 64 mobile) | 9 web | [[2026-04-24-gap-analysis-design-system#a]] |
| 4 | **Aggregates ausentes em `01-domain/aggregates/`** mesmo declarados no UL | 13 | 3 | [[2026-04-24-gap-analysis-dominio#b]] |

**Interpretação**: nada aqui é "surpresa" — a maioria dos gaps decorre do mesmo padrão: as telas foram **destiladas fielmente do que o cliente especificou** (Inventario-Telas + PDFs + canvas), e o backend + domain + DS foram destilados de **outra fonte** (código Go + specs técnicas). As duas destilações não conversavam de forma sistemática. Este relatório é o cross-check que fecha o triângulo.

---

## 1. Endpoints fantasma (333)

### Veredito
**98% dos endpoints citados em telas não estão formalizados em `backend/requirements/`**. Não é código faltante — handlers existem em `Development/backend/internal/modules/*` (backend roda, passou gates Sprint 9) — é **falta de formalização REST** que quebra rastreabilidade FR-UI ↔ FR-BE e o DoR de Sprint 10.

### Distribuição por severidade
| Severidade | Qtd | Característica |
|---|---|---|
| 🔴 CRITICAL M1 | 112 | Bloqueia M1 (institutional + identity + billing) |
| 🟠 HIGH M2 | 140 | Commercial (Connect Me), Content, Reputação, Vizinhança |
| 🟡 MEDIUM M3 | 81 | LMS, Academia, BT completa |

### Top 5 bloqueadores M1

1. **`GET /api/v1/institutional/timeline/{id}`** — núcleo M1, citado em S11, S14, M6, M7, mobile/morador/M6 (backend formaliza `/api/v1/condominiums/:id/timeline` — namespace diverge)
2. **`POST /api/v1/condominiums/draft` + `/{id}/finalize` + `/{id}/mandate`** — cadastro condomínio (S3→S5), fluxo M1 inteiro inferido
3. **`GET /api/v1/condominiums/{id}/dashboard/summary` + `/cards/badges`** — dashboard síndico (S6), home M1
4. **`GET /api/v1/institutional/memberships/me/default` + `/dashboard/resident`** — painel M1 morador (M1 tela)
5. **Suite announcements `GET|POST /api/v1/condominiums/{id}/announcements` + validate + reject + adjustment** (S23–S25) — institucional base M1

### Outros achados
- **162 endpoints no backend sem tela consumidora** — 3 webhooks (OK) + 24 admin (OK) + 4 infra (OK) + **131 feature-APIs suspeitas** (podem ser mortas ou sub-documentadas)
- **51 citações explícitas "inferido"** em 33 telas
- **45 divergências web × mobile** (mesmo fluxo usa path diferente)

### Ação (Sprint 10)
- Zerar 112 🔴 em `backend/requirements/institutional.md` até 2026-05-01 (T-7 do M1)
- Eliminar 51 "inferido" até 2026-05-03
- Padronizar web canônico + mobile reusa mesmo path (2026-05-04)
- Triagem dos 131 feature-APIs órfãs (tela existente / criar backlog / killar)

---

## 2. ABAC actions órfãs (221)

### Veredito
Telas usam **227 ABAC actions distintas** (ex: `timeline.create`, `membership.register`, `vote.cast.delegated`). A matriz funcional canônica (`04-requirements/matrix-functional.md`) + steering declaram só **177**. **221 órfãs** — telas são **a fonte primária** de definições ABAC, não o contrário.

### Impacto
Se nada mudar, quando o backend for seedar o ABAC matrix em produção, ele terá que derivar dos screens MDs (fonte inesperada) em vez do spec canônico. **Resultado esperado**: divergências sutis de permissão em M1.

### Top 3 categorias de action órfãs (M1)
1. **`validation.*`** (review, reject, approve, adjust, aprove-with-note) — 14 actions em S21/S22/S26 (validation hub) sem entrada na matriz
2. **`compliance.*`** (declaration.sign, matrix.update, audit.read, report.export) — 9 actions em C2/C5/C11
3. **`dashboard.*`** (syndic.view, resident.view, empresa.view, agencia.view, export, custom-range) — 8 actions em S6, S27, M1, E1, E10, MK1, MK7

### Ação
Consolidar em `04-requirements/abac-actions.md` derivado das 227 actions. Formato EARS: `<role>.<plan_tier>.<action>.<resource> → allow|deny + reason`. Executar antes do Sprint 10 para não regar bugs.

---

## 3. Componentes design system órfãos (500)

### Veredito
**Crise de composições de domínio ausentes**, não de primitives. Kobalte genérico está coberto no `design-system.md` web; mas os **aliases e composições** que as telas citam (PageHeader, FilterBar, ConfirmContextModal, VoiceTextarea) não estão catalogados.

### Números
- **Componentes citados**: 442 web + 68 mobile (704+79 citações totais, muitas repetidas)
- **Formalizados**: 35 web + 17 mobile
- **Órfãos**: 436 web + 64 mobile
- **Bloqueadores M1 (≥5 telas)**: 9 componentes web

### Top 10 componentes que DEVEM entrar no DS antes M1

| # | Componente | Telas que usam | Stack | Esforço |
|---|---|---|---|---|
| 1 | `PageHeader` | 11 M1 | web | S |
| 2 | `FilterBar` | 16 M1, 30 total | web | M |
| 3 | `Form` + `FormField` | 6 M1 | web | L (integração TanStack Form) |
| 4 | `VoiceTextarea` | 5 M1 (governança/auditoria) | web | M (STT Web API) |
| 5 | `ConfirmContextModal` | 5 M1 (padrão Dialog+motivo+LGPD) | web | S |
| 6 | `ExportButton` | 5 M1 | web | S |
| 7 | `FileUpload` ↔ `UploadDropzone` | 3 M1 (nomes divergentes — escolher um) | web | M |
| 8 | `BadgeStatus` ↔ `StatusBadge` | 5 M1 combinados (duplicidade) | web | S |
| 9 | `BottomNavScaffold` + `HeaderCondominio` | shell app inteiro | mobile | L |
| 10 | `SpeakerQueueDialog` + `BiometricAuth` | M3 mas fundação | mobile | M |

### Gaps transversais
- **Estados Loading/Empty/Error** formalizados em 67% das telas mas sem componente formal no DS (Skeleton, EmptyState, ErrorBoundary ausentes)
- **Paleta OKLCH divergente** web (`#C49A2A`) vs mobile (`#C69C3F`) — precisa D-### decisão
- **DS mobile severamente subestimado** — 46/53 telas mobile sem seção `## Componentes`

### Ação (Dias 1-2 Sprint 10)
- D-### nova: tabela de aliases oficiais (`PrimaryButton = <Button variant=default>`) — resolve 40+ órfãos num ato
- D-### paleta OKLCH canônica
- Criar 10 componentes top-10 no `packages/ui` + doc no `design-system.md`

---

## 4. Aggregates ausentes (13)

### Veredito
`01-domain/ubiquitous-language.md` declara 47 aggregates; apenas 40 têm arquivo em `01-domain/aggregates/`. **13 aggregates declarados mas sem arquivo MD** — documentação incompleta.

### Top 3 bloqueadores M1

1. **`ValidationItem.md` ausente** — S21/S22/S26 (validation hub) é fluxo M1 inteiro; sem aggregate formal, backend opera "por intuição" do código Go. **Decisão pendente**: `ValidationItem` ≡ `ServiceRecord`? Reconciliar.
2. **`ComplianceDeclaration.md` ausente** — INV-034/INV-035 citam-na como aggregate primário do módulo compliance; onboarding M1 registra aceite LGPD (6 telas ONB-*/4 aceites) sem aggregate canônico.
3. **`TermAcceptance.md` ausente** — onboarding registra aceite LGPD (5 tipos: Geral, LGPD, Avaliações, Connect Me, Conteúdo Técnico); sem aggregate, persistência fica ad-hoc.

### Outros achados

- **10 INVs órfãos** (7 não-promovidos a `invariants.md` + 3 inéditos sendo 1 novo INV-130 e 2 reclassificações SLO/scope) — ex: INV-ASM-015 (voto secreto por item), INV-LMS-003 (trilha de progresso), INV-BT-010 (trava 90d vídeo) citados em telas sem entrada em `invariants.md`. Detalhe no sub-relatório.
- **4 eventos LMS órfãos** (CourseCompleted, VideoWatched, TopicReplied, CertificateIssued) — já marcados G-04 pelas telas (aggregates LMS são gap M3).
- **0 enums órfãos de fato** (bem coberto por 90+ enums em Enums-Consolidados). Só 1 divergência numérica: "18+ tipos de evento" (telas) vs 13 (canônico).

### Ação (Sprint 10)

Sprint de **domain hardening 3 dias** (~1 dia-pessoa efetivo):
- Criar `ValidationItem.md` + `ComplianceDeclaration.md` + `TermAcceptance.md`
- Formalizar 7 INVs órfãos em `invariants.md`
- Resolver divergência numérica tipos evento (18+ vs 13) com produto

---

## 5. Gaps cross-corte — produto & processos

### 5.1. Sub-produtos com cobertura incompleta

| Sub-produto | Telas dedicadas | Gap |
|---|---|---|
| 11-administradoras-condominiais | **0** | GAP-J-01 🔴 — 5 endpoints `empresa_sindica_handler.go` sem UI |
| Admin Master Síndico (transversal) | **0** (só `GAP-admin-master-sindico.md`) | GAP-J-02 🔴 — 5 capacidades documentadas no legado mas sem spec concreto |

### 5.2. Frontmatter inconsistente (32 telas)

- 23 telas com `sub_produto: onboarding` (não é sub-produto canônico — é fluxo transversal)
- 6 telas com `sub_produto: banco-talentos` (canônico é `banco-de-talentos`)
- 3 telas com `sub_produto: marketing` (canônico é `marketing-agencias`)

**Ação**: batch rename via `sed` nos 32 frontmatters. 5min de trabalho.

### 5.3. Plan tier inconsistente (5 telas)

5 telas mobile/assembly com `plan_requirement: premium` — enum fora do canônico trial/base/plus/pro (D-057/D-080/D-081). **Ação**: rename para valor canônico.

### 5.4. Runbooks citados mas inexistentes

1 runbook citado mas ausente: `admin-mfa-lockout.md` (referenciado em `03-subprojects/admin-privilegios.md`). **Pré-requisito de ADR-0026** (MFA admin M1). **Ação**: criar stub mínimo em `12-docs/runbooks/`.

### 5.5. Referências bidirecionais saudáveis ✅

- **17 D-### citadas em telas → 0 fantasmas** (todas existem em STATE.md)
- **0 A-### citadas em telas** — mas isso é **gap inverso**: nenhum finding formal foi linkado a uma tela específica. Rastreabilidade finding→tela ausente.

---

## 6. Priorização consolidada (top 20 ações Sprint 10)

> ⚠️ **Pré-condição global**: [[../../AUDIT#a-fase10-dev-005]] (web M1 praticamente não iniciado — 15-25 dias de trabalho restante, 2 semanas até 2026-05-08) torna o escopo M1 inviável na data original. **Assumir Opção A (M1 = 2026-05-18)** ou shipping M1 backend+mobile-only com web em onda M1.1. Se Opção A não for tomada, priorizar apenas os itens #1-#5 desta lista e adiar o resto para hotfix.
>
> Reforços cruzados: [[../../AUDIT#a-fase10-dev-002]] (matriz N1/N2/N3 backend) alimenta gap §2 (ABAC órfãs); [[../../AUDIT#a-fase10-dev-008]] (IAuditLogger LGPD incompleto cross-BC) alimenta §4 (aggregate `AuditLog`/TermAcceptance ausente).

Ordenadas por impacto M1 (2026-05-08 ou 2026-05-18 se Opção A):

| # | Ação | Esforço | Responsável | Pré-requisito |
|---|---|---|---|---|
| 1 | Formalizar 112 endpoints 🔴 CRITICAL em `backend/requirements/institutional.md` | 1 dia | backend agent | — |
| 2 | Consolidar `04-requirements/abac-actions.md` (227 actions) | 0.5 dia | product agent | — |
| 3 | Criar `ValidationItem.md` + `ComplianceDeclaration.md` + `TermAcceptance.md` em `01-domain/aggregates/` | 0.5 dia | domain agent | — |
| 4 | D-094: tabela de aliases DS (resolve 40+ órfãos) | 0.25 dia | dev lead | — |
| 5 | Implementar 10 componentes top-10 em `packages/ui` | 3-4 dias | web team | #4 |
| 6 | Rename 32 frontmatters `sub_produto:` não-canônico | 15min | bash script | — |
| 7 | Rename 5 `plan_requirement: premium` → canônico | 10min | bash script | — |
| 8 | Criar stub `admin-mfa-lockout.md` em `12-docs/runbooks/` | 0.25 dia | ops agent | — |
| 9 | Formalizar 7 INVs órfãos em `invariants.md` | 0.5 dia | domain agent | #3 |
| 10 | D-095: web canônico / mobile reusa path (45 divergências) | 0.25 dia | arch agent | #1 |
| 11 | Eliminar 51 "inferido" em telas (formalizar ou remover) | 0.5 dia | web agent | #1 |
| 12 | Triagem 131 endpoints backend órfãos (tela/backlog/kill) | 1 dia | arch agent | — |
| 13 | D-096: paleta OKLCH canônica (web ≡ mobile) | 0.25 dia | DS lead | — |
| 14 | Criar 4-6 telas mínimas `S33..S36` / `E17..E18` para Administradora | 2 dias | web agent | #1, #3 |
| 15 | Formalizar Skeleton/EmptyState/ErrorBoundary no DS | 0.5 dia | DS lead | #4 |
| 16 | Rastreabilidade A-### → tela (adicionar wikilink bidirecional) | 0.5 dia | audit agent | — |
| 17 | Expandir `traceability.md` dos ~20 endpoints listados para os 340 | 0.5 dia | audit agent | #1 |
| 18 | Resolver divergência "18+ tipos evento" vs "13" — enum + matriz | 0.25 dia | product agent | — |
| 19 | Decisão: `ValidationItem` ≡ `ServiceRecord`? | 0.25 dia | domain lead | #3 |
| 20 | Auditoria complementar DS mobile (46/53 sem seção Componentes) | 1 dia | mobile agent | — |

**Total estimado**: ~14 dia-pessoa para zerar gaps M1. Cabe em 2 semanas com 3-4 agentes paralelos.

---

## 7. O que NÃO é gap

Para evitar alarme falso:

- **17 D-### citadas em telas são todas reais** — STATE.md está íntegro (D-001..D-093)
- **0 enums inventados** — Enums-Consolidados tem 90+ valores, cobertura excelente
- **Aggregates centrais (User, Assembly, Vote, Subscription, Plan, TimelineEntry, Condominium, Unit, Membership, MasterPlanActivity) estão todos formalizados**
- **Backend roda** — código funciona, Sprint 9 fechou com gates verdes; gap é de **spec**, não de implementação
- **Traceability já existe** (`03-subprojects/traceability.md` 939 linhas) — só precisa expandir para os 340 endpoints
- **Zero wikilinks quebrados** após fix F6 (290 links normalizados)
- **0 non-MD no vault** (100% markdown)

---

## 8. Conclusão

O trabalho de cross-check sistemático revelou **que o vault é mais completo do que os 4 relatórios-filhos individualmente sugerem** — mas a integração entre as camadas foi feita de forma **assíncrona**:

1. **Telas** (UI catalog) destiladas do cliente (Content/, PDFs, canvas)
2. **Backend requirements** destilado do código Go + .kiro/ do backend
3. **Domain** destilado do UL + aggregates canônicos
4. **DS** destilado genericamente de Kobalte/Flutter

As 4 destilações ficaram **consistentes cada uma internamente**, mas o cross-product (tela↔endpoint↔aggregate↔component) **não foi formalizado**. Este relatório é o primeiro cross-product completo do projeto.

**Principal ação pós-relatório**: executar as 20 ações priorizadas (§6) em 3 ondas paralelas (endpoints / domínio / DS) para zerar bloqueadores M1 antes de 2026-05-08 (ou 2026-05-18 se Opção A).

---

## Links

- [[../AUDIT]] — AUDIT principal do projeto
- [[../../STATE]] — decisões vivas
- [[../../03-subprojects/traceability]] — matriz rastreabilidade
- [[2026-04-24-gap-analysis-endpoints]] — detalhamento §1
- [[2026-04-24-gap-analysis-dominio]] — detalhamento §4 + INVs/enums/eventos
- [[2026-04-24-gap-analysis-produto-processos]] — detalhamento §5
- [[2026-04-24-gap-analysis-design-system]] — detalhamento §3
- [[../../CLAUDE]] — contrato do agente
