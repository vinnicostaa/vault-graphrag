---
title: ASM-PROC-CAD — Cadastrar Procuração (Mobile, Subset)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, assembly]
project: master-sindico
persona: morador
screen_id: ASM-PROC-CAD
sub_produto: assembleia-live
plan_requirement: premium
status: specification
stack: mobile
created: 2026-04-24
---

# ASM-PROC-CAD — Cadastrar Procuração (Mobile, Subset)

## Finalidade

Subset mobile do fluxo de procuração (Q28 aberta): outorgar voto a terceiro. **Subset**: apenas cadastro simples pré-assembleia com poderes default. Poderes customizados + validação jurídica avançada = web.

## Persona e ABAC

- Morador outorgante.
- `assembly.proxy.grant.self`.

## Fluxo mobile

1. Entrada via card "Procuração" em home assembleia futura.
2. Busca outorgado: CPF/email (usuário deve ser cadastrado no condomínio).
3. Seleciona assembleia específica OU "todas de 2026".
4. Lê poderes default (fixos M3).
5. Biometria (local_auth) para assinatura digital.
6. `POST /assembly/{id}/proxies { grantee_id, scope }`.
7. Recibo gerado + PDF para salvar.

## Widget Flutter principal

```dart
class ProcuracaoCadPage extends StatelessWidget {
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Cadastrar procuração')),
      body: Stepper(
        steps: [
          Step(title: Text('Outorgado')),
          Step(title: Text('Escopo')),
          Step(title: Text('Assinatura biométrica')),
        ],
      ),
    );
  }
}
```

## Platform-specific notes

- Biometria obrigatória: `local_auth.authenticate(localizedReason: '...')`.
- iOS: FaceID/TouchID.
- Android: BiometricPrompt.

## Estados

- Idle / Searching grantee / Reviewing / Authenticating / Submitting / Done / Error.

## Offline behavior

- Online obrigatório (assinatura server-side timestamp).

## Push notification trigger

- Outorgado recebe `proxy.granted.{user_id}`.

## Gaps/ressalvas

- **Q28**: escopo de poderes (substabelecer? recusar itens específicos?) definição aberta.
- Revogação mobile: M3 básica; completa = web.

## Links

- [[_moc]] · [[ASM-VOTO]]
- [[../../../AUDIT#Q28|Q28 Procuração assembleia Pro]]
