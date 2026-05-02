---
title: MOC — 05-tasks
type: moc
tags: [moc, master-sindico, tasks]
project: master-sindico
created: 2026-04-22
---

# 05-tasks — Tasks (global + by-sprint + by-module + backlog)

Tasks **canônicas** organizadas por corte:

## Notas canônicas

### Global
- [[global]] — tasks **transversais** ou de infraestrutura que atravessam sub-projetos / módulos (CI setup, release, dep-audit recorrente, onboarding de dev, ADR backlog)

### Por sprint
- [[by-sprint/_moc]] — índice de sprints (0 a N)
- [[by-sprint/sprint-0]] a [[by-sprint/sprint-9]] — já entregues; migrar de `specs/tasks/`
- [[by-sprint/sprint-10]] — em curso (M1 finalização)
- [[by-sprint/sprint-backlog]] — propostas ainda não alocadas a sprint

### Por módulo
- [[by-module/_moc]] — índice por bounded context
- [[by-module/identity]] · [[by-module/billing]] · [[by-module/institutional]] · [[by-module/commercial]] · [[by-module/content]] · [[by-module/assembly]] · [[by-module/cross-domain]]

### Backlog priorizado
- [[backlog]] — ideias e débitos não-prioritários, ordenados por valor/custo/risco

## Formato de task

```markdown
## T-<ID>: <título imperativo>

- **Módulo**: identity | billing | ...
- **Sub-projetos**: backend | web | mobile (múltiplos se cross)
- **Sprint alvo**: <N> ou "backlog"
- **Requirement**: [[functional/<módulo>#Req N]]
- **Status**: [ ] pending · [⏳] in_progress · [x] done · [~] blocked · [d] deferred
- **Blocked by**: T-<ID> ou D-<ID>
- **DoR**: <quando pode entrar em execute, ver [[../DoR-DoD]]>
- **DoD**: <critérios de aceite testáveis>
- **Patterns aplicáveis**: [[../07-quality/PATTERNS#...]]
- **Plano**: <link pro plan block se foi escrito>
```

## Princípio: task pequena

- **Tamanho alvo**: 1-3 dias de dev
- **Sem descoberta**: se DoR tem "a decidir" → criar task de spike/research separada
- **Uma camada por vez** quando possível (segue ordem canônica do SDD)
- **Dependências explícitas** via Blocked-by
- **Cross-sub-projeto** é uma única task só se vertical slice é simples; caso contrário, split

## Fontes

- [[../specs/tasks/README]]
- [[../specs/tasks/sprint-0]] [[../specs/tasks/sprint-1]]
- [[../specs/tasks]] — monolítico (68KB, fonte rica)
- [[../client-vault/04 - Execution/Sprint 1 Identity]]
- [[../content/tasks]] — matriz extensa

## Vizinhos

- [[../_moc]]
- [[../04-requirements/_moc]] — cada task mapeia pra ≥ 1 requirement
- [[../02-architecture/_moc]] — tasks arquiteturais viram ADR
- [[../10-schedule/_moc]] — cronograma consolidado dos sprints
- [[../11-audit/_moc]] — findings viram tasks de fix
