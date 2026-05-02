---
title: Destilado — WebRTC & Arquitetura de Conferência (Google Meet, LiveKit, Jitsi, Zoom, Daily)
type: research-destilado
tags: [research, master-sindico, v2, webrtc, livekit, sfu, assembleia-live, google-meet]
created: 2026-04-23
updated: 2026-04-23
status: draft
sources: 10
aplicacao: master-sindico/assembleias-live
---

# Destilado — WebRTC & Arquitetura de Conferência aplicado ao Master Síndico

> Contexto: sub-produto **Assembleias Live** do Master Síndico já usa **LiveKit** (SFU). Este destilado confirma os padrões, dimensiona pra realidade condominial BR (tipicamente 50–500 participantes por assembleia) e resolve as decisões abertas: votação RT, fila de fala, gravação pra ata, E2EE e TURN.

## TL;DR (o que decidir)

1. **Manter LiveKit como SFU** — dimensionamento confirmado pra 100–500 participantes numa mesma assembleia (limite folgado: um único nó roteia centenas de streams; LiveKit Cloud vai a 100k por sessão). Ver [[source-02]], [[source-04]].
2. **Votação RT em `DataStream` reliable do LiveKit** (WebRTC DataChannel via SFU). Não levantar WebSocket paralelo só pra isso — economiza infra e mantém ordem de mensagens dentro da mesma sessão de sinalização. Persistir em Postgres (fonte de verdade pra quórum e ata). Ver [[source-06]].
3. **Fila de fala** = `metadata`/`attributes` por participante + `UpdateParticipant` (quem tem `canPublish=true` fala; moderador alterna). Não existe "raise hand" built-in — é pattern app-level. Ver [[source-07]].
4. **Gravação pra ata BR = LiveKit Egress (Room Composite → MP4 em S3)**. Sem Mux; Egress já entrega o que precisamos e não adiciona CDN que assembleia privada não usa. Ver [[source-08]].
5. **E2EE (Insertable Streams) = overkill e contraproducente pra assembleia condominial**. Quebra gravação server-side (que precisamos pra ata). Manter SRTP + TLS + isolamento por tenant. Ver [[source-10]].
6. **TURN = delegar ao LiveKit Cloud** em M1; operar próprio só em M3+ se custo de egress justificar. Ver [[source-09]].

---

## 1. Fundamentos WebRTC (base do raciocínio)

Confirmando a stack que LiveKit abstrai e pode vazar em troubleshooting ([[source-01]]):

- **ICE** orquestra descoberta de conectividade entre peers (candidatos: host, srflx via STUN, relay via TURN).
- **STUN** descobre IP público do cliente atrás de NAT.
- **TURN** relaya mídia quando NAT simétrico/firewall corporativo bloqueia P2P direto. **Custa banda e é o item mais caro em egress**.
- **SDP** é o formato (não protocolo) que descreve codec, resolução, crypto.
- **DTLS+SRTP** criptografam media (transport-layer). É o "E2E" de fato que Meet/LiveKit entregam por padrão.

**Aplicado**: síndicos e moradores entram de redes heterogêneas (celular 4G, Wi-Fi doméstico, rede corporativa). Planejar **fallback TURN/TLS na :443** é obrigatório — rede de escritório síndico frequentemente bloqueia UDP. [[source-01]], [[source-09]].

## 2. SFU vs MCU vs P2P — decisão reconfirmada

| Topologia | Uso server | Upload cliente | Escalabilidade | Casa com assembleia? |
|---|---|---|---|---|
| P2P (mesh) | ~0 | O(N) cresce linear | 2–4 | Não (cai com 10+) |
| SFU | Roteia pacotes, sem transcode | O(1) | 5–100+ (até 10k com cascade) | **Sim** |
| MCU | Mixa e re-encoda (alto CPU) | O(1) | Baixa, cara | Não (custo explode) |

LiveKit é SFU confirmado ([[source-02]], [[source-04]]). Zoom ([[source-05]]) e Google Meet ([[source-03]]) também são SFU em grupo. Jitsi ([[source-04]]) é SFU. **Não há trade-off a revisitar em M1**. [[source-00-overview]] em [[source-01]].

**Dimensionamento assembleia condominial BR**:

- Típico condomínio vertical: 80–250 unidades; assembleia com quórum 25% = ~20–60 participantes ativos.
- Edge case (grande cond./intercondominial): 100 ativos + 400 passivos (só áudio + slides).
- **LiveKit um único nó**: "hundreds of simultaneous video streams" em hardware moderno ([[source-02]]). Folgado pra nosso pico realista.
- Caminho pra 1k+ simultâneos (ex: live de prestação de contas com vários cond. do cliente): LiveKit Cloud escala horizontal automático ([[source-04]]), ou self-host com Redis.

## 3. Padrões de codec e SVC (por que importa)

Google Meet ([[source-03]]):
- **VP9 com SVC (L3T3_KEY)** em webcam 2+ pessoas. SVC = 1 stream codificado em múltiplas camadas espaciais/temporais; SFU escolhe camada por subscriber.
- **VP8 simulcast** em screen-share (RTCPeerConnection separado).
- **AV1** experimental só em pré-call / bandwidth estimation (encoder custo-alto ainda).
- VP9 SVC reduz upload em 40–60% vs VP8 simulcast em chamadas multi-party.

**Aplicado**: LiveKit suporta simulcast e SVC. Para assembleia com muita gente passiva (câmera desligada; só olhando slides), **forçar AV1/VP9 no screen-share do síndico** é ganho claro. Documentar no playbook de operação.

## 4. Votação em tempo real: DataChannel vs WebSocket externo

Opções técnicas ([[source-06]], [[source-07]]):

- **DataChannel WebRTC (via LiveKit DataStream reliable)**: mesma conexão da mídia, ordem garantida dentro de um stream, NAT-friendly (já atravessou ICE). Limite: mensagens só chegam a quem está online na sala. Não tem persistência.
- **WebSocket externo**: canal paralelo; fonte de verdade do app (nossa API). Serve auditoria, reconciliação, histórico, quem chegou atrasado.

**Decisão recomendada** (híbrida):
- **UI em tempo real (mostrar contagem de votos subindo)** → LiveKit DataStream reliable com topic `votacao:<id>`.
- **Fonte de verdade (contabilização oficial, ata)** → API REST/WebSocket nossa, gravando em Postgres com timestamp servidor + hash do voto.
- Regra: voto só conta quando persistido; UI é otimista com reconciliação.

Justificativa: **um voto de assembleia é documento legal** (procuração, peso de fração ideal). Não pode depender só de um pacote WebRTC que pode cair. DataStream serve pra UX (ver quórum ao vivo), não pra contabilidade.

## 5. Fila de fala cronometrada — pattern LiveKit

Não tem "raise hand" built-in. Montamos com primitivas expostas ([[source-07]]):

1. **Estado por participante** via `attributes` (map string→string): `{"hand_raised": "true", "queue_position": "3", "requested_at": "2026-04-23T19:32:10Z"}`.
2. **Moderador** (síndico / mesa) tem `roomAdmin` → chama `UpdateParticipant` pra dar `canPublish=true` ao próximo da fila, revoga dos que falaram.
3. **Revogação de `canPublish`** desinscreve tracks automaticamente → efeito "cortar microfone" imediato.
4. **Cronômetro** = timer client-side + enforcement server-side (quando T=0, API nossa chama `UpdateParticipant` revogando publish).
5. **Remote mute** precisa config `enable_remote_unmute: true` se quisermos que moderador *desmute* também (desligado por padrão por UX; habilitar em nosso caso está ok, usuário assinou TOS da assembleia).

## 6. Gravação pra ata — decisão LiveKit Egress (não Mux)

Comparação ([[source-08]], [[source-10]]):

| Opção | Prós | Contras | Custo indicativo |
|---|---|---|---|
| **LiveKit Egress Room Composite → MP4 S3** | Mesma plataforma, layout do grid ou speaker, templates HTML custom, Jibri-like mas já integrado | Precisa processo Chrome server-side (headless Chromium) — custo CPU | Transcode vídeo $0.02/min overage |
| Mux Real-Time + record | Player pronto, CDN global | Dois vendors; complexidade sinc | Input + storage + delivery (3 dimensões) |
| Jibri (Jitsi) | OSS | Stack alheia — requeriria migrar SFU | Self-host |
| Track Egress (raw) + muxar depois | Mais barato ($0.001/min) | Pós-processamento externo | Bom pra áudio-only de conferências pequenas |

**Decisão M1**: **Room Composite → MP4 direto S3**. Assembleia BR não precisa CDN (é privada, tenant-isolated). Player do Master Síndico usa HTML5 `<video>` com stream S3/CloudFront assinado.

**Requisito legal BR (Código Civil art. 1.354 + convenção)**: ata vale com gravação anexada; não precisa ser vídeo com qualidade broadcast, só identificar votos e manifestações. MP3 em paralelo (Track Egress áudio-only) é redundância barata.

## 7. E2EE — overkill pra nosso caso

Análise ([[source-10]]):

- **E2EE via Insertable Streams** cifra frames antes de sair do encoder; SFU só vê bytes opacos.
- Overhead: ~25% em áudio, <1% em vídeo.
- **Quebra gravação server-side** — SFU não pode entregar frames decifrados pro Egress. Pra gravar precisaria cliente-side recording, que é frágil.
- Só Chromium-based (Firefox parcial, Safari limitado).

**Veredicto**: Master Síndico **não ativa E2EE**. Argumentos:
1. Gravação é requisito legal da ata; E2EE bloqueia.
2. Privacidade já garantida por **isolamento de tenant** (sala = condomínio; só moradores daquele CNPJ entram; RLS Postgres na camada de dados).
3. Ameaça de modelo: não estamos defendendo contra operador hostil do SFU (LiveKit Cloud é parceiro contratual + DPA LGPD). Defendemos contra interceptação de rede — SRTP/DTLS já resolve.

Documentar explicitamente no `ADR-assembleias-live-e2ee-off.md`.

## 8. TURN — quem opera?

Opções ([[source-09]]):

- **LiveKit Cloud**: TURN incluso, global, TLS:443. Zero operação. **Escolha M1**.
- **Self-host LiveKit + TURN próprio** (coturn ou LiveKit com config TURN): controle total, precisa cert SSL dedicado em subdomínio turn.*, monitoração de banda, 10Gbps NIC recomendado pra produção.
- **STUNner em K8s**: pattern avançado (ingress WebRTC nativo no cluster); explorar só quando custo egress LiveKit Cloud > 30% infra total.

**Gatilho pra mudar**: quando um condomínio sozinho passar de ~200h/mês de assembleia/reunião gravada com mais de 30 participantes ativos. Antes disso, Cloud é mais barato que o custo do nosso time operar coturn.

## 9. Sinalização

LiveKit abstrai ([[source-02]], [[source-04]]):

- Signaling próprio via WebSocket → servidor (quem ingressa, quem publica track, SDP offer/answer, ICE candidates, Data messages reliable).
- Multi-node: Redis como pub-sub pra roteamento de participantes pro mesmo nó.

**Aplicado**: nossa API (FastAPI/Node) não implementa sinalização WebRTC. Só emite **JWT de acesso** pra sala LiveKit (`room_join`, `canPublish` condicional, metadata do usuário: `cpf_mask, unidade, fração_ideal`).

## 10. Roadmap de escalabilidade (10 → 10k)

Inspirado em [[source-04]] (LiveKit distributed mesh) e [[source-04]]/[[source-05]] (Jitsi Octo, Zoom MMR):

| Fase | Participantes típicos | Arquitetura | Infra |
|---|---|---|---|
| M1 (2026-05-08, 1º cliente) | 10–50 por sala, ~5 salas simultâneas | LiveKit Cloud single-region (São Paulo) | nenhuma infra própria WebRTC |
| M2 | 50–200/sala, 20 salas | LiveKit Cloud multi-region BR+US | monitorar billing |
| M3 | 200–500/sala (intercondominial) | LiveKit Cloud ou self-host K8s + Redis + STUNner | ops dedicada |
| M4 (visionário) | 1k–10k broadcast (prestação contas pública) | Cascade SFU (modelo Jitsi Octo / LiveKit mesh) + HLS out via Egress pra passivos | RTMP → HLS CDN |

Nota M4: a partir de ~500 passivos só-consumo, **parar de mandar WebRTC e sair via HLS** (latência 10–30s aceitável pra audiência grande, custo muito menor). Egress já suporta via Track Composite → HLS.

---

## Referências (fontes neste destilado)

- [[source-01]] — WebRTC fundamentals (STUN/TURN/ICE/SDP) — MDN
- [[source-02]] — LiveKit SFU internals — docs.livekit.io
- [[source-03]] — Google Meet codecs (VP9 SVC / AV1) — webrtcHacks
- [[source-04]] — LiveKit distributed mesh (100k+ participants) — livekit.com blog + Jitsi Octo — jitsi.support
- [[source-05]] — Zoom architecture (MMR, UDP/TCP/SSL fallback) — Cometchat
- [[source-06]] — LiveKit Data Channels (text/byte/rpc/data tracks) — docs.livekit.io
- [[source-07]] — LiveKit participant management (permissions, mute) — docs.livekit.io
- [[source-08]] — LiveKit Egress (Room/Track Composite, outputs) — docs.livekit.io
- [[source-09]] — LiveKit TURN/ICE production config — docs.livekit.io + L7mp STUNner
- [[source-10]] — E2EE WebRTC Insertable Streams — webrtcHacks

## Vizinhos

- [[_moc]] — MOC desta pasta
- [[../_moc]] — MOC 13-research
- [[../../00-product/adr/|ADRs do produto]] — destino dos ADRs acima (e2ee-off, mux-vs-livekit, voting-channel)
