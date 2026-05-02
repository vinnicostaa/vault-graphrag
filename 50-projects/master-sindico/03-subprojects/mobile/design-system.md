---
title: "Mobile — Design System (Flutter)"
type: spec
tags: [mobile, design-system, flutter, manual-iv, master-sindico, v2, fase-8-consolidado]
source: 03-subprojects/web/design-system.md + Content/contents/Manual Identidade Visual + Development/app/lib/core/theme/
created: 2026-04-23
updated: 2026-04-23
status: consolidado-fase-8
dual_check: pending
---

# Design System — Mobile Flutter

Espelha o design system canônico do web (`../web/design-system.md`), adaptado para Flutter (Material 3 + custom theme). Fonte de cores e tipografia: **Manual de Identidade Visual Master Síndico** (ouro #C69C3F / navy #071A33 / Michroma wordmark + Inter/Manrope body). **⚠️ gap atual**: `app/lib/` usa paleta default Flutter `#6C63FF` (deepPurple) — precisa migração para Manual IV (tarefa DT-MB-008 em [[../mobile/tasks]]).

---

## § 1. Tokens de cor — Manual IV canônico

### 1.1 Paleta primária (OKLCH → ARGB conversion)

| Token | OKLCH (web canônico) | ARGB Flutter | Hex | Uso |
|---|---|---|---|---|
| `primaryGold` | `oklch(0.715 0.120 84.2)` | `Color(0xFFC69C3F)` | `#C69C3F` | CTAs primárias, destaque |
| `primaryGoldLight` | `oklch(0.850 0.080 84.2)` | `Color(0xFFFFCC6A)` | `#FFCC6A` | Hover/press gold |
| `secondaryNavy` | `oklch(0.215 0.045 260)` | `Color(0xFF071A33)` | `#071A33` | Background escuro, headers |
| `secondaryNavyLight` | `oklch(0.320 0.060 260)` | `Color(0xFF0A274C)` | `#0A274C` | Navy secondary |
| `accent` | (derivado) | `Color(0xFFC69C3F)` | = primaryGold | |
| `destructive` | standard red | `Color(0xFFE74C3C)` | `#E74C3C` | Ações destrutivas, errors |
| `success` | standard green | `Color(0xFF27AE60)` | `#27AE60` | Sucesso, confirmação |
| `warning` | standard amber | `Color(0xFFF39C12)` | `#F39C12` | Atenção, pending |
| `info` | standard blue | `Color(0xFF3498DB)` | `#3498DB` | Info, neutral alert |

### 1.2 Neutros

| Token | Light | Dark | Uso |
|---|---|---|---|
| `background` | `#FFFFFF` | `#071A33` | Canvas |
| `surface` | `#F8F9FA` | `#0A274C` | Cards |
| `surfaceVariant` | `#E9ECEF` | `#1A3354` | Dividers, subtle |
| `onBackground` | `#071A33` | `#FFFFFF` | Text on bg |
| `onSurface` | `#071A33` | `#FFFFFF` | Text on card |
| `border` | `#DEE2E6` | `#2A4466` | Borders default |
| `muted` | `#6C757D` | `#8FA4BF` | Secondary text |

### 1.3 ThemeData (Dart)

```dart
// app/lib/core/theme/master_sindico_theme.dart
final lightTheme = ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.light(
    primary: Color(0xFFC69C3F),         // primaryGold
    onPrimary: Color(0xFF071A33),
    secondary: Color(0xFF071A33),        // navy
    onSecondary: Color(0xFFFFFFFF),
    surface: Color(0xFFFFFFFF),
    onSurface: Color(0xFF071A33),
    error: Color(0xFFE74C3C),
    onError: Color(0xFFFFFFFF),
  ),
  textTheme: _textTheme,
  // ...
);

final darkTheme = ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.dark(
    primary: Color(0xFFC69C3F),          // gold mantém-se em dark
    onPrimary: Color(0xFF071A33),
    secondary: Color(0xFFFFCC6A),        // gold light
    onSecondary: Color(0xFF071A33),
    surface: Color(0xFF0A274C),
    onSurface: Color(0xFFFFFFFF),
    error: Color(0xFFE74C3C),
    onError: Color(0xFFFFFFFF),
  ),
  textTheme: _textTheme.apply(
    bodyColor: Colors.white,
    displayColor: Colors.white,
  ),
);
```

---

## § 2. Tipografia

### 2.1 Famílias

- **Michroma** — brand-only (wordmark "MASTER SÍNDICO", splash screen, logo em headers). Nunca em body copy.
- **Inter** (ou **Manrope** fallback) — body text, labels, botões. Sistema sans-serif moderno, legível em mobile.
- **Roboto Mono** (fallback monospace) — códigos, IDs, campos numéricos.

### 2.2 TextTheme Flutter

```dart
final _textTheme = TextTheme(
  displayLarge: TextStyle(fontFamily: 'Michroma', fontSize: 32, fontWeight: FontWeight.bold),
  displayMedium: TextStyle(fontFamily: 'Michroma', fontSize: 24),
  displaySmall: TextStyle(fontFamily: 'Michroma', fontSize: 20),
  headlineLarge: TextStyle(fontFamily: 'Inter', fontSize: 22, fontWeight: FontWeight.w700),
  headlineMedium: TextStyle(fontFamily: 'Inter', fontSize: 18, fontWeight: FontWeight.w600),
  headlineSmall: TextStyle(fontFamily: 'Inter', fontSize: 16, fontWeight: FontWeight.w600),
  titleLarge: TextStyle(fontFamily: 'Inter', fontSize: 16, fontWeight: FontWeight.w500),
  titleMedium: TextStyle(fontFamily: 'Inter', fontSize: 14, fontWeight: FontWeight.w500),
  bodyLarge: TextStyle(fontFamily: 'Inter', fontSize: 16),
  bodyMedium: TextStyle(fontFamily: 'Inter', fontSize: 14),
  bodySmall: TextStyle(fontFamily: 'Inter', fontSize: 12),
  labelLarge: TextStyle(fontFamily: 'Inter', fontSize: 14, fontWeight: FontWeight.w600),
  labelMedium: TextStyle(fontFamily: 'Inter', fontSize: 12, fontWeight: FontWeight.w600),
  labelSmall: TextStyle(fontFamily: 'Inter', fontSize: 11),
);
```

### 2.3 Fontes bundle em `pubspec.yaml`

```yaml
flutter:
  fonts:
    - family: Michroma
      fonts:
        - asset: assets/fonts/Michroma-Regular.ttf
    - family: Inter
      fonts:
        - asset: assets/fonts/Inter-Regular.ttf
        - asset: assets/fonts/Inter-Medium.ttf
          weight: 500
        - asset: assets/fonts/Inter-SemiBold.ttf
          weight: 600
        - asset: assets/fonts/Inter-Bold.ttf
          weight: 700
```

**Alternativa**: `google_fonts` package pra download automático de Inter (evita bundle 500KB).

---

## § 3. Grid & spacing (8px base)

Mantém consistência com web:

```dart
class Spacing {
  static const double xs = 4;    // 0.5 * 8
  static const double sm = 8;    // 1 * 8
  static const double md = 16;   // 2 * 8
  static const double lg = 24;   // 3 * 8
  static const double xl = 32;   // 4 * 8
  static const double xxl = 48;  // 6 * 8
}

class Radius {
  static const double sm = 4;
  static const double md = 8;
  static const double lg = 12;
  static const double pill = 9999;
}
```

---

## § 4. Componentes catalogados (20+ equivalentes ao web)

| Componente | Flutter widget | Notas de acessibilidade |
|---|---|---|
| Botão primário | `FilledButton` custom com `primaryGold` bg | min 44×44 (WCAG), `Semantics` label |
| Botão secundário | `OutlinedButton` com navy border | idem |
| Botão destrutivo | `FilledButton` com `destructive` | confirmation dialog obrigatória pra ações irreversíveis |
| Botão icon | `IconButton` com min 48×48 dp (WCAG AAA mobile) | ⚠️ **NÃO 40×40 como web** — mobile exige 48 por guidelines Material/iOS |
| Card | `Card` com `surface` bg + elevation 2 + radius lg | `Semantics` container |
| Badge | `Chip` custom com cores de status | |
| Input | `TextFormField` com `InputDecoration` brand | label acima, erro abaixo |
| Select | `DropdownButtonFormField` | idem |
| Checkbox | `Checkbox` custom | min 24×24 tap target 48 |
| Radio | `Radio` custom | |
| Switch | `Switch` com brand gold | |
| Modal | `showModalBottomSheet` (preferido em mobile) ou `showDialog` | backdrop 0.5, drag handle |
| Bottom sheet | `showModalBottomSheet` com drag handle | max height 90% |
| Snackbar | `ScaffoldMessenger.showSnackBar` com `SnackBarBehavior.floating` | duration 4s, action label |
| AppBar | `AppBar` com brand | height 56 padrão |
| Tab | `TabBar` com indicator gold | |
| Divider | `Divider` com `border` color | |
| CircularProgress | `CircularProgressIndicator` gold | |
| LinearProgress | `LinearProgressIndicator` gold | |
| EmptyState | custom widget — ícone 64×64 + texto 14px + action button | |
| SkeletonLoader | `shimmer` package ou custom shimmer | |

---

## § 5. Ícones

Sugestão consistente com web: **Lucide Icons** (via `lucide_icons` package) ou **Phosphor** (via `phosphor_flutter`). Material Icons como fallback.

```dart
import 'package:lucide_icons/lucide_icons.dart';

Icon(LucideIcons.shield, size: 24, color: Theme.of(context).colorScheme.primary);
```

---

## § 6. Dark mode

- Padrão Light
- Switch automático via `MediaQuery.platformBrightnessOf(context)` 
- Storage de preferência em `shared_preferences` ou `hive` (quando migrar)
- Gold permanece gold em dark (alta visibilidade); backgrounds invertem

---

## § 7. Acessibilidade (WCAG 2.1 AA — platform-specific)

- Contraste AA validado automaticamente via `ColorScheme` Material 3
- Tap targets ≥ 48×48 dp (Android) / 44×44 pt (iOS) — **mobile é mais rigoroso que web**
- `Semantics` em todo widget interativo com label descritiva pt-BR
- Font scaling: respeitar `MediaQuery.textScaleFactor` (usuário aumenta fonte nas config do device)
- Reduce motion: respeitar `MediaQuery.disableAnimations`
- Screen readers: TalkBack (Android) + VoiceOver (iOS) testados em screens M1

---

## § 8. Animações

- Duração padrão: 200ms (short), 300ms (medium), 500ms (long)
- Curve: `Curves.easeOut` (enter), `Curves.easeIn` (exit), `Curves.easeInOut` (transition)
- Page transitions: `PageTransitionsBuilder` customizado (slide horizontal por default iOS; fade+slide Android)

---

## § 9. Platform-specific guidelines

### 9.1 iOS
- Botão primário pode usar style iOS `CupertinoButton` em certas telas (ex: settings sheets)
- Back gesture swipe-from-edge funciona nativo via `go_router`
- SafeArea obrigatório em todo Scaffold root

### 9.2 Android
- Material 3 padrão
- Back button hardware trata via `WillPopScope` / `PopScope` (Flutter 3.12+)
- Edge-to-edge display (status bar transparente com tema)

---

## § 10. Gaps e dívidas

| ID | Título | Severidade | Ação |
|---|---|---|---|
| **DT-MB-008** | Paleta atual `#6C63FF` (default Flutter deepPurple) — substituir por Manual IV (`#C69C3F` gold + `#071A33` navy) | alta | Sprint 10 |
| DT-MB-009 | Fontes Michroma + Inter não bundled em `assets/fonts/` | média | Sprint 10 |
| DT-MB-010 | ThemeData Material 3 não aplicado ainda em `app/lib/core/theme/` | média | Sprint 10 |
| DT-MB-011 | Dark mode toggle não implementado em settings | baixa | Sprint 11 |
| DT-MB-012 | Tap targets podem estar < 48dp — auditar widgets customizados | média | Sprint 10 |
| DT-MB-013 | Semantics labels faltando em várias telas assembly já implementadas | média | Sprint 10 |
| DT-MB-014 | TextScaleFactor não testado — risco de quebra em configurações acessíveis | baixa | Sprint 11 |

---

## § 11. Dependências com outros artefatos

- [[../web/design-system]] — fonte canônica cross-platform (mesmas cores, tipografia, componentes)
- [[../../00-product/sub-produtos/_moc]] — produtos consumidores
- [[../../08-security/lgpd]] — masking de PII em campos display
- [[../../STATE]] — DT-010 (criação deste arquivo) + DT-MB-008..014

---

## § 12. Referências

- Manual de Identidade Visual Master Síndico (PDF, Content/contents/)
- [Material 3 color system](https://m3.material.io/styles/color/overview)
- [WCAG 2.1 — Mobile accessibility best practices](https://www.w3.org/WAI/standards-guidelines/mobile/)
- [iOS Human Interface Guidelines — Color](https://developer.apple.com/design/human-interface-guidelines/color)
- [Flutter Adaptive & Platform-Specific Code](https://docs.flutter.dev/ui/adaptive-responsive)
- [[../web/design-system]] — cross-ref token values (OKLCH canônico)
