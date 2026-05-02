---
title: MOC — Zitadel (Identity + OIDC + M2M + ABAC)
type: moc
tags: [moc, provider, zitadel, identity, oidc, auth, abac]
category: identity
mcp: null
sdk-official-go: zitadel-go v3 (github.com/zitadel/zitadel-go/v3)
sdk-official-node: "@zitadel/node"
doc-oficial: https://zitadel.com/docs
doc-consulted: 2026-04-23
created: 2026-04-22
updated: 2026-04-23
---

# Zitadel

**Categoria**: Identity provider open-source · OIDC / OAuth2 · Machine-to-Machine (JWT Profile) · granular permissions (ABAC-ready).

**MCP disponível**: ❌ — usar WebFetch em https://zitadel.com/docs ou Context7 `resolve-library-id "zitadel"`.

**SDKs oficiais**:
- Go: `github.com/zitadel/zitadel-go/v3` (V4 stable em 2026)
- Node: `@zitadel/node`
- Web: `@zitadel/web`
- Dart: community plugin

**Doc oficial**: https://zitadel.com/docs — consultada em 2026-04-23.

**Self-hosted vs Cloud**: cloud = `<tenant>.zitadel.cloud` (primary recomendado em 2026); self-host via Docker Compose / K8s / Kubernetes operator.

**Modelo de preço**: free tier generoso (até 25k MAUs) · pay-per-user acima. Self-hosted = infra only.

---

## Quando usar Zitadel

1. **Identity provider OIDC-compliant** com granular permissions fora-da-caixa
2. **Multi-tenant** (organizations nativas; não precisa herdar tenancy manualmente)
3. **Audit trail imutável** de eventos de identity (compliance)
4. **Custom login UI** + self-service (reset password, MFA, passwordless)
5. **JWT Profile (RFC 7523)** para serviço-a-serviço com key assimétrica — alternativa superior a client_secret

## Quando NÃO usar Zitadel

- **Apenas social login simples** → Auth0 free ou Clerk são mais plug-and-play
- **Brazilian compliance very-strict** → gov.br ou hub nacional pode ser obrigatório
- **Equipe sem operações DevOps pra self-host** → cloud simplifica mas custa acima 25k MAUs

---

## Sub-docs do Zitadel

- [[overview]] — organizações, projects, applications, users, roles, actions
- [[integration-guide]] — OIDC flow + M2M JWT Profile + introspection
- [[patterns]] — introspection + cache, refresh token rotation, ABAC via custom claims
- [[antipatterns]] — ex: guardar access_token; comparar `sub` como chave primária local
- [[usage-in-projects]] — quais projetos

## Glossário interno do Zitadel

- **Organization** — tenant raiz; tem projects, users, roles.
- **Project** — aplicação lógica; agrupa applications (clients OIDC).
- **Application** — client OIDC; tem client_id, redirect URIs, flow type.
- **User** ≠ Human User. Pode ser Machine User (service account com key JSON para JWT Profile).
- **Grant** — ligação user↔project↔role (com escopos); é onde ABAC pode viver.
- **Action** — serverless function JavaScript executada em eventos (pre-token, post-login). Análogo a Auth0 Actions.
- **Introspection** — endpoint RFC 7662 pra validar access_token opaco server-side.

## Fontes brutas (pendente destilar)

> **Estado**: destilação pendente. Zitadel não tem arquivo bruto em `contextos/` — conteúdo tem que ser destilado direto da doc oficial + uso no master-sindico. Ver [[usage-in-projects]].

## Vizinhos no grafo

- [[../_moc]]
- [[../../security/_moc]] — OIDC patterns, token storage
- [[../../patterns/_moc]] — introspection + cache é padrão reutilizável
