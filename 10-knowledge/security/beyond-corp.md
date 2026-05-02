---
title: BeyondCorp — Zero Trust aplicado
type: concept
tags: [security, zero-trust, beyond-corp, authentication, authorization, abac]
source: Google BeyondCorp papers (2014-2016)
created: 2026-04-23
aliases: [BeyondCorp, Zero Trust, Zero Trust Security]
---

# BeyondCorp — Zero Trust aplicado

**BeyondCorp** é o modelo de segurança que o Google publicou em série de papers entre 2014-2016, formalizando o que hoje chamam de **Zero Trust Architecture**. Princípio central: **não existe "rede interna confiável"**. Toda request é autenticada, autorizada e auditada — de onde quer que venha.

> "The network **must** be assumed to be **untrusted**." — Rory Ward, Betsy Beyer, *BeyondCorp: A New Approach to Enterprise Security* (2014)

## Os 3 princípios fundantes

### 1. Identidade forte, verificada em toda request

- **Sem** VPN como "passe de acesso" — VPN garante rede, não identidade
- Toda request carrega **credencial de usuário** (token, cookie, mTLS)
- Verificação é **feita em cada borda**, não só no perímetro
- IdP (Identity Provider) é **serviço dedicado**: Google, Okta, Auth0, Zitadel, Keycloak

Em prática:
- OIDC com **PKCE** obrigatório em clients públicos (SPA, mobile)
- Sessão no IdP ≠ sessão na aplicação — aplicação valida token a **cada request** (com cache curto pra evitar latência)
- MFA como fator não-opcional para roles sensíveis
- Refresh tokens com rotação

Ver [[pkce]] *(a criar)* e [[security-principles]] (seção "OIDC — Introspection via JWT Profile").

### 2. Postura do device como sinal

Identidade de **usuário + device** é a unidade de decisão. Mesmo user em 2 devices = 2 contextos distintos.

Sinais coletados do device:
- **Device fingerprint** — hash de props únicas (user-agent + platform + screen + fonts + canvas)
- **Saúde do device** — jailbreak/root detection (mobile), anti-tampering (freeRASP, DexGuard)
- **Certificate pinning** — mobile valida cert do servidor contra pin local
- **Localização** / **IP range** — bloqueio de regiões não autorizadas
- **Tempo desde última verificação**

**1-device policy** (variante forte): um user ativo em **um** device por vez. Login em 2º invalida 1º. Simples, efetivo contra token stolen.

### 3. Autorização granular (ABAC > RBAC)

RBAC dá perfis (`admin`, `user`, `guest`) — muito grosseiro. **ABAC** combina:
- **Who** (role, plan_tier, department, mfa_level)
- **What** (resource type, resource owner, classification)
- **Where** (IP, geo, device posture)
- **When** (business hours, deadline windows)
- **How** (HTTP method, action severity)

Matrix declarativa substitui `if`s espalhados. Ver [[authorization-models]] *(a criar)*.

```go
// ✅ ABAC rule engine
type Rule struct {
    Role            Role
    PlanTierMin     PlanTier
    Action          string
    Resource        string
    RequireTenant   bool
    RequireMFA      bool
    Allow           bool
    DenyReason      string
}

var matrix = []Rule{
    {Role: RoleSyndic, PlanTierMin: PlanBase, Action: "timeline.create", Resource: "timeline", RequireTenant: true, Allow: true},
    {Role: RoleSyndic, PlanTierMin: PlanPlus, Action: "contract.sign", Resource: "contract", RequireTenant: true, RequireMFA: true, Allow: true},
    {Role: RoleResident, Action: "contract.sign", Resource: "contract", Allow: false, DenyReason: "role_not_allowed"},
}
```

## Adaptação pragmática (BeyondCorp "lite")

Google tem escala pra implementar o modelo puro. Produtos menores adaptam:

### Network proxy dedicado (Access Proxy)
Google tem GAP — um proxy que aplica política antes de chegar ao app. Alternativas enterprise:
- **Cloudflare Zero Trust** (antes: Cloudflare Access)
- **Tailscale** + SSO
- **BeyondCorp Enterprise** (Google Cloud)
- **Azure AD Application Proxy**

**Para SaaS típico**, não se usa Access Proxy dedicado — a política aplica direto no **middleware da aplicação**. Trade-off: política e app escalam juntos.

### Device inventory
Google mantém inventário de todo device corporativo. Alternativa SaaS:
- **Device fingerprint** via JS/mobile SDK
- **Client certificate** em devices gerenciados (MDM: Intune, Jamf, Kandji)
- **Sem inventário formal** + fingerprint + 1-device policy = boa aproximação

### Trust tiers
Google classifica devices em tiers (managed/BYOD/external). Aplicação lê tier e decide. Adaptação simples:
- `device_trust: {managed, byod, public}`
- Middleware lê do header assinado ou cookie
- ABAC incorpora como dimensão

## O que BeyondCorp **NÃO** diz

- Não diz "sem perímetro" — perímetro interno (segmentação, firewall) continua, apenas **não é a defesa primária**
- Não diz "sem VPN nunca" — VPN pode existir como canal; mas **sozinha não autoriza**
- Não diz "agnóstico de rede" — rede importa (bloqueio por geo, DDoS defense), mas **não é identidade**

## Stack típico de aplicação "BeyondCorp-ready"

```
┌──────────────────────────────────────────────────────┐
│ Layer 1: TLS/mTLS everywhere                         │
│   ✓ HTTPS obrigatório, HSTS, cert válido             │
│   ✓ Mobile pinning                                   │
├──────────────────────────────────────────────────────┤
│ Layer 2: Strong identity (IdP externo)               │
│   ✓ OIDC + PKCE (Zitadel, Okta, Auth0, Keycloak)     │
│   ✓ MFA TOTP/WebAuthn                                │
│   ✓ Refresh token rotation                           │
├──────────────────────────────────────────────────────┤
│ Layer 3: App-level session                           │
│   ✓ Cookie httpOnly + Secure + SameSite=Lax          │
│   ✓ CSRF double-submit ou Same-Site strict           │
│   ✓ Device fingerprint registrado                    │
│   ✓ 1-device policy (opcional, alta segurança)       │
├──────────────────────────────────────────────────────┤
│ Layer 4: Authentication middleware                   │
│   ✓ Valida token a cada request (cache curto 5min)   │
│   ✓ Extrai claims (user_id, role, plan, tenant_id)   │
│   ✓ Sincroniza mirror local se IdP externo           │
├──────────────────────────────────────────────────────┤
│ Layer 5: Authorization (ABAC engine)                 │
│   ✓ Matriz declarativa role × plan × action × resource│
│   ✓ Tenant isolation enforced                        │
│   ✓ Admin bypass global (audit log)                  │
│   ✓ Denial com reason estruturado                    │
├──────────────────────────────────────────────────────┤
│ Layer 6: Rate limiting + anomaly detection           │
│   ✓ Per-IP + per-user tiers                          │
│   ✓ Baseline por usuário (ML opcional)               │
│   ✓ Geo/velocity checks (login BR → 1min depois US?) │
├──────────────────────────────────────────────────────┤
│ Layer 7: Audit trail (LGPD-grade)                    │
│   ✓ Append-only, 5+ anos retenção                    │
│   ✓ Cada request sensível logada (user_id, resource) │
│   ✓ Alteração de dados pessoais = evento auditável   │
└──────────────────────────────────────────────────────┘
```

## Ordem de middleware canônica

```go
router.Use(
    RequestID(),        // 1. ID da request (audit)
    TLSRedirect(),      // 2. Força HTTPS
    CORS(),             // 3. Bloqueia wildcard em prod
    Recovery(),         // 4. Captura panic
    Logger(),           // 5. Estruturado, sem PII
    RateLimit(),        // 6. DDoS + brute force
    Authenticate(),     // 7. Valida token → ctx.User
    TenantExtractor(),  // 8. tenant_id no ctx
    Tracing(),          // 9. OTel
    Authorize(action),  // 10. ABAC por rota (via factory)
)
```

Ordem importa: Recovery **antes** de handlers (captura panic de qualquer lugar); Authenticate **antes** de Authorize (precisa user); Tracing **depois** de auth pra ter user_id no span.

## Anti-patterns BeyondCorp

- **"VPN dá acesso"**: VPN é rede, não identidade. User autenticado em VPN continua não-autorizado a recurso X.
- **Session cookie sem httpOnly/Secure**: XSS rouba. Sempre httpOnly + Secure + SameSite.
- **Token em localStorage**: JS acessível → XSS rouba. **Sempre** cookie httpOnly.
- **ABAC inline em handler** (`if user.role == "admin" { ... }`): espalha regra. Centraliza em engine.
- **CORS wildcard em prod**: `Access-Control-Allow-Origin: *` + credenciais = CSRF universal. Listar origens explícitas.
- **Rate limit só por IP**: múltiplos users atrás de NAT/proxy compartilham limite. Tier por userID também.
- **PII em logs**: CPF/email/token em log estruturado é leak. Mascarar sempre (`cpf.Masked()`).
- **Auth baseada em IP**: NAT, VPN, troca de wifi. IP é sinal **fraco**, nunca primário.
- **Secrets em .env commitado**: `.env` gitignored, `.env.example` committed. Secret real em vault (1Password, Doppler, AWS Secrets Manager).

## Anti-pattern da "confiança interna"

> "É request interna, vem de outro serviço nosso, confio."

**Não confia**. Service mesh com mTLS (Istio, Linkerd) ou assinatura de request (HMAC, JWT service-to-service) — prove identidade do chamador em cada request. Compromisso de 1 serviço não deve virar RCE em toda a frota.

## BeyondCorp + LGPD/GDPR

BeyondCorp dá **técnica**; LGPD/GDPR dão **política**. Complementam:

- **Audit trail** (BeyondCorp) → evidência de quem acessou dado pessoal (LGPD art. 46)
- **MFA para admins** (BeyondCorp) → protege dado pessoal (LGPD art. 49)
- **Data minimization** (LGPD art. 6) → ABAC só expõe campo necessário
- **Right to be forgotten** (LGPD art. 18) → tabela `pii_erasures` com hash irreversível

Ver [[security-principles]], [[owasp-top-10]] *(a criar)*.

## Checklist de conformidade BeyondCorp

- [ ] HTTPS + HSTS + cert válido
- [ ] IdP externo com OIDC + PKCE
- [ ] Cookie session httpOnly + Secure + SameSite
- [ ] Device fingerprint coletado
- [ ] Middleware valida token a cada request
- [ ] ABAC engine declarativa (não `if`s espalhados)
- [ ] Tenant isolation testada sistematicamente
- [ ] Rate limit tier por user (não só IP)
- [ ] Audit trail append-only com retenção adequada
- [ ] CORS sem wildcard em prod
- [ ] PII nunca em log
- [ ] Secrets em vault externo, zero no repo

## Referências

- Rory Ward, Betsy Beyer. *BeyondCorp: A New Approach to Enterprise Security* (2014) — Google
- Luca Cittadini et al. *BeyondCorp: Design to Deployment at Google* (2016)
- Betsy Beyer et al. *BeyondCorp: The User Experience* (2017)
- NIST SP 800-207. *Zero Trust Architecture* (2020) — padrão norte-americano
- CISA. *Zero Trust Maturity Model* (2023)
- John Kindervag. *No More Chewy Centers: Introducing The Zero Trust Model* (Forrester, 2010) — origem conceitual

## Links

- [[_moc]]
- [[security-principles]]
- [[../providers/cloudflare/sase]] — implementação comercial completa (Cloudflare One)
- [[../providers/cloudflare/access]] — ZTNA para apps internos
- [[../providers/cloudflare/tunnel]] — reverse-proxy outbound-only
- [[../providers/cloudflare/warp]] — client-side device trust agent
- [[../principles/solid]]
- [[../architecture/clean-architecture-layers]]
- [[../architecture/ddd-strategic-tactical]]
- [[../../20-stacks/go/go-patterns]]
