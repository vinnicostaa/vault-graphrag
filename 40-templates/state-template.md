---
title: STATE.md — template
type: template
tags: [template, governance, sdd]
source: .kiro/STATE.md (padrão extraído)
created: 2026-04-22
aliases: [STATE template]
---

# STATE — `<PROJETO>` Backend

Arquivo **vivo** de bloqueadores, decisões em aberto e dívidas técnicas. Complementa `tasks.md` (estático) com contexto dinâmico. **Atualize sempre que descobrir um muro, abrir uma decisão, ou registrar débito.**

Formato de cada entrada: data ISO + bloco curto. Entradas resolvidas movem para "Histórico" (mantém auditoria).

Última revisão: `YYYY-MM-DD` (`<contexto curto da última revisão>`)

---

## 🚧 Bloqueadores Ativos

*Nenhum bloqueador crítico no momento.* — **ou lista estruturada abaixo**

### B-0XX — `<título curto>` 🔴 Aberto
- **Descoberta**: `YYYY-MM-DD` | **Severidade**: `critical|high|medium`
- **Contexto**: `<o problema em 1-2 linhas>`
- **Impacto**: `<quem sofre e por quê>`
- **Responsável**: `<pessoa/squad>`
- **Prazo/alvo**: `<sprint/data>` para resolver
- **Dependências**: `<o que precisa pra destravar>`

---

## 🎯 Progresso — Sprint `<N>` (`<nome>`)

### `<subárea / feature>` — `✅ CONCLUÍDO <data>` | `🔨 EM ANDAMENTO` | `📋 PENDENTE`

**`<Resumo de 3-5 linhas do que foi feito ou o que está previsto>`**

- `✅` item concluído (+ commit hash se aplicável)
- `🔨` item em andamento
- `⚠️` item com risco — ver A-XXX em AUDIT
- `📋` item pendente

**Gates finais da entrega**: `<build>` ✅, `<vet>` ✅, `<test>` ✅, `<security-scanner>` ✅, `<vuln-check>` ✅

**Fixes críticos aplicados nesta sessão**:
1. `<fix 1 com arquivo:linha>`
2. `<fix 2>`

---

## 📐 Decisões Arquiteturais (D-0XX)

### D-0XX — `<título curto>`
- **Data**: `YYYY-MM-DD`
- **Contexto**: `<problema que motivou>`
- **Decisão**: `<escolha tomada>`
- **Fontes consultadas**:
  - Context7: `<lib>@<versão>` topic `"<topic>"`
  - Source package: `$CACHE/<lib>@<v>/<path>`
  - Grandes empresas: `<referência (Uber/Netflix/Stripe/GitHub)>`
  - Boas práticas: `<princípio aplicado (DDD/SOLID/OWASP)>`
  - Contexto do projeto: `<arquivo/padrão existente>`
- **Alternativas rejeitadas**: `<A, B, C>` — por quê
- **Dual-agent usado**: `sim|não` — se sim, resumo da convergência/divergência
- **Reversibilidade**: `baixa|média|alta`
- **Dual-validate?**: `sim|não` (decisão de alto impacto?)

---

## 💳 Dívidas Técnicas (DT-0XX)

### DT-0XX — `<título curto>`
- **Data**: `YYYY-MM-DD`
- **O quê**: `<descrição da dívida>`
- **Por que adiado**: `<razão>`
- **Custo de pagar depois**: `<estimativa>`
- **Sprint alvo**: `<N ou "Sem alvo">`

---

## 📚 Histórico (resolvidos)

### B-0XX — `<título>` ✅ `YYYY-MM-DD`
Resolução: `<1-2 linhas do que foi feito>`

### D-0XX — `<título>` ✅ aplicado em `<sprint N>`
Implementação concluída em `<arquivo:linha>`.

### DT-0XX — `<título>` ✅ pago em `<sprint N>`
Refactor em `<pacote>`.

---

## Convenções

- **B-0XX** = Bloqueador ativo (impede progresso)
- **D-0XX** = Decisão arquitetural registrada (já aplicada ou ativa)
- **DT-0XX** = Dívida técnica registrada (adiada, com alvo)
- IDs nunca são reutilizados (auditoria)
- Entradas resolvidas migram pro "Histórico" do próprio arquivo
- `STATE.md` complementa — **nunca substitui** — `tasks.md` (estático) nem `AUDIT.md` (findings pós-revisão)

## Relacionamento com outros arquivos

- **`tasks.md`** — estático: lista de tasks planejadas por sprint, com `[x]`/`[ ]`
- **`STATE.md`** — dinâmico: bloqueadores + decisões + dívidas em tempo real
- **`AUDIT.md`** — findings pós-revisão (A-XXX) por sprint-auditor — ver [[audit-template]]
- **`SESSION_CHARTER.md`** — briefing da sessão corrente — ver [[session-charter-template]]

## Links

- [[_moc]]
- [[audit-template]]
- [[session-charter-template]]
- [[../10-knowledge/methodology/sdd-workflow]]
- [[../30-agents/autonomous-execution]]
