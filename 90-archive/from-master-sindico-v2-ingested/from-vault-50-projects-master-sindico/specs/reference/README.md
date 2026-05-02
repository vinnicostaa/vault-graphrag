---
title: Reference — Índice
version: 1.0
status: stable
---

# Reference — Material histórico e lições aprendidas

Esta pasta contém material que **não é spec ativa** do projeto atual, mas serve como referência para:

- Evitar repetir erros do passado (antipatterns)
- Entender contexto histórico de decisões
- Navegar para material externo (Obsidian, full-stack legacy)

**Regra dura**: nenhum arquivo aqui é fonte de verdade para implementação atual. Se conflita com `requirements/` ou `design/`, o conflitante **vence**; atualize este arquivo para refletir.

## Navegação

| Arquivo | Conteúdo |
|---|---|
| [`antipatterns-to-avoid.md`](antipatterns-to-avoid.md) | **12 antipatterns do legacy com fix Go**. Leitura obrigatória antes de qualquer implementação sensível (auth, quotas, timeline, votação, billing). |
| [`legacy-summary.md`](legacy-summary.md) | Sumário do que o `full-stack/` TypeScript tentou, o que foi abandonado, o que sobreviveu como conhecimento de domínio. |
| [`obsidian-mapping.md`](obsidian-mapping.md) | Mapa de notas no vault Obsidian do João — quais são úteis, quais estão desatualizadas. |

## Quando consultar

| Cenário | Arquivo |
|---|---|
| Implementando ABAC, quotas, tenant isolation | `antipatterns-to-avoid.md` (A1, A2, A8) |
| Implementando profile update por tipo | `antipatterns-to-avoid.md` (A3, A4, A5) |
| Implementando domain events | `antipatterns-to-avoid.md` (A7) |
| Entendendo por que a migração para Go aconteceu | `legacy-summary.md` |
| Procurando visualizações/diagramas | `obsidian-mapping.md` |
| Decidindo se um padrão do legacy deve ser copiado | `antipatterns-to-avoid.md` + validar com `requirements/` atual |

## O que **não** está aqui (e por quê)

- **Código do legacy**: fica em `../../../../full-stack/` — não copiamos para dentro do backend
- **Specs antigas do legacy**: nunca foram migradas integralmente (muita coisa mudou); ver `legacy-summary.md`
- **Notas pessoais do João**: ficam no Obsidian — ver `obsidian-mapping.md`
