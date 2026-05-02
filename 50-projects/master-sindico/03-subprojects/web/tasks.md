---
title: Web — Tasks seed
type: note
tags: [subproject, web, master-sindico, v2, tasks, sprint, seed, backlog]
source: Development/web/CLAUDE.md §Sprints + ROADMAP M1/M2/M3 + ui-catalog.md
created: 2026-04-23
updated: 2026-04-23
---

# Web — Tasks seed

Tasks alvo do sub-projeto `web/` organizadas por marco. Seed inicial destilado de:
- [[README]] + [[architecture]] + [[patterns]] + [[design-system]] + [[security]] + [[requirements]] + [[ui-catalog]].
- `Development/web/CLAUDE.md` §Sprints (linhas 475-637) — plano 8 sprints do backend; web é paralelo.
- [[../../ROADMAP]] M1/M2/M3.

Formato: `FE-###` · app · feature · descrição · arquivos-alvo · estimativa (S/M/L/XL).

| Tam | Horas aprox |
|---|---|
| S | 2-4h |
| M | 4-8h |
| L | 8-16h |
| XL | > 16h (quebrar) |

---

## Sprint 10 — M1 (entregar 2026-05-08)

### Setup & infra (compartilhado)

- **FE-001** M · root · Validar scaffold real (apps + packages já criados); documentar stack divergência (D-020) e atualizar STATE
- **FE-002** M · root · Normalizar `railway.json` dos 6 apps (remover `turbo` — DT-WEB-001); CI pipeline
- **FE-003** S · root · `bun run check` + `bun run format` passam em CI; Dependabot habilitado
- **FE-004** S · root · GitHub Actions: typecheck (`tsc --noEmit` por app) + biome + build + coverage threshold
- **FE-005** S · root · Sentry Browser SDK em cada app (6×) com `release: $COMMIT_HASH`; PII `beforeSend` scrubber
- **FE-006** M · root · `web-vitals` lib + endpoint `POST /api/v1/metrics/web-vitals` (coordena com backend)
- **FE-007** S · root · Biome custom rules: ban `innerHTML`, `eval`, `Function`, `dangerouslySetInnerHTML` (security-guidance reforça)
- **FE-008** M · root · OpenAPI → TS generator (Orval ou openapi-typescript) — `packages/schemas` importa types gerados
- **FE-009** S · `apps/*` · Preload fonts Michroma + Manrope em `index.html` de cada app

### Packages compartilhados

- **FE-010** L · `packages/schemas` · Scaffold Zod schemas canônicos: `auth.ts`, `profile.ts`, `billing.ts` (tier Stripe-style `trial|base|plus|pro`), `institutional.ts`, `commercial.ts`, `content.ts`, `common.ts` (branded: ULID, CPF, CNPJ, Email, Money=int centavos, FracaoIdeal=string decimal)
- **FE-011** S · `packages/schemas` · Coverage 95%; property-based tests com fast-check em Money + FracaoIdeal
- **FE-012** M · `packages/ui` · Adicionar shortcut `btn-destructive` em `uno.config.ts` (preset compartilhado via import entre apps) + token `--info`
- **FE-013** M · `packages/ui` · Atoms base: `Input`, `Textarea`, `Checkbox`, `Radio`, `Switch`, `Badge`, `Spinner`, `Avatar` — padrão Kobalte + CVA + splitProps (template Button.tsx)
- **FE-014** M · `packages/ui` · Molecules: `FormField`, `Card`, `TabStrip`, `Tooltip`
- **FE-015** M · `packages/ui` · Organism: `Dialog` (Kobalte Dialog.Root + Portal + Overlay) + `Toast` host
- **FE-016** S · `packages/ui` · Jest-axe em unit tests; zero violations; coverage 90%
- **FE-017** S · `packages/ui` · Exportar `packages/ui/src/uno.preset.ts` com shortcuts/tokens compartilhados; apps importam no `uno.config.ts` próprio

### Core transversal (por app, mas código duplicado inicialmente — extrair pra `packages/core` M2)

- **FE-020** M · `apps/*` · `src/core/api/http.ts` — Axios instance + `withCredentials: true` + interceptors 401/403/CSRF + `X-Request-Id` + `X-Device-Fingerprint`
- **FE-021** M · `apps/*` · `src/core/auth/session.context.tsx` — SessionContext + `useSession()` hook + `GET /api/v1/auth/me` on mount + logout
- **FE-022** S · `apps/*` · Route guards `_authenticated.tsx` beforeLoad (redirect sign-in se não autenticado)
- **FE-023** M · `apps/*` · `src/core/security/fingerprint.ts` — canvas + webgl + UA hash SHA-256
- **FE-024** S · `apps/*` · `src/core/api/csrf.ts` — leitura cookie `csrf_token` + header `X-CSRF-Token` em mutations
- **FE-025** S · `apps/*` · `src/core/config/env.ts` — validação env vars `PUBLIC_*` obrigatórios + fail fast
- **FE-026** S · `apps/*` · Wire `QueryClientProvider` + `AuthProvider` descomentando em `index.tsx` (CLAUDE.md linha 57-61)

### App `auth` (entregar completo M1)

- **FE-030** S · AUTH1 Splash com logo dourado + CTAs
- **FE-031** M · AUTH2 Sign In com Zod + TanStack Form + rate-limit UI
- **FE-032** L · AUTH3 Sign Up (5 personas cards) + AUTH4 forms multi-step por persona com auto-save Redis (coord backend)
- **FE-033** M · AUTH5 Recuperação senha + OTP (SMS via backend)
- **FE-034** S · AUTH6 Callback Zitadel (exibe spinner; backend redireciona)
- **FE-035** S · AUTH7 Device Mismatch (recebe 403 DEVICE_MISMATCH) + [Continuar aqui] com `force_login=true`
- **FE-036** S · AUTH8 Logout flow + `queryClient.clear()` + redirect

### App `cms` — Morador M1 (M1-M15)

- **FE-040** M · M1 Painel do Morador (4 cards) + seletor de condomínio ativo
- **FE-041** M · M2 Selecionar Condomínio — lista memberships
- **FE-042** M · M3 Buscar por ID + rate-limit anti-scraping (backend)
- **FE-043** L · M4 Cadastro Unidade + pending approval fluxo
- **FE-044** M · M5 Home do Condomínio (6 cards)
- **FE-045** M · M6 Timeline (feed infinite scroll + filtros)
- **FE-046** S · M7 Detalhe Publicação (DOMPurify render)
- **FE-047** M · M8 Plano Diretor (kanban)
- **FE-048** S · M9 Detalhe Ação PD
- **FE-049** S · M10 Eventos (ciente/confirmar presença)
- **FE-050** S · M11 Comunicados (tracking leitura)
- **FE-051** M · M12 Avaliar Gestão (3 perguntas 1-10 anônima; already_submitted state)
- **FE-052** M · M13 Meu Vínculo + M14 Dependentes + M15 Encerrar (double-confirm)

### App `cms` — Síndico onboarding + base institucional

- **FE-060** L · S3 Cadastrar Condomínio (endereço + ViaCEP)
- **FE-061** M · S4 Dados da Gestão (mandato + upload PDF)
- **FE-062** M · S5 Empresa Síndica vinculada (5 toggles)
- **FE-063** XL · S6 Home da Governança (10 cards + indicadores real-time) — **tela-pivô**
- **FE-064** M · S11 Timeline síndico (criar/listar) + voice input (R2)
- **FE-065** L · S12 Criar Atividade (26 tipos, 9 riscos, voice, confirmação contexto R1)
- **FE-066** S · S13 Editar Atividade (nova versão; R3)
- **FE-067** S · S14 Histórico LT
- **FE-068** M · Conta SYS1-SYS3 (dados, assinatura com Stripe Customer Portal link, dispositivos)

### App `cms` — Admin MS embutido (AD1-AD2 M1; resto M2-M3)

- **FE-070** M · AD1 Admin Home — gate ABAC duplo; dashboard metrics placeholder
- **FE-071** M · AD2 Gestão de Tenants (suspender/reativar com audit); banner "Modo Admin"

### App `lp` (M2 prep — scaffold apenas M1)

- **FE-080** M · PUB1 Home (blob system já implementado apps/lp/src/routes/index.tsx; refinar copy)
- **FE-081** S · Meta tags + Open Graph + canonical

### Testing (transversal)

- **FE-090** M · Setup Vitest + `@solidjs/testing-library` + `@testing-library/user-event` + MSW em cada app
- **FE-091** S · Test harness `renderWithProviders` (QueryClient + ColorMode)
- **FE-092** M · Fixtures de sessão (síndico, morador, empresa, admin MS) + MSW handlers base
- **FE-093** L · Unit tests para todas as telas M1 (75% threshold rotas, 85% hooks, 90% lib, 90% ui, 95% schemas)

---

## Sprint 11 — M2 (entregar 2026-06-20)

### Packages

- **FE-100** L · `packages/ui` organisms M2: `DataTable`, `Combobox`, `DatePicker`, `Stepper`, `UploadDropzone` (Mux signed), `CommandPalette`
- **FE-101** M · `packages/schemas` adicionais: `assembly.ts`, `content.ts` (video), `onboarding.ts`, `admin.ts`
- **FE-102** M · Extrair `packages/core` — hooks compartilhados (`useFingerprint`, `useCSRF`, `useQuota`) + pure functions

### App `cms` — Connect Me 4 fluxos

- **FE-110** L · S18 Connect Me Hub (4 tabs) + quotas UI
- **FE-111** M · S19 Criar Connect Me + modal LGPD (consent version)
- **FE-112** M · S20 Connect Me Recebido + revelar contato pós-LGPD
- **FE-113** M · E2 Oportunidades empresa + filtros
- **FE-114** S · E3 Recusar (6 motivos estruturados)
- **FE-115** M · E4 Detalhe Oportunidade (revela contato + LGPD log)

### App `cms` — Empresa + Execução

- **FE-120** L · E1 Painel Empresarial (10 cards + indicadores) — simétrica S6
- **FE-121** M · E5 Registrar Execução (form + CEP + anexos)
- **FE-122** M · E6 Atividade Técnica
- **FE-123** M · E7 Comunicados Técnicos (vai a S26)
- **FE-124** M · E8 Status de Envios + termos versionados + LGPD log
- **FE-125** M · E9 Detalhe Registro + estados (rascunho→enviado→aprovado→publicado→bloqueado)
- **FE-126** L · S26 Validações Pendentes (hub bulk actions)
- **FE-127** M · E10 Dashboard Empresarial (conversão CM, avaliações, garantias)
- **FE-128** M · E11 Perfil Institucional (edit com trava 90d em vídeos)
- **FE-129** S · E12 Usuários Empresa (admin + operador técnico)

### App `cms` — Conteúdo (vídeos)

- **FE-130** L · Upload Mux signed URL fluxo (ticket + direct upload + webhook polling)
- **FE-131** M · Player Mux web component + paywall (25% preview base)
- **FE-132** M · Visibilidade vídeo empresa (gate votação fornecedor ativa)

### App `cms` — Agência + Reputação

- **FE-140** M · MK1 Painel Agência
- **FE-141** M · MK2-MK3 Empresas Atendidas + switch contexto (delegation)
- **FE-142** L · MK4 Publicar Vídeo (12 tipos) + checkbox autorizar votação
- **FE-143** M · MK5 Biblioteca + MK6 Dashboard Empresa
- **FE-144** M · MK7 Dashboard Geral + MK8 Marketing Link
- **FE-145** M · E13-E16 Gestão Agência lado empresa (convite + permissões)

### App `cms` — Compliance

- **FE-150** L · S28-S31 + C1-C11 telas compliance (11 telas)
- **FE-151** M · Gatilhos compliance (encerrar mandato, dossiê) bloqueadores
- **FE-152** M · C11 Dossiê PDF (backend gera; frontend download signed URL)

### App `forum` (M2 completo)

- **FE-160** L · VZ1-VZ6 Parceiro Vizinhança (cadastro + perfil + promoções)
- **FE-161** L · VS1-VS10 Síndico vizinhança
- **FE-162** M · VE1-VE6 Empresa vizinhança
- **FE-163** M · VM1-VM7 Morador vizinhança (feed bairro)

### App `lp`

- **FE-170** L · PUB2-PUB6 páginas LP + SSG via Rsbuild
- **FE-171** M · Cookie banner LGPD granular + registro consent backend
- **FE-172** S · Sitemap.xml + robots.txt
- **FE-173** M · SEO structured data + blog RSS

### App `cms` — Admin MS M2

- **FE-180** L · AD3 Moderação de Conteúdo (vídeos pendentes, harassment reports)
- **FE-181** L · AD4 Financeiro (Stripe reconciliation)
- **FE-182** M · AD5 Broadcasts globais (com double-confirm)
- **FE-183** M · AD6 Audit Trail (read-only logs cross-tenant)
- **FE-184** M · MFA TOTP setup (SYS4) + gate acesso `/admin/*`

### Segurança M2

- **FE-190** M · CSP headers fine-tune + reporting endpoint
- **FE-191** M · OWASP ZAP baseline CI
- **FE-192** S · SRI em CDN scripts (Mux, Sentry)
- **FE-193** M · Rate limit UI feedback (429 + Retry-After)

---

## Sprint 12 — M3 (entregar 2026-07-20)

### App `assembly` (completo)

- **FE-200** XL · AS1 Configuração + AS2 Pauta (drag-drop + lock jurídico)
- **FE-201** L · AS3 Edital + PDF generator + procurações tracking
- **FE-202** M · AS4 Simulador de Quórum (fração ideal DECIMAL frontend display)
- **FE-203** L · AS5 Check-in (app / QR / terminal)
- **FE-204** XL · AS6 Votação Live — WebSocket + ReconnectingWS + auto-save Redis + 4 tipos voto
- **FE-205** XL · AS7 Telão 2 áreas + latência < 200ms + modo apresentação
- **FE-206** L · AS8 Ata + 6 relatórios (CSV + PDF) + score transparência + homologação imutável

### App `lms` (completo — pós-lançamento na prática)

- **FE-210** L · LMS1-LMS5 Cursos + certificado PDF
- **FE-211** M · LMS6-LMS7 Treinamentos
- **FE-212** L · LMS8-LMS10 Lives (player + chat + replay)
- **FE-213** M · LMS11-LMS13 Fórum
- **FE-214** M · LMS14-LMS15 Biblioteca
- **FE-215** M · Stripe Checkout redirect para cursos pagos (+ return flow)
- **FE-216** S · Gate Parceiro Vizinhança bloqueado (botão oculto)

### App `cms` — Banco de Talentos

- **FE-220** L · TEL1-TEL11 vídeo-currículo 90s morador + lock 90d + Mux upload
- **FE-221** L · Visualização empresa (7 funcionalidades: favoritos, export PDF, histórico, funil, tags auto/manual, busca avançada)

### App `cms` — Admin MS M3

- **FE-230** L · AD7 ABAC Override com 4-eyes policy (2 admins aprovam)
- **FE-231** M · AD8 Suporte (DSAR export JSON + delete com double-confirm)
- **FE-232** M · IP allowlist para super_admin (M3 feature)

### E2E (Playwright — M3)

- **FE-240** L · Setup Playwright + `@axe-core/playwright`
- **FE-241** M · E2E Login OIDC + 1-device
- **FE-242** L · E2E Cadastro condomínio síndico → Timeline → PD
- **FE-243** L · E2E Onboarding morador completo
- **FE-244** L · E2E Connect Me síndico → empresa (envio + aceite + LGPD)
- **FE-245** L · E2E Assembleia: configuração + pauta + edital + checkin + votação happy path
- **FE-246** M · E2E Stripe Checkout upgrade trial → pro
- **FE-247** M · E2E Admin MS: gate ABAC + MFA + destrutiva com double-confirm

### Performance + A11Y M3

- **FE-250** M · Lighthouse CI score budgets; bundle-size CI
- **FE-251** M · Validação contraste WCAG AA em todos tokens (pa11y)
- **FE-252** S · `prefers-reduced-motion` respeitado em todas animations
- **FE-253** M · PWA manifest + ícones em `cms`, `forum`, `assembly`

---

## Backlog (pós-M3 / M4+)

### Performance

- **FE-B01** M · Service Worker offline read-only (timeline, PD, eventos)
- **FE-B02** M · Virtualização lista notificações + audit
- **FE-B03** L · SSR/SSG estratégia mista (LP SSR, app CSR)

### UX polish

- **FE-B10** M · CommandPalette `⌘K` em `cms`
- **FE-B11** M · Dark mode auto-schedule (night shift)
- **FE-B12** M · Skeleton micro-interactions
- **FE-B13** L · i18n EN fallback + language switcher
- **FE-B14** M · Tour guiado (react-joyride equivalente Solid) pós-onboarding

### Advanced

- **FE-B20** XL · Storybook (opcional — avaliar ROI)
- **FE-B21** XL · Figma tokens sync (Design tokens W3C format)
- **FE-B22** L · AI assistant panel (Master Síndico Assistant — M4+)
- **FE-B23** M · Animações Lottie para empty states charmosos

### Segurança avançada

- **FE-B30** L · Playwright security probes (XSS/CSRF injection tests)
- **FE-B31** M · Snyk ou Socket.dev integration CI
- **FE-B32** M · Honeypot em forms públicos (cadastro, recovery)
- **FE-B33** L · Bot detection (Cloudflare Turnstile ou similar)

---

## Transversais / contínuas (toda sprint)

- **FE-T01** toda sprint · Revisão CLAUDE.md (399 linhas) alinhado com realidade do código
- **FE-T02** toda sprint · Revisão AGENTS.md (647 linhas) alinhado com padrões
- **FE-T03** toda sprint · Dependabot PRs semanais → triage + merge seguros
- **FE-T04** toda sprint · Web Vitals report semanal (RUM) — identifica regressões
- **FE-T05** toda sprint · Sentry triage (≥ 90% erros resolvidos no sprint)
- **FE-T06** toda sprint · A11y smoke test manual (NVDA + VoiceOver) em telas novas
- **FE-T07** toda sprint · Biome + typecheck green em todas 6 apps
- **FE-T08** toda sprint · Coverage threshold mantido

---

## Dívidas técnicas identificadas (DT-WEB-###)

- **DT-WEB-001** · 🔄 **REDIRECIONADA 2026-04-27** · Após [[../../02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]] / [[../../STATE#D-138]] (Cloudflare-only, Railway descontinuado): remover **todos** os `railway.json` dos 7 apps; criar `wrangler.toml` + script `deploy: wrangler pages deploy dist`. Resolve junto NEW-001 (filter=web-lp em todos). **FE-002 reescrita**.
- **DT-WEB-002** · S · Shortcut `btn-destructive` referenciado em `packages/ui/src/button.tsx` mas NÃO definido em `uno.config.ts`. Adicionar. **FE-012**
- **DT-WEB-003** · S · Scrollbar customizada usa `var(--color-border)` e `var(--color-muted-foreground)` mas CSS vars são `--border` e `--muted-foreground` (sem prefix `color-`). Corrigir em `main.css`.
- **DT-WEB-004** · M · `uno.config.ts` de 212 linhas idêntico entre 6 apps (duplicação). Extrair para `packages/ui/src/uno.preset.ts` e importar. **FE-017**
- **DT-WEB-005** · 🟢 **RESOLVIDO 2026-04-27** · `packages/schemas/src/{auth,common}.ts` implementados; 30+ casos de teste passam (Vitest 95% threshold). Pendente expandir para billing/institutional/commercial/content/assembly (ver A-WEB-2026-04-27-021)
- **DT-WEB-006** · S · `apps/*/README.md` é stub bun init. Substituir com README real do app.
- **DT-WEB-007** · S · `index.ts` raiz é stub `console.log`. Remover (não é usado).
- **DT-WEB-008** · M · Token `--info` ausente (apenas success/warning/destructive). Adicionar. **FE-012**
- **DT-WEB-009** · S · `btn-icon` é 40×40 — abaixo do WCAG touch target 44×44. Alinhar mobile ou usar em desktop-only.
- **DT-WEB-010** · M · `packages/ui/src/uno.config.ts` ausente — packages deveriam também ter UnoCSS config próprio ou herdar preset compartilhado.
- **DT-WEB-011** · M · Sem `packages/core` — ainda duplicando http client + session context entre apps. Extrair em M2.
- **DT-WEB-012** · S · `PUBLIC_` env convention não documentada em `apps/*/src/env.d.ts` (atualmente só referencia Rsbuild types).
- **DT-WEB-013** · L · Sem storybook / vre para catálogo visual do design system. Avaliar ROI M3+.
- **DT-WEB-014** · S · `tsconfig.json` root sem `paths` alias; apps podem ter imports longos. Decidir se vale o alias.
- **DT-WEB-015** · M · `old/` é 192 bytes de legado (auth-context.tsx + query-context.tsx + __root.tsx) — confirmar gitignored + nunca importar.

---

## Gates de entrega (Definition of Done — DoD)

Cada task concluída exige:

1. ✅ `bun run check` (biome) sem warnings.
2. ✅ `tsc --noEmit` por app sem erros.
3. ✅ `bun run build` sem warnings de chunk grande.
4. ✅ Coverage mínimo da camada (75/85/90/95 conforme tipo).
5. ✅ Unit tests (Vitest) + jest-axe zero violations.
6. ✅ MSW para toda chamada HTTP (sucesso + 401 + 403 + 500 + timeout).
7. ✅ Property-based para regra crítica (fração ideal, quota, trial).
8. ✅ A11y smoke (keyboard + screen reader) em tela nova.
9. ✅ Dark mode funciona sem quebra visual.
10. ✅ Mobile responsive (375px + 768px + 1280px).
11. ✅ `credentials: "include"` em toda chamada API.
12. ✅ Zod validation em toda entrada (UX) + toda resposta (defesa).
13. ✅ Zero PII em logs/console em prod.
14. ✅ Commit pt-BR via plugin `commit-commands`.
15. ✅ PR review com scoring ≥ 80 via plugin `code-review`.

---

## Auditoria 2026-04-27

Auditoria sistemática completa em [[AUDIT-2026-04-27]]: **26 findings catalogados** (3 🔴 críticos M1-bloqueantes + 10 🟠 high + 10 🟡 médios + 3 🟢 informativos/resolvidos). Stack 100% alinhada com docs oficiais (Vitest 4 + Zod 4 + TanStack 1.168 + SolidJS 1.9 + Kobalte 0.13 + MSW 2.13). 80 testes Vitest passam (52 schemas + 13 ui + 15 auth). `bun run check` falha com 6 erros a11y em apps/auth + 2 erros idênticos em cada um dos 5 apps base.

### Novas DTs identificadas

- **DT-WEB-016** · 🟠 · biome a11y errors em apps/auth (useFocusableInteractive, useSemanticElements) bloqueiam DoD #1
- **DT-WEB-017** · 🟠 · `bun test` (Bun runner) ≠ `bun run test` (vitest) — usar sempre `bun run test`
- **DT-WEB-018** · 🟠 · auth-context.tsx faz POST /sign-in com email+password (não PKCE) contra STEERING #18
- **DT-WEB-019** · 🟠 · auth-context.fetchSession sem AuthSessionSchema.parse (defesa em profundidade)
- **DT-WEB-020** · 🟠 · OTP code passa via search params em /forgot-password/new-password e /set-password (vazamento browser/referer/proxy)
- **DT-WEB-021** · 🟠 · ServerErrorPage exibe error.message raw em prod (info disclosure)
- **DT-WEB-022** · 🟠 · createEffect+setInterval com onCleanup fora — SolidJS docs exigem dentro do effect
- **DT-WEB-023** · 🔴 · apps/admin (port 3010) D-134 ainda não scaffolded
- **DT-WEB-024** · 🟠 · 5 apps base (cms/lms/forum/assembly/lp) têm código morto + ColorModeProvider sem initialColorMode + routes/index.tsx duplicado

### Atualizações de status

- DT-WEB-005 → 🟢 RESOLVIDO (schemas implementados + tests passam)
- packages/ui evoluiu de "apenas Button" para 7 componentes (Button, Input, Card, OtpInput, PasswordStrength, ProgressDots, Tabs)
- apps/auth evoluiu de "scaffold + 2 stubs" para 13 rotas completas (sign-in, sign-up, callback OIDC, 4× forgot-password, set-password, verify-email, welcome, logout, device-mismatch + layouts)

---

## Pass de execução Agente 2 (2026-04-27 tarde) — DTs fechadas + apps/admin

Detalhes em [[../../11-audit/audit-log/2026-04-27-agente-2-systematic-pass]].

### DTs fechadas 🟢 (ciclo 1 — Lotes 1-7)

- **DT-WEB-002** 🟢 — `btn-destructive` shortcut definido em 7× `uno.config.ts`
- **DT-WEB-003** 🟢 — scrollbar `var(--color-border)` → `var(--border)` em 7× `main.css`
- **DT-WEB-016** 🟢 — biome a11y errors apps/auth → 0 (`bun run check` 0/0/1)
- **DT-WEB-019** 🟢 — `AuthProvider.fetchSession` valida `AuthSessionSchema.safeParse` + log estruturado
- **DT-WEB-021** 🟢 — `ServerErrorPage` prod-safe (oculta msg crua em prod, mostra em dev/test) + novo prop `requestId`
- **DT-WEB-022** 🟢 — `createEffect+setInterval` → `onMount(setInterval; onCleanup)` em verify-email + forgot-password/code
- **DT-WEB-023** 🟢 — **apps/admin port 3010 scaffolded** com banner Modo Admin + 5 áreas planejadas + guard placeholder + initialColorMode="dark" desde scaffold

### DTs fechadas 🟢 (ciclo 2 — após review do Agente 1)

- **NEW-001** 🟢 (ciclo 2) — `railway.json` 6 apps base reescritos (`bun run --filter=<APP> build/preview` sem turbo) → fecha **DT-WEB-001** e desbloqueia deploy prod Railway
- **NEW-003** 🟢 (ciclo 2) — `extractRequestId(error)` em `apps/auth/src/lib/api.ts` + axios request interceptor gera `X-Request-Id` UUID por request; `defaultErrorComponent` propaga ao `ServerErrorPage`. Em prod usuário vê referência real, não "—"
- **NEW-004** 🟢 (ciclo 2) — `biome.json` override `noUnknownProperty: off` removido; permanece apenas `noUnknownAtRules: off` em `**/main.css` (escopo correto para `@unocss preflights;`)
- **NEW-002** 🟢 falso positivo — divergência `tsconfig.json` admin vs 6 apps base era apenas **comentários** `/* modules */ /* type checking */`; idêntico funcional. Marcar reclassificado.
- **DT-WEB-008** 🟢 (ciclo 2) — token `--info` (azul) adicionado em :root + .dark dos 7 main.css + theme `info: { DEFAULT, foreground }` + shortcut `btn-info` em 7 uno.config.ts
- **DT-WEB-009** 🟢 (ciclo 2) — `btn-icon` 40×40 → **44×44** (h-11 w-11) em 7 uno.config.ts → atende WCAG 2.1 AA touch target
- **A-013** 🟢 (ciclo 2) — 5 apps base (cms/lms/forum/assembly/lp) `index.tsx` reescritos sem código morto + `initialColorMode="dark"` no ColorModeProvider; `routes/index.tsx` distintas por app (CMS/LMS/Forum/Assembly = placeholders próprios; LP mantém hero blob)
- **A-018 / FE-018** 🟢 (ciclo 2) — **5 test files novos** em `packages/ui/src/`: `button.test.tsx` (variants × sizes × polymorphic × disabled × class merge — 12 cases), `input.test.tsx` (validationState/Zod-aligned 12 cases), `card.test.tsx` (composição 7 cases), `otp-input.test.tsx` (length/disabled/aria 6 cases após mock ResizeObserver+IntersectionObserver no setup.ts), `tabs.test.tsx` (Kobalte Tabs 7 cases). Total 58 tests (era 13). Setup compartilhado mocka observers ausentes em jsdom

### DT em andamento / 🟡 deferred

- **DT-WEB-004 / A-006-bis** 🟡 — extração de `packages/ui/src/uno.preset.ts` foi **tentada e revertida** no ciclo 2: `@unocss/postcss` + `unconfig`/`bundle-require` falham ao resolver `import { masterSindicoPreset } from "ui/uno.preset"` cross-workspace em build-time. O preset.ts permanece em `packages/ui/src/uno.preset.ts` como **referência canônica conceitual** (não usado em runtime). Solução completa requer ou (a) `tsdown build` gerando `dist/uno.preset.js` + dual-export, ou (b) migrar para `@unocss/vite`/plugin equivalente. Postergada (não bloqueia M1)
- **DT-WEB-018 / A-WEB-2026-04-27-003** 🟠 — PKCE gap (auth-context POST direto sem PKCE) — requer ADR + decisão backend
- **DT-WEB-020 / A-WEB-2026-04-27-002** 🟠 — OTP via search params (LGPD) — requer mudança contratual `reset_token` no backend

### Novas tasks decorrentes

- **FE-300** S · root · Atualizar `package.json` raiz para incluir `apps/admin` no script de filter (FE-002 evolui para 7 apps)
- **FE-301** M · `apps/admin` · Implementar fluxo OIDC redirect ao login Master Síndico (após decisão D-### PKCE) + integração cookie `__Host-mastersindico-admin-session`
- **FE-302** M · `apps/admin` · Routes guard real: `beforeLoad` chama `GET /api/v1/admin/me` e redireciona em 401/403
- **FE-303** L · `apps/admin` · AD1-AD2 funcionalidades reais (KPI cards + Gestão Tenants suspender/reativar)

## Pass de execução Agente 1 (2026-04-28) — fixes apps/auth + packages

Detalhes em [[../../11-audit/audit-log/2026-04-28-agente-1-fixes]].

### DTs fechadas 🟢 (ciclo 3 — pós-Agente-2)

- **A-WEB-2026-04-28-001** 🟢 — `apps/auth/src/routes/_auth/forgot-password/index.tsx` — bug de build descoberto via `tsc --noEmit` (não detectado por audits anteriores): `<Input>` linha 74 e `<Button>` linha 93 usados sem import. Fix: `import { Button, Input } from "ui"`.
- **A-WEB-2026-04-28-002** 🟢 — `apps/auth/src/routes/_auth/callback.tsx:74` + `apps/auth/src/routes/_auth/logout.tsx:39` — `await auth.refresh()` em método inexistente. Nome correto é `refreshSession` (auth-context.tsx:72). Fix: rename.
- **A-WEB-2026-04-28-003** 🟢 — `apps/auth/src/env.d.ts` faltava `PUBLIC_USE_MOCKS` (usado em `api/client.ts`, `api/index.ts`, scripts `dev:mock` / `build:mock`). Fix: declarado `readonly PUBLIC_USE_MOCKS?: "true" | "false"` com JSDoc apontando vault.
- **A-WEB-2026-04-28-004** 🟢 — `packages/schemas/src/billing.ts MoneyCentsSchema` ganhou `.max(Number.MAX_SAFE_INTEGER, "...")` — defesa em profundidade IEEE-754 (cento de bilhões de centavos R$ ainda dentro do safe integer; agregados históricos > 2^53 abortam parse com mensagem específica).
- **A-WEB-2026-04-28-005** 🟢 — Zod 4 modernização packages/schemas:
  - `EmailSchema` → `z.email()` top-level
  - `UuidV7Schema` → `z.uuid({ version: "v7" })` ESTRITO (antes `z.string().uuid()` aceitava v1-v8)
  - `CepSchema` canonicalizado em `common.ts` (re-exportado em `institutional.ts` para backward compat; deduplicado de `auth.ts`)
  - 15 `z.string().datetime()` → `z.iso.datetime()` (auth/billing/institutional)
  - `z.string().url()` → `z.url()`
  - `MoneyCentsSchema` → `z.int()` alias top-level
  - **Novo**: `SignUpResponseSchema` (defesa contrato backend)
- **A-WEB-2026-04-28-006** 🟢 — `packages/ui/src/button.tsx`:
  - Base CVA agora usa shortcut `btn-base` do preset compartilhado + classes Button-específicas
  - Variant `info: "btn-info"` adicionado (preset já tinha shortcut)
- **A-WEB-2026-04-28-007** 🟢 — `apps/auth/src/api/http-client.ts`:
  - `signUp` valida response com `SignUpResponseSchema.safeParse` + erro explícito
  - `fetchSession` distingue 401 (`null`) de network/5xx (re-throw — TanStack Query mostra `isError`)
- **A-WEB-2026-04-28-008** 🟢 — `apps/auth/src/lib/api.ts`:
  - Interceptor 401 redirect global REMOVIDO (rotas públicas como `/sign-up` e `/forgot-password` não saltam mais para `/sign-in` em 401 do backend; `fetchSession` cuida do caso de sessão expirada)
  - Mantido apenas `403 NEW_DEVICE` redirect (1-device-policy IDN-026/D-135)
  - `extractRequestId` usa `error.config?.headers?.[...]` (acesso por chave AxiosHeaders) em vez de `.get?.()`
- **A-WEB-2026-04-28-009** 🟢 — `apps/auth/src/contexts/auth-context.tsx isAuthenticating` agora é signal próprio (`createSignal(false)`); antes refletia `signOutMutation.isPending` (bug oculto pela ausência de consumers).
- **A-WEB-2026-04-28-010** 🟢 — `apps/auth/src/api/mock-client.ts loadSessionFromStorage` valida com `AuthSessionSchema.safeParse` (paridade defensiva com `httpApiClient`).
- **A-WEB-2026-04-28-011** 🟢 — `apps/auth/src/routes/__root.tsx` `TanStackRouterDevtools` com guard `!import.meta.env.PROD` via `<Show>` — Rsbuild build-time tree-shake elimina dev-tools do bundle de produção.

### DTs fechadas 🟢 (ciclo 4 — fixes 🟡 menores apps/auth)

- **A-WEB-2026-04-28-018** 🟢 — `apps/auth/src/lib/role-labels.ts` novo: dedup de `ROLE_LABEL` (antes em `sign-in.tsx:12-19` e `sign-up.tsx:25-32`). Justificado em `apps/auth/src/lib` (não `packages/schemas`) porque é label UI, não validation.
- **A-WEB-2026-04-28-019** 🟢 — `apps/auth/src/routes/_auth/logout.tsx`: `toast.success` agora diferencia sucesso (servidor revogou) vs warning (servidor falhou) — UX honesta sobre estado do logout.
- **A-WEB-2026-04-28-020** 🟢 — 4 signup forms (resident/syndic/enterprise/local-company) propagam 1ª mensagem específica do Zod em vez de "Verifique os campos." genérica — UX informativa + acessibilidade ARIA live region.

### DTs fechadas 🟢 (ciclo 5 — auditoria apps/lms + replicação patterns auth)

- **A-WEB-2026-04-28-021** 🟢 — `apps/lms/src/routes/__root.tsx`: `TanStackRouterDevtools` com guard `!import.meta.env.PROD` via `<Show>` (mesmo bug que apps/auth tinha — replicado fix). Tree-shake elimina dev-tools do bundle prod.
- **A-WEB-2026-04-28-022** 🟢 — `apps/lms/src/env.d.ts`: declarado `PUBLIC_USE_MOCKS?: "true" | "false"` (paridade com apps/auth — DT-WEB-012 web-wide).
- **A-WEB-2026-04-28-023** 🟢 — `apps/lms/src/main.css`: removidas linhas 190-329 (140 linhas) de LP blob dead CSS

### DTs fechadas 🟢 (ciclo 6 — paridade cross-app de fixes em todos os apps base)

Auditorias paralelas (apps/cms via agente background) revelaram que **forum, assembly, admin** tinham os MESMOS 3 gaps que `lms` tinha — todos copiaram o template antigo de `apps/auth` antes do fix do Devtools guard ser aplicado. Aplicada paridade:

- **A-WEB-2026-04-28-024** 🟢 — `apps/cms/src/routes/__root.tsx`: Devtools guard `!import.meta.env.PROD` via `<Show>` (mesmo fix do lms/auth).
- **A-WEB-2026-04-28-025** 🟢 — `apps/cms/src/env.d.ts`: declarado `PUBLIC_USE_MOCKS?: "true" | "false"`.
- **A-WEB-2026-04-28-026** 🟢 — `apps/cms/src/main.css`: removidas linhas 190-329 (LP blob dead CSS).
- **A-WEB-2026-04-28-027** 🟢 — `apps/forum/src/routes/__root.tsx`: Devtools guard.
- **A-WEB-2026-04-28-028** 🟢 — `apps/forum/src/env.d.ts`: `PUBLIC_USE_MOCKS` declarado.
- **A-WEB-2026-04-28-029** 🟢 — `apps/forum/src/main.css`: LP blob dead CSS removido (329 → 189 linhas).
- **A-WEB-2026-04-28-030** 🟢 — `apps/assembly/src/routes/__root.tsx`: Devtools guard.
- **A-WEB-2026-04-28-031** 🟢 — `apps/assembly/src/env.d.ts`: `PUBLIC_USE_MOCKS` declarado.
- **A-WEB-2026-04-28-032** 🟢 — `apps/assembly/src/main.css`: LP blob dead CSS removido.
- **A-WEB-2026-04-28-033** 🟢 — `apps/admin/src/routes/__root.tsx`: Devtools guard.
- **A-WEB-2026-04-28-034** 🟢 — `apps/admin/src/env.d.ts`: `PUBLIC_USE_MOCKS` declarado.
- **A-WEB-2026-04-28-035** 🟢 — `apps/admin/src/main.css`: LP blob dead CSS removido.

`apps/lp` NÃO tocado — landing page deve manter os blobs LP (uso real). 5 apps base agora consistentes (auth ainda original, mas era o template para os outros — recebeu o fix Devtools no lote 4).

**Total CSS removido**: 5 apps × 140 linhas LP blob = **700 linhas dead CSS**. Bundle de prod cada app reduziu (TBM medir após `bun run build` final).

### DTs fechadas 🟢 (ciclo 7 — BeyondCorp + cleanup + code quality)

- **A-WEB-2026-04-28-036** 🟢 — `apps/auth/src/lib/sensitive-flow.ts` novo + warning vault `12-docs/lgpd-flow-token-warning.md`. LGPD email out of search params (DT-WEB-012 stopgap; evolução para token opaco backend documentada).
- **A-WEB-2026-04-28-037** 🟢 — refactor 4 rotas para usar sensitive-flow: `sign-up.tsx`, `verify-email.tsx`, `forgot-password/index.tsx`, `forgot-password/code.tsx`. Email NÃO trafega mais por URL.
- **A-WEB-2026-04-28-038** 🟢 — `apps/lp/src/routes/__root.tsx` Devtools guard `<Show when={IS_DEV}>` (era CRÍTICO — landing page pública em prod expunha devtools).
- **A-WEB-2026-04-28-039** 🟢 — `apps/auth/src/routes/_auth/callback.tsx` `createEffect` → `onMount` (PKCE code single-use; re-execução causava 401 invalid_grant).
- **A-WEB-2026-04-28-040** 🟢 — `apps/auth/src/routes/__root.tsx`: `SolidQueryDevtools` ativo com guard IS_DEV (era comentado mesmo com dep instalada).
- **A-WEB-2026-04-28-041** 🟢 — `@tanstack/zod-form-adapter@0.42.1` instalado via `bun add`.
- **A-WEB-2026-04-28-042** 🟢 — `apps/auth/src/lib/device-fingerprint.ts` novo: SHA-256 hash de UA+canvas+screen+tz+cores. BeyondCorp Tenet 5 (1-Device Policy IDN-026/D-135). SSR-safe + cached.
- **A-WEB-2026-04-28-043** 🟢 — `apps/auth/src/lib/api.ts` interceptor async injeta `X-Device-Fingerprint` em todo request. Backend correlaciona device login.
- **A-WEB-2026-04-28-044** 🟢 — `web/old/` legacy folder removida (4939L profiles-context.ts + auth-context.tsx). Estava fazendo tsc cross-workspace falhar com 5 erros falsos. Era gitignored, sem imports.
- **A-WEB-2026-04-28-045** 🟢 — `logout.tsx` `if/else` → ternário (Code Calisthenics #2 no-else 100% compliance no apps/auth/src).

### DTs fechadas 🟢 (ciclo 8 — Red team defense-in-depth hardening)

Red team scan completo (10 vetores, 0 exploits, 5 gaps fechados):

- **A-WEB-2026-04-28-050** 🟢 — `apps/auth/src/lib/sensitive-flow.ts`: `StoredEnvelopeSchema` Zod rigoroso (EmailSchema + expiresAt window com clock-skew + meta.record). Bloqueia tampering via DevTools/extensions.
- **A-WEB-2026-04-28-051** 🟢 — `apps/auth/src/lib/otp-flow.ts`: `OtpFlowEnvelopeSchema` Zod com `code` regex `/^\d{6}$/` + EmailSchema. Mesmo nível.
- **A-WEB-2026-04-28-052** 🟢 — `apps/auth/src/contexts/auth-context.tsx readPendingVerificationEmail`: `EmailSchema.safeParse` antes de expor localStorage value; auto-limpa corrompido.
- **A-WEB-2026-04-28-053** 🟢 — `apps/auth/src/routes/_auth/callback.tsx`: `sanitizeReturnTo()` allowlist (`["/", "/welcome", "/sign-in", "/dashboard", "/perfil", "/admin"]`) bloqueia `//evil.com`, `javascript:`, `data:`, `vbscript:` + `CallbackSearchSchema` rigoroso (code/state min/max length, error/desc capped).
- **A-WEB-2026-04-28-054** 🟢 — `bun remove oidc-client-ts` (era unused; backend Go faz exchange-code; reduz superfície supply chain).
- **A-WEB-2026-04-28-055** 🟢 — `bun update follow-redirects@1.16.0 postcss@8.5.12` — fecha 2 CVEs moderate (GHSA-r4q5-vmmm-2653 + GHSA-qx2v-qp2m-jg93).

### DTs ainda pendentes (próximos lotes)

- **DT-WEB-2026-04-28-056** 🟡 — Defense-in-depth opcional (red team skipped): PKCE state local nonce client-side (requer coordenação backend) + double-submit `inFlight` flag por mutation. `isPending` já mitiga 99.9% no caso double-submit.

### DTs fechadas 🟢 (ciclo 9 — atoms M1 packages/ui)

- **A-WEB-2026-04-28-057** 🟢 — `packages/ui/src/spinner.tsx`: i-lucide-loader-2 + animate-spin, sizes sm/md/lg, `role="status"` + aria-label "Carregando" override. Para `pendingComponent` TanStack Router + boot loading. 6 tests TDD.
- **A-WEB-2026-04-28-058** 🟢 — `packages/ui/src/skeleton.tsx`: animate-pulse + bg-muted, aria-hidden decorativo. Para listas com `useQuery` isLoading. 4 tests.
- **A-WEB-2026-04-28-059** 🟢 — `packages/ui/src/badge.tsx`: CVA 6 variants (default/success/warning/destructive/info/outline). Para plan status, reputação, persona role, Connect Me state. 13 tests.

### DTs fechadas 🟢 (ciclo 10 — Kobalte primitives)

- **A-WEB-2026-04-28-060** 🟢 — `packages/ui/src/checkbox.tsx`: Kobalte primitive composto (label + description + error + validationState). Para FE-061 governance markers, LGPD consent. 4 tests.
- **A-WEB-2026-04-28-061** 🟢 — `packages/ui/src/switch.tsx`: Kobalte primitive composto. Para SYS3 prefs, dark mode, S5 5-toggle. 4 tests.
- **A-WEB-2026-04-28-062** 🟢 — `packages/ui/src/form-field.tsx`: wrapper composição para custom inputs (dropzone, masked, custom selects). label + description + error com `role="alert"` + asterisk required. 6 tests.

### DTs fechadas 🟢 (ciclo 11 — error-pages cross-app — DT-047)

- **A-WEB-2026-04-28-063** 🟢 — `packages/ui/src/error-pages.tsx` novo: NotFoundPage/ServerErrorPage/ForbiddenPage genéricos. Sem dependência TanStack Router (usam `<a href>` puro). Recebem `homeHref`/`signInHref`/`isProd` via props. 11 tests cobrindo prod vs dev message + custom hrefs + reset visibility.
- **A-WEB-2026-04-28-064** 🟢 — `apps/auth/src/index.tsx` refactor: importa de `"ui"` em vez de `components/error-pages.tsx` (deletado). `IS_PROD` flag para suprimir error.message em prod. Test antigo removido (dedup com packages/ui).
- **A-WEB-2026-04-28-065** 🟢 — Paridade 5 apps base (cms/lms/forum/assembly/admin): `defaultNotFoundComponent` + `defaultErrorComponent` adicionados via NotFoundPage/ServerErrorPage de `"ui"`. Antes comentados — exposição de stack traces em prod (vault `security.md §5`).
- `lp/` mantido sem error pages — landing pública ganha 404 customizado em M2 com SEO.

### DTs ainda pendentes (próximos lotes)

### DTs fechadas 🟢 (ciclo 12 — Dialog + Toaster — DT-048 100% atoms M1)

- **A-WEB-2026-04-28-067** 🟢 — `packages/ui/src/dialog.tsx`: Kobalte Dialog primitive composto (Portal + Overlay + Content + Title + Description + CloseButton). UnoCSS animations data-[expanded]/[closed]. Title/description obrigatórios para a11y. Para confirmações destrutivas, LGPD consent, edit overlays. 4 tests via `screen.*` (portal renderiza fora do container).
- **A-WEB-2026-04-28-068** 🟢 — `packages/ui/src/toaster.tsx`: re-export solid-sonner Toaster + toast. Centraliza host em packages/ui (SOLID DIP — apps dependem de abstração, não concretion). Apps podem trocar provider futuramente sem cascade changes. `bun add solid-sonner` em packages/ui.
- **A-WEB-2026-04-28-069** 🟢 — `apps/auth/src/index.tsx` importa `Toaster` de `"ui"` em vez de `"solid-sonner"` direto. (Imports de `toast()` em rotas/components mantidos — mesma referência via re-export; cleanup pass futuro opcional.)

### Atoms FE-013 — 100% done ✅

| Atom | Origem | Status |
|---|---|---|
| Button | template Master Síndico | ✅ |
| Input + Composable | Kobalte TextField | ✅ |
| Tabs | Kobalte | ✅ |
| Card + sub-components | custom | ✅ |
| ProgressDots | custom (a11y progressbar) | ✅ |
| OtpInput | @corvu/otp-field | ✅ |
| PasswordStrength | custom | ✅ |
| Spinner | custom (i-lucide + animate-spin) | ✅ NOVO |
| Skeleton | custom (animate-pulse) | ✅ NOVO |
| Badge | custom CVA 6 variants | ✅ NOVO |
| Checkbox | Kobalte | ✅ NOVO |
| Switch | Kobalte | ✅ NOVO |
| FormField | composição custom | ✅ NOVO |
| Dialog | Kobalte | ✅ NOVO |
| Toaster | solid-sonner re-export | ✅ NOVO |

**15 componentes total em packages/ui.**

### DTs ainda pendentes (próximos lotes)

### DTs fechadas 🟢 (ciclo 13 — DT-046 zod-form-adapter migration)

- **A-WEB-2026-04-28-070** 🟢 — `packages/ui/src/input.tsx`: prop `onBlur?: () => void` adicionada, repassada a `TextField.Input`. API mínima preserva contratos existentes.
- **A-WEB-2026-04-28-071** 🟢 — `apps/auth/src/components/signup-forms/resident-form.tsx`: `validatorAdapter: zodValidator()` + `validators: { onBlur: SignupResidentSchema.shape.{name,email,cpf} }` por campo. Feedback inline ao sair do campo.
- **A-WEB-2026-04-28-072** 🟢 — `syndic-form.tsx`: idem com schema + phone.
- **A-WEB-2026-04-28-073** 🟢 — `enterprise-form.tsx`: idem com 5 fields (companyName/cnpj/email/contactName/phone).
- **A-WEB-2026-04-28-074** 🟢 — `local-company-form.tsx`: idem com 5 fields (businessName/cnpj/email/cep/address).
- **A-WEB-2026-04-28-075** 🟢 — `apps/auth/src/routes/_auth/set-password.tsx`: `PasswordSchema` validator onBlur no password; cross-field `fieldApi.form.getFieldValue("password")` no confirmPassword.
- **A-WEB-2026-04-28-076** 🟢 — `forgot-password/new-password.tsx`: mesmo pattern.

Pattern padrão aplicado nos 6 forms:
1. import `zodValidator` + schema relevante
2. `createForm` com `validatorAdapter: zodValidator()`
3. cada `form.Field` com `validators: { onBlur: SchemaShape.field }`
4. cada `Input` com `onBlur={() => f().handleBlur()}`
5. error coerced via `.toString()` (adapter retorna `ValidationError`)
6. `safeParse` no submit mantido como defesa em profundidade

Resolve Findings F1 + F2 + F3 da auditoria TanStack (Q) — adapter instalado no ciclo 6 estava unused; agora alavancado.
- **DT-WEB-2026-04-28-047** 🟠 — `errorComponent`/`notFoundComponent` em 5 apps base (cms/lms/forum/assembly/admin/lp). Extrair `error-pages.tsx` de auth para `packages/ui` ou copiar.
- **DT-WEB-2026-04-28-048** 🟠 — packages/ui Atoms M1: `Spinner`, `Skeleton`, `Checkbox`, `Switch`, `Badge`, `FormField`, `Dialog`, `Toast` host. Kobalte primitives existem para 5 dos 8.
- **DT-WEB-2026-04-28-049** 🟢 — `device-fingerprint.ts` é STOPGAP M1; M2 evoluir para Play Integrity (Android), DeviceCheck (iOS), reCAPTCHA Enterprise (web). Documentado no JSDoc.

### Métricas finais sessão 2026-04-28 (após ciclo 7)

| Métrica | Pré-sessão | Pós-sessão |
|---|---|---|
| `bun run check` cross-workspace | 0/0/1 (auth) | **0/0/0** em todos 7 apps |
| `bunx tsc --noEmit` apps/auth | 2 erros | **0 erros** |
| Tests packages/schemas | 93 | **93** |
| Tests packages/ui | 58 | **58** |
| Tests apps/auth | 24 | **24** |
| Bugs CRÍTICOS de build | 3 | **0** ✅ |
| Findings 🔴 apps/auth | 4 | **0** ✅ |
| Findings 🔴 apps/cms/lms/forum/assembly/admin (Devtools guard) | 5 | **0** ✅ |
| Findings 🟠 fechados | — | 17 |
| Linhas LP blob dead CSS removidas | — | **700** |
| DTs fechadas 🟢 (A-WEB-2026-04-28-001..035) | 0 | **35** | (`.lp-hero`, `.lp-blob-purple-corner`, etc.) — eram cópia da landing page sem uso no LMS, hex hardcoded `#06080f` `#0a1628` etc. Bundle reduzido. Final: 189 linhas.

### DTs ainda pendentes (próximos lotes Agente 1)

- **DT-WEB-2026-04-28-024** 🟠 — `apps/lms/package.json` sem scripts test + devDeps de teste (vitest, @solidjs/testing-library, msw, jsdom). Aplicar via `bun add -d ...` (regra do João: nunca editar package.json direto). Mesmo gap em outros apps base provavelmente.
- **DT-WEB-2026-04-28-012** 🟠 — Email em search params LGPD (`sign-up.tsx:68`, `forgot-password/index.tsx:33-46`, `verify-email.tsx:10`). Solução: estender `otp-flow.ts` com chave `"ms.signup-init"` / `"ms.recover-init"` na etapa 1; etapas seguintes leem do sessionStorage. Vault `08-security/security.md §5` + LGPD art. 5º + STEERING #20. Lote dedicado.
- **DT-WEB-2026-04-28-013** 🟡 — `ROLE_LABEL` duplicado em `sign-in.tsx:12-19` e `sign-up.tsx:25-32`. Mover para `packages/schemas` ou util compartilhado.
- **DT-WEB-2026-04-28-014** 🟡 — `logout.tsx:40` `toast.success` enganoso quando backend falha (deveria diferenciar "sessão local encerrada" de "sessão servidor revogada").
- **DT-WEB-2026-04-28-015** 🟡 — MSW handlers stale: `POST /auth/sign-in` morto pós-PKCE; sem handler para `GET /me` autenticado.
- **DT-WEB-2026-04-28-016** 🟡 — Signup forms (4) mensagem genérica `"Verifique os campos."` deveria propagar `parsed.error.flatten().fieldErrors` para ARIA live region.
- **DT-WEB-2026-04-28-017** 🔵 — Brand types em VOs (`Cpf`, `Cnpj`, `Cep`, `FracaoIdeal`) — Code Calisthenics #3 mandate. Audit consumers em apps antes de aplicar.

### Métricas finais (smoke completo após lotes 1-4)

| Métrica | Pré-sessão | Pós-sessão |
|---|---|---|
| `bun run check` (Biome) cross-workspace | 0/0/1 (auth) | **0/0/0** em todos 7 apps |
| `bunx tsc --noEmit` apps/auth | 2 erros (callback.tsx, logout.tsx — `refresh`) | **0 erros** |
| Tests packages/schemas | 93 | **93** (mantido após Zod 4 mods + SignUpResponseSchema) |
| Tests packages/ui | 58 | **58** (mantido após Button info + base shortcut) |
| Tests apps/auth | 24 | **24** (mantido após http-client + lib/api refactors) |
| Bugs CRÍTICOS de build | 3 (forgot-password, callback, logout) | **0** ✅ |
| Findings 🔴 apps/auth | 4 (lib/api 401, http-client signUp+fetchSession, Button variant) | **0** ✅ |
| Findings 🟠 fechados | — | 6 (isAuthenticating, mock JSON, devtools prod, MoneyCents max, EmailSchema, UuidV7 strict) |

## Links

- [[AUDIT-2026-04-27]] — auditoria sistemática 26 findings
- [[../../11-audit/audit-log/2026-04-28-agente-1-fixes]] — fixes sessão 2026-04-28
- [[../../11-audit/audit-log/2026-04-27-agente-2-systematic-pass]] — pass de execução Agente 2 (Lotes 1-7)
- [[README]]
- [[architecture]]
- [[patterns]]
- [[requirements]]
- [[design-system]]
- [[security]]
- [[ui-catalog]]
- [[../../ROADMAP]]
- [[../../STATE]]
- [[../../DoR-DoD]]
- [[_moc]]
