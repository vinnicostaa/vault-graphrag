---
title: 'Livekit — Overview (Rooms, Participants, SFU, WebRTC)'
type: note
tags:
  - provider
  - livekit
  - webrtc
  - realtime
  - overview
  - sfu
source: 'https://docs.livekit.io'
source_date: '2026-04-24'
created: '2026-04-24'
updated: '2026-04-24'
---

# Livekit — Overview

Destilação dos conceitos fundamentais do Livekit. Contexto global para decisões de arquitetura realtime (audio/video/data). Para quando usar, ver [[_moc]]; para concorrente em broadcast 1→N, ver [[../mux/_moc]].

> Fonte: https://docs.livekit.io — consultada 2026-04-24.

---

## Modelo de objetos

O Livekit expõe uma topologia simples e composicional:

- **Room** — container lógico de uma sessão. Tem nome único no projeto, metadata opcional, `empty_timeout` (segundos até auto-delete após último participant sair) e `max_participants`. Pode ser **criada manualmente** (via `RoomServiceClient.CreateRoom`) ou **auto-criada** no primeiro `connect` válido.
- **Participant** — usuário conectado a uma Room. Identificado por **identity** (string arbitrária, única dentro da Room). Traz metadata opcional. Tem um conjunto de **tracks**.
- **Track** — um fluxo individual de mídia. Câmera + microfone = 2 tracks separados. Screen share = mais um track. Data tracks carregam payloads arbitrários.
- **LocalParticipant vs RemoteParticipant** — no client SDK, o participant atual (`localParticipant`) publica tracks; os outros (`remoteParticipants`) são observáveis e tem tracks subscribable.

A comunicação entre objetos é reativa: mudanças de estado (mute, track publish, metadata update) geram eventos tanto no client SDK quanto em webhook server-side.

---

## SFU vs MCU — por que Livekit é SFU

Em videoconferência multi-party há três topologias clássicas:

| Topologia | Como funciona | Custo CPU | Custo banda |
|---|---|---|---|
| **Mesh** (P2P direto) | Cada peer manda pros outros N-1 | Baixo servidor, alto client | O(N²) por cliente |
| **MCU** (Multipoint Control Unit) | Servidor decodifica, compõe em uma imagem única, re-encoda | Altíssimo (transcode) | Baixo no client |
| **SFU** (Selective Forwarding Unit) | Servidor recebe e roteia streams sem transcodar | Baixo (só routing) | O(N) no client |

Livekit é **SFU**. Implicações práticas:

- Escala melhor que mesh em rooms com mais de ~4 participants (mesh degrada exponencialmente)
- Cliente decodifica N-1 streams — em mobile/low-end pode saturar (daí **simulcast** e **adaptive stream**)
- CPU servidor baixa → um nó SFU aguenta centenas de participants concorrentes
- MCU seria melhor só em cenários onde o cliente é burro (ex: PSTN bridge, telefonia) — fora disso SFU ganha

Livekit resolve o custo client com **simulcast** (publisher envia múltiplas qualidades simultâneas; SFU encaminha a melhor que caiba na banda do subscriber) e **dynacast** (para de enviar layers não-subscritos).

---

## WebRTC por baixo — ICE, STUN, TURN

Livekit usa WebRTC como transporte. O estabelecimento de conexão passa pelo **ICE** (Interactive Connectivity Establishment):

1. Client descobre candidatos locais (IP da máquina) e reflexivos (IP público via **STUN server**)
2. Candidatos são trocados via signaling (WebSocket ao SFU)
3. Client tenta conexão direta UDP; se firewall bloquear, cai para:
   - ICE sobre UDP (preferencial)
   - TURN sobre UDP (servidor relay)
   - ICE sobre TCP
   - TURN sobre TLS (última opção, atravessa proxy corporativo)

**Implicação de deploy**: em self-host, precisa abrir porta UDP (range 50000-60000 default) e TCP 7881. Se atrás de firewall corporativo onde UDP é bloqueado, TURN sobre TLS (porta 443) salva.

**Signaling** (não mídia): WebSocket na porta 7880. Por aí vão joins, publish/subscribe notifications, metadata updates, disconnect.

---

## Egress — exportar mídia para fora

Egress pega o que está acontecendo numa Room e joga em um destino externo. Tipos:

- **RoomCompositeEgress** — renderiza a Room inteira via Chrome headless + template HTML configurável e gera um único stream composto. Bom para replay tipo Zoom com grid layout.
- **ParticipantEgress** — grava um participant específico (todos os tracks dele) em arquivo ou stream.
- **TrackEgress** — pega 1 track cru, sem transcode. Baixo custo, mas sem composição.
- **TrackCompositeEgress** — sincroniza audio + video de um participant sem renderizar grid.

**Outputs suportados**:
- Arquivo MP4 → S3, GCS, Azure Blob
- HLS segments → S3/CDN
- RTMP(S) → YouTube Live, Twitch, Facebook Live
- WebSocket stream (raw frames)

Ciclo é **assíncrono**: cliente/servidor pede egress → Livekit dispara worker → webhooks `egress_started`, `egress_updated`, `egress_ended` notificam estado. Para gravação, ver [[patterns]] (cleanup + persistência).

---

## Ingress — importar mídia de fora

Caminho inverso: fonte externa entra como participant virtual na Room. Inputs aceitos:

- **Push**: RTMP/RTMPS (OBS, hardware encoder), WHIP (WebRTC-HTTP Ingestion Protocol)
- **Pull**: HTTP (HLS, MP4, MOV, MKV), SRT

Livekit transcoda (GStreamer) para compatibilidade WebRTC e pode publicar com simulcast. Útil para:

- Operador com OBS fazendo broadcast dentro de uma Room interativa
- Playback de vídeo pré-gravado dentro da sala (ex: institucional antes da assembleia)
- Bridge de streams legados (câmeras IP com RTSP → RTMP → Ingress)

---

## Cloud vs self-hosted — como decidir

| Critério | Livekit Cloud | Self-hosted |
|---|---|---|
| Setup | Dashboard + API keys → pronto | Docker/K8s + Redis + TURN + TLS certs |
| Custo | Pay-per-minute por participant | Infra fixa (compute + banda egress) |
| Escala horizontal | Automática | Manual, multi-node com Redis shared |
| SLA/uptime | Garantido (99.9%+) | Você é o SRE |
| Compliance/dados | Dados trafegam pela infra deles | Total soberania (PII, LGPD/HIPAA) |
| Egress/Ingress | Gerenciado | Deploy separado (livekit-egress, livekit-ingress) |

**Regra prática**: começar em Cloud, migrar pra self-host quando (a) custo mensal > custo de 1 SRE ou (b) compliance exige dado local.

---

## Concorrentes — quando cada um vence

| Concorrente | Vence Livekit quando... | Perde quando... |
|---|---|---|
| **Daily.co** | Time pequeno quer SDK ultra-simples, UI kit pronto | Precisa self-host; precisa customizar SFU |
| **Agora** | Cobertura global massiva (China, SEA) é crítica | Custo por minuto importa; quer open-source |
| **100ms** | Interactive audio rooms (Clubhouse-like) com low-level hooks | Quer ecossistema Go/TS oficial robusto |
| **Twilio Video** | Enterprise já em Twilio stack; compliance enterprise | Custo; produto em manutenção (EOL anunciado) |
| **[[../mux/_moc|Mux]]** | Broadcast 1→N pra 10k+ espectadores passivos, HLS+CDN | Qualquer coisa interativa (Mux não é SFU) |
| **Pion (lib Go)** | Quer construir SFU customizado do zero | Não quer reinventar roteamento, recording, auth |

Livekit é o **default pragmático** para realtime interativo N-to-N quando o time quer SFU pronto, SDK oficial em múltiplas linguagens e opção de self-host.

---

## Vizinhos no grafo

- [[_moc]] — visão geral do provider
- [[integration-guide]] — setup + token + client connect
- [[patterns]] — boas práticas operacionais
- [[antipatterns]] — o que não fazer
- [[../_moc]] — MOC de providers
- [[../mux/_moc]] — concorrente em broadcast-scale
