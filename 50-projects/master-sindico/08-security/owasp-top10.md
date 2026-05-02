---
title: OWASP Top 10 2021 — Mapping Master Síndico v2
type: spec
tags: [security, owasp, master-sindico, v2, mapping]
source: https://owasp.org/Top10/ + 90-ingested legacy
created: 2026-04-23
updated: 2026-04-23
aliases: [owasp, owasp-top10]
---

# OWASP Top 10 2021 → Controles Master Síndico

Mapeamento explícito **OWASP Top 10 2021** aos controles do master-sindico v2. Documento de due-diligence: auditor externo vai perguntar "como vocês cobrem A01? A02?". Aqui está a resposta canônica com apontadores para specs + código.

**Fonte primária**: https://owasp.org/Top10/ (acesso 2026-04-23). Versão 2021 é a mais recente (próxima atualização prevista 2025).

**Herança**: [[BEYOND_CORP]] + [[threat-model]]. Este doc é overlay categorizado por OWASP; STRIDE por BC continua em threat-model.

---

## Sumário por categoria

| Categoria OWASP | Severidade | Cobertura Master Síndico | Status |
|---|---|---|---|
| [A01 Broken Access Control](#a01) | 🔴 | ABAC matriz + tenant isolation + 1-device + step-up auth | 🟢 M1 base + 🟡 M2 step-up |
| [A02 Cryptographic Failures](#a02) | 🔴 | TLS 1.3 + PKCE + securecookie + Stripe SAQ-A | 🟢 M1 |
| [A03 Injection](#a03) | 🔴 | sqlc parametrizado + VO validation + PII mask | 🟢 M1 |
| [A04 Insecure Design](#a04) | 🟡 | Threat-model first + DoR inclui risco | 🟢 M1 (processo) |
| [A05 Security Misconfiguration](#a05) | 🟡 | Config.Validate() + CORS strict + secrets gitignored | 🟢 M1 |
| [A06 Vulnerable Components](#a06) | 🟡 | govulncheck + npm audit + osv-scanner + Dependabot | 🟢 M1 |
| [A07 Auth Failures](#a07) | 🔴 | Zitadel OIDC + PKCE + MFA (M2+) + rate limit | 🟡 M2 (MFA obrigatório) |
| [A08 Data Integrity Failures](#a08) | 🟡 | HMAC webhook + UNIQUE + UoW + Saga | 🟢 M1 |
| [A09 Logging & Monitoring Failures](#a09) | 🟡 | structured logs sem PII + audit_log + OTel | 🟢 M1 base + 🟡 M2 hash-chain |
| [A10 SSRF](#a10) | 🟡 | Allowlist URLs externas + sem user-controlled fetch | 🟢 M1 |

---

## A01 — Broken Access Control {#a01}

> "Access control enforces policy such that users cannot act outside of their intended permissions."
> — OWASP Top 10 2021

### Exemplos no contexto master-sindico

- IDOR: morador de Condomínio X lendo `GET /api/v1/units/<id_de_Y>` via trocando UUID.
- Cross-tenant: síndico de X usando token pra editar recurso de Y.
- Privilege escalation: morador mudando seu próprio `role` via PATCH /me.
- Force browse: acessar `/admin/*` sem role admin.
- Session fixation: reutilizar cookie após logout.

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **ABAC matrix ordenada** | `admin_bypass → role → action → tenant → scope → plan → reputation → step_up` | [[BEYOND_CORP]] §3 |
| **Tenant isolation em queries** | `WHERE condominium_id = $tenant_id` obrigatório; CI grep bloqueia | [[threat-model]] §5.1 |
| **PostgreSQL RLS (defesa em profundidade)** | ⚠️ M2 — `POLICY tenant_isolation ON X USING (condominium_id = current_setting('app.tenant_id'))` | [[threat-model]] §5.1 |
| **1-device policy** | Fingerprint + sessions invalidation | [[BEYOND_CORP]] §4 |
| **Step-up MFA em ações críticas** | ⚠️ M2 — approve transaction, publish minutes, delete account | [[BEYOND_CORP]] §1 Tenet 6 |
| **Cross-tenant tests** | 100+ integration tests sistemáticos | [[threat-model]] §5.1 |
| **PBT ABAC** | ≥ 400 casos cobrindo bypass attempts | [[BEYOND_CORP]] §3.4 |
| **Audit log DENY** | Toda decisão DENY registrada com deny_reason | [[BEYOND_CORP]] §11 |
| **UUIDv7** ao invés de sequential IDs | Inviabiliza enumeration | — |
| **Route params validados** | UUID format enforced no decode + ABAC re-check | [[threat-model]] §4.2 |

### Gaps (A-###)

- Gap-SEC-003: PostgreSQL RLS (M2).
- Gap-SEC-002: step-up MFA (M2).
- Gap-SEC-012: PDP sidecar OPA (M2).

---

## A02 — Cryptographic Failures {#a02}

> "Failures related to cryptography... often lead to sensitive data exposure or system compromise."

### Exemplos master-sindico

- PAN (cartão) armazenado em plaintext → **nunca ocorre** (Stripe Elements SAQ-A).
- CPF/CNPJ em plaintext DB → mitigado com encryption at rest.
- JWT assinado com `none` → nunca aceitar.
- HMAC comparado com `==` (timing attack) → usar `hmac.Equal`.
- TLS fraco (1.0/1.1) → forçar 1.3.

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **TLS 1.3 norte-sul** | Railway / CloudFront force TLS 1.3; plaintext HTTP bloqueado | [[BEYOND_CORP]] §1 Tenet 2 |
| **mTLS leste-oeste (M2+)** | 3 caminhos críticos; service mesh M3 | [[BEYOND_CORP]] §12.6 |
| **PKCE obrigatório OIDC** | SHA256 code_challenge; verify server-side | [[BEYOND_CORP]] §5.1 |
| **Cookie encryption** | gorilla/securecookie (AES-256-GCM + HMAC-SHA256) | [[BEYOND_CORP]] §5.2 |
| **Postgres encryption at rest** | Cloud provider transparent (Railway/AWS RDS) | — |
| **Redis TLS + AUTH** | Prod only (dev relax) | [[BEYOND_CORP]] §1 Tenet 2 |
| **Stripe SAQ-A** | Nunca tocamos PAN; Stripe Elements + redirect | [[threat-model]] §4.6 |
| **HMAC constant-time** | `hmac.Equal()` em toda signature verify | [[threat-model]] §7.2 |
| **UUIDv7** | Ordenável + não-previsível (vs sequential int) | — |
| **Password storage** | Delegado a Zitadel (bcrypt/argon2) — nunca armazenamos password | — |
| **Secret rotation** | Trimestral (stripe, mux); Zitadel key on-compromise | [[BEYOND_CORP]] §10 |
| **Certificate pinning mobile (prod)** | Flutter HTTP client com pinning em hosts conhecidos | [[BEYOND_CORP]] §1 Tenet 2 |

### Gaps

- **Field-level encryption CPF/CNPJ em repouso**: ⚠️ PENDÊNCIA M3 — avaliar pgcrypto vs application-level envelope encryption.
- Gap-SEC-015: AWS Secrets Manager (M2).

---

## A03 — Injection {#a03}

> "An application is vulnerable to injection attacks when user-supplied data is not validated, filtered, or sanitized."

### Exemplos master-sindico

- SQLi em query dinâmica → mitigado com sqlc (todas as queries parametrizadas).
- NoSQL injection em OpenSearch → mitigado com client library + query builder.
- Command injection via upload filename → mitigado (filename não usado em shell).
- LDAP injection → não aplicável (sem LDAP direto).
- XXE em XML → não aplicável (sem parser XML user-facing).
- Template injection (SSTI) → mitigado (PDF gen usa lib confiável; no user-controlled template).
- XSS reflected/stored → mitigado (SolidJS escape default; CSP header; sanitizar rich text via DOMPurify).
- Log injection → mitigado (structured logging; sem `\n` em user input).

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **sqlc queries parametrizadas** | Todas as queries via sqlc; zero string concat | [[../02-architecture/patterns]] (a escrever) §sqlc |
| **VO constructors** | `NewCPF`, `NewEmail`, `NewPhone` validam formato + regras | [[../90-ingested/from-vault-50-projects-master-sindico/legacy-reference]] §VOs |
| **OpenSearch client parametrizado** | `client.Search(indexName, query)` com query builder typed | — |
| **Body size limit 1MB** | Middleware bloqueia payload gigante | [[BEYOND_CORP]] §2 |
| **SolidJS default escape** | `{var}` escapa HTML; `innerHTML` proibido por review | [[../03-subprojects/web/_moc]] (a escrever) |
| **CSP header strict** | `default-src 'self'; script-src 'self' https://js.stripe.com` | [[BEYOND_CORP]] §15 |
| **DOMPurify em rich text** | Comentário, descrição RFP usam sanitizador | — |
| **Filename sanitize** | Upload Mux: filename é opaque; nunca usado em path/shell | — |
| **PII mask em logs** | Previne log injection com payload malicioso | [[BEYOND_CORP]] §7 |
| **gosec SAST** | `-severity=high` 0 findings (gate CI) | [[cve-process]] §1 |

### Gaps

- **ReDoS em validação de email/regex**: ⚠️ PENDÊNCIA — regex validação com timeout ou RE2 (Go regexp já é RE2).
- **Path traversal em upload**: validar que filename não contém `..` ou `/` — controle existe, mas PBT pra confirmar.

---

## A04 — Insecure Design {#a04}

> "Insecure design refers to risks related to design and architectural flaws. Secure design is a culture and methodology."

### Exemplos master-sindico

- Feature nova sem threat-model → **bloqueado pelo processo**.
- Business logic flaw (ex: desconto negativo aceito) → mitigado por invariantes de domínio.
- Missing rate limit em operação cara → mitigado (rate limit estratégico).
- Race condition em vote → mitigado (DB UNIQUE + UoW).

### Controles (processo > tech)

| Controle | Implementação | Ref |
|---|---|---|
| **Threat-model first** | Toda feature passa por STRIDE antes de virar task | [[../CLAUDE]] §4 |
| **DoR inclui risco de segurança** | Checklist: "requer threat-model? requer pentester review?" | [[../DoR-DoD]] |
| **SDD 5 fases** | Discuss → Plan → Execute → Verify → Ship — ShipGate exige threat review | [[../CLAUDE]] §7 |
| **Dual-check em decisões críticas** | Security policy muda → dual-check obrigatório | [[../STEERING]] §32 |
| **Invariantes de domínio** | PBT em fração ideal, quota, vote único, ABAC, money BRL | [[threat-model]] §14 |
| **State machines strict** | Video/Vote/Contract/Minutes lifecycle explicit com transitions válidas | [[../01-domain/state-machines]] (a escrever) |
| **Saga com compensação** | UoW intra-módulo; Saga inter-BC com compensação explícita | [[../02-architecture/patterns]] (a escrever) |
| **Legacy-as-reference** | Antipatterns v1 catalogados (AP-###); evitados na v2 | [[../90-ingested/from-vault-50-projects-master-sindico/legacy-reference]] |
| **Architecture decision records** | ADR obrigatório em decisões arquiteturais | [[../CLAUDE]] §5 |
| **Pentester role na equipe (simulado)** | Papel formal no protocolo 5-papéis | [[../CLAUDE]] §4 |

### Gaps

- **Abuse cases em spec**: ⚠️ PENDÊNCIA — cada requirement deve listar "abuse cases" explicitamente (M2 process upgrade).

---

## A05 — Security Misconfiguration {#a05}

> "Missing appropriate security hardening across any part of the application stack."

### Exemplos master-sindico

- CORS wildcard `*` em prod → **bloqueado** `Config.Validate()` (A-005).
- Secrets em `.env` commitado → bloqueado gitignore + gitleaks.
- Debug endpoints em prod → bloqueado.
- Default Zitadel settings inseguros → hardened.
- Error stack trace vazando em response → apenas `request_id`; stack em logs.
- Postgres default port público → rede privada Railway.

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **Config.Validate() em boot** | Falha startup se configuração insegura | [[BEYOND_CORP]] §9 |
| **CORS allowlist strict** | Wildcard `*` bloqueado staging/prod (A-005 resolvido) | [[BEYOND_CORP]] §9 |
| **Secrets gitignored** | `.env`, `zitadel-key*.json`, `*.pem`, `*.key` + CI gitleaks | [[BEYOND_CORP]] §10.3 |
| **Security headers** | `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Content-Security-Policy`, `Referrer-Policy: strict-origin` | [[BEYOND_CORP]] §15 |
| **No debug in prod** | `DEBUG=false`; verbose logs só em dev | — |
| **Error response sanitizado** | `{error: {code, message, request_id}}`; stack só em log | — |
| **Postgres conn string sslmode=require** | Prod/staging forçam TLS | [[BEYOND_CORP]] §1 Tenet 2 |
| **Docker image minimal** | Distroless base; no shell; no package manager | ⚠️ PENDÊNCIA M2 |
| **Railway / AWS variables encrypted** | Provider default | [[BEYOND_CORP]] §10.2 |
| **Security baseline CIS Docker/K8s** | ⚠️ PENDÊNCIA M3 quando migrar ECS | — |
| **Boot banner check** | Startup log imprime versão sanitized; não imprime secrets | — |

### Gaps

- **Distroless container**: ⚠️ PENDÊNCIA M2 — hoje Alpine base.
- **CSP refinement**: ⚠️ PENDÊNCIA — hash inline scripts SolidJS build time.
- **HTTP security headers completos**: review M1 (HSTS, X-Content-Type-Options etc).

---

## A06 — Vulnerable and Outdated Components {#a06}

> "Applications using components with known vulnerabilities may undermine defenses."

### Exemplos master-sindico

- CVE em `golang.org/x/crypto` → detectado por govulncheck.
- axios CVE em web → detectado por npm audit.
- Dio CVE em mobile → detectado por osv-scanner (F26 legado histórico).
- gin/pgx/go-redis outdated → Dependabot alert.

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **govulncheck CI gate** | 0 reachable CVEs em PR | [[cve-process]] §7 |
| **gosec SAST CI gate** | -severity=high 0 findings | [[cve-process]] §2 |
| **npm audit CI gate (web)** | --audit-level=high --production | [[cve-process]] §7 |
| **osv-scanner (mobile)** | pubspec.lock scan | [[cve-process]] §2 |
| **GitHub Dependabot** | Alertas diários + PRs automáticas | [[cve-process]] §2 |
| **trivy fs + image scan** | CI + pre-deploy | [[cve-process]] §2 |
| **dep-audit playbook** | Semanal review | [[../06-execution/playbooks/dependency-audit]] (a escrever) |
| **SLA CVE por severidade** | CRITICAL 12h, HIGH 5d, MEDIUM 30d, LOW 90d | [[cve-process]] §1 |

### Gaps

- **SBOM geração**: ⚠️ PENDÊNCIA M2 — `syft` ou `cyclonedx` produz SBOM em cada build.
- **Sigstore / cosign**: ⚠️ PENDÊNCIA M3 — Binary Authorization.

---

## A07 — Identification and Authentication Failures {#a07}

> "Applications need confirmation of the user's identity, authentication, and session management."

### Exemplos master-sindico

- Credential stuffing → rate limit + (M2) CAPTCHA após 3 falhas.
- Weak password policy → delegado a Zitadel (mínimo 12 chars + MFA).
- Session fixation → session ID renovado pós-login; 1-device invalidation.
- Exposed session IDs em URL → nunca — sempre httpOnly cookie / Bearer.
- MFA ausente → MFA opcional M1 / obrigatório M2 pra síndico/empresa/financeiro.

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **Zitadel OIDC** | IdP gerenciado com MFA built-in | [[BEYOND_CORP]] §5.1 |
| **PKCE obrigatório** | OAuth 2.1 em SPAs e mobile | [[BEYOND_CORP]] §5.1 |
| **MFA opcional M1 / obrigatório M2** | Síndico + empresa + admin + financeiro | [[BEYOND_CORP]] §14.2 |
| **Rate limit /auth** | 20rpm burst 5 por IP | [[BEYOND_CORP]] §6 |
| **1-device policy** | Invalida session anterior | [[BEYOND_CORP]] §4 |
| **Session TTL curto** | Access 10-15min; refresh 30d rotativo | [[BEYOND_CORP]] §1 Tenet 3 |
| **Refresh token bound a device_fp** | Invalidado se device muda | [[BEYOND_CORP]] §4 |
| **Logout revoga refresh** | Denylist Redis | [[BEYOND_CORP]] §1 Tenet 3 |
| **Step-up MFA em ações críticas** | ⚠️ M2 | [[BEYOND_CORP]] §1 Tenet 6 |
| **Forgot password cooldown** | Zitadel + não reenvia se anterior válido | [[../90-ingested/from-vault-50-projects-master-sindico/legacy-reference]] |
| **Passkeys (WebAuthn) M3** | Mais seguro que TOTP | ⚠️ PENDÊNCIA M3 |
| **CAPTCHA em brute force** | ⚠️ PENDÊNCIA M2 — reCAPTCHA Enterprise ou hCaptcha |  |

### Gaps

- Gap-SEC-002: step-up MFA (M2).
- Gap-SEC-001: Play Integrity / DeviceCheck (M2).
- **Passkeys**: M3.
- **CAPTCHA**: M2.

---

## A08 — Software and Data Integrity Failures {#a08}

> "Making assumptions related to software updates, critical data, and CI/CD pipelines without verifying integrity."

### Exemplos master-sindico

- Webhook Stripe forjado → HMAC verify.
- Race condition em vote → UNIQUE + UoW.
- Dependency comprometida → govulncheck + lockfile pin.
- CI/CD não assina imagens → ⚠️ M3 cosign.
- Deserialização insegura → não aplicável (JSON-only).

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **HMAC webhook (Stripe/Mux/Livekit)** | Verify antes de parse | [[BEYOND_CORP]] §8 |
| **Idempotency webhook** | UNIQUE(event_id) | [[threat-model]] §4.3 |
| **UNIQUE DB constraints** | Vote, ConnectMeProposal, membership_active | [[threat-model]] §14 |
| **UoW intra-módulo** | Atomic writes com rollback | [[../02-architecture/patterns]] §uow |
| **Saga inter-BC com compensação** | Video upload (A-027), Livekit room (A-028) | [[threat-model]] §13 |
| **Go lockfile (`go.sum`)** | Pinned + checksum verify | — |
| **CI/CD no merge sem gates** | build+test+gosec+govulncheck+coverage | [[BEYOND_CORP]] §15.1 |
| **Binary Authorization (M3)** | cosign + Kyverno/OPA Gatekeeper | [[BEYOND_CORP]] §14.3 |
| **Audit log append-only** | Sem UPDATE/DELETE grants | [[BEYOND_CORP]] §11 |
| **Timeline/Minutes immutable** | Domain invariant + DB trigger M2 | [[threat-model]] §14 |

### Gaps

- Gap-SEC-004: hash-chain rolling no audit_log (M2).
- **Cosign**: M3.
- **SBOM**: M2.

---

## A09 — Security Logging and Monitoring Failures {#a09}

> "Without logging and monitoring, breaches cannot be detected."

### Exemplos master-sindico

- PII em log → bloqueado (CI grep + VO Masked).
- Auth events não logados → audit_log cobre login/logout/MFA/1-device/step-up.
- Logs mutáveis → audit_log append-only + (M2) hash-chain.
- Sem alerta em DoS → ⚠️ M2 Grafana alerts.

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **Structured logging** | zerolog JSON; correlation ID via request_id | [[BEYOND_CORP]] §7 |
| **PII masking** | CPF/CNPJ/email/token sempre mascarados; CI grep | [[BEYOND_CORP]] §7 |
| **audit_log append-only** | 5 anos retention; partitioned monthly | [[BEYOND_CORP]] §11 |
| **IAuditLogger interface** | DT-010 centralizado cross-BC | [[BEYOND_CORP]] §11 |
| **OpenTelemetry** | Traces + metrics export | [[../02-architecture/patterns]] (a escrever) §observability |
| **Sentry** | Error tracking + alert | — |
| **Grafana dashboards** | SLI/SLO por BC | ⚠️ PENDÊNCIA M2 |
| **Alertas automáticos** | Rate de erros 5xx, latência p99, falhas ABAC DENY spike | ⚠️ PENDÊNCIA M2 |
| **Log retention 5 anos LGPD** | audit_log partitioned + S3 export pós-5y | [[BEYOND_CORP]] §11.3 |
| **Hash-chain rolling (M2)** | Detecção de tampering | [[BEYOND_CORP]] §11.4 |
| **Correlation ID ponta-a-ponta** | request_id propagado cliente → API → DB → providers | [[BEYOND_CORP]] §2 |

### Gaps

- Gap-SEC-004: hash-chain (M2).
- **Grafana dashboards por BC**: M2.
- **Alertas proativos**: M2 (Sentry release tracking + Grafana threshold).
- **SIEM integration**: M4+ (Datadog ou Elastic).

---

## A10 — Server-Side Request Forgery (SSRF) {#a10}

> "SSRF flaws occur whenever a web application is fetching a remote resource without validating the user-supplied URL."

### Exemplos master-sindico

- Thumbnail upload por URL → **não aplicável** (upload direto Mux, não por URL).
- Webhook de terceiros por URL user-controlled → **não aplicável** (providers fixos: Stripe, Mux, Livekit).
- Avatar fetch por URL → ⚠️ se existir, passar por allowlist.
- PDF template generation por URL → **não aplicável** (PDF gen local).

### Controles

| Controle | Implementação | Ref |
|---|---|---|
| **Allowlist URLs externas** | Apenas providers conhecidos: Stripe, Mux, Livekit, Zitadel, Twilio, SES, FCM, Receita Federal (M2) | [[BEYOND_CORP]] §1 Tenet 1 |
| **Sem user-controlled URL fetch** | Feature proibida por design review | [[../CLAUDE]] §4 |
| **IP allowlist outbound (M3)** | Firewall / security group bloqueia egress arbitrário | ⚠️ PENDÊNCIA M3 |
| **Rejeitar internal IPs em webhook config** | Se algum dia aceitarmos webhook config (ex: Connect Me external webhook), validar IP não é 127/10/172.16/192.168/169.254/fc00:: | [[pentest-checklist]] §SSRF |
| **DNS rebinding protection** | Resolver IP → verificar → conectar pelo IP (não re-resolver) | ⚠️ PENDÊNCIA M3 |
| **Timeout curto em outbound** | 10s default; previne long-running SSRF | — |

### Gaps

- **Egress firewall**: M3 (quando migrar AWS ECS, security group egress allowlist).
- **Outbound HTTP client com validation**: todas as `http.Get` passando por `safeHTTPClient` (M2 refactor).

---

## Relacionados (fora Top 10 mas aplicáveis)

### Business Logic Vulnerabilities

Não numeradas no Top 10 mas críticas em SaaS:

- **Vote stuffing** (voto duplicado, bot) — ver [[threat-model]] §8.
- **Reputation gaming** (farm contracts) — §11.
- **Connect Me fraud** (fake RFP, fake empresa) — §10.
- **Quota bypass** (burla limite plano) — ABAC + DB counter atomic.
- **Trial abuse** (recriar conta pra trial infinito) — CPF/CNPJ único enforce.
- **Race conditions** (TOCTOU A-029, A-030, A-031) — UoW + DB constraints.

### LGPD-specific (fora OWASP, em LGPD dedicado)

Ver [[lgpd]] pra:
- Consent versionado.
- Direito acesso/correção/portabilidade/exclusão.
- Breach notification 72h.
- DPO + DPA + transferência internacional.

### Assembly-specific (fora OWASP)

- Minutes immutability.
- Proxy vote registration.
- Fração ideal weight computation.
Ver [[threat-model]] §8.

---

## Gates CI ligados a OWASP

```yaml
# .github/workflows/security.yml (shape)
jobs:
  sast:
    - gosec -severity=high ./...               # A03, A04, A05
    - eslint-plugin-security (web)              # A03, A07
  dependencies:
    - govulncheck ./...                         # A06
    - npm audit --audit-level=high              # A06
    - osv-scanner -L pubspec.lock               # A06
    - trivy fs .                                # A06
  secrets:
    - gitleaks detect --source .                # A05
  tests:
    - go test -tags=integration                 # A01 cross-tenant
    - go test -tags=pbt internal/shared/middleware  # A01 ABAC
```

---

## Cobertura por OWASP (dashboard check)

Manter uma planilha mental (ou check-list em Fase 5):

- [ ] A01: ABAC ≥ 400 PBT + 100+ cross-tenant tests + audit DENY + step-up M2
- [ ] A02: TLS 1.3 + PKCE + SAQ-A + field-encryption M3
- [ ] A03: sqlc parametrizado + gosec 0 + VO validation + CSP
- [ ] A04: threat-model first + DoR + 5 papéis + ADR
- [ ] A05: Config.Validate + CORS strict + gitleaks + sec headers + distroless M2
- [ ] A06: govulncheck 0 + npm audit 0 + osv-scanner + dep-audit semanal + SBOM M2
- [ ] A07: Zitadel OIDC + PKCE + MFA M2 + 1-device + rate limit + passkeys M3
- [ ] A08: HMAC webhook + UNIQUE + UoW + Saga + hash-chain M2 + cosign M3
- [ ] A09: structured logs + PII mask + audit_log + OTel + Grafana M2 + SIEM M4
- [ ] A10: allowlist URL + sem user-controlled fetch + egress firewall M3

---

## Links

- [[_moc]]
- [[BEYOND_CORP]] — controles detalhados
- [[threat-model]] — STRIDE por BC
- [[cve-process]] — A06 detalhado
- [[lgpd]] — LGPD compliance (A09 + privacidade)
- [[pentest-checklist]] — checklist A01-A10
- [[beyond-corp-references]]
- OWASP oficial: https://owasp.org/Top10/
- [[../13-research/beyond-corp/_destilado]]
