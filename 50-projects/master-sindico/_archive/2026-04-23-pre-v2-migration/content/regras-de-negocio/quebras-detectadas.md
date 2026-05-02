---
title: "Quebras de Regra Detectadas — Cross-Doc Audit"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: audit
status: open
---

# Quebras de Regra Detectadas — Cross-Doc Audit (2026-04-22)

Auditoria cruzada entre `Content/`, `Content/contents/`, `backend/.kiro/specs/master-sindico/requirements/`, `backend/.kiro/STATE.md`, `backend/.kiro/AUDIT.md`, `backend/CLAUDE.md`, `web/CLAUDE.md`, `app/CLAUDE.md` e memórias persistentes.

**Severidade**: 🔴 critical (regra de negócio rompida ou ambiguidade pode gerar bug em prod) · 🟡 medium (drift documental sem impacto imediato em código) · 🟢 low (cosmético / desatualização sem risco).

**Como ler**: cada item lista *fontes em conflito*, *raiz*, *fix proposto*, *quem deve resolver*. Status `🔴 Aberto` permanece até alguém aplicar o fix.

> **Regra de prioridade**: quando spec viva (`backend/.kiro/specs/master-sindico/`) divergir de qualquer outra fonte, a spec viva vence — atualizar o conflitante. Quando o **código** divergir da spec viva, ler o código e atualizar a spec.

---

## 🔴 CRITICAL

### Q1 — Stack desatualizada em `product-overview.md` (3 menções stale)

- **Status**: 🔴 Aberto
- **Fontes em conflito**:
  - `backend/.kiro/specs/master-sindico/requirements/product-overview.md:36-43` — menciona "AWS SES (`IEmailProvider`)", "OpenSearch (Sprint 5)", "Infra: AWS (ECS Fargate, RDS, ElastiCache, S3, SES, CloudFront, Route53, CloudWatch) + Railway (alternativa/complementar)"
  - `web/CLAUDE.md`, `backend/CLAUDE.md`, `backend/.kiro/SESSION_CHARTER.md` — confirmam **Resend** (não SES), **PostgreSQL tsvector** (não OpenSearch), **Railway primário** (não AWS ECS)
  - `backend/.kiro/AUDIT.md` F30 — entregou 7 templates pt-BR via Resend
  - Memória `project_railway_url.md` — Railway é stack canônica (project ID efe664b7-…)
- **Raiz**: `product-overview.md` foi escrito antes da decisão de migração de cloud (anterior a 2026-04-21). Sprints 0-9 corrigiram `web/AGENTS.md`, `web/CLAUDE.md`, `backend/CLAUDE.md`, mas o `product-overview.md` recortado não foi atualizado.
- **Impacto**: agentes futuros que lerem **só** o product-overview vão considerar provedores errados. Pode gerar PRs com adaptador de SES ou queries de OpenSearch.
- **Fix proposto**: substituir as 3 menções:
  - Linha 36 → "Email: **Resend** (`IEmailProvider`)"
  - Linha 38 → "Busca: **PostgreSQL `tsvector`** (Sprint 5 — descartado OpenSearch por simplicidade operacional)"
  - Linha 43 → "Infra: **Railway** (primário) — plano AWS futuro registrado em `design/deploy-config.md`"
- **Quem resolve**: agente 1 (executor) ou principal — mudança documental, sem código. Bumpar `version: 1.1`.

---

### Q2 — Connect Me Morador→Síndico: quota canônica vs functional-matrix divergem para Base

- **Status**: 🟢 RESOLVIDO 2026-04-24 — Morador Base = 2/ano (via D-094 no STATE; confirma Decision 10). Matriz funcional atualizada na Fase 11.
- **Fontes em conflito**:
  - `requirements/personas-and-plans.md:80` → "Morador Base | 2/ano"
  - `requirements/commercial.md` Req 19.1 → "Quotas: Base 2/ano, Pagante 4/ano"
  - `requirements/cross-domain.md` Decision 10 → "N1: 2/ano · N2: 4/ano · N3: ilimitado · Base: 2/ano · Pagante: 4/ano"
  - `requirements/functional-matrix.md:43` → "**Connect Me Enviar síndico** | Base ❌ | Pagante 2/ano | N1 ❌ | N2 2/ano | N3 4/ano | Plus ❌ | Pro ✅"
  - `requirements/functional-matrix.md:152` (Quotas Connect Me Anual) → "Morador Base | 0 | Morador Pagante | 4/ano | Sind. N1 | 2/ano | N2 | 4/ano | N3 | ∞"
- **Raiz**: dois documentos canônicos discordam sobre quota do Morador Base. `personas-and-plans` + `commercial` + Decision 10 dizem **2/ano**; `functional-matrix` diz **0** (Base sem Connect Me) e **2/ano para Pagante** (que conflita com Pagante = 4/ano nas outras três fontes).
- **Impacto**: quando ABAC for codificar a quota, vai escolher uma fonte e furar outra. Síntoma em prod: morador Base recebe 403 "QUOTA_EXCEEDED" no 1º envio (se ABAC seguir matrix), ou ultrapassa o cap pretendido (se seguir commercial/personas).
- **Fix proposto**: tomar Decision 10 como autoridade (alinha com `personas-and-plans` e `commercial`):
  - **Morador Base = 2/ano** (corrigir matrix linha 43 + linha 152)
  - **Morador Pagante = 4/ano** (corrigir matrix linha 43)
  - **Sind. N2 = 4/ano** (corrigir matrix linha 43 — está como 2/ano)
  - Confirmar com João se prefere mudar a Decision para "Base = 0, Pagante = 4" (mais restritivo) — abrir D-031 em STATE.md.
- **Quem resolve**: agente principal alinha com João (decisão de produto) ou aplica Decision 10. Atualizar `functional-matrix.md` + `version`.

---

### Q3 — Empresa "Base" mencionada onde só existem `enterprise_plus` e `enterprise_pro`

- **Status**: 🔴 Aberto (semântico)
- **Fontes em conflito**:
  - `requirements/commercial.md` Req 18 → "Visibilidade por plano: **Base** visível apenas para Síndico Gestor/Plus/Pro; Plus/Pro visível para todos"
  - `requirements/personas-and-plans.md` (Empresa) → só lista `enterprise_plus` e `enterprise_pro`. Não há plano "Base" para Empresa.
- **Raiz**: texto herdado do legacy (Drizzle/TS) onde existia "Empresa Base". Stack atual decidiu apenas Plus/Pro.
- **Impacto**: leitor não sabe se "Base" se refere a Empresa em trial, em paywall expirado, ou a uma persona descartada.
- **Fix proposto**: trocar Req 18 para algo como _"Visibilidade por plano: **trial / soft-block** visível apenas para Síndico Gestor / Plus / Pro; Plus/Pro visível para todos."_
- **Quem resolve**: agente principal ou executor — mudança de redação.

---

### Q4 — Agência de Marketing: "não tem login" vs role `marketing` no Zitadel

- **Status**: 🔴 Aberto (conflito real entre dois requisitos canônicos)
- **Fontes em conflito**:
  - `requirements/content.md` Req 30 → "Agência de Marketing — **não é tenant** — **não tem login próprio, não é persona com role**"
  - `requirements/identity.md` Req 2 + decisão T7 → roles primárias incluem `marketing`
  - `requirements/personas-and-plans.md` linha 21 → "Agência de Marketing | Actor delegado (não é tenant) | Sem perfil próprio, vinculado a Empresa"
  - `requirements/personas-and-plans.md` linha 50-53 → "Agência / Marketing — Plano: `marketing_standard`"
- **Raiz**: o Req 30 é absoluto ("não tem login"), mas T7 e o plano `marketing_standard` implicam que existe sim um usuário com role `marketing` que faz login para operar (em nome de empresas que a contrataram).
- **Impacto**: confusão arquitetural — implementador não sabe se cria login + role `marketing` (e usa token só para escopo de empresa) ou se elimina role e usa só token-only delegation.
- **Fix proposto**: clarificar — **agência tem login com role `marketing`**, mas opera **sempre delegada a uma empresa via token**. Trocar Req 30 para: _"Agência de Marketing — **não é tenant**; tem usuário com role `marketing` no Zitadel mas opera apenas delegada a empresas via token de escopo restrito (`videos:upload`, `videos:edit`, `analytics:read`)."_
- **Quem resolve**: agente principal alinha com João, registra D-031/D-032 em STATE.md.

---

### Q5 — Síndico N1 e Academia LMS: `content.md` e `functional-matrix` divergem sobre certificado

- **Status**: 🔴 Aberto
- **Fontes em conflito**:
  - `requirements/content.md` Req 98.1 (Acesso por persona) → "Síndico N1+ | Cursos, Treinamentos, Lives, Fórum, Biblioteca | Fazer | **Certificado**"
  - `requirements/functional-matrix.md` Síndico N1 → "Academia LMS | Certificado | ❌ (N1) | ✅ (N2, N3)"
- **Raiz**: `content.md` lista "N1+" como tendo certificado; `functional-matrix` exclui N1.
- **Impacto**: implementador pode habilitar geração de certificado para N1 (segundo content.md) e gerar surpresa para a área comercial que vendeu N1 sem certificado (segundo matrix).
- **Fix proposto**: alinhar com matrix (mais restrito = consistente com N1 = "Aprender", tier de descoberta sem entrega de credencial). Atualizar `content.md` Req 98.1 para "Síndico **N2+**".
- **Quem resolve**: agente principal alinha com João.

---

## 🟡 MEDIUM

### Q6 — `Content/contents/master-sindico-arquitetura-solucoes.md`: assembly marcada como CONFLITO PENDENTE quando já há Decision 1

- **Status**: 🟡 Aberto (drift documental)
- **Fontes em conflito**:
  - `Content/contents/master-sindico-arquitetura-solucoes.md` (Mar/2026) §3.2.4 → "🔴 PENDENTE: Definir se assembleia ao vivo entra no escopo contratual atual"
  - `requirements/product-overview.md` Decision 1 (estável) → "Live Assembly com WebSocket real-time; modalidade online/híbrida inativa no MVP"
- **Raiz**: documento de Content é mais antigo que a Decision 1. Nunca foi marcado como stale.
- **Fix proposto**: adicionar nota no topo do arquivo: _"⚠️ Documento histórico — Decisões de escopo de Assembly resolvidas em product-overview.md Decision 1 (Mar/2026)."_ E linkar para spec viva.
- **Quem resolve**: este analista (agente 3) — já encaminhado neste vault via [[_MOC]] e este documento.

---

### Q7 — `Content/contents/master-sindico-domain-mapping.md`: usa pt-BR roles, TypeScript, Mercado Pago, Awilix DI

- **Status**: 🟡 Aberto
- **Fontes em conflito**:
  - `Content/contents/master-sindico-domain-mapping.md:150` → roles em pt-BR: `"sindico" | "morador_titular" | "morador_dependente" | "empresa_sindica"`
  - `Content/contents/master-sindico-domain-mapping.md:313` → "Estrutura de Pastas (Backend)" com path `backend/src/` e exemplos TypeScript
  - `Content/contents/master-sindico-domain-mapping.md:503` → "Payment Gateway | Mercado Pago / Stripe — recomendação Mercado Pago (Brasil)"
  - Realidade atual (`requirements/identity.md` T7, `requirements/billing.md`, `STATE.md`) → roles em **inglês**, **Go**, **Stripe**.
- **Raiz**: documento herdado da fase TypeScript/legacy.
- **Fix proposto**: marcar arquivo inteiro como `status: deprecated` na frontmatter; nota no topo apontando para spec viva.
- **Quem resolve**: agente 3 (já fará no commit deste vault).

---

### Q8 — Connect Me E→E: 3 redações diferentes para a mesma quota

- **Status**: 🟡 Aberto
- **Fontes em conflito**:
  - `requirements/personas-and-plans.md:87` → "Empresa Plus | (não aplicável — E→E só Plus/Pro)"
  - `requirements/functional-matrix.md:156` → "Empresa Plus | (E→E apenas)"
  - `requirements/commercial.md` Req 19.2 → "Quota: conforme contrato (ilimitado para Pro, configurável para Plus)"
- **Raiz**: cada arquivo descreve a mesma regra com palavras diferentes; o `personas-and-plans` chega a sugerir "não aplicável" o que pode confundir.
- **Fix proposto**: padronizar em **"Empresa Plus: E→E ilimitado dentro do Connect Me; Empresa Pro: ilimitado e disponibilidade ampliada (D2D, parceria técnica)."** Replicar em todas as três fontes. Eliminar "não aplicável" de Plus.
- **Quem resolve**: agente principal alinha com João sobre se Plus tem cap ou não, depois replica.

---

### Q9 — Vídeos institucionais: matriz mensal vs trava trimestral coexistem mas referências não são cruzadas

- **Status**: 🟡 Aberto (cosmético, mas confunde)
- **Fontes**:
  - `requirements/content.md` Req 29 → quotas mensais por plano + "Trava trimestral: 90 dias entre atualizações do mesmo vídeo"
  - `requirements/billing.md` Req 6 (Quotas — Vídeos Institucionais) → mesmas quotas mensais, mas não menciona explicitamente que a trava trimestral é **por vídeo**, não global
- **Raiz**: a regra é "quota mensal de upload" (cap de N novos vídeos/mês) **separada** de "trava trimestral por vídeo" (substituição do mesmo arquivo). Lendo só billing dá pra interpretar como cap global.
- **Fix proposto**: adicionar nota explícita em `billing.md` Req 6 logo após a tabela de vídeos: _"A quota mensal cobre **novos uploads**. Substituir um vídeo já existente respeita a trava trimestral (Rule 7) — 90 dias mínimo entre atualizações do mesmo `video_id`."_
- **Quem resolve**: agente principal ou executor.

---

### Q10 — Visibilidade de empresa "Base" (Q3) também aparece em `personas-and-plans` matriz

- **Status**: 🟡 Aberto (decorre de Q3)
- **Fonte**: `requirements/personas-and-plans.md:117` → tabela "Vídeos empresa (preview 25%)" inclui colunas Base/Pagante para Empresa, mas Empresa não tem Base.
- **Fix proposto**: junto com Q3 — remover "Base" das colunas da Empresa nas matrizes.

---

### Q11 — `deploy-config.md` declara Railway primário, mas `product-overview.md` ainda lista AWS

- **Status**: 🟡 Aberto (subconjunto de Q1)
- **Fontes**:
  - `requirements/design/deploy-config.md` (criado em F19) → Railway primário (URL conhecida na memória `project_railway_url.md`)
  - `requirements/product-overview.md:43` → AWS ECS Fargate como infra
- **Fix**: incluir junto com Q1.

---

### Q12 — `obsidian-mapping.md` lista vault externo do João como autoridade — mas Obsidian externo está stale

- **Status**: 🟡 Documentado (não é defeito; é nota)
- **Fonte**: `backend/.kiro/specs/master-sindico/reference/obsidian-mapping.md` declara que o vault externo cita Ent ORM, Casbin, Uber-FX (descartados); MS atual é fonte de verdade.
- **Ação**: este vault `Content/` agora consolida tudo. Recomendação: deprecar parcialmente o vault externo (notas pessoais do João continuam, mas notas técnicas devem migrar/linkar para `backend/.kiro/`).
- **Quem resolve**: João (vault é dele). Este analista apenas documenta.

---

### Q13 — `excopo-tecnico-antigo.txt` e `main.go.md` em `Content/` raiz — sem classificação

- **Status**: 🟡 Aberto (organizacional)
- **Fonte**: `Content/excopo-tecnico-antigo.txt`, `Content/main.go.md`
- **Raiz**: arquivos legados sem indicação de status.
- **Fix proposto**: mover para `Content/_inbox/` com nota "histórico — não modificar".
- **Quem resolve**: agente 3 (este). Vou aplicar abaixo.

---

## 🟢 LOW (cosmético / drift sem risco)

### Q14 — `requirements/content.md` Req 29 — 12 tipos de vídeo difere de `Content/contents` (Mar/2026)

- **Detalhe**: `content.md` lista "Apresentação da empresa, Portfólio de serviços, Depoimento de cliente, Tutorial..." (12 itens). `Content/contents/master-sindico-arquitetura-solucoes.md` lista 12 ligeiramente diferentes ("Apresentação institucional, Apresentação da equipe técnica..."). Mesmo cardinal, rótulos divergentes.
- **Fix**: adotar a lista de `content.md` como canônica. Sem ação imediata.

### Q15 — Recortes (`requirements/*.md`) não trazem "Bronze → Diamante" do tier de reputação mencionado em `product-overview.md` "Visão"

- **Detalhe**: `product-overview.md:21` cita "Reputação → sistema de status baseado em evidência (Bronze → Diamante)" mas nenhum recorte detalha critérios, fórmulas, transições.
- **Fix**: criar `requirements/reputation.md` quando o tema entrar em sprint (post-launch — Sprint 10+).

### Q16 — Numeração de migrations: limite por módulo é 100 (identity 001-099, billing 100-199 etc) — risco de overflow se módulo crescer

- **Detalhe**: identity já em ~14 migrations (não preocupante); billing 100-199 dá folga. Sem ação até crescer.

### Q17 — `requirements/cross-domain.md` Req 65 ainda menciona "MinIO/Cloudflare R2" como Storage — D-029 (decisão entre R2 vs S3) está aberta

- **Detalhe**: documento lista as opções mas não fixa. Aceitável porque D-029 está aberta.
- **Fix**: quando D-029 fechar, atualizar Req 65.

### Q18 — Confusão terminológica: "GSD" do projeto = "Get Shit Done" (gsd-build/get-shit-done) ≠ "GSD" da indústria = "GitHub Spec-Driven" (Spec Kit)

- **Detalhe**: `backend/.kiro/steering/sdd-workflow.md:7` cita "Inspirado em parte pelo gsd-build/get-shit-done — adotamos em 3 conceitos específicos". Em 2026, indústria usa "GSD" mais comumente para GitHub Spec-Driven (do GitHub Spec Kit).
- **Impacto**: em conversas externas e leitura de literatura SDD, leitor pode confundir.
- **Fix**: ver [[../sdd-gsd-aplicado]] — recomendação de adicionar nota explicativa em `sdd-workflow.md` (1 linha) e mencionar Spec Kit como referência cognata.
- **Quem resolve**: principal (mudança de doc steering).

---

## Resumo executivo

| Severidade | Quantidade |
|---|---|
| 🔴 Critical | 5 (Q1, Q2, Q3, Q4, Q5) |
| 🟡 Medium | 8 (Q6–Q13) |
| 🟢 Low | 5 (Q14–Q18) |
| **Total** | **18** |

**Prioridade Marco 1 (08/05)**: resolver Q1, Q2, Q3 — afetam código que vai entrar no MVP (ABAC quotas, dashboard de planos, deploy). Q4 e Q5 podem ir para Sprint 10 sem bloqueio.

**Prioridade documental** (sem impacto em código mas alinha o projeto): Q6, Q7, Q8, Q9, Q11. Bom para a próxima sessão de `claude-md-management` ou pré-Marco 1.

**Sem ação necessária**: Q10 (decorre de Q3), Q12 (vault externo é do João), Q14-Q18.

---

## Acompanhamento

Quando uma quebra for resolvida:
1. Mover entrada para seção `## ✅ Resolvidos` com data + commit/PR.
2. Atualizar fontes envolvidas (bumpar `version:` no frontmatter).
3. Se a resolução abriu nova decisão → registrar em `backend/.kiro/STATE.md` como `D-0XX`.

## ✅ Resolvidos

### 2026-04-24 — Fase 11 (9 decisões D-094..D-102 no STATE do vault)

- **Q2 — Connect Me Morador Base** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-094-—-morador-base-connect-me-2-ano-fecha-q2|D-094]]. Canônico final: **Base = 2/ano** (alinha Decision 10 + personas-and-plans + commercial). Matriz funcional corrigida na mesma Fase 11. Sobrescreve D-058/D-079 que diziam 0.
- **Q4 — Agência de Marketing sem login vs role `marketing`** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-095-—-agência-de-marketing-tem-login-zitadel-painel-admin-próprio-fecha-q4|D-095]]. Canônico: **agência tem login Zitadel com role `marketing`** + painel admin próprio (8 telas MK1-MK8). Sempre opera delegada a uma empresa via token escopo restrito. Req 30 de `content.md` era STALE (atualizado).
- **Q5 — Síndico N1/N2/N3 e certificado LMS** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-096-—-síndico-n1-n2-n3-não-existe-—-só-trial-base-plus-pro-fecha-q5-dt-004|D-096]]. A pergunta original era **inválida** sob o canônico pós-D-081 (N1/N2/N3 abolidos). Enum universal: `trial | base | plus | pro`. Mapping histórico: N1 ≡ base (consumo) ou trial (discovery); N2 ≡ plus; N3 ≡ pro. DT-004 fechada junto.

### 2026-04-24 — Quebras fora desta lista (referenciadas em outros artefatos do vault)

Estas Q-### estavam documentadas em outros lugares do vault (não neste arquivo), mas formalizo o fechamento aqui por ergonomia:

- **Q28 — Lives LMS (admin-only vs abertas)** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-097-—-lives-lms-admin-ms-empresa-pro-podem-criar-fecha-q28|D-097]]. Canônico: **Admin MS + Empresa Pro** podem criar Lives; agência via delegação de empresa Pro. Sobrescreve D-063/D-076 (admin-only MVP). Primeira live ainda é apresentação oficial da plataforma pela Admin MS.
- **Q29 — 12 vs 15 condomínios por síndico** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-098-—-15-condomínios-por-síndico-confirma-q29-reforço-de-d-068|D-098]]. Canônico: **15 condomínios** (alinha personas-and-plans Decision 11 + D-068). PDF "JORNADA DO SÍNDICO" stale neste ponto.
- **Q39 — 3 vs 5 vínculos Banco de Talentos** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-099-—-banco-de-talentos-5-vínculos-profissionais-fecha-q39-sobrescreve-d-060-d-064|D-099]]. Canônico: **5 vínculos** (PDF original do cliente "ESTRUTURA DE CADASTRO DE CURRÍCULO" TELA 08 vence). Sobrescreve D-060/D-064 que haviam fechado em 3 erroneamente.

### 2026-04-24 — Fase 12 (decisões adicionais)

- **Q25 — Score Governança: 1 score 0-100 vs 2 scores separados** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-103-—-score-duplo-no-perfil-do-síndico-fecha-q25|D-103]]. Canônico: **2 scores separados**: Governança (1-10) + Compliance (0-100). Tela [[../../../../03-subprojects/web/ui-catalog/sindico/S32-score-governanca]] nova; [[../../../../03-subprojects/web/ui-catalog/compliance/C10|C10]] atualizada. Novos invariantes [[../../../../01-domain/invariants#inv-gov-001|INV-GOV-001]] e [[../../../../01-domain/invariants#inv-gov-002|INV-GOV-002]].
- **Q40 — Avaliação escondida de gestão em eleição** → ✅ **RESOLVIDA 2026-04-24** via [[../../../../STATE#d-104-—-avaliação-escondida-de-gestão-em-eleição-ativada-fecha-q40|D-104]]. **ATIVADA** como regra canônica. Spec M1 / impl M3. Tela [[../../../../03-subprojects/web/ui-catalog/assembleia/ASM-AVAL-ELEICAO]] (web + mobile) nova. Req ASM-ELE-AVAL em [[../../../../04-requirements/functional/assembly#asm-ele-aval|assembly.md]]. Invariante [[../../../../01-domain/invariants#inv-asm-023|INV-ASM-023]].

### Sumário Fase 11

- 6 Q-### fechadas (Q2, Q4, Q5, Q28, Q29, Q39).
- 2 🔴 bloqueadores M1 fechados ([[../../../../AUDIT#a-dc-sec-004|A-DC-SEC-004]] + [[../../../../AUDIT#lgpd-m1-007|LGPD-M1-007]] via [[../../../../STATE#d-101-…|D-101]] + [[../../../../02-architecture/adr/0037-soft-delete-universal-pseudonymize|ADR-0037]]).
- 1 DT fechada (DT-004 via D-096).
- 1 D- aberto fechado (D-046 via D-100).
- ADR nova: [[../../../../02-architecture/adr/0037-soft-delete-universal-pseudonymize|ADR-0037]] (soft_delete universal + pseudonimização HMAC-keyed + retenção 5y).
