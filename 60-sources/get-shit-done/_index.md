---
title: get-shit-done (GSD) — índice local
type: source
tags: [source, gsd, agents, sdd, methodology]
source_url: https://github.com/gsd-build/get-shit-done
ingested: 2026-04-22
---

# GSD — Get Shit Done

Projeto [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) — **sistema de agentes especializados** pra trabalho autônomo com Claude Code. Inspirou a fase **Discuss → Plan → Execute → Verify → Ship** adotada em [[../../10-knowledge/methodology/sdd-workflow]].

## Conceitos-chave

1. **Ciclo de 5 fases explícito** por task (exatamente o que adotamos)
2. **`<verify>` declarativo** — testes declarados antes do código
3. **STATE.md como arquivo vivo** — bloqueadores, decisões, dívidas
4. **Agentes especializados** — 1 agente por responsabilidade (advisor, code-fixer, assumptions-analyzer, etc.)
5. **`.plans/` como diretório de planos executáveis**

## Estrutura canônica (pós reorg 2026-04-22)

```
get-shit-done/
├── _index.md
├── _toc.md           # índice curado por tópico ← começar aqui
├── raw/              # material útil do repo (346 MDs, EN only)
│   ├── README.md
│   ├── agents/       # 33 gsd-* agents
│   ├── commands/gsd/ # 85 slash commands
│   ├── docs/         # AGENTS, ARCHITECTURE, FEATURES, COMMANDS, USER-GUIDE
│   ├── get-shit-done/
│   │   ├── contexts/       # dev, research, review
│   │   ├── references/     # 30+ conceitos (agent-contracts, gates, ...)
│   │   ├── templates/      # artefatos (SPEC, STATE, UAT, CLAUDE.md, ...)
│   │   └── workflows/      # 81 procedimentos executáveis
│   ├── sdk/          # SDK docs + prompts + src docs
│   └── .plans/1755-install-audit-fix.md
└── _notes/           # staging pre-promoção
```

**Arquivados em** `90-archive/2026-04-22-sources-raw/get-shit-done/`: README.{ja-JP,ko-KR,zh-CN,pt-BR}.md, CHANGELOG.md, CONTRIBUTING.md, SECURITY.md, VERSIONING.md, .github/, docs/{ja-JP,ko-KR,zh-CN,pt-BR}/.

## Como o agente consulta

Este é **material de referência pra design do sistema multi-agente** do automation-agents:

**Consultar ao**:
- Definir novo agente especializado → ver como GSD modela em `agents/`
- Escrever `.plans/<task>.md` — ver formato GSD
- Implementar slash command → ver `commands/`
- Adotar fase nova no SDD → ver se GSD tem conceito equivalente

**Destilar em**:
- `[[../../30-agents/_moc]]` — patterns de sub-agentes especializados
- `[[../../10-knowledge/methodology/sdd-workflow]]` — já destilado parcialmente; elaborar `.plans/` format
- Templates em `[[../../40-templates/_moc]]` — `agent-spec-template.md` (formato de um agente GSD)

## Status de ingestão

- ✅ 410 MDs originalmente movidos do repo git
- ✅ 2026-04-22 reorg: 346 MDs em `raw/` (EN only) + 64 movidos pro archive (i18n + admin)
- 📋 A destilar: ver [[_toc#Prioridade de destilação]]

## Licença

Open-source (ver `LICENSE.md` no repo). Adotar conceitos livremente, referenciando origem.

## Links

- [[_toc]] — **índice curado por tópico (entrada principal)**
- [[_notes/_moc]] — staging de destilados
- [[../_moc]] — sources
- [[../../10-knowledge/methodology/sdd-workflow]] — SDD adotado daqui
- [[../../30-agents/_moc]] — destino dos agents spec
- [[../../40-templates/_moc]] — destino dos templates GSD-inspired
- [[../spec-kit/_index]] — fonte complementar (spec-driven development)
- Repo original: https://github.com/gsd-build/get-shit-done
