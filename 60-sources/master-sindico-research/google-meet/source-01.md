---
title: "Source 01 — WebRTC Protocols (ICE, STUN, TURN, SDP) — MDN"
type: research-source
tags: [research, master-sindico, v2, webrtc, fundamentals, ice, stun, turn, sdp]
created: 2026-04-23
updated: 2026-04-23
source_type: oficial
source_org: "Mozilla Developer Network"
source_urls:
  - "https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Protocols"
  - "https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 01 — WebRTC Protocols (MDN)

## Resumo

Documentação canônica da Mozilla sobre a stack de protocolos que constitui WebRTC: ICE (orquestração), STUN (descoberta NAT), TURN (relay fallback), SDP (descrição de sessão) e DTLS/SRTP (segurança). É a referência pra entender o que LiveKit abstrai por baixo.

## Pontos-chave

- **ICE** é framework: combina candidatos host (IP local), srflx (descoberto por STUN) e relay (via TURN), tenta em ordem de prioridade.
- **STUN** descobre mapeamento NAT do cliente; falha com NAT simétrico (frequente em redes corporativas e CGNAT de operadoras mobile BR).
- **TURN** é último recurso: relaya todo tráfego via servidor. Custa banda (ingress + egress). Precisa TLS na :443 pra atravessar firewalls corporativos.
- **SDP** é formato texto (UTF-8) com m-lines descrevendo codecs, resolução, criptografia. Não é protocolo de transporte; é metadata que o sinalizador troca.
- **DTLS** faz handshake e chaveamento; **SRTP** cifra RTP (mídia). Isso é o que garante confidencialidade no "fio" mesmo sem E2EE explícito.

## Aplicação ao Master Síndico

- Planejar obrigatório fallback TURN/TLS:443 desde M1. Síndicos acessam de redes de escritório que bloqueiam UDP.
- Em BR, CGNAT mobile (Claro, Vivo, TIM) é endêmico → STUN sozinho falha com frequência → TURN vira caminho comum (custo).
- Não expor detalhes SDP no app; LiveKit SDK esconde. Mas logar eventos ICE no client SDK pra diagnosticar conexões falhando.

## Citações relevantes

> "ICE is a framework that allows your web browser to connect with peers. [...] ICE uses STUN and/or TURN servers to accomplish this, or does so directly if your peer is on the same local network."

> "TURN is meant to bypass the Symmetric NAT restriction by opening a connection with a TURN server and relaying all information through that server."

## URL(s)

- https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Protocols
- https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling
