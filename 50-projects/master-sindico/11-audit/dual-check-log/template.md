---
title: Template — Dual-Check Log
type: template
tags: [template, dual-check, master-sindico, v2]
source: 06-execution/playbooks/dual-check.md
created: 2026-04-23
updated: 2026-04-23
---

# Template — Dual-Check Log

Copiar este arquivo pra `YYYY-MM-DD-<slug>.md` (ex: `2026-05-01-topologia-multitenant.md`).

---

```markdown
---
title: Dual-Check — <título curto>
type: audit
tags: [dual-check, <tópico>, master-sindico, v2]
decision_ref: D-### (em STATE) ou — (se não gerou decisão)
adr_ref: ADR-#### (em 02-architecture/adr) ou — 
created: YYYY-MM-DD
propositor: <agente ou humano>
revisor: <agente/humano em contexto limpo>
status: em-andamento | consolidado | pendente-nova-rodada
---

# Dual-Check — <título>

## Contexto

<1-3 parágrafos: o quê tá em jogo, por que precisa decidir, o quê depende disso>

## Proposta do propositor

### Opções consideradas

1. **<Opção A>** — <descrição>
   - Prós: <lista>
   - Contras: <lista>
2. **<Opção B>** — <descrição>
   - Prós: <lista>
   - Contras: <lista>
3. **<Opção C>** — <descrição>
   - Prós: <lista>
   - Contras: <lista>

### Opção escolhida

**<Opção X>** — <justificativa>

### Fontes consultadas

- <URL + data + trecho relevante se aplicável>
- [[../../13-research/<vertical>/_destilado]] (se research rodou)
- [[../../90-ingested/<arquivo>]] (se legado tem referência)
- Doc oficial: <link>
- Engineering blog: <link> (big tech X, data Y)

### Impacto em invariantes

- I-### afetado? <sim/não — qual>
- Alinha com [[../../STEERING]]? <sim/não — qual regra>
- Afeta threat-model ([[../../08-security/threat-model]])? <sim/não>
- Afeta SLO ([[../../02-architecture/observability]])? <sim/não>

---

## Revisão do revisor (contexto limpo)

### Veredito

- [ ] **Concordo** (com caveats abaixo)
- [ ] **Discordo** (proposta alternativa abaixo)
- [ ] **Dúvidas** (lista pra propositor esclarecer)

### Justificativa

<parágrafo>

### Caveats / dúvidas / proposta alternativa

1. <item>
2. <item>

### Fontes adicionais

<se revisor trouxe fontes que propositor não considerou>

---

## Consolidação

<após ida e volta, decisão final>

### Decisão final

**<Opção X>** — <resumo>

### Action items

- [ ] Registrar D-### em [[../../STATE]]
- [ ] (Se impacto arquitetural) Criar ADR-#### em [[../../02-architecture/adr/]]
- [ ] Atualizar [[../../04-requirements/]] se afeta requirement
- [ ] Atualizar [[../../02-architecture/patterns]] se afeta pattern
- [ ] Fechar finding A-### se dual-check foi pra fechamento de finding
- [ ] Comunicar ao time em #dev ou #product

---

## Links

- [[_moc]]
- [[template]]
- [[../AUDIT]]
- [[../../STATE]]
- [[../../06-execution/playbooks/dual-check]]
```

---

## Notas de uso

- Use nome do arquivo `YYYY-MM-DD-<slug>`. Slug curto, kebab-case.
- Contexto limpo do revisor: **nova sessão sem histórico**. Não compartilhar rascunho; revisor lê só a proposta + fontes.
- Se discordância forte: registrar os dois lados em STATE (D-### com rationale dual).
- Status progride: `em-andamento` → `consolidado` (ou `pendente-nova-rodada`).

## Links

- [[_moc]]
- [[../AUDIT]]
- [[../../06-execution/playbooks/dual-check]]
