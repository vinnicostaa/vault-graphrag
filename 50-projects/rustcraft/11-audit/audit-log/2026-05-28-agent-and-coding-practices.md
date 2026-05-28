---
title: Audit log — Agent and coding practices
type: audit-log
tags:
  - rustcraft
  - audit-log
  - docs
  - agents
  - quality
created: '2026-05-28'
updated: '2026-05-28'
---
# Audit log — Agent and coding practices

Resumo:

- Criado `AGENTS.md` no repositorio para orientar futuras sessoes de agentes.
- Criado `CODING_PRACTICES.md` no repositorio com praticas de codificacao, documentacao, Bevy/Rust e gates.
- README passou a apontar para os dois documentos.
- Criada nota [[../../07-quality/coding-practices]] no vault.
- Atualizados `CLAUDE.md`, `07-quality/_moc`, `12-docs/_moc` e `06-execution/_moc`.
- Criada task guiada [[../../06-execution/next-task-chunk-spawn]] para o usuario implementar o spawn por chunk mesh.

Decisao operacional reforcada:

- O usuario codifica as tasks de implementacao por padrao.
- O agente explica, pesquisa, revisa, documenta e so altera codigo quando houver pedido explicito na sessao corrente.

Pendencia:

- Implementar a troca do spawn por bloco para spawn de mesh por chunk.
