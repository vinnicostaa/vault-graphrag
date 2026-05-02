---
title: fernando-franco-valle — TOC curado
type: moc
tags: [sources, fernando, claude, ia, aws, toc]
created: 2026-04-22
---

# fernandofrancovalle.com — Índice curado

Navegação **por trilha** (71 trilhas, 1 MD por trilha). Cada `raw/<trilha>.md` é a página-landing da trilha no site Next.js, extraída via pandoc. Conteúdo profundo de aulas vive nos JS bundles (site é SPA+RSC) — foram arquivados por falta de contexto semântico.

**Observação**: essas landings têm descrição de curriculum, objetivos, pré-requisitos — servem de **mapa de trilhas** + retrieval de "o que o Fernando recomenda pra X". Pro conteúdo aprofundado, navegar ao vivo em https://fernandofrancovalle.com.

## 🎯 Prioridade ALTA (pro automation-agents)

### Claude / Anthropic (★)

- [[raw/claude-anthropic|claude-anthropic]] — trilha Claude/Anthropic (playbook completo)
- [[raw/claude-api-agents|claude-api-agents]] — API + agents
- [[raw/claude-code-masterclass|claude-code-masterclass]] — masterclass Claude Code
- [[raw/claude-code-pro|claude-code-pro]] — Claude Code pro
- [[raw/ferramentas-ia-codigo|ferramentas-ia-codigo]] — tooling com IA

### IA (fundamentos e avançado)

- [[raw/ia|ia]] — entry point IA geral
- [[raw/fundamentos-da-ia|fundamentos-da-ia]]
- [[raw/ia-alem-do-llm|ia-alem-do-llm]]
- [[raw/ai-native|ai-native]]
- [[raw/ai-safety|ai-safety]]
- [[raw/machine-learning|machine-learning]]
- [[raw/fine-tuning|fine-tuning]]
- [[raw/llm-evals|llm-evals]]
- [[raw/multimodal|multimodal]]
- [[raw/computer-vision|computer-vision]]
- [[raw/mlops|mlops]]

### Engenharia de software

- [[raw/engenharia|engenharia]]
- [[raw/engenharia-software|engenharia-software]]
- [[raw/fundamentos|fundamentos]]
- [[raw/fundamentos-tecnicos|fundamentos-tecnicos]]
- [[raw/api-design|api-design]]
- [[raw/system-design-interview|system-design-interview]]
- [[raw/sistemas-distribuidos|sistemas-distribuidos]]
- [[raw/product-engineering|product-engineering]]
- [[raw/platform-engineering|platform-engineering]]

## 🔬 AWS / Cloud

- [[raw/aws|aws]]
- [[raw/aws-cloud-practitioner|aws-cloud-practitioner]]
- [[raw/aws-developer-associate|aws-developer-associate]]
- [[raw/aws-saa-c03|aws-saa-c03]] — solutions architect associate
- [[raw/aws-sap-c03|aws-sap-c03]] — solutions architect professional
- [[raw/edge-computing|edge-computing]]
- [[raw/finops|finops]]
- [[raw/devops-containers|devops-containers]]
- [[raw/chaos-engineering|chaos-engineering]]

## 🧱 Linguagens e stacks

- [[raw/typescript-profissional|typescript-profissional]]
- [[raw/go-profissional|go-profissional]]
- [[raw/rust-profissional|rust-profissional]]
- [[raw/java-moderno|java-moderno]]
- [[raw/cpp-moderno|cpp-moderno]]
- [[raw/c-programming|c-programming]]
- [[raw/csharp-dotnet|csharp-dotnet]]
- [[raw/python-engenheiros|python-engenheiros]]
- [[raw/linguagens-comparadas|linguagens-comparadas]]

## 📐 Data / DB

- [[raw/dados|dados]]
- [[raw/data-engineering|data-engineering]]
- [[raw/sql-databases|sql-databases]]
- [[raw/postgres-internals|postgres-internals]]
- [[raw/nosql-vector-dbs|nosql-vector-dbs]]
- [[raw/kafka-streaming|kafka-streaming]]

## 🌐 Frontend / Web / Mobile

- [[raw/frontend-moderno|frontend-moderno]]
- [[raw/graphql|graphql]]
- [[raw/redes-web|redes-web]]
- [[raw/jsdom|jsdom]]
- [[raw/lib-authoring|lib-authoring]]
- [[raw/ios-native|ios-native]]
- [[raw/android-native|android-native]]
- [[raw/mobile-rn|mobile-rn]]
- [[raw/real-time-systems|real-time-systems]]

## 🔐 Segurança

- [[raw/security-engineering|security-engineering]]
- [[raw/cryptography-applied|cryptography-applied]]

## 📈 Observabilidade & Performance

- [[raw/observabilidade-sre|observabilidade-sre]]
- [[raw/performance-engineering|performance-engineering]]

## 🧪 Qualidade & Testing

- [[raw/testing-engineering|testing-engineering]]

## 🧭 Carreira & Meta

- [[raw/career-engineering|career-engineering]]
- [[raw/tech-leadership|tech-leadership]]
- [[raw/technical-writing|technical-writing]]
- [[raw/dx-productivity|dx-productivity]]
- [[raw/acessibilidade|acessibilidade]]
- [[raw/ds-algoritmos|ds-algoritmos]]
- [[raw/como-computador-funciona|como-computador-funciona]]
- [[raw/programacao|programacao]]

## Prioridade de destilação

**P0 — destilar em `_notes/`:**
1. `raw/claude-anthropic.md` + `raw/claude-code-masterclass.md` + `raw/claude-code-pro.md` → extrair patterns de Claude Code / prompts / skills relevantes pro `[[../../30-agents/_moc]]`
2. `raw/ferramentas-ia-codigo.md` → comparar com nosso pipeline

**P1:**
3. `raw/ia.md` + `raw/fundamentos-da-ia.md` → base conceitual pra `[[../../10-knowledge/methodology/graph-rag-basics]]`
4. `raw/sistemas-distribuidos.md` + `raw/system-design-interview.md` → referências cruzadas em `[[../../10-knowledge/architecture/_moc]]`

**Nota de viés**: landings do Fernando são marketing/curriculum — conteúdo profundo vive nas aulas (paid/SPA). Pra destilar de verdade, complementar com browsing ao vivo ou pedir ao Fernando notas próprias.

## Vizinhos no grafo

- [[_index]]
- [[_notes/_moc]]
- [[../../30-agents/_moc]] — destino Claude Code
- [[../../10-knowledge/methodology/_moc]]
- Site original: https://fernandofrancovalle.com
