---
title: MK5 — Biblioteca de Vídeos
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias, conteudo-videos]
project: master-sindico
persona: marketing
category: MK
screen_id: MK5
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK5 — Biblioteca de Vídeos

## Finalidade
Lista todos os vídeos publicados pela agência para a empresa selecionada (contexto via [[MK3]]). Permite editar metadados ou ocultar vídeos. **Ocultar não remove o histórico** (R3 — INSERT-only / soft-hide).

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK5 — Biblioteca de Vídeos + observação devs)

## Persona e ABAC
- **Persona primária**: Marketing
- **Plan tier mínimo**: any
- **ABAC action**: `content.video.list` + `content.video.update` (scoped)
- **Restrições**: scoped a `empresa_id` do contexto + `agency_id`.

## Fluxo da tela
1. Agência em [[MK3]] → clica card "Biblioteca de vídeos".
2. Sistema lista vídeos da empresa (mais recentes primeiro).
3. Cada card: thumbnail, título, tipo, data publicação, visualizações, status (visível/oculto).
4. Agência clica [Editar vídeo] → abre modal de edição (campos similares a [[MK4]]).
5. Agência clica [Ocultar vídeo] → modal double-confirm; vídeo fica oculto mas preservado em histórico.
6. Filtros: tipo de vídeo, período, visível/oculto.

## Componentes
- `VideoGrid` (cards com thumbnail Mux)
- `VideoCard` (título + tipo + data + views + ações)
- `EditVideoModal`
- `HideVideoDialog` (double-confirm)
- `FilterBar` (tipo, período, status)
- `EmptyState`
- `BadgeStatus` (Visível / Oculto)

## Informações exibidas por card (fonte PDF MK5)
- Thumbnail (Mux)
- Título
- Tipo de vídeo (1 das 12)
- Data de publicação
- Visualizações (contador agregado)
- Status (Visível / Oculto)
- Botões: [Editar vídeo] / [Ocultar vídeo]

## Estados
- **Loading**: skeleton 6 cards
- **Empty**: "Nenhum vídeo publicado ainda. Comece em [[MK4]]."
- **Error**: toast retry
- **Hidden**: card cinza com badge "Oculto" (não remove)
- **Filtered empty**: "Nenhum vídeo para os filtros aplicados"

## Integração com backend
| Ação UI | Endpoint | Método | Retorno |
|---|---|---|---|
| Listar vídeos | `/api/v1/content/videos?empresa_id=:id` | GET | Video[] |
| Editar vídeo | `/api/v1/content/videos/:id` | PUT | partial |
| Ocultar vídeo | `/api/v1/content/videos/:id/hide` | POST | - |
| Restaurar (pós-ocultado) | `/api/v1/content/videos/:id/restore` | POST | - |

Use cases `content.ListVideos` + `content.UpdateVideo` + `content.HideVideo` + `content.RestoreVideo`.

## Regras de negócio críticas
- **R3 — Ocultar não remove histórico**: vídeo permanece no DB com `visible = false`; auditoria mostra quem ocultou
- **Lock 90 dias**: edição de vídeo possivelmente bloqueada nos primeiros 90 dias após publicação (D-066 — confirmar)
- **Visualizações**: contagem incrementa apenas quando vídeo é exibido em assembleia (votação fornecedor) ou no perfil empresa quando síndico consulta (não conta views internas da agência)
- **Ocultar afeta visibilidade futura**: vídeos ocultos NÃO aparecem mais em votação de assembleia mesmo se autorizados originalmente
- **Auditoria**: edit + hide + restore registrados (agency_user, timestamp)

## Ligações
- **Tela origem**: [[MK3]]
- **Tela destino**: [[MK4]] (publicar novo), modal de edição
- **Sub-produto**: [[../../../../00-product/sub-produtos/04-conteudo-videos]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- "Restaurar vídeo oculto" não está explicitamente no PDF — proposto para coerência com R3 (operação reversível mas auditada)
- Lock 90d para edição: política não totalmente clara — sugerir desabilitar campos de tipo/título nos primeiros 90d, permitir editar descrição
- Player inline na biblioteca: PDF não menciona — sugerir preview ao hover (Mux thumbnail + play)

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Visibilidade Videos]]
