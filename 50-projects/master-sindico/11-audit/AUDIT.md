---
title: AUDIT — Master Síndico v2 (canônico 11-audit)
type: audit
tags: [audit, findings, master-sindico, v2, canonical]
source: raiz AUDIT.md + herança legado 11-audit/AUDIT.md
created: 2026-04-23
updated: 2026-04-23
---

# AUDIT — Master Síndico v2 (11-audit canônico)

Espelho canônico do [[../AUDIT]] raiz. Usado por skills legadas (`task-executor`, `sprint-auditor`) que esperam path `11-audit/AUDIT.md`.

**Double-write**: atualizar este + raiz simultaneamente até skills serem refactoradas pra apontar pra este path.

> **Semântica**: 🔴 crítico (bloqueia task nova) · 🟡 relevante (acomodar antes da migração) · 🟢 resolvido.

---

## Regras

1. **AUDIT vs STATE**:
   - AUDIT = findings **descobertos em revisão** (A-###)
   - STATE = decisões (D-###) e dívidas **conscientes** (DT-###)
2. 🔴🟡 aberto **bloqueia** task nova do sprint seguinte
3. `task-executor` abre este arquivo no início de toda sessão
4. `sprint-auditor` roda ao fim de cada sprint e materializa findings
5. ID sequencial (A-###) nunca reutilizado
6. **Dual-check** obrigatório antes de marcar 🟢 em finding 🔴

---

## Seções obrigatórias

- 🔴 Itens Abertos
- 🟡 Em Andamento (acomodar antes da migração)
- ✅ Resolvidos nesta rodada
- 📜 Histórico

---

## 🔴 Itens Abertos

**Nenhum 🔴 no momento.** Últimos 🔴 (A-025, A-026, A-027, A-028, A-019, A-022) foram resolvidos no legado Sprint 9.

---

## 🟡 Em andamento — herdados do legado

Acomodar antes da migração (Fase 6 do plano). Todos já mapeados em [[../05-tasks/backlog#audit-findings--tasks]].

| ID | Título | Sprint | Task derivada |
|---|---|---|---|
| **A-020** | N+1 UPDATE em `master_plan_timeline_handler` | 10 | T-003 |
| **A-021** | 12 `SELECT *` em queries commercial | 10 | T-005 (doc) + T-005a (fix) |
| **A-023** | 1-device change sem audit log comparativo | 10 | T-004 |
| **A-024** | `rawBodyReader` interface privada acopla handlers | 12 | T-811 |
| **A-029** | Company Evaluation TOCTOU | 11 | T-104 |
| **A-030** | Commercial Vote status race (voto pós-close) | 11 | T-105 |
| **A-031** | Event Response TOCTOU (falso positivo) | 12 | T-812 (fechar via dual-check) |
| **A-032** | Goroutines lifecycle explícito (cleanup) | 12 | T-813 |
| **A-033** | Livekit join/ListParticipants retry | 14 | T-304 |
| **A-034** | Livekit EndRoom retry + NotFound | 14 | T-305 |

---

## ✅ Resolvidos na rodada atual

Vazio — ainda não houve sprint sob este AUDIT v2. Próxima sprint (Sprint 10) gerará 🟢 pros fixes.

---

## 📜 Histórico

Findings A-001..A-018 + A-022/A-025/A-026/A-027/A-028/A-019 resolvidos no legado vivo. Ver arquivo herdado em [[../90-ingested/from-vault-50-projects-master-sindico/11-audit/AUDIT]].

Resumo cronológico dos resolvidos:
- **A-001..A-008** (Sprint 1-3): PII logs, error handling, middleware cleanup
- **A-009..A-012** (Sprint 4): User God Object split, ABAC matrix populate startup, race quotas
- **A-013..A-018** (Sprint 5-7): Assembly edital pre-publish, Mux upload saga, Livekit room orphan
- **A-019** (Sprint 8): Timeline DELETE/UPDATE attempt guard
- **A-022** (Sprint 8): CORS wildcard Config.Validate()
- **A-025** (Sprint 9): Vote UNIQUE constraint enforced
- **A-026** (Sprint 9): SupplierVote UNIQUE constraint
- **A-027** (Sprint 9): Saga compensation Mux upload
- **A-028** (Sprint 9): Saga compensation Livekit StartLive

---

## Buckets previstos — findings novos da Fase 5 (remontagem)

Pré-mapeados em [[../AUDIT#findings-novos-desta-remontagem-a-descobrir]]. Conforme dual-check/QA/pentester pass avança, A-### novos nestes buckets:

### Modelagem / DDD

- Cross-BC coupling (interfaces em `shared/types/` vs domain events)
- Aggregate boundaries (tamanho, consistência transacional)
- Ubiquitous language (termos ambíguos)

Placeholder tasks bucket: **T-900 series**.

### Segurança / BeyondCorp

- ABAC matrix cobertura (pentester pass)
- Tenant isolation em 10 queries críticas (sample)
- Webhook HMAC em todos os providers
- Rate limit por endpoint
- LGPD data-flow por persona

Placeholder tasks bucket: **T-910 series**.

### Multi-produto

- Feature flag por plano vs persona vs tenant
- Quotas reset anual vs mensal vs sem reset
- Visibilidade cross-tenant em marketplace

Placeholder tasks bucket: **T-920 series**.

### Operação

- SLI/SLO/SLA por sub-produto
- Backup/restore estratégia
- DR/BCP (multi-região ou single-region)

Placeholder tasks bucket: **T-930 series**.

---

## Sincronização com [[../AUDIT]]

**Política atual**: double-write. Atualizar ambos arquivos simultaneamente. Última sincronização: **2026-04-23**.

**Decisão pendente**: criar D-### "unificar AUDIT path" — refactor skills pra ler daqui (elimina double-write).

---

## audit-log (histórico datado)

Sprint-auditor gera arquivo novo a cada rodada. Ver [[audit-log/_moc]].

Entradas existentes:
- [[audit-log/2026-04-23]] — seed (Fase 0 remontagem, descreve estado herdado)

---

## dual-check-log

Registros cronológicos de dual-checks executados. Ver [[dual-check-log/_moc]] + [[dual-check-log/template]].

---

## postmortems

Formato blameless para incidents > 15min. Ver [[postmortems/_moc]] + [[postmortems/template]].

---

## Links

- [[_moc]]
- [[../AUDIT]] — raiz (double-write)
- [[../STATE]]
- [[../SESSION_CHARTER]]
- [[../05-tasks/backlog]]
- [[../06-execution/playbooks/dual-check]]
- [[../06-execution/playbooks/cve-scan]]
- [[../CLAUDE]]
- [[../../vault/30-agents/playbooks/dual-check]]
