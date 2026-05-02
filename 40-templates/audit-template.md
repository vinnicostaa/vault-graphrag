---
title: AUDIT.md — template
type: template
tags: [template, audit, governance, sdd]
source: .kiro/AUDIT.md (padrão extraído)
created: 2026-04-22
aliases: [AUDIT template]
---

# AUDIT — `<PROJETO>` Backend

Arquivo **vivo** de findings pós-sprint gerados pela skill **`sprint-auditor`**. Complementa `STATE.md` (bloqueadores e dívidas planejadas) com **bugs, más implementações e divergências de spec** descobertas em revisão sistemática ao fim de cada sprint.

**Regra dura**: próximo sprint **não começa** enquanto existir item `🔴 Aberto` ou `🟡 Em andamento`. O agente que roda `task-executor` é **obrigado** a abrir este arquivo antes de puxar próxima task de `tasks.md` — se houver items abertos, resolve primeiro.

## Formato de cada entrada

- **ID sequencial** (`A-###`) por ordem de criação — **nunca reutilizar**
- **Data ISO de descoberta** (+ data de resolução quando fixado)
- **Sprint alvo** que originou o finding
- **Severidade**: `critical` (bloqueia deploy) | `high` (bug com impacto) | `medium` (cheiro forte) | `low` (cosmético)
- **Arquivo(s) afetado(s)** com linha quando aplicável
- **Raiz** do problema (não sintoma)
- **Fix proposto** em uma frase
- **Status**: 🔴 Aberto / 🟡 Em andamento / ✅ Resolvido

Entradas resolvidas movem pra "Histórico" mantendo ID original.

Última revisão: `YYYY-MM-DD` (`<contexto curto da rodada>`)

---

## 🔴 Itens Abertos

### A-0XX — `<título curto>` 🔴 Aberto
- **Descoberta**: `YYYY-MM-DD` | **Sprint**: `<N>` | **Severidade**: `critical|high|medium|low`
- **Arquivo**: `<path:linha>`
- **Raiz**: `<causa real, não sintoma — em 1-2 frases>`
- **Fix proposto**: `<em uma frase>`
- **Sprint alvo**: `<N>` — `<justificativa (bloqueia marco, dívida, etc.)>`

---

## 🟡 Em Andamento

*Vazio.* — **ou listar aqui os A-### que começaram mas ainda não fecharam**

### A-0XX — `<título>` 🟡 Em andamento
- **Status**: `<o que foi feito até agora>`
- **Pendente**: `<o que falta>`
- **ETA**: `<data estimada>`

---

## ✅ Resolvidos nesta Rodada

### A-0XX — `<título curto>` ✅ Resolvido
- **Descoberta**: `YYYY-MM-DD` | **Resolução**: `YYYY-MM-DD` | **Sprint**: `<N>` | **Severidade**: `<grau>`
- **Arquivo**: `<path:linha>`
- **Raiz**: `<causa>`
- **Fix aplicado**: `<descrição + commit hash se aplicável>`
- **Verificação**: `<como foi validado: teste X passa, grep Y retorna, gate Z verde>`

---

## 📚 Histórico (resolvidos de rodadas anteriores)

### A-0XX — `<título>` ✅ `YYYY-MM-DD`
`<1 linha do fix>` — commit `<hash>`

### A-0XX — `✅ Fechado (falso positivo)`
`<justificativa da invalidação em 1-2 linhas>`

---

## Distinção STATE vs AUDIT

| Dimensão | STATE.md | AUDIT.md |
|---|---|---|
| Natureza | Bloqueadores/decisões/dívidas **conscientes** | Bugs/má implementação **descobertos em revisão** |
| Momento | Qualquer hora (pro-ativo) | Pós-sprint (reativo) |
| Alvo | Sprint alvo pra pagar | Sem alvo — **prioridade máxima** |
| Criado por | Humano ou agente em sessão | Skill `sprint-auditor` |
| Severidade | `B-XXX`, `D-XXX`, `DT-XXX` | `critical`, `high`, `medium`, `low` |
| Bloqueia próxima sprint? | Só se `B-XXX` crítico | `critical` e `high` bloqueiam |

## Severidade — critérios

| Nível | Critério |
|---|---|
| `critical` | Bloqueia deploy. Segurança, perda de dados, duplicação financeira, bypass de autorização. |
| `high` | Bug funcional com impacto visível ao usuário OU violação de princípio arquitetural central (dependency rule, tenant isolation, idempotência). |
| `medium` | Cheiro forte — antipattern replicado, manutenibilidade comprometida, risco latente. |
| `low` | Cosmético — naming, formatting, comentário desatualizado. |

`critical` e `high` **bloqueiam próximo sprint**. `medium` vai pro backlog com sprint alvo. `low` pode ficar pendurado.

## Regras duras

- **`task-executor` lê AUDIT.md** no início de cada sessão (Gate 0 do SDD) — antes de puxar task nova de `tasks.md`
- **`sprint-auditor` nunca implementa fix** — apenas aponta. Fix é responsabilidade do `task-executor` na sessão seguinte, tratando cada item como micro-task (mesmo ciclo 5-fase, escala reduzida)
- **Catálogo de antipatterns cross-linguagem** → [[../10-knowledge/antipatterns/_moc]] — todo finding em AUDIT pode referenciar um `AP-XXX` do catálogo

## Links

- [[_moc]]
- [[state-template]]
- [[session-charter-template]]
- [[../10-knowledge/methodology/sdd-workflow]] — Fase 6 (Sprint Audit)
- [[../10-knowledge/antipatterns/_moc]] — catálogo AP-XXX
- [[../30-agents/autonomous-execution]] — protocolo de decisão
