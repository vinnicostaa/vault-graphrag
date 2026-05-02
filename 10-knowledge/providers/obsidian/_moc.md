---
title: MOC — Obsidian MCP (vault mutation)
type: moc
tags: [moc, provider, mcp, obsidian, vault]
category: vault-mutation
mcp: stdio (plugin Obsidian local)
sdk-official: "@anthropic-ai/mcp-obsidian (oficial Anthropic) ou terceiros (MCPtools, mcp-obsidian)"
doc-oficial: https://docs.obsidian.md
doc-consulted: 2026-04-24T00:00:00.000Z
created: 2026-04-24
updated: 2026-04-24
aliases: [Obsidian MCP]
---

# Obsidian MCP

**Categoria**: vault mutation · Markdown Personal Knowledge Management · **única via** de mutação autorizada no vault `automation-agents` (regra zero-B do [[../../../CLAUDE]]).

**MCP disponível**: ✅ via STDIO (plugin Obsidian local) ou implementação via servidor (cloudflared tunnel).

**SDKs/Implementações conhecidas**:
- `@anthropic-ai/mcp-obsidian` (oficial Anthropic)
- Community: `mcp-obsidian` (MCPtools), `obsidian-mcp-bridge`
- Funciona como MCP server invocado pelo host Claude Code/Claude Desktop.

**Doc oficial**: https://docs.obsidian.md (app) + https://modelcontextprotocol.io/servers (catálogo). Consultada 2026-04-24.

## Quando usar

1. **Leitura/escrita em vault Obsidian clone filesystem** que NÃO sincroniza — MCP é a única via que garante sincronização com o vault real (regra zero-B do [[../../../CLAUDE]]).
2. Notas atômicas, atualização de frontmatter, movimentação, listagem — todas via `mcp__obsidian__*`.
3. Busca semântica (`search_notes`) sem precisar cat+grep em milhares de arquivos.

## Quando **NÃO** usar

- Código-fonte versionado git — `Write`/`Edit` são a via correta (não vault).
- Operações em lote massivas (> 200 arquivos) — pode ser mais rápido via `Bash` git, depois commit único; mas verificar regra zero-B do projeto.

## Funções principais

| Função | Uso |
|---|---|
| `write_note(path, content, mode)` | Cria ou sobrescreve nota (mode: overwrite/append/prepend) |
| `read_note(path)` | Lê nota + frontmatter estruturado |
| `list_directory(path)` | Lista arquivos/pastas; **essencial para verificação pós-write** |
| `update_frontmatter(path, fm, merge)` | Atualiza só frontmatter preservando corpo |
| `patch_note(path, oldString, newString, replaceAll)` | Substituição cirúrgica; falha se oldString não-único (sem replaceAll) |
| `move_note(from, to)` | Renomeia/move preservando backlinks |
| `delete_note(path)` | Remove com confirmPath |
| `search_notes(query)` | Busca full-text + frontmatter |
| `get_vault_stats()` | Volumetria + recently modified |

## Antipattern crítico — alucinação de sucesso

**Histórico interno**: A-vault-010, A-vault-042, A-vault-043 — subagents Claude `general-purpose` com prompt "APENAS MCP" mesmo assim usaram `Write`/`Bash echo` que falha silenciosamente neste ambiente.

**Mitigação obrigatória**:
1. Prompt explícito: *"PROIBIDO `Write`/`Bash echo`; causa falha silenciosa"*.
2. `list_directory` pós-write **obrigatório** + spot-check do operador principal antes de declarar completude.
3. Quando subagent reporta sucesso, **não confiar** — verificar.

## Glossário interno

- **Clone filesystem**: cópia local do vault real (no disco). Mutações via filesystem NÃO propagam automaticamente ao vault real.
- **Vault real**: a instância que o Obsidian app lê; MCP é a ponte autoritativa.
- **Frontmatter**: bloc YAML no topo; lido estruturadamente por `read_note` (`fm` no retorno).

## Sub-docs

- [[overview]] — arquitetura do MCP server + flow STDIO
- [[patterns]] — pós-write verification, batch em sessões longas, recuperação de alucinação

## Vizinhos

- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../../../30-agents/autonomous-execution]]
- [[../../../00-meta/AUDIT]] — A-vault-010/042/043 (incidentes de uso)
