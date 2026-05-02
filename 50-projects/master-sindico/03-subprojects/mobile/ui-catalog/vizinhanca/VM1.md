---
title: VM1 — Feed Vizinhança (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, vizinhanca, morador]
project: master-sindico
persona: morador
screen_id: VM1
sub_produto: vizinhanca
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# VM1 — Feed Vizinhança (Mobile)

## Finalidade

Feed de ofertas e serviços de comerciantes próximos. Ordenação por distância (se geo autorizada) ou relevância.

## Persona e ABAC

- Morador, `vizinhanca.feed.read`.

## Fluxo mobile

1. Primeira entrada no módulo → verificar permissão localização → VM7 se negada.
2. `GET /vizinhanca/feed?lat&lng&radius&category` paginado.
3. Lista vertical com thumb + nome + distância + desconto.
4. Filter FAB → VM6.
5. Pull-to-refresh.

## Widget Flutter principal

```dart
class VizinhancaFeedPage extends StatelessWidget {
  Widget build(BuildContext context) {
    return BlocBuilder<VizinhancaFeedBloc, VizinhancaFeedState>(
      builder: (_, s) => Scaffold(
        appBar: AppBar(title: Text('Vizinhança')),
        floatingActionButton: FilterFab(),
        body: RefreshIndicator(onRefresh: (...) async {...}, child: ListView.builder(...)),
      ),
    );
  }
}
```

- `OfferTile(offer)` com thumb `cached_network_image`.

## Platform-specific notes

- **Geolocalização** `geolocator` + permission whileInUse.
- iOS: `NSLocationWhenInUseUsageDescription`.
- Android: `ACCESS_FINE_LOCATION`.

## Estados

- Loading / Loaded / LocationDenied (degrada para ordenação por relevância) / Empty / Error / Offline.

## Offline behavior

- Cache Hive `vizinhanca_feed_{cond}` 10min.

## Push notification trigger

- `vizinhanca.offer.new.{cond}` canal `ms_vizinhanca` (opt-in).

## Gaps/ressalvas

- Radius default a definir com comercial.

## Links

- [[_moc]] · [[VM2]] · [[VM6]] · [[VM7]]
- [[../../requirements/vizinhanca-mobile]]
