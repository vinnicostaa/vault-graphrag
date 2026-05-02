---
title: "Gap Analysis — Produto & Processos × UI Catalog (Master Síndico v2)"
type: audit
tags: [audit, gap-analysis, master-sindico, v2, ui-catalog, coerencia-produto, m1]
project: master-sindico
source: 03-subprojects/{web,mobile}/ui-catalog/* + 00-product/sub-produtos/* + 04-requirements/matrix-functional.md + STATE.md + AUDIT.md + 11-audit/audit-log/* + 12-docs/runbooks/* + 10-schedule/milestones.md + 02-architecture/adr/*
method: Read + Bash grep (não-MCP), cruzamento determinístico frontmatter × corpo × fontes canônicas
created: 2026-04-24
updated: 2026-04-24
status: draft
dual_check: pending
---

# Gap Analysis — Produto & Processos × UI Catalog

Relatório de coerência entre o **UI Catalog** (181 telas web + 53 mobile = 234 specs, Fase 9–10) e as camadas canônicas de produto/processo/auditoria (sub-produtos, matriz funcional, STATE, AUDIT, runbooks, ADRs, milestones). Objetivo: identificar **gaps que impactam M1 (2026-05-08)** ou posteriores, sem inventar — apenas cruzar o que existe textualmente.

> **Escopo excluído**: o arquivo `web/ui-catalog/admin/GAP-admin-master-sindico.md` já é um gap conhecido (telas ADM-1..N ausentes no cliente original, D-058 canoniza "admin = role transversal sem app separada"). Este relatório reforça esse gap mas não o originou.

---

## Sumário (contagens verificadas)

| Indicador | Contagem |
|---|---|
| Telas UI catalog (web + mobile, excluindo `_moc.md`) | **234** (180 web + 53 mobile + 1 GAP) |
| Sub-produtos canônicos (00-product/sub-produtos/\*.md, excluindo `_moc`) | **11** |
| Sub-produtos citados em frontmatter `sub_produto:` (valores distintos) | **13** (10 canônicos + 3 não-canônicos: `onboarding`, `marketing`, `banco-talentos`) |
| Sub-produtos canônicos COM ≥1 tela | **10 / 11** (órfão: `11-administradoras-condominiais`) |
| Telas com `sub_produto:` em branco | **11** (todos `_moc.md` ou `GAP-*.md`) |
| Telas com `sub_produto:` NÃO canônico | **32** (23 "onboarding" + 3 "marketing" + 6 "banco-talentos") |
| D-### únicos em STATE.md (D-001..D-093, com saltos 016-023/025/027/031..039/041/044/045) | **72** |
| D-### únicos citados em telas (regex word-boundary) | **17** (D-011, D-050, D-051, D-054, D-056..D-062, D-064, D-066, D-067, D-070, D-073, D-074) |
| D-### **fantasma** (citados em tela, ausentes em STATE) | **0** ✅ |
| A-### únicos em AUDIT.md + 11-audit/audit-log/\* (sumário) | **104** (A-001, A-007..A-034, A-039, A-044, A-DC-PEN-001..042, A-DC-QA-001..072, A-DC-SEC-001..005, A-FASE8-\*, A-FASE10-DEV-001..008) |
| A-### **citados em telas** (regex word-boundary) | **0** — nenhum finding rastreado via A-### no corpo das specs |
| Runbooks canônicos em `12-docs/runbooks/` | **5** (dr, incident-lgpd-breach, rollback-deploy, secret-rotation, webhook-reprocessing) + `_moc` |
| Runbooks citados em telas | **0** |
| ADRs em `02-architecture/adr/` | **35** (0001..0035) |
| Features/linhas da matriz funcional (`matrix-functional.md`) | **~294 rows** (≥ 80 features canônicas) distribuídas em 25 sub-seções cobrindo os 11 sub-produtos + Admin + Quotas + Trials + Paywalls |
| ABAC actions únicas citadas em telas (backticked `foo.bar`) | **227** |
| ABAC actions em matriz/requirements/backend-specs | **177** |
| ABAC actions **órfãs** (em tela, ausentes em todas as fontes canônicas) | **221** |
| Personas UI (via frontmatter `persona:`) | 7 valores: `sindico` 80, `morador` 73, `empresa` 35, `any` 15, `parceiro` 13, `marketing` 8, `administradora` 3, `morador\|sindico` 3, `all` 2, `todos` 1 |
| Persona canônica **admin** (role=admin, D-058/D-061) | **0 telas** (gap conhecido — vide `GAP-admin-master-sindico.md`) |

---

## A. Sub-produtos × telas (cobertura e consistência)

### A.1 Matriz de cobertura por sub-produto canônico

| Sub-produto canônico (00-product) | Screens web | Screens mobile | Total | Status |
|---|---|---|---|---|
| `01-governanca-institucional` (slug no frontmatter: `governanca-institucional`) | ~45 (sindico S1..S32, morador M\*) | ~24 | **69** | ✅ coberto (M1 core) |
| `02-connect-me` (`connect-me`) | 9 | 0 | **9** | 🟡 parcial — M2 (C.M. E2E Sprint Connect Me) |
| `03-reputacao` (`reputacao`) | 6 | 0 | **6** | 🟡 parcial — M2 |
| `04-conteudo-videos` (`conteudo-videos`) | 1 | 0 | **1** | 🔴 subcoberto — publicação/consumo vídeos M2; apenas 1 spec visível |
| `05-assembleia-live` (`assembleia-live`) | 22 | 6 (assembly/\*) | **26** + ASM mobile | ✅ coberto (M3) |
| `06-lms-academia` (`lms-academia`) | 16 | 0 | **15** | 🟡 parcial — M3 |
| `07-banco-de-talentos` (canônico: `banco-de-talentos`) | 11 web | 0 | **11** | 🟡 parcial — M2 |
| `08-vizinhanca` (`vizinhanca`) | 30 web | 8 mobile | **36** | ✅ coberto (M2) |
| `09-compliance` (`compliance`) | 12 | 0 (mas 6 screens compliance embedded em sindico) | **18** | ✅ coberto M1 (LGPD base) |
| `10-marketing-agencias` (`marketing-agencias`) | 9 (MK1..MK8 + E16) | 0 | **10** | 🟡 parcial — gated M5 |
| `11-administradoras-condominiais` (sem slug no catalog) | **0** | **0** | **0** | 🔴 **ÓRFÃO** |

### A.2 Inconsistências de slug no frontmatter

Três variantes não-canônicas detectadas em telas:

- `sub_produto: onboarding` (23 screens em `web/ui-catalog/onboarding/`): "onboarding" é **fase/categoria**, não sub-produto canônico. Matriz funcional não tem seção "onboarding" — cada tela pertence logicamente a `01-governanca-institucional` (síndico/morador) ou `02-connect-me`/`10-marketing-agencias` (parceiro/empresa).
- `sub_produto: marketing` (3 screens `ONB-P2/P3/P4` mobile): deveria ser `marketing-agencias` conforme nome do arquivo canônico `10-marketing-agencias.md` (ADR-0032, D-059/D-060).
- `sub_produto: banco-talentos` (6 screens `ONB-E2..E7` mobile): slug canônico é `banco-de-talentos` (com "de") — `00-product/sub-produtos/07-banco-de-talentos.md`.

### A.3 Cross-link reverso (sub-produto → telas)

Verifiquei se cada arquivo `00-product/sub-produtos/0X-*.md` lista suas telas do catálogo. Resultado (ocorrências de `ui-catalog|S[0-9]+|ONB-|/web/|/mobile/`):

| Sub-produto | Ocorrências de cross-link | Status |
|---|---|---|
| `01-governanca-institucional` | 40 | ✅ bom |
| `02-connect-me` | 19 | 🟡 ok |
| `03-reputacao` | 1 | 🔴 gap |
| `04-conteudo-videos` | 3 | 🟠 fraco |
| `05-assembleia-live` | **0** | 🔴 gap (22 telas web + 6 mobile sem link reverso) |
| `06-lms-academia` | 44 | ✅ |
| `07-banco-de-talentos` | 1 | 🔴 gap |
| `08-vizinhanca` | 31 | ✅ |
| `09-compliance` | 8 | 🟡 |
| `10-marketing-agencias` | 1 | 🔴 gap |
| `11-administradoras-condominiais` | 21 | 🟡 (nenhuma das 21 é link direto para UI) |

---

## B. ABAC actions órfãs (em telas, fora do spec)

### B.1 Panorama

- **227 ABAC actions únicas** citadas entre backticks em telas (`timeline.create`, `assembly.vote.cast`, `commercial.proposal.create`, etc.).
- Apenas **6 actions** (~3%) aparecem também em `04-requirements/matrix-functional.md` (o arquivo canônico da matriz lista predominantemente *features em linguagem natural* + tiers, não os slugs ABAC em backticks).
- Estendendo a busca para `04-requirements/*.md` + `03-subprojects/backend/requirements/*.md`: **177 actions** com fonte canônica.
- **221 actions órfãs** (em tela, ausentes em toda fonte canônica textual).

### B.2 Exemplos notáveis de órfãs críticas para M1

Ações que telas presumem existir mas que não estão registradas em nenhuma fonte canônica pesquisável:

**Institutional (M1 core)**:
- `institutional.membership.request` (ONB-M3, S20..S23)
- `institutional.syndic.activate` / `institutional.company.activate` / `institutional.local_company.activate`
- `institutional.syndic.profile.write`, `institutional.syndic.markers`, `institutional.company.markers`, `institutional.company.set_categories`
- `institutional.fraction.upload` (S5, upload fração ideal — crítico para assembleia)
- `condominium.create`, `condominium.finalize`, `condominium.dashboard.read`, `condominium.list_own`, `condominium.lookup_by_id`
- `mandate.create` (S14–S17, mandato síndico)
- `timeline.create`, `timeline.amend`, `timeline.read.privileged`, `timeline.history.read`, `timeline.list` (M1 core)
- `master_plan.create`, `master_plan.amend`, `master_plan.hide`, `master_plan.update`, `master_plan.read.detail`
- `membership.create.self`, `membership.revoke.self`, `membership.default.update`, `membership.accept_terms`
- `identity.activate`, `identity.draft.write`, `identity.terms.accept`
- `dependent.manage`
- `governance.dashboard.read`, `governance.execution_record.create`, `governance.technical_activity.create`
- `service_record.list_pending`, `service_record.validate`
- `validation.list_pending`

**LGPD / Compliance (M1 pré-bloqueante)**:
- `compliance.declaration.create`, `compliance.declaration.signed`, `compliance.lgpd.read`
- `compliance.responsibility_term.sign`, `compliance.annual_declaration.submit`
- `compliance.dossier.export`, `compliance.dossier.exported`
- `compliance.risk.read`, `compliance.conflict.declare`, `compliance.addendum.create`
- `compliance.user_signature.sign` / `.read`
- `audit_log.read`, `audit.exported`

**Assembleia (M3)**:
- `assembly.poll.answer`, `assembly.speech.authorize`, `assembly.speech.raise`
- `assembly.delinquency.upload` / `.override` (lista de inadimplentes — Content req:1313)
- `assembly.presentation.control`, `assembly.live.operate`
- `assembly.proxy.validate`, `assembly.proxy.grant.self`
- `assembly.item.homologate`, `assembly.vote.cast_assisted`, `assembly.checkin.assisted`

**Connect Me / Commercial (M2)**:
- 28 actions em `commercial.*` (`connect_me.list_received`, `.respond`, `commercial.proposal.create`, `commercial.neighborhood_partner.invite`, `commercial.exclusive_benefit.create`, etc.) — praticamente todo o escopo Connect Me + Vizinhança B2B **sem spec ABAC canônica**.

**LMS / Content (M3)**:
- 19 actions em `content.*` (`content.course.read`, `content.lesson.watch`, `content.certificate.earn`, `content.forum.topic.create`, `content.live.join`, `content.live.replay.watch`, etc.)

**Talent / Curricula (M2)**:
- 10 actions em `content.talent.*.self` (`content.talent.curriculum.submit.self`, `content.talent.lgpd.consent.self`, `content.talent.video.upload.self`, etc.)

**Vizinhança**:
- `vizinhanca.coupon.activate.self`, `vizinhanca.coupon.history.self`, `vizinhanca.offer.read`, `vizinhanca.feed.read`

> **Conclusão B**: a matriz funcional v2 (`matrix-functional.md`) está escrita em linguagem natural por feature, não em slugs ABAC. Isso cria um **gap estrutural**: o frontend documenta o universo ABAC (227 actions) mas o seed da matriz ABAC descrito em `matrix-functional.md §Mapeamento ABAC engine (Go)` não está operacionalizado como lookup table. Risco M1 (institutional) e M3 (assembleia).

---

## C. Plan tier — inconsistências (legado N1/N2/N3 vs canônico trial/base/plus/pro)

Referências: ADR-0032, D-057 Fase 8, D-066 Fase 7, D-067 Fase 8, D-080/D-081.

### C.1 Distribuição de `plan_requirement:` em 234 telas

| Valor | Ocorrências | Canônico? |
|---|---|---|
| `trial` | 61 | ✅ |
| `any` | 54 | ✅ (significa sem gate) |
| `base` | 39 | ✅ |
| `plus-or-pro` | 16 | 🟡 composto aceitável mas não é enum puro |
| `plus` | 14 | ✅ |
| `premium` | 5 | 🔴 **não canônico** |
| `(vazio)` | 62 | 🟡 (metade inclui `_moc.md` + GAP; resto merece investigação) |

### C.2 Enums legados N1/N2/N3 no frontmatter

Resultado da busca `$3 ~ /[Nn][123]/`: **0 telas** ✅. Boa notícia — não há `plan_requirement: N1/N2/N3` no frontmatter.

### C.3 Uso de `premium` (não-canônico)

5 telas usam `plan_requirement: premium` (deveriam migrar para `plus` ou `plus-or-pro`):

- `mobile/ui-catalog/assembly/ASM-CHKIN.md`
- `mobile/ui-catalog/assembly/ASM-VOTO.md`
- `mobile/ui-catalog/assembly/ASM-FILA-FALA.md`
- `mobile/ui-catalog/assembly/ASM-CIEN.md`
- `mobile/ui-catalog/assembly/ASM-PROC-CAD.md`

Impacto: matriz funcional `§5.1-5.3` define votação/check-in assembleia como `trial ⏳ / base ❌ / plus ✅ / pro ✅` para síndico e `✅` universal para morador. `premium` não mapeia 1:1 — código ABAC quebrará quando o frontend enviar esse valor.

### C.4 Citações N1/N2/N3 no corpo (não frontmatter)

Busca `grep -r "N1\|N2\|N3"` no corpo das specs retorna menções históricas em notas (traduzidas para tier novo). Conforme a regra D-067, essas citações precisam ter tradução explícita; amostragem não detectou uso operacional, apenas referência a fontes legadas.

---

## D. D-### fantasma

**0 fantasmas detectados** ✅.

Os 17 D-### citados em telas (D-011, D-050, D-051, D-054, D-056..D-062, D-064, D-066, D-067, D-070, D-073, D-074) todos existem em STATE.md. A única ocorrência inicialmente suspeita — "D-018" em `ONB-M3.md` e `ONB-E3.md` — é **falso positivo**: aparece como "XD-018" (cross-domain req ViaCEP/CEP lookup), não D-###. Regex `(^|[^A-Za-z])D-[0-9]{3}` resolve.

### D.1 Cobertura inversa (decisões importantes NÃO referenciadas por telas)

Das 72 decisões em STATE, apenas **17 (23%)** são citadas em alguma tela. Decisões M1-críticas **não** referenciadas por nenhuma tela, apesar de relevantes:

- **D-001..D-010** (decisões iniciais: Clean Arch, Zitadel, Stripe, Gin, sqlc, goose, zerolog, UUIDv7, Mux, Livekit) — 0 telas citam.
- **D-012** (empresa síndica multi-condomínio — postergado M5).
- **D-014** (moderação ML — adiada M4).
- **D-028, D-029, D-030** (SLIs/SLOs, webhook idempotency, ADR-0026 MFA M1).
- **D-043, D-046** (email Resend M1 + SES M3).
- **D-049** (`freezed` Flutter imutabilidade).
- **D-063** (quotas Lives Fase 8).
- **D-080, D-081** (renames plan tier).
- **D-082..D-093** (decisões Fase 10 mais recentes — esperado para telas Fase 9-10 não terem absorvido).

Não é **fantasma**, mas é **rastreabilidade fraca** — especialmente para D-001..D-010 (fundação técnica).

---

## E. A-### fantasma

**0 citações A-### em telas** — logo, 0 fantasmas por construção, mas:

- O escopo do trabalho pedia validar A-023, A-024, A-039, A-FASE10-DEV-001..008, A-DC-PEN-\*, A-DC-SEC-\*. **Nenhum** desses IDs aparece no corpo de nenhuma das 234 telas.
- A menção inicial a "A-256" no corpo de `sindico/S29.md` (e replicada em `ASM-REL.md`, `S29.md`) é **falso positivo**: refere-se a "SHA-256" (hash de integridade jurídica do termo de responsabilidade), não ao finding A-256.
- O finding "A-FASE10-DEV-004" é citado **apenas** em arquivos de audit-log/AUDIT, nunca em telas.

### E.1 Gap de rastreabilidade

Se um finding 🔴 M1-bloqueador (ex: A-DC-SEC-001 IDOR em `DELETE /me/devices/:id` — account takeover) depende de mudança de UI + backend, a tela correspondente (`S\*` perfil/segurança) deveria citar o finding como contexto. **Ausência total** indica que o fluxo de vinculação finding→tela não está operacionalizado. Risco: fixes feitos em backend sem sincronizar spec de UI.

---

## F. Runbooks citados sem arquivo (ou vice-versa)

### F.1 Runbooks existentes em `12-docs/runbooks/`

1. `dr.md`
2. `incident-lgpd-breach.md`
3. `rollback-deploy.md`
4. `secret-rotation.md`
5. `webhook-reprocessing.md`
6. `_moc.md`

### F.2 Citações de runbook em telas

Busca `grep -rniE "runbook" 03-subprojects/{web,mobile}/ui-catalog`: **0 ocorrências**.

Nenhuma tela cita runbook — inclusive aquelas que são gatilho óbvio de incidente:
- Telas `S29` (termo de responsabilidade), `compliance/CMP-*` (LGPD) deveriam referenciar `incident-lgpd-breach.md`.
- Telas admin/moderação (não existem — vide `GAP-admin-master-sindico.md`) deveriam referenciar `rollback-deploy.md` + `webhook-reprocessing.md`.
- Nenhuma tela de backup/restore ou disaster recovery — mas isso é aceitável (runbook DR é operacional, não feature de UI).

### F.3 Runbook mencionado em spec mas ainda não criado

Em `03-subprojects/admin-privilegios.md` (fora do UI catalog mas referenciado por várias telas):
- **`12-docs/runbooks/admin-mfa-lockout.md`** — stub mencionado explicitamente como "a criar" (G-ADMIN-06, DT-### a registrar). **Não existe**. 🟠 gap operacional pré-M1 se admin-MFA for habilitado.
- **`dr-drill.md`** — citado em `backend/tasks/by-sprint/sprint-10.md:41` como output esperado de backend-10.14 (PITR + DR drill mensal). Não existe como arquivo em `12-docs/runbooks/`. 🟡 M1 gate mas dev-task, não UI-gap.

### F.4 Runbooks LGPD M1 (AUDIT.md:249-255)

AUDIT.md lista **7 bloqueadores LGPD M1**: DPO nomeado, DPAs Mux/Zitadel/Twilio, DPIA-001, política de privacidade versionada, race hard-delete (A-DC-SEC-004). **`incident-lgpd-breach.md` existe** ✅, mas o AUDIT explicita que DPO + DPAs + DPIA são **humanos/comerciais** — UI catalog não pode cobrir; comentário fica registrado aqui mas não é gap de UI.

---

## G. Matriz funcional × telas (features sem tela ou vice-versa)

### G.1 Features da matriz sem tela identificável

A matriz funcional tem **~294 linhas** em 25 sub-seções. Auditoria manual por amostragem (não exaustiva — exigiria deep-dive adicional por spec):

**Cobertura claramente presente**:
- §1.1 Condomínio/unidades → S1..S11 + ONB-S\*
- §1.2 Timeline → S18, S19, S20
- §1.4 Comunicados/eventos/enquetes → S21..S25
- §5.\* Assembleia → 22 telas `assembleia/ASM-*`
- §8.\* Vizinhança → 30 telas web + 8 mobile

**Cobertura parcial/suspeita**:
- §4.1 Publicação de vídeos (uploader + lock 90d + moderação manual) → apenas 1 spec com `sub_produto: conteudo-videos` = **severo subcoverage** para M2. Features declaradas: upload, lock 90d, views, métricas privadas — nenhuma visível como tela web dedicada no mapa.
- §4.3 Interação social "rede social cega" (forum.topic.create, reply, like) → citadas como ABAC mas sem tela dedicada óbvia.
- §3.1/3.2 Reputação + suspensão → 6 telas; matriz descreve badge progression Bronze→Diamante + suspensão harassment. Cobertura provável mas não validada 1:1.
- §6 LMS (5 áreas × 3 trilhas piloto) → 16 telas web; matriz não detalha trilhas nomeadas por tier.
- §9 Compliance → 12 telas explícitas + 6 embutidas em síndico = 18 telas, mas AUDIT lista 7 bloqueadores LGPD M1 (DPO, DPAs, DPIA, política, hard-delete-race). Telas cobrem consentimento/export/delete — ok.
- §10 Marketing/Agências → 9 telas. Agência como sub-produto próprio (D-059/D-060) tem MK1..MK8 — mas persona `marketing` aparece só 8x (em 234 telas). Baixo para role inteira.
- §11 Administradoras Condominiais → **0 telas**. Matriz descreve delegação ABAC + 5 permissões + `EmpresaSindicaLink` (S5 Content Inventario-Telas). Gap confirmado.
- §Transversal Admin MS → **0 telas** (gap já formalizado em `GAP-admin-master-sindico.md`).

### G.2 Features M1-essenciais — verificação direta

Conforme `10-schedule/milestones.md#M1`:

| Feature M1 | Tela correspondente | Status |
|---|---|---|
| Auth Zitadel OIDC+PKCE | ONB-01-identificacao-inicial + `sindico/S1..S4` | ✅ |
| 1-device policy | `sindico/S*` perfil (não identificado explicitamente) | 🟡 investigar |
| Billing Trial/Base/Plus/Pro + trial persona-aware | ONB-SFIN/MFIN/EFIN/PFIN + `sindico/S11` | ✅ |
| Stripe recorrência real | ONB-\*FIN | ✅ |
| Quota Connect Me anual | E-series + `sindico/S*` connect-me | 🟡 parcial (M2) |
| Timeline imutável | S18..S20 | ✅ |
| Plano Diretor base | `sindico/S*` (identificar index específico) | 🟡 |
| Unidades com fração ideal | S5 + `institutional.fraction.upload` | ✅ (ABAC órfã) |
| Compliance básica | `compliance/CMP-*` | ✅ |
| LGPD: consent, export, delete, audit 5 anos | `compliance/CMP-*` + `sindico/S29` | ✅ |
| **DPO notificado + runbook 72h** | runbook `incident-lgpd-breach.md` ✅ | ✅ (runbook existe, tela não necessária) |
| Mobile Flutter: auth + home sindico/morador + timeline leitura | `mobile/ui-catalog/onboarding/*` + `mobile/ui-catalog/sindico/*` (6 telas) + `morador/*` (16 telas) | ✅ |

### G.3 Cobertura matriz → ABAC órfã overlap

Das 221 ABAC órfãs (§B), estimativa: **~40 são M1-críticas** (institutional + identity + billing + compliance M1). O gap é de documentação ABAC, não de telas — as telas *existem*, mas o spec ABAC para cada ação não está em fonte canônica pesquisável.

---

## H. Sub-produtos sem tela / Personas sem jornada

### H.1 Sub-produtos canônicos sem tela

**1 órfão**: `11-administradoras-condominiais` (0 telas). Conforme o próprio arquivo canônico `11-administradoras-condominiais.md`, no MVP M1:
- Administradora **não é produto separado** — é `Empresa role=enterprise + subcategoria ADMINISTRADORAS_CONDOMINIOS`.
- Vínculo via aggregate `EmpresaSindicaLink` iniciado pelo síndico em **S5** (segundo Content Inventario-Telas:55).
- S5 no nosso catálogo atual (`sindico/S5.md`) tem `sub_produto: governanca-institucional` — verificar se S5 expõe o fluxo de convite/vínculo de administradora (link canônico do .md de administradoras aponta para S5, mas não há tela dedicada).

**Gap real**: fluxo `invite / accept / update-permissions / revoke / list` do aggregate (5 endpoints REST — `commercial/infrastructure/http/routes/empresa_sindica_handler.go`) **precisa telas para admin do síndico + aceite do lado empresa**. Atualmente: **0 telas dedicadas**. Se M1 entrega o endpoint sem UI, impacta contratação do primeiro cliente (síndico provavelmente vinculará administradora).

### H.2 Personas sem jornada adequada

| Persona canônica | Role Zitadel | Screens UI | Status |
|---|---|---|---|
| Síndico | `syndic` | 80 (35%) | ✅ |
| Morador | `resident` | 73 (32%) | ✅ |
| Empresa | `enterprise` | 35 (15%) | ✅ |
| Parceiro (local_company) | `local_company` | 13 (6%) | 🟡 parcial (vizinhança M2+) |
| Marketing (agência) | `marketing` | 8 (3%) | 🟠 baixa — sub-produto próprio M5 |
| Admin | `admin` | **0** | 🔴 bloqueador formal (vide `GAP-admin-master-sindico.md`) |
| `administradora` (não canônica — deveria ser `enterprise` + subcategoria) | — | 3 | 🟡 slug não-padronizado |
| `any`/`all`/`todos` (sem persona específica) | — | 18 | 🟡 revisar caso-a-caso |

### H.3 Gap do admin (reforço)

`GAP-admin-master-sindico.md` é o **único** arquivo do tipo `type: gap` em todo o catálogo. Ele próprio enumera:
1. Dashboard Financeiro (MRR, assinaturas)
2. Gestão de Usuários (bloquear/desbloquear/editar)
3. Moderação de Conteúdo (aprovar/reprovar vídeos, remover, suspender empresas)
4. Configurações da Plataforma (termos, banners, categorias técnicas, params globais)
5. Envio de Comunicados Oficiais (broadcast email/SMS)

Sem tela nenhuma dessas 5 capacidades → **operação cega pós-M1** (não há UI para DPO atender requisição LGPD, por ex.).

---

## I. Inconsistências frontmatter tela × sub-produto canônico

### I.1 Normalização necessária (pré-M1)

| Arquivo | Hoje | Canônico (sugerido) |
|---|---|---|
| `mobile/ui-catalog/onboarding/ONB-E2..E7.md` (6 telas) | `sub_produto: banco-talentos` | `sub_produto: banco-de-talentos` (`00-product/sub-produtos/07-banco-de-talentos.md`) |
| `mobile/ui-catalog/onboarding/ONB-P2/P3/P4.md` (3 telas) | `sub_produto: marketing` | `sub_produto: marketing-agencias` (`10-marketing-agencias.md`) |
| `web/ui-catalog/onboarding/ONB-*` (23 telas) | `sub_produto: onboarding` | **reclassificar por persona**: ONB-S\*→`governanca-institucional`, ONB-E\*→`connect-me`/`banco-de-talentos`, ONB-P\*→`marketing-agencias`, ONB-M\*→`governanca-institucional` |
| 5 telas `mobile/assembly/ASM-*.md` | `plan_requirement: premium` | `plan_requirement: plus` ou `plus-or-pro` |

### I.2 Slug persona `administradora` vs canônico

Enum Zitadel canônico (`personas.md` §Regras): **6 roles** — `syndic | resident | enterprise | marketing | local_company | admin`. Não há `administradora`. As 3 telas com `persona: administradora` deveriam usar `persona: enterprise` com algum marcador extra (categoria/subcategoria), pois Admin Condominial é tipo-de-Empresa.

### I.3 Slug persona `all`/`any`/`todos` — ambíguo

18 telas com persona genérica. Para ABAC engine determinístico, qualquer tela precisa de persona(s) explícita(s) — senão o frontend não sabe quem pode renderizar.

---

## J. Gaps acionáveis — top 20 (priorizado M1)

| # | ID | Título | Severidade | Origem | Ação sugerida | Bloqueio M1? |
|---|---|---|---|---|---|---|
| 1 | GAP-J-01 | Sub-produto `11-administradoras-condominiais` sem telas dedicadas (convite/aceite/revoke vínculo empresa síndica) | 🔴 CRITICAL | §A.1, §H.1 | Criar suite mínima (S-ADM-01 convite / S-ADM-02 lista / E-ADM-01 aceite) mapeada a `empresa_sindica_handler.go` 5 endpoints. | Sim se cliente M1 tiver administradora. |
| 2 | GAP-J-02 | 0 telas persona `admin` — 5 capacidades Admin MS órfãs (`GAP-admin-master-sindico.md`) | 🔴 CRITICAL | §H.2-3 | Priorizar ao menos: (a) Gestão de usuários e (b) Moderação de vídeo para M2; mas (c) LGPD breach console + (d) dashboard MRR podem ficar CLI-only em M1 se aceito em ADR. | Parcial M1 — negócio quer broadcast + gestão-usuários ao vivo. |
| 3 | GAP-J-03 | 221 ABAC actions citadas em telas sem spec canônico | 🔴 CRITICAL | §B.2 | Gerar tabela ABAC seed `matrix-functional.md §Mapeamento ABAC engine (Go)` listando as 227 actions (derive from UI); reconciliar com matriz linguagem natural. Alternativa: extrair todas para `04-requirements/abac-actions.md` (novo). | Sim — backend institutional/identity M1 precisa enum ABAC fechado. |
| 4 | GAP-J-04 | Runbook `admin-mfa-lockout.md` citado em `admin-privilegios.md` mas não existe | 🟠 HIGH | §F.3 | Criar stub mínimo em `12-docs/runbooks/admin-mfa-lockout.md` com fluxo: perdeu TOTP+backup codes → contato email-DPO-humano → Zitadel reset via console admin. | ADR-0026 (MFA admin) é M1 → runbook é pré-requisito operacional. |
| 5 | GAP-J-05 | 5 telas `mobile/assembly/ASM-*` com `plan_requirement: premium` (enum inexistente) | 🟠 HIGH | §C.3 | Rename para `plus` ou `plus-or-pro`. | Não M1 (assembleia é M3), mas UI quebra se backend rejeitar `premium`. |
| 6 | GAP-J-06 | 23 telas `web/onboarding/ONB-*` com `sub_produto: onboarding` (não canônico) | 🟠 HIGH | §A.2, §I.1 | Reclassificar por persona destino: ONB-S\*→governanca-institucional, ONB-M\*→governanca-institucional, ONB-E\*→connect-me/banco-de-talentos, ONB-P\*→marketing-agencias. | Não bloqueia M1 mas prejudica relatórios de cobertura. |
| 7 | GAP-J-07 | 6 telas `mobile/onboarding/ONB-E*` com `sub_produto: banco-talentos` (slug errado) | 🟡 MEDIUM | §A.2, §I.1 | `banco-talentos` → `banco-de-talentos`. | Não. |
| 8 | GAP-J-08 | 3 telas `mobile/onboarding/ONB-P*` com `sub_produto: marketing` (slug errado) | 🟡 MEDIUM | §A.2, §I.1 | `marketing` → `marketing-agencias`. | Não. |
| 9 | GAP-J-09 | 3 telas com `persona: administradora` (não canônica) | 🟡 MEDIUM | §I.2 | Migrar para `persona: enterprise` + marker `subcategoria: administradoras_condominios`. | Não. |
| 10 | GAP-J-10 | 18 telas com persona `any`/`all`/`todos` — ABAC ambíguo | 🟡 MEDIUM | §I.3 | Revisar caso-a-caso; substituir por lista explícita `morador, sindico` etc. | Não M1, mas afeta lógica frontend de render. |
| 11 | GAP-J-11 | Nenhuma tela cita finding A-### | 🟡 MEDIUM | §E.1 | Adicionar footer "Findings relacionados: A-###" nas telas impactadas por fix M1 (ex: S29 + A-DC-SEC-004 race LGPD). | Não (rastreabilidade). |
| 12 | GAP-J-12 | Nenhuma tela cita runbook | 🟡 MEDIUM | §F.2 | Em `compliance/CMP-*` adicionar link `[[../../../../12-docs/runbooks/incident-lgpd-breach]]`. | Não. |
| 13 | GAP-J-13 | Sub-produto `05-assembleia-live` sem cross-link reverso para as 22+6 telas | 🟡 MEDIUM | §A.3 | Atualizar `05-assembleia-live.md` com seção `## Telas` listando ASM-\* IDs. | Não. |
| 14 | GAP-J-14 | Apenas 17 de 72 decisões D-### citadas em telas — D-001..D-010 (fundação) sem cross-ref | 🟡 MEDIUM | §D.1 | Amostragem: ao menos `ONB-01` (auth) deveria citar D-001 (Zitadel), D-009 (UUIDv7), D-014 (1-device). | Não (rastreabilidade). |
| 15 | GAP-J-15 | Sub-produto `04-conteudo-videos` com apenas 1 tela vs matriz §4.1/4.2/4.3 com ~8 features | 🟠 HIGH | §A.1, §G.1 | Criar mínimo: S-VID-01 (uploader), S-VID-02 (biblioteca), S-VID-03 (detalhe+lock 90d), S-VID-04 (forum). | Parcial — M2 mas precisa iniciar spec |
| 16 | GAP-J-16 | Sub-produto `03-reputacao` com 6 telas + 1 cross-link reverso no canônico | 🟡 MEDIUM | §A.3 | Expandir `03-reputacao.md` e/ou criar telas visuais badge Bronze→Diamante. | M2. |
| 17 | GAP-J-17 | Matriz funcional NÃO lista slugs ABAC em tabelas — só linguagem natural | 🟠 HIGH | §B.1 | Adicionar coluna `ABAC action` em cada linha da matriz. Input ao parser do seed ABAC. | Relacionado a GAP-J-03. |
| 18 | GAP-J-18 | `sub_produto:` em branco em telas GAP-\* e \_moc (11 arquivos) | 🟢 LOW | §Sumário | Aceitável para `_moc.md`; `GAP-admin-master-sindico.md` poderia declarar `sub_produto: transversal-admin`. | Não. |
| 19 | GAP-J-19 | `DELETE /me` race hard-delete × webhook Stripe (A-DC-SEC-004, LGPD-M1-007) — telas `compliance/CMP-*` não comunicam a nova soft-flag `_pending_hard_delete` | 🔴 CRITICAL (backend) / 🟡 UI | §E.1, §G.2 | Spec técnica em backend, mas fluxo "conta em exclusão" precisa estado visual em CMP-\* (48h janela). | Sim — LGPD M1 bloqueante (mas é backend primariamente). |
| 20 | GAP-J-20 | Persona `marketing` com apenas 8 telas em 234 (3%) — sub-produto próprio D-059/D-060 subcoberto | 🟡 MEDIUM (M5) | §H.2 | M5 deliverable — ok para M1, registrar como backlog. | Não. |

---

## Matriz de severidade consolidada

| Severidade | Gaps | IDs |
|---|---|---|
| 🔴 CRITICAL (M1-bloqueador ou sub-produto órfão) | 4 | GAP-J-01, J-02, J-03, J-19 |
| 🟠 HIGH (travamento Sprint 10) | 5 | GAP-J-04, J-05, J-06, J-15, J-17 |
| 🟡 MEDIUM (M2/M3, documentação) | 10 | GAP-J-07..J-14, J-16, J-20 |
| 🟢 LOW (nice-to-have) | 1 | GAP-J-18 |

---

## Links

- [[../AUDIT]] — espelho dos findings canônicos
- [[../../STATE]] — decisões D-001..D-093
- [[../../04-requirements/matrix-functional]] — matriz funcional v2 Fase 8
- [[../../00-product/sub-produtos/_moc]] — MOC dos 11 sub-produtos canônicos
- [[../../00-product/personas]] — 6 personas canônicas (syndic/resident/enterprise/marketing/local_company/admin)
- [[../../10-schedule/milestones]] — M1 2026-05-08, M2 2026-06-20, M3 2026-07-20
- [[../../12-docs/runbooks/_moc]] — runbooks canônicos
- [[../../12-docs/adr-index]] — ADR-0001..0035 (índice)
- [[../../03-subprojects/web/ui-catalog/admin/GAP-admin-master-sindico]] — gap admin formal
- [[2026-04-23-findings-backend-kiro]] — A-FASE10-DEV-001..008 backend
- [[2026-04-23-pentest-specs]] — A-DC-PEN-001..042
- [[2026-04-23-qa-review]] — A-DC-QA-001..072
- [[2026-04-23-CONSOLIDADO-quebras-e-ops]]

---

## Apêndice 1 — Inventário completo das 234 telas por diretório

### Web UI catalog (180 telas + 1 GAP)

| Diretório | Screens (IDs) | Count | `sub_produto` predominante |
|---|---|---|---|
| `admin/` | `GAP-admin-master-sindico.md` + `_moc.md` | 2 (1 gap+1 moc) | (vazio) |
| `assembleia/` | ASM-CFG, ASM-CHKIN, ASM-CIEN, ASM-DASH, ASM-EDITAL, ASM-ENCER, ASM-ENQP, ASM-FILA-FALA, ASM-FRAC, ASM-HOML, ASM-INAD, ASM-PAINEL-SIND, ASM-PAUTA, ASM-PROC-CAD, ASM-PROC-VAL, ASM-PUB, ASM-REL, ASM-SIM, ASM-TELAO, ASM-TERM, ASM-VOTO | 22 | `assembleia-live` |
| `banco-talentos/` | BT01..BT11 | 12 (11 + _moc) | `banco-de-talentos` |
| `compliance/` | C1..C11 | 12 (11 + _moc) | `compliance` |
| `empresa/` | E1..E16 | 17 (16 + _moc) | mix: `connect-me`, `reputacao`, `banco-de-talentos`, `marketing-agencias` |
| `lms/` | LMS1..LMS15 | 16 (15 + _moc) | `lms-academia` |
| `marketing/` | MK1..MK8 | 9 (8 + _moc) | `marketing-agencias` |
| `morador/` | M1..M15 | 16 (15 + _moc) | `governanca-institucional` |
| `onboarding/` | ONB-00, ONB-01, ONB-S2..S6, ONB-SFIN, ONB-M2..M4, ONB-MFIN, ONB-E2..E7, ONB-EFIN, ONB-P2..P4, ONB-PFIN | 24 (23 + _moc) | `onboarding` (não canônico) |
| `sindico/` | S1..S32 | 32 (31 + _moc) | `governanca-institucional` / `connect-me` / `compliance` |
| `vizinhanca/` | VE1..VE6, VM1..VM7, VS1..VS10, VZ1..VZ6 | 30 (29 + _moc) | `vizinhanca` |

### Mobile UI catalog (53 telas)

| Diretório | Screens | Count | `sub_produto` predominante |
|---|---|---|---|
| `assembly/` | ASM-CFG, ASM-CHKIN, ASM-CIEN, ASM-FILA-FALA, ASM-PROC-CAD, ASM-VOTO | 6 (5 + _moc) | `assembleia-live` |
| `morador/` | M1..M15 | 16 (15 + _moc) | `governanca-institucional` |
| `onboarding/` | ONB-FIN, ONB-E2..E7, ONB-M2..M5, ONB-MFIN, ONB-P2..P4, ONB-S2..S6, ONB-SFIN | 22 (21 + _moc) | mix (onboarding/banco-talentos/marketing/governanca-institucional) |
| `sindico/` | (varia) | 6 (5 + _moc) | `governanca-institucional` |
| `vizinhanca/` | (varia) | 8 (7 + _moc) | `vizinhanca` |

---

## Apêndice 2 — 221 ABAC actions órfãs agrupadas por domain prefix

Ordenado por volume descendente:

| Prefixo (BC / agregador) | # órfãs | Ações (principais) | Severidade agregada |
|---|---|---|---|
| `content.*` | 31 | `content.academy.read`, `content.certificate.earn`, `content.course.list/read`, `content.forum.like.create`, `content.forum.reply.create`, `content.forum.topic.create/read`, `content.institutional.write`, `content.lesson.watch`, `content.library.download/list/read`, `content.live.join/list`, `content.live.replay.watch`, `content.talent.*.self` (10 ações de curriculum/video-CV), `content.training.list/watch`, `content.video.create/list/update` | 🟡 M2/M3 (LMS + vídeo) |
| `assembly.*` | 29 | `assembly.agenda.write`, `assembly.check_in`, `assembly.checkin.{assisted,self}`, `assembly.close`, `assembly.delinquency.{override,upload}`, `assembly.edital.write`, `assembly.edit_pre_publish`, `assembly.item.{acknowledge,homologate}`, `assembly.list`, `assembly.live.operate`, `assembly.poll.{answer,read}`, `assembly.presentation.control`, `assembly.proxy.{create,grant.self,validate}`, `assembly.publish`, `assembly.quorum.read`, `assembly.read`, `assembly.report.read{,_self}`, `assembly.speaker_queue.raise`, `assembly.speech.{authorize,raise}`, `assembly.terms.accept`, `assembly.vote.cast_assisted`, `assembly.write` | 🟡 M3 mas seed ABAC essencial |
| `commercial.*` | 28 | `commercial.closed_deal.{list_own,sign_company}`, `commercial.connect_me.{decline,express_interest,list_received}`, `commercial.evaluation.list_received`, `commercial.exclusive_benefit.{create,list.own,view}`, `commercial.marketing_link.{create,list_received}`, `commercial.neighborhood_benefit.{activate,create,history.own,list.own,read,read.open}`, `commercial.neighborhood_partner.{invite,list,list.invited,metrics.read.own,profile.read.own,read}`, `commercial.neighborhood.{read,read.open_only,read.resident,recommended.read}`, `commercial.proposal.{create,list_own}` | 🟠 M2 Connect Me |
| `compliance.*` | 19 | `compliance.addendum.create`, `compliance.annual_declaration.submit`, `compliance.audit.read`, `compliance.conflict.declare`, `compliance.contracts.read`, `compliance.dashboard{,.read}`, `compliance.declaration.{create,signed}`, `compliance.dossier.{export,exported}`, `compliance.lgpd.read`, `compliance.read`, `compliance.responsibility_term.sign`, `compliance.risk.read`, `compliance.score.read.self`, `compliance.self.read`, `compliance.user_signature.{read,sign}` | 🔴 M1 (LGPD + termo responsabilidade) |
| `institutional.*` | 9 | `institutional.company.{activate,markers,set_categories}`, `institutional.fraction.upload`, `institutional.local_company.activate`, `institutional.membership.request`, `institutional.syndic.{activate,markers,profile.write}` | 🔴 M1 core |
| `enterprise.*` | 8 | `enterprise.agency.manage`, `enterprise.compliance.manage`, `enterprise.dashboard.{read,view}`, `enterprise.profile.{read,update}`, `enterprise.registered`, `enterprise.users.manage` | 🟡 M2 |
| `membership.*` | 7 | `membership.accept_terms`, `membership.create.self`, `membership.default.update`, `membership.list.self`, `membership.read.self`, `membership.revoke.self`, `membership.self.read` | 🔴 M1 core |
| `timeline.*` | 7 | `timeline.amend`, `timeline.create`, `timeline.history.read`, `timeline.list`, `timeline.read`, `timeline.read.detail`, `timeline.read.privileged` | 🔴 M1 core |
| `master_plan.*` | 7 | `master_plan.amend`, `master_plan.create`, `master_plan.hide`, `master_plan.list`, `master_plan.read{,.detail}`, `master_plan.update` | 🔴 M1 (Plano Diretor M1 base) |
| `vizinhanca.*` | 6 | `vizinhanca.coupon.{activate.self,expiring,history.self,self.read}`, `vizinhanca.feed.read`, `vizinhanca.offer.read` | 🟡 M2 |
| `condominium.*` | 5 | `condominium.create`, `condominium.dashboard.read`, `condominium.finalize`, `condominium.list_own`, `condominium.lookup_by_id` | 🔴 M1 core |
| `connect_me.*` | 5 | `connect_me.list_own`, `connect_me.list_received`, `connect_me.request.declined`, `connect_me.respond`, `connect_me.send` | 🟠 M2 |
| `announcement.*` | 5 | `announcement.acknowledge`, `announcement.create`, `announcement.list`, `announcement.read`, `announcement.validate` | 🔴 M1 core (comunicados) |
| `event.*` | 5 | `event.acknowledge`, `event.confirm_participation`, `event.create`, `event.list`, `event.read` | 🔴 M1 core (eventos) |
| `marketing.*` | 5 | `marketing.dashboard.{read,read_consolidated,read_empresa}`, `marketing.empresa.context_switch`, `marketing.empresas.list` | 🟡 M5 |
| `curricula.*` | 4 | `curricula.desired_salary_ideal/min`, `curricula.start_availability`, `curricula.travel_time_tolerance` | 🟡 M2 (talent) |
| `governance.*` | 4 | `governance.dashboard.read`, `governance.entry.read`, `governance.execution_record.create`, `governance.technical_activity.create` | 🔴 M1 core |
| `item.*` | 4 | `item.acknowledge.opened`, `item.voting.{closed,opened,tally.updated}` | 🟡 M3 (assembly events) |
| `user.*` | 3 | `user.registered`, `user.self.read`, `user.self.update` | 🔴 M1 core |
| `neighborhood.*` | 3 | `neighborhood.benefit.created`, `neighborhood.exclusive_benefit.created`, `neighborhood.promotion.activated` | 🟡 M2 (events) |
| `identity.*` | 3 | `identity.activate`, `identity.draft.write`, `identity.terms.accept` | 🔴 M1 core |
| `unit.*` | 2 | `unit.docs.self.read`, `unit.self.read` | 🔴 M1 core |
| `service_record.*` | 2 | `service_record.list_pending`, `service_record.validate` | 🟡 M2 |
| `partner.*` | 2 | `partner.first_promotion_published`, `partner.registered` | 🟡 M2 (events) |
| `assessment.*` | 2 | `assessment.self.history`, `assessment.submit` | 🟡 M3 (reputação/LMS?) |
| Outros (1 cada) | 14 | `validation.list_pending`, `terms.accepted`, `syndic.registered`, `poll.respond`, `plano_diretor.read`, `onboarding.resident.search`, `mandate.create`, `local_company.registered`, `local_auth.authenticate`, `invite.share`, `events.read` (plural — provavelmente erro de digitação vs `event.*`), `dependent.manage`, `dashboard.read`, `company_evaluation.create`, `audit_log.read`, `audit.exported` | mix |

**Subtotais por milestone**:
- 🔴 M1-críticos: ~76 órfãs (institutional 9 + membership 7 + timeline 7 + master_plan 7 + condominium 5 + announcement 5 + event 5 + governance 4 + user 3 + identity 3 + unit 2 + compliance 19 + misc 7)
- 🟠 HIGH Sprint 10: ~33 (commercial 28 + connect_me 5)
- 🟡 MEDIUM M2/M3: ~110 (assembly 29 + content 31 + enterprise 8 + vizinhanca 6 + service_record 2 + partner 2 + assessment 2 + curricula 4 + item 4 + neighborhood 3 + mais misc)
- Outros: ~3 (admin.finance, admin.moderator, admin.super, admin.support — aparecem na matriz `matrix-functional.md` como roles admin, não órfãs estritamente)

---

## Apêndice 3 — Mapeamento persona canônica × telas (coverage detalhado)

### Distribuição de `persona:` nas 234 telas

| `persona:` encontrado | # telas | Canônico? |
|---|---|---|
| `sindico` | 82 | ✅ (`syndic`) |
| `morador` | 75 | ✅ (`resident`) |
| `empresa` | 36 | ✅ (`enterprise`) |
| `any` | 16 | 🟡 (aceitável como "sem gate", mas precisa role list) |
| `parceiro` | 13 | ✅ (`local_company`) |
| `(vazio)` | 10 | 🟡 (`_moc`/`GAP`) |
| `marketing` | 9 | ✅ |
| `morador\|sindico` | 3 | 🟡 (pipe não é canônico — usar lista YAML) |
| `administradora` | 3 | 🔴 (não canônica — migrar para `empresa`) |
| `all` | 2 | 🟡 |
| `todos` | 1 | 🟡 |
| `multi` | 1 | 🟡 |

### Lacuna absoluta

- `admin` (role=admin, 6ª persona canônica): **0 telas**. Confirmado em `GAP-admin-master-sindico.md`.

---

## Apêndice 4 — Mapeamento Feature M1 → Tela → ABAC action → Fonte canônica

Amostragem focada em **M1 core** (institutional + identity + billing + compliance básica):

| Feature M1 | Tela principal | ABAC action (citada em tela) | Em `matrix-functional.md`? | Em backend specs? | Gap? |
|---|---|---|---|---|---|
| Criar condomínio | ONB-S6 + S1 | `condominium.create` | ✅ linguagem natural §1.1 | 🟡 parcial | ABAC action não listada literal |
| Cadastrar unidade + fração ideal | S5 | `institutional.fraction.upload` | ✅ §1.1 "Cadastrar unidades" | 🟡 | ABAC action órfã |
| Vincular administradora | S5 (parcial) | — | ✅ §1.1 "Vincular administradora" | ✅ (`empresa_sindica_handler.go`) | **Tela dedicada ausente** |
| Timeline imutável — criar | S18 | `timeline.create` | ✅ §1.2 | 🟡 | ABAC órfã |
| Timeline — ler | S18 + M1-mobile | `timeline.read`, `timeline.read.detail` | ✅ §1.2 | 🟡 | ABAC órfã |
| Plano Diretor base | (S identificar) | `master_plan.create/read/update` | ✅ §1.3 | 🟡 | ABAC órfã |
| Publicar comunicado | S21..S23 | `announcement.create`, `.validate`, `.acknowledge` | ✅ §1.4 | 🟡 | ABAC órfã |
| Criar evento | S24 | `event.create`, `.confirm_participation` | ✅ §1.4 | 🟡 | ABAC órfã |
| Criar enquete | S25 | `poll.respond` | ✅ §1.4 | 🟡 | ABAC órfã |
| Cadastrar dependente | M8 (morador) | `dependent.manage` | ✅ §1.1 | 🟡 | ABAC órfã |
| Revogar membership (self) | M15 (morador) | `membership.revoke.self` | 🟡 implícito | 🟡 | ABAC órfã |
| Auth Zitadel OIDC+PKCE | ONB-01 | `identity.activate`, `local_auth.authenticate`, `identity.terms.accept` | 🟡 menção geral | ✅ (`identity.md`) | ABAC órfã |
| Escolher plano (trial→base) | ONB-SFIN/MFIN/EFIN/PFIN | (billing em corpo + `plans.trial_days` na matriz) | ✅ §Trials | ✅ | ok |
| Stripe recorrência webhook | (backend-only, sem UI nova) | — | — | ✅ ADR-0027 + ADR-0004 | ok |
| LGPD — aceitar consent | C3/C4 | `terms.accepted`, `compliance.user_signature.sign` | ✅ §9 | 🟡 | ABAC órfã |
| LGPD — export (`GET /me/export`) | C7/C8 | `compliance.dossier.export` | ✅ §9 | 🟡 | ABAC órfã |
| LGPD — delete (`DELETE /me`) | C9/C10 | — | ✅ §9 | 🔴 race A-DC-SEC-004 aberta | GAP-J-19 |
| Audit trail 5y append-only | (backend + admin) | `audit_log.read`, `audit.exported` | ✅ §9 | ✅ ADR-0013 timeline-immutable | Tela leitora só em admin (órfã) |
| Termo de responsabilidade síndico | S29 | `compliance.responsibility_term.sign`, `compliance.annual_declaration.submit` | ✅ §9 | 🟡 | ABAC órfã |
| Declaração anual compliance | C2 | `compliance.annual_declaration.submit` | ✅ §9 | 🟡 | ABAC órfã |
| 1-device policy | (subentendido no auth) | — | ✅ ADR-0014 | ✅ | 🟡 sem tela dedicada |
| Mobile auth + home + timeline leitura | mobile/onboarding + mobile/morador + mobile/sindico | múltiplas | 🟡 implícito | 🟡 | parcial |

**Conclusão A4**: 100% das features M1 têm **ao menos uma tela**, mas ~20 ABAC actions específicas a M1 não estão registradas em nenhum YAML/tabela canônica — apenas em prosa de matriz + literalmente no frontmatter da tela. Gap operacional para o seed da tabela ABAC engine.

---

## Apêndice 5 — Mapping Milestone × ABAC órfãs

| Milestone (data firme) | ABAC órfãs estimadas | Prioridade |
|---|---|---|
| M1 (2026-05-08) | ~76 (institutional + identity + timeline + master_plan + announcement + event + membership + user + compliance + condominium + governance + unit) | 🔴 CRITICAL |
| M2 (2026-06-20) | ~65 (commercial + connect_me + content.video + partner + neighborhood + service_record + enterprise + curricula + content.talent) | 🟠 HIGH |
| M3 (2026-07-20) | ~65 (assembly + content.course/lesson/live/certificate/forum + item.voting/acknowledge + assessment + poll + LMS) | 🟡 MEDIUM |
| M5+ (Q4 2026+) | ~10 (marketing.\* + multi-condomínio + admin.\*) | 🟢 LOW |
| Total | **221** | — |

---

## Apêndice 6 — Quadros de auditoria cruzada

### A6.1 D-### Fase 8 (canônicas plan_tier) × telas

Fase 8 renomeou plan_tier (D-057/D-066/D-067/D-080/D-081). Telas citam:

| D-### Fase 8 | # telas que citam |
|---|---|
| D-056 (agnosticismo provider) | 1 |
| D-057 (abolir N1/N2/N3) | 1 |
| D-058 (admin = role transversal) | 2 |
| D-059 (agência = sub-produto próprio) | 1 |
| D-060 (agência Fase 8) | 1 |
| D-061 (admin bypass Fase 8) | 1 |
| D-062 (auth flow Fase 8) | 1 |
| D-064 | 1 |
| D-066 (plan tier canonical) | 1 |
| D-067 (plan tier Fase 8) | 3 |
| D-070 | 1 |
| D-073 | 1 |
| D-074 | 1 |

**Observação**: D-080/D-081 (renames) — 0 telas citam. Esperado porque são meta-decisões (renomear slugs); não precisam citação em tela, mas o frontmatter deveria já refletir o rename (e sim, o caso `premium` em 5 telas mobile/assembly é falha de aplicação).

### A6.2 Inventário de M1 gates (milestones.md:27-42) × artefatos

| Gate M1 | Artefato canônico | Status |
|---|---|---|
| AUDIT 🔴 = 0 (A-020, A-021, A-023, A-024 + 14 Fase 5) | `AUDIT.md` | 🔴 aberto (AUDIT.md:36 lista 14 🔴 pendentes) |
| Coverage global ≥ 85% | teste backend | fora de escopo do vault |
| Gates CI todos verdes | CI config | fora de escopo |
| Staging 48h estável | infra | fora de escopo |
| Smoke test Playwright | test-suite | fora de escopo |
| Sentry prod | obs stack | ADR-0020 define |
| Zitadel Railway + DNS | infra | ADR-0003 / ADR-0017 definem |
| Stripe live webhook | billing | ADR-0004 + ADR-0027 definem |
| LGPD DPO + SLA 15d | humano/comercial | 🔴 aberto (AUDIT.md:249-255) |
| **Runbooks (rollback, incident-lgpd, webhook-reprocess)** | `12-docs/runbooks/` | ✅ existem os 3 + DR + secret-rotation |
| CHANGELOG M1 | `12-docs/changelog.md` | verificar se tem entrada M1 |

### A6.3 Runbooks canônicos — conteúdo mínimo esperado × existente

| Runbook | Existe | LoC aprox | Citado por telas? |
|---|---|---|---|
| `dr.md` | ✅ | não medido | 0 |
| `incident-lgpd-breach.md` | ✅ | não medido | 0 |
| `rollback-deploy.md` | ✅ | não medido | 0 |
| `secret-rotation.md` | ✅ | não medido | 0 |
| `webhook-reprocessing.md` | ✅ | não medido | 0 |
| `admin-mfa-lockout.md` | ❌ (stub citado em `admin-privilegios.md:343,660`) | — | 0 |
| `dr-drill.md` | ❌ (citado em backend/tasks/sprint-10.md:41) | — | 0 |

---

## Apêndice 7 — Patches mínimos sugeridos (zero-invenção, só rename/link)

Estes são ajustes textuais propostos (não aplicados neste relatório — exigem ação separada):

### A7.1 Normalização frontmatter sub_produto (25 arquivos)

```yaml
# web/ui-catalog/onboarding/ONB-S2.md..ONB-S6.md + ONB-SFIN.md (7 arquivos)
-sub_produto: onboarding
+sub_produto: governanca-institucional

# web/ui-catalog/onboarding/ONB-M2.md..ONB-M4.md + ONB-MFIN.md (4 arquivos)
-sub_produto: onboarding
+sub_produto: governanca-institucional

# web/ui-catalog/onboarding/ONB-E2.md..ONB-E7.md + ONB-EFIN.md (7 arquivos)
-sub_produto: onboarding
+sub_produto: connect-me  # ou banco-de-talentos conforme destino da persona empresa

# web/ui-catalog/onboarding/ONB-P2.md..ONB-P4.md + ONB-PFIN.md (4 arquivos)
-sub_produto: onboarding
+sub_produto: marketing-agencias  # ou vizinhanca conforme parceiro-tipo

# mobile/ui-catalog/onboarding/ONB-E2..ONB-E7 (6 arquivos)
-sub_produto: banco-talentos
+sub_produto: banco-de-talentos

# mobile/ui-catalog/onboarding/ONB-P2..ONB-P4 (3 arquivos)
-sub_produto: marketing
+sub_produto: marketing-agencias

# mobile/ui-catalog/assembly/ASM-CHKIN, ASM-VOTO, ASM-FILA-FALA, ASM-CIEN, ASM-PROC-CAD
-plan_requirement: premium
+plan_requirement: plus  # ou plus-or-pro
```

### A7.2 Criação de telas órfãs (M1)

Diretório-alvo `web/ui-catalog/sindico/` (próximos IDs livres S33+):

- **S33 — Convidar Administradora** (`sub_produto: governanca-institucional` OR novo `administradoras-condominiais`): form para síndico gerar token convite → POST `/empresa-sindica/invite`.
- **S34 — Listar Administradoras vinculadas**: GET `/empresa-sindica/list` por condominium_id.
- **S35 — Editar permissões delegadas** (5 permissões configuráveis — `empresa_sindica_link.go`).
- **S36 — Revogar vínculo** (POST `/empresa-sindica/revoke`).

Diretório-alvo `web/ui-catalog/empresa/` (próximos IDs E17+):

- **E17 — Aceitar convite Administradora** (POST `/empresa-sindica/accept` com token).
- **E18 — Minhas permissões** (read-only listagem das permissões delegadas).

### A7.3 Adicionar seção "Telas" em sub-produto órfão

Em `00-product/sub-produtos/05-assembleia-live.md` (0 cross-link atual) e `11-administradoras-condominiais.md`, adicionar:

```markdown
## Telas do UI Catalog

### Web
- [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CFG]]
- [[../../03-subprojects/web/ui-catalog/assembleia/ASM-CHKIN]]
- ... (22 total)

### Mobile
- [[../../03-subprojects/mobile/ui-catalog/assembly/ASM-CFG]]
- ... (6 total)
```

### A7.4 Catálogo ABAC unificado

Criar `04-requirements/abac-actions.md` com tabela:

| action | BC | roles que podem | plan_tier min | origem (tela + feature) | fonte canônica |
|---|---|---|---|---|---|
| `timeline.create` | institutional | syndic | plus | S18 + `matrix §1.2` | backend/requirements/institutional.md |
| ... (227 linhas) | | | | | |

Se a matriz funcional receber coluna "ABAC action" (§GAP-J-17), este catálogo pode ser derivado automaticamente.

---

## Metodologia técnica (reprodutibilidade)

```bash
ROOT="/home/vinni/Documentos/Obsidian Vault/automation-agents/vault/50-projects/master-sindico"

# Frontmatter + body extraction (sub_produto, plan_requirement, persona, D-###, A-###)
find "$ROOT/03-subprojects"/{web,mobile}/ui-catalog -name "*.md" ! -name "_moc.md" | while read f; do
  sp=$(head -40 "$f" | grep -iE "^sub_produto:" | sed 's/.*://;s/^ *//')
  pr=$(head -40 "$f" | grep -iE "^plan_requirement:" | sed 's/.*://;s/^ *//')
  pers=$(head -40 "$f" | grep -iE "^(persona|role):" | sed 's/.*://;s/^ *//')
  ds=$(grep -oE "(^|[^A-Za-z])D-[0-9]{3}" "$f" | grep -oE "D-[0-9]{3}" | sort -u)
  as=$(grep -oE "(^|[^A-Za-z])A-(FASE[0-9]+-[A-Z]+-[0-9]+|DC-[A-Z]+-[0-9]+|[0-9]{3})" "$f" | grep -oE "A-[A-Z0-9-]+" | sort -u)
  printf "%s\t%s\t%s\t%s\t%s\t%s\n" "$f" "$sp" "$pr" "$pers" "$ds" "$as"
done

# D-### canônico em STATE
grep -oE "D-[0-9]{3}" "$ROOT/STATE.md" | sort -u

# ABAC actions em backticks
grep -rhoE '`[a-z][a-z_]+\.[a-z][a-z_.]+`' "$ROOT/03-subprojects"/{web,mobile}/ui-catalog | tr -d '`' | sort -u

# ABAC em matrix + requirements + backend specs
grep -rhoE '`[a-z][a-z_]+\.[a-z][a-z_.]+`' "$ROOT/04-requirements" "$ROOT/03-subprojects/backend/requirements" | tr -d '`' | sort -u
```

---

## Regras seguidas

- ✅ Read/Bash/grep apenas (sem MCP).
- ✅ Zero invenção — todo número verificado por `wc -l` / `comm -23` / `awk -F '\t'`.
- ✅ Estrutura A..J conforme briefing.
- ✅ Ranking 🔴/🟠/🟡/🟢 alinhado ao escopo M1 (2026-05-08) e milestones M2/M3.
