---
title: AUDIT — rustcraft canonico
type: audit
tags:
  - audit
  - rustcraft
  - canonical
created: '2026-05-27'
updated: '2026-05-28'
---
# AUDIT — rustcraft

Arquivo canonico de findings do projeto. Este path segue o scaffold do vault: `50-projects/<slug>/11-audit/AUDIT.md`.

## Regras

- A-### = finding descoberto em revisao.
- High/Critical aberto bloqueia task nova ate haver plano ou fix.
- STATE registra decisoes e dividas; AUDIT registra problemas encontrados.
- Audit detalhado vai em `[[audit-log/_moc]]`.

## Itens abertos

| ID | Severidade | Status | Finding | Acao recomendada |
| --- | --- | --- | --- | --- |
| A-009 | Medium | open | Ainda nao ha politica documentada para manter ou remover o `AUDIT.md` legado na raiz do projeto. | Escolher single-source em `11-audit/AUDIT.md` ou double-write temporario como no `master-sindico`. |
| A-014 | Low | open | `Block` e `GeneratedChunkBlock` permanecem como componentes legados sem uso ativo no caminho principal de spawn por chunk. | Remover quando raycast/interacao/chunk editing tiverem modelo proprio, ou documentar como compatibilidade temporaria com prazo. |

## Resolvidos nesta rodada

| ID | Severidade | Status | Finding | Verificacao |
| --- | --- | --- | --- | --- |
| A-001 | Medium | closed | Separacao inicial confundiu modulos internos com organizacao correta de workspace/crates. | Arvore Cargo agora usa workspace com packages `crates/rustcraft` e `crates/rc-*`; README/ARCHITECTURE documentam workspace, library target, bin target e crates internas. |
| A-002 | Medium | closed | Nome `protocol` era inadequado para acoes de jogador no contexto de jogo local. | Codigo local usa `rc-input` com `PlayerAction`/`ActionState`; `rg protocol` nos arquivos vivos nao retorna ocorrencias relevantes. |
| A-003 | Low | closed | Arquivos placeholder de UI poderiam confundir o aprendizado se nao tivessem funcao clara. | Placeholders legados foram removidos da base atual durante a reorganizacao multi-crate. |
| A-004 | High | closed | O projeto foi criado como scaffold parcial, sem instanciar completamente `40-templates/project-scaffold` com pastas 00-12. | `list_directory 50-projects/rustcraft` mostra areas 00-12, `SESSION_CHARTER`, `STEERING`, `DoR-DoD`, `overview`, MOCs e `11-audit/AUDIT`. |
| A-005 | Low | closed | `cargo clippy --workspace --all-targets --all-features -- -D warnings` falhava por `clippy::derivable_impls` em `GameConfig`. | O antigo `GameConfig` foi removido; validacao completa do workspace passou. |
| A-006 | High | closed | O vault aceitava ADR-0003 multi-crate `rc-*`, mas o repositorio ainda concentrava a implementacao em uma package. | ADR-0003 implementada no codigo com `rc-input`, `rc-player`, `rc-voxel`, `rc-render`, `rc-world` e `rustcraft` como composicao; `cargo check --workspace` passou. |
| A-007 | Low | closed | `context-1.md` e `context-2.md` estavam registrados como pendencia aberta, mas nao existem no workspace atual. | `find`/`git status` nao mostram os arquivos; `.gitignore` agora bloqueia `context-*.md` como artefato de sessao. |
| A-008 | High | closed | A sessao anterior nao seguiu o contrato do vault com audit canonico, `SESSION_CHARTER` e leitura profunda do scaffold. | Criados `SESSION_CHARTER.md`, `STEERING.md`, `11-audit/AUDIT.md` e audit logs datados. |
| A-010 | Medium | closed | A area Rust em `20-stacks/rust` ainda nao tinha sido estudada para apoiar `rustcraft`. | Resolvido por D-010: `20-stacks/rust` permanece lazy; padroes Rust/Bevy ficam locais ate estabilizarem ou servirem outro projeto. |
| A-011 | Low | closed | Algumas notas do `rustcraft` usavam `type` fora da lista canonica (`tasks`, `requirements`, `review`, `architecture`). | Frontmatter normalizado para `note`, `requirement`, `audit` e `guide`. |
| A-012 | Low | closed | O projeto teve package/binario historico chamado `minecraft`, mas o vault nao registrava essa origem. | Confirmado em `git show e7df14e:Cargo.toml`; registrado como D-011 em `STATE`. |
| A-013 | Low | closed | `crates/rustcraft/src/components/` existia vazio como residuo local nao versionado. | Diretorio vazio removido; `git ls-files` nao rastreava nenhum arquivo nele. |
| A-015 | Low | closed | `crates/rustcraft/Cargo.toml` declarava `rc-voxel` como dependencia direta sem uso atual na composicao do app. | Fechado pelo commit `28cff15`: `crates/rustcraft/src/interaction.rs` usa `rc_voxel::block_pos_from_hit`, justificando a dependencia direta. |

## Critérios de fechamento

### A-001

- O usuario consegue explicar workspace, package/crate, lib target, bin target e modulo.
- A arvore do projeto reflete essa diferenca.
- README e arquitetura documentam a decisao final.

### A-004

- Pastas 00-12 existem com MOCs.
- `SESSION_CHARTER`, `STEERING`, `DoR-DoD`, `overview` e `11-audit/AUDIT` existem.
- `_moc.md` do projeto aponta para as fontes de verdade corretas.

### A-006

- ADR-0003 e codigo deixam de divergir.
- `cargo check --workspace` passa apos a decisao.

## Links

- [[_moc]]
- [[audit-log/2026-05-27-vault-usage-audit]]
- [[audit-log/2026-05-27-multi-crate-restructure]]
- [[../STATE]]
- [[../SESSION_CHARTER]]
- [[../../../40-templates/audit-template]]
