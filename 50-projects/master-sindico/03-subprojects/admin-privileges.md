---
title: Admin Privileges (transversal)
type: spec
tags:
  - master-sindico
  - admin
  - abac
  - ms-internal
  - stub
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# Admin Privileges — documento transversal

> 🟡 **Stub** criado em 2026-04-27 (per STATE D-103/D-072). Documento transversal descrevendo permissões da role **Admin Master Síndico** nas apps existentes (não é sub-projeto independente).

## Persona

**Admin Master Síndico** — equipe interna do Master Síndico (não cliente). Provisionada manualmente no Zitadel (sem fluxo auto-registro). Visibilidade transversal a todos os tenants.

## Acessos

### CMS (web — `app.mastersindico.com.br`)
- Read-only em todas as views de síndico (auditoria)
- Override em casos específicos (suspensão de mandato, freeze de conta, etc.)

### Admin (web — futuro `admin.mastersindico.com.br`)
- Painel `/admin` exclusivo
- Gestão de planos globais
- Provisão de Empresas Síndicas
- Quota override
- Audit trail viewer
- Compliance dashboard

### Mobile
- Acesso de leitura via flag (sem UI principal)

## ABAC matriz

A role `admin_ms` em [[../01-domain/aggregates/Membership]] (com `tenant_id = MASTER_TENANT`) tem permissões transversais. Detalhes em [[../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]].

## Audit obrigatório

Toda ação de `admin_ms` em tenant de cliente gera entrada em [[../01-domain/aggregates/AuditEntry]] com `admin_reason` (motivo declarado obrigatório — A-DC-PEN-015).

## Links

- [[../00-product/personas]]
- [[../00-product/portfolio-de-produtos]]
- [[../01-domain/aggregates/Membership]]
- [[../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]]
- [[../STATE]] §D-072
