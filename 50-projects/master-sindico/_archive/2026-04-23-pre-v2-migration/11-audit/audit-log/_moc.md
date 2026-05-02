---
title: MOC — audit-log
type: moc
tags: [moc, master-sindico, audit-log]
project: master-sindico
created: 2026-04-22
---

# audit-log — Histórico datado de auditorias

Snapshots em Markdown datados (`YYYY-MM-DD-<tipo>.md`) de auditorias feitas no projeto.

Tipos:
- `dep-audit` — resultado do [[../../../../30-agents/playbooks/dependency-audit]]
- `security-review` — revisão de superfície + ABAC
- `sprint-auditor` — saída do skill `sprint-auditor`
- `code-review` — revisão manual de código
- `postmortem` — post-mortem de incidente

Formato:

```yaml
---
title: <tipo> — YYYY-MM-DD
type: audit-log
tags: [audit, <tipo>, sprint-<N>]
date: YYYY-MM-DD
---
```

Corpo: relatório estruturado (contexto, findings, action items, owner, prazo).

## Vizinhos

- [[../_moc]]
- [[../AUDIT]]
