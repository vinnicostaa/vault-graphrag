---
title: lms — Web Requirements (SolidJS)
type: requirement
tags: [master-sindico, web, solidjs, lms, feature-req, academia, mux]
project: master-sindico
stack: web
app: lms
port: 3002
milestone: m3
status: not-started
created: 2026-04-24
---

# lms — Web Requirements

## Escopo e marco

**Marco**: M3 (Sprint 12, 2026-07-20). Entregável principal: Academia Master Síndico — catálogo + player Mux VOD + live streaming + certificados + progresso + quiz + Fórum embutido (LMS11-LMS13) **ou** integração via iframe com app `forum`.

**Depende de**: backend LMS (Sprint M3) + Mux Video/Live (ADR-0004) + cookie httpOnly global.

## Personas servidas

- `sindico` (N1/N2/N3) — aluno default; N3 tem acesso premium a cursos pro.
- `morador` — acesso free ao catálogo público; paga por curso premium.
- `admin_academia` (sub-role de `admin_ms` D-058 ou instrutor) — CRUD cursos + lives + relatórios.
- `empresa` (plus/pro) — acesso a trilhas comerciais.

## Telas cobertas

| ID | Título | Rota | UI catalog |
|---|---|---|---|
| LMS1 | Home Academia (catálogo) | `/` | [[../ui-catalog/lms/_moc]] |
| LMS2 | Detalhe de curso | `/curso/$slug` | [[../ui-catalog/lms/_moc]] |
| LMS3 | Player aula (VOD) | `/curso/$slug/aula/$id` | [[../ui-catalog/lms/_moc]] |
| LMS4 | Live streaming | `/live/$id` | [[../ui-catalog/lms/_moc]] |
| LMS5 | Meus cursos | `/me/cursos` | [[../ui-catalog/lms/_moc]] |
| LMS6 | Certificados | `/me/certificados` | [[../ui-catalog/lms/_moc]] |
| LMS7 | Quiz | `/curso/$slug/quiz/$id` | [[../ui-catalog/lms/_moc]] |
| LMS8 | Progresso | `/me/progresso` | [[../ui-catalog/lms/_moc]] |
| LMS9 | Busca + filtros | `/busca` | [[../ui-catalog/lms/_moc]] |
| LMS10 | Notas de aula | `/curso/$slug/aula/$id/notas` | [[../ui-catalog/lms/_moc]] |
| LMS11 | Fórum curso (lista) | `/curso/$slug/forum` | [[../ui-catalog/lms/_moc]] |
| LMS12 | Fórum thread | `/curso/$slug/forum/$threadId` | [[../ui-catalog/lms/_moc]] |
| LMS13 | Fórum novo post | `/curso/$slug/forum/novo` | [[../ui-catalog/lms/_moc]] |
| LMS14 | Admin cursos | `/admin/cursos` | [[../ui-catalog/lms/_moc]] |
| LMS15 | Admin lives | `/admin/lives` | [[../ui-catalog/lms/_moc]] |

## Funcionais (FR-WEB-LMS-NNN)

### Catálogo e descoberta

- **FR-WEB-LMS-001** — LMS1 home apresenta trilhas em destaque + cursos recentes + "continuar assistindo".
- **FR-WEB-LMS-002** — Filtros: nível (iniciante/intermediário/avançado), duração, tema (compliance, finanças, assembleias), idioma.
- **FR-WEB-LMS-003** — Busca full-text (backend `ISearchProvider` real — OpenSearch ou Meilisearch via ADR-0042 pending dual-check; D-120 revoga D-067 PG tsvector) — resultados em < 300ms.
- **FR-WEB-LMS-004** — Cards de curso: thumbnail (Mux poster), título, duração total, progresso (0-100%), lock icon se premium.

### Detalhe e player

- **FR-WEB-LMS-005** — LMS2 detalhe: sinopse, grade curricular (aulas expansíveis), instrutor, CTA "Começar" / "Continuar".
- **FR-WEB-LMS-006** — LMS3 player Mux via `@mux/mux-player` (Web Component) com `playback-id` signed.
- **FR-WEB-LMS-007** — Controles custom opcionais: velocidade 0.5x-2x, legendas pt-BR, qualidade auto, marcador de capítulos.
- **FR-WEB-LMS-008** — Progresso aula enviado a cada 10s via `POST /lms/progresso` (debounce); retomada exata no reload.
- **FR-WEB-LMS-009** — Auto-próxima aula com countdown 5s (pular).
- **FR-WEB-LMS-010** — Picture-in-Picture + fullscreen + keyboard shortcuts (space, arrows).

### Live

- **FR-WEB-LMS-011** — LMS4 live embeda Mux Live player; fallback para VOD replay após término.
- **FR-WEB-LMS-012** — Chat lateral via WebSocket `wss://api.../lms/live/:id/chat` (reconnect backoff).
- **FR-WEB-LMS-013** — Indicador "AO VIVO" vermelho + contador de espectadores (delay 30s).
- **FR-WEB-LMS-014** — Latência alvo **< 5s glass-to-glass** (Mux Low-Latency HLS).
- **FR-WEB-LMS-015** — Moderação chat: admin pode bloquear msg + ban user; flag spam por usuário.

### Meus cursos / progresso / certificados

- **FR-WEB-LMS-016** — LMS5 lista cursos matriculados com % progresso e último acesso.
- **FR-WEB-LMS-017** — LMS6 certificados: grid + preview PDF + download + compartilhar LinkedIn (OG image).
- **FR-WEB-LMS-018** — Certificado emitido automaticamente ao atingir 100% + nota quiz ≥ 70%.
- **FR-WEB-LMS-019** — LMS8 progresso gráfico semanal (minutos assistidos, aulas concluídas).

### Quiz

- **FR-WEB-LMS-020** — LMS7 quiz multi-escolha com 1 pergunta por vez; feedback imediato após submit da questão.
- **FR-WEB-LMS-021** — Tempo limite opcional (timer visível); tentativa persistida server-side (anti-cheat).
- **FR-WEB-LMS-022** — Revisão pós-submit: mostra gabarito + explicação por questão.

### Notas e fórum

- **FR-WEB-LMS-023** — LMS10 notas pessoais por aula (rich-text minimalista), privadas.
- **FR-WEB-LMS-024** — LMS11-13 fórum do curso: threads por aula/trilha; decisão aberta (embed vs app separado).

## Não-funcionais (NFR-WEB-LMS)

- **Performance**: LCP < 2.5s; INP < 200ms. Player Mux lazy-loaded (não no initial bundle). Bundle < **300 KB** initial.
- **Acessibilidade**: WCAG 2.1 AA. Player Mux acessível nativamente (legendas, audio descriptions opcionais). `role="region" aria-label="Player de vídeo"`.
- **SEO**: catálogo LMS1/LMS2 **indexável** (SSR/SSG via Rsbuild plugin) — `<title>`, OG tags, schema.org `Course`.
- **i18n**: pt-BR; EN futuro (M3+ se conteúdo EN for produzido).
- **Streaming**: live latência < 5s; VOD time-to-first-frame < 1.5s.

## Rotas TanStack Router

```
src/routes/
├── __root.tsx
├── index.tsx                            # LMS1
├── busca.tsx                            # LMS9
├── curso.$slug.tsx                      # LMS2
├── curso.$slug.aula.$id.tsx             # LMS3
├── curso.$slug.aula.$id.notas.tsx       # LMS10
├── curso.$slug.quiz.$id.tsx             # LMS7
├── curso.$slug.forum.tsx                # LMS11
├── curso.$slug.forum.$threadId.tsx      # LMS12
├── curso.$slug.forum.novo.tsx           # LMS13
├── live.$id.tsx                         # LMS4
├── me.cursos.tsx                        # LMS5
├── me.certificados.tsx                  # LMS6
├── me.progresso.tsx                     # LMS8
└── admin/
    ├── cursos.tsx                       # LMS14
    └── lives.tsx                        # LMS15
```

## Estado

- **TanStack Solid Query** — `['lms','courses']`, `['lms','course',slug]`, `['lms','progress', userId]`; optimistic em progresso.
- **createSignal** — player state (currentTime, paused, volume) — mux-player expõe eventos.
- **createStore** — quiz: `{currentQuestion, answers, remainingTime}`.
- **WebSocket singleton** (`lms-chat-ws.ts`) para LMS4 live chat — reconnect + buffer.
- **TanStack Solid Form** — LMS13 novo post fórum, LMS10 notas.

## Integração backend

- **Base URL**: `http://localhost:8000/api/v1/lms/*` / prod idem.
- **Auth**: Cookie httpOnly compartilhado.
- **Mux signed URLs**: `playback-id` assinado pelo backend (não decode no client).

### Endpoints consumidos

| Método | Path | Payload | Retorno |
|---|---|---|---|
| GET | `/lms/courses?page&q&level` | — | `{items,total}` |
| GET | `/lms/courses/:slug` | — | `{course,lessons,instructor}` |
| GET | `/lms/lessons/:id/playback` | — | `{muxPlaybackId,token,chapters}` |
| POST | `/lms/progress` | `{lessonId,seconds,completed}` | 204 |
| GET | `/lms/me/courses` | — | `{items}` |
| GET | `/lms/me/certificates` | — | `{items}` |
| GET | `/lms/me/progress` | — | `{byCourse,weekly}` |
| POST | `/lms/quizzes/:id/attempts` | `{answers[]}` | `{score,correct}` |
| GET | `/lms/live/:id` | — | `{muxLivePlaybackId,status}` |
| WSS | `/lms/live/:id/chat` | `{text}` | stream msgs |
| POST | `/lms/courses/:slug/forum/threads` | `{titulo,corpo}` | 201 |
| GET | `/lms/courses/:slug/forum?page` | — | `{items}` |

## Design system

- Player: `@mux/mux-player` com wrapper CVA para tokens de cores (gold progress bar).
- Cards de curso: reuso `Card` + `Badge` + `Progress` (novo atom).
- Trilhas: `Tabs` Kobalte.
- Chat live: layout 70/30 (player/chat) desktop; stack vertical mobile.

## Segurança

- **Mux signed playback**: token 5min TTL; backend emite por request; frontend nunca decodifica.
- **CSP**: `connect-src` + `media-src` inclui `*.mux.com`, `*.mux.dev`; `img-src` para thumbnails.
- **XSS fórum**: DOMPurify em markdown + sanitize no backend também (defense in depth).
- **Rate limit chat live**: 5 msgs/min por user; feedback UX de cooldown.
- **Anti-share**: proibir `<video>` download; Mux DRM opcional para pro courses (M3 stretch).

## Testes

- **Vitest** — 90% hooks (`useProgress`, `useCourseQuery`), 85% components player wrapper.
- **@solidjs/testing-library** — LMS1 catálogo render, LMS2 detalhe, LMS7 quiz fluxo.
- **MSW** — mock `/lms/*` endpoints.
- **Playwright (E2E M3)** — matricular → assistir 10s → progresso persiste → certificado.
- **jest-axe** em player + catálogo + quiz.

## Status atual (real) — AUDIT A-FASE10-DEV-005

🔴 `apps/lms/src/index.tsx` = stub `<div>Hello lms!</div>`. Zero das 15 telas. Zero testes. `@mux/mux-player` NÃO em `bun.lock`. `@tanstack/solid-query` e `@tanstack/solid-form` ausentes. Backend LMS ainda não existe (Sprint M3). Não bloqueador M1, mas requires parallel track.

## Gaps/decisões abertas

- **G1**: **Fórum embutido vs app separado** — decisão aberta. Embed mantém contexto do curso; separado reutiliza infra de `forum`. Sugestão: embed com deep-link para `forum` app para threads cross-course.
- **G2**: Mux Low-Latency HLS requer configuração backend específica; validar custo.
- **G3**: Certificado PDF gerado client-side (react-pdf? — equivalente Solid) vs backend.
- **G4**: Quiz anti-cheat — timer server-side obrigatório?
- **G5**: Picture-in-Picture iOS Safari limitações.
- **G6**: SEO SSG para catálogo — Rsbuild suporta prerender rotas estáticas?

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../architecture]]
- [[../design-system]]
- [[../security]]
- [[../requirements]]
- [[../ui-catalog/lms/_moc]]
- [[../../../02-architecture/adr/0010-mux-video-provider]]
- [[forum]]
