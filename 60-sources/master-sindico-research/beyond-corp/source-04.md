---
title: "BeyondCorp Part III: The Access Proxy (Cittadini et al., 2016)"
type: source
tags: [research, beyond-corp, zero-trust, google, canonical, access-proxy]
source: https://research.google/pubs/beyondcorp-the-access-proxy/
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 04 — BeyondCorp Part III: The Access Proxy

## Metadata

- **Autores**: Luca Cittadini, Batz Spear, Betsy Beyer, Max Saltonstall
- **Instituição**: Google Inc.
- **Publicação**: `;login:` Vol. 41, No. 4, Winter 2016, pp. 28-33
- **URL oficial**: https://research.google/pubs/beyondcorp-the-access-proxy/
- **URL USENIX PDF**: https://www.usenix.org/system/files/login/articles/login_winter16_05_cittadini.pdf
- **Data de acesso**: 2026-04-23
- **Tipo**: whitepaper / implementation deep dive

## Resumo

Terceira parte da série BeyondCorp. Foca na **Access Proxy (AP)** — o ponto único de enforcement HTTPS/SSH usado internamente pela Google. Descreve design, desafios de implementação e lições aprendidas.

Conteúdo central:

- AP é um **gateway centralizado** que faz authN/authZ de policy coarse-grained e encaminha a request autenticada ao backend.
- Backend **não re-autentica** o usuário final; confia no header assinado inserido pelo AP (ex.: `X-BeyondCorp-User`, `X-BeyondCorp-Device`).
- AP foi construído em cima do **Google Front End (GFE)**, a infraestrutura TLS+LB global da Google.
- AP suporta múltiplos protocolos:
  - HTTP(S) web apps.
  - SSH (via SSH CA + proxy SSH).
- Integra com **Access Control Engine (ACE)**: AP → ACE query → decisão (allow/deny/challenge).
- Logs centralizados no AP permitem forense unificada e atualização rápida de política (mudança no ACE aplica em todos APs).

Desafios abordados:
- Latência: AP é caminho crítico; mitigado com cache de decisões.
- Upload de arquivos grandes: streaming.
- Clientes não-browser (CLI, CI): certificados de client.
- Rollout: migração gradual, sombra paralela.

## Trechos-chave

> "We call the AP a 'coarse-grained policy enforcer' (...). The AP is a fleet of Google's front-end (GFE) servers, specially configured with additional logic and support for protocols other than HTTPS."
> — Cittadini et al., 2016

> "The Access Proxy enforces coarse-grained policies, for example, 'Group X is allowed to access Web Application Y'. Fine-grained authorization, at a per-request basis, is still the responsibility of the back-end applications."
> — Cittadini et al., 2016

> "The combination of the AP and the centralized Access Control Engine provides two main benefits: logs in a single location for forensic analysis, and a single point of policy update."
> — Cittadini et al., 2016 (síntese)

> "The AP must be generic enough to allow the implementation of logically different gateways using the same AP codebase."
> — Cittadini et al., 2016 (parafraseado do abstract)

## Aplicabilidade ao Master Síndico

**Parcial — adaptado ao contexto de SaaS B2B2C.**

Justificativa: master-sindico não opera SSH gateway nem intranet; opera APIs HTTPS de internet pública. Mas o conceito de AP como **ponto único de enforcement coarse-grained** se mapeia perfeitamente em um **API Gateway corretamente configurado**:

- **AWS API Gateway** ou **Kong** ou **Envoy (se self-host)** funcionam como AP.
- Autenticação JWT (validação assinatura + expiry + issuer + audience) no Gateway.
- Gateway injeta headers assinados (`X-MSS-User-Id`, `X-MSS-Condo-Id`, `X-MSS-Roles`, `X-MSS-Device-Id`, `X-MSS-Trust-Tier`) pro backend.
- Backend (cada BC) **não re-valida JWT** — confia nos headers e aplica autorização fine-grained (ex.: "este síndico pode editar esta ata específica?").
- Policy coarse-grained no Gateway: "quem pode bater em `/api/assembleia/*`" → delegado ao ACE (OPA/Cedar).
- Policy fine-grained no serviço: "quem pode editar ata 123 do condo 456" → lógica de negócio no BC-institutional.

**Divisão coarse vs fine** é a grande lição do paper. Evita que cada BC reimplemente toda a stack de auth.

⚠️ PENDÊNCIA: decidir se Gateway é AWS API GW (gerenciado, lock-in) ou Kong/Envoy (self-host, portável). ADR a abrir.

## Tags/tópicos

- zero-trust / beyond-corp
- access-proxy
- api-gateway
- coarse-grained-authz
- policy-enforcement-point
- gfe
