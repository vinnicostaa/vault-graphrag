---
title: CLAUDE.md SPEC — contrato do agente
type: spec
tags:
  - template
  - spec
  - claude-md
  - agent-contract
created: '2026-04-22'
updated: '2026-04-27'
---
# CLAUDE_SPEC.md

Template e guia para produzir um `CLAUDE.md` de qualquer projeto. Destilado de implementações reais em 4 stacks (Go backend, SolidJS web workspaces, Flutter mobile, Fastify legacy).

Três camadas:
1. **Princípios** — o papel do `CLAUDE.md` e por que existe
2. **Estrutura canônica** — seções obrigatórias + seções opcionais
3. **Templates por stack** — snippets parametrizados (backend/web/mobile/monorepo/raiz multi-projeto)

---

## 1. Princípios

### 1.1 O que é o CLAUDE.md

Guia **de primeira leitura** carregado pelo Claude Code ao abrir o repositório. Função: dar ao agente o mínimo necessário para ser produtivo sem forçá-lo a ler o código inteiro.

- **Deve ser enxuto** — alvo ~100 linhas na raiz de cada projeto. Detalhes vivem em arquivos carregados sob demanda (`.kiro/steering/*.md`, `AGENTS.md`, docs por módulo).
- **Alto prompt-caching** — o arquivo é lido toda sessão; se muda raramente, o cache do provedor amortiza.
- **Ordem de leitura explícita** — diz ao agente **em que ordem** consultar fontes de verdade.
- **Não duplica conteúdo** de `AGENTS.md` ou steering — aponta para eles.
- **Um CLAUDE.md por sub-projeto** em monorepo/workspace multi-stack. O da raiz só cobre o que **spans** os sub-projetos.

### 1.2 Fontes de verdade (ordem canônica)

Qualquer projeto que adota SDD deve expor esta ordem no `CLAUDE.md`:

1. **Charter da sessão ativa** (se houver) — ordens da sessão corrente: `.kiro/SESSION_CHARTER.md` ou equivalente.
2. **Specs do produto**: `.kiro/specs/<produto>/` — `requirements/`, `design/`, `tasks/`. Primeira parada para qualquer implementação.
3. **Steering técnico**: `.kiro/steering/*.md` — regras duras por tópico, carregadas sob demanda.
4. **Estado**: `.kiro/STATE.md` — bloqueadores vivos + decisões em aberto (`D-0XX`).
5. **Audit**: `.kiro/AUDIT.md` — findings pendentes (`A-0XX`). **Gate 0** de qualquer sessão: se houver `🔴 Aberto` ou `🟡 Em andamento`, resolver antes da próxima task.
6. **Contrato do agente**: `AGENTS.md` — papel, princípios, quando recusar, padrões completos.
7. **Código existente** — quando spec e código divergem, ler o código e atualizar a spec.

**Regra dura**: se spec não existe ou está ambígua → **parar e atualizar spec antes de implementar**.

### 1.3 Ciclo obrigatório por task (SDD)

```
Discuss → Plan (com <verify>) → Execute → Verify (build+lint+test+coverage) → Ship (PR)
```

1. **Discuss** — ler requirement → design → task. Puxar docs oficiais via MCP (Context7 ou similar) para cada SDK. Checar STATE/AUDIT.
2. **Plan** — declarar arquivos que vão mudar, docs consultados, critérios de `<verify>` (testes que vão passar).
3. **Execute** — TDD: RED (teste falha) → GREEN (implementa mínimo) → REFACTOR.
4. **Verify** — gates duros passando (abaixo).
5. **Ship** — commit pt-BR imperativo, PR com `Release Notes:`, atualizar README.md do sub-projeto em mudanças feature-level (bug-fix não atualiza).

**Sprint Audit** obrigatório ao fim de cada sprint via skill dedicado (ex: `/sprint-auditor`).

### 1.4 Gates duros universais

Todo projeto expõe no CLAUDE.md:

- **Build clean**: `<cmd-build> ./...` sem warning
- **Lint/vet**: `<cmd-lint>` sem erro
- **Tests**: `<cmd-test>` verde com `-race -count=1` (ou equivalente)
- **Coverage**: threshold por camada (ver `AGENTS_SPEC.md` §3.21)
- **Security**: scanner de vulnerabilidades + static analysis de segurança (`gosec`/`semgrep`/`snyk`) — severity high passa antes do PR
- **Vendor sec**: `govulncheck`/`npm audit`/`cargo audit`/equivalente
- **Code generation up-to-date**: `sqlc generate`/`build_runner`/`drizzle-kit generate` + `git diff --exit-code` no gerado
- **Context7 / docs oficiais consultados** antes de usar SDK externo

### 1.5 Convenções de linguagem (invariantes)

Que valem em **qualquer** CLAUDE.md de projeto seu:

- **UI copy, log messages, commit text**: idioma do negócio (ex: pt-BR)
- **Identifiers, filenames, comentários técnicos**: inglês
- **Money**: inteiro 64 bits (centavos). Nunca float.
- **Fração/proporção com regra de negócio**: `DECIMAL/NUMERIC`. Nunca float.
- **IDs**: UUIDv7 (ordenável por tempo + aleatório).
- **Auth** via OIDC (cookie httpOnly no web, `Authorization: Bearer` no mobile).
- **"Never trust the frontend"** — frontend é UX; backend revalida tudo.
- **Regras de negócio na API**. Frontend exibe.

---

## 2. Estrutura canônica do CLAUDE.md

Seções obrigatórias (em ordem):

```
1.  Header + descrição curta do projeto          ← 1-3 linhas
2.  Stack (bullets ou tabela)                    ← o que importa no nome + papel
3.  Fontes de verdade (ordem de consulta)        ← §1.2 aplicado ao projeto
4.  Ciclo obrigatório por task                   ← SDD em 1 bloco de código
5.  Regras duras (links)                         ← aponta pra steering, não duplica
6.  Comandos (quick reference)                   ← bloco bash executável
7.  Estrutura do código                          ← aponta pra AGENTS.md ou desenha uma árvore simples
8.  Skills / plugins disponíveis                 ← skills do .claude/skills/ com descrição curta
9.  Sprint atual / status (opcional)             ← estado operacional
```

Seções por sub-projeto (quando aplicável):

```
10. Fontes de verdade cross-repo                 ← pra sub-projetos que consomem specs do sibling
11. Steering — aplicabilidade                    ← tabela "regra × aplica-se? × como adaptar"
12. Architecture (o que não é óbvio)             ← invariantes não-deriváveis do filesystem
13. Auth (estado atual e alvo)                   ← flow + secrets no mobile/web consumidor
14. Workflow SDD adaptado                        ← diagrama Discuss→Plan→Execute→Verify→Ship do stack
15. TDD & Testing — thresholds                   ← tabela por camada (stack-específica)
16. Segurança — específica do stack              ← nunca fazer / sempre fazer
17. Plugins Claude Code relevantes               ← tabela "plugin × uso no projeto"
18. MCP e credenciais                            ← onde vivem, como ativar no workspace
19. Consumo / integração com backend sibling     ← contrato HTTP + interceptors
20. Troubleshooting                              ← tabela "sintoma × causa provável"
21. Próximas features / roteiro                  ← link com tasks.md do backend
```

---

## 3. Templates

### 3.1 CLAUDE.md raiz de workspace multi-projeto

Cenário: diretório contém múltiplos sub-repos (backend, web, app, legacy). Cada um tem seu próprio `CLAUDE.md`.

```markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace Layout

This directory is a **multi-project workspace**, not a single monorepo. Each sub-project is an independent git repository with its own build tooling, dependencies, and its own `CLAUDE.md` / `AGENTS.md` / `README.md`. Read the relevant sub-project doc before editing — the root file here only covers what spans them.

GitHub organization: **<org-url>**.

| Folder | GitHub repo | Stack | Own CLAUDE.md |
|---|---|---|---|
| `<backend>/` | <org/backend> | <lang + framework + DB + auth + payment> | ✅ |
| `<web>/`     | <org/web>     | <framework + bundler + design-system + ...>   | — (see AGENTS.md) |
| `<app>/`     | <org/app>     | <mobile stack>                                  | ✅ |
| `<legacy>/`  | <org/legacy>  | <legacy stack> — **superseded**                | ✅ |

**Important**: `<legacy>/` is an older iteration. Do not port code or conventions between it and the current stack — they have different architectures.

## Product Context (shared across sub-projects)

**<product-name>** is <one-line>. It connects <personas> around <value>. Personas/plans drive feature access via an ABAC matrix (role × plan_tier × action).

Language conventions that hold everywhere:
- User-facing copy, code comments, log messages, and commit text are **<pt-BR/en-US>**. Identifiers and filenames are English.
- Money is always integer cents (<int64 equivalent>). Never floats.
- IDs are UUIDv7.
- Auth is **<OIDC provider>** (cookie for web, `Authorization: Bearer` for mobile).

## Common Commands by Sub-Project

Run each from inside its own folder. Deeper details live in each sub-project's own docs.

### `<backend>/` (<lang>)
```bash
docker compose up -d                 # <deps>
<cmd-run-dev>
<cmd-build>
<cmd-lint>
<cmd-generate>                       # after editing any <gen-trigger>
```

### `<web>/` (<framework>)
```bash
<cmd-install>
<cmd-dev>
<cmd-build>
<cmd-check>
```

### `<app>/` (<mobile>)
```bash
<cmd-deps>
<cmd-run>
<cmd-test>
<cmd-generate>                       # after editing DI / code-gen annotations
```

## Architectural Invariants (<backend>)

The `<backend>/CLAUDE.md` is authoritative. Non-obvious ones worth surfacing:

- **Dependency rule**: `infrastructure → application → domain`.
- **Handlers never import the HTTP framework directly**. Everything goes through `contracts.HTTPRouter`.
- **Errors never get compared by string**. Sentinels + `errors.Is/As`.
- **Middleware order is fixed**: `<lista-exata>`.
- **Code-gen output is not hand-edited** (`<path>`).
- **Naming**: full words, not abbreviations.

## Spec-Driven Workflow

`.kiro/specs/<product>/` — `requirements.md`, `design.md`, `tasks.md`. Read task + requirement + design before touching code, then verify with `<build>` and `<lint>`.
```

### 3.2 CLAUDE.md de sub-projeto backend

```markdown
# CLAUDE.md

Guia **enxuto** de orientação para agentes Claude Code trabalhando neste repositório.
Detalhes vivem em `.kiro/steering/` (carregados sob demanda) e `AGENTS.md` (contrato completo). Este arquivo só lista ponteiros e o essencial para a primeira leitura — maximiza prompt caching.

## Projeto

**<produto>** — <descrição-uma-linha>. Este repositório é o **backend <lang>** (<arquitetura>), parte do monorepo em `../` (ver `../CLAUDE.md`).

Stack: <lang + versão> + <framework> + <banco> + <cache> + <auth> + <providers>.
<padrão-de-segurança> adaptado (<princípios-duros>).

## Fontes de verdade (ordem de consulta)

1. `.kiro/SESSION_CHARTER.md` — ordens ativas da sessão corrente. Leia sempre antes de tudo.
2. `.kiro/specs/<product>/` — requirements, design, tasks.
3. `.kiro/steering/*.md` — regras duras por tópico. Carregue o relevante antes de codar:
   - <lista-dos-principais-steering>
4. `.kiro/STATE.md` — bloqueadores vivos (D-0XX).
5. `.kiro/AUDIT.md` — findings pendentes (A-0XX). **Gate 0**.
6. `AGENTS.md` — contrato completo do agente.
7. Código existente — quando spec e código divergem, ler o código e atualizar spec.

Se spec não existe ou está ambígua: **pare e atualize spec antes de implementar**.

## Ciclo obrigatório por task

```
Discuss → Plan (com <verify>) → Execute → Verify → Ship (PR)
```

- Consulte doc oficial via <MCP-name> antes de usar qualquer SDK externo.
- Coverage thresholds são gate duro: <valores-por-camada>.
- Security gates: `<scanner>` + `<vuln-check>` passam antes do PR.
- Never trust the frontend: validação server-side sempre.
- README do sub-projeto é atualizado na fase Ship em mudança feature-level.
- Sprint Audit (Fase 6) obrigatória ao fim de cada sprint.

## Comandos (quick reference)

```bash
docker compose up -d
<build> ./... && <lint> ./...
<test> -race -count=1 ./...
<test> -tags=integration -count=1 ./...
<coverage-tool> --threshold-file=.coverage.yml
<codegen-tool> generate && git diff --exit-code <gen-path>
<migrate-tool> up
<sec-scanner> -severity high ./...
<vuln-check> ./...
<run-dev>
```

## Estrutura do código

Ver `AGENTS.md` — seção "Estrutura do projeto". Este arquivo **não** duplica para evitar drift.

## Regras absolutas

Ver `.kiro/steering/do-dont.md` (checklist consolidado) e `AGENTS.md` (princípios).

## Skills disponíveis (`.claude/skills/`)

- `<skill-1>` — <descrição curta>
- `<skill-2>` — <descrição curta>

Invoque com `/<nome>` no Claude Code.

## Sprint atual

**Sprint <N> — <nome>** (iniciado <data>). Status em `.kiro/specs/<product>/tasks.md` + `.kiro/STATE.md`.
```

### 3.3 CLAUDE.md de sub-projeto frontend (web workspaces)

Pontos que **sempre** aparecem em CLAUDE.md de frontend — adapte ao framework:

```markdown
## Workspace Layout

<bundler/package-manager> workspaces. <N> apps + <M> packages compartilhados:

| Path              | Role                           | Dev port |
|-------------------|--------------------------------|----------|
| `apps/<app-1>`    | <função>                       | <porta>  |
| `apps/<app-N>`    | <função>                       | <porta>  |
| `packages/ui`     | design system                  | —        |
| `packages/schemas`| validação compartilhada        | —        |

Portas são hardcoded em cada `package.json`. Mudar porta = atualizar CORS do backend + redirects OIDC. **Nunca casualmente.**

## Fontes de verdade

O `<web>/` **não tem `.kiro/` próprio**. Specs vivem em `../<backend>/.kiro/specs/<product>/`.

| Steering | Aplicável ao web? | Como aplicar |
|---|---|---|
| `sdd-workflow.md` | ✅ | Ciclo 5 fases adaptado |
| `mcp-integration.md` | ✅ | Context7 para docs oficiais (<libs>) |
| `security.md` | ✅ | "Never trust the frontend" |
| `testing.md` | ⚠️ parcial | TDD adaptado (<runner> + <testing-lib>) |
| `transaction-patterns.md` | ❌ | Backend only |
| `<backend-patterns>.md` | ❌ | Específicos do backend |

## App Architecture (o que não é óbvio)

Cada app segue o mesmo esqueleto — pegue `apps/<referência>` como template:

- **Entry point**: `src/<entry>`. Monta framework em `#root`, envolve com <providers>.
- **Routing**: file-based via <router>. `<gen-file>` é **gerado** — nunca editar à mão.
- **Styling**: <engine>. Shortcuts/tokens como design system — **reuse** em vez de utilitários crus.
- **Env vars**: declaradas em `src/env.d.ts`, expostas via `import.meta.env.*`.

Apps atualmente **não têm auth/query providers wireados** — implementar context **dentro de cada app**, não em package compartilhado.

## Shared Packages

`packages/ui` e `packages/schemas` declaram `"main": "src/index.ts"` — apps importam direto do TypeScript source via workspace link, sem build intermediário.

Ao criar novo componente compartilhado:
1. Seguir pattern do `<componente-referência>` (CVA + headless + `splitProps`)
2. Re-exportar de `packages/ui/src/index.ts`
3. Adicionar `"ui": "*"` nas deps do app consumidor

## Design System (convenções)

- **Attributify** com `prefix: "un-"` e `prefixedOnly: true` — `<div un-flex>`, **nunca** `<div flex>`
- **Icons** com `i-` prefix
- **Shortcuts** são o DS (`btn-primary`, `text-display-h1`, ...) — **nunca** hex solto
- **Fonts** via `presetWebFonts` ou equivalente
- **Color tokens** em CSS variables (`--primary`, `--surface`, `--destructive`, ...) — paleta **imutável** sem aprovação

## Segurança frontend

### Nunca fazer
- ❌ Token em `localStorage`/`sessionStorage`/`IndexedDB`/cookies não-httpOnly
- ❌ `dangerouslySetInnerHTML` (o equivalente no framework)
- ❌ Validar autorização no frontend — só mostra/esconde UI; backend reavalia
- ❌ Calcular monetário/quotas/prazos — exibe o que backend retorna
- ❌ Hardcodar secret/key/token em código
- ❌ Request a domínio arbitrário via user input
- ❌ `eval` / `new Function` / `setTimeout(string)`

### Sempre fazer
- ✅ `credentials: "include"` em toda chamada — cookie httpOnly vai sozinho
- ✅ Validar com <schema-lib> toda entrada **antes** de enviar (UX)
- ✅ Validar resposta do backend com <schema-lib> (defesa em profundidade)
- ✅ CSP via headers no deploy
- ✅ Revogar cache ao fazer logout — `queryClient.clear()` + redirect OIDC

## Consumo da API backend

```typescript
const api = axios.create({
  baseURL: import.meta.env.PUBLIC_API_URL,
  withCredentials: true,
  timeout: 10_000,
});

api.interceptors.response.use(
  (r) => r,
  (error) => {
    if (error.response?.status === 401) window.location.href = "/auth/login";
    return Promise.reject(error);
  }
);
```

Response format padrão:
```typescript
type ApiResponse<T> = { success: true; data: T };
type ApiError       = { success: false; error: { code: string; message: string } };
```
```

### 3.4 CLAUDE.md de sub-projeto mobile

Pontos que **sempre** aparecem em CLAUDE.md de mobile:

```markdown
## Comandos essenciais

```bash
<pkg-tool> <install-cmd>
<run-cmd>                                       # device/emulador conectado
<test-cmd>
<test-cmd> <single-file>                        # arquivo único
<test-cmd> --name "should return X on success"  # por nome
<test-cmd> --coverage
<analyze-cmd>

# Code generation (OBRIGATÓRIO ao alterar DI annotations)
<codegen-cmd>
<codegen-watch>                                  # durante sessão DI intensa

# Formatação
<format-cmd>
```

`<di-config-file>` é **gerado** — nunca editar à mão. Se DI falhar com "not registered" no startup, a causa mais comum é config stale — rode codegen de novo.

## Architecture — o que importa antes de editar

O walkthrough detalhado está em `ARCHITECTURE.md` (por arquivo) e `CODING_MANUAL.md` (regras e proibições).

- **Layout é Feature First + Clean Architecture**: `lib/app/` (bootstrap, DI, router), `lib/core/` (shared infra), `lib/features/<nome>/{domain,data,presentation}/`. Testes em `test/` espelham `lib/` exatamente.
- **Dependency Rule (inviolável)**: `presentation → domain ← data`. Features cruzam via `core/` ou `domain/entities/` de outra feature.
- **Erros nunca propagam como exceção além de `data/`**. `RepositoryImpl` é o único lugar com `try/catch` — converte exceção em `Failure` e retorna `Either<Failure, T>`.
- **<async-dep> registrado manualmente em `main`** antes de `configureDependencies()` — é async, não cabe no getter sync.
- **DI annotation é semântica, não estilo**:
  - Singleton → Repositories, Datasources, HTTP client, NetworkInfo
  - Factory → State mgmt + UseCases (fresh por resolução)
  - Module → third-party que não aceita anotação

## Auth (estado atual e alvo)

**Estado atual**: <onde-está-o-shape>; token em <storage-atual>. `<constants-file>` tem placeholders:
- `baseUrl = 'https://api.example.com'` → **deve virar** `<prod>` / `<dev>`
- `<oidc-issuer>` → preencher com instância <provider>
- `<client-id>` e redirect URI → **reais**

**Alvo**:
1. Login → OIDC PKCE com <provider>
2. AccessToken + RefreshToken recebidos
3. Ambos armazenados em **secure storage nativo** (NÃO em key-value simples)
4. HTTP interceptor injeta `Authorization: Bearer <token>`
5. Refresh automático antes de expirar
6. Logout → end_session <provider> + secure storage clear + nav login

**Regra dura**: token nunca em storage não-seguro, nunca em log, nunca em analytics. Keychain iOS + Keystore Android.

## Segurança mobile — específica

### Nunca fazer
- ❌ Token em storage não-seguro
- ❌ PII em log
- ❌ Accept `validateStatus: (_) => true` em produção
- ❌ HTTP plain em produção — TLS obrigatório
- ❌ Certificate pinning removido ou comentado
- ❌ HTTP lib nativa solta — use client tipado com interceptors
- ❌ WebView com JS sem CSP
- ❌ Secrets em constantes do app — nunca chegarem ao bundle

### Sempre fazer
- ✅ Tokens em secure storage nativo
- ✅ Certificate pinning em prod
- ✅ HTTPS-only — config nativa (`NSAppTransportSecurity` iOS, `networkSecurityConfig` Android)
- ✅ Device fingerprint header em requests — `<lib-device-info>`
- ✅ Biometric unlock para re-auth quando aplicável
- ✅ Logout → secure storage clear + http cookie jar clear + nav
- ✅ Detecção de jailbreak/root — reportar ao backend, não bloquear sem justificativa
- ✅ CSP em WebViews
```

### 3.5 CLAUDE.md de sub-projeto legacy (superseded)

```markdown
# CLAUDE.md

Guia para Claude Code operando no sub-projeto **`<legacy>/`**.

> **Status: LEGACY / SUPERSEDED** — esta pasta é uma iteração antiga do mesmo produto. O stack atual é `<current>/`. Só toque aqui se for explicitamente pedido.

Stack: <legacy-stack>
Papel: referência histórica. Patches críticos de bug apenas.

Não **port code or conventions** entre legacy e current — são arquiteturas distintas.
```

---

## 4. Skills + plugins — tabela padrão

Todo CLAUDE.md de sub-projeto deve ter uma tabela de plugins com a coluna "uso neste projeto":

```markdown
| Plugin | Uso no <projeto> |
|---|---|
| `<lsp-lang>`        | ✅ obrigatório — diagnostics |
| `figma`             | ✅ sempre que implementar tela |
| `security-guidance` | ✅ pre-tool hook detecta <padrões-perigosos> |
| `code-review`       | ✅ PR review com scoring |
| `feature-dev`       | ✅ exploration/design/review |
| `superpowers`       | ✅ `/brainstorm`, `/write-plan`, TDD skills |
| `commit-commands`   | ✅ `/commit` em <idioma> |
| `claude-md-management` | ✅ mantém CLAUDE.md e AGENTS.md coerentes |
| `context7`          | ✅ docs oficiais de <libs-do-stack> |
| `playwright`        | ⏳ quando entrar E2E |
| `<outro>`           | ❌ não se aplica |
```

---

## 5. MCP e credenciais — padrão

```markdown
## MCP e credenciais

MCPs em `.mcp.json` + `.claude/settings.json`. Quando abrir Claude Code diretamente em sub-projeto:

- Se workspace root = `<projeto-com-mcp>/`, MCPs carregam. Ótimo.
- Se workspace root = sub-projeto sem `.mcp.json` próprio, replicar ambos arquivos aqui OU abrir Claude Code da raiz do workspace.

Env vars vivem em `<projeto-com-env>/.env` (gitignored).
```

---

## 6. Troubleshooting — padrão

Tabela sintoma × causa. Capturar os 5-10 problemas que **todo dev novo** encontra no primeiro dia:

```markdown
| Sintoma | Causa provável |
|---|---|
| `<erro-de-DI>` no startup | <config-file> stale — rodar codegen |
| "X not registered" | Anotação esquecida ou codegen não rodou |
| Teste falha com "state never emitted" | `props` ausente em state equatable |
| Deep link não navega | Scheme não registrado em AndroidManifest.xml + Info.plist |
| Build iOS falha em release | Provisioning profile / codesigning — build `--no-codesign` em dev |
| Cookie não vai em dev | `Domain=...` config; localhost precisa CORS + cookie sem Domain attribute |
| Gen file quebrado em build | Editou à mão — apagar e rodar dev pra regenerar |
| Workspace não resolve package | `<install>` na raiz; se persiste, rebuild `dist/` do package |
```

---

## 7. Checklist de qualidade

Antes de commitar um CLAUDE.md novo ou atualização, validar:

- [ ] Cabeçalho padrão ("This file provides guidance to Claude Code...")
- [ ] Descrição do projeto em ≤ 3 linhas
- [ ] Stack listado com versões
- [ ] Fontes de verdade ordenadas
- [ ] Ciclo SDD mencionado com link para steering
- [ ] Comandos "quick reference" como bash executável
- [ ] Zero duplicação com `AGENTS.md` ou steering — apontar, não copiar
- [ ] Para sub-projeto: tabela "steering × aplica-se? × como adaptar"
- [ ] Para sub-projeto: seção "Architecture (o que não é óbvio)"
- [ ] Para frontend: "Never trust the frontend" seção explícita
- [ ] Para mobile: "Secure storage + certificate pinning" seção explícita
- [ ] Tabela de plugins com "uso neste projeto"
- [ ] Troubleshooting com 5-10 casos reais
- [ ] Sprint atual / status (opcional mas útil)
- [ ] Alvo ~100 linhas na raiz; ~300-500 em sub-projetos com mais nuance

---

## 8. Como replicar em um novo projeto

1. Copiar este SPEC como seed.
2. Escolher template base (raiz multi-projeto / backend / frontend / mobile / legacy).
3. Preencher placeholders `<...>` com valores reais do projeto.
4. Montar a tabela de "steering × aplica-se?" — lendo cada steering do sibling.
5. Extrair "Architecture (o que não é óbvio)" do código existente — não da sua memória.
6. Identificar skills instaladas em `.claude/skills/` e descrevê-las em uma linha.
7. Ordenar fontes de verdade conforme o projeto real (nem todo projeto tem `SESSION_CHARTER.md`; adapte).
8. Listar 5-10 entradas de troubleshooting após a primeira semana de uso — o CLAUDE.md **evolui** com o projeto.
9. Validar contra o checklist §7.
10. Publicar na raiz do projeto como `CLAUDE.md`.

**`CLAUDE.md` é documento vivo**. Revisar a cada sprint. Se drift com `AGENTS.md` ou steering — priorizar atualização do steering e apontar daqui.
