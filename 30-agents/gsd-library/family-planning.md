---
title: Família — Planning (GSD)
type: note
tags:
  - agents
  - gsd
  - planning
  - family
created: '2026-04-24T00:00:00.000Z'
updated: '2026-04-24T00:00:00.000Z'
aliases:
  - Planning agents
  - family-planning
---

# Família GSD — Planning

Agrupa os agents que **decidem o que fazer antes de fazer**: transformam visão de produto e decisões do usuário em artefatos executáveis (ROADMAP, PLAN, intel, escolha de stack). No ciclo GSD, Planning é o estágio em que ambiguidade vira contrato: goal-backward extrai *truths* observáveis, cobertura de requirements é validada, dependências viram ondas paralelas. Nenhum byte de código é escrito aqui — o output é instrução suficiente pra um executor aplicar sem reinterpretar.

A família opera em dois níveis: **macro** (roadmapper define fases; framework-selector trava stack AI; intel-updater materializa o mapa do código) e **micro** (planner decompõe uma fase em plans paralelos; plan-checker verifica antes da execução que os plans vão entregar o goal).

## 5 agents na família

- [[card-gsd-roadmapper]] — converte requirements em fases com cobertura 100% e success criteria observáveis.
- [[card-gsd-framework-selector]] — entrevista curta + hard constraints para escolher framework AI/LLM.
- [[card-gsd-intel-updater]] — materializa intel estrutural (`stack/files/apis/deps/arch`) baseado em evidência real.
- [[card-gsd-planner]] — decompõe fase em PLAN.md com grafo de dependências, ondas e `must_haves`.
- [[card-gsd-plan-checker]] — verifica adversarialmente que PLAN.md vai entregar o goal antes da execução.

## Quando usar esta família

1. **Bootstrap de projeto novo**: `roadmapper` transforma REQUIREMENTS.md em ROADMAP com N fases cobrindo 100% dos requirements; `framework-selector` só entra se o projeto é AI/LLM e o stack ainda está aberto. Output alimenta primeiro ciclo do Planner.
2. **Início de fase nova**: `intel-updater` refresca o mapa do código (stack/files/apis/deps drift em toda fase); `planner` decompõe a fase em PLAN.md com waves de paralelismo; `plan-checker` audita o PLAN antes do Worker ser autorizado a executar.
3. **Replan após AUDIT 🔴**: findings bloqueantes exigem novas tasks. `planner --gaps` recebe lista de gaps e emite PLAN-FIX.md incremental em vez de reescrever o PLAN original — preserva rastreabilidade.
4. **Mudança de stack travada em D-###**: `framework-selector` não é só pra AI — o padrão de entrevista curta + hard constraints aplica a qualquer escolha travada (ex: mensageria, cache, orchestrador). Output vira ADR + entry em STATE.md.
5. **Checagem pré-sprint**: antes do Orchestrator iniciar sprint, `plan-checker` verifica que todo requirement ID aparece em algum plan, nenhuma task é órfã e todas as waves têm at least 1 task sem blockers abertos.

## Quando NÃO usar

- **Task única e clara ("renomeie X → Y em 3 arquivos")**: Planner full é overkill. Use o Worker com instrução direta; o próprio commit message vira o registro.
- **Exploração aberta sem goal ainda**: é família Research, não Planning. Planning parte de requirements; Research parte de incerteza.
- **Fix de bug conhecido com hotfix curto**: vai direto pro Executor/Fixer. Planner é pra decomposição multi-arquivo/multi-wave, não pra mudança pontual.
- **Decisão já travada em D-### há 2 semanas**: não re-invocar `framework-selector` — isso gera re-decisão e risco de inconsistência com código já escrito. Consultar STATE.md, confirmar, seguir.
- **Projeto maduro em modo manutenção**: `intel-updater` é exagero se nada mudou desde último ciclo; só refrescar quando grep mostra drift real (novos arquivos, novas deps, novas APIs).

## Interação entre agents da família (pipeline interno)

```
  REQUIREMENTS.md ──► roadmapper ──► ROADMAP.md (fases + success criteria)
                                          │
                                          ▼
                     intel-updater ──► INTEL.md (snapshot stack/files/apis)
                                          │
                   (opcional: AI) framework-selector ──► AI-SPEC §1-2 + STATE D-###
                                          │
                                          ▼
                                      planner ──► PLAN.md (waves + tasks + must_haves)
                                          │
                                          ▼
                                  plan-checker ──► VERDICT: APPROVED / REWORK
                                          │
                                          ▼ (se APPROVED)
                                      Executor (família Executor toma conta)
```

`intel-updater` é side-effect-free em relação ao ROADMAP — pode rodar a qualquer momento. `planner` **depende** de intel atualizado (senão, vai planejar com mapa drifted). `plan-checker` é gate duro antes do Executor ser spawned; rejeição gera `planner --rework` em vez de skip.

## Padrões reutilizáveis

- **Goal-backward como contrato**: todo agent de Planning parte de "o que deve ser TRUE para o usuário?" antes de tocar em tarefas. Verdades viram artefatos, artefatos viram wiring, wiring vira key_links. O plano existe pra satisfazer truths — não o contrário. O mesmo raciocínio é a base de *Inverted Funnel* em OKRs: partir do impacto desejado e derivar tarefas, nunca o inverso.
- **Fidelidade de decisão é não-negociável**: decisões travadas (D-XX) não podem ser simplificadas silenciosamente. Linguagem proibida: "v1", "static for now", "hardcoded", "future enhancement". Se não cabe, retorna `PHASE SPLIT RECOMMENDED` — planner não tem autoridade para julgar algo "difícil demais".
- **Cobertura explícita**: roadmapper garante 100% de requirements mapeados; planner garante que todo requirement ID aparece em pelo menos um plan; plan-checker verifica cobertura como dimensão própria. Órfão é blocker, nunca warning. Espelha o princípio de *traceability matrix* do IEEE 830 e ISO 26262.
- **Evidência > narrativa**: intel-updater só escreve o que Glob/Read confirmam; plan-checker não aceita task vaga; framework-selector escaneia o codebase antes de perguntar. Claims sem path = claim inválido.
- **Checker adversarial antes da execução**: introduzir `plan-checker` antes do Executor é equivalente a um lint estrutural do plano. Mais barato falhar aqui do que 3 waves depois; mesma lógica do *shift-left testing*.

## Correspondência com nosso pipeline

No pipeline Planner / Worker / Reviewer / Orchestrator do `automation-agents`, esta família se concentra no **Planner** — é literalmente o cérebro de planejamento. O `gsd-planner` é o contrato de referência para nosso Planner decompor fases em tarefas executáveis pelo Worker; o `gsd-roadmapper` inspira como o Planner deve encarar a derivação do ROADMAP a partir de `REQUIREMENTS.md` de um projeto; o `gsd-framework-selector` mostra como transformar escolha de stack em decisão rastreável (alinhado com nosso protocolo D-### em STATE.md).

Dois agents têm viés de **Orchestrator** em vez de Planner puro: `gsd-intel-updater` é invocado como serviço de contexto que outros agents consultam (cabe em `30-agents/orchestrator/` como rotina de housekeeping), e `gsd-roadmapper` opera no bootstrap do projeto (papel de orchestrator ao inicializar STATE + ROADMAP + traceability). O `gsd-plan-checker` é o único com natureza de **Reviewer** — executa antes da fase começar, usando a mesma mentalidade adversarial do Reviewer do `automation-agents`.

**Insight operacional**: a separação planner → plan-checker é exatamente o Dual-check do nosso CLAUDE.md §5 aplicado ao artefato PLAN.md. A família sugere tornar essa duplicata explícita no nosso pipeline — emitir PLAN + spawn plan-checker automaticamente sempre que o Planner decompõe fase, em vez de deixar como opcional.

## Patterns atemporais extraídos

1. **Goal-backward + truth enumeration**: metodologia transferível pra qualquer contexto de planejamento — product spec, sprint planning, arquitetura. Se você não consegue enumerar as "truths" que seriam observáveis no fim, o goal está mal definido.
2. **Plan-as-contract com cobertura verificável**: tratar o plano como contrato auditável contra uma matriz de requirements previne scope creep silencioso. Replicável em qualquer workflow que tenha `ROADMAP → PLAN → EXECUÇÃO`.
3. **Gate adversarial antes da execução**: barato falhar no plano, caro falhar no commit. Plan-checker implementa o mesmo princípio que precommit hooks, mas no nível da intenção — é o "lint do plano".
4. **Intel estrutural vs doc narrativo**: INTEL.md é um mapa (stack/apis/deps em tabela), não uma descrição. Separar intel (dado estrutural) de doc (narrativa) evita drift — intel regenera a partir do código, doc não.

## Links externos

- [GitHub Spec Kit — SDD](https://github.com/github/spec-kit) — ciclo Discuss/Plan/Execute/Verify casa com roadmapper + planner + plan-checker + verifier
- [Google SRE — Error Budget Policy](https://sre.google/sre-book/embracing-risk/) — raciocínio sobre "o que pode falhar antes de estancar release" espelha plan-checker
- [Shape Up (Basecamp)](https://basecamp.com/shapeup) — pitch (roadmapper) + shaping (planner) + betting table (plan-checker)
- [Amazon — Working Backwards](https://www.amazon.jobs/en/landing_pages/working-backwards) — PR-FAQ é o primo do goal-backward do roadmapper
- [IEEE 830-1998 — Software Requirements Specifications](https://standards.ieee.org/ieee/830/1222/) — cobertura 100% de requirements espelha traceability matrix canônica

## Fonte

- [[../../60-sources/get-shit-done/_toc]]

## Links

- [[_moc]]
- [[pipeline-mapping]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
- [[../../10-knowledge/methodology/sdd-workflow]]
