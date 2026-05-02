---
title: MOC — 10-knowledge/methodology
type: moc
tags: [moc, methodology, sdd, gsd, tdd, bdd]
created: 2026-04-22
updated: 2026-04-23
---

# 10-knowledge/methodology — Metodologias atemporais

Metodologias de engenharia de software com base conceitual: SDD (Kiro + GitHub Spec Kit), GSD, ciclo 5-fase, TDD/BDD, Graph-RAG, uso de legacy como referência.

Conteúdo aqui é **linguagem-agnóstico e projeto-agnóstico**. Aplicações concretas vivem em `50-projects/<slug>/` e linkam de volta para cá.

---

## Spec-Driven Development (SDD)

Núcleo conceitual:

- [[spec-driven-development]] — manifesto "specs são fonte de verdade; código é artefato"
- [[kiro-vs-spec-kit]] — comparação das duas escolas de 2026 (Kiro vs GitHub Spec Kit) + gaps comuns G1-G4
- [[sdd-workflow]] — ciclo operacional 5-fase (Discuss → Plan → Execute → Verify → Ship)

Adoção em projeto:

- [[sdd-maturity-checklist]] — 10 itens pra medir cobertura SDD no seu projeto
- [[sdd-adoption-gaps]] — catálogo G-### de gaps típicos em adoção (G-001 constitution implícita, G-002 workflow manual, G-003 terminologia ambígua, G-004 `<verify>` não-parseável, G-005 reference layer ausente, G-006 STATE/AUDIT só do tech lead)

Aplicações concretas (exemplos):

- [[../../50-projects/master-sindico/06-execution/sdd-gsd-aplicado]] — SDD + GSD aplicado ao projeto Master Síndico (8/10 maturity; G-001 e G-003 abertos). Versão histórica preservada em [[../../50-projects/master-sindico/content/sdd-gsd-aplicado]].

## Get Shit Done (GSD)

- [[gsd-get-shit-done]] — ergonomia de entrega (ship small ship often, DoD explícito, `<verify>` declarativo, STATE.md vivo, matar tasks zumbi, feedback imediato, WIP baixo)

## Legado como referência

- [[legacy-as-reference]] — padrão de uso de projeto legado na camada Reference (preserva regras de negócio validadas, não decisões arquiteturais ruins)

## Graph-RAG do vault

- [[graph-rag]] — como este vault implementa Graph-RAG (estrutura + links + Smart Connections + Karpathy LLM Wiki pattern)

---

## Vizinhos no grafo

- [[../_moc]] — 10-knowledge raiz
- [[../antipatterns/_moc]] — antipatterns de código (AP-###) — complementa [[sdd-adoption-gaps]] (G-###)
- [[../principles/_moc]] — SOLID, clean-code, code-calisthenics, do-dont (princípios independentes de metodologia)
- [[../testing/bdd-behavior-driven-development]] — BDD (Given-When-Then) como specification executável
- [[../../30-agents/playbooks/_moc]] — playbooks operacionais que aplicam estas metodologias
- [[../../40-templates/_moc]] — templates replicáveis (CLAUDE.md.template, project-scaffold)
- [[../../60-sources/spec-kit/_toc]] — fonte canônica Spec Kit
- [[../../60-sources/get-shit-done/_toc]] — fonte Get Shit Done (agentes especializados, não GitHub SDD)

## Regra dura

Nada aqui cita projeto específico por nome em título/conteúdo. Exemplo de aplicação é **link pro case em `50-projects/`**, não replicação do conteúdo. Ver [[../../CLAUDE#11-o-que-não-fazer-neste-vault]].
