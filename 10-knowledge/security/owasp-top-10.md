---
title: OWASP Top 10
type: concept
tags: [security, owasp, web-security, threats]
source: https://owasp.org/Top10/
created: 2026-04-24
updated: 2026-04-24
aliases: [OWASP Top 10 2021, Top 10 web risks]
---

# OWASP Top 10 (2021)

Lista consensual das **10 categorias de risco** mais críticas para aplicações web, mantida pela OWASP Foundation. Base mínima de threat modeling — qualquer sistema exposto à internet deve cobrir os 10.

## A01 — Broken Access Control

**Descrição**: usuário consegue agir fora do que sua role/ownership permite. Promovido a #1 em 2021 por ser o mais prevalente em testes.

**Exemplo**: endpoint `GET /invoices/{id}` checa auth (token válido) mas não autorização (esse token pertence ao dono da invoice?). Atacante troca `id` na URL → IDOR.

```go
// vulnerável
invoice := repo.FindByID(ctx, id)
return invoice
// correto
if invoice.TenantID != auth.TenantID { return ErrForbidden }
```

**Mitigação**: ABAC/RBAC consistente, deny-by-default, ownership check em toda query. [[beyond-corp]] aplica aqui.

## A02 — Cryptographic Failures

**Descrição**: dados sensíveis em trânsito ou repouso sem proteção adequada. Inclui TLS fraco, hash de senha inadequado (MD5, SHA1), chave hardcoded.

**Exemplo**: armazenar senha com `sha256(password)` sem salt nem KDF. Rainbow table resolve em minutos.

**Mitigação**: argon2id ou bcrypt para senhas; AES-GCM ou ChaCha20-Poly1305 para AEAD; TLS 1.3; rotação de chave via KMS; nunca logar PII/secret.

## A03 — Injection

**Descrição**: input do usuário interpretado como código/query. SQLi, NoSQLi, LDAP, command injection, template injection.

**Exemplo**:

```go
// vulnerável
db.Query("SELECT * FROM users WHERE email = '" + email + "'")
// correto
db.Query("SELECT * FROM users WHERE email = $1", email)
```

**Mitigação**: prepared statements/parametrized queries; ORMs que escapam por default; validação de schema (não blacklist); princípio do menor privilégio no DB user.

## A04 — Insecure Design

**Descrição**: falha **arquitetural**, não de implementação. Ex: fluxo de "esqueci senha" sem rate limit, recuperação via pergunta de segurança trivial.

**Exemplo**: endpoint de transferência entre contas sem limite diário nem 2FA em valor alto.

**Mitigação**: threat modeling no design (STRIDE, LINDDUN); secure-by-design patterns (rate limit, idempotency, step-up auth para operações sensíveis); abuse cases no requisito.

## A05 — Security Misconfiguration

**Descrição**: default inseguro, painel admin exposto, verbose errors em produção, CORS `*`, header de segurança ausente.

**Exemplo**: Elasticsearch com porta 9200 exposta sem auth; bucket S3 público; `DEBUG=true` em prod vazando stack trace.

**Mitigação**: hardening checklist por stack; IaC review; secret scanning; scan de portas; headers CSP, HSTS, X-Content-Type-Options, Referrer-Policy. [[../principles/solid]] SRP aplica: config separada de código.

## A06 — Vulnerable and Outdated Components

**Descrição**: lib/runtime com CVE conhecido.

**Exemplo**: Log4j 2.14 (CVE-2021-44228 / Log4Shell), libs com RCE públicos sem patch.

**Mitigação**: inventário de dependências (SBOM); dependabot/renovate; [[cve-response-workflow]]; pin de versão + reprodutibilidade de build; cut supply chain (Sigstore, SLSA).

## A07 — Identification and Authentication Failures

**Descrição**: auth fraca — senha curta, brute force sem lockout, sessão eterna, reuso de token.

**Exemplo**: cookie de sessão sem `HttpOnly` + `Secure` + `SameSite`; logout não invalida token server-side; JWT sem `exp`.

**Mitigação**: senha forte + MFA; lockout exponencial; session store server-side com revogação; JWT curto + refresh rotativo; OIDC/SSO sempre que possível.

## A08 — Software and Data Integrity Failures

**Descrição**: confiança em código/dados sem verificação. CI/CD comprometido, deserialização insegura, update sem assinatura.

**Exemplo**: `pickle.loads(user_input)` em Python; pipeline que puxa action do GitHub sem pin de SHA; deploy sem checksum.

**Mitigação**: assinatura de artefato (cosign, Sigstore); pin de SHA em action/imagem; evitar deserialização nativa pra input não-confiável; SLSA nível ≥ 2.

## A09 — Security Logging and Monitoring Failures

**Descrição**: invasão passa despercebida por dias/meses por falta de sinal. Mediana de detecção histórica > 200 dias.

**Exemplo**: endpoint de login sem log de tentativa falha; acesso admin sem audit trail; alerta sem dono.

**Mitigação**: log estruturado com correlation ID; métrica+trace+log (três pilares); alertas em eventos críticos (login admin, mudança de permissão, export de dados); SIEM; retenção adequada; PII **fora** do log.

## A10 — Server-Side Request Forgery (SSRF)

**Descrição**: servidor faz request a URL controlada por atacante, vazando metadata interna (ex: AWS IMDS em `169.254.169.254`) ou pivotando pra rede interna.

**Exemplo**: feature "buscar thumbnail de URL" aceita `http://169.254.169.254/latest/meta-data/iam/...` e retorna credencial IAM.

```go
// mitigação: validar host + bloquear ranges privados
if isPrivateIP(host) || host == "169.254.169.254" { return ErrForbidden }
```

**Mitigação**: allowlist de hosts; resolver DNS server-side e checar IP final; bloquear CIDRs privados (RFC 1918, link-local, loopback); IMDSv2 na AWS; egress firewall.

## Como aplicar

1. Em toda feature nova, mapear quais categorias são relevantes (não são sempre as 10)
2. Adicionar test case por categoria aplicável no suite de segurança
3. Re-review anual — lista é atualizada a cada ~4 anos (próxima esperada 2025/2026)

## Links

- [[_moc]]
- [[../_moc]]
- [[beyond-corp]]
- [[../principles/solid]]
