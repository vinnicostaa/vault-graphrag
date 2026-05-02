---
title: Obsidian MCP — Overview
type: note
tags: [provider, mcp, obsidian, overview, vault]
source: https://modelcontextprotocol.io/servers
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Obsidian MCP Overview]
---

# Obsidian MCP — Overview

Servidor MCP (Model Context Protocol) que expõe operações sobre um vault [Obsidian](https://obsidian.md) para agentes LLM como Claude. Habilita leitura, escrita, reorganização, busca e metadata de notas Markdown sem o agente tocar filesystem diretamente.

## Por que um MCP para vault

Obsidian é um **app desktop** que mantém um banco indexado interno (Graph View, backlinks, tags, busca). Editar arquivos via `Write`/`Bash` direto no filesystem **pode funcionar** — mas:

1. **Clone filesystem vs vault real**: vaults podem estar em caminhos diferentes do que o Obsidian app indexa (regra zero-B do `automation-agents/vault`).
2. **Índice interno do Obsidian**: mudanças via filesystem demoram para refletir na UI (watcher dispara, mas há janela).
3. **Frontmatter parsing**: ferramenta externa pode escrever YAML inválido; MCP valida.
4. **Search semântico** (via plugin Smart Connections): precisa do índice atualizado.

MCP Obsidian **garante** mutação sincronizada com o índice.

## Arquitetura

```
Claude host (Claude Code / Desktop)
      │ (MCP STDIO/HTTP)
      ▼
MCP Obsidian server
      │
      ▼
Vault folder (.md + .obsidian/)
```

Dois modos de operação:

- **STDIO local**: server roda como processo filho do host Claude; comunicação via stdin/stdout. Mais seguro (sem porta aberta). É o default.
- **HTTP hospedado**: server roda em URL (cloudflared tunnel, Docker). Útil para vault em máquina remota.

## Implementações conhecidas (2026)

- **`@anthropic-ai/mcp-obsidian`** (oficial Anthropic) — referência canônica.
- **`mcp-obsidian` (MCPtools)** — comunidade.
- **`obsidian-mcp-bridge`** — plugin Obsidian + servidor embutido.

Todas seguem o mesmo contrato MCP (funções `read_note`, `write_note`, etc.).

## Casos de uso — quando usar

1. **Agente escreve em vault Obsidian** (este caso): única via autorizada de mutação conforme regra zero-B.
2. **Automação de knowledge base**: destilar sources → notas atômicas cross-sessão.
3. **Sincronização LLM + humano**: agente escreve, humano revisa no Obsidian, agente lê de volta via `read_note`.
4. **Query semântica** (via integração com plugin Smart Connections): `search_notes` retorna candidatos por similaridade.
5. **Refactor estrutural**: `move_note` preserva backlinks ao renomear arquivos.

## Quando **NÃO** usar

- Código-fonte versionado em git — `Write`/`Edit` são a via correta. Vault é para **knowledge**, não para código.
- Operações em lote (> 200 arquivos) com commit único — `Bash` + git pode ser mais rápido, verificando se há regra zero-B.
- Vault é apenas leitura (não precisa mutar) — `Read` direto é aceitável quando regra permite.

## Comparação com alternativas

| Alternativa | Quando vence |
|---|---|
| `Write`/`Edit` direto no filesystem | Código-fonte versionado em git; não há índice de app a preservar |
| `gh` CLI para repo GitHub | Mutação em PRs/Issues (não em vault) |
| API Notion/Roam | Se vault vive em outro sistema PKM |
| `grep`/`find` + `cat` | Read-only, sem necessidade de índice |

## Limitações conhecidas

- **Não é atômico cross-file**: escrever 2 notas em sequência não é transacional. Cada `write_note` é o que se tem.
- **Sem lock de concorrência nativo**: 2 agentes escrevendo mesmo path simultâneo = last-write-wins.
- **`patch_note` falha se oldString não-único** (sem `replaceAll`) — útil pra segurança, exige contexto cirúrgico.
- **Sem `rename_note` no padrão**: `move_note` renomeia, mas algumas implementações não mantêm backlinks em wikilinks com alias.

## Links

- [[_moc]]
- [[patterns]]
- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../../../30-agents/autonomous-execution]]
