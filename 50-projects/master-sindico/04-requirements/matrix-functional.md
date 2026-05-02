---
title: "Matriz Funcional — Master Síndico v2 (consolidado Fase F, ADR-0032)"
type: spec
tags: [requirement, matrix, feature-matrix, abac, plan-tier, role, master-sindico, v2, fase-8-consolidado, fase-f-pdfs-absorvidos]
source: "sub-produtos/* + ADR-0032 + STATE D-056..D-069 (Fase 8) + D-070..D-076 (Fase 7) + D-094..D-103 (Fase 11) + _chaos/Matriz Funcional GERAL.pdf + _chaos/MATRIZ FUNCIONAL MÓDULO TRIAL.pdf (Fase F absorção 2026-04-25)"
created: 2026-04-23
updated: 2026-04-25
status: consolidado-fase-f
dual_check: pending
fase_f_absorvido: true
---

# Matriz Funcional — Master Síndico v2

> **⚠️ Atualização Fase F (2026-04-25)**: matriz reconciliada com `_chaos/Matriz Funcional GERAL.pdf` (2026-03-09) + `_chaos/MATRIZ FUNCIONAL MÍDULO TRIAL.pdf` (2026-03-09) via tradução **N1/N2/N3 → plan_tier** (D-081/D-096). Tabelas usam tiers universais `{trial, base, plus, pro}` em todo conteúdo canônico vivo; rótulos UI legados ("Síndico Pro" = `plan_tier=pro`, "Morador Pagante" = `(resident, plus)`, etc.) ficam apenas em §"Tradução histórica" abaixo e em banners explícitos. **Modelo backend usa `plan_tier` strict — slugs/labels compostos proibidos** (lint CI). PDFs originais arquivados em `60-sources/master-sindico-research/client-material/pdfs/2026-03-09-matriz-funcional-{geral,trial}.pdf`.

Matriz **completa** feature × **plan_tier** × **role**, alinhada com ADR-0032 (planos globais Stripe-style). Fonte de verdade para ABAC engine (seed da matriz de permissão) e gate de visibilidade UI (frontend não renderiza botão sem permissão).

> **Reescrita Fase 8** — versão anterior usava colunas `Base | Pg | N1 | N2 | N3 | Plus | Pro | Mkt | Viz | Adm` (slugs compostos por persona). Esta versão consolidada por **ADR-0032** + **D-067 Fase 8** (abolir N1/N2/N3) usa colunas `Trial | Base | Plus | Pro` universais + coluna "Aplicável a role" quando restrita.

## Legenda

| Símbolo | Significado |
|---|---|
| ✅ | Permitido (sem quota numérica) |
| ❌ | Bloqueado (`403 FORBIDDEN` ou `402 PLAN_REQUIRED`) |
| ⏳ | Disponível durante trial (bloqueia estratégico pós-conversão — soft-block) |
| 🔒 | Requer upgrade de plan tier |
| `N/u` | Quota numérica: N por unidade (`/a`=ano, `/m`=mês, `/d`=dia, `/4h`=cooldown) |
| ∞ | Ilimitado |
| `25%` | Preview parcial (paywall; ver conteúdo integral requer tier) |
| `n/a` | Não aplicável à role |
| `⊃` | Superset — role pode fazer via bypass (ex: admin) |

## Enum canônico (invariante ADR-0032)

```
plan_tier ∈ {trial, base, plus, pro}
user.role ∈ {syndic, resident, enterprise, marketing, local_company, admin}
```

Slugs compostos (`syndic_n1`, `enterprise_plus`, `resident_paid`, `local_company_standard`, `marketing_standard`) **proibidos** em código, banco, UI, docs.

Admin bypass: `role=admin` curto-circuita matriz (allow para tudo, exceto ações com `admin_scoped=false`) — não mostra como coluna; implícito em todas as linhas com coluna extra "**Admin**".

## Tradução histórica (N1/N2/N3 + labels legados → tier)

Para referências legadas no backend/Content/PDFs do cliente, usar:

| Terminologia antiga (Matriz Funcional GERAL.pdf 2026-03-09 + outros docs) | Equivalente canônico |
|---|---|
| Síndico N1 "Aprender" | `role=syndic, tier=trial` OU `role=syndic, tier=base` (consumo/leitura) |
| Síndico N2 "Atuar" | `role=syndic, tier=plus` |
| Síndico N3 "Consolidar" | `role=syndic, tier=pro` |
| Síndico Gestor (PDF "Perfil e Visibilidade") | sinônimo de **Síndico Base** → `role=syndic, tier=base` |
| Síndico Plus (PDF "Perfil e Visibilidade") | `role=syndic, tier=plus` |
| Síndico Pro (PDF "Perfil e Visibilidade") | `role=syndic, tier=pro` |
| Empresa Plus / Emp. Plus | `role=enterprise, tier=plus` |
| Empresa Pro / Emp. Pro | `role=enterprise, tier=pro` |
| Morador Base | `role=resident, tier=base` |
| Morador Pg / Morador Pagante (rótulo histórico) | `role=resident, tier=plus` (**não usar como label vivo** em UI/backend; permanece só nesta seção) |
| Parceiro / Vizinhança / Comércio Local (sinônimos) | `role=local_company, tier=plus` ou `pro` |
| Marketing / Agência Marketing | `role=marketing, tier=plus` ou `pro` (sub-role rotas internas — D-102) |
| "My Síndico" (nomenclatura antiga do produto) | "Master Síndico" (nome canônico atual) |
| Lives (criação "My Síndico" no PDF) | Lives broadcast `role=admin` + `role=enterprise, tier=pro` (D-097/D-133) |
| Admin MS | `role=admin` (tier ortogonal/irrelevante) |

> Qualquer código/spec/UI que ainda use nomenclatura antiga precisa ser reescrito. DT-004 (backend) e DT-005 (artefatos v2) rastreiam a migração. **Lint CI** rejeita strings hardcoded `syndic_n1`, `enterprise_plus`, `resident_paid`, `morador_pagante`, `my_sindico` no código (ADR-0032).

---

# Sub-produto 1 — Governança Institucional

Fonte: `00-product/sub-produtos/01-governanca-institucional.md` §5 + `backend/.kiro/specs/.../institutional.md` + STATE D-067/D-069 Fase 8.

## 1.1 Condomínio e unidades

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Criar condomínio | ⏳ | ❌ | ✅ (1) | ✅ (até 15) | syndic | ⊃ |
| Cadastrar unidades | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Vincular administradora | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Editar permissões delegadas administradora | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Visualizar condomínio (leitura) | ✅ | ✅ | ✅ | ✅ | resident, syndic | ⊃ |
| Cadastrar dependentes (titular) | n/a | ✅ | ✅ | ✅ | resident (titular) | ⊃ |
| Encerrar vínculo morador (cascateia dependentes) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

## 1.2 Timeline da gestão

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ler Timeline | ✅ | ✅ | ✅ | ✅ | resident, syndic, enterprise | ⊃ |
| Criar atividade Timeline | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Criar adendo (C6) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Exportar PDF Timeline | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

## 1.3 Plano Diretor

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ver Plano Diretor | ✅ | ✅ | ✅ | ✅ | resident, syndic | ⊃ |
| Criar atividade PD | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Ver % de execução PD | ✅ | ✅ | ✅ | ✅ | resident, syndic | ⊃ |

## 1.4 Comunicados, eventos, enquetes

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ler comunicados | ✅ | ✅ | ✅ | ✅ | resident, syndic | ⊃ |
| Criar comunicado oficial | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| RSVP/ciência em evento | ✅ | ✅ | ✅ | ✅ | resident | n/a |
| Criar evento condomínio | ⏳ | ✅ | ✅ | ✅ | syndic | ⊃ |
| Responder enquete interna | ✅ | ✅ | n/a | n/a | resident | n/a |
| Criar enquete interna | ⏳ | ✅ | ✅ | ✅ | syndic | ⊃ |
| Responder avaliação gestão (morador) | ✅ | ✅ | n/a | n/a | resident | n/a |
| Ver agregado avaliação gestão | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

## 1.5 Validações e mandato

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ver hub de validações pendentes | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Decidir validação (validate/reject/adjust) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Encerrar mandato | ⏳ | ❌ | ✅ (compliance OK) | ✅ | syndic | ⊃ |
| Transferir mandato | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Gerar dossiê PDF fim de mandato | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

**Quotas**:
- Condomínios por síndico: hard limit 15 (BIL-032); tier=base máx 1.
- Exportações: sem quota numérica (tier-gated).

---

# Sub-produto 2 — Connect Me

Fonte: `sub-produtos/02-connect-me.md` §5 (quotas por tier) + ADR-0032 matriz + **D-094 Fase 11** (morador base = 2/ano — sobrescreve D-058/D-079 que diziam 0).

## 2.1 Connect Me — 4 direções

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Morador → Síndico (frio) | ❌ | **2/a** (D-094) | 4/a | 4/a | resident | ⊃ |
| Síndico → Empresa (RFP quente) | ⏳ | ❌ | 4/a | ∞ | syndic | ⊃ |
| Empresa → Síndico (responder RFP) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Empresa → Empresa (E→E parceria) | ⏳ | ❌ | ⚠️ (só responde) | ∞ (abre+responde) | enterprise | ⊃ |
| Síndico → Parceiro Vizinhança | ⏳ | ❌ | ✅ | ∞ | syndic | ⊃ |
| Parceiro Vizinhança recebe | n/a | n/a | ✅ | ✅ | local_company | ⊃ |
| Marketing Link (Empresa → Agência) | n/a | n/a | ✅ | ✅ | enterprise → marketing | ⊃ |
| Agência inicia Connect Me proativo | n/a | n/a | ❌ | ❌ | marketing (anti-prospecção) | ⊃ |
| Botão denunciar (harassment report) | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |

**Notas**:
- Morador Base Connect Me = **2/ano** (D-094 Fase 11, 2026-04-24 — **sobrescreve D-058/D-079**). Alinha com Decision 10 em `personas-and-plans.md` + `commercial.md` Req 19.1. Versão anterior (0/ano) foi descartada por fechamento Q2.
- E→E: Plus apenas responde convites; Pro abre + responde.
- Agência Marketing não inicia Connect Me (anti-prospecção — `novo escopo-7.md` §1.6).

## 2.2 Quotas Connect Me consolidadas

| Role + tier | Quota/ano |
|---|---|
| resident, base | **2/a** (D-094 Fase 11 — sobrescreve D-058) |
| resident, plus | 4/a |
| syndic, trial | ⏳ 0 (bloqueio trial) |
| syndic, base | ❌ (tier insuficiente) |
| syndic, plus | 4/a |
| syndic, pro | ∞ |
| enterprise, plus | ∞ (só responde) |
| enterprise, pro | ∞ (abre + responde) |
| local_company | N/A (apenas recebe) |
| marketing | N/A (anti-prospecção proativa; apenas responde via Marketing Link) |

Reset: 1º de janeiro 00:00 America/Sao_Paulo (BIL-030).

---

# Sub-produto 3 — Reputação Bronze→Diamante

Fonte: `sub-produtos/03-reputacao.md` + `00-product/business-model.md`.

## 3.1 Ver badges

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ver badge próprio (empresa) | ✅ | ✅ | ✅ | ✅ | enterprise | ⊃ |
| Ver badge empresas terceiras | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Ver histórico de tier changes (própria empresa) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Ver score detalhado empresas terceiras | ❌ | ❌ | ❌ | ❌ | todos (apenas tier aparente) | ⊃ |
| Avaliar empresa pós-Connect Me (síndico) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

## 3.2 Suspensão por harassment

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Suspender empresa por harassment | n/a | n/a | n/a | n/a | — | ✅ (admin-only) |
| Reverter suspensão | n/a | n/a | n/a | n/a | — | ✅ (admin-only) |

Score de Governança: **2 scores** (D-103 Fase 12 revoga D-066/D-075):
- **Governança 1-10** (qualitativo, INV-GOV-001) — média ponderada de avaliações de gestão (mandato + assembleia + eleição)
- **Compliance 0-100** (quantitativo, INV-GOV-002) — % de itens C1-C11 em conformidade

Telas: [[../03-subprojects/web/ui-catalog/sindico/S32-score-governanca]] (web) + Mobile correlato.

---

# Sub-produto 4 — Conteúdo & Vídeos

Fonte: `sub-produtos/04-conteudo-videos.md` §5.1 + §5.2 + matriz.

## 4.1 Publicação de vídeos

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Publicar vídeo institucional | ⏳ | ❌ | 4/m | 12/m | syndic | ⊃ |
| Publicar vídeo institucional | ⏳ | ❌ | 8/m | 12/m | enterprise | ⊃ |
| Publicar vídeo-currículo (90s, lock 90d) | ❌ | ❌ | 1 fixo | 1 fixo | resident | ⊃ |
| Publicar vídeo via delegação (agência→empresa) | ⏳ | n/a | via empresa cliente | via empresa cliente | marketing | ⊃ |
| Publicar ebook | ⏳ | ❌ | ❌ | ✅ | enterprise | ⊃ |
| Criar curso/módulo LMS | ⏳ | ❌ | ❌ | ✅ | enterprise | ⊃ |
| Criar certificado LMS | ⏳ | ❌ | ❌ | ✅ | enterprise | ⊃ |
| Anexar vídeo a pauta assembleia | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

## 4.2 Consumo de conteúdos

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Busca geral | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Busca síndicos/empresas/comércio local | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Ver vídeos institucionais síndicos (ilimitado) | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Ver vídeos institucionais empresas (preview 25%) | 25% | 25% | 25% (consumindo) | ✅ integral | todos | ⊃ |
| Ver vídeos institucionais empresas (integral) | ❌ | ❌ | ✅ (resident plus) | ✅ | resident, syndic, enterprise | ⊃ |
| Ver módulos/cursos LMS | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise | ⊃ |
| Ver ebook | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise | ⊃ |
| Obter certificado (consumidor de curso) | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise | ⊃ |
| Ver lives ao vivo | ❌ | ✅ | ✅ | ✅ | resident, syndic, enterprise, marketing | ⊃ |
| Ver replay lives (30d) | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Ver podcast YouTube integrado | ❌ | ✅ | ✅ | ✅ | resident, syndic, enterprise, marketing | ⊃ |

## 4.3 Interação social (rede social cega)

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Curtir conteúdo | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Avaliar conteúdo (rating) | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Ver contadores de likes próprios | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Ver contadores de likes alheios | ❌ | ❌ | ❌ | ❌ | todos | ⊃ |
| Ver comentários próprios (dono) | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Ver comentários alheios (terceiros) | ❌ | ❌ | ❌ | ❌ | todos | ⊃ |
| Ler fórum | ❌ | ❌ | ✅ | ✅ | syndic, enterprise, marketing | ⊃ |
| Criar tópico fórum | ❌ | ❌ | ✅ | ✅ | syndic, enterprise, marketing | ⊃ |
| Comentar fórum | ❌ | ❌ | ✅ | ✅ | syndic, enterprise, marketing | ⊃ |

**Trava trimestral 90d** (GR-029): vídeo institucional síndico/empresa + vídeo-currículo.

---

# Sub-produto 5 — Assembleia Live

Fonte: `sub-produtos/05-assembleia-live.md` §5 + D-062 Fase 8 / D-070 Fase 7 (aditivo Live).

## 5.1 Pré-assembleia

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ver edital de convocação | ✅ | ✅ | ✅ | ✅ | resident, syndic | ⊃ |
| Criar edital/convocação | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Adicionar item de pauta | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Anexar vídeo/áudio a pauta | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Anexar vídeos de fornecedores | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Outorgar procuração | ✅ | ✅ | ✅ | ✅ | resident (outorgante) | ⊃ |
| Aceitar procuração | ✅ | ✅ | ✅ | ✅ | resident (procurador) | ⊃ |
| Responder enquete pré-assembleia | ✅ | ✅ | n/a | n/a | resident | ⊃ |
| Criar enquete pré-assembleia | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Confirmar participação (RSVP) | ✅ | ✅ | n/a | n/a | resident | ⊃ |
| Simulador de quórum | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

## 5.2 Assembleia ao vivo (WebSocket)

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Check-in (presencial/remoto/outorgado) | ✅ | ✅ | ✅ | ✅ | resident | ⊃ |
| Votar (fração ideal real-time) | ❌ | ✅ | ✅ | ✅ | resident (morador pagante vota; base também — ver §5.3) | ⊃ |
| Voto presencial assistido | ✅ | ✅ | n/a | n/a | resident | ⊃ |
| Pedir a palavra (fila de fala) | ✅ | ✅ | n/a | n/a | resident | ⊃ |
| Presidir assembleia (painel) | ⏳ | ❌ | ✅ | ✅ | syndic (principal designado) | ⊃ |
| Controlar fila de fala | ⏳ | ❌ | ✅ | ✅ | syndic (presidente) | ⊃ |
| Abrir/fechar votação | ⏳ | ❌ | ✅ | ✅ | syndic (presidente) | ⊃ |
| Ativar modo contingência | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Telão 2 áreas (painel compartilhado) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

## 5.3 Pós-assembleia

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Homologar ata | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Ler ata (morador) | ✅ | ✅ | n/a | n/a | resident | ⊃ |
| Gerar ata PDF | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Eleição gera avaliação escondida ativa | n/a | n/a | n/a | n/a | sistema (D-065/D-074) | auditável admin |

**Nota voto morador**: contrato prevê voto por fração ideal independente de tier (morador Base vota em assembleia legal — direito civil inalienável); morador Plus tem features **adicionais** (histórico, export).

---

# Sub-produto 6 — LMS / Academia

Fonte: `sub-produtos/06-lms-academia.md` §6 + **D-097 Fase 11** (Lives: Admin MS + Empresa Pro) — sobrescreve D-063/D-076 que diziam admin-only MVP.

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ver catálogo de cursos | ✅ | ✅ | ✅ | ✅ | todos | ⊃ |
| Matricular em curso | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise, marketing | ⊃ |
| Assistir aulas | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise, marketing | ⊃ |
| Responder quiz | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise, marketing | ⊃ |
| Receber certificado ao concluir curso | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise | ⊃ |
| Ver biblioteca de materiais | ❌ | ❌ | ✅ | ✅ | resident, syndic, enterprise, marketing | ⊃ |
| Ler fórum LMS | ❌ | ❌ | ✅ | ✅ | syndic, enterprise, marketing | ⊃ |
| Criar curso/módulo | ⏳ | ❌ | ❌ | ✅ | enterprise, marketing (via delegação) | ⊃ |
| Criar quiz | ⏳ | ❌ | ❌ | ✅ | enterprise, marketing (via delegação) | ⊃ |
| Emitir certificado (criador do curso) | ⏳ | ❌ | ❌ | ✅ | enterprise | ⊃ |
| Publicar material biblioteca | ⏳ | ❌ | ❌ | ✅ | enterprise | ⊃ |
| **Criar live** | ❌ | ❌ | ❌ | ✅ (enterprise) | enterprise (tier=pro) + marketing (via delegação empresa pro) (**D-097 Fase 11** sobrescreve D-063/D-076) | ✅ (admin também) |
| Assistir live (quando admin inicia) | ❌ | ✅ | ✅ | ✅ | resident, syndic, enterprise, marketing | ⊃ |

**Parceiro (local_company)** não tem acesso LMS (D-063 sub-produtos).

---

# Sub-produto 7 — Banco de Talentos

Fonte: `sub-produtos/07-banco-de-talentos.md` §5 + **D-099 Fase 11** (5 vínculos — sobrescreve D-060/D-064/D-074 que diziam 3). PDF original do cliente "ESTRUTURA DE CADASTRO DE CURRÍCULO" TELA 08 canônico.

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Cadastrar currículo (perfil profissional) | ❌ | ❌ | ✅ | ✅ | resident | ⊃ |
| Upload vídeo-currículo 90s | ❌ | ❌ | ✅ (lock 90d) | ✅ (lock 90d) | resident | ⊃ |
| Declarar 5 vínculos profissionais (D-099) | ❌ | ❌ | ✅ | ✅ | resident | ⊃ |
| Editar áreas de atuação (até 18) | ❌ | ❌ | ✅ | ✅ | resident | ⊃ |
| Declarar CNH/NR/certificações | ❌ | ❌ | ✅ | ✅ | resident | ⊃ |
| Buscar currículos | ⏳ | ❌ | ✅ | ✅ | enterprise, marketing, local_company | ⊃ |
| Filtrar currículos (área/região/palavra) | ⏳ | ❌ | ✅ | ✅ | enterprise, marketing, local_company | ⊃ |
| Favoritar currículo | ⏳ | ❌ | ✅ | ✅ | enterprise, marketing, local_company | ⊃ |
| Demonstrar interesse | ⏳ | ❌ | ✅ | ✅ | enterprise, marketing, local_company | ⊃ |
| Ver vídeo-currículo integral | ⏳ | ❌ | ✅ | ✅ | enterprise, marketing, local_company | ⊃ |
| Tag privado de currículo (lado empresa) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |

**Nota**: síndico (`role=syndic`) **não** acessa Banco de Talentos — Banco é relação B2B (empresa busca morador) + parceiro local (`local_company`).

---

# Sub-produto 8 — Vizinhança / Clube de Benefícios

Fonte: `sub-produtos/08-vizinhanca.md` §6 + D-059 Fase 8 (cupons ativos).

## 8.1 Morador (consumidor)

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ver feed benefícios locais | ❌ | ❌ | ✅ | ✅ | resident | ⊃ |
| Ativar promoção do dia (gerar cupom) | ❌ | ❌ | 1/4h por parceiro | 1/4h por parceiro | resident | ⊃ |
| Buscar palavra-chave do dia | ❌ | ❌ | ✅ | ✅ | resident | ⊃ |
| Redimir cupom em estabelecimento | ❌ | ❌ | ✅ | ✅ | resident | ⊃ |

## 8.2 Parceiro (local_company)

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Cadastrar perfil parceiro | ⏳ | ✅ | ✅ | ✅ | local_company | ⊃ |
| Publicar cupom Vizinhança | ⏳ | ✅ | ✅ | ✅ | local_company | ⊃ |
| Definir palavra-chave do dia | ⏳ | ❌ | ❌ | ✅ | local_company (só tier pro) | ⊃ |
| Criar benefício exclusivo condomínio | ⏳ | ❌ | ✅ | ✅ | local_company (convite síndico) | ⊃ |
| Ver métricas cupons (dono) | ⏳ | ✅ | ✅ | ✅ | local_company | ⊃ |
| Criar campanha sazonal | ⏳ | ❌ | ✅ | ✅ | local_company | ⊃ |
| Receber Connect Me de síndico | ✅ | ✅ | ✅ | ✅ | local_company | ⊃ |

## 8.3 Síndico (curador)

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ver feed parceiros | ✅ | ✅ | ✅ | ✅ | syndic | ⊃ |
| Criar benefício exclusivo condomínio (convidar parceiro) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Conceder Seal "Síndico-aprovado" | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| Revogar Seal | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |

**Cooldown cupom**: 4h por morador por parceiro (BIL-034 / D-059 Fase 8).

---

# Sub-produto 9 — Compliance

Fonte: `sub-produtos/09-compliance.md` §7 (tiers × role) + C1-C11.

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| C1 — Termo de responsabilidade (assinar) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| C2 — Declaração anual obrigatória | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| C3 — Ver auditoria do condomínio (principal) | ⏳ | ❌ | ✅ | ✅ | syndic principal | ⊃ |
| C4 — Auditoria total cross-tenant | n/a | n/a | n/a | n/a | — | ✅ (admin-only) |
| C5 — Declarar conflito de interesse | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| C6 — Criar adendo (registro bloqueado reaberto) | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| C7 — Ver score Governança próprio | ✅ | ✅ | ✅ | ✅ | syndic | ⊃ |
| C7 — Ver score Governança condomínio (morador) | ✅ | ✅ | n/a | n/a | resident | ⊃ |
| C8 — Dossiê PDF fim de mandato | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| C9 — Ver LGPD logs próprios | ✅ | ✅ | ✅ | ✅ | todos (próprio user_id) | ⊃ |
| C10 — Risk Map 8×4 | ⏳ | ❌ | ✅ | ✅ | syndic | ⊃ |
| C11 — Risk Assessment (avaliação de risco) | ⏳ | ❌ | ❌ | ✅ | syndic | ⊃ |
| Moderação compliance (reverter validação) | n/a | n/a | n/a | n/a | — | ✅ (admin-only) |

**Score Governança**: **2 scores separados** (D-103 Fase 12 revoga D-066/D-075):
- **Governança 1-10** (INV-GOV-001) — qualitativo, ponderado por avaliações
- **Compliance 0-100** (INV-GOV-002) — quantitativo, % itens C1-C11 conformes

---

# Sub-produto 10 — Marketing / Agências

Fonte: `sub-produtos/10-marketing-agencias.md` §5 + D-060 Fase 8 / D-073 Fase 7 (sub-produto interno).

## 10.1 Empresa (cliente da agência)

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Delegar para agência (criar grant ABAC) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Revogar delegação imediatamente | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Ver audit trail da delegação | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Editar scope de permissões delegadas | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Marketing Link (solicitar apoio agência) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |

## 10.2 Agência (role=marketing)

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Login próprio via Zitadel | ✅ | ✅ | ✅ | ✅ | marketing | ⊃ |
| Listar empresas clientes (grants ativos) | ⏳ | ❌ | ✅ | ✅ | marketing | ⊃ |
| Selecionar empresa (ativar scoped session) | ⏳ | ❌ | ✅ | ✅ | marketing | ⊃ |
| Publicar vídeo em nome de empresa cliente | ⏳ | ❌ | via quota empresa | via quota empresa | marketing (via grant) | ⊃ |
| Gerenciar biblioteca vídeos empresa | ⏳ | ❌ | ✅ (se grant permite) | ✅ | marketing | ⊃ |
| Criar curso LMS em nome de empresa | ⏳ | ❌ | ❌ | ✅ (se empresa tier=pro) | marketing | ⊃ |
| Acessar dados financeiros empresa (BILLING) | ❌ | ❌ | ❌ | ❌ | marketing **NUNCA** | — |
| Acessar Connect Me da empresa | ❌ | ❌ | ❌ | ❌ | marketing **NUNCA** | — |
| Acessar Banco de Talentos (currículos favoritados empresa) | ❌ | ❌ | ❌ | ❌ | marketing **NUNCA** | — |
| Iniciar Connect Me proativo | ❌ | ❌ | ❌ | ❌ | marketing (anti-prospecção) | — |
| Responder Marketing Link (intenção contato) | ⏳ | ❌ | ✅ | ✅ | marketing | ⊃ |

**Modelo de delegação**: ABAC grants scoped por empresa (D-060 Fase 8 / D-073 Fase 7 — não token impersonation). Agência é persona de primeira classe com login próprio.

---

# Sub-produto 11 — Administradoras Condominiais

Fonte: `sub-produtos/11-administradoras-condominiais.md` §6 + `EmpresaSindicaLink`.

| Feature | Trial | Base | Plus | Pro | Role | Admin |
|---|---|---|---|---|---|---|
| Ser vinculada como administradora do condomínio | ✅ | ✅ | ✅ | ✅ | enterprise | ⊃ |
| Visualizar publicações do condomínio | ⏳ | ❌ | ✅ | ✅ | enterprise (via EmpresaSindicaLink) | ⊃ |
| Editar publicações do síndico (se permission=true) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Ocultar publicações (se permission=true) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Validar registros de empresas (se permission=true) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Validar comunicados de empresas (se permission=true) | ⏳ | ❌ | ✅ | ✅ | enterprise | ⊃ |
| Assinar compliance no lugar do síndico | ❌ | ❌ | ❌ | ❌ | enterprise (NUNCA — síndico é PF responsável) | — |
| Encerrar mandato do síndico | ❌ | ❌ | ❌ | ❌ | enterprise (NUNCA) | — |

**Permissões configuráveis** (5 flags bool em `EmpresaSindicaPermissions` — sub-produto 11 §7.1): view, edit, hide, validate_execution, validate_communication.

---

# Transversal — Admin Master Síndico

Fonte: D-134 Fase H (revoga D-058/D-061/D-072) — admin = `apps/admin` dedicada no monorepo web (Cloudflare Access SSO+MFA + 4-eyes em ações críticas).

| Feature | Admin |
|---|---|
| Bypass cross-tenant (ver qualquer condomínio/empresa) | ✅ |
| Moderação: banir usuário | ✅ |
| Moderação: suspender conteúdo (vídeo, comentário, post) | ✅ |
| Moderação: reverter validação (cascade) | ✅ |
| Broadcast email/SMS segmentado | ✅ |
| Aprovação manual CNPJ | ✅ |
| Gestão de planos (CRUD) + seed matriz | ✅ (com 4-eyes em price change) |
| Override quota manual | ✅ |
| Refunds Stripe | ✅ (com 4-eyes) |
| Dashboard financeiro (MRR, churn, conversion) | ✅ |
| Dashboard analytics (top empresas/síndicos) | ✅ |
| Controle inadimplência | ✅ |
| **Iniciar live RTMP** (**único papel com essa capacidade**) | ✅ (D-063/D-076) |
| Aprovar/revogar delegação agência marketing | ✅ |
| Pausar/despausar subscription | ✅ |
| Audit trail completo cross-tenant | ✅ |

**Sub-papéis operacionais admin** (scoped permissions via ABAC — não são roles Zitadel separadas):

- `admin.super` — bypass total
- `admin.moderator` — só moderação/suspensão
- `admin.support` — só read + refund request (precisa 4-eyes)
- `admin.finance` — só billing/refunds

---

# Quotas consolidadas (referência cruzada billing.md)

## Connect Me (reset 1º Jan)

| Role + tier | Quota/ano |
|---|---|
| resident, base | **2/a** (D-094 Fase 11 — sobrescreve D-058) |
| resident, plus | 4/a |
| syndic, trial | ⏳ 0 |
| syndic, base | ❌ |
| syndic, plus | 4/a |
| syndic, pro | ∞ |
| enterprise, plus | ∞ (só responde) |
| enterprise, pro | ∞ (abre + responde) |
| local_company | N/A (apenas recebe) |
| marketing | N/A (anti-prospecção) |

## Vídeos institucionais (reset 1º do mês)

| Role + tier | Uploads/mês |
|---|---|
| syndic, trial | ⏳ 0 |
| syndic, base | 0 |
| syndic, plus | 4/m |
| syndic, pro | 12/m |
| enterprise, plus | 8/m |
| enterprise, pro | 12/m |
| resident | 0 (só vídeo-currículo) |
| marketing | via quota empresa |
| local_company | 0 (publica cupom, não vídeo institucional) |
| admin | ∞ |

## Vídeo-currículo

| Role + tier | Quota |
|---|---|
| resident, plus | 1 fixo, substituível após 90d |

## Condomínios por síndico

| Role + tier | Limite |
|---|---|
| syndic, base | 1 |
| syndic, plus | 15 |
| syndic, pro | 15 |
| admin override manual | ilimitado |

## Cupons Vizinhança

| Ator | Limite |
|---|---|
| resident plus+ (por parceiro) | 1 a cada 4h (cooldown) |

## Lives (D-063 Fase 8)

| Role | Capacidade de **criar** live |
|---|---|
| admin | ∞ |
| todos os outros | 0 (independe de tier) |

---

# Trials (persona-aware)

Duração definida em `plans.trial_days` + mapping `role → trial_plan` (BIL-008).

| Role | Duração trial | Comportamento |
|---|---|---|
| resident | — | Sem trial; inicia em tier=base (grátis) |
| syndic | 15 dias | Bloqueia criar timeline, comunicado, export relatório, criar condomínio, publicar vídeo |
| enterprise | 7 dias | Bloqueia publicar vídeo, registrar execução, criar curso, gerenciar delegações |
| local_company | 30 dias | Bloqueia criar promoção/cupom, ver métricas, criar campanha |
| marketing | 7 dias (ou compartilha trial da primeira empresa cliente) | Bloqueia publicar vídeo em nome de empresa, criar curso |
| admin | — | Não tem trial (provisionado manualmente) |

**Soft-block pós-trial** (BIL-011, sem grace period): dados intactos; ações premium → 402 `PLAN_REQUIRED`.

## Matriz de Trial detalhada (absorvido de `MATRIZ FUNCIONAL MÓDULO TRIAL.pdf` 2026-03-09)

Reconciliação Fase F: PDF do cliente lista features liberadas/bloqueadas durante trial **por persona**. Tabela abaixo é a leitura canonizada (com tradução N1/N2/N3 → tier e Morador Pagante removido como label).

### Trial Síndico (15 dias)

**Objetivo (PDF)**: Demonstrar como a plataforma organiza a gestão do condomínio e conecta o síndico ao ecossistema de fornecedores e moradores.

| Feature | Trial syndic | Pós-trial (vira `base` ou paga `plus+`) | Observação |
|---|---|---|---|
| Criar perfil de síndico | ✅ | ✅ | onboarding obrigatório |
| Criar perfil institucional do síndico | ⏳ visível | ❌ base / ✅ plus+ | mini-bio + foto + vídeo (vídeo só plus+) |
| Cadastrar 1 condomínio (soft-block) | ⏳ (1 provisional) | ✅ base 1 / plus+ até 15 | BIL-032 |
| Cadastrar Plano Diretor | ⏳ (cria/lê draft) | ❌ base / ✅ plus+ | publicação só plus+ |
| Assistir vídeos de síndicos | ✅ | ✅ todos tiers | ilimitado |
| Assistir vídeos de empresas | ✅ preview 25% | ✅ preview / integral plus+ | gate Conteúdo §4.2 |
| Buscar empresas no mercado | ✅ | ✅ | livre |
| Buscar comércio local | ✅ | ✅ | livre |
| Criar Connect Me com empresas | ⏳ 0 | base 2/a → plus 4/a → pro ∞ | D-094 |
| Criar perguntas aos moradores (enquetes) | ⏳ visível | ✅ todas tiers (criar enquete interna) | spec §1.4 |
| Acessar dashboard da gestão | ✅ leitura | ✅ | métricas plenas plus+ |
| Visualizar módulo assembleia | ✅ | ✅ | leitura ata em todos os tiers |
| Criar atividades na Linha do Tempo | ⏳ | ❌ base / ✅ plus+ | hard paywall §1.2 |
| Criar comunicados (oficiais) | ⏳ | ❌ base / ✅ plus+ | hard paywall §1.4 |
| Criar eventos | ⏳ | ✅ todas tiers | spec §1.4 |
| Exportar relatório de gestão (PDF Timeline) | ⏳ | ❌ base / ✅ plus+ | spec §1.2 |
| Cadastrar parceiro da vizinhança (curador/seal) | ⏳ | ❌ base / ✅ plus+ | spec §8.3 |
| Validar registros de execução (empresa) | ⏳ | ✅ todas tiers | spec Síndico §1.3 |
| Acessar módulo Compliance (assinar termo / declaração) | ⏳ | ❌ base / ✅ plus+ | spec §C1-C11 |
| Cadastrar múltiplos condomínios | ❌ | ❌ base / ✅ plus+ até 15 | BIL-032 |

**Regra do trial síndico (PDF, ratificada)**: o síndico explora toda a plataforma e interage com empresas, mas **não pode executar publicações oficiais da gestão** (timeline, comunicados, exports, validações compliance, cadastro múltiplos condomínios).

### Trial Empresa Prestadora (7 dias — confirmado pelo PDF; legado mencionava 30d, descartado)

**Objetivo (PDF)**: Permitir que a empresa compreenda o potencial da plataforma para geração de negócios e posicionamento institucional no mercado condominial.

| Feature | Trial enterprise | Pós-trial (`plus`/`pro`) | Observação |
|---|---|---|---|
| Criar perfil institucional da empresa | ✅ | ✅ | cadastro CNPJ obrigatório |
| Cadastrar segmento e subcategoria (31 cat / ~180 subcat) | ✅ | ✅ | enum canônico |
| Adicionar logo e descrição | ✅ | ✅ | livre |
| Visualizar perfil público | ✅ | ✅ todos | gate visibilidade §1.x |
| Assistir vídeos de síndicos | ✅ | ✅ | livre |
| Assistir vídeos de empresas | ✅ | ✅ | concorrentes vêem-se (legado §1.6) |
| Buscar síndicos no mercado | ✅ | ✅ | livre |
| Buscar empresas no mercado | ✅ | ✅ | livre |
| Acessar Marketing Link das agências | ✅ | ✅ plus+ | spec §10.1 |
| Acessar dashboard empresarial | ✅ leitura | ✅ pleno plus+ | métricas plus+ |
| Receber Connect Me de síndicos | ✅ | ✅ plus / pro | spec §2.1 |
| Publicar vídeos institucionais | ⏳ | 8/m plus / 12/m pro | spec §4.1 + GR-029 trava 90d |
| Registrar execução de serviço | ⏳ | ⏳ plus / ✅ pro | hard paywall pro |
| Enviar atividades técnicas | ⏳ | ✅ plus+ | spec §3.3 |
| Enviar comunicados técnicos | ⏳ | ✅ plus+ | spec §3.3 |
| Gerenciar usuários da empresa (operadores) | ⏳ | ⏳ plus / ✅ pro | spec §3.3 |
| Convidar agência de marketing (delegação) | ⏳ | ✅ plus+ | spec §10.1 |

**Regra do trial empresa (PDF, ratificada)**: a empresa explora o painel empresarial completo, mas **não pode executar ações operacionais ou publicar conteúdos** (vídeos, registros de execução, gestão de usuários, delegações ativas).

### Trial Parceiro da Vizinhança (30 dias)

**Objetivo (PDF)**: Demonstrar o potencial de divulgação local para comércios e prestadores de serviço da região do condomínio.

| Feature | Trial local_company | Pós-trial (`plus`/`pro`) | Observação |
|---|---|---|---|
| Criar perfil do estabelecimento | ✅ | ✅ | self-service ou convite síndico |
| Cadastrar endereço e CEP | ✅ | ✅ | base do raio hyperlocal |
| Adicionar logo e descrição | ✅ | ✅ | livre |
| Visualizar condomínios da região | ✅ | ✅ | leitura geográfica |
| Visualizar promoções locais (próprias e do feed) | ✅ leitura | ✅ pleno | métrica detalhada plus+ |
| Criar promoção / cupom | ⏳ | ✅ plus+ | sistema cupons §8.2 |
| Criar palavra-chave da promoção | ⏳ | ❌ plus / ✅ pro | spec §8.2 |
| Acessar métricas de promoção | ⏳ | ✅ plus+ | spec §8.2 |
| Criar promoções exclusivas para condomínio | ⏳ | ✅ plus+ (convite síndico) | spec §8.2 |
| Participar de campanhas locais | ⏳ | ✅ plus+ | spec §8.2 |

**Regra do trial parceiro (PDF, ratificada)**: o parceiro vê todo o ecossistema mas **não pode emitir cupom, palavra-chave, métricas ou campanha** durante trial — alinhado com regra de upsell BIL-014.

### Trial — outras personas

- **Morador (`resident`)**: **sem trial** (PDF não cobre — confirma personas-and-plans). Acesso via vínculo do síndico em tier `base` gratuito; upgrade direto para `plus`/`pro` (sem trial).
- **Agência Marketing (`marketing`)**: PDF não cobre. Conforme D-060/personas: opera sob trial da empresa cliente que delegou; quando empresa em trial, agência também — sem trial próprio no MVP.
- **Admin (`admin`)**: provisionado manualmente; sem trial.

### Regras gerais do Trial (PDF, canonizadas)

1. **Visibilidade total**: TODAS as funcionalidades da plataforma são VISÍVEIS durante trial (não há ocultação por feature flag).
2. **Bloqueio estratégico**: ações de execução crítica permanecem bloqueadas (criar timeline oficial, publicar vídeo, criar promoção, validar execução).
3. **Mensagem de bloqueio padrão (PDF, EN i18n traduzida)**: *"Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico. Ative seu plano para desbloquear esta função."* — código de erro `402 PAYMENT_REQUIRED, code=trial_feature_blocked` (BIL-009).
4. **Mensagens contextuais de upgrade** (PDF):
   - Síndico: *"Você já conectou empresas ao seu condomínio. Com o plano completo você poderá registrar atividades da gestão e gerar assembleias digitais."*
   - Empresa: *"Seu perfil institucional está ativo. No plano completo sua empresa poderá registrar execuções de serviços e construir reputação técnica."*
   - Parceiro: *"Sua promoção já pode ser encontrada por moradores da região. Com o plano completo você poderá criar campanhas exclusivas para condomínios."*
5. **Soft-block sem grace period** (D-058 confirmado / BIL-011): trial expira → bloqueio imediato; tolerância 3-5 dias só em PIX/boleto pendente via Stripe Smart Retries (BIL-013).

---

# Hard Paywalls (tier mínimo por ação)

| Feature | Tier mínimo | Role |
|---|---|---|
| Votar em assembleia | base | resident (direito civil garantido) |
| Criar comunicado oficial | plus | syndic |
| Criar atividade Timeline | plus | syndic |
| Exportar relatório/PDF Timeline | plus | syndic |
| Validar execução empresa | plus | syndic |
| Publicar vídeo institucional | plus | syndic, enterprise |
| Publicar vídeo-currículo | plus | resident |
| Abrir Connect Me → Empresa | plus | syndic |
| Abrir Connect Me E→E | pro | enterprise |
| Consumir certificado LMS | plus | resident, syndic, enterprise |
| Criar curso LMS (publisher) | pro | enterprise, marketing (delegado) |
| Ler fórum | plus | syndic, enterprise, marketing |
| Ver clube benefícios (feed) | plus | resident |
| Buscar currículos (Banco Talentos) | plus | enterprise, local_company, marketing |
| Criar benefício exclusivo condomínio | plus | syndic |
| Presidir assembleia (painel) | plus | syndic principal |
| Criar live (RTMP) | — (irrelevante) | **admin apenas** (D-063) |
| Broadcast email/SMS plataforma | — | **admin apenas** |
| Delegar para agência | plus | enterprise |
| Definir palavra-chave do dia Vizinhança | pro | local_company |

---

# Observações globais (contrato + STATE)

1. **Backend como verdade única** (`novo escopo-7.md` §5.1): frontend reflete API; ABAC decide.
2. **Anti-prospecção**: empresas e agências não listam moradores para mensagens em massa (`novo escopo-7.md` §5.5; §1.6).
3. **Uploads trimestrais**: 90d hard-coded (GR-029); exceções via admin manual.
4. **Lives = admin + Empresa Pro + agência delegada** (D-097 Fase 11 revoga D-063/D-076 admin-only): primeira live = apresentação da plataforma.
5. **Feedback privado** (§5.8): métricas só ao dono (rede social cega).
6. **1 dispositivo por conta** (IDN-010 / ADR-0014).
7. **Device fingerprint progressivo** em signup (D-135 Fase G revoga "1 IP 1 cadastro" de Req 102 §1 — famílias compartilham wifi do condomínio): silent log → CAPTCHA Turnstile → revisão manual. Spec em IDN-026.
8. **Admin bypass**: role=admin curto-circuita matriz, exceto em ações marcadas `admin_scoped=false` (ex: "assinar termo de responsabilidade do síndico" não pode ser por admin).
9. **Slugs por persona abolidos** (D-066/D-067 Fase 8): refs antigas traduzidas via tabela §"Tradução histórica".
10. **Plano preservado em mudança de role** (ADR-0032): morador plus eleito síndico mantém plano; features premium destravam se tier cobre role nova.

---

# Mapeamento ABAC engine (Go)

```go
// Criar comunicado oficial
action := "institutional.communication.create"
resource := "communication"
// matrix lookup: (role=syndic, tier ∈ {plus, pro}, action) → allow + plan.features["institutional.announcement.create"]=true

// Votar em assembleia
action := "assembly.vote.cast"
resource := "vote"
// matrix lookup: (role=resident, tier ∈ {base, plus}, action) → allow + quota n/a (voto é direito)

// Publicar vídeo institucional
action := "content.video.publish.institutional"
resource := "video"
// matrix: (role=syndic, tier=plus) → allow + quota 4/mês
// matrix: (role=syndic, tier=pro) → allow + quota 12/mês
// matrix: (role=enterprise, tier=plus) → allow + quota 8/mês
// matrix: (role=enterprise, tier=pro) → allow + quota 12/mês

// Criar live
action := "content.live.start"
resource := "live"
// matrix: (role=admin) → allow; todos outros → deny (D-063)

// Abrir Connect Me → Empresa
action := "commercial.connect_me.to_company"
resource := "connect_me_request"
// matrix: (role=syndic, tier=plus) → allow + quota 4/a
// matrix: (role=syndic, tier=pro) → allow + quota ∞

// Delegar para agência
action := "marketing.delegation.grant_access"
resource := "marketing_agency_link"
// matrix: (role=enterprise, tier ∈ {plus, pro}) → allow

// Admin bypass
// role=admin → curto-circuita matriz; allow para todas ações onde admin_scoped != false
```

---

# Seed da matriz (referência)

Arquivo de seed `seed_abac_matrix_v2.sql` deve popular `abac_matrix_entries {role, plan_tier, action, resource, allowed, quota_key, quota_limit, trial_allowed, admin_scoped}`.

Estimativa: ~300 entries (60 features × 4 tiers × avg 1.25 roles aplicáveis).

---

# Deltas Fase F (PDFs Matrizes 2026-03-09 vs canônico)

Reconciliação Fase F (2026-04-25) — comparação `Matriz Funcional GERAL.pdf` + `MATRIZ FUNCIONAL MÓDULO TRIAL.pdf` (cliente, 09/03/2026) versus matriz canônica pós-Fase 11.

| # | Feature/Cota | PDF (legado N1/N2/N3) | Canônico atual (D-094..D-126) | Resolução |
|---|---|---|---|---|
| Δ1 | Connect Me Morador→Síndico (Base) | 2/ano | **2/ano** (D-094 sobrescreveu D-058 que era 0) | ✅ Alinhado (PDF original já dizia 2/4) |
| Δ2 | Connect Me Morador→Síndico (Pagante=Plus) | 4/ano | 4/ano | ✅ Alinhado |
| Δ3 | Vídeos institucionais Síndico Plus | 4/mês (PDF "8/mês 8/mês 12/mês" parece mistos) | 4/m plus / 12/m pro | ✅ Canônico vence (PDF ambíguo na coluna `Sín. N2`) |
| Δ4 | Vídeos institucionais Empresa | 8/m Plus / 12/m Pro | 8/m plus / 12/m pro | ✅ Alinhado |
| Δ5 | Lives — criação | "My Síndico" único criador (PDF) | **Admin + Empresa Pro + Agência delegada** (D-097/D-133) | ✅ Canônico vence (PDF é pré-D-097/2026-04-24); Lives institucionais (broadcast 1→N) são produto separado em `content` BC, não Assembly |
| Δ6 | "My Síndico" (nome do produto) | nome legado em todas as referências | **Master Síndico** | ✅ Canônico vence |
| Δ7 | Score Governança | PDF não detalha; menciona "Avaliação objetiva da gestão" | **2 scores** (Governança 1-10 + Compliance 0-100 — D-103) | ✅ Canônico vence |
| Δ8 | Banco Talentos vínculos | PDF não fixa numeral; ESTRUTURA CADASTRO PDF dizia 3 (legado) | **5** (D-099 sobrescreve D-064) | ✅ Canônico vence |
| Δ9 | Sub-produtos count | PDF cita features sem contar | **11 sub-produtos** (D-115); 12 com Reservas Espaços M3+ | ✅ Canônico vence |
| Δ10 | "Morador Pagante" como label | usado pelo PDF como coluna ("Morador Pg") | **Removido como label vivo** em UI/backend; só permanece em §"Tradução histórica" como rótulo histórico mapeando `(resident, plus)` | ✅ Canônico vence (D-067/D-096 abolem slugs; espírito da consolidação 2026-04-25) |
| Δ11 | Trial Empresa | **7 dias** (PDF MÓDULO TRIAL) | **7 dias** (business-model.md §5.1) | ✅ Alinhado (legado de 30d descartado) |
| Δ12 | Trial Síndico | **15 dias** (PDF) | **15 dias** | ✅ Alinhado |
| Δ13 | Trial Parceiro Vizinhança | **30 dias** (PDF) | **30 dias** | ✅ Alinhado |
| Δ14 | Conteúdo de Empresa Plus limitado | "2 fotos + 1 vídeo 60s" (PDF Perfil e Visibilidade) | **Não documentado canonicamente** — features adicionais Plus vs Pro de perfil empresa | ⚠️ **PEND-MATRIX-06** (novo): registrar limite específico de perfil Empresa Plus |
| Δ15 | "Quais síndicos veem perfil da empresa?" | linha do PDF (sem resposta no PDF) | **Não respondido canonicamente** — visibilidade de perfil empresa por tier do síndico observador | ⚠️ **PEND-MATRIX-04** (atualizada): destrinchar matriz de visibilidade quem-vê-quem |
| Δ16 | Link de votação de fornecedores (síndico) | listado em GESTÃO CONDOMINIAL | **Coberto** em §5.1 "Anexar vídeos de fornecedores" + supplier_vote_session (Assembly BC) | ✅ Alinhado |
| Δ17 | Link compartilhamento atividades/manutenções (com quem?) | linha do PDF sem resposta | **Não respondido canonicamente** — escopo do link público | ⚠️ **PEND-MATRIX-07** (novo): definir escopo do link de compartilhamento timeline (público/restrito-condomínio/oauth) |
| Δ18 | Plano de eleição/reeleição | feature listada no PDF | **Não documentado canonicamente como feature dedicada** — overlap com Plano Diretor + dossiê fim de mandato | ⚠️ **PEND-MATRIX-08** (novo): decidir se "Plano de Eleição/Reeleição" é feature distinta ou dimensão do Plano Diretor |
| Δ19 | Marketing Link (Empresa→Agência) | fluxo descrito no PDF como "Empresa clica Marketing Link da agência → envia formulário" | **Coberto** em §10.1 + sub-produto 10 — confirmado: agência **recebe** solicitação (anti-prospecção) | ✅ Alinhado |
| Δ20 | "1 dispositivo + 1 IP por cadastro" | obs final do PDF GERAL | **Coberto** em §obs globais (1-device ADR-0014 / 1-IP IDN-026) | ✅ Alinhado |
| Δ21 | Botão denunciar harassment | obs final do PDF | **Coberto** em §2.1 "Botão denunciar (harassment report)" | ✅ Alinhado |
| Δ22 | Vídeos de empresa não aparecem automaticamente para moradores | regra de governança PDF | **Coberto** em §4.2 + Rule 4 Req 103 (visíveis só em contexto votação fornecedor) | ✅ Alinhado |
| Δ23 | Mensagens upgrade contextuais por persona | PDF MÓDULO TRIAL §EXPERIÊNCIA DE CONVERSÃO | **Adicionado** em §"Regras gerais do Trial" item 4 (síndico/empresa/parceiro) | ✅ Absorvido na Fase F |

**Total**: 23 deltas analisados — 14 alinhados, 4 canônico vence (PDF é pré-Fase 11), 4 novas pendências (PEND-MATRIX-06/07/08 + atualização da 04).

**Observação geral**: PDFs do cliente (09/03/2026) precedem D-080..D-103 (Fase 11 + ADR-0032). Onde há divergência, **canônico vence** (decisões mais recentes). Apenas pendências nas linhas onde o PDF traz informação que ainda não foi formalizada no canônico (PEND-06/07/08).

---

# Pendências

- ⚠️ **PEND-MATRIX-01**: E→E — Plus "só responde, não abre" — confirmar com stakeholder se há edge cases que permitem Plus abrir em situações específicas.
- ⚠️ **PEND-MATRIX-02**: Agência Marketing — tier próprio vs derivado da empresa cliente — decidir se agência tem plano próprio ou compartilha tier cliente.
- ⚠️ **PEND-MATRIX-03**: Local company visibilidade Seal — distinção entre "parceiro aprovado" vs "parceiro comum" na UI.
- ⚠️ **PEND-MATRIX-04**: Matriz de visibilidade de perfis (quem vê quem) — destilar das linhas §Perfil e Visibilidade do legado → sub-produto dedicado. **Atualizado Fase F**: PDF GERAL pergunta explicitamente "Quais síndicos veem perfil da empresa?" sem resposta — destravar com stakeholder.
- ⚠️ **PEND-MATRIX-05**: Price exato por tier (Plus/Pro) — travado em BIL-003 pendência.
- ⚠️ **PEND-MATRIX-06** (novo Fase F): Limite de conteúdo do perfil Empresa Plus — PDF cita "2 fotos + 1 vídeo 60s" como restrição mas não está formalizado em sub-produto 03/04. Documentar limite ou descartar.
- ⚠️ **PEND-MATRIX-07** (novo Fase F): Escopo do "Link de compartilhamento de atividades/manutenções" da gestão (síndico) — público / restrito-condomínio / com OAuth? PDF deixa em aberto ("Com quem o link será compartilhado?").
- ⚠️ **PEND-MATRIX-08** (novo Fase F): "Plano de Eleição/Reeleição" — é feature distinta ou dimensão do Plano Diretor? PDF lista como linha separada em GESTÃO CONDOMINIAL.

---

# Referências

- **ADR-0032** `02-architecture/adr/0032-global-plans-stripe-style.md` — fonte canônica do modelo
- **[[functional/billing]]** — BIL-001..BIL-055 (quotas, webhooks, ABAC lookup order)
- **[[../00-product/sub-produtos/_moc]]** — 11 sub-produtos (features catalogadas)
- **STATE** D-056..D-069 (Fase 8): D-057 planos globais; ~~D-058 morador base 0/a~~ (**sobrescrito por D-094 Fase 11 = 2/a**); ~~D-063 lives admin~~ (**sobrescrito por D-097 Fase 11 = Admin + Empresa Pro**); ~~D-064 BT 3 vínculos~~ (**sobrescrito por D-099 Fase 11 = 5 vínculos**); D-065 avaliação eleição; D-066 score único; D-067 abolir N1/N2/N3 (materializado por D-096 Fase 11)
- **STATE** D-094..D-102 (Fase 11, 2026-04-24): D-094 Morador Base 2/a (fecha Q2); D-095 Agência tem login (fecha Q4); D-096 materializa abolição N1/N2/N3 (fecha Q5 + DT-004); D-097 Lives Admin + Empresa Pro (fecha Q28); D-098 confirma 15 condomínios (fecha Q29); D-099 5 vínculos (fecha Q39); D-100 email Resend + {SendGrid|SES} M3 (fecha D-046); D-101 soft_delete universal + pseudonimização (fecha A-DC-SEC-004 + LGPD-M1-007); D-102 desambigua painéis admin (agência vs plataforma)
- **STATE** D-070..D-076 (Fase 7): D-071 agnosticismo; D-072 admin role; D-073 agência sub-produto; D-074 BT 3; D-075 score único; D-076 lives admin
- **Backend canônico** `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` v1.0 — **a reescrever** (DT-004)
- [[functional/_moc]] — requirements funcionais
- [[global]] — GR-002 (ABAC), GR-027 (trial), GR-032 (money)
- [[registration-data]] — dados por persona
- [[_moc]]
- **Fonte legada absorvida Fase F (2026-04-25)**:
  - `60-sources/master-sindico-research/client-material/pdfs/2026-03-09-matriz-funcional-geral.pdf` — Matriz Funcional GERAL (CONSUMO + COMUNIDADE + PUBLICAÇÃO + PERFIL + CONNECT ME + GESTÃO + CURRÍCULOS + CLUBE BENEFÍCIOS); usa N1/N2/N3 + "Morador Pg" — traduzido em §Tradução histórica.
  - `60-sources/master-sindico-research/client-material/pdfs/2026-03-09-matriz-funcional-trial.pdf` — Matriz Trial (Síndico 15d / Empresa 7d / Parceiro 30d) — absorvido em §"Matriz de Trial detalhada".
