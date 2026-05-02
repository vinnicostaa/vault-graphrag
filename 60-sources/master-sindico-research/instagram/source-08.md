---
title: Instagram — moderação de conteúdo (AI + humano)
type: source
tags: [research, instagram, meta, content-moderation, ml, nlp, cnn, rosetta, community-standards, source]
source: https://help.instagram.com/423837189385631 + https://transparency.meta.com/reports/community-standards-enforcement/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-08-moderation]
---

# source-08 — Instagram content moderation (AI + human)

- **URLs consultadas**:
  - https://help.instagram.com/423837189385631 (Instagram Help oficial — fetch bloqueado, apenas metadata)
  - https://transparency.meta.com/reports/community-standards-enforcement/ (Meta Transparency Center)
  - https://about.fb.com/news/2018/11/enforcing-our-community-standards-2/
  - Análise terciária: analyticsvidhya.com (2025)
- **Publicado**:
  - 2018-11 — Meta Newsroom sobre Community Standards (proactive detection 24% → 52%)
  - 2020-06 — Meta reporta ~95% proactive detection de hate speech
  - 2025 Q2 — mudança de política (reduziu escopo proativo, −50% total removals)
- **Tipo**: mix de docs oficiais Meta + análise terciária
- **Ressalva**: componentes técnicos específicos (Rosetta OCR CNN+LSTM, YOLO, transformer NLP) vieram de análise terciária (Analytics Vidhya). Rosetta é documentado por Meta em outros posts (não pesquisado nesta sessão). Fluxo híbrido (AI + humano) é documentado oficialmente.

## Sinopse

Instagram usa pipeline híbrido: classificadores ML (imagem, texto, OCR) + fila de revisão humana + feedback loop. Meta publica taxas de detecção proativa trimestralmente no Community Standards Enforcement Report.

## Pontos-chave

1. **Classificadores**:
   - **Texto**: transformers + RNN pra dezenas de línguas.
   - **Imagem**: CNN classifier treinado em dataset labelled; object detection (YOLO / Faster R-CNN) pra localizar conteúdo explícito.
   - **OCR**: Rosetta (Meta) extrai texto sobre imagem com CNN + LSTM.
2. **Confidence threshold**: alto → auto-remove; médio → fila humana; baixo → passa.
3. **Feedback loop**: decisões humanas retreinam modelos.
4. **Community Standards Enforcement Report** publica métricas trimestrais (proactive rate, prevalence, actioned content).
5. **Histórico de proactive rate em hate speech**: 24% (2018 Q1) → 52% (2018 Q3) → ~95% (2020 Q2).
6. **2025 pivot**: Meta reduziu escopo proativo; espera report de usuário pra conteúdo menos grave; takedowns caíram ~50%.

## Aplicabilidade ao Master Síndico

Detalhado em `_destilado.md` §4.

**Cronograma sugerido**:

- **M1**: 100% manual. Síndico modera seu condomínio; Admin Master modera cross-tenant. Fila de denúncia + SLA definido.
- **M2**: Filtros estáticos (regex CPF/RG/tel, blocklist palavras, hash de imagens banidas). Zero ML.
- **M3**: Avaliar **API externa** antes de ML próprio — Google Perspective, Azure Content Moderator, AWS Rekognition. Menos custo, menos manutenção, atende 80% dos casos.
- **M4+**: Só então considerar ML próprio pra PT-BR + contexto condominial (jargão, siglas), se volume justificar.

**Patterns a adotar desde M1**:

- **Sinais negativos tipados** — botão "reportar" com motivo estruturado.
- **Confidence threshold / fluxo auto-humano** — mesmo que "classificador" seja regex, o fluxo deve estar em lugar.
- **Feedback loop** — decisões de moderador viram labels gravados, pré-dataset pra ML futuro.
- **ABAC**: `moderate_content` escopado (síndico → próprio; admin → cross).

**Não replicar**: Rosetta, transformer próprio, OCR custom — não cabe no horizonte visível.

## Confiança / datas

- Taxas de proactive detection 2018/2020 — **alta confiança** (Meta Newsroom oficial).
- Mudança 2025 — **alta confiança** (múltiplas fontes: Oversight Board, emarketer).
- Componentes técnicos (Rosetta, YOLO etc.) — **média confiança** (análise terciária; não verificado em blog oficial Meta nesta sessão).
