---
title: lp — Web Requirements (SolidJS)
type: requirement
tags: [master-sindico, web, solidjs, lp, feature-req, marketing, seo]
project: master-sindico
stack: web
app: lp
port: 3005
milestone: m1
status: in-progress
created: 2026-04-24
---

# lp — Web Requirements

## Escopo e marco

**Marco**: M1 (Sprint 10, 2026-05-08). Entregável principal: landing page institucional (/ hero já implementado — único app web com conteúdo real), /pricing (planos Stripe-style `trial|base|plus|pro`), /sobre, /blog (estrutura), /contato. SEO obrigatório.

M2: blog CMS completo + casos de uso por persona + comparativos concorrência.

M3: webstories + calculadora de economia + landing pages customizadas por campanha (MK4-MK6 feeder).

## Personas servidas

- **Público anônimo** (sem auth) — visitantes, leads, prospects.
- Personas target por página:
  - `/` — hero direcionado a síndicos profissionais (principal).
  - `/pricing` — síndicos + empresas comparam planos.
  - `/para-sindicos`, `/para-empresas`, `/para-moradores` (M2) — landing por persona.
- Pós-CTA "Começar grátis" → redirect para `auth/sign-up?plano=trial`.

## Telas cobertas

| ID | Título | Rota | UI catalog |
|---|---|---|---|
| PUB1 | Home (hero) | `/` | [[../ui-catalog/marketing/_moc]] |
| PUB2 | Pricing | `/pricing` | [[../ui-catalog/marketing/_moc]] |
| PUB3 | Sobre | `/sobre` | [[../ui-catalog/marketing/_moc]] |
| PUB4 | Blog lista | `/blog` | [[../ui-catalog/marketing/_moc]] |
| PUB5 | Blog post | `/blog/$slug` | [[../ui-catalog/marketing/_moc]] |
| PUB6 | Contato | `/contato` | [[../ui-catalog/marketing/_moc]] |
| PUB7 | Termos | `/termos` | — |
| PUB8 | Privacidade (LGPD) | `/privacidade` | — |

## Funcionais (FR-WEB-LP-NNN)

### Home / Hero (PUB1)

- **FR-WEB-LP-001** — Hero com headline Michroma + subhead Manrope + CTA primary "Começar grátis" → `https://auth.mastersindico.com.br/_auth/sign-up?plano=trial`.
- **FR-WEB-LP-002** — Sistema de blobs OKLCH animados (design-system LP blob) respeitando `prefers-reduced-motion`.
- **FR-WEB-LP-003** — Seções: hero, features (3-4 cards), depoimentos (M2), logos clientes (M2 se houver), CTA final.
- **FR-WEB-LP-004** — Tema claro/escuro toggle em header; default `prefers-color-scheme`.

### Pricing (PUB2)

- **FR-WEB-LP-005** — Toggle mensal/anual (anual com desconto) — preços fetchados de `GET /public/plans` (source-of-truth backend, sem hardcode).
- **FR-WEB-LP-006** — 4 planos Stripe-style: `trial` (gratuito 14d), `base`, `plus`, `pro` — sem slugs persona-specific no código; apresentação persona-aware via query param `?persona=sindico|empresa|morador`.
- **FR-WEB-LP-007** — Cards com features comparativas (checkmark/X), destaque "Mais popular" configurável.
- **FR-WEB-LP-008** — FAQ pricing (accordion Kobalte) com 6-10 perguntas.
- **FR-WEB-LP-009** — CTA por plano → `auth/sign-up?plano=<plano>`.

### Blog (PUB4/PUB5)

- **FR-WEB-LP-010** — PUB4 lista paginada (10/pg) com busca + filtro por tag/categoria.
- **FR-WEB-LP-011** — PUB5 post individual com schema.org `Article`, TOC sidebar, share buttons (nativo Web Share API com fallback links), related posts.
- **FR-WEB-LP-012** — Conteúdo MDX ou markdown em `content/blog/*.md` (Rsbuild plugin).
- **FR-WEB-LP-013** — RSS feed `/blog/rss.xml` gerado em build.

### Contato (PUB6)

- **FR-WEB-LP-014** — Form nome/email/empresa/mensagem + dropdown tipo ("dúvida", "vendas", "parceria", "suporte").
- **FR-WEB-LP-015** — Validação Zod + reCAPTCHA v3 (ou Turnstile) + honeypot field.
- **FR-WEB-LP-016** — Submit → `POST /public/contact` backend; feedback sucesso/erro com retry.
- **FR-WEB-LP-017** — Consentimento LGPD explícito (checkbox) com link para PUB8.

### Legal (PUB7/PUB8)

- **FR-WEB-LP-018** — Páginas markdown estáticas + `last_updated` visível.
- **FR-WEB-LP-019** — Versionamento: exibir versão atual + link para histórico (M2).

### Navegação global

- **FR-WEB-LP-020** — Header fixo com logo Master Síndico (link `/`) + menu (Pricing, Sobre, Blog, Contato) + CTA "Entrar" + CTA "Grátis".
- **FR-WEB-LP-021** — Footer com 4 colunas (Produto, Empresa, Legal, Social) + newsletter signup + redes sociais + LGPD link.
- **FR-WEB-LP-022** — Mobile nav `Sheet` (Kobalte-based) com hamburger.

## Não-funcionais (NFR-WEB-LP)

- **Performance CRÍTICA** (single page decide conversão):
  - LCP < **1.5s desktop / 2.0s mobile** (stretch 1.8s mobile).
  - CLS < 0.05 (reservar espaço em hero images).
  - INP < 150ms.
  - TTFB < 300ms desktop (CDN Railway).
  - Bundle initial JS < **80 KB gzip** (budget estrito).
  - Imagens: AVIF + WebP via `<picture>`; hero `fetchpriority="high"`.
- **Acessibilidade**: WCAG 2.1 AA obrigatório + AAA aspiracional em hero (primeira impressão).
- **SEO** (obrigatório):
  - `<title>` + `<meta name="description">` únicos por página.
  - Schema.org JSON-LD: `Organization`, `WebSite`, `Article` (blog), `FAQPage` (pricing), `BreadcrumbList`.
  - `sitemap.xml` gerado em build (`/sitemap.xml`).
  - `robots.txt` permitindo tudo em `lp`, bloqueando outros subdomínios.
  - OG + Twitter cards (imagem 1200x630).
  - Canonical URLs.
  - Lang `pt-BR` + hreflang EN (preparado M3).
  - SSG preferido (Rsbuild prerender); SSR fallback para rotas dinâmicas.
  - Core Web Vitals verdes no PageSpeed Insights.
- **i18n**: pt-BR primário; EN preparado (lp tem mais razão para EN que outros apps — expansão).
- **Analytics**: Plausible (privacy-first) ou Umami self-hosted — **sem Google Analytics** (LGPD friendly).

## Rotas TanStack Router

```
src/routes/
├── __root.tsx               # header/footer layout
├── index.tsx                # PUB1
├── pricing.tsx              # PUB2
├── sobre.tsx                # PUB3
├── blog/
│   ├── index.tsx            # PUB4
│   └── $slug.tsx            # PUB5
├── contato.tsx              # PUB6
├── termos.tsx               # PUB7
└── privacidade.tsx          # PUB8
```

Todas estáticas pre-renderizadas em build.

## Estado

- **Minimal state** — lp é majoritariamente estático.
- **createSignal** — toggle mensal/anual, mobile menu open, blog filter, form fields.
- **TanStack Solid Query** — `['public','plans']` com `staleTime: 1h`; `['public','blog','posts',filters]`.
- **TanStack Solid Form** — PUB6 contato com Zod.
- **NO createStore global** — lp é leve por design.

## Integração backend

- **Base URL**: `http://localhost:8000/api/v1/public/*` / prod idem.
- **Sem auth** — endpoints públicos.
- **Rate limit backend**: 100 req/min por IP para `/public/*`.

### Endpoints consumidos

| Método | Path | Payload | Retorno |
|---|---|---|---|
| GET | `/public/plans` | — | `{trial,base,plus,pro}` com features + preços |
| GET | `/public/blog/posts?page&tag` | — | `{items,total}` |
| GET | `/public/blog/posts/:slug` | — | `{post,related}` |
| POST | `/public/contact` | `{nome,email,empresa,tipo,mensagem,captcha}` | 202 |
| POST | `/public/newsletter` | `{email,consent}` | 202 |

Blog **pode ser 100% estático** (MDX em build) sem backend — simplifica M1.

## Design system

- Design system dedicado LP blob (manual IV): blobs OKLCH animados, tipografia dual Michroma+Manrope, paleta navy+gold.
- Shortcut UnoCSS `lp-*` (ex: `lp-hero`, `lp-section`, `lp-card`, `lp-blob-bg`).
- `<Button variant="hero">` CVA variant exclusivo LP (glassmorphism + gold).
- Ícones Lucide via `i-lucide-*`.

## Segurança

- **CSP strict** com nonce; `script-src` allowlist Plausible/Umami domain.
- **XSS**: MDX/markdown blog via DOMPurify; frontmatter validado Zod.
- **CSRF**: `/public/contact` + `/public/newsletter` com captcha + honeypot + rate limit IP.
- **Clickjacking**: `frame-ancestors 'none'` — lp nunca embed em iframe de terceiros.
- **HTTPS + HSTS** obrigatórios em prod.
- **No PII em analytics** — Plausible privacy-first.

## Testes

- **Vitest** — 85% utilities (plan pricing formatter, blog frontmatter parser).
- **@solidjs/testing-library** — hero CTA click, pricing toggle, contato form validation.
- **MSW** — mock `/public/*`.
- **Playwright (E2E M2+)** — Lighthouse CI com thresholds CWV.
- **jest-axe** — todas 8 páginas.
- **Visual regression** (backlog M3) — Chromatic ou Percy.

## Status atual (real) — AUDIT A-FASE10-DEV-005

🟡 **Único app com conteúdo real**. Hero implementado com blob system. Mas: `AuthProvider`/`QueryClientProvider`/`Toaster` comentados em `apps/lp/src/index.tsx`; zero testes; rotas PUB2-PUB8 não implementadas; sitemap/robots/OG ausentes; `@tanstack/solid-query` não instalado. Hero é estático (sem fetch); aceita.

Ação M1:
1. Remover comentários de provider onde aplicável (lp é apenas Query, não Auth).
2. Implementar PUB2 (Pricing) — crítico para trial conversion.
3. Implementar PUB6 (Contato) simples.
4. Gerar `sitemap.xml` + `robots.txt` + OG tags no build.

## Gaps/decisões abertas

- **G1**: Rsbuild SSG prerender — plugin oficial maduro? Fallback build manual de HTML estático via script.
- **G2**: Blog MDX: `solid-mdx` é experimental — alternativa markdown + marked + highlight.js.
- **G3**: reCAPTCHA v3 vs Turnstile (Cloudflare) — Turnstile mais LGPD-friendly.
- **G4**: Plausible vs Umami — Plausible cloud vs Umami self-hosted Railway.
- **G5**: i18n EN — quando ativar? Decisão comercial; arquitetura pronta.
- **G6**: Blog CMS (M2) — headless (Strapi? Payload?) ou só MDX em git?
- **G7**: Newsletter — integração (ConvertKit? Listmonk self-hosted?) — M2+.

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../architecture]]
- [[../design-system]]
- [[../security]]
- [[../requirements]]
- [[../ui-catalog/marketing/_moc]]
- [[../../../00-product/portfolio-de-produtos]]
