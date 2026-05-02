---
title: S32 — Perfil do Síndico (2 scores)
type: ui-screen
tags: [master-sindico, ui-catalog, web, sindico, reputacao, governanca]
project: master-sindico
persona: sindico
category: S
screen_id: S32
sub_produto: governanca-institucional
plan_requirement: any
status: specification
stack: web
milestone: m1
created: 2026-04-24
---

# S32 — Perfil do Síndico (2 scores Governança + Compliance)

## Finalidade

Tela de perfil público do síndico exibindo os **2 scores** canônicos (D-103): Governança 1-10 e Compliance 0-100. Também visível para morador, empresa e parceiro (quando aplicável), com breakdown e histórico por mandato.

## Fonte canônica

- [[../../../../STATE#D-103 — Score duplo no perfil do síndico (fecha Q25)|D-103]]
- [[../../../../01-domain/invariants#INV-GOV-001]]
- [[../../../../01-domain/invariants#INV-GOV-002]]
- [[../../../../04-requirements/functional/cross-domain]] (§ Req 33 atualizado)
- [[../../../../00-product/sub-produtos/09-compliance]] (C10 = Score Compliance)
- [[../../../../00-product/sub-produtos/03-reputacao]] (distingue: reputação Bronze→Diamante é de empresas, não síndicos)

## Persona e ABAC

- **Persona primária**: Síndico (vendo próprio perfil) ou Morador (vendo perfil do síndico do seu condomínio)
- **Persona secundária**: Admin MS, Empresa (vendo perfil síndico em Connect Me / marketplace)
- **Plan tier mínimo**: any (perfil público)
- **ABAC action**: `syndic_profile.read`
- **Restrições**:
  - Morador só vê perfil do(s) síndico(s) do(s) próprio(s) condomínio(s) — tenant match
  - Empresa/Parceiro vê apenas agregados + anos de mandato (não histórico individual)
  - Breakdown detalhado **só o próprio síndico e admin** podem ver

## Fluxo da tela

1. Usuário acessa `/syndics/<id>` (web) ou via Connect Me / seleção condomínio
2. UI carrega:
   - Avatar + nome + mandato atual (condomínio + período)
   - **2 cards de score lado a lado**:
     - Card Governança: ScoreGauge 1-10 + label "Score Governança" + tooltip "Qualidade de gestão — ver breakdown"
     - Card Compliance: ScoreGauge 0-100 + label "Score Compliance" + tooltip "Documentação regulatória em dia — ver C10"
   - Bio + vídeo institucional (apenas plan Plus/Pro)
   - 15 marcadores autodeclarados de governança (badges)
   - Mandatos anteriores (se houver) com scores históricos
3. Clique em card de score → drill-down (tela S32-DETAIL) com breakdown da fórmula

## Componentes

- `ScoreGauge` (novo) — ver [[../../design-system]] seção "Fase 12"
  - Variants: `scale=1-10` (governança) / `scale=0-100` (compliance)
  - Props: `value`, `scale`, `label`, `trend?`
  - Visual: círculo semicircular colorido (vermelho/amarelo/verde) com número central
- `ScoreHistory` (novo) — gráfico linha por mandato (só drill-down)
- `ScoreBreakdown` (novo) — lista fatores ponderados com barra progresso
- `MarkerBadge` (novo/existente) — 15 marcadores de governança
- `MandateCard` (novo) — card de mandato histórico com período + scores finais
- `BioViewer` + `VideoPlayer` (existentes)

## Estados

- **Loading**: skeleton 2 gauges + skeleton bio
- **Empty (síndico novato)**: valor default Governança 5.0 + "Score em formação — 3 avaliações pendentes"; Compliance 100 (início, ainda sem prazos vencidos)
- **Error**:
  - 403 (morador fora do tenant): redirect para home com banner "Acesso não autorizado"
  - 404: banner "Síndico não encontrado"
- **Success**: scores renderizados com animação de progressão

## Integração com backend

| Ação UI | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Carregar perfil completo | `/api/v1/identity/syndics/:id/profile` | GET | - | `{user, mandate, markers[], bio, video_url}` |
| Carregar scores combinados | `/api/v1/identity/syndics/:id/scores` | GET | - | `{governance: {score: 8.4, breakdown}, compliance: {score: 78, breakdown}, updated_at}` |
| Carregar histórico (drill-down) | `/api/v1/identity/syndics/:id/scores/history` | GET | `?since_mandate=X` | `[{mandate_id, start, end, governance, compliance}]` |

Endpoints marcados **a formalizar** em [[../../../backend/requirements/identity]] no Sprint 10.

## Regras de negócio críticas

1. **2 scores INDEPENDENTES**: um síndico pode ter Governança 9.0 e Compliance 45 (boa gestão, documentação atrasada). UX precisa deixar claro.
2. **Governança 1-10 arredondado a 1 decimal** (9.4, não 9.37); Compliance é inteiro 0-100.
3. **Cache Redis TTL 1h**: scores não são tempo-real; aceitar staleness.
4. **Gate ABAC**: `compliance_score < 60` bloqueia ações críticas — mostrar banner **nesta tela** "⚠️ Regularize sua conformidade" com CTA para C1.
5. **Privacy**: morador NUNCA vê breakdown individual (lista de avaliações específicas). Só valor final + período.
6. **Histórico preservado**: mandatos encerrados mantêm scores finais imutáveis (regra ADR-0033 timeline append-only).
7. **Distinção explícita**: tooltip/link para [[../../../../00-product/sub-produtos/03-reputacao]] diferenciando "estes scores são do síndico" vs "reputação Bronze→Diamante é de empresas".

## Ligações

- **Tela origem**: S2 (Seletor Condomínios — card síndico), M-PERFIL-SINDICO (morador vê), Connect Me (quando empresa inspeciona síndico), S-HOME card "Meu perfil"
- **Tela destino**: S32-DETAIL (drill-down breakdown), C10 (score compliance canônico), S29 (declaração anual — se Compliance baixo)
- **Sub-produto**: [[../../../../00-product/sub-produtos/01-governanca-institucional]]
- **Backend**: [[../../../backend/requirements/identity]]
- **Mobile equivalente**: [[../../../mobile/ui-catalog/sindico/_moc|mobile sindico (S-PERFIL a criar)]] *(a criar)*
- **Compliance tela relacionada**: [[../compliance/C10]]

## Gaps/ressalvas

- **Fórmula exata** de `governance_score` (pesos dos 5 componentes em INV-GOV-001) a confirmar com produto — stub 40/25/15/10/10
- **Valor default** para síndico recém-empossado (interpretei como 5.0 Governança + 100 Compliance); confirmar
- **Trend indicator** (seta ↑↓) no ScoreGauge — implementação M2 (calcula delta vs mandato anterior)
- **Compartilhar publicamente** (LinkedIn-style "share my score"): backlog M3 — LGPD implications
- **Disputa de score**: morador pode contestar avaliação? Mecanismo de revisão? Não documentado

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../../../../01-domain/invariants#INV-GOV-001]]
- [[../../../../01-domain/invariants#INV-GOV-002]]
- [[../../../../STATE#D-103]]
- [[../compliance/C10]]
- [[../../../../00-product/sub-produtos/03-reputacao]]
- [[../../../../00-product/sub-produtos/09-compliance]]
