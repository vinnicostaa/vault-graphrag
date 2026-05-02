---
title: "LMS1 — Entrada na Academia"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia]
project: master-sindico
persona: any
category: LMS
screen_id: LMS1
sub_produto: lms-academia
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# LMS1 — Entrada na Academia Master Síndico

## Finalidade
Hub de entrada da Academia. Apresenta as 5 áreas (Cursos, Treinamentos, Lives, Fórum, Biblioteca) + trilha recomendada por persona + atalho para "Meu progresso".

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS1
- `vault/90-archive/from-development-content-2026-04-23/contents/ARQUITETURA DO MÓDULO LMS.md`
- `vault/50-projects/master-sindico/00-product/sub-produtos/06-lms-academia.md`

## Mensagem institucional canônica
> "O conhecimento transforma gestão. A Academia Master Síndico reúne cursos, treinamentos e experiências para fortalecer o ecossistema condominial."

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador, Admin, Marketing
- **❌ BLOQUEADO**: Parceiro Vizinhança (`local_company`) — INV-067 hard ABAC + UI oculta o botão Academia.
- **Plan tier mínimo**: `trial` (entrada visível; certificado exige `plus`+)
- **ABAC action**: `content.academy.read`
- **Caminho**: Menu principal → "Academia Master Síndico"

## Seções exibidas (canônico)
- Cursos · Treinamentos · Lives · Fórum · Biblioteca

## Botões canônicos
- Explorar cursos → [[LMS2]]
- Explorar treinamentos → [[LMS6]]
- Ver agenda de lives → [[LMS8]]
- Participar do fórum → [[LMS11]]
- Acessar biblioteca → [[LMS14]]

## Componentes
- `AcademyHero` (mensagem institucional + tagline)
- `AreaCardGrid` (5 cards das áreas)
- `RecommendedTrailCard` (sugere trilha conforme `user.role`)
- `MyProgressShortcut` (KPIs rápidos)
- `UpgradeBanner` (se tier `base`/`trial` — "Desbloqueie certificação - Assine Plus")

## Estados
- **Loading**: skeleton dos cards.
- **Trial / Base**: banner de upgrade visível.
- **Plus / Pro**: progresso e recomendações em destaque.
- **Parceiro Vizinhança**: rota retorna 403 (e botão "Academia" oculto na navegação).
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Visão home academia | `/api/v1/academy/dashboard` | GET |
| Trilha recomendada | `/api/v1/academy/paths/recommended` | GET |

## Regras de negócio críticas
- **INV-067**: Parceiro Vizinhança (`local_company`) → DENY com botão oculto.
- Trilha recomendada depende de `user.role`: Síndico → Trilha Síndico, Empresa → Trilha Empresa, Morador → Trilha Morador.
- Botão "Lives": ver-only para todos; **apenas admin agenda** ([[LMS8]]/[[LMS9]]) — D-062 MVP.

## Ligações
- [[LMS2]] · [[LMS6]] · [[LMS8]] · [[LMS11]] · [[LMS14]] · [[_moc]]
- Sub-produto: [[../../../../00-product/sub-produtos/06-lms-academia]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Algoritmo da trilha recomendada (peso por behavior) sem ADR.
- "Meu progresso" como rota dedicada vs widget — sem decisão.
