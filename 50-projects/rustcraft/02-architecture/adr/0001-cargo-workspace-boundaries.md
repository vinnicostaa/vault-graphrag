---
title: ADR-0001 — Cargo workspace com fronteiras graduais
type: adr
tags:
  - adr
  - rustcraft
  - cargo
  - workspace
status: accepted
date: '2026-05-27'
created: '2026-05-27'
updated: '2026-05-27'
---
# ADR-0001 — Cargo workspace com fronteiras graduais

## Status

accepted — 2026-05-27

## Contexto

O projeto começou como um único crate Bevy com todo o código em `main.rs`. A migração para workspace é útil para crescimento futuro, mas há risco de overengineering se cada responsabilidade virar uma crate cedo demais.

Também há um objetivo pedagógico: o usuário está aprendendo Rust, Cargo, Bevy e arquitetura de jogos. A estrutura precisa ensinar conceitos corretos sem aumentar complexidade artificial.

## Decisão

Usar Cargo workspace com uma package jogável principal inicialmente:

```text
crates/rustcraft
```

Dentro dela:

- `src/lib.rs` contém sistemas/módulos compartilhados;
- `src/bin/rustcraft.rs` pode conter o executable target principal;
- módulos internos separam `input`, `player`, `voxel`, `world`, `rendering`, `ui`;
- novas crates só serão criadas quando houver fronteira real e estável.

## Consequências positivas

- Mantém o projeto didático.
- Evita ciclos prematuros entre crates.
- Preserva flexibilidade para múltiplos binários experimentais.
- Permite extrair `rustcraft_voxel` depois, quando chunk/meshing estiver claro.

## Consequências negativas

- Algumas fronteiras ficam disciplinares, não impostas pelo compilador.
- Pode ser necessário refactor posterior para extrair crates.
- O usuário precisa entender módulos e visibilidade Rust para manter separação limpa.

## Alternativas consideradas

### Tudo em uma crate sem workspace

Mais simples, mas perde a chance de aprender workspace e preparar tools/crates futuras.

### Uma crate por domínio desde já

Parece organizado, mas é prematuro para o estado atual e aumenta boilerplate.

### Engine própria sem Bevy

Interessante para aprendizado de baixo nível, mas não é o objetivo imediato.
