---
title: MOC — UI Catalog (specs por feature)
type: moc
tags: [moc, master-sindico, ui-catalog, web]
created: 2026-04-25
updated: 2026-04-25
---

# MOC — UI Catalog (specs por feature)

> Catálogo de **UI specs por feature/tela** absorvidas em 2026-04-25 (Fase B do plano de consolidação `_chaos/`). Complementa o catálogo macro [[../ui-catalog|web/ui-catalog]] que cobre ≥ 141 telas com IDs estáveis (M1-M15, S1-S31, etc.).
>
> Estes arquivos vivem em pastas por **bounded context** (BC) ou contexto cross. Cada um tem:
> - Frontmatter com `bc`, `source`, `absorbed_in`.
> - Banner de origem com tradução aplicada (`N1/N2/N3 → trial/base/plus/pro`, "Morador Pagante" removido — D-126, "My Síndico" → "Master Síndico").
> - Wikilinks para canônicos relacionados.

## Identity (BC)

- [[auth/onboarding|auth/onboarding]] — Login / Cadastro / OTP / Welcome (AUTH1-AUTH8 catálogo macro)

## Cross (transversais entre BCs)

- [[shell/app-shell|shell/app-shell]] — Topbar / Sidebar / Bottom Nav / Drawer / Breadcrumb
- [[shell/topbar-sidebar-bottomnav|shell/topbar-sidebar-bottomnav]] — Deltas do FRONTEND_DESIGN_SYSTEM (User Switcher, badges NEW/count, tooltip collapsed)
- [[cross/search-engine|cross/search-engine]] — Busca + filtros + categorias técnicas
- [[cross/private-feedback|cross/private-feedback]] — "Rede social cega" — likes / comentários privados ao dono
- [[cross/notifications|cross/notifications]] — Bell + dropdown + página full + 14 tipos
- [[cross/settings|cross/settings]] — 6 seções (Conta / Notif. / Aparência / Privacidade / Segurança / Plano)
- [[_components-and-screens|_components-and-screens]] — **Índice consolidado** componentes × spec canônica

## CMS / Profile

- [[cms/home-dashboard|cms/home-dashboard]] — Home híbrida (Greeting + Quick Actions + Feed)
- [[cms/profile-management|cms/profile-management]] — Perfil multi-persona (Síndico / Morador / Empresa / Comércio)

## Content (BC)

- [[content/video-player|content/video-player]] — Player + paywall 25 % + TikTok-style fullscreen
- [[content/video-upload|content/video-upload]] — Upload + trava 90 d
- [[content/live-streaming|content/live-streaming]] — Lives broadcast (D-133 — NÃO é Assembleia; provider Mux Live / Cloudflare Stream)
- [[content/podcast|content/podcast]] — YouTube embed (decisão João: feature interna de Conteúdo, não sub-produto separado)

## Commercial (BC)

- [[commercial/connect-me|commercial/connect-me]] — Formulário unidirecional 4 fluxos
- [[commercial/coupon-engine|commercial/coupon-engine]] — Engine cupom `ID_LOJA(5)+SEQ(3)+PALAVRA(5)`, lock 4 h
- [[commercial/benefits-club|commercial/benefits-club]] — Hub Vizinhança + perfil estabelecimento + painel lojista
- [[commercial/evaluation|commercial/evaluation]] — Avaliação bimestral 5 estrelas (alimenta Bronze→Diamante)
- [[commercial/talent-bank|commercial/talent-bank]] — Banco de Talentos (D-099 — 5 vínculos, M3, addon livre)

## Assembly (BC)

- [[assembly/management|assembly/management]] — Wizard 4 steps + lista + detalhe (D-133 — conferência Livekit, NÃO live broadcast)
- [[assembly/voting|assembly/voting]] — Sistema de votação (Aprovar/Reprovar, Múltipla Escolha, Fração ideal, Eleição)
- [[assembly/pre-assembly|assembly/pre-assembly]] — **Fase E** — Pré-assembleia: configuração + procuração 100% digital + análise fornecedores + simulador quórum
- [[assembly/live-session|assembly/live-session]] — **Fase E** — Conferência Livekit ao vivo: mesa diretora + telão 2 áreas + fila fala + votação real-time
- [[assembly/check-in|assembly/check-in]] — **Fase E** — 3 modalidades de check-in (app, QR code, terminal)
- [[assembly/post-assembly|assembly/post-assembly]] — **Fase E** — Encerramento + ata homologada + relatórios + histórico
- [[assembly/administrator-panel|assembly/administrator-panel]] — **Fase E** — Painel administradora (D-127 — 4 capabilities + audit trail)
- [[assembly/transparency-score|assembly/transparency-score]] — **Fase E** — Indicador transparência (10 componentes em 4 subíndices)
- [[assembly/contingency-mode|assembly/contingency-mode]] — **Fase E** — Fallback offline + auto-save Redis

## Institutional (BC)

- [[institutional/transparency|institutional/transparency]] — Painel Transparência + Plano de Reeleição + Timeline imutável

## LMS / Cursos

- [[lms/courses|lms/courses]] — LMS Curso → Módulo → Aula + Wizard criação + Certificado 100 %

## Forum / Vizinhança

- [[forum/main|forum/main]] — Fórum 31 categorias técnicas + posts + threads lineares

## Admin (D-134 — `apps/admin` separada)

- [[admin/main|admin/main]] — Painel Admin (5 módulos: Dashboard / Usuários / Moderação / Configurações / Broadcast)

## Jornadas (UI specs por persona — Fase D 2026-04-25)

- [[jornadas/_moc|jornadas/_moc]] — **MOC das 8 jornadas absorvidas a partir de 9 PDFs do cliente**:
  - [[jornadas/sindico|sindico]] (S1-S31, 31 telas)
  - [[jornadas/morador|morador]] (M1-M15, 15 telas)
  - [[jornadas/empresa|empresa]] (E1-E16, 16 telas)
  - [[jornadas/agencia-marketing|agencia-marketing]] (MK1-MK8, 8 telas)
  - [[jornadas/parceiro-vizinhanca|parceiro-vizinhanca]] (VZ1-VZ6 + VM1-VM7, 13 telas — Vizinhança sub-produto 8)
  - [[jornadas/onboarding|onboarding]] (4 perfis × 3-6 telas, 17 telas)
  - [[jornadas/curriculo-cadastro|curriculo-cadastro]] (M-CV-1 a M-CV-11, 11 telas — Banco de Talentos)
  - [[jornadas/curriculo-empresa-view|curriculo-empresa-view]] (E-CV-1 a E-CV-7, 7 telas — visualização empresa)
  - [[jornadas/_pendencias-fase-h|_pendencias-fase-h]] — divergências/gaps detectados para Fase H avaliar

## Patterns transversais (não-ui-catalog)

- [[../patterns/ui-states|../patterns/ui-states]] — 6 estados obrigatórios + Skeleton + Empty + Error + Restricted + Toast + Micro-interações

---

## Fonte e tradução

- **Origem**: 27 MDs em `_chaos/` (gerados 2026-02-23 — pré-migração).
- **Absorvidos em**: 2026-04-25 — Fase B do plano de consolidação.
- **Tradução**: `N1/N2/N3` → `plan_tier {trial, base, plus, pro}` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".
- **Originais arquivados**: `_archive/2026-04-24-chaos-processed/feature-specs-23-fev/`.

## Vizinhos

- [[../_moc|web/_moc]] — MOC sub-projeto web
- [[../ui-catalog|web/ui-catalog]] — catálogo macro de telas (≥ 141 telas)
- [[../design-system|web/design-system]] — guidelines DS
- [[../patterns|web/patterns]] — patterns gerais
- [[../../../02-architecture/design-tokens|design-tokens]] — fonte canônica de tokens
- [[../../../STATE]] — D-126, D-133, D-134, D-099 (decisões revisadas 2026-04-25)
