---
title: ONB-S5 — Selfie com Documento (Mobile, Síndico)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, sindico]
project: master-sindico
persona: sindico
screen_id: ONB-S5
sub_produto: governanca-institucional
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-S5 — Selfie com Documento (Mobile, Síndico)

## Finalidade

Selfie com documento visível para face-matching (KYB anti-fraude).

## Persona e ABAC

- Síndico em onboarding.

## Fluxo mobile

1. Instrução visual (holding doc próximo ao rosto).
2. Camera front (`camera` package) com overlay oval + retângulo doc.
3. Captura → preview → aceitar ou refazer.
4. Upload presigned como ONB-S4.

## Widget Flutter principal

```dart
class OnbS5SelfiePage extends StatefulWidget {
  State<OnbS5SelfiePage> createState() => _State();
}

class _State extends State<OnbS5SelfiePage> with WidgetsBindingObserver {
  CameraController? _controller;
  // ...
}
```

- `camera` package, front camera, resolução medium.

## Platform-specific notes

- Permissão câmera obrigatória.
- iOS: `NSCameraUsageDescription` com texto claro.
- Android: runtime permission.

## Estados

- RequestingPermission / PermissionDenied / Ready / Capturing / Preview / Uploading / Done.

## Offline behavior

- Captura livre; upload enfileira.

## Push notification trigger

Nenhum.

## Gaps/ressalvas

- Liveness detection (piscar/sorrir) M3+.

## Links

- [[_moc]] · [[ONB-S4]] · [[ONB-S6]]
