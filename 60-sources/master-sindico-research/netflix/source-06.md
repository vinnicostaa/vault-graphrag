---
title: Netflix Open Connect CDN — arquitetura de distribuição de vídeo
type: source
tags: [cdn, open-connect, netflix, video-delivery, peering, bgp, freebsd, research]
source: https://www.anthony-balitrand.fr/2024/09/26/under-the-hood-netflix-open-connect/
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-open-connect]
---

# Netflix Open Connect: arquitetura CDN proprietária

> Fontes: Anthony Balitrand (set/2024), APNIC blog, openconnect.netflix.com, vdocipher — destilado 2026-04-23.

## Números (escala)

- ~**12.000 OCAs** em ~150 países.
- ~**265 Tbps** em peak (2024).
- **~95% do tráfego Netflix** entregue por peering direto com ISPs.
- **~7,5 TB/dia** de catálogo atualizado por OCA.
- Open Connect economizou **~US$1-1,25 bi/ano** pra ISPs em 2021 (redução de bandwidth upstream).

## Arquitetura em 2 camadas

1. **OCA embarcada no ISP** — hardware grátis pra qualifying ISPs. ISP fornece energia + espaço + conectividade. Netflix fornece HW + content management.
2. **OCA em IXP (Internet Exchange Point)** — fallback/complement, peering settlement-free.

## Modelos de hardware

| Modelo | Storage | Throughput | Uso |
|---|---|---|---|
| Global (2U, ~270W) | até 120 TB HDD | ~18 Gbps | ISPs menores |
| Storage (2U, ~650W) | até 360 TB mix HDD/SSD | ~96 Gbps | IXPs, grandes ISPs |
| Flash (2U, ~400W) | 24 TB all-flash | ~190 Gbps | Conteúdo popular |

OS: **FreeBSD**, servidor: **NGINX**.

## Estratégia-chave: proactive caching (não reativa)

- CDNs genéricas (Cloudflare, Akamai, Fastly) são **reactive** — cacheiam no miss.
- Open Connect é **proactive** — Netflix **prevê** o que cada região vai assistir e faz fill durante **off-peak** (2-6 AM local, "fill window").
- Content propaga hierarquicamente: OCAs "master" recebem primeiro, depois distribuem pra outras OCAs próximas (evita bottleneck S3).
- Novos títulos cacheados **dias antes** do release.

## Otimizações técnicas

- **Zero-copy networking** (`sendfile()` FreeBSD) → disk → socket sem context switch.
- **HDD positioning** — popular titles na borda externa do platter (menor seek time).
- **SSL offloading na NIC** — TLS pós-handshake feito pela placa.
- **BGP-driven steering** — ASN 40027; seleciona OCA ótima por availability + AS path + distância geo + MED.

## Por que Netflix construiu CDN própria (não usa Akamai)

- Escala justifica (TB/s sustained → CDN genérica seria proibitivamente cara).
- Otimizações específicas pra vídeo streaming long-form (proactive cache baseado em conhecimento de catálogo).
- Controle total do plane de routing.

## Trade-off

- Custo fixo enorme de HW + programa de parceria global.
- Inflexível pra non-video workloads.
- Só faz sentido acima de **~1 EB/mês** de delivery.

## Aplicabilidade ao Master Síndico

**Não construir CDN própria. Nunca.**

### Vídeo (sub-produto Conteúdo)

- **Mux já resolve** — entrega via sua CDN (Fastly + outros), com ABR, DRM, analytics. Custo variável mas aceitável pro tamanho do projeto.
- Aprender: **prefetching baseado em predição** (ex: quando usuário entra na tela de curso, prefetch próximos capítulos) — aplicável em layer cliente.

### Assets estáticos (sub-produto Governança, Connect Me)

- Cloudflare R2 + Cloudflare CDN é suficiente (preço/performance imbatível no BR).
- Fastly como alternativa se precisar edge computing.

### Insight aplicável sem construir CDN

- **Cache warming** em deploy — popular cache de catálogo de empresas do Connect Me após deploy, antes de abrir tráfego (evita cold start).
- **Proactive invalidation** — quando edita empresa, invalida cache de ranking antes do primeiro hit.

## Quando reavaliar

- Projeto Conteúdo tiver > 10 TB/mês sustained em vídeo (improvável nos primeiros 3 anos).
- Mux virar gargalo de custo (> R$ 50k/mês só em bandwidth).

## Links

- [[source-07]] — caching EVCache (cache patterns aplicáveis)
- [[_destilado]]
