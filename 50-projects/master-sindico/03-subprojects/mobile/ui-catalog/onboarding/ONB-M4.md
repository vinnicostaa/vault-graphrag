---
title: ONB-M4 — Termo de Ciência (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, morador]
project: master-sindico
persona: morador
screen_id: ONB-M4
sub_produto: governanca-institucional
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-M4 — Termo de Ciência (Mobile)

## Finalidade

Aceite formal do termo (versionado, compliance C2).

## Persona e ABAC

- Morador em onboarding.

## Fluxo mobile

1. Exibe texto legal versionado (scroll).
2. Checkbox "Li e aceito" (obrigatório para avançar).
3. Próximo → ONB-M5.
4. Recusar → logout + tela final.

## Widget Flutter principal

```dart
Scaffold(
  body: Column(
    children: [
      Expanded(child: SingleChildScrollView(child: MarkdownBody(data: terms))),
      CheckboxListTile(value: accepted, onChanged: (...) {...}),
    ],
  ),
  bottomNavigationBar: _CtaRow(onCancel: logout, onAccept: next),
);
```

## Platform-specific notes

- Scroll obrigatório até o fim antes de habilitar checkbox (pattern web — opcional mobile).
- SafeArea full.

## Estados

- Idle / Accepted / Submitting / Done.

## Offline behavior

- Versão do termo cached; aceite registrado no próximo sync (enfileira).

## Push notification trigger

Nenhum.

## Gaps/ressalvas

- Versão fixa na build vs fetch: fetch preferível (`GET /compliance/terms/current`).

## Links

- [[_moc]] · [[ONB-M3]] · [[ONB-M5]]
