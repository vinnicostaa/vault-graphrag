---
title: Web — Security posture
type: spec
tags: [subproject, web, master-sindico, v2, security, xss, csp, csrf, cors, idor, lgpd, cookie, beyondcorp, admin-abac]
source: Development/web/CLAUDE.md §Segurança (linhas 272-302) + AGENTS.md §Segurança + 08-security/BEYOND_CORP.md
created: 2026-04-23
updated: 2026-04-23
---

# Web — Security posture

Defesa browser-side do sub-projeto `web/`. **Never trust the frontend** — decisão de segurança sempre vive no backend. Este documento cobre os vetores específicos do browser: cookie httpOnly, CSP strict, XSS prevention, CSRF via SameSite + double-submit, CORS allowlist, IDOR via 404, isolamento admin ([[../admin-privilegios|D-134 — app `apps/admin` dedicada atrás de Cloudflare Access+MFA]]), LGPD UI.

Fontes:
- `Development/web/CLAUDE.md` §Segurança frontend (linhas 272-302).
- `Development/web/AGENTS.md` §Segurança + §Zero Trust + §1 device.
- [[../../08-security/BEYOND_CORP]] — zero-trust posture.
- OWASP Top 10 2021 + OWASP ASVS 4.0.3.

---

## 1. Cookie httpOnly — armazenamento de sessão

### 1.1 Característica emitido pelo backend

Backend seta o cookie `session` com:
- `HttpOnly: true` — JS **não acessa**.
- `Domain: .mastersindico.com.br` — compartilhado entre `auth.`, `app.`, `academia.`, `vizinhanca.`, `assembleia.`, apex.
- `Secure: true` (prod obrigatório) — HTTPS only.
- `SameSite: Lax` — previne CSRF cross-site no caso default.
- `Path: /`.
- `Max-Age`: alinhado à expiração da sessão (1h access, refresh transparente no backend).

### 1.2 Proibições frontend (CLAUDE.md linhas 278-286)

- ❌ **Nunca** armazenar token/JWT em `localStorage`, `sessionStorage`, `IndexedDB`, ou cookies JS-acessíveis.
- ❌ **Nunca** fazer `document.cookie` read de sessão.
- ❌ **Nunca** serializar `AuthUser` com PII em store persistido (persistedState libs).
- ❌ **Nunca** escrever em cookie de sessão via JS.

### 1.3 Dev local

Em `localhost`, browsers **exigem** cookie **sem** `Domain` attribute. Backend ajusta via env `COOKIE_DOMAIN` (empty em dev). Docs: CLAUDE.md linha 399.

### 1.4 Logout

```ts
async function logout() {
  await api.post('/api/v1/auth/logout');   // backend end_session Zitadel + clear cookie
  queryClient.clear();                      // limpa cache TanStack Query
  sessionStore.reset();                     // limpa store local
  window.location.href = `${AUTH_URL}/sign-in`;
}
```

Esquecer `queryClient.clear()` = PII do user anterior visível para o próximo em caches → vazamento.

---

## 2. Content Security Policy (CSP)

### 2.1 Header (servido pelo backend ou Railway CDN)

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{random}' https://cdn.mux.com https://browser.sentry-cdn.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: blob: https://image.mux.com https://*.stripe.com;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.mastersindico.com.br wss://api.mastersindico.com.br https://*.mux.com https://o*.ingest.sentry.io https://*.stripe.com;
  media-src 'self' blob: https://stream.mux.com;
  frame-src https://*.stripe.com https://checkout.stripe.com;
  frame-ancestors 'none';
  form-action 'self' https://*.stripe.com https://checkout.stripe.com;
  base-uri 'self';
  upgrade-insecure-requests;
  report-uri /api/v1/csp/report;
```

### 2.2 Regras

- **Nonce** por request para `<script>` inline — gerado backend, injetado no HTML, validado pelo browser.
- **`style-src 'unsafe-inline'`** — UnoCSS gera inline styles em runtime; tolerado mas avaliar `nonce` estendido M2+.
- **`frame-ancestors 'none'`** — previne clickjacking (nunca renderizar Master Síndico em iframe de terceiros).
- **`connect-src`** — lista explícita dos endpoints permitidos (API backend, WS, Mux, Sentry, Stripe).
- **`media-src`** — Mux HLS signed URLs.
- **`frame-src`** — apenas Stripe Checkout (iframe de pagamento).
- **Reporting**: `report-uri` → backend agrega violações; alerta se pattern suspeito.

### 2.3 Ambiente

- **Dev**: CSP mais relaxada (permite `localhost:*`, devtools). Biome + plugin `security-guidance` bloqueia `innerHTML` / `eval` em código mesmo em dev.
- **Prod**: CSP estrita, zero `unsafe-eval`.

---

## 3. XSS Prevention

### 3.1 SolidJS auto-escape

Por default, `{variable}` em JSX **escapa HTML**:

```tsx
const userName = '<script>alert(1)</script>';
return <div>{userName}</div>;   // renderiza literal: &lt;script&gt;...
```

### 3.2 Proibições duras (CLAUDE.md linha 280)

- ❌ `innerHTML =` / `outerHTML =` em qualquer código.
- ❌ `dangerouslySetInnerHTML` (é React; SolidJS usa `innerHTML` prop — **igualmente proibido**).
- ❌ `eval`, `new Function`, `setTimeout(string, ...)`, `setInterval(string, ...)`.
- ❌ `document.write`.
- ❌ Fazer `fetch` a URL vinda de user input (SSRF indireto).

Plugin Claude Code `security-guidance` bloqueia via pre-tool hook (CLAUDE.md linha 328).

### 3.3 Conteúdo rich-text permitido (markdown/HTML)

Para renderização de markdown (ex: descrição rich em atividade timeline, comunicado):

```ts
import { marked } from 'marked';
import DOMPurify from 'isomorphic-dompurify';

function renderMarkdown(source: string): string {
  const rawHtml = marked.parse(source);
  return DOMPurify.sanitize(rawHtml, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'ul', 'ol', 'li', 'a', 'blockquote', 'h1','h2','h3','h4'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
    ALLOWED_URI_REGEXP: /^(https?:|mailto:|#|\/)/,
  });
}

// uso
<div innerHTML={renderMarkdown(content)} />   // ✅ OK após DOMPurify
```

**Regra**: `innerHTML` só permitido após `DOMPurify.sanitize()`. Linter-rule custom sugerido FE-backlog M1.

### 3.4 URL user-provided em `<a href>`

Validar protocol:

```ts
function safeHref(url: string): string | undefined {
  if (!/^https?:\/\//.test(url)) return undefined;   // bloqueia javascript:, data:, vbscript:
  return url;
}

<a href={safeHref(link.url)} target="_blank" rel="noopener noreferrer">
```

**Sempre** `rel="noopener noreferrer"` em `target="_blank"` (previne tab-nabbing).

---

## 4. CSRF (Cross-Site Request Forgery)

### 4.1 Defesas em camadas

1. **`SameSite=Lax` no cookie** (primary defense) — cookies de sessão não viajam em requests cross-origin de outros sites (exceto GET top-level).
2. **CORS allowlist restrito** — backend só aceita Origin `*.mastersindico.com.br` + `localhost:30NN` dev.
3. **Double-submit cookie pattern** (reforço para mutations):
   - Backend emite cookie adicional `csrf_token` (não-httpOnly, SameSite=Lax) com valor randômico.
   - Frontend lê via JS e envia em header `X-CSRF-Token` em toda mutation (POST/PUT/PATCH/DELETE).
   - Backend compara cookie value == header value; mismatch = 403.

### 4.2 Implementação frontend

```ts
// src/core/api/http.ts
api.interceptors.request.use((cfg) => {
  if (['post','put','patch','delete'].includes(cfg.method?.toLowerCase() ?? '')) {
    const csrf = readCookie('csrf_token');
    if (csrf) cfg.headers['X-CSRF-Token'] = csrf;
  }
  return cfg;
});
```

### 4.3 Exceções

- Requests público (`GET /api/v1/health`, LP) não precisam CSRF token.
- Webhooks não chegam no frontend (backend-only).

---

## 5. CORS (Cross-Origin Resource Sharing)

### 5.1 Allowlist backend

```
Access-Control-Allow-Origin:
  https://auth.mastersindico.com.br,
  https://app.mastersindico.com.br,
  https://academia.mastersindico.com.br,
  https://vizinhanca.mastersindico.com.br,
  https://assembleia.mastersindico.com.br,
  https://mastersindico.com.br,
  http://localhost:3000, http://localhost:3001, http://localhost:3002,
  http://localhost:3003, http://localhost:3004, http://localhost:3005

Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, X-CSRF-Token, X-Request-Id, X-Device-Fingerprint
Access-Control-Max-Age: 86400
```

### 5.2 Proibições

- ❌ `Access-Control-Allow-Origin: *` **jamais** com `credentials: true`. Config backend validation bloqueia (Viper `Config.Validate()` — ver [[../backend/security]]).
- ❌ Wildcards em subdomínios (`*.mastersindico.com.br`) — listar explicitamente.

---

## 6. Clickjacking

### 6.1 `frame-ancestors 'none'` no CSP

Cobre a maioria dos browsers modernos.

### 6.2 Legacy: `X-Frame-Options: DENY`

Para Safari/IE antigos ainda usarem — defesa em profundidade.

### 6.3 Exceção: checkout Stripe

Iframe Stripe roda **dentro** da página MS (não o contrário). Permitido via `frame-src https://checkout.stripe.com`.

---

## 7. HTTPS-only + HSTS

### 7.1 Produção

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

2 anos + subdomains + preload list submit.

### 7.2 `upgrade-insecure-requests` em CSP

Browser reescreve `http://` → `https://` automaticamente em recursos referenciados.

### 7.3 Dev

`localhost` em HTTP tolerado; Railway dev-preview em HTTPS (auto-issued cert).

---

## 8. Dependências — supply chain

### 8.1 Audit automatizado

- `bun audit` em CI (se/quando suportado); fallback `npm audit` via `package-lock.json` gerado.
- **Dependabot** GitHub semanal.
- **Snyk** ou **Socket.dev** (M2+) para deep scan de comportamento.

### 8.2 Regras

- **Lockfile commitado** (`bun.lock`) — reprodutibilidade.
- **Zero `postinstall` scripts** em deps novas sem review.
- **Updates major** entram por PR separada com changelog review.
- **Block list**: nunca `crypto-js` (obsoleto) — use Web Crypto nativo. Nunca `axios@<1.6` (CVE). Nunca `lodash` (tree-shakable só individualmente).

### 8.3 Subresource Integrity (SRI)

Para scripts externos (Mux CDN, Sentry CDN):

```html
<script
  src="https://cdn.mux.com/player/1.2.3/player.js"
  integrity="sha384-..."
  crossorigin="anonymous"
></script>
```

Hash validado a cada deploy.

---

## 9. IDOR (Insecure Direct Object Reference)

### 9.1 Regra raiz

**Backend retorna `404 Not Found` em vez de `403 Forbidden`** quando usuário tenta acessar recurso de outro tenant que não é seu. Isso previne enumeration (atacante descobrir IDs válidos).

Exemplo:
- `GET /api/v1/condominios/ABC12345` → tenant Y não é membro → **404** (não 403).
- `GET /api/v1/condominios/XYZ99999` (não existe) → **404** igual.

Atacante não distingue "existe mas não tenho acesso" vs "não existe".

### 9.2 UI frontend

Na tela, **toda** rota `/$param` trata 404 uniformemente:

```tsx
// src/routes/_authenticated/condominios/$condominiumId.tsx
export const Route = createFileRoute('/_authenticated/condominios/$condominiumId')({
  loader: async ({ params }) => {
    try {
      return await get(`/api/v1/condominios/${params.condominiumId}`, CondominiumSchema);
    } catch (err) {
      if (axios.isAxiosError(err) && err.response?.status === 404) {
        throw notFound();   // redirect para 404 page genérica
      }
      throw err;
    }
  },
});
```

Mensagem: **"Este condomínio não existe ou foi removido."** (nunca "Você não tem acesso").

### 9.3 IDs opacos

- **ULIDv7 / UUIDv7** — não sequenciais, não preditíveis.
- Nunca expor IDs internos de banco (`id: 123` → `id: "01HXYZ..."`).
- Para condomínio: ID alfanumérico 8 chars visível (feature de produto — S3 tela), mas randomness alta + rate-limit na rota `/api/v1/condominios/lookup?code=XXX` (anti-scraping).

---

## 10. Admin isolation — D-134 app `apps/admin` dedicada (revoga D-072 transversal)

### 10.1 Princípio

Admin MS **não tem app separada**. Views admin ficam dentro das apps existentes (principalmente `cms`, sob `/admin/*`), **gated por ABAC**.

### 10.2 Gates

**Layer 1 — Route guard (UI only)**:

```tsx
// src/routes/_authenticated/admin/_layout.tsx
export const Route = createFileRoute('/_authenticated/admin')({
  beforeLoad: async ({ context }) => {
    const roles = context.auth.user.roles;
    if (!roles.includes('admin_ms') && !roles.includes('super_admin')) {
      throw redirect({ to: '/' });   // redirect silent para raiz (não 403 revelador)
    }
  },
});
```

**Layer 2 — Backend ABAC**: toda rota `/api/v1/admin/*` exige `role in [admin_ms, super_admin, moderator, support, financial]` + tenant-specific constraints. Backend é a verdade.

**Layer 3 — Sub-role matrix** (impl backend ABAC policies; frontend reflete):

| Sub-role | Moderação | Suspender tenant | Broadcast | Financeiro | Audit read |
|---|---|---|---|---|---|
| `super_admin` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `moderator` | ✅ | ✅ | ❌ | ❌ | ✅ |
| `support` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `financial` | ❌ | ❌ | ❌ | ✅ | ✅ |

### 10.3 UI diferenciada (Admin Mode)

Quando user em rota `/admin/*`:
- Topbar com banner **"⚠ Modo Admin · {sub-role}"** (cor `warning` suave).
- Sidebar item "Admin MS" destaca com ícone cadeado.
- Toda ação destrutiva exige double-confirm.
- Audit trail registra todo view/action (backend faz, frontend só mostra confirmação).

### 10.4 MFA obrigatório (M2+)

Admin só acessa `/admin/*` com MFA ativo (TOTP). Frontend checa `auth.user.mfa_active`; se false → redirect `/account/security/mfa-setup`.

### 10.5 4-eyes policy (M3+, D-054)

Ações destrutivas exigem aprovação de 2 admins:

```
Admin A: "Suspender tenant X" → cria PendingApproval → status pending
Admin B (diferente): vê em /admin/approvals → aprova → ação executa
```

Frontend UI: lista `/admin/pending-approvals` + action sheet.

### 10.6 Ver também

[[../admin-privilegios]] — especificação completa D-058 (stub Fase 8; expand M2).

---

## 11. Secret management frontend

### 11.1 Regra

**Apenas env vars com prefixo `PUBLIC_` ou `VITE_PUBLIC_` podem chegar ao frontend**. Tudo mais fica no backend.

Público permitido:
- `PUBLIC_API_URL` — URL da API (conhecido publicamente)
- `PUBLIC_WS_URL` — WebSocket endpoint
- `PUBLIC_AUTH_URL` — `https://auth.mastersindico.com.br`
- `PUBLIC_SENTRY_DSN` — DSN é público por design
- `PUBLIC_STRIPE_PUBLISHABLE_KEY` — key pública Stripe (começam `pk_`)
- `PUBLIC_MUX_ENV_KEY` — env key pública Mux
- `PUBLIC_APP_VERSION` — hash do commit
- `PUBLIC_ENV` — `dev | staging | prod`

**Proibido em frontend**:
- ❌ `STRIPE_SECRET_KEY` (sk_*)
- ❌ `MUX_SECRET_KEY`
- ❌ `ZITADEL_CLIENT_SECRET`
- ❌ Tokens de DB/Redis
- ❌ Qualquer chave de provider backend

### 11.2 Validação CI

Script em `.github/workflows/web.yml`:

```bash
# Falha build se encontrar key suspeita sem prefix PUBLIC_
grep -rE "import\.meta\.env\.(?!PUBLIC_)[A-Z_]+" apps/ packages/ && exit 1 || exit 0
```

---

## 12. Device fingerprint

### 12.1 Coletor

```ts
// src/core/security/fingerprint.ts
export async function computeFingerprint(): Promise<string> {
  const parts = [
    navigator.userAgent,
    navigator.language,
    screen.width + 'x' + screen.height,
    new Date().getTimezoneOffset(),
    navigator.hardwareConcurrency,
    await canvasHash(),        // pinta padrão; lê dataURL; SHA-256
    await webglHash(),         // vendor + renderer
  ];
  const joined = parts.join('|');
  const hash = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(joined));
  return bytesToHex(new Uint8Array(hash));
}
```

### 12.2 Envio

Header em todas requests: `X-Device-Fingerprint: <sha256>`. Backend consolida para IR-011 (1-device policy).

### 12.3 Limitações (honestidade)

- Não é anti-bot definitivo; bypassa com headless determinado.
- Fingerprinting agressivo (audio context, fontes instaladas) **proibido LGPD** — limitar a dados básicos do UA.
- Trocas legítimas de fingerprint (troca de dispositivo, update SO) devem ter fluxo de re-auth suave.

---

## 13. Rate limit — UI feedback

Backend rate-limit (token bucket Redis) retorna 429 com header `Retry-After: <seconds>`. Frontend:

```tsx
if (response.status === 429) {
  const retryAfter = response.headers['retry-after'];
  toast.warning(`Muitas tentativas. Aguarde ${retryAfter}s.`);
  disableButton(retryAfter);   // countdown visual
}
```

Respeitar `Retry-After` — não fazer retry automático abaixo desse tempo.

---

## 14. LGPD no frontend (defesa em profundidade)

### 14.1 Consentimento cookies

Banner em `lp` primeira visita:
- Granular: necessários (sempre on) · analytics (opt-in) · marketing (opt-in).
- Registra escolha em backend `POST /api/v1/lgpd/consent` + versão do termo.
- Re-prompt quando versão do termo muda (signal backend).

### 14.2 Connect Me modal LGPD (ver [[requirements#13]])

Antes de revelar contato:

```
⚠ Este contato está protegido por LGPD.
Ao prosseguir, você declara:
- Finalidade: contratar serviço para o condomínio "Residencial X"
- Dados compartilhados: nome, email, telefone de <Empresa Y>
- Retenção: 5 anos (audit trail LGPD)
- Este registro é auditável pelo DPO

[ ] Confirmo que li e concordo
[Cancelar]  [Revelar contato]
```

### 14.3 PII em logs/telemetria

- **Sentry `beforeSend`**: scrub `email, cpf, cnpj, phone, address, password`.
- **Web Vitals**: sem user identifier (nem user_id). Agrega por route + persona + device.
- **Console logs**: `maskedInfo` helper em vez de `console.log` direto.
- **URL query params**: nunca `?email=...` ou `?cpf=...` em analytics.

### 14.4 Portabilidade e esquecimento (DSAR)

`/conta/dados-pessoais`:
- **Exportar meus dados**: backend gera JSON completo; SPA faz download (signed URL TTL 5min).
- **Excluir minha conta**: double-confirm + backend soft-delete + anonymization jobs assíncronos.

### 14.5 Retenção

- Cookie session: max-age 1h access token; 30d refresh.
- `localStorage`: apenas preferências (theme, idioma) — **nunca PII**.
- IndexedDB: não usar no MVP.

---

## 15. OWASP Top 10 (2021) — mapeamento

| OWASP | Risco | Mitigação web |
|---|---|---|
| A01 Broken Access Control | Acesso a recurso alheio | ABAC backend + 404 IDOR + route guards UI (só UX) |
| A02 Cryptographic Failures | PII em claro | HTTPS + HSTS + cookie Secure + sem tokens em storage |
| A03 Injection | XSS | SolidJS auto-escape + proibir innerHTML + DOMPurify em markdown |
| A04 Insecure Design | Falha arquitetural | BeyondCorp + threat model por BC |
| A05 Security Misconfiguration | CSP fraca, CORS `*` | CSP strict + CORS allowlist + config validation backend |
| A06 Vulnerable Components | Dep outdated | bun audit + Dependabot + lockfile |
| A07 ID & Auth Failures | Session fixation | Zitadel OIDC + 1-device + session rotation |
| A08 Software & Data Integrity | Supply chain | SRI + lockfile + Dependabot + Snyk (M2+) |
| A09 Logging Failures | PII logs | Sentry scrubbing + masked console + LGPD logs |
| A10 SSRF | Fetch arbitrário | Validar protocol em `<a href>` + CORS `connect-src` strict |

---

## 16. Security checklist headers (prod)

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Content-Security-Policy: (ver §2)
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(self "https://app.mastersindico.com.br"), geolocation=(self "https://vizinhanca.mastersindico.com.br")
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-site
Cross-Origin-Embedder-Policy: credentialless
```

Validar em cada deploy: [securityheaders.com](https://securityheaders.com) meta ≥ A+.

---

## 17. Auth flow (Zitadel OIDC + PKCE) — security review

1. User → `app.mastersindico.com.br/...` sem cookie → redirect `auth.mastersindico.com.br/sign-in`.
2. `auth` app inicia PKCE:
   - `code_verifier` = random 128 bytes Base64-URL (apenas no backend — não confiar no frontend para gerar).
   - `code_challenge` = SHA-256(verifier) Base64-URL.
   - `state` random — validação retorno.
3. Redirect Zitadel authz endpoint.
4. Zitadel autentica → callback `/api/v1/auth/callback?code=X&state=Y`.
5. Backend:
   - Valida `state` (anti-CSRF).
   - Troca `code` + `code_verifier` por tokens.
   - Valida `id_token` (JWK Zitadel).
   - Checa 1-device: se user já tem sessão ativa em outro device → invalida anterior, mas só se `force_login=true` (usuário confirma) — senão retorna 403 DEVICE_MISMATCH.
   - Seta cookie httpOnly.
   - Redirect de volta para SPA de origem (URL guardada no `state`).
6. SPA chama `GET /api/v1/auth/me` → popula `session` store.

### 17.1 Riscos mitigados

- **CSRF em OIDC** → `state` validado.
- **Code injection** → PKCE.
- **Token leak** → tokens nunca no frontend; apenas cookie httpOnly.
- **Session fixation** → backend rotaciona cookie a cada login.

---

## 18. Storage strategy

| Tipo de dado | Onde | Justificativa |
|---|---|---|
| Tokens (access, refresh) | ❌ Lugar nenhum frontend — cookie httpOnly backend | Único caminho XSS-safe |
| User preferences (theme, locale) | `localStorage` (`ms:pref:theme`) | Baixo risco, não PII |
| CSRF token (double-submit) | cookie não-httpOnly **+** header `X-CSRF-Token` | Padrão OWASP |
| Query cache | memória (TanStack Query) | Limpa em logout |
| Form drafts sensíveis | backend Redis via auto-save | PII não pode ir em localStorage |
| Upload progress | memória (hook) | Efêmero |
| LGPD consent | backend (single source); cache memória | Auditável |

---

## 19. Ferramentas auxiliares

- **Biome rules**: `noDangerouslySetInnerHtml`, `noGlobalEval` (custom).
- **Plugin Claude `security-guidance`**: hook bloqueia `innerHTML`, `eval`, hardcoded keys (CLAUDE.md linha 328).
- **Plugin Claude `code-review`**: analisa PR com scoring segurança.
- **OWASP ZAP baseline** em CI M3+ (passive scan diário).
- **Playwright + `playwright-security`** para XSS/CSRF E2E probes (M3+).

---

## 20. Resumo: Never fazer / Sempre fazer

### 20.1 Nunca fazer (CLAUDE.md linhas 278-286)

- ❌ Token em `localStorage/sessionStorage/IndexedDB` ou cookie JS-acessível.
- ❌ `dangerouslySetInnerHTML` / `innerHTML` sem `DOMPurify`.
- ❌ Validar autorização no frontend para decisão real (só UX).
- ❌ Calcular valor monetário/quota/prazo — frontend exibe backend.
- ❌ Confiar em query params/hash para decisão de segurança.
- ❌ Hardcode secret/key/token em código.
- ❌ Fetch a domínio arbitrário via input (SSRF indireto).
- ❌ Rodar `eval(stringUserProvided)` ou equivalente.

### 20.2 Sempre fazer (CLAUDE.md linhas 288-294)

- ✅ `credentials: "include"` em toda chamada API.
- ✅ Validar Zod toda entrada de form (UX).
- ✅ Validar Zod toda resposta backend (defesa em profundidade).
- ✅ Escapar strings usuário antes de renderizar (SolidJS auto-escapa `{var}`).
- ✅ CSP strict via headers Railway.
- ✅ Mask PII em logs console (`masked(cpf)`).
- ✅ `queryClient.clear()` no logout.
- ✅ Header `X-CSRF-Token` em mutations.
- ✅ 404 uniforme em IDOR.
- ✅ Double-confirm em ações destrutivas (admin + síndico/empresa).

---

## Links

- [[README]] — overview
- [[architecture]] — entry, auth flow, API client
- [[patterns]] — anti-patterns XSS, Code Calisthenics
- [[requirements]] — LGPD + MFA + CWV
- [[ui-catalog]] — telas × gates de acesso
- [[../admin-privilegios]] — D-058 role transversal
- [[../../08-security/BEYOND_CORP]] — zero-trust posture
- [[../../08-security/threat-model]] — STRIDE por BC
- [[../backend/security]] — backend posture (espelho server-side)
- OWASP Top 10 · OWASP ASVS 4.0.3 · W3C CSP Level 3
