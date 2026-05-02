---
title: ASM-CHKIN — Check-in (App / QR Code / Terminal)
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: morador
secondary_persona: administradora
category: ASM
screen_id: ASM-CHKIN
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-CHKIN — Check-in na Assembleia

## Finalidade

Três modalidades simultâneas de check-in no dia da assembleia (§5.1): pelo app do morador, por QR code único da assembleia, ou via terminal de apoio na entrada (tablet/notebook operado pela administradora ou funcionário).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §5.1 (Formas de check-in) e §5.2 (Voto presencial assistido)
- [[../../../../04-requirements/functional/assembly]] Req 58.1
- [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]]

## Persona e ABAC

- **Primária morador**: self check-in via app.
- **Primária terminal**: `administradora` ou operador designado pelo síndico.
- **Secundária**: `syndic` (pode fazer check-in assistido).
- ABAC: `assembly.checkin.self` vs `assembly.checkin.assisted`.

## Fluxo da tela

**Modo App (morador)**:
1. Card da assembleia com CTA **[Fazer check-in]**.
2. Se geolocalização disponível + opcional, valida proximidade do local físico.
3. Registra check-in e mostra "Você está presente — aguarde início".

**Modo QR (morador)**:
1. Morador abre câmera do app / scanner web.
2. Escaneia QR único da assembleia → identifica via token.
3. Mesmo fluxo do app (confirma identidade).

**Modo Terminal (operador)**:
1. Terminal exibe formulário: **CPF · bloco · unidade**.
2. Botão **[Confirmar presença]**.
3. Sistema registra presença digitalmente + marca morador como **voto presencial assistido** (§5.2).

## Componentes

- Botão de check-in grande (acessibilidade).
- Scanner QR (@zxing/browser).
- Form terminal com inputs grandes e mascarados.
- Timeline mostrando hora de entrada.

## Estados (loading/empty/error/success)

- **Error**: CPF não encontrado · unidade inadimplente não liberada · morador já fez check-in.
- **Success**: badge "Presente" + som/vibração.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Check-in self | `/api/v1/assemblies/:id/checkin` | POST | `{mode:"app"|"qr", token?}` | `{checkin_id, at}` |
| Check-in assistido | `/api/v1/assemblies/:id/checkin/assisted` | POST | `{cpf, block, unit, operator_id}` | `{checkin_id, at, mode:"terminal"}` |
| Status | `/api/v1/assemblies/:id/attendance` | GET | — | `{present:n, list}` |

## Regras de negócio críticas

- QR code único por assembleia (§4.7) — gerado em [[ASM-PUB]].
- Voto presencial assistido (§5.2): check-in via terminal vincula morador ao **voto lançado pelo operador**; registra `operator_id`, `timestamp`, `vote_mode=presencial_assistido`.
- **Abandono** (§5.3): se morador sai após check-in, status muda para `ausente_momentaneo` ou `desconectado`; votos já computados permanecem válidos; itens ainda não votados entram como **abstenção**.
- Audit trail por evento (append-only).
- Latência WebSocket < 200ms P95 para sincronizar painel operacional em tempo real (§11 NFR).

## Ligações

- Origem: [[ASM-TERM]] (morador aceita termos e chega aqui).
- Destino: [[ASM-PAINEL-SIND]] (painel síndico vê lista), [[ASM-VOTO]].
- Relacionados: [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- Geolocalização: opcional M1 (UX mobile iOS/Android varia); backlog M2 para anti-fraude.
- Reconhecimento facial em terminal: **fora do escopo** — custo + LGPD sensível.

## Links

- [[_moc]]
- [[ASM-PAINEL-SIND]]
- [[ASM-VOTO]]
