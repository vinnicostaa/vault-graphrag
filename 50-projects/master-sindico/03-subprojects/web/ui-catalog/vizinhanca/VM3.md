---
title: "VM3 — Detalhe da Promoção (Morador)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, morador]
project: master-sindico
persona: morador
category: VM
screen_id: VM3
sub_produto: vizinhanca
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# VM3 — Detalhe da Promoção (Morador)

## Finalidade
Detalhe da promoção (aberta ou exclusiva). Botão para ativar. Mensagem institucional canônica de pertencimento à rede.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VM3
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM3

## Mensagem institucional canônica
> "Esta promoção faz parte da rede de vizinhança da Master Síndico."

## Persona e ABAC
- **Persona**: Morador (role `resident`)
- **Plan tier**: `base`
- **ABAC action**: `commercial.neighborhood_benefit.read`

## Informações exibidas
- Título da promoção
- Descrição
- Tipo (1 dos 7)
- Validade
- Estabelecimento (link [[VM2]])

## Botão
- **Ativar promoção** → [[VM4]]

## Componentes
- `PromotionHero`
- `PromotionDescription`
- `InstitutionalNote` (mensagem fixa)
- `ActivateButton`
- `ExclusiveBadge` (se exclusiva do condomínio)

## Estados
- **Loading**: skeleton.
- **Encerrada**: badge + botão desabilitado.
- **Lock 4h ativo**: aviso.
- **Tentativa exclusiva de outro condomínio** (deep-link): 403 + redirect [[VM1]].
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe promoção | `/api/v1/neighborhood/promotions/{id}?audience=resident` | GET |

## Regras de negócio críticas
- ABAC `commercial.exclusive_benefit.view` exige `unit.condominium_id == benefit.condominium_id` para exclusivas.
- Lock 4h aplicável.
- Cupom: D-064 (ID_LOJA(5)+SEQ(3)+PALAVRA(5)).
- Métrica `actor_type=morador`.

## Ligações
- [[VM2]] · [[VM4]] · [[VM5]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Mensagem institucional precisa estar configurável (vs hardcoded).
- Tratamento visual da `ExclusiveBadge` sem design system definido.
