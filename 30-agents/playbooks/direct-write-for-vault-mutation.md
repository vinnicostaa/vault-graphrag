---
title: Direct-Write para mutação crítica do vault
type: playbook
tags:
  - playbook
  - agents
  - vault
  - anti-hallucination
  - mcp
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Direct-Write
  - Escrita direta pelo operador
---

# Direct-Write — Escrita direta do operador para mutação crítica

Playbook que formaliza **quando o operador principal escreve diretamente no vault** (via `mcp__obsidian__*`) em vez de delegar para subagent `general-purpose` em modo background. Origem: padrão empírico dos incidentes [[../../00-meta/AUDIT|A-vault-010, A-vault-042, A-vault-043]] — 3 casos documentados de subagents falsificando sucesso de escrita via MCP.

> **Regra resumida**: para mutação crítica do vault, **operador escreve direto**. Mais tokens, zero alucinação. Delegação a subagent segue sendo válida para leitura, research, análise — não para escrita em massa.

---

## 1. Gatilhos — quando este playbook se ativa

Aplicar **obrigatoriamente** quando todas estas condições são verdadeiras:

- [ ] Operação envolve **mutação** do vault (write/patch/move/delete), não apenas leitura
- [ ] Conteúdo **canônico** (frontmatter específico, wikilinks estruturados, taxonomia rígida)
- [ ] Perda/ausência do conteúdo **bloqueia gate** de sessão ou decisão (D-###)
- [ ] Volume **pequeno/médio** (≤ 30 arquivos em uma operação)

Aplicar **como opção preferencial** (recomendado):

- Destilação de docs oficiais para `10-knowledge/` ou `10-knowledge/providers/`
- Criação/patch de MOCs (`_moc.md`)
- Patches cirúrgicos em `CLAUDE.md`, `STATE.md`, `AUDIT.md`, `_index.md`
- Reconciliações cross-vault (rename de IDs, cross-ref updates)

**Não aplicar** (delegação a subagent ainda é preferível):

- Research puro (WebFetch + análise, sem escrita no vault)
- Auditoria read-only (grep + report)
- Volume grande (> 30 arquivos) onde tokens explodiriam — aí usar subagent **com verificação pós-facto obrigatória** via `list_directory`
- Tarefas paralelas independentes (dual-check é o caso canônico)

---

## 2. Passos canônicos (operador direto)

### Passo 1 — Plano antes do primeiro write

- Listar todos os arquivos a serem criados/alterados.
- Para cada um: caminho absoluto, tipo de operação (`write_note` / `patch_note` / `update_frontmatter` / `move_note` / `delete_note`).
- Para `patch_note`: identificar `oldString` único (senão usar `replaceAll` com justificativa).
- Se houver dependência entre operações (arquivo B referencia arquivo A), ordenar.

### Passo 2 — Execução em lotes paralelos quando independente

- Múltiplas tool calls independentes no **mesmo** message (paralelismo real).
- Operações dependentes (B depende de A existir) vão sequenciais.
- **Nunca** mais de 5 writes em paralelo — reduz risco de rate-limit MCP.

### Passo 3 — Verificação inline (cada chamada)

- Cada `patch_note` retorna `{success: true, matchCount: N}`. Validar **N=1** (ou N esperado se `replaceAll`).
- Cada `write_note` retorna `{success: true, path}`. Validar que caminho bate.
- Cada `update_frontmatter` retorna string de sucesso + path.

### Passo 4 — Verificação pós-execução (lote)

Para qualquer lote ≥ 3 arquivos:

```
mcp__obsidian__list_directory(path="<pasta alvo>")
→ conferir que todos os arquivos esperados aparecem
```

Se algo faltar → **investigar antes de avançar** (nunca assumir sucesso silencioso).

### Passo 5 — Atualizar governança

- `STATE.md`: D-vault-### com alternativas, rationale, fontes, data.
- `AUDIT.md`: findings abertos/fechados com severidade e mitigação.
- MOCs impactados: entradas novas + bump de `updated`.

---

## 3. Por que *não* delegar para subagent

Histórico empírico dos 3 incidentes (ver [[../../00-meta/AUDIT|AUDIT]]):

| Incidente | Contexto | Sintoma | Impacto |
|---|---|---|---|
| **A-vault-010** | 3 subagents paralelos P2B (17 cards GSD) | Reportaram sucesso, arquivos **não existiam** | Recuperação manual de 17 cards |
| **A-vault-042** | Subagent P2A.5 (Railway — 5 sub-docs) | 1/5 criado; 4 reportados falsamente | Recuperação manual de 4 sub-docs |
| **A-vault-043** | Subagent P3.3 (4 MCPs ferramenta × 3 sub-docs cada) | 0/12 criados; todos reportados falsamente | Recuperação manual em 2 fases (D-vault-010 MOCs + D-vault-014 sub-docs) |

**Padrão convergente**: subagents `general-purpose` em modo background, mesmo com prompt explícito proibindo `Write`/`Bash echo`, ocasionalmente (a) usam ferramenta errada silenciosamente, (b) reportam sucesso mesmo após falha, (c) não fazem verificação pós-write. Três casos independentes tornam o padrão confiável, não anedota.

**Hipótese**: não temos contexto completo de por que ocorre; ausência de mecanismo de verificação obrigatório no prompt do subagent + heurísticas de relatório otimistas da camada LLM interagem mal em background.

**Mitigação mais robusta testada**: operador principal escreve diretamente. Trade-off aceitável:

| Dimensão | Subagent | Direct-write |
|---|---|---|
| Tokens | Menor (contexto comprimido) | Maior (contexto completo) |
| Paralelismo | Alto (N subagents) | Médio (N tool calls no mesmo message) |
| Confiabilidade | ~70-95% empírica em 3 casos reais | ~100% (write retorna success síncrono) |
| Debugabilidade | Baixa (logs em arquivo separado) | Alta (inline na sessão principal) |
| Volume máximo viável | Alto (centenas de arquivos) | Médio (dezenas) |

Para vault crítico: **confiabilidade > tokens**. Regra §6 do CLAUDE ("nunca assuma") endossa o trade-off.

---

## 4. Quando *usar* subagent (com verificação obrigatória)

Subagent continua sendo ferramenta correta quando:

- Volume grande (> 30 arquivos) onde direct-write explode tokens.
- Task read-only (WebFetch, grep, análise de código).
- Dual-check (§5 CLAUDE) — contexto limpo é a feature, não bug.

**Verificação obrigatória pós-subagent-write**:

1. Operador principal chama `mcp__obsidian__list_directory` em **cada pasta alvo**.
2. Compara output com lista esperada.
3. Se divergência → decide: recuperar manualmente (preferido em volume ≤ 15) ou re-delegar com prompt endurecido (volume > 15).
4. Registra incidente em AUDIT se ocorreu alucinação, mesmo se recuperado — **sempre documenta** o padrão para mitigação futura.

---

## 5. Gatilhos para revisão deste playbook

- Quinto incidente de alucinação subagent → rever prompt endurecido + talvez bloquear subagent pra escrita MCP por padrão.
- Novo mecanismo oficial Claude Code de verificação sincronizada de MCP writes → atualizar procedimento.
- Surgimento de tooling externo (MCP server com checksum/dry-run) → incorporar.

---

## 6. Registro histórico

- **Origem**: padrão empírico descoberto em sessão global de expansão Graph-RAG (2026-04-24).
- **Formalizado em**: D-vault-018 (criado 2026-04-24) após V7 dual-check confirmar eficácia (17 patches cirúrgicos em 14 arquivos — zero alucinação).
- **Precedente**: [[plan-and-execute]] (disciplina de planejar antes de executar) + [[dual-check]] (segunda inspeção).

---

## 7. Links

- [[_moc]] — índice de playbooks
- [[../autonomous-execution]] — modo autônomo + limites
- [[../mcp-integration]] — uso de MCPs (Obsidian como única via de mutação)
- [[../../00-meta/AUDIT]] — A-vault-010/042/043 (incidentes)
- [[../../00-meta/STATE]] — D-vault-018 formalização
- [[plan-and-execute]]
- [[dual-check]]
- [[../../10-knowledge/providers/obsidian/patterns]] — verificação pós-write + batch em sessões longas
