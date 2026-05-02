---
title: ASM-CIEN — Ciência de Item (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, assembly]
project: master-sindico
persona: morador|sindico
screen_id: ASM-CIEN
sub_produto: assembleia-live
plan_requirement: premium
status: specification
stack: mobile
created: 2026-04-24
---

# ASM-CIEN — Ciência de Item (Mobile)

## Finalidade

Item não-votável (ex: informe, prestação de contas apenas para conhecimento) → morador dá ciência formal. Registra audit.

## Persona e ABAC

- Morador check-in.
- `assembly.item.acknowledge`.

## Fluxo mobile

1. Evento WebSocket `item.acknowledge.opened` → navega/exibe esta tela ou card.
2. Exibe título + descrição + botão único "Estou ciente".
3. `POST /items/{id}/acknowledge`.
4. Confirma → aguarda próximo item.

## Widget Flutter principal

`_AcknowledgeCard` dentro da `ASM-VOTO` page ou standalone.

## Platform-specific notes

- Acessibilidade: botão grande, haptic light.
- Online-only.

## Estados

- Open / Acknowledging / Acknowledged / Error.

## Offline behavior

- Online obrigatório.

## Push notification trigger

- `assembly.{id}.item.ack.opened`.

## Gaps/ressalvas

- Se morador não dá ciência em tempo hábil → decisão de política (bloquear próximos votos?) pendente.

## Links

- [[_moc]] · [[ASM-VOTO]]
