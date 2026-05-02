---
title: Chaos Engineering — Chaos Monkey, Simian Army, Chaos Kong, ChAP
type: source
tags: [chaos-engineering, chaos-monkey, simian-army, resilience, netflix, research]
source: https://en.wikipedia.org/wiki/Chaos_engineering
created: 2026-04-23
updated: 2026-04-23
aliases: [chaos-engineering-netflix]
---

# Chaos Engineering Netflix

> Fontes: Wikipedia Chaos Engineering, Netflix/chaosmonkey (GitHub Pages), SEI-CMU case study, Gremlin — destilado 2026-04-23.

## Origem e marcos

- **1983** — Apple, Steve Capps, "Monkey" (random UI events pra achar bugs Mac).
- **2003** — Amazon "Game Day" (Jesse Robbins).
- **2006** — Google "DiRT" (Disaster Recovery Testing).
- **2011** — Netflix **Chaos Monkey** (após migrar pra AWS). Nora Jones + Casey Rosenthal + Greg Orzell formalizam *Principles of Chaos*.
- **2014+** — Simian Army evolui; **Chaos Kong** derruba região AWS inteira.
- **Atual** — Simian Army *não mantido ativamente*; funcionalidades migraram pra **Chaos Monkey standalone** + **ChAP (Chaos Automation Platform)** interno da Netflix.

## Simian Army (hierarquia de blast radius)

| Tool | Blast radius | Quando usar |
|---|---|---|
| **Chaos Monkey** | 1 instância | Diário, baseline resiliência |
| **Latency Monkey** | Latência artificial em calls | Testar timeout/retry |
| **Conformity Monkey** | Instâncias fora de padrão | Compliance |
| **Doctor Monkey** | Health check | Observabilidade |
| **Janitor Monkey** | Garbage collection recursos | FinOps |
| **Security Monkey** | Config SG/IAM vulnerável | Security baseline |
| **Chaos Gorilla** | 1 AZ inteira | Semestral, validar multi-AZ |
| **Chaos Kong** | 1 região AWS inteira | Trimestral, validar multi-região |

## Princípios canônicos

1. Hipótese primeiro: "se X falhar, sistema deve Y".
2. Variar eventos reais (server, latência, partição de rede, erro de provider).
3. Rodar **em produção** com blast radius controlado.
4. Automatizar pra rodar contínuo.
5. **Minimizar blast radius** — aborto rápido se impacto real.

## Quando NÃO fazer chaos

- Sem monitoring capaz de detectar regressão rápido.
- Sem runbook de rollback/mitigation.
- Sem buy-in do time de operação.
- Startup com < 3 meses de prod estável.

## Ferramentas modernas (alternativas ao Simian Army)

- **Gremlin** — SaaS failure-as-a-service (pago, maduro).
- **LitmusChaos** — CNCF, Kubernetes-native.
- **Chaos Mesh** — PingCAP, Kubernetes.
- **AWS Fault Injection Service** — AWS-native, managed.

## Aplicabilidade ao Master Síndico

- **M1-M2 (até fim 2026)**: NÃO adotar. Priorizar testing tradicional (unit 95%, integration, E2E). Sem chaos.
- **M3 (2027+)**: introduzir **Latency Monkey equivalent** em staging — injetar latência artificial em providers (Stripe, Mux, Livekit) pra validar timeout/circuit breaker/fallback.
- **M4+**: Chaos Monkey em prod com blast radius = 1 pod em janela de manutenção.
- **Nunca chegaremos em Chaos Kong** — não temos multi-região nem escala que justifique.

## Aprendizado principal

"Chaos engineering não é quebrar coisas random — é **validar hipóteses específicas de resiliência** com experimento controlado." Ordem: instrumente → defina hipótese → execute em staging → só depois em prod.

## Links

- [[source-04]] — patterns de resiliência (o que o chaos valida)
- [[source-10]] — observability (pré-req pra chaos)
- [[_destilado]]
