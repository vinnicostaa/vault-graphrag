---
title: Mobile Master Síndico — README
type: project
tags: [master-sindico, mobile, flutter, dart]
project: master-sindico
subproject: mobile
created: 2026-04-22
---

# Mobile — Master Síndico

App iOS + Android — morador + síndico em mobilidade. Stack **Flutter + Dart**.

Repo local: `Development/app/`.

---

## Stack

| Camada | Tech | Motivo |
|---|---|---|
| Framework | Flutter (stable) | 1 codebase, performance nativa, hot reload |
| Linguagem | Dart 3.x (null-safety) | Type system maduro |
| Estado | Riverpod 2.x | Compile-safe, testável, escala |
| Navegação | go_router | Declarativo, deep-linking, auth guard |
| HTTP | Dio 5.8+ | Interceptors, retry, cancelToken (CVE-2024-41881 fixado) |
| Local storage | flutter_secure_storage + hive | Token em secure storage; cache em Hive |
| Push | firebase_messaging | FCM via IPushProvider backend |
| Testing | flutter_test + integration_test | Unit + widget + integration |
| Observability | Sentry Flutter | Erros em produção |
| Deep link | app_links | Universal links + app links |
| i18n | easy_localization | Pt-BR first, EN prep |
| Design tokens | via json + code gen | Compartilhado com web |

---

## Estrutura (feature-first + clean)

```
app/
├── pubspec.yaml
├── lib/
│   ├── main.dart
│   ├── app.dart                    # MaterialApp + ProviderScope + Router
│   ├── core/
│   │   ├── api/                    # Dio client + generated from OpenAPI
│   │   ├── auth/                   # auth state notifier
│   │   ├── config/                 # env (--dart-define)
│   │   ├── theme/                  # design tokens + light/dark
│   │   ├── errors/                 # AppException hierarchy
│   │   ├── network/                # connectivity monitor
│   │   └── utils/                  # constants, extensions
│   ├── features/
│   │   ├── identity/               # auth, session, onboarding
│   │   │   ├── data/               # repositories, datasources
│   │   │   ├── domain/             # use cases, entities
│   │   │   └── presentation/       # screens, widgets, controllers (Riverpod)
│   │   ├── billing/ institutional/ commercial/ content/ assembly/
│   ├── shared/
│   │   ├── widgets/                # design system widgets
│   │   └── models/                 # DTOs compartilhados
│   └── routes/
│       └── app_router.dart
├── assets/
│   ├── translations/               # pt_BR.json, en.json
│   ├── images/
│   └── fonts/
├── test/
│   ├── unit/ widget/ integration/
├── android/ ios/
└── .env.example
```

---

## Princípios

1. **Feature-first**: código relacionado à feature mora junto (data + domain + presentation)
2. **Clean layers dentro da feature**: data depende de domain; presentation depende de domain (via Riverpod); domain NÃO depende de nada externo
3. **Offline-first em leituras críticas** (perfil, condomínio, timeline recente): Hive cache + sync em background
4. **Secure storage** para tokens/sessão (nunca em prefs comuns)
5. **Certificate pinning** em produção (hosts conhecidos)
6. **Push notifications** via FCM — tópicos por tenant e por role
7. **Biometria opcional** pra re-login (local_auth)
8. **Accessibility**: semanticsLabel em todos os botões/ícones; fontes escaláveis

---

## Auth flow

1. App abre → checa token em secure storage
2. Token válido → `GET /api/v1/auth/me` pra popular AuthState
3. Token inválido/ausente → tela de login → OIDC via WebView + Custom Tab
4. Zitadel redireciona pra custom URL scheme `mastersindico://auth/callback`
5. App recebe `code + state`, troca por token via `POST /auth/mobile/token` (a implementar)
6. Armazena access + refresh token em secure storage
7. Dio intercepta 401 → refresh → retry

---

## Comandos

```bash
flutter pub get
flutter run -d chrome                  # dev web
flutter run --flavor dev                # dev Android/iOS com flavor
flutter test
flutter test integration_test/
flutter build apk --release --flavor prod --dart-define=API_BASE_URL=...
flutter build ipa --release --flavor prod
```

---

## Env (dart-define)

Constantes via `String.fromEnvironment` (A-030 fix em F29):

```dart
class AppConfig {
  static const apiBaseUrl = String.fromEnvironment(
    'API_BASE_URL',
    defaultValue: 'http://10.0.2.2:8000', // Android emu → localhost host
  );
  static const env = String.fromEnvironment('APP_ENV', defaultValue: 'development');
  static const sentryDsn = String.fromEnvironment('SENTRY_DSN', defaultValue: '');
}
```

Build com: `flutter build ... --dart-define=API_BASE_URL=https://api.mastersindico.com.br`.

---

## Telas por sprint

- Sprint 10 (M1): login, splash, home morador (dashboard), home síndico, minha unidade, meu condomínio
- Sprint 11 (M2): Connect Me, perfis empresa, marketplace, player vídeo Mux (HLS)
- Sprint 12 (M3): assembleia live (WebRTC Livekit Flutter SDK), votação, ata, LMS

---

## Specs

**Mobile não mantém specs próprias**. Mesmo tratamento do web: consome `backend/.kiro/specs/master-sindico/`.

`app/.kiro/specs/master-sindico/` foi DEPRECATED em 2026-04-22 (A-017).

---

## Links

- [[../_moc]]
- [[../../CLAUDE]]
- [[../../02-architecture/_moc]]
- [[../../04-requirements/_moc]]
- [[../../05-tasks/_moc]]
- [[../../../../20-stacks/flutter/_moc]] *(a criar)*
- [[architecture]] *(a criar)*
- [[patterns]] *(a criar)*
- [[requirements]] *(a criar)*
- [[tasks]] *(a criar)*
- [[security]] *(a criar)*
