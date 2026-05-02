---
title: Source — Transparência algorítmica TikTok + DSA EU
type: source
tags: [research, tiktok, transparency, algorithm, dsa, eu, regulation, lgpd, policy]
source: https://digital-strategy.ec.europa.eu/en/news/commission-preliminarily-finds-tiktoks-addictive-design-breach-digital-services-act
created: 2026-04-23
updated: 2026-04-23
aliases: [tiktok-transparency, tiktok-dsa]
---

# Source — Transparência algorítmica TikTok e debate regulatório

## Metadados

### Fontes oficiais
- **EU Commission (2026-02-06) — preliminary finding DSA**: https://digital-strategy.ec.europa.eu/en/news/commission-preliminarily-finds-tiktoks-addictive-design-breach-digital-services-act
- **TikTok — Transparency Center**: https://www.tiktok.com/transparency/en-us/
- **TikTok — Transparency and Accountability Center (Dublin page)**: https://www.tiktok.com/transparency/en-us/tac-dublin
- **TikTok Developers — open-source AI for privacy**: https://developers.tiktok.com/blog/transparency-by-design-open-source-ai-privacy
- **TikTok Newsroom — launch TAC**: https://newsroom.tiktok.com/en-us/tiktok-to-launch-transparency-center-for-moderation-and-data-practices

### Análises / debate
- **Verfassungsblog — TikTok addictive design & DSA**: https://verfassungsblog.de/tiktok-dsa/
- **Schjødt — legal analysis**: https://schjodt.com/news/addictive-platform-design-under-dsa-scrutiny-the-european-commissions-preliminary-findings-on-tiktok
- **Slaughter and May — legal analysis**: https://www.slaughterandmay.com/insights/new-insights/tiktok-s-addictive-design-provisionally-found-to-be-a-breach-of-the-dsa-where-is-the-line-between-engagement-and-addiction/
- **AI4Pol — DSA addictive design**: https://ai4pol.eu/news/dsa-takes-on-addictive-design
- **Mozilla Foundation — transparency critique**: https://www.mozillafoundation.org/en/blog/is-transparency-trending-on-tiktok/
- **EDRi (European Digital Rights) — welcome of finding**: https://edri.org/our-work/edri-welcomes-eu-preliminary-findings-on-tiktoks-addictive-platform-design/
- **Brandon Silverman — Substack analysis**: https://brandonsilverman.substack.com/p/how-transparent-is-tiktok

**Autoridade**: mix — fontes primárias (Comissão Europeia, TikTok oficial) + análises jurídicas (escritórios advocacia) + crítica civil (Mozilla, EDRi). Debate bem documentado.

## Transparency and Accountability Centers (TAC)

- Locais: **Dublin, Los Angeles, DC, Singapura**.
- Modelo: **visitas sob NDA**. Visitantes (jornalistas, reguladores, acadêmicos) podem ver:
  - Source code da plataforma.
  - Fluxo do algoritmo FYP (For You Page).
  - Data flows, key processes.
  - **Code simulation** interativa.
- Propósito declarado: **gerar confiança**, responder críticas sobre caixa-preta.

### Crítica pública (Mozilla, EDRi)

- Acesso restrito por NDA — quem vê não pode publicar detalhes.
- TikTok define o que "audit" significa; sem garantia de independent civil rights audit.
- "Teatro de transparência" (Mozilla Foundation).

## Dedicated Transparency Centers (DTC) — Projeto Texas

- Oracle + "Independent Security Inspectors" têm **full-time access** ao source code completo.
- Cobre: mobile app code, backend services, third-party libraries, recommendation algorithms.
- **5 DTCs** ativos: primeiro em Columbia/Maryland (2023) + US, UK, Austrália.
- **100+ analistas** com acesso contínuo.
- Contexto: resposta à pressão regulatória US (ban ou forced divestiture 2024-2025).

## DSA — Violação preliminar (fevereiro 2026)

### Achado da Comissão Europeia

- **Data**: 2026-02-06.
- **Acusação**: TikTok viola DSA Art. 25 e 27 por "**addictive design**".

### Features consideradas problemáticas

- **Infinite scroll** — sem breakpoints naturais.
- **Autoplay** de próximo vídeo.
- **Persistent push notifications**.
- **Highly personalized recommender systems** — rabbit hole effect.
- Especialmente preocupante pra **menores e adultos vulneráveis**.

### Multa potencial

- Até **6% do faturamento global anual** por violação DSA.
- Para TikTok/ByteDance, isso significa **ordem de grandeza US$ bilhões**.

### Histórico de investigações paralelas

- **Advertising transparency** — fechado via binding commitments em dezembro 2025.
- **Researcher access** — preliminary findings em outubro 2025.
- **Addictive design** — fevereiro 2026 (atual).

## Aplicabilidade ao master-sindico

### Threshold DSA não se aplica

- DSA só atinge **VLOPs** (Very Large Online Platforms) com **45M+ usuários ativos mensais na UE**.
- Master-sindico opera no Brasil com escala condominial. Sem exposição direta.

### MAS: marco regulatório brasileiro em gestação

- **PL 2.630/2020** ("Fake News Bill") — discussão ativa 2024-2026; inclui artigos sobre transparência de algoritmos.
- **ANPD + LGPD Art. 20** — já vigente. "Direito à revisão por pessoa natural de decisões tomadas unicamente com base em tratamento automatizado dos dados pessoais".
- **CDC** — direito do consumidor à informação clara. Aplicável a decisões automatizadas que afetam serviço.

### Decisões automatizadas no master-sindico que precisam transparência

1. **Ranking de empresas no Connect Me** — por que essa empresa apareceu primeiro?
2. **Moderação de vídeo-currículo (Banco de Talentos)** — por que meu vídeo foi rejeitado?
3. **Score de reputação de síndico** — como foi calculado?
4. **Moderação de propostas Connect Me** — por que minha proposta foi bloqueada?
5. **Triagem/sugestão de respostas automáticas** (se houver).

### Padrão canônico recomendado

Toda decisão automatizada que impacta o usuário final exibe:

- **Top-K fatores** (feature importance simplificada em linguagem natural).
- **Link pra appeal humano** (revisão em ≤ 48h).
- **Log imutável** pra auditoria (input + modelo + versão + output + decisão humana subsequente).
- **Direito de explicação** — usuário pode pedir "me explica por que" em UI.

### Dark patterns a evitar no master-sindico

Inspirado em achados da EU Commission:

- **Não usar infinite scroll viciante** no feed de vídeos/descoberta. Usar paginação explícita ou "carregar mais".
- **Não autoplay agressivo** — síndico é adulto profissional, não quer experiência TikTok-ificada.
- **Push notifications controlados** — default restritivo (quiet hours, categorias desabilitadas por default).
- **Não rabbit hole** — feed de descoberta deve ter **breakpoints naturais** (ex: "essas são as últimas 20, volte amanhã").
- Audiência-alvo é profissional/utilitária, não entretenimento. **Design ético é também produto-mercado fit**.

## Links cruzados

- [[../../08-security/beyond-corp-references]] — destino de princípios de transparência na migração.
- [[_destilado#8. Transparência algorítmica e debate público]]
- [[_moc]]
