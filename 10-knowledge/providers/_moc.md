---
title: MOC — 10-knowledge/providers
type: moc
tags: [moc, providers, integrations, saas]
created: 2026-04-22
updated: 2026-04-24
---

# 10-knowledge/providers — Provedores SaaS / infra

Nó global do vault com conhecimento **atemporal e reutilizável** sobre provedores externos (SaaS, infra, auth, pagamentos, video, realtime, etc.) que aparecem na stack de vários projetos.

> **Filosofia**: conhecimento de provedor é **global** (Stripe funciona igual em qualquer projeto; webhook idempotency é padrão universal). Cada projeto consome esse nó e adiciona **só** a parte específica (chaves, produtos cadastrados, planos do negócio). Evita duplicação e deixa o agente com contexto amplo quando troca de projeto.

---

## Providers ativos no vault (2026-04-24 pós-Passada 2+3)

### Providers SaaS de domínio (operacionais)

| Provider | Categoria | MCP disponível? | Doc consultada | Status destilação |
|---|---|---|---|---|
| [[stripe/_moc\|Stripe]] | Pagamentos / Connect / Subscriptions | ✅ [mcp.stripe.com](https://mcp.stripe.com) | 2026-04-23 | ✅ **destilado** (overview, integration-guide, patterns, antipatterns, usage-in-projects — D-vault-009) |
| [[mux/_moc\|Mux]] | Video streaming on-demand + live | ❌ | 2026-04-23 | ✅ **destilado** (5 sub-docs — D-vault-009) |
| [[zitadel/_moc\|Zitadel]] | Identity (OIDC, M2M, ABAC) | ❌ | 2026-04-23 | ✅ **destilado** (5 sub-docs — D-vault-009) |
| [[livekit/_moc\|Livekit]] | WebRTC realtime (audio/video/chat) | ❌ | 2026-04-23 | ✅ **destilado** (5 sub-docs — D-vault-009) |

### Providers de infraestrutura / plataforma

| Provider | Categoria | MCP disponível? | Doc consultada | Status destilação |
|---|---|---|---|---|
| [[railway/_moc\|Railway]] | PaaS (deploy + Postgres + Redis) | ❌ | 2026-04-23 | ✅ **destilado** (5 sub-docs — D-vault-009, recuperação manual A-vault-042) |
| [[cloudflare/_moc\|Cloudflare]] | **Edge cloud** (CDN + Workers + R2 + D1 + KV + DOs + Zero Trust + 30+ produtos) | ✅ **14 MCPs oficiais** (`*.mcp.cloudflare.com/mcp`) | 2026-04-24 | 🟡 **piloto em curso** — 6 núcleo + 8 produtos piloto (workers/pages/r2/d1/kv/DOs/access/tunnel — D-vault-020). 11 produtos em backlog lazy. |

### MCPs de ferramenta (agente-facing)

| MCP | Categoria | MCP URL | Doc consultada | Status destilação |
|---|---|---|---|---|
| [[obsidian/_moc\|Obsidian]] | vault mutation | stdio local | 2026-04-24 | 🟡 _moc destilado + sub-docs pendentes (D-vault-010, A-vault-027 parcial) |
| [[context7/_moc\|Context7]] | doc oficial de libs | [mcp.context7.com](https://mcp.context7.com) | 2026-04-24 | 🟡 _moc destilado + sub-docs pendentes |
| [[github/_moc\|GitHub]] | repo/PR/issues | hosted | 2026-04-24 | 🟡 _moc destilado + sub-docs pendentes |
| [[aws-docs/_moc\|AWS Docs]] | doc oficial AWS | awslabs/mcp | 2026-04-24 | 🟡 _moc destilado + sub-docs pendentes |

**Pendências conhecidas** (não tem pasta ainda):
- **Figma MCP** — projeto-específico (uso incidental); destilar só quando projeto concreto demandar.
- **Postgres MCP** (crystaldba) — stack-specific; cobrir em `20-stacks/postgres/`.

Status: 🟢 destilado completo · 🟡 stub + _moc rico · 🔴 só pasta vazia

---

## Estrutura canônica de cada provider

Cada pasta `providers/<provider>/` deve ter:

- **`_moc.md`** — índice do provider + campos obrigatórios (ver abaixo)
- **`overview.md`** — o que é, categoria, modelo de negócio, vs concorrente
- **`integration-guide.md`** — passo-a-passo de integração oficial por ambiente (test → prod)
- **`patterns.md`** — padrões atemporais (idempotência de webhook, retry/backoff, reconciliação, rate limit, circuit breaker)
- **`antipatterns.md`** — erros comuns que o agente deve evitar
- **`usage-in-projects.md`** — tabela: projeto | uso concreto | link pro aplicado em `50-projects/<slug>/`

Campos obrigatórios no `_moc.md` de cada provider:

- **Categoria** (Pagamentos / Auth / Video / Realtime / Email / PaaS / vault-mutation / documentation / etc.)
- **MCP** — server MCP disponível? URL ou "❌ não existe"
- **SDKs oficiais** — linguagens + versão mínima recomendada
- **Doc oficial** — URL + data consultada
- **Sandbox/test mode** — como acessar
- **Modelo de preço** — resumo (pra decisão arquitetural)
- **Quando usar** — 3–5 cenários onde esse provider é first-choice
- **Quando NÃO usar** — concorrentes que vencem em casos específicos
- **Glossário interno do provider** — apenas termos que o agente precisa entender

---

## Como o agente usa este nó

1. **Antes de integrar provider novo num projeto**: ler `<provider>/_moc.md` completo (obrigatório).
2. **Antes de escrever webhook handler**: ler `<provider>/patterns.md` (idempotência, replay, assinatura).
3. **Dual-check (§5 do [[../../CLAUDE]])**: se decisão envolve provider, consultar `doc oficial` + `data consultada` no `_moc.md`. Se > 30 dias: re-verificar com Context7 MCP antes de afirmar.
4. **MCP first**: se provider tem MCP (ex: Stripe, Obsidian, Context7), priorizar ele pra consultas em tempo real.
5. **Onboarding novo projeto**: `usage-in-projects.md` mostra quais projetos já usam e como — copiar padrão vs. criar novo.
6. **Research-loop Etapa 2**: [[context7/_moc]] é obrigatório para validar APIs de libs.

---

## Vizinhos no grafo

- [[../_moc]] — 10-knowledge raiz
- [[../../20-stacks/_moc]] — stacks (Go/TS/Rust/Flutter) que consomem providers
- [[../../30-agents/mcp-integration]] — uso agregado de MCPs
- [[../../30-agents/playbooks/dependency-audit]] — auditoria periódica inclui providers
- [[../../00-meta/STATE]] — D-vault-009 + D-vault-010

## Regra dura

- Nada aqui é específico de projeto. Se escreveu "Master Síndico", "Condomínio", "Assembleia" — errou de arquivo, vai pra `50-projects/<slug>/`.
- `usage-in-projects.md` **linka** pro aplicado, não replica o conteúdo.
- Data de consulta da doc é **obrigatória** no `_moc.md` de cada provider (providers mudam rápido).
