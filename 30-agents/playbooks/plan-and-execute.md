---
title: Playbook — Plan-and-Execute
type: playbook
tags: [playbook, execution, sdd]
created: 2026-04-22
---

# Playbook — Plan-and-Execute

Para qualquer task **não-trivial** (≥ 3 arquivos, ≥ 30 linhas, ou qualquer toque em domínio/segurança), o agente escreve um plano explícito **antes** de editar código. Tasks triviais (typo, rename contido) podem pular.

Operacionaliza a **Fase 2 (Plan)** de [[../../10-knowledge/methodology/sdd-workflow]].

## Estrutura do plano

```xml
<plan task="<task-id>: <título>">
  <contexto>
    <!-- 1 parágrafo: o que precisa ser feito e por quê.
         Links pra spec, requirement, AUDIT/STATE se aplicável -->
  </contexto>

  <arquivos>
    <!-- cada linha: <path> [novo|editar|deletar] — motivo -->
  </arquivos>

  <ordem-de-implementacao>
    <!-- seguindo a ordem canônica do sdd-workflow:
         migration → query → entity → repo interface → domain event →
         use case → mapper → repo impl → provider externo → handler → wire-up → testes -->
  </ordem-de-implementacao>

  <docs-consultadas>
    <!-- lib@versão: topic, URL, data
         Context7: resolve-library-id(...) → get-library-docs(...) -->
  </docs-consultadas>

  <patterns-aplicados>
    <!-- linka 10-knowledge/patterns/<...>.md explicitando por quê se aplica aqui -->
  </patterns-aplicados>

  <verify>
    <!-- critérios de aceite NÃO-VAGOS:
         - Teste unitário X cobre comportamento Y
         - Teste integração (tag:integration) Z cobre W
         - Coverage domain ≥ 95% nesse módulo
         - go build ✅, go vet ✅, gosec -severity high ✅, govulncheck ✅
         - Linha específica do código tem comentário se houver workaround -->
  </verify>

  <riscos>
    <!-- o que pode dar errado; mitigação prevista -->
  </riscos>

  <rollback>
    <!-- como reverter se der errado em prod (linka [[rollback]]) -->
  </rollback>

  <dual-check>
    <!-- sim/não; se sim, preparar pacote pra [[dual-check]] -->
  </dual-check>
</plan>
```

## Regras

1. **Plano precede código**. Se começou a editar sem plano, pare, plane e continue.
2. **`<verify>` é contrato**. Se escreveu "teste X cobre Y", teste X existe e cobre Y ao final. Nada de "depois faço".
3. **Sem SDK "de cabeça"**. Se lib externa está envolvida, `<docs-consultadas>` tem Context7 ou URL oficial — sem isso, não executa.
4. **Plano curto é bom**. Não enfeite. 30 linhas bem escritas > 300 linhas genéricas.
5. **Se plano sai de escopo** (descobre trabalho não previsto), **pare**, registre como subtask, replanee, não invente.

## Critério de "não trivial"

| Trivial (plano opcional) | Não trivial (plano obrigatório) |
|---|---|
| Rename de variável local | Rename que atravessa camadas |
| Fix typo em comment/string | Fix em lógica de negócio |
| Bump de patch sem breaking | Bump de major |
| 1-2 arquivos, < 30 linhas | Qualquer mudança em domain/, application/ ou contratos |
| Test solo faltante | Novo use case / rota / tabela |
| - | Qualquer SDK/integração externa |
| - | Qualquer mudança de schema DB |
| - | Qualquer mexida em middleware/segurança |

## Subagentes pra tasks grandes

Orchestrator-worker pattern. Task com módulo inteiro:

```
Main agent (orquestrador)
 ├── Subagent 1: domain/ — entidades, VOs, repo interfaces, events
 ├── Subagent 2: application/ — use cases CQRS, mappers
 ├── Subagent 3: infrastructure/database/ — repo impl, queries
 └── Subagent 4: infrastructure/http/ — handlers, routes, wire-up
```

Orquestrador:
- Dá a cada subagente: task do spec + arquivos já criados + critério de aceite específico
- Subagente retorna **apenas** resumo + paths criados/modificados
- Orquestrador **não** recebe conteúdo completo (evita poluir contexto)

Usar via `Agent` tool.

## Verificação ao fim

```bash
# Exemplo Go
go build ./...
go vet ./...
go test -race -count=1 ./...
go test -tags=integration -count=1 ./...
# coverage
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out | awk '/total/ {print $3}'
# security
gosec -severity=high ./...
govulncheck ./...
```

Pra TS/JS/Flutter/Rust, adaptar com comandos equivalentes do stack (documentado em `20-stacks/<stack>/`).

## Links

- [[_moc]]
- [[research-loop]]
- [[dual-check]]
- [[rollback]]
- [[../../10-knowledge/methodology/sdd-workflow]]
- [[../../CLAUDE]]
