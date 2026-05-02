---
title: Fighting spam with Haskell — Sigma anti-abuse system
type: source
tags: [research, meta, sigma, haskell, haxl, fxl, spam, abuse, anti-fraud, source]
source: https://engineering.fb.com/2015/06/26/security/fighting-spam-with-haskell/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-10-sigma-haskell]
---

# source-10 — Sigma / Haskell (spam & abuse)

- **URL**: https://engineering.fb.com/2015/06/26/security/fighting-spam-with-haskell/
- **URL relacionada (FXL original 2013)**: https://engineering.fb.com/2013/01/24/web/fighting-spam-with-pure-functions/
- **Publicado**: 2015-06-26 (Haskell migration) — antes, FXL em 2013-01-24
- **Autor/Canal**: Engineering at Meta
- **Tipo**: post técnico
- **Contexto**: Sigma é o sistema de detecção proativa de spam/phishing/malware do Facebook. Originalmente escrito em FXL (Feature eXtraction Language, patterned after SML). Migrado pra Haskell em 2015 pra ganhar abstração + performance.

## Sinopse

Sigma classifica em tempo real ações maliciosas no Facebook (spam, phishing, links de malware). Migrou de FXL (linguagem interna Facebook, 2013) pra Haskell com framework **Haxl** (2015). Pós-migração: +20-30% throughput, alguns request types 3x mais rápidos, 1M+ req/s.

## Pontos-chave

1. **Sigma = regra engine** pra detecção proativa de abuso — spam, phishing, malware.
2. **FXL (2013)** — DSL funcional, tipagem forte, patterned em SML. Bom mas faltou abstrações (tipos definidos pelo usuário, módulos).
3. **Haxl (2015)** — framework Haskell que faz fetch de dados implicitamente concorrente. Engenheiro escreve policy como função pura; Haxl paraleliza sozinho.
4. **Resultados Haskell**: +20-30% throughput geral, até 3x em alguns request types, 1M+ req/s.
5. **Contribuições upstream ao GHC** — Meta corrigiu bugs críticos (GC crash "every few hours", finalizer handling).
6. **Adversarial learning** (contexto): adversários fazem reverse-engineering das features, limitando ground truth. Exige retreino contínuo.

## Aplicabilidade ao Master Síndico

**Stack Haskell não aplicável** — backend Master Síndico é Go (inegociável no padrão canônico).

**Patterns conceituais aplicáveis** ao sistema de anti-fraud / moderação:

1. **Regra engine desacoplada do código de aplicação** — moderação e anti-abuse viram políticas executáveis, versionáveis, hot-reloadable. Em Go, opções:
   - Scripts em Lua/JS embarcados (GopherLua, goja).
   - Cedar/OPA (Open Policy Agent) — declarativo, cross-stack, usado em BeyondCorp-style ABAC.
   - DSL custom YAML + interpretação (mais simples, menos flexível).
2. **Feature fetcher com paralelização automática** — Haxl inspirou bibliotecas Go como `dataloader`. Relevante se moderação precisar buscar 10 atributos do usuário/condomínio em paralelo pra decidir.
3. **Sinais de aversão real-time** — Sigma processa ações em stream. Pro Master Síndico, quando moderação ML chegar (M3+), arquitetura event-driven (Kafka/SNS/EventBridge) permite classificação em tempo quase real.
4. **Política: declarar regra, não codificar** — `"se usuário criado há <7 dias AND postou 10+ vídeos AND 0 likes, marcar review"` — como configuração, não código em handler. Facilita auditoria + mudança rápida.

**Conexão com `source-08` (moderation)**: Sigma é específico de spam/fraude. Moderação de conteúdo (hate speech, PII vazada) é pipeline adjacente mas não idêntico. Master Síndico pode começar com um único "moderation engine" simples e separar em dois só quando necessidades divergirem.

## Citações

> "Its job is to proactively identify malicious actions on Facebook, such as spam, phishing attacks, posting links to malware, etc."

> "The Haskell-powered version now runs in production, serving more than one million requests per second."

> "20 percent to 30 percent improvement in overall throughput" vs FXL.

## Confiança / datas

- Data confirmada (2015-06-26) — **alta confiança**.
- FXL original (2013-01-24) — **alta confiança**.
- Performance gains — **alta confiança** (citados no próprio post Meta).
- Fonte primária Meta Engineering.
