---
title: "LMS2 — Explorar Cursos"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, cursos]
project: master-sindico
persona: any
category: LMS
screen_id: LMS2
sub_produto: lms-academia
plan_requirement: trial
status: specification
stack: web
created: 2026-04-23
---

# LMS2 — Explorar Cursos

## Finalidade
Catálogo navegável de cursos com filtros (categoria, nível, tipo) + busca full-text. Card mostra título, professor, carga horária, nível, tipo (gratuito/pago).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS2
- `vault/50-projects/master-sindico/00-product/sub-produtos/06-lms-academia.md` §9 LMS3

## Mensagem institucional canônica
> "Aprofunde seus conhecimentos com cursos criados para quem vive o universo condominial."

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador (pagante), Marketing, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier mínimo**: `trial` (catálogo visível; matrícula exige `plus`)
- **ABAC action**: `content.course.list`

## Filtros
- **Categoria** (10 valores: gestão_condominial, saúde_ambiental, segurança_predial, manutenção_predial, legislação_condominial, mediação_conflitos, tecnologia_condomínios, sustentabilidade, administração_financeira, outros)
- **Nível** (3: básico / intermediário / avançado)
- **Tipo** (2: gratuito / pago)

## Card do curso
- Título
- Professor (link perfil)
- Carga horária
- Nível
- Tipo
- Botão **Ver curso** → [[LMS3]]

## Componentes
- `CourseCardGrid`
- `FilterBar` (3 filtros + busca)
- `Pagination`
- `EmptyState`

## Estados
- **Loading**: skeleton 9 cards.
- **Empty (filtros)**: "Nenhum curso com esses filtros."
- **Empty (geral)**: "Catálogo em construção" (M3 piloto com poucos cursos).
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar cursos | `/api/v1/academy/courses` | GET (filtros + paginação) |

## Regras de negócio críticas
- Cursos pagos abrem fluxo Stripe (`IPaymentGateway`) ao matricular.
- Tier `trial` mostra preview; matrícula exige `plus`.
- Busca full-text via `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending; D-120 Fase 14 revoga D-067 PG tsvector).
- Empresas Pro podem aparecer como criadoras (com badge).

## Ligações
- [[LMS1]] · [[LMS3]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- "Outros" categoria não tem filtro hierárquico.
- Conteúdo M3 = 3 trilhas piloto (~5 cursos cada). M4+ via parceria ABRACS.
