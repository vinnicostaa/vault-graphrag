---
title: "VS5 — Ativar Promoção (Síndico) — Cupom"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico, cupom]
project: master-sindico
persona: sindico
category: VS
screen_id: VS5
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS5 — Ativar Promoção (Síndico)

## Finalidade
Tela de ativação que **gera o cupom** para o síndico apresentar no estabelecimento. Mostra nome do estabelecimento, nome da promoção, validade e código do cupom. **Lock 4h** anti-abuso.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS5
- `master-sindico-v2/STATE.md` D-064 (Sistema de Cupons + lock 4h)

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_benefit.activate`

## Mensagem canônica
> "Apresente esta tela no estabelecimento para ativar seu benefício."

## Conteúdo exibido
- Nome do estabelecimento
- Nome da promoção
- Validade da promoção
- **Cupom**: código no formato canônico **`ID_LOJA(5) + SEQ(3) + PALAVRA(5)`** (ex: `LIVR1-007-VERDE`)
- QR code do cupom (opcional MVP)
- Timer "Cupom válido por 4 horas" (lock de revalidação para o mesmo síndico-promoção)

## Componentes
- `CouponBigDisplay` (código grande + QR)
- `CountdownTimer` (24h / 4h conforme regra)
- `EstablishmentInfo` (nome + endereço resumo)
- `BackToList` (link para histórico [[VS10]])

## Estados
- **Generating**: spinner curto enquanto gera código.
- **Active**: cupom visível + timer.
- **Lock 4h ativo (já ativou)**: tela mostra cupom anterior + texto "Aguarde X minutos para gerar novo cupom desta promoção."
- **Promoção encerrada**: redireciona para [[VS4]] com aviso.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Gerar/recuperar cupom | `/api/v1/neighborhood/promotions/{id}/coupon` | POST |
| Status do cupom | `/api/v1/neighborhood/coupons/{code}` | GET |

## Regras de negócio críticas
- **Sistema de Cupons D-064**: ID_LOJA(5)+SEQ(3)+PALAVRA(5).
- **Lock 4 horas**: a mesma combinação (síndico × promoção) só pode gerar cupom novamente após 4h. Anti-abuso/anti-replay.
- Ativação gera **métrica para o parceiro** com `actor_type=sindico` (separado de morador/empresa).
- Evento `neighborhood.promotion.activated` publicado em NATS com `actor_type`.
- Cupom é **público para o portador** (sem auth para validar no estabelecimento) mas log é mantido para auditoria.
- Linha do tempo do síndico recebe entrada da ativação? — sugerido sim, pendência.

## Ligações
- [[VS4]] · [[VS10]] · [[_moc]]
- D-064: `master-sindico-v2/STATE.md`

## Gaps/ressalvas
- Formato canônico de PALAVRA(5) (dicionário curado vs aleatória) sem ADR.
- Endpoint REST inferido.
- Sem decisão se cupom expira em 4h ou só revalida em 4h (interpretação atual: revalidar em 4h, exibição segue até validade da promoção).
- QR code adoption não decidida (MVP textual basta).
