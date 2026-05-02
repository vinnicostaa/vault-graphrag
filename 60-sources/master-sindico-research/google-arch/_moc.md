---
title: MOC — 13-research/google-arch
type: moc
tags: [moc, master-sindico, v2, research, google]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/google-arch

Spanner, Bigtable, Borg→Kubernetes, Google SRE book, IAM/Identity Platform, IAP, Dapper→OpenTelemetry, Monarch, Pub/Sub, Colossus, F1, Dremel/BigQuery, Chubby.

> Pasta da v2 (remontagem do Master Síndico). 20 arquivos (1 destilado + 19 sources). Research realizada 2026-04-23 via WebSearch + WebFetch em fontes oficiais (research.google, sre.google, cloud.google.com).

## Arquivos

### Síntese
- [[_destilado]] — síntese pt-BR dos 19 sources, matriz M1/M2/M3+, decisões candidatas D-###.

### Sources (ordem temática)

**Databases distribuídos**
- [[source-01]] — Spanner (globally-distributed SQL, TrueTime).
- [[source-02]] — Bigtable (wide-column NoSQL, SSTable).
- [[source-07]] — Spanner multi-tenancy patterns (instance / database / row-based) — **alta aplicabilidade**.
- [[source-08]] — F1 (hybrid SQL on Spanner, AdWords).

**Storage e coordenação**
- [[source-06]] — Colossus (sucessor GFS).
- [[source-19]] — Chubby (lock service, Paxos).

**SRE / reliability — alta aplicabilidade M1**
- [[source-04]] — SRE book: SLI/SLO/SLA.
- [[source-09]] — SRE Workbook: Implementing SLOs.
- [[source-10]] — SRE Workbook: Error Budget Policy.
- [[source-11]] — SRE book: Postmortem Culture.

**Identity e zero-trust**
- [[source-05]] — Google Cloud IAP (Identity-Aware Proxy).
- [[source-12]] — Firebase Auth vs Identity Platform (benchmark vs Zitadel).

**Observability**
- [[source-03]] — Dapper (origem OpenTelemetry) — **já adotado**.
- [[source-13]] — Monarch (time-series planet-scale).

**Messaging**
- [[source-14]] — Pub/Sub ordering + at-least-once + idempotency.

**Cluster management**
- [[source-15]] — Borg (paper original).
- [[source-16]] — Borg, Omega, Kubernetes (evolução).

**Analytics**
- [[source-17]] — Dremel / BigQuery (columnar, disaggregated).

**Meta**
- [[source-18]] — Research at Google Publications portal (papers futuros a monitorar).

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../beyond-corp/_moc]] — pesquisa zero-trust correlata
- [[../netflix/_moc]], [[../linkedin/_moc]], [[../instagram/_moc]] — big techs correlatas (reliability patterns)
- [[../../CLAUDE]] — contrato do agente
- [[../../STATE]] — decisões vivas (destino dos D-### candidatos no destilado)
