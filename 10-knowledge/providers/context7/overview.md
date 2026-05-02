---
title: Context7 MCP — Overview
type: note
tags: [provider, mcp, context7, overview, documentation]
source: https://context7.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Context7 Overview]
---

# Context7 MCP — Overview

Servidor MCP que entrega **documentação oficial atualizada** de libs de programação para agentes LLM. Resolve o problema de **staleness do treino**: modelos foram treinados em cutoff date específico, APIs de libs mudam continuamente, treino fica desatualizado.

## Por que Context7 existe

**Problema**: Claude treinado em data X tem doc de `react@18`. Projeto usa `react@19`. API mudou. Modelo gera código válido em 18, quebrado em 19.

**Solução Context7**: fetch da doc **oficial** da versão atual, indexado semanticamente, entregue via MCP ao agente no momento do uso.

## Arquitetura

```
Claude host
   │ (MCP call)
   ▼
Context7 server (mcp.context7.com — hosted)
   │
   ├─→ Crawler periódico → docs oficiais (upstream)
   ├─→ Indexer (embedding + keyword)
   └─→ Query API (resolve-library-id + get-library-docs)
```

Mantido por [Upstash](https://upstash.com/). Open source.

## Funções principais

| Função | Uso |
|---|---|
| `resolve-library-id(libraryName)` | Mapeia nome humano (ex.: `react`, `pgx`, `zustand`) para ID canônico Context7 (`/facebook/react`, `/jackc/pgx`, `/pmndrs/zustand`) |
| `get-library-docs(libraryId, topic?)` | Retorna trechos da doc filtrados por tópico (opcional). Sem tópico: doc geral |

## CLI fallback (quando MCP indisponível)

```bash
npx --yes ctx7@latest library <name> "<query>"    # step 1: resolve
npx --yes ctx7@latest docs <libraryId> "<query>"  # step 2: fetch
```

Útil em ambientes onde MCP Context7 não está instalado mas `npx` roda. CLI produz output equivalente.

## Casos de uso — quando usar

1. **Antes de usar lib externa em código** — validar API pela versão exata do `go.mod`/`package.json`/`Cargo.toml`.
2. **Durante research-loop (Etapa 2)** — validar afirmação técnica contra fonte oficial.
3. **Antes de dual-check sobre decisão arquitetural** — garantir que fonte consultada é versão em produção.
4. **Durante CVE response** — validar se API vulnerável existe na versão em uso.
5. **Migração de lib** (ex.: `react@18 → react@19`): pegar doc de ambas versões para diff conceitual.

## Quando **NÃO** substitui outras fontes

- **Tutorial conceitual profundo** ("como fazer clean architecture"): doc oficial geralmente não cobre. Usar livros/blogs/talks.
- **Patterns emergentes não-oficiais**: engineering blogs de empresas (Netflix, Stripe) têm padrões que docs ignoram.
- **Debugging com logs locais**: source code cache local (`$GOMODCACHE`, `node_modules`) é melhor.
- **Casos edge**: issues do repo GitHub revelam bugs conhecidos que doc oficial não admite.

## Comparação com alternativas

| Alternativa | Quando vence |
|---|---|
| `WebFetch` diretamente no site oficial | Lib não indexada no Context7 ou precisa HTML renderizado |
| Source code cache local (`node_modules/`, `$GOMODCACHE`) | Debugging com shape real da estrutura |
| Repo GitHub READMEs | README tem exemplos rápidos; Context7 tem API completa |
| Stack Overflow / blogs | Patterns em produção; Context7 é doc "como fazer" |

## Limitações

- **Cobertura**: nem toda lib está indexada. Context7 prioriza libs populares. Libs pequenas podem não ter.
- **Delay de indexação**: nova release pode demorar horas/dias para indexar.
- **Sem version-pinning explícito em todas libs**: alguns retornos são "latest", não "pinned to X.Y.Z" — agente deve conferir version no lockfile local.

## Links

- [[_moc]]
- [[patterns]]
- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../../../30-agents/playbooks/research-loop]] — Etapa 2 obrigatória
