---
title: source-10 — Rethinking Endorsements Infrastructure (Parts 1 & 2)
type: source
tags: [research, linkedin, endorsements, avaliacoes, graph-db, kafka, samza, reputation]
source: https://engineering.linkedin.com/blog/2016/09/rethinking-endorsements-infrastructure--part-1
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-endorsements, endorsements-infra]
---

# source-10 — LinkedIn Endorsements Infrastructure

**URLs**:
- Part 1: https://www.linkedin.com/blog/engineering/infrastructure/rethinking-endorsements-infrastructure-part-1
- Part 2: https://www.linkedin.com/blog/engineering/infrastructure/rethinking-endorsements-infrastructure-part-2-the-new-endorsem
**Data**: 2016-09
**Fonte primária**: sim (LinkedIn Engineering)

## Por que essa fonte importa

**Análogo estrutural direto das avaliações pós-contrato do master-sindico.** Lições de fraude, integridade de dados e infraestrutura de grafo de relações "A endossou B em skill X".

## Pontos extraídos

### Part 1 — arquitetura original e dor
- Tudo em **SQL monolítico** single-instance.
- 10 bilhões de rows de endorsement; milhares QPS.
- Index: `recipient_id → skill → endorser_id`.
- **Limitação crítica**: buscar "tudo que Alice **endossou**" exigia full table scan de 10B rows — index hierárquico otimizou recipient lookup, não endorser lookup.
- Suggestion pipelines (offline Hadoop):
  1. **Endorsable Skills** — predizer que skill Alice poderia ter/promover.
  2. **Suggested Endorsements** — gerar tupla (recipient, skill) pra Alice endossar.
- Resultados persistidos em **Voldemort** (KV store) pra serving online.
- ML features: mutual connections, profile info, previous endorsements. 80 signals candidatos → 12 top selecionados.
- Definição de "good endorsement" adotada: "feito por conexão que **conhece a pessoa** **e** **o skill**". **Interpretável**, não caixa-preta.

### Part 2 — nova infra
- **SQL vira source of truth**.
- **GraphDB próprio** vira serving layer (antes do LIquid público).
- **Kafka + Samza** no meio: endorsement service emite eventos → Samza consumer processa + enrich + escreve no GraphDB.
- Benefícios: queries de grafo em escala (traversal bi-direcional, endorser-view rápida), mas source of truth ainda em SQL pra integridade transacional.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#7-endorsements--análogo-às-avaliações-pós-contrato-fraude-é-o-problema]].

### Análogo direto
- Avaliação pós-contrato = endorsement com estrutura: `avaliador → (contrato, dimensões, estrelas, texto) → avaliado`.
- Bilateral: síndico ↔ empresa. LinkedIn é unilateral; nós precisamos de ambos.

### Fraude — o risco real
- **Self-review via fake**: mitigar exigindo avaliação **derivada de contrato registrado com pagamento processado**. Não é input livre; é evento confirmado por cadeia transacional.
- **Conluio** (A avalia B, B avalia A, ciclo): detectar via grafo quem-avalia-quem. Ciclos densos entre poucos nodes = alerta.
- **Retaliação** (cada um avalia mal porque o outro avaliou mal): resolver via **dupla-cega até ambos submeterem ou prazo 14 dias** (padrão Airbnb, já proposto em D-LKD-004).

### Integridade de dados
- Lição do Part 1: **pensar cedo nas queries inversas**. Se nosso schema de avaliações otimiza "avaliações **recebidas** por X", a query "avaliações **dadas** por X" (útil pra detectar reviewer tendencioso) não pode virar full scan. Planejar index bi-direcional desde início.
- **Source of truth em Postgres**, como LinkedIn manteve em SQL. OpenSearch (ou view materializada) vira serving. Não inverter.

### ML features interpretáveis
- 12 top features do LinkedIn eram todas interpretáveis (mutual connections, shared profile). Nosso score de "avaliação de peso" deve seguir mesma filosofia — priorizar:
  - Avaliações de contratos **maiores** (valor, duração).
  - Avaliações **com texto** (não só estrelas).
  - Avaliadores com reputação consolidada (reputação Bronze→Diamante do avaliador afeta peso do voto dele — como em Amazon/Airbnb).
- **Determinístico sempre**. Auditável, configuração explícita em ADR.

## Quando releer

- Antes de modelar schema de avaliações (Sprint onde entrar).
- Quando surgir pressão pra "alterar avaliação" (caso real de processo) — reler Part 2 sobre imutabilidade vs correção via nova entrada.
- Ao discutir detecção de conluio.

## Vizinhos

- [[_destilado]]
- [[source-01]] — Kafka + Samza (mecanismo que LinkedIn usou; nós usamos outbox + worker)
- [[source-05]] — Trust & Safety (anti-fraud é cross-cutting)
