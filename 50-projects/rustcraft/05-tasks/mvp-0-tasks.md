---
title: Tasks — MVP-0 rustcraft
type: note
tags:
  - tasks
  - rustcraft
  - mvp0
created: '2026-05-27'
updated: '2026-05-28'
---
# Tasks — MVP-0 rustcraft

> Tasks desenhadas para o usuário codificar com apoio do agente/tutor.

## T-001 — Entender Cargo workspace/package/lib/bin/module

Status: done

Objetivo: antes de mexer no código, explicar em voz alta/escrito a diferença entre:

- workspace;
- package;
- crate;
- library target;
- binary target;
- módulo.

Saída esperada:

- atualizar README ou nota curta com essa distinção.

Validação:

- o usuário consegue apontar onde fica cada coisa no projeto.

Resultado em 2026-05-28:

- O README e a arquitetura documentam workspace, packages internas `rc-*`, library target, binary target e módulos.
- O projeto já foi reorganizado e evoluído usando essa separação, com `crates/rustcraft` como composição do app e crates internas por domínio.

## T-002 — Escolher árvore inicial

Status: done

Objetivo: decidir entre:

### Opção A — simples

```text
crates/rustcraft/src/main.rs
crates/rustcraft/src/lib.rs
```

### Opção B — explícita com bin target

```text
crates/rustcraft/src/lib.rs
crates/rustcraft/src/bin/rustcraft.rs
```

Recomendação atual: Opção B, porque o usuário pediu separação em binários e isso ensina Cargo targets.

Validação:

```sh
cargo run --bin rustcraft
```

## T-003 — Restaurar comportamento original

Status: done

Objetivo: garantir que a reorganização não mude o jogo.

Arquivos prováveis:

- `crates/rustcraft/src/bin/rustcraft.rs`;
- `crates/rustcraft/src/app.rs`;
- `crates/rustcraft/src/lib.rs`;
- `crates/rc-player/src/lib.rs`;
- `crates/rc-world/src/lib.rs`;
- `crates/rc-render/src/lib.rs`.

Critérios:

- terreno aparece;
- câmera aparece;
- movimento funciona.

Resultado em 2026-05-28:

- `cargo run` foi executado com sucesso depois da troca para spawn por chunk.
- Logs confirmaram `entity_count = 20`, `rustcraft/chunks_rendered = 1`, `rustcraft/chunk_faces = 1776` e `rustcraft/chunk_vertices = 7104`.
- O comportamento visual mínimo foi validado em runtime: app abre janela, mundo inicial existe e a câmera/player continua presente.

## T-004 — Renomear `protocol` para linguagem de jogo

Status: done

Objetivo: trocar qualquer módulo/conceito `protocol` usado para input local por `actions`/`controls`.

Estrutura implementada:

```text
crates/rc-input/src/lib.rs
crates/rc-player/src/lib.rs
```

Critérios:

- `PlayerAction` fica em `rc-input`;
- mapeamento `KeyCode` → `PlayerAction` fica em `rc-input`;
- `rc-player` consome `ActionState`, não `KeyCode`.

## T-005 — Separar blocos do render

Status: done

Objetivo: manter `BlockType` como dado lógico, e materiais/meshes em `rendering`.

Critério:

- `crates/rc-voxel/src/lib.rs` não importa `StandardMaterial`, `Mesh`, `Color` ou tipos Bevy de render.

## T-006 — Separar geração de terreno

Status: done

Objetivo: mover `terrain_height` e regras de coluna para `rc-world` ou equivalente.

Critério:

- função testável sem abrir janela;
- teste simples garante altura dentro do range.

Resultado:

- `terrain_height` está em `crates/rc-world/src/lib.rs` com testes unitários.

## T-007 — Documentar a estrutura final

Status: done

Objetivo: atualizar README com a árvore realmente usada.

Critérios:

- comandos corretos;
- explicação curta do porquê de `src/bin` se for usado;
- lista de próximos passos.

## T-008 — Validar

Status: done

Rodar:

```sh
cargo check --workspace
cargo test --workspace
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

Critério:

- todos passam antes de fechar MVP-0.

Resultado em 2026-05-28:

- `cargo check --workspace`: passou;
- `cargo test --workspace`: passou, 33 testes (`rc-render`: 7, `rc-voxel`: 19, `rc-world`: 7);
- `cargo fmt --all -- --check`: passou;
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`: passou.

## T-009 — Reconciliar ADR-0003 com o código atual

Status: done

Objetivo: decidir e executar a correção entre duas verdades hoje divergentes:

- o vault aceita multi-crate didático com `rc-*`;
- o repositório ainda implementa uma package principal com módulos internos.

Opções:

### Opção A — implementar ADR-0003

Criar packages internas controladas:

```text
crates/rc-input
crates/rc-player
crates/rc-voxel
crates/rc-render
crates/rc-world
```

Começar por `rc-voxel`, porque é mais puro e testável.

### Opção B — rebaixar ADR-0003

Alterar ADR-0003 para proposta/futuro e manter módulos internos até chunk/meshing ficar mais claro.

Critérios:

- README e ARCHITECTURE não contradizem STATE/ADR;
- `cargo check --workspace` continua passando;
- o usuário consegue explicar por que cada crate existe ou por que ainda não existe.

Resultado em 2026-05-27:

- Opção A executada: ADR-0003 implementada com `crates/rustcraft`, `crates/rc-input`, `crates/rc-player`, `crates/rc-voxel`, `crates/rc-render` e `crates/rc-world`.
- README e ARCHITECTURE atualizados para refletir a árvore real.
- Validações: `cargo check --workspace`, `cargo test --workspace`, `cargo fmt --all -- --check` e `cargo clippy --workspace --all-targets --all-features -- -D warnings` passaram.

## T-010 — Normalizar uso do vault

Status: in_progress

Objetivo: alinhar `rustcraft` ao contrato raiz do vault e ao scaffold canônico.

Entregas já feitas em 2026-05-27:

- criado `SESSION_CHARTER.md`;
- criado `STEERING.md`;
- criado `DoR-DoD.md`;
- criado `11-audit/AUDIT.md`;
- criado audit log `11-audit/audit-log/2026-05-27-vault-usage-audit.md`;
- criados MOCs iniciais 00-12;
- atualizado `_moc.md`, `CLAUDE.md`, `STATE.md` e `50-projects/_moc.md`.

Pendências:

- decidir DT-007: política final para `AUDIT.md` raiz;
- preencher conteúdo substantivo das áreas 00-12.

Correção em 2026-05-27:

- DT-006/A-007 fechados: `context-1.md` e `context-2.md` nao existem no workspace atual; `.gitignore` ignora `context-*.md`.

Validação:

- `list_directory 50-projects/rustcraft` deve mostrar scaffold mais próximo do canônico;
- `11-audit/AUDIT.md` deve ser o audit principal;
- `_moc.md` deve apontar para SESSION_CHARTER, STEERING, DoR-DoD e 11-audit.
