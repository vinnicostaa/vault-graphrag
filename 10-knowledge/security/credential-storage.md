---
title: Credential Storage — client-side (browser + mobile + desktop)
type: concept
tags:
  - security
  - credential-storage
  - client-side
  - localstorage
  - sessionstorage
  - cookie
  - keychain
scope: client
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Client Credential Storage
  - Token Storage
---

# Credential Storage — client-side

**O que é**: onde armazenar **credenciais/tokens do user** no lado cliente (browser, mobile, desktop, CLI). Contraparte de [[secrets-management]] (que cobre infra/dev). Decisão errada = XSS rouba tokens, localStorage é backdoor, hardcode em mobile vaza via reverse engineering.

> **Scope**: client-side, user-facing. Token de sessão, refresh token, encryption key do user. **Não** é secret de infra (API key do backend).

## Espectro — do pior ao melhor

### Browser (web)

| Storage | Acesso JS | Persiste após tab close? | Persiste após browser close? | XSS exposure | Bom para |
|---|---|---|---|---|---|
| **In-memory (React state, variável)** | ✅ (próprio JS) | ❌ | ❌ | ⚠️ (acessível por XSS while running) | Access token curto; PKCE verifier |
| **sessionStorage** | ✅ (same-origin JS) | ❌ (aba fechou = some) | ❌ | ⚠️ (XSS lê tudo) | Flow em andamento (PKCE verifier durante redirect) |
| **localStorage** | ✅ (same-origin JS) | ✅ | ✅ (até user limpar) | 🔴 (XSS rouba tudo, persistente) | Dado não-sensível (preferences, draft UI state) |
| **IndexedDB** | ✅ (same-origin JS) | ✅ | ✅ | 🔴 (XSS rouba) | Dado estruturado não-sensível (offline cache) |
| **Cookie (sem flags)** | ✅ (document.cookie) | Depende | Depende | 🔴 (XSS) | Antipattern — não usar |
| **Cookie `HttpOnly`** | ❌ (JS não lê) | ✅ | ✅ | ✅ (XSS **não** rouba) | **Session tokens** (padrão-ouro) |
| **Service Worker cache** | ✅ (JS com SW) | ✅ | ✅ | ⚠️ (escopo de SW) | Responses cacheadas, não tokens |
| **WebAuthn / PRF** | ✅ (via API) | ✅ | ✅ | ✅ (device-bound) | Keys assimétricas; [[webauthn-passkeys]] |

### Mobile nativo

| Storage | Secure? | Bom para |
|---|---|---|
| **SharedPreferences (Android), UserDefaults (iOS)** | ❌ | Antipattern para token (plain text) |
| **Android Keystore / iOS Keychain** | ✅ (HW-backed quando disponível) | Session tokens, refresh tokens |
| **iOS Secure Enclave** | ✅ (SE chip) | Biometric-gated operations |
| **Android StrongBox** | ✅ (tamper-resistant) | High-value keys |
| **Expo SecureStore / flutter_secure_storage** | ✅ (wrappers above) | Cross-platform Keychain/Keystore |

### Desktop

| Storage | Bom para |
|---|---|
| **macOS Keychain** | Tokens, passwords, SSH keys |
| **Windows Credential Manager / DPAPI** | Tokens, SSH keys |
| **Linux Secret Service (gnome-keyring, kwallet)** | Tokens |
| **libsecret** (GNOME) / **kwallet** (KDE) | Cross-desktop |
| **File encrypted with age/GPG** | Fallback quando sem keyring |

### CLI / server (processo longo-running)

| Storage | Bom para |
|---|---|
| **In-memory only** | Token short-lived; re-fetch em cada start |
| **Encrypted file + passphrase** | Persistent sessions (gh CLI, aws CLI) |
| **System keyring** (via libsecret/Keychain) | Credential sharing between CLIs |
| **Hardware key (YubiKey PIV)** | Private keys; high assurance |

## Regras de decisão (browser)

### Para session / access token (JWT curto)

**✅ Recomendado**: Cookie `HttpOnly` + `Secure` + `SameSite=Lax` ou `Strict`.
- XSS não rouba.
- Envia automaticamente nas requests same-origin.
- CSRF defendida pelo `SameSite` + token extra se necessário ([[csrf-defense]]).

**❌ Evitar**: localStorage/sessionStorage para JWT session.
- Auth0 desaconselhou oficialmente em 2023+ por conta de XSS epidemic.
- Storage.js ataques recorrentes.

### Para refresh token

**✅ Recomendado**: Cookie `HttpOnly` + `Secure` + `SameSite=Strict` + path dedicado (`/auth/refresh`).
- Mais restritivo que access token cookie.
- Path específico limita envio.

### Para PKCE `code_verifier`

**✅ Recomendado**: sessionStorage (vida do flow curta, ~5min) + flag de uso único.
- Aba fecha → some (acceitável; user re-inicia login).
- Em mobile/desktop nativo: memória.

**❌ Evitar**: localStorage (persiste entre tentativas — confuso se fluxo repetido).

### Para crypto keys (E2E encryption, PRF passkey)

**✅ Recomendado**: IndexedDB com `CryptoKey` não-extractable (`extractable: false`).
- Web Crypto API gera chave que JS **não consegue exportar**; só operar (sign, decrypt).
- Mesmo XSS não rouba (acessa a referência, mas não extrai bytes).

```ts
const key = await crypto.subtle.generateKey(
  { name: 'AES-GCM', length: 256 },
  false,                                  // extractable = false
  ['encrypt', 'decrypt']
)
await db.put('keys', key, 'user-encryption-key')
```

## Mobile — padrão canônico

### iOS

```swift
import Security

// Store
let data = sessionToken.data(using: .utf8)!
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "session-token",
    kSecValueData as String: data,
    kSecAttrAccessible as String: kSecAttrAccessibleAfterFirstUnlock,
]
SecItemAdd(query as CFDictionary, nil)
```

### Android (Keystore)

```kotlin
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val sharedPrefs = EncryptedSharedPreferences.create(
    context, "session", masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
sharedPrefs.edit().putString("token", token).apply()
```

### Flutter

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

final storage = FlutterSecureStorage(
  aOptions: AndroidOptions(encryptedSharedPreferences: true),
  iOptions: IOSOptions(accessibility: KeychainAccessibility.first_unlock),
);
await storage.write(key: 'session', value: token);
```

## Cross-platform padrão

### Expo SecureStore
```ts
import * as SecureStore from 'expo-secure-store'
await SecureStore.setItemAsync('session', token)
```

### Capacitor (Ionic)
```ts
import { SecureStoragePlugin } from 'capacitor-secure-storage-plugin'
await SecureStoragePlugin.set({ key: 'session', value: token })
```

## Biometric-gated access (iOS/Android)

Tokens "desbloqueados" só com biometria + PIN:

### iOS
```swift
let access = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    .biometryAny,
    nil
)!
```

### Android
```kotlin
val keyGenParameterSpec = KeyGenParameterSpec.Builder("key", KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
    .setUserAuthenticationRequired(true)
    .setUserAuthenticationValidityDurationSeconds(60)
    .build()
```

User precisa FaceID/fingerprint para cada acesso (ou recente durante N segundos).

## Anti-patterns

1. **Access token em localStorage** — XSS rouba tudo. Use cookie httpOnly.
2. **SharedPreferences (Android) plain text** — acessível com root / backup. Use EncryptedSharedPreferences ou Keystore.
3. **UserDefaults (iOS) para token** — mesmo problema. Use Keychain.
4. **Hardcoded credentials em mobile app binary** — `strings <app>` vaza. Use SafetyNet Attestation + runtime fetch.
5. **Cookie sem `Secure` em HTTPS** — MITM em hotspot.
6. **Cookie sem `SameSite`** — CSRF default. `Lax` mínimo. Ver [[cookies]].
7. **Token in URL** (`?access_token=...`) — referrer leak, histórico browser, server logs. Sempre Authorization header ou cookie.
8. **IndexedDB com chave extractable** — XSS exporta. `extractable: false` sempre que possível.
9. **CLI token em `~/.env` sem ACL** — qualquer processo do user lê. Keychain + fallback encrypted file.
10. **"Remember me" forever** sem rotação — token eterno é backdoor.
11. **Misturar secrets em mesmo storage** — keychain entry por purpose; scope por service.

## Modelos híbridos

### "SPA sem backend": JWT em cookie

SPA servido por CDN; API direct. Cookie httpOnly ainda funciona — API tem mesmo domain.

### "SPA + BFF" (Backend-for-Frontend)

Backend leve intermedia entre SPA e API. Cookie httpOnly → BFF valida + adiciona Authorization header para backend. SPA não vê token.

### "Split tokens" (Auth0 Silent Authentication pattern)

- Access token: em memória JS (curto, re-obtain silent).
- Refresh token: cookie httpOnly.
- Silent refresh via iframe ou BFF.

## LGPD/GDPR

- Tokens permitem acesso a dados pessoais → storage deve seguir princípios de proteção.
- Backup encryption (keychain já protege em bom mode).
- Right to be forgotten: logout limpa storage local.
- Consent banners **não** se aplicam a session tokens (necessários para função do serviço).

## Como grandes empresas fazem

- **Google, Microsoft, Apple, Amazon**: cookie httpOnly primary; sessão server-side; refresh via endpoint dedicado.
- **Auth0**: SDK oferece storage options; recomendação default = cookie httpOnly desde 2023.
- **Stripe Dashboard**: cookie httpOnly + 2FA cookie; session curta.
- **GitHub**: cookie httpOnly + per-session token; device tracking + suspicious logout.
- **Mobile apps (WhatsApp, Slack, Twitter)**: Keychain/Keystore + biometric gate para features sensíveis.

## Relações

- **Cookie profundo (atributos, CHIPS, prefixes)**: [[cookies]]
- **CSRF defense**: [[csrf-defense]]
- **Session server-side**: [[session-management]]
- **Passkeys** (alternativa moderna): [[webauthn-passkeys]]
- **Contraparte infra**: [[secrets-management]]
- **Runtime antipatterns**: [[../runtime-antipatterns/op-022-log-com-pii|OP-022]] (nunca logar token), [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]]

## Fontes

- [OWASP HTML5 Security Cheat Sheet — Local Storage](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html) (consultada 2026-04-24)
- [Auth0 — Token Storage Best Practices](https://auth0.com/docs/secure/security-guidance/data-security/token-storage) (consultada 2026-04-24)
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/) (consultada 2026-04-24)
- [iOS Keychain Services](https://developer.apple.com/documentation/security/keychain_services) (consultada 2026-04-24)
- [Android Keystore](https://developer.android.com/privacy-and-security/keystore) (consultada 2026-04-24)
- [Web Crypto API — extractable keys](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[cookies]]
- [[session-management]]
- [[csrf-defense]]
- [[webauthn-passkeys]]
- [[secrets-management]]
- [[security-principles]]
