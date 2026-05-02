---
title: MOC — 08-security
type: moc
tags: [moc, master-sindico, v2, security, beyond-corp, owasp, lgpd]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 08-security

Pilar de segurança da remontagem (Fase 3). Destilação do research BeyondCorp/Zero-Trust + legado vivo + threat-model STRIDE + OWASP + LGPD + pentest checklist. **Corpo PT-BR / identifiers EN** conforme [[../CLAUDE]] §6 + [[../STEERING]].

**Base conceitual**: BeyondCorp adaptado é **core do produto**, não add-on. Governança condominial exige auditabilidade forte por lei (LGPD + disputa judicial de ata).

**Herança principal**: [[../90-ingested/from-vault-50-projects-master-sindico/08-security/BEYOND_CORP|BEYOND_CORP legado]] (v1 Sprint 10) + research destilado em [[../13-research/beyond-corp/_destilado]].

---

## Arquivos

### Pilar central

- [[BEYOND_CORP]] — **pilar central** BeyondCorp aplicado: 7 tenets NIST traduzidos; stack middleware ordenado; ABAC matrix; 1-device; PKCE; rate limit; PII mask; HMAC webhook; CORS; secrets; audit trail; pilares herdados do research (device posture, PE/PA/PEP, continuous auth, mTLS, JIT/JEA, SPIFFE); mapping pilar × BC; roadmap M1/M2/M3.

### Ameaças e OWASP

- [[threat-model]] — **STRIDE por BC** + LINDDUN (privacidade) + Tenant boundary threat model + Provider compromise (Stripe/Mux/Zitadel key leak) + Webhook tampering + Assembly vote integrity + Video moderation bypass + Connect Me fraud + Reputation gaming + regras PBT + Gaps esperados Fase 5 (Gap-SEC-### → A-SEC-###).

- [[owasp-top10]] — mapping OWASP Top 10 2021 completo → controles master-sindico. Cada A01-A10 com controles implementados + gaps + sprints. Gates CI por categoria.

### Vulnerabilidades e LGPD

- [[cve-process]] — SLA por severidade (CRITICAL 12h, HIGH 5d, MEDIUM 30d, LOW 90d); ferramentas por stack (govulncheck, gosec, npm audit, osv-scanner, trivy); workflow detect → triage → patch → verify → ship → postmortem; CVE em transitive deps; gates CI.

- [[lgpd]] — Lei 13.709/2018 integral; bases legais; data classification; consent versionado; direitos do titular (acesso, correção, portabilidade, exclusão SLA 15d workflow); revisão decisão automatizada (Art. 20); lgpd_logs; breach notification 72h ANPD; DPO; DPA com terceiros; transferência internacional; retention policy; DPIA; menores.

### Processo e referências

- [[pentest-checklist]] — **NOVO** — checklist exaustivo pro papel pentester do protocolo 5-papéis. 20 categorias (auth, authz, input, upload, rate, webhook, multi-tenant, biz logic, secrets, logs, deps, infra, API, SSRF, client-side, LGPD, IR). Template A-SEC-### pra findings.

- [[beyond-corp-references]] — **NOVO** — ponte com 13-research. Catálogo das 12 fontes canônicas (NIST, BeyondCorp I/II/III/IV, BeyondProd, CISA ZTMM, IAP, AWS, MS, Cloudflare, Tailscale, Okta) com URLs + data de acesso + trechos literais + aplicação. Dual-check discipline. Pendências ADR.

---

## Matriz pilar × arquivo (rapid-access)

| Pilar | Arquivo principal | Arquivo secundário |
|---|---|---|
| Zero-trust sem rede confiável | [[BEYOND_CORP]] §1 | [[beyond-corp-references]] Fonte 01-02 |
| ABAC matrix + cache | [[BEYOND_CORP]] §3 | [[owasp-top10]] A01 |
| 1-device + fingerprint | [[BEYOND_CORP]] §4 | [[pentest-checklist]] §1.3 |
| PKCE + cookies | [[BEYOND_CORP]] §5 | [[pentest-checklist]] §1.1 |
| Rate limiting escalonado | [[BEYOND_CORP]] §6 | [[threat-model]] §4 |
| PII masking | [[BEYOND_CORP]] §7 | [[lgpd]] §2 + [[pentest-checklist]] §10.1 |
| HMAC webhook | [[BEYOND_CORP]] §8 | [[threat-model]] §7 + [[pentest-checklist]] §6 |
| CORS allowlist | [[BEYOND_CORP]] §9 | [[owasp-top10]] A05 |
| Secrets gitignored | [[BEYOND_CORP]] §10 | [[cve-process]] §8 |
| Audit trail LGPD 5y | [[BEYOND_CORP]] §11 | [[lgpd]] §5 |
| Device posture (M2) | [[BEYOND_CORP]] §4.3 + §12.1 | [[beyond-corp-references]] Fonte 12 |
| PE/PA/PEP (OPA/Cedar) | [[BEYOND_CORP]] §12.2 | [[beyond-corp-references]] Fonte 02 |
| Continuous auth + risk scoring | [[BEYOND_CORP]] §12.3 | [[pentest-checklist]] §1 |
| mTLS leste-oeste | [[BEYOND_CORP]] §12.6 | [[beyond-corp-references]] Fonte 05 (BeyondProd) |
| SPIFFE/SPIRE (M3+) | [[BEYOND_CORP]] §12.5 | [[beyond-corp-references]] Fonte 05 |
| JIT/JEA admin access | [[BEYOND_CORP]] §12.7 | [[beyond-corp-references]] Fontes 10, 11 |
| OWASP Top 10 | [[owasp-top10]] | [[threat-model]] |
| LGPD concreto | [[lgpd]] | [[threat-model]] §12 LINDDUN |
| CVE workflow + SLA | [[cve-process]] | [[owasp-top10]] A06 |
| Pentester role checklist | [[pentest-checklist]] | [[threat-model]] gaps §15 |

---

## Gaps conhecidos → A-SEC-### esperados (Fase 5)

Consolidado de [[threat-model]] §15:

| Gap | Descrição | Mitigação planejada |
|---|---|---|
| Gap-SEC-001 | Device posture real (Play Integrity/DeviceCheck) ausente | M2 |
| Gap-SEC-002 | Step-up MFA ausente em endpoints críticos | M2 |
| Gap-SEC-003 | PostgreSQL RLS ausente | M2 |
| Gap-SEC-004 | Hash-chain no audit_log ausente | M2 |
| Gap-SEC-005 | Rate limit distribuído (Redis) ausente | M2 |
| Gap-SEC-006 | WAF edge não decidido | M2 ADR |
| Gap-SEC-007 | CNPJ validação Receita Federal ausente | M2 |
| Gap-SEC-008 | Reputation decay ausente | M3 |
| Gap-SEC-009 | Dual-approval moderation ausente | M3 |
| Gap-SEC-010 | Reputation ring detection ausente | M4 |
| Gap-SEC-011 | Mux antivirus scan ausente | M4+ |
| Gap-SEC-012 | PDP sidecar (OPA) ausente | M2 ADR |
| Gap-SEC-013 | mTLS leste-oeste ausente | M2 (3 caminhos) / M3 (mesh) |
| Gap-SEC-014 | Continuous auth / risk scoring ausente | M3 |
| Gap-SEC-015 | AWS Secrets Manager ausente | M2 |

Herdados legado Sprint 10 (fechar antes da migração Fase 6):
- A-023 device change audit comparativo.
- DT-010 IAuditLogger cross-módulo.
- A-029, A-030, A-031 TOCTOU.
- A-032 goroutines cleanup.
- A-033, A-034 Livekit retry.

---

## ⚠️ PENDÊNCIAS ADR (decisões abertas)

Ver [[beyond-corp-references]] §Pendências + [[../STATE]]:

- ADR-### IdP final (Zitadel continua vs Cognito vs Keycloak vs Auth0 vs Entra ID).
- ADR-### PDP (OPA vs Cedar).
- ADR-### Service mesh M1 ou M2? Qual?
- ADR-### Edge/WAF (Cloudflare vs AWS WAF).
- ADR-### Audit storage (S3 Object Lock vs TimescaleDB + hash-chain).
- ADR-### Cloud final (Railway vs AWS ECS vs GCP vs multi-cloud) — D-024 legado.
- ADR-### Admin access (Cloudflare Access vs Tailscale vs AWS Verified Access).
- ADR-### Email provider (SES vs Resend) — D-026 legado.
- DPO nomeação (nome público + vínculo).

---

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente (§9 security posture)
- [[../STEERING]] — não-negociáveis (§princípios-de-segurança 15-24)
- [[../STATE]] — decisões vivas
- [[../AUDIT]] — findings A-### + herdados Sprint 10
- [[../13-research/beyond-corp/_destilado]] — research destilado
- [[../13-research/beyond-corp/_moc|beyond-corp research MOC]]
- [[../90-ingested/from-vault-50-projects-master-sindico/08-security/BEYOND_CORP|legado BEYOND_CORP v1]]
- [[../04-requirements/_moc]] — (a escrever) requisitos funcionais com thread de segurança
- [[../06-execution/playbooks/_moc]] — (a escrever) playbooks incident response / dependency audit / cve-scan
