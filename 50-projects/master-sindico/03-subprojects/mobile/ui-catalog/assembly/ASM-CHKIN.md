---
title: ASM-CHKIN — Check-In Assembleia (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, assembly]
project: master-sindico
persona: morador|sindico
screen_id: ASM-CHKIN
sub_produto: assembleia-live
plan_requirement: premium
status: specification
stack: mobile
created: 2026-04-24
---

# ASM-CHKIN — Check-In Assembleia (Mobile)

## Finalidade

Check-in presencial/remoto na assembleia. Duas modalidades: QR (scan) ou código numérico 6 dígitos.

## Persona e ABAC

- Morador ou síndico com membership ativo.
- `assembly.check_in` ABAC + flag `assembly.{id}.state=open`.

## Fluxo mobile

1. Trigger: push `assembly.{id}.opened` ou card home "Check-in disponível".
2. Duas tabs: [QR] [Código].
3. QR: permissão câmera on-demand → scanner ativo → ler payload → submit.
4. Código: 6 inputs digitais (auto-advance) → submit.
5. Sucesso → navega `ASM-VOTO`.

## Widget Flutter principal

```dart
class AssemblyCheckInPage extends StatefulWidget {
  State<AssemblyCheckInPage> createState() => _State();
}

class _State extends State<AssemblyCheckInPage> {
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 2,
      child: Scaffold(
        appBar: AppBar(
          title: Text('Check-in'),
          bottom: TabBar(tabs: [Tab(text: 'QR'), Tab(text: 'Código')]),
        ),
        body: TabBarView(children: [_QrScanner(), _NumericInput()]),
      ),
    );
  }
}
```

- `mobile_scanner` package.
- `_NumericInput`: 6 `TextField` com `FocusNode` e keyboard `number`.

## Platform-specific notes

- **Camera permission**: `permission_handler` on-demand; rationale text claro.
- iOS: `NSCameraUsageDescription` em Info.plist.
- Android: `<uses-permission android:name="android.permission.CAMERA"/>`.
- **Back button** durante submit = ignorado.

## Estados

- PermissionAsking / Scanning / Submitting / Success / Expired / AlreadyCheckedIn / Error.

## Offline behavior

- **Online obrigatório**. Banner "Check-in exige conexão".

## Push notification trigger

- `assembly.{id}.opened` → canal `ms_assembly_live` (time-sensitive).

## Integridade D-050

- M3 gated: `hooked|jailbroken` → bloquear (modal "dispositivo não seguro para votar").

## Integração backend

| Ação | Endpoint | Método |
|---|---|---|
| Submit check-in | `/api/v1/assembly/{id}/check-in` | POST |

Payload: `{ nonce|code, device_fingerprint }`; idempotente.

## Gaps/ressalvas

- QR payload expira em 60s server-side.
- Código numérico exibido na tela do auditório (projeção).

## Links

- [[_moc]] · [[ASM-VOTO]]
- [[../../requirements/assembly-check-in]]
