---
title: Aplicações Master Síndico — Google Meet / WebRTC research → Assembleia Live
type: note
tags: [research, applications, master-sindico, v2, assembleia-live, webrtc, livekit, google-meet]
source: 13-research/google-meet/_destilado.md
created: 2026-04-23
updated: 2026-04-23
aliases: [aplicacoes-google-meet, assembleia-live-cruzamento]
---

# Aplicações — Google Meet / WebRTC → Assembleia Live (M3)

Cruza [[_destilado]] (SFU, E2EE, SVC, DataChannel, Egress, TURN) com o sub-produto **05-assembleia-live** (já usa LiveKit — [[../../02-architecture/adr/0011-livekit-sfu]]). Foco: o que do research já está decidido, o que muda específico pro condomínio BR, o que vira invariante.

> Contexto: ASM-019 fila fala, ASM-014 votação RT, ASM-020 streaming, ASM-024 ata automática, ASM-022 modo contingência. Invariantes: [[../../01-domain/invariants#INV-071]] voto único, [[../../01-domain/invariants#INV-072]] ata imutável, [[../../01-domain/invariants#INV-073]] quórum fracional, [[../../01-domain/invariants#INV-084]] saga compensável.

---

## Top-5 aplicações (prioridade M3 2026-07-20)

### 1. SFU + escala condominial BR dimensionada ([[_destilado]] §2)

Assembleia típica BR: 80–250 unidades, quórum 25% = **20–60 ativos**; edge intercondominial: 100 ativos + 400 passivos. LiveKit single-node roteia centenas de streams — **folgado**. MCU rejeitado (custo explode com transcode), P2P rejeitado (cai em 10+). **Decisão confirmada**: LiveKit Cloud SP single-region M3; cascade mesh só M4+ se prestação de contas pública >500 passivos → nesse ponto, sair de WebRTC e emitir HLS via Track Composite (latência 10-30s aceitável pra audiência broadcast). Não revisitar em M3.

### 2. Votação RT híbrida: DataStream (UX) + POST HTTP (fonte de verdade) ([[_destilado]] §4)

**Invariante legal**: voto de assembleia é documento (procuração, fração ideal, [[../../01-domain/invariants#INV-071]] UNIQUE DB). **Não** persistir só via DataChannel — pode cair. **Padrão**: UI mostra contagem subindo via LiveKit DataStream reliable `topic=votacao:<id>`; contabilização oficial via `POST /assemblies/{id}/voting/{vid}/vote` que grava Postgres com timestamp servidor + hash. UI é **otimista com reconciliação**; voto só conta quando persistido. Resolve ASM-022 modo contingência (Redis offline queue + sync). E2EE **off** ([[_destilado]] §7): quebraria Egress gravação + nosso threat model defende contra rede, não contra SFU parceiro (DPA LGPD LiveKit).

### 3. Egress Room Composite → MP4 S3 como ata imutável ([[_destilado]] §6)

Código Civil art. 1.354 + convenção: ata vale com gravação anexada. **Decisão**: Room Composite (grid layout) → MP4 S3 + Track Egress áudio-only MP3 paralelo (redundância barata, ~$0.001/min). Mux rejeitado (dois vendors, sem CDN precisa pra privado). **Invariante proposto (novo)**: `INV-ASM-REC-001` — toda assembleia online homologada exige `recording_s3_key NOT NULL` antes de `homologated_at` (bloqueio ASM-027). Chromium headless custa CPU — aceitável no budget M3. Player HTML5 com signed URL S3/CloudFront (TTL 24h, alinhado INV-070).

### 4. Quórum fracional ao vivo + desconexão → ausência 15min ([[_destilado]] §5 + ASM-016)

LiveKit emite `ParticipantDisconnected` em DataStream. Handler atualiza `assembly_check_ins.disconnected_at`; **recalcula Σ fração ideal presente** ([[../../01-domain/invariants#INV-073]]) via `QuorumCalculator.IsReached(snapshot, presentes)`. Se morador cai mid-votação: janela **15 min de reconexão** (ASM-016); voto já persistido permanece (INV-071 imutável); procuração ativa **não** reassume se titular já votou (ASM-010 INV-077). Voto novo só se ainda não votou E reconectou dentro da janela — caso contrário, abstenção automática. Painel Presidente (ASM-017) mostra alerta "quórum abaixo de threshold" em WebSocket < 200ms.

### 5. Degradação gráfica BR: SVC + áudio-only fallback + Krisp opcional ([[_destilado]] §3)

Moradores em 3G/4G domiciliar → **forçar VP9 SVC no screen-share do síndico** (40-60% menos upload vs VP8 simulcast). LiveKit bandwidth estimation já desce camada espacial/temporal automático. **Fallback áudio-only**: se SVC camada mínima ainda não entrega, cliente desativa vídeo inbound mantém áudio + DataStream voto — documentar no playbook operação. Krisp noise suppression (LiveKit plugin): **postergar pra M4** — custo adicional, não crítico (assembleia tem ruído aceitável; ata via Egress captura texto via transcrição posterior, não via ML ao vivo). Captions ao vivo / ata-automática via transcrição: **M4+** (ASM-024 segue template HTML+síndico humano em M3).

---

## Decisões não prioritárias (parking lot)

- **MFA pra votar** ([[../../08-security/BEYOND_CORP]] §4.3 + INV-110): step-up obrigatório em `vote.cast` só M2+ via Play Integrity (Android) / DeviceCheck (iOS). M1 MVP: device fingerprint + 1-device policy suficiente (INV-005 + A-023 audit log).
- **Votação secreta**: não é requisito atual (assembleia BR pede voto identificado por unidade — fração ideal rastreável). Se emergir em tenant enterprise → E2EE por track específica, com trade-off: quebra auditoria de ata. Não mexer M3.
- **Captions live → ata automática**: economiza secretário humano, mas Whisper/Gladia custam $$ e nosso M3 MVP aceita ata manual + gravação anexa. Reavaliar M4.
- **Cascade SFU / HLS broadcast**: só disparar se condomínio pedir "live aberta ao público" >500 passivos. Não é caso master-sindico condomínio clássico (privado por CNPJ/RLS).

---

## Invariantes propostos (candidatos a ticket INV pós-ADR)

1. **INV-ASM-REC-001**: assembleia online homologada exige `recording_s3_key NOT NULL` + checksum (enforcement: ASM-027 handler + Saga compensável INV-084).
2. **INV-ASM-VOTE-RT-001**: voto via DataStream é otimista; único PERMIT oficial é `INSERT INTO assembly_votes` com UNIQUE (INV-071). DataStream packet sem persist commit = display only.
3. **INV-ASM-QUORUM-001**: quórum recalculado a cada `ParticipantConnected/Disconnected` event; cache TTL 0 (live); persistido em `assembly_quorum_log` append-only.
4. **INV-ASM-REC-002**: E2EE **proibido** em assembleia — `config.Validate()` falha startup se `LIVEKIT_E2EE_ENABLED=true` (quebra Egress → quebra ata → quebra lei).

> Pendências: abrir ADR-0033 "Recording obrigatório assembleia online" + ADR-0034 "E2EE off formal policy". Ambos pós-dual-check pré-M3.

---

## Links

- [[_destilado]] — fonte canônica (TL;DR + §2/§4/§6/§7)
- [[_moc]]
- [[../../00-product/sub-produtos/05-assembleia-live]]
- [[../../04-requirements/functional/assembly]] (ASM-014, ASM-016, ASM-019, ASM-020, ASM-022, ASM-024, ASM-027)
- [[../../02-architecture/adr/0011-livekit-sfu]]
- [[../../02-architecture/research-inspirations]] §1.17, §1.18
- [[../../01-domain/invariants#INV-071]] voto único
- [[../../01-domain/invariants#INV-072]] ata imutável
- [[../../01-domain/invariants#INV-073]] quórum fracional
- [[../../01-domain/invariants#INV-084]] saga compensável Livekit
- [[../../08-security/BEYOND_CORP]] §4.3 device posture
