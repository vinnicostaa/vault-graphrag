---
title: Context7 MCP — Patterns
type: note
tags: [provider, mcp, context7, patterns, research-loop]
source: https://context7.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Context7 Patterns]
---

# Context7 MCP — Patterns

Padrões atemporais de uso do MCP Context7 em pesquisa + research-loop.

## 1. Integração canônica no research-loop (Etapa 2)

[[../../../30-agents/playbooks/research-loop]] prescreve 6 etapas; Context7 é **obrigatório na Etapa 2** (docs oficiais atualizadas).

```
1. Analisar vault                           # o que já destilado
2. Context7 → resolve + get-library-docs    # doc oficial atualizada  ← OBRIGATÓRIO
3. Engineering blogs big tech               # grandes empresas
4. Patterns aplicáveis                      # 10-knowledge/patterns
5. Dual-check                               # segundo agente
6. Registrar decisão                        # STATE D-###
```

Pular Etapa 2 é causa comum de alucinação de API (modelo cita função que não existe na versão atual).

## 2. Sempre cruzar versão com lockfile local

`get-library-docs` pode retornar doc de `latest`. Projeto pode estar em versão anterior com API diferente.

**Protocolo**:
1. `resolve-library-id("pgx")` → `/jackc/pgx`
2. `get-library-docs("/jackc/pgx", "transactions")` → doc genérica
3. `cat go.sum | grep jackc/pgx` → confirma versão local (ex.: `v5.5.1`)
4. Se doc retornada diverge de `v5.5.1` semantically: buscar no GitHub tags a doc da versão específica (ex.: `/jackc/pgx@v5.5.1`).

## 3. CLI fallback em ambiente sem MCP

Se MCP Context7 não está configurado no host:

```bash
npx --yes ctx7@latest library pgx "connection pool"
# → output: library id "/jackc/pgx" + short description

npx --yes ctx7@latest docs /jackc/pgx "connection pool"
# → output: trechos da doc sobre pool
```

Saída equivalente ao MCP. Útil em subagents que não têm acesso MCP + agente precisa research.

## 4. Tópico específico > doc geral

Sempre passar `topic` quando conhece o sub-assunto:

```
get-library-docs("/jackc/pgx")                     # doc geral, muito grande
get-library-docs("/jackc/pgx", "BeginTxFunc")      # cirúrgico
```

Reduz tokens + precisão maior.

## 5. Cache local da resposta durante sessão

Context7 fetch não é gratuito (tempo + quota). Dentro da mesma sessão do agente, salvar resposta em variável/nota e referenciar, em vez de re-fetch.

Em modo autônomo longo: se a mesma lib/tópico for re-consultado, **primeira resposta vale** até novo commit no `go.sum` do projeto.

## 6. Falha de Context7 não bloqueia — fallback controlado

Se `resolve-library-id` falha (lib não indexada):
1. Tentar CLI fallback (`npx ctx7`).
2. Se ainda falha: `WebFetch` no site oficial da lib.
3. Se ainda falha: grepa `node_modules/<lib>/README.md` ou `$GOMODCACHE/<lib>/README.md`.
4. **Registrar** em STATE como `⚠️ research limitation` — fontes alternativas usadas.

**Nunca** assumir que "deve funcionar" sem validar. Anti-alucinação.

## 7. Awareness de breaking changes entre majors

Quando agente percebe diff entre doc atual e uso local:
- Verificar `CHANGELOG.md` da lib (Context7 pode não ter).
- Confirmar via WebFetch em `github.com/<lib>/releases`.
- Se breaking change em major, alertar operador antes de aplicar mudança.

## 8. Integração com dependency-audit playbook

[[../../../30-agents/playbooks/dependency-audit]] pode usar Context7 para validar cada dep na auditoria:
- Version atual local vs latest no Context7.
- API calls ainda válidos na latest?
- Breaking change anunciado?

Context7 é **research**, não scanner de CVE — combinar com `govulncheck`/`npm audit` via [[../../../30-agents/playbooks/cve-scan]].

## 9. Versioning awareness na destilação

Ao destilar doc em nota do vault (`10-knowledge/providers/<lib>/`), incluir no frontmatter:

```yaml
source: https://docs.<lib>.com
source_date: 2026-04-24
lib_version: v5.5.1  # versão consultada
```

Permite detectar staleness: se passar 6 meses, fazer re-consulta.

## 10. Preferência sobre memória de treino

**Regra dura** (§6 CLAUDE anti-alucinação): nunca citar API de lib baseado apenas em memória de treino. Sempre Context7 + version lookup. Se Context7 indisponível: documentar limitação.

## Links

- [[_moc]]
- [[overview]]
- [[../_moc]]
- [[../../../30-agents/playbooks/research-loop]] — Etapa 2
- [[../../../30-agents/playbooks/dependency-audit]]
- [[../../../30-agents/playbooks/cve-scan]]
- [[../../../30-agents/autonomous-execution]] — Fase 1 Levantamento
