---
title: Comments (excessive) (AP-016)
type: antipattern
tags: [antipattern, code-smell, dispensable, readability]
source: https://refactoring.guru/smells/comments
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [over-commenting, deodorant-comments]
---

# AP-016 — Comments (excessive)

Comentário **não é inerentemente ruim** — mas quando é usado como *deodorante* (pra disfarçar código obscuro), vira code smell. "Ah, esse trecho é complicado, deixa eu explicar com 10 linhas de `//`." Não. Melhore o código.

## Sintomas

- Bloco de comentário explicando **o que** uma função faz (nome devia bastar)
- "Explain the workaround" com 20 linhas — provavelmente é Extract Function
- Comentários que contradizem o código (desatualizados)
- `// TODO: fix this` de 2 anos atrás, ninguém lembra o contexto
- Comentário `// HACK:` seguido de gambiarra permanente
- Cabeçalhos ASCII-art `// ========= SECTION =========`
- `// Step 1:`, `// Step 2:` — passos são nome de função, não comentário

## Por que é problema

**Mentira em potencial**: compilador não valida comentário. Código muda, comentário fica. Em pouco tempo, comentários viram noise contraditório.

**Distração**: leitor precisa validar se o comentário ainda reflete a realidade. Comentário errado é pior que nenhum.

**Mascarar complexidade**: comentário que "explica por que isso é assim" frequentemente sinaliza que a abstração está errada — extrair função/classe com nome bom resolve.

**Ruído visual**: 40% de linhas que são `//` afeta scanning e onboarding. Código limpo é o que o nome promete.

## Exemplo

```python
# ❌ Deodorant comment
def p(d):
    # Calculate the price with discount applied
    # if customer is gold, apply 20%, else 5% for silver, nothing otherwise
    if d.c == "G":
        return d.p * 0.8
    elif d.c == "S":
        return d.p * 0.95
    return d.p

# ✅ Código auto-explicativo dispensa o comentário
GOLD_DISCOUNT   = 0.20
SILVER_DISCOUNT = 0.05

def apply_customer_discount(price: Money, tier: CustomerTier) -> Money:
    match tier:
        case CustomerTier.GOLD:   return price.discount(GOLD_DISCOUNT)
        case CustomerTier.SILVER: return price.discount(SILVER_DISCOUNT)
        case _:                   return price
```

## Como refatorar

- **Extract Method** com nome que **é** o comentário: `// validate inputs` → `validateInputs()`
- **Rename Variable/Method**: renomear torna comentário redundante
- **Introduce Explaining Variable**: `if (x > 86_400)` → `if (elapsed > ONE_DAY_SECONDS)`
- **Delete dead comments**: comentários desatualizados ou `TODO` órfãos saem (substituir por issue rastreável)
- **Assertion** em vez de comentário: `// must be non-null` → `assert input != null` ou validação explícita

## Quando tolerar (comentário bom)

Comentários **legítimos** que não são smell:

- **Por que** (e não o que): rationale de decisão não óbvia, trade-off, workaround de bug upstream com link
- **Avisos legais**: licença no topo, copyright
- **API pública**: docstrings que geram documentação (Javadoc, TSDoc, rustdoc, Godoc)
- **Ref a ticket/RFC**: `// see RFC-7231 §6.3.1` ou `// fixes #4523`
- **Aviso de invariante/contrato** difícil de enforcer por tipo
- **Performance caveat**: "este loop é hot path, evitar alloc"
- **Segurança**: `// safe: caller validated schema upstream`

Regra: comentário explica **por que**; código explica **o que** e **como**.

## Patterns relacionados (que resolvem)

- [[../principles/clean-code]] — "Comments are a last resort" (Martin)
- [[long-method]] — funções curtas frequentemente não precisam de comentário
- [[magic-numbers]] — nome de constante substitui `// significa X segundos`
- [[../principles/code-calisthenics]] — naming como primeira ferramenta

## Fontes

- Refactoring Guru — [Comments](https://refactoring.guru/smells/comments)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Martin, R. C. (2008). *Clean Code*, cap. 4 — "Comments".
- Kernighan, B. & Pike, R. (1999). *The Practice of Programming* — "don't comment bad code, rewrite it".

## Links

- [[_moc]]
- [[../_moc]]
- [[../principles/clean-code]]
- [[long-method]]
- [[magic-numbers]]
