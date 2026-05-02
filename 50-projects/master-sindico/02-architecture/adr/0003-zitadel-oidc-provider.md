---
title: ADR-0003 — Zitadel como IdP OIDC
type: adr
tags: [adr, identity, oidc, zitadel, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0003 — Zitadel como IdP OIDC

## Status

accepted — 2026-04-23 (herdado de D-001 legado)

## Contexto

Identidade é o perímetro zero-trust ([[../../13-research/beyond-corp/_destilado]] §2.2). Precisamos IdP com:

- OIDC Authorization Code + PKCE (mobile + SPA seguros).
- MFA TOTP / WebAuthn.
- Organizations nativas (mapeadas 1:1 para tenants).
- Self-host viável (soberania, custo controlado em early-stage).
- Licença aberta.

## Decisão

**Zitadel** como IdP OIDC. Self-host via container em M1 (Railway); avaliar managed Zitadel Cloud em M3+.

- **Authorization Code + PKCE** obrigatório em SPA e mobile.
- **MFA TOTP** obrigatório para papéis sensíveis (síndico, admin, financeiro empresa).
- **WebAuthn** como factor premium (M2+).
- **Organizations Zitadel** = tenants Master Síndico (sync via Zitadel Actions → webhook).
- **Cookie httpOnly** `.mastersindico.com.br` para web; Bearer JWT para mobile.

## Consequências

### Positivas

- Paridade feature-to-feature com Google Identity Platform + Auth0 — [[../../13-research/google-arch/_destilado]] §3.1.
- Self-host: zero vendor lock, custo zero em early-stage.
- Actions hook permite customizar fluxo (pré/pós login, pré registro).
- MFA / WebAuthn nativo.

### Negativas

- Operação self-host (upgrade, cert TLS, backup, HA) é responsabilidade do time.
- Comunidade menor que Keycloak — documentação às vezes falta.
- UI admin Zitadel é menos polida que Auth0.

## Alternativas consideradas

1. **Cognito (AWS)** — lock AWS; ruim se migrar para GCP. Preço escalonado ok.
2. **Keycloak** — self-host pesado (JVM), complexo. Comunidade enorme.
3. **Auth0** — $$ alto em scale; vendor lock.
4. **Entra ID (Microsoft)** — enterprise-heavy; overkill.
5. **Senha local + JWT próprio** — reinventar a roda; inseguro.

## Referências

- [[../../13-research/beyond-corp/_destilado]] §2.2
- [[../../13-research/google-arch/_destilado]] §3.1 (Identity Platform comparison)
- [[0016-beyondcorp-security-posture]]
- Benchmark defensivo se cliente pedir "Google auth": ver fonte source-12 da pasta google-arch.
