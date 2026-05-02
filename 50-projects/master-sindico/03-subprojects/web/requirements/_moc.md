---
title: MOC — Web Requirements por App
type: moc
tags: [moc, master-sindico, web, solidjs, requirements, feature-req, per-app]
project: master-sindico
stack: web
created: 2026-04-24
updated: 2026-04-24
---

# MOC — Web Requirements por App

Requirements específicos **por app** do sub-projeto web (6 apps Bun workspaces). Complementa [[../requirements]] (requirements transversais web) e [[../ui-catalog]] (catálogo exaustivo de telas).

**Fontes canônicas**:
- `Development/web/CLAUDE.md` §3 apps (399 linhas)
- `Development/web/AGENTS.md` §Filosofia + §Segurança
- `vault/AUDIT.md` A-FASE10-DEV-005 (status real)
- [[../../../00-product/personas]] (7 personas + sub-roles ABAC)
- [[../../../04-requirements/_moc]] (FRs globais)

---

## 6 Apps × Marco × Status

| Arquivo | App | Porta | Marco | Status real | FR count | Persona-alvo |
|---|---|---|---|---|---|---|
| [[auth]] | auth | :3000 | **M1** | scaffold-stub (🔴 bloqueador) | 22 | Todas (pre-auth) |
| [[cms]] | cms | :3001 | **M1** | scaffold-stub (🔴 bloqueador) | 68 | Síndico, Morador, Empresa, Agência, Admin MS |
| [[lms]] | lms | :3002 | M3 | not-started | 24 | Síndico, Morador (aluno), Admin Academia |
| [[forum]] | forum | :3003 | M2 | not-started | 28 | Morador, Síndico, Empresa (Vizinhança) |
| [[assembly]] | assembly | :3004 | M3 | not-started | 32 | Síndico, Morador (votante), Secretário |
| [[lp]] | lp | :3005 | **M1** | em progresso (hero ✅) | 20 | Público anônimo |

**Total**: 194 FRs específicos por app + NFRs transversais por [[../requirements]].

---

## Dependências entre apps

```
lp  (público)
  └─▶ auth (OIDC Zitadel)
       └─▶ cms ──┬──▶ forum (M2)
                  ├──▶ lms (M3)
                  └──▶ assembly (M3)
```

- **lp** é standalone mas linka `/auth/sign-in` via subdomínio.
- **auth** é único ponto de entrada autenticado; emite cookie `.mastersindico.com.br` que cobre todos os apps.
- **cms** é o hub: links para forum/lms/assembly ficam em sidebar ou menu contextual.
- **forum/lms/assembly** podem ser chamados standalone via deep link (ex: `https://assembly.mastersindico.com.br/live/:id`).

---

## Status real (AUDIT.md A-FASE10-DEV-005)

| Item | Status | Nota |
|---|---|---|
| 5/6 apps são placeholders `<div>Hello <app>!</div>` | 🔴 | Somente `lp` tem hero real |
| `AuthProvider`, `QueryClientProvider`, `Toaster` comentados | 🔴 | TODOS os `index.tsx` |
| `errorComponent`, `notFoundComponent` comentados | 🔴 | TanStack Router rootRoute |
| Zero testes em todos apps | 🔴 | Vitest/MSW/Playwright NÃO instalados |
| `@tanstack/solid-query`, `@tanstack/solid-form` NÃO instalados | 🔴 | `bun.lock` confirma |
| `packages/ui` Button ✅ apenas | 🟡 | 20+ componentes faltam |
| `packages/schemas` stub | 🔴 | Zod schemas vazios |

Cada MD registra esse status no bloco `## Status atual (real)`.

---

## Convenção de IDs

- `FR-WEB-AUTH-001` a `FR-WEB-AUTH-099` — auth app
- `FR-WEB-CMS-001` a `FR-WEB-CMS-299` — cms (maior app)
- `FR-WEB-LMS-001` a `FR-WEB-LMS-099`
- `FR-WEB-FORUM-001` a `FR-WEB-FORUM-099`
- `FR-WEB-ASM-001` a `FR-WEB-ASM-099`
- `FR-WEB-LP-001` a `FR-WEB-LP-099`
- `NFR-WEB-<APP>-<slug>` para não-funcionais

---

## Links

- [[../../../_moc]] — MOC web
- [[../../../CLAUDE]] — contrato agente v2
- [[../architecture]]
- [[../design-system]]
- [[../security]]
- [[../requirements]] — requirements transversais web
- [[../ui-catalog]] — catálogo de telas (141+)
- [[../../../04-requirements/_moc]] — requirements globais master-sindico
