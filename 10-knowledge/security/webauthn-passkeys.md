---
title: WebAuthn & Passkeys — FIDO2 phishing-resistant auth
type: concept
tags:
  - security
  - webauthn
  - passkey
  - fido2
  - phishing-resistant
  - mfa
doc-oficial: https://www.w3.org/TR/webauthn-3/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - WebAuthn
  - Passkeys
  - FIDO2
---

# WebAuthn & Passkeys — FIDO2

**O que é**: padrão W3C (**WebAuthn Level 2 é a Recommendation vigente, abr/2021**; **Level 3 em Candidate Recommendation Snapshot, jan/2026**) + FIDO Alliance (FIDO2/CTAP2) para autenticação com **chaves criptográficas origin-bound**. "Passkey" é o termo de consumo para credenciais WebAuthn sincronizadas via iCloud Keychain, Google Password Manager, 1Password, Bitwarden.

> **Principal valor**: phishing-resistant. Credencial está amarrada ao **origin real** (`https://app.example.com`). Site falso não consegue usar. **A** defesa moderna contra credential phishing.

## Taxonomia

- **FIDO2** = WebAuthn (browser/platform API) + CTAP2 (Client to Authenticator Protocol para hardware keys).
- **Passkey** = credencial WebAuthn que é **discoverable** (resident key) + **sync fabric** (iCloud/Google/third-party). Substitui senha; "passwordless".
- **Security key** = hardware authenticator (YubiKey, SoloKey, Titan Key). Pode ser FIDO2 ou U2F (legado).
- **Platform authenticator** = built-in do device (Touch ID, Windows Hello, Android biometrics).
- **Cross-platform / roaming authenticator** = hardware key USB/NFC/Bluetooth.

## Como funciona (alto nível)

### Registration (enroll)

```
1. Server → Client: challenge + user info + rpID + pubKeyCredParams
2. Client → Authenticator: create key pair, tied to (rpID, user)
3. Authenticator: armazena secret; devolve (publicKey, credentialId, attestation)
4. Client → Server: publicKey, credentialId
5. Server: armazena associado a user account
```

Chave privada **nunca sai** do authenticator (nem do secure enclave em platform).

### Authentication

```
1. User → App: "login com passkey"
2. App → Client: challenge + rpID + credentialIds opcionais
3. Client → Authenticator: "sign this challenge for rpID"
4. Authenticator: verifica user (biometric/PIN local); assina
5. Client → App: signature + authenticatorData
6. App: valida assinatura contra publicKey armazenada
```

**Origin check**: browser verifica que `rpID` bate com origin. Site falso `app.evil.com` não consegue assinar para `rpID=app.example.com`.

## Fluxo de código (server)

### Registration

```ts
// Server: gera options
import { generateRegistrationOptions } from '@simplewebauthn/server'

const options = await generateRegistrationOptions({
  rpName: 'My App',
  rpID: 'app.example.com',
  userID: Buffer.from(user.id),
  userName: user.email,
  attestationType: 'none',        // ou 'direct' se precisa verificar device
  excludeCredentials: existingCreds,
  authenticatorSelection: {
    residentKey: 'preferred',      // passkey-ready
    userVerification: 'preferred', // biometric/PIN
  },
})
// Enviar ao cliente; cliente chama navigator.credentials.create(options)
```

### Verify registration

```ts
import { verifyRegistrationResponse } from '@simplewebauthn/server'

const verification = await verifyRegistrationResponse({
  response: attestationResponse,   // vindo do cliente
  expectedChallenge: sessionChallenge,
  expectedOrigin: 'https://app.example.com',
  expectedRPID: 'app.example.com',
})

if (verification.verified) {
  // Persist credentialId + publicKey + counter
  await db.insert({
    userId: user.id,
    credentialId: verification.registrationInfo.credentialID,
    publicKey: verification.registrationInfo.credentialPublicKey,
    counter: verification.registrationInfo.counter,
    deviceType: verification.registrationInfo.credentialDeviceType,
    backedUp: verification.registrationInfo.credentialBackedUp,
    transports: attestationResponse.response.transports,
  })
}
```

### Authentication

```ts
import { generateAuthenticationOptions, verifyAuthenticationResponse } from '@simplewebauthn/server'

// 1. Options
const opts = await generateAuthenticationOptions({
  rpID: 'app.example.com',
  allowCredentials: user ? user.credentials : [],  // empty → discoverable (passkey)
  userVerification: 'preferred',
})

// 2. Client envia assertionResponse
const v = await verifyAuthenticationResponse({
  response: assertionResponse,
  expectedChallenge: sessionChallenge,
  expectedOrigin: 'https://app.example.com',
  expectedRPID: 'app.example.com',
  authenticator: credFromDb,
})

if (v.verified) { /* session */ }
```

## Passkey vs 2FA hardware key

| Dimensão | Passkey (synced) | Hardware key (YubiKey) |
|---|---|---|
| **Storage** | Sync fabric (iCloud/Google/1Password) | Chip no device físico |
| **Portabilidade** | Multi-device automático | 1 device (plug USB/NFC) |
| **Recuperação** | iCloud/Google backup | Backup requer 2ª key |
| **Phishing-resistant** | ✅ | ✅ |
| **Sem senha?** | ✅ (primary factor) | ✅ ou como 2º fator |
| **Attestation forte** | ⚠️ (sync = não é device-bound) | ✅ (device attestation certificado) |
| **Admin/regulated** | OK user-facing | Preferido high-assurance |

## Passkey como **primary factor** (passwordless)

Fluxo moderno:
1. User abre app → "Sign in with passkey".
2. Browser: "Pick an account" (se múltiplas passkeys para o site).
3. Device pede biometric/PIN local.
4. App recebe assertion → sessão criada.

**Sem senha**, **sem OTP**, **sem SMS**. Experience > e segurança > .

## Conditional UI (autofill)

Passkey aparece como sugestão no campo username (como senha salva). UX de zero fricção.

```html
<input type="text" name="username" autocomplete="username webauthn">
```

Cliente chama `navigator.credentials.get({ mediation: 'conditional' })` — silencioso até user selecionar.

## Attestation

Prova criptográfica do **tipo de authenticator** (YubiKey vs platform vs software). Níveis:

- `none`: não pede attestation (padrão CIAM). Aceita qualquer.
- `indirect`: Anonymization CA (privacy-preserving).
- `direct`: certificate chain do vendor. Útil em enterprise (permitir só YubiKey 5).
- `enterprise`: enhanced attestation; exige BYO-authenticator corporativo.

## PRF (Pseudo-Random Function) extension — cripto use cases

WebAuthn L3 adiciona PRF: authenticator deriva secret de 32 bytes para cada rpID. Permite criptografia de dados **gated by passkey** (E2E encryption, password managers como Proton Pass).

## Como grandes empresas adotaram

- **Google**: passkey default em 2023+; option em accounts.google.com. Synced via Google Password Manager.
- **Apple**: passkey desde iOS 16 / macOS 13 (2022); sync via iCloud Keychain; airdrop sharing.
- **Microsoft**: Windows Hello passkey; Entra ID suporta passkey admin.
- **GitHub**: passkeys + security keys; 2FA obrigatório.
- **Shopify**: passkey em 2024.
- **Amazon**: passkey + SMS fallback em 2024.
- **1Password, Bitwarden, Proton Pass**: gerenciadores com passkey sync cross-platform.
- **Cloudflare**: passkey support em dashboard.

## Libs / SDKs

| Plataforma | Lib |
|---|---|
| **Server (Node)** | `@simplewebauthn/server` |
| **Server (Go)** | `github.com/go-webauthn/webauthn` |
| **Server (Python)** | `webauthn` (Duo Labs) |
| **Server (Java)** | `webauthn4j`, Yubico `java-webauthn-server` |
| **Client (JS)** | `@simplewebauthn/browser` ou Web API nativa |
| **iOS** | `AuthenticationServices` framework |
| **Android** | Credential Manager API |

## Anti-patterns

1. **SMS fallback quando passkey falha** — anula segurança. Se user perde device, recovery via support + identity verification, não SMS.
2. **Attestation forçado em CIAM** — alguns users tem authenticators raros; bloquear é UX péssimo. Use `none`.
3. **rpID mal escolhido** — `rpID` deve ser o registrable domain ou um ancestral do origin atual. Setando `rpID=app.example.com` você **consegue** assinar em subdomains descendentes (`x.app.example.com`), mas **não** sibling/parent (`www.example.com`). Risco típico: escolher rpID específico demais e depois não conseguir acessar de subdomínios irmãos. Use o ancestral comum mínimo necessário.
4. **Origin check laxo** — aceitar qualquer subdomain é antipattern. `expectedOrigin` strict.
5. **Sem counter check** — authenticators (não-passkey) incrementam counter; server deve verificar crescimento. Reset = device clonado.
6. **Passkey + senha antiga ainda válida** — migração: ao enrolar passkey, dar opção de desabilitar senha.
7. **Enrollment sem re-auth** — user logado com cookie antigo enrolla passkey maliciosa. Step-up obrigatório.

## Relações

- **Parte de**: [[mfa-step-up]]
- **Alternativas de fator**: [[otp-hotp-totp]], [[magic-links]]
- **Integra com**: [[oidc]] (acr/amr claims), [[identity-provider]]
- **Defende contra**: OWASP A07 (phishing, credential stuffing)
- **Zero Trust**: [[beyond-corp]] (passkey é peça do "strong identity")

## Fontes

- [WebAuthn Level 3 — W3C](https://www.w3.org/TR/webauthn-3/) (consultada 2026-04-24)
- [FIDO Alliance — Passkeys](https://fidoalliance.org/passkeys/) (consultada 2026-04-24)
- [passkeys.dev — developer guide](https://passkeys.dev/) (consultada 2026-04-24)
- [passkeys.com — end-user info](https://www.passkeys.com/) (consultada 2026-04-24)
- [SimpleWebAuthn — lib reference](https://simplewebauthn.dev/) (consultada 2026-04-24)
- [Apple Passkeys](https://developer.apple.com/passkeys/) (consultada 2026-04-24)
- [Google Passkeys](https://developers.google.com/identity/passkeys) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[mfa-step-up]]
- [[otp-hotp-totp]]
- [[identity-provider]]
- [[oidc]]
- [[beyond-corp]]
- [[owasp-top-10]] — A07 (passkeys são mitigação direta de phishing/credential stuffing)
- [[../runtime-antipatterns/op-024-rate-limit-ausente]] — auth endpoints exigem rate limit mesmo com passkey
