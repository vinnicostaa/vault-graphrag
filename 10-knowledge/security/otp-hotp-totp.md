---
title: OTP / HOTP / TOTP — One-Time Passwords (RFC 4226, RFC 6238)
type: concept
tags:
  - security
  - otp
  - hotp
  - totp
  - 2fa
  - mfa
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - OTP
  - HOTP
  - TOTP
  - One-Time Password
---

# OTP — One-Time Passwords

**O que é**: código de uso único para autenticação. Geralmente 6-8 dígitos numéricos, válido por uma única verificação ou janela curta de tempo. Implementações principais: **HOTP** (counter-based, RFC 4226) e **TOTP** (time-based, RFC 6238).

## Desambiguação: OTP em 3 sentidos

**Em tecnologia**, "OTP" pode significar três coisas **completamente diferentes**:

1. **One-Time Password** — esta nota. RFC 4226 / 6238. 2FA/MFA.
2. **Erlang/OTP** — *Open Telecom Platform*, framework e conjunto de libs da BEAM VM (Erlang/Elixir) para sistemas distribuídos, supervisão e concorrência. **Não tem relação** com autenticação. Ver knowledge em `20-stacks/erlang/` (lazy) quando destilado.
3. **One-Time Pad** — cifra teórica com perfect secrecy (Shannon 1949). Fora do escopo desta nota.

No resto desta nota, "OTP" = One-Time Password.

## HOTP — HMAC-based One-Time Password (RFC 4226)

**Mecânica**:

```
secret_key = 160-bit random (compartilhado entre server e authenticator)
counter    = 8-byte integer (incrementado a cada uso)

HOTP(secret, counter) = Truncate(HMAC-SHA1(secret, counter)) mod 10^d
                       onde d = tamanho (geralmente 6)
```

- Counter **não** sincroniza com tempo; sincroniza por uso.
- Server mantém "look-ahead window" (ex: +10 counters) para tolerar OTPs usados fora de ordem.
- Usado em hardware tokens tipo YubiKey no modo OTP, RSA SecurID legado.

**Downside**: sincronização quebrada (user aperta botão várias vezes sem usar → counter avança) requer re-sync.

## TOTP — Time-based One-Time Password (RFC 6238)

**Mecânica**: HOTP onde counter = `floor(Unix_time / period)`, typically period = 30s.

```
T_now = floor(unix_time() / 30)
OTP   = HOTP(secret, T_now)    # 6 dígitos padrão
```

Cada 30s o OTP muda. RFC 6238 §5.2 recomenda **1 step backward** (`T_now` + `T_now-1`) para tolerar clock drift normal; muitas implementações aceitam também `T_now+1` como defesa extra contra drift positivo — excede o RFC mas é prática de mercado. Janela típica em libs = ±1 step (±30s).

**Algoritmos**: SHA-1 padrão (RFC 6238); SHA-256 e SHA-512 opcionais. SHA-1 aqui é aceitável (HMAC, não hash direto); mas apps modernos suportam SHA-256.

### TOTP URI format (QR code)

Apps (Google Authenticator, Authy, 1Password, Bitwarden) aceitam:

```
otpauth://totp/My App:alice@example.com?
  secret=JBSWY3DPEHPK3PXP
  &issuer=My App
  &algorithm=SHA1
  &digits=6
  &period=30
```

Server gera o URI → renderiza como QR → user scaneia → app armazena `secret` + metadata.

## Padrão canônico (setup + verify)

```ts
// Node — usando otplib
import { authenticator } from 'otplib'

// 1. Enrollment
const secret = authenticator.generateSecret()   // base32 encoded
const uri = authenticator.keyuri(
  'alice@example.com',
  'My App',
  secret
)
// Render uri como QR; user scaneia no app.

// 2. Verify on login
const token = '123456'   // usuário digita
const isValid = authenticator.verify({
  token,
  secret,
  window: 1,      // tolerar ±30s
})
```

**Armazenamento server-side**: `secret` deve ser **criptografado at-rest**. Se vaza, atacante gera OTPs válidos indefinidamente.

## OTP via email / SMS

Sub-tipo: código curto enviado via canal externo em vez de gerado por app.

- **Email OTP**: link mágico também usa isso. Ver [[magic-links]].
- **SMS OTP**: vulnerável a SIM swap + SS7 interception. **NIST SP 800-63B desencoraja** SMS para novos designs.
- **Push notification** (app próprio): melhor UX, mais seguro que SMS.

## Comparação com outros fatores

| Mecanismo | Phishing-resistant? | UX | Recovery |
|---|---|---|---|
| TOTP | ❌ (phishing real-time) | App externo necessário | Recovery codes ou admin reset |
| HOTP | ❌ | Botão físico (YubiKey OTP mode) | Re-sync |
| SMS | ❌❌ (SIM swap) | Bom (sem app) | Automático se número OK |
| Email magic link | ❌ | Excelente | Automático |
| Push app | ⚠️ | Bom | Depende do app |
| **Passkey/WebAuthn** | ✅ | Excelente | Sync fabric ou backup |

**2026 — regra prática**: **passkey é preferido**; TOTP como fallback aceitável; SMS evitar em novo design.

## Anti-patterns

1. **Secret TOTP em plain text no banco** — rotação obrigatória se DB vaza. Sempre criptografar at-rest.
2. **OTP reutilizável** — aceitar o mesmo OTP duas vezes. Marca usado em cache/DB por período.
3. **Window muito grande** (±2min) — amplia superfície. ±30s-60s é adequado.
4. **Sem rate limit em `/verify`** — brute force de 6 dígitos = 1M combinações, viável em minutos sem limit.
5. **SMS como único fator 2** — vulnerável. Oferecer como **fallback** apenas.
6. **TOTP e passkey ao mesmo tempo sem precedência** — UX confusa; deixar passkey primary.
7. **Backup secret junto com passkey no mesmo device** — recovery fica vulnerável; use recovery codes offline.
8. **Clock drift tolerated demais** — 5+ minutes → janela de abuso.

## Como grandes empresas implementam

- **Google**: Google Authenticator (TOTP), com backup via conta Google. Opção de TOTP + passkey concomitante.
- **Apple**: iOS 15+ tem TOTP integrado no Keychain (iCloud sync). Apps de 2FA tipo Authy não necessários.
- **Microsoft Authenticator**: TOTP + push notification + passkey.
- **GitHub**: TOTP supported via app externo; SMS deprecated; encoraja passkey.
- **Bancos brasileiros (Itaú, Nubank, etc.)**: push via app próprio + biometria local, evita TOTP.

## Libs

| Linguagem | Lib |
|---|---|
| JS/Node | `otplib`, `speakeasy` (menos ativo) |
| Go | `github.com/pquerna/otp` |
| Python | `pyotp` |
| Java | `com.warrenstrange:googleauth` |
| Rust | `totp-rs` |

## Relações

- **Parte de**: [[mfa-step-up]]
- **Alternativas modernas**: [[webauthn-passkeys]] (preferido), [[magic-links]]
- **Não confundir com**: Erlang/OTP (framework distribuído) — ver `20-stacks/erlang/` quando destilado; contexto [[../methodology/_moc]] para concorrência
- **Storage**: [[secrets-management]] (seeds TOTP cifrados at-rest)
- **Integra com**: [[oidc]] (amr claim `otp` ou `mfa`)

## Fontes

- [RFC 4226 — HOTP](https://datatracker.ietf.org/doc/rfc4226/) (consultada 2026-04-24)
- [RFC 6238 — TOTP](https://datatracker.ietf.org/doc/rfc6238/) (consultada 2026-04-24)
- [Google Authenticator Key URI format](https://github.com/google/google-authenticator/wiki/Key-Uri-Format) (consultada 2026-04-24)
- [NIST SP 800-63B §5.1.4 — Out-of-Band verifier restrictions](https://pages.nist.gov/800-63-3/sp800-63b.html) (consultada 2026-04-24)
- [otplib (Node)](https://github.com/yeojz/otplib) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[mfa-step-up]]
- [[webauthn-passkeys]]
- [[magic-links]]
- [[secrets-management]]
- [[oidc]]
- [[../runtime-antipatterns/op-024-rate-limit-ausente]] — brute force de 6 dígitos (1M combos) exige rate limit
