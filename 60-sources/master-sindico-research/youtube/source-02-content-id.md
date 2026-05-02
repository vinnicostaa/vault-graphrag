---
title: "Source 02 — YouTube Content ID (fingerprinting de copyright)"
type: source
tags: [research, youtube, copyright, content-id, fingerprinting, moderation]
source: https://support.google.com/youtube/answer/2797370
created: 2026-04-23
updated: 2026-04-23
aliases: [content-id, copyright-match-tool]
---

# Source 02 — YouTube Content ID

## Metadados

- **Fonte oficial**: https://support.google.com/youtube/answer/2797370?hl=en
- **Complementar**: https://www.youtube.com/howyoutubeworks/copyright/
- **Wiki técnico**: https://en.wikipedia.org/wiki/Content_ID
- **Tipo**: produto/sistema do YouTube, operante desde ~2007

## O que é

Sistema de **digital fingerprinting** que compara uploads contra uma base de reference files fornecida por rights holders (labels, estúdios, agregadores). Cobre áudio + vídeo + melodia.

## Como funciona (síntese)

1. **Reference files**: rights holder sobe master do conteúdo (áudio/vídeo) e metadados de ownership por território.
2. **Fingerprinting**: YouTube gera fingerprint (hash perceptual) — robusto contra modificações (flip horizontal, stretch, pitch shift, crop, color change).
3. **Scan**: todo upload é scanned automaticamente no pipeline de ingestão.
4. **Match**: se bate com reference, gera um **Content ID claim**.
5. **Ação configurada pelo rights holder** — 3 opções:
   - **Block**: impede visualização (por país se quiser).
   - **Monetize**: roda ads, compartilha receita.
   - **Track**: só coleta stats (sem bloqueio, sem ad).

## Ferramentas relacionadas

- **Copyright Match Tool**: versão simplificada pra creators comuns (detecta reuploads).
- **Webform DMCA**: disponível em 80+ idiomas.
- **Content ID propriamente dito**: gated — só pra rights holders com "substantial body of original material" frequentemente re-uploaded.

## Escala (dados oficiais)

- 1 bilhão+ de claims Content ID no H2/2023.
- <1% contestado; 65%+ das disputas resolvidas por claimant withdrawing.
- Investimento cumulativo: $100M+ (até 2018, declarado por Google).
- Cobre ~98% de copyright infringement conhecido (declaração Google 2016).

## Features técnicas de fingerprinting

- Áudio: robusto contra pitch shift, time stretch, EQ, noise.
- Vídeo: robusto contra crop, aspect ratio change, flip, color grading.
- Melody detection: detecta melodia mesmo re-interpretada.
- Visual recognition: logos, imagens, animações.
- Metadata matching: título/descrição/tags.

## Relevância para Master Síndico

### Aplicável? **Parcialmente e só como conceito.**

**Contexto do sub-produto Conteúdo/Vídeos**:
- Quem sobe: empresas/prestadores fazendo vídeo **institucional próprio** (apresentação de serviços, depoimentos, tutoriais condominiais).
- Violação de copyright por malícia direta é baixa — empresa não costuma subir filme pirata em canal profissional.
- Mas **violação inocente** existe: trilha sonora com música licenciada, clipe de TV como background, voice-over sobre áudio de outro canal.

### Proposta pragmática

- **NÃO construir fingerprinting próprio** — custo-benefício ruim pro nosso volume.
- **Usar sinal terceiro** quando relevante: Mux não faz copyright detection nativamente. Se virar problema, integrar **AudD** (https://audd.io, ~$10/mês por volume baixo) ou **ACRCloud** (fingerprint SaaS) só em spot-check de vídeos de maior alcance.
- **Moderação manual 48h** já pega 90% de casos problemáticos (moderador reconhece Adele tocando de fundo).
- **Cláusula contratual**: termo de uso obriga empresa a declarar posse de direitos; violação = takedown + ban. Mais eficaz e barato que tech solution.
- **Lock 90d** do master-sindico protege contra alteração pós-publish, mas NÃO substitui copyright detection (vídeo já está no ar).

### Lição aplicável

- A arquitetura Content ID ensina que **ações configuráveis (block/monetize/track)** são melhor UX que binário. Para master-sindico: rights holder hipotético (ex: condomínio que quer remover vídeo antigo sobre fornecedor) deveria ter opção **block em tenant específico** sem derrubar em todos.

## O que NÃO aplicar

- Fingerprint DB global — irrelevante pro nosso domínio (não somos plataforma de mídia pública).
- Revenue share via claim — não temos monetização por views.

## Links internos

- [[_destilado]]
- [[source-06-rekognition-moderation]]
- [[../_moc]]
