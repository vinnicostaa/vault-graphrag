---
title: MOC — Jornadas (UI specs por persona)
type: moc
tags: [moc, master-sindico, ui-catalog, jornadas, web]
created: 2026-04-25
updated: 2026-04-25
---

# MOC — Jornadas (UI specs por persona)

> Catálogo de **fluxos de tela e UX** absorvidos a partir de 9 PDFs do cliente (Fase D do plano de consolidação `_chaos/`). Cada arquivo descreve **telas numeradas** (S1-S31, M1-M15, E1-E16, MK1-MK8, etc.), com app alvo, persona, rota, propósito, estados, ações e cross-links com Reqs canônicos.
>
> **Princípio do João (2026-04-25)**: *"os documentos tambem diziam sobre fluxos de tela, isso tem que ir para os fronts do projeto, é de suma importancia que esses fluxos sejam do front e nao do back, ja que isso é relavante para o front, o backend cuida de prover as features para essas telas"*
>
> Regras de negócio canônicas vivem em `04-requirements/functional/<bc>.md`. Estes arquivos só **referenciam** Reqs/aggregates, nunca duplicam regras.

## Índice das 8 jornadas

### Por persona principal

- [[sindico|sindico]] — **31 telas** (S1-S31) · jornada do síndico (governança, plano diretor, timeline, validações, compliance)
- [[morador|morador]] — **15 telas** (M1-M15) · jornada do morador (painel, condomínio, vínculo, avaliação)
- [[empresa|empresa]] — **16 telas** (E1-E16) · jornada da empresa prestadora (oportunidades, execução, marketing link)
- [[agencia-marketing|agencia-marketing]] — **8 telas** (MK1-MK8) · jornada da agência de marketing (apenas conteúdo institucional)
- [[parceiro-vizinhanca|parceiro-vizinhanca]] — **13 telas** (VZ1-VZ6 + VM1-VM7) · jornadas do parceiro local **e** do morador no módulo Vizinhança

### Cross-persona (fluxos compartilhados)

- [[onboarding|onboarding]] — **17 telas** (4 perfis × ~4-6 telas) · onboarding sequencial (Boas-vindas → Identificação → identidade → vínculo/atuação → aceites)
- [[curriculo-cadastro|curriculo-cadastro]] — **11 telas** (M-CV-1 a M-CV-11) · morador cadastra seu currículo (Banco de Talentos)
- [[curriculo-empresa-view|curriculo-empresa-view]] — **N telas** (E-CV-1 a E-CV-N) · empresa visualiza/exporta/favorita candidatos

## Fonte e tradução aplicada

- **Origem**: 9 PDFs entregues pelo cliente (datas internas Mar-2026). Fontes preservadas em `60-sources/master-sindico-research/client-material/pdfs/`:
  - `2026-03-09-jornada-sindico-completa.pdf`
  - `2026-03-09-jornada-morador.pdf`
  - `2026-03-09-jornada-empresa.pdf`
  - `2026-03-09-jornada-agencia-marketing.pdf`
  - `2026-03-10-jornada-parceiro-vizinhanca.pdf`
  - `2026-03-09-estrutura-cadastro-curriculo.pdf`
  - `2026-03-09-empresa-ve-curriculo-morador.pdf`
  - `2026-03-10-modulo-vizinhanca-morador.pdf`
  - `2026-03-09-onboarding-perfis.pdf`
- **Absorvidos em**: 2026-04-25 — Fase D.
- **Tradução obrigatória aplicada**:
  - `N1/N2/N3` → `plan_tier ∈ {trial, base, plus, pro}` (D-081, D-096, ADR-0032)
  - "Morador Pagante" → REMOVIDO (D-126); morador é base gratuito + addon Banco de Talentos opt-in
  - "My Síndico" → "Master Síndico"
  - Mandato → flags em Membership (D-129); não aggregate
  - Admin admin owner → app dedicada `apps/admin` (D-134)
  - Lives broadcast (Mux/Cloudflare) ≠ Assembleia (Livekit) — D-133
  - 2 tenant kinds (D-126); user/partner são EntityID

## Convenção dos arquivos

Cada jornada segue o template:

```yaml
---
title: Jornada — <Persona>
type: ui-catalog-journey
tags: [ui-catalog, jornada, master-sindico, <persona>]
persona: <persona>
total_screens: <N>
source: 60-sources/...
absorbed_in: 2026-04-25 — Fase D
status: absorbed
---
```

E o conteúdo segue: **Sumário → Fluxo macro → Telas (cada uma com App / Persona / Rota / Propósito / Estados / Ações / Campos / Regras canônicas referenciadas / Cross-links)**.

## Vizinhos

- [[../_moc|web/ui-catalog/_moc]] — MOC do ui-catalog (Fase B — 27 MDs absorvidos)
- [[../../ui-catalog|web/ui-catalog (catálogo macro)]] — ≥ 141 telas com IDs estáveis
- [[../_components-and-screens|_components-and-screens]] — índice consolidado componentes × spec
- [[../../../04-requirements/functional/_moc|04-requirements/functional/_moc]] — Reqs canônicos referenciados
- [[../../../00-product/personas|00-product/personas]] — 5+ personas
- [[../../../STATE|STATE]] — D-126, D-129, D-133, D-134, D-099 etc.
