---
title: Dual-Check Fase 8 — 10-marketing-agencias + 11-administradoras-condominiais + admin-privilegios
type: audit
tags: [dual-check, fase-8, marketing, administradoras, admin, master-sindico, v2]
decision_ref: D-054, D-058, D-059, D-060, D-061, D-066, D-067, ADR-0026, ADR-0032
adr_ref: ADR-0026, ADR-0032
created: 2026-04-23
propositor: destilador Fase 8 (sessão primária)
revisor: agente dual-check em contexto limpo (esta sessão)
status: consolidado
---

# Dual-Check — Revisão independente (contexto limpo)

Revisor em **sessão nova**, sem participação na redação. Insumos consultados: integrais de `10-marketing-agencias.md`, `11-administradoras-condominiais.md`, `admin-privilegios.md`; [[../../STATE]] (D-054, D-058, D-060, D-061, D-066, D-067); [[../../CLAUDE]]; ADR-0026, ADR-0032. Cross-check com grep por slugs banidos, N1/N2/N3, validação das 5 permissões canônicas e das citações D-###/ADR-####.

**Método**: (1) Read integral; (2) Cross-check 3–5 citações por arquivo; (3) Grep de slugs/`N[1-3]`; (4) Validação D-### e ADR-####; (5) Coerência com regras do briefing; (6) Pentester adversarial em gaps.

---

## 0. Tabela-resumo executiva

| Arquivo | Veredito | Bloqueadores | Críticos | Ajustes | Findings abertos |
|---|---|---|---|---|---|
| `00-product/sub-produtos/10-marketing-agencias.md` | ⚠️ Aprovar com 1 ajuste crítico | 0 | 1 (G-MKT-06 não resolvido — §2.3) | 3 | A-DC-F8-MKT-01..03 |
| `00-product/sub-produtos/11-administradoras-condominiais.md` | ⚠️ Aprovar com 2 ajustes | 0 | 2 (N2/N3 vestígio; IDOR; race UNIQUE) | 4 | A-DC-F8-ADM-01..05 |
| `03-subprojects/admin-privilegios.md` | ❌ **REDIGIR CORREÇÃO ANTES DE APROVAR** | **1** (N1/N2/N3 proibido em §8.1) | 3 | 3 | A-DC-F8-ADMIN-01..04 |

**Regras-chave checadas** (todas do briefing deste dual-check):
- ✅ Planos globais Stripe-style `trial/base/plus/pro` universal — **2/3 arquivos limpos**; **1 violação dura em admin-privilegios §8.1** (menciona `N1, N2, N3` em `plan_tier IN`).
- ✅ Admin = role privilegiada (D-058/D-061) — **cristalizado** corretamente em `admin-privilegios.md` §1 (título, §1.1, §1.2, §1.3, §1.4).
- ✅ Agência = sub-produto interno (D-059/D-060) — reforçado com login Zitadel `marketing` + ABAC grants (não impersonation) no topo do arquivo 10 e em §3.1/§3.2.
- ✅ Administradora = `Company` com subcategoria `ADMINISTRADORAS_CONDOMINIOS` + `EmpresaSindicaLink` — §4.1/§4.2 fielmente reproduzidos.
- ✅ 4-eyes D-054 em lock 90d — §5 de `admin-privilegios` ancora em STATE:221 corretamente.
- ✅ MFA M1 obrigatório em 8 endpoints críticos (ADR-0026) — §6.2 de `admin-privilegios` lista os 8.

---

## 1. `10-marketing-agencias.md`

### 1.1 Veredito
**⚠️ Aprovar com ajustes pontuais.** Destilação densa, fiel ao PDF canônico do cliente, reproduz MK1-MK8 literalmente (§9.1–9.8), amarra corretamente D-059+D-060 (login próprio Zitadel + grant ABAC, **não** impersonation). Zero slug banido no corpo operacional; único uso de `marketing_standard` é contextual (linha 28, 53, 54, 185, 772, 853) com tag explícita "ABOLIDO por ADR-0032/D-057" ou "proibidos" — conforme.

### 1.2 Cross-check de citações (amostragem)

| # | Citação no arquivo | Verificado |
|---|---|---|
| C1 | `identity.md:52,91` — role `marketing` + "não se registra sozinha" (L26, L27) | ✅ coerente com D-060 "agência se registra quando convidada"; reinterpretação canônica explícita em §L44 |
| C2 | `personas-and-plans.md:51-53` — `marketing_standard` **ABOLIDO** (L28) | ✅ alinhado com STATE D-067 (slugs abolidos) + ADR-0032 |
| C3 | `functional-matrix.md:114-125` — tabela Delegated Actor (L29, §5) | ✅ reproduzida fielmente + reinterpretada à luz D-060 |
| C4 | PDF cliente `jornada-empresa-marketing.md:28-272` — MK1-MK8 literal | ✅ §9 segue ordem + mensagens literais entre aspas |
| C5 | Req 29 `requirements.md:314` — `autorizar_exibicao_votacao` (L452) | ✅ cruza corretamente com §11 regra 8–9 + §15 (Assembly Live) |

### 1.3 Reprodução MK1–MK8 (fidelidade)

| Tela | §9.x | Mensagem literal entre aspas | Fluxo preservado |
|---|---|---|---|
| MK1 Painel | ✅ 9.1 | ✅ (L393) | ✅ 5 cards + notificações |
| MK2 Empresas | ✅ 9.2 | ✅ (L411) | ✅ seletor de sessão scoped |
| MK3 Gerenciar | ✅ 9.3 | ✅ (L430) | ✅ 3 cards |
| MK4 Publicar | ✅ 9.4 | ✅ (L442, L458) | ✅ campos + quota + ABAC check |
| MK5 Biblioteca | ✅ 9.5 | ✅ (L469, L479) | ✅ editar/ocultar |
| MK6 Dashboard | ✅ 9.6 | ✅ (L488) | ✅ 5 indicadores |
| MK7 Geral | ✅ 9.7 | ✅ (L506) | ✅ 5 indicadores cross-empresa |
| MK8 Link | ✅ 9.8 | ✅ (L524, L538–541) | ✅ lista + botões |

**Nenhum vácuo MK** — os 8 estão cobertos fielmente.

### 1.4 Grep de conformidade

| Checagem | Comando equivalente | Resultado |
|---|---|---|
| Slug `marketing_standard` sem tag proibitiva | grep -nE `marketing_standard` | ✅ 6 ocorrências **todas** em contexto "ABOLIDO"/"proibidos"/"gap" |
| `enterprise_plus`/`enterprise_pro`/`syndic_n*` | grep | ✅ zero |
| N1/N2/N3 | grep | ✅ zero no arquivo |
| Slugs `{trial,base,plus,pro}` universais | grep | ✅ usados apenas como tiers universais |

### 1.5 Validação das D-### citadas

- **D-059** (STATE:497-502) ✅ existe; mesmo conteúdo reproduzido no topo e §11 R1.
- **D-060** (STATE:354-364) ✅ existe; conteúdo "login próprio + ABAC grants" corresponde ao registrado em STATE (linhas 356-364).
- **D-057/ADR-0032** ✅ abolição de slugs confere.
- **D-066/D-067** ✅ nomenclatura banida confere.
- **D-056** ✅ agnosticismo provider citado em §15.
- **D-067** (Stack Abril/2026) ✅ Resend canônico.

### 1.6 Gaps adversariais (pentester + DDD)

| ID | Severidade | Finding |
|---|---|---|
| **A-DC-F8-MKT-01** | 🟡 médio | Arquivo assume **`DelegationGrant` não modelado** (G-MKT-01 🔴) mas **expõe SQL UNIQUE PARTIAL em §12 INV-DELEG-003** antes do aggregate existir. Sequência correta: aggregate em `01-domain/aggregates/DelegationGrant.md` **primeiro**, depois migration. Sem isso, G-MKT-01/02/03 continuam 🔴 dependentes. |
| **A-DC-F8-MKT-02** | 🟡 médio | **G-MKT-06 (Plano próprio de agência)** segue sem decisão formal (§5 + G-MKT-06). Briefing deste dual-check explicita "G-MKT-06 vácuo agência paga plano próprio". Recomendar: registrar D-### novo decidindo **"agência não paga plano próprio no MVP; quotas vêm da Empresa cliente (plus/pro)"** — já é a premissa implícita em §5, só falta o selo de decisão. |
| **A-DC-F8-MKT-03** | 🟡 médio | **DelegationGrant não modelado no aggregate catalog** (G-MKT-01 🔴). Risco: MK4/MK5 quoted em §11 dependem desse aggregate para ABAC check. Criar [[../../01-domain/aggregates/AgenciaLink|DelegationGrant (renamed)]] com invariantes INV-DELEG-001..005. |
| — | 🟢 baixo | §9.8 (MK8) registra divergência com inventário v2 (`_reconciliacao-dev/04-telas-jornadas.md:166` Audit Log Delegation); tratado como G-MKT-05 + G-MKT-11 — ok, basta PO decidir. |
| — | 🟢 baixo | §16 threat-model "Agência maliciosa publicar difamatório" dependente de feature "aprovação pré-publicação" (G-MKT-10) — backlog, ok. |

### 1.7 Veredito detalhado

**Concordo** com a destilação como **peça canônica do sub-produto**, com caveats:
1. Registrar D-### fechando G-MKT-06 antes de migrar para vault.
2. Criar aggregate `DelegationGrant` em `01-domain/aggregates/` (G-MKT-01) antes de escrever requirements em `04-requirements/functional/commercial.md` (G-MKT-04).
3. Ajustes 🟡 não são bloqueadores de dual-check; são tasks de follow-up.

---

## 2. `11-administradoras-condominiais.md`

### 2.1 Veredito
**⚠️ Aprovar com 2 ajustes.** Destilação exaustiva e minuciosa, corretamente modela Administradora como `Company` + `EmpresaSindicaLink`, **reproduz fielmente as 5 permissões canônicas** (§7), cobre S5 ponta-a-ponta (§10), expõe **honestamente o IDOR no Accept** (§10.2 passo 4 + §18.2) e a **race UNIQUE ausente** (§13 INV-AC-01). Esses dois findings são **valor** do arquivo, não defeito.

### 2.2 Cross-check de citações

| # | Citação | Verificado |
|---|---|---|
| C1 | Schema TS `ADMINISTRADORAS_CONDOMINIOS` ∈ `ADMINISTRACAO_GESTAO_GOVERNANCA` (§4.1, linha 89) | ✅ confere com tabela L36 |
| C2 | `empresa_sindica_link.go:32-46` campos (§4.2) + `Accept(companyID)` (§8.1) | ✅ campos completos e guards listados em §14.2 |
| C3 | Migration `00307` + UNIQUE `token` global + index parcial `WHERE status='pending'` | ✅ L379 (INV-AC-02), conf. em §18.1 tabela |
| C4 | `EmpresaSindicaLinkStatus` ENUM `pending|active|revoked` (enums.go:270-289) | ✅ §9.1 e state machine §14 batem |
| C5 | Req 93 AC1-10 (`requirements.md:3118`) | ✅ referenciado em §1, §7, §11, §20.2 |

### 2.3 Validação das 5 permissões canônicas (checklist briefing)

| # | Nome Go | JSON | SQL | Presente? |
|---|---|---|---|---|
| 1 | `ViewPublications` | `view_publications` | `perm_view_publications` | ✅ L165 |
| 2 | `EditSyndicPosts` | `edit_syndic_posts` | `perm_edit_syndic_posts` | ✅ L166 |
| 3 | `HidePublications` | `hide_publications` | `perm_hide_publications` | ✅ L167 |
| 4 | `ValidateExecutions` | `validate_executions` | `perm_validate_executions` | ✅ L168 |
| 5 | `ValidateCommunications` | `validate_communications` | `perm_validate_communications` | ✅ L169 |

**5/5 permissões canônicas** presentes com triplo mapeamento Go↔JSON↔SQL. §7.2 cobre divergência de nomenclatura `validar_comunicados_empresas` vs `validar_comunicados_tecnicos` — bom (registra como A-###).

### 2.4 State machine (§14) — checklist briefing

- ✅ `pending → active` (via `Accept(company_id)`)
- ✅ `pending → revoked` (via `Revoke()` antes do aceite)
- ✅ `active → revoked` (via `Revoke()`)
- ✅ Guards: `IsPending`, `AlreadyRevoked`
- ✅ Transições inválidas explicitamente rejeitadas (§14.3)

### 2.5 Findings de pentester

| ID | Severidade | Finding |
|---|---|---|
| **A-DC-F8-ADM-01** | 🔴 **crítico** | **IDOR em Accept** (§10.2 passo 4 + §18.2): "não há validação de que o `company_id` do body corresponde ao email do convite". Atacante com token pode aceitar com outra empresa. Listado corretamente pelo destilador como "**Alta — pentester**". **Confirmado**. Ação: validar `{user.company_id == body.company_id}` + `link.email == company.primary_email` no use case `Accept`. **Bloqueador de produção** (não de dual-check). |
| **A-DC-F8-ADM-02** | 🔴 **crítico** | **Race condition UNIQUE constraint ausente** (§13 INV-AC-01): SQL não tem `CREATE UNIQUE INDEX ... WHERE status='active'`. Dois aceites simultâneos ⇒ dois links `active` para o mesmo condomínio. **Ação pentester confirmada**: migration hotfix com UNIQUE PARTIAL. |
| **A-DC-F8-ADM-03** | 🟡 médio | **Vestígio N2/N3** (§4.3 linha 120 + §20.2 linha 593): `"administradoras_vinculadas ... perfil do Síndico para N2/N3"` e `"Req 6.2 ... perfil Síndico N2/N3"`. Nomenclatura abolida por **D-067** (STATE:560-564: "Síndico N1/N2/N3 **abolidos inclusive na UI**"). Ajustar para referência neutra (ex: "perfil do Síndico nos tiers Plus/Pro" ou "tiers pagantes"). |
| **A-DC-F8-ADM-04** | 🟡 médio | **Sem domain events** (§15.1 gap): aggregate `EmpresaSindicaLink` não emite eventos. Req 93 AC4 exige `EmpresaSindicaLinked`. Destilador lista em §15.2 os 5 eventos canônicos — ok. Recomendar D-### ou ADR para domain events desse BC. |
| **A-DC-F8-ADM-05** | 🟡 médio | **Purge de cache ABAC depende de D-001** (§18.2 + §19 Q1): revogação instantânea do Req 93 AC9 é **ainda em aberto**. Elevar prioridade D-001; impacto em Req 93 AC9 torna isso security-critical. |

### 2.6 Grep de conformidade

| Checagem | Resultado |
|---|---|
| Slugs banidos (`*_standard`, `syndic_n*`, `empresa_plus`) | ✅ zero |
| `{trial,base,plus,pro}` universal | ✅ §6 alinhado com ADR-0032 |
| N1/N2/N3 | ⚠️ 2 vestígios (L120, L593) — ver A-DC-F8-ADM-03 |

### 2.7 Veredito detalhado

**Concordo** com a destilação como **peça canônica do sub-produto**, sem bloqueadores de dual-check (os pentester findings são defeitos de código/migration, não do documento — que **os registra corretamente**). Ajustar apenas:
1. Remover/neutralizar 2 vestígios `N2/N3` (A-DC-F8-ADM-03).
2. Migrar A-DC-F8-ADM-01..05 para [[../AUDIT]] como A-### formais.
3. Considerar ADR para D-001 (purge instantâneo) antes de M1.

---

## 3. `03-subprojects/admin-privilegios.md`

### 3.1 Veredito
**❌ REDIGIR CORREÇÃO ANTES DE APROVAR** — há **1 bloqueador duro** (violação de regra universal de planos) e 3 findings críticos de coerência/segurança a endereçar. Fora isso, o documento é **ouro arquitetural**: cristaliza D-058/D-061 em 16 seções, detalha os 4 sub-papéis com matriz exaustiva, ancora 4-eyes D-054 com fluxo canônico, e lista os 8 endpoints MFA M1 (ADR-0026) corretamente.

### 3.2 Cross-check de citações

| # | Citação | Verificado |
|---|---|---|
| C1 | D-058 (title "Painel Admin MS NÃO é app separada") em §1.1, §1.2, §1.3, §1.4 | ✅ STATE:488 confirma título exato |
| C2 | ADR-0026 MFA M1 (§6) — lista 8 endpoints | ✅ 8 itens em §6.2 batem com briefing |
| C3 | D-054 4-eyes (§5) — fluxo canônico solicitante A + aprovador B ≤24h | ✅ STATE:221-238 reproduzido fielmente |
| C4 | ABAC cadeia `admin_bypass → role → action → tenant → scope` (§3.1) | ✅ coerente com ADR-0012 + INV-086 |
| C5 | 4 sub-papéis (super/moderador/suporte/financeiro) em §2 | ✅ todos com hierarquia + permissões + MFA + audit |

### 3.3 Validação regras-chave do briefing

| Regra | Validado? |
|---|---|
| Admin = role privilegiada (D-058), NÃO app separada | ✅ cristalizado em título, §1.1, §1.2 ("O que NÃO é"), §1.3 (Consequência arquitetural) |
| 4 sub-papéis: Super Admin, Moderador, Suporte, Financeiro | ✅ §2.1–§2.4 + quadro-resumo §2.5 |
| 4-eyes policy D-054 em lock 90d bypass | ✅ §5 inteira, fluxo canônico em §5.2 |
| MFA M1 ADR-0026 obrigatório em 8 endpoints críticos | ✅ §6.2 lista os 8 (incluindo `POST /admin/v1/*` como item 8) |
| Audit trail **separado** de `lgpd_logs` | ✅ §7.1 tabela dupla distinta (audit_trail vs lgpd_logs) + §7.2 schema canônico |

### 3.4 🔴 BLOQUEADOR DURO: violação de `N1/N2/N3` proibido

**Linha 462** (§8.1 Broadcasts globais, Segmentação):
```
- **Por plano**: `plan_tier IN (base, plus, pro, N1, N2, N3, ...)`.
```

**Violação direta** de:
- **D-067** (STATE:560-567): "Termos 'Síndico N1', 'N2', 'N3', 'Empresa Plus', 'Empresa Pro' (etc.) são **abolidos** inclusive na UI e na matriz funcional"
- **D-066 + ADR-0032**: planos globais Stripe-style **universais** = `{trial, base, plus, pro}` — **zero** slug adicional.
- Briefing deste dual-check regra 1: "Planos globais Stripe-style `trial/base/plus/pro` universal — **zero slugs**".

**Correção obrigatória**: substituir por `plan_tier IN (trial, base, plus, pro)` (4 tiers universais exatos; **sem** ellipse `...` que sugere extensibilidade com slugs persona-específicos). **Uma linha; impacto zero em outras seções.**

### 3.5 Findings adicionais

| ID | Severidade | Finding |
|---|---|---|
| **A-DC-F8-ADMIN-01** | 🔴 crítico (bloqueador) | **Violação N1/N2/N3** em §8.1 L462 (detalhada em §3.4 acima). |
| **A-DC-F8-ADMIN-02** | 🟡 médio | **Referência a `D-058` e `D-061` misturada**: STATE registra título D-058 como "Painel Admin MS NÃO é app separada" (entrada 488-502, Fase 7) **e** D-061 repete o mesmo conteúdo (366-376, Fase 8). O arquivo **prefere D-058** como âncora (§1 abertura + §16.1). Coerente com o briefing deste dual-check ("Admin = role privilegiada D-058"), mas **o documento também lista D-061** em depends/refs. **Ação**: adicionar nota no §16.1 explicando que D-058 (Fase 7) e D-061 (Fase 8) são **a mesma decisão renumerada**; evitar ambiguidade em consumidores. |
| **A-DC-F8-ADMIN-03** | 🟡 médio | **§1.2 cita "`admin-web/` separada — rejeitado"** — consistente com D-058, porém o §4.2 "Em `sindico-web` (conceitual — hoje em `web/`)" e §4.3, §4.4 repetem **"(conceitual — hoje em `web/`)"** 3x. Ok como nota transitória, mas sugerir consolidar em 1 nota de rodapé para reduzir ruído. |
| **A-DC-F8-ADMIN-04** | 🟡 médio | **Tabela §3.2 (matriz ABAC exaustiva)** lista actions `institutional.tenant.suspend` com `Tenant scope = cross` e `SuperAdmin = ✅` — coerente. Mas **`Moderador`/`Suporte`/`Financeiro`** estão **todos com `❌`** E **"Tenant scope = cross"** — campo "tenant scope" fica inconsistente (para ❌, tenant scope é irrelevante). Sugerir: usar `—` em "tenant scope" quando permissão = `❌` para clareza. Não afeta produção; polimento editorial. |

### 3.6 Grep de conformidade

| Checagem | Resultado |
|---|---|
| N1/N2/N3 | ❌ **1 violação dura (L462)** |
| Slugs compostos (`marketing_standard`, `enterprise_plus`, etc.) | ✅ zero |
| `{trial,base,plus,pro}` universal | ⚠️ usado em §3.2 corretamente, **mas** L462 quebra a regra |
| Admin = app separada? | ✅ nenhuma menção positiva — sempre contrastado |

### 3.7 Validação dos 4 sub-papéis (§2)

| Sub-papel | bypass | ABAC scope | MFA M1 | IP allowlist | 4-eyes | Audit duplo | Presente? |
|---|---|---|---|---|---|---|---|
| Super Admin | 100% | sem restrição | ✅ | M3+ obrig | aprovador | ✅ | §2.1 ✅ |
| Moderador | parcial | content/harassment | ✅ | não | só remove_pre90d | ✅ (se PII) | §2.2 ✅ |
| Suporte | read + impersonate | cross-tenant read | ✅ | não | só impersonate ≥30min | **sempre** | §2.3 ✅ |
| Financeiro | parcial | billing.* | ✅ | não | refund ≥ R$10k | ✅ | §2.4 ✅ |

Quadro-resumo em §2.5 sintetiza corretamente. **4/4 sub-papéis** canônicos.

### 3.8 Validação 8 endpoints MFA M1 (§6.2)

Briefing deste dual-check: "MFA M1 obrigatório em 8 endpoints críticos (ADR-0026)". Arquivo lista 8:
1. `DELETE /api/v1/me` ✅
2. `DELETE /api/v1/me/devices/:id` ✅
3. `GET /api/v1/me/export` ✅
4. `POST /api/v1/assemblies/:id/minutes/publish` ✅
5. `POST /api/v1/contracts/:id/approve` (≥R$10k) ✅
6. `POST /api/v1/condominiums/:id/transfer-mandate` ✅
7. `PATCH /api/v1/me` (email/phone) ✅
8. `POST /admin/v1/*` (qualquer rota admin mutating) ✅

**8/8 confere.** 🎯

### 3.9 Veredito detalhado

**Discordo da aprovação até correção do bloqueador L462.** Aplicar patch único:
```diff
- - **Por plano**: `plan_tier IN (base, plus, pro, N1, N2, N3, ...)`.
+ - **Por plano**: `plan_tier IN (trial, base, plus, pro)`.
```

Pós-correção, o arquivo é **canônico exemplar** — destilação com profundidade adequada à natureza transversal do tema, cruzando 5 fontes (STATE, ADR, personas, ui-catalog, invariants). **Único bloqueador é a linha 462**.

---

## 4. Consolidação cross-arquivos

### 4.1 Coerência entre os 3 arquivos

| Tema | `10-marketing` | `11-administradoras` | `admin-privilegios` | Coerente? |
|---|---|---|---|---|
| Admin ≠ Administradora Condominial | §4 "Admin Master Síndico papel transversal" | §3 "Não confundir com Admin da plataforma. `admin` é role privilegiada transversal da equipe Master Síndico (D-058)" | §1 título | ✅ 3x reforçado |
| Agência ≠ Administradora ≠ Empresa comum | §2, §4 | §3 "Não é Agência de Marketing" | — | ✅ |
| Role Zitadel `marketing` | §3.1 | §3 "Não é Agência" | §3.2 tabela permissions | ✅ |
| 4-eyes D-054 | §15 "Admin MS pode suspender Company" | §12 BR-AC-13 | §5 inteira | ✅ |
| MFA M1 (ADR-0026) | — | — | §6 inteira | ✅ (escopo admin) |
| `trial/base/plus/pro` universal | §5 | §6 | §3.2, §8.1 (❌ vestígio) | ⚠️ `admin` tem 1 vazamento |
| Audit trail append-only + separado de lgpd_logs | §6.5 | §11 | §7.1 tabela dupla | ✅ |

### 4.2 Dependências cruzadas

```
admin-privilegios.md
    ├── referencia 10-marketing (G-ADMIN-05 push mobile 4-eyes)
    ├── referencia 11-administradoras (indireto via "Company administradora suspensa")
    └── canonical para D-058, D-054, ADR-0026

10-marketing-agencias.md
    └── referencia admin-privilegios em §4 (Admin MS) + §15 (moderação + suspensão)

11-administradoras-condominiais.md
    └── referencia admin-privilegios em §5 (Admin MS pode suspender) + §12 BR-AC-13
```

Links cruzados bem posicionados. Ciclo de `[[wiki-links]]` íntegro (não há link quebrado detectável pelo nome de arquivo).

### 4.3 Sumário findings (para migrar a [[../AUDIT]] como A-### formais)

| ID dual-check | Arquivo | Severidade | Descrição curta | Ação sugerida |
|---|---|---|---|---|
| **A-DC-F8-ADMIN-01** | admin-privilegios §8.1 L462 | 🔴 bloqueador | N1/N2/N3 em `plan_tier IN` | Patch 1-linha obrigatório |
| **A-DC-F8-ADM-01** | 11-administradoras §10.2, §18.2 | 🔴 crítico | IDOR Accept (company_id não validado vs email) | Fix no use case Accept |
| **A-DC-F8-ADM-02** | 11-administradoras §13 INV-AC-01 | 🔴 crítico | Race UNIQUE PARTIAL ausente | Migration hotfix |
| **A-DC-F8-MKT-01** | 10-marketing §12 INV-DELEG-003 | 🟡 médio | UNIQUE PARTIAL proposto sem aggregate criado | Aggregate primeiro, SQL depois |
| **A-DC-F8-MKT-02** | 10-marketing §5, G-MKT-06 | 🟡 médio | Plano próprio de agência indefinido | Abrir D-### para fechar |
| **A-DC-F8-MKT-03** | 10-marketing §7.3, G-MKT-01 | 🟡 médio | `DelegationGrant` não em aggregate catalog | Criar aggregate |
| **A-DC-F8-ADM-03** | 11-administradoras L120, L593 | 🟡 médio | Vestígio N2/N3 (D-067 abolido) | Reescrever neutro |
| **A-DC-F8-ADM-04** | 11-administradoras §15.1 | 🟡 médio | Domain events não emitidos (Req 93 AC4) | ADR ou D-### |
| **A-DC-F8-ADM-05** | 11-administradoras §18.2, §19 Q1 | 🟡 médio | Purge cache ABAC depende D-001 | Elevar D-001 a security-critical |
| **A-DC-F8-ADMIN-02** | admin-privilegios §16.1 | 🟡 baixo | D-058 ≡ D-061 (renumeração) | Nota de rodapé explicando |
| **A-DC-F8-ADMIN-03** | admin-privilegios §4.2-4.4 | 🟢 baixo | Repetição "(conceitual — hoje em web/)" | Consolidar em footnote |
| **A-DC-F8-ADMIN-04** | admin-privilegios §3.2 | 🟢 baixo | "tenant scope" inconsistente para ❌ | Editorial |

### 4.4 Decisão final

| Arquivo | Decisão |
|---|---|
| `10-marketing-agencias.md` | **✅ Aprovado** (ajustes 🟡 migrar como A-### em AUDIT, não bloqueiam migração) |
| `11-administradoras-condominiais.md` | **✅ Aprovado** (ajustes 🟡 e 🔴 **de código**, não do documento — que registra honestamente; patch editorial N2/N3 recomendado) |
| `admin-privilegios.md` | **❌ CORRIGIR L462 PRIMEIRO** — depois **✅ Aprovado** |

---

## 5. Action items (migrar para AUDIT + ROADMAP)

- [ ] **Patch obrigatório**: corrigir `admin-privilegios.md` L462 → `plan_tier IN (trial, base, plus, pro)` (A-DC-F8-ADMIN-01).
- [ ] Migrar A-DC-F8-ADMIN-01..04 + A-DC-F8-ADM-01..05 + A-DC-F8-MKT-01..03 para [[../AUDIT]] com IDs A-### sequenciais.
- [ ] Patch editorial em `11-administradoras-condominiais.md` L120, L593 (trocar N2/N3 por linguagem neutra — A-DC-F8-ADM-03).
- [ ] Abrir D-### para fechar G-MKT-06 ("agência não paga plano próprio no MVP; quotas vêm da Empresa cliente") — A-DC-F8-MKT-02.
- [ ] Criar aggregate `01-domain/aggregates/DelegationGrant.md` com INV-DELEG-001..005 (A-DC-F8-MKT-03).
- [ ] Elevar D-001 (purge instantâneo cache ABAC) a security-critical — impacta Req 93 AC9 da Administradora (A-DC-F8-ADM-05).
- [ ] Registrar IDOR Accept (A-DC-F8-ADM-01) + race UNIQUE (A-DC-F8-ADM-02) como tasks M1 de hotfix no backend.
- [ ] Nota de rodapé em `admin-privilegios.md` §16.1 explicando D-058 ≡ D-061 (renumeração Fase 7→Fase 8) — A-DC-F8-ADMIN-02.

---

## 6. Links

- [[_moc]]
- [[template]]
- [[../AUDIT]]
- [[../../STATE]]
- [[../../STATE#D-054 — Lock 90d bypass admin via 4-eyes policy]]
- [[../../STATE#D-058 — Painel Admin MS NÃO é app separada — é role privilegiada transversal]]
- [[../../STATE#D-060 — Agência de Marketing = sub-produto interno próprio (não externo)]]
- [[../../STATE#D-067 — N1/N2/N3 e outros nomes de plano por persona ABOLIDOS COMPLETAMENTE]]
- [[../../02-architecture/adr/0026-admin-mfa-m1]]
- [[../../02-architecture/adr/0032-global-plans-stripe-style]]
- [[../../00-product/sub-produtos/10-marketing-agencias]]
- [[../../00-product/sub-produtos/11-administradoras-condominiais]]
- [[../../03-subprojects/admin-privilegios]]
- [[../../06-execution/playbooks/dual-check]]
