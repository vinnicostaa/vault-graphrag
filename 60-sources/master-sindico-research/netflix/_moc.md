---
title: MOC — 13-research/netflix
type: moc
tags: [moc, master-sindico, v2, research, netflix]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/netflix

Microservices, Chaos Engineering, Hystrix/Eureka/Zuul (deprecação), EVCache/Dynomite, Open Connect CDN, Spinnaker, Atlas/Mantis observability, personalization.

> Pasta da v2 (remontagem do Master Síndico). Paridade com scaffold canônico em `vault/40-templates/project-scaffold/`. **Populada em 2026-04-23** com 12 fontes + destilado.

## Arquivos

- [[_destilado]] — **começar aqui**. Síntese das 12 fontes aplicada ao Master Síndico: o que usar, o que ignorar, priorização M1-M4.
- [[source-01]] — Evolução da arquitetura de API Netflix (monolito → direct → gateway aggregation → federated).
- [[source-02]] — Monolito → microservices: trigger Netflix (DB corruption 2008), pitfalls, success factors.
- [[source-03]] — Netflix OSS deprecado (Hystrix, Eureka, Zuul, Ribbon) → service mesh moderno.
- [[source-04]] — Resilience4j vs Hystrix; patterns (CB, bulkhead, retry, timeout, rate limiter) e libs Go equivalentes.
- [[source-05]] — Chaos Engineering (Chaos Monkey, Simian Army, Chaos Kong, ChAP); quando introduzir.
- [[source-06]] — Open Connect CDN: OCAs, BGP steering, proactive caching, FreeBSD/NGINX; por que NÃO construir.
- [[source-07]] — EVCache + Dynomite: cache distribuído em escala; patterns aplicáveis a Redis.
- [[source-08]] — Polyglot persistence Netflix (Cassandra, ES, MySQL, Redis, Iceberg) vs Postgres-first no nosso caso.
- [[source-09]] — Spinnaker CD (pipelines, canary, blue/green); alternativas GitHub Actions + Argo Rollouts.
- [[source-10]] — Atlas + Mantis observability; mapeamento pra OTel + Grafana + Sentry.
- [[source-11]] — Recomendação/personalização (3 camadas, bandits, Hydra); versão simplificada pra Connect Me.
- [[source-12]] — Netflix OSS: o que sobreviveu (Spinnaker, Chaos Monkey, EVCache) e o que morreu; meta-lições.

## Mapa temático

| Tema | Fonte primária | Destilado § |
|---|---|---|
| Monolito → micro | source-01, source-02 | §1 |
| Resilience (CB, bulkhead, retry, timeout) | source-03, source-04 | §2 |
| Chaos engineering | source-05 | §3 |
| CDN | source-06, source-07 | §4 |
| Recomendação | source-11 | §5 |
| Observability | source-10 | §6 |
| Lições "não copiar" | source-12 | §7 |
| Priorização M1-M4 | (síntese) | §8 |

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas (abrir D-### pós-destilação)
- [[../../02-architecture/_moc]] — destino das decisões arquiteturais
- [[../../02-architecture/adr/_moc]] — ADRs a criar
- [[../instagram/_moc]] — research-irmã
- [[../google-arch/_moc]] — research-irmã
