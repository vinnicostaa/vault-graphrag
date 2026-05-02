---
title: SDD — Checklist de maturidade de adoção
type: guide
tags: [methodology, sdd, gsd, checklist, adoption]
created: 2026-04-22
updated: 2026-04-23
aliases: [sdd-maturity, SDD coverage checklist, espírito SDD]
---

# SDD — Checklist de maturidade de adoção

Checklist atemporal pra medir quanto um projeto **já incorpora o espírito** do Spec-Driven Development (independente da escola — ver [[kiro-vs-spec-kit]]). Use em auditoria de projeto ou onboarding de equipe nova.

A filosofia canônica é [[spec-driven-development]]: _"Specifications are the source of truth; code is the artifact that implements the spec"_.

> **Como usar**: marcar ✅ (implementado), 🟡 (parcial), ❌ (ausente). Projeto maduro tem ≥ 8/10 itens. < 5/10 = spec está sendo tratada como documentação opcional, não fonte de verdade.

---

## Itens do checklist

### 1. Spec é fonte de verdade (hard rule)

- Existe regra dura "sem spec = sem implementação" em doc de steering/constitution
- PR que muda invariante **sem** atualizar spec é rejeitado em review
- Drift entre spec e código é tratado como bug (não como "documentação desatualizada")

### 2. Decomposição por bounded context ou feature

- Specs não são um arquivão monolítico; estão recortadas por contexto/domínio/feature
- Cada recorte tem frontmatter versionado (`version:`) que muda quando a spec muda
- `functional-matrix` ou `cross-domain` conecta specs que compartilham invariantes

### 3. Plan + `<verify>` antes de codar

- Fase de planejamento explícita antes de execução (Spec Kit: `plan.md`; Kiro: `design.md`; ciclo 5-fase: Fase 2)
- Critério de pronto declarativo (`<verify>` bloco, acceptance criteria, ou equivalente)
- Plan é revisado antes de gerar tasks — não começa a codar com plan ambíguo

### 4. Tasks estruturadas em ciclo discreto

- Formato de task único (identificadores estáveis, gates duros, checklist)
- Gates automatizados: build, lint, vet, testes, coverage, SAST (gosec/semgrep), SCA (govulncheck/npm audit)
- Task referencia a spec que a originou (rastreabilidade 2-way)

### 5. STATE vivo (decisões + dívida)

- Arquivo dedicado (STATE.md, DECISIONS.md) com D-### ou ADRs pequenos por decisão aberta
- Dívidas técnicas catalogadas (DT-###, TD-###) e priorizadas
- Atualizado a cada sessão significativa — não é documentação de outubro

### 6. AUDIT vivo (bugs descobertos em revisão)

- Arquivo dedicado (AUDIT.md) com A-### por finding
- Severidade explícita (🔴 bloqueante / 🟡 médio / 🟢 nice-to-have)
- Gate 0 de toda sessão: AUDIT 🔴/🟡 aberto bloqueia task nova

### 7. Reference layer (lições do legado)

- Catálogo de antipatterns observados em legacy code (AP-### ou equivalente)
- Sumário de regras de negócio validadas em produção (legacy como fonte de verdade **parcial**) — ver [[legacy-as-reference]]
- Mapa de onde o legacy documenta cada regra (pra consultar sem ler 500K LOC)

### 8. Sprint Audit (ou equivalente)

- Revisão formal antes de começar próximo sprint/milestone
- Skill/agente automatizado roda o audit (não depende de vontade humana)
- Saída: lista de ações pro próximo ciclo

### 9. Constitution (explícita ou rastreável)

- Ou um `CONSTITUTION.md` explícito (Spec Kit style)
- Ou um conjunto rastreável de steering files com índice condensado
- Lista os princípios **invioláveis**: padrões inegociáveis, categorias destrutivas que pausam autonomia, hierarquia de fontes de verdade
- Ver gap G1 em [[sdd-adoption-gaps]] se este item falha

### 10. Workflow commands/skills

- Comandos repetíveis (slash commands, skills, tasks) pras fases do ciclo
- Pode ser manual (guiado por agente) ou automatizado (Spec Kit slash)
- Automação opcional ganha força conforme modo autônomo cresce — ver gap G2 em [[sdd-adoption-gaps]]

---

## Sinais de imaturidade (red flags)

- ❌ "A spec existe mas o código diverge em 30% do que ela diz"
- ❌ "Decisões de arquitetura nascem em PR description, não em spec + D-###"
- ❌ "AUDIT findings ficam abertos indefinidamente sem SLA"
- ❌ "Não sabemos quais são os princípios invioláveis do projeto"
- ❌ "Refactors grandes não acionam update de spec correspondente"
- ❌ "Onboarding de dev novo depende de reunião, não de ler specs"

---

## Exemplos de aplicação

- **Master Síndico** (Go+DDD+Clean Arch, 2026): 8/10 itens ✅ — ver [[../../50-projects/master-sindico/06-execution/sdd-gsd-aplicado]] seção 1

---

## Links no grafo

- [[_moc]]
- [[spec-driven-development]] — filosofia canônica
- [[kiro-vs-spec-kit]] — as duas escolas em 2026
- [[sdd-workflow]] — ciclo 5-fase operacional
- [[sdd-adoption-gaps]] — gaps típicos em adoção
- [[legacy-as-reference]] — como usar legado na camada Reference
- [[graph-rag]] — como este vault indexa o conhecimento
