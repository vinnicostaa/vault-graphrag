---
title: ReBAC — Relationship-Based Access Control (Zanzibar)
type: concept
tags:
  - security
  - rebac
  - authorization
  - zanzibar
  - openfga
  - spicedb
  - graph
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Relationship-Based Access Control
  - ReBAC
  - Zanzibar
---

# ReBAC — Relationship-Based Access Control

**O que é**: modelo de autorização onde permissões derivam de **relações** entre entidades num grafo. Formalizado pelo paper *Zanzibar: Google's Consistent, Global Authorization System* (USENIX ATC 2019). Inspiração para engines modernos: OpenFGA, SpiceDB, Warrant, Permify, Topaz.

> **Core idea**: "user A pode fazer X em Y **se existe relação** entre A e Y." Escala para Google Docs, GitHub, Google Drive — onde cada documento tem N sharings únicos.

**Quando usar**: ver [[authorization-models#Como-escolher]]. Resumo — compartilhamento intenso, hierarquia de recursos (folder/subfolder), grafo de permissões (team → repo → branch).

## Zanzibar 30s

Permission check é percurso em grafo. Tuples (`object#relation@user`) expressam arestas:

```
doc:report-q4#editor@user:alice
doc:report-q4#editor@group:finance#member
group:finance#member@user:bob
doc:report-q4#parent@folder:2026-reports
folder:2026-reports#viewer@user:charlie
```

"alice pode editar doc:report-q4?" → existe tupla direta → **sim**.
"bob pode editar doc:report-q4?" → bob ∈ group:finance#member + finance é editor de report-q4 → **sim** (2 hops).
"charlie pode ler doc:report-q4?" → report-q4.parent = folder:2026-reports + charlie é viewer da folder → **sim** (via herança definida no schema).

## Namespaces e schema

Schema define **relations** por tipo de objeto e computa permissões:

```
// OpenFGA DSL
model
  schema 1.1

type user

type group
  relations
    define member: [user, group#member]     # herança de grupo aninhado

type folder
  relations
    define viewer: [user, group#member]
    define editor: [user, group#member] or owner
    define owner: [user]

type doc
  relations
    define parent: [folder]
    define viewer: [user, group#member] or viewer from parent   # herança de folder
    define editor: [user, group#member] or editor from parent
    define owner: [user]
```

`viewer from parent` = compute viewer herdando de parent folder. "Rewrites" canônicos Zanzibar:
- `this`: relação direta.
- `union`: união de múltiplas fontes.
- `intersection`: interseção.
- `difference`: exclusão.
- `tuple_to_userset`: travessia de grafo (herança via relação).
- `computed_userset`: permissão A implica permissão B.

## Consistência — o diferencial de Zanzibar

Problema: user compartilha doc → outro user tenta abrir imediatamente → race condition.

Zanzibar resolve com **Zookies** (versões causais):
- Toda escrita retorna zookie.
- Client anexa zookie no check.
- Leitor **espera** até replica estar consistent com zookie → sem flaps.

Implementações open-source (OpenFGA, SpiceDB) implementam Zookies também.

## Implementações open-source (2024-2026)

| Engine | Origem | Schema | API |
|---|---|---|---|
| **OpenFGA** | Auth0/Okta (open-sourced 2022) | DSL próprio (spec v1.1) | gRPC + HTTP |
| **SpiceDB** | Authzed (Y Combinator 2021) | Zed DSL | gRPC + HTTP |
| **Permify** | Open-source 2022 | Permify DSL | gRPC + HTTP + GraphQL |
| **Warrant** | Acquired by Zendesk (2024) | Object-oriented | HTTP |
| **Topaz** | Aserto (open-source) | Directory model | gRPC + HTTP |

**Escolha**: OpenFGA tem momentum mais amplo (CNCF Sandbox 2024); SpiceDB é maduro e fiel ao paper.

## Padrão canônico (check, list, expand)

### `check` (pode?)

```
POST /stores/<store-id>/check
{
  "user": "user:alice",
  "relation": "editor",
  "object": "doc:report-q4",
  "contextual_tuples": []       // tuples "what-if"
}
→ { "allowed": true }
```

### `list_objects` (quais docs user pode editar?)

```
POST /stores/<store-id>/list-objects
{
  "user": "user:alice",
  "relation": "editor",
  "type": "doc"
}
→ { "objects": ["doc:report-q4", "doc:report-q3", ...] }
```

### `expand` (quem tem acesso a Y?)

```
POST /stores/<store-id>/expand
{ "relation": "editor", "object": "doc:report-q4" }
→ árvore: usuários + groups + herança via parent
```

Útil para UI "with whom is this shared?".

## Arquitetura de referência

```
[App]                  [Authorization Service]
  │ check(u, r, o)          ┌──────────────┐
  │────────────────────────▶│ PDP (Zanzibar│
  │                          │  spiceDB /   │
  │                          │  openFGA)    │
  │◀────────── allowed?     │              │
  │                          │  ┌─────────┐ │
  │                          │  │ Storage │ │
  │                          │  │ (Postgres│ │
  │                          │  │  / MySQL)│ │
  │                          └──────────────┘
  │
  │ write_tuple(share)
  │────────────────────────▶
```

- **Storage**: Postgres/MySQL (SpiceDB), própria DB interna (OpenFGA + qualquer SQL/NoSQL).
- **Caching**: service local + distributed cache para check volumoso.
- **Replication**: geo-distributed com Zookies para consistency.

## Quando ReBAC é mais adequado

- **Compartilhamento user-to-user intenso** (Google Docs, Notion, Linear).
- **Hierarquia de recursos profunda** (folder → subfolder → file, org → team → repo → branch).
- **Herança de permissões** (viewer de folder = viewer de todos os docs dentro).
- **Delegation fine-grained** (share com quem tenha relação X).
- **"List objects" performance** (RBAC puro obrigaria iteração).

## Quando ReBAC **não** é a resposta

- **Sistema pequeno** com poucos sharings — overhead de infra de grafo não compensa. Use [[acl]] ou [[rbac]].
- **Regras principalmente contextuais** (horário, IP, MFA) — [[abac]]/PBAC; grafo não resolve.
- **Auditabilidade cross-system requer policy-as-code** — [[opa-rego]] (PBAC) pode ser mais adequado.

## Anti-patterns

1. **Implementar Zanzibar do zero** — complexo (zookies, distribuído, consistency). Use OpenFGA ou SpiceDB.
2. **Schema sem review** — schema mal feito explode em queries O(N²). Pensar bem em herança.
3. **Grafo sem limite de depth** — ciclos ou profundidade infinita. Engines têm guardas; config-as-code.
4. **Tuples como log** — cada evento vira tuple. Cresce sem controle. Use apenas para estado atual.
5. **Ignorar zookies** — race condition em sharing recente.
6. **Cache de check sem invalidation** — tuple muda mas cache antigo → authz stale ([[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]]).

## Integração com [[oauth-2|OAuth]] + [[jwt|JWT]]

API flow típico:
1. Client bate API com `Authorization: Bearer <JWT>`.
2. API valida JWT → obtém `sub` (user ID).
3. API chama PDP: `check(user:<sub>, <relation>, <object>)` → allow/deny.
4. API responde.

PDP **não recebe** JWT; API extrai claims e passa atributos relevantes.

## Como grandes empresas usam

- **Google**: Zanzibar original roda Gmail, Photos, YouTube, Drive, Calendar — trilhões de checks/dia.
- **GitHub**: equivalente interno para permissions de repos/teams (não open-source).
- **Auth0/Okta**: OpenFGA é o derivado público de Auth0 FGA (Fine-Grained Authorization) product.
- **Canva / Carta / Intercom**: adotaram SpiceDB/OpenFGA em escala.
- **Airbnb**: Himeji (interno) = sistema Zanzibar-like.

## Relações

- **Decision tree**: [[authorization-models]]
- **Alternativas**: [[rbac]], [[abac]], [[acl]]
- **Engine PBAC alternativa**: [[opa-rego]] (policy-as-code; pode implementar ReBAC)
- **Implementações concretas**: [[zanzibar-openfga]] (nota dedicada)
- **Runtime antipattern**: [[../runtime-antipatterns/op-008-autorizacao-apenas-cache-hit|OP-008]]

## Fontes

- [Zanzibar: Google's Consistent, Global Authorization System (USENIX 2019)](https://research.google/pubs/pub48190/) (consultada 2026-04-24)
- [OpenFGA docs](https://openfga.dev/docs) (consultada 2026-04-24)
- [SpiceDB / AuthZed docs](https://authzed.com/docs) (consultada 2026-04-24)
- [Permify docs](https://docs.permify.co/) (consultada 2026-04-24)
- [Aserto Topaz docs](https://www.topaz.sh/docs) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[authorization-models]]
- [[zanzibar-openfga]]
- [[rbac]]
- [[abac]]
- [[acl]]
- [[opa-rego]]
