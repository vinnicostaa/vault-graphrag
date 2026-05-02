---
title: OP-006 — Cache falha quebra fluxo principal
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - cache
  - resilience
id: OP-006
severity: High
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Cache hard-dependency
doc-consulted: '2026-04-24'
---

# OP-006 — Cache falha quebra fluxo principal

**Severidade**: High (disponibilidade)
**Categoria**: Cache

## Sintoma

`cacheProvider.get()` lança exceção (Redis indisponível, timeout, network partition) e a exceção sobe pelo stack. Resultado: cache down = produto down. Cache deixou de ser otimização e virou dependência dura.

## Por que é problema

- **Disponibilidade**: sistema inteiro fica offline quando cache cai, mesmo com banco íntegro.
- **Fragilidade**: cache é deliberadamente efêmero; tratar como dep crítica contradiz seu papel.
- **Blast radius**: upgrade/reboot do Redis vira incidente.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — cache down derruba o handler
abilities, err := cache.Get(ctx, key)
if err != nil {
    return nil, err  // Redis down → 500 pro usuário
}
```

## Fix preferido — cache como fallback, não pré-requisito

```go
func (s *AbilitiesService) Get(ctx context.Context, userID string) ([]Ability, error) {
    // Tentativa best-effort no cache
    if cached, err := s.cache.Get(ctx, key); err == nil && cached != nil {
        return cached, nil
    } else if err != nil {
        s.logger.Warn("cache miss com erro, seguindo para fonte de verdade",
            "err", err, "key", key)
    }

    // Fonte de verdade
    abilities, err := s.computeFromDB(ctx, userID)
    if err != nil { return nil, err }

    // Write best-effort no cache (erro não propaga)
    if err := s.cache.Set(ctx, key, abilities, ttl); err != nil {
        s.logger.Warn("cache set falhou (não crítico)", "err", err)
    }
    return abilities, nil
}
```

## Alternativa

- Circuit breaker no cliente cache: se N falhas seguidas, bypass por X segundos evitando timeout em cada request.
- Cache local in-memory (LRU) como L1 antes do L2 Redis.

## Quando tolerar

Raríssimo. Se o cache é sessão/lock distribuído (não cache no sentido clássico) e ausência dele viola correção, aí é dep dura mesmo — explicitar no código + documentar como dependência, não cache.

## Relações

- **Patterns**: Circuit Breaker, [[../patterns/proxy]] (cache-aside transparent)
- **OPs relacionados**: [[op-005-cache-sem-versionamento]], [[op-004-sem-retry-strategy]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-006
- Netflix Hystrix — stability patterns
- AWS ElastiCache — [Best practices: availability](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/BestPractices.HA.html)

## Links

- [[_moc]]
- [[../patterns/proxy]]
- [[../observability/alerting]]
- [[op-005-cache-sem-versionamento]]
