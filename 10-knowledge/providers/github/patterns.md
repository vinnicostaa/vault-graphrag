---
title: GitHub MCP — Patterns
type: note
tags: [provider, mcp, github, patterns, scm]
source: https://docs.github.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [GitHub MCP Patterns]
---

# GitHub MCP — Patterns

Padrões atemporais de uso do MCP GitHub em automação.

## 1. Read via MCP, write via `gh` CLI

Regra heurística canônica (ver [[overview#MCP vs gh CLI]]):
- **Read** (issues, PRs, commits, diffs, reviews): MCP — retorno estruturado JSON consumível.
- **Write** (merge, close, release, tag): `gh` CLI — audit via shell history + menor chance de batch errado.

Exceção: bot de automação 24×7 em que shell history é irrelevante — MCP para tudo, com rollback script à mão.

## 2. Paginação explícita em queries grandes

APIs de list (`list_commits`, `list_pull_requests`, `list_issues`) retornam 30 por página default. Para repos grandes:

```
list_commits(owner, repo, per_page=100, page=1)
list_commits(owner, repo, per_page=100, page=2)
...
```

Nunca assumir "primeiros 30 cobrem o escopo". Pagar até exaustão em análises completas.

## 3. GraphQL para queries multi-entidade

REST força N chamadas para montar contexto. GraphQL monta em uma:

```graphql
query {
  repository(owner: "X", name: "Y") {
    pullRequest(number: 42) {
      title, body, state
      reviews(first: 10) { nodes { author { login }, state } }
      files(first: 100) { nodes { path, additions, deletions } }
    }
  }
}
```

1 chamada vs 3+ REST. Usar quando disponível no servidor MCP.

## 4. Authenticação com menor privilégio

- PAT **fine-grained** com escopo por repo + permissão por feature (ex.: `Issues: Read`, `Metadata: Read`, nada mais).
- GitHub App com `permissions:` minimal no `app.yml`.
- **Nunca** `repo` full scope + `admin:org` só pra "segurança" — aumenta blast radius de vazamento.

## 5. Rate limit awareness + backoff

5000 req/h autenticado; 60 req/h não autenticado.

- Headers `X-RateLimit-Remaining` e `X-RateLimit-Reset` em cada resposta.
- Se `Remaining < 100`: começar backoff exponencial.
- Batch queries: prefera GraphQL (1 chamada = 1 rate-limit-unit, mesmo com N entidades).

## 6. Trust boundary em conteúdo de issue/PR

Texto em issue/PR/comment vem de **usuários externos**. Trust model:

- Markdown pode conter imagens remotas (tracking beacons).
- Links em comentários podem ser phishing.
- Código em blocos pode conter prompt injection em repo público.

Para agente: tratar conteúdo de issue/PR como **input não-confiável**. Nunca executar `curl` em URLs citadas; não inferir credenciais de "assinaturas" de usuário.

## 7. Webhook signing em integrações

Quando aplicação recebe webhook do GitHub:
- Verificar header `X-Hub-Signature-256` (HMAC SHA-256 do body + secret).
- **Antes de parsear** o body — comparar em constant-time.

Antipattern: aceitar body como válido, parsear, agir. Ver [[../../runtime-antipatterns/_moc|OP-023]].

## 8. Advisory cross-reference no CVE workflow

`list_global_advisories(ecosystem="go", severity=["HIGH","CRITICAL"])` + cruzar com `go.sum`. Integra com [[../../../30-agents/playbooks/cve-scan]]:

1. `govulncheck ./...` → lista pacotes vulneráveis.
2. Para cada: `get_advisory(ghsa_id)` via GitHub MCP → mitigação + fix version.
3. Priorizar por CVSS + reachability.

## 9. PR auto-review bot: review → comment, não merge

Bot de review não faz merge. Cria review:

```
create_pull_request_review(
  event: "REQUEST_CHANGES" | "APPROVE" | "COMMENT",
  body: "review markdown",
  comments: [ { path, line, body } ]
)
```

Humano decide merge. Previne bot com bug aprovar PR malicioso.

## 10. Actions workflow como contrato — `gh workflow run` com inputs

Para workflows com `workflow_dispatch`: trigger programático via `gh workflow run <name> -f key=value`. Útil em:
- Deploy manual programático (staging → prod com input do operador).
- Re-run de job com parâmetro alterado.
- Integração Claude Code → workflow custom.

MCP pode ter `trigger_workflow_dispatch` direto; conferir servidor.

## Links

- [[_moc]]
- [[overview]]
- [[../_moc]]
- [[../../../30-agents/playbooks/cve-scan]]
- [[../../../30-agents/playbooks/dependency-audit]]
- [[../../security/cve-response-workflow]]
- [[../../runtime-antipatterns/_moc]] — OP-023 (webhook sem HMAC)
