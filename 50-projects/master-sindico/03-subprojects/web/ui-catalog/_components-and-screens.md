---
title: Components & Screens — Index consolidado (UI Spec)
type: moc
tags:
  - ui-catalog
  - master-sindico
  - index
  - components
  - screens
bc: cross
source: _chaos/MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md §4-5 + §11 (2026-02-23)
created: 2026-04-25T00:00:00.000Z
updated: 2026-04-25T00:00:00.000Z
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# Components & Screens — Índice consolidado

> **Origem**: `_chaos/MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md` (398 KB, 7814 linhas) seções §4 (Componentes — Design Detalhado), §5 (Telas Sprint 2 — Gestão Condominial) e §11 (Mapa de Telas Sprint 1 & 2).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Estratégia de absorção**: o monolito original cobria praticamente as mesmas 27 features que entraram em arquivos dedicados nesta fase. Em vez de duplicar 7800 linhas, este índice mapeia cada componente / tela ao seu **destino canônico** absorvido.

---

## Mapeamento componente → canônico

### Atoms / Primitives

| Componente | Spec canônica |
|---|---|
| `Button` (CVA + Kobalte `ButtonPrimitive.Root`) | [[../../../../02-architecture/design-tokens#185-cva—button—badge-canônicos]] |
| `Badge` (CVA) | [[../../../../02-architecture/design-tokens#185-cva—button—badge-canônicos]] |
| `Status Badge` Bronze→Diamante | [[../../../../02-architecture/design-tokens#182-status-badges—reputação-bronze-diamante]] |
| `OTPField` (`@corvu/otp-field`) | [[./auth/onboarding|auth/onboarding]] §OTP |
| `Skeleton` primitives + variants | [[../patterns/ui-states|patterns/ui-states]] §2 |
| `Toast` (solid-sonner) | [[../patterns/ui-states|patterns/ui-states]] §6 |
| `LoadingButton` | [[../patterns/ui-states|patterns/ui-states]] §7.1 |
| `Spinner`, `ProgressBar`, `CountdownTimer` | [[../patterns/ui-states|patterns/ui-states]] §7 |

### Molecules / Organisms

| Componente | Spec canônica |
|---|---|
| `AppShell` / `Topbar` / `Sidebar` / `BottomNav` / `Drawer` / `Breadcrumb` | [[./shell/app-shell|shell/app-shell]] |
| `UserSwitcher`, `SidebarPlanCard`, `SidebarBadgeNew`, `SidebarBadgeCount`, `SidebarTooltip`, `BottomNavFabActionSheet` | [[./shell/topbar-sidebar-bottomnav|shell/topbar-sidebar-bottomnav]] |
| `VideoPlayer` + controles + `PaywallOverlay` | [[./content/video-player|content/video-player]] |
| `VideoCard` (variantes: institucional, currículo, pauta, preview 25 %, travado 90 d) | [[./content/video-player#video-card-componente-reutilizado-em-toda-app]] |
| `TikTokPlayer` (vertical fullscreen) | [[./content/video-player#tiktok-style-fullscreen-mobile-vídeos-curtos]] |
| `DragDropZone`, `UploadProgress`, `UploadBlockedState` (trava 90 d) | [[./content/video-upload|content/video-upload]] |
| `PromoCard`, `CouponModal`, `BlockedModal`, `WordOfDayEditor`, `MetricsCard`, `PromoForm`, `PromoListItem`, `CountdownTimer` | [[./commercial/coupon-engine|commercial/coupon-engine]] |
| `VizinhancaHub`, `CEPBar`, `CategoryGrid`, `PromoScroll`, `StoreCard`, `StoreProfile`, `MeuNegocioForm`, `GalleryUpload`, `HorarioGrid`, `NoCEPBanner` | [[./commercial/benefits-club|commercial/benefits-club]] |
| `ConnectMeForm`, `QuotaBar`, `ConnectMeDisclaimer` | [[./commercial/connect-me|commercial/connect-me]] |
| `CurriculumCard`, `CurriculumDetail`, `CurriculumForm`, `QuarterlyLockBar`, `InterestModal`, `CurriculumMetrics`, `SpecialtyPills`, `TalentFilters` | [[./commercial/talent-bank|commercial/talent-bank]] |
| `EvaluationPage`, `StarRating`, `EvalQuestion`, `EvalProgress`, `EvalResult`, `EvalBreakdown` | [[./commercial/evaluation|commercial/evaluation]] |
| `AssemblyCard`, `AssemblyWizard`, `WizardSteps`, `PautaAccordion`, `VendorVideoSearch`, `TempLinkCard`, `EditalViewer` | [[./assembly/management|assembly/management]] |
| `VotingPage`, `VoteOption`, `VoteRadioGroup`, `VoteApproveReject`, `VoteConfirmation`, `VoteTimer` | [[./assembly/voting|assembly/voting]] |
| `TransparencyDashboard`, `PlanItemList`, `PlanItem`, `PlanTotalBar`, `Timeline`, `TimelineItem`, `PDFExportButton` | [[./institutional/transparency|institutional/transparency]] |
| `CourseCard`, `CourseDetail`, `ModuleAccordion`, `LessonItem`, `LessonPlayer`, `LessonSidebar`, `CourseProgress`, `CertificateCard`, `CourseWizard` | [[./lms/courses|lms/courses]] |
| `ForumPostCard`, `ForumPostDetail`, `ForumReplyItem`, `ForumReplyInput`, `ForumNewPostForm`, `ForumSearch`, `ForumTabs`, `ForumCategoryPills` | [[./forum/main|forum/main]] |
| `LiveNowCard`, `LiveUpcomingCard`, `LiveRecordedCard`, `LivePlayer`, `LiveChat`, `LiveAdminPanel` | [[./content/live-streaming|content/live-streaming]] |
| `PodcastFeatured`, `PodcastEpisodeItem`, `PodcastPlayer`, `PodcastSearch` | [[./content/podcast|content/podcast]] |
| `InteractionBar` (curtir / comentar / denunciar), `LikeButton`, `CommentInput`, `OwnerMetrics`, `OwnerCommentList`, `ReportModal` | [[./cross/private-feedback|cross/private-feedback]] |
| `NotificationBell`, `NotificationDropdown`, `NotificationItem`, `NotificationPage`, `NotificationTabs`, `EmptyNotifications` | [[./cross/notifications|cross/notifications]] |
| `SearchPage`, `SearchBar`, `FilterPills`, `FilterSidebar`, `SearchResultCard`, `CategoryBrowseGrid`, `CEPFilter` | [[./cross/search-engine|cross/search-engine]] |
| `SettingsShell`, `SettingsSidebar`, `SettingsSection`, `ToggleRow`, `ThemeSelector`, `PasswordStrength`, `SessionCard`, `PlanCard`, `PaymentHistory`, `DangerZone`, `PhotoUpload` | [[./cross/settings|cross/settings]] |
| `AdminShell`, `AdminSidebar`, `KpiCard`, `ChartLine`, `ChartDonut`, `UserTable`, `ModerationQueue`, `RichTextEditor`, `BannerManager`, `CategoryTree`, `ParamInput`, `BroadcastForm`, `EmailPreview`, `MinScreenGuard` | [[./admin/main|admin/main]] |
| `LoginForm`, `SignupForm`, `PasswordForm`, `WelcomeScreen`, `InterestTagSelector` | [[./auth/onboarding|auth/onboarding]] |
| `HomePage`, `HomeGreeting`, `QuickActionGrid`, `QuickActionCard`, `CategoryIconRow`, `CategoryIcon`, `HomeSection`, `QuotaWidget`, `LiveBadge` | [[./cms/home-dashboard|cms/home-dashboard]] |
| `ProfilePage`, `ProfileCover`, `ProfileAvatar`, `ProfileInfo`, `ProfileStats`, `ProfileTabs`, `ProfileVideoGrid`, `StatusDashboard`, `EditProfileForm` | [[./cms/profile-management|cms/profile-management]] |

### Estados / patterns transversais

| Pattern | Spec |
|---|---|
| 6 estados (Loading / Skeleton / Success / Empty / Error / Restricted) | [[../patterns/ui-states|patterns/ui-states]] §1 |
| Empty states catalog (19+ contextos) | [[../patterns/ui-states|patterns/ui-states]] §3.2 |
| Error boundary + 404 + Network banner + Inline form errors | [[../patterns/ui-states|patterns/ui-states]] §4 |
| Paywall overlay (vídeo 25 %) + Gate page + Trava 90 d + Cooldown 4 h + Cota Connect Me | [[../patterns/ui-states|patterns/ui-states]] §5 |
| Toasts (solid-sonner — variantes, durações, catálogo de mensagens) | [[../patterns/ui-states|patterns/ui-states]] §6 |
| Micro-interactions (Like animation, Vote confirmation, Form focus, Card hover, Toggle, Tabs, Accordion, Page transitions, Notification pulse) | [[../patterns/ui-states|patterns/ui-states]] §8 |
| Maintenance Mode, Session Expired modal, Force Update | [[../patterns/ui-states|patterns/ui-states]] §9 |

---

## Mapeamento telas Sprint 1 & 2 (§11 do monolito)

O §11 listava ~50 telas para Sprint 1 e Sprint 2. Estas telas estão consolidadas no catálogo macro:

- [[../../../web/ui-catalog|web/ui-catalog]] — catálogo exaustivo ≥ 141 telas × 6 apps × persona, com IDs estáveis (M1-M15 morador, S1-S31 síndico, E1-E16 empresa, MK1-MK8 marketing, VZ/VS/VE/VM vizinhança, TEL1-TEL11 banco talentos, AS1-AS8 assembly, LMS1-LMS15, PUB1-PUB6 LP, AD1-AD8 admin, SYS1-SYS8 conta, C1-C11 compliance).

A correspondência entre §11 do monolito e catálogo macro:

| §11 Tela | ID catálogo macro |
|---|---|
| Login / Signup / OTP / Welcome | AUTH1-AUTH8 |
| Home / Discovery / Category Filter | M1, S6 (home governança), E1 (home empresarial) |
| Profile (Síndico / Morador / Empresa / Comércio) | abrange 04-PROFILE-MANAGEMENT consolidado |
| Search + Browse Categories | cobre [[./cross/search-engine]] |
| Video Player + Upload | [[./content/video-player]] + [[./content/video-upload]] |
| Connect Me Form | [[./commercial/connect-me]] |
| Coupons / Vizinhança / Meu Negócio | [[./commercial/coupon-engine]] + [[./commercial/benefits-club]] |
| Assembleias (Lista / Wizard / Detalhe / Votação) | [[./assembly/management]] + [[./assembly/voting]] |
| Avaliação (Form / Result) | [[./commercial/evaluation]] |
| Painel Transparência | [[./institutional/transparency]] |
| LMS (Catálogo / Detalhe / Player / Wizard / Certificado) | [[./lms/courses]] |
| Fórum (Lista / Detalhe / Novo) | [[./forum/main]] |
| Lives (Lista / Player / Admin) | [[./content/live-streaming]] |
| Podcast | [[./content/podcast]] |
| Banco de Talentos | [[./commercial/talent-bank]] |
| Notificações (Bell / Dropdown / Page) | [[./cross/notifications]] |
| Settings (6 seções) | [[./cross/settings]] |
| Admin Panel (5 módulos) | [[./admin/main]] |
| Empty / Error / Loading / Paywall / Restricted states | [[../patterns/ui-states]] |

---

## Notas de absorção

1. **Não há informação perdida** — todo conteúdo do monolito está em algum dos arquivos canônicos absorvidos nesta Fase B (27 arquivos). O original 7814 linhas está preservado em `_archive/2026-04-24-chaos-processed/feature-specs-23-fev/MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md`.
2. **Tradução vocabulário**: `N1/N2/N3` (do monolito) → `trial/base/plus/pro` em todos os destinos. "Morador Pagante" removido (D-126). "My Síndico" → "Master Síndico".
3. **Decisões revisadas**: D-058 (admin transversal embutido) **revogada** por D-134 — Admin agora em `apps/admin` separada (refletido em [[./admin/main]]). D-064 (3 vínculos) **substituída** por D-099 (5 vínculos — refletido em [[./commercial/talent-bank]]). D-133: Lives ≠ Assembleia — refletido em [[./content/live-streaming]] (broadcast Mux/Cloudflare) vs [[./assembly/management]] (conferência Livekit).
4. **Tokens não duplicados** — design-tokens canônico em [[../../../../02-architecture/design-tokens]] já cobre §1-2 do monolito + delta absorvido em §18.

## Links

- [[../../../web/ui-catalog|web/ui-catalog]] — catálogo macro de telas
- [[../../../web/design-system|web/design-system]] — guidelines DS
- [[../patterns|web/patterns]]
- [[../../../../02-architecture/design-tokens|design-tokens]] — tokens canônicos
- [[../../../../STATE]] — D-126, D-133, D-134, D-099 (decisões revisadas 2026-04-25)
