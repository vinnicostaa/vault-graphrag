---
title: ADR-0002 — Usar actions/controls em vez de protocol para input local
type: adr
tags:
  - adr
  - rustcraft
  - input
  - actions
  - naming
status: accepted
date: '2026-05-27'
created: '2026-05-27'
updated: '2026-05-27'
---
# ADR-0002 — Usar actions/controls em vez de protocol para input local

## Status

accepted — 2026-05-27

## Contexto

Durante a pesquisa de arquitetura, padrões de projetos como COSMIC trouxeram a ideia de separar input bruto de ações de alto nível. A ideia é boa, mas o nome `protocol` carrega semântica de compositor, IPC ou protocolo de rede.

Em um jogo local, o termo mais comum é `actions`, especialmente quando inspirado por sistemas como Unity Input Actions.

## Decisão

Não usar `protocol` para representar input semântico local.

Usar:

- `input/actions.rs` para ações semânticas (`PlayerAction`);
- `input/bindings.rs` para teclado/gamepad → ação;
- `controls` quando o foco for configuração/remapeamento;
- `player_controller` quando o foco for aplicar ações ao player.

Reservar `protocol` para casos reais de protocolo:

- multiplayer/network wire protocol;
- save-game format;
- asset pipeline format;
- comunicação entre processos/ferramentas.

## Consequências positivas

- Vocabulário mais alinhado com jogos.
- Menos confusão com arquitetura de compositor/OS.
- Melhor encaixe para gamepad/remapping/replay no futuro.

## Consequências negativas

- Se multiplayer surgir, um módulo `protocol` pode existir depois, mas com outro significado.
- Pode ser necessário renomear código já criado.

## Referências conceituais

- Unity Input System: Actions separam significado lógico do controle físico.
- Unreal: Controller/Pawn separam decisão de controle da entidade controlada.
- Bevy: systems/resources/components permitem separar coleta de input e aplicação de gameplay.
