---
title: Gap Analysis — Design System (componentes e tokens)
type: audit
tags: [audit, design-system, ui-catalog, gap-analysis, master-sindico, sprint-10]
project: master-sindico
created: 2026-04-24
author: claude-code
scope: 03-subprojects/web/ui-catalog + 03-subprojects/mobile/ui-catalog vs design-system.md (web + mobile)
method: grep seções "## Componentes" / "## Widget Flutter principal" + CamelCase backtick + cruzamento com catálogo formalizado no DS
---

# Gap Analysis — Design System (componentes e tokens)

Análise **rigorosa** de gaps entre componentes citados nas 234 telas do `ui-catalog` (web 181 + mobile 53) e o design system documentado (`web/design-system.md` 541 linhas + `mobile/design-system.md` 275 linhas). Zero invenção — toda citação ancorada em arquivo + wikilink.

## Sumário

| Métrica | Web | Mobile |
|---|---:|---:|
| Telas auditadas | 181 | 53 |
| Telas com seção `## Componentes` | 179 (99%) | 7 (13%) — resto usa `## Widget Flutter principal` |
| Componentes distintos citados | **442** | **68** |
| Citações totais | 704 | 79 |
| Componentes formalizados no DS | **35** (§6 Atoms/Molecules/Organisms) | **17** (§4 tabela) |
| Órfãos (citados, não formalizados) | **436** | **64** |
| Órfãos com ≥5 telas (bloqueadores M1) | **9** | **0** (mobile tem volume pequeno — maior crítico é ListView×4) |
| Componentes DS sem uso nas telas | 29/35 (83%) | 13/17 (76%) |

**Leitura crítica**: o DS documenta um catálogo **conceitual** (Kobalte primitives genéricos: `Button`, `Select`, `Dialog`, `EmptyState`…), mas as telas citam componentes de **domínio** com nomes próprios (`PrimaryButton`, `FilterBar`, `PageHeader`, `ConfirmContextModal`, `LegalTextBlock`, `VoiceTextarea`). O gap real não é de primitives — é de **composições de domínio ausentes**. Dos 29 "DS sem uso" web, ~20 são primitives (Button/Input/Checkbox/…) que de fato existem nas telas sob **alias de composição** (`PrimaryButton` = Button variant=default; `MandatoryCheckbox` = Checkbox + variant legal). Verdadeiros gaps: composições reutilizáveis 5+ telas sem nome canônico no DS.

---

## A. Componentes órfãos — top 30 web (ordem por frequência)

| # | Componente | Freq | Freq M1* | Telas exemplo | Severidade | Esforço |
|--:|---|--:|--:|---|---|---|
| 1 | `FilterBar` | 30 | 16 | [[../../03-subprojects/web/ui-catalog/sindico/S9]], [[../../03-subprojects/web/ui-catalog/morador/M11]], [[../../03-subprojects/web/ui-catalog/compliance/C5]] | 🔴 CRITICAL | M (combobox + chips + range) |
| 2 | `PrimaryButton` | 27 | 13 | [[../../03-subprojects/web/ui-catalog/sindico/S1]], [[../../03-subprojects/web/ui-catalog/compliance/C2]], [[../../03-subprojects/web/ui-catalog/banco-talentos/BT01]] | 🔴 CRITICAL (alias `Button variant=default`) | S (documentar alias) |
| 3 | `PageHeader` | 11 | 11 | [[../../03-subprojects/web/ui-catalog/sindico/S7]], [[../../03-subprojects/web/ui-catalog/sindico/S26]] | 🔴 CRITICAL | S (título + subtítulo + actions slot) |
| 4 | `ButtonGroup` | 10 | 4 | [[../../03-subprojects/web/ui-catalog/morador/M3]], [[../../03-subprojects/web/ui-catalog/empresa/E4]] | 🟠 HIGH | S |
| 5 | `Pagination` | 8 | — | [[../../03-subprojects/web/ui-catalog/sindico/S9]], [[../../03-subprojects/web/ui-catalog/lms/LMS14]] | 🟠 HIGH (DS lista como M1 mas sem primitive — precisa custom) | M |
| 6 | `BadgeStatus` | 8 | 3 | [[../../03-subprojects/web/ui-catalog/morador/M13]], [[../../03-subprojects/web/ui-catalog/compliance/C1]] | 🟠 HIGH (duplica `StatusBadge`×4 — normalizar nome) | S |
| 7 | `Form` | 6 | 6 | [[../../03-subprojects/web/ui-catalog/sindico/S8]], [[../../03-subprojects/web/ui-catalog/sindico/S24]] | 🔴 CRITICAL (wrapper form conceitual — definir modelo base) | M |
| 8 | `VoiceTextarea` | 5 | 5 | [[../../03-subprojects/web/ui-catalog/sindico/S4]], [[../../03-subprojects/web/ui-catalog/sindico/S22]] | 🔴 CRITICAL (específico governança — STT acoplado) | L |
| 9 | `ExportButton` | 5 | 5 | [[../../03-subprojects/web/ui-catalog/sindico/S17]], [[../../03-subprojects/web/ui-catalog/compliance/C5]] | 🔴 CRITICAL (dropdown CSV/PDF) | S |
| 10 | `ConfirmContextModal` | 5 | 5 | [[../../03-subprojects/web/ui-catalog/sindico/S12]], [[../../03-subprojects/web/ui-catalog/sindico/S25]] | 🔴 CRITICAL (Dialog especializado com motivo + obs + aceite) | M |
| 11 | `TabBar` | 4 | 4 | [[../../03-subprojects/web/ui-catalog/sindico/S2]], [[../../03-subprojects/web/ui-catalog/sindico/S26]] | 🟠 HIGH (DS chama `TabStrip` — divergência nomenclatura) | S |
| 12 | `SearchInput` | 4 | 4 | [[../../03-subprojects/web/ui-catalog/sindico/S9]], [[../../03-subprojects/web/ui-catalog/sindico/S31]] | 🟠 HIGH (Input + ícone + debounce) | S |
| 13 | `MandatoryCheckbox` | 4 | 3 | [[../../03-subprojects/web/ui-catalog/sindico/S29]], [[../../03-subprojects/web/ui-catalog/compliance/C2]] | 🟠 HIGH (compliance LGPD) | S |
| 14 | `LegalTextBlock` | 4 | 3 | [[../../03-subprojects/web/ui-catalog/sindico/S30]], [[../../03-subprojects/web/ui-catalog/compliance/C2]] | 🟠 HIGH | S |
| 15 | `LegalNote` | 4 | 0 | [[../../03-subprojects/web/ui-catalog/banco-talentos/BT02]] | 🟡 MEDIUM (M2/M3 — duplica `LegalTextBlock`) | S |
| 16 | `FileUpload` | 4 | 3 | [[../../03-subprojects/web/ui-catalog/sindico/S3]], [[../../03-subprojects/web/ui-catalog/empresa/E4]] | 🔴 CRITICAL (DS tem `UploadDropzone` — alinhar) | M |
| 17 | `ActionGroup` | 4 | 0 | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS3]] | 🟡 MEDIUM | S |
| 18 | `StatusBadge` | 4 | 2 | [[../../03-subprojects/web/ui-catalog/morador/M10]], [[../../03-subprojects/web/ui-catalog/compliance/C1]] | 🟠 HIGH (duplica `BadgeStatus` — escolher um) | S |
| 19 | `VisibilityToggle` | 3 | 3 | [[../../03-subprojects/web/ui-catalog/sindico/S12]] | 🟡 MEDIUM | S |
| 20 | `SecondaryButton` | 3 | 3 | [[../../03-subprojects/web/ui-catalog/sindico/S2]], [[../../03-subprojects/web/ui-catalog/morador/M15]] | 🟡 MEDIUM (alias `Button variant=secondary`) | S |
| 21 | `AttachmentList` | 3 | 3 | citado web + mobile [[../../03-subprojects/mobile/ui-catalog/morador/M7]] | 🟠 HIGH | S |
| 22 | `VideoPlayer` | 3 | 0 | [[../../03-subprojects/web/ui-catalog/lms/LMS4]] | 🟡 MEDIUM (M2 LMS) | L |
| 23 | `PromotionHero`/`PromotionList`/`PromotionDescription` | 3×3 | 0 | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS4]] | 🟡 MEDIUM (M2/M3 vizinhança) | M |
| 24 | `PartnerHero`/`PartnerInfoBlock` | 3×2 | 0 | [[../../03-subprojects/web/ui-catalog/vizinhanca/VS3]] | 🟡 MEDIUM (M2/M3) | M |
| 25 | `IndicatorGrid` | 3 | 2 | [[../../03-subprojects/web/ui-catalog/sindico/S7]] | 🟡 MEDIUM (dashboard cards) | S |
| 26 | `CouponBigDisplay`, `CountdownTimer`, `ActivationTable`, `ActivateButton` | 3 cada | 0 | vizinhança VM3/VM4 | 🟢 LOW (M3) | — |
| 27 | `WizardStepper` | 2 | 2 | [[../../03-subprojects/web/ui-catalog/sindico/S3]], [[../../03-subprojects/web/ui-catalog/sindico/S4]] | 🟠 HIGH (DS já tem `Stepper` — alinhar nome) | S |
| 28 | `RiskMatrixField`/`RiskHeatmap` | 2 cada | 2 | [[../../03-subprojects/web/ui-catalog/sindico/S22]] | 🔴 CRITICAL domínio (risco ABNT — exclusivo governança) | L |
| 29 | `LGPDLogTable`, `LGPDCheckbox` | 2 + citações compliance | 2 | compliance/C5 | 🟠 HIGH | M |
| 30 | `DateRangePicker` | 3 | 2 | [[../../03-subprojects/web/ui-catalog/sindico/S9]] | 🟢 (DS já formaliza — nenhum gap) | — |

\* Freq M1 = telas em `sindico/`, `morador/`, `onboarding/`, `compliance/`, `assembleia/` (escopo Sprint 10 / M1 conforme [[../../00-product/sub-produtos/01-governanca-institucional]] + [[../../00-product/sub-produtos/09-compliance]] + [[../../00-product/sub-produtos/05-assembleia-live]]).

Esforço: S ≈ 0.5–1d; M ≈ 1–3d; L ≈ 3–5d.

### Órfãos web — categorias secundárias (freq 2)

Total: 436 componentes distintos citados e não formalizados. Top agrupados por domínio:

- **Compliance/LGPD**: `LGPDLogTable`, `LGPDCheckbox`, `LegalDisclaimer`, `TermsBlock`, `SignatureReceipt`, `MandatoryCheckbox`, `LegalTextBlock`, `LegalNote` — 8 componentes, cluster forte para M1 compliance ([[../../03-subprojects/web/ui-catalog/compliance/_moc]]).
- **Riscos/Matriz ABNT**: `RiskMatrixField`, `RiskHeatmap`, `RiskClassifier`, `RejectionModal`, `NaturezaSelector`, `PrioritySelect` — M1 governança (síndico S22 e S12/S24).
- **Vizinhança/Promoções M3**: `PromotionHero`, `PromotionList`, `PromotionDescription`, `PromotionTypeSelect`, `PartnerHero`, `PartnerInfoBlock`, `PartnerCardGrid`, `EstablishmentInfo`, `CouponBigDisplay`, `CountdownTimer`, `ActivationTable`, `ActivateButton`, `RatingDisplay` — 13 componentes exclusivos do sub-produto 08-vizinhanca. Adiável Sprint 11+.
- **Assembleia live** (mobile principalmente): `SpeakerQueueDialog`, `SpeakerQueueButton`, `SubmitVote`, `VoteButtonRow` — M1 assembleia [[../../00-product/sub-produtos/05-assembleia-live]].
- **Banco de talentos M3**: `SyndicBadge`, `LockedBanner`, `MetaInfo`, `MaterialList` — 4 componentes banco-talentos (BT*).
- **Dashboards síndico**: `IndicatorGrid`, `StatsRow`, `PrimaryActionGrid`, `DashboardIndicatorCard`, `StatusChip`, `StatusBanner` — M1 governança, 6 componentes similares (consolidáveis em **3** no DS).

Padrão recorrente: componentes similares com nomes diferentes por tela. Agrupar e normalizar reduz 436 → ~120 distintos reais.

### Órfãos mobile (top 15)

| # | Componente | Freq | Telas | Severidade | Notas |
|--:|---|--:|---|---|---|
| 1 | `ListView` | 4 | [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-S2]], [[../../03-subprojects/mobile/ui-catalog/morador/M4]] | 🟢 (Flutter built-in — não precisa formalizar) | — |
| 2 | `RadioListTile` | 3 | [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-M5]] | 🟢 (Material built-in) | — |
| 3 | `TimelineItemTile` | 2 | [[../../03-subprojects/mobile/ui-catalog/morador/M6]], [[../../03-subprojects/mobile/ui-catalog/sindico/S7]] | 🟠 HIGH (domínio — paridade com web `TimelineEntryCard`) | S |
| 4 | `BottomNavScaffold` | 1 | [[../../03-subprojects/mobile/ui-catalog/morador/M1]] | 🔴 CRITICAL (shell app inteiro — DS mobile não documenta) | M |
| 5 | `HeaderCondominio` | 1 | [[../../03-subprojects/mobile/ui-catalog/morador/M1]] | 🟠 HIGH (reutilizável — provavelmente M2+ multi-screen) | S |
| 6 | `MembershipTile`, `AddMembershipButton` | 1 cada | M2 | 🟠 HIGH | S |
| 7 | `OfflineBanner` | 1 | [[../../03-subprojects/mobile/ui-catalog/morador/M6]] | 🔴 CRITICAL (offline é [[../../03-subprojects/mobile/offline-strategy]] — precisa widget formal) | S |
| 8 | `SkeletonList` | 1 | M6 | 🟠 HIGH (DS lista `SkeletonLoader` abstrato — especializar) | S |
| 9 | `FilterChipBar` | 1 | M6 | 🟠 HIGH (paridade com web `FilterBar`) | S |
| 10 | `RsvpBottomSheet` | 1 | M10 | 🟡 MEDIUM (M2+ eventos) | S |
| 11 | `SpeakerQueueDialog`/`SpeakerQueueButton` | 1 | [[../../03-subprojects/mobile/ui-catalog/assembly/ASM-FILA-FALA]] | 🔴 CRITICAL (assembleia live M1 vai depender) | M |
| 12 | `UrgentBanner` | 1 | [[../../03-subprojects/mobile/ui-catalog/morador/M11]] | 🟠 HIGH (comunicados urgentes — feature central) | S |
| 13 | `DocumentSlot` | 1 | [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-E6]] | 🟠 HIGH | S |
| 14 | `CouponCard`/`MyCouponsPage`/`CouponHistoryPage` | 1 cada | vizinhança VM | 🟢 LOW (M3) | — |
| 15 | `BiometricAuth` | 1 | [[../../03-subprojects/mobile/ui-catalog/onboarding/ONB-S6]] | 🔴 CRITICAL (onboarding — segurança) | M |

Observação mobile: **46 de 53 telas não têm seção `## Componentes`** — usam blocos `## Widget Flutter principal` com código Dart inline. Muitos "órfãos" são na verdade widgets privados por tela (`_CepField`, `_CtaRow`, `OnbM2Page`) filtrados parcialmente na extração. **DS mobile está severamente subestimado** — praticamente nenhum widget de domínio está formalizado ali.

---

## B. Componentes top-10 obrigatórios M1 (bloqueadores Sprint 10)

Ordem de prioridade para formalização no DS antes do início da implementação Sprint 10 (2026-05-08):

| Rank | Componente | Stack | Freq M1 | Por que bloqueador |
|---:|---|---|--:|---|
| 1 | **`PageHeader`** | web | 11 | Todo dashboard síndico precisa. Sem ele, 11 telas ficam com layout ad-hoc divergente. Simples. |
| 2 | **`FilterBar`** | web | 16 | 30 telas no total, 16 M1. Maior ROI. Composta (chips + range + combobox) — precisa spec consolidada. |
| 3 | **`Form`** + **`FormField`** integrado | web | 6 | Já listado como M1 no DS (§6.2), mas sem especificação completa — validation + helper + erro padrão. |
| 4 | **`VoiceTextarea`** | web | 5 | Exclusivo governança. STT + fallback texto. Sem formalização, cada tela reimplementa. |
| 5 | **`ConfirmContextModal`** | web | 5 | Padrão Dialog + motivo + textarea + aceite LGPD. Bloqueador compliance. |
| 6 | **`ExportButton`** | web | 5 | Dropdown CSV/PDF com analytics. Usado em dashboards síndico + compliance. |
| 7 | **`FileUpload`/`UploadDropzone`** alinhamento nome | web | 3 | DS tem `UploadDropzone`, telas usam `FileUpload`. Decidir nome canônico **antes** de codar. |
| 8 | **`BadgeStatus`/`StatusBadge`** normalização | web | 5 (combined) | Duplicidade de nome — escolher um. DS tem `Badge` genérico. |
| 9 | **`BottomNavScaffold` + `HeaderCondominio`** | mobile | 1 cada, mas são **shell** de app | [[../../03-subprojects/mobile/architecture]] depende — M1 inteiro mobile fica bloqueado. |
| 10 | **`SpeakerQueueDialog`** + **`BiometricAuth`** | mobile | 1 cada | Assembleia live M1 (fila de fala) + segurança onboarding. Decisão de UX e backend. |

**Decisão necessária antes da Sprint 10**: consolidar alias. Decidir se DS usa nomes **primitives** (Button, Select) ou **aliases de domínio** (PrimaryButton, CondoSelect). Recomendação: DS documenta primitives + **tabela de aliases oficiais** ("PrimaryButton = `<Button variant='default'>`") — resolve 40+ órfãos sem criar componente novo.

---

## C. Componentes DS sem uso em telas

### Web (29/35 aparentemente não usados)

Primitives Kobalte que as telas usam **via alias**:

| DS formal | Alias nas telas | Efetivamente usado? |
|---|---|---|
| `Button` | `PrimaryButton`×27, `SecondaryButton`×3, `ExportButton`×5, `EditButton`×3, `ActivateButton`×3, … | ✅ sim (via alias) |
| `Input` | `SearchInput`×4, `CEPField`, `NumberField`, `TextField` | ✅ sim |
| `Checkbox` | `MandatoryCheckbox`×4, `LGPDCheckbox` | ✅ sim |
| `Select` | `Select`×6 (nome direto) + `PromotionTypeSelect`, `PrioritySelect`, `NaturezaSelector` | ✅ sim |
| `Dialog` | `ConfirmContextModal`×5, `RejectionModal`×2, `SignatureModal`, … | ✅ sim (via alias) |
| `FormField` | via `Form`×6 | parcial — não uso direto do nome |
| `TabStrip` | telas usam `TabBar`×4 (nome Material, inconsistente) | ⚠️ divergência nome |
| `Stepper` | telas usam `WizardStepper`×2 | ⚠️ divergência nome |
| `DatePicker`/`DateRangePicker` | `DateRangePicker`×3 usa o nome | ✅ sim |
| `EmptyState` | `EmptyState`×38 — **batente DS ativo** | ✅ sim (campeão de uso) |

**Genuinamente não usados nas telas auditadas**: `Avatar`, `Breadcrumb`, `Combobox`, `CommandPalette`, `Drawer`, `Kbd`, `MenuItem`, `NavSidebar`, `Spinner`, `Switch`, `Toast`, `Tooltip`, `Popover`, `AnnouncementBanner`, `TimelineEntryCard` (web — mas mobile cita `TimelineItemTile`×2).

> Nem todos são "DS inchado" — `Toast`, `Tooltip`, `Popover`, `Spinner` são infraestrutura esperada implicitamente. Mas `CommandPalette`, `AnnouncementBanner`, `Breadcrumb`, `NavSidebar` **devem aparecer em shell/layout global** e telas não os citam — provavelmente **telas de shell/layout ainda não foram escritas** no ui-catalog.

### Mobile (13/17)

Não citados explicitamente: `AppBar`, `IconButton`, `OutlinedButton`, `Switch`, `Radio`, `Chip`, `Divider`, `CircularProgressIndicator`, `LinearProgressIndicator`, `SkeletonLoader`, `EmptyState`, `TabBar`, `Card`.

A maioria é widget Material default usado **implicitamente** no código Flutter (Scaffold, Card, ListTile). Problema: **sem citação explícita**, gap-analysis fica cego a padrão de uso — DS mobile precisa de auditoria pós-implementação.

---

## D. Inconsistências nomenclatura web × mobile

| Conceito | Web (ui-catalog ou DS) | Mobile (ui-catalog ou DS) | Decisão recomendada |
|---|---|---|---|
| Card de condomínio | `CondoCard` ([[../../03-subprojects/web/ui-catalog/sindico/S2]]) | não citado | Definir canônico: `CondominiumCard` (EN) em ambos |
| Item timeline | `TimelineEntryCard` (DS §6.3) | `TimelineItemTile` (M6, S7) | Unificar nome: `TimelineEntryCard` (web) / `TimelineEntryTile` (mobile) |
| Filtros | `FilterBar`×30 (web) | `FilterChipBar` (M6) / `FiltersBottomSheet` (VM6) | Mobile usa paradigma bottom-sheet por UX — OK divergir, mas **documentar equivalência** |
| Stepper | `WizardStepper` (web) + `Stepper` (DS) | não citado | Alinhar: `Stepper` canônico, `Wizard*` é alias |
| Badge status | `BadgeStatus`×8 vs `StatusBadge`×4 (mesmo web!) | `Chip` (DS mobile) | Web: escolher **um** nome. Mobile: documentar map para web `Badge`. |
| Tab | `TabBar`×4 (web, nome Material) vs `TabStrip` (DS web) | `TabBar` (DS mobile — Material) | Web muda para `TabStrip` ou DS web renomeia para `TabBar` (alinhar com Material/mobile). **Recomendação**: `TabBar` em ambos. |
| Confirm modal | `ConfirmContextModal`×5 (web) | `showModalBottomSheet` (DS mobile) | Paradigma diferente por plataforma — OK. Documentar par: `ConfirmContextModal` (web dialog) ↔ `ConfirmBottomSheet` (mobile). |
| Upload | `FileUpload`×4 + `MediaUpload` (web) / `UploadDropzone` (DS web) | `image_picker`/`file_picker` (DS mobile) | Unificar nome web: `UploadDropzone`. Manter bindings Flutter nativos em mobile. |
| Header de página | `PageHeader`×11 (web) | `AppBar` (DS mobile) | Divergência aceitável (web card vs mobile toolbar). Documentar paridade. |

---

## E. Estados (Loading/Empty/Error/Success) — formalização

**Citações nas telas**:
- 122 telas web citam **Loading** (67%)
- 89 citam **Empty** (49%)
- 165 citam **Error** (91%)
- 101 citam **Success** (56%)
- **181/181 web tem seção "## Estados"** (100%)

**Formalização no DS**:
- ✅ Web DS §6.3 lista `EmptyState` como organism M1 (**consistente** — é o componente mais citado em telas, 38×)
- ❌ Web DS **não** formaliza `LoadingState`/`Skeleton`/`ErrorBoundary`/`SuccessState` como organisms
- ⚠️ Web DS §6.3 lista `Spinner` (atom) — insuficiente para Loading states em cards/listas (skeletons variados)
- ⚠️ Web DS cita `Skeleton` **zero vezes**. Telas citam "skeleton" 6 vezes ([[../../03-subprojects/web/ui-catalog/sindico/S1]], S2, M6…)
- ❌ Mobile DS §4 lista `SkeletonLoader` mas vago ("shimmer package ou custom shimmer"). Não há spec de quais listas/cards ganham qual variante.
- ❌ **Sem padrão formal para `ErrorBoundary`** — cada tela descreve ação ad-hoc (toast + retry)

**Recomendação M1**:
1. Adicionar ao DS (`§6.3` web + `§4` mobile): `LoadingSkeleton` (variantes: card, list-item, avatar-row, table-row, heading-block), `ErrorState` (ícone + título + descrição + retry CTA), `SuccessState` (usado em confirmações).
2. Criar organismo `StateRenderer` que recebe `state: 'loading'|'empty'|'error'|'success'|'content'` e rende o componente apropriado — reduz boilerplate.
3. Documentar **contratos de Loading**: por quanto tempo mostrar skeleton vs spinner (>200ms skeleton; <200ms nada).

---

## F. Tokens citados × formalizados

### Cores

- Telas citam `--primary`/`--accent`/etc.: **0 ocorrências**. Telas não referenciam tokens CSS por nome.
- Telas descrevem cores em prosa: "gold", "navy", "dourado", "status verde". DS formaliza 22 tokens OKLCH + 9 LP extra — suficiente, mas descoberta é fraca sem mapeamento.

### Tipografia

- `Manrope` citado em 6 telas web + 1 mobile (S2, S4, S16, M10, M11, BT04, BT06, C4)
- `Michroma` citado em 2 telas
- `Inter` citado no DS mobile + BT04
- **Gap**: shortcuts `text-display-h1..h3`, `text-body`, `text-muted` formalizados em DS §3.3 — **nenhuma tela os cita**. Conteúdo descreve "título em destaque", "legenda pequena". Telas precisam ser **ligadas aos shortcuts** na próxima auditoria.

### Spacing/radius

- DS define 4px/8px grid e `--radius: 0.5rem`. Telas não citam medidas em px — apenas "gap pequeno", "card arredondado". Normal para ui-catalog conceitual, mas criar **checklist dev** para garantir grid 8px na implementação.

### Tokens ausentes no DS identificados por uso nas telas

| Token implícito | Onde citado | Precisa no DS |
|---|---|---|
| `--info` (cor informativa) | tooltips/alerts — DS §2.6 já nota ausência | ✅ adicionar M1 |
| `--pending` / `--draft` | status síndico, compliance | considerar adicionar (ou usar `--warning`) |
| Shadows | `SectionPreviewCard`, `PromotionHero` implicam elevation | Formalizar 3 níveis (`shadow-sm/md/lg`) |
| `z-index` scale | modais, popovers, bottom-sheets | Adicionar tabela M1 |
| Animation durations | DS §9.3 "sugestão" — não oficial | Promover para M1 |
| Tier Diamante/Ouro/Prata | `SyndicBadge` banco-talentos + DS diz "Diamante" usa `--success` — gaps outras tiers | Formalizar palette tier (3 cores) |
| Urgent / Priority | `UrgentBanner`, `PrioritySelect` | Token `--urgent` ou regra: use `--destructive` em urgent |

### Fontes OKLCH × ARGB reconciliação (gap cross-platform)

- DS web §2.1 canoniza `oklch(0.715 0.120 84.2)` ≈ `#C49A2A` (gold) e `oklch(0.218 0.055 256.1)` ≈ `#1A2340` (navy).
- DS mobile §1.1 usa `Color(0xFFC69C3F)` (gold) e `Color(0xFF071A33)` (navy) — valores do Manual IV.
- **Divergência**: web `#C49A2A` vs mobile `#C69C3F` (±2 pts matiz). DS web §2.1 reconhece e propõe D-### em [[../../STATE]] ratificando OKLCH. **Status atual: não ratificado**. Bloqueia alinhamento exato cross-platform nos assets (logos, ícones, impressões/prints).
- Mobile DS §10 gap DT-MB-008 (paleta `#6C63FF` default Flutter) — Sprint 10 deveria migrar. **Confirma bloqueador M1**.

---

## G. Plano de formalização priorizado (Sprint 10)

### Sprint 10 (semana 1 — antes implementação)

**DS web** — adicionar seções:
1. **§6.4 Composições de domínio** (novo) — catalogar com API stub:
   - `PageHeader` (title, subtitle, actions, breadcrumb?)
   - `FilterBar` (filters: FilterConfig[], onChange)
   - `ExportButton` (options: {csv, pdf, xlsx}, onExport)
   - `ConfirmContextModal` (title, reason, mandatoryCheckbox?, onConfirm)
   - `VoiceTextarea` (value, onChange, stt: boolean, maxLength)
   - `UploadDropzone` (accept, maxSize, onUpload) — **renomear FileUpload→UploadDropzone** nas telas ou manter alias
   - `StateRenderer` / `LoadingSkeleton` / `ErrorState`
2. **§6.5 Aliases oficiais** (tabela): `PrimaryButton = <Button variant=default>`, `SecondaryButton = <Button variant=secondary>`, `WizardStepper = <Stepper mode=wizard>`, `StatusBadge = <Badge variant=status>`, `TabBar = <TabStrip>` etc.
3. **§2.9 Tokens M1 obrigatórios**: `--info`, shadow scale, z-index scale, durations promovidos.
4. **§6.6 Padrões de estado** (matriz Loading/Empty/Error/Success com guidance por tipo de tela).

**DS mobile** — adicionar:
1. **§4.1 Shell e navegação** (novo): `BottomNavScaffold`, `HeaderCondominio`, `OfflineBanner`, `UrgentBanner`.
2. **§4.2 Assembleia** (novo M1): `SpeakerQueueDialog`, `SpeakerQueueButton`, `VoteBottomSheet`.
3. **§4.3 Onboarding/Auth**: `BiometricAuth`, `DocumentSlot`, `MembershipTile`.
4. Paridade com web `TimelineEntryCard` → `TimelineEntryTile` mobile.

### Sprint 11+

- Vizinhança: `PromotionHero/List/Description`, `PartnerHero/InfoBlock`, `CouponCard`, `CouponBigDisplay`, `CountdownTimer`.
- LMS: `VideoPlayer`, `SectionPreviewCard`.
- Compliance avançado: `LGPDLogTable`, `RiskMatrixField`, `RiskHeatmap`.

### Métricas de sucesso

- **Zero telas M1** com componente órfão após consolidação DS Sprint 10 (hoje: 9 bloqueadores web + 5 mobile).
- **Paridade nome web↔mobile** (hoje: 8 divergências catalogadas em §D).
- **100% estados** (Loading/Empty/Error) usando componente formal (hoje: 0% — nenhum componente Loading/Skeleton no DS web).
- Todos tokens citados em telas M1 mapeiam 1:1 para CSS var/theme.
- `--info` token adicionado ao `main.css` (já flagado §2.6 do DS).
- `btn-destructive` shortcut adicionado (gap já flagado §7 DS web "Ausente").
- Scrollbar bug `var(--color-border)` → `var(--border)` corrigido (§11 DS web).

### Riscos se não formalizar antes Sprint 10

1. **Reimplementação divergente**: 16 telas M1 citando `FilterBar` viram 16 implementações diferentes. Custo retrabalho Sprint 11+ ≈ 2 semanas dev.
2. **Inconsistência visual**: `BadgeStatus`×8 e `StatusBadge`×4 implementados em paralelo por devs diferentes.
3. **Compliance risk**: `MandatoryCheckbox` + `LegalTextBlock` sem spec formal vira risco LGPD (validação versão termos, log de aceite inconsistente) — ver [[../../08-security/lgpd]].
4. **Mobile shell indefinido**: `BottomNavScaffold` + `HeaderCondominio` sem formalização → app mobile inteiro dependendo de decisões ad-hoc do primeiro dev que pegar M1.
5. **Assembleia live bloqueada**: `SpeakerQueueDialog` e fluxo de fala dependem de decisão UX + backend — não codável sem spec.

### Ordem executiva recomendada

1. **Dia 1-2 Sprint 10**: decisão D-### em [[../../STATE]] sobre (a) aliases oficiais (resolve 40+ órfãos num ato), (b) palette OKLCH ratificada, (c) nome canônico TabBar/TabStrip/BadgeStatus.
2. **Dia 3-5**: spec dos 10 componentes top-M1 (tabela §B) — API, variants, estados, acessibilidade. Parallel dev web + mobile.
3. **Dia 6-10**: implementação dos atoms que faltam (`Input`, `Checkbox`, `Radio`, `Switch`, `Badge`, `Spinner`, `Avatar`) — bloqueador dos molecules.
4. **Dia 11-15**: molecules + primeiros organisms (`PageHeader`, `FilterBar`, `ConfirmContextModal`, `StateRenderer`).
5. **Contínuo**: auditoria inversa — cada tela M1 implementada valida que citação → componente real casa.

---

## Links

- [[../../03-subprojects/web/design-system]] — fonte web (541 linhas)
- [[../../03-subprojects/mobile/design-system]] — fonte mobile (275 linhas)
- [[../../03-subprojects/web/ui-catalog/_moc]]
- [[../../03-subprojects/mobile/ui-catalog/_moc]]
- [[_moc]] — audit-log MoC
- [[../../05-tasks/_moc]] — criar tarefas derivadas (DS-web-01 a DS-web-10; DS-mb-01 a DS-mb-05)
- [[../../00-product/sub-produtos/01-governanca-institucional]] — M1 primeiro escopo
- [[../../00-product/sub-produtos/09-compliance]] — M1 legal
- [[../../00-product/sub-produtos/05-assembleia-live]] — M1 assembleia
- [[../../STATE]] — registrar decisão sobre aliases (D-### a criar)
