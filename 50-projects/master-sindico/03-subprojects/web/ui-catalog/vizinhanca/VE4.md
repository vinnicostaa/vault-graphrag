---
title: "VE4 — Detalhe da Promoção (Empresa) — Apenas Abertas"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, empresa]
project: master-sindico
persona: empresa
category: VE
screen_id: VE4
sub_produto: vizinhanca
plan_requirement: plus
status: specification
stack: web
created: 2026-04-23
---

# VE4 — Detalhe da Promoção (Empresa)

## Finalidade
Detalhe de uma promoção **aberta do bairro** (única visibilidade da empresa). Botão "Ativar promoção" → [[VE5]].

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VE4

## Persona e ABAC
- **Persona**: Empresa (role `enterprise`)
- **Plan tier**: `plus`
- **ABAC action**: `commercial.neighborhood_benefit.read.open`

## Informações exibidas
- Título · Descrição · Tipo · Validade · Termos
- Estabelecimento (link [[VE3]])

## Botão
- **Ativar promoção** → [[VE5]]

## Componentes
- `PromotionHero`
- `PromotionDescription`
- `TermsBlock`
- `ActivateButton`

## Estados
- **Loading**: skeleton.
- **Encerrada**: badge + botão desabilitado.
- **Lock 4h ativo** (já ativou): aviso.
- **Tentativa de URL exclusiva** (deep-link): 403 + redirect [[VE2]] com mensagem "Promoção exclusiva para moradores."
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe promoção | `/api/v1/neighborhood/promotions/{id}?audience=enterprise` | GET |
| Iniciar ativação | `/api/v1/neighborhood/promotions/{id}/activate` | POST |

## Regras de negócio críticas
- **Empresa NÃO acessa** promoção `exclusiva_condominio` — ABAC retorna 403 mesmo com URL direta.
- Lock de 4h se aplica.
- Cupom: formato D-064 (ID_LOJA(5)+SEQ(3)+PALAVRA(5)).
- Métrica `actor_type=empresa`.

## Ligações
- [[VE3]] · [[VE5]] · [[VE6]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Mensagem de 403 sem cópia oficial.
- Sem decisão sobre log de "tentativa empresa de acessar exclusiva" (sugerido — anti-bypass).
