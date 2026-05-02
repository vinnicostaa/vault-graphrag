# Master Síndico — Plataforma de Governança Condominial

Plataforma SaaS de gestão condominial que conecta **síndicos**, **moradores**, **empresas do mercado condominial** e **comércio local** em um ecossistema integrado de governança, capacitação e visibilidade.

> My Síndico não é um ERP, não é app de comunicação, não é sistema financeiro.
> É uma plataforma de **governança aplicada** à gestão condominial.

## O Problema

A gestão condominial sofre de ausência de estrutura decisória:
- Decisões tomadas por preço, pressão ou urgência
- Sem registro, sem contexto, sem memória institucional
- Síndico sobrecarregado, sem ferramentas adequadas
- Ecossistema desconectado (moradores, empresas, comércio)

## A Solução

My Síndico estrutura o processo de decisão, registro e reputação:
- **Critério** → Contratações baseadas em método, não em urgência
- **Registro** → Memória institucional auditável
- **Reputação** → Sistema de status baseado em evidência (Bronze → Diamante)
- **Integração** → Ecossistema unificado de perfis, vídeos, cursos e contato

---

## Arquitetura do Monorepo

```
full-stack/
├── apps/
│   ├── api/               → Backend — Fastify + DDD (Clean Architecture)
│   └── web/               → Frontend — SolidJS + Rsbuild
├── packages/
│   └── schemas/           → Zod schemas compartilhados (contrato API ↔ Web)
├── contextos/             → Documentação de negócio (PDFs, análises, contextos)
├── docs/                  → Gap analysis e status vs escopo
├── plans/                 → Planos de arquitetura e implementação
├── turbo.json             → Orquestração de tasks (Turborepo)
└── package.json           → Workspace root (Bun)
```

## Tech Stack

| Camada | Tecnologia |
|---|---|
| **Monorepo** | Turborepo + Bun 1.3 |
| **Backend** | Fastify 5, TypeScript 5.9, DDD (Clean Architecture) |
| **Frontend** | SolidJS 1.9, Rsbuild |
| **Banco de Dados** | PostgreSQL + Drizzle ORM 1.0-beta |
| **Cache** | Redis (@fastify/redis) |
| **Autenticação** | JWT (jose) + Argon2 + OAuth via Arctic (Google PKCE) |
| **Autorização** | ABAC (CASL) — permissões dinâmicas por plano via banco |
| **Pagamentos** | Stripe (subscriptions, webhooks, refunds) |
| **Validação** | Zod v4 (compartilhado via `mastersindico-schemas`) |
| **DI** | Awilix + @fastify/awilix |
| **Estilização** | UnoCSS (attributify, icons, typography, web-fonts) |
| **Roteamento Web** | TanStack Solid Router (file-based) |
| **Data Fetching** | TanStack Solid Query |
| **Forms** | TanStack Solid Form + Zod adapter |
| **Componentes UI** | Kobalte (headless) + CVA |
| **Linting** | Biome |
| **API Docs** | Swagger + Scalar (gerada dos schemas Zod) |

## Personas e Planos

| Persona | Planos | Descrição |
|---|---|---|
| **Morador** | Base, Pagante | Consumo de conteúdo, busca, vídeo-currículo (pg) |
| **Síndico** | N1 (Gestor), N2 (Plus), N3 (Pro) | Gestão condominial, vídeos, assembleias, status |
| **Empresa** | Plus, Pro | Vídeos institucionais, cursos, Connect Me |
| **Marketing** | Standard | Publicação de conteúdo, marketing link |
| **Vizinhança** | Standard | Comércio local por CEP, cupons |

> Permissões são governadas pela **Matriz Funcional** — cada plano tem um conjunto exato de features habilitadas.

## Sprints (Roadmap)

| Sprint | Foco | Status |
|---|---|---|
| **1** | Fundação: Auth, ABAC, Perfis, Onboarding, Busca, Connect Me | 🔨 Em progresso |
| **2** | Conteúdo: Vídeos (upload, trava 90 dias), galeria | 📋 Planejado |
| **3** | Gestão: Assembleias, votações, plano diretor | 📋 Planejado |
| **4** | Educação: LMS (cursos, módulos, certificados) | 📋 Planejado |
| **5** | Engajamento: Fórum, cupons, gamificação | 📋 Planejado |

## Pré-Requisitos

- **Node.js** >= 25
- **Bun** >= 1.3.5
- **PostgreSQL** (local ou Docker)
- **Redis** (local ou Docker)

## Instalação

```bash
git clone <repo-url>
cd full-stack
bun install
```

## Variáveis de Ambiente

Crie `apps/api/.env`:

```env
# Server
HOST=0.0.0.0
PORT=8000
NODE_ENV=development
LOG_LEVEL=info

# Database & Cache
DATABASE_URL=postgresql://user:pass@localhost:5432/mastersindico
REDIS_URL=redis://localhost:6379

# Auth
JWT_SECRET=...
COOKIE_SECRET=...
SESSION_EXPIRATION_DAYS=30
SESSION_MAX_ACTIVE_SESSIONS=5
VERIFICATION_EXPIRATION_MINS=15

# OAuth (Google)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GOOGLE_CALLBACK_URL=http://localhost:8000/v1/auth/oauth/google/callback

# Email (Resend ou Gmail)
RESEND_API_KEY=...
RESEND_SENDER_EMAIL=noreply@mastersindico.com.br
GMAIL_USER=...
GMAIL_APP_PASSWORD=...
GMAIL_SENDER_EMAIL=...

# Frontend
FRONTEND_URL=http://localhost:3000
CORS_ORIGIN=http://localhost:3000

# Stripe
STRIPE_KEY_PUB=pk_test_...
STRIPE_KEY_PRIV=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

## Scripts

```bash
# Dev (API + Web em paralelo via Turborepo)
bun run dev

# Apenas API ou Web
bun run dev --filter=api
bun run dev --filter=web

# Build de produção
bun run build

# Type checking em todo o monorepo
bun run check-types

# Lint
bun run lint
```

## Documentação de Contexto

A pasta `contextos/` contém todo o material de referência do projeto:

| Documento | Conteúdo |
|---|---|
| `novo escopo-7.pdf` | Escopo técnico e cronograma (SOW) — 5 sprints detalhadas |
| `Manual Técnico My sindico.pdf` | Documento conceitual completo — 8 capítulos sobre governança |
| `Matriz Funcional.pdf` | Matriz de permissões: feature × plano (9 tiers) |
| `Manual Identidade Visual.pdf` | Branding: cores, tipografia, logotipos |
| `ANALISE_COMPLETA_API.md` | Análise detalhada da API existente |
| `ANALISE_COMPLETA_MODELAGEM.md` | Análise da modelagem de dados |
| `DATABASE_VALIDATION.md` | Validação do schema do banco |
| `stripe-context.md` | Contexto de integração com Stripe |

## Regras de Negócio Chave

1. **Trava Trimestral (90 dias)** — Vídeos institucionais/currículo só podem ser atualizados 1x a cada 90 dias
2. **Privacidade de Métricas** — Likes/comentários visíveis apenas para o dono do conteúdo
3. **Onboarding Parametrizado** — URL com `?role=sindico&plan=pro` define perfil automaticamente
4. **Status do Síndico** — Bronze/Prata/Ouro/Diamante baseado em consumo, atuação e capacitação
5. **Connect Me** — Formulário unidirecional (não é chat, não salva histórico)
6. **Visibility Engine** — Perfil só aparece na busca se o plano permite

## Licença

Proprietário — Master Síndico © 2025
