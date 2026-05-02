---
title: Zanzibar & OpenFGA / SpiceDB — ReBAC engines
type: concept
tags:
  - security
  - zanzibar
  - openfga
  - spicedb
  - rebac
  - graph
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Zanzibar
  - OpenFGA
  - SpiceDB
  - FGA
---

# Zanzibar & OpenFGA / SpiceDB

**O que é**: engines de autorização **ReBAC** (relationship-based) baseadas no paper *Zanzibar: Google's Consistent, Global Authorization System* (2019). **OpenFGA** (Auth0/Okta) e **SpiceDB** (Authzed) são as implementações open-source dominantes.

> Nota-mãe do modelo: [[rebac]]. Aqui foca nos **engines concretos** — como implantar, quando escolher qual, padrões operacionais.

## Zanzibar — o paper (30s)

Google construiu para resolver authz de Google Docs/Drive/YouTube/Gmail:
- **Escala**: trilhões de checks/dia, baixa latência global.
- **Consistency**: via Zookies (versões causais).
- **Grafo**: tuples `object#relation@user` representam arestas; check = BFS com rewrites.
- **Schema**: relations + rewrites (union, intersection, computed, tuple-to-userset).

Implementações open-source derivam disso com adaptações.

## OpenFGA — detalhes

**Origem**: Auth0 FGA (2021) open-sourced em 2022; **CNCF Sandbox desde set/2022**, promovido a **Incubating em 28 out 2025**.

**Stack**:
- Backend: Go.
- Storage: Postgres, MySQL, MongoDB (experimental).
- API: gRPC + HTTP REST.
- SDK oficiais: Node, Go, Python, Java, .NET, PHP, Ruby.

**Schema DSL** (`.fga`):
```fga
model
  schema 1.1

type user

type group
  relations
    define member: [user, group#member]

type folder
  relations
    define owner: [user]
    define viewer: [user, group#member]
    define editor: [user, group#member] or owner

type document
  relations
    define parent: [folder]
    define owner: [user]
    define viewer: [user, group#member] or viewer from parent
    define editor: [user, group#member] or editor from parent or owner
```

**API core**:
- `Write`/`Delete` tuples.
- `Check` (boolean).
- `ListObjects` (quais docs user pode editar?).
- `ListUsers` (quem acessa este doc?).
- `Expand` (árvore de permissões).
- `BatchCheck`, `StreamedListObjects` (performance).

**Modelo de deployment**: standalone server (Docker/K8s) + DB. Cloud-managed via Auth0 FGA, Aserto, AuthZed.

## SpiceDB — detalhes

**Origem**: Authzed (Y Combinator 2021); closely implementa o paper original.

**Stack**:
- Backend: Go.
- Storage: Postgres, MySQL, CockroachDB, Spanner, in-memory.
- API: gRPC + HTTP + a "Authzed" hosted service.
- SDK: Go, Node, Python, Java, .NET, Ruby.

**Schema (Zed)**:
```zed
definition user {}

definition group {
  relation member: user | group#member
}

definition folder {
  relation owner: user
  relation viewer: user | group#member
  permission view = viewer + owner
}

definition document {
  relation parent: folder
  relation owner: user
  relation viewer: user | group#member
  permission view = viewer + owner + parent->view
}
```

Zed é mais **parecido com o paper** que OpenFGA DSL. Compilador forte; erros de schema detectados antes de deploy.

**Diferenciadores**:
- **ZedToken** (zookie nativo) — consistency causal testada extensivamente.
- **Caveats** — condicional em tuples (ABAC-flavored dentro de ReBAC).
- **Watch API** — stream de eventos de tuple changes.

## OpenFGA vs SpiceDB — escolha

| Dimensão | OpenFGA | SpiceDB |
|---|---|---|
| **Origem** | Auth0/Okta (empresa grande) | Authzed (startup dedicada) |
| **Foco** | Developer experience | Fidelidade ao paper |
| **DSL** | `.fga` próprio | Zed (mais próximo do paper) |
| **Caveats** | Parcial (1.5+) | First-class |
| **Hosted** | Okta FGA, Aserto | Authzed SaaS |
| **Storage** | Postgres, MySQL, MongoDB | Postgres, MySQL, Cockroach, Spanner |
| **CNCF** | **Incubating (out/2025)** — Sandbox desde 2022 | Não |
| **Momentum** | Maior (Okta brand) | Maduro; comunidade sólida |

**Escolha**: para novo projeto, **OpenFGA** é aposta segura (brand + momentum). **SpiceDB** se precisa de caveats fortes, consistency estrita, ou fidelidade Zanzibar.

## Padrão operacional

### Modelando authz

1. **Identificar entidades** (user, team, organization, document, folder, workspace).
2. **Identificar relações** (member, owner, viewer, editor, parent, admin).
3. **Desenhar grafo** no papel — permissões herdam como?
4. **Schema first** — começar pelo `.fga` / `.zed`; testar com tuples mock.
5. **Iterar** — schema é vivo, mas mudanças requerem migration.

### Armazenamento de tuples

Cada compartilhamento/grant vira **tuple**. Scales bem para milhões; organizar por `namespace`/`store` para isolamento multi-tenant.

### Consistência

```
1. App escreve tuple: share doc X com user Y → OpenFGA retorna consistency token
2. App armazena token na sessão do user Y
3. User Y bate API para ler doc X → App passa token para check
4. OpenFGA/SpiceDB garante replica consistent ≥ token antes de responder
```

### Caching

- **Check result cache** com invalidação ativa (watch API) ou TTL curto (segundos).
- Preferir **BatchCheck** quando lista de permissões por item → reduz RTT.

## Integração com IdP / OIDC

```
1. Client → IdP: login → JWT com sub=alice
2. Client → App: request com Bearer JWT
3. App valida JWT (sub=alice) → chama OpenFGA: check(user:alice, viewer, document:1234)
4. OpenFGA responde allowed true/false
```

OpenFGA **não** valida JWT; app faz authn, passa sub como principal. Ver [[iam]] e [[identity-provider]].

## Casos reais

- **Canva**: SpiceDB para permissions de designs/folders (escala ~100M usuários).
- **Carta**: SpiceDB.
- **Intercom**: OpenFGA.
- **Atmosfera (startup brasileira)**: OpenFGA.
- **Dynatrace**: Cedar (alternativa similar).

## Anti-patterns

1. **Implementar Zanzibar do zero** — complexo (Zookies, consistency distribuída). Use engine.
2. **Schema mal modelado** — refactor custa. Invista tempo na modelagem.
3. **Escrever 1 tuple por operação de negócio** — tuples são **estado**, não log. Eventos vão em audit trail separado.
4. **Hit direto no OpenFGA de cliente browser** — expõe service; sempre atrás do app.
5. **Sem batching** — N requests sequenciais em list view. Use BatchCheck.
6. **Ignorar ZedToken/consistency token** em fluxos crítico de share.
7. **Cache sem watch/invalidation** — authz stale ([[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]]).
8. **Misturar tuple e log** — tuple vira spaghetti.

## Relações

- **Modelo**: [[rebac]] (teoria)
- **Alternativas**: [[opa-rego]] (PBAC geral), Cedar (AWS), Casbin
- **Integração**: [[iam]], [[identity-provider]], [[oidc]] (app valida JWT, chama engine)
- **Modelo comparação**: [[authorization-models]]
- **Runtime antipattern**: [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]]

## Fontes

- [Zanzibar paper — USENIX 2019](https://research.google/pubs/pub48190/) (consultada 2026-04-24)
- [OpenFGA docs](https://openfga.dev/docs) (consultada 2026-04-24)
- [OpenFGA playground](https://play.fga.dev/) (consultada 2026-04-24)
- [SpiceDB docs / Authzed](https://authzed.com/docs) (consultada 2026-04-24)
- [Cedar Language — AWS](https://www.cedarpolicy.com/) (consultada 2026-04-24)
- [CNCF OpenFGA](https://www.cncf.io/projects/openfga/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[rebac]]
- [[authorization-models]]
- [[opa-rego]]
- [[iam]]
- [[identity-provider]]
