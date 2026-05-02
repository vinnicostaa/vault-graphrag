---
title: MOC — web/patterns
type: moc
tags:
  - moc
  - master-sindico
  - ui
  - web
  - patterns
created: '2026-04-27'
updated: '2026-04-27'
---

# MOC — web/patterns

UI patterns aplicados no frontend web do Master Síndico (SolidJS + Kobalte + UnoCSS + Bun). Catálogo de padrões reutilizáveis em telas de governança, comercial, conteúdo, assembly e cross-domain.

## Patterns destilados

### Estado e fluxo
- [[ui-states]] — estados UI canônicos (loading/empty/error/success) — **701 linhas (referência principal)**
- [[forms]] — convenções de formulário (Zod + Kobalte + accessibility)
- [[state-aware-actions]] — botões habilitados conforme state machine

### Dashboard e visualização
- [[dashboard-kpi-cards]] — cards de KPI com delta + sparkline + drill-down
- [[dashboard-cards-pivot]] — cards alternam entre visões (tabs/segmented)
- [[quota-display]] — barra consumo vs quota do plano
- [[advanced-filters]] — filtros avançados collapsible com estado em URL

### Feed e listas
- [[feed-infinite-scroll]] — feed paginado cursor-based + IntersectionObserver
- [[feed-prioritized]] — feed Vizinhança ordenado por prioridade determinística
- [[timeline-evolution]] — timeline append-only com agrupamento mensal
- [[append-only-feed]] — feed imutável (Timeline, Audit log)
- [[saved-list]] — lista persistida do user (favoritos, salvos)
- [[saved-search]] — busca salva com notification de matches novos
- [[bulk-actions]] — seleção múltipla + action bar

### Legal e compliance
- [[legal-terms-checkbox]] — aceite legal explícito antes de submit
- [[self-declaration-disclaimer]] — disclaimer + auto-declaração
- [[sensitive-data-disclaimer]] — banner antes de campo LGPD sensível
- [[recurring-required-action]] — ação obrigatória recorrente bloqueante

### Inputs especializados
- [[currency-input]] — input formatado para R$ (Intl.NumberFormat)
- [[voice-input-textarea]] — textarea com Web Speech API
- [[multi-image-upload]] — upload múltiplo via R2/S3 presigned
- [[video-recorder-mux]] — gravação browser + Mux Direct Upload

### Confirmação e moderação
- [[destructive-double-confirm]] — modal + digitar nome + checkbox
- [[content-moderation]] — fila de moderação (cascade rules ADR-0024)
- [[permission-matrix]] — UI matriz role × action para ABAC
- [[version-history]] — histórico de versões com diff

### Auxiliares
- [[context-switch-banner]] — banner persistente em contexto delegado
- [[auto-tags-rule-based]] — tags determinísticas a partir de campos
- [[pdf-export-template]] — template PDF (currículo, dossiê, ata)
- [[token-invite]] — convite via link único TTL
- [[expirable-content]] — conteúdo com expiração (countdown + auto-archive)
- [[qr-fullscreen]] — QR fullscreen para apresentação

## Convenções

- Todos os patterns destilados aqui são para **frontend web**. Mobile (Flutter) tem catálogo paralelo em `mobile/ui-catalog/` se aplicável.
- Cada pattern tem ≥ 2 cross-refs. Status `stub` indica destilação preliminar — promover quando entrar em sprint dedicado.
- Stack canônica: SolidJS + Kobalte + UnoCSS + Zod + `@motionone`.

## Vizinhos

- [[../_moc]] — MOC web (sub-projeto)
- [[../../mobile/_moc]] — sub-projeto mobile
- [[../../../07-quality/PATTERNS]] — registro de patterns aplicados no projeto
- [[../../../../20-stacks/typescript/frontend-solidjs]] — stack template
