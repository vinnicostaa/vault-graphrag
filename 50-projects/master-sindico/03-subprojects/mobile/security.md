---
title: Mobile — Security posture
type: spec
tags: [subproject, mobile, master-sindico, v2, security, secure-storage, cert-pinning, biometric, owasp-mobile]
source:
  - 08-security/BEYOND_CORP.md
  - Clientes/Joao/MasterSindico/Development/app/CLAUDE.md (§Segurança mobile, 2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/lib/features/auth/data/* (2026-04-21)
  - STATE.md D-050 (jailbreak escalonado MASVS-R)
created: 2026-04-23
updated: 2026-04-23
---

# Mobile — Security posture

Security do sub-projeto `app/` Flutter. Foco em **defesas mobile-specific**: Bearer token em Keychain/Keystore (não cookie), certificate pinning, OIDC PKCE em browser externo, biometria, integridade escalonada (D-050), PII handling, OWASP Mobile Top 10.

> **Princípio**: mesma garantia zero-trust + 1-device + PKCE OIDC do [[../web/security|web]] — **implementação diferente**. Cookie httpOnly não aplicável a app → Bearer em secure storage. Admin (D-058) não tem app separada → telas gated ABAC na mesma base de código.

---

## 1. Bearer token (nunca cookie)

### 1.1 Armazenamento obrigatório

- **`flutter_secure_storage`** — iOS Keychain (`accessibility: first_unlock_this_device`) + Android Keystore (AES-256-GCM com StrongBox quando disponível).
- Keys canônicas (alvo — `AppConstants`):
  - `auth_access_token`
  - `auth_refresh_token`
  - `device_fingerprint`
  - `hive_encryption_key`
  - `oidc_state` (state+nonce, TTL curto)

### 1.2 NUNCA usar

- ❌ `SharedPreferences` / `NSUserDefaults` para tokens — plain text readable em dispositivo comprometido.
- ❌ Hive box sem encryption.
- ❌ Arquivo em `getApplicationDocumentsDirectory()`.
- ❌ Variável global estática no Dart.
- ❌ Cookie jar Dio para auth — cookie é web-only.

### 1.3 Estado atual vs alvo

**Real (hoje)**: `AuthLocalDatasourceImpl` salva **token** em `FlutterSecureStorage` chave `AUTH_TOKEN` ✅ mas salva **snapshot user** em `SharedPreferences` chave `CACHED_USER` (⚠️ aceitável: snapshot não-sensível) e `AppConstants` reserva `auth_access_token` / `auth_refresh_token` (alvo canônico — refactor pendente).

**Alvo M1**:
- Keys canônicas consistentes em todo código.
- `refresh_token` **também** em `flutter_secure_storage` (hoje não persiste — re-login OIDC a cada expiração).
- Snapshot user pode continuar em `SharedPreferences` (é só UX — tokens é que não podem).

### 1.4 Interceptor Dio (alvo M1)

```dart
class AuthInterceptor extends Interceptor {
  final FlutterSecureStorage _storage;
  AuthInterceptor(this._storage);

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    final token = await _storage.read(key: AppConstants.tokenKey);
    if (token != null) options.headers['Authorization'] = 'Bearer $token';
    handler.next(options);
  }
}
```

### 1.5 Refresh flow (alvo M1)

`RefreshInterceptor` detecta 401 → usa `refresh_token` para obter novo `access_token` em Zitadel → retry original. Concorrência controlada (mutex) para evitar N refreshes simultâneos. Falha refresh = logout global (clear storage + redirect `/login`).

---

## 2. Certificate pinning

### 2.1 Prod obrigatório

Pin **SHA-256** dos certs de `api.mastersindico.com.br` + `auth.mastersindico.com.br`. Rotação semestral com **coexistência**: app versão N+1 pinneia cert novo + cert antigo → quando > 95% user base atualiza → release N+2 remove antigo.

### 2.2 Implementação via Dio adapter

```dart
class CertPinningAdapter extends IOHttpClientAdapter {
  static const pinnedSha256 = [
    'SHA256_PROD_CERT_ATUAL',
    'SHA256_PROD_CERT_BACKUP',   // rotação segura
  ];

  CertPinningAdapter() {
    createHttpClient = () {
      final client = HttpClient();
      client.badCertificateCallback = (X509Certificate cert, String host, int port) {
        if (host != 'api.mastersindico.com.br' && host != 'auth.mastersindico.com.br') return false;
        final hash = sha256.convert(cert.der).toString();
        return pinnedSha256.contains(hash);
      };
      return client;
    };
  }
}
```

Alternativa: `http_certificate_pinning` package (manutenção a validar — P-APP-003). Decisão final pelo que esteja ativo em 2026.

### 2.3 Rotação segura

- Novo cert deployado no backend 30 dias antes da rotação.
- App versão N+1 pinneia `[novo, antigo]` → coexiste.
- Após N+1 atingir > 95% user base → N+2 release remove antigo.
- Monitor Sentry: alertar em spike de `HandshakeException` ou similar.

### 2.4 Dev / staging

Sem pinning — facilita proxy de inspeção (Charles, mitmproxy). Controlado por `--dart-define=APP_ENV=dev` que desabilita o adapter.

---

## 3. HTTPS-only (ATS iOS + networkSecurityConfig Android)

### 3.1 iOS — `NSAppTransportSecurity`

Em `Info.plist` (⚠️ pendente — hoje não declarado):

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <!-- Sem NSExceptionDomains em prod -->
</dict>
```

### 3.2 Android — `networkSecurityConfig`

`android/app/src/main/res/xml/network_security_config.xml` (⚠️ pendente):

```xml
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    <debug-overrides cleartextTrafficPermitted="true" />
</network-security-config>
```

E em `AndroidManifest.xml` `<application android:networkSecurityConfig="@xml/network_security_config" ...>`.

Dev (`localhost:8000`, `10.0.2.2:8000`) via `debug-overrides`. Prod **sempre** HTTPS bloqueando cleartext.

---

## 4. OIDC PKCE flow (mobile-specific)

### 4.1 `oidc` package — config real

Hoje em `AuthRemoteDatasourceImpl._getUserManager`:

```dart
_userManager = OidcUserManager.lazy(
  discoveryDocumentUri: OidcUtils.getOpenIdConfigWellKnownUri(Uri.parse(AppConstants.zitadelIssuer)),
  clientCredentials: const OidcClientAuthentication.none(
    clientId: AppConstants.zitadelClientId,
  ),
  store: OidcDefaultStore(),
  settings: OidcUserManagerSettings(
    redirectUri: Uri.parse(AppConstants.zitadelRedirectUri), // com.mastersindico:/auth
  ),
);
```

### 4.2 External user agent obrigatório

- **iOS**: `ASWebAuthenticationSession` (o `oidc` package escolhe isso via `AppAuth` under the hood em prod) — isolated, no cookie share com outros apps.
- **Android**: **Custom Tabs** — isolation similar.

**Nunca WebView embedada** para auth:
- Zitadel **bloqueia** (detecta UA `WebView` e responde 403).
- User não vê URL bar — indistinguível de phishing.
- Cookies escapam isolation do browser system.

### 4.3 Custom scheme registration

**Real (confirmado no código)**:
- `AndroidManifest.xml`:
  ```xml
  <intent-filter>
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <data android:scheme="com.mastersindico" />
  </intent-filter>
  ```
- `Info.plist`:
  ```xml
  <key>CFBundleURLTypes</key>
  <array>
      <dict>
          <key>CFBundleTypeRole</key>
          <string>Editor</string>
          <key>CFBundleURLSchemes</key>
          <array>
              <string>com.mastersindico</string>
          </array>
      </dict>
  </array>
  ```

### 4.4 State + nonce

`oidc` package gera automaticamente `state` + `nonce` + PKCE `code_verifier` / `code_challenge`. Validação no callback: state match (previne CSRF) + nonce match (previne replay). Se mismatch → aborta fluxo + audit log.

### 4.5 Scopes mínimos

Confirmado em `AppConstants.zitadelScopes`:
```dart
static const List<String> zitadelScopes = [
  'openid',
  'profile',
  'email',
  'offline_access',   // obrigatório para obter refresh_token
];
```

---

## 5. Device fingerprint (IR-011)

### 5.1 Componentes — `device_info_plus`

```dart
Future<String> computeFingerprint() async {
  final info = DeviceInfoPlugin();
  final data = StringBuffer();
  if (Platform.isAndroid) {
    final a = await info.androidInfo;
    data.write('${a.id}|${a.model}|${a.version.sdkInt}|${a.fingerprint}');
  } else if (Platform.isIOS) {
    final i = await info.iosInfo;
    data.write('${i.identifierForVendor}|${i.model}|${i.systemVersion}');
  }
  return sha256.convert(utf8.encode(data.toString())).toString();
}
```

### 5.2 Persistência

Calculado 1x na primeira execução → persistido em `flutter_secure_storage` chave `device_fingerprint`. Estável por device (mesmo após reinstall no mesmo device, via `identifierForVendor` iOS / `id + fingerprint` Android).

### 5.3 Header

`DeviceInterceptor` Dio injeta `X-Device-Fingerprint: <hash>` em toda request autenticada.

### 5.4 1-device policy

Backend rejeita com `403 NEW_DEVICE_DETECTED` se fingerprint diferente da session ativa para o mesmo user. App trata via `ErrorInterceptor` → emit `NewDeviceFailure` → `App.router` redireciona APP-SYS-004 → user volta para login (sessão transferida para outro device).

---

## 6. Biometric authentication — `local_auth`

### 6.1 API

```dart
final canCheck = await _auth.canCheckBiometrics;
final available = await _auth.getAvailableBiometrics();

final success = await _auth.authenticate(
  localizedReason: 'Confirme sua identidade para continuar',
  options: const AuthenticationOptions(
    biometricOnly: true,
    stickyAuth: true,
    useErrorDialogs: false,
  ),
);
```

### 6.2 Strings de uso (⚠️ pendentes em `Info.plist`)

```xml
<key>NSFaceIDUsageDescription</key>
<string>Usamos sua biometria para confirmar ações sensíveis (procuração, ata, exclusão de conta).</string>
```

### 6.3 Usos

- **Re-open** após inatividade > 30min (opt-in configurável em APP-ACC-004).
- **Ações críticas — obrigatório** (BR-MOB-BIO-003):
  - Outorgar procuração (APP-ASM-004).
  - Assinar ata (M3).
  - Delete account (APP-ACC-006 LGPD).
  - Upgrade plano (double-confirm).
  - Bypass Lock 90d admin (APP-ADM-002/003).

### 6.4 NÃO substitui OIDC

Primeiro login **sempre** OIDC com senha via PKCE Zitadel. Biometria **suplementa** — valida presença do titular sem revalidar credencial Zitadel.

### 6.5 Fallback

Device sem biometria OU 3× biometric failure → cai no fluxo padrão (re-login OIDC). Mensagem clara: "Biometria indisponível. Faça login novamente."

---

## 7. Jailbreak / root / integrity — D-050 escalonada (MASVS-R)

### 7.1 Contexto

OWASP MASWE-0097: root/jailbreak detection é **mandatory control** quando app lida com dados sensíveis / decisões críticas (votação, assembleia, pagamento, assinatura contratual). `flutter_jailbreak_detection` é **trivially bypassed** com Frida. MASVS-R explicitamente: *"any system-level API could be easily bypassed using reverse engineering tools"*.

**Solução adotada (D-050)**: **freeRASP (Talsec) Flutter** combina root/jailbreak + Frida/hook detection + debugger + app signature + device binding.

### 7.2 Estados de integridade

Verificados em startup e em intervalos durante sessão:

- `ok` → fluxo normal.
- `dev-mode` → log + audit + **aviso amarelo** ("modo dev detectado, alguns recursos podem ser limitados").
- `jailbroken` → log + audit + **bloqueia features críticas** (votação, assembleia live, assinatura contrato); modal: "Dispositivo não seguro, use outro para esta operação."
- `hooked` (Frida/injetado) → log + audit + **bloqueia sessão** + força re-login.

### 7.3 Header `X-Device-Integrity`

Enviado em toda request autenticada: `X-Device-Integrity: ok | dev-mode | jailbroken | hooked`.

### 7.4 Roll-out escalonado

- **M1 report-only**: todos os estados só geram audit log + header. **Sem bloqueio**. Valida taxa de falso-positivo em campo (síndicos técnicos com device enraizado legítimo não podem ser barrados na estreia).
- **M3 gated**: features críticas bloqueiam em `jailbroken` / `hooked` (produção). UX vem de APP-SYS-006.

### 7.5 Config freeRASP mínima

```dart
await FreeRASP.init(
  TalsecConfig(
    androidConfig: AndroidConfig(
      expectedPackageName: 'com.mastersindico.app',
      expectedSigningCertificateHashBase64: ['SHA256_BASE64'],
      supportedAlternativeStores: ['com.sec.android.app.samsungapps'],
    ),
    iosConfig: IOSConfig(
      bundleIds: ['com.mastersindico.app'],
      teamId: 'TEAM_ID',
    ),
    watcherMail: 'security@mastersindico.com.br',
  ),
  callback: (t) { /* handle threat */ },
);
```

---

## 8. PII handling

### 8.1 Logs via `logger`

- **Nunca** `print(...)` ou `log(...)` com PII direto.
- Wrapper canônico em `core/utils/pii_masker.dart` (alvo M1):
  ```dart
  String maskCpf(String cpf)   => cpf.length == 11 ? '${cpf.substring(0,3)}.***.***-**' : '***';
  String maskEmail(String e)   => e.replaceFirstMapped(RegExp(r'(.)[^@]*'), (m) => '${m[1]}***');
  String maskToken(String t)   => t.length > 12 ? '${t.substring(0,6)}...${t.substring(t.length-4)}' : '***';
  ```
- Uso: `Logger().i('User loaded: id=${user.id} email=${maskEmail(user.email)}')`.

### 8.2 Sentry scrubbing

```dart
SentryFlutter.init((options) {
  options.dsn = '...';
  options.tracesSampleRate = 0.2;
  options.beforeSend = (event, hint) async {
    return event.copyWith(
      extra: _scrubMap(event.extra),
      request: event.request?.copyWith(
        headers: _scrubHeaders(event.request?.headers),
        data: null,    // nunca enviar body com PII
      ),
      contexts: event.contexts,   // device info OK
    );
  };
});
```

Scrub keys (case-insensitive): `Authorization`, `X-Device-Fingerprint`, `X-Device-Integrity`, `Cookie`, `Set-Cookie`, `email`, `cpf`, `cnpj`, `token`, `refresh_token`, `password`, `phone`.

### 8.3 Clipboard

Nunca copiar CPF/senha/token automaticamente. User action explícita only (ex: tap "Copiar código de condomínio" é aceitável).

### 8.4 Screenshot / screen recording

Telas com dados sensíveis (assembleia live, ata, docs compliance, dados financeiros):
- **Android**: `FLAG_SECURE` na Activity via `flutter_windowmanager.addFlags(FlutterWindowManagerFlags.FLAG_SECURE)` no `initState` e `clearFlags` no `dispose`.
- **iOS**: `UIScreen.capturedDidChangeNotification` — em captura detectada, overlay a tela com placeholder ou esconder o scaffold via `AppLifecycleState`.

---

## 9. Secret management

### 9.1 Dart-define, não hardcode

```bash
flutter build ... \
  --dart-define=SENTRY_DSN=... \
  --dart-define=ZITADEL_CLIENT_ID=... \
  --dart-define=API_BASE_URL=https://api.mastersindico.com.br \
  --dart-define=ZITADEL_ISSUER=https://auth.mastersindico.com.br
```

Confirmado: `AppConstants` usa `String.fromEnvironment(...)` com defaults seguros (localhost dev).

### 9.2 Secrets fora do bundle

- **SDK keys públicas** (Firebase config, Mux public) — OK em bundle.
- **Secrets reais** — **nenhum do mobile** deve ser secret (toda assinatura JWT/Mux é no backend). Se aparecer algum, revisar arquitetura.

### 9.3 Google Services files

`google-services.json` (Android) + `GoogleService-Info.plist` (iOS) — commitados por flavor em diretórios específicos (`android/app/src/dev/`, `src/staging/`, `src/prod/`). Config não-secreta mas não deve misturar ambientes.

---

## 10. Code obfuscation + stack traces

### 10.1 Release builds

```bash
flutter build apk --release  --obfuscate --split-debug-info=build/symbols/
flutter build ipa --release  --obfuscate --split-debug-info=build/symbols/
```

Obfuscation dificulta reverse engineering mas **não é segurança por obscuridade** — regras de negócio ficam no backend.

### 10.2 Stack traces legíveis — Sentry

Upload symbols pra Sentry em CI:
```bash
sentry-cli upload-dif build/symbols
```

Stack traces em produção ficam simbolicados no dashboard Sentry — equipe debug sem rebuild.

---

## 11. Dependency security

### 11.1 Audit

- `flutter pub outdated` semanal (script CI).
- `flutter pub upgrade --dry-run` review antes de upgrade major.
- Snyk ou `flutter pub audit` (quando disponível) para CVE monitoring.

### 11.2 Pinned deps

- `pubspec.yaml` com caret ranges (`^5.8.0`) — cap em major.
- `pubspec.lock` **commitado** (confirmado no repo).

### 11.3 Dio CVE-2024-41881

Dio ≥ 5.7.0 corrige. Versão atual: **5.8.0** (confirmado em pubspec) ✅.

---

## 12. Storage encryption adicional (Hive + secure_storage)

### 12.1 Hive box encryption

Boxes com dados sensíveis usam `HiveAesCipher` com chave derivada de `flutter_secure_storage`:

```dart
Future<HiveAesCipher> _buildCipher() async {
  final existing = await secureStorage.read(key: AppConstants.hiveEncryptionKey);
  final bytes = existing != null
      ? base64Url.decode(existing)
      : Hive.generateSecureKey();
  if (existing == null) {
    await secureStorage.write(key: AppConstants.hiveEncryptionKey, value: base64Url.encode(bytes));
  }
  return HiveAesCipher(bytes);
}

await Hive.openBox('memberships', encryptionCipher: await _buildCipher());
```

### 12.2 Cache expiry

TTLs agressivos em cache de dados sensíveis (ver [[requirements#br-mob-off-001-offline-first-seletivo|BR-MOB-OFF-001]]). **Clear em logout obrigatório** — todas as boxes.

---

## 13. Session lifecycle

### 13.1 Inatividade

App inativo > **30min** em background → require biometric OR senha OIDC na próxima abertura (configurável em APP-ACC-004).

### 13.2 Lock manual

User pode "lockar" via menu → re-auth no próximo uso.

### 13.3 Backgrounded screen

iOS/Android mostram screenshot thumbnail no app switcher. Para telas sensíveis:
- Overlay com logo/placeholder em `AppLifecycleState.inactive` → `.paused`:
  ```dart
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.inactive && _isSensitiveRoute) {
      showOverlay();
    } else if (state == AppLifecycleState.resumed) {
      hideOverlay();
    }
  }
  ```

### 13.4 Logout clean

Processo obrigatório ao logout (expandir `AuthRepositoryImpl.logout` atual):

1. `manager.logout()` Zitadel (end_session).
2. `flutter_secure_storage.deleteAll()` — remove todas as chaves.
3. Clear Hive boxes (incluindo `sync_queue` — ações pendentes descartadas).
4. Clear Dio cookie jar (se houver).
5. **Unsubscribe FCM topics** (`user:<id>`, todos os `tenant:<condId>`).
6. Clear `hydrated_bloc` storage.
7. Navigate `/login`.

---

## 14. OWASP Mobile Top 10 — controles

| Risco MASVS | Controle no app |
|---|---|
| **M1 Improper Platform Usage** | Keychain/Keystore usado corretamente; `NSFaceIDUsageDescription` declarado; intent-filters restritos ao custom scheme |
| **M2 Insecure Data Storage** | `flutter_secure_storage` para tokens; Hive encrypted para cache; sem sensível em `SharedPreferences` |
| **M3 Insecure Communication** | HTTPS-only (ATS + networkSecurityConfig) + cert pinning prod |
| **M4 Insecure Authentication** | OIDC PKCE via browser externo; biometric reauth em ações críticas; 1-device enforcement IR-011 |
| **M5 Insufficient Cryptography** | SDK nativo (Keychain/Keystore + AES-GCM); **no rolling own crypto** |
| **M6 Insecure Authorization** | Backend ABAC re-valida toda request; frontend só UX (admin D-058 gated via token) |
| **M7 Client Code Quality** | Dart null-safety + `flutter analyze` + tests thresholds |
| **M8 Code Tampering** | Obfuscation (`--obfuscate`) + cert pinning + freeRASP runtime check (D-050) |
| **M9 Reverse Engineering** | Obfuscation; símbolos fora do bundle; secrets no backend |
| **M10 Extraneous Functionality** | Dev menus strip em release; feature flags ocultam screens em prod |

---

## 15. Incident response (device comprometido)

User reporta roubo/perda:

1. User em `/account/seguranca` (ou via web) → [Encerrar sessões em outros dispositivos].
2. Backend invalida `refresh_token` daquele device fingerprint + revoga session.
3. Próxima request no device roubado → 401 → RefreshInterceptor tenta refresh → fails (refresh revoked) → logout + clear secure_storage + redirect `/login`.
4. Se device ainda consegue abrir app no cached state: `FLAG_SECURE` + `AppLifecycleState` overlay protege screenshots; mas dados locais (Hive cached) continuam no device até reinstall — **hardening**: em próxima conexão forçar re-download + purge local.

---

## 16. Admin (D-058) — considerações mobile

Admin = `apps/admin` dedicada no monorepo web (D-134 revoga D-058/D-072) — **mobile não tem telas admin**. Acessam via desktop em `admin.mastersindico.com.br` com Cloudflare Access SSO+MFA. Implicações de segurança no mobile:

- Token admin carrega `scope: admin` + permissões específicas na matriz ABAC. Backend valida em **toda** request; frontend apenas UX.
- Telas admin gated (APP-ADM-*) aparecem conforme permissão; bot de inspeção de tela decompilando binário vê as rotas admin mas **não consegue** invocar — 403 do backend.
- **4-eyes policy** (D-054) para bypass Lock 90d: Admin A solicita (mobile OK), Admin B aprova em janela ≤ 24h (mobile OK, mas recomenda-se web para contexto completo). Ambos com **biometria** obrigatória (BR-MOB-BIO-003).
- Auditoria: toda ação admin loga em `audit_trail` + `lgpd_logs` (se afeta dados pessoais). Mobile registra via header `X-Admin-Action-Context` opcional.

---

## 17. Checklist pré-release (gate security)

Antes de submeter store (TestFlight / Play Internal → Prod):

- [ ] Certificate pinning ativo e cert SHA-256 confirmado em build.
- [ ] `--obfuscate --split-debug-info=build/symbols` aplicado.
- [ ] Symbols upload Sentry completo.
- [ ] ATS (iOS) + networkSecurityConfig (Android) bloqueando cleartext prod.
- [ ] `minSdk = 26` (Android) e `iOS 15+` (iOS) confirmados.
- [ ] Permissões declaradas em `Info.plist` / `AndroidManifest.xml` (camera, mic, photo lib, biometria).
- [ ] Deep-link allowlist testada (fuzz rápido).
- [ ] `FlutterSecureStorage` confirmado não-gravando em backup nativo (iOS `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly` considerar; Android `autoBackup=false` para keystore).
- [ ] freeRASP integrado (M3) ou header `X-Device-Integrity` reportando (M1).
- [ ] `flutter analyze` + `flutter test --coverage` gates passando.
- [ ] Privacy Labels (iOS) + Data Safety (Google Play) declarados — BR-MOB-STORE-001/002.
- [ ] Sentry top 20 crashes revisados; crash-free ≥ 99.5%.
- [ ] PII scrubbing confirmado em logs reais (spot check sample).

---

## 18. Links

- [[README]]
- [[architecture]]
- [[patterns]]
- [[requirements]]
- [[../backend/security]]
- [[../web/security]]
- [[../../08-security/BEYOND_CORP]]
- [[../../08-security/owasp-top10]]
- [[../../STEERING]] §17-23
- [[../../STATE#D-050|D-050 Integrity escalonada MASVS-R]]
- [[../../STATE#D-054|D-054 Lock 90d 4-eyes]]
- [[../../STATE#D-055|D-055 UUIDv7]]
- [[../../STATE#D-058|D-058 Admin role privilegiada]]
