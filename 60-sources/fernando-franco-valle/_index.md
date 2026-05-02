---
title: fernandofrancovalle.com — playbook AI/Claude — índice local
type: source
tags: [source, claude, ai, claude-code, playbook, aws]
source_url: https://fernandofrancovalle.com
ingested: 2026-04-22
---

# fernandofrancovalle.com

Site do amigo Fernando — **playbook gamificado sobre IA, AWS, Claude Code, sistemas distribuídos**. Autoria única, sem paywall, sem cadastro. Fonte primária pra **como trabalhar com IA/agentes**, patterns de prompting e design de sistemas com Claude.

## Conteúdo principal

- **Trilha IA** (`ia/`) — fundamentos de IA, prompting, Claude features, MCPs, agents
- **Trilha Claude/Anthropic** (`claude-anthropic/`) — específico sobre Claude Code, skills, hooks, SDK
- **Trilha AWS** (`aws/`) — serviços, arquiteturas, IaC
- **Sistemas distribuídos** — microservices, event-driven, saga, CQRS aplicados
- **Progresso/gamificação** — cada artigo dá XP, quizzes ao fim

## Por que é valiosa

Fernando é **colega do time** que escreveu este material com viés prático-brasileiro. Linguagem pt-BR. Exemplos aterrados ao contexto que o usuário trabalha (AWS, Claude). Referência de **peer**, não manual vendor.

## Estrutura canônica (pós reorg 2026-04-22)

```
fernando-franco-valle/
├── _index.md
├── _toc.md           # índice por trilha (ALTA PRIORIDADE no topo) ← começar aqui
├── raw/              # 71 trilhas como <slug>.md (achatado do Next.js routing)
│   ├── claude-anthropic.md          ★ alta prioridade
│   ├── claude-code-masterclass.md   ★
│   ├── claude-code-pro.md           ★
│   ├── claude-api-agents.md         ★
│   ├── ferramentas-ia-codigo.md
│   ├── ia.md, fundamentos-da-ia.md, ai-native.md, machine-learning.md, ...
│   ├── aws.md, aws-saa-c03.md, ...
│   ├── engenharia-software.md, sistemas-distribuidos.md, ...
│   └── ... (71 trilhas no total)
└── _notes/           # staging pre-promoção
```

**Arquivados em** `90-archive/2026-04-22-sources-raw/fernando-franco-valle/`:
- `_js-content/` (17 MDs — fragmentos de bundles Next sem contexto semântico)
- `fernandofrancovalle.com/news/`, `simulados/`, `playlists/`, `progresso/`, `_next/`, `static/`, `search/`, `preferencias/`, `roadmaps/`, `construcao/`, `glossario/`, `aprenda/`, `revisar/`, `verificar/`, `ROOT/`, `$&/`
- Arquivos soltos inválidos (`e.md`, `e,t.md`, `N(t.md`, `sw.md`, `window.location.md`, `manifest.md`)

## Como o agente consulta

**Retrieval-first** pra dúvida sobre Claude Code, MCPs, skills, hooks:
1. Buscar neste diretório **antes** do Context7 (é contexto aplicado, não doc vendor)
2. Se achar resposta, citar com atribuição
3. Se divergir do oficial (docs Anthropic), oficial vence — mas registrar divergência

**Destilação prioritária**:
- Notas sobre Claude Code (skills, hooks, MCPs) → [[../../30-agents/_moc|30-agents/]]
- Patterns de prompting → criar `30-agents/prompting-patterns.md`
- Exemplos de multi-agent → complementar `[[../../30-agents/autonomous-execution]]`

## Status de ingestão

- ✅ ~660+ HTMLs convertidos via pandoc; Next.js routing fez cada rota virar `<rota>/index.html`
- ✅ 2026-04-22 reorg: 71 landings de trilhas em `raw/<slug>.md` (EN+pt-BR misturado — é o que o Fernando entregou)
- ✅ `_js-content/` + páginas admin/ruído arquivados (17 JS fragments sem contexto; news/_next/static/ etc.)
- ⚠️ **Site é SPA+RSC**: conteúdo profundo de aulas vive em JS bundles, não no HTML renderizado. As landings em `raw/` cobrem descrição de trilhas, currículo, pré-requisitos.
- 📋 A destilar: ver [[_toc#Prioridade de destilação]]

## Licença / atribuição

Conteúdo **gratuito/público** do Fernando. Citação em docs internas OK com atribuição. **Não republicar** conteúdo íntegro em docs públicas sem autorização dele.

## Links

- [[_toc]] — **índice por trilha (entrada principal)**
- [[_notes/_moc]] — staging de destilados
- [[../_moc]] — sources
- [[../../30-agents/_moc]] — destino destilado sobre Claude Code
- [[../../10-knowledge/methodology/_moc]] — destino sobre patterns de IA/prompting
- Site original: https://fernandofrancovalle.com
