---
title: Audit — uso do vault no rustcraft
type: audit
tags:
  - audit
  - rustcraft
  - vault
  - governance
created: '2026-05-27'
updated: '2026-05-27'
---
# Audit — uso do vault no rustcraft

## Escopo

Auditoria sistematica do uso do vault pelo projeto `rustcraft`, motivada pela correcao do usuario de que a sessao anterior nao estudou o vault a fundo nem criou audits no caminho canonico.

## Fontes consultadas

- `[[../../../../CLAUDE]]` — contrato raiz do vault.
- `[[../../../../_index]]` — indice raiz.
- `[[../../../../40-templates/project-scaffold/_moc]]` — scaffold canonico.
- `[[../../../../30-agents/playbooks/onboarding-novo-projeto]]` — onboarding obrigatorio.
- `[[../../../../30-agents/playbooks/research-loop]]` — metodo de pesquisa.
- `[[../../../../30-agents/playbooks/dual-check]]` — validacao por segundo olhar.
- `[[../../../master-sindico/_moc]]` e `[[../../../master-sindico/11-audit/AUDIT]]` — exemplo completo.
- `[[../../../websocket-chat/_moc]]` — exemplo menor.

## Metodo

1. Listagem da raiz do vault.
2. Leitura do contrato raiz e templates.
3. Comparacao de `rustcraft` contra `master-sindico`, `websocket-chat` e `project-scaffold`.
4. Registro dos findings em `[[../AUDIT]]`.
5. Criacao das notas faltantes de governanca minima.

## Findings

### F-001 — `rustcraft` era scaffold parcial

O projeto tinha `CLAUDE`, `STATE`, `AUDIT`, `ROADMAP`, `_moc`, `02-architecture`, `04-requirements`, `05-tasks` e `06-reviews`, mas nao tinha `SESSION_CHARTER`, `STEERING`, `11-audit`, nem MOCs 00-12.

Impacto: o agente nao tinha entrada operacional canonica para sessao, audits e gate 0.

Status: parcialmente corrigido nesta sessao; ainda falta preencher conteudo substantivo nas areas 00-12.

### F-002 — Audit estava no lugar errado para o contrato canonico

`AUDIT.md` na raiz existe em exemplos, mas o scaffold e varias notas apontam para `11-audit/AUDIT.md` como path canonico. `master-sindico` mantem double-write por compatibilidade.

Impacto: automatismos ou agentes que seguem o scaffold poderiam nao encontrar findings do `rustcraft`.

Status: criado `[[../AUDIT]]` canonico. Falta decidir politica para o `AUDIT.md` raiz.

### F-003 — Faltava `SESSION_CHARTER`

O contrato raiz exige registrar ordens da sessao. `rustcraft` nao tinha isso.

Impacto: perda de rastreabilidade dos inputs do usuario e maior risco de agente agir por suposicao.

Status: criado `[[../../SESSION_CHARTER]]`.

### F-004 — Faltava `STEERING`

O scaffold espera nao-negociaveis por projeto. `rustcraft` tinha regras espalhadas no MOC/CLAUDE, mas nao uma nota dedicada.

Status: criado `[[../../STEERING]]`.

### F-005 — A regra tutor-first precisa aparecer em mais de um ponto operacional

Ela existia no MOC e CLAUDE, mas deve ser reforcada em `SESSION_CHARTER`, `STEERING` e tasks para evitar que o agente volte a codar demais sem pedido explicito.

Status: reforcada nas novas notas.

## Melhorias aplicadas

- Criado `SESSION_CHARTER.md`.
- Criado `STEERING.md`.
- Criado `11-audit/AUDIT.md`.
- Criados MOCs para `11-audit`, `audit-log`, `dual-check-log` e `postmortems`.
- Criado este audit log datado.
- Iniciada normalizacao de MOCs 00-12.
- Criados `overview.md` e `DoR-DoD.md`.
- Atualizado `CLAUDE.md` para espelhar o fluxo canonico do vault.
- Normalizados tipos de frontmatter nao canonicos em notas existentes (`tasks`, `requirements`, `review`, `architecture`).

## Proximas melhorias recomendadas

1. Escolher politica de audit: canonical-only em `11-audit/AUDIT.md` ou double-write temporario com raiz `AUDIT.md`.
2. Completar conteudo substantivo das areas 00-12 alem dos MOCs iniciais.
3. `20-stacks/rust/_moc` foi revisado: a stack Rust esta em modo lazy; por D-010, padroes Rust/Bevy ficam locais ao `rustcraft` ate estabilizarem ou servirem outro projeto.
4. A-006 foi resolvido depois neste mesmo dia em [[2026-05-27-multi-crate-restructure]].
5. A-007/DT-006 foram resolvidos depois neste mesmo dia em [[2026-05-27-repository-consistency-correction]].

## Veredito

A sessao anterior estava incompleta como uso do vault. A correcao criou o caminho canonico de audit e melhorou a governanca do `rustcraft`. Mais tarde, em 2026-05-27, a divergencia entre ADR-0003 e codigo foi resolvida em [[2026-05-27-multi-crate-restructure]].

## Links

- [[../AUDIT]]
- [[../../SESSION_CHARTER]]
- [[../../STEERING]]
- [[../../STATE]]
- [[../../../../40-templates/project-scaffold/_moc]]
