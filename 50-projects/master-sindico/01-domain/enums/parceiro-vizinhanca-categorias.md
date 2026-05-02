---
title: Parceiro Vizinhança — 38 categorias B2C (Clube de Benefícios)
type: enum
tags: [master-sindico, domain, enums, vizinhanca, parceiros, categorias, b2c]
project: master-sindico
origin: _chaos/requirements(6).md R27 + R-_chaos Partner Categories (2026-04-13) — absorvido Fase C 2026-04-25
created: 2026-04-25
updated: 2026-04-25
---

# Parceiro Vizinhança — 38 categorias B2C

Fonte canônica de **categorias de Parceiros da Vizinhança** (estabelecimentos de comércio local B2C que oferecem benefícios/cupons aos moradores).

> **Distinção importante**:
> - Esta lista (38 categorias B2C) é dos **Parceiros Vizinhança** — comércio local que se cadastra para oferecer cupons aos moradores via Clube de Benefícios.
> - A lista de [[vizinhanca-categorias-subcategorias]] (32 cat × 202 subcat) é dos **Fornecedores B2B** — empresas que prestam serviços a condomínios via Connect Me.
> - **NÃO confundir**: parceiros Vizinhança ≠ empresas Connect Me.

**Totais**: 38 categorias agrupadas em 12 grupos.

---

## 12 grupos × 38 categorias

### Alimentação (9)
```
padaria, restaurante, lanchonete, cafeteria, pizzaria,
hamburgueria, sorveteria, doceria, acai
```

### Mercado (3)
```
supermercado, hortifruti, adega
```

### Saúde (1)
```
farmacia
```

### Pet (3)
```
pet_shop, clinica_veterinaria, banho_tosa
```

### Academia (4)
```
academia, crossfit, pilates, yoga
```

### Estética (3)
```
salao_beleza, barbearia, spa
```

### Serviços Domésticos (3)
```
lavanderia, costureira, sapateiro
```

### Profissionais (5)
```
eletricista, encanador, chaveiro, marceneiro, pintor
```

### Assistência Técnica (2)
```
assistencia_celular, assistencia_computadores
```

### Automotivo (2)
```
oficina, lava_jato
```

### Cursos (2)
```
cursos_idiomas, reforco_escolar
```

### Outros (1)
```
outros_servicos
```

---

## Implementação

- Enum Postgres `partner_category` (38 valores).
- Seed migration alinhado com R27 do `_chaos/requirements(6).md`.
- Catalog table opcional `partner_categories {category, group, label_pt_br}` se i18n M2+ exigir.

## Cross-references

- [[../../04-requirements/functional/commercial#COM-047 — Vizinhança taxonomia]] (Req canônico).
- [[../../04-requirements/functional/commercial#COM-030 — Vizinhança (Parceiro): perfil público]].
- [[../../00-product/sub-produtos/08-vizinhanca]] (sub-produto).
- [[vizinhanca-categorias-subcategorias]] — categorias B2B (Fornecedores Connect Me — distinto).

## Fonte

`_chaos/requirements(6).md` Req 27 + R-_chaos Partner Categories enum (linhas 3674-3737), absorvido em Fase C 2026-04-25.
