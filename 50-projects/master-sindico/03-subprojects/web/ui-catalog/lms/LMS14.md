---
title: "LMS14 — Biblioteca"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, biblioteca]
project: master-sindico
persona: any
category: LMS
screen_id: LMS14
sub_produto: lms-academia
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# LMS14 — Biblioteca

## Finalidade
Catálogo de materiais de consulta: vídeos, guias, manuais, livros digitais, e-books, materiais técnicos. Filtros por tipo e categoria.

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS14

## Mensagem institucional canônica
> "Materiais de consulta para aprofundar conhecimentos sobre o universo condominial."

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador, Marketing, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `base` (parcial); integral em `plus`+ (CNT-025; INV-069)
- **ABAC action**: `content.library.list`

## Tipos de conteúdo (`LibraryMaterialType` — 6 valores)
- vídeos
- guias_técnicos
- manuais
- livros_digitais
- ebooks
- materiais_técnicos

## Filtros
- Tipo de conteúdo
- Categoria (reaproveita `CourseCategory` 10)
- Autor

## Card por material
- Título
- Autor
- Tipo (badge)
- Tamanho / duração
- Categoria
- Botão "Ver detalhes" → [[LMS15]]

## Componentes
- `LibraryCardGrid`
- `FilterBar` (3 filtros)
- `Pagination`
- `EmptyState`

## Estados
- **Loading**: skeleton 9 cards.
- **Empty (filtros)**: "Nenhum material com esses filtros."
- **Empty (geral)**: "Catálogo em construção."
- **Tier limitação**: badge "Plus" em materiais bloqueados.
- **Error**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Listar | `/api/v1/academy/library` | GET (filtros + paginação) |

## Regras de negócio críticas
- **INV-069**: Ebook access filtrado por `plan_tier`.
- Base parcial: alguns materiais gratuitos abertos; resto exige `plus`.
- Signed URL TTL 24h para download de PDF/EPUB.
- Storage MinIO/R2.
- Sem DRM (CNT-025).

## Exemplos canônicos
- Guia de prevenção de pragas
- Manual de manutenção predial
- Guia de convivência condominial
- Manual do síndico iniciante

## Ligações
- [[LMS1]] · [[LMS15]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Aggregate `LibraryMaterial` ausente (G-02).
- Categorias da biblioteca: nota canônica diz "reaproveita CourseCategory" mas pode haver categorias específicas.
- Evento `LibraryMaterialPublished` não formalizado (G-04).
