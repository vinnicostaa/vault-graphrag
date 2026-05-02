---
title: Security tests — Master Síndico
type: guide
tags: [testing, security, sast, dast, gosec, govulncheck, owasp, master-sindico, v2]
source: 08-security/owasp-top10.md + 08-security/BEYOND_CORP.md + 13-research
created: 2026-04-23
updated: 2026-04-23
aliases: [security-tests-master-sindico]
---

# Security tests — Master Síndico v2

Testes de segurança automatizados em CI + revisões manuais periódicas. Cobre SAST (`gosec`), SCA/CVE (`govulncheck`), DAST (OWASP ZAP baseline em M3+), pen-test checklist derivado de [[../08-security/owasp-top10]] e [[../08-security/threat-model]].

> Herda [[../CLAUDE]] §9 (security posture BeyondCorp) e [[../STEERING]] §§15-24.

---

## 1. Pirâmide de security testing

```
              MANUAL PEN-TEST (trimestral / pré-GA)
            DAST (OWASP ZAP baseline — M3+ stretch)
          SAST (gosec + Semgrep custom)
        SCA (govulncheck + osv-scanner)
      SECRET SCAN (trufflehog / gh secret scanning)
    DEP AUDIT (go mod audit + deps diff review)
  CONTRACT / UNIT TESTS DE SEGURANÇA (auth, ABAC, tenant)
BASE: controles no código (HMAC, sanitização, ABAC default-deny)
```

---

## 2. SAST — `gosec`

Static Application Security Testing em CI. **Falha em HIGH**.

```bash
go install github.com/securego/gosec/v2/cmd/gosec@latest
gosec -severity=high -exit-status ./...
```

### Regras importantes (não ignorar):

| Rule | Descrição |
|---|---|
| G101 | Hardcoded credentials |
| G201-G203 | SQL injection (sqlc mitiga, mas adapters inline) |
| G301-G306 | File/path traversal |
| G401, G501-G505 | Cryptografia fraca (DES, MD5, SHA1) |
| G107 | URL user-supplied em HTTP request |
| G108 | Profiling endpoint exposto |
| G109 | Integer overflow (int → int32/uint) |

### Regras custom Semgrep

```yaml
# .semgrep/rules/master-sindico.yml
rules:
  - id: pii-in-log
    pattern-either:
      - pattern: logger.$LVL().Str("cpf", $X).Msg(...)
      - pattern: logger.$LVL().Str("email", $X).Msg(...)
    message: "PII em log sem Masked() — AP-007"
    severity: ERROR

  - id: select-star-sqlc
    pattern: SELECT * FROM $TABLE
    paths:
      include: ['**/queries/*.sql']
    message: "SELECT * proibido (AP-016, A-021)"
    severity: ERROR

  - id: webhook-parse-before-hmac
    pattern-inside: |
      func $HANDLER($CTX $T) $RET {
        ...
        json.Unmarshal($BODY, ...)
        ...
        verifyHMAC(...)
        ...
      }
    message: "HMAC deve ser antes do parse (AP-023, INV-018)"
    severity: ERROR
```

---

## 3. SCA — `govulncheck` + `osv-scanner`

Detecta CVEs em dependências + módulos stdlib.

```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...
```

**Gate**: 0 CVEs `reachable` (o módulo vulnerável é de fato chamado pelo código). CVEs `imports-only` geram ticket `DT-###` com prazo 30d.

Alternativa/complemento: `osv-scanner` cobre também `go.sum` e lockfiles de outros stacks (bun, pub).

---

## 4. Secret scanning

- **GitHub**: habilitar secret scanning push protection no repo.
- **CI**: `trufflehog` em cada PR:

```yaml
- uses: trufflesecurity/trufflehog@main
  with:
    base: ${{ github.event.repository.default_branch }}
    head: HEAD
    extra_args: --only-verified
```

- **Pré-commit hook local**: `gitleaks protect --staged --verbose`.
- `.env`, `zitadel-key*.json`, `*.pem`, `*.p12` no `.gitignore` (AP-018 fix).

---

## 5. DAST — OWASP ZAP baseline (M3+ stretch)

DAST dinâmico só faz sentido após observability madura.

```yaml
# .github/workflows/dast.yml (scheduled weekly staging)
- uses: zaproxy/action-baseline@v0.13.0
  with:
    target: 'https://staging.mastersindico.com.br'
    rules_file_name: '.zap/rules.tsv'
    cmd_options: '-a'
```

Regras baseadas em [[../08-security/owasp-top10]]: CSP, HSTS, XSS reflected, clickjacking, mixed content, cookie flags.

---

## 6. Testes de segurança como código

### Auth flow (unit + integration)

```go
func TestAuthCallback_when_invalidPKCEVerifier_should_reject(t *testing.T) {
    // INV-018 variante PKCE (§8-security/BEYOND_CORP)
    app := bootstrapTestApp(t)
    code := "fake_code"
    resp := app.Post("/auth/callback", map[string]string{"code": code, "code_verifier": "wrong"})
    require.Equal(t, 401, resp.Code)
}
```

### Webhook HMAC (integration)

```go
func TestWebhookStripe_when_invalidSignature_should_return400BeforeParse(t *testing.T) {
    // INV-093 + AP-023
    app := bootstrapTestApp(t)
    body := fixture("stripe_invoice_paid.json")
    resp := app.Post("/webhooks/stripe", body, headers{"Stripe-Signature": "invalid"})
    require.Equal(t, 400, resp.Code)

    // Verifica que DB NÃO recebeu o evento
    count, _ := app.DB.QueryRow("SELECT count(*) FROM webhook_events").Scan()
    require.Equal(t, 0, count)
}
```

### Tenant isolation (integration — ≥ 100 casos)

Ver [[integration]] §2 "Cross-tenant isolation".

### ABAC policy (unit + PBT)

Ver [[pbt]] — 400 casos rapid pra matriz `admin_bypass → role → action → tenant → scope`.

### Rate limit (integration)

```go
func TestRateLimitAuth_when_burstExceeded_should_return429(t *testing.T) {
    // INV-097, BR-062
    app := bootstrapTestApp(t)
    for i := 0; i < 30; i++ {
        resp := app.Post("/auth/callback", map[string]string{"code": "x"})
        if i < 20 { require.NotEqual(t, 429, resp.Code) }
        if i > 25 { require.Equal(t, 429, resp.Code) }
    }
}
```

### PII masking (unit)

```go
func TestCPF_Masked(t *testing.T) {
    cpf, _ := NewCPF("123.456.789-09")
    require.Equal(t, "***.***.789-09", cpf.Masked())
}
```

### 1-device invalidation (integration — race)

```go
func TestLogin_when_secondDevice_should_invalidatePreviousSession(t *testing.T) {
    // INV-005, BR-004, A-023 race
    app := bootstrapTestApp(t)
    t1 := app.Login(user, device1)
    app.Login(user, device2)

    me := app.Get("/api/v1/me", headers{"Authorization": "Bearer " + t1})
    require.Equal(t, 401, me.Code) // sessão anterior invalidada
}
```

---

## 7. Pen-test checklist (trimestral / pré-GA)

Derivado de [[../08-security/owasp-top10]] e [[../08-security/threat-model]]. Checklist executado manual por pentester (interno ou contratado). Cada item tem A-### em [[../11-audit/AUDIT]] se vulnerabilidade encontrada.

### OWASP Top 10 2021

- [ ] **A01 Broken Access Control**
  - [ ] IDOR: tenant X acessa recurso tenant Y (100 combinações).
  - [ ] Escalação vertical: morador → síndico.
  - [ ] Escalação horizontal: síndico A manda comando em condomínio B.
  - [ ] Force browsing rotas admin sem role.
- [ ] **A02 Cryptographic Failures**
  - [ ] Senhas só no Zitadel (Argon2); app nunca armazena hash.
  - [ ] Cookies Secure + httpOnly + SameSite=Lax.
  - [ ] TLS 1.2+ enforcing; HSTS max-age ≥ 31536000.
- [ ] **A03 Injection**
  - [ ] SQLi via params (sqlc mitiga, mas adapters LIKE).
  - [ ] NoSQLi (Redis keys concatenados sem sanitização).
  - [ ] Command injection em integração OS calls.
  - [ ] XSS refletido/stored em campos user-submitted.
- [ ] **A04 Insecure Design**
  - [ ] Threat model STRIDE atualizado por feature nova.
  - [ ] Revisão de invariantes (INV-### bateram com código).
- [ ] **A05 Security Misconfiguration**
  - [ ] Default secrets ausentes.
  - [ ] CORS allowlist restrita (sem wildcard em prod — A-005).
  - [ ] Headers de segurança (CSP, X-Frame, X-Content-Type).
  - [ ] Debug mode off em prod.
- [ ] **A06 Vulnerable Components**
  - [ ] `govulncheck` 0 reachable.
  - [ ] Deps críticas atualizadas (ver [[../08-security/cve-process]]).
- [ ] **A07 Auth & Identity Failures**
  - [ ] PKCE obrigatório; verify matches.
  - [ ] 1-device enforcement (BR-004).
  - [ ] Rate limit `/auth/*` 20rpm burst 5.
  - [ ] Session rotation after privilege change.
- [ ] **A08 Software & Data Integrity**
  - [ ] SRI em scripts externos (CDN).
  - [ ] Imagens Docker signed (cosign).
  - [ ] Webhook HMAC antes de parse (INV-093).
- [ ] **A09 Logging & Monitoring**
  - [ ] Audit trail append-only (INV-090).
  - [ ] PII em logs: 0 (INV-092, grep CI).
  - [ ] Alerting em auth failures sustained, 5xx rate.
- [ ] **A10 SSRF**
  - [ ] Webhook out (client notifications): allowlist por host.
  - [ ] Upload de imagens: não permite URL remota.
  - [ ] Fetch de OpenGraph metadata: timeout + allowlist.

### LGPD-specific

- [ ] Direito de exclusão SLA 15d funciona (BR-053).
- [ ] Direito de portabilidade `/me/export` produz JSON completo.
- [ ] Consent versionado; aceite rastreado.
- [ ] LGPD logs 5 anos append-only.
- [ ] PII masking em dashboards Grafana.

### Multi-tenant

- [ ] 100+ cross-tenant tests passando.
- [ ] ABAC default-deny confirmado (INV-087).
- [ ] Admin bypass só com audit log (INV-088).

---

## 8. CI pipeline security

```yaml
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.26' }
      - name: Secret scan
        uses: trufflesecurity/trufflehog@main
      - name: SAST gosec
        run: |
          go install github.com/securego/gosec/v2/cmd/gosec@latest
          gosec -severity=high -exit-status ./...
      - name: Semgrep custom
        uses: returntocorp/semgrep-action@v1
        with: { config: '.semgrep/rules/' }
      - name: SCA govulncheck
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...
```

---

## 9. CVE process

Processo dedicado em [[../08-security/cve-process]]. Resumo:

1. Alerta govulncheck → `A-### 🔴` em [[../11-audit/AUDIT]].
2. Triagem: reachable? severity? affected module?
3. Reachable HIGH/CRITICAL → bloqueia deploy; patch imediato.
4. Imports-only MEDIUM → DT-### com prazo 30d.
5. Post-mortem se CVE ficou reachable em prod.

---

## 10. Anti-patterns security testing

| Sintoma | Fix |
|---|---|
| Testes de segurança só no final | CI em cada PR |
| Gosec com `-exclude=...` sem justificativa | Cada exclusão tem ticket + prazo |
| DAST em prod sem coordenação | DAST só em staging/canário |
| Pen-test anual apenas | Trimestral + pré-GA + pós-feature crítica |
| Secret scan só no default branch | Push protection + pre-commit hook |

---

## 11. Checklist review security

- [ ] `gosec -severity=high` 0 findings?
- [ ] `govulncheck` 0 reachable CVEs?
- [ ] Semgrep custom rules passam?
- [ ] PII masking testado em unit?
- [ ] HMAC antes de parse verificado em contract test?
- [ ] Cross-tenant isolation 100+ casos passando?
- [ ] ABAC default-deny coberto em PBT?
- [ ] Rate limit testado sob load?
- [ ] Secret scan no CI?
- [ ] Checklist pen-test atualizado com novos endpoints?

---

## 12. Links

- [[_moc]]
- [[test-strategy]]
- [[integration]]
- [[pbt]]
- [[load]]
- [[../08-security/BEYOND_CORP]]
- [[../08-security/owasp-top10]]
- [[../08-security/threat-model]]
- [[../08-security/cve-process]]
- [[../11-audit/AUDIT]]
- [[../07-quality/antipatterns]]
