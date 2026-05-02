---
title: Playbook — GSD Library Resync
type: playbook
tags: [playbook, agent, gsd, library, staleness, resync]
created: 2026-04-24
updated: 2026-04-24
aliases: [gsd-resync, resync gsd-library]
---

# Playbook — GSD Library Resync

Re-sincroniza `30-agents/gsd-library/` quando o repo upstream [`gsd-build/get-shit-done`](https://github.com/gsd-build/get-shit-done) evolui. Sem este processo, a biblioteca congela no *commit/date* do clone inicial.

## Gatilho

- **Manual**: operador executa `/gsd-resync` ou comando equivalente.
- **Periódico**: cron trimestral (verificar upstream a cada 3 meses).
- **Reativo**: agente percebe divergência entre card e comportamento esperado do upstream (raro).

## Pré-condições

- `60-sources/get-shit-done/raw/agents/` ainda é o clone canônico.
- Cards e famílias em `30-agents/gsd-library/` têm frontmatter `source_date` e (quando conhecido) `source_commit`.

## Protocolo — 6 passos

### 1 — Capturar estado upstream

Via `Bash`:

```bash
cd /tmp && git clone --depth=1 https://github.com/gsd-build/get-shit-done gsd-fresh
cd gsd-fresh && git rev-parse HEAD > /tmp/gsd-upstream-sha.txt
ls .claude/agents/ > /tmp/gsd-upstream-agents.txt
```

Guardar `sha` e lista de arquivos.

### 2 — Diff de inventário

Comparar `/tmp/gsd-upstream-agents.txt` com `ls 60-sources/get-shit-done/raw/agents/`:
- Agents **adicionados** upstream.
- Agents **removidos** upstream.
- Agents **renomeados** (detectar por prefixo `gsd-*`).

Resultado em `/tmp/gsd-inventory-diff.md`.

### 3 — Diff de conteúdo

Para cada agent presente em ambos, `diff` semântico entre raw do vault e upstream. Classificar:
- **Unchanged** — nenhuma ação.
- **Minor** (frontmatter, wording): atualizar `source_commit` e `source_date` no card; sem mudança de corpo necessária.
- **Major** (novas seções, mudança de tools, mudança de `description`): flag `reviewed: false` no card + gerar issue interna.

### 4 — Aplicar mudanças controladas

Por ordem:

1. **Remover** cards de agents deletados upstream → `mcp__obsidian__delete_note` + atualizar `_moc.md` + `family-*.md` respectivos.
2. **Atualizar raw**: overwrite de `60-sources/get-shit-done/raw/agents/` com conteúdo do clone fresh.
3. **Atualizar cards Minor**: `mcp__obsidian__update_frontmatter` só com `source_commit` + `source_date` + `updated`.
4. **Marcar cards Major com `reviewed: false`**: patch no frontmatter; card **não sai do ar** mas sinaliza que destilação pode estar desatualizada.
5. **Criar cards para agents novos**: seguir template canônico do [[../gsd-library/_moc|_moc]] da gsd-library.

### 5 — Dual-check amostral

Para cards classificados como **Major** (ver [[dual-check]]):
- Agente-B independente valida que a nova destilação captura a mudança upstream sem alucinação.
- Se aprovado → `reviewed: true`.
- Se reprovado → registrar DT em STATE + manter `reviewed: false`.

### 6 — Registro em STATE

Adicionar ou atualizar D-vault-### específico (ex.: D-vault-004-resync-YYYY-MM-DD) com:
- Commit upstream antes e depois.
- Contagem de cards Added / Removed / Minor / Major.
- Issues abertas de cards Major não-validados.
- Data alvo de próxima resync.

## Detecção de staleness por idade

Regra operacional: se `source_date < now - 180 dias`, rodar resync **obrigatoriamente** antes de usar cards em decisões ativas. Cards stale são sinalizados como referência histórica, não operacional.

## Anti-padrão

- ❌ Overwrite dos cards sem diff — perde destilação pt-BR própria.
- ❌ Importar upstream 1:1 sem aplicar atomic notes (40-60 linhas) — violaria D-vault-004.
- ❌ Remover raw do vault achando que "basta o card" — raw é fonte canonical para disputa de fidelidade.
- ❌ Pular dual-check em cards Major — propaga drift.

## Requisitos de execução

- `Bash` (git clone/diff), `mcp__obsidian__*` (read/write/patch/delete/update_frontmatter), `Agent` para dual-check amostral.
- Tempo estimado por resync: 1-3h em modo autônomo, dependendo do volume de diff.

## Links

- [[_moc]]
- [[../gsd-library/_moc]] — biblioteca mantida por este playbook
- [[dual-check]] — amostral no passo 5
- [[research-loop]] — padrão de leitura sistemática reutilizado
- [[../../00-meta/STATE]] — D-vault-004 e sucessores
- [[../../60-sources/get-shit-done/_toc]] — TOC original
