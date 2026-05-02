---
title: Kiro (AWS) vs GitHub Spec Kit vs Tessl — 3 escolas de SDD em 2026
type: concept
tags: [methodology, sdd, kiro, spec-kit, tessl, comparison]
source: vault/50-projects/master-sindico/content/sdd-gsd-aplicado.md
created: 2026-04-22
updated: 2026-04-24
aliases: [Kiro vs Spec Kit, Kiro vs Spec Kit vs Tessl, SDD tools comparison]
---

# Kiro (AWS) vs GitHub Spec Kit vs Tessl

Em 2026 existem **três escolas principais** de Spec-Driven Development, todas compartilhando a filosofia de [[spec-driven-development|specs como fonte de verdade]] mas diferindo em ergonomia, artefatos, automação e **nível de spec-anchoring**.

A literatura canônica de comparação é [Martin Fowler — *Understanding SDD: Kiro, spec-kit, and Tessl*](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) (15 out 2025). O artigo identifica as 3 ferramentas **e** uma taxonomia ortogonal de 3 níveis de implementação (§ Taxonomia).

## 1. Kiro (AWS)

- **Origem**: IDE/CLI da AWS (cloud-agnóstica apesar do nome).
- **Estrutura canônica**: `.kiro/specs/<feature>/` com 3 arquivos:
  - `requirements.md` — o quê e por quê
  - `design.md` — como (interfaces, contratos, fluxos)
  - `tasks.md` — lista executável
- **Agent hooks** — callbacks de eventos do agente configuráveis.
- **Filosofia**: conhecimento em camadas (requirements → design → tasks → implementation).
- **Posicionamento (Fowler)**: ferramenta **mais leve**; **spec-first**.

## 2. GitHub Spec Kit

- **Origem**: toolkit open-source mantido pelo GitHub (84k+ ⭐).
- **CLI distribuída** com **30+ AI agents** suportados (Claude Code, Copilot, Gemini CLI, Cursor, Qwen, opencode, Codex, Kiro CLI, Goose, Tabnine, entre outros — ver [docs oficiais](https://github.github.io/spec-kit/)).
- **Quatro artefatos canônicos** persistentes por feature:
  - `constitution.md` — princípios invioláveis do projeto
  - `spec.md` — o quê / por quê (requirements)
  - `plan.md` — como (design + research)
  - `tasks.md` — lista executável
- **Diretório raiz**: `.specify/`.
- **Workflows automatizados** via 9 slash commands divididos em core + optional (ver abaixo).
- **Posicionamento (Fowler)**: ferramenta **elaborada**; **spec-anchored**.

### 9 slash commands (Spec Kit 2026) — 6 core + 3 optional

**Core (6) — ciclo principal**

| Comando | Função |
|---|---|
| `/speckit.constitution` | Cria/atualiza constitution do projeto |
| `/speckit.specify` | Captura requisitos a partir de descrição em linguagem natural |
| `/speckit.plan` | Gera plano técnico (research + data model + contracts) |
| `/speckit.tasks` | Decompõe plan em lista executável para implementação |
| `/speckit.implement` | Executa tasks com o agente de código |
| `/speckit.taskstoissues` | Converte tasks em GitHub Issues (bidirectional traceability) |

**Optional (3) — análise/checagem**

| Comando | Função |
|---|---|
| `/speckit.clarify` | Identifica e resolve ambiguidades na spec |
| `/speckit.analyze` | Análise cruzada (consistência spec ↔ plan ↔ tasks) |
| `/speckit.checklist` | Gera checklist de validação pró-feature |

Ver [[../../60-sources/spec-kit/_toc]] — material original indexado.

### 3 fases macro do Spec Kit

Spec Kit estrutura o trabalho por **tamanho do problema**:

1. **Greenfield (0-to-1 Development)** — construir do zero a partir de requisitos de alto nível, gerando specs + plan + código.
2. **Creative Exploration** — testar implementações paralelas em múltiplos stacks/arquiteturas a partir da mesma spec.
3. **Iterative Enhancement (Brownfield)** — adicionar features em sistemas existentes, modernizar código legacy.

## 3. Tessl

- **Origem**: startup focada em *spec-as-source*.
- **Produtos (2026)**: dois distintos:
  - **Spec Registry** — em **open beta**.
  - **Tessl Framework** — em **closed beta**.
- **Posição única**: specs não apenas guiam a geração — geram código **diretamente**, com marcadores explícitos indicando seções auto-geradas.
- **Filosofia**: push do spec-driven até o limite — spec **é** source-of-truth operacional; código-fonte tradicional é artefato regenerável.
- **Posicionamento (Fowler)**: ferramenta **mais ambiciosa**; **spec-as-source**.
- **Observação editorial do vault** (não é afirmação da fonte): spec-as-source rememora o Model-Driven Development dos anos 2000 — Fowler coloca como pergunta aberta se os mesmos desafios históricos reaparecerão.

## Taxonomia de Fowler — 3 níveis de implementação SDD

Ortogonal às 3 ferramentas, Fowler identifica 3 **níveis de spec-anchoring**:

| Nível | Significado | Ferramenta dominante |
|---|---|---|
| **Spec-first** | Spec é escrita primeiro, guia a implementação, mas não está ligada ao código de modo executável | Kiro |
| **Spec-anchored** | Spec e código coexistem; regeneração parcial possível; constitution como guard rail | Spec Kit |
| **Spec-as-source** | Código é gerado **a partir** da spec; marcadores delimitam seções auto-geradas | Tessl |

A contribuição central do artigo é mostrar que a ferramenta certa depende **do nível** que o time quer atingir, não apenas de ergonomia superficial.

## Diferenças práticas (tabela)

| Dimensão | Kiro | GitHub Spec Kit | Tessl |
|---|---|---|---|
| Diretório raiz | `.kiro/` | `.specify/` | (proprietário) |
| Constituição | Implícita em `steering/` | **Explícita** em `constitution.md` | Embutida na spec |
| Workflow | Manual (skills, sub-agentes) | 9 comandos slash (6 core + 3 optional) | Agente proprietário |
| Numeração de feature | Manual | **Automática** + branch creation | N/A |
| Agentes suportados | Claude Code (principal) | 30+ | Proprietário |
| Pause gates | Decisão manual por fase | Configurável em `workflows/` | N/A |
| Parseable machine | MD humano | Específico por agente | Binding direto ao código |
| Nível de spec-anchoring | Spec-first | Spec-anchored | Spec-as-source |
| Aspiração (Fowler) | Leve | Elaborada / arquitetural | Extrema / spec-as-source |
| Status (2026) | Estável | Estável | Spec Registry open beta / Framework closed beta |

## Qual escolher

### Use **Kiro** quando
- Time Claude Code-first
- Projeto grande com múltiplos bounded contexts (quer expandir `requirements/` em recortes por módulo)
- Quer hooks customizados por evento
- Prefere flexibilidade máxima sobre automação
- Aceita ciclo **spec-first** sem regeneração automática

### Use **Spec Kit** quando
- Time multi-agent (Cursor + Claude + Copilot no mesmo repo)
- Quer workflow automatizado com 9 comandos slash (core + optional)
- Prefere `constitution.md` explícito a steering distribuído
- Quer branch automation + feature numbering sem fricção
- Open-source purism (GitHub/MIT license)
- Aceita **spec-anchored** com regeneração parcial

### Use **Tessl** quando
- Aceita beta (open/closed conforme produto) + vendor lock-in experimental
- Quer explorar **spec-as-source** no limite
- Aceita risco aberto de desafios históricos do MDD

### Use **Kiro + extensões Spec Kit** (híbrido)
- Estrutura `.kiro/` + adotar comandos `/speckit.*` como skills Claude Code
- Steering de Kiro + `constitution.md` índice do Spec Kit apontando pra ele
- Caminho natural quando se **começou com Kiro** e quer ganhar automação

## Gaps comuns a cobrir (independente de escolha)

Quando adota SDD (qualquer das 3 escolas), cuidar de 8 gaps frequentes. **Catálogo atemporal completo** em [[sdd-adoption-gaps]].

### Catálogo inicial (2026-04-22)

- **G-001** — Constitution implícita em vez de explícita
- **G-002** — Workflow manual em vez de slash/skill automatizado
- **G-003** — Terminologia ambígua ("GSD", "spec", "plan")
- **G-004** — `<verify>` / acceptance não-parseáveis por máquina
- **G-005** — Reference layer ausente (só spec + código)
- **G-006** — STATE/AUDIT que só o tech lead mantém

### Adições 2026-04-24 (críticas de Martin Fowler)

- **G-007** — Workflow mismatch: spec com granularidade errada pro tamanho do problema
- **G-008** — Review burden: verbosidade de artefatos MD excede esforço de revisar código direto

Ver [[sdd-adoption-gaps]] para sintomas, impacto e fix de cada.

## Convergência em 2026

As 3 escolas convergem nos mesmos princípios fundacionais:

1. **Spec é source of truth** — código é artefato (em Tessl literalmente; em Kiro/Spec Kit via regeneração).
2. **Fases explícitas** (3–5, conforme framework).
3. **Constituição** (implícita ou explícita) como guard rails.
4. **Agents-first** — frameworks desenhados assumindo IA no loop.
5. **Research-driven context** (ver [[spec-driven-development]]).
6. **Branches por feature** (Spec Kit automatiza; Kiro manual; Tessl N/A).

Diferença-chave entre elas é o **nível de spec-anchoring** e o trade-off entre flexibilidade e determinismo.

## Nosso projeto adota

O projeto **automation-agents** adota **Kiro + extensões Spec Kit** (híbrido spec-first → spec-anchored):

- Estrutura `.kiro/` ✅
- Camada `reference/` **além do Kiro** (antipatterns, obsidian-mapping)
- Camada `steering/` **além do Kiro** (regras duras por tópico — 13 arquivos)
- Skills Claude Code customizadas (`/task-executor`, `/sprint-auditor`, `/dual-validate`) — equivalente manual aos comandos Spec Kit
- STATE.md + AUDIT.md + SESSION_CHARTER **vivos** — vão além do Kiro canônico

**A adotar futuramente**: `/speckit.specify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.clarify`, `/speckit.implement`, `/speckit.constitution` como skills customizadas — pegar o melhor do Spec Kit mantendo a ergonomia Kiro.

**Tessl**: em monitoramento; sem adoção ativa até sair do beta + evidência de resolução dos riscos históricos do MDD.

## Referências

- [GitHub Spec Kit (repo)](https://github.com/github/spec-kit)
- [GitHub Spec Kit — Documentation](https://github.github.io/spec-kit/)
- [spec-driven.md manifesto](https://github.com/github/spec-kit/blob/main/spec-driven.md) — destilado em [[spec-driven-development]]
- [Martin Fowler — *Understanding SDD: Kiro, spec-kit, and Tessl* (15 out 2025)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) — comparação canônica; origem das críticas G-007 e G-008
- [Kiro.dev](https://kiro.dev/)
- [Tessl — Spec Registry + Framework launch](https://tessl.io/blog/tessl-launches-spec-driven-framework-and-registry/)
- [Microsoft Learn — SDD with Spec Kit](https://learn.microsoft.com/en-us/training/modules/spec-driven-development-github-spec-kit-greenfield-intro/)

## Links

- [[_moc]]
- [[spec-driven-development]] — filosofia SDD (spec-kit manifesto destilado)
- [[sdd-workflow]] — ciclo operacional 5-fase
- [[sdd-maturity-checklist]] — 10 itens pra medir cobertura SDD num projeto
- [[sdd-adoption-gaps]] — catálogo G-### de gaps típicos em adoção (G-001..G-008)
- [[gsd-get-shit-done]] — ergonomia de entrega complementar
- [[../../60-sources/spec-kit/_toc]] — fonte
- [[../../60-sources/get-shit-done/_toc]] — fonte complementar (agentes especializados; não confundir com GitHub SDD)
- [[../../00-meta/STATE]] — D-vault-002 registra esta atualização
