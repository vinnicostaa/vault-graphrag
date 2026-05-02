---
title: Beyond-Corp / Zero-Trust — Destilado aplicável ao Master Síndico
type: note
tags: [research, beyond-corp, zero-trust, security, architecture, master-sindico]
source:
created: 2026-04-23
updated: 2026-04-23
---

# Beyond-Corp / Zero-Trust — Destilado aplicável ao Master Síndico

## 0. Contexto

Este destilado consolida pesquisa em fontes primárias (Google BeyondCorp/BeyondProd, NIST SP 800-207, CISA ZTMM v2, Cloudflare Zero Trust, Tailscale, Google Cloud IAP, AWS Zero Trust, Okta, Microsoft) e mapeia princípios à arquitetura multi-tenant do **Master Síndico** (SaaS governança condominial BR com 7 sub-produtos / BCs: governança institucional, Connect Me marketplace, reputação, conteúdo/vídeos, assembleias live, LMS, vizinhança).

Cada fonte primária foi arquivada em `source-NN.md` com URL + data de acesso + trechos literais.

---

## 1. Princípios canônicos Beyond-Corp / Zero-Trust

### 1.1 Premissa fundamental

> "Zero trust (ZT) provides a collection of concepts and ideas designed to minimize uncertainty in enforcing accurate, least privilege per-request access decisions in information systems and services in the face of a network viewed as compromised."
> — NIST SP 800-207, seção 2

Traduzindo: **a rede não é mais perímetro confiável**. Identidade do usuário + postura do dispositivo + contexto da requisição são o novo perímetro.

### 1.2 Os 7 tenets do NIST SP 800-207

1. **All data sources and computing services are considered resources** — qualquer endpoint (API, storage, serviço interno, SaaS) é um recurso que precisa de controle de acesso próprio.
2. **All communication is secured regardless of network location** — TLS/mTLS em tudo, inclusive "intranet".
3. **Access to individual enterprise resources is granted on a per-session basis** — nada de sessão longa abrindo portas; cada request reavalia.
4. **Access to resources is determined by dynamic policy** — política ABAC baseada em identidade + device posture + comportamento + risco.
5. **The enterprise monitors and measures the integrity and security posture of all owned and associated assets** — nenhum dispositivo é "confiável por padrão".
6. **All resource authentication and authorization are dynamic and strictly enforced before access is allowed** — continuous re-authentication.
7. **The enterprise collects as much information as possible about the current state of assets, network infrastructure, and communications and uses it to improve its security posture** — telemetria alimenta policy engine.

(Fonte: `source-02.md`)

### 1.3 BeyondCorp — os pilares Google (2014-2018)

Do paper original (Ward & Beyer, 2014) e da série "Design to Deployment" (Osborn et al., 2016), "The Access Proxy" (Cittadini et al., 2016) e "The User Experience" (2018):

- **Identity-centric**: identidade do usuário autenticada em IdP central, com MFA obrigatório.
- **Device-centric**: todo dispositivo com certificado único registrado em **Device Inventory** (meta-inventory). Trust Inferrer avalia postura e emite tier de confiança.
- **Access Proxy (AP)**: ponto único de entrada HTTPS que executa política coarse-grained (quem pode acessar o quê) e encaminha pra backends via identidade já verificada (ex.: cabeçalho assinado).
- **Access Control Engine (ACE)**: motor de decisão que cruza usuário × dispositivo × recurso × contexto.
- **Tiers de acesso**: trust inferrer classifica requisição em níveis (ex.: tier 1 / tier 2 / tier 3); recursos exigem tier mínimo.
- **Sem VPN privilegiada**: remover a intranet "confiável"; aplicações ficam expostas na internet mas protegidas pelo AP.

(Fontes: `source-01.md`, `source-03.md`, `source-04.md`)

### 1.4 BeyondProd — zero trust para microserviços (Google, 2019)

> "Service trust should depend on characteristics like code provenance, trusted hardware, and service identity, rather than the location in the production network. (...) Only authenticated, trusted, and specifically authorized callers or services can access any other service."
> — BeyondProd whitepaper

Princípios concretos:

- **Identidade de serviço** (não de máquina): SPIFFE-like, cada microserviço tem SVID próprio.
- **mTLS em toda comunicação leste-oeste** (ALTS na Google, Istio/Linkerd no resto do mundo).
- **Code provenance / Binary Authorization**: só roda binário que passou por revisão + build verificável.
- **Sandboxing** (gVisor): defesa em profundidade isolando workloads.
- **Automatic rollout / rollback**: pipelines CI/CD com canary e rollback garantem que falhas sejam revertidas rápido.

(Fonte: `source-05.md`)

### 1.5 CISA Zero Trust Maturity Model v2 — 5 pilares + 3 transversais

**Pilares**: Identity • Devices • Networks • Applications & Workloads • Data
**Transversais**: Visibility & Analytics • Automation & Orchestration • Governance
**Estágios de maturidade**: Traditional → Initial → Advanced → Optimal

Útil como **checklist de auditoria**: pra cada pilar, em que estágio está o master-sindico hoje, e onde precisa chegar no M1/M2?

(Fonte: `source-06.md`)

### 1.6 Microsoft Zero Trust — 3 princípios didáticos

1. **Verify explicitly** — autenticar/autorizar com todos os sinais disponíveis (identidade, localização, device health, classificação de dados, anomalias).
2. **Use least privilege access** — JIT/JEA, políticas adaptativas baseadas em risco.
3. **Assume breach** — minimizar blast radius, segmentar, cifrar end-to-end, analisar.

Vale como framing de 1 linha pra comunicar ao stakeholder não-técnico.

(Fonte: `source-09.md`)

---

## 2. Aplicação ao Master Síndico

### 2.1 Mapeamento por sub-produto (BC)

| BC | Superfície | Identidades | Dispositivos típicos | Políticas ABAC chave |
|---|---|---|---|---|
| **BC-identity** (IdP, sessão, RBAC/ABAC) | API `/auth`, `/session`, `/roles` | todos | todos | self-acesso |
| **BC-billing** (cobrança, inadimplência) | API `/billing`, integrações bancárias | síndico, admin, financeiro | mobile + web | condo_id + role=financeiro |
| **BC-institutional** (governança, atas, convenção) | API `/assembleias`, `/atas`, `/convencao` | síndico, conselho, morador | web + mobile | condo_id + role + unidade |
| **BC-commercial** (Connect Me marketplace) | API `/marketplace`, `/anuncios`, `/pedidos` | fornecedor, comprador, admin | web + mobile + PWA pública | tier de verificação do fornecedor, score de reputação |
| **BC-content** (vídeos, LMS, comunidade) | API `/videos`, `/cursos`, `/feed` | todos + criadores | web + mobile + TV | tier de conteúdo (público / moradores / síndicos) |
| **BC-assembly** (assembleias live) | WebRTC + API `/live`, `/voto` | morador elegível | web + mobile | quórum, device fingerprint (anti-bot), unidade registrada |
| **BC-neighborhood** (vizinhança) | API `/vizinhanca`, `/chat` | morador | mobile | condo_id + unidade + consent |

### 2.2 Princípios Beyond-Corp aplicados

**Princípio 1 — identidade é o perímetro.**
- IdP centralizado (Cognito/Keycloak/Auth0/Entra ID — escolha da M1) com MFA obrigatório pra papéis sensíveis (síndico, admin, financeiro).
- `user_id` + `tenant_id (condo_id)` + `roles[]` + `unidade_id` (quando aplicável) assinados em JWT com TTL curto (≤ 15 min) + refresh rotativo.
- OAuth 2.1 + **PKCE obrigatório** em SPAs e apps mobile (`source-07.md`).

**Princípio 2 — device como cidadão de primeira classe.**
- Device fingerprint (client-side) + bind de refresh token a `device_id` → invalidar sessão se device_id mudar.
- Em assembleia live, exigir atestação de device (mobile: Play Integrity / DeviceCheck; web: reCAPTCHA Enterprise + fingerprint) pra **prevenir fraude de votação**.
- Device inventory: tabela `devices` vinculada a `user_id` com trust_tier (novo / confirmado / suspeito / banido).

**Princípio 3 — ABAC + tiers, não apenas RBAC.**
- Políticas não são "síndico pode tudo"; são **ABAC**: `action = read_ata AND subject.role in (sindico, conselho) AND subject.condo_id = resource.condo_id AND resource.classificação <= subject.clearance`.
- Implementar com **Cedar (AWS), OPA (Rego), ou Casbin** (`source-08.md` AWS Verified Permissions usa Cedar).
- Tiers de recurso: `público`, `moradores`, `conselho`, `síndico`, `admin-plataforma`.

**Princípio 4 — todo tráfego cifrado, inclusive leste-oeste.**
- mTLS entre microserviços (BC-identity ↔ BC-billing ↔ BC-assembly). Service mesh (Istio ou Linkerd) se o footprint justificar; caso contrário, TLS mútuo manual com certs curtos rotados via SPIFFE/SPIRE.
- BeyondProd: **code provenance** = binary authorization no registry de containers; só deploys assinados pelo pipeline CI passam.

**Princípio 5 — audit trail imutável.**
- Log append-only pra todos os eventos sensíveis (login, mudança de papel, voto em assembleia, transação financeira, moderação de conteúdo).
- Hash-chain ou WORM storage (S3 Object Lock) pra atender LGPD + possibilidade de disputa judicial (assembleia contestada).
- Telemetria alimenta Policy Engine: sinais de risco (geolocalização nova, device novo, horário atípico) elevam nível de challenge (step-up auth).

**Princípio 6 — per-session, per-request access.**
- JWT curto + refresh rotativo. Revogação via denylist em Redis (short TTL) ou introspection endpoint.
- Para ações críticas (aprovar transação, encerrar votação): **step-up MFA** na hora, mesmo que sessão esteja válida (`source-07.md` IAP context-aware, `source-11.md` Okta device assurance).

**Princípio 7 — assume breach.**
- Segmentar rede: cada BC em VPC/namespace próprio; comunicação só via APIs explícitas passando pelo API Gateway + mTLS.
- Blast radius: comprometimento do BC-content não dá acesso a BC-billing. Chaves/segredos por BC, não globais.
- Simular breach: game days, chaos engineering, red team exercises (pós-M1).

### 2.3 Pilares concretos por camada

| Camada | Tecnologia sugerida | Responsabilidade |
|---|---|---|
| **Edge / CDN** | Cloudflare (ou CloudFront + WAF) | DDoS, WAF, bot protection, TLS termination |
| **Access Proxy equivalente** | API Gateway (AWS API GW, Kong, Envoy) | AuthN, rate limit, logging central |
| **IdP** | Cognito / Keycloak / Auth0 / Entra ID | OAuth 2.1 + PKCE, MFA, federation |
| **Policy Engine (PDP)** | OPA (Rego) ou Cedar (AWS Verified Permissions) | Decisão ABAC centralizada |
| **Policy Enforcement Point (PEP)** | Middleware em cada serviço ou Envoy com ext_authz | Aplica decisão por request |
| **Service mesh (leste-oeste)** | Istio / Linkerd / AWS App Mesh | mTLS, observabilidade, retry |
| **Device posture** | Play Integrity / DeviceCheck / reCAPTCHA Enterprise | Atestação device |
| **Audit trail** | PostgreSQL + hash-chain + S3 Object Lock | LGPD + disputas |
| **Observability** | OpenTelemetry + Grafana/Datadog | Sinais pra Policy Engine |

---

## 3. Gaps identificados no modelo atual do master-sindico

O legado vivo (`master-sindico-v1`) já tem conceito de BeyondCorp adaptado. Onde esta pesquisa amplia/corrige:

### 3.1 Gap: device trust é implícito, não explícito

**Atual**: o legado parece assumir que request autenticada já é confiável.
**Research**: BeyondCorp exige **device_id registrado + trust_tier**; Okta Device Assurance e Google IAP context-aware reforçam que identidade sem device é metade do Zero Trust.
**Recomendação**: introduzir tabela `devices` + fingerprint + trust_tier já no M1; em M2 adicionar atestação de plataforma (Play Integrity etc).

### 3.2 Gap: policy engine distribuído (lógica espalhada nos serviços)

**Atual**: regras de autorização tendem a ficar espalhadas em cada BC.
**Research**: NIST PE/PA/PEP é explícito em separar decisão de enforcement. OPA e Cedar são o estado da arte.
**Recomendação**: centralizar Policy Decision em serviço próprio (PDP) — OPA sidecar ou Cedar — e cada BC vira PEP. Facilita audit e reduz drift.

### 3.3 Gap: mTLS leste-oeste ausente / inconsistente

**Atual**: provável que comunicação entre serviços seja TLS simples ou HTTP interno.
**Research**: BeyondProd é claro — "no inherent mutual trust between services".
**Recomendação**: service mesh com mTLS automático (Linkerd é mais leve que Istio; em M1 talvez sidecar manual + SPIFFE/SPIRE).

### 3.4 Gap: audit trail não-imutável

**Atual**: logs em BD relacional mutável.
**Research**: LGPD + contestações de assembleia exigem prova forense (hash-chain, WORM).
**Recomendação**: dual-write — BD mutável pra query + S3 Object Lock (WORM) pra audit com hash-chain rolling. M2 pode adotar QLDB ou equivalente.

### 3.5 Gap: step-up auth não é padrão

**Atual**: sessão válida = acesso total.
**Research**: IAP + Okta + Microsoft exigem reautenticação em ações críticas.
**Recomendação**: marcar endpoints como `auth_level: normal | step_up | continuous`; step_up dispara MFA/biometria on-the-fly. Obrigatório em: aprovar transação, encerrar votação, mudar síndico, apagar ata.

### 3.6 Gap: segmentação multi-tenant não é defesa em profundidade

**Atual**: tenant isolation via `condo_id` em queries.
**Research**: assume breach — se atacante compromete um nível de auth, segmentação de rede e policy em PDP evitam escalar.
**Recomendação**: além do `condo_id` em queries, reforçar com RLS (row-level security) no Postgres + policy ABAC em OPA/Cedar. Duplo barrier.

### 3.7 Gap: device-fingerprint em assembleia live

**Atual**: risco de fraude em votação (mesmo usuário votando de múltiplos devices, bots).
**Research**: Beyond-Corp device inventory + atestação de plataforma.
**Recomendação (M1)**: restringir voto a 1 device_id ativo por sessão de assembleia + atestação via Play Integrity / DeviceCheck.

---

## 4. Recomendações priorizadas

### 4.1 M1 (2026-05-08) — fundações inegociáveis

1. **IdP centralizado** com OAuth 2.1 + PKCE + MFA pra papéis sensíveis.
2. **JWT curto (≤ 15 min) + refresh rotativo bound a device_id**.
3. **PDP centralizado** (OPA sidecar ou Cedar) com políticas ABAC versionadas em Git.
4. **mTLS entre serviços críticos** (pelo menos BC-identity ↔ BC-billing ↔ BC-assembly).
5. **Audit trail append-only** para: auth, mudança de papel, transação, voto.
6. **Device fingerprint** + tabela `devices` com trust_tier.
7. **Step-up auth** pra ações críticas listadas em 3.5.
8. **WAF + rate limit** no edge (Cloudflare ou CloudFront + AWS WAF).

### 4.2 M2 (6-12 meses pós-M1) — hardening

1. **Service mesh** com mTLS automático (Linkerd) e observabilidade leste-oeste.
2. **SPIFFE/SPIRE** pra identidade de workload.
3. **Binary Authorization** no pipeline (só containers assinados rodam em prod).
4. **Atestação de plataforma** em mobile (Play Integrity, DeviceCheck) pra assembleia.
5. **Audit WORM** (S3 Object Lock ou QLDB) com hash-chain.
6. **Continuous risk scoring** (behavior anomaly → step-up auth dinâmico).
7. **Game days / red team** pra validar assume-breach.

### 4.3 Futuro (M3+) — maturidade "Optimal" CISA

1. **Sandbox de workloads sensíveis** (gVisor / Firecracker).
2. **Zero trust pra integrações externas** (Open Banking, registros, CRM) via ZTNA (Tailscale/Cloudflare Access).
3. **Machine learning pra detecção de anomalias comportamentais** alimentando PDP.
4. **Automação de resposta** (SOAR): revogar sessão, bloquear device automaticamente em incidente.
5. **Certificações** (ISO 27001, SOC 2) com auditor externo validando o stack.

---

## 5. Riscos e trade-offs

- **Complexidade operacional**: service mesh + PDP + audit WORM são custo fixo. No master-sindico v2 (arquitetura monolito-modular ou microserviços moderados?), talvez **simplificar em M1**: OPA + JWT curto + audit em Postgres + hash-chain, adiando mesh pra M2.
- **Custo MFA**: MFA via SMS é caro + inseguro. Priorizar TOTP / passkeys (WebAuthn) — passkeys é free, sem SMS gateway.
- **UX em mobile**: step-up auth pode frustrar. Desenhar fluxos onde step-up é contextual ("confirmar com biometria"), não "login de novo".
- **LGPD x device fingerprint**: device fingerprint + geo-localização são dados pessoais. Consent explícito + finalidade legítima (prevenir fraude) documentados em DPA.
- **Vendor lock-in**: Cedar é AWS-only, OPA é portável; se master-sindico cogita multi-cloud ou self-host on-prem pra governo/sindicato grande, **preferir OPA**.

---

## 6. Decisões ainda em aberto (⚠️ PENDÊNCIAS)

- **IdP**: Cognito (lock-in AWS, simples) vs Keycloak (self-host, complexo) vs Auth0 ($$) vs Entra ID. Decidir em STEERING.
- **PDP**: OPA (portável) vs Cedar (AWS Verified Permissions, gerenciado). Decidir em STEERING após ADR.
- **Service mesh em M1?** — custo operacional vs ganho de segurança. Proposta: **M1 sem mesh, mTLS manual nos 3 caminhos críticos**; mesh só em M2.
- **Audit storage**: S3 Object Lock (simples) vs QLDB (deprecated pela AWS) vs TimescaleDB + hash-chain (self-host). Validar em ADR.

---

## 7. Referências

Ver arquivos `source-01.md` a `source-11.md` neste diretório. Cada fonte tem URL + data de acesso + trechos literais + aplicabilidade ao master-sindico.

| # | Fonte | Peso |
|---|---|---|
| 01 | BeyondCorp: A New Approach to Enterprise Security (Ward & Beyer, 2014) | canônico |
| 02 | NIST SP 800-207 Zero Trust Architecture (2020) | canônico |
| 03 | BeyondCorp: Design to Deployment at Google (Osborn et al., 2016) | canônico |
| 04 | BeyondCorp Part III: The Access Proxy (Cittadini et al., 2016) | canônico |
| 05 | BeyondProd — Google Cloud whitepaper (2019) | canônico |
| 06 | CISA Zero Trust Maturity Model v2 (2023) | canônico |
| 07 | Google Cloud Identity-Aware Proxy (IAP) docs | alto |
| 08 | AWS Zero Trust — Prescriptive Guidance | alto |
| 09 | Microsoft Zero Trust overview | médio |
| 10 | Cloudflare One / Zero Trust docs | alto |
| 11 | Tailscale Zero Trust Networking | médio |
| 12 | Okta Identity-centric Zero Trust whitepaper | médio |
