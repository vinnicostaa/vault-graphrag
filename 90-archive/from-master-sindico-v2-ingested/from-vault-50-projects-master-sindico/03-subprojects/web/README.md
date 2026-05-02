---
title: Web Master Síndico — README
type: project
tags: [master-sindico, web, solidjs, typescript, frontend]
project: master-sindico
subproject: web
created: 2026-04-22
---

# Web — Master Síndico

Portal web responsivo (mobile-first) — síndico, empresa, morador (fallback quando sem app), admin interno. Stack **SolidJS + TypeScript**.

Repo local: `Development/web/`.

---

## Stack

| Camada | Tech | Motivo |
|---|---|---|
| Framework | SolidJS + Solid Router | Reatividade fina-grained; bundle pequeno; performance |
| Linguagem | TypeScript 5.x (strict) | Type safety ponta-a-ponta |
| Build | Vite 5 | HMR rápido, output ESM |
| Estado | SolidJS stores + contexts | Nativo; sem Redux |
| Styling | Tailwind CSS 4.x | Utility-first, design tokens |
| Forms | solid-forms / manual | Controle fino |
| i18n | @solid-primitives/i18n | Pt-BR first, EN prep |
| HTTP | fetch nativo + axios quando precisa interceptors | axios 1.15 com CVE audit (ver [[../../11-audit/AUDIT]] F25) |
| Realtime | WebSocket nativo + ReconnectingWebSocket | Assembleia live |
| Testing | Vitest (unit) + Playwright (E2E) | |
| Observability | Sentry browser SDK | Erros em produção |
| Deploy | Railway static serve ou Vercel | A decidir (D-###) |

---

## Estrutura

```
web/
├── index.html
├── src/
│   ├── main.tsx                    # entry
│   ├── App.tsx                     # router root
│   ├── routes/                     # file-based routing
│   │   ├── (public)/
│   │   │   ├── login.tsx
│   │   │   └── signup.tsx
│   │   ├── (authenticated)/
│   │   │   ├── condominium/
│   │   │   │   ├── [id]/dashboard.tsx
│   │   │   │   ├── [id]/units.tsx
│   │   │   │   └── [id]/assembly/
│   │   │   ├── connect-me/
│   │   │   └── account/
│   │   └── (admin)/                # backoffice interno
│   ├── features/                   # feature slice por bounded context
│   │   ├── identity/ billing/ institutional/ commercial/ content/ assembly/
│   ├── ui/                         # design system components (atoms, molecules, organisms)
│   ├── core/
│   │   ├── api/                    # http client + generated OpenAPI types
│   │   ├── auth/                   # session bootstrap, cookie parsing
│   │   ├── i18n/                   # traduções (pt-BR, en)
│   │   └── config/
│   ├── lib/                        # helpers puros
│   └── styles/
├── public/
├── tests/
│   ├── unit/                       # Vitest
│   └── e2e/                        # Playwright
├── vite.config.ts
├── tsconfig.json
├── tailwind.config.ts
├── package.json
└── .env.example
```

---

## Princípios arquiteturais (feature-sliced)

1. **Features independentes** — cada bounded context = pasta em `features/`; não importam entre si sem passar por `core/api`
2. **UI components sem lógica de domínio** — só props, eventos, estado local
3. **API client tipado** gerado do OpenAPI do backend (`02-architecture/openapi/*.yaml`)
4. **Zero dependências do domínio no frontend** — apenas shapes de DTOs
5. **Route-based code split** (lazy) pra bundle pequeno no cold start
6. **Accessibility WCAG AA** desde o início (ARIA, keyboard, contraste)
7. **Mobile-first responsive** com breakpoints Tailwind

---

## Comandos

```bash
npm install            # ou pnpm / bun
npm run dev            # vite dev server em :5173
npm run build          # build de produção (output em dist/)
npm run preview        # preview do build

npm run test           # vitest
npm run test:e2e       # playwright (headless)
npm run test:e2e:ui    # playwright com UI

npm run lint           # eslint + prettier
npm run typecheck      # tsc --noEmit

# gera types a partir do OpenAPI do backend
npm run gen:api
```

---

## Auth flow

1. User vai em `/login` → redirect pra Zitadel (`/auth/login` do backend que faz PKCE)
2. Zitadel autentica → redirect pra `/auth/callback` do backend
3. Backend seta cookie httpOnly `session` em `.mastersindico.com.br`
4. Backend redireciona pra `origin` (SPA) com `?authenticated=1`
5. SPA chama `GET /api/v1/auth/me` pra popular store
6. Logout: SPA chama `/auth/logout` do backend

---

## Env vars

`.env.local` (gitignored):

```bash
VITE_API_BASE_URL=http://localhost:8000
VITE_ENV=development
VITE_SENTRY_DSN=
VITE_FIGMA_PROJECT=https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/
```

---

## Telas por sprint (alinhado com backend)

- Sprint 10 (M1): login, signup síndico, dashboard condomínio, unidades, vínculo de moradores, minha assinatura
- Sprint 11 (M2): Connect Me (4 fluxos), empresas, marketplace, player Mux
- Sprint 12 (M3): assembleia live, votação, atas, LMS

---

## Specs

**Web não mantém specs próprias**. Consome specs canônicas de `backend/.kiro/specs/master-sindico/` (ver [[../../specs/reference/README]]).

O arquivo `web/.kiro/specs/master-sindico/` foi **DEPRECATED** em 2026-04-22 (A-017); tinha cópias stale com stack errada (Fastify/SES/ECS). Tratamento: remover e apontar pro backend.

---

## Links

- [[../_moc]]
- [[../../CLAUDE]]
- [[../../02-architecture/_moc]]
- [[../../04-requirements/_moc]]
- [[../../05-tasks/_moc]]
- [[../../../../20-stacks/typescript/frontend-solidjs]]
- [[architecture]] *(a criar)*
- [[patterns]] *(a criar)*
- [[requirements]] *(a criar)*
- [[tasks]] *(a criar)*
- [[security]] *(a criar)*
- [[design-system]] *(a criar)*
