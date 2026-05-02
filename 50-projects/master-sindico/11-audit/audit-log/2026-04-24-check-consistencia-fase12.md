---
title: Check consistência cross-vault — pós Fase 11+12 (2026-04-24)
type: audit
tags:
  - master-sindico
  - consistency-check
  - qa
  - fase-12
project: master-sindico
date: '2026-04-24'
created: '2026-04-24'
---
# Check consistência cross-vault — pós Fase 11+12

## Veredito

**RESSALVAS** — 12 decisões D-094..D-105 presentes em `STATE.md` e majoritariamente refletidas. Estrutura (arquivos, aggregates, ADRs, invariants) OK. Algumas atualizações de referências cruzadas **stale** (Q40 como pendente em ASM-PAUTA/_moc, sub-produto 05 sem link para ASM-AVAL-ELEICAO, ui-catalog onboarding com resíduo N1/N2/N3, ADR-0038 fora do adr-index, 1 wikilink quebrado).

Nenhum problema bloqueia release. Limpeza incremental recomendada.

## Checagens (A–J)

### A. N1/N2/N3 abolidos (D-096)

Resultado: **3 ocorrências canônicas vivas** em ui-catalog (esperado 0).

- `03-subprojects/web/ui-catalog/onboarding/ONB-S2.md:76`
- `03-subprojects/web/ui-catalog/onboarding/ONB-SFIN.md:34`
- `03-subprojects/web/ui-catalog/onboarding/ONB-SFIN.md:58`

Observação: dezenas de outras menções em vault são **explicitamente marcadas como legado/abolidas** (backend/README, personas, business-model, sub-produtos) — essas estão corretas. Apenas as 3 acima aparecem como uso ativo/canônico sem etiqueta de obsolescência.

### B. Score duplo (D-103)

- [OK] `01-domain/invariants.md:289` INV-GOV-001 existe
- [OK] `01-domain/invariants.md:308` INV-GOV-002 existe
- [OK] `03-subprojects/web/ui-catalog/compliance/C10.md:44` tem "Q25 RESOLVIDA 2026-04-24 via D-103"
- [OK] `03-subprojects/web/ui-catalog/sindico/S32-score-governanca.md` existe
- [RESSALVA] Menções a "1 score unificado 0-100" ainda aparecem em S6.md/S28.md/S27.md/_moc.md como "canônico (D-061)" — tecnicamente não viola D-103 porque referenciam decisão antiga, mas são **stale** pós-D-103.
- [OK] `04-requirements/functional/cross-domain.md:496` "Req 33 atualizado — Score duplo do síndico (via D-103)"

### C. Avaliação escondida eleição (D-104)

- [OK] `03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO.md` existe
- [OK] `03-subprojects/mobile/ui-catalog/assembly/ASM-AVAL-ELEICAO.md` existe
- [OK] `01-domain/invariants.md:328` INV-ASM-023 existe
- [OK] `04-requirements/functional/assembly.md:531–557` Req ASM-ELE-AVAL
- [FALHA] `00-product/sub-produtos/05-assembleia-live.md` **NÃO** menciona ASM-AVAL-ELEICAO nem D-104. Há apenas §17 "Eleição gera avaliação (D-063)" desatualizado.
- [OK parcial] ASM-AVAL-ELEICAO.md (web+mobile) descreve explicitamente bloqueio (NavGuard, redirect, "bloqueia tela de voto"). Q40 coberta na spec UX. `ASM-VOTO` não cita bloqueio por avaliação; fluxo inverso (AVAL→VOTO) está na AVAL.

### D. IValidatable Opção B (D-105)

- [OK] `01-domain/patterns/validatable-interface.md` existe
- [OK] `01-domain/patterns/_moc.md` existe
- [OK] `02-architecture/adr/0038-validatable-interface-cross-bc.md` existe
- [OK] 3 stubs: `ServiceRecord.md`, `TechnicalActivity.md`, `Adjustment.md`
- [OK] `01-domain/aggregates/_moc.md:44–46` referencia os 3 stubs com rótulo `(stub)`
- [OK] `01-domain/aggregates/ValidationItem.md` **não existe** (correto — design D-105)
- [FALHA] `12-docs/adr-index.md` **NÃO** referencia ADR-0038. Tabela pula de 0037 direto para "Links"; só ADR-0037 listado na seção Fase 11.

### E. Soft_delete universal (D-101)

- [OK] `02-architecture/adr/0037-soft-delete-universal-pseudonymize.md` existe
- [OK] `08-security/lgpd.md:201` seção 4.4.1 soft_delete universal
- [OK] `AUDIT.md:61` A-DC-SEC-004 🟢 RESOLVIDO
- [OK] `AUDIT.md:257` LGPD-M1-007 🟢 RESOLVIDO

### F. Wikilinks quebrados

Scan em 10 arquivos Fase 12 (128 wikilinks únicos): **1 quebrado**.

- `03-subprojects/web/ui-catalog/sindico/S32-score-governanca.md` → `[[../../../mobile/ui-catalog/sindico/S-PERFIL]]`. Diretório `mobile/ui-catalog/sindico/` contém S1/S2/S6/S7 mas não S-PERFIL.

Observação: `2026-04-24-findings-backend-kiro.md` listado no prompt **não existe** (existe `2026-04-23-findings-backend-kiro.md`). Verificado apenas como dado de entrada do prompt, não conta como quebra do vault.

### G. Frontmatter canônico

Spot-check 15 arquivos Fase 11/12: **13 OK, 2 sem `created:`**.

- `11-audit/audit-log/2026-04-24-gap-analysis-consolidado.md` — falta `created`
- `02-architecture/adr/0038-validatable-interface-cross-bc.md` — falta `created`

### H. Bidirecional D-### ↔ artefatos

Spot-check 5 decisões (D-094, D-096, D-101, D-103, D-105):

- D-094: backlinks em `02-connect-me.md`, `matrix-functional.md`, `feature-plan`, `gap-analysis-consolidado` [OK]
- D-096: backlinks em `matrix-functional.md`, `feature-plan`, `gap-analysis-consolidado` [OK]
- D-101: backlinks em `ADR-0037`, `lgpd.md`, `compliance-lgpd.md`, `AUDIT.md`, `adr-index` [OK]
- D-103: backlinks em `invariants.md`, `S32-score-governanca`, `C10`, `ASM-AVAL-ELEICAO`, `cross-domain.md` [OK]
- D-105: backlinks em 3 aggregates, `validatable-interface.md`, `patterns/_moc`, `ADR-0038`, `cross-domain.md` [OK]

### I. Total arquivos vault

- **635 MDs vivos** em `50-projects/master-sindico/` (excluindo `_archive/`)
- **858 MDs totais** (incluindo `_archive/`)
- Esperado: ~540–560. Real: 635 → **+~75 além do esperado** (Fase 12 produziu mais artefatos que previsão, principalmente Fase 11 + 12). Não é problema; apenas discrepância de estimativa.

### J. Zero non-MD no vault

- **0 arquivos não-MD** em `vault/` (excluindo `.obsidian/` e `.smart-env/`). [OK]

## Problemas encontrados

Concretos, listados com path:linha.

1. `03-subprojects/web/ui-catalog/onboarding/ONB-S2.md:76` — "Nível (N1/N2/N3)" sem etiqueta de obsolescência. Pós-D-096 deveria ser `plan_tier`.
2. `03-subprojects/web/ui-catalog/onboarding/ONB-SFIN.md:34` — "badge plano (N1/N2/N3)".
3. `03-subprojects/web/ui-catalog/onboarding/ONB-SFIN.md:58` — "Plano inicial (N1/N2/N3) derivado do parâmetro URL".
4. `03-subprojects/web/ui-catalog/assembleia/ASM-PAUTA.md:71,83` — Q40 marcada como "pendente" e "não mapeada em assembly.md canônico". Hoje resolvida via D-104 + Req ASM-ELE-AVAL.
5. `03-subprojects/web/ui-catalog/assembleia/_moc.md:91` — "Q40 — regra não migrada para requirements canônico" (stale).
6. `00-product/sub-produtos/05-assembleia-live.md` — §17 cita D-063 genérico; não linka ASM-AVAL-ELEICAO nem D-104 nem INV-ASM-023.
7. `00-product/sub-produtos/05-assembleia-live.md:1060–1075` — §17 "Eleição gera avaliação (D-063)" precisa indicar que D-104 ATIVA e mandatoriza a avaliação em eleição.
8. `12-docs/adr-index.md:144` — tabela termina em 0037; ADR-0038 ausente; seção Fase 11 não lista ADR-0038 (que é Fase 12).
9. `03-subprojects/web/ui-catalog/sindico/S32-score-governanca.md:XX` — wikilink `[[../../../mobile/ui-catalog/sindico/S-PERFIL]]` alvo inexistente (nenhum `S-PERFIL.md` em `mobile/ui-catalog/sindico/`).
10. `11-audit/audit-log/2026-04-24-gap-analysis-consolidado.md` — frontmatter sem `created:`.
11. `02-architecture/adr/0038-validatable-interface-cross-bc.md` — frontmatter sem `created:`.
12. `03-subprojects/web/ui-catalog/sindico/S6.md:84`, `S28.md:92`, `S27.md:95`, `_moc.md:97` — "adotar 0-100 (D-061)" / "inconsistência Score Gov 1-10 vs 0-100 (Q25)". Pós-D-103 → são **stale**; canônico agora é score duplo.

## Recomendação

**AJUSTAR** — ressalvas pontuais, não bloqueadoras. Estimativa: 45–60 min de trabalho editorial.

### Fixes sugeridos (ordem de execução)

| # | Fix | Esforço |
|---|-----|---------|
| 1 | Reescrever `ONB-S2.md:76` + `ONB-SFIN.md:34,58` trocando N1/N2/N3 por `plan_tier` (base/plus/pro) com nota D-096 | 10 min |
| 2 | `ASM-PAUTA.md:71,83` + `assembleia/_moc.md:91` → marcar Q40 como RESOLVIDA via D-104; linkar ASM-AVAL-ELEICAO + Req ASM-ELE-AVAL | 10 min |
| 3 | `00-product/sub-produtos/05-assembleia-live.md` §17 → adicionar bloco D-104 + link ASM-AVAL-ELEICAO (web+mobile) + INV-ASM-023 | 10 min |
| 4 | `12-docs/adr-index.md` → adicionar linha ADR-0038 na tabela + bloco na seção Fase 12 (ou nova seção "ADRs Fase 12") | 5 min |
| 5 | `S32-score-governanca.md` → remover wikilink `S-PERFIL` (tela mobile inexistente) ou criar stub `mobile/ui-catalog/sindico/S-PERFIL.md` | 5 min |
| 6 | S6.md/S27.md/S28.md/sindico/_moc.md → substituir "inconsistência Q25 → adotar 0-100 (D-061)" por "Q25 RESOLVIDA via D-103 — score duplo (gov 1-10 + compliance 0-100)" | 10 min |
| 7 | Adicionar `created: 2026-04-24` aos 2 arquivos com FM incompleto | 2 min |

Após esses fixes → **APROVAR** sem ressalvas.

## Links

- [[../../STATE#decisões-fase-12-2026-04-24|STATE Fase 12]]
- [[../../AUDIT]]
- [[2026-04-24-gap-analysis-consolidado]]
- [[../../03-subprojects/feature-plan-2026-04-24]]
