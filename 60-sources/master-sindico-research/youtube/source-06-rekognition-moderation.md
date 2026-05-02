---
title: "Source 06 — Content moderation: AWS Rekognition, OpenAI Moderation, Hive"
type: source
tags: [research, moderation, aws-rekognition, openai-moderation, hive, ml, video]
source: https://aws.amazon.com/rekognition/pricing/
created: 2026-04-23
updated: 2026-04-23
aliases: [rekognition-pricing, moderation-apis]
---

# Source 06 — Moderação automática (ML) vs manual

## Metadados

- **AWS Rekognition pricing**: https://aws.amazon.com/rekognition/pricing/
- **AWS Rekognition content moderation**: https://aws.amazon.com/rekognition/content-moderation/
- **AWS blog (imagem vs vídeo API)**: https://aws.amazon.com/blogs/machine-learning/how-to-decide-between-amazon-rekognition-image-and-video-api-for-video-moderation/
- **OpenAI Moderation docs**: https://platform.openai.com/docs/guides/moderation (referência geral — só texto/imagem)
- **Comparativo 2026**: https://wavespeed.ai/blog/posts/best-ai-content-moderation-apis-tools-2026/

## Comparativo (preço + cobertura, abril/2026)

| Serviço | Vídeo | Imagem | Texto | Preço vídeo | Preço imagem | Observação |
|---|---|---|---|---|---|---|
| **AWS Rekognition** | ✅ | ✅ | ❌ | $0.10/min | $0.001/img (1M) → $0.0006/img (>5M) | Free: 60min vídeo/mês 12m |
| **OpenAI Moderation** | ❌ | ✅ (omni) | ✅ | — | Grátis | Só texto/imagem; sem vídeo |
| **Hive** | ✅ | ✅ | ✅ | ~$0.13/min (OCR video) | custom | Enterprise, combina AI + human review |
| **Google Vertex AI Vision** | ✅ | ✅ | — | similar a AWS | — | Integrado ao ecosistema GCP |
| **Sightengine** | ✅ | ✅ | ✅ | tier (~$0.05-0.10/min) | — | Focado em adulto/violência/nudez |

### Free tiers aproveitáveis

- AWS: 60 min vídeo/mês por 12 meses + 1000 img/mês.
- OpenAI Moderation: grátis pra clientes API (imagem/texto).

## Diferença importante: Rekognition Image API vs Video API

Pra vídeo, AWS oferece 2 caminhos:

1. **Video API** (`StartContentModeration` → `GetContentModeration`) — async, processa o arquivo inteiro, $0.10/min. Bom pra VoD.
2. **Image API frame sampling** — extrair frames (ex: 1fps) e chamar `DetectModerationLabels`. Pode sair mais barato dependendo do fps:
   - 1 frame/s em vídeo de 5 min = 300 imgs × $0.001 = $0.30 (vs $0.50 Video API)
   - 1 frame/2s em vídeo de 5 min = 150 imgs × $0.001 = $0.15
   - Trade-off: perde continuidade temporal (áudio, gestos entre frames).

## Relevância para Master Síndico

### Volume esperado MVP (estimativa)

- Piloto (1 condomínio, Sprint 10): ~5 vídeos/semana × 3min média = **15 min/semana**.
- Ano 1 (10 condomínios): ~50 vídeos/semana × 3min = **150 min/semana** ≈ **650 min/mês**.
- Ano 2 (100 condomínios): ~500 vídeos/semana × 3min = **6500 min/mês**.

### Custo AWS Rekognition Video

| Cenário | min/mês | Custo |
|---------|---------|-------|
| MVP (piloto) | 60 | **grátis** (free tier) |
| Ano 1 | 650 | $65/mês |
| Ano 2 | 6500 | $650/mês |

### Proposta

1. **Hoje (M1)**: **moderação manual 48h** — mantém. Volume é baixo demais pra justificar stack ML.
2. **Gatilho pra ML**: quando ≥300 min/mês OU quando backlog moderador >48h consistente.
3. **Primeira iteração ML**: **AWS Rekognition Video API** (Brazilian portuguese labels via `ContentModeration`):
   - Executar async pós-upload Mux (webhook → SQS → lambda Rekognition).
   - Output: score de {nudity, violence, hate, spam_visual}.
   - Score > threshold (ex: 0.85) → **bloqueia publish**, move pra fila manual prioritária.
   - Score < threshold baixo (ex: 0.20) → **libera auto**, fica em moderação passiva.
   - Mid-range → fila manual normal.
4. **Layer 2 (áudio)**: transcrever via Whisper/Mux auto-caption → passar texto em **OpenAI Moderation** (grátis) → detecta hate speech, assédio, discurso ilegal em português.
5. **Layer 3 (futuro, não MVP)**: custom classifier treinado em dados próprios do master-sindico (assuntos proibidos específicos: ofensa entre condôminos, reclamação com xingamento, etc.).

### Por que Rekognition e não Hive

- Rekognition: preço transparente, sem contrato, integra com AWS (se for nosso cloud). Free tier útil pra MVP.
- Hive: melhor em contextos NSFW niche + combina com human review (nosso caso é simples). Enterprise pricing = atrito comercial desnecessário no M1.
- OpenAI: não cobre vídeo, mas **complementar** pra texto (transcrição).

### Por que NÃO construir custom (agora)

- Training data escassa no domínio condominial.
- Overhead ML-ops (feature store, training pipeline, drift monitoring) >> valor em MVP.
- Rekognition + OpenAI resolvem 80-90% dos casos comuns.

### Lock 90d vs moderação

Importante: **moderação acontece ANTES de publish** (ou em janela de grace antes do lock). Depois do lock 90d ativar, o vídeo não pode ser editado/removido unilateralmente (só via processo de compliance/LGPD). Isto implica:
- Moderação **DEVE ser síncrona ou bloqueante** pro publish.
- Async é aceitável se janela "pending" bloquear visibilidade até aprovação.
- **Nunca aceitar upload → publish imediato sem moderação** — lock 90d torna erro irreversível barato de cometer.

## Links internos

- [[source-02-content-id]]
- [[_destilado]]
- [[../_moc]]
