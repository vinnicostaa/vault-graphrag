---
title: ADR-0017 — Railway em M1, AWS ECS em M4+
type: adr
tags: [adr, infra, railway, aws, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0017 — Railway em M1, AWS ECS em M4+

## Status

`SUPERSEDED` — 2026-04-27 por [[0045-cloudflare-only-stack-deprecate-railway]]. Decisão cliente em [[../../STATE#D-138]]: Cloudflare é stack única; Railway descontinuado totalmente; AWS ECS M4+ também não será usada.

> **⚠️ ADR HISTÓRICA — não usar como guia operacional.** Mantida para rastreabilidade. A nova arquitetura é **Cloudflare-only** (Pages + Workers + Containers + R2 + KV/DO + Hyperdrive→Postgres externo).

### Status original (revogado)
accepted — 2026-04-23 (herdado D-024 legado, AWS migration plan)

## Contexto

M1 (2026-05-08) precisa infra funcional o mais rápido possível, com operação zero. M3+ com SLO 99.9% e compliance (SOC 2) exige controle mais fino, multi-AZ, custos previsíveis.

## Decisão

**Railway** em M1/M2 (até ~100 condomínios / 2k users); **AWS ECS Fargate + RDS + ElastiCache** em M4+ (quando SLO 99.9% e compliance formal entrarem).

### M1 — Railway

- **Compute**: 1 container API + 1 worker (scalable).
- **DB**: PostgreSQL 18 managed (Railway).
- **Cache**: Redis 7 managed (Railway).
- **Object storage**: Cloudflare R2 (S3-compatible, sem egress fee).
- **Deploy**: GitHub Actions → Railway (push to `main` → staging; manual promote → prod).
- **Logs**: Railway logging + shipping opcional pra Grafana Loki.
- **Assinatura**: $20-100/mês baseline.

### M4+ — AWS ECS Fargate

Quando atingir gatilho (abaixo) migrar para:

- **Compute**: ECS Fargate (task per BC future microservice).
- **DB**: RDS PostgreSQL 18 Multi-AZ.
- **Cache**: ElastiCache Redis 7 cluster mode.
- **Object storage**: S3 + CloudFront.
- **DNS**: Route53.
- **WAF**: AWS WAF + Shield.
- **Logs**: CloudWatch + shipping para Grafana.
- **Deploy**: CodePipeline ou GitHub Actions → ECR → ECS blue/green.

### Gatilhos de migração

Qualquer um destes aciona ADR de migração:

1. >500 condomínios ativos (volume de escala).
2. SLO 99.9% contratado com cliente enterprise.
3. Compliance formal (SOC 2, ISO 27001).
4. Cliente enterprise exige residência de dado em região específica.
5. Custo Railway > 3× equivalente AWS baseline.

## Consequências M1 (Railway)

### Positivas

- Zero opex em M1.
- Deploy em minutos.
- DB managed + upgrade automático.
- Focar em produto, não em infra.

### Negativas

- Sem multi-AZ nativo (Railway tem alta disponibilidade mas não é AZ failover clássico).
- Escalabilidade vertical limitada; horizontal ok até ~5 instâncias.
- Custo escala com CPU/RAM/bandwidth de forma opaca.
- Menor controle fino (cgroups, custom network, VPC peering).

## Consequências M4+ (AWS)

### Positivas

- Multi-AZ + disaster recovery reais.
- Auto-scaling fine-grained.
- VPC isolation, security groups, PrivateLink.
- Compliance AWS already (SOC 2, ISO, PCI) aproveitável.

### Negativas

- Opex: dev-ops em time dedicado ou 1 FTE.
- Migração custa 4-8 semanas de um sênior.
- Vendor lock maior.

## Alternativas consideradas

1. **Fly.io** — similar a Railway; preço ok; multi-region nativo — reavaliar em M3 se BR-only insuficiente.
2. **GCP Cloud Run** — serverless HTTP; requer refactor session handling.
3. **Self-host K8s (Hetzner/OVH)** — custo mais baixo; opex pesado.
4. **Heroku** — caro, deprecated em features.
5. **Straight AWS desde M1** — overkill de opex.

## Referências

- [[../c4-containers]] §2
- [[../../13-research/netflix/_destilado]] §8 (priorização M1/M2/M3)
- Histórico legado: D-024 (Railway vs ECS em aberto no legado; este ADR consolida).
