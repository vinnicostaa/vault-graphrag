---
title: "VS6 — Indicar Parceiro Local"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, sindico]
project: master-sindico
persona: sindico
category: VS
screen_id: VS6
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VS6 — Indicar Parceiro Local

## Finalidade
Síndico convida um estabelecimento ainda **não cadastrado** na plataforma a se tornar Parceiro da Vizinhança. Sistema envia link de cadastro (entry para [[VZ1]]).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VS6

## Persona e ABAC
- **Persona**: Síndico (role `syndic`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_partner.invite`

## Campos
| Campo | Tipo | Obrigatório |
|---|---|---|
| Nome do estabelecimento | Texto | Sim |
| Categoria | Select (38+ canônicas) | Sim |
| Telefone / WhatsApp | Texto + máscara | Sim (pelo menos 1) |
| Endereço | Texto | Sim |
| Observação | Textarea | Não |

## Componentes
- `InviteForm`
- `CategoryAutocomplete`
- `PrimaryButton` ("Enviar convite")
- `RecentInvitesPreview` (link para [[VS9]])

## Estados
- **Inicial**: form vazio.
- **Validating**: validação inline.
- **Submitting**: spinner.
- **Success**: toast "Convite enviado para {telefone}" + reset do form + atualiza [[VS9]].
- **Error**: toast retry; preserva campos.

## Integração com backend
| Ação | Endpoint (inferido) | Método | Payload |
|---|---|---|---|
| Enviar convite | `/api/v1/neighborhood/partners/invite` | POST | `{ name, category, phone?, whatsapp?, address, observation? }` |

## Regras de negócio críticas
- Sistema **gera link único** (signed URL ou token) e envia via SMS/WhatsApp (sem ADR sobre canal final).
- Link aponta para fluxo [[VZ1]] (cadastro) com pré-preenchimento dos campos enviados.
- Cada convite registra entrada em [[VS9]] com status `convite_enviado`.
- Status muda automaticamente para `parceiro_cadastrado` quando o estabelecimento completa [[VZ1]] e para `parceiro_ativo` quando publica primeira promoção.
- Sem CNPJ obrigatório no convite — basta CPF do dono (Decision 12).

## Ligações
- [[VS1]] · [[VS9]] · [[VZ1]] · [[_moc]]
- Connect Me Síndico → Parceiro: [[../../../../00-product/sub-produtos/02-connect-me]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Canal de envio do convite (SMS vs WhatsApp Business API vs email) sem ADR.
- TTL do token de cadastro pendente.
- Sem proteção anti-spam (síndico pode mandar 100 convites?).
