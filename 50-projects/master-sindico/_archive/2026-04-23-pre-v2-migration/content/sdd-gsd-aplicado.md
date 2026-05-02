---
title: "SDD + GSD aplicado ao Master Síndico (versão histórica)"
updated: 2026-04-23
maintainer: agent-3 (analista)
type: guide
status: historical
aliases: [sdd-gsd-aplicado-historical]
---

> **⚠️ Versão histórica preservada (sem excluir)**. A versão **canônica** (atualizada) desse documento vive agora em [[../06-execution/sdd-gsd-aplicado]] — com links para os nós genéricos destilados em `10-knowledge/methodology/`:
> - [[../../../10-knowledge/methodology/kiro-vs-spec-kit]]
> - [[../../../10-knowledge/methodology/sdd-maturity-checklist]]
> - [[../../../10-knowledge/methodology/sdd-adoption-gaps]]
>
> Este arquivo **permanece como histórico** (versão 2026-04-22 original, criada pelo agente-3 analista). Não editar daqui pra frente — atualizações vão em `06-execution/`.

---

# SDD + GSD aplicado ao Master Síndico

Síntese das duas correntes principais de **Spec-Driven Development** em 2026 e como o projeto **já as aplica** (com pontos finos a ajustar).

---

## 1. As duas escolas de SDD relevantes em 2026

### Kiro (AWS) — IDE agêntica + workflow `requirements → design → tasks → implementation`

- IDE/CLI da AWS, cloud-agnóstico apesar do nome
- Estrutura `.kiro/specs/<feature>/` com 3 arquivos: `requirements.md`, `design.md`, `tasks.md`
- Inclui *agent hooks* (callbacks de eventos do agente)
- Foco em **estruturar conhecimento da feature** antes de codar
- **Adotado pelo Master Síndico** desde Sprint 0: a estrutura `.kiro/specs/master-sindico/` segue o padrão Kiro (com extensão para `reference/` e `steering/`).

### GitHub Spec Kit (GSD = "GitHub Spec-Driven") — toolkit open-source com workflow `Constitution → Specify → Plan → Tasks`

- CLI distribuída pelo GitHub (84k+ estrelas, 14+ AI agents suportados)
- Quatro artefatos canônicos: `constitution.md` (princípios invioláveis), `spec.md` (o que/por que), `plan.md` (como), `tasks.md` (lista executável)
- *Workflows* orquestram comandos automaticamente com pause em human review gates
- Filosofia chave: **"Specifications are the source of truth; code is the artifact that implements the spec"**
- Templates otimizados por agente (Claude, Cursor, etc)

### Diferenças práticas

| Dimensão | Kiro (atual do projeto) | GitHub Spec Kit |
|---|---|---|
| Diretório raiz | `.kiro/` | `.github/spec/` ou `.spec/` |
| Constituição explícita | Implícita em `steering/` | Explícita em `constitution.md` |
| Workflow autônomo | Manual (skills, sub-agentes) | Comandos slash automatizados |
| Numeração de feature | Manual | Automática + branch creation |
| Agents suportados | Claude Code (principal) | 14+ |

---

## 2. O que o Master Síndico **já faz** (cobertura: ~90% do espírito SDD/GSD)

✅ **Spec é fonte de verdade**: regra dura no `backend/.kiro/steering/sdd-workflow.md` Fase 1 (Discuss) — "sem spec = sem implementação". Comprovado por D-030 (deletar `web/.kiro/specs/` stale ao detectar drift).

✅ **Decomposição por bounded context**: `requirements/{identity, billing, institutional, commercial, content, assembly, cross-domain, personas-and-plans, functional-matrix}.md` — não é o `spec.md` único do Spec Kit, é melhor (cada bounded context tem seu recorte versionado).

✅ **Plan + `<verify>` declarativo antes de codar** — `sdd-workflow.md` Fase 2. Equivalente ao `plan.md` do Spec Kit, mas embutido na task em vez de arquivo separado (mais ergonômico para sprint loops).

✅ **Tasks com formato 5-fase (Discuss → Plan → Execute → Verify → Ship)** — `tasks/README.md` define formato único, identificadores estáveis (`sprint.task.sub`), checklist de gates duros (build/vet/test/coverage/gosec/govulncheck).

✅ **STATE.md vivo** — captura D-0XX (decisões em aberto) e DT-0XX (dívidas), atualizado a cada sessão. AUDIT.md captura A-0XX (bugs descobertos em revisão).

✅ **Reference layer** — `reference/{antipatterns-to-avoid, legacy-summary, obsidian-mapping}.md` — guarda lições do legacy + mapa do Obsidian externo. Vai além do Spec Kit (que não tem essa camada).

✅ **Sprint Audit (Fase 6)** — `/sprint-auditor` skill obriga revisão antes do próximo sprint. Não tem equivalente direto no Spec Kit.

✅ **Steering (regras duras carregadas sob demanda)** — 13 arquivos em `backend/.kiro/steering/` com regras versionadas por tópico. **Esta é a Constituição implícita** do Spec Kit.

---

## 3. O que falta (gaps em relação a GSD/Spec Kit)

### G1 — Sem `constitution.md` explícito

- **Problema**: a "Constitution" do projeto vive **distribuída** em 13 arquivos de `steering/`. Bom para granularidade, mas falta um índice/condensação que liste em uma página os princípios invioláveis.
- **Recomendação leve**: criar `backend/.kiro/CONSTITUTION.md` (ou renomear `steering/do-dont.md` para `CONSTITUTION.md`) com:
  - Lista das 10 Regras de Ouro (já em `product-overview.md`)
  - Padrões de engenharia inegociáveis (DDD, TDD, Clean Arch, CQRS, SOLID, Code Calisthenics, ACID, Saga, no-`else`)
  - 5 categorias destrutivas que pausam modo autônomo (já em `steering/autonomous-execution.md`)
  - Hierarquia de fontes de verdade (já em `CLAUDE.md`)
- **Não é obrigatório** — funciona sem. É polish.

### G2 — Workflow command/slash não declarativo

- **Problema**: o Spec Kit tem comandos `/specify`, `/plan`, `/tasks` que avançam fases automaticamente. O projeto tem skills (`/task-executor`, `/sprint-auditor`, `/dual-validate`, `/go-review`, `/migration-writer`, `/sqlc-writer`) mas o ciclo 5-fase é principalmente manual ou guiado por agente.
- **Recomendação opcional**: se modo autônomo de longa duração (10h+) for recorrente, considerar uma skill `/sdd-cycle` que execute as 5 fases sem intervenção, com checkpoints de aprovação opcional. Não vital.

### G3 — Terminologia "GSD" ambígua

- **Problema** (ver [[regras-de-negocio/quebras-detectadas]] Q18): `steering/sdd-workflow.md` cita "gsd-build/get-shit-done", mas a indústria em 2026 chama "GSD" o GitHub Spec-Driven (Spec Kit). Risco de confundir leitores que pesquisarem GSD externamente.
- **Recomendação**: adicionar **uma linha** em `steering/sdd-workflow.md` após a citação: _"Não confundir com 'GitHub Spec-Driven Development' do [GitHub Spec Kit](https://github.com/github/spec-kit), padrão da indústria que usa workflow Constitution → Specify → Plan → Tasks — projeto adota a mesma filosofia ('spec é fonte de verdade'), mas com taxonomia Kiro."_
- 2 minutos de fix, alto retorno cognitivo.

### G4 — `<verify>` declarativo não é parseável por máquina

- **Problema**: o bloco `<verify>` é XML-like dentro de Markdown, lido por humano e agente. Spec Kit tem o `tasks.md` parseable.
- **Recomendação**: nada urgente. Se quiser automação CI futura (gate de "task tem `<verify>` válido?"), considerar formato YAML ou checkbox-only. Por ora, MD humano funciona melhor para LLM consumir.

---

## 4. Recomendações priorizadas para os outros agentes (operacionais)

### Para o agente 1 (executor)

1. **Antes de cada task, abrir AUDIT.md e STATE.md** (já é regra) — garantir que itens 🔴/🟡 abertos não vão ser ignorados.
2. **Quando ler `product-overview.md`, ter em mente Q1** (stack desatualizada — usar Resend/Railway/PG tsvector como reais).
3. **Quotas Connect Me Morador**: enquanto Q2 não for resolvida, **codificar Decision 10** (Base 2/ano, Pagante 4/ano) — é a fonte mais autoritativa.
4. **Não criar adapters de SES, OpenSearch, ECS** — stack atual já decidida.
5. **Antes de implementar qualquer feature de empresa "Base"**, ler Q3 — provavelmente o conceito não existe.

### Para o agente 2 (revisor)

1. **Validar que cada PR atualiza spec quando muda invariante** (alinhamento Spec Kit principle).
2. **Atualizar `version:` no frontmatter quando spec muda** (já é regra, mas reforço).
3. **Cruzar `functional-matrix` ↔ `personas-and-plans` ↔ `commercial.md` em cada review de quota** — risco de re-introduzir Q2.
4. Quando uma decisão vira fix de quebra detectada aqui, **mover entrada para "Resolvidos"** em [[regras-de-negocio/quebras-detectadas]] e criar D-0XX em `STATE.md`.

### Para o agente principal (coordenador)

1. **Decidir Q2 com João** (quota Morador Base Connect Me) — é Critical e bloqueia ABAC consistente.
2. **Aplicar Q1** (3 substituições no `product-overview.md`) — 5 min, alto valor.
3. **Decidir Q4** (agência de marketing tem login ou não) — abrir D-0XX em STATE.md.
4. **Considerar G1** (criar `CONSTITUTION.md` explícito) — opcional, polish.
5. **Aplicar G3** (1 linha esclarecendo GSD vs Spec Kit) — 2 min.

---

## 5. Referências

- [GitHub Spec Kit](https://github.com/github/spec-kit) — repositório oficial
- [Spec-Driven Development (spec-kit doc)](https://github.com/github/spec-kit/blob/main/spec-driven.md) — manifesto
- [Martin Fowler — Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — comparação canônica
- [Kiro (kiro.dev)](https://kiro.dev/) — IDE agêntica
- [Microsoft Learn — Get Started with Spec-Driven Development and GitHub Spec Kit](https://learn.microsoft.com/en-us/training/modules/spec-driven-development-github-spec-kit-greenfield-intro/) — tutorial oficial
- [DeepWiki — Spec-Driven Development Methodology](https://deepwiki.com/github/spec-kit/3-spec-driven-development-methodology) — análise estrutural
- `backend/.kiro/steering/sdd-workflow.md` — implementação interna do projeto
- `backend/.kiro/steering/sdd-layers.md` — SDD por camada (domain/application/infrastructure)
