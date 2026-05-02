---
title: auth — Web Requirements (SolidJS)
type: requirement
tags: [master-sindico, web, solidjs, auth, feature-req, oidc, zitadel]
project: master-sindico
stack: web
app: auth
port: 3000
milestone: m1
status: scaffold
created: 2026-04-24
---

# auth — Web Requirements

## Escopo e marco

**Marco**: M1 (Sprint 10, deadline 2026-05-08). Entregável principal: fluxo OIDC Zitadel+PKCE completo + 1-device policy lado cliente + callback + device-conflict screen. **Bloqueador M1** — sem auth nada autenticado roda.

App dedicado à porta `:3000` em subdomínio `auth.mastersindico.com.br` (prod) / `localhost:3000` (dev). Emite cookie httpOnly no domínio pai `.mastersindico.com.br` que cobre todos os outros 5 apps.

## Personas servidas

Todas as 7 personas (pre-auth). Roles Zitadel que podem completar sign-in:
- `sindico` (N1/N2/N3 sub-roles ABAC)
- `morador`
- `empresa` (base/plus/pro sub-roles)
- `agencia` (marketing)
- `admin_ms` (transversal D-058, MFA obrigatório M2+)
- `secretario` (assembleia, M3)

Pre-auth = anônimo. Pós-auth = redirect para `cms` ou `lms`/`forum`/`assembly` conforme deep-link original.

## Telas cobertas

| ID | Título | Rota | UI catalog |
|---|---|---|---|
| AUTH1 | Sign-in | `/_auth/sign-in` | [[../ui-catalog/onboarding/_moc]] |
| AUTH2 | Sign-up (step 1 persona) | `/_auth/sign-up` | [[../ui-catalog/onboarding/_moc]] |
| AUTH3 | Sign-up (step 2 dados) | `/_auth/sign-up/profile` | [[../ui-catalog/onboarding/_moc]] |
| AUTH4 | Callback OIDC | `/_auth/callback` | — (redirect-only) |
| AUTH5 | Device conflict | `/_auth/device-conflict` | [[../ui-catalog/onboarding/_moc]] |
| AUTH6 | Forgot password | `/_auth/forgot` | [[../ui-catalog/onboarding/_moc]] |
| AUTH7 | Logout confirmation | `/_auth/logout` | — |
| AUTH8 | Session expired | `/_auth/expired` | — |

## Funcionais (FR-WEB-AUTH-NNN)

### Fluxo OIDC + PKCE

- **FR-WEB-AUTH-001** — Botão "Entrar" gera `code_verifier` (random 128 chars base64url) + `code_challenge` (SHA-256 do verifier) e redireciona para `GET /auth/login?redirect_uri=...` do backend Go (que delega ao Zitadel).
- **FR-WEB-AUTH-002** — `code_verifier` armazenado em `sessionStorage` sob chave `pkce_verifier` (nunca `localStorage`; limpo após callback).
- **FR-WEB-AUTH-003** — `state` param anti-CSRF gerado random + validado no callback; mismatch → erro hard.
- **FR-WEB-AUTH-004** — Callback em `/_auth/callback` lê `?code=&state=` → POST `/auth/callback` backend com verifier → backend seta cookie httpOnly.
- **FR-WEB-AUTH-005** — Após callback OK, redirect para `returnTo` (default `https://app.mastersindico.com.br`) dentro de allowlist de domínios `*.mastersindico.com.br`.
- **FR-WEB-AUTH-006** — Erro OIDC (code inválido, state mismatch, backend 5xx) exibe tela com retry + link "Voltar ao login".

### 1-device policy

- **FR-WEB-AUTH-007** — Antes de `/auth/login` calcular device fingerprint SHA-256 de `canvas + webgl + UA + screen + timezone` e enviar via `X-Device-Fingerprint` header.
- **FR-WEB-AUTH-008** — Se backend retorna `409 device_conflict` com payload `{activeDevice: {name, lastSeen, location}}`, redirecionar para `AUTH5` (device-conflict).
- **FR-WEB-AUTH-009** — Tela device-conflict oferece 2 ações: "Deslogar outro dispositivo" (chama `POST /auth/devices/force-logout`) ou "Cancelar" (volta ao login).
- **FR-WEB-AUTH-010** — Fingerprint nunca armazena PII no cliente; apenas hash opaco enviado ao backend.

### Sign-up

- **FR-WEB-AUTH-011** — AUTH2 apresenta cards de persona (Síndico, Morador, Empresa, Agência) com preview do plano `trial` universal. Seleção → `sessionStorage.signupPersona`.
- **FR-WEB-AUTH-012** — AUTH3 coleta dados mínimos (nome, email, CPF/CNPJ, senha forte ≥12 chars com 1 upper+1 digit+1 symbol) validados Zod + async CPF check.
- **FR-WEB-AUTH-013** — Senha: indicador de força real-time (fraca/média/forte/muito forte) + toggle mostrar/ocultar com `aria-pressed`.
- **FR-WEB-AUTH-014** — Checkbox obrigatório "Aceito [Termos] e [LGPD]" com links abrindo `lp/termos` e `lp/privacidade` em nova aba.
- **FR-WEB-AUTH-015** — Submit chama `POST /auth/register` backend → se OK, redireciona para fluxo OIDC (auto-login).

### Forgot / Expired / Logout

- **FR-WEB-AUTH-016** — AUTH6 aceita email → `POST /auth/password-reset` → mensagem neutra "Se o email existir, enviamos instruções" (não revela existência, mitiga enumeration).
- **FR-WEB-AUTH-017** — AUTH7 exige confirmação click → `POST /auth/logout` → backend invalida cookie + revoga token Zitadel → redirect `lp/`.
- **FR-WEB-AUTH-018** — AUTH8 (sessão expirada) mostra mensagem + botão "Entrar novamente" preservando `returnTo=<rota-anterior>`.

### Cross-app session

- **FR-WEB-AUTH-019** — Após login OK cookie `__Host-ms_sess` (HttpOnly, Secure, SameSite=Lax, Domain=`.mastersindico.com.br`) válido em todos 5 apps.
- **FR-WEB-AUTH-020** — Apps consumidores (cms/forum/lms/assembly) não tocam cookie; apenas observam 401 → redirect para auth.
- **FR-WEB-AUTH-021** — `GET /auth/me` é endpoint de source-of-truth; cacheado 5min via TanStack Query `['auth','me']`.
- **FR-WEB-AUTH-022** — Logout em QUALQUER app → redirect global para `auth/logout` → cascateia em todos.

## Não-funcionais (NFR-WEB-AUTH)

- **Performance**: LCP < 1.5s desktop / 2.0s mobile (tela login leve, sem ilustração pesada). INP < 150ms (formulário ágil). CLS < 0.05 (sem layout shift em loading).
- **Acessibilidade**: WCAG 2.1 **AA** obrigatório; **AAA** em sign-in (tela crítica). Kobalte TextField + Form primitives. `autocomplete="username|current-password"` em inputs.
- **SEO**: auth NÃO indexável — `<meta name="robots" content="noindex,nofollow">`. Exceto AUTH1 que pode aparecer em sitemap de lp.
- **i18n**: pt-BR primário. EN preparado via `@solid-primitives/i18n` (M3). Mensagens de erro nunca em inglês para o usuário.
- **Segurança**: PKCE obrigatório (nunca implicit flow). CSP strict sem `unsafe-inline`. Senha nunca em log/Sentry breadcrumb.

## Rotas TanStack Router

```
src/routes/
├── __root.tsx                  # root: ColorModeProvider + QueryClientProvider + ThemeSync + ErrorBoundary
├── _auth.tsx                   # layout (logo + card centralizado)
├── _auth.sign-in.tsx           # AUTH1
├── _auth.sign-up.tsx           # AUTH2
├── _auth.sign-up.profile.tsx   # AUTH3
├── _auth.callback.tsx          # AUTH4 (beforeLoad: process OIDC)
├── _auth.device-conflict.tsx   # AUTH5
├── _auth.forgot.tsx            # AUTH6
├── _auth.logout.tsx            # AUTH7
├── _auth.expired.tsx           # AUTH8
└── _auth.index.tsx             # redirect para /_auth/sign-in
```

Auto code-splitting via Rsbuild TanStack plugin. `__root` emite `errorComponent` e `notFoundComponent`.

## Estado

- **Signals/Store**: `createSignal` para input values; `createStore` para form step state (AUTH2→AUTH3).
- **TanStack Solid Query**: `useQuery(['auth','me'])` com `staleTime: 5min`; `useMutation` para login/logout/signup.
- **TanStack Solid Form**: AUTH3 sign-up-profile (multi-field + Zod adapter + async CPF check).
- **Store global**: **não existe** — cookie é fonte; `auth/me` query é cache. Evitar duplicar estado de auth em signal local.

## Integração backend

- **Base URL**: `http://localhost:8000/api/v1/*` (dev) / `https://api.mastersindico.com.br/api/v1/*` (prod).
- **Auth**: Cookie httpOnly `__Host-ms_sess` (Domain=`.mastersindico.com.br`, SameSite=Lax, Secure em prod).

### Endpoints consumidos

| Método | Path | Payload | Retorno |
|---|---|---|---|
| GET | `/auth/login` | `?redirect_uri&state&code_challenge&code_challenge_method=S256` | 302 Zitadel |
| POST | `/auth/callback` | `{code, state, code_verifier}` | 200 `{user,roles}` + Set-Cookie |
| POST | `/auth/register` | `{persona,name,email,cpf,password,consent}` | 201 `{userId}` |
| POST | `/auth/password-reset` | `{email}` | 204 (sempre) |
| GET | `/auth/me` | — | `{userId,roles,persona,plan,deviceId}` |
| POST | `/auth/logout` | — | 204 + Clear-Cookie |
| POST | `/auth/devices/force-logout` | `{deviceId}` | 204 |
| GET | `/auth/devices/active` | — | `[{id,name,lastSeen,ua,ip}]` |

Todos retornam erros tipados Zod `{code, message, details}`. Frontend mapeia `code → copy pt-BR humano`.

## Design system

- **UnoCSS + Kobalte + CVA** via `packages/ui`.
- **Tokens**: Manrope body + Michroma logo "Master Síndico". Paleta OKLCH: `navy #071A33` bg, `gold #C69C3F` CTA, `accent-gold-light` ring focus.
- **Componentes esperados**: `<Button variant="primary|ghost" size="lg">`, `<TextField>`, `<PasswordField>` (toggle show/hide), `<Checkbox>`, `<Alert variant="error|success|warning">`, `<Spinner>`, `<Card>`, `<Link>`.
- **Atoms faltantes M1**: `PasswordField`, `Alert`, `Checkbox` (apenas `Button` existe hoje).

## Segurança

- **CSP strict**: `default-src 'self'; connect-src 'self' https://*.mastersindico.com.br https://*.zitadel.cloud; script-src 'self' 'nonce-{random}'; frame-ancestors 'none';`.
- **XSS**: SolidJS auto-escape + zero `innerHTML`. Mensagens de erro nunca incluem HTML do backend.
- **CSRF**: SameSite=Lax + CORS allowlist explícito `app.mastersindico.com.br,auth.mastersindico.com.br,...` sem `*`.
- **Rate limit**: client mostra `Retry-After` do 429 como countdown visível. Backend enforces 10 tentativas/5min por IP+email.
- **Storage**: senha jamais persiste. `code_verifier` em `sessionStorage` (limpo pós-callback). Cookie httpOnly — JS **nunca** acessa.
- **Device fingerprint**: hash SHA-256 opaco, não PII.
- **OIDC**: validar `iss`, `aud`, `exp`, `nonce` no backend (frontend não toca token).

## Testes

- **Vitest (unit)** — thresholds **95%** (camada crítica auth): PKCE helpers, device fingerprint hash, Zod schemas, copy mapping.
- **@solidjs/testing-library** — `sign-in.test.tsx`, `sign-up-profile.test.tsx` (render, interação teclado, aria-invalid).
- **MSW** — mocka `/auth/*` endpoints (login success/fail, device-conflict 409, register 201/409).
- **Playwright (E2E M3)** — fluxo completo sign-in → redirect → logout cross-app.
- **jest-axe** — zero violations em todas 8 telas.

## Status atual (real) — AUDIT A-FASE10-DEV-005

🔴 **Bloqueador M1**. `apps/auth/src/routes/__root.tsx` e `index.tsx` são stubs `<div>Hello auth!</div>`. `AuthProvider`/`QueryClientProvider` comentados. Nenhuma das 8 telas implementadas. Zero testes. `@tanstack/solid-query` e `@tanstack/solid-form` não instalados em `bun.lock`. `PasswordField`/`Alert`/`Checkbox` ausentes em `packages/ui`.

## Gaps/decisões abertas

- **G1**: persistir `returnTo` via `state` param (assinado backend) vs `sessionStorage` — decidir na Sprint 10.
- **G2**: 1-device UX quando usuário marca "lembrar" — backend já suporta 2 devices para plano Pro?
- **G3**: MFA TOTP em sign-in para admin MS (D-058) — entra em M2 ou M1?
- **G4**: biblioteca i18n M1 manual vs `@solid-primitives/i18n` — alinhar com cms.

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../architecture]]
- [[../design-system]]
- [[../security]]
- [[../requirements]]
- [[../ui-catalog/onboarding/_moc]]
- [[../../../08-security/BEYOND_CORP]]
- [[../../../00-product/personas]]
