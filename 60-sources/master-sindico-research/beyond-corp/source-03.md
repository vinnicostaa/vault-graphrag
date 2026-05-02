---
title: "BeyondCorp: Design to Deployment at Google (Osborn et al., 2016)"
type: source
tags: [research, beyond-corp, zero-trust, google, canonical]
source: https://research.google/pubs/beyondcorp-design-to-deployment-at-google/
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 03 — BeyondCorp: Design to Deployment at Google

## Metadata

- **Autores**: Barclay Osborn, Justin McWilliams, Betsy Beyer, Max Saltonstall
- **Instituição**: Google Inc.
- **Publicação**: `;login:` Vol. 41, No. 1, Spring 2016, pp. 28-34
- **URL oficial (Google Research)**: https://research.google/pubs/beyondcorp-design-to-deployment-at-google/
- **URL USENIX**: https://www.usenix.org/publications/login/spring2016/osborn
- **URL PDF**: https://www.usenix.org/system/files/login/articles/login_spring16_06_osborn.pdf
- **Data de acesso**: 2026-04-23
- **Tipo**: whitepaper / experience report

## Resumo

Segundo paper da série BeyondCorp. Descreve como Google passou do whitepaper conceitual (Ward & Beyer 2014) para a implementação real em escala de ~60k funcionários.

Pontos-chave do paper:

- Diferente da segurança de perímetro convencional, BeyondCorp **não gata acesso por localização física ou rede de origem**. Políticas de acesso são baseadas em **informação do dispositivo, estado do dispositivo e usuário associado**, tratando redes internas e externas como completamente não-confiáveis.
- Enforcement dinâmico de **tiers** de acesso (ex.: "fully trusted" pode acessar tudo, "partially trusted" só recursos menos sensíveis, "untrusted" só páginas públicas).
- Apresenta componentes de implementação:
  - **Device Inventory Service**: fonte autoritativa de todos os devices corporativos; agrega dados de múltiplos sistemas.
  - **Trust Inferrer**: analisa estado do device e emite "trust tier".
  - **Access Control Engine**: motor centralizado de decisão.
  - **Access Proxy**: ponto único de enforcement em protocolo HTTPS (baseado em GFE, Google Front End).
  - **Pipeline de dados** ligando inventory → trust inference → ACE → AP.
- Desafios reais: migração gradual, suporte a aplicações legacy, manutenção de produtividade (tópico do paper "Maintaining Productivity", 2017).

## Trechos-chave

> "Unlike the conventional perimeter security model, BeyondCorp doesn't gate access to services and tools based on a user's physical location or the originating network; instead, access policies are based on information about a device, its state, and its associated user."
> — Osborn et al., 2016

> "BeyondCorp considers both internal networks and external networks to be completely untrusted, and gates access to applications by dynamically asserting and enforcing levels, or 'tiers,' of access."
> — Osborn et al., 2016

> "The Device Inventory Service (...) aggregates and processes data from multiple sources to maintain an up-to-date catalog of devices, users, and the relationships between them."
> — Osborn et al., 2016 (parafraseado da descrição do componente)

## Aplicabilidade ao Master Síndico

**Sim, canônico — especialmente o conceito de Device Inventory + Trust Tiers.**

Justificativa: master-sindico tem um problema real de **assembleia online com fraude potencial** (mesma pessoa votando de múltiplos devices, bots). Device Inventory + Trust Tier resolvem isso:

- Cada login registra `device_id` (fingerprint + cookie + atestação de plataforma quando disponível).
- Trust tier é computado: `novo` (recém-registrado, tier baixo) → `confirmado` (uso ≥ 30d sem anomalia) → `atestado` (Play Integrity/DeviceCheck ok) → `suspeito` (sinais de VM, emulator, fingerprint mudou).
- Políticas de assembleia: só devices `confirmado` ou `atestado` podem votar.
- Políticas financeiras (aprovar transação alta): requer tier `atestado` + step-up MFA.

**Implementação gradual**: M1 implementa fingerprint básico + tier `confirmado`; M2 integra Play Integrity/DeviceCheck pra tier `atestado`.

## Tags/tópicos

- zero-trust / beyond-corp
- device-inventory
- trust-tiers
- trust-inferrer
- access-proxy
- deployment-lessons
