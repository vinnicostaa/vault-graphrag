---
title: Sprint 0 — Web (Scaffolding 6 apps + stack)
type: sprint-spec
tags: [master-sindico, sprint-0, web, tasks]
project: master-sindico
stack: web
sprint: 0
period: "2026-04-13 → 2026-04-13 (scaffolding 1 dia)"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 0 Web — Scaffolding Bun + Rsbuild + SolidJS + UnoCSS + Kobalte

**Período**: 2026-04-13 (1 dia — scaffolding)
**Status**: ✅ parcial (esqueleto pronto, apps continuam placeholders até Sprint 10)
**Objetivo**: Materializar a decisão de stack web (SolidJS + Rsbuild + Kobalte + UnoCSS + Biome + Bun workspaces) como 6 apps (`auth`, `cms`, `lms`, `forum`, `assembly`, `lp`) scaffolded, com packages compartilhados.

## Escopo desta sprint (web)

Sprint 0 web foi um sprint de 1 dia com objetivo único: criar o esqueleto repo-wide. Diferente do backend (que em 2 semanas entregou base + OIDC + DB + middleware), web parou no scaffolding e ficou **6 meses sem progressão real** até Sprint 10 (A-FASE10-DEV-005 🔴). Essa divergência de ritmo é o principal risco do M1.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| web-0.1 | Bun workspaces `package.json` raiz + `apps/*` + `packages/*` | ✅ | `Development/web/package.json` | — | D-020 |
| web-0.2 | Scaffold `apps/auth` (Rsbuild + SolidJS + TanStack Router) | ✅ placeholder | `Development/web/apps/auth/` | A-FASE10-DEV-005 | — |
| web-0.3 | Scaffold `apps/cms` | ✅ placeholder | `Development/web/apps/cms/` | A-FASE10-DEV-005 | — |
| web-0.4 | Scaffold `apps/lms` | ✅ placeholder | `Development/web/apps/lms/` | A-FASE10-DEV-005 | — |
| web-0.5 | Scaffold `apps/forum` | ✅ placeholder | `Development/web/apps/forum/` | A-FASE10-DEV-005 | — |
| web-0.6 | Scaffold `apps/assembly` | ✅ placeholder | `Development/web/apps/assembly/` | A-FASE10-DEV-005 | — |
| web-0.7 | Scaffold `apps/lp` (+ blob hero implementado) | ✅ | `Development/web/apps/lp/src/routes/index.tsx` | — | — |
| web-0.8 | Scaffold `packages/ui` (Kobalte + CVA + UnoCSS preset) | ✅ inicial | `Development/web/packages/ui/` | — | — |
| web-0.9 | Scaffold `packages/schemas` (stub `console.log`) | ✅ stub | `Development/web/packages/schemas/` | DT-WEB-005 | — |
| web-0.10 | Biome + tsconfig raiz + UnoCSS config duplicado 6× | ✅ | `biome.json`, `tsconfig.json`, `apps/*/uno.config.ts` | DT-WEB-004 | — |
| web-0.11 | `railway.json` por app (cita `turbo` — débito DT-WEB-001) | ✅ | `apps/*/railway.json` | DT-WEB-001 | — |

## Dependências

- Sprint anterior: nenhuma (primeira entrega web).
- Cross-stack: backend Sprint 6 publica OpenAPI `/docs` — será consumido via Orval/openapi-typescript em Sprint 10 (FE-008).
- Decisões: D-020 (stack SolidJS + Rsbuild + Kobalte + UnoCSS + Bun); D-068 (Rsbuild + Biome 2).

## Entregáveis

- `bun install` no root instala workspaces.
- `bun run dev` por app sobe dev server (configurado mas apps retornam `<div>Hello "/_auth/sign-in"!</div>`).
- `bun run check` e `bun run build` passam localmente (sem conteúdo real).

## Gates (`<verify>`)

- `bun install` ✅
- `bun run check` (biome) ✅
- `bun run build` por app ✅ (bundles vazios/triviais)
- `tsc --noEmit` por app ✅ (sem código real)

## Pós-sprint

- **O que deu certo**: stack consolidado num dia; 6 apps + 2 packages scaffolded.
- **Bloqueadores**: nenhum nesta janela (problema é o que **não aconteceu** entre Sprint 0 e Sprint 10).
- **Dívidas criadas**: DT-WEB-001..015 (ver [[../../tasks#dividas-tecnicas-identificadas-dt-web]]).
- **Próxima sprint**: pulo até [[sprint-10]] — Sprints 1-8 web não tiveram entregas significativas ("apps ficaram placeholders", A-FASE10-DEV-005).

## Links

- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
- [[../../tasks|web tasks monolítico]]
