---
title: Convenções de Notação — Master Síndico
type: guide
tags: [guide, notation, glossary, master-sindico, onboarding, non-technical]
project: master-sindico
created: 2026-04-27
updated: 2026-04-27
aliases: [Glossário de siglas, Notação, Convenções]
---

# Convenções de Notação — Master Síndico

> **Para quem está entrando agora** (técnico ou não): este vault usa siglas curtas para referenciar decisões, requisitos, dívidas e marcos. Elas viraram "código compartilhado" entre o time. Esta página explica cada uma com exemplos reais do projeto.

## Tipos de identificadores numerados

### `D-###` — Decisão

**O que é**: registro formal de uma decisão importante que foi tomada (arquitetura, produto, processo).

**Como funciona**:
- Numerada sequencial: D-001, D-002, ..., D-137
- Vive em [[STATE]]
- Cada uma tem **contexto, decisão, alternativas, motivo (rationale), fontes consultadas**
- Quando muda de ideia: **NÃO apaga** — cria nova `D-###` que **revoga** a antiga (preserva histórico)

**Exemplos reais**:
- **D-094** — "Morador Base = 2 Connect Me/ano" *(revogou D-058 que dizia 0/ano)*
- **D-106** — "M1 postergado para 2026-05-18"
- **D-117** — "Cloudflare é edge primário; Railway permanece como origin"
- **D-135** — "Rejeitar bloqueio múltiplas contas por IP; usar device fingerprint" *(famílias compartilham wifi do condomínio)*

**Por que numerar**: quando alguém pergunta "por que escolhemos Mercado Pago?", a resposta é "ver D-###" — toda a história está lá.

---

### `ADR-####` — Architecture Decision Record

**O que é**: irmã mais formal de `D-###`, **só para decisões puramente arquiteturais** (escolha de framework, padrão de design, infra).

**Diferença para D-###**:
- ADR usa **4 dígitos** (ADR-0001, ADR-0042) — D-### usa 3
- ADR vive em arquivo próprio em [[02-architecture/adr/_moc]]
- Tem formato padronizado mundial (Michael Nygard template)
- D-### pode ser sobre produto, processo, regra de negócio — ADR é só arquitetura

**Exemplos**:
- **ADR-0001** — "Clean Architecture + DDD + CQRS"
- **ADR-0032** — "Planos globais Stripe-style (sem N1/N2/N3)"
- **ADR-0042** — "ISearchProvider real M1 — OpenSearch ou Meilisearch (pending dual-check)"

> **Regra prática**: se a decisão envolve "**como o sistema é construído por dentro**" → ADR. Se é "**o que o produto faz**" ou "**como o time opera**" → D-###.

---

### `A-###` — Audit Finding (Problema descoberto)

**O que é**: bug, falha, inconsistência ou risco encontrado em revisão, auditoria, dual-check ou pentest.

**Como funciona**:
- Vive em [[AUDIT]] e [[11-audit/audit-log/]]
- Tem severidade colorida (ver §Severidades abaixo)
- Quando resolvido: marcado 🟢 (não apaga — fica na história)
- 🔴 e 🟠 abertos **bloqueiam tasks novas** (gate de qualidade)

**Exemplos reais**:
- **A-035** — "Unit sem unit_type/fracao_ideal — quórum por fração ideal impossível" 🔴
- **A-077** — "admin-privilegios.md tem banner D-134 mas reescrita completa pendente" 🟡
- **A-052** — "BIL-035 stale — D-097 não refletido" 🟢 (resolvido em 2026-04-27)

> **Diferença para `D-###`**: D-### é **decisão proposital** ("vamos fazer X"). A-### é **problema encontrado** ("isso está errado/quebrado").

---

### `DT-###` — Dívida Técnica

**O que é**: "sabemos que está mal feito mas vai ficar pra depois" — débito assumido conscientemente.

**Quando vira DT-###**:
- A correção é conhecida mas **não-prioritária** agora
- Resolver exigiria refactor grande
- Bloqueia outra prioridade maior

**Exemplos**:
- **DT-010** — "IAuditLogger LGPD cross-BC incompleto" (só compliance grava hoje)
- **DT-012** — "Vocabulário N2+/N3+ ainda em arquivos não-críticos"
- **DT-WEB-005** — "packages/schemas é stub, não foi gerado de OpenAPI"

> **Diferença para A-###**: A-### é **achado** (alguém descobriu). DT-### é **dívida** (a gente sabia, deixou pra depois).

---

### `INV-###` — Invariante

**O que é**: regra que **NUNCA pode ser violada** sob nenhuma condição. É absoluta. Se quebra, sistema está em estado ilegal.

**Exemplos reais**:
- **INV-097** — "Σ fração ideal de votos ≤ 100%" (quórum nunca passa do total do prédio)
- **INV-PLN-001** — "Plan.tier ∈ {trial, base, plus, pro} sempre; nunca slug por persona"
- **INV-IAM-005** — "Permission Boundary nunca pode ser maior que do criador"

> Diferença para Business Rule: BR pode ter exceções condicionais. INV nunca.

---

### `BR-###` — Business Rule (Regra de Negócio)

**O que é**: regra de operação do negócio. Pode ter exceções documentadas.

**Exemplos**:
- **BR-063** — "Antifraude camadas: 1 device por conta, CAPTCHA após 3 falhas, device fingerprint progressivo"
- **BR-064** — "Cupom Vizinhança: lock 4h por usuário-loja"
- **BR-065** — "Formato CouponCode: ID_LOJA(5)+SEQ(3)+PALAVRA(5) = 13 chars"

---

### `GR-###` — Global Requirement (Requisito Global)

**O que é**: requisito que se aplica a **todos os módulos** do sistema (não só um BC).

**Exemplos**:
- **GR-008** — "Todo webhook externo verifica HMAC antes de parsear body"
- **GR-027** — "Trial persona-aware: síndico 15d, empresa 7d, parceiro 30d"
- **GR-029** — "Lock 90 dias após publicar vídeo (todas personas)"

---

### `FR-###` ou `<BC>-###` — Functional Requirement

**O que é**: requisito funcional **específico de um módulo (BC = Bounded Context)**.

**Padrão de nome**: `<sigla-do-BC>-###` ou `FR-<stack>-<BC>-###`.

**Exemplos**:
- **IDN-026** — Identity: device fingerprint progressivo
- **BIL-035** — Billing: regras de criação de Lives
- **CNT-005** — Content: busca full-text
- **FR-BE-INS-023** — Backend Institutional: search de condomínios
- **FR-WEB-LMS-003** — Web LMS: busca full-text na Academia

---

## Marcos (Milestones)

### `M1`, `M2`, `M3` — Marcos de Entrega

**O que é**: datas firmes de entrega com escopo definido.

| Marco | Data | Escopo |
|---|---|---|
| **M1** | **2026-05-18** (firme via D-106) | MVP contratual: Identity + Billing + Institutional + Web onboarding + Mobile auth + Busca real |
| **M2** | 2026-06-20 | Connect Me 4 fluxos + Conteúdo Mux + Reputação + Cupons Vizinhança |
| **M3** | 2026-07-20 | Assembleia Live (Livekit) + LMS + Banco Talentos + acessibilidade WCAG 2.1 AA |
| **M4** | Q3 2026 | Compliance avançado + Receita Federal + ML moderação |
| **M5** | Q4 2026 | Empresa-síndica multi-condomínio + Multi-região |
| **M6** | 2027 | AI copiloto + Mobile premium + Offline-first |

> **Regra de ouro**: priorização **sempre protege M1**. Se algo ameaça M1, vai pro M2 ou backlog.

---

## Estado de trabalho

### Backlog

**O que é**: lista de tudo que **precisa ser feito eventualmente** mas ainda não tem data nem está em sprint ativa.

**No vault**: [[05-tasks/backlog]]

**Exemplos**:
- "Implementar motor de Reputação Bronze→Diamante" → backlog M2
- "Cloudflare Containers re-avaliar como origin" → backlog M3+
- "Validação Receita Federal CNPJ" → backlog M4

> **Backlog ≠ esquecido**. É "vamos fazer eventualmente, mas hoje não".

---

### Sprint

**O que é**: ciclo de trabalho focado, geralmente 2-3 semanas, com tasks priorizadas para entregar.

**Convenção**: **Sprint 10** = sprint atual M1 (no Master Síndico).

**No vault**: [[05-tasks/by-sprint/sprint-10]] (e seguintes)

> Tasks saem do backlog → entram em sprint → quando completas, ficam registradas no histórico.

---

### Task

**O que é**: trabalho atômico para fazer (geralmente cabe em horas/poucos dias).

**Padrão de ID**: depende do contexto:
- **T-OUTBOX-01** — task transversal numerada
- **BE-001** — task backend numerada
- **MB-240** — task mobile numerada
- **FE-008** — task frontend (web) numerada

---

## Severidades visuais

Quando você ver emoji de bolinha colorida em `A-###` ou `D-###`, é severidade:

| Símbolo | Significado | Ação |
|---|---|---|
| 🔴 | **Crítico** | Bloqueia tudo. Resolver antes de qualquer task nova. Bloqueia o marco. |
| 🟠 | **Alto** | Resolver no sprint atual. Se ficar pra próximo, virar 🔴. |
| 🟡 | **Médio** | Documentar plano, resolver quando possível. Não bloqueia M1 se já tem mitigação. |
| 🟢 | **Baixo / Resolvido** | Resolvido ou prioridade baixa. Apenas registro. |
| ✅ | **Confirmado / Aceito / Implementado** | Decisão fechada. Task entregue. |
| ⏳ | **Pendente / Em progresso** | Está em curso. |
| ⚠️ | **Atenção / Pendência** | Aviso de algo que precisa ser revisitado. |
| 📝 | **A escrever / A documentar** | Conteúdo planejado mas ainda não criado. |
| 🔄 | **Atualizado** | Mudou recentemente; pode haver inconsistência. |

---

## Outras siglas que você pode topar

| Sigla | Significado | Uso |
|---|---|---|
| **MOC** | Map of Content | Índice navegacional de uma pasta (`_moc.md`) |
| **BC** | Bounded Context | "Pedaço" lógico do sistema. O Master Síndico tem 6 + cross-domain (identity, billing, institutional, commercial, content, assembly). |
| **VO** | Value Object | Objeto imutável que não tem identidade própria (ex: `CPF`, `Email`, `Money`) — DDD |
| **DDD** | Domain-Driven Design | Metodologia de modelar software seguindo o "domínio" (negócio) |
| **CQRS** | Command Query Responsibility Segregation | Separar leitura (query) de escrita (command) — padrão arquitetural |
| **ABAC** | Attribute-Based Access Control | Sistema de permissões baseado em atributos (role × plano × ação × tenant) |
| **RLS** | Row-Level Security | Postgres filtra linhas automaticamente baseado em quem está logado |
| **PKCE** | Proof Key for Code Exchange | Padrão de segurança para login OAuth/OIDC (especialmente mobile) |
| **OIDC** | OpenID Connect | Padrão moderno de autenticação (Zitadel implementa) |
| **SDD** | Spec-Driven Development | Metodologia: spec → plan → execute → verify → ship |
| **SLA** | Service Level Agreement | Garantia formal de disponibilidade/tempo (ex: 99.5%) |
| **SLO** | Service Level Objective | Meta interna de SLA (ex: P95 latência < 200ms) |
| **PR** | Pull Request | Solicitação de mesclar código no GitHub |
| **CI/CD** | Continuous Integration / Continuous Deployment | Pipeline automático de teste e deploy |
| **MFA** | Multi-Factor Authentication | Autenticação com mais de um fator (senha + código SMS/app) |
| **TOTP** | Time-based One-Time Password | Código MFA gerado por app (Google Authenticator, etc.) |
| **DPO** | Data Protection Officer | Responsável legal por LGPD na empresa |
| **DPA** | Data Protection Agreement | Contrato com fornecedor sobre tratamento de dados |
| **DPIA** | Data Protection Impact Assessment | Avaliação de impacto LGPD obrigatória para tratamentos sensíveis |
| **LGPD** | Lei Geral de Proteção de Dados (BR) | Lei brasileira equivalente ao GDPR |
| **WCAG** | Web Content Accessibility Guidelines | Padrão internacional de acessibilidade web (M3 = nível AA) |

---

## Como ler uma referência cruzada

No vault, links entre arquivos usam `[[colchetes-duplos]]`. Exemplos:

- `[[STATE#D-094]]` = "vai pro arquivo STATE.md, seção D-094"
- `[[01-domain/business-rules#BR-063]]` = "vai pra arquivo business-rules.md, regra BR-063"
- `[[../STATE]]` = "vai um nível acima e abre STATE.md"

> Se você está usando Obsidian: clica e ele te leva. Se está lendo no GitHub/texto: copia e abre o arquivo no path indicado.

---

## Quando duvidar

Se você não souber alguma sigla nova que apareça nos docs, **pergunte ao agente**. Convenções podem evoluir; novas siglas surgem com novas decisões.

Esta página deve ser **a primeira leitura** para qualquer pessoa entrando no projeto sem conhecimento prévio.

---

## Vizinhos

- [[CLAUDE]] — contrato do agente neste projeto
- [[STATE]] — onde vivem todas as `D-###` e `DT-###`
- [[AUDIT]] — onde vivem todas as `A-###`
- [[ROADMAP]] — detalhamento dos marcos M1/M2/M3
- [[STEERING]] — princípios não-negociáveis do projeto
- [[00-product/glossary]] — glossário de termos de **negócio** (síndico, fração ideal, ata, etc.) — diferente desta página, que é sobre **siglas técnicas**
- [[00-product/_moc]] — visão de produto
- [[02-architecture/adr/_moc]] — todos os ADRs catalogados
- [[05-tasks/_moc]] — backlog + sprints + tasks por módulo
