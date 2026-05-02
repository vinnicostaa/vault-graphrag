---
title: "VZ2 — Perfil do Parceiro"
type: ui-screen
tags: [master-sindico, ui-catalog, web, vizinhanca, parceiro]
project: master-sindico
persona: parceiro
category: VZ
screen_id: VZ2
sub_produto: vizinhanca
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# VZ2 — Perfil do Parceiro

## Finalidade
Página principal do parceiro pós-cadastro. Mostra dados do estabelecimento, seções Promoções + Métricas e CTAs principais (criar promoção, ver métricas, editar perfil).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Vizinhanca-4-jornadas.md` §VZ2
- `vault/90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança.md` §TELA VZ2

## Mensagem institucional canônica
> "Seu estabelecimento agora faz parte da rede de vizinhança da Master Síndico."

## Persona e ABAC
- **Persona**: Parceiro (role `local_company`)
- **Plan tier**: `trial`
- **ABAC action**: `commercial.neighborhood_partner.profile.read.own`
- **Caminho**: VZ1 → cadastro concluído OU login direto

## Informações exibidas
- Nome do estabelecimento
- Categoria
- Endereço
- Contato
- Descrição
- Logo (se houver)

## Seções
- **Promoções** (preview + link [[VZ4]])
- **Métricas** (preview + link [[VZ5]])

## Botões
- **Criar promoção** → [[VZ3]]
- **Ver métricas** → [[VZ5]]
- **Editar perfil** (modal ou tela dedicada — abre form de [[VZ1]] em modo edit)

## Componentes
- `PartnerProfileHero` (logo + nome + categoria)
- `InfoBlock`
- `SectionPreviewCard` × 2
- `ActionGroup`

## Estados
- **Loading**: skeleton.
- **First time**: mensagem institucional em destaque + foco no CTA "Criar promoção".
- **Populated**: previews com dados reais.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Perfil próprio | `/api/v1/neighborhood/partners/me` | GET |
| Editar perfil | `/api/v1/neighborhood/partners/me` | PATCH |

## Regras de negócio críticas
- Parceiro **só edita o próprio perfil**.
- Edição de campos críticos (categoria principal) pode requerer revisão admin (sem ADR).
- Linha do tempo do parceiro? — não definido.

## Ligações
- [[VZ1]] · [[VZ3]] · [[VZ4]] · [[VZ5]] · [[VZ6]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Sem decisão sobre revisão admin para mudanças de categoria.
- Sem flow de "ferias / pausa" do parceiro.
