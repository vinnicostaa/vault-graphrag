---
title: Obsidian MCP — Patterns
type: note
tags: [provider, mcp, obsidian, patterns, vault]
source: https://modelcontextprotocol.io/servers
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Obsidian MCP Patterns]
---

# Obsidian MCP — Patterns

Padrões atemporais de uso do MCP Obsidian em automação autônoma.

## 1. Pós-write verification obrigatório

Após **qualquer** `write_note` ou `patch_note`, rodar `list_directory` no path pai e confirmar que o arquivo aparece. Padrão crítico — ver [[../../../00-meta/AUDIT|A-vault-010, A-vault-042, A-vault-043]] (3 incidentes de alucinação de sucesso).

```
write_note(path, content)
   ↓ (resultado parece OK)
list_directory(parent_path)
   ↓ (confere que arquivo existe)
read_note(path) # amostra spot-check do conteúdo
```

Sem verificação, subagents podem reportar sucesso falsamente.

## 2. Template canônico sempre via `write_note`, nunca `Write`

No vault `automation-agents` (regra zero-B), **zero uso** de `Write`/`Bash echo`/heredoc para mutação. Motivo: clone filesystem não sincroniza ao vault real; `Write` cria arquivo que não aparece no Obsidian app.

Prompt de subagent deve explicitar:
- `PROIBIDO Write/Bash echo/heredoc para mutação`
- `APENAS mcp__obsidian__write_note`
- `list_directory obrigatório pós-write`

## 3. Frontmatter estruturado via `update_frontmatter`

Para mudar apenas metadata (sem tocar corpo), **preferir** `update_frontmatter(path, fm, merge=true)`. Benefícios:
- Não reescreve o corpo (economiza tokens)
- Valida YAML automaticamente
- `merge=true` preserva campos existentes

Evitar `write_note` completo quando muda só 1 campo.

## 4. `patch_note` cirúrgico com `oldString` único

`patch_note(path, oldString, newString)` falha se `oldString` aparece múltiplas vezes (sem `replaceAll=true`). Isso é **feature, não bug**: força contexto suficiente para evitar substituição ambígua.

Heurística: incluir 5-10 caracteres de contexto adicional quando string curta se repete:

```
# Ruim (múltiplos matches):
oldString = "AP-001"

# Bom (contexto único):
oldString = "[[../antipatterns/_moc]] — AP-001 (race)"
```

Para renomeação em massa (ex.: 26 headers OP-### em AGENTS_SPEC), usar 26 patches individuais (não `replaceAll`) para permitir falha fail-fast se string mudou.

## 5. Batching paralelo seguro

Múltiplos `write_note` / `patch_note` em paralelo **em arquivos distintos** é seguro — não há lock cross-file. Em **mesmo arquivo**: serializar (patch sequencial).

MCP tool calls independentes: lançar em paralelo (1 response, N tool_use blocks). Economiza round-trips.

## 6. `search_notes` para retrieval seed

Antes de criar nota nova, rodar `search_notes(query)` para detectar duplicação ou nota pré-existente que pode ser atualizada. Previne notas órfãs sobrepostas.

## 7. Paths relativos ao root do MCP (não do vault)

MCP server pode ter root diferente do vault raiz. Em `automation-agents`, root MCP é o pai do vault (`automation-agents/` contendo `automation-agents/vault/`), então paths começam com `automation-agents/vault/...`. **Sempre confirmar** com `list_directory('/')` na primeira sessão.

## 8. Recuperação de alucinação de subagent

Quando subagent reporta escrita bem-sucedida mas `list_directory` não mostra arquivo:

1. `find / -name '<arquivo>'` para checar se foi escrito em outro path (ex.: `/tmp/`, worktree sandbox).
2. Se não achou em lugar nenhum: **operador principal escreve direto** via `write_note`.
3. Registrar em AUDIT como incidente (`A-vault-###`) para padrão não se repetir.

## 9. Versionamento via `updated` no frontmatter

Toda mutação via MCP deve bumpar `updated: <ISO date>` no frontmatter. Use `update_frontmatter` para isolar a mudança. Permite queries Dataview por "notas modificadas após X".

## 10. `delete_note` com `confirmPath` defensivo

`delete_note` normalmente exige `confirmPath` igual ao path para executar. Em scripts automáticos, sempre passar explicitamente — evita deleção acidental por bug de construção de path.

## Links

- [[_moc]]
- [[overview]]
- [[../_moc]]
- [[../../../00-meta/AUDIT]] — A-vault-010/042/043 (lições dos incidentes)
- [[../../../30-agents/autonomous-execution]] — protocolo de escrita em modo autônomo
