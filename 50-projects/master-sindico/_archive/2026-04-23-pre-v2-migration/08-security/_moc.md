---
title: MOC — 08-security
type: moc
tags: [moc, master-sindico, security, beyond-corp, owasp]
project: master-sindico
created: 2026-04-22
---

# 08-security — Segurança (BeyondCorp adaptado)

**Security posture** deste projeto — multi-tenant, zero-trust, ABAC, LGPD.

## Notas canônicas

### Postura
- [[BEYOND_CORP]] — BeyondCorp adaptado: stack middleware (ordem rígida), ABAC matrix, 1-device, PKCE, tenant isolation
- [[threat-model]] — STRIDE + LINDDUN; superfícies: web, mobile, webhooks, admin; assets: PII, credenciais, fundos via Stripe
- [[attack-surface-map]] — endpoints públicos vs autenticados; inventário de superfície
- [[auth-authz-matrix]] — matriz role × action × scope × effect

### Processos
- [[cve-process]] — SLAs específicos deste projeto (crítica 12h, high 5d, medium 30d); integrado com [[../../../30-agents/playbooks/cve-scan]]
- [[dependency-audit]] — govulncheck + npm audit + osv-scanner + trivy em CI; alignment com [[../../../30-agents/playbooks/dependency-audit]]
- [[secret-management]] — `.env` gitignored, zitadel-key*.json protegido; vault AWS para prod; rotação trimestral
- [[incident-response]] — detecção, triagem, mitigação, postmortem, LGPD notificação
- [[pen-test-schedule]] — interno semestral; externo anual

### Compliance
- [[owasp-top10]] — A01..A10 aplicados e controles existentes
- [[lgpd-compliance]] — consentimento, direito de exclusão, audit trail, DPO, retenção, transferência internacional
- [[pci-scope]] — Master Síndico é merchant Stripe (SAQ A); não armazena PAN; scope PCI mínimo
- [[data-retention]] — políticas por tipo de dado (auth, billing, content, assembly)

### Hardening por camada
- [[hardening-frontend]] — CSP, SRI, SameSite Lax, X-Frame-Options deny
- [[hardening-api]] — rate limit, input validation, output encoding, CORS strict
- [[hardening-db]] — connections via pool, prepared statements (sqlc), privilege minimum, backup cifrado
- [[hardening-infra]] — Railway settings, NAT, firewall; plano ECS WAF + GuardDuty
- [[hardening-mobile]] — certificate pinning, biometria opcional, secure storage, SSL pinning

## Fontes

- [[../.kiro/steering/security-principles]] *(backend)* — princípios de segurança originais
- [[../../../10-knowledge/security/_moc]]
- [[../../../10-knowledge/security/security-principles]]
- [[../11-audit/AUDIT]] — findings de segurança históricos (A-018, A-022, A-023, A-027..A-034)
- [[../STATE]] — D-### de segurança

## Vizinhos

- [[../_moc]]
- [[../02-architecture/_moc]]
- [[../07-quality/_moc]]
- [[../09-testing/_moc]] — security testing (SAST, DAST, SCA)
- [[../11-audit/_moc]]
