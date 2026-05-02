---
title: ADR-0044 — Broadcast Live Provider (Cloudflare Stream vs Mux Live)
type: adr
tags: [adr, content, broadcast, live, video, master-sindico]
status: proposed (pending dual-check)
date: 2026-04-25
created: 2026-04-25
updated: 2026-04-25
deciders: [@vinni]
consulted: []
informed: []
supersedes: []
superseded-by: []
---

# ADR-0044 — Broadcast Live Provider

## Status

**proposed (pending dual-check)** — 2026-04-25

> ⚠️ Decisão **não-implementada**. Requer dual-check formal (STEERING §32) antes de adoção. ADR criada como follow-up de [[../../STATE#D-133]] que separou conceitualmente Live (Content BC) ≠ Assembleia (Assembly BC).

## Contexto

[[../../STATE#D-133]] (2026-04-25) corrigiu confusão conceitual no canônico:
- **Live** = broadcast 1→N (vídeos ao vivo de empresas/marketing) → produto do `content` BC
- **Assembleia** = conferência N↔N tipo Meet → produto do `assembly` BC, usa Livekit (já implementado)

Como consequência, surge a necessidade de um **provider de broadcast** dedicado para Lives institucionais — diferente do Livekit (que é otimizado para conferência interativa, não broadcast em larga escala).

[[../../STATE#D-097]] (Lives LMS Empresa Pro) já estabelece o caso de uso: Empresas Pro + Admin Master Síndico podem criar Lives institucionais para audiências grandes (centenas a milhares de viewers concorrentes). Mux já é provider canônico de **VOD** ([[0010-mux-video-provider]]). [[../../STATE#D-117]] (Cloudflare edge primário) introduz Cloudflare Stream como candidato natural.

### Características de Lives broadcast (≠ Assembly)

| Característica | Lives broadcast (Content BC) | Assembleia (Assembly BC) |
|---|---|---|
| Direção | 1 → N | N ↔ N |
| Concorrência típica | 100-10.000 viewers | 10-200 participantes |
| Interatividade | chat texto, reações | voz/vídeo/votação |
| Gravação | obrigatória (DVR) | opcional |
| Latência aceitável | 5-30s | < 200ms (real-time) |
| Provider | **decisão pendente** (este ADR) | Livekit (já adotado) |

## Decisão (proposta)

Adotar **`IBroadcastProvider`** como interface canônica do `content` BC, com **um adapter implementado em produção** (não mock, não stub — vital para o produto desde lançamento, alinhado com filosofia de [[../../STATE#D-120]] sobre `ISearchProvider`).

Candidatos (a fechar via dual-check):

### Candidato A — Cloudflare Stream
- **Pro**: alinha com [[../../STATE#D-117]] (Cloudflare edge primário); zero-egress entre CF Pages/CDN/Stream; preço previsível ($1/1.000 minutos delivered + $5/1.000 minutos stored); Adaptive Bitrate (ABR) automático; sub-second latency disponível; Player web embed pronto; SRT/RTMP ingest.
- **Contra**: ecossistema mais novo; menos integrações third-party; SDK menos maduro vs Mux.
- **Stack atual**: Cloudflare já é DPA assinado pré-M1 ([[../../STATE#D-117]]).

### Candidato B — Mux Live
- **Pro**: mesmo provider de VOD ([[0010-mux-video-provider]]) — dev experience consistente; Mux Data analytics integrado; SDKs maduros (web/iOS/Android); Live → VOD automático (gravação vira asset VOD na mesma conta).
- **Contra**: custo maior ($0.040/minuto encoded + $0.0010/minuto delivered) — pode escalar 2-3× CF Stream em volume alto; mais um vendor para LGPD/DPA (já há Mux DPA assinado pra VOD).
- **Stack atual**: Mux já é provider canônico VOD; reusar facilita ops.

### Candidato C — Hybrid
- Cloudflare Stream para audiência large-scale (>1000 viewers concorrentes)
- Mux Live para audiência small-scale com analytics rico (Empresa Pro com ≤100 viewers)
- **Contra**: 2 vendors, dobra esforço LGPD/DPA/onboarding

## Modelagem (independente do candidato)

```go
// internal/modules/content/domain/providers/i_broadcast_provider.go

type IBroadcastProvider interface {
    // CreateLiveStream cria stream key + ingest URL para o Empresa publicar.
    CreateLiveStream(ctx context.Context, opts CreateLiveOpts) (*LiveStream, error)
    
    // GetPlaybackURL retorna URL pública para embed (HLS).
    GetPlaybackURL(ctx context.Context, streamID string) (string, error)
    
    // EndLiveStream encerra stream + dispara gravação para VOD.
    EndLiveStream(ctx context.Context, streamID string) error
    
    // GetRecordingURL retorna gravação (após Live ended).
    GetRecordingURL(ctx context.Context, streamID string) (string, error)
}

type CreateLiveOpts struct {
    CompanyID    CompanyID  // tenant kind=company (D-126)
    Title        string
    StartsAt     time.Time
    MaxViewers   int        // limite por plano (Pro: 1000; Enterprise: 10k)
    DVREnabled   bool       // gravação automática
}
```

## Aggregate novo (em `content` BC)

`BroadcastLive` aggregate root:

```go
type BroadcastLive struct {
    id              BroadcastLiveID
    companyID       CompanyID         // tenant
    streamID        string            // do provider
    status          BroadcastStatus   // scheduled | live | ended | archived
    title           string
    scheduledAt     time.Time
    startedAt       *time.Time
    endedAt         *time.Time
    viewersPeak     int               // métrica
    recordingURL    *string           // após ended
    visibilityScope BroadcastScope    // public | empresa-only | empresa+sindicos-vinculados
    // ...
}

type BroadcastStatus string
const (
    Scheduled BroadcastStatus = "scheduled"
    Live      BroadcastStatus = "live"
    Ended     BroadcastStatus = "ended"
    Archived  BroadcastStatus = "archived"
)
```

**Distinto de `LiveSession`** (do `assembly` BC) — `LiveSession` modela sessão Livekit de assembleia.

## Consequências

### Positivas (quaisquer dos 3 candidatos)
- Separa BC corretamente: Lives broadcast em `content`, Assembly em `assembly`
- Permite escala broadcast (10k+ viewers) sem sobrecarregar Livekit
- Aggregate dedicado em `content` mantém boundary clean

### Negativas
- Custo adicional vs Livekit-only (Livekit não suporta broadcast 10k bem)
- Dependência de provider externo adicional
- DPA + LGPD compliance para mais 1 vendor (se Mux Live novo, ou Cloudflare se Stream)

## Decisão pendente — items para dual-check

1. **Volume estimado M1-M3**: quantos Lives/mês × viewers por live? (afeta custo)
2. **Latência aceitável**: 5s OK ou precisa <2s? (CF Stream sub-second extra cost)
3. **Analytics**: Mux Data é diferencial real ou cobertura via Grafana basta?
4. **Vendor lock-in**: priorizar reusar Mux (já contratado) ou Cloudflare (alinhamento estratégico)?

## Alternativas rejeitadas

1. **AWS IVS** — alinhamento com AWS é M4+ ([[0017-railway-initial-aws-m4]]); fora de escopo M1-M3.
2. **Twitch / YouTube Live como provider terceiro** — perda de controle UX, branding, dados; rejeitado.
3. **Self-hosted SRT/HLS** — overkill operacional; M5+ se escala extrema justificar.
4. **Estender Livekit para broadcast** — Livekit é otimizado para conferência interativa SFU; broadcast 10k+ é caso fora do design.

## Referências

- [[../../STATE#D-133]] — Live ≠ Assembleia (correção conceitual, criação deste ADR)
- [[../../STATE#D-097]] — Lives LMS Empresa Pro (caso de uso primário)
- [[../../STATE#D-117]] — Cloudflare edge primário (favorece Cloudflare Stream)
- [[../../STATE#D-120]] — ISearchProvider real desde M1 (precedente de provider real, não mockado)
- [[0010-mux-video-provider]] — Mux como provider VOD canônico
- [[../../00-product/sub-produtos/05-assembleia-live]] — renomear pra `05-assembleias-votacoes` (D-133)
- [[../../01-domain/aggregates/BroadcastLive|sub-produto 13 lives-broadcast (D-133)]] — criar (ou seção em sub-produto 4 Conteúdo)
- Cloudflare Stream pricing: https://developers.cloudflare.com/stream/pricing/
- Mux Live pricing: https://www.mux.com/pricing
- Comparativo broadcast providers (a pesquisar via dual-check)
