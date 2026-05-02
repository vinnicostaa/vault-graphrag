---
title: MOC — Context7 MCP (doc oficial de libs)
type: moc
tags: [moc, provider, mcp, context7, documentation]
category: documentation
mcp: https://mcp.context7.com (hosted) / stdio local
sdk-official: npx ctx7@latest (CLI fallback)
doc-oficial: https://context7.com
doc-consulted: 2026-04-24T00:00:00.000Z
created: 2026-04-24
updated: 2026-04-24
aliases: [Context7 MCP, ctx7]
---

# Context7 MCP

**Categoria**: documentation retrieval · **obrigatório antes de afirmar API de lib** (§6 CLAUDE anti-alucinação; research-loop Etapa 2).

**MCP disponível**: ✅ hosted (`mcp.context7.com`) + stdio local via npm.

**CLI fallback** (quando MCP indisponível):
```bash
npx --yes ctx7@latest library <name> "<query>"        # resolve library-id
npx --yes ctx7@latest docs <libraryId> "<query>"      # fetch docs
```

**Doc oficial**: https://context7.com + https://github.com/upstash/context7 (servidor open source).

## Quando usar

1. **Antes de usar lib externa em código**: resolver library-id + get-library-docs pela versão exata do `go.mod` / `package.json` / `Cargo.toml`.
2. **Durante research-loop (Etapa 2)**: validar afirmação técnica contra fonte oficial atualizada.
3. **Before dual-check**: garantir que fonte consultada é a mesma versão em produção.
4. **Durante CVE response**: validar se API vulnerável da versão em uso existe.

## Quando **NÃO** substitui

- Deep tutorial ou guia conceitual (ex.: "como escrever clean architecture") — docs oficiais geralmente não cobrem.
- Debugging com contexto específico — logs + source code cache local é melhor.
- Patterns emergentes não-oficiais (engineering blogs, talks).

## Funções principais

| Função | Uso |
|---|---|
| `resolve-library-id(name)` | Mapeia nome humano → ID canônico Context7 (ex: `react` → `/facebook/react`) |
| `get-library-docs(libraryId, topic?)` | Fetch docs filtradas por tópico (opcional) |

## Padrão de uso (research-loop Etapa 2)

```
1. resolve-library-id("pgx")
   → /jackc/pgx
2. get-library-docs("/jackc/pgx", "transactions")
   → retorna trechos da doc sobre BeginTx/BeginTxFunc
3. Cross-check versão no lockfile local:
   cat go.sum | grep jackc/pgx    # bate com doc-consulted?
4. Se divergência de versão → fetch doc da versão específica via GitHub tag
```

## Antipattern

- ❌ **Confiar em treinamento do modelo** para API de lib — treino pode estar stale (meses a anos). Context7 entrega **versão atual**.
- ❌ Pular Context7 "porque conhece a lib" — falsa confiança; API muda.
- ❌ Aceitar resposta Context7 sem conferir versão (a lib pode ter breaking change entre 2 versões que o agente consultou).

## Glossário

- **library-id**: identificador canônico `<org>/<repo>` usado como chave.
- **topic**: filtro opcional de tópico/seção dentro da lib (ex.: "streaming", "authentication").

## Sub-docs

- [[overview]] — arquitetura (crawler + indexer + MCP server)
- [[patterns]] — integração com ciclo SDD, versioning awareness, cache

## Vizinhos

- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../../../30-agents/playbooks/research-loop]] — etapa 2 obrigatória
- [[../../../30-agents/autonomous-execution]] — Fase 1 Levantamento
