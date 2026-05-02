---
title: "VZ3 — Criar Promoção (Parceiro)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, parceiro, promocao]
project: master-sindico
persona: parceiro
category: VZ
screen_id: VZ3
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VZ3 — Criar Promoção

## Finalidade
Form para o parceiro criar uma promoção. Define título, descrição, **tipo** (1 dos 7), validade e **segmentação obrigatória** (aberta_bairro OU exclusiva_condominio).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VZ3
- `vault/90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança.md` §TELA VZ3

## Mensagem institucional canônica
> "Ofereça um benefício especial para quem vive e trabalha na sua região."

## Persona e ABAC
- **Persona**: Parceiro (role `local_company`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_benefit.create`

## Campos
| Campo | Tipo |
|---|---|
| Título da promoção | Texto |
| Descrição | Textarea |
| **Tipo** (1 dos 7) | Select |
| Data de início | Data |
| Data de término | Data |
| Termos | Textarea |

## 7 Tipos canônicos
- `desconto_percentual`
- `desconto_fixo`
- `produto_gratuito`
- `combo_promocional`
- `avaliacao_gratuita`
- `brinde`
- `beneficio_exclusivo`

## Segmentação OBRIGATÓRIA (literal — PDF)
**Tipo de alcance** (radio):
- **Opção 1: Promoção aberta para o bairro** — exibida para moradores + síndicos + empresas
- **Opção 2: Promoção exclusiva para condomínio** — campo extra `Selecionar condomínio` (lista de condomínios cadastrados)

## Componentes
- `PromotionForm`
- `PromotionTypeSelect`
- `DateRangePicker`
- `ScopeRadio` (com sub-form condicional)
- `CondominiumAutocomplete` (revelado se exclusiva)
- `PrimaryButton` ("Publicar promoção")

## Estados
- **Inicial**: form vazio.
- **Validating**: inline.
- **Submitting**: spinner.
- **Success**: redireciona [[VZ4]] + toast.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar condomínios | `/api/v1/condominiums?searchable=true` | GET |
| Criar promoção | `/api/v1/neighborhood/promotions` | POST |

## Regras de negócio críticas
- **Segmentação obrigatória** (`scope: aberta_bairro | exclusiva_condominio`).
- Se exclusiva: aparece **apenas para moradores daquele condomínio** (cross com [[VM5]]).
- Se aberta: visível a moradores + síndicos + empresas da região.
- Tipo `beneficio_exclusivo` é tipo distinto de scope `exclusiva_condominio` (atenção — não confundir).
- Evento `neighborhood.benefit.created` publicado.
- Parceiro NÃO publica comunicados em condomínios — apenas promoções.

## Ligações
- [[VZ2]] · [[VZ4]] · [[VS7]] (síndico cria exclusiva paralelo) · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem decisão sobre opt-in do síndico para exclusiva criada por parceiro (parceiro escolhe livremente?).
- Validação de "data início < data término" + "data início >= hoje" sem regra dura.
- Sem limite de promoções simultâneas por parceiro.
