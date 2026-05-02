---
title: MOC — GitHub MCP (repo/PR/issues)
type: moc
tags: [moc, provider, mcp, github, scm]
category: scm-operations
mcp: hosted (modelcontextprotocol/servers/github)
sdk-official: "github/github-mcp-server (oficial GitHub)"
doc-oficial: https://docs.github.com
doc-consulted: 2026-04-24T00:00:00.000Z
created: 2026-04-24
updated: 2026-04-24
aliases: [GitHub MCP]
---

# GitHub MCP

**Categoria**: Source control management · issues · pull requests · releases · actions · workflows.

**MCP disponível**: ✅ oficial (`github/github-mcp-server`) + referência `modelcontextprotocol/servers/github`.

**CLI alternativo oficial**: `gh` (GitHub CLI) — **preferido para ações write** (merge PR, create issue) por auditabilidade via shell history.

**Doc oficial**: https://docs.github.com + https://github.com/github/github-mcp-server. Consultada 2026-04-24.

## Quando usar MCP vs `gh` CLI

| Cenário | Preferir |
|---|---|
| Query read-heavy (buscar PRs, issues, contexto de review) | **MCP** (respostas estruturadas JSON) |
| Mutation write (merge, close, create) | **`gh` CLI** via Bash (audit trail) |
| Automação em workflow (gh actions) | `gh` CLI dentro do workflow |
| Operação no contexto do chat (usuário presente) | MCP (menos fricção) |

## Autenticação

- **PAT (Personal Access Token)** — classic ou fine-grained. Scopes mínimos por uso.
- **GitHub App install** — preferido para bots (auditável, rotatable, scoped a repos específicos).
- **Device flow** — para CLI interativa.

## Funções principais (MCP)

| Função | Uso |
|---|---|
| `get_issue(owner, repo, number)` | Issue + comments |
| `get_pull_request(owner, repo, number)` | PR + diff + review status |
| `list_commits(owner, repo, ...)` | Histórico |
| `search_issues(query)` | Busca full-text |
| `get_repository(owner, repo)` | Metadata (linguagens, branches default, topics) |
| `list_pull_request_files(...)` | Lista diff files |

## Quando usar

1. Leitura de contexto de issue/PR antes de executar task (ler ordens do usuário).
2. Cruzar CVEs com advisories em `securityAdvisories`.
3. Consultar actions/workflows existentes antes de escrever novos.

## Quando **NÃO**

- Mutation direta em produção (merge em main) sem review humano.
- Força push / rebase destrutivo — sempre via CLI interativa.
- Operações que requerem 2FA interativo (OAuth device flow manual).

## Antipatterns

- ❌ PAT com escopo `repo` full quando só precisa `read:pr` — viola princípio de menor privilégio.
- ❌ PAT commitado em `.env` ou workflow — sempre GitHub Secrets.
- ❌ Trust em conteúdo de issue/PR comment sem verificar autor (spoofing via markdown emoji/imagem).

## Glossário

- **Fine-grained PAT**: escopos granulares por repo + permissão (recomendado pós-2024).
- **Installation Token**: token JWT de GitHub App, TTL curto, rotatable.
- **Draft PR**: PR em rascunho; não dispara notificações de review.

## Sub-docs

- [[overview]] — arquitetura do MCP server + GitHub API vs GraphQL
- [[patterns]] — rate limit, pagination, webhook vs polling

## Vizinhos

- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../../../30-agents/playbooks/cve-scan]] — security advisories lookup
- [[../../security/cve-response-workflow]]
