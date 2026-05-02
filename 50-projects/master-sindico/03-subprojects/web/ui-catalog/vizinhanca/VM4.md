---
title: "VM4 — Ativar Promoção (Morador) — Cupom"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, morador, cupom]
project: master-sindico
persona: morador
category: VM
screen_id: VM4
sub_produto: vizinhanca
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# VM4 — Ativar Promoção (Morador)

## Finalidade
Tela de ativação que gera cupom para o morador apresentar no estabelecimento. Aplica regras canônicas: cupom D-064 + lock 4h + métrica `actor_type=morador`.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VM4
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM4

## Mensagem canônica
> "Apresente esta tela no estabelecimento para ativar seu benefício."

## Persona e ABAC
- **Persona**: Morador (role `resident`)
- **Plan tier**: `base`
- **ABAC action**: `commercial.neighborhood_benefit.activate`

## Conteúdo exibido
- Nome do estabelecimento
- Nome da promoção
- Validade
- **Cupom** D-064 (ID_LOJA(5)+SEQ(3)+PALAVRA(5))
- QR opcional
- Timer "Lock 4h"

## Botão canônico
- **Mostrar promoção** (apenas exibe — usado quando cliente pede para mostrar de novo no caixa)

## Componentes
- `CouponBigDisplay`
- `CountdownTimer`
- `EstablishmentInfo`
- `ShowAgainButton`
- `BackToHistory` (link [[VM7]])

## Estados
- **Generating**: spinner.
- **Active**: cupom + timer + texto institucional.
- **Lock 4h ativo**: cupom anterior + aviso.
- **Promoção encerrada**: redirect [[VM3]].
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Gerar/recuperar cupom | `/api/v1/neighborhood/promotions/{id}/coupon` | POST (actor_type=resident) |

## Regras de negócio críticas
- A ativação **gera métrica para o parceiro** (regra explícita no PDF).
- Lock 4h: revalidação só após 4h por (morador × promoção).
- Sistema de Cupons D-064.
- Evento `neighborhood.promotion.activated` com `actor_type=morador`.
- Linha do tempo do morador? — pendência.

## Ligações
- [[VM3]] · [[VM7]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Política de "cupom expira em quanto tempo" sem ADR (interpretação atual: validade da promoção).
- QR code adoption opcional MVP.
