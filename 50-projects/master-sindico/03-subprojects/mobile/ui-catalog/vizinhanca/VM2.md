---
title: VM2 — Detalhe da Oferta (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, vizinhanca, morador]
project: master-sindico
persona: morador
screen_id: VM2
sub_produto: vizinhanca
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# VM2 — Detalhe da Oferta (Mobile)

## Finalidade

Detalhe de uma oferta: galeria, descrição, termos, botão ativar.

## Persona e ABAC

- Morador, `vizinhanca.offer.read`.

## Fluxo mobile

1. Tap item em VM1 → navegar `/vizinhanca/offers/{id}`.
2. `GET /vizinhanca/offers/{id}`.
3. Carousel galeria + descrição markdown + termos (collapsible) + CTA "Ativar promoção" → VM3.

## Widget Flutter principal

```dart
class OfferDetailPage extends StatelessWidget {
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(slivers: [
        SliverAppBar(expandedHeight: 240, flexibleSpace: _Gallery()),
        SliverToBoxAdapter(child: _Body()),
      ]),
      bottomNavigationBar: _ActivateCta(),
    );
  }
}
```

## Platform-specific notes

- Carousel via `PageView` + indicator.
- `photo_view` para zoom.

## Estados

- Loading / Loaded / Expired / Error / Offline.

## Offline behavior

- Detail cached 7d após visualização.

## Push notification trigger

Nenhum direto.

## Links

- [[_moc]] · [[VM1]] · [[VM3]]
