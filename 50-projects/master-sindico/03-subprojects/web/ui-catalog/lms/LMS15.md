---
title: "LMS15 — Página do Material"
type: ui-screen
tags: [master-sindico, ui-catalog, web, lms, academia, biblioteca, material]
project: master-sindico
persona: any
category: LMS
screen_id: LMS15
sub_produto: lms-academia
plan_requirement: base
status: specification
stack: web
created: 2026-04-23
---

# LMS15 — Página do Material (Biblioteca)

## Finalidade
Detalhe de um material da biblioteca: título, descrição, autor, tipo. Botões para baixar (signed URL) ou ler online (viewer embutido).

## Fonte canônica
- `vault/90-archive/from-development-content-2026-04-23/contents/MÓDULO ACADEMIA MASTER SÍNDICO.md` §LMS15

## Persona e ABAC
- **Personas**: Síndico, Empresa, Morador, Marketing, Admin
- **❌ BLOQUEADO**: Parceiro Vizinhança
- **Plan tier**: `base` (alguns gratuitos); `plus`+ download integral (CNT-025; INV-069)
- **ABAC action**: `content.library.read` · `content.library.download`

## Informações exibidas
- Título
- Descrição
- Autor
- Tipo de material (badge)
- Categoria
- Tamanho / duração
- Idioma (se aplicável)

## Botões
- **Baixar material** (PDF/EPUB via signed URL TTL 24h MinIO/R2)
- **Ler online** (viewer embutido)

## Componentes
- `MaterialHero`
- `MaterialDescription`
- `DownloadButton`
- `OnlineReader` (viewer PDF/EPUB embutido)
- `LockedBanner` (se tier insuficiente)

## Estados
- **Loading**: skeleton.
- **Disponível**: download ativo.
- **Bloqueado por plano**: viewer/download bloqueado, banner upgrade.
- **Sem permissão**: 403 (Parceiro Vizinhança).
- **Error download**: toast retry.

## Integração com backend
| Ação | Endpoint (inferido) | Método |
|---|---|---|
| Detalhe material | `/api/v1/academy/library/{id}` | GET (signed URL no payload) |
| Download direto | (signed URL para MinIO/R2) | GET |

## Regras de negócio críticas
- Signed URL **TTL ≤ 24h** (anti-hotlinking).
- Sem DRM no MVP (CNT-025).
- Storage MinIO dev / R2 ou S3 prod.
- Visibilidade por plano (CNT-025; INV-069).
- Tracking de download para métricas admin (CNT-025).

## Ligações
- [[LMS14]] · [[_moc]]

## Gaps/ressalvas
- Endpoint REST inferido.
- Provedor de viewer (PDF.js, react-pdf, Mozilla EPUB?) sem ADR.
- DRM: M4+ se necessário.
- Aggregate `LibraryMaterial` ausente (G-02).
- Política de re-download (rate limit) sem definição.
