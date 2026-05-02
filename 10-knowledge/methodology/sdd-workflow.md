---
title: SDD Workflow — ciclo de 5 fases
type: concept
tags: [methodology, sdd, workflow]
source: .kiro/steering/sdd-workflow.md
created: 2026-04-22
aliases: [SDD, Spec-Driven Development, Ciclo SDD, 5 fases]
---

# SDD Workflow — ciclo de 5 fases

**SDD (Spec-Driven Development)** é o processo oficial do automation-agents. Inspirado em [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) e [GitHub spec-kit](https://github.com/github/spec-kit), adota 3 conceitos específicos (não o framework completo):

1. **Ciclo explícito de 5 fases** por task (Discuss → Plan → Execute → Verify → Ship)
2. **`<verify>` declarativo** — testes e critérios de aceite declarados **antes** da implementação (casa com TDD)
3. **STATE.md como arquivo vivo** de bloqueadores, decisões em aberto e dívidas técnicas

Spec canônica vive em `specs/<produto>/` (requirements + design + tasks + reference). **Sem spec = sem implementação.** Pular fases produz retrabalho.

## O ciclo

```
Discuss  →  Plan  →  Execute  →  Verify  →  Ship   →  Audit (fim de sprint)
 contexto   plano    código     testes +   PR +      finding → AUDIT.md
 + spec     + verify + tests    coverage   review
```

## Fase 1 — Discuss

**Antes de tocar qualquer arquivo**, ler nesta ordem:

1. `specs/<produto>/tasks/<sprint>.md` — descrição da task
2. `specs/<produto>/requirements/*.md` relacionado — o quê e por quê
3. `specs/<produto>/design/*.md` relacionado — como (interfaces, contratos, fluxos)
4. Código existente que vai ser modificado (`src/modules/<ctx>/` ou equivalente)
5. `specs/<produto>/reference/antipatterns-to-avoid.md` — o que deu errado antes
6. `STATE.md` — bloqueadores e decisões em aberto que impactam a task

**Não** avançar se qualquer item está incompleto ou ambíguo. Atualizar spec **antes** de implementar.

## Fase 2 — Plan

Escrever plano **explícito** com blocos:

```xml
<plan task="<id-da-task>: <título>">
  <files>
    <path>                  [novo|editar — motivo]
  </files>

  <docs-oficiais-via-context7>
    - <lib>@<versão>: topic "<topic>"
    - Context7: resolve-library-id("<lib>") → get-library-docs(...)
  </docs-oficiais-via-context7>

  <verify>
    - Teste unitário: <descrição do comportamento>
    - Teste integração: <descrição do fluxo>
    - Gates: <build>, <lint>, <codegen>, <coverage target>
  </verify>

  <subagents>
    - Se task explode para 10+ arquivos, delegar por camada
    - Subagent retorna APENAS resumo, não polui contexto principal
  </subagents>
</plan>
```

**Regra dura**: task com SDK externo → plano declara **qual fonte oficial** foi consultada via Context7 MCP (**não memória de treino**). Ver [[../../30-agents/mcp-integration]].

## Fase 3 — Execute

**Ordem canônica de implementação** (adaptar nomes ao stack):

```
1. Migration SQL + CHECK constraints
   └── Numeração particionada por módulo (ex: identity 001-099, billing 100-199)

2. Query tipada / codegen (sqlc, Prisma, drizzle-kit, etc.)

3. Domain entity / value object
   └── Estado privado, factory NewX/ReconstructX, métodos de domínio

4. Repository interface (domínio)

5. Domain event (se cruzar bounded contexts)

6. Use case (CQRS — command ≠ query, arquivo separado)
   └── Compile-time interface assertion

7. Mapper (entidade → DTO)

8. Repository implementation (infraestrutura)
   └── Suporte a transação via context

9. Provider externo (SDK isolado)
   └── Interface no domain/providers, impl em infrastructure/providers

10. Handler + rota (contratos abstratos, não framework cru)

11. Wire-up no module + main

12. Testes (unidade + integração)
```

**Nunca** codificar fora dessa ordem. Pular migration antes do código é o erro mais caro.

### Orchestrator-Worker pra tasks grandes

Tasks que envolvem **módulo inteiro** são **delegadas em subagentes**:

```
Main agent (orquestrador)
 ├── Subagent 1: domain/ — entities, VOs, repo interfaces, events
 ├── Subagent 2: application/ — use cases CQRS, mappers
 ├── Subagent 3: infrastructure/database/ — repo impl, queries
 └── Subagent 4: infrastructure/http/ — handlers, routes, wire-up
```

Cada subagente recebe: task do spec, arquivos já criados pelos anteriores, critério de aceitação específico. Main agent **apenas coordena e revisa** — não polui contexto com detail de cada camada.

## Fase 4 — Verify

Rodar localmente **nesta ordem** antes de considerar task pronta:

```bash
<build> ./...                                   # compila sem erro
<lint/vet> ./...                                # sem warning
<codegen> && git diff --exit-code <gen-path>    # generated em dia
<test> -count=1 ./...                           # unit
<test> -tags=integration -count=1 ./...         # integração
<coverage-tool> --threshold-file=.coverage.yml  # thresholds por camada
```

**Thresholds mínimos canônicos**:

| Camada | Mínimo |
|---|---|
| `domain/` (entities, VOs, services) | **95%** |
| `application/` (use cases, orquestração) | **90%** |
| `infrastructure/database/` (repo) | **85%** |
| `infrastructure/http/` (handlers) | **75%** |
| `shared/middleware/` | **90%** |
| `core/` / `pkg/` | **95%** |
| **Global** | **≥ 85%** |

Task que **não** atende threshold **não é mergeada**. Se threshold é injusto pro caso (ex: handler 74% por error branch impossível), documentar no plano com `<verify-exception>` e justificar.

### Security gates (task de segurança)

Task envolvendo auth, billing, webhook, input externo, dados sensíveis:
- Security scanner `-severity high` passa
- Vuln check sem CVE aberto
- Validação input server-side **sempre**
- Webhook signature validada **antes** de parsing
- Secrets **nunca** em código, commit, log, erro

## Fase 5 — Ship

1. **Commit** com mensagem imperativa, prefixada pelo domínio quando escopo claro:
   ```
   billing: Add Stripe webhook signature validation
   ```
2. **Atualizar README do sub-projeto** quando mudança é feature-level (não micro-fix):
   - Backend: novo endpoint, tabela, domínio, SDK integrado, invariante ABAC, contrato cross-module
   - Frontend: novo app, nova rota, nova feature flag, nova env var
   - Mobile: novo feature slice, novo provider, nova tela, integração nativa
   - Seções a manter: stack, endpoints, env vars, comandos, status de sprint
   - **Bug-fix não atualiza README. Feature sempre atualiza.**
3. **Abrir PR** via GitHub MCP
4. **PR body** com seção `Release Notes:` obrigatória
5. **CI valida**: build + lint + test + coverage + security scanner + vuln check
6. **Review**: pelo menos 1 pessoa; em trabalho solo, aguardar 30min antes de merge pra reler com olhos frescos

## Fase 6 — Sprint Audit (gate antes do próximo sprint)

Ao fim de cada sprint, **antes** de qualquer task do próximo, rodar skill **`sprint-auditor`**. Lê AUDIT.md, revisa sistematicamente o entregue (seguindo `go-review.md` ou equivalente), materializa findings em AUDIT.md.

```
Sprint N tasks ✅ → sprint-auditor → AUDIT.md com findings
                                      ↓
  task-executor próxima sessão lê AUDIT.md PRIMEIRO
                                      ↓
          itens 🔴/🟡 abertos? — resolver ANTES de task nova
                                      ↓
          sprint-auditor re-valida (modo fechamento) → move ✅
                                      ↓
                  próximo sprint liberado pra Discuss
```

**Regras duras**:
- `task-executor` é **obrigado** a abrir AUDIT.md no início de cada sessão. Itens 🔴/🟡 **bloqueiam** task nova.
- `sprint-auditor` **nunca** implementa fix — apenas aponta. Fix é do `task-executor` na sessão seguinte (micro-task com ciclo 5-fase reduzido).
- Distinção vs STATE:
  - **STATE** = bloqueadores/decisões/dívidas **conscientes** (com sprint alvo)
  - **AUDIT** = bugs/má implementação **descobertos em revisão** (sem sprint alvo — prioridade máxima)
- Severidades em AUDIT: `critical` (bloqueia deploy) → `high` (bug funcional) → `medium` (cheiro forte) → `low` (cosmético). `critical` e `high` **bloqueiam** próximo sprint.

## `<verify>` é contrato, não sugestão

Bloco `<verify>` do plano é **compromisso**. Se você escreveu "teste unitário X passa", esse teste **existe** ao final. **Não existe "depois eu faço"** — se ficar fora do escopo, documentar explicitamente em STATE como dívida.

## STATE.md — arquivo vivo

STATE rastreia o que o `tasks.md` (estático) não comporta:
- Bloqueadores no momento (com responsável, data, impacto)
- Decisões em aberto (com alternativas e prazo pra resolver)
- Dívidas técnicas registradas (com sprint alvo pra pagar)
- Questões pendentes de produto que afetam spec

Atualizar STATE **sempre** que descobrir bloqueador, abrir decisão, registrar débito. Sem isso, próxima sessão do agente tromba no mesmo muro.

## Plugins Claude Code úteis por fase

| Fase | Plugins |
|---|---|
| Discuss | `context7`, `feature-dev` (exploration agent), `claude-md-management` |
| Plan | `superpowers` (`/brainstorm`, `/write-plan`), `feature-dev` (architecture design) |
| Execute | LSP do stack (`gopls-lsp`, `typescript-lsp`), `figma` (design-to-code), SDK-specific |
| Verify | `code-review` (scoring), `security-guidance` (pre-tool hook), `superpowers` (TDD red/green) |
| Ship | `commit-commands` (`/commit`, `/push`, `/pr`), MCP do repo host, plataforma de deploy |

## Regras de PR

- Título imperativo, capitalizado, **sem prefixos `fix:`/`feat:`**, sem pontuação final
- Prefixo de domínio quando escopo claro: `billing: Add X`
- Corpo descreve **o quê e por quê**, não como
- Sempre seção `Release Notes:` no final
- Release Notes vira CHANGELOG ao publicar

## Não fazer

- ❌ Commitar `.env` ou qualquer arquivo com secret
- ❌ Editar codegen manualmente
- ❌ Abstração sem 2+ implementações reais (YAGNI)
- ❌ Features não solicitadas ("enquanto estou aqui…")
- ❌ Lógica de negócio em handlers ou middleware
- ❌ `any`/`interface{}` onde tipo concreto serve
- ❌ Pular Discuss porque "já sei o que fazer"

## Links

- [[_moc]] — metodologia
- [[legacy-as-reference]] — como usar projeto anterior sem contaminar
- [[../principles/do-dont-checklist]] — checklist consolidado
- [[../antipatterns/_moc]] — catálogo AP-### a evitar
- [[../../30-agents/autonomous-execution]] — modo não-supervisionado
- [[../../30-agents/mcp-integration]] — Context7 é obrigatório
- [[../../30-agents/claude-code-plugins]] — mapeamento plugins × fases
