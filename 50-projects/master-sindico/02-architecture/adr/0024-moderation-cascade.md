---
title: ADR-0024 — Moderação cascade (Mux + heurística + LLM + humano)
type: adr
tags: [adr, moderation, content, lgpd, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0024 — Moderação cascade: heurística → LLM → humano, evoluindo com volume

## Status

accepted — 2026-04-23

## Contexto

Conteúdo user-gerado (vídeo-currículo morador, vídeo institucional empresa/síndico, harassment reports, Connect Me messages) precisa moderação antes de expor publicamente. **Lock 90d** em vídeos ([[0010-mux-video-provider]]) torna erro irreversível → **moderação pré-publish obrigatória**.

Research cascade 2-stage: TikTok MLLM ([[../../13-research/tiktok/_destilado]] §3), YouTube Rekognition+OpenAI ([[../../13-research/youtube/_destilado]] §3), LinkedIn Trust & Safety ([[../../13-research/linkedin/_destilado]] §4). **Todos fazem**: barato filtra, caro julga, humano decide borderline.

LGPD Art. 20 exige **direito à revisão humana** de decisões automatizadas — não-negociável.

## Decisão

### Trilha por volume

| Fase | Volume estimado | Stack | Custo |
|---|---|---|---|
| **M1 (hoje)** | <100 min vídeo / <500 mensagens mês | **100% manual** SLA 48h | $0 + 2h humano/semana |
| **M2 (ano 1)** | 100-1000 min vídeo / 500-5k mensagens | **Cascade 2-stage**: Mux → heurísticas → AWS Rekognition Video ($0.10/min) + OpenAI Moderation (transcript, grátis) → fila humana mid-confidence | $10-100/mês |
| **M3 (ano 2+)** | >1000 min / >5k mensagens | Idem + custom classifier treinado em dataset próprio (Claude Sonnet em few-shot se ML-ops imaturo) | $100-1000/mês + ML-ops |

### Gatilhos para avançar fase

- Backlog manual > 48h SLA → avançar.
- Volume de vídeo > 300 min/mês sustentado → Fase 2.
- Volume manual > 1000 min/mês sustentado → Fase 3.

### Arquitetura Fase 2 (cascade)

```
[Upload Mux] → asset.ready webhook
    ↓
[ModerationCascadeOrchestrator]
    ├─→ Heurísticas (Go, rápido, grátis):
    │   • CPF/RG/telefone exposto no transcript (regex PT-BR)
    │   • Palavrões em texto (dicionário curado BR)
    │   • Duração anômala (<5s ou >5min no vídeo-currículo)
    ├─→ AWS Rekognition Video ($0.10/min) → score {nudity, violence, hate}
    ├─→ Mux auto-caption → OpenAI Moderation (texto, grátis) → flags
    └─→ Ensembler: soma scores ponderados
            ↓
        decision:
          score > 0.85 → bloqueia publish + fila manual PRIORIDADE ALTA
          0.20 < score < 0.85 → fila manual NORMAL
          score < 0.20 → libera auto; spot-check 5% pós-publish
```

### Appeals (LGPD Art. 20)

- Todo usuário cujo conteúdo foi removido/bloqueado pode **apelar**.
- Revisão humana dentro de 7 dias.
- Decisão final + motivo justificado em `/me/moderation-decisions`.
- Registro em `lgpd_logs` append-only.

### Unified content filtering library

Herança [[../../../../60-sources/master-sindico-research/linkedin/_moc]] §4 — `internal/safety/content` Go library usada por:

- Harassment report.
- Assembly comment.
- Connect Me message.
- Post body (futuro).
- Vídeo metadata (título, descrição).

Interface única, mesma log de auditoria.

## Consequências

### Positivas

- M1 zero custo infra (só tempo humano).
- M2 custo controlado com 96% automação inspiração TikTok.
- Evolução incremental — não big-bang.
- LGPD Art. 20 cumprido em toda fase (appeal humano sempre).

### Negativas

- Fase 1 exige disponibilidade humana ≥ 8h/semana.
- Cascade requer ensembler calibrado — time de mod ops precisa tune.
- Falso positivo pode gerar frustração — appeal + SLA mitigam.

## Não faremos

- **Content ID próprio** — $100M YouTube ([[../../13-research/youtube/_destilado]] §4). Usar AudD SaaS se problema emergir.
- **Custom MLLM** — overkill M1-M3.
- **Auto-unbloq** sem revisão humana — LGPD.

## Alternativas consideradas

1. **Só moderação humana para sempre** — inviável em >500 min/mês.
2. **Só ML automático** — LGPD proíbe sem appeal; FN/FP custam reputação.
3. **Hive (enterprise video moderation)** — $$, contract-only.

## Referências

- [[../../13-research/tiktok/_destilado]] §3
- [[../../13-research/youtube/_destilado]] §3
- [[../../13-research/linkedin/_destilado]] §4
- [[../../13-research/instagram/_destilado]] §4
- [[../c4-components]] §3.5 content BC
- [[0010-mux-video-provider]]
- LGPD Art. 20
