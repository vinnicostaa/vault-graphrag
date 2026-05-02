---
title: MOC — 06-execution
type: moc
tags: [moc, master-sindico, execution, devops]
project: master-sindico
created: 2026-04-22
---

# 06-execution — Zero → Produção

**Como** o projeto flui do primeiro commit até rodar em prod estável.

## Notas canônicas

### Macro
- [[STEPS]] — sequência macro: discovery → onboarding → setup repo → Sprint 0 (base) → Sprint N (feature) → staging → prod → operação → iteração
- [[process-overview]] — o ciclo SDD 5-fase aplicado ao Master Síndico
- [[sdd-gsd-aplicado]] — SDD + GSD (Spec Kit) aplicado ao projeto: cobertura 8/10, gaps G1–G4 (→ [[../../../10-knowledge/methodology/sdd-adoption-gaps|catálogo G-### genérico]])

### Ciclos
- [[development-cycle]] — dev loop local: branch → migration → code → test → push → PR
- [[qa-cycle]] — revisão + dual-check + `sprint-auditor` antes de merge
- [[deploy-cycle]] — Railway atual (docker + GitHub Actions); plano ECS
- [[release-cycle]] — versionamento, release notes, changelog, tag

### Playbooks
- [[playbooks/rollback]] — rollback específico deste projeto (Railway promote + migrations)
- [[playbooks/incident-response]] — detecção → triagem → mitigação → post-mortem
- [[playbooks/hotfix]] — fluxo pra bug crítico em prod
- [[playbooks/new-feature-kickoff]] — primeiro commit → plano → execução → deploy
- [[playbooks/stripe-webhook-local]] — Stripe CLI forward (`whsec_...`)
- [[playbooks/zitadel-dev]] — subir Zitadel v4 local via Traefik
- [[playbooks/db-migration]] — goose up/down, expand-contract, backup

## Fontes

- [[../.claude/skills/task-executor]]
- [[../.claude/skills/sprint-auditor]]
- [[../.claude/skills/dual-validate]]
- [[../.claude/skills/go-review]]
- [[../.claude/skills/sqlc-writer]]
- [[../.claude/skills/migration-writer]]
- [[../client-vault/00 - Visão Geral/Definition of Done]]

## Vizinhos

- [[../_moc]]
- [[../09-testing/_moc]] — QA cycle detalhado
- [[../11-audit/_moc]] — sprint audit
- [[../../../10-knowledge/methodology/_moc]] — metodologias atemporais (SDD, GSD, TDD/BDD)
- [[../../../10-knowledge/methodology/sdd-workflow]]
- [[../../../10-knowledge/methodology/sdd-maturity-checklist]] — checklist de cobertura SDD
- [[../../../10-knowledge/methodology/sdd-adoption-gaps]] — catálogo G-### de gaps típicos
- [[../../../30-agents/playbooks/_moc]]
