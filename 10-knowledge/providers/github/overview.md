---
title: GitHub MCP — Overview
type: note
tags: [provider, mcp, github, overview, scm]
source: https://docs.github.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [GitHub MCP Overview]
---

# GitHub MCP — Overview

Servidor MCP que expõe operações da API GitHub (issues, pull requests, releases, actions, workflows, advisories, repositories) para agentes LLM. Alternativa estruturada ao uso exclusivo do CLI `gh` via `Bash`.

## Implementações conhecidas

- **`github/github-mcp-server`** (oficial GitHub) — canônico, atualização ativa.
- **`modelcontextprotocol/servers/github`** (referência community).

Dois modos de deploy:
- **Hosted**: endpoint gerenciado por GitHub.
- **Self-hosted (Docker)**: quando compliance exige dados permanecerem em região específica.

## Autenticação

Três caminhos:

| Método | Quando usar |
|---|---|
| **PAT (fine-grained)** | Uso pessoal/bot simples. Escopo mínimo por repo. |
| **PAT classic** | Legacy; evitar em novos setups (escopos amplos demais). |
| **GitHub App install** | Bot organizacional. Auditável, scoped, rotatable. **Recomendado** para automação em prod. |
| **Device flow** | CLI interativo humano. |

Secrets **nunca** em git — sempre GitHub Secrets (workflows) ou env var do host MCP.

## Funções principais

| Categoria | Funções (exemplo) |
|---|---|
| Repo metadata | `get_repository`, `list_repositories`, `search_repositories` |
| Issues | `get_issue`, `list_issues`, `create_issue`, `add_issue_comment` |
| Pull requests | `get_pull_request`, `list_pull_requests`, `list_pull_request_files`, `create_pull_request_review` |
| Commits | `list_commits`, `get_commit` |
| Actions | `list_workflows`, `get_workflow_run` |
| Advisories | `list_global_advisories`, `get_advisory` |
| Search | `search_issues`, `search_code`, `search_users` |

API v3 REST (maioria) + GraphQL (queries complexas de múltiplas entidades).

## MCP vs `gh` CLI — quando cada vence

| Cenário | Preferido |
|---|---|
| **Query read-heavy** (buscar PRs, issues, diffs, reviews) | **MCP** — respostas JSON estruturadas consumíveis pelo agente |
| **Mutation write** (merge, close, criar release) | **`gh` CLI** — audit trail via shell history, menos chance de acidente em batch |
| **Dentro de GitHub Action** | **`gh` CLI** — já instalado no runner |
| **Paginação controlada** | **MCP** — melhor estrutura de resposta |
| **Exposição de tokens** | **MCP** — servidor mantém credencial isolada |
| **Operações interativas humano-no-loop** | **`gh` CLI** — UX superior em terminal |

Regra heurística: **read via MCP, write via `gh`**.

## Casos de uso — quando usar

1. **Contexto de issue/PR antes de executar task** — ler ordens do usuário e reviews.
2. **Cruzar CVE com advisory**: `list_global_advisories` retorna dados estruturados para `cve-scan` playbook.
3. **Inspeção de workflow existente** antes de escrever novo YAML.
4. **Análise de PR enorme**: `list_pull_request_files` + `get_pull_request_review_comments` consolida contexto.
5. **Bot de automação**: sync issue → Jira, auto-label, stale bot.

## Quando **NÃO** usar

- **Merge em `main` sem review humano** — gate humano obrigatório.
- **Força push / rebase destrutivo** em branch protegido — sempre via CLI interativo com 2FA.
- **2FA interativo** (device flow) — MCP não mediará código de 6 dígitos.
- **Operações em repo privado de outra org sem install explícito** — install do GitHub App é explícito por org.

## Limitações

- **Rate limit**: 5000 req/h autenticado (PAT/App). Pagination grande em repo enorme pode bater.
- **Eventual consistency**: mudança feita via CLI pode demorar segundos para aparecer em query MCP.
- **Permissions scoping**: MCP respeita o token; se PAT não tem escopo `workflow`, chamadas de Actions retornam 403.

## Links

- [[_moc]]
- [[patterns]]
- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../../../30-agents/playbooks/cve-scan]]
- [[../../../30-agents/playbooks/dependency-audit]]
