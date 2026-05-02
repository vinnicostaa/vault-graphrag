---
title: MOC — 13-research/linkedin
type: moc
tags: [moc, master-sindico, v2, research, linkedin]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/linkedin

Destilado LinkedIn Engineering → Master Síndico. Foco: Kafka/log unificado, Espresso, Voyager/Galene search, feed ranking, Trust & Safety, Skills Graph, endorsements, PYMK, Pemberly frontend.

Pesquisa realizada em 2026-04-23 como parte da remontagem v2. Conteúdo destila-se em: `02-architecture/patterns.md` (outbox, unified content filter, fetch-vs-rank), `08-security/*` (anti-abuse, anti-fake), `STATE.md` (D-LKD-###), `AUDIT.md` (A-LKD-TS-###).

## Arquivos

- [[_destilado]] — síntese aplicada ao master-sindico (seções 1-10 + decisões D-LKD-###)
- [[source-01]] — The Log (Jay Kreps): log unificado, Kafka origin, M×N problem
- [[source-02]] — Espresso: distributed document store, change capture (Databus)
- [[source-03]] — Search Federation / Galene: query understanding, scatter-gather, ranking separation
- [[source-04]] — Feed ranking LLM + Generative Recommender (2025-2026 state of art)
- [[source-05]] — Defending Against Abuse at LinkedIn's Scale (unified content filtering, velocity counters)
- [[source-06]] — Keeping LinkedIn Professional: CNN classifier pra perfis inapropriados
- [[source-07]] — Skills Graph taxonomy: 39k skills, aliases, dual human+ML curation
- [[source-08]] — Skill Assessments: Rasch Model, adaptive testing, anti-cheat
- [[source-09]] — PYMK candidate generation + Pemberly frontend (agrupados)
- [[source-10]] — Rethinking Endorsements Infrastructure Parts 1+2 (SQL → GraphDB + Kafka/Samza)

## Decisões candidatas (a ratificar em STATE.md)

- **D-LKD-001** — NATS JetStream só quando ≥ 3 produtores × ≥ 3 consumidores reais (outbox Postgres default em M1)
- **D-LKD-002** — Reputação Bronze→Diamante **determinística e auditável**; ML fica de fora
- **D-LKD-003** — Unified content filtering library (`internal/safety/content`) pra harassment, comment, message, post
- **D-LKD-004** — Avaliações pós-contrato **bilaterais e dupla-cegas** até ambos submeterem ou 14 dias
- **D-LKD-005** — Separar document fetching de ranking no OpenSearch (re-rank determinístico em app)
- **D-LKD-006** — Anti-fake Connect Me além de CNPJ: device fingerprint + IP velocity + behavioral post-signup
- **D-LKD-007** — Skills taxonomy curada manualmente (~200-500 skills BR), sem NLP auto-extract em M1/M2
- **D-LKD-008** — Grafo relacional em Postgres (`WITH RECURSIVE`) até provar gargalo; graph DB dedicado é M3+

## Findings candidatos (a registrar em AUDIT.md se lacuna confirmada)

- **A-LKD-TS-001** — Device fingerprint + IP velocity no cadastro empresa Connect Me
- **A-LKD-TS-002** — Behavioral signals pós-cadastro (throttle empresa nova que manda muitas propostas)
- **A-LKD-TS-003** — Graceful degradation policy documentada por endpoint crítico
- **A-LKD-TS-004** — Auditoria LGPD de toda ação automática (log imutável + recurso humano)

## Vizinhos

- [[../_moc]] — MOC raiz de 13-research
- [[../../CLAUDE]] — contrato do agente
- [[../../STATE]] — onde vão as D-LKD-###
- [[../../AUDIT]] — onde vão A-LKD-###
- [[../../02-architecture/patterns]] — destino de padrões (outbox, unified content filter, fetch-vs-rank)
- [[../instagram/_moc]] — pesquisa irmã (social network)
- [[../netflix/_moc]] — pesquisa irmã (infra large scale)
