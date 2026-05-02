---
title: MOC — 10-knowledge
type: moc
tags:
  - moc
  - knowledge
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# 10-knowledge — Conhecimento técnico atemporal

Conceitos, padrões, princípios e metodologia que valem em qualquer projeto. Nada específico de cliente aqui — para isso, ver [[../50-projects/_moc]].

## Sub-áreas

- [[architecture/_moc]] — Clean Architecture, DDD, Hexagonal, CQRS, Vertical Slices, Bounded Contexts
- [[patterns/_moc]] — GoF + refactorings (Strategy, Builder, Observer, Adapter, etc.)
- [[principles/_moc|principles]] — SOLID, DRY, YAGNI, KISS, Dependency Rule, Tell-Don't-Ask
- [[methodology/_moc]] — SDD, GSD, TDD, BDD, PARA, Zettelkasten + Graph-RAG
- [[antipatterns/_moc]] — catálogo AP-### cross-linguagem (code smells Fowler/GoF, design)
- [[runtime-antipatterns/_moc]] — catálogo OP-### operacional (concorrência, idempotência, cache, segurança runtime, performance) — reconciliação em [[../00-meta/STATE|D-vault-012]]
- [[security/_moc]] — HMAC, LGPD/GDPR, OWASP, zero-trust, rate limit, audit trail
- [[testing/_moc]] — unit, integration, property-based, E2E, coverage thresholds
- [[database/_moc]] — naming, PKs, indexes, transactions, migrations, UUIDv7
- [[observability/_moc]] — structured logging, tracing (OTel), metrics, audit
- [[realtime/_moc]] — WebSocket, SSE, Webhooks, WebRTC (protocolos tempo-real) — decision tree + patterns produção (D-vault-023)
- [[ui-design/_moc]] — Design System Architecture, Tokens, case study Primer (GitHub) — arquitetura de DS para multi-app/multi-brand (D-vault-024)
- [[providers/_moc]] — **SaaS/infra externos** (Stripe, Mux, Zitadel, Livekit, Railway, etc.) — nó global reutilizável com MCP, SDKs oficiais, patterns, antipatterns, data de consulta da doc

## Destaques

- [[methodology/graph-rag]] — como este vault implementa Graph-RAG
- [[methodology/kiro-vs-spec-kit]] — duas escolas de SDD + gaps comuns
- [[methodology/sdd-maturity-checklist]] — 10 itens pra auditar adoção SDD em qualquer projeto
- [[methodology/sdd-adoption-gaps]] — catálogo G-### de antipatterns de adoção
- [[providers/stripe/_moc]] — Stripe (pagamentos / Connect / subscriptions) + MCP oficial

## Vizinhos no grafo

- [[../_index]] — raiz
- [[../20-stacks/_moc]] — conhecimento aplicado por linguagem/framework
- [[../40-templates/_moc]] — extratos reusáveis deste conhecimento
- [[../30-agents/playbooks/_moc]] — playbooks operacionais que aplicam este conhecimento
- [[../30-agents/mcp-integration]] — providers com MCP server

## Regra dura

Conteúdo aqui é **linguagem-agnóstico e projeto-agnóstico** sempre que possível. Exemplo de código em pseudocódigo ou com comentário `// cross-linguagem: adaptar sintaxe ao stack`. Implementação concreta por linguagem vai em [[../20-stacks/_moc]]; aplicação concreta por projeto em [[../50-projects/_moc]].

Se citar termos de domínio específicos de algum projeto (ex.: entidades, fluxos, stakeholders) — errou de arquivo. Deve estar em `50-projects/<slug>/01-domain/`, com link **de volta** pra cá.
