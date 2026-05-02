---
title: "Source 09 — LiveKit TURN/ICE Production Configuration + STUNner"
type: research-source
tags: [research, master-sindico, v2, webrtc, livekit, turn, ice, firewall, producao]
created: 2026-04-23
updated: 2026-04-23
source_type: oficial
source_org: "LiveKit, L7mp"
source_urls:
  - "https://docs.livekit.io/transport/self-hosting/deployment/"
  - "https://github.com/livekit/livekit/blob/master/config-sample.yaml"
  - "https://kb.livekit.io/articles/1724892785-establishing-media-connection-firewall-troubleshooting"
  - "https://docs.l7mp.io/en/latest/examples/livekit/"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 09 — TURN/ICE em produção LiveKit

## Resumo

Guia de deploy self-host LiveKit com foco em TURN e traversal de firewall. Introduz STUNner (L7mp) como pattern Kubernetes-native pra WebRTC ingress.

## Pontos-chave

### TURN config LiveKit

- Suporte nativo embutido no server: config YAML com `turn.enabled`, `turn.tls_port`, `turn.udp_port`, `turn.domain`.
- **TURN/TLS na :443** dá cobertura máxima (redes corporativas bloqueiam tudo menos 80/443 TCP).
- Sem LB: `turn.tls_port=443` direto. Com LB: TLS termination no LB, LiveKit escuta alt port.
- Precisa cert SSL válido (Let's Encrypt serve) em subdomínio dedicado (`turn.masterssindico.com.br`).

### STUN inicialização

- LiveKit nó envia STUN Binding Request em startup → descobre IP público → cacheia → anuncia como ICE candidate.
- Em JoinResponse o cliente recebe lista STUN/TURN servers → browser gathering candidates → troca via signaling → ICE connectivity checks.

### Produção recomendada

- **NIC 10Gbps+** pra nó com 500+ streams simultâneos.
- Domínio TURN separado + cert dedicado.
- Monitorar `livekit_ice_connection_*` metrics.

### STUNner (avançado, K8s-native)

- Gateway WebRTC pra Kubernetes que faz TURN ingress direto pros pods do SFU.
- Elimina LB tradicional pra media (WebRTC não faz bem com L4/L7 LB).
- Pattern recomendado se self-host em EKS/GKE.

## Aplicação ao Master Síndico

### M1 (cliente 1, maio/2026)

- **LiveKit Cloud — delegar TURN totalmente**. Zero config nossa. Cloud expõe edge global com TURN/TLS:443.
- Custo diluído em participant-minutes ($0.0005/min WebRTC).

### Quando migrar pra TURN próprio

Gatilhos (qualquer um):
1. Custo egress LiveKit Cloud > $500/mês por 3 meses consecutivos.
2. Requisito contratual de soberania de dados (cliente enterprise exige servidor BR específico).
3. Volume > 3TB/mês egress (onde self-host egress sai mais barato).

### Arquitetura self-host quando escalar

- K8s (AWS EKS São Paulo) com LiveKit deployment + Redis (usamos já) + STUNner Gateway.
- TURN próprio coturn se precisar *só* TURN sem SFU (improvável; LiveKit já embute).
- Certs automatizados via cert-manager + Let's Encrypt.

### Troubleshooting BR-específico

- CGNAT mobile (Claro/Vivo/TIM) vira TURN na maior parte das conexões — orçar banda.
- Firewalls corporativos de síndicos profissionais (admin prediais) frequentemente só liberam 443 → TURN/TLS obrigatório.

## Citações relevantes

> "Enabling TURN/TLS gives you the broadest coverage in client connectivity, including those behind corporate firewalls."

> "If you are not using a load balancer, turn.tls_port needs to be set to 443, as that will be the port that's advertised to clients."

## URL(s)

- https://docs.livekit.io/transport/self-hosting/deployment/
- https://github.com/livekit/livekit/blob/master/config-sample.yaml
- https://kb.livekit.io/articles/1724892785-establishing-media-connection-firewall-troubleshooting
- https://docs.l7mp.io/en/latest/examples/livekit/
