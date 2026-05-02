---
title: MOC — 11-audit
type: moc
tags: [moc, master-sindico, audit]
project: master-sindico
created: 2026-04-22
---

# 11-audit — Auditorias e findings

Findings **descobertos em revisão** (A-###) + audit logs históricos + registros de dual-check.

## Notas canônicas

- [[AUDIT]] — arquivo vivo de findings A-### (canônico)
- [[audit-log/_moc]] — histórico datado de auditorias
- [[dual-check-log]] — registro de dual-checks executados
- [[sprint-auditor-log]] — resultado de cada sprint-auditor
- [[postmortems]] — incident post-mortems

## Status das seções de AUDIT

Ver [[AUDIT]] seções:
- 🔴 Abertos (bloqueiam sprint novo)
- 🟡 Em andamento
- ✅ Resolvidos nesta rodada
- 📜 Histórico

## Regras

1. **task-executor** abre AUDIT no início de toda sessão
2. 🔴🟡 **bloqueiam** task nova; resolver primeiro
3. **sprint-auditor** materializa findings ao fim de cada sprint
4. Nunca reutilizar ID A-### (sequencial, imutável)
5. AUDIT vs STATE:
   - AUDIT = bugs e má implementação **descobertos em revisão**
   - STATE = decisões e dívidas **conscientes** (com sprint alvo)

## Fontes

- [[../AUDIT]] — **arquivo original na raiz do projeto; mover/sincronizar para 11-audit/AUDIT.md**
- [[../.claude/skills/sprint-auditor]]

## Vizinhos

- [[../_moc]]
- [[../STATE]] — decisões vivas (D-###)
- [[../SESSION_CHARTER]] — sessão atual
- [[../08-security/_moc]] — findings de segurança alimentam AUDIT
- [[../07-quality/_moc]] — findings de qualidade alimentam AUDIT
- [[../../../30-agents/playbooks/dual-check]]
