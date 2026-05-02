---
title: GSD — Get Shit Done (ergonomia de entrega)
type: concept
tags: [methodology, gsd, execution, shipping]
source: gsd-build/get-shit-done (GitHub)
created: 2026-04-23
aliases: [GSD, Get Shit Done, Ergonomia de entrega]
---

# GSD — Get Shit Done

**GSD** (Get Shit Done) é uma **ergonomia de entrega** mais do que metodologia formal. Origem no repositório [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done). Complementa SDD — enquanto SDD dá o **ciclo** (Discuss → Plan → Execute → Verify → Ship), GSD dá as **heurísticas** que evitam travamento.

> **Filosofia**: progresso > perfeição. Entregar algo **pequeno e frequente** que funciona supera entregar "tudo que o projeto precisa" meses depois.

## Princípios canônicos

### 1. Ship small, ship often

- **Tasks pequenas** (1-2 dias idealmente, nunca mais de 1 sprint)
- **Merge semanal no mínimo**; diário se possível
- **Feature flags** pra ship atrás de toggle — separa *deploy* de *release*
- **MVP real**, não "MVP" que significa "produto completo faltando detalhes"

Tasks gigantes quebram por:
- Dificuldade de review (PR > 500 linhas pede atalhos)
- Merge conflicts escalam exponencialmente
- Feedback chega tarde — correção é cara
- Risco de "scope creep" sem fim

### 2. Definir "Done" antes de começar

**DoD (Definition of Done)** é **contrato explícito**. Sem DoD, task "nunca termina" — sempre tem um detalhe a mais.

Template:
```markdown
## Task 1.3 — Stripe Adapter
Estima: 8h
DoD:
- [ ] Interface IPaymentGateway em domain/providers/
- [ ] StripeGateway em infrastructure/providers/ implementa IPaymentGateway
- [ ] Webhook signature validada antes de parse
- [ ] Coverage em infrastructure/providers/ ≥ 85%
- [ ] Integration test com sandbox real (sk_test_*)
- [ ] README do backend atualizado (seção Integrações)
- [ ] Gates verdes: build, vet, test -race, gosec, govulncheck
```

Nada **além** dessa lista entra. Nada **aquém** sai.

### 3. `<verify>` declarativo — compromisso, não sugestão

Na fase de **Plan** (antes de escrever código), declarar explicitamente:
- Quais testes vão existir
- Quais gates vão passar
- Qual coverage vai atingir

Na fase de **Verify**, medir:
- Tudo que foi prometido existe e passa?

Declarar `<verify>` obriga **pensar** antes de codar. "Descobrir" na fase de teste o que vai testar é sintoma de plano fraco.

```xml
<verify>
  - TestStripeGateway_CreateSubscription_WithTrial passa
  - TestStripeGateway_VerifyWebhookSignature_InvalidSignature retorna erro
  - Coverage em infrastructure/providers/ ≥ 85%
  - gosec -severity high → 0 findings
  - Integration test via sandbox: cria customer, cria subscription, cancela
</verify>
```

Ver [[sdd-workflow]].

### 4. STATE.md como arquivo vivo

Tudo que **não cabe em tasks.md (estático)** vai em STATE.md:

- **Bloqueadores** no momento (com responsável, data, impacto)
- **Decisões em aberto** (alternativas, prazo pra resolver)
- **Dívidas técnicas** aceitas (sprint alvo pra pagar)
- **Questões pendentes de produto** que afetam spec

STATE não é relatório — é **cérebro da sprint**. Toda sessão do agente começa lendo STATE. Descobriu bloqueador novo? Registra em STATE **imediatamente**. Sem isso, próxima sessão tromba no mesmo muro.

Formato:
```markdown
### D-042 — Cache TTL do ABAC
- **Status**: 🟡 em avaliação
- **Abriu**: 2026-04-15 (sprint 5)
- **Decisão**: 5 min fixo vs invalidação via webhook
- **Impact**: mudança de plano demora até 5min pra refletir no ABAC
- **Fontes**: Redis docs, Uber engineering blog
- **Prazo**: sprint 7

### DT-003 — Sem circuit breaker Zitadel
- **Status**: 🟢 dívida aceita
- **Sprint alvo**: 7 (infra)
- **Impact**: Zitadel down = 503 sem backoff
```

### 5. "Se trava, documenta"

GSD manda **não ficar preso**. Regra: se travou > 2h em algo, **documenta o travamento** (em STATE ou numa nota) e:
- Pede ajuda (dual-check, research-loop)
- Muda pra próxima task viável
- Escala pro humano se nada resolve

Travamento silencioso = perda de velocidade invisível. Travamento documentado = dado de processo.

### 6. Matar tasks zumbi

Task que ficou semanas "em andamento" sem merge:
- **Ou quebra em subtarefas** (era grande demais)
- **Ou arquiva** (virou irrelevante)
- **Ou reinicia** com escopo repensado

"Task aberta há 30 dias" é **mentira** — ninguém está trabalhando nela ativamente. Status irreal polui dashboard.

### 7. Feedback imediato, não adiado

Rodar o gate **durante** a task, não só no final. Ex:
- `go build ./...` a cada save (via IDE/watcher)
- `go test` do pacote alterado toda mudança significativa
- `go vet` no pre-commit hook
- CI em cada push

Quanto mais tarde o feedback, mais cara a correção. **Ship often** obriga feedback rápido.

### 8. Batch work = context switch minimizado

- Agrupar **tarefas do mesmo tipo** em janela: fazer todas as migrations juntas, todos os handlers juntos
- **Não misturar** refactor com feature nova (PR separado)
- **Drain tasks abertas** antes de começar nova epic — WIP baixo

## GSD × SDD — complementaridade

**SDD** responde *como* — ciclo rígido Discuss→Plan→Execute→Verify→Ship.
**GSD** responde *como não travar* — ergonomia dentro do ciclo.

| Fase SDD | Heurística GSD |
|---|---|
| Discuss | Ler spec antes de tudo; se ambíguo, **atualizar spec antes de avançar** |
| Plan | `<verify>` declarativo; DoD explícito |
| Execute | Tasks pequenas; feedback imediato; documentar travamento |
| Verify | Gates duros; sem exceção silenciosa |
| Ship | Merge frequente; feature flag se necessário |

Time que faz SDD sem GSD: cerimônia demais, entrega pouco. Time que faz GSD sem SDD: entrega caos — código "funcional" incoerente com negócio.

## "Just do it" não é GSD

GSD **não** é "código rápido, refactor depois". Isso é o oposto — gera dívida que trava tudo em 3 sprints.

GSD **é**:
- Task pequena **completa** (com teste, com doc)
- Ship **agora** o que **serve**
- Não gold-plating — mas não skip-plating também

## Integração com LLM/agentes

Agentes LLM (Planner/Worker/Reviewer) operam melhor sob GSD:
- **Tasks pequenas** = contexto cabe no prompt sem truncar
- **DoD explícito** = agente sabe quando parar
- **STATE.md vivo** = agente retoma sessão sem recomeçar do zero
- **`<verify>` declarativo** = agente pode verificar próprio trabalho

Ver [[../../30-agents/autonomous-execution]] e [[../../30-agents/playbooks/plan-and-execute]].

## Anti-patterns GSD

- **"Preciso de tempo pra pensar antes de codar"**: Plan é 15min, não 2 dias. Se precisa 2 dias, spec está incompleta — atualizar spec.
- **"Vou mergear quando estiver completo"**: merge pequeno e frequente > merge grande e raro. Feature flag se precisa esconder.
- **"Meu ticket está quase pronto"**: "quase" por > 3 dias = faltando quebrar em subtasks.
- **"Vou ajustar o teste depois que o código estabilizar"**: teste vem junto ou no mesmo commit. Depois = nunca.
- **"Dívida técnica zero"**: zero dívida é ilusão. Dívida **registrada e priorizada** (DT-###) > dívida invisível.
- **"Task gigante, mas bem planejada"**: planejamento não substitui tamanho. Dividir.

## Checklist GSD

- [ ] Task atual tem DoD explícito?
- [ ] Task está abaixo de 1 sprint de estimativa?
- [ ] `<verify>` foi declarado antes do código?
- [ ] STATE.md foi atualizado com bloqueadores/decisões novas?
- [ ] Última merge foi há menos de 1 semana?
- [ ] WIP (tasks em andamento) < 3 por dev?
- [ ] Travamentos > 2h foram documentados?

## Referências

- [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done) — origem
- Reinertsen, Donald. *The Principles of Product Development Flow* (2009) — WIP, batch size
- *Accelerate* (Forsgren, Humble, Kim, 2018) — DORA metrics, deploy frequency
- *The Phoenix Project* (Kim, 2013) — work-in-progress e teoria das restrições

## Links

- [[_moc]]
- [[sdd-workflow]]
- [[spec-driven-development]]
- [[sdd-maturity-checklist]]
- [[sdd-adoption-gaps]]
- [[legacy-as-reference]]
- [[../../30-agents/autonomous-execution]]
- [[../../30-agents/playbooks/plan-and-execute]]
