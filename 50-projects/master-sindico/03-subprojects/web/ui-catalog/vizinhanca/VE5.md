---
title: "VE5 — Ativar Promoção (Empresa) — Cupom"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, empresa, cupom]
project: master-sindico
persona: empresa
category: VE
screen_id: VE5
sub_produto: vizinhanca
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# VE5 — Ativar Promoção (Empresa)

## Finalidade
Tela de ativação para a empresa. Mesma estrutura de VS5, mas registrada com `actor_type=empresa`. Apenas promoções **abertas** podem ser ativadas pela empresa.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VE5

## Persona e ABAC
- **Persona**: Empresa (role `enterprise`)
- **Plan tier**: `plus`
- **ABAC action**: `commercial.neighborhood_benefit.activate`

## Mensagem canônica
> "Apresente esta tela no estabelecimento para ativar seu benefício."

## Conteúdo exibido
- Nome do estabelecimento
- Nome da promoção
- Validade
- **Cupom** no formato `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` (D-064)
- QR opcional
- Timer de lock 4h

## Componentes
- `CouponBigDisplay`
- `CountdownTimer`
- `EstablishmentInfo`
- `BackToHistory` (link [[VE6]])

## Estados
- **Generating**: spinner.
- **Active**: cupom + timer.
- **Lock 4h ativo**: cupom anterior + aviso.
- **Promoção encerrada**: redirect [[VE4]].
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Gerar/recuperar cupom | `/api/v1/neighborhood/promotions/{id}/coupon` | POST (actor_type=enterprise via auth) |

## Regras de negócio críticas
- **Apenas promoções abertas** ativáveis pela empresa.
- **Métrica separada `actor_type=empresa`** (separação canônica de 3 contadores).
- Lock 4h aplicável.
- Sistema de Cupons D-064.
- Evento `neighborhood.promotion.activated` com `actor_type=empresa`.

## Ligações
- [[VE4]] · [[VE6]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre limite mensal de ativações por empresa (anti-abuso).
- Caso de uso real (empresa ativa mesmo cupom recorrente?) sem validação prática.
