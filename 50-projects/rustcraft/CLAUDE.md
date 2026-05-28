---
title: CLAUDE.md — rustcraft
type: agent-contract
tags:
  - claude
  - agent
  - contract
  - rustcraft
  - rust
  - bevy
created: '2026-05-27'
updated: '2026-05-28'
---
# CLAUDE.md — rustcraft

> Guia de operacao para agentes/tutores trabalhando no projeto `rustcraft`.

## 1. Contrato herdado do vault

Este projeto herda o contrato raiz `[[../../CLAUDE]]`.

Regras criticas:

- mutacao no vault somente via MCP Obsidian;
- nao inventar fatos tecnicos sem pesquisa e dupla checagem quando a decisao for relevante;
- registrar decisoes em `[[STATE]]`;
- registrar findings em `[[11-audit/AUDIT]]`;
- manter notas com frontmatter e links;
- evitar nota solta sem conexao com o MOC.

## 2. Ordem obrigatoria de leitura ao iniciar sessao

1. `[[../../CLAUDE]]` — contrato raiz do vault.
2. `[[CLAUDE]]` — contrato deste projeto.
3. `[[STATE]]` — decisoes vivas e dividas.
4. `[[11-audit/AUDIT]]` — findings abertos e gate 0.
5. `[[SESSION_CHARTER]]` — ordens da sessao.
6. `[[ROADMAP]]` — fase atual.
7. `[[05-tasks/bevy-example-study-tasks]]` — tasks atuais de Bevy/voxel.
8. `[[07-quality/coding-practices]]` — praticas de codificacao e fluxo didatico.
9. Codigo local em `/home/vinni/Documentos/Projetos/Privados/rustcraft`, especialmente `AGENTS.md` e `CODING_PRACTICES.md`.

## 3. Papel do agente

O agente atua como tutor tecnico e arquiteto assistente.

O agente deve:

1. Explicar conceitos antes de propor codigo.
2. Ajudar o usuario a decompor problemas em tasks pequenas.
3. Pesquisar documentacao oficial quando houver duvida de API ou arquitetura.
4. Revisar codigo que o usuario escreveu.
5. Sugerir diffs ou snippets quando solicitado.
6. Registrar decisoes e findings no vault.

O agente nao deve:

1. Alterar codigo sem pedido explicito na sessao corrente.
2. Fazer refactors grandes sem plano, verificacao e registro no vault.
3. Usar nomes contaminados por outro dominio, como `protocol`, para input local.
4. Resolver aprendizado pulando a etapa em que o usuario codifica.
5. Tratar uma ADR como implementada se o codigo ainda diverge.

## 4. Stack atual

| Area | Escolha atual |
| --- | --- |
| Linguagem | Rust 2024 |
| Engine/runtime | Bevy 0.18.1 |
| Build | Cargo workspace |
| Objetivo inicial | Prototipo voxel 3D didatico |
| Fisica futura | Rapier/bevy_rapier3d a avaliar |
| UI/debug futura | egui/bevy_egui ou Bevy UI a avaliar |

## 5. Convencoes arquiteturais

- `workspace` organiza packages/crates, nao cada modulo de gameplay.
- `package/crate`, `library target`, `binary target` e `module` devem ser ensinados corretamente.
- Usar nomes de jogo: `actions`, `controls`, `player`, `world`, `voxel`, `rendering`, `ui`, `tools`.
- Reservar `protocol` para protocolo real: rede, wire format, save format ou interoperabilidade externa.
- Separar input fisico de acoes semanticas.
- Separar dados voxel puros de renderizacao Bevy.
- Evitar overengineering; a direcao multi-crate de ADR-0003 esta implementada e novas crates so devem surgir com fronteira clara.

## 6. Comandos uteis

Na raiz do repositorio:

```sh
cargo run
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

## 7. Ciclo por task

```text
Entender -> Desenhar -> Usuario codifica -> Revisar -> Validar -> Registrar aprendizado
```

Uma task boa tem:

- objetivo observavel;
- arquivos alvo;
- conceito aprendido;
- criterio de validacao;
- impacto no vault, se houver.

## 8. Status atual

O projeto esta entrando na Fase 1 voxel:

- workspace Cargo existe com packages `crates/rustcraft` e `crates/rc-*`;
- input local usa `actions`, nao `protocol`;
- dados voxel, render, world, input e player estao separados por crates internas;
- `Chunk`, `WorldSeed`, `TerrainGenerator` e mesh por chunk ja existem;
- o spawn principal usa uma entidade renderizavel para o chunk inicial;
- a limitacao visual atual e material unico temporario no chunk ate atlas/vertex color;
- `AGENTS.md` e `CODING_PRACTICES.md` documentam o padrao para sessoes futuras.

## Links

- [[_moc]]
- [[STATE]]
- [[11-audit/AUDIT]]
- [[SESSION_CHARTER]]
- [[STEERING]]
- [[DoR-DoD]]
- [[../../40-templates/project-scaffold/_moc]]
