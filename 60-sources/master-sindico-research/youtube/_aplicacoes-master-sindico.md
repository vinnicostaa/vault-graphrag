---
title: "YouTube — Aplicações concretas ao Master Síndico (vídeos, LMS, lives, Banco de Talentos)"
type: note
tags: [research, youtube, aplicacoes, master-sindico, mux, moderation, lms, live, banco-talentos, accessibility]
source: 13-research/youtube/_destilado.md + sub-produtos 04/05/06/07 + CNT-* + ADR-0010/0011/0024
created: 2026-04-23
updated: 2026-04-23
aliases: [youtube-aplicacoes, youtube-cross-ms]
---

# YouTube → Master Síndico (aplicações)

> Cruzamento operacional do research `youtube/_destilado.md` (8 fontes) com **04-conteudo-videos**, **05-assembleia-live**, **06-lms-academia** e **07-banco-de-talentos**. Foco M1 (2026-05-08) + trilha M2/M3. Todas decisões herdam D-056 (agnóstico de provedor) e D-058 (admin transversal).

---

## 1. Mapa rápido (o que o YouTube resolve × o que o MS precisa)

| Preocupação YouTube | Hoje no MS | Aplicação proposta | Sub-produto | Criticidade M1 |
|---|---|---|---|---|
| Ingestion pipeline (GCS→VCU→Colossus) | Mux Direct Upload tus.io + webhook `video.asset.ready` (CNT-001, ADR-0010) | **Manter**. Adicionar DLQ webhook + idempotência por `mux_webhook_id` (patterns §11, §14) | 04/06/07 | Alta |
| Transcoding ladder | Default Mux 360/480/720/1080p (ADR-0010) | **Confirmar** que 360p tem cap ≤ 800 kbps (3G/4G BR sub-urbano) — hoje não há ADR explicitando bitrate mínimo | 04/06/07 | Média |
| Content ID moderation | 100% manual 48h (CNT-003, ADR-0024 fase 1) | **Manter M1**. Fase 2 cascade (Rekognition $0.10/min + OpenAI Moderation transcript + ensembler) já descrita em ADR-0024 | 04/06/07 | Baixa |
| ABR/HLS player | Mux Player via `IVideoProvider` (CNT-010) | **Manter**. Habilitar Mux Data (QoE) desde dia 1 (já em ADR-0010) — verificar se `infrastructure/providers/mux/` está instrumentando | 04/06/07 | Média |
| Autogen captions | Não especificado (CNT-015 "SHOULD", C/M3+) | **Elevar para M2** em LMS (06) e Banco de Talentos (07) — LGPD+eMAG acessibilidade | 06/07 | Alta (eMAG) |
| Thumbnails auto | Mux gera (`GetThumbnailURL`) (04 §6.1) | **Confirmar preview em UI** + allow override manual por dono (CNT-032 já prevê; validar implementação) | 04/06/07 | Baixa |
| DRM premium | Signed URLs JWT TTL curto (ADR-0010 §DRM NÃO M1) | **Manter sem DRM** em M1/M2. LMS cursos pagos M3+ reavaliar só se houver evidência de piracy (ver §5) | 06 | Baixa |
| Live streaming HLS (5-10s delay) | Assembleia = LiveKit WebRTC SFU <400ms (ADR-0011); LMS Live = Mux Live RTMP (CNT-022 pendência) | **Assembleia: manter LiveKit** (voto exige <500ms). **LMS Live: Mux Live RTMP basta** (broadcast admin-only, D-062) | 05/06 | Média |
| Recommendation 2-stage | Retrieval+ranking heurístico previsto em ADR-0023 (deterministic M1) | Aplicar em Connect Me (02) e em recs cross-domain LMS (CNT-007) — **fora do escopo desta análise** | 02/06 | — |
| Lock pós-publish (freshness vs auditoria) | GR-029 lock 90d (CNT-004) | **Já temos, é nosso diferencial**. Estender formalmente a LMS (INV-LMS-C / RC-7 gap G-14 em 06) | 04/06 | Alta (gap G-14) |
| Perceptual hash anti-duplicata vídeo-currículo | Não existe | Fase 2 (M3+) se emergir spam; hoje lock 90d + 1 currículo/morador (INV-BT-05) basta | 07 | Baixa |

---

## 2. Moderation — proposta consolidada

**Estado**: ADR-0024 já formaliza cascade 2-stage `Mux + heurísticas → Rekognition + OpenAI Moderation → ensembler → fila humana`. YouTube Content ID (fingerprint) **não** vai ser implementado — cláusula contratual cobre 90% (destilado §4).

**Ação M1 (nada novo)**: manter CNT-003 manual 48h. Admin transversal D-058 faz a review.

**Ação M2 (refinamento, não novo)**: instanciar pipeline ADR-0024 Fase 2 quando qualquer gatilho disparar (volume >300 min/mês OU backlog >48h). Pontos que o research YouTube acrescenta e que ainda não estão no ADR-0024:

1. **Unified content filter lib** — ADR-0024 já cita `internal/safety/content` (Linkedin-inspired) pra reuso em harassment/Connect Me/assembly comment/vídeo. **Reforço**: incluir `curriculum_videos.transcript` (07) e `course_lessons.transcript` (06) como inputs da mesma lib — hoje não estão listados como consumers.
2. **Appeal LGPD Art. 20** — ADR-0024 prevê appeal 7d. **Confirmar**: endpoint `/me/moderation-decisions` existe em CNT- ? Não encontrei. Adicionar como **A-### novo em AUDIT**.
3. **Cascade stage 0: pHash duplicate check** (ainda não no ADR) — só se volume BT escalar; adiar.

**Não fazer**: Content ID próprio (custo), Hive (enterprise lock), auto-unblock sem humano (LGPD).

---

## 3. Transcoding ladder — bitrates adequados Brasil

**Estado**: ADR-0010 usa default Mux (360/480/720/1080p, 6s segment, 2s keyframe). Não há ADR explicitando bitrates-alvo.

**Dado de mercado (destilado §2 + Mux best-practices)**: banda residencial BR urbana sustenta 1080p; 4G sub-urbano/rural sustenta 480-720p com rebuffer; 3G (ainda presente em municípios pequenos) sustenta 360p.

**Recomendação**:
- Confirmar (com QoE Mux Data) que ladder default cobre p95 3G BR ≤ 800 kbps em 360p. Mux ajusta — só auditar pelo rebuffer ratio em prod.
- **Não** adicionar 240p (destilado §TL;DR) — empresa condominial não está nessa faixa.
- **Não** habilitar 4K (custo + sem payoff institucional).
- H.265 opt-in (quando volume justificar, M3+).
- **Não** ativar AV1 (Mux Data mostra cliente nativo inconsistente — ADR-0010 "premium tier" já dita).

**Ação**: nenhuma mudança M1. Em M2, adicionar dashboard Grafana "QoE M2: startup time p95 < 2s, rebuffer ratio < 1%" com dados Mux Data (já instrumentado per ADR-0010).

---

## 4. Assembleia Live: por que WebRTC e não HLS

**Estado** (ADR-0011 + 05-assembleia-live §1-6): LiveKit Cloud, SFU, <400ms, bidirecional, Egress Room Composite MP4 pra ata. Votação reliable DataStream + fonte de verdade em Postgres (HTTP POST).

**Research YouTube Live** (destilado §7): RTMP/RTMPS/HLS/DASH com 5-10s delay — **irrelevante pra assembleia** porque voto ao vivo + fala + Q&A exigem <500ms. YouTube Live serve broadcast monológico.

**Decisão reforçada**:
- **Assembleia (05)**: LiveKit WebRTC — correto. ADR-0011 cobre.
- **LMS Live (06) — CNT-022 pendência**: broadcast admin-only (D-062). Aqui **Mux Live RTMP basta** (delay 5-10s aceitável: síndico apresentando obra/palestra sem Q&A síncrona; chat tolera delay). Decisão proposta: **Mux Live como provider do LMS Live** (reaproveita `IVideoProvider` + `ILiveStreamProvider` já implementado em `content/infrastructure/providers/mux_provider.go` per 04 §1.4). Fecha CNT-022.
- **Não** unificar LiveKit com LMS Live — LiveKit para broadcast 1→N é overkill; bandwidth cost explode.

**Lição arquitetural YouTube aplicada**: separar ingest (RTMPS/WebRTC) de delivery (HLS) — ADR-0011 já faz isso no lado Assembleia (WebRTC ingress + MP4 egress via Egress Room Composite para ata). Padrão replicável em LMS Live (RTMP ingress + HLS egress via Mux).

---

## 5. DRM e signed URLs — cursos LMS pagos

**Estado**: ADR-0010 diz "DRM NÃO em M1. Signed URLs JWT TTL 5-15min". INV-070 trava TTL ≤ 24h em playback LMS.

**Research YouTube**: DRM só em Premium (Widevine/PlayReady/FairPlay). Custo operacional alto, UX friction (player compatibility matrix), rotação de keys, CDN premium.

**Decisão proposta** (ratifica ADR-0010):
- **M1/M2/M3**: Signed URLs JWT TTL 15min + lock 90d (pra vídeos institucionais) + LGPD audit trail de playback. **Sem DRM.**
- **Gatilho pra revisitar**: telemetria Mux Data mostrar hot-linking sistemático (download via stream capture ≥ 1% dos plays) em cursos pagos M4+.
- Se gatilho disparar: Mux Signed Playback URLs (já temos) + fingerprint do player (Widevine SL3 mobile, ClearKey web) — ainda não é DRM full, é anti-hotlink reforçado.

**Não fazer em MVP**: Widevine L3, FairPlay, PlayReady — cada um é 6-12 semanas de integração.

---

## 6. Captions / legendas — acessibilidade LGPD + eMAG

**Estado**: CNT-015 "subtítulos Mux" marcado **C (M3+)**. Destilado §2 recomenda captions auto "habilitar desde dia 1 — acessibilidade + base pra moderação de áudio".

**Regulatório BR**:
- **eMAG** (Modelo de Acessibilidade de Governo Eletrônico) — recomenda legendas.
- **LGPD** — não exige, mas acessibilidade trata-se de inclusão (public-facing apps condominiais têm moradores idosos/surdos).
- **Lei 13.146/2015 (LBI — Estatuto da Pessoa com Deficiência)** — Art. 67: comunicação acessível. Aplicável a cursos LMS certificados (06).

**Proposta**: **elevar CNT-015 de C (M3+) para S (M2)** como default ON em:
- **06 LMS**: toda vídeo-aula (`course_lessons.video_id`) — eMAG + LBI compliance. Mux autogen PT-BR.
- **07 Banco Talentos**: vídeo-currículo 90s — acessibilidade + input pra moderação (transcript → OpenAI Moderation per ADR-0024 cascade).
- **04 Conteúdo & Vídeos**: default OFF, toggle ON por dono (empresa pode não querer legenda em pitch muito curto).

**Custo**: Mux autogen ~$0.03/min. LMS M3 estimado 100h de vídeo inicial → ~$180 one-time + ~$30/mês marginal. Aceitável.

**Ação**: atualizar CNT-015 priority M→S e escrever ADR-curta "0033-captions-autogen-mux-lgpd-emag" (delta de 1 página ao ADR-0010).

---

## 7. Banco de Talentos — perceptual hash e anti-duplicata

**Estado**: INV-BT-01 (≤90s), INV-BT-02 (lock 90d), INV-BT-05 (1 currículo/morador), INV-BT-07 (search log append-only).

**Research YouTube Content ID**: fingerprinting áudio+vídeo custa $100M. Não aplicar.

**Gap potencial**: morador A publica vídeo com voz/rosto do morador B sem consentimento (violação LGPD imagem). Ou morador publica vídeo genérico baixado do YouTube/TikTok (fraude).

**Proposta**:
- **M1/M2**: não implementar pHash. INV-BT-05 + lock 90d + moderação manual (CNT-003/ADR-0024 cascade) + denúncia do usuário + admin D-058 cobrem.
- **Gatilho M3+**: ≥3 casos confirmados de fraude por trimestre → avaliar AudD ($10/mês low volume) pra fingerprint áudio + Rekognition Face Match (opt-in) cross-check com selfie de cadastro.
- **Sem** tabela `curriculum_video_fingerprints` agora — YAGNI.

**Lock 90d aplicado ao BT** (já existe): é análogo correto ao "don't republish quickly" do YouTube, com justificativa extra de anti-fraude (morador não cria/apaga/recria com discurso contraditório — BT §16). **Manter**.

---

## 8. Thumbnails e preview em UI

**Estado**: CNT-032 (thumbnail auto Mux) + `GetThumbnailURL` implementado em `mux_provider.go`. Storyboard Mux disponível, user pode override (04 §1.4).

**Research**: YouTube treina modelo pra selecionar 3 candidatos + testa CTR. Não aplicar (sem ML-ops).

**Gap UX**: preciso confirmar se o frontend (web + mobile) mostra `thumbnail_url` consistentemente em:
- LMS4 detalhe curso (06) — card de aula tem thumb?
- BT empresa-view — card de morador tem poster do vídeo-currículo?
- 04 catálogo — card de vídeo institucional?

**Ação**: **A-### novo em AUDIT** — pentester/QA passo: "thumbnail aparece em todos os cards que referenciam `video_id`? storyboard é usado em seek UI do player?".

---

## 9. Anti-padrões explícitos (do destilado §9) aplicados ao MS

| Anti-padrão YouTube-style | Risco no MS hoje | Mitigação |
|---|---|---|
| Operar CDN própria | Nenhum (Mux cobre) | — |
| Transcoding custom FFmpeg | Nenhum | — |
| Fingerprint copyright próprio | Nenhum | Cláusula TOS + ADR-0024 |
| DNN recommendation MVP | ADR-0023 já barra | — |
| hls.js custom | Nenhum (Mux Player via `IVideoProvider`) | Validar que `03-subprojects/web/` + `03-subprojects/mobile/` não vazaram hls.js próprio |
| Autopublish sem moderação | **Risco real**: CNT-022 LMS Live admin-only; mas CNT-001 vídeo empresa tem `status=pending_moderation`? Confirmar | Reforçar: lock 90d + moderação pré-publish obrigatória (INV-LMS-C/RC-7 gap G-14 em 06) |
| Feed "freshness" estilo YouTube | Nenhum — modelo MS é "ata" imutável | Lock 90d é diferenciador, não anti-padrão |

---

## 10. Gaps/ações derivados desta análise

| # | Gap | Sub-produto | Proposta |
|---|---|---|---|
| G-YT-01 | Endpoint `/me/moderation-decisions` pra appeal LGPD Art. 20 não encontrado em CNT-* | 04/06/07 | Adicionar CNT- novo + ADR-0024 refinement |
| G-YT-02 | CNT-015 captions C→S; default ON em LMS/BT | 06/07 | ADR-0033 curto + edit CNT-015 |
| G-YT-03 | CNT-022 LMS Live provider pendente — propor Mux Live | 06 | Fechar CNT-022 + ADR-curto "0034-mux-live-lms" |
| G-YT-04 | INV-LMS-C / RC-7 lock 90d LMS curso certificado — gap G-14 de 06-lms-academia ainda aberto | 06 | Adicionar BR- em `business-rules.md` + método `Course.Lock()` + reforçar via playbook |
| G-YT-05 | Unified moderation lib (ADR-0024) não lista `curriculum_videos.transcript` nem `course_lessons.transcript` como consumers | 06/07 | Edit ADR-0024 §"Unified content filtering" |
| G-YT-06 | DLQ webhook Mux + idempotência `mux_webhook_id` — verificar implementação em `content/infrastructure/http/routes/` | 04/06/07 | QA pentester task |
| G-YT-07 | Thumbnails + storyboard consumo em UI (web+mobile) — confirmar em todos cards | 04/06/07 | A-### em AUDIT |
| G-YT-08 | QoE dashboard Mux Data (startup p95 <2s, rebuffer <1%) — existe instrumentação | 04/06/07 | M2 task SRE |
| G-YT-09 | Bitrate 360p cap ≤ 800 kbps pra 3G BR — não confirmado em ADR | 04/06/07 | Auditar Mux config |
| G-YT-10 | pHash vídeo-currículo — gatilho M3+ | 07 | Monitorar fraude; não implementar agora |

---

## 11. Ordem recomendada (M1 2026-05-08 focus)

1. **Nada novo em M1** — CNT-003 manual + CNT-004 lock 90d + CNT-010 Mux Player bastam.
2. **M2**: Fechar **G-YT-02 (captions autogen)** + **G-YT-03 (Mux Live LMS)** + **G-YT-04 (lock LMS curso)** + **G-YT-06 (DLQ webhook)** em 4 ADRs curtos. Executar ADR-0024 Fase 2 cascade quando gatilho disparar.
3. **M3+**: Revisar DRM se hotlink >1% (§5). Perceptual hash BT só se fraude emergir.

---

## Referências

- [[_destilado]] — síntese 8 fontes
- [[../../00-product/sub-produtos/04-conteudo-videos]]
- [[../../00-product/sub-produtos/05-assembleia-live]]
- [[../../00-product/sub-produtos/06-lms-academia]]
- [[../../00-product/sub-produtos/07-banco-de-talentos]]
- [[../../04-requirements/functional/content]]
- [[../../02-architecture/adr/0010-mux-video-provider]]
- [[../../02-architecture/adr/0011-livekit-sfu]]
- [[../../02-architecture/adr/0024-moderation-cascade]]
- [[../../02-architecture/patterns]] §11, §14 (idempotency + outbox)
- [[../../STATE]] D-056, D-058, D-062, GR-029
