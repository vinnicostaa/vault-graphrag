---
title: Client vault mapping — template
type: template
tags: [template, obsidian, vault, mcp]
source: .kiro/specs/master-sindico/reference/obsidian-mapping.md (padrão extraído)
created: 2026-04-22
---

# Client vault mapping — template

Template pra criar o `reference/client-vault-mapping.md` em cada projeto que consulta **vault Obsidian externo** (do cliente, do arquiteto, do especialista de domínio) como fonte de contexto de negócio.

## Quando usar este template

- Projeto tem vault Obsidian em path fora do repo (ex: `~/Documentos/ObsidianVault/Clients/<projeto>/`)
- Vault contém **contexto de negócio** (discovery, arquitetura visualizada, decisões históricas, roadmap)
- Vault **não** é spec executável — é reflexão do stakeholder
- Agente tem MCP Obsidian configurado
- Precisa saber: **quando consultar, quando validar, o que ignorar**

## Estrutura da nota `reference/client-vault-mapping.md` no projeto

```markdown
---
title: Obsidian Vault Mapping
version: 1.0
status: stable
type: reference
---

# Obsidian Vault — mapa de notas úteis

O `<stakeholder>` mantém vault Obsidian em `<path>/` com notas de discovery, arquitetura, roadmap e decisões. **Contexto de arquiteto**, não spec executável.

Esta tabela existe pra **localizar** contexto relevante rapidamente. MCP Obsidian (ver `steering/mcp-integration.md`) permite ao agente ler/pesquisar notas em runtime.

## Estrutura do vault

<árvore de pastas do vault do cliente, com 2-3 níveis>

## Status de atualização (o que é confiável hoje)

| Nota | Atualização | Status |
|---|---|---|
| `<nota>` | `<YYYY-MM>` | ✅ confiável / ⚠️ desatualizado / 🟡 útil como visão geral |

**Legendas**:
- ✅ confiável — corresponde ao estado real do projeto
- ⚠️ desatualizado — cita stack/decisões já revistas (listar quais)
- 🟡 útil como visão geral — contexto sim, detalhe técnico não

## Quando consultar o Obsidian

- **Sim**: contexto de negócio não presente em `requirements/`, visualizações (canvas, tldraw), decisões históricas com cliente
- **Não**: decisões técnicas (stack, arquitetura, dependências) — código + specs são fonte de verdade
- **Não**: roadmap operacional — `tasks/` é fonte de verdade

## Como usar o MCP Obsidian

```
mcp__obsidian__search_notes(query="<termo>")
mcp__obsidian__read_note(path="<path-dentro-do-vault>")
mcp__obsidian__read_multiple_notes(paths=[...])         # até 10 em batch
mcp__obsidian__list_directory(path="<subpasta>")
```

**Permissões** (`.claude/settings.json`): apenas read-only por default. Tools de escrita (`write_note`, `delete_note`, `update_frontmatter`) em `ask`.

## Regra de divergência (Obsidian × projeto)

Quando o Obsidian diz **X** e o projeto diz **Y**:

1. **Projeto (código + specs) vence** — regra dura
2. Abrir issue/nota: "Obsidian note X divergente do backend requirements Y"
3. **Sugerir** ao stakeholder atualizar a nota — não editar por conta própria (nota é dele)
4. Se divergência é nome/rótulo, documentar em STATE.md

## Notas deprecadas (ignorar)

- Qualquer menção a **<stack rejeitada>** — decisão arquitetural revista
- Datas de sprint do roadmap — projeto tem datas diferentes (listar)
- Menções a legacy já reimplementado

## Fontes de atualidade

Se quiser saber o **estado real** do projeto:

1. **Código**: `<path-código>/` — é isso que está rodando
2. **Specs atuais**: `requirements/`, `design/`, `tasks/`
3. **Bloqueadores**: `STATE.md`
4. **Config local**: `.claude/settings.json`, `.mcp.json`, `.env`, `<lockfile>`
```

## Por que este template existe

Sem mapa, agente fica navegando em vault gigante sem rumo, lendo nota stale e propagando decisão já revisada. Um `client-vault-mapping.md` curado:

- **Economiza tokens** — vault pode ter centenas de notas, só algumas são relevantes
- **Evita alucinação sobre stack** — marca notas que citam stack antiga como "⚠️ desatualizado"
- **Dá rastreabilidade** — quando resposta do agente vem do Obsidian, fica claro qual nota
- **Respeita fronteira humano/agente** — vault é do stakeholder humano; agente consulta, não edita

## Integração com MCP

Plugin Obsidian MCP lista tools expostas. Configuração típica em `.claude/settings.json`:

```jsonc
{
  "permissions": {
    "allow": [
      "mcp__obsidian__search_notes",
      "mcp__obsidian__read_note",
      "mcp__obsidian__read_multiple_notes",
      "mcp__obsidian__list_directory",
      "mcp__obsidian__get_notes_info",
      "mcp__obsidian__get_frontmatter",
      "mcp__obsidian__get_vault_stats"
    ],
    "ask": [
      "mcp__obsidian__write_note",
      "mcp__obsidian__delete_note",
      "mcp__obsidian__patch_note",
      "mcp__obsidian__update_frontmatter",
      "mcp__obsidian__move_note",
      "mcp__obsidian__move_file",
      "mcp__obsidian__manage_tags"
    ]
  }
}
```

## Exemplo aplicado

Ver [[../50-projects/master-sindico/specs/reference/obsidian-mapping]] — instância real do template aplicada ao cliente Master Síndico.

## Links

- [[_moc]]
- [[../30-agents/mcp-integration]] — configuração genérica de MCP
- [[../50-projects/master-sindico/specs/reference/obsidian-mapping]] — exemplo aplicado
- [[CLAUDE_SPEC]] — seção MCP e credenciais
