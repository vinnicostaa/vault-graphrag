---
title: Compliance — requisitos funcionais (stub)
type: requirement
tags:
  - requirement
  - master-sindico
  - compliance
  - stub
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# Compliance — requisitos funcionais

> **Status**: 🟡 stub criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Requisitos detalhados estão dispersos em [[../../08-security/lgpd]], [[../../08-security/BEYOND_CORP]] e [[../../08-security/cve-process]]; a destilação aqui formaliza-se quando o sub-domínio for promovido a doc-requirement próprio.

Sub-domínio "compliance" do BC `cross-domain` — cobre **LGPD**, **NR-1**, **BeyondCorp posture**, **trilha de auditoria** e **bloqueios regulatórios** (compliance_status para mandato, dossier exportado, score de compliance).

**Contexto canônico**:
- LGPD operacional → [[../../08-security/lgpd]] + [[../../02-architecture/adr/0028-lgpd-keyed-hmac|ADR-0028]] + [[../../02-architecture/adr/0037-soft-delete-universal-pseudonymize|ADR-0037]]
- NR-1 (saúde mental no trabalho) → [[../../08-security/lgpd]] §NR-1 e [[institutional]] §governance markers
- BeyondCorp → [[../../08-security/BEYOND_CORP]] e [[../../02-architecture/adr/0016-beyondcorp-security-posture|ADR-0016]]
- Trilha de auditoria → [[../../01-domain/aggregates/AuditEntry]]
- Bloqueios pré-export dossier → [[institutional]] §Compliance gate

---

## REQs (a destilar)

### REQ-COMP-LGPD
LGPD compliance: consentimento, direitos do titular, retenção 5y, pseudonimização HMAC keyed, audit trail. Ver [[../../08-security/lgpd]].

### REQ-COMP-NR-1
NR-1 (NR de saúde mental): controles + indicadores. Ver [[../../08-security/lgpd]] §NR-1 e [[institutional]] §Governance markers.

### REQ-COMP-AUDIT-TRAIL
Trilha de auditoria append-only para ações relevantes (admin actions, mudanças de role, export de dados). Ver [[../../01-domain/aggregates/AuditEntry]] e [[../../10-knowledge/observability/audit-trail|knowledge global de audit-trail]].

### REQ-COMP-DOSSIER-EXPORT
Export de dossier do mandato (PDF assinado) só permitido se `compliance_status = em_dia`. Ver [[institutional]] §Dossier Export.

### REQ-COMP-SCORE
Compliance score (0-100) calculado a partir de checklist regulatório. Ver [[institutional]].

---

## Links

- [[_moc]]
- [[institutional]]
- [[cross-domain]]
- [[../../08-security/_moc]]
- [[../../STATE]]
