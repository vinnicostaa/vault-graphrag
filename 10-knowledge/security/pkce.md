---
title: PKCE — Proof Key for Code Exchange (RFC 7636)
type: concept
tags:
  - security
  - oauth
  - pkce
  - authorization-code
  - public-client
doc-oficial: https://datatracker.ietf.org/doc/rfc7636/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - PKCE
  - Proof Key for Code Exchange
---

# PKCE — Proof Key for Code Exchange

**O que é**: extensão de OAuth 2.0 (RFC 7636, 2015) que protege o fluxo **Authorization Code** contra interceptação do authorization code. Criado originalmente para mobile; em OAuth 2.1 + RFC 9700 BCP virou **obrigatório para todos os clients** (confidential e public).

**Pronúncia**: "pixie".

> Sem PKCE, atacante que intercepta o `code` em redirect (via URL handler malicioso, proxy, log) consegue trocar por token. Com PKCE, precisa também do `code_verifier` — que só o client original conhece.

## Problema que resolve

```
1. Client legítimo: GET /authorize?response_type=code&redirect_uri=myapp://cb
2. AS: redirect myapp://cb?code=ABC123
3. App malicioso registra handler myapp:// primeiro → intercepta code
4. App malicioso: POST /token code=ABC123 → recebe access_token ❌
```

Em browser, o problema é semelhante (referrer leak, logs de proxy, XSS).

## Como funciona

```
Client gera: code_verifier (32-96 bytes random, URL-safe)
             code_challenge = BASE64URL(SHA-256(code_verifier))

1. Client → AS: GET /authorize?
                 response_type=code
                 &code_challenge=<challenge>
                 &code_challenge_method=S256
                 &client_id=...&redirect_uri=...&state=...

2. AS armazena: (code → code_challenge, code_challenge_method)

3. AS → Client: redirect ...?code=<auth-code>&state=...

4. Client → AS: POST /token
                 grant_type=authorization_code
                 &code=<auth-code>
                 &code_verifier=<verifier>           ← prova de posse

5. AS valida: BASE64URL(SHA-256(code_verifier)) == code_challenge?
              Sim → emite tokens. Não → 400 invalid_grant.
```

Atacante que intercepta o `code` **não tem** o `code_verifier` → não consegue trocar.

## `S256` vs `plain`

- **`S256`** (SHA-256): **obrigatório**; todos os clients e AS modernos suportam.
- **`plain`**: `code_challenge == code_verifier` — sem hash. **Fortemente desencorajado** (SHOULD NOT) por RFC 7636 §7.2 e RFC 9700 BCP — não é "deprecated" formalmente, mas clientes **DEVEM** usar `S256` exceto onde comprovadamente impossível (extremamente raro).

## Requisitos do `code_verifier`

- **Alfabeto**: `[A-Z] / [a-z] / [0-9] / "-" / "." / "_" / "~"` (unreserved).
- **Comprimento**: 43-128 chars (32-96 bytes pré-base64).
- **Entropia**: cryptographically random (RFC 7636 §4.1 — "cryptographically random").

```ts
// Geração canônica (Node, Web Crypto)
function generateVerifier(): string {
  const arr = new Uint8Array(32)
  crypto.getRandomValues(arr)
  return base64urlEncode(arr)
}

async function generateChallenge(verifier: string): Promise<string> {
  const hash = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(verifier))
  return base64urlEncode(new Uint8Array(hash))
}
```

## Onde guardar o `code_verifier` durante o flow

**Problema**: entre redirect para AS e callback ao client, o `code_verifier` precisa persistir.

**Opções**:
- **sessionStorage** (SPA): ok para single-tab. Perde em refresh de tab.
- **Cookie httpOnly** (se client tem backend): preferível. Ver [[cookies]].
- **Memória do mobile/native app**: nativo via Keychain/Keystore. Ver [[credential-storage]].

**Nunca** guardar em localStorage — sobrevive a logout/fechamento; superfície de ataque XSS maior. Ver [[credential-storage]].

## PKCE + confidential client

OAuth 2.1 exige PKCE **também** para confidential clients (com `client_secret`). Não é ou-ou:
- Public client: PKCE é a **única** defesa.
- Confidential client: PKCE é **defesa adicional** ao `client_secret`.

## Anti-patterns

1. **`code_challenge_method=plain`** — RFC 7636 §7.2 "SHOULD NOT be used"; usar só em legado que comprovadamente não suporta SHA-256.
2. **`code_verifier` com entropia fraca** (timestamp, UUID v1) — use random real.
3. **Reutilizar `code_verifier`** entre fluxos — gerar novo a cada `/authorize`.
4. **Armazenar `code_verifier` em localStorage** — persiste entre sessões, expõe a XSS.
5. **`code_verifier` em URL/log** — igual a access token: sensível.
6. **Pular PKCE "porque é confidential client"** — OAuth 2.1 exige.
7. **Implementar validação custom em AS próprio** — use lib testada (jose, oauth2-server).

## SDKs que fazem PKCE automaticamente

| Lib | Linguagem | Detalhe |
|---|---|---|
| `openid-client` | Node | PKCE habilitado por default |
| `@auth/core` | JS | Handling interno |
| `msal-browser` | JS (Microsoft) | PKCE obrigatório |
| `zitadel/oidc` | Go | Gerado automaticamente |
| `authlib` | Python | `code_challenge_method='S256'` padrão |
| `AppAuth-iOS` / `AppAuth-Android` | Mobile | Gerado internamente |

## Grandes empresas

- **Google, Microsoft, Apple, GitHub**: exigem PKCE em novos clients mobile/SPA.
- **Auth0, Okta, Zitadel**: exigem PKCE em public clients; avisam em confidential.
- **Cloudflare Access**: usa OIDC do IdP configurado — PKCE delegado ao IdP (Zitadel/Google/Okta).

## Relações

- **Extensão de**: [[oauth-2]] (ativa no grant Authorization Code)
- **Combina com**: [[oidc]] (nonce + PKCE = proteção completa do code flow)
- **Alternativa/complemento**: DPoP (RFC 9449 — ver [[jwt#Sender-constrained-tokens]]) para sender-constrained tokens
- **Storage do verifier**: [[credential-storage]], [[cookies]]

## Fontes

- [RFC 7636 — PKCE](https://datatracker.ietf.org/doc/rfc7636/) (consultada 2026-04-24)
- [RFC 9700 §4.5 — PKCE BCP](https://datatracker.ietf.org/doc/rfc9700/) (consultada 2026-04-24)
- [OAuth 2.1 §4.1 — PKCE required](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) (consultada 2026-04-24)
- [Auth0 — Authorization Code + PKCE flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[oauth-2]]
- [[oidc]]
- [[jwt]]
- [[credential-storage]]
- [[csrf-defense]]
- [[owasp-top-10]] — A07 (proteção do code flow)
