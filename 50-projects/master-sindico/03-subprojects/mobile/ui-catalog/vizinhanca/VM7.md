---
title: VM7 — Permissão Localização (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, vizinhanca, permissions]
project: master-sindico
persona: morador
screen_id: VM7
sub_produto: vizinhanca
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# VM7 — Permissão Localização (Mobile)

## Finalidade

Tela explicativa antes de pedir permissão de localização (rationale) — aumenta taxa de aprovação.

## Persona e ABAC

- Morador primeira entrada vizinhança.

## Fluxo mobile

1. `permission_handler.checkPermission(Permission.location)`.
2. Se `denied` ou `notDetermined` → exibe VM7.
3. Explica benefício: "Para mostrar ofertas próximas da sua unidade."
4. CTA "Permitir" → chama `requestPermission`.
5. Se `granted` → VM1 com geo.
6. Se `denied` → VM1 sem geo (ordenação por relevância).
7. Se `permanentlyDenied` → modal "Abrir Configurações" + `openAppSettings()`.

## Widget Flutter principal

`LocationRationalePage` com ilustração + texto + 2 botões (Permitir / Continuar sem).

## Platform-specific notes

- **iOS 14+**: pode pedir precisão exata vs aproximada; OK para nossa uso `aproximada`.
- **Android 10+**: background location NÃO solicitada (apenas foreground).
- `NSLocationWhenInUseUsageDescription` + `ACCESS_FINE_LOCATION`.

## Estados

- Rationale / Requesting / Granted / Denied / PermanentlyDenied.

## Offline behavior

- Não aplicável (permission é local).

## Push notification trigger

Nenhum.

## Gaps/ressalvas

- Se usuário nega permanente → feed sem geo é funcional (ordenação por relevância do condomínio).

## Links

- [[_moc]] · [[VM1]]
- [[../../security]] §14 (LGPD — localização)
