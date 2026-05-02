---
title: README.md SPEC — estrutura canônica de README de projeto
type: spec
tags:
  - template
  - spec
  - readme
created: '2026-04-22'
updated: '2026-04-27'
---
# README_SPEC.md

Template e guia para produzir um `README.md` de qualquer projeto. Destilado de READMEs reais de backend (Go), monorepo (TypeScript/Bun), API e web.

Três camadas:
1. **Princípios** — a quem o README serve e como difere do CLAUDE.md/AGENTS.md
2. **Estrutura canônica** — seções obrigatórias + opcionais
3. **Templates** — snippets por tipo de projeto (monorepo, backend, frontend, mobile)

---

## 1. Princípios

### 1.1 Três leitores, três documentos

| Documento | Leitor | Pergunta que responde |
|---|---|---|
| `README.md` | **humano (dev novo, recruiter, curioso)** | O que é, como rodar, como contribuir |
| `CLAUDE.md` | **agente de IA** | Como operar aqui com contexto mínimo |
| `AGENTS.md` | **agente ou dev sênior** | Como escrever código seguindo os padrões |

README é a **porta de entrada humana** — não repete comando do CLAUDE.md, não repete regra do AGENTS.md. README responde: "o que isso resolve, que stack usa, como rodar em 15 minutos, que sprints já rolaram, qual o status".

### 1.2 Tom

- **Produto-first**: começa pelo problema e pela solução, não pela stack.
- **Tabelas** onde couber — stack, env vars, personas, sprints, rotas.
- **Código executável** em todo bloco — `docker compose up -d`, não "rode o Docker".
- **Status visível** — checklist emoji (`✅ 🔨 📋 ⏳`) para sprints/features.
- **Idioma do negócio** — se o produto é pt-BR, README é pt-BR. Identifiers/comandos mantêm inglês.
- **Zero filler** — sem "Welcome to our amazing project", sem agradecimentos, sem boilerplate gerado por scaffolder (`flutter create`/`bun init`/`create-next-app` produzem lixo — deletar).

### 1.3 Fronteiras

README **não** vive:
- Padrões arquiteturais profundos → `AGENTS.md`
- Ciclo SDD, fontes de verdade, ordem de consulta → `CLAUDE.md`
- Specs (requirements/design/tasks) → `.kiro/specs/`
- Decisões arquiteturais (ADRs) → `.kiro/specs/<produto>/decisions/`
- Troubleshooting exaustivo → `CLAUDE.md` ou `docs/`

README **vive**:
- Visão de produto
- Stack (com papel de cada item)
- Arquitetura em alto nível (árvore de diretórios)
- Setup local
- Env vars
- Scripts principais
- Roadmap/sprints com status
- Modelo de dados (quando aplicável)

---

## 2. Estrutura canônica

Seções em ordem (obrigatórias em **negrito**):

```
1.  **Título + descrição de uma linha**
2.  **Visão do produto** (ou "Problema / Solução")
3.  **Stack** (tabela "Camada × Tecnologia")
4.  **Arquitetura** (tree + 2-3 linhas de narrativa)
5.  Personas / Planos / Modos de uso (quando aplicável)
6.  **Pré-requisitos**
7.  **Instalação**
8.  **Variáveis de ambiente** (tabela)
9.  **Scripts** (bash executável, comentado)
10. Auth Flow (para projetos com auth custom/OIDC)
11. Endpoints / Health checks (para API)
12. Modelo de dados (para backend)
13. Observabilidade (OTel, Prometheus, dashboards)
14. Roadmap / Sprints (tabela com status)
15. Decisões técnicas (opcional — link para ADRs)
16. Licença
```

---

## 3. Templates por tipo

### 3.1 Monorepo — visão de produto

```markdown
# <Produto> — <Tagline>

Plataforma/SaaS/CLI/etc. que <problema-que-resolve> para <personas>. Conecta <atores> em um ecossistema de <valor-principal>.

> <Produto> não é X, não é Y, não é Z.
> É uma plataforma de **<categoria-correta>** aplicada a <domínio>.

## O Problema

<Domínio> sofre de:
- <dor-1>
- <dor-2>
- <dor-3>
- <dor-4>

## A Solução

<Produto> estrutura <processo>:
- **<pilar-1>** → <como-resolve>
- **<pilar-2>** → <como-resolve>
- **<pilar-3>** → <como-resolve>
- **<pilar-4>** → <como-resolve>

---

## Arquitetura do Monorepo

```
<repo-root>/
├── apps/
│   ├── api/               → Backend — <framework> (<arquitetura>)
│   └── web/               → Frontend — <framework>
├── packages/
│   └── schemas/           → Schemas compartilhados (contrato API ↔ Web)
├── contextos/             → Documentação de negócio (PDFs, análises)
├── docs/                  → Gap analysis e status vs escopo
├── plans/                 → Planos de arquitetura e implementação
├── <orquestrador>.json    → Orquestração de tasks
└── package.json           → Workspace root
```

## Tech Stack

| Camada | Tecnologia |
|---|---|
| **Monorepo** | <turborepo/nx/bun-workspaces> |
| **Backend** | <framework + lang + versão> |
| **Frontend** | <framework + bundler> |
| **Banco** | <db + orm/query-builder> |
| **Cache** | <cache> |
| **Autenticação** | <auth-method> |
| **Autorização** | <abac/rbac> — <engine> |
| **Pagamentos** | <gateway> |
| **Validação** | <schema-lib> |
| **DI** | <di-lib> |
| **Estilização** | <css-engine> |
| **Roteamento** | <router> |
| **Data Fetching** | <query-lib> |
| **Forms** | <form-lib> |
| **Componentes UI** | <headless-lib> |
| **Linting** | <linter> |
| **API Docs** | <openapi-gen> |

## Personas e Planos

| Persona | Planos | Descrição |
|---|---|---|
| **<persona-1>** | <tiers> | <uso-principal> |
| **<persona-2>** | <tiers> | <uso-principal> |
| **<persona-N>** | <tiers> | <uso-principal> |

> Permissões governadas pela **Matriz Funcional** — cada plano tem conjunto exato de features habilitadas.
```

### 3.2 Backend (API) — template completo

```markdown
# <Produto> — Backend

API do ecossistema <produto>.

## Stack

- **<lang + versão>** + <framework> (abstraído via <contracts-layer>)
- **<banco + versão>** (<driver> + <query-gen>)
- **<cache + versão>** (<client> — sessões, ABAC cache, quotas)
- **<queue>** (pub/sub assíncrono)
- **<oidc-provider>** (OIDC + JWT introspection)
- **<payment-gateway>** (billing via interface)
- **<video-provider>** (VOD + live via interface)
- **<storage>** (S3-compatible via interface)
- **<live-provider>** (WebRTC via interface)
- **<email-provider>** (transacional via interface)
- **OpenTelemetry** (traces + métricas Prometheus)

## Arquitetura

Clean Architecture + DDD + CQRS + Vertical Slices. Cada bounded context é um módulo autocontido. Dependências apontam para dentro: `infrastructure → application → domain`.

```
cmd/api/main.go
internal/
  core/
    contracts/        ← IModule, IUseCase[I,O], IRepository[T,ID], IUnitOfWork
    errors/           ← AppError e variantes
  modules/
    <ctx-1>/          ← <descrição-do-contexto>
    <ctx-2>/          ← <descrição-do-contexto>
    <ctx-N>/          ← <descrição-do-contexto>
  shared/
    config/           ← loader + validação de config
    middleware/       ← cadeia HTTP compartilhada
    providers/
      cache/          ← <cache> (interface + impl)
      database/       ← pool + UnitOfWork (tx via context)
pkg/
  logger/             ← structured logger wrapper
  server/             ← engine, health checks, error handler
```

## Pré-requisitos

- <lang + versão mínima>
- Docker + Docker Compose

## Ambiente local

```bash
docker compose up -d       # Sobe <deps-externas>
cp .env .env.local         # Copiar e ajustar variáveis
<run-cmd>                  # API em http://localhost:<porta>
```

## Health checks

```
GET /health        → liveness (sempre 200)
GET /health/ready  → readiness (verifica <deps-críticas>)
```

## Migrations

Gerenciadas com <tool> + embedding (dentro do binário).

```
internal/shared/providers/database/migrations/
  00001_<desc>.sql
  00002_<desc>.sql
  ...
```

Em dev, migrations rodam automaticamente no startup. Em prod, binário separado `cmd/migrate`.

## Variáveis de ambiente

| Variável | Descrição | Default |
|---|---|---|
| `APP_ENV` | `development` / `staging` / `production` | `development` |
| `APP_PORT` | Porta da API | `<porta>` |
| `DB_HOST` | Host do banco | `localhost` |
| `DB_NAME` | Nome do banco | `<nome>` |
| `REDIS_HOST` | Host do cache | `localhost` |
| `<OIDC>_DOMAIN` | Domínio do provedor OIDC | — |
| `<OIDC>_KEY_PATH` | Path do service account key | — |
| `<OIDC>_PROJECT_ID` | ID do projeto | — |
| `<OIDC>_WEB_CLIENT_ID` | Client ID do App Web PKCE | — |
| `<OIDC>_WEB_REDIRECT_URI` | URI de callback | `http://localhost:<porta>/auth/callback` |
| `<OIDC>_WEB_ENCRYPTION_KEY` | Chave para cookies PKCE | — |
| `<PAYMENT>_KEY_PRIV` | API key privada | — |
| `<PAYMENT>_WEBHOOK_SECRET` | Secret para validar webhook | — |
| `OTEL_ENABLED` | Liga SDK OTel | `false` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | URL base OTLP | — |
| `OTEL_PROMETHEUS_ENABLED` | Habilita `/metrics` | `false` (dev) |
| `OTEL_SAMPLE_RATE` | Fração de traces | `1.0` (dev), `0.1` (prod) |

Ver `.env` para a lista completa.

## Auth Flow (OIDC PKCE)

```
Browser → GET /auth/login → <OIDC> Login UI → GET /auth/callback
       → backend seta cookie access_token httpOnly → redirect para frontend

Mobile → SDK nativo (PKCE) → token em secure storage
       → requests com Authorization: Bearer <token>

Ambos → GET /api/v1/auth/me → middleware valida via introspection
```

### Rotas de auth

```
GET  /auth/login     → inicia OIDC (pública)
GET  /auth/callback  → troca code por token (pública)
GET  /auth/logout    → end_session + delete cookie (pública)
GET  /api/v1/auth/me → retorna usuário autenticado (protegida)
```

## Geração de código

```bash
<codegen-tool> generate     # Regenerar após alterar <trigger>
```

Output em `<gen-path>`. **Não editar manualmente.**

## Modelo de dados

```
users
  id, <oidc>_id, email, username, phone
  role (<lista-roles>)
  role_type (<lista-subtipos>)
  plan_id → plans
  status, <feature-flags>
  device_fingerprint, last_ip

sessions
  id, user_id → users
  <oidc>_session_id
  ip_address, user_agent, device_fingerprint
  expires_at

plans
  id, role, tier (<tiers>)
  name, price_monthly, price_yearly
  <payment>_product_id, <payment>_price_monthly_id, <payment>_price_yearly_id
  UNIQUE (role, tier)

plan_features
  plan_id → plans
  feature_key, feature_value, feature_type

subscriptions
  id, user_id → users, plan_id → plans
  status (<lista-status>)
  current_period_start, current_period_end
  trial_ends_at, canceled_at
  <payment>_customer_id, <payment>_subscription_id
```

## Observabilidade

Backend expõe telemetria via OpenTelemetry SDK.

### Instrumentação automática

- **HTTP**: todo request recebe span OTel. `trace_id` e `span_id` injetados nos logs.
- **Banco**: toda query gera span com `db.system.name=<driver>` e `db.query.text`.
- **Propagação**: W3C TraceContext + Baggage — compatível com Jaeger, X-Ray.

### Endpoint de métricas

```
GET /metrics     → Prometheus scrape (quando OTEL_PROMETHEUS_ENABLED=true)
```

---

## Sprint <N> — <nome> (<data>)

| Task | Status |
|---|---|
| <N>.1 <descrição> | ✅ |
| <N>.2 <descrição> | 🔨 |
| <N>.3 <descrição> | 📋 |

### Pendências (Sprint <N> → Marco <M>: <data>)

- [ ] <pendência-1>
- [ ] <pendência-2>

---

## Status dos módulos

| Módulo | Endpoints | Status |
|---|---|---|
| `<ctx-1>` | <lista-rotas> | ✅ |
| `<ctx-2>` | <lista-rotas> | ✅ |
| `<ctx-N>` | <lista-rotas> | 🔨 |

> **<total> endpoints** — ver `routes.http` para exemplos.

### Rotas de saúde e observabilidade

```
GET  /health          → liveness probe
GET  /health/ready    → readiness (<deps>)
GET  /metrics         → Prometheus (quando habilitado)
```
```

### 3.3 Frontend — template mínimo

```markdown
# <Produto> — Web

Frontend da plataforma <produto>. SPA construída com **<framework>** + **<bundler>**, usando o ecossistema <router-query> para roteamento e data-fetching, e <css-engine> para estilização.

## Tech Stack

| Categoria | Tecnologia |
|---|---|
| **Framework** | <framework + versão> |
| **Bundler** | <bundler> |
| **Roteamento** | <router> (file-based) |
| **Data Fetching** | <query-lib> |
| **Forms** | <form-lib> + <schema-adapter> |
| **Estilização** | <css-engine> (attributify + icons + typography + web-fonts) |
| **Componentes** | <headless-lib> + <variance> |
| **HTTP Client** | <http> |
| **Validação** | <schema-lib> (via `<shared-package>`) |
| **Linting** | <linter> |

## Estrutura

```
src/
├── index.<ext>           → Entrypoint (Router + QueryClient + Providers)
├── main.css              → Estilos globais
├── <route-gen-file>      → Gerado automaticamente
├── components/
│   ├── auth/             → Componentes de autenticação
│   ├── form/             → Formulário reutilizável
│   ├── handlers/         → Error/loading handlers
│   ├── layouts/          → Layouts base
│   ├── shared/           → Componentes genéricos
│   └── ui/               → Design system base
├── context/              → Providers (auth, theme)
├── hooks/                → Custom hooks
├── lib/                  → HTTP instance, helpers
└── routes/               → File-based routing
    ├── <root>.<ext>      → Layout raiz
    ├── _auth/            → Rotas de autenticação
    ├── _private/         → Rotas protegidas
    └── _public/          → Rotas públicas
```

## Roteamento

File-based. `<route-gen-file>` é gerado automaticamente pelo plugin do router — **não editar**.

### Route Groups

| Grupo | Prefixo | Descrição |
|---|---|---|
| `_auth` | `/auth/*` | Login, cadastro, verificação |
| `_private` | `/*` | Autenticadas (requer <auth>) |
| `_public` | `/*` | Públicas (landing, busca) |

## Comunicação com a API

```
Web (<framework>) ←→ <http-lib> ←→ API (<backend>:<porta>)
         ↖________________________↗
              <shared-schemas-package>
```

- Schemas request/response compartilhados via `<package>`
- Validação client-side e server-side usando os mesmos schemas
- <query-lib> gerencia cache, refetch e estados

## Scripts

```bash
<cmd-dev>        # Desenvolvimento (hot-reload) — http://localhost:<porta>
<cmd-build>      # Build de produção
<cmd-preview>    # Preview do build local
<cmd-check>      # Lint
<cmd-format>     # Format
```
```

### 3.4 Flutter/mobile — template mínimo

```markdown
# <produto>

<Descrição curta>.

## Stack

- <lang> <sdk-version>
- <state-mgmt>
- <di>
- <router>
- <http>
- <functional-either> (tratamento de erros)
- <testing-stack>
- <secure-storage>
- <oidc-lib>

## Estrutura por Feature (Clean Architecture)

```
lib/
  app/            ← bootstrap, DI, router, theme
  core/
    network/      ← HTTP client + interceptors
    auth/         ← OIDC integration
    storage/      ← secure storage wrapper
  features/
    <feature>/
      data/         ← repositories impl, datasources
      domain/       ← entities, use cases, repository interfaces
      presentation/ ← state mgmt, screens, widgets
```

## Comandos

```bash
<cmd-install>
<cmd-run>
<cmd-test>
<cmd-analyze>

# Code generation
<cmd-codegen>
```

## Roteiro

Pelo `<backend>/.kiro/specs/<produto>/tasks/`:

- **<sprint-N>**: <descrição>
- **<sprint-N+1>**: <descrição>
```

---

## 4. Descartar — boilerplate de scaffolder

READMEs gerados por `<framework> create` / `bun init` / `create-next-app` têm **zero valor** no repo real. Detectar pelos sinais:

- "A new <framework> project."
- "Getting Started — This project is a starting point..."
- "Learn <framework>" / lista de links pra docs oficiais
- "This project was created using `<scaffold-cmd>` in <framework> v<x>"
- Título genérico idêntico ao nome da pasta (`mastersindico`, `my-app`)

**Ação**: deletar o README gerado e escrever o seu do zero seguindo o template §3.

---

## 5. Como manter atualizado (ligação com SDD)

README **evolui com as sprints**. Na fase **Ship** do ciclo SDD:

- **Feature-level change** (novo módulo, nova persona, nova rota pública) → atualizar README
- **Bug-fix** → **não** atualiza README; muda apenas AUDIT.md
- **Nova sprint iniciada** → adicionar seção `## Sprint <N+1>` com tasks
- **Sprint concluída** → mover tasks para ✅, adicionar "Pendências" da próxima
- **Novo módulo entregue** → adicionar linha em "Status dos módulos"
- **Novos endpoints** → atualizar `routes.http` + contagem em "Status dos módulos"

Dica: pedir ao skill `claude-md-management` (ou equivalente) que revise coerência entre README, CLAUDE.md, AGENTS.md após cada sprint.

---

## 6. Checklist de qualidade

Antes de mergear:

- [ ] Título começa com `# <Produto> — <tagline>`
- [ ] Descrição de 1 linha logo abaixo
- [ ] Blockquote com "<Produto> não é X, é Y" (quando monorepo/plataforma)
- [ ] Seção "O Problema" com 3-5 dores
- [ ] Seção "A Solução" com 3-5 pilares
- [ ] Stack em tabela com **versões explícitas**
- [ ] Arquitetura em ``` tree com comentário `←` em cada linha-chave
- [ ] Personas / Planos (se aplicável) em tabela
- [ ] Pré-requisitos com versões mínimas
- [ ] Instalação em ≤ 5 passos, todos `bash` executável
- [ ] Env vars em tabela (Variável / Descrição / Default)
- [ ] Auth flow diagramado em ```  ```  (ASCII) — para projeto com auth
- [ ] Rotas de health check com verbo + path + descrição
- [ ] Modelo de dados em ``` tree — para backend
- [ ] Sprints em tabela com emoji de status (✅ 🔨 📋 ⏳)
- [ ] Pendências da sprint atual como `[ ]` checklist
- [ ] Observabilidade se OTel habilitado
- [ ] Licença (proprietary / MIT / Apache / etc.)
- [ ] **Zero duplicação** com CLAUDE.md e AGENTS.md
- [ ] **Zero boilerplate** de scaffolder
- [ ] **Zero filler** ("Welcome to...", agradecimentos, etc.)

---

## 7. Como replicar em um novo projeto

1. Escolher template conforme tipo (monorepo, backend, frontend, mobile).
2. Copiar snippet §3 e preencher placeholders `<...>` com valores reais.
3. Descrever "O Problema" ouvindo stakeholder real — **nunca** inventar dores.
4. Listar stack com **versões** — grepar `package.json`, `go.mod`, `pubspec.yaml`, `Cargo.toml` para não errar.
5. Desenhar árvore de arquitetura do código **real**, não do ideal.
6. Modelo de dados: copiar do último migration + anotar FKs em setas `→`.
7. Primeira sprint: listar tasks concluídas + pendentes + próximas (ler `.kiro/specs/<produto>/tasks.md`).
8. Rodar o checklist §6 antes de commitar.
9. Pedir review da equipe.
10. Na próxima feature, atualizar incrementalmente na fase Ship.
