---
title: forum — Web Requirements (SolidJS)
type: requirement
tags: [master-sindico, web, solidjs, forum, feature-req, vizinhanca]
project: master-sindico
stack: web
app: forum
port: 3003
milestone: m2
status: not-started
created: 2026-04-24
---

# forum — Web Requirements

## Escopo e marco

**Marco**: M2 (Sprint 11, 2026-06-20). Entregável principal: Fórum + Vizinhança Portal Morador. Cobre Vizinhança Morador (VM1-VM7) e threads de fórum transversais (LMS11-LMS13 se decisão for app separado).

**Depende de**: backend commercial module (já pronto no backend Go) + auth funcionando + cms morador linkando para cá.

## Personas servidas

- `morador` — acesso a Vizinhança (VM1-VM7), threads públicas e por condomínio.
- `sindico` — moderação da Vizinhança do seu condomínio (VZ1-VZ3 embed).
- `empresa` — listagem pública + mensagens DM para leads.
- `admin_ms` — moderação global (D-058) + ban + report resolve.

## Telas cobertas

| ID | Título | Rota | UI catalog |
|---|---|---|---|
| VM1 | Vizinhança home morador | `/` | [[../ui-catalog/vizinhanca/_moc]] |
| VM2 | Lista empresas próximas | `/empresas` | [[../ui-catalog/vizinhanca/_moc]] |
| VM3 | Perfil empresa | `/empresa/$slug` | [[../ui-catalog/vizinhanca/_moc]] |
| VM4 | Solicitar serviço | `/empresa/$slug/solicitar` | [[../ui-catalog/vizinhanca/_moc]] |
| VM5 | Minhas conversas | `/me/conversas` | [[../ui-catalog/vizinhanca/_moc]] |
| VM6 | Avaliações recebidas/dadas | `/me/avaliacoes` | [[../ui-catalog/vizinhanca/_moc]] |
| VM7 | Favoritos | `/me/favoritos` | [[../ui-catalog/vizinhanca/_moc]] |
| F1 | Fórum home (feed) | `/forum` | [[../ui-catalog/vizinhanca/_moc]] |
| F2 | Categoria | `/forum/c/$slug` | [[../ui-catalog/vizinhanca/_moc]] |
| F3 | Thread detalhe | `/forum/t/$id` | [[../ui-catalog/vizinhanca/_moc]] |
| F4 | Novo post | `/forum/novo` | [[../ui-catalog/vizinhanca/_moc]] |
| F5 | Busca fórum | `/forum/busca` | [[../ui-catalog/vizinhanca/_moc]] |
| F6 | Moderação | `/forum/moderar` | [[../ui-catalog/vizinhanca/_moc]] |

## Funcionais (FR-WEB-FORUM-NNN)

### Vizinhança — Morador (VM1-VM7)

- **FR-WEB-FORUM-001** — VM1 home: CTA "Buscar serviços", cards de categorias (encanador, eletricista, faxina, pintor, consultoria síndico), destaques e avaliações recentes.
- **FR-WEB-FORUM-002** — VM2 lista empresas filtráveis (categoria, distância via CEP condomínio, rating mínimo, preço-médio badge).
- **FR-WEB-FORUM-003** — Paginação infinita com `intersectionObserver` + TanStack Query `useInfiniteQuery`.
- **FR-WEB-FORUM-004** — VM3 perfil empresa: galeria, serviços oferecidos, avaliações, CTA "Solicitar orçamento" + "Chat".
- **FR-WEB-FORUM-005** — VM4 solicitar serviço: form Zod (descrição, tipo, urgência, disponibilidade, anexos fotos) + voice input R2.
- **FR-WEB-FORUM-006** — Submit → cria Connect Me request (backend `/connect-me/requests`); redirect para VM5.
- **FR-WEB-FORUM-007** — VM5 lista conversas (threads 1-1 morador↔empresa); ordenadas por última msg; badge unread.
- **FR-WEB-FORUM-008** — VM5 detalhe conversa com histórico, anexos, timeline (data, lido em).
- **FR-WEB-FORUM-009** — VM6 avaliações: morador pode avaliar (1-5 estrelas + comentário) após serviço concluído; modera antes de publicar.
- **FR-WEB-FORUM-010** — VM7 favoritos: toggle coração em perfis; lista persistida server-side.

### Fórum transversal (F1-F6)

- **FR-WEB-FORUM-020** — F1 feed: threads ordenadas por hot/new/top; filtros por categoria e tags.
- **FR-WEB-FORUM-021** — F2 categoria: página por slug (ex: `/forum/c/finanças-condominiais`).
- **FR-WEB-FORUM-022** — F3 thread: post original + árvore de respostas (até 3 níveis); ações: reply, like, report, share link.
- **FR-WEB-FORUM-023** — F4 novo post: título, corpo markdown, categoria, tags, anexos (imagens). Preview antes publicar.
- **FR-WEB-FORUM-024** — F5 busca: full-text backend + filtros data, categoria, autor.
- **FR-WEB-FORUM-025** — F6 moderação (síndico/admin): fila de reports + ações (aprovar, remover, ban user, marcar resolvido).

### Transversal

- **FR-WEB-FORUM-026** — Markdown renderer com DOMPurify + highlight.js (code blocks).
- **FR-WEB-FORUM-027** — Menções `@username` com autocomplete (TanStack Query debounced).
- **FR-WEB-FORUM-028** — Notificações in-app para menções + respostas (M3).

## Não-funcionais (NFR-WEB-FORUM)

- **Performance**: LCP < 2.5s; listas infinitas com virtualização (TanStack Solid Virtual) em >100 itens.
- **Acessibilidade**: WCAG 2.1 AA. Threads árvore navegável por teclado (arrow keys expandem/collapse).
- **SEO**: threads públicas **indexáveis** (SSG via Rsbuild prerender ou SSR) — schema.org `DiscussionForumPosting`.
- **i18n**: pt-BR; termos "Vizinhança", "condomínio vizinho", "avaliação" preservados.
- **Anti-abuso**: rate limit client UX (3 posts / 10min — backend enforcement real).

## Rotas TanStack Router

```
src/routes/
├── __root.tsx
├── index.tsx                        # VM1
├── empresas.tsx                     # VM2
├── empresa.$slug.tsx                # VM3
├── empresa.$slug.solicitar.tsx      # VM4
├── me.conversas.tsx                 # VM5
├── me.conversas.$id.tsx             # VM5 detalhe
├── me.avaliacoes.tsx                # VM6
├── me.favoritos.tsx                 # VM7
└── forum/
    ├── index.tsx                    # F1
    ├── c.$slug.tsx                  # F2
    ├── t.$id.tsx                    # F3
    ├── novo.tsx                     # F4
    ├── busca.tsx                    # F5
    └── moderar.tsx                  # F6 (guard role)
```

## Estado

- **TanStack Solid Query** — `useInfiniteQuery` para listas; `['forum','threads',filters]`, `['vizinhanca','empresas',filters]`.
- **createSignal** — filtros UI, input busca debounced 300ms.
- **createStore** — composer novo post (rascunho auto-save local em `sessionStorage`).
- **TanStack Solid Form** — VM4 solicitar, F4 novo post, VM6 avaliar — todos com Zod.
- **Optimistic updates**: like/unlike em threads e posts; reply; favoritar.

## Integração backend

- **Base URL**: `http://localhost:8000/api/v1/*`.
- **Cookie compartilhado**.

### Endpoints consumidos

| Método | Path | Payload | Retorno |
|---|---|---|---|
| GET | `/vizinhanca/empresas?page&cat&cep&rating` | — | `{items,total,nextCursor}` |
| GET | `/vizinhanca/empresas/:slug` | — | `{profile,services,reviews}` |
| POST | `/vizinhanca/connect-me/requests` | `{empresaId,descricao,urgencia,anexos}` | 201 `{threadId}` |
| GET | `/vizinhanca/conversas` | — | `{items}` |
| GET | `/vizinhanca/conversas/:id` | — | `{msgs,empresa}` |
| POST | `/vizinhanca/conversas/:id/msg` | `{text,anexos?}` | 201 |
| POST | `/vizinhanca/avaliacoes` | `{empresaId,rating,comentario}` | 201 |
| POST | `/vizinhanca/favoritos/:empresaId` | — | 204 toggle |
| GET | `/forum/threads?page&cat&sort` | — | `{items,nextCursor}` |
| GET | `/forum/threads/:id` | — | `{thread,replies}` |
| POST | `/forum/threads` | `{titulo,corpo,cat,tags}` | 201 |
| POST | `/forum/threads/:id/reply` | `{corpo,parentId?}` | 201 |
| POST | `/forum/threads/:id/like` | — | 204 toggle |
| POST | `/forum/reports` | `{targetType,targetId,motivo}` | 201 |

## Design system

- Feed: `Card` + `Avatar` + `Badge` + `Button` (reuso de packages/ui).
- Composer: `RichText` atom (tiptap Solid ou textarea+markdown preview) + `FileUpload`.
- Avaliações: `Rating` atom (5 stars, ARIA-compliant via Kobalte radio group).
- Chat VM5: burbujas alternadas; timeline agrupada por dia; ícones Lucide (`i-send`, `i-paperclip`).

## Segurança

- **XSS**: markdown via DOMPurify configurado com whitelist tight (no `<script>`, no `on*` attrs).
- **Rate limit**: backend 3 posts/10min + 30 likes/min; frontend mostra toast + cooldown.
- **Report abuse**: feedback imediato ao reportar; dedup server-side.
- **IDOR**: backend valida tenant + ownership para editar/deletar; frontend nunca confia.
- **Menções**: sanitizar `@username` para impedir injection.

## Testes

- **Vitest** — 90% hooks/queries, 85% components (markdown render, thread tree).
- **@solidjs/testing-library** — VM2 lista filtrável, VM4 form, F3 reply fluxo.
- **MSW** — mock `/vizinhanca/*` + `/forum/*`.
- **Playwright (E2E M3)** — morador busca → solicita → empresa responde → avalia.
- **jest-axe** — feed, composer, thread.

## Status atual (real) — AUDIT A-FASE10-DEV-005

🔴 `apps/forum/src/index.tsx` = stub `<div>Hello forum!</div>`. Zero das 13 telas. Zero testes. Nenhuma dependência markdown/RichText instalada. Backend commercial module existe — integração pendente. Não bloqueador M1.

## Gaps/decisões abertas

- **G1**: Rich-text markdown editor — tiptap tem wrapper Solid maduro? Fallback textarea + preview.
- **G2**: SSR/SSG para SEO de threads — Rsbuild prerender ou migrar rotas públicas para lp?
- **G3**: Notificações in-app real-time — SSE vs polling (30s) no M2, WS no M3?
- **G4**: Avaliação após serviço — quem valida conclusão (síndico? automático 30d?).
- **G5**: Geolocalização fuzzy por CEP ou geohash — precisão vs privacidade.
- **G6**: Moderação — fila por condomínio (síndico) vs global (admin MS).

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../architecture]]
- [[../design-system]]
- [[../security]]
- [[../requirements]]
- [[../ui-catalog/vizinhanca/_moc]]
- [[cms]]
- [[lms]]
