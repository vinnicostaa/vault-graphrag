# Skill: Task Executor (SDD)

Quando solicitado a implementar uma task de `.kiro/specs/master-sindico/tasks/`, siga o **protocolo SDD de 5 fases** definido em `.kiro/steering/sdd-workflow.md`.

**Se operando em modo autônomo** (usuário autorizou "vou dormir", "autonomia total", "você decide", etc — ver `.kiro/steering/autonomous-execution.md`): dúvida técnica **não** pausa o fluxo. Decisões não-triviais seguem o protocolo de 4 fases (5 fontes obrigatórias + dual-agent em Opus para alto impacto via `.claude/skills/dual-validate.md`) e ficam registradas em `.kiro/STATE.md` como `D-0XX`. As únicas pausas válidas são as 5 categorias destrutivas (ver `autonomous-execution.md` e `AGENTS.md`).

```
Discuss  →  Plan          →  Execute   →  Verify              →  Ship
contexto    <verify>         código +      build+vet+test+cov    PR +
+ spec      + subagents?     tests         + gosec + vuln        Release Notes
```

## Gate 0 — `AUDIT.md` bloqueia task nova

**Primeira coisa a fazer em qualquer sessão**: abrir `.kiro/AUDIT.md`.

- Se houver item `🔴 Aberto` ou `🟡 Em andamento`: **pausar** o fluxo normal da task. Tratar cada item como micro-task seguindo o próprio ciclo de 5 fases (escopo reduzido — tipicamente 1 commit). Marcar `🟡 Em andamento` ao começar, `✅ Resolvido` quando build+vet+test+coverage passarem para o fix.
- Quando tudo no AUDIT for `✅`: invocar `/sprint-auditor` em modo fechamento (valida fixes + move ✅ → Histórico + libera próximo sprint).
- Só então puxar a próxima task de `tasks.md`.

Ver `.claude/skills/sprint-auditor.md` para o protocolo completo.

## Preparação obrigatória (fase Discuss)

Antes de tocar em qualquer arquivo, leia nesta ordem:

1. Task completa em `.kiro/specs/master-sindico/tasks/{sprint}.md`
2. Requirements relacionados em `.kiro/specs/master-sindico/requirements/*.md`
3. Design relacionado em `.kiro/specs/master-sindico/design/*.md`
4. `.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` — o que deu errado no legacy
5. `.kiro/STATE.md` — bloqueadores e decisões em aberto
6. `.kiro/AUDIT.md` — findings pendentes de sprint auditor (gate 0 acima)
7. Código existente do módulo — arquivos a modificar + vizinhos para padrão
8. Steering relevante ao tipo de task (ver tabela em `AGENTS.md`)

Se algo está incompleto ou ambíguo: **pare e atualize o spec antes de implementar**.

## Plano com `<verify>` antes de codar

Declare explicitamente:
- **Arquivos a criar** (lista completa)
- **Arquivos a modificar** (com o que muda em cada)
- **Arquivos que NÃO serão tocados** (confirma escopo)
- **Dependências**: task anterior concluída? migration aplicada?
- **Docs oficiais consultadas** via Context7 MCP (ID da lib + topic)
- **`<verify>`**: quais testes passam, qual coverage alvo por arquivo/pacote

Se a task depende de outra não concluída, pare e informe.

## Ordem de implementação (fase Execute)

Detalhe em `.kiro/steering/sdd-workflow.md`. Resumo:

```
1. Migration SQL (goose) + CHECK constraints
2. sqlc query + sqlc generate
3. Domain: entities + value objects + repository interfaces + events
4. Application: use cases (CQRS) + mappers
5. Infrastructure: repository impl + provider externo (SDK isolado)
6. HTTP: handler (contracts.HandlerFunc) + routes
7. Wire-up: module.go + cmd/api/main.go
8. Testes (unit + integração com testcontainers-go)
```

Para tasks grandes (módulo inteiro): delegue por camada a subagentes (**Orchestrator-Worker** em `sdd-workflow.md`).

## Validação (fase Verify)

```bash
go build ./...
go vet ./...
sqlc generate && git diff --exit-code internal/**/generated
go test -race -count=1 ./...
go test -tags=integration -count=1 ./...
go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
gosec -severity high ./...
govulncheck ./...
```

**Gates duros**: qualquer um falhando → task não é considerada pronta. Exceção exige documentação em `.kiro/STATE.md` com prazo.

## Checklist final (aplicar `.kiro/steering/do-dont.md`)

Pontos de atenção mais comuns:

- [ ] Compile-time interface checks: `var _ IFoo = (*Impl)(nil)`
- [ ] `_ = fn()` ausente em operações falíveis
- [ ] `float64` ausente em valores monetários (use `int64` centavos)
- [ ] Gin importado apenas em `internal/server/`
- [ ] Early returns / guard clauses — sem if/else aninhado
- [ ] `context.Context` como primeiro parâmetro em I/O
- [ ] Nomes completos (sem `uow`, `condo`, `q`)
- [ ] Comentários explicam **porquê**, não o quê
- [ ] Tenant isolation: queries com `WHERE condominium_id = $N`
- [ ] `WHERE deleted_at IS NULL` em tabelas com soft delete
- [ ] Domain events em vez de chamada direta entre bounded contexts
- [ ] Webhook: signature validada **antes** de parse
- [ ] Testes de integração com testcontainers-go (tag `integration`)
- [ ] Coverage atende threshold por camada
- [ ] **README.md atualizado** do sub-projeto impactado (backend/web/app) quando a mudança é feature-level — ver `steering/sdd-workflow.md` fase Ship para escopo. Bug-fix pontual não atualiza README; feature nova **sempre** atualiza

## O que NÃO fazer

- Criar arquivos fora do escopo da task
- Adicionar features não solicitadas ("enquanto estou aqui...")
- Mudar arquitetura sem discussão explícita
- Implementar a próxima task automaticamente
- Ignorar erro de compilação — corrija antes de avançar
- Pular a fase Discuss porque "já sei o que fazer"
- Usar SDK externo sem consultar Context7 primeiro
