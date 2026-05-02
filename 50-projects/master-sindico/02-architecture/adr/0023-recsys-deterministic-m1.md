---
title: ADR-0023 — Ranking Connect Me determinístico M1, reavaliação ML M3+
type: adr
tags: [adr, recsys, connect-me, ranking, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0023 — Recsys determinístico em M1, reavaliação ML em M3+

## Status

accepted — 2026-04-23

## Contexto

Connect Me é marketplace de contratação formal (síndico ↔ empresa). Precisamos de ranking de empresas em busca. Research convergente (Netflix, TikTok, LinkedIn, YouTube) diz: **ML sem dados é ruído**. LinkedIn/Netflix/TikTok/YouTube têm escala de 8-9 ordens de grandeza acima de nós no MVP. Além disso, LGPD Art. 20 exige explicabilidade de decisões automatizadas — reputação e ranking opacos são risco regulatório.

## Decisão

**Ranking Connect Me em M1/M2 é 100% determinístico + explicável**. ML só reavaliação em M3+ se volume justificar.

### Padrão 2-stage: retrieval ≠ ranking

Herança [[../../13-research/instagram/_destilado]] §1 + [[../../13-research/youtube/_destilado]] §5 + [[../../13-research/linkedin/_destilado]] §3 + [[../../13-research/tiktok/_destilado]] §1.

### Retrieval (OpenSearch)

Filtros duros:
- Categoria do serviço (ex: "hidráulica").
- Região do condomínio (raio ~20km).
- Empresa ativa + verificada.
- Disponibilidade (M2+ calendário).

### Ranking (Go-side re-rank)

Score composto ponderado:

```
score = w1 * rating_avg                          -- reputação média normalizada [0..1]
      + w2 * recency_of_last_service             -- último serviço / max(recência)
      + w3 * 1/distance_km                       -- proximidade geográfica
      + w4 * historical_success_rate             -- contratos concluídos / contratos totais
      + w5 * premium_tier_multiplier             -- Basic=1, Plus=1.1, Pro=1.2
      + w6 * reputation_tier_bonus               -- Bronze=0, Prata=0.05, ..., Diamante=0.20
```

Pesos `w1..w6` versionados em ADR sempre que mudar. M1 valores iniciais:
`w1=0.35, w2=0.15, w3=0.25, w4=0.15, w5=0.05, w6=0.05`.

### Proxy a otimizar (não CTR)

Herança [[../../13-research/youtube/_destilado]] §5 (YouTube optou por expected watch time, não CTR, para combater clickbait). Nossa métrica-alvo:

**Taxa de "contato iniciado" + "serviço concluído"**, não click em card.

### Explicabilidade

Endpoint `/api/v1/companies/:id/ranking-explain?query=...` devolve top-5 fatores e contribuições ao score. Cumpre LGPD Art. 20.

## Anti-padrões rejeitados

- **ML caixa-preta** em ranking ou reputação — LGPD + perda de diferencial.
- **DNN ou two-tower** em M1 — sem dado, vira ruído.
- **Ranking por CTR puro** — clickbait.

## Quando reavaliar ML (M3+)

Qualquer combinação:

- ≥ 10k interações/mês registradas (labels implícitos: click, contato iniciado, contratação, avaliação).
- ≥ 1k empresas cadastradas.
- Stakeholder valida que ganho de qualidade > custo operacional.

Primeira iteração ML (se entrar): **LambdaMART ou XGBoost** batch diário, resultados cached Redis. **Nunca** deep learning.

## Consequências

### Positivas

- Explicável desde dia 1 (LGPD compliant).
- Zero infra ML (sem feature store, sem serving).
- Trivial tunar via pesos (ADR amendment).
- Baseline robusto para comparar ML futuro.

### Negativas

- Pesos são subjetivos — ajuste via A/B test manual.
- Cold start: empresa nova sem reviews aparece mal (mitigação: `recency_of_signup` bonus por 30 dias).

## Alternativas consideradas

1. **ML dia 1** — descartado (sem dado, sem explicabilidade).
2. **Só ordenação alfabética / recency** — simples demais, não reflete qualidade.
3. **Collaborative filtering matrix factorization** — precisa volume; reavaliar M3+.

## Referências

- [[../../13-research/linkedin/_destilado]] §3, §5
- [[../../13-research/instagram/_destilado]] §1
- [[../../13-research/youtube/_destilado]] §5
- [[../../13-research/tiktok/_destilado]] §1
- [[../../13-research/netflix/_destilado]] §5
- [[../patterns]] §9 (Specification), §2 (Repository)
- [[../c4-components]] §3.4 commercial BC
