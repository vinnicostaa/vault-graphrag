---
title: "Tailscale — Zero Trust Networking com WireGuard"
type: source
tags: [research, zero-trust, tailscale, wireguard, mesh-vpn, ztna]
source: https://tailscale.com/docs/concepts/zero-trust
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 11 — Tailscale Zero Trust Networking

## Metadata

- **Instituição**: Tailscale Inc.
- **URL zero-trust concept**: https://tailscale.com/docs/concepts/zero-trust
- **URL use case ZT networking**: https://tailscale.com/use-cases/zero-trust-networking
- **URL how it works**: https://tailscale.com/blog/how-tailscale-works
- **URL about WireGuard**: https://tailscale.com/docs/concepts/wireguard
- **Data de acesso**: 2026-04-23
- **Tipo**: documentação de produto

## Resumo

Tailscale é uma **mesh VPN zero-config** baseada em WireGuard. Aplica princípios Zero Trust na camada de rede: cada nó se autentica contra SSO (Google, GitHub, Okta, Entra ID) via coordination server; ACLs baseadas em identidade + tags controlam tráfego peer-to-peer; default deny.

**Arquitetura**:
- **WireGuard** provê criptografia e tunneling point-to-point (handshakes Curve25519, ChaCha20-Poly1305).
- **Coordination server (cloud Tailscale)** resolve key exchange, NAT traversal, ACLs. **Não vê tráfego** (só metadata).
- **Tailnet** = rede lógica de nós autenticados. Cada nó tem `MagicDNS` e IP 100.64.0.0/10.
- **ACLs (JSON/Huon-style)** definem: de `group:eng` pra `tag:prod-db` porta 5432.
- **Tailscale SSH** + **Serverless SSH CA** substitui SSH keys estáticas.
- **Identity providers**: SSO obrigatório, sem contas locais.

Comparado a BeyondCorp: Tailscale é Zero Trust **na camada de rede** (layer 3/4); complementa Access Proxy (layer 7). Ideal pra:
- Acesso admin a bancos de dados de prod.
- Conectar CI runners a recursos privados.
- VPN pra equipe remota com micro-segmentação.

## Trechos-chave

> "Zero trust means that you can't trust the physical network anymore."
> — Tailscale docs, Zero Trust concept

> "[Put] a firewall around every device and every service, and ensure that sessions are always encrypted between every pair of endpoints."
> — Tailscale docs

> "Tailscale enforces identity-based authentication, authorization, and continuous verification directly at the network layer."
> — Tailscale use cases / ZT networking

> "Granular control of every device (...) each protected by its own policies" enabling micro-segmentation.
> — Tailscale use cases

> "By default, all access is denied in Tailscale, so you can write access rules that follow the principle of least privilege, effectively micro-segmenting your network — so that each node is firewalled from and will not accept traffic from any other node unless explicitly allowed."
> — Tailscale blog/docs (síntese)

## Aplicabilidade ao Master Síndico

**Parcial — foco em acesso a infraestrutura interna, não em tráfego de produto.**

Justificativa:

1. **Tailscale não substitui o access proxy de produto** (síndicos não vão instalar Tailscale no celular). Fica fora do caminho do usuário final.

2. **Tailscale é ouro pra acesso admin**:
   - Devs acessando **banco de dados de prod** pra debug — sem bastion host, sem SSH keys estáticas.
   - Runners CI/CD acessando artifacts / registries privados.
   - Time de suporte acessando dashboards internos (Grafana, admin panels) — alternativa a Cloudflare Access.
   - Equipe remota (BR distribuído) com segurança sem VPN tradicional.

3. **Tailscale SSH + SSH CA** — substitui `authorized_keys` por certificados emitidos por SSO. Auditoria automática de quem SSH-ou em que máquina. Grande melhoria sobre VPS com chaves manuais.

4. **ACLs identity-based** são um treino gratuito de pensamento ABAC pra time — policies em JSON declarativo.

5. **Limitação**: Tailscale depende do coordination server (cloud Tailscale, Canadá). Pra dados ultra-sensíveis (compliance estrito), há opção **Headscale** (implementação open-source do control plane). M1 usa Tailscale cloud; M2 avaliar Headscale se regulatório exigir soberania.

**Recomendação**: adotar Tailscale em **M1 pra acesso admin/dev** (low-hanging fruit, ROI altíssimo em 1 dia de setup). Não usar Tailscale como caminho de produto pros end-users.

## Tags/tópicos

- tailscale
- wireguard
- mesh-vpn
- ztna
- network-layer
- admin-access
- ssh-ca
- developer-experience
