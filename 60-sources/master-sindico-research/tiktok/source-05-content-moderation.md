---
title: Source — TikTok content moderation com LLMs multimodais em escala
type: source
tags: [research, tiktok, trust-safety, moderation, llm, multimodal, cascade]
source: https://medium.com/@parklize/how-tiktok-uses-llms-to-power-content-moderation-at-scale-84ae2287c526
created: 2026-04-23
updated: 2026-04-23
aliases: [tiktok-moderation-llm]
---

# Source — TikTok content moderation com LLMs multimodais

## Metadados

- **Medium (Guangyuan Piao)**: https://medium.com/@parklize/how-tiktok-uses-llms-to-power-content-moderation-at-scale-84ae2287c526
- **TikTok — Transparency (content moderation)**: https://www.tiktok.com/transparency/en-us/content-moderation/
- **TikTok — Integrity & authenticity guidelines**: https://www.tiktok.com/safety/en/policies-and-engagement/integrity-authenticity
- **ICUC Social — análise policy**: https://www.icuc.social/resources/blog/tiktok-content-moderation
- **Neowork — AI+moderators**: https://www.neowork.com/insights/how-does-tiktok-moderate-content
- **Rappler (2025)**: https://www.rappler.com/technology/tiktok-expands-safety-policies-ai-moderation-creator-features/
- **PYMNTS (2025 — layoffs moderadores humanos)**: https://www.pymnts.com/artificial-intelligence-2/2025/tiktok-to-lay-off-content-moderators-and-adopt-ai-powered-solutions/
- **Autoridade**: mix — TikTok oficial (alta) + analista independente descrevendo paper/publicação TikTok sobre MLLM (média-alta, mas não paper peer-reviewed).

## Arquitetura de 2 estágios (cascade)

### Stage 1 — Recall (filtro barato)
- Modelo computacionalmente leve.
- **Embedding-based retrieval**: novo vídeo é embedado e comparado contra "bank" de exemplos de alto risco (golden seed set).
- Identifica conteúdo suspeito rapidamente; descarta low-risk.

### Stage 2 — Ranking (julgamento caro)
- **MLLM (Multimodal LLM)** baseado em arquitetura **LLaVA**, fine-tuned pra moderação.
- Analisa só o subconjunto suspeito saído do Stage 1.
- Produz **single classification token** + **probability score** (confidence).
- Anota cada vídeo com **granular issue tags** + overall moderation label.

## Humanos no loop

- Annotators escolhem "**golden seed videos**" — referência curada, diversa.
- Clustering algorithms ajudam a expandir golden set automaticamente.
- Adaptabilidade a emerging risks: novos exemplos entram no bank → Stage 1 começa a detectar novo padrão.
- **Apelações**: humanos revisam decisões contestadas.

## Escala / números

- **34M vídeos/dia** postados (cifra TikTok 2024).
- **96%** do conteúdo removido em 2024 foi derrubado **antes de 0 views** (automated).
- **Multilíngue** em dezenas de idiomas — cada idioma tem seus modelos e moderadores humanos.
- TikTok tem "milhares" de safety professionals globalmente (número exato não divulgado).

## Trend 2025: redução de moderação humana

- Reports de layoffs em times de moderadores humanos (PYMNTS, 2025).
- Substituição parcial por MLLM scaling.
- Debate: qualidade de edge cases (sarcasmo, contexto cultural) sofre.

## Aplicabilidade ao master-sindico

### Problema escopo PT-BR only

- Master-sindico é **PT-BR monolíngue**. Não sofre o problema de coordenação multilíngue.
- **Mas as lições de ML aplicam**.

### Padrão cascade 2-stage — reaproveitável

Aplicável ao fluxo de moderação de **Banco de Talentos** (vídeo-currículo) e **Connect Me** (anúncios/propostas):

1. **Stage 1 barato**:
   - Heurísticas determinísticas: CNPJ inválido (API Receita), CPF exposto no vídeo (OCR + regex), áudio muito curto, resolução inaceitável.
   - OpenAI Moderation API (texto de descrição/transcrição).
   - Nudity hash (PhotoDNA-like) — contra upload malicioso.
2. **Stage 2 caro**:
   - Claude Sonnet / GPT-4o julga vídeos marcados suspeitos com prompt estruturado: "é conteúdo apropriado pra candidatura a emprego em condomínio?".
   - Output estruturado: `{allow | review | reject}` + `reason[]` + `confidence`.

### Golden seed bank

- Cultivar coleção curada de vídeos OK + vídeos rejeitados com razão.
- Usar few-shot no prompt (mandar 5-10 exemplos positivos + 5-10 negativos contextuais).

### Appeal humano obrigatório — LGPD Art. 20

- **Toda decisão automatizada tem que ter review humano via appeal**.
- UI: morador rejeitado vê motivo + botão "solicitar revisão".
- SLA: humano revê em ≤ 48h, dá veredito final + feedback textual.
- Auditoria: log imutável de decisão automática + decisão humana.

### Moderação multimídia no master-sindico

- **Vídeo-currículo**: nudity + info sensível (CPF, endereço completo) + linguagem imprópria.
- **Vídeos empresa/síndico**: garantia de que não é propaganda enganosa + respeito ao condomínio competitor (defamation check).
- **Texto de propostas Connect Me**: scam phrases, preço absurdo, info de contato fora da plataforma (anti-desintermediação).

### Volume realista M1

- ~500 vídeos-currículo/mês + ~100 vídeos empresa/mês + ~1000 propostas Connect Me/mês = ~1600 itens/mês.
- **Viável moderação humana auxiliada por LLM** — não precisa MLLM próprio.
- Custo LLM: ~US$ 0.01/item em Claude Haiku = ~US$ 16/mês. Negligível.

## Fontes complementares

- [TikTok — approach to content moderation](https://www.tiktok.com/transparency/en-us/content-moderation/) — oficial.
- [Logora — moderation for brands](https://www.logora.com/blog-posts/tiktok-content-moderation-safeguarding-your-brand).
- [TikTok — integrity policies](https://www.tiktok.com/safety/en/policies-and-engagement/integrity-authenticity).

## Links

- [[_destilado#3. Moderação multi-idioma em escala]]
- [[_destilado#7. Trust & Safety em plataforma social]]
- [[_moc]]
