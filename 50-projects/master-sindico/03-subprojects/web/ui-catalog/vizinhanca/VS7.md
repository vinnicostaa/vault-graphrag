---
title: "VS7 — Criar Benefício Exclusivo para Condomínio"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico, exclusivo]
project: master-sindico
persona: sindico
category: VS
screen_id: VS7
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS7 — Criar Benefício Exclusivo para Condomínio

## Finalidade
Síndico cria, em **parceria com um estabelecimento já cadastrado**, uma promoção **exclusiva** visível **apenas aos moradores do condomínio**. Empresa NÃO vê. Síndico de outros condomínios NÃO vê. Outros moradores NÃO veem.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS7

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.exclusive_benefit.create`

## Campos
| Campo | Tipo | Notas |
|---|---|---|
| Selecionar parceiro | Autocomplete | Lista parceiros cadastrados (preferencialmente do bairro) |
| Título da promoção | Texto | — |
| Descrição | Textarea | — |
| Tipo | Select 1 dos 7 | `desconto_percentual | desconto_fixo | produto_gratuito | combo_promocional | avaliacao_gratuita | brinde | beneficio_exclusivo` |
| Validade | Range datas | Início + Fim |
| Termos | Textarea | Opcional |

## Componentes
- `PartnerAutocomplete`
- `PromotionTypeSelect`
- `DateRangePicker`
- `LegalNote` ("Esta promoção será visível APENAS aos moradores do seu condomínio.")
- `PrimaryButton` ("Publicar benefício exclusivo")

## Estados
- **Inicial**: form vazio.
- **Validating**: inline.
- **Submitting**: spinner.
- **Success**: toast "Benefício exclusivo publicado!" + redireciona [[VS8]] + push aos moradores do condomínio.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método | Payload |
|---|---|---|---|
| Listar parceiros disponíveis | `/api/v1/neighborhood/partners?available_for_exclusive=true` | GET | — |
| Criar exclusivo | `/api/v1/neighborhood/exclusive-benefits` | POST | `{ partner_id, title, description, type, start, end, terms? }` |

## Regras de negócio críticas
- **Visibilidade rígida**: ABAC `commercial.exclusive_benefit.view` exige `unit.condominium_id == benefit.condominium_id`.
- **Empresa NÃO vê** promoções exclusivas (regra dura — derivada para [[VE4]]).
- **Outros síndicos NÃO veem**.
- **Outros moradores (de outros condos) NÃO veem**.
- Evento `neighborhood.exclusive_benefit.created` publicado.
- Push notification opcional aos moradores do condomínio.
- Parceiro precisa estar cadastrado (sem CNPJ ok — Decision 12).

## Ligações
- [[VS1]] · [[VS8]] · [[VM5]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem decisão sobre quem pode encerrar o exclusivo (síndico que criou + parceiro? só síndico?).
- Sem validação de "parceiro consente em participar" — fluxo opt-in pendente (sugerido Connect Me Req 19.3).
- Limite de promoções exclusivas simultâneas por condomínio sem ADR.
