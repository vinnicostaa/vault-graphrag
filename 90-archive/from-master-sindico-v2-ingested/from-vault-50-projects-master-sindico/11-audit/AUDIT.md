---
title: AUDIT — Master Síndico (canônico 11-audit)
type: audit
tags: [master-sindico, audit, canonical]
project: master-sindico
created: 2026-04-22
---

# AUDIT — Master Síndico

Este é o arquivo **canônico** de findings. O arquivo original continua em [[../AUDIT]] (raiz do projeto) por compat com skills legadas (`task-executor`, `sprint-auditor`). Quando atualizar, **atualizar os dois** OU substituir a leitura das skills por este path.

**Formato de entrada**: ver [[../AUDIT#como-usar]] (seção "Para o auditor" e "Para o task-executor").

**Status**: sincronizado com [[../AUDIT]] até 2026-04-22. A partir desta data, manter os dois arquivos **espelho** ou refactor skills pra ler daqui.

---

## Regras

1. **AUDIT vs STATE**:
   - AUDIT = findings **descobertos em revisão** (A-###)
   - STATE = decisões (D-###) e dívidas **conscientes** (DT-###)
2. 🔴🟡 aberto **bloqueia** task nova do sprint seguinte
3. `task-executor` abre este arquivo no início de toda sessão
4. `sprint-auditor` roda ao fim de cada sprint e materializa findings
5. ID sequencial (A-###) nunca reutilizado

---

## Seções obrigatórias

- 🔴 Itens Abertos
- 🟡 Em Andamento
- ✅ Resolvidos nesta Rodada
- 📜 Histórico

---

## Sincronização com [[../AUDIT]]

Opções:
1. **Double-write**: agente atualiza ambos (custo baixo enquanto conteúdo é estável)
2. **Refactor skills**: apontar `task-executor` e `sprint-auditor` pra este path (novo canônico)

**Decisão pendente**: D-### "unificar AUDIT path" — criar no próximo sprint.

**Enquanto isso**: manter AUDIT.md raiz como fonte, este arquivo como **mirror** atualizado no fim de cada sprint pelo sprint-auditor.

---

## Espelho do conteúdo

Ver [[../AUDIT]] (autoritativo). Este arquivo sincroniza conteúdo após cada rodada.

Última sincronização: **2026-04-22** (snapshot fim Sprint 9).

### Resumo de status (da última sincronização)

- 🔴 Abertos: A-023, A-024 (médio, Sprint 10)
- 🟡 Em andamento: vazio
- ✅ Resolvidos na rodada: A-019, A-020, A-021, A-022, A-025, A-026, A-027, A-028, A-029, A-030, A-031, A-032, A-033, A-034
- 📜 Histórico: A-001..A-018

---

## audit-log (histórico datado)

Ver [[audit-log/_moc]]. Sprint-auditor gera arquivo novo a cada rodada.

---

## dual-check-log

Ver [[dual-check-log]] *(a criar quando primeiro dual-check for documentado)*.

---

## postmortems

Ver [[postmortems]] *(a criar quando primeiro incident > 15min)*.

---

## Links

- [[_moc]]
- [[../AUDIT]] — autoritativo até refactor
- [[../STATE]]
- [[../SESSION_CHARTER]]
- [[../../../30-agents/playbooks/dual-check]]
- [[../../../30-agents/playbooks/cve-scan]]
- [[../CLAUDE]]
