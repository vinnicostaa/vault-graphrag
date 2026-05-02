---
title: Web — Review Agente 1 do trabalho do Agente 2 (Lotes 2 + 3)
type: audit
tags:
  - audit
  - master-sindico
  - web
  - '2026-04-27'
  - review
  - dual-check
  - agente-1
project: master-sindico
subproject: web
created: '2026-04-27T19:00:00.000Z'
updated: '2026-04-27T19:00:00.000Z'
aliases:
  - web-review-agente-1-2026-04-27
---

# Web — Review Agente 1 do trabalho do Agente 2

Espelho da revisão executada como Agente 1 sobre os Lotes 2 + 3 do Agente Coder 2 no projeto `Development/web/`. Critério: dual-check rigoroso de cada finding, mesmo padrão da auditoria inicial [[AUDIT-2026-04-27]].

> **Conteúdo completo (versão FS):** `Development/web/.audit/REVIEW-2026-04-27-AGENTE-1.md`

## Veredito macro

✅ **APROVADO COM CONFIANÇA ALTA**. Padrão sênior do Agente 2, comentários referenciando vault canônico (D-134, STEERING #20, security.md). Zero claims falsos.

## Resumo numérico

| Categoria | Qtd |
|---|---|
| Findings marcados como resolvidos pelo Agente 2 | 11 |
| ✅ Aprovados sem ressalva | 8 |
| ⚠️ Aprovados com ressalva | 3 |
| ❌ Rejeitados | 0 |
| 🟢 Resoluções extras (não marcadas) | 3 |
| 🔴 Bug crítico descoberto na revisão | 1 |
| 🟡 Issues secundárias | 3 |

## Findings revisados (status final)

| ID | Severidade | Veredito Agente 1 |
|---|---|---|
| A-001 apps/admin port 3010 | 🟢 | ✅ EXEMPLAR — qualidade muito acima dos 5 apps base |
| A-004 btn-destructive | 🟢 | ✅ correto |
| A-005 scrollbar var(--border) | 🟢 | ✅ correto |
| A-007 createEffect+interval | 🟢 | ✅ pattern correto SolidJS docs |
| A-008 ServerErrorPage prod-safe | 🟢 | ⚠️ requestId interface dangling (router não passa) |
| A-009 AuthSession Zod | 🟢 | ✅ padrão sênior |
| A-010 biome a11y auth | 🟢 | ✅ confirmado via execução real |
| A-014 acentuação 'á'→'à' | 🟢 | ✅ 3 arquivos confirmados (LP, CMS, hero-carousel) |
| A-016 env.d.ts type-safe | 🟢 | ✅ excelente |
| A-006 uno.config drift | 🟡 | ⚠️ drift zerado AGORA, preset não extraído (causa permanece) |
| A-027 biome 5 apps base | 🟢 | ✅ funciona, ressalva tática `noUnknownProperty: off` amplo |
| A-020 cn re-export | 🟢 | extra (não marcado pelo Agente 2) |
| A-022 packages/core mention | 🟢 | extra (AGENTS.md reescrito) |
| `.kiro/specs` deletado | 🟢 | extra — alinha com D-030 (web/.kiro DEPRECATED) |

## 🔴 Bug crítico descoberto

**NEW-001 — 🟢 RESOLVIDO 2026-04-27 por D-138 / ADR-0045 (Cloudflare-only):** `apps/{auth,cms,lms,forum,assembly,lp}/railway.json` tinham `bun x turbo run build --filter=web-lp`:

1. `bun x turbo` — Turborepo não instalado (vault confirma "não Turborepo")
2. `--filter=web-lp` em TODOS — todos subdomínios serviriam o MESMO LP
3. `web-lp` não bate com `name` do package.json

Origem: pré-existente desde commit `30ce148` (DT-WEB-001 do vault original). Exposto agora porque admin foi criado correto. Fix: 30min trocando filter para nome real de cada app.

## 🟡 Issues secundárias

- **NEW-002:** tsconfig.json divergência (admin tem `noUnusedLocals: true` + `paths`, outros 6 não)
- **NEW-003:** ServerErrorPage requestId no tipo mas router não injeta — sempre exibe "Referência —" em prod
- **NEW-004:** biome.json `noUnknownProperty: off` mais amplo que necessário

## Pendentes ainda (do AUDIT-2026-04-27 original)

🔴 críticos backend: A-002 (OTP token), A-003 (PKCE flow)
🟠 high: A-011, A-012 (parcial), A-013 (5 apps base código morto)
🟡 médios: A-015, A-017, A-018, A-019, A-021, A-023 (verify), DT-WEB-008, DT-WEB-009

## Recomendação ao coordenador

1. **Aceitar 11 fixes** do Agente 2 com ressalvas A-006/A-008 anotadas
2. **Fix urgente NEW-001** (railway.json) — delegar ao Agente 2
3. **Próximo lote**: A-013, A-006-bis (preset), A-017/A-018 (testes), NEW-003 (wire requestId)
4. **Aguardar backend** para A-002 e A-003

## Validação executada (Agente 1)

- ✅ `bun run check` exit 0 (idêntico claim Agente 2)
- ✅ Vitest 81 pass / 0 fail (idêntico claim Agente 2)
- ✅ Conteúdo verificado linha-a-linha em 15+ arquivos
- ✅ Cross-check Context7 (SolidJS onMount/onCleanup, TanStack search params, Kobalte TextField)
- ✅ Cross-check vault (D-134, STEERING #20, security.md, ROADMAP)

## Links

- [[AUDIT-2026-04-27]] — auditoria inicial
- [[tasks]] — backlog técnico
- [[../../STATE]] — D-134 apps/admin
- [[../../STEERING]] — não-negociáveis
- [[security]] — defesa browser-side
- [[../../11-audit/AUDIT]] — findings backend
- `Development/web/.audit/REVIEW-2026-04-27-AGENTE-1.md` — conteúdo completo
