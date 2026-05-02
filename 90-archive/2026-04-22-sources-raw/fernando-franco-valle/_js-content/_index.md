---
title: Fernando — conteúdo extraído dos bundles JS
type: source
tags: [source, fernando, next-js, js-extracted]
created: 2026-04-22
---

# Conteúdo extraído dos JS bundles (Next.js Turbopack)

Site `fernandofrancovalle.com` é SPA Next.js com React Server Components. Parte do conteúdo pt-BR das trilhas (AWS, IA, Claude, Engenharia) **não está nos HTMLs** — mora como strings dentro dos bundles JS de `_next/static/chunks/`.

Esta pasta contém strings com 50+ caracteres filtradas de cada bundle. **Nota**: extração é heurística — mistura prose real com fragmentos de código JS. Limpeza manual ou com LLM é recomendada antes de destilar.

## Bundles com conteúdo

| Bundle | Linhas | Conteúdo provável |
|---|---|---|
| `0dd57-5l6osnf.md` | 1892 | Bundle principal (curriculum, trilhas, descrições de aulas, quiz data) |
| `0x0.07m2n1_ce.md` | 779 | (vazio/misto — verificar) |
| `0h9l_p_xvhu-7.md` | 465 | |
| `03_yq9q893hmn.md` | 411 | |
| `0z88i6hep6p5n.md` | 380 | |
| `0n0vt6q67jgi3.md` | 341 | |
| `0i.l9589uvx0j.md` | 170 | |
| `0om5pg-iw3c7f.md` | 140 | |
| `0zjs5hlwbvch9.md` | 141 | Framework Next.js runtime (descarte) |
| `0abilttusqay6.md`, `120jgy8djys9-.md`, outros | <100 | Pequenos chunks, verificar |

Total: **4998 linhas** de strings extraídas.

## Método de extração

```bash
for js in fernandofrancovalle.com/_next/static/chunks/*.js; do
  grep -oP '"[^"]{50,}"' "$js" | sort -u | \
    grep -vE '^(https?://|/[a-zA-Z0-9_-]+|\./|\.\./)' | \
    grep -vE '^(globalThis|TURBOPACK|__esModule|Object\.defineProperty)' | \
    grep -vE '^(Bail out|Dynamic server|Could not validate)' | \
    grep -E '[a-záéíóúçãõâêîôûäëïöü]+ [a-záéíóúçãõâêîôûäëïöü]+'
done
```

Filtros aplicados:
- Mín 50 caracteres
- Remove URLs e paths
- Remove identificadores JS (globals, TURBOPACK)
- Remove mensagens de erro do framework Next.js
- **Exige** padrão de 2+ palavras com letras pt-BR (prose)

## Tópicos detectados (amostra)

AWS: CCP/SAA/SAP/DVA/SOA certs, Migration, VPC, IAM, ECS, Lambda, CloudWatch, cost optimization  
IA/Claude: Anthropic API, prompt caching, context window, MCP, agents, tool use, vision, extended thinking  
Engenharia: DDD, Clean Arch, CQRS, Saga, ACID, ADR, SOLID, TDD, property-based testing  
Web: Accessibility (WCAG 2.2, POUR, ADA), A/B testing, feature flags, observability (OTel, Grafana), Next.js  
Banco: PostgreSQL, isolation levels, rate limiting, token bucket  
Mobile: Android Kotlin + Compose, iOS Swift, React Native  
Security: OWASP Top 10, LGPD, rate limiting, webhook HMAC

## Próximos passos

Pra usar como fonte de destilação:

1. **Filtrar noise**: rodar cada `.md` em passada LLM pra separar prose real de código JS vazado
2. **Clusterizar por tópico**: usar Smart Connections pra achar temas recorrentes
3. **Destilar tópicos relevantes**:
   - AWS specs → `[[../../../20-stacks/aws/_moc]]` *(a criar)*
   - Claude API patterns → `[[../../../30-agents/claude-api-patterns]]` *(a criar)*
   - Observability → `[[../../../10-knowledge/observability/_moc]]`
   - Accessibility → `[[../../../10-knowledge/frontend/accessibility]]` *(a criar)*
4. **Arquivar raw** após destilação — estas notas são matéria-prima, não conhecimento final

## Aviso

Este conteúdo foi extraído de código JS minificado — **alguns trechos podem estar fora de ordem ou incompletos** (strings template com interpolação resultam em strings parciais). Sempre validar contra o HTML renderizado correspondente quando possível.

## Links

- [[../_index]] — Fernando (source raiz)
- [[../../_moc]] — sources
- [[../../../30-agents/_moc]] — destino da destilação de patterns Claude/Agents
- [[../../../10-knowledge/_moc]] — destino de conhecimento técnico
