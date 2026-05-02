---
title: Web — Requirements específicos
type: requirement
tags: [subproject, web, master-sindico, v2, requirements, performance, a11y, pwa, core-web-vitals]
source: Development/web/CLAUDE.md + 04-requirements/global.md + WCAG 2.1 AA
created: 2026-04-23
updated: 2026-04-23
---

# Web — Requirements específicos

Requirements que **apenas o web implementa** — além dos canônicos em [[../../04-requirements/global]] e da stack já confirmada em [[README]]. Complementa [[architecture]], [[patterns]], [[design-system]] e [[security]].

Fontes:
- `Development/web/CLAUDE.md` §TDD + §Testing (linhas 226-270).
- `Development/web/AGENTS.md` §Filosofia Geral + §Segurança.
- WCAG 2.1 AA (web.dev + W3C).
- Core Web Vitals 2025 targets.

---

## 1. Performance — Core Web Vitals (BR-WEB-PERF)

### BR-WEB-PERF-001 — Targets CWV

| Métrica | Mobile (4G) | Desktop | Observação |
|---|---|---|---|
| **LCP** | < 2.5s | < 1.5s | Hero image/headline do LP; primeira tela autenticada após redirect |
| **CLS** | < 0.1 | < 0.1 | Reservar `width/height` em images; skeletons sem reflow |
| **INP** | < 200ms | < 200ms | Interação percebida fluida (click → pintura) |
| **TTFB** | < 600ms | < 300ms | Rsbuild SSG em LP; CDN Railway |
| **FCP** | < 1.8s | < 1.0s | Preload fonts (Michroma + Manrope) |

Coleta via `web-vitals` lib; envia `POST /api/v1/metrics/web-vitals` (body Zod-validated).

### BR-WEB-PERF-002 — Bundle budget (gzip)

| Target | Limite |
|---|---|
| LP (`apps/lp`) initial JS | < **80 KB** |
| App autenticado initial JS (`apps/cms` primeira rota) | < **300 KB** |
| Chunk lazy por rota pesada | < **150 KB** |
| CSS total | < **50 KB** (UnoCSS on-demand ajuda) |

Verificação CI: `rsbuild build --analyze` + threshold check.

### BR-WEB-PERF-003 — Cache strategy

| Asset | Header | Motivo |
|---|---|---|
| JS/CSS hash-named | `Cache-Control: public, max-age=31536000, immutable` | Imutável via hash |
| HTML | `Cache-Control: no-cache, must-revalidate` | SPA updates |
| Imagens hash-named | `public, max-age=31536000, immutable` | — |
| API (JSON) | `private, no-cache` | Dados mutáveis |
| Webfonts (Google) | `public, max-age=31536000` | — |

### BR-WEB-PERF-004 — Imagens

- **Formatos**: AVIF → WebP → JPEG (fallback em `<picture>`).
- **Lazy loading**: `loading="lazy"` em below-the-fold.
- **Dimensões explícitas**: `width`/`height` attrs obrigatórios (previne CLS).
- **Responsive**: `srcset` com 1x/2x/3x para logos + hero.
- **Decoding**: `decoding="async"` exceto hero/LCP.

### BR-WEB-PERF-005 — Fonts

- **font-display: swap** — evita invisible text.
- **Preload** em `<head>` do `index.html`:
  ```html
  <link rel="preload" href="..." as="font" type="font/woff2" crossorigin>
  ```
- **WOFF2 only** em prod (Michroma + Manrope do Google Fonts).
- **Subset** latin + latin-ext (pt-BR + EN futuro).

---

## 2. Acessibilidade WCAG 2.1 AA (BR-WEB-A11Y)

### BR-WEB-A11Y-001 — Conformidade

**Obrigatório**: WCAG 2.1 **AA** em todas as telas. Meta stretch AAA em fluxos críticos (voto, pagamento).

Critérios mais violados que passam por auditoria:
- **1.4.3 Contraste** (texto 4.5:1, texto grande 3:1) — tokens calibrados.
- **1.4.11 Contraste UI** (3:1 para bordas, ícones informativos).
- **2.1.1 Keyboard** — toda ação navegável/ativável via teclado.
- **2.4.3 Focus order** — DOM order lógico, tabindex só quando necessário.
- **2.4.7 Focus visible** — `focus-visible:ring-2 ring-ring ring-offset-2` em todo elemento focável.
- **3.3.1 Error identification** — inputs com `aria-invalid` + `aria-describedby` apontando erro.
- **4.1.2 Name/role/value** — todo componente custom tem `role` + nome acessível.
- **4.1.3 Status messages** — toasts com `role="status"` ou `aria-live="polite"`.

### BR-WEB-A11Y-002 — Ferramental

- **Kobalte** primitives — cobre 80% dos requisitos automaticamente (focus trap, ARIA, keyboard).
- **Vitest + jest-axe** — zero violations em unit tests.
- **Playwright + @axe-core/playwright** — E2E a11y em fluxos críticos (M2+).
- **Manual** — NVDA (Windows) + VoiceOver (macOS/iOS) em fluxos: login, cadastro, assembleia votação, Connect Me envio.

### BR-WEB-A11Y-003 — Copy acessível

- Linguagem clara, sem jargão técnico.
- Botões com verbo claro ("Enviar Connect Me", não "OK"/"Prosseguir" genérico).
- Mensagens de erro humanas ("Este CPF já está cadastrado" em vez de "UNIQUE_VIOLATION").
- `aria-label` para ícones-only buttons (`<button aria-label="Fechar"><i class="i-lucide-x" /></button>`).

### BR-WEB-A11Y-004 — Preferências do usuário

- **Reduce motion** respeitado: `@media (prefers-reduced-motion: reduce)` desativa `animate-fade-in`, `animate-slide-in-right`.
- **Color scheme**: Kobalte `ColorModeProvider` detecta `prefers-color-scheme` automaticamente; override manual persiste em `localStorage` (só a preferência, **nunca** PII).
- **Font size**: respeita `rem` (não `px` fixo em body); zoom até 200% sem quebra.

---

## 3. Responsividade (BR-WEB-RESP)

### BR-WEB-RESP-001 — Breakpoints UnoCSS

| Breakpoint | Min-width | Dispositivo target |
|---|---|---|
| (default, mobile) | 0 | iPhone SE 375px, Android base |
| `sm:` | 640px | Phones landscape, small tablets |
| `md:` | 768px | Tablets portrait (iPad) |
| `lg:` | 1024px | Tablets landscape, laptops pequenos |
| `xl:` | 1280px | Desktops |
| `2xl:` | 1536px | Monitores grandes |

### BR-WEB-RESP-002 — Prioridade mobile-first

- Layout base sem prefixo (mobile).
- Progressively enhance com `sm:`, `md:`, ... (desktop).
- Teste em devices reais ou Chrome DevTools emulation: iPhone 13, Galaxy S21, iPad Air.

### BR-WEB-RESP-003 — Touch targets

- Mínimo **44×44 px** em áreas clickáveis (botão, ícone, link em nav). Shortcut `btn-icon = h-10 w-10 p-0` = 40×40, precisa revisão (M1 FE-backlog).
- Espaçamento mínimo 8px entre targets adjacentes.

### BR-WEB-RESP-004 — Orientation

App deve funcionar portrait + landscape. Telão da assembleia (telão 2 áreas) assume landscape/desktop.

---

## 4. PWA (BR-WEB-PWA)

### BR-WEB-PWA-001 — Manifest

Por app que fizer sentido instalar (`cms`, `forum`, `assembly`):

```json
// apps/cms/public/manifest.json
{
  "name": "Master Síndico — Gestão",
  "short_name": "MS Gestão",
  "icons": [{ "src": "/icon-192.png", "sizes": "192x192" }, { "src": "/icon-512.png", "sizes": "512x512" }],
  "start_url": "/",
  "display": "standalone",
  "background_color": "#0E1118",
  "theme_color": "#C69C3F"
}
```

### BR-WEB-PWA-002 — Service Worker (backlog M3+)

- **NÃO MVP** — Mux player + WebSocket + cookie httpOnly já atendem.
- M3+ opcional: Workbox para offline de telas read-only (timeline, plano diretor, eventos publicados).
- Cache strategy: NetworkFirst para API; CacheFirst para assets hash-named.

### BR-WEB-PWA-003 — Install banner

- Respeitar `beforeinstallprompt`; mostrar CTA só após 2ª visita + tempo de engajamento > 30s.

---

## 5. i18n (BR-WEB-I18N)

### BR-WEB-I18N-001 — Escopo MVP

- **Idioma único MVP**: pt-BR.
- **EN preparado** (M3+) via arquitetura i18n mas não ativo.

### BR-WEB-I18N-002 — Biblioteca

- `@solid-primitives/i18n` (M2 — FE-backlog) ou dicionário manual por app.
- Chaves em EN (`auth.sign_in.title`), traduções em pt-BR.

### BR-WEB-I18N-003 — Termos preservados em pt-BR

Não traduzir mesmo quando EN ativar (termos regulatórios/contextuais BR):

- Síndico, Condomínio, Assembleia, Edital, Pauta, Fração Ideal
- Plano Diretor, Timeline (ou "Linha do Tempo"), Gestão, Mandato
- Connect Me (brand)
- Outorgante, Outorgado (procuração)
- Morador, Unidade, Bloco
- "ata", "quórum", "adendo"

### BR-WEB-I18N-004 — Formatação

- Datas: `DD/MM/YYYY HH:mm` (America/Sao_Paulo).
- Moeda: `R$ 1.234,56` (Intl.NumberFormat `pt-BR`, `style: 'currency'`).
- Números: `1.234,56` (vírgula decimal).
- Endereço: CEP 8 dígitos com hífen `00000-000`.

---

## 6. Forms (BR-WEB-FORMS)

### BR-WEB-FORMS-001 — Stack

**TanStack Solid Form + Zod adapter** (`packages/schemas`). Ver [[patterns#44]].

### BR-WEB-FORMS-002 — Validação

- **Synchronous onChange** para feedback imediato (required, format, min/max length).
- **Async onBlur** para validações remotas (CPF disponível, CNPJ Receita Federal).
- **onSubmit** consolidado — bloqueia até tudo válido.

### BR-WEB-FORMS-003 — Double-confirm em fluxos críticos

Ações irreversíveis exigem dialog de confirmação:

- Publicar comunicado oficial.
- Fechar Connect Me (após contato revelado).
- Homologar ata de assembleia.
- Encerrar mandato síndico.
- Publicar registro de execução (vira validação pendente).
- **Admin MS**: suspender tenant, ban usuário, broadcast global.

Dialog copy: **"Tem certeza? Esta ação não pode ser desfeita. [Cancelar] [Confirmar <ação>]"**. Confirmação exige clicar no botão (não submit-on-enter).

### BR-WEB-FORMS-004 — Voice input

AGENTS.md §Análise de Telas R2: "Voz em todas descrições". Implementação:

- Web Speech API (`SpeechRecognition`) onde suportado.
- Mic button ao lado de textarea `<i class="i-lucide-mic" />` → transcreve em pt-BR.
- Fallback invisível quando API não disponível (não quebra UX).
- Telas obrigatórias: criar atividade timeline (S12), criar comunicado (S24), criar evento (S16), registrar execução (E5), ata manual de assembleia.

### BR-WEB-FORMS-005 — Auto-save

Formulários multi-step (onboarding 4-7 telas, assembleia config):
- Auto-save em Redis backend a cada step completo.
- Indicador "Rascunho salvo às 14:32".
- Recuperação se conexão cair: `GET /api/v1/onboarding/progress` retorna step atual + data.

---

## 7. Upload (BR-WEB-UPLOAD)

### BR-WEB-UPLOAD-001 — Mux signed URL

Fluxo padrão vídeo:
1. Frontend solicita `POST /api/v1/videos/upload-ticket` → backend retorna `{uploadUrl, uploadId}` (Mux Direct Upload).
2. Frontend envia arquivo direto ao Mux via `uploadUrl` (PUT multipart).
3. Backend recebe webhook Mux `video.asset.ready` → atualiza DB.
4. Frontend escuta via polling `GET /api/v1/videos/:id` ou via WS notification.

### BR-WEB-UPLOAD-002 — Limites por persona

| Persona | Tipo | Limite tamanho | Limite duração | Lock |
|---|---|---|---|---|
| Síndico N1 | Institucional | 500 MB | 10 min | 90d |
| Síndico N2 | Institucional | 500 MB | 10 min | 90d — 4/mês |
| Síndico N3 | Institucional | 1 GB | 15 min | 90d — ilimitado |
| Empresa Plus | Institucional | 1 GB | 10 min | 90d — 8/mês |
| Empresa Pro | Institucional | 1 GB | 15 min | 90d — 12/mês |
| Morador | Vídeo-currículo 90s | 500 MB | 90s | 90d |

Frontend **reflete** limites do backend (`GET /api/v1/user/plan-limits`), nunca calcula.

### BR-WEB-UPLOAD-003 — UI

- Dropzone com preview.
- Progress bar (chunks Mux).
- Cancel + retry.
- Validação client-side (tipo, tamanho, duração estimada) **antes** de enviar — UX, não segurança.

---

## 8. Realtime (BR-WEB-REAL)

### BR-WEB-REAL-001 — WebSocket (Assembleia Live)

- `apps/assembly` mantém singleton `assembly-ws.ts`.
- **ReconnectingWebSocket** wrapper (backoff 1s → 2 → 4 → 8 → 16 → 30s cap).
- Mensagens tipadas Zod (`packages/schemas/src/assembly.ts`).
- Buffer de mensagens durante disconnect (spinner "Reconectando...").
- Heartbeat ping 30s.

### BR-WEB-REAL-002 — SSE (backlog M2)

Eventos não-críticos que não precisam WS bidirecional:
- Notificações in-app.
- Status de upload de vídeo.
- Progresso de processamento de relatório.

### BR-WEB-REAL-003 — Polling fallback

Para ambientes que bloqueiam WS/SSE (corporate proxies):
- Fallback automático via TanStack Query `refetchInterval: 5000`.

---

## 9. Error handling (BR-WEB-ERR)

### BR-WEB-ERR-001 — Camadas

1. **Loader-level** (TanStack Router `errorComponent`): erro ao carregar dados da rota → tela dedicada com retry.
2. **Component-level** (`ErrorBoundary`): erro de render → fallback amigável + reporta Sentry.
3. **Global** (`__root.tsx` ErrorBoundary): última linha → tela "Ops, algo deu errado" + link para suporte.
4. **Mutation-level** (TanStack Query `onError`): toast de erro + não apaga input do usuário.

### BR-WEB-ERR-002 — Copy

Erros para usuário humanizados; nunca exibir stack trace em prod. Exemplos:

| HTTP | Copy UI |
|---|---|
| 401 | "Sessão expirada. Faça login novamente." → redirect |
| 403 DEVICE_MISMATCH | "Você está conectado em outro dispositivo. Para continuar aqui, desconecte o outro." |
| 403 genérico | "Você não tem permissão para esta ação." |
| 404 (recurso) | "Este item não existe ou foi removido." (evita 403 revelador — ver [[security#9-idor]]) |
| 409 CONFLICT | "Este cadastro já existe." (CPF/CNPJ duplicado) |
| 429 | "Muitas tentativas. Aguarde X segundos." |
| 500+ | "Problema no servidor. Tente novamente em alguns minutos." + reporta Sentry |
| Network | "Sem conexão. Verifique sua internet." |

---

## 10. Observability (BR-WEB-OBS)

### BR-WEB-OBS-001 — Sentry

- `@sentry/browser` + `@sentry/integrations` por app.
- `release: hash-do-commit` para deploy tracking.
- **PII scrubbing**: configurar `beforeSend` removendo `email, cpf, cnpj, phone` de breadcrumbs e exceptions.
- Source maps upload em build (via `@sentry/vite-plugin` equivalente para Rsbuild).

### BR-WEB-OBS-002 — Web Vitals

- `web-vitals` lib envia métricas.
- Endpoint backend `POST /api/v1/metrics/web-vitals` agrupa por route + persona + device.

### BR-WEB-OBS-003 — Logs console

- `PROD`: `console.log/debug/info` silenciados (Biome rule `no-console` em prod) — só `error` permitido.
- `DEV`: liberado; mas **nunca** PII mesmo em dev (hábito que migra pra prod).
- Helper: `log.maskedInfo(obj)` que mascara campos `/cpf|cnpj|email|phone|password/`.

---

## 11. SEO (BR-WEB-SEO) — aplicável ao `lp` principalmente

### BR-WEB-SEO-001 — Meta tags essenciais

Por rota do LP:
- `<title>` único, max 60 chars.
- `<meta name="description">` max 160 chars.
- Open Graph (`og:title`, `og:description`, `og:image`, `og:url`, `og:type`).
- Twitter Card.
- `<link rel="canonical">`.

### BR-WEB-SEO-002 — Estrutura

- 1 `<h1>` por página; hierarquia H1 → H2 → H3 sem pular.
- URLs semânticas (`/preco`, `/sobre`, `/blog/governanca-condominial`).
- Sitemap.xml gerado no build.
- robots.txt permitindo LP, bloqueando app autenticado.

### BR-WEB-SEO-003 — Structured data

`Product` / `Organization` schema.org JSON-LD em LP. `Article` em blog posts.

### BR-WEB-SEO-004 — SSG/SSR

- LP: SSG via Rsbuild (`rsbuild build --ssg`) — pre-rendered HTML por rota.
- Apps autenticadas: **CSR only** (sem benefício SEO).

---

## 12. Paywall e quotas — backend-first (BR-WEB-QUOTA)

### BR-WEB-QUOTA-001 — Regra raiz

**Frontend exibe; backend calcula + bloqueia.**

Nunca o frontend decide sozinho se ação é permitida. O backend responde 403 com código `QUOTA_EXCEEDED` / `PLAN_LIMIT` / `TRIAL_EXPIRED` → frontend mostra UI apropriada + CTA upgrade.

### BR-WEB-QUOTA-002 — UI padrão

Bloqueios:

```tsx
<QuotaBlocker
  reason="Você atingiu o limite de 2 Connect Me deste ano."
  resetAt={new Date('2027-01-01')}
  upgradeCta="Aumentar limite com Morador Pagante"
  upgradeHref="/account/subscription"
/>
```

Trial expirado:

```tsx
<TrialBlocker
  trialEndedAt={date}
  features={['Connect Me', 'Publicação de comunicados', 'Dashboard']}
  upgradeHref="/account/subscription"
/>
```

### BR-WEB-QUOTA-003 — Tiers Stripe-style

**Sem slugs** — tier universal: `trial`, `base`, `plus`, `pro`. Cada persona (síndico, morador, empresa, parceiro) tem seu próprio `PlanTier` que é essa mesma enumeração — sem nome específico por persona no código.

```ts
export const PlanTierSchema = z.enum(['trial', 'base', 'plus', 'pro']);
export type PlanTier = z.infer<typeof PlanTierSchema>;
```

A **apresentação** ao usuário (copy marketing) pode ser persona-aware ("Síndico N3 Consolidar" para `pro` síndico), mas o código/API usa só os 4 slots.

---

## 13. LGPD no frontend (BR-WEB-LGPD)

### BR-WEB-LGPD-001 — Consentimento

- **Cookie banner** em `lp` na primeira visita — aceita ou configura.
- **Granular**: necessários (sempre) · analytics (opt-in) · marketing (opt-in).
- Registra escolha em backend (`POST /api/v1/lgpd/consent` com versão do termo).

### BR-WEB-LGPD-002 — Connect Me modal

Antes de **revelar contato** via Connect Me, modal LGPD:

```
[Ícone cadeado]
Ao revelar este contato, você declara:
- Finalidade: contratar serviço para o condomínio X
- Dados compartilhados: nome, email, telefone da empresa Y
- Retenção: 5 anos (audit trail LGPD)
- Você consente com o registro desta ação.
[Cancelar] [Concordo e revelo]
```

Registro obrigatório em backend com `{user_id, target_id, timestamp, ip, consent_version, fingerprint}`.

### BR-WEB-LGPD-003 — Portabilidade e esquecimento

Em `/account/dados-pessoais`:
- **Exportar meus dados** → backend gera JSON; SPA faz download.
- **Excluir minha conta** → backend soft-delete + broadcast anonymization jobs; SPA confirma com double-confirm.

### BR-WEB-LGPD-004 — PII em telemetria

- Sentry `beforeSend` remove PII.
- Web Vitals sem user-identifier.
- Analytics (M3+): opt-in only.

---

## 14. Cobertura de testes (BR-WEB-TEST)

### BR-WEB-TEST-001 — Thresholds duros (CI gate)

| Camada | Coverage mínimo |
|---|---|
| `packages/schemas/` (Zod) | **95%** |
| `packages/ui/` (componentes) | **90%** |
| `apps/*/src/hooks/` (TanStack Query, lógica) | **85%** |
| `apps/*/src/routes/` (telas) | **75%** |
| `apps/*/src/lib/` (utils) | **90%** |
| **Global** | ≥ **80%** |

### BR-WEB-TEST-002 — Obrigatórios

- Vitest unit em todo componente/hook/util.
- Teste de validação em todo form (required, formato, bloqueio de submit inválido).
- MSW mock em toda chamada HTTP (sucesso, 401, 403, 500, timeout).
- Property-based (fast-check) em: quotas, trial expiração, visibilidade vídeo, fração ideal.
- jest-axe em toda tela: zero violations.

### BR-WEB-TEST-003 — E2E (Playwright, M2+)

- Login OIDC + seleção persona → painel.
- Cadastro condomínio (síndico) end-to-end.
- Onboarding morador completo.
- Connect Me síndico → empresa (envio + aceite + LGPD modal).
- Assembleia configuração + votação happy path.
- Stripe Checkout redirect + return.

---

## 15. Admin MS (role transversal — D-058) requisitos web-específicos (BR-WEB-ADMIN)

### BR-WEB-ADMIN-001 — Isolamento visual

Admin views dentro de `apps/cms` sob `/admin/*`:
- Sidebar extra "Admin MS" visível **somente** se `auth.user.roles.includes('admin_ms')`.
- Topbar com banner `"⚠️ Modo Admin"` vermelho suave quando dentro de `/admin/*`.
- Cookie separado opcional (backlog — cookie admin-specific com expiry mais curto).

### BR-WEB-ADMIN-002 — Auditoria

Toda ação admin gera audit trail backend; frontend mostra confirmação explicitamente. Double-confirm obrigatório em:
- Suspender tenant.
- Ban usuário.
- Refund Stripe.
- Broadcast global.
- ABAC override (grant temporário).

### BR-WEB-ADMIN-003 — MFA obrigatório (M2+)

Admin acessa `/admin/*` somente com MFA ativo (TOTP). Frontend valida `auth.user.mfa_active`; se false redireciona para `/account/security/mfa-setup`.

### BR-WEB-ADMIN-004 — 4-eyes (M3+)

Ações destrutivas (ban, delete tenant) exigem aprovação de 2 admins. UI: admin A faz request → entra em `pending_approvals` → admin B aprova → executa.

---

## 16. Navegador target (BR-WEB-BROWSER)

Suporte oficial:

| Browser | Versão mínima |
|---|---|
| Chrome | 110+ |
| Edge | 110+ |
| Firefox | 110+ |
| Safari | 16+ (iOS 16+ incluso) |
| Samsung Internet | 21+ |

Não suportado: IE (zero). Android WebView stock: MVP best-effort; M3+ teste explícito.

---

## 17. Feature flags (BR-WEB-FLAGS)

- **Runtime** via backend `GET /api/v1/feature-flags/me` → retorna lista de flags ativas por user/tenant.
- Frontend wrapper: `<Feature name="assembly-live">{children}</Feature>`.
- Exemplos: `assembly-live` (M3), `lms-access` (M3), `pwa-install`, `admin-broadcasts`, `ai-assistant` (M4+).
- Nunca armazenar chave sensível em flag (flags são públicas por natureza).

---

## 18. Matriz de requirements ↔ CWV/a11y/etc

Resumo:

| Requirement | Métrica objetiva | Como validar |
|---|---|---|
| BR-WEB-PERF-001 LCP < 2.5s mobile | Core Web Vital | `web-vitals` + CI lighthouse budget |
| BR-WEB-A11Y-001 WCAG 2.1 AA | Zero violations jest-axe | CI unit tests |
| BR-WEB-RESP-003 Touch 44px | Hit area | Playwright inspect |
| BR-WEB-PWA-001 Manifest | `manifest.json` válido | lighthouse pwa |
| BR-WEB-FORMS-002 Zod sync+async | validação | Vitest |
| BR-WEB-UPLOAD-001 Mux signed | fluxo | E2E Playwright M2 |
| BR-WEB-QUOTA-001 Backend-first | zero cálculo local | code review + test |
| BR-WEB-LGPD-001 Consent banner | modal visível | Playwright M2 |
| BR-WEB-TEST-001 Coverage thresholds | `vitest --coverage` | CI gate |
| BR-WEB-ADMIN-003 MFA admin | `auth.user.mfa_active` gate | Vitest + E2E |

---

## Links

- [[../../04-requirements/global]] — requirements globais GR-###/DR-###
- [[../../04-requirements/functional/_moc]] — funcionais por BC
- [[architecture]] — implementação
- [[patterns]] — patterns
- [[design-system]] — tokens/contraste
- [[security]] — CSP/XSS/CSRF/CORS
- [[ui-catalog]] — telas → requirements
- [[README]] — overview
- [[_moc]] — MOC do web
