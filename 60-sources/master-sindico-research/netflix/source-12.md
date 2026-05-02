---
title: Netflix OSS — evolução e lições (o que sobrou, o que morreu)
type: source
tags: [netflix-oss, lessons-learned, open-source, spring-cloud, eureka, hystrix, research]
source: https://pravesh-sharma.medium.com/netflix-eureka-in-2025-is-it-time-to-say-goodbye-20901d15a287
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-oss-lessons]
---

# Netflix OSS: evolução e lições

> Fontes: Medium "Eureka in 2025", DZone "Future of Spring Cloud after Netflix era", Spring.io project status, Piotr Minkowski TechBlog — destilado 2026-04-23.

## Timeline canônica

- **2010-2015**: Netflix OSS era referência. Eureka, Hystrix, Zuul, Ribbon, Archaius, Atlas, Chaos Monkey, Spinnaker, EVCache, Dynomite — todos open-source.
- **2015-2018**: indústria adotou (Spring Cloud Netflix colou em todo projeto Java enterprise).
- **2018**: Hystrix entra em **maintenance mode**. Netflix anuncia publicamente.
- **2018-2020**: Netflix migra Eureka pra tooling interno; não comunica claramente.
- **2020**: Spring Cloud anuncia deprecation plan de Netflix integrations.
- **2023**: Spring Cloud oficialmente para de manter Netflix integrations.
- **2026 (hoje)**: Netflix OSS projects que sobreviveram são os que **não dependem de stack específica**: Chaos Monkey, Spinnaker, EVCache. Os que eram libs Java in-process (Hystrix, Ribbon, Eureka) foram substituídos por service mesh.

## O que sobreviveu (e por quê)

| Projeto | Status | Razão da sobrevivência |
|---|---|---|
| **Chaos Monkey** | Vivo, standalone service | Independente de linguagem; plugin pattern |
| **Spinnaker** | Muito vivo, CNCF-adjacent | Platform tool, não lib in-process |
| **EVCache** | Vivo (Netflix usa internamente), alguns adotam | Niche mas claro use case |
| **Atlas** | Vivo, Netflix usa; pequena adoção externa | Prometheus ganhou o mercado |

## O que morreu (e lições)

| Projeto | Lição |
|---|---|
| **Hystrix** | Lib in-process envelhece rápido; infrastructure-level (mesh) vence no longo prazo |
| **Eureka** | Service discovery deveria ser commodity — DNS/k8s Service resolveu |
| **Zuul (v1)** | Edge proxy single-thread não aguentou escala; Envoy/Nginx ganharam |
| **Ribbon** | Client-side LB vira sidecar hoje |
| **Turbine** | OpenTelemetry + Prometheus consolidaram métricas distribuídas |
| **Archaius** | Config dinâmico foi absorvido por platform tools (Consul, Vault, k8s ConfigMap) |

## Meta-lição

**"Netflix OSS morreu porque Netflix resolveu problemas de Netflix, que eram cloud-native AWS 2012."** Quando Kubernetes + service mesh amadureceram (2018+), o problema-alvo (operar microserviços resilientes) ficou commoditizado na infra, tornando libs in-process redundantes.

## Lição pra quem constrói SaaS 2026

1. **Não crie plataforma antes do problema existir** — Netflix tinha problema real de escala que justificou tooling custom. Sem ele, é over-engineering.
2. **Prefira standard open** — OpenTelemetry > APM proprietário; Kubernetes API > orchestrador custom; Prometheus > Atlas clone.
3. **Lib in-process vs infra-level** — infra vence em 5-10 anos. Aposte em mesh/k8s/OTel, não em lib que você vai ter que substituir.
4. **Evite fork pesado** — Netflix fez fork do Memcached (EVCache) porque tinha time. Você não tem.
5. **Open-source ≠ bem mantido** — checar commits/issues antes de adotar. Muito Netflix OSS está zombie.

## Aplicabilidade ao Master Síndico

### NÃO usar (em 2026)

- Spring Cloud Netflix (nem estamos em Java)
- Eureka, Hystrix, Ribbon, Zuul (obsoletos)
- Archaius (Viper/env no Go resolve)
- Turbine (OTel resolve)

### Usar (adaptado pra Go)

- `sony/gobreaker` (circuit breaker) em vez de Hystrix
- Kubernetes Service em vez de Eureka
- Chi/Echo/Fiber HTTP router + middleware em vez de Zuul
- OpenTelemetry SDK em vez de Turbine/Atlas
- Chaos Monkey / LitmusChaos em staging quando maduros (M3+)

### Princípio de decisão

> Antes de adotar qualquer "ferramenta Netflix", perguntar: isso resolve meu problema **hoje**, ou estou importando complexidade que Netflix construiu pra escala que nunca terei?

## Links

- [[source-03]] — deprecação detalhada
- [[source-04]] — alternativas resilience
- [[_destilado]] — síntese
