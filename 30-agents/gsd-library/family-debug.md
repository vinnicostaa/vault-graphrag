---
title: Família — Debug (GSD)
type: note
tags:
  - agents
  - gsd
  - debug
  - family
created: '2026-04-24T00:00:00.000Z'
updated: '2026-04-24T00:00:00.000Z'
aliases:
  - Debug agents
  - family-debug
---

# Família GSD — Debug

Os 2 agents desta família implementam **investigação post-mortem disciplinada** de bugs em codebase vivo. O núcleo é o `gsd-debugger`, que aplica método científico (hipótese falsificável → experimento → evidência → conclusão) e mantém **estado persistente em arquivo** que sobrevive a `/clear` e resets de contexto. O `gsd-debug-session-manager` orquestra o loop multi-ciclo em **contexto isolado**, protegendo o orchestrator principal de poluição por dezenas de hipóteses eliminadas e evidências acumuladas.

A premissa de fundo é que debugging sério **não cabe num único contexto** de LLM: o problema dura mais que a janela, o raciocínio precisa reiniciar sem perder terreno, e o usuário precisa ser consultado em checkpoints sem que a sessão vire monolito. Por isso o arquivo é o cérebro, o manager é o loop, e o debugger é a cabeça científica rodando uma rodada de cada vez.

## 2 agents na família

- [[card-gsd-debugger]] — investigador científico com debug file persistente (Current Focus / Symptoms imutável / Eliminated / Evidence / Resolution) e knowledge base append-only de casos resolvidos.
- [[card-gsd-debug-session-manager]] — roda o loop completo em isolamento, spawna debuggers, dispara specialist skills por `specialist_hint`, aplica TDD gate e devolve summary compacto (≤ 2K tokens) ao orchestrator.

## Quando usar esta família

1. **Bug heisenbug (intermitente, hard-to-reproduce)**: o arquivo persistente registra ciclos sucessivos, eliminações e experimentos reproduzíveis. Sem ele, o agente re-executa investigações já feitas a cada reset de contexto.
2. **Bug cross-layer (frontend → API → DB → queue)**: múltiplos ciclos e múltiplos specialists (DB expert, frontend expert). `debug-session-manager` isola o loop e spawn debuggers com `specialist_hint` apropriado por rodada.
3. **Bug em legado com poucos testes**: método científico força hipótese falsificável — elimina o "tenta isso, tenta aquilo" que adiciona commits sem entender. Cada hipótese rejeitada vira entrada em "Eliminated", nunca repetida.
4. **Session interrompida por timeout/crash**: arquivo `.gsd-debug-<id>.md` sobrevive. Próximo spawn do debugger retoma pelo `next_action` sem perder raciocínio.
5. **Investigação compartilhada entre dev humano e agent**: o arquivo é plain markdown — humano lê, anota no debug file, o agent continua de onde o humano parou.

## Quando NÃO usar

- **Bug óbvio com stack trace claro e fix de 1 linha**: família Executor com `code-fixer` basta. Debugger é overhead sem retorno quando causa é trivial.
- **"Investigação" que é na verdade discovery (o que existe, como está wired)**: família Research (`codebase-mapper`, `assumptions-analyzer`). Debug assume sintoma concreto de defeito; research assume território desconhecido.
- **Verificação que contrato foi cumprido**: família Audit. Debug não compara spec vs impl — ele procura causa raiz de comportamento errado.
- **Otimização de performance sem bug**: é perf-engineering, não debug. Método científico vale, mas o toolkit é profiler/tracer/flame graph, não o debugger agent.
- **Fix já conhecido de outra sessão (cache de KB)**: a knowledge base append-only pode sugerir hipótese candidata — aplicar direto o fix anterior (fast-path), só escalar pra loop completo se o match de keywords não for forte.

## Interação entre agents da família (pipeline interno)

```
  orchestrator principal (contexto A)
         │
         │ spawn (contexto B isolado)
         ▼
  debug-session-manager
         │
         │ cria/lê debug-file.md
         │
         │ spawn debugger (ciclo 1) ──► lê file, forma hipótese, experimento, update file
         │ spawn debugger (ciclo 2) ──► lê file, elimina ou confirma, update file
         │ spawn debugger (ciclo N) ──► Resolution escrita, status=resolved
         │
         │ (opcional) specialist_hint dispara dual-check ou skill específica
         │
         │ consulta knowledge-base.md append-only
         │
         ▼
  summary compacto ≤ 2K tokens ──► orchestrator principal (contexto A)
         │
         ▼
  knowledge-base.md recebe entrada nova (se resolvido)
```

O segredo do isolamento: contexto B pode crescer até 100K tokens de hipóteses eliminadas sem contaminar contexto A. O summary de volta é `{root_cause, fix_applied, files_touched, test_added}` — denso, acionável, sem histórico narrativo.

## Padrões reutilizáveis

1. **Estado persistente como contrato anti-entropia**: um arquivo markdown estruturado (frontmatter + seções com regras de update distintas — overwrite/append/immutable) é **suficiente** para retomar debugging de qualquer ponto após crash/reset. A disciplina de "atualizar antes da ação, não depois" garante que o `next_action` concreto sempre existe. Análogo ao `journal` do PostgreSQL WAL: estado no disco garante retomada.
2. **Hipóteses falsificáveis + reasoning checkpoint mandatório**: antes de tocar código, o agent escreve hypothesis + confirming_evidence + falsification_test + fix_rationale + blind_spots. Se algum campo fica vago, volta a investigar — o próprio checkpoint serve de rubber duck e filtra fixes que atacam sintoma em vez de causa. Alinha com método de *Five Whys* (Toyota) e *blameless postmortem* (Google SRE).
3. **Context isolation para loops longos**: manager rodando em contexto próprio + passagem de **file paths** (não conteúdo inline) para sub-agents evita que orchestrator principal estoure. Padrão aplicável a qualquer workflow multi-ciclo — research-synthesizer (família Research) usa a mesma técnica.
4. **Knowledge base append-only como memória cross-sessão**: sessões resolvidas viram entradas com `Error patterns`; match por keyword overlap (2+ tokens) em novas investigações sugere hipótese candidata — sempre como hipótese, nunca como certeza. Espelha postmortem repository de SRE e runbook library.
5. **TDD gate como closure**: antes de considerar o bug resolved, o manager exige teste que (a) falhava antes do fix, (b) passa com o fix. Sem teste comprovadamente falhando previamente, a "resolução" é ainda hipótese — mesmo princípio de *regression test* canônico.

## Correspondência com nosso pipeline

| GSD agent | Nosso role | Uso |
|---|---|---|
| `gsd-debugger` | **Worker** investigativo (variante científica) | Quando Worker enfrenta bug não trivial, ativa protocolo debug file + hipóteses falsificáveis |
| `gsd-debug-session-manager` | **Orchestrator** de sub-feature | Padrão para qualquer sub-loop que exigiria muitos ciclos de Review/Task — isolar em contexto próprio com summary compacto |

O **Dual Reviewer** (dual-check §5 do contrato do vault) pode ser invocado como `specialist_hint` dentro do loop de debug, revisando a direção do fix em um stack específico antes de aplicar.

**Insight operacional**: o padrão "arquivo como cérebro + manager como loop + worker por ciclo" é replicável além de debug. Pode ser aplicado em qualquer workflow que exceda janela de contexto — migração de codebase longa, destilação de sources enorme, refactoring cross-module. O `automation-agents` ganharia um agent genérico `long-running-session-manager` inspirado diretamente no `debug-session-manager`.

## Patterns atemporais extraídos

1. **Hypothesis-first debugging**: método científico importado pra engenharia de software, com disciplina de escrever a hipótese **antes** de rodar experimento. Anti-padrão oposto (shotgun debugging, "muda X e vê se resolve") gera commits sem aprendizado.
2. **Estado no disco para processos de longa duração**: qualquer workflow que excede a janela de contexto do agente precisa de estado persistente com regras de update claras (overwrite vs append vs immutable). Padrão canônico em sistemas resilientes — WAL, event sourcing, redo logs.
3. **Isolamento de contexto para paralelismo seguro**: manager spawn em contexto isolado com summary compacto de retorno. Mesma ideia de process boundary em OS: cada processo tem heap próprio, comunica por message passing enxuto.
4. **Knowledge base append-only + match fuzzy**: memória cross-sessão como heurística, não como verdade. O pattern resolve o trade-off entre "esquecer tudo" (caro) e "cachear fix como dogma" (perigoso).

## Links externos

- [Google SRE Book — Postmortem Culture](https://sre.google/sre-book/postmortem-culture/) — blameless postmortem espelha a disciplina da knowledge base
- [Toyota — Five Whys](https://en.wikipedia.org/wiki/Five_whys) — hypothesis chaining é variante moderna
- [Why Programs Fail (Andreas Zeller)](https://www.whyprogramsfail.com/) — referência canônica em debugging científico
- [Brendan Gregg — USE Method](https://www.brendangregg.com/usemethod.html) — método estruturado de investigação de performance complementa o debugger
- [Hyrum's Law](https://www.hyrumslaw.com/) — contexto cultural pro motivo de bugs cross-layer serem comuns em codebases maduros

## Fonte
- [[../../60-sources/get-shit-done/_toc]]

## Links
- [[_moc]]
- [[pipeline-mapping]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
