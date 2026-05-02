---
title: ONB-M2 — Buscar Condomínio (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, morador]
project: master-sindico
persona: morador
screen_id: ONB-M2
sub_produto: governanca-institucional
plan_requirement: any
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-M2 — Buscar Condomínio (Mobile)

## Finalidade

Primeira etapa do onboarding morador. Busca o condomínio por CEP + número da rua.

## Persona e ABAC

- Morador pós-login sem vínculo.
- `onboarding.resident.search`.

## Fluxo mobile

1. Input CEP (mask `00000-000`).
2. Autocomplete rua via ViaCEP ou provider interno.
3. Input número do prédio.
4. `GET /institutional/condominiums/search?cep=X&number=N` retorna lista.
5. Tap na opção → persiste `selected_condominium_id` em draft → próximo.

## Widget Flutter principal

```dart
class OnbM2Page extends StatelessWidget {
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Buscar condomínio')),
      body: BlocBuilder<OnbM2Cubit, OnbM2State>(
        builder: (_, s) => Column(
          children: [
            _CepField(),
            _NumberField(),
            _ResultsList(s.results),
          ],
        ),
      ),
    );
  }
}
```

- Mask CEP via `mask_text_input_formatter`.

## Platform-specific notes

- Keyboard `TextInputType.number` para CEP e número.
- iOS: toolbar "Próximo" entre campos.
- Android: imeAction `next`.

## Estados

- Idle / Searching / Results / Empty ("Não encontramos. Solicite acesso ao síndico.") / Error / Offline.

## Offline behavior

- Input livre; busca fica pendente até conectar.
- Se condomínio previamente buscado (cached) — sugerir.

## Push notification trigger

Nenhum.

## Integração backend

- `GET /institutional/condominiums/search`
- `PATCH /onboarding/sessions/me` (auto-save).

## Links

- [[_moc]] · [[ONB-M3]]
