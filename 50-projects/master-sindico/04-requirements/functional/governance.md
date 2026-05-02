---
title: Governance — requisitos funcionais (stub)
type: requirement
tags:
  - requirement
  - master-sindico
  - governance
  - stub
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# Governance — requisitos funcionais

> **Status**: 🟡 stub criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Requisitos detalhados serão migrados de [[institutional]] (que cobre os mesmos conceitos a partir do BC `institutional`) ou destilados separadamente quando o sub-domínio "governance" for promovido a requirement-doc próprio.

Sub-domínio "governance" do BC `institutional` — cobre **Plano Diretor** (PD), **Linha do Tempo** (Timeline), **Eventos**, **Comunicados**, **Avaliação de Gestão**, **Perguntas aos Moradores** e **Dashboard** do síndico.

**Contexto canônico**: a maioria desses REQs já está coberta semanticamente em [[institutional]] (Timeline imutável, MasterPlan, Announcement, Event, governance score). Este arquivo serve como **âncora de jornadas** até a destilação completa.

---

## Anchors stub (REQ-GOV-*)

### Linha do Tempo (Timeline)

#### REQ-GOV-TIMELINE
Memória oficial append-only do condomínio. Tudo relevante gera registro: atividade da gestão, registro de execução validado, comunicado, evento, adendo, resultado de pergunta. Ver [[institutional]] §Timeline e [[../../01-domain/aggregates/TimelineEntry]].

#### REQ-GOV-TIMELINE-CREATE
Criação de entrada na Timeline (autorizada). Imutável pós-publish. Ver [[institutional]] e [[../../02-architecture/adr/0033-ata-timeline-db-immutability|ADR-0033]].

#### REQ-GOV-TIMELINE-HISTORY
Visualização histórica da Timeline com filtro por tipo, autor, data. Ver [[institutional]].

### Plano Diretor

#### REQ-GOV-PD
Plano Diretor plurianual do condomínio (M1 a M3+ anos). Status atualizado via Timeline (não manual). Ver [[institutional]] §MasterPlan e [[../../01-domain/aggregates/MasterPlan]].

#### REQ-GOV-PD-CREATE
Criação/edição de itens do Plano Diretor. Ver [[institutional]].

#### REQ-GOV-PD-LIST
Listagem do Plano Diretor por categoria/prioridade/status. Ver [[institutional]].

#### REQ-GOV-PD-MORADOR
Visualização do Plano Diretor pelo morador (read-only, com filtros públicos). Ver [[institutional]].

### Eventos & Comunicados

#### REQ-GOV-EVENT
Evento condominial (reunião, obra, festa, manutenção). Ver [[institutional]] §Event e [[../../01-domain/aggregates/Event]].

#### REQ-GOV-EVENT-MORADOR
Visualização do evento pelo morador, com confirmação de presença quando aplicável. Ver [[institutional]].

#### REQ-GOV-COMUNICADO
Comunicado oficial do síndico (imutável pós-publish). Ver [[institutional]] §Announcement e [[../../01-domain/aggregates/Announcement]].

#### REQ-GOV-COMUNICADO-MORADOR
Visualização do comunicado pelo morador. Ver [[institutional]].

### Home & Dashboard

#### REQ-GOV-HOME
Tela inicial de Governança do síndico — agrega Timeline, PD, Eventos, KPIs. Ver [[institutional]].

#### REQ-GOV-DASHBOARD
Dashboard de KPIs do síndico (governance score, compliance status, indicadores). Ver [[institutional]] §Governance Score.

#### REQ-GOV-MORADOR-HOME
Tela inicial do morador na vertente governança — feed de Timeline + PD + Comunicados. Ver [[institutional]].

### Engagement

#### REQ-GOV-AVALIACAO-GESTAO
Avaliação anônima da gestão pelo morador (3 perguntas escala 1-10, ciclo bimestral). Ver [[institutional]] §Governance Score componente AvaliacaoGestao.

#### REQ-GOV-PERGUNTA-MORADOR
Síndico pergunta aos moradores; resposta não-vinculante; resultado vai pra Timeline. Ver [[institutional]].

---

## Links

- [[_moc]]
- [[institutional]] — fonte canônica dos conceitos
- [[cross-domain]] — integrações cross-BC
- [[../../01-domain/aggregates/_moc]]
- [[../../STATE]]
