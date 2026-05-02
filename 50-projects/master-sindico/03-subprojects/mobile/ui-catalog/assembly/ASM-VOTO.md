---
title: ASM-VOTO — Voto em Item (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, assembly]
project: master-sindico
persona: morador|sindico
screen_id: ASM-VOTO
sub_produto: assembleia-live
plan_requirement: premium
status: specification
stack: mobile
created: 2026-04-24
---

# ASM-VOTO — Voto em Item (Mobile)

## Finalidade

Tela principal da assembleia live. Exibe item em votação, timer, 3 botões (Sim/Não/Abstenção) e resultado stream via WebSocket.

## Persona e ABAC

- Morador ou síndico com check-in confirmado + fração ideal > 0.
- `assembly.vote`.

## Fluxo mobile

1. Check-in sucedeu → navega aqui.
2. Conecta WebSocket `/ws/assembly/{id}`.
3. Aguarda evento `item.voting.opened`.
4. Renderiza item + timer + 3 botões.
5. Tap botão → `POST /vote` (idempotency key).
6. Recebe `item.voting.tally.updated` → atualiza gráfico.
7. `item.voting.closed` → tela resultado → aguarda próximo item.

## Widget Flutter principal

```dart
class AssemblyVotePage extends StatelessWidget {
  Widget build(BuildContext context) {
    return BlocBuilder<VoteBloc, VoteState>(
      builder: (_, s) => Scaffold(
        body: s.when(
          waiting: () => _WaitingNextItem(),
          itemOpen: (item, tally, mySubmitted) =>
              _ItemBody(item, tally, mySubmitted),
          itemClosed: (item, result) => _ResultBody(item, result),
          disconnected: () => _Reconnecting(),
        ),
      ),
    );
  }
}
```

- `_ItemBody`: título item + countdown + 3 `FilledButton.tonal` altura 64dp + barra progresso tally.
- `droppable()` no `SubmitVote` event.

## Platform-specific notes

- **Screen wake lock**: ativar `Wakelock.enable()` enquanto em sessão (evita bloqueio).
- **Modo não perturbe**: iOS ignorar silence mode (push `time-sensitive`).
- Back button desabilitado durante voto aberto.

## Estados

- Connecting / Waiting / ItemOpen / Submitting / Submitted / Closed(result) / Disconnected / Expired.

## Offline behavior

- **Proibido offline**. Sem rede → modal "Sem conexão — tentando reconectar..." + reconnect automático.

## Push notification trigger

- `assembly.{id}.item.opened` (quando app minimizado) → notificação crítica com deep-link de volta.

## Integridade D-050 (M3 gated)

- `integrity=hooked|jailbroken` + modo gated → bloquear voto + exibir modal "Use outro dispositivo para votar".
- Ciência ainda permitida.

## WebSocket events

- `item.voting.opened { item_id, question, opened_at, timeout_seconds }`
- `item.voting.tally.updated { yes, no, abstain, fracao_total }`
- `item.voting.closed { result }`
- `speaker_queue.updated { queue }`

## Gaps/ressalvas

- Voto secreto: após cast, esconder próprio voto (flag `is_secret` do item).
- Empate: server decide via voto minerva do síndico.

## Links

- [[_moc]] · [[ASM-CHKIN]] · [[ASM-FILA-FALA]]
- [[../../requirements/assembly-vote]]
- [[../../../STATE#D-070|D-070 WebSocket]] · [[../../../STATE#D-050|D-050 Integrity]]
