---
title: Pattern — Feed Infinite Scroll
type: pattern
tags:
  - pattern
  - ui
  - web
  - master-sindico
  - feed
  - performance
created: '2026-04-27'
updated: '2026-04-27'
---

# Pattern — Feed Infinite Scroll

Feed paginado com **scroll infinito** (cursor-based) para Vizinhança, Connect Me, Banco de Talentos, Notificações.

## Estrutura

- **Cursor pagination** (não offset) — `?cursor=<base64-encoded-key>&limit=20`.
- **IntersectionObserver** dispara fetch quando sentinel próximo do bottom (margem 200px).
- **Skeleton loaders** durante fetch — 3 placeholders com pulse animation.
- **End-of-feed**: mensagem clara "Você chegou no fim. Voltar ao topo ↑".
- **Pull-to-refresh** em mobile (gesto nativo) re-puxa do cursor inicial.
- **Real-time updates** (opcional): WebSocket adiciona itens novos no topo com badge "1 novo".

## Quando usar

- Feeds com volume crescente que humano lê linearmente.
- Vizinhança (promoções), Connect Me (RFPs), notificações.

## Quando evitar

- Conteúdo que precisa estado consistente (carrinho, fluxo de assinatura) — paginação tradicional ou tabela.
- SEO crítico — usar paginação numerada que crawler entende.

## Performance

- Virtual scrolling se > 200 items renderizados (Solid Virtual ou Tanstack Virtual).
- `loading="lazy"` em imagens.
- Debounce do IntersectionObserver para não disparar fetch duplicado.

## Cross-refs

- [[timeline-evolution]] — variante temporal
- [[ui-states]]
- [[../../07-quality/PATTERNS]]

## Links

- [[_moc]]
