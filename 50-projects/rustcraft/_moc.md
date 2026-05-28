---
title: MOC — rustcraft
type: moc
tags:
  - moc
  - rustcraft
  - rust
  - bevy
  - voxel
created: '2026-05-27'
updated: '2026-05-27'
---
# MOC — rustcraft

`rustcraft` e um projeto de estudo em Rust/Bevy para construir, passo a passo, um jogo sandbox/voxel 3D inspirado por Minecraft, mas orientado a aprendizado de arquitetura, ECS, mundo voxel, renderizacao, gameplay, fisica e ferramentas.

> Entrada obrigatoria: [[CLAUDE]] -> [[STATE]] -> [[11-audit/AUDIT]] -> [[SESSION_CHARTER]] -> [[05-tasks/mvp-0-tasks]].

## Governanca viva

- [[CLAUDE]] — contrato do agente/tutor para este projeto.
- [[STATE]] — decisoes vivas, dividas e pendencias.
- [[11-audit/AUDIT]] — audit canonico do projeto.
- [[AUDIT]] — audit legado/compatibilidade ate DT-007 ser resolvida.
- [[SESSION_CHARTER]] — ordens da sessao corrente.
- [[STEERING]] — nao-negociaveis do projeto.
- [[DoR-DoD]] — Definition of Ready / Done.
- [[ROADMAP]] — fases macro.
- [[overview]] — visao resumida.

## Regra de operacao

Este projeto e tutor-first:

- o usuario codifica de fato;
- o agente ajuda a entender, planejar, revisar, pesquisar e decompor tasks;
- o agente so altera codigo quando houver pedido explicito na sessao corrente;
- decisoes arquiteturais devem ser registradas no vault antes de virar refactor grande.

## Scaffold do projeto

- [[00-product/_moc]] — visao, personas, glossario.
- [[01-domain/_moc]] — linguagem ubiqua, conceitos de jogo, invariantes.
- [[02-architecture/_moc]] — arquitetura e ADRs.
- [[03-subprojects/_moc]] — packages/crates/subprojetos.
- [[04-requirements/_moc]] — requisitos.
- [[05-tasks/_moc]] — tasks.
- [[06-execution/_moc]] — execucao e fluxo por task.
- [[07-quality/_moc]] — qualidade tecnica.
- [[08-security/_moc]] — seguranca futura.
- [[09-testing/_moc]] — testes.
- [[10-schedule/_moc]] — marcos e agenda.
- [[11-audit/_moc]] — audits e logs.
- [[12-docs/_moc]] — documentacao de uso.

## Arquitetura

- [[02-architecture/workspace-and-crate-organization]]
- [[02-architecture/adr/0001-cargo-workspace-boundaries]]
- [[02-architecture/adr/0002-game-actions-not-protocol]]
- [[02-architecture/adr/0003-multi-crate-workspace-for-learning]]

## Requisitos e tasks

- [[04-requirements/mvp-0-learning-prototype]]
- [[05-tasks/mvp-0-tasks]]

## Reviews e audits datados

- [[06-reviews/context-audit-2026-05-27]] — revisao dos contextos Zed e estado do repositorio.
- [[11-audit/audit-log/2026-05-27-vault-usage-audit]] — auditoria do uso do vault e normalizacao.
- [[11-audit/audit-log/2026-05-27-multi-crate-restructure]] — implementacao da estrutura multi-crate no repositorio.
- [[11-audit/audit-log/2026-05-27-repository-consistency-correction]] — correcao de pendencias fantasma e registro do historico `minecraft`.

## Repositorio

Workspace local:

```text
/home/vinni/Documentos/Projetos/Privados/rustcraft
```

## Vizinhos

- [[../_moc]] — projetos ativos.
- [[../../40-templates/project-scaffold/_moc]] — scaffold canonico.
- [[../../30-agents/playbooks/onboarding-novo-projeto]] — onboarding de projeto.
- [[../../20-stacks/rust/_moc]] — stack Rust.
- [[../../10-knowledge/architecture/the-clean-architecture]] — dependency rule.
- [[../../10-knowledge/runtime-antipatterns/op-011-logica-negocio-infra]] — evitar regra de dominio em infra.
