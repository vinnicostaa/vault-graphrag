---
title: settings.local.json (config residual)
type: note
tags: [archive, master-sindico, config, residual]
created: 2026-04-24
---

# `settings.local.json` — config residual

Config residual do cliente, arquivado.

```json
{
  "permissions": {
    "allow": [
      "Bash(grep -E \"\\\\.\\(md|json|ini|yml|yaml|pdf|txt\\)$\")",
      "mcp__obsidian__list_directory",
      "mcp__obsidian__get_vault_stats",
      "mcp__obsidian__read_note",
      "mcp__obsidian__write_note",
      "Bash(find /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development -maxdepth 3 -type f -name *.md)",
      "Bash(find /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico -name requirements.md -o -name blueprint* -o -name architecture*)",
      "Bash(python3 -c \"import json,sys; data=json.load\\(sys.stdin\\); print\\(data[0][''''text'''']\\)\")",
      "Bash(python3 -c \"import json,sys; data=json.load\\(sys.stdin\\); text=data[0][''''text'''']; lines=text.split\\(''''\\\\n''''\\);:*)",
      "Bash(python3 -c \"import sys,json; data=json.load\\(sys.stdin\\); text=data[0][''''text'''']; print\\(text\\)\")",
      "Bash(find '/home/vinni/Documentos/Obsidian Vault' -name *.tldr -o -name *.tldraw)"
    ]
  }
}
```
