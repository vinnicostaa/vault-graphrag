---
title: MFA & Step-up Authentication
type: concept
tags:
  - security
  - mfa
  - 2fa
  - step-up
  - authentication
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Multi-Factor Authentication
  - 2FA
  - Step-up Auth
---

# MFA & Step-up Authentication

**MFA (Multi-Factor Authentication)**: exigir ≥ 2 fatores de **categorias diferentes** para autenticar. **2FA** é o caso específico de exatamente 2 fatores. **Step-up authentication**: elevar temporariamente o nível de autenticação quando user tenta operação sensível.

## Categorias de fatores (NIST SP 800-63B)

```
1. Something you KNOW       — senha, PIN, pergunta de segurança
2. Something you HAVE       — token hardware (YubiKey), celular (TOTP app),
                              chave assinatura (passkey)
3. Something you ARE        — biometria (fingerprint, face, voice)
4. (emergente) Somewhere    — location/IP context (sinal, não prova)
5. (emergente) Something    — behavior (typing cadence, ML signals)
   you DO
```

**MFA "real"** exige ≥ 2 categorias **distintas**. Senha + pergunta de segurança = **ambos KNOW** → não é MFA real.

## Força dos fatores (do pior ao melhor)

| Fator | Phishing-resistant? | Nível |
|---|---|---|
| Senha | ❌ | Baixo |
| SMS OTP | ❌ (SIM swap, interception) | Baixo-médio (evitar em novo design; NIST desencoraja) |
| Email OTP / Magic link | ⚠️ (phishable) | Médio |
| TOTP app (Authenticator, Authy, 1Password) | ⚠️ (phishable via real-time relay) | Médio |
| Push notification (app próprio) | ⚠️ | Médio-alto |
| WebAuthn / FIDO2 (passkey, YubiKey) | ✅ | Alto |
| Smart card + PIN | ✅ | Alto |
| Biometria device-bound + passkey | ✅ | Alto |

**Phishing-resistant** significa: mesmo se user cai em site falso, credencial não funciona no site real. Passkeys (origin-bound) são o padrão-ouro em 2026.

## Step-up authentication

User tem sessão baixa-assurance (logou com senha). Tenta operação sensível (wire transfer, data export, admin config). App exige **novo fator** para essa ação — não só re-auth completo.

Em OIDC, via claim `acr` (Authentication Context Class Reference):

```
Request inicial:
  /authorize?scope=openid&acr_values=urn:mace:incommon:iap:bronze

Token retornado: acr=1 (password-only)

User tenta /wire-transfer:
  App valida: acr >= 2? não.
  App → IdP: /authorize?prompt=login&acr_values=urn:mace:incommon:iap:silver
  IdP força re-auth com MFA.
  Token novo: acr=2 (password+MFA).
```

Claim `amr` (Authentication Methods References) lista métodos usados: `["pwd", "otp"]`, `["pwd", "hwk"]`, `["passkey"]`.

## Quando exigir MFA / step-up

### Sempre

- Login administrador.
- Mudança de senha / email / telefone do próprio user.
- Add/remove de payment methods.
- Permission grants (compartilhar admin, transferir ownership).
- Acesso a dados sensíveis (PII, financeiros, saúde).

### Sob critério

- Login de novo device (sinal: fingerprint novo).
- IP/geolocation anômala.
- Após 30d sem MFA.
- Tentativa de login falhada recente.

### Operações com step-up obrigatório

- Transferência financeira acima de limite.
- Export de dados em bulk.
- Revogação de session de outro user (logout forçado).
- Acesso a "break-glass" account.

## Padrões operacionais

### Factor enrollment

1. User loga com senha.
2. App oferece "Configure MFA" (ou obriga em role admin).
3. Enroll passkey/TOTP/SMS conforme política.
4. Armazena prova (public key passkey; seed TOTP criptografado).
5. Recovery codes 10x de 8 chars — impressos uma vez, guardados pelo user.

### Recovery flow

Sempre presente, sempre auditado:
- Recovery codes (offline, uso único).
- Fallback SMS/email (só se não há passkey).
- Manual admin reset (com audit + identity verification).

**Regra**: recovery é a **via principal de ataque**. Mais fatores + alerta ao user + cooldown.

### Trust on first use (TOFU) + remember device

- Primeiro login com MFA: marcar device como "trusted" (cookie longo + fingerprint).
- Próximos 30d: só senha.
- Expiração: re-exigir MFA.

## Como grandes empresas implementam

- **Google**: Passkey default para Google accounts; 2SV (SMS/TOTP) como fallback; backup codes; Advanced Protection Program para alto risco (FIDO2 hardware).
- **Microsoft Entra ID**: Passkey + Authenticator app + WebAuthn; Conditional Access dispara step-up por risk level (ML-based).
- **GitHub**: 2FA obrigatório desde 2023 para maintainers; passkey desde 2024; SMS deprecated para novos.
- **Apple**: 2FA obrigatório em Apple ID; Security Keys support (FIDO2).
- **Bancos (Brasil)**: biometria + PIN + device fingerprint + SMS OTP (apesar de SMS ser fraco, mantido por compliance BACEN).
- **Stripe**: MFA obrigatório para dashboard; SMS/TOTP/WebAuthn; step-up em operações como API key creation.

## Anti-patterns

1. **SMS como único segundo fator** — vulnerável a SIM swap, SS7 interception. NIST SP 800-63B desencoraja SMS.
2. **"2FA" com senha + pergunta de segurança** — ambos KNOW; não é MFA real.
3. **Sem recovery** — user perde celular, fica trancado. Recovery codes obrigatórios.
4. **Enrollment sem verify** — atacante enrolla seu próprio passkey na conta vítima. Exigir re-auth antes de enroll.
5. **MFA bypass em "remember device"** longo demais — 30d é max razoável; 1 ano é bypass permanente.
6. **Step-up sem expiration** — user faz step-up uma vez, token válido 24h para ops sensíveis.
7. **Fallback para senha quando passkey falha** — defeat o propósito. Deny + support.
8. **MFA opt-in em vez de opt-out** — maioria não habilita; ataques exploram.

## Relações

- **Factors específicos**: [[webauthn-passkeys]], [[otp-hotp-totp]], [[magic-links]]
- **Integração**: [[oidc]] (acr/amr claims), [[identity-provider]]
- **Princípio**: [[least-privilege]] (step-up alinha com POLP no tempo)
- **Session state**: [[session-management]]
- **OWASP**: [[owasp-top-10]] A07 (Authentication Failures)
- **Zero Trust**: [[beyond-corp]] (MFA é peça central)

## Fontes

- [NIST SP 800-63B — Authentication & Lifecycle Management](https://pages.nist.gov/800-63-3/sp800-63b.html) (consultada 2026-04-24)
- [FIDO Alliance Passkey docs](https://passkeys.dev/) (consultada 2026-04-24)
- [OWASP MFA Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html) (consultada 2026-04-24)
- [OIDC acr_values spec](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest) (consultada 2026-04-24)
- [Google Advanced Protection Program](https://landing.google.com/advancedprotection/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[webauthn-passkeys]]
- [[otp-hotp-totp]]
- [[magic-links]]
- [[oidc]]
- [[identity-provider]]
- [[session-management]]
- [[owasp-top-10]] — A07 Identification/Authentication Failures
- [[../runtime-antipatterns/op-024-rate-limit-ausente]] — auth flows exigem rate limit estrito
