---
title: MOC — 03-subprojects/web
type: moc
tags:
  - moc
  - master-sindico
  - v2
  - subprojects
  - web
  - solidjs
  - typescript
  - tanstack
  - rsbuild
  - unocss
  - kobalte
  - bun
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-23T00:00:00.000Z
---

# MOC — 03-subprojects/web

Sub-projeto **`web/`** — Portal web do Master Síndico. Stack canônica: **SolidJS + TypeScript + TanStack Router/Query/Form + Rsbuild (Rspack) + UnoCSS + Kobalte + Axios + Zod + Biome + Bun workspaces**. **7 apps + 2 packages** (apps/admin scaffolded em 2026-04-27 — D-134). Consome API backend Go via cookie httpOnly `.mastersindico.com.br`.

**Repo alvo pós-migração**: `github.com/mastersindico/web` (código real em `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/web/`).

**Decisões de destaque**:
- **D-020** (reconciliação Fase 8): stack real é **Rsbuild + TanStack + UnoCSS + Kobalte** (Fase 2 inferiu Vite + Solid Router + Tailwind — reverter).
- **D-134** (revoga D-058/D-072): admin MS **tem app dedicada `apps/admin`** no monorepo web (Cloudflare Access SSO+MFA), NÃO mais embutida em `cms` sob `/admin/*`.
- **Planos Stripe-style universal**: `trial | base | plus | pro` — **sem slugs** persona-specific no código; apresentação ao usuário pode ser persona-aware.

---

## Arquivos

- [[README]] — overview, stack confirmada (versões `bun.lock`), workspace layout (6 apps + 2 packages), 6 apps (papel/porta/persona-alvo), contratos backend (REST cookie httpOnly + WS + auth Zitadel OIDC+PKCE), heatmap persona × app.
- [[architecture]] — monorepo Bun (regras de dependência), Rsbuild/Rspack (+ TanStack router plugin autoCodeSplitting), TS config strict + `noUncheckedIndexedAccess`, Biome, entry point canônico (ColorModeProvider + RouterProvider + ThemeSync), state via Solid primitives (`createSignal/Store/Memo/Resource/Context`), API client Axios, WebSocket Assembleia, device fingerprint, dark mode CSS vars OKLCH, shared packages (template `Button` CVA×Kobalte+splitProps+PolymorphicProps), rotas file-based + guards, deploy Railway (DT `turbo`), CORS, observability, testing stack.
- [[patterns]] — SolidJS primitives (quando usar cada), component library atomic design leve, Kobalte + CVA + splitProps (template), UnoCSS attributify prefix `un-` + shortcuts como design system, TanStack Router/Query/Form idioms, A11Y WCAG 2.1 AA, 4+1 estados obrigatórios (loading/empty/error/success/permission-denied), error boundaries, Code Calisthenics TS/SolidJS (9 regras adaptadas: branded types, first-class collections, componentes pequenos etc), TanStack Query patterns (keys, invalidate, optimistic), i18n pt-BR (termos preservados), 15+ anti-patterns, testing patterns (Vitest + `@solidjs/testing-library` + MSW + fast-check PBT).
- [[ui-catalog]] — **catálogo exaustivo ≥ 141 telas distintas × 6 apps × persona**, com ID, rota, ações, estados, regras de visibilidade: Auth 8 (AUTH1-AUTH8) · CMS Morador 15 (M1-M15) · CMS Síndico 31 (S1-S31) + Compliance 11 (C1-C11) · CMS Empresa 16 (E1-E16) · CMS Agência 8 (MK1-MK8) · CMS Vizinhança subset 29 (VZ/VS/VE/VM) · CMS Banco Talentos 11 (TEL1-TEL11) · CMS **Admin MS embutido D-058** 8 (AD1-AD8) · CMS Conta+Sistema 8 (SYS1-SYS8) · LMS 15 (LMS1-LMS15) · Assembly 8 (AS1-AS8) · LP 6 (PUB1-PUB6). Regras R1-R10 transversais. 9 telas-pivô arquiteturais. Mapa persona ↔ app.
- [[design-system]] — Manual IV ouro primário `#C69C3F` (OKLCH `oklch(0.715 0.120 84.2)`) + navy `#071A33` (OKLCH `oklch(0.218 0.055 256.1)`) + accent gold light + paleta LP blob system. CSS vars tokens OKLCH light + dark (`main.css`). Tipografia Michroma (brand/headings) + Manrope (body) + Fira Code (mono) via Google Fonts. Shortcuts UnoCSS `btn-*/text-display-*/flex-*/glass-*/lp-*` como design system. 20+ componentes catalogados (atoms/molecules/organisms) padrão Kobalte+CVA+splitProps. Ícones Lucide (prefix `i-`). Motion + reduce-motion. Dark mode via Kobalte ColorModeProvider. Contraste WCAG AA matriz.
- [[requirements]] — Core Web Vitals (LCP < 2.5s mobile / 1.5s desktop; CLS < 0.1; INP < 200ms; TTFB < 600ms), bundle budgets (LP 80KB, app 300KB, chunk 150KB), cache strategy, imagens AVIF/WebP, fonts preload + swap, WCAG 2.1 AA (criterions + ferramental), responsive mobile-first (breakpoints UnoCSS), PWA manifest (SW opcional M3+), i18n pt-BR (termos BR preservados), forms TanStack + Zod + voice input R2 + auto-save + double-confirm crítico, upload Mux signed, realtime WS/SSE/polling, error handling 4 camadas, observability (Sentry PII scrub + Web Vitals), SEO LP, paywall backend-first (planos Stripe-style), LGPD (consent granular + Connect Me modal + export/delete + PII scrubbing), coverage thresholds (95/90/85/75/80), admin MS MFA + 4-eyes + isolamento visual, browser support matrix, feature flags runtime.
- [[security]] — cookie httpOnly (domain `.mastersindico.com.br`, Lax, Secure prod, nunca localStorage), CSP strict com nonce + report-uri + frame-ancestors 'none', XSS prevention (SolidJS auto-escape + proibições innerHTML/eval/dangerouslySetInnerHTML + DOMPurify em markdown + safeHref validação protocol + rel noopener), CSRF (SameSite=Lax + CORS allowlist + double-submit `X-CSRF-Token`), CORS allowlist explícito (sem `*`), clickjacking (frame-ancestors + X-Frame-Options), HTTPS + HSTS + upgrade-insecure-requests, supply chain (bun audit + Dependabot + Snyk + SRI + lockfile + blocklist), IDOR → 404 uniforme + ULID/UUIDv7 opacos, **admin isolation D-058** (3 layers: route guard UI + backend ABAC + sub-role matrix + MFA M2+ + 4-eyes M3+ + banner "Modo Admin"), secret management (prefix `PUBLIC_` only; CI gate), device fingerprint canvas+webgl+UA SHA-256, rate limit UI feedback 429 Retry-After, LGPD frontend (Connect Me modal + consent versioning + DSAR export/delete + PII scrub Sentry), OWASP Top 10 mapping, security headers checklist produção, auth flow Zitadel OIDC+PKCE security review, storage strategy.
- [[tasks]] — seed FE-### por sprint M1/M2/M3 + backlog + transversais + **24 dívidas técnicas catalogadas** (DT-WEB-001 a 024 — DT-WEB-002, 003, 005, 016, 019, 021, 022, 023 🟢 resolvidos em 2026-04-27); cross-link para [[AUDIT-2026-04-27]]
- [[AUDIT-2026-04-27]] — auditoria sistemática 2026-04-27 com 26 findings (3 🔴 + 10 🟠 + 10 🟡 + 3 🟢); stack confirmada 100% alinhada com docs oficiais via Context7
- [[../../11-audit/audit-log/2026-04-27-agente-2-systematic-pass]] — pass de execução do Agente 2: 11 fixes web + apps/admin scaffold + backend A-035 foundation + Flutter D-048/049 deps; `bun run check` 0/0/1 (zero errors/warnings/1 info); 81 testes verdes

---

## Estrutura real confirmada (código em disco Apr 2026)

```
Development/web/                                      # root (Apr 21)
├── CLAUDE.md                                          # 399 linhas — fonte operacional
├── AGENTS.md                                          # 647 linhas — guidelines cross-stack
├── README.md                                          # stub bun init (DT-WEB-006)
├── package.json                                       # workspaces: ["packages/*", "apps/*"]; scripts raiz
├── biome.json                                         # schema 2.3.11 — fonte de formatação
├── tsconfig.json                                      # strict + bundler moduleResolution + noUncheckedIndexedAccess
├── bun.lock                                           # 107 KB — versões resolvidas de todas apps
├── index.ts                                           # stub "Hello via Bun" (DT-WEB-007)
├── .env                                               # template Zitadel (local-only)
├── apps/
│   ├── auth/      (port 3000)   rsbuild + tanstack + uno + kobalte + solid + zod + axios
│   ├── cms/       (port 3001)   mesma stack — painel principal multi-persona (admin owner agora em apps/admin — D-134)
│   ├── lms/       (port 3002)   mesma stack — Academia
│   ├── forum/     (port 3003)   mesma stack — Fórum / Vizinhança
│   ├── assembly/  (port 3004)   mesma stack — Assembleia live (WS)
│   ├── lp/        (port 3005)   mesma stack + blob hero system
│   └── admin/     (port 3010)   ✅ scaffolded 2026-04-27 — D-134 (Cloudflare Access + ABAC backend); banner persistente "Modo Admin" + 5 áreas planejadas (Dashboard / Tenants / Moderação / Financeiro / Broadcast)
├── packages/
│   ├── ui/                                             # atoms base (Button ✅) + CVA + Kobalte
│   └── schemas/                                        # STUB (DT-WEB-005); M1 preenche
├── old/                                                # legado gitignored — auth-context, query-context, __root antigo, docs PDFs/analyses
└── node_modules/                                       # bun install raiz + apps
```

---

## Marcos (alinhados a [[../../ROADMAP]])

| Marco | Entrega web |
|---|---|
| **M1 (2026-05-08)** — Sprint 10 | Setup + packages/ui atoms + schemas scaffold + `apps/auth` completo + `apps/cms` Morador (M1-M15) + Síndico base (S3-S6, S11-S14) + Admin D-058 AD1-AD2 + Conta SYS1-SYS3 + 75-95% coverage |
| **M2 (2026-06-20)** — Sprint 11 | Connect Me 4 fluxos (S18-S20 + E2-E4) + Empresa painel E1-E16 + Validações S26 + Conteúdo/Mux + Agência MK1-MK8 + Compliance C1-C11 + `apps/forum` completo (VZ/VS/VE/VM) + `apps/lp` completo + Admin AD3-AD6 + MFA TOTP |
| **M3 (2026-07-20)** — Sprint 12 | `apps/assembly` completo (AS1-AS8 + WS + telão <200ms) + `apps/lms` (LMS1-LMS15) + Banco Talentos (TEL1-TEL11) + Admin AD7-AD8 (4-eyes) + E2E Playwright + a11y/perf audit |

---

## Coordenação cross-stack

- **Backend coord**: OpenAPI 3.1 contract-first em [[../../02-architecture/api-design]]; gerador TS types → `packages/schemas/generated/*` (FE-008).
- **Mobile coord**: paridade de personas Morador/Síndico no app Flutter ([[../mobile/README]]); tokens visuais espelhados ([[../mobile/ui-catalog]]).
- **Design coord**: Figma frame oficial `https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX`; plugin Claude `figma` → `/implement-design <frame>`.
- **Backend Zitadel**: callback URLs + CORS allowlist sincronizados (emitir cookie `.mastersindico.com.br` alcança todas 6 apps).

---

## Vizinhos

- [[../../_moc]] — MOC 03-subprojects
- [[../../CLAUDE]] — contrato do agente v2
- [[../../STEERING]] — não-negociáveis
- [[../../STATE]] — decisões vivas (D-020 stack, D-134 admin app dedicada — revoga D-058/D-072)
- [[../../ROADMAP]] — marcos M1/M2/M3
- [[../../DoR-DoD]] — ready/done gates
- [[../backend/_moc]] — sub-projeto backend Go (contratos REST/WS)
- [[../mobile/_moc]] — sub-projeto mobile Flutter (paridade personas + tokens)
- [[../admin-privilegios]] — admin (banner superseded; canônico atual D-134 = `apps/admin` dedicada — reescrita pendente A-077)
- [[../../01-domain/bounded-contexts]] — 6 BCs (paridade feature-sliced web)
- [[../../04-requirements/_moc]] — requirements funcionais
- [[../../08-security/BEYOND_CORP]] — zero-trust posture
- [[../../00-product/personas]] — 7 personas + sub-roles ABAC
- [[../../00-product/portfolio-de-produtos]] — 7 sub-produtos


---

## Fase B — Absorção `_chaos/` 2026-04-25 (delta sobre canônico existente)

**27 MDs feature-specs / design-system** (gerados 2026-02-23) absorvidos do `_chaos/` para overlay canônico:

### Novos índices

- [[ui-catalog/_moc|ui-catalog/_moc]] — MOC do **catálogo de UI specs por feature** (27 specs) em pastas por BC: auth · shell · cms · content · commercial · assembly · institutional · lms · forum · admin · cross.
- [[ui-catalog/_components-and-screens|ui-catalog/_components-and-screens]] — Índice consolidado componentes × spec canônica (mapeia ~120 componentes do monolito original ao destino canônico).
- [[patterns/ui-states|patterns/ui-states]] — **PATTERN cross-app** (NOVO): 6 estados obrigatórios (Loading / Skeleton / Success / Empty / Error / Restricted) + Toast (solid-sonner) + Micro-interações + acessibilidade. Filosofia: "o usuário nunca olha tela em branco". Aplica a TODAS as telas em [[ui-catalog]].

### Tradução vocabulário aplicada

- `N1/N2/N3` → `plan_tier {trial, base, plus, pro}` (ADR-0032 / D-081 / D-096)
- "Morador Pagante" removido (D-126) — morador é gratuito; **Banco de Talentos** é addon livre opt-in (D-099, 5 vínculos profissionais visíveis, M3)
- "My Síndico" → "Master Síndico"

### Decisões consolidadas no path da Fase B

- **D-133** — Lives broadcast (Mux Live / Cloudflare Stream) ≠ Assembleia (Livekit conferência tipo Meet). Refletido em [[ui-catalog/content/live-streaming]] vs [[ui-catalog/assembly/management]].
- **D-134** (revoga D-058) — Admin agora em **`apps/admin`** dedicada no monorepo web, não embutida em `cms/`. Refletido em [[ui-catalog/admin/main]]. **Pendência**: atualizar [[ui-catalog]] §B.7 (AD1-AD8) que ainda diz "embutido em cms".
- **D-099** (substitui D-064) — Banco de Talentos: 5 vínculos profissionais (não 3), M3.
- **D-126** — "Morador Pagante" descontinuado.

### Delta em `02-architecture/design-tokens.md`

- Seção §18 adicionada com regras inegociáveis de aplicação (proibições de uso semântico) + Status Badges Bronze→Diamante (CSS pronto) + keyframes canônicos (`fadeIn`, `fadeInUp`, `slideInRight`, `scaleIn`, `heartBounce`) + convenção Iconify (`i-lucide-*`) + CVA Button/Badge canônicos + Kobalte primitives mínimos.

### Originais arquivados

`_archive/2026-04-24-chaos-processed/feature-specs-23-fev/` (27 arquivos preservados na íntegra). Ver [[../../_archive/2026-04-24-chaos-processed/_README|_README do archive]] §"Fase B concluída".
