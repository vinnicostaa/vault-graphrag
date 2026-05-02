---
title: "VM6 — Parceiros Indicados pelo Síndico (Morador)"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, morador]
project: master-sindico
persona: morador
category: VM
screen_id: VM6
sub_produto: vizinhanca
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# VM6 — Parceiros Indicados pelo Síndico

## Finalidade
Lista de parceiros que o síndico do condomínio recomendou explicitamente aos moradores. Diferenciados com badge "Indicado pelo Síndico".

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VM6
- `vault/90-archive/master-sindico-pdfs-cliente-originais/MÓDULO VIZINHANÇA - MORADOR.md` §TELA VM6

## Mensagem institucional canônica
> "Estabelecimentos indicados pelo seu síndico para a comunidade."

## Persona e ABAC
- **Persona**: Morador (role `resident`)
- **Plan tier**: `base`
- **ABAC action**: `commercial.neighborhood.recommended.read`
- **Caminho**: Vizinhança → Parceiros indicados

## Lista exibida (canônico)
| Coluna |
|---|
| Parceiro |
| Categoria |
| Descrição |
| Badge "Indicado pelo Síndico" |

## Componentes
- `RecommendedPartnerList`
- `SyndicBadge`
- `InstitutionalNote`
- `EmptyState`

## Estados
- **Loading**: skeleton.
- **Empty**: "Seu síndico ainda não indicou parceiros."
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar indicados | `/api/v1/neighborhood/partners/recommended?condominium_id=mine` | GET |

## Regras de negócio críticas
- Indicados aparecem aqui quando o síndico clica "Indicar para moradores" em [[VS3]] OU convida via [[VS6]] e o parceiro completa cadastro.
- Badge é **diferenciador chave** vs descobertos espontaneamente em [[VM1]].
- Cross-tenant: morador só vê indicações do próprio síndico.

## Ligações
- [[VM1]] · [[VM2]] · [[VS3]] · [[VS9]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem ADR sobre frescor da indicação (aparece para sempre? expira?).
- Sem decisão sobre permitir morador "esconder" indicações já vistas.
