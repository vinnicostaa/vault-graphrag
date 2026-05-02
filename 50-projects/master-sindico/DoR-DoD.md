---
title: DoR-DoD — Master Síndico v2
type: guide
tags: [dor, dod, ready, done, master-sindico, v2]
created: 2026-04-23
updated: 2026-04-23
---

# DoR-DoD — Master Síndico v2

Definition of Ready (DoR) e Definition of Done (DoD) aplicados à **remontagem arquitetural** (specs/domínio/DDD) — não implementação de código.

---

## Definition of Ready (DoR) — antes de começar uma spec

Um artefato da v2 só entra em produção (ou seja, só é escrito com intenção canônica, não stub) quando:

1. **Contexto claro**: objetivo da spec em 1-3 frases; sub-produto + bounded context identificados.
2. **Fontes consultadas**: research-loop feito — vault/10-knowledge, vault/50-projects/master-sindico (legado), big-techs em 13-research, doc oficial de providers relevantes.
3. **Decisões dependentes identificadas**: D-### pendentes que bloqueiam a spec registrados em STATE e com plano de resolução (research-loop + dual-check).
4. **Personas e jornadas**: qual persona usa + qual jornada é afetada + qual plano/quota aplica.
5. **Invariantes enumerados**: o que nunca pode ser violado (ex: Σ fração ideal ≤ 100%, vote único, timeline imutável).
6. **Risco de segurança identificado**: LGPD, tenant isolation, ABAC, PII em logs — aplicável?
7. **Impacto em M1/M2/M3**: spec afeta qual marco?
8. **Critério de sucesso**: como saberemos que a spec está boa? (teste previsto, review, dual-check).

---

## Definition of Done (DoD) — spec pronta pra mergear / marcar como completa

Um artefato da v2 está pronto quando:

### Conteúdo

- [ ] **Frontmatter canônico** (title, type, tags, source se destilado, created, updated).
- [ ] **Corpo pt-BR**, identifiers EN.
- [ ] **kebab-case.md** nome do arquivo; profundidade ≤ 3 níveis.
- [ ] **≥ 2 `[[links]]`** no corpo (MOCs, referências cruzadas).
- [ ] **Fontes citadas** — caminho absoluto ou URL + data.
- [ ] **Sem placeholder `⚠️ PENDÊNCIA`** sem dono + data alvo.
- [ ] **Sem alucinação** — toda afirmação técnica tem fonte linkada.

### Estrutura

- [ ] `_moc.md` da pasta foi atualizado (novo arquivo listado com 1-liner).
- [ ] Se é requirement: tem ID estável (GR-###, IDN-###, BIL-###, etc.) + critério de aceite.
- [ ] Se é ADR: tem número (ADR-####), status, contexto, decisão, consequências, alternativas rejeitadas.
- [ ] Se é aggregate: tem entidades privadas documentadas + invariants + factories + repository interface.
- [ ] Se é domain event: tem publisher + payload schema + subscribers + side-effects.
- [ ] Se é state machine: tem todos os estados + transições + gatilhos + guards.

### Qualidade cruzada (multi-papel)

- [ ] **Arquiteto de soluções**: decisão justificada vs alternativas, trade-offs explicitados, impacto SLO identificado.
- [ ] **Full-stack senior**: contratos claros (OpenAPI/schema/evento), testabilidade garantida.
- [ ] **QA**: critério de aceite ligável a teste; edge cases + jornada alternativa mapeados.
- [ ] **Pentester**: threat-model STRIDE passado (pelo menos nos artefatos de BCs sensíveis: identity, billing, commercial, assembly).
- [ ] **Arquiteto BeyondCorp**: posição zero-trust, ABAC, tenant isolation, PKCE, LGPD verificados quando aplicável.

### Governança

- [ ] Decisões novas registradas em [[STATE]] como D-###.
- [ ] Findings descobertos em [[AUDIT]] como A-###.
- [ ] Se dual-check foi executado: log em [[11-audit/dual-check-log/_moc]].
- [ ] [[SESSION_CHARTER]] reflete o progresso.
- [ ] Se material veio do legado: entrada em [[_ingestion-log]] e [[90-ingested/INDEX]].

---

## Gates que bloqueiam a migração (Fase 6 do plano)

Em adição aos critérios em `/home/vinni/.claude/plans/squishy-drifting-avalanche.md#verification`, nenhum dos seguintes pode estar aberto:

- 🔴 em [[AUDIT]] (qualquer).
- D-### abertos que bloqueiam M1 do produto.
- Bounded context sem: bounded-contexts.md + functional/*.md + c4-components.md + threat-model.
- Sub-produto sem: portfolio-de-produtos.md entry + personas-alvo + BC(s) mapeado(s).
- Provider externo sem: fluxos outbound + webhooks inbound + crons + retry policy + Saga-vs-UoW documentados.
- Persona sem: jornada feliz + alternativa + cadastro vs perfil mapeado.

---

## Referências

- Herança: [[../vault/50-projects/master-sindico/DoR-DoD]] (legado vivo).
- Multi-papel checklist: [[CLAUDE]] §4.
- Fluxo canônico: [[CLAUDE]] §5.
