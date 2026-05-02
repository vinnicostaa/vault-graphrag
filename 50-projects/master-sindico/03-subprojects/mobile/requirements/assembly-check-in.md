---
title: Assembly Check-In — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, assembly, feature-req]
project: master-sindico
stack: mobile
priority: m3
status: pending
bounded_context: assembly
created: 2026-04-24
---

# Assembly Check-In — Mobile

Check-in ao vivo do morador/síndico na assembleia via app. Duas modalidades: **QR Code** (scan da mesa diretora) ou **código numérico 6 dígitos** (projetado na tela do auditório). Define quórum + habilita voto.

## Escopo M1/M2/M3

- **M1**: fora do escopo (D-versão-assembly-fora-escopo-M1 — A-FASE10-DEV-004).
- **M2**: fora do escopo.
- **M3**: fluxo completo (QR + numérico + feedback visual + reconexão).

## Telas envolvidas

- `ASM-CHKIN` (tela de check-in ativa)
- `ASM-CHKIN-QR` (scanner câmera)
- `ASM-CHKIN-NUM` (input 6 dígitos)
- `ASM-CHKIN-OK` (confirmação + redireciona para `ASM-VOTO`)
- `ASM-CHKIN-FAIL` (erro: expirado, já checked-in, fração inválida)

## Funcionais (FR-MOB-ASM-CHKIN-N)

- **FR-MOB-ASM-CHKIN-1** Trigger: tap em notification "Assembleia começou" ou card home "Check-in disponível".
- **FR-MOB-ASM-CHKIN-2** Dois caminhos em tabs: [QR] [Código].
- **FR-MOB-ASM-CHKIN-3** QR scanner via `mobile_scanner` (camera permission on-demand); lê payload `{ assembly_id, nonce, expires_at }`.
- **FR-MOB-ASM-CHKIN-4** Código numérico: 6 inputs digitais com auto-advance.
- **FR-MOB-ASM-CHKIN-5** `POST /api/v1/assembly/{id}/check-in { nonce|code, device_fingerprint }` — idempotente.
- **FR-MOB-ASM-CHKIN-6** 200 OK → Hive guarda `assembly_{id}_checkedin = true` + navega para `ASM-VOTO`.
- **FR-MOB-ASM-CHKIN-7** 409 `ALREADY_CHECKED_IN` → modal "Já registrado. Entrar na sala?" → `ASM-VOTO`.
- **FR-MOB-ASM-CHKIN-8** 422 `EXPIRED` → "Código expirou. Peça novo." + CTA refresh.
- **FR-MOB-ASM-CHKIN-9** Após check-in, conecta WebSocket `/ws/assembly/{id}` para receber eventos (voto aberto, fila fala, etc).
- **FR-MOB-ASM-CHKIN-10** Se `freeRASP integrity != ok` e D-050 em modo gated → bloquear com modal (assembleia = feature crítica).

## Não-funcionais (NFR-MOB)

- **NFR-MOB-A11Y-1** QR scanner com fallback numérico; input numérico com keyboard `TextInputType.number`.
- **NFR-MOB-SEC-1** Payload QR válido apenas 60s (nonce + expiry server-side).
- **NFR-MOB-PERF-1** Scan → feedback visual < 200ms.

## Dados locais

- **Hive `assembly_{id}`**: `checked_in_at`, `fracao_ideal` (cached para exibir).
- **Secure Storage**: nenhum específico (reusa device_fingerprint).
- **Invalidação**: após encerramento da assembleia (push `assembly.closed`) → limpar cache.

## Integração backend

- `GET /api/v1/assembly/active/me` — retorna assembleia em andamento onde usuário pode votar.
- `POST /api/v1/assembly/{id}/check-in`
- WebSocket `wss://api.mastersindico.com.br/ws/assembly/{id}?token=...`

## Padrões Flutter aplicáveis

- `CheckInBloc` + states freezed `Idle / Scanning / Submitting / Success / Failure(reason)`.
- `bloc_concurrency.droppable()` no `SubmitRequested`.
- `mobile_scanner` package com `MobileScannerController`.
- Permission handling via `permission_handler`.

## Gaps/decisões abertas

- **Q-MOB-ASM-CHKIN-01** Check-in com GPS + proximidade opcional? Assumido **não** em M3 (QR + código suficientes).
- **Q-MOB-ASM-CHKIN-02** Se morador não tem unidade (inquilino não cadastrado) → fração ideal = 0, pode observar mas não votar; UX a definir.
- **Q-MOB-ASM-CHKIN-03** Assembleia híbrida (presencial + remoto): check-in remoto usa mesmo fluxo, sem scan físico.

## Links

- [[../../../_moc]]
- [[../../../00-product/sub-produtos/05-assembleia-live]]
- [[assembly-vote]]
- [[../../../STATE#D-050|D-050 Integrity]]
- [[../../../AUDIT#A-FASE10-DEV-004|A-FASE10-DEV-004 Assembly fora M1]]
