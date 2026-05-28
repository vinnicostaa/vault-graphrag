---
title: Audit log — Correção de consistência repo/vault rustcraft
type: audit
tags:
  - audit
  - rustcraft
  - consistency
  - repository
  - vault
created: '2026-05-27'
updated: '2026-05-27'
---
# Audit log — Correção de consistência repo/vault rustcraft

## Escopo

Revisar divergências apontadas depois do commit `28c419a` e separar estado versionado, cache de build e pendências reais do vault.

## Evidências verificadas

Comandos/fatos verificados no repositório local:

- `git status --short` nao mostra `context-1.md` nem `context-2.md`.
- `git ls-files` nao rastreia `crates/rustcraft/src/components`.
- `find crates/rustcraft/src/components` mostrava apenas o diretorio vazio.
- `git show e7df14e:Cargo.toml` mostra `name = "minecraft"`.
- `target/debug/minecraft.d` aponta para o antigo `src/main.rs`, mas `target/` e cache ignorado por `.gitignore`.

## Correções aplicadas

- `.gitignore` passou a ignorar `context-*.md` como artefato de sessao/conversa.
- Diretorio vazio `crates/rustcraft/src/components/` foi removido.
- A-007 foi fechado: os arquivos `context-1.md` e `context-2.md` nao existem no workspace atual.
- D-011 foi registrado em `STATE`: o nome historico `minecraft` existiu no commit `e7df14e`, mas `rustcraft` e o nome canonico atual.
- A-012 registra o gap historico de nome e fica fechado pela documentacao em `STATE`.
- A-013 registra o residuo local `components/` vazio e fica fechado pela remocao do diretorio.

## Decisões de severidade

Nao foi criado finding para cache `target/` nem para `flycheck0` stale. Esses arquivos contam historico local de build/editor, mas nao representam estado fonte do projeto. Fonte de verdade para arquitetura atual: `git ls-files`, manifests Cargo, README/ARCHITECTURE e vault canonico.

## Pendências restantes

- A-009 / DT-007: decidir politica definitiva para `AUDIT.md` raiz vs `11-audit/AUDIT.md`.
- DT-004 / T-003: validacao visual final do prototipo em janela Bevy continua como task de fluxo, nao como bloqueio de estrutura.

## Links

- [[../AUDIT]]
- [[../../STATE]]
- [[2026-05-27-multi-crate-restructure]]
