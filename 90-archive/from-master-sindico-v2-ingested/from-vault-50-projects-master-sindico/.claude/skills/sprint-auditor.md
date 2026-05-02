# Skill: Sprint Auditor (pós-sprint review loop)

Skill acionada **ao fim de cada sprint** (ou no início do próximo, antes de qualquer task nova) para revisar sistematicamente o que foi entregue, abrir findings em `.kiro/AUDIT.md` e **bloquear** o agente `task-executor` de pegar nova task enquanto houver itens abertos.

Esta skill **não substitui** o `task-executor` — ela convive. O ciclo virtuoso:

```
task-executor (implementa) → ... → fim do sprint
                                     ↓
                              sprint-auditor (revisa, abre items)
                                     ↓
                              task-executor (resolve items, marca ✅)
                                     ↓
                              sprint-auditor (valida que tudo ✅, libera)
                                     ↓
                              task-executor (próximo sprint)
```

## Referências obrigatórias

- `.claude/skills/go-review.md` — check-list de revisão por prioridade (dependency rule → errors → padrões → DDD → segurança → antipatterns do legacy)
- `.kiro/steering/do-dont.md` — regras duras consolidadas
- `.kiro/steering/autonomous-execution.md` — protocolo de decisão em 4 fases + 5 fontes obrigatórias. Ao classificar severidade ou propor fix em finding ambíguo, o auditor **obedece** este protocolo: não inventa severidade, pesquisa.
- `.claude/skills/dual-validate.md` — quando o finding envolve decisão arquitetural de alto impacto (ex: "esta regra ABAC precisa ser reescrita inteira?"), o auditor spawna dual-validate em Opus+Opus antes de abrir o item em `AUDIT.md`.
- `.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` — armadilhas herdadas do legacy
- `.kiro/AUDIT.md` — arquivo vivo que esta skill lê e escreve

## Gatilhos de invocação

Invoque esta skill via `/sprint-auditor` quando:

1. `tasks.md` do sprint atual marcar todas as tasks do Dev1 como ✅.
2. Última entrada em `STATE.md` sinalizar conclusão de sprint.
3. Usuário explicitamente pedir auditoria.
4. **Obrigatório**: início de sessão em que `AUDIT.md` tenha itens `🔴 Aberto` / `🟡 Em andamento` — a própria skill faz o fechamento antes de liberar.

## Protocolo de execução

### Fase A — Levantamento

1. Ler `.kiro/AUDIT.md` — saber o que já foi apontado e ainda não resolvido.
2. Ler `.kiro/STATE.md` — saber decisões e dívidas já conscientes (não duplicar).
3. `git log --since="{data-da-última-rodada}" --oneline` — enumerar commits do sprint.
4. `git diff {hash-início-sprint}..HEAD --stat` — ver escopo tocado.
5. Ler `.kiro/specs/master-sindico/tasks/sprint-{N}.md` — contrato do sprint.

### Fase B — Revisão sistemática

Para **cada** módulo tocado no sprint, aplicar **em ordem** os check-points de `go-review.md`. Usar o subagent `feature-dev:code-reviewer` para camadas grandes (>10 arquivos) — cada subagente revisa 1 camada e retorna só o resumo de achados (não polui o contexto do orquestrador).

Check-points obrigatórios por ordem (vermelho short-circuita os demais):

1. **Dependency rule** — `domain/` puro? `shared/middleware/` sem import de `modules/`? Handlers sem Gin?
2. **Error handling** — `_ = fn()` ausente? `errors.Is/As` em vez de string match?
3. **Padrões Go** — early returns, nomes completos, `int64` centavos, `context.Context` primeiro param.
4. **DDD / Clean Arch** — entidades com estado privado? VOs imutáveis? Um UC por arquivo? Compile-time check em toda implementação de interface?
5. **Segurança** — webhook signature antes do parse? Validação server-side? Tokens mascarados em log? CPF/CNPJ mascarados?
6. **Antipatterns do legacy** (A1–A12) — ver tabela em `do-dont.md`.
7. **Threshold de coverage** — threshold por camada atingido? (95/90/85/75)
8. **Spec alignment** — o que foi implementado bate com `requirements/` + `design/`?
9. **Cross-module isolation** — integrações novas passam por `shared/types/` (ex: `ITrialChecker`, `IQuotaConsumer`, `IValidationSubmitter`, `ITimelinePublisher`)? Zero import direto entre `modules/*`?

### Fase C — Materialização de findings em `AUDIT.md`

Para cada finding:

1. Gerar ID sequencial `A-###` (continuar numeração do último ID no Histórico).
2. Classificar severidade:
   - `critical` — bloqueia deploy (SQL injection, race em quota, ABAC bypass, webhook sem signature, etc)
   - `high` — bug funcional, dependency rule quebrada, erro silenciado em caminho crítico
   - `medium` — código confuso, duplicação, função longa demais, nome abreviado
   - `low` — cosmético, comentário obsoleto, variável não usada
3. Apontar **raiz**, não sintoma. "Handler escreve resposta de erro direto em vez de `ctx.SetError`" é sintoma; "ausência de teste de unidade no handler X permitiu que o padrão se instalasse" é raiz.
4. Propor fix em uma frase. Não implementar o fix nesta skill — apenas documentar.
5. Adicionar à seção `🔴 Aberto` no `AUDIT.md` com data ISO + sprint de origem.

### Fase D — Validação (apenas leitura)

```bash
go build ./...
go vet ./...
go test -race -count=1 ./...
go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
gosec -severity high ./...
govulncheck ./...
```

Qualquer gate vermelho **vira um finding** `critical` no AUDIT. Não rodar fix — só reportar.

### Fase E — Síntese de relatório

Ao final da rodada, retornar ao usuário:

```
Sprint auditor — Sprint {N} ({data})
  Commits analisados:    {X}
  Arquivos revisados:    {Y}
  Findings novos:        {Z} (critical: {a} | high: {b} | medium: {c} | low: {d})

  🔴 Bloqueadores para próximo sprint: {lista IDs}
  🟡 Já em andamento:                 {lista IDs}
  ✅ Resolvidos nesta rodada:         {lista IDs}

  Próxima ação: {task-executor resolve open items | próximo sprint liberado}
```

## Modo "verificar fechamento"

Quando invocada e `AUDIT.md` tem apenas items `🟡 Em andamento` / `✅ Resolvidos nesta rodada` (sem `🔴`):

1. Para cada item `✅`, verificar que o fix de fato resolveu — ler o arquivo alvo, confirmar que o padrão antigo não reapareceu.
2. Rodar novamente gates Fase D. Se tudo verde:
3. Mover todos os `✅` para Histórico com timestamp + hash do commit de fix.
4. Limpar seção "Em Andamento" (se restar algo lá, volta pra 🔴 com nota).
5. Reportar: "AUDIT fechado. Próximo sprint liberado."

## Regras duras

- **Nunca** implementar o fix na skill. Aponta, não corrige. Corrigir é responsabilidade do `task-executor` na próxima invocação.
- **Nunca** apagar entrada histórica de `AUDIT.md`. Só mover `🔴 → 🟡 → ✅ → Histórico`.
- **Nunca** duplicar um item já em STATE como DT-###. Se em STATE: referenciar + nota, não nova entrada.
- **Nunca** liberar o `task-executor` se houver gate (build/vet/test/coverage/gosec/govulncheck) vermelho no fim da Fase D.

## O que esta skill NÃO faz

- Não edita código de produção.
- Não abre PR nem commita.
- Não atualiza README.md (responsabilidade do `task-executor` na fase Ship — ver `sdd-workflow.md`).
- Não altera spec (`requirements/`, `design/`, `tasks/`) — se finding revela lacuna de spec, registra em AUDIT com severidade `high` e nota "spec-gap"; quem arruma spec é o João na próxima rodada de Discuss.
