---
title: Família — Research & Análise (GSD)
type: note
tags:
  - agents
  - gsd
  - research
  - family
created: '2026-04-24T00:00:00.000Z'
updated: '2026-04-24T00:00:00.000Z'
aliases:
  - Research agents
  - family-research
---

# Família GSD — Research & Análise

Os 10 agentes desta família formam a camada **pré-plan, pré-código** do ciclo GSD: antes de decidir, antes de codar, antes de revisar. Eles investigam o terreno — domínio, ecossistema, codebase, assumptions, decisões cinzentas — e devolvem artefatos estruturados que constrangem as fases seguintes. Nenhum deles escreve código de produção; todos produzem markdown que vira input duro para `gsd-planner`, `gsd-roadmapper` ou outros researchers.

A lógica central é **"training data = hypothesis"**: o que o LLM "sabe" só vira fact depois de verificado em Context7, docs oficiais ou no próprio codebase. Cada agente opera com hierarquia explícita de fontes e tagueia provenance (`[VERIFIED]` / `[CITED]` / `[ASSUMED]`).

## 10 agents na família

- [[card-gsd-advisor-researcher]] — pesquisa UMA gray area e retorna tabela comparativa 5-colunas com rationale condicional
- [[card-gsd-ai-researcher]] — destila docs oficiais de frameworks de AI em guia implementation-ready (seções 3, 4, 4b de `AI-SPEC.md`)
- [[card-gsd-assumptions-analyzer]] — extrai assumptions tácitas do codebase com evidência (file paths) e confidence
- [[card-gsd-codebase-mapper]] — explora o codebase por focus area e produz STACK/ARCH/CONVENTIONS/CONCERNS
- [[card-gsd-domain-researcher]] — pesquisa o **domínio de negócio** de um AI system (critérios de eval, failure modes, compliance)
- [[card-gsd-pattern-mapper]] — classifica arquivos planejados por role + data flow e acha analog no codebase com excerpts concretos
- [[card-gsd-phase-researcher]] — pesquisa stack/patterns/pitfalls/security/testing de UMA fase antes do planner criar tasks
- [[card-gsd-project-researcher]] — mapeia ecossistema de domínio inteiro antes do roadmap (modos ecosystem / feasibility / comparison)
- [[card-gsd-research-synthesizer]] — reducer que funde 4 researches paralelos em `SUMMARY.md` único com implicações para roadmap
- [[card-gsd-user-profiler]] — analisa sessões passadas do dev e produz perfil comportamental em 8 dimensões (instruction doc para Claude)

## Quando usar esta família

1. **Bootstrap de projeto novo**: `project-researcher` mapeia ecossistema de domínio + `codebase-mapper` inspeciona scaffolds existentes + `user-profiler` calibra comportamento do agente para o dev. Output alimenta o primeiro ROADMAP.
2. **Início de fase com ambiguidade técnica**: `phase-researcher` + `pattern-mapper` em paralelo antes do planner decompor. Evita que tasks sejam criadas com stack/pattern indefinido e vire re-planejamento caro depois.
3. **Decisão cinzenta (gray area) emergindo no meio da execução**: Worker paralisado em "Redis Streams vs Kafka?", "JWT vs sessão?" — spawn `advisor-researcher` focado naquela decisão única, recebe tabela 5-colunas e decide sem derrubar contexto.
4. **Produto AI com stack novo**: `ai-researcher` + `domain-researcher` antes de eval-planner desenhar rubric. O domínio de negócio determina failure modes; o framework determina APIs e constraints — os dois em conjunto travam AI-SPEC §3–4b.
5. **Revisão arquitetural retroativa**: `assumptions-analyzer` pega código já escrito e extrai assumptions tácitas que viraram dívida (timezone assumido UTC, tenant assumido single, cache assumido local). Output vira entradas em AUDIT.md.

## Quando NÃO usar

- **Task de 30 minutos com stack já decidida**: pular pra Planner/Executor — research-loop aqui vira over-engineering. Use Read/Grep diretos.
- **Bug reprodutível com stack trace claro**: é família Debug, não Research. Debug é post-mortem de falha concreta; Research é pre-discovery.
- **Documentar decisão já tomada**: família Docs (writer + verifier) captura retroativamente — Research destila o que **vai** ser decidido.
- **Refatoração puramente mecânica (rename, split de arquivo)**: Executor com plano curto resolve; invocar researcher é ruído.
- **Pergunta que WebSearch/Context7 responde em 1 chamada direta**: o agente principal pode consultar sem spawn de subagent. A família Research ganha quando o output é **estruturado e reutilizável** (tabela, markdown persistente), não quando é uma resposta pontual.

## Interação entre agents da família (pipeline interno)

```
  ┌─── project-researcher ────┐
  │    (ecossistema macro)    │
  │                           ▼
  ├─── phase-researcher ─┐    │
  │    (1 por fase)      │    │
  │                      ▼    ▼
  ├─── pattern-mapper ───► research-synthesizer ──► SUMMARY.md
  │    (1 por área)      ▲    ▲                    (entry point do planner)
  │                      │    │
  ├─── codebase-mapper ──┘    │
  │    (1 por focus area)     │
  │                           │
  └─── advisor-researcher ────┘
       (N por gray area)

  assumptions-analyzer ─── lateral (inspeciona codebase vivo, não alimenta SUMMARY)
  ai-researcher        ─── feeds AI-SPEC §3-4b (domínio AI específico)
  domain-researcher    ─── feeds AI-SPEC §1-2 (domínio de negócio AI)
  user-profiler        ─── feeds CLAUDE.md do projeto (comportamento do agente)
```

O `research-synthesizer` é o fan-in: recebe file paths dos outros 4–9 researchers (nunca conteúdo inline — evita context bloat), lê cada arquivo uma vez, aplica detecção de contradição e emite SUMMARY.md único com seções "Findings Convergentes", "Conflitos Não Resolvidos" e "Implicações para Roadmap". Contradições não são fundidas — são listadas pro Planner decidir.

## Padrões reutilizáveis

**Especialização por fonte/escopo** — não há "o researcher genérico". Cada agente tem um corte: UMA gray area (advisor), UMA fase (phase, pattern), UM foco (codebase-mapper), O domínio inteiro (project), O domínio de negócio de AI (domain), as assumptions do próprio codebase (assumptions), o perfil do dev (user-profiler). A especialização força janelas de contexto pequenas e paralelismo seguro.

**Fan-out / fan-in via synthesizer** — o padrão mais replicável do grupo: N researchers paralelos escrevem arquivos mas **não commitam**; um `research-synthesizer` (ou orquestrador) lê tudo, produz `SUMMARY.md` e commita atomicamente. Resolve coordenação sem memória compartilhada entre workers. Variante: synthesizer pode falhar de propósito se N arquivos não existem no timeout — evita SUMMARY baseado em research incompleto.

**Provenance tagging anti-alucinação** — claims factuais carregam `[VERIFIED]` / `[CITED: url]` / `[ASSUMED]`. O que está `[ASSUMED]` entra em `Assumptions Log` explícito para confirmação humana. Reduz o incentivo de "parecer seguro" sem evidência. Aplicável além de research: qualquer artefato LLM-gerado que informa decisão irreversível deveria tagear.

**Output prescritivo, nunca descritivo** — "Use camelCase" > "some functions use camelCase"; "Rec if mobile-first" > "X é uma boa opção". Os docs deste grupo são consumidos por outros agentes/LLMs que precisam de imperativos acionáveis, não de narrativa. Regra de cabeceira: se você pode ler a seção e **não** saber o que fazer a seguir, a seção é descritiva demais.

## Correspondência com nosso pipeline

No nosso pipeline **Planner / Worker / Reviewer / Orchestrator**, 9 dos 10 agentes alimentam o **Planner** — ou seja, a família research é basicamente a camada de discovery que o nosso Planner precisa fazer antes de emitir plano. `advisor-researcher`, `assumptions-analyzer` e o discuss-phase (não é agent, é fluxo) mapeiam para o que nosso Planner faz no ciclo SDD fase **Discuss**. `phase-researcher`, `pattern-mapper`, `codebase-mapper`, `domain-researcher` e `ai-researcher` alimentam a fase **Plan**. `project-researcher` é pre-Plan, quando o roadmap ainda não existe.

`research-synthesizer` é o único que mapeia em **Orchestrator** — ele é o reducer do fan-out dos 4 researchers paralelos, função claramente orquestração. `user-profiler` é **Support** — não participa do ciclo SDD, é um utilitário transversal que informa comportamento de todos os agentes.

**Insight operacional**: o Planner do `automation-agents` hoje tende a ser monolítico — um único agent tentando discovery + decomposição. A família GSD sugere quebrar em **discovery sub-agents paralelos + Planner como reducer**, mapeando nosso Research-Loop (§4 do CLAUDE.md) para ser executado em paralelo em vez de serial. Isso acelera o ciclo Discuss sem sacrificar cobertura.

Para detalhamento por agente ver [[pipeline-mapping]].

## Patterns atemporais extraídos

1. **Hipótese vs fato é problema de tipagem**: tagear claim como `[VERIFIED]`/`[CITED]`/`[ASSUMED]` é equivalente a tipar valor num sistema de tipos — força o consumidor (outro agent, humano) a saber a confiabilidade antes de usar. Padrão aplicável a qualquer output LLM-gerado que alimenta decisão crítica.
2. **Fan-out + reducer com arquivos no disco > memória compartilhada**: workers paralelos sem coordenação direta, reducer lendo arquivos, commit atômico no fim. Não requer locks, não requer IPC, rollback é `rm -rf workdir/`. Padrão aplicável muito além de research — é o mesmo padrão de MapReduce em escala humana.
3. **Especialização estreita + janela de contexto pequena**: um agent com escopo amplo (research genérico) diverge rápido; 10 agents com escopo estreito convergem. Menos contexto por agent = menos hallucination e mais verificabilidade.
4. **Contrato de output acionável**: a saída de um agent que alimenta outro agent deve ser um imperativo ("faça X se Y"), não descrição ("X existe"). Documentação pra humano pode ser narrativa; documentação pra LLM downstream precisa ser prescritiva.

## Links externos

- [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) — orquestrador + workers espelha fan-out/fan-in deste grupo
- [Google SRE Workbook — On-Call](https://sre.google/workbook/on-call/) — mesma disciplina de "hypothesis first, evidence second" usada pelos researchers
- [Spec-Driven Development Guide (GitHub)](https://github.com/github/spec-kit) — fase Discuss/Plan casa com papel dos researchers
- [Martin Fowler — Refactoring: Characterization Tests](https://martinfowler.com/bliki/CharacterizationTest.html) — `assumptions-analyzer` é versão LLM dessa técnica
- [Diátaxis framework](https://diataxis.fr/) — separação de explanation/reference/how-to/tutorial informa output prescritivo vs descritivo

## Fonte

- [[../../60-sources/get-shit-done/_toc]]

## Links

- [[_moc]]
- [[pipeline-mapping]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
