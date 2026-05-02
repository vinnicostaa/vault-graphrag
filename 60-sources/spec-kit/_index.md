---
title: spec-kit (GitHub) — índice local
type: source
tags: [source, spec-kit, sdd, github, methodology]
source_url: https://github.com/github/spec-kit
ingested: 2026-04-22
---

# spec-kit — GitHub Spec-Driven Development

Projeto oficial [github/spec-kit](https://github.com/github/spec-kit) — **framework de Spec-Driven Development** mantido pelo GitHub. Complementa GSD com foco em **specs como código executável** (requirements/design/tasks versionados, validados, rastreáveis).

## Conceitos-chave

1. **Spec-as-code** — requirements, design, tasks em arquivos MD versionados
2. **Discovery workflow** — processo pra descobrir requirements antes de codar
3. **Living specs** — specs evoluem com o código via CI/CD
4. **Validation** — ferramentas pra validar se implementação bate com spec
5. **Templates** — estruturas reutilizáveis pra Spec/Design/Plan/Tasks

## Estrutura canônica (pós reorg 2026-04-22)

```
spec-kit/
├── _index.md         # este arquivo
├── _toc.md           # índice curado por tópico ← começar por aqui
├── raw/              # material bruto do repo
│   ├── spec-driven.md
│   ├── docs/         # docs oficiais (EN)
│   ├── templates/    # spec/plan/tasks/constitution/checklist
│   ├── extensions/   # sistema de extensões + git extension
│   ├── presets/      # lean / scaffold / self-test
│   ├── workflows/    # workflows architecture
│   ├── integrations/
│   ├── newsletters/  # updates mensais
│   └── tests/hooks/
└── _notes/           # staging pre-promoção pro 10-knowledge / 40-templates
```

**Arquivados em** `90-archive/2026-04-22-sources-raw/spec-kit/`: AGENTS.md, CHANGELOG.md, CODE_OF_CONDUCT.md, CONTRIBUTING.md, DEVELOPMENT.md, README.md, SECURITY.md, SUPPORT.md, .github/ (PULL_REQUEST_TEMPLATE, RELEASE-PROCESS).

## Como o agente consulta

**Consultar ao**:
- Escrever requirement novo em `specs/requirements/<module>.md` — conferir formato spec-kit
- Estruturar design doc — ver seções canônicas
- Adotar template de tasks.md — comparar com spec-kit
- Planejar workflow de validação spec↔código

**Destilar em**:
- [[../../10-knowledge/methodology/sdd-workflow]] — cruzar com GSD (ambos inspiraram nosso ciclo)
- `[[../../40-templates/_moc]]` — criar `requirements-template.md`, `design-template.md`, `tasks-template.md` baseados em spec-kit
- `[[../../10-knowledge/methodology/spec-driven-development.md]]` *(a criar)* — destilar `spec-driven.md` do repo

## Status de ingestão

- ✅ 89 MDs movidos do repo git (código Python descartado)
- 📋 A destilar: `spec-driven.md` + `docs/quickstart.md` em `10-knowledge/methodology/`
- 📋 A destilar: formato de spec em `40-templates/spec-structure-template.md`

## Licença

Open-source MIT (GitHub). Adotar livremente, atribuir quando citar.

## Comparação GSD × spec-kit

| Aspecto | GSD | spec-kit |
|---|---|---|
| Origem | Community-driven | GitHub oficial |
| Foco | Agentes especializados + ciclo 5 fases | Specs como código + validação |
| Artefato primário | `.plans/<task>.md` | `specs/{requirements,design,tasks}.md` |
| Ênfase | **Como** executar | **O quê** especificar |
| Integração com IDE | Claude Code skills | GitHub workflows + docs |

**Nosso pipeline usa os dois**: GSD pra ciclo de execução + estrutura de agentes; spec-kit pra formato de specs e validação cruzada.

## Links

- [[_toc]] — **índice curado por tópico (entrada principal)**
- [[_notes/_moc]] — staging de destilados
- [[../_moc]] — sources
- [[../../10-knowledge/methodology/sdd-workflow]] — destino do destilado
- [[../../40-templates/_moc]] — templates de spec
- [[../get-shit-done/_index]] — fonte complementar (execução)
- Repo original: https://github.com/github/spec-kit
