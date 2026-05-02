---
title: SDD + GSD aplicado ao Master Síndico
type: guide
tags: [methodology, sdd, gsd, master-sindico, execution]
project: master-sindico
created: 2026-04-22
updated: 2026-04-22
aliases: [sdd-gsd-aplicado, SDD GSD no projeto]
---

# SDD + GSD aplicado ao Master Síndico

Como o projeto **aplica** as duas correntes principais de Spec-Driven Development (Kiro + GitHub Spec Kit), estado atual de cobertura, gaps e recomendações operacionais.

> Conceitos atemporais vêm de 10-knowledge/methodology (Graph-RAG):
> - [[../../../10-knowledge/methodology/kiro-vs-spec-kit]] — as duas escolas de SDD em 2026
> - [[../../../10-knowledge/methodology/sdd-maturity-checklist]] — checklist de 10 itens (projeto marca 8/10 abaixo)
> - [[../../../10-knowledge/methodology/sdd-adoption-gaps]] — catálogo G-### de gaps típicos (G-001 e G-003 abertos no projeto)
>
> Aqui fica **só a aplicação concreta** ao Master Síndico. Lições generalizáveis já foram destiladas pra os arquivos acima.

---

## 1. Estado atual (cobertura: 8/10 — ver [[../../../10-knowledge/methodology/sdd-maturity-checklist|checklist genérico]])

✅ **Spec é fonte de verdade** — regra dura em `backend/.kiro/steering/sdd-workflow.md` Fase 1 (Discuss): "sem spec = sem implementação". Comprovado por **D-030** (deletar `web/.kiro/specs/` stale ao detectar drift).

✅ **Decomposição por bounded context** — `requirements/{identity, billing, institutional, commercial, content, assembly, cross-domain, personas-and-plans, functional-matrix}.md`. Não é o `spec.md` único do Spec Kit — é mais granular (cada bounded context versionado separado).

✅ **Plan + `<verify>` declarativo antes de codar** — `sdd-workflow.md` Fase 2. Equivalente ao `plan.md` do Spec Kit, mas embutido na task em vez de arquivo separado (mais ergonômico para sprint loops).

✅ **Tasks com formato 5-fase (Discuss → Plan → Execute → Verify → Ship)** — `tasks/README.md` define formato único, identificadores estáveis (`sprint.task.sub`), checklist de gates duros (build/vet/test/coverage/gosec/govulncheck).

✅ **STATE.md vivo** — captura D-0XX (decisões em aberto) e DT-0XX (dívidas), atualizado a cada sessão. AUDIT.md captura A-0XX (bugs descobertos em revisão).

✅ **Reference layer** — `reference/{antipatterns-to-avoid, legacy-summary, obsidian-mapping}.md`. Guarda lições do legacy + mapa do Obsidian externo. Vai além do Spec Kit (que não tem essa camada).

✅ **Sprint Audit (Fase 6)** — `/sprint-auditor` skill obriga revisão antes do próximo sprint. Sem equivalente direto no Spec Kit.

✅ **Steering (regras duras carregadas sob demanda)** — 13 arquivos em `backend/.kiro/steering/` com regras versionadas por tópico. **É a Constituição implícita** do Spec Kit.

---

## 2. Gaps em relação a GSD/Spec Kit

Cada gap local mapeia em um genérico do catálogo [[../../../10-knowledge/methodology/sdd-adoption-gaps]]:

| Local | Genérico | Status |
|---|---|---|
| G1 | [[../../../10-knowledge/methodology/sdd-adoption-gaps#g-001 — constitution implícita em vez de explícita\|G-001]] | aberto, opcional |
| G2 | [[../../../10-knowledge/methodology/sdd-adoption-gaps#g-002 — workflow manual em vez de slash-skill automatizado\|G-002]] | aberto, opcional |
| G3 | [[../../../10-knowledge/methodology/sdd-adoption-gaps#g-003 — terminologia ambígua gsd spec plan\|G-003]] | aberto, **2 min de fix** |
| G4 | [[../../../10-knowledge/methodology/sdd-adoption-gaps#g-004 — verify acceptance não-parseáveis por máquina\|G-004]] | aberto, não urgente |

### G1 — Sem `constitution.md` explícito

- **Problema**: a Constitution do projeto vive **distribuída** em 13 arquivos de `steering/`. Bom para granularidade; falta um índice/condensação de uma página com os princípios invioláveis.
- **Recomendação leve**: criar `backend/.kiro/CONSTITUTION.md` (ou renomear `steering/do-dont.md`) com:
  - 10 Regras de Ouro (já em `product-overview.md`)
  - Padrões inegociáveis (DDD, TDD, Clean Arch, CQRS, SOLID, Code Calisthenics, ACID, Saga, no-`else`)
  - 5 categorias destrutivas que pausam modo autônomo (já em `steering/autonomous-execution.md`)
  - Hierarquia de fontes de verdade (já em `CLAUDE.md`)
- **Prioridade**: opcional. Funciona sem.

### G2 — Workflow slash não declarativo

- **Problema**: Spec Kit tem `/specify`, `/plan`, `/tasks` que avançam fases automaticamente. O projeto tem skills (`/task-executor`, `/sprint-auditor`, `/dual-validate`, `/go-review`, `/migration-writer`, `/sqlc-writer`) mas o ciclo 5-fase é principalmente manual ou guiado por agente.
- **Recomendação**: se modo autônomo de longa duração (10h+) for recorrente, considerar skill `/sdd-cycle` que execute as 5 fases sem intervenção, com checkpoints opcionais. Não vital.

### G3 — Terminologia "GSD" ambígua

- **Problema** (ver [[../11-audit/quebras-detectadas-2026-04-22]] Q18): `steering/sdd-workflow.md` cita "gsd-build/get-shit-done", mas a indústria em 2026 chama "GSD" o GitHub Spec-Driven (Spec Kit). Risco de confundir leitores externos.
- **Recomendação**: adicionar **uma linha** em `steering/sdd-workflow.md` após a citação: _"Não confundir com 'GitHub Spec-Driven Development' do [GitHub Spec Kit](https://github.com/github/spec-kit), padrão da indústria que usa workflow Constitution → Specify → Plan → Tasks — projeto adota a mesma filosofia ('spec é fonte de verdade'), mas com taxonomia Kiro."_
- **Prioridade**: 2 min de fix, alto retorno cognitivo.

### G4 — `<verify>` declarativo não é parseável por máquina

- **Problema**: o bloco `<verify>` é XML-like dentro de Markdown, lido por humano e agente. Spec Kit tem o `tasks.md` parseable.
- **Recomendação**: nada urgente. Se quiser automação CI futura (gate "task tem `<verify>` válido?"), considerar YAML ou checkbox-only. Por ora, MD humano funciona melhor pra LLM consumir.

---

## 3. Recomendações operacionais por papel

### Agente executor

1. **Antes de cada task, abrir AUDIT.md e STATE.md** (já é regra). Garante que 🔴/🟡 abertos não são ignorados.
2. **Ao ler `product-overview.md`, ter em mente Q1** — stack desatualizada. Usar Resend/Railway/PG tsvector como reais.
3. **Quotas Connect Me Morador**: enquanto Q2 não for resolvida, **codificar Decision 10** (Base 2/ano, Pagante 4/ano). Fonte mais autoritativa.
4. **Não criar adapters de SES, OpenSearch, ECS** — stack atual já decidida.
5. **Antes de implementar feature de empresa "Base"**, ler Q3 — provavelmente o conceito não existe.

### Agente revisor

1. **Validar que cada PR atualiza spec quando muda invariante** (alinhamento Spec Kit principle).
2. **Atualizar `version:` no frontmatter quando spec muda** (já é regra; reforço).
3. **Cruzar `functional-matrix` ↔ `personas-and-plans` ↔ `commercial.md` em cada review de quota** — risco de re-introduzir Q2.
4. Quando uma decisão resolve quebra detectada aqui, **mover entrada para "Resolvidos"** em [[../11-audit/quebras-detectadas-2026-04-22]] e criar D-0XX em `STATE.md`.

### Agente coordenador

1. **Decidir Q2 com João** (quota Morador Base Connect Me) — Critical, bloqueia ABAC consistente.
2. **Aplicar Q1** (3 substituições no `product-overview.md`) — 5 min, alto valor.
3. **Decidir Q4** (agência de marketing tem login ou não) — abrir D-0XX em STATE.md.
4. **Considerar G1** (criar `CONSTITUTION.md` explícito) — opcional, polish.
5. **Aplicar G3** (1 linha esclarecendo GSD vs Spec Kit) — 2 min.

---

## 4. Referências

- [[../../../10-knowledge/methodology/kiro-vs-spec-kit]] — conceitos atemporais das duas escolas
- [[../../../10-knowledge/methodology/sdd-workflow]] — ciclo operacional 5-fase (genérico)
- [[../../../10-knowledge/methodology/spec-driven-development]] — filosofia SDD destilada
- [[../11-audit/quebras-detectadas-2026-04-22]] — Q1, Q2, Q3, Q4, Q18 referenciadas acima
- [[../01-domain/regras-canonicas]] — regras canônicas do projeto
- [[../STATE]] — D-0XX e DT-0XX vivos
- `backend/.kiro/steering/sdd-workflow.md` — implementação interna
- `backend/.kiro/steering/sdd-layers.md` — SDD por camada (domain/application/infrastructure)

### Fontes externas

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [Martin Fowler — Understanding SDD: Kiro, spec-kit, and Tessl](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
- [Kiro.dev](https://kiro.dev/)
- [Microsoft Learn — SDD with Spec Kit](https://learn.microsoft.com/en-us/training/modules/spec-driven-development-github-spec-kit-greenfield-intro/)

---

## 5. Histórico

- 2026-04-22 — criado em `50-projects/master-sindico/content/sdd-gsd-aplicado.md` (agente-3 analista)
- 2026-04-22 — promovido equivocadamente pra `10-knowledge/methodology/sdd-gsd.md` (reorg vault)
- 2026-04-22 — **devolvido** pra `50-projects/master-sindico/06-execution/sdd-gsd-aplicado.md` (corrigindo isolamento de conteúdo específico); conceitos genéricos seguem em [[../../../10-knowledge/methodology/kiro-vs-spec-kit]]
