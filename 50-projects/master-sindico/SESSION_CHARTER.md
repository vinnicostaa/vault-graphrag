---
title: SESSION_CHARTER — Master Síndico v2
type: state
tags:
  - session
  - charter
  - master-sindico
  - v2
  - agente-2
created: '2026-04-23'
updated: '2026-04-27'
---

# SESSION_CHARTER — Master Síndico v2

Ordens vivas da sessão atual. Substituir a cada novo começo de sessão.

---

## Sessão 2026-04-27 (tarde) — Pass sistemático Agente 2

### Ordem do usuário (verbatim)

> "irmao analisa os projetos, veja onde esta, como esta, verifica o vault tambem do MCP do obsidian, a cada etapa do que voce fizer vai avisar o grupo do telegram via bot, vai analisar no vault o automation-agents e dentro vai ter a pasta vault, sempre analise de forma sistematica, profunda e minuciosa com dual check, so trate algo como verdade absoluta apos revisar em docs ou exemplos oficiais, lembre-se de analisar o codigo com as regras do vault, sempre revise e autualize o vault com o progresso do projeto com as tasks, audit, testes de seguranca, tdd, ddd, solid, etc, no bot voce vai notificar o group Princesas e nao minha DM, analise todos os arquivos que for preciso, nada de ignorar ou pular algo por ser complexo, nao quero que seja veloz, quero que seja seguro, escalavel, legivel e mantivel, isso 'e importante, pedi pra outro agente que tambem esta rodando colocar os gaps atuais que ele encontrou em findings pra voce quando terminar a sua analise comecar resolvendo, vai fazer tudo de forma autonoma"
>
> + ajuste posterior: "voce vai sinalizar no telegram e em seus relatorios que manda la como agente 2"

### Identificação

- **Agente 1** — pass anterior do dia (deixou findings em `Development/web/.audit/FINDINGS-2026-04-27.md` + 38 patches de consistência no vault — registro em [[11-audit/audit-log/2026-04-27-consistency-pass]]).
- **Agente 2 (esta sessão)** — execução autônoma resolvendo gaps + scaffold + foundation backend + deps mobile.

### Lotes executados

| Lote | Escopo | Status | Detalhes |
|---|---|---|---|
| **1** | Quick wins web (10 fixes) | ✅ | scrollbar var, btn-destructive, env.d.ts type-safe, acentuação 3 lugares, ServerErrorPage prod-safe, AuthSession Zod safeParse, createEffect+setInterval refactor, cn export, AGENTS.md cleanup, role schema (falso positivo) |
| **2** | A11y biome apps/auth | ✅ | tabIndex span ativo, fieldset+legend SSO, section aria-roledescription carousel, catch sem unused, biome overrides routeTree+main.css. **`bun run check` 0 errors / 0 warnings / 1 info** (down from 6/14/1). 81 testes verdes (1 novo) |
| **4** | apps/admin scaffold (D-134) | ✅ | porta 3010, banner Modo Admin, 5 áreas planejadas, guard placeholder, initialColorMode dark, 17 arquivos novos |
| **5** | Backend A-035 / BE-001 foundation | ✅ partial | VOs UnitType+FracaoIdeal+tests, migration 00211 com trigger Σ ≤ 100 ERRCODE 23514, entity Unit ampliada (backward compat), 17 tests novos. ⏳ refactor sqlc/mapper/usecase/handler ~1-2h sequencial |
| **6** | Flutter D-048 + D-049 deps | ✅ | bloc_concurrency, hydrated_bloc, freezed, freezed_annotation, json_*, path_provider. analyze 0 issues, 27 tests passed |
| **7** | Vault updates | ✅ | audit-log do dia, web/_moc, web/tasks, backend/tasks BE-001 partial, mobile/_moc, AUDIT.md A-035 e A-FASE10-DEV-005 atualizados |
| 3 | UnoCSS preset compartilhado + cleanup 5 apps base | 🟡 deferred | Não bloqueia M1; documentado em web/tasks.md FE-017 + DT-WEB-024 abertos |

### Métricas finais

| Métrica | Antes | Depois (Agente 2) |
|---|---|---|
| `bun run check` (web) | 6 errors + 14 warnings + 1 info | **0 errors + 0 warnings + 1 info** |
| Testes web | 80 (52+13+15) | **81** (52+13+16) |
| Apps web | 6 | **7** (+ admin port 3010) |
| Backend Unit entity fields | 4 | **6** (+ unit_type, + fracao_ideal) |
| Backend VOs institutional | 1 (Address) | **3** (+ UnitType, + FracaoIdeal) |
| Migrations backend | 39 | **40** (+ 00211) |
| Backend tests novos | — | **17** (entity + VOs) |
| Flutter pubspec deps | 16 | **22** |
| Flutter analyze | (não rodado) | **No issues found** |
| Flutter tests | 27 | 27 (sem regressão) |

### Decisões 🔴 abertas requerendo João/backend

- **PKCE flow** (A-WEB-2026-04-27-003 / DT-WEB-018) — auth-context POST direto sem PKCE; STEERING #18 obrigatório. Confirmar com backend se `/sign-in` é stub temporário (cenário B) ou backend tem 2 fluxos (cenário A) — abrir D-### nova com decisão.
- **OTP via reset_token** (A-WEB-2026-04-27-002 / DT-WEB-020) — `?code=XXXXXX` em URL vaza em histórico/Referer/proxy/CDN. Mitigação backend: emitir reset_token JWT/opaco TTL 5min em `/confirm-reset-code`.

### Próximos passos sugeridos (próxima sessão)

1. **Lote 5 cont.** — sqlc generate + mapper + use case + handler para fechar A-035 100 % (1-2h mecânico).
2. **Lote 3** — extrair `packages/ui/src/uno.preset.ts`; cleanup 5 apps base (initialColorMode dark + remover routes/index.tsx duplicado + railway.json sem turbo).
3. **D-### PKCE** — coordenar com João/backend para resolver A-WEB-003.
4. **D-### OTP reset_token** — coordenar com backend para resolver A-WEB-002.
5. **Adoção Flutter D-048/D-049** — refactor entity → @freezed e bloc → bloc_concurrency.concurrent/droppable por feature (incremental, sem regressão).

### Vizinhos

- [[11-audit/audit-log/2026-04-27-agente-2-systematic-pass]] — pass detalhado com diffs por arquivo
- [[11-audit/audit-log/2026-04-27-consistency-pass]] — pass do Agente 1 (manhã)
- [[STATE]]
- [[AUDIT]]
- [[ROADMAP]]
- [[STEERING]]

---

## Sessão 2026-04-23 — Kickoff da remontagem (histórico)

Histórico preservado para rastreabilidade. Conteúdo original abaixo.

### Ordens originais do usuário

1. "vamos criar uma pasta nova master-sindico, no qual iremos analisar projeto de forma minuciosa de ponta a ponta..."
2. "vamos focar exclusivamente na arquitetura, specs, dominios, DDD, sem codigo..."
3. "quero um research amplo pois a master-sindico abrange diversos produtos"
4. "passar tudo para essa nova pasta... quando tivermos 100% de certeza apagamos o legado"
5. "voce vai atuar como arquiteto de soluções, desenvolvedor full-stack senior, QA, pentester, e arquiteto de soluções empresarial de arquitetura de empresas beyond-corp"
6. "voce vai ler a parte antiga, tudo que referenciar o projeto mastersindico nao pode ser ignorado, tem que ser minucioso..."

### Modo de operação

- **Multi-papel**: arquiteto de soluções + full-stack senior + QA + pentester + arquiteto BeyondCorp enterprise.
- **Nada é apagado do legado** até Fase 6.
- **Minucioso, não apressado** — "não temos pressa".
- **Tudo rastreável**: D-### em STATE, A-### em AUDIT, frontmatter em todo .md, links mínimos 2 por nota.
