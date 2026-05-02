---
title: Requirements — índice por domínio
version: 1.0
status: stable
---

# Requirements — Master Síndico

Requisitos funcionais organizados **por bounded context DDD**. Cada arquivo é self-contained para uma leitura dirigida por domínio. Para contexto completo com todos os Reqs numerados (Req 1–103), ver o original em `../requirements.md` (será deprecado quando todos os recortes estiverem completos).

## Navegação por domínio

| Arquivo | Domínio | Reqs principais |
|---|---|---|
| [`product-overview.md`](product-overview.md) | Transversal | Decisões 1-12, regras de ouro, marcos |
| [`identity.md`](identity.md) | Identity | Req 1-3 (auth, authz, registro) |
| [`identity-mirror-pattern.md`](identity-mirror-pattern.md) | Identity | **Novo** — decisão User/Session ↔ Zitadel |
| [`billing.md`](billing.md) | Billing | Req 4, 4.1, 5, 6 (planos, trial, paywall) |
| [`institutional.md`](institutional.md) | Institutional | Req 6.2-6.4 + Req 7-17 |
| [`commercial.md`](commercial.md) | Commercial | Req 18-27.3 (Connect Me, propostas, vizinhança) |
| [`content.md`](content.md) | Content | Req 29-32, 97, 98.1 |
| [`assembly.md`](assembly.md) | Assembly | Req 48-60.12 |
| [`cross-domain.md`](cross-domain.md) | Cross-Domain | Req 33-47, 61-67, 102, 103 |
| [`talent-bank.md`](talent-bank.md) | Talent Bank | Banco de Talentos (Decision 6) |
| [`admin-panel.md`](admin-panel.md) | Admin MS | Painel Admin |
| [`personas-and-plans.md`](personas-and-plans.md) | Transversal | Personas × Planos × Trials × Quotas |
| [`functional-matrix.md`](functional-matrix.md) | Transversal | Matriz funcional por feature × plano |

## Convenções dos arquivos

Cada arquivo tem frontmatter:

```yaml
---
title: <nome do domínio>
version: <número>
status: <draft | stable | deprecated>
type: requirement
domain: <identity | billing | institutional | commercial | content | assembly | cross-domain | talent-bank | admin>
tags: [...]
---
```

Campos:
- `version` — incrementa a cada mudança material
- `status` — `draft` (em debate), `stable` (acordado), `deprecated` (migrado ou revisado)
- `domain` — bounded context Go correspondente (bate com `internal/modules/{domain}/`)

## Como adicionar um novo requisito

1. Identificar o bounded context (`domain`)
2. Adicionar no arquivo correspondente (ex: novo req de billing → `billing.md`)
3. Usar numeração estável: `Req XX.Y` — cada domínio tem faixa exclusiva de número (ver `product-overview.md`)
4. Atualizar `version` no frontmatter
5. Se é requisito que cruza domínios, adicionar em `cross-domain.md` com links para os domínios afetados
6. Code review de qualquer task que implementa o req confirma que spec existe e está `stable`

## Invariantes que valem para todos os Reqs (regras de ouro)

Definidas em `product-overview.md`. Resumo:

1. **Separação de identidade**: nunca misture User, Profile e Subscription
2. **Imutabilidade**: Timeline, negócio fechado, votos homologados — nunca editados
3. **Connect Me ≠ Chat**: formulário unidirecional
4. **Visibilidade de vídeos de empresa**: blindada a moradores, exceto em votação
5. **Métricas privadas**: likes/comentários visíveis apenas ao dono
6. **Backend como verdade única**: toda regra de negócio na API; frontend é UI
7. **Trava trimestral**: vídeos institucionais + vídeo-currículo
8. **1 dispositivo por conta**: fingerprint + IP
9. **Tenant isolation**: cross-tenant apenas via grants explícitos
10. **LGPD by design**: log obrigatório de revelação de dados pessoais
