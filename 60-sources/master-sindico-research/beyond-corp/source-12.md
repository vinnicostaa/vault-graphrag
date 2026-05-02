---
title: "Okta — Identity-centric Zero Trust"
type: source
tags: [research, zero-trust, okta, identity, device-posture, mfa]
source: https://www.okta.com/resources/whitepaper/zero-trust-with-okta-modern-approach-to-secure-access/
created: 2026-04-23
updated: 2026-04-23
---

# Fonte 12 — Okta Identity-centric Zero Trust

## Metadata

- **Instituição**: Okta, Inc.
- **URL whitepaper principal**: https://www.okta.com/resources/whitepaper/zero-trust-with-okta-modern-approach-to-secure-access/
- **URL whitepaper OSIC**: https://www.okta.com/resources/whitepaper-leveraging-the-okta-secure-identity-commitment-to-enable-zero-trust/
- **URL PDF OSIC**: https://www.okta.com/sites/default/files/2024-08/Okta-Secure-Identity-Commitment-Zero-Trust.pdf
- **URL State of Zero Trust 2021 (PDF)**: https://www.okta.com/sites/default/files/2021-07/WPR-2021-ZeroTrust-070821.pdf
- **URL blog device context 2024**: https://www.okta.com/blog/2024/09/okta-advances-device-context-features-for-zero-trust-security/
- **Data de acesso**: 2026-04-23
- **Tipo**: whitepaper comercial / framework

## Resumo

Okta posiciona-se como **fornecedor de identidade** e enquadra Zero Trust como "identity-powered": identidade é o perímetro moderno, e IAM é o ponto de partida obrigatório de qualquer jornada ZT.

Componentes Okta relevantes pra Zero Trust:
- **Okta Identity Cloud** — IdP federado (OAuth/OIDC/SAML).
- **Okta Adaptive MFA** — MFA contextual (fator adicional baseado em risco: device novo, geo anômala, horário).
- **Okta Device Assurance** — validação de atributos do device pra aplicar policies.
- **Okta FastPass** — autenticação passwordless baseada em device trust.
- **Okta Advanced Server Access (ASA)** — SSH/RDP baseado em identity, sem chaves estáticas (concorrente Tailscale/Teleport).
- **Lifecycle Management** — provisioning/deprovisioning automático.

**5 níveis de maturidade "identity-powered Zero Trust"** propostos pela Okta:
1. Fragmented identity.
2. Unified IAM (SSO centralizado).
3. Contextual access (policies adaptativas).
4. Adaptive workforce (continuous authentication).
5. Frictionless access (passkeys, passwordless).

## Trechos-chave

> "Trust can no longer be implied (...). Identity is the modern perimeter."
> — Okta Zero Trust whitepaper

> "IAM solutions offer the core technology that organizations should start with on their zero trust journeys."
> — Okta whitepaper

> "Credential abuse and highly targeted phishing attacks remain the leading cause of breaches today."
> — Okta State of Zero Trust

> "Verifying the identity of users accessing corporate resources and the security posture of the devices on which users are accessing those resources from becomes critical."
> — Okta whitepaper (Zero Trust positioning)

> "Okta's Device Assurance validates device attributes to enhance security by ensuring devices accessing resources meet the security and compliance conditions configured."
> — Okta blog, Sept 2024

> "Zero Trust is not a novel concept (...) this is not something that happens overnight, or is in fact ever actually 'complete'."
> — Okta whitepaper

## Aplicabilidade ao Master Síndico

**Parcial — útil como referência conceitual + opção comercial de IdP.**

Justificativa:

1. **Okta como IdP do master-sindico**: opção se o budget aguentar. Okta Customer Identity (CIAM) é caro comparado a Cognito / Keycloak, mas entrega UX superior e **catalogo de integrações** (SCIM, SAML, OIDC com dezenas de provedores). Relevante se clientes corporate exigirem federation com seus próprios IdPs.

2. **Adaptive MFA / step-up** é exatamente o padrão que master-sindico precisa implementar:
   - Login normal: senha + 2FA.
   - Ação sensível (aprovar transação, encerrar votação): step-up (biometria, push).
   - Sinal de risco (device novo, geo nova): step-up automático.

3. **Device Assurance**: conceito 1:1 aplicável ao `trust_tier` do device no master-sindico.

4. **Advanced Server Access (ASA)**: alternativa a Tailscale pra acesso admin baseado em identity+SSH CA. Valor adicional: integra nativo com o IdP Okta.

5. **5 níveis de maturidade identity-powered** são uma **lente complementar ao CISA ZTMM** — focada só em identidade, útil pra especificar roadmap detalhado do BC-identity.

**Recomendação**:
- Se M1 optar por **Keycloak self-host** ou **Cognito** (mais provável pelo custo), reaproveitar os **padrões** de Okta (adaptive MFA, device assurance, step-up) mesmo em IdP mais barato.
- Se a tese mudar e master-sindico mirar vendas a redes corporate grandes (administradoras com 500+ condomínios), avaliar Okta Customer Identity em M2/M3.

⚠️ PENDÊNCIA: ADR "IdP: Cognito vs Keycloak vs Okta vs Auth0 vs Entra ID".

## Tags/tópicos

- okta
- identity
- iam
- zero-trust
- adaptive-mfa
- device-assurance
- fastpass
- passwordless
- identity-perimeter
