---
title: "02-connect-me — Sub-produto Connect Me (destilação Fase 8)"
type: note
tags: [sub-produto, master-sindico, v2, fase-8, connect-me, marketplace, lgpd, anti-assedio]
created: 2026-04-23
updated: 2026-04-23
status: destilado
source: commercial.md + cross-domain.md + Connect-Me-4-direcoes.md + Inventario-Telas.md + functional-matrix.md + código Go /commercial/ + SQL /migrations/ + Obsidian vault MasterSindico
---

# 02-connect-me — Sub-produto Connect Me

> Regras arquiteturais herdadas (ADR-0032, D-056..D-068):
> - **Planos globais Stripe-style**: `plan_tier ∈ {trial, base, plus, pro}` universal. Sem slugs (N1/N2/N3 abolidos nas specs canônicas; usados abaixo apenas ao citar trechos literais de fontes legadas).
> - **Personas**: syndic, resident, enterprise, marketing, local_company, admin.
> - **Admin** opera via `apps/admin` dedicada (D-134 revoga D-058/D-072) — revisa denúncias de assédio.
> - **Agência de Marketing** é sub-produto interno próprio (Zitadel próprio) — **não** consome/envia Connect Me. Marketing Link é feature separada.
> - **Morador Base = 2/ano** (D-094 Fase 11 — 2026-04-24 fecha Q2 alinhando com Decision 10 + personas-and-plans + commercial.md. Sobrescreve leitura anterior de "0/ano" que vinha de `functional-matrix.md:148` stale; matriz já foi corrigida nesta Fase 11).
> - **Connect Me unidirecional** (Regra de Ouro 3): sem chat, sem reply thread, sem histórico de conversa.

---

## 1. Fontes consultadas (inventário com data)

| # | Arquivo | Linhas lidas | Data origem | Uso |
|---|---|---|---|---|
| F1 | `backend/.kiro/specs/master-sindico/requirements/commercial.md` | 1-483 | 2026 (versão 1.0 stable) | Req 18–27 — canônico texto |
| F2 | `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` | 1-501 | 2026 (versão 1.0 stable) | Req 39 LGPD registry, Req 102 antifraude, Req 37 audit trail |
| F3 | `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` | 1-228 | 2026 (versão 1.0 stable) | Quotas Connect Me por persona/plan |
| F4 | `Development/Content/03-Modulos/Connect-Me-4-direcoes.md` | 1-247 | 2026-04-22 | Fonte canônica produto (4 fluxos + Marketing Link) |
| F5 | `Development/Content/02-Jornadas/Inventario-Telas.md` | 1-371 | 2026-04-22 | Telas S18-S20, E2-E4, E16, MK8, M12 |
| F6 | `Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Comercial.md` | 1-259 | 2026-03-10 | Critérios de aceite Req 19 (vale como histórico) |
| F7 | `Obsidian Vault/Clients/MasterSindico/01 - Arquitetura de Sistema/System Architecture - Commercial.md` | 1-244 | 2026 | Pipeline e events/NATS |
| F8 | `enterprise-catergories-subcategories-enum.schema.ts` | 1-832 | 2026 | 31 categorias + ~180 subcategorias (legendas do filtro de Connect Me Síndico→Empresa) |
| F9 | `backend/internal/modules/commercial/domain/enums/enums.go` | 1-289 | vivo | Enums ConnectMeFlow, ConnectionStatus, ProposalStatus, ClosedDealStatus, HarassmentReportStatus, HarassmentAdminAction |
| F10 | `.../domain/entities/connect_me_request.go` | 1-166 | vivo | Aggregate ConnectMeRequest |
| F11 | `.../domain/entities/proposal.go` | 1-198 | vivo | Aggregate Proposal |
| F12 | `.../domain/entities/closed_deal.go` | 1-196 | vivo | Aggregate ClosedDeal |
| F13 | `.../domain/entities/harassment_report.go` | 1-147 | vivo | Aggregate HarassmentReport |
| F14 | `.../domain/entities/company_evaluation.go` | 1-113 | vivo | Avaliação pós-execução (5 critérios 1-10) |
| F15 | `.../domain/entities/company.go` | 1-183 | vivo | Empresa com 31 categorias |
| F16 | `.../domain/repositories/i_connect_me_repository.go` | 1-16 | vivo | Interface repo (Create/Save/FindByID/ListBySender/ListByReceiver) |
| F17 | `.../application/use_cases/connect_me_use_cases.go` | 1-195 | vivo | Use cases Send/Respond/List |
| F18 | `.../application/use_cases/proposal_use_cases.go` | 1-258 | vivo | 7 use cases (Create/Update/Submit/Review/Withdraw/Get/List) |
| F19 | `.../application/use_cases/closed_deal_use_cases.go` | 1-216 | vivo | Create/Submit/CompanySign/SyndicSign/List |
| F20 | `.../application/use_cases/harassment_use_cases.go` | 1-192 | vivo | Report/Review/ListPending |
| F21 | `.../infrastructure/http/routes/connect_me_handler.go` | 1-142 | vivo | Rotas POST/GET/Respond |
| F22 | `.../module.go` | 160-381 | vivo | ABAC permissions + roteamento |
| F23 | `backend/internal/shared/providers/database/migrations/00302_create_connect_me.sql` | 1-74 | vivo | DDL `connect_me_requests` + enums `connect_me_flow`/`connection_status` |
| F24 | `.../migrations/00305_create_harassment_reports.sql` | 1-68 | vivo | DDL `harassment_reports` |
| F25 | `.../migrations/00306_create_proposals.sql` | 1-73 | vivo | DDL `proposals`, `supplier_vote_sessions`, `supplier_vote_casts` |
| F26 | `.../migrations/00304_create_closed_deals.sql` | 1-67 | vivo | DDL `closed_deals` |
| F27 | `.../migrations/00301_create_companies.sql` | 1-64 | vivo | DDL `companies`, `company_users` |
| F28 | `.../migrations/00101_create_feature_usages.sql` | 1-44 | vivo | Tracking quota Connect Me anual |

Total: 28 fontes × mediana 180 linhas = ~5000 linhas lidas integralmente, zero grep.

---

## 2. O que é / propósito

> "Domínio de relações comerciais entre síndicos, empresas, moradores e comércio local. Inclui Connect Me (formulário unidirecional), propostas, negócios fechados, registro de empresas e vizinhança." — `commercial.md:12`

Connect Me é **marketplace unidirecional de contratação formal** do Master Síndico. Formulário estruturado que registra interesse entre personas, **preservando privacidade** do emissor até o receptor demonstrar interesse explícito. Não é agregador de anúncios, não é chat, não é job-board. É um primitivo de **primeiro contato auditável + LGPD by design**.

Propósitos concretos (F4, F1):

1. **Reduzir fricção de contratação**: síndico acha empresa no Master Síndico e envia RFP sem trocar dados pessoais de cara.
2. **Proteger síndicos de spam**: empresa só vê telefone/e-mail depois de clicar "Tenho Interesse" (Req 19 critério 3, F1:52).
3. **Dar canal formal a morador pagante**: "duvida_gestao / solicitacao_manutencao / reclamacao / sugestao / outro", frio (sem histórico), limitado a 2–4/ano.
4. **Permitir B2B entre empresas Plus/Pro**: subcontratação, parceria técnica, fornecimento.
5. **Conectar condomínio a comércio local**: "Síndico → Parceiro Vizinhança" para benefícios exclusivos.
6. **Auditar toda revelação de dados pessoais**: log LGPD obrigatório (`lgpd_logs` via `commercial.md:73`).
7. **Dar trilha anti-assédio**: botão "Denunciar" em todo Connect Me + ação admin (warning/suspension/ban).

> "PRINCÍPIO FUNDAMENTAL: Connect Me é marketplace unidirecional com LGPD by design. NÃO é chat, NÃO salva histórico de conversas, NÃO tem reply (exceção: Decision 11 pendente para Morador→Síndico)." — `Domínio - Comercial.md:67` (F6)

---

## 3. O que NÃO é

| Não-confundir | Por quê |
|---|---|
| **Não é chat nem thread** | Regra de Ouro 3 (F4:13). Não existe reply nem histórico de conversa. |
| **Não é fonte de prospecção** | Empresa → Morador e Empresa → Síndico ativo **proibidos** (F4:30-34). |
| **Não é canal público** | Cada Connect Me é 1:1 entre personas identificadas. Visibilidade estrita (Req 103 F2:463). |
| **Não cria vínculo automático** | Mesmo depois de "Tenho Interesse", não há contrato; só exposição de contato + log. Proposal/Deal são aggregates separados. |
| **Não é Marketing Link** | Marketing Link é formulário Empresa → Agência registrando **intenção**, não revela contato de síndico. Sem quota. (F4:165-194). |
| **Não é adendo ou comunicado** | Adendo (Req 25) e Comunicado Pós-Deal (Req 24) vivem no ciclo de serviço pós-deal, não no Connect Me. |
| **Não tem estado "em_negociacao" no código** | F4:51 menciona `open → interested → proposal_sent → em_negociacao → closed → expired`, mas a entity Go só tem 4 estados: `pending → accepted|declined|expired` (F9:68, F10:71-74). Divergência documentada em §18. |
| **Admin não envia Connect Me** | Admin só revisa denúncias (HarassmentReport). Não há rota `admin.connect_me.create`. |
| **Agência Marketing não envia nem recebe Connect Me** | F3:122, F4:34. |

---

## 4. Personas-alvo

| Persona | Pode enviar Connect Me? | Pode receber? | Observação |
|---|---|---|---|
| **syndic** (Síndico) | Sim — 3 fluxos: Síndico→Empresa (Req 19), Síndico→Parceiro Vizinhança (Req 19.3). Também Síndico→Morador? **Não** (não há fluxo). | Sim — recebe de Morador (Req 19.1). Não recebe de Empresa ativa (anti-prospecção). | Quotas anuais por plan_tier (§5). |
| **resident** (Morador) | Sim — **Base (2/ano) e Pagante (4/ano)**. Fluxo único: Morador→Síndico (Req 19.1). | Não. | Base = **2/ano** (D-094 Fase 11 fecha Q2); Pagante = 4/ano. Divergência anterior Q2 (F3:148 dizia 0) **resolvida**. |
| **enterprise** (Empresa) | Só **Plus/Pro** em fluxo Empresa→Empresa (Req 19.2). | Sim — recebe de Síndico (Req 19) e de outra Empresa Plus/Pro (Req 19.2). | Não pode iniciar Empresa→Síndico nem Empresa→Morador. |
| **local_company** (Parceiro Vizinhança) | Não inicia Connect Me. | Sim — recebe de Síndico (Req 19.3). | Pode responder proposta de campanha/desconto. |
| **marketing** (Agência) | **Proibido** (F4:34). Usa Marketing Link (feature separada §7). | **Recebe** Marketing Link de Empresa (E16→MK8). | Não tem quota Connect Me, não aparece em ABAC de Connect Me. |
| **admin** (Master Síndico) | Não inicia. | Não recebe (mas recebe notificação de HarassmentReport). | Role privilegiada — revisa denúncias e aplica warning/suspension/ban. |

> Fonte canônica das 6 personas: `functional-matrix.md` e `personas-and-plans.md` (referenciado em F1:482).

### Direções permitidas (decision 5)

1. syndic → enterprise (Req 19 — `ConnectMeFlowSyndicToEnterprise` F9:49)
2. resident → syndic (Req 19.1 — `ConnectMeFlowResidentToSyndicCold` F9:50)
3. enterprise → enterprise (Req 19.2 — `ConnectMeFlowEnterpriseToEnterprise` F9:51)
4. syndic → local_company (Req 19.3 — `ConnectMeFlowSyndicToLocalCompany` F9:52)

### Direções proibidas (anti-prospecção)

> "❌ Empresa → Morador (anti-spam) / ❌ Empresa → Síndico ativo (Empresa só RECEBE Connect Me, não inicia) / ❌ Agência de Marketing → Qualquer / ❌ Parceiro → Qualquer (Parceiro só recebe)" — `Connect-Me-4-direcoes.md:30-35` (F4)

---

## 5. Tiers aplicáveis × role (quotas Connect Me por tier)

Mapeamento Stripe-style (**sem slugs**) — tradução canônica dos valores legados N1/N2/N3 para tier universal:

| Persona \ tier | trial | base | plus | pro |
|---|---|---|---|---|
| **syndic** | 0 | 2/ano | 4/ano | ilimitado |
| **resident** | n/a | **2/ano** (D-094 Fase 11 fecha Q2 — sobrescreve leitura F3:148 stale de 0/ano) | 4/ano (F3:149) | 4/ano |
| **enterprise** | 0 | n/a (Base empresa não existe) | Plus pode iniciar E→E (F3:156 "E→E apenas") | ilimitado (F3:157) |
| **local_company** | 0 | n/a | n/a | n/a (só recebe) |
| **marketing** | n/a | n/a | n/a | n/a |
| **admin** | n/a | n/a | n/a | n/a |

> Nota de mapping: Morador "Pagante" = qualquer tier pago (base+/plus/pro). Síndico N1 ≡ tier `base`; N2 ≡ tier `plus`; N3 ≡ tier `pro`. ADR-0032 aboliu slugs.

### Implementação real (F17:18-20 + F28)

```
const ConnectMeFeature = "connect_me"
// feature_key em feature_usages. Limite calculado em runtime pelo domain service
// QuotaLimits a partir de (role, plan_tier, feature). Motivo: mudança de plano
// não migra dados (F28:6-10).
```

Janela = ano-calendário (jan-1 00:00 UTC → jan-1 próximo ano — F28:22).

Reset: 1º de janeiro (F1:64). Redis tem cache `ms:quota:{userID}:connect_me:{year}` (F4:68, F1:73).

### Rule SQL de concorrência (antipattern A8 evitado)

> "ConsumeQuotaUseCase usa SELECT ... FOR UPDATE dentro de transação para serializar incrementos concorrentes. UNIQUE (user_id, feature_key, period_start) garante que só existe 1 linha por janela." — `00101_create_feature_usages.sql:11-13` (F28)

> "Consome quota ANTES de persistir — se ultrapassou limite anual, falha aqui. Antipattern A8 já evitado dentro do ConsumeQuotaUseCase (FOR UPDATE)." — `connect_me_use_cases.go:66-67` (F17)

Resposta HTTP em `QUOTA_EXHAUSTED`: **409 Conflict** + business error code `QUOTA_EXHAUSTED` (F21:71-74).

### Soft-block quando assinatura expira

> "Soft-block (Decision 4): Quando subscription expira: 403 QUOTA_EXCEEDED em quotas; 403 PLAN_REQUIRED em features; banner 'Reativar assinatura' + link Stripe Customer Portal. Sem grace period." — `Connect-Me-4-direcoes.md:210-211` (F4)

---

## 6. Os 4 fluxos detalhados

### Fluxo 1 — Síndico → Empresa (Req 19)

**Flow enum**: `syndic_to_enterprise` (F9:49, F23:9).

**Persona sender**: syndic | **receiver**: enterprise (ao menos 1 usuário da empresa).

#### Formulário RFP (campos)

Fontes:

- F4:41 "`categoria_servico, descricao_demanda, urgencia, budget_range`"
- F6:78 "`service_category, description, urgency, budget_range`"
- F10:33-42 (campos da entidade Go): `subject` (2–300), `message` (1–5000), `consent_version` (1–32), `sender_ip` (3–45)

Campos canônicos (síntese fontes + código):

| Campo | Constraint | Origem |
|---|---|---|
| `flow` | enum `syndic_to_enterprise` | F23:8 |
| `sender_id` | UUID users, obrigatório | F23:35 |
| `sender_company_id` | null (síndico individual) | F23:36 |
| `receiver_id` | UUID de 1 usuário admin da empresa | F23:38 |
| `receiver_company_id` | UUID `companies(id)` | F23:39 |
| `subject` | 2–300 chars — TRIM, `CHECK length BETWEEN 2 AND 300` | F23:42 |
| `message` | 1–5000 chars | F23:43 |
| `consent_version` | 1–32 chars — versão do termo aceito no envio | F23:46 |
| `sender_ip` | 3–45 chars (IPv4/IPv6) | F23:47 |
| `service_category` | uma das 31 categorias F8 — `ADMINISTRACAO_GESTAO_GOVERNANCA`...`CONVENIENCIA_COMERCIO_LOCAL_COMUNIDADE` | F8:6-70 |
| `urgency` | `baixa / media / alta / urgente` (F6:108) | F6 |
| `budget_range` | texto livre ou faixa (F4:41, F6:78) — **não persistido na entity Go atual**; provavelmente embutido em `message` | divergência §18 |

> A entity `ConnectMeRequest` (F10:26-43) não tem `service_category`, `urgency`, `budget_range` como campos tipados. Estão no corpo `message` ou ainda não implementados.

#### Sequência (propostas chegando, escolha, fechamento, avaliação pós-execução)

> "1. Síndico preenche formulário no perfil da empresa: categoria_servico, descricao_demanda, urgencia, budget_range / 2. Empresa recebe notificação 'Novo Connect Me' / 3. **Dados do síndico OCULTOS** até empresa clicar 'Tenho Interesse' / 4. Empresa clica 'Tenho Interesse' OR 'Recusar' / 5. Auto-close em 48h sem interesse / 6. Síndico recebe notificação de auto-close" — `Connect-Me-4-direcoes.md:40-49` (F4)

Passos completos:

1. **S18 — Connect Me (hub)** síndico escolhe "Criar novo" e seleciona fluxo Síndico→Empresa.
2. **S19 — Criar Connect Me** preenche formulário. Frontend aceita termo (gera `consent_version`). Rascunho pode ficar salvo em Redis (SHOULD — F1:67).
3. Submit → `POST /api/v1/connect-me` (F22:180). ABAC permission `commercial.connect_me.create` (F22:181). Handler valida: auth + body (F21:45-69).
4. Use case `SendConnectMeUseCase.Execute` (F17:61-99):
   - Consome quota (F17:68-76). Se estourou → `ErrQuotaExhausted` → 409 `QUOTA_EXHAUSTED`.
   - Cria entidade `NewConnectMeRequest` (F10:45-87): valida flow, IDs, subject, message, consent_version, sender_ip. Status inicial `pending`. `expires_at = now + 48h` (F10:83, constante `AutoCloseAfter` F10:12).
   - Persist `connectMeRepo.Create` (F17:87).
   - Log estruturado (F17:91-97): id, flow, sender_id, receiver_id, consent_version.
5. **Evento NATS** `connect_me.request.created` (F1:74) → consumer de notificação envia push + e-mail para a empresa (provider IEmailProvider Req 62 F2:338-351).
6. **E2 — Oportunidades** (empresa). Empresa vê card minimizado com `subject` + `service_category` + prazo, SEM nome/telefone/e-mail (F5:117, F1:52).
7. Empresa escolhe:
   - **E4 — Detalhe da Oportunidade** "Tenho Interesse" → `POST /api/v1/connect-me/:request_id/respond {accept: true}` (F22:190-193, F21:90-122).
   - **E3 — Recusar Oportunidade** com motivo estruturado (1 dos 6) + justificativa → `{accept: false, reason: "..."}`.
8. Use case `RespondConnectMeUseCase.Execute` (F17:119-155):
   - Carrega request.
   - **Isolamento**: só receiver pode responder (F17:126-128) → 403.
   - Accept/Decline transiciona estado (F10:131-156). `ErrConnectionAlreadyDecided` → 409 `ALREADY_DECIDED`.
   - Persist `Save`.
9. **Evento NATS** `connect_me.request.interested` (F1:75) → dispara:
   - Reveal dos dados do síndico (resposta da API detalhada inclui nome/telefone/e-mail/descricao_completa — F4:46).
   - **Log LGPD obrigatório** em `lgpd_logs` (§11). Campos: `timestamp, source_id (syndic), target_id (company_user), ip_address, terms_version, purpose` (F4:199-203).
10. Empresa cria **Proposal** (Req 20) vinculada via `connect_me_request_id` (F25:13). Ver §8.2.
11. Síndico revê/aprova propostas. Se 2+ validadas → abre **SupplierVoteSession** (Req 21, §8.6).
12. Proposta aceita → cria **ClosedDeal** (Req 22, §8.3). Dupla assinatura.
13. Após `signed` → auto-publish TimelineEntry (F19:156-176).
14. Empresa registra **ExecutionRecord** (Req 23). Síndico valida.
15. Pós-execução: Síndico submete **CompanyEvaluation** (5 critérios 1-10, anônima para empresa — Req 26, F14:34-67).

#### Auto-close 48h

> "AutoCloseAfter é o prazo de auto-expiração (Req 4.1: 48h)." — `connect_me_request.go:12` (F10)

Job cron (fora do commercial, provavelmente `jobs/` + NATS sched):

> "MarkExpired — cron noturno chama quando now > expires_at e ainda pending." — `connect_me_request.go:158-165` (F10)

Idempotente: se `status != pending`, não faz nada (F10:161).

SQL índice suportando o cron:

> `CREATE INDEX idx_connect_me_expires ON connect_me_requests(expires_at) WHERE status = 'pending';` — `00302_create_connect_me.sql:66-67` (F23)

#### 6 motivos de recusa estruturados

Canônicos (F4:55-61):

1. Sem disponibilidade operacional
2. Escopo incompatível
3. Capacidade técnica insuficiente
4. Fora da área de atendimento
5. Orçamento incompatível
6. Outro motivo (campo aberto obrigatório, até 2000 chars — F10:22 `ErrDeclineReasonTooLong`)

A entidade Go (F10:147-156) aceita qualquer `reason` string ≤ 2000 chars — **não enum**. Enumeração fica no frontend/E3. Divergência §18.

#### Status do Connect Me

**Código Go** (F9:67-72, F10:76):

```
pending | accepted | declined | expired
```

**Doc canônico produto** (F4:51, F6:84, F7:85-89):

```
open | interested | proposal_sent | em_negociacao | closed | expired
```

Observação: o modelo de produto descreve uma state machine **estendida** que inclui estágios fora do ConnectMeRequest puro (proposal_sent = depende de Proposal; em_negociacao = UI aggregate, não estado DB). A implementação canônica tem apenas 4 estados. Divergência §18 / gap G-03.

---

### Fluxo 2 — Morador → Síndico (Req 19.1) — FRIO

**Flow enum**: `resident_to_syndic_cold` (F9:50, F23:10). Sufixo `_cold` sinaliza: frio, sem reply.

**Persona sender**: resident | **receiver**: syndic.

#### Natureza

> "Formulário unidirecional **frio** (sem reply, sem histórico). Morador não vê histórico — cada novo Connect Me é iniciativa separada." — `commercial.md:81, 100` (F1)

> "Connect Me Morador→Síndico — formulário unidirecional frio (sem reply, sem histórico)" — Decision 9, `Connect-Me-4-direcoes.md:84`

#### Canais (5 categorias)

| Canal | Descrição | Fonte |
|---|---|---|
| `duvida_gestao` | Dúvida sobre gestão condominial | F1:86 |
| `solicitacao_manutencao` | Solicitação de reparo/manutenção | F1:87 |
| `reclamacao` | Reclamação (ex: barulho, vizinho) | F1:88 |
| `sugestao` | Sugestão de melhoria | F1:89 |
| `outro` | Livre | F1:90 |

#### Urgência (4 níveis)

> "`baixa`, `media`, `alta`, `urgente`" — `commercial.md:94` (F1)

#### Quotas

| Plano morador | Quota/ano | Fonte |
|---|---|---|
| `base` | **0** | F3:148 |
| `pagante` (qualquer tier ≥ base pago) | **4** | F3:149 (matriz), **2** (F4:26, Decision 10, `commercial.md:99`) |

**Divergência crítica Q2** — **RESOLVIDA 2026-04-24 via [[../../STATE#d-094-—-morador-base-connect-me-2-ano-fecha-q2|D-094]]**: Canônico final é **Base=2/ano + Pagante=4/ano** (alinha Decision 10 + personas-and-plans + commercial.md). A matriz funcional foi corrigida na mesma Fase 11 — leitura anterior "F3:148 = 0/ano" foi descartada. Q2 fechada.

#### Fluxo (opção A — frio, vencendo Regra 3)

1. **M12** (ou tela de Connect Me do morador — inventário cita M12 como "Avaliar Gestão" — há drift de IDs; o mapa do morador é 15 telas) — morador preenche form.
2. `POST /api/v1/connect-me` com `flow=resident_to_syndic_cold`. Quota consumida.
3. Síndico recebe notificação em **S20 — Connect Me Recebido (M→S)** (F5:70).
4. Síndico escolhe:
   - **Opção A (canônica / Regra 3)**: só muda status (`visualizado/em_análise/resolvido`), morador vê só status. Sem reply thread. Sem tabela de respostas (F4:90-91, F1:104).
   - **Opção B (ticket system)**: reply com `message` + `status_update`, histórico. **PENDENTE (Decision 11)**.
5. Síndico pode responder publicamente via **S24 — Criar Comunicado** (F5:74), que vira TimelineEntry (não privado).
6. Se Opção A: no DB fica `accepted` (sinaliza "vi") ou `declined` (negar pedido). Não há texto de resposta armazenado.

#### Decisão pendente (Decision 11)

> "Decisão pendente (Decision 11 do requirements.md): Síndico tem reply ou não? Duas opções: A (consistente com Decision 9 / Regra 3): unidirecional, síndico só marca status (visualizado/em_análise/resolvido), morador vê só status; B (conforme Req 19.1 escrito): ticket-system com asynchronous conversation, histórico, reply do síndico. **Provável**: Opção A para consistência com Regra de Ouro 3." — `Connect-Me-4-direcoes.md:83-91` (F4)

Implementação atual vence A (entity sem campo de "reply"). §18 registra como pendente.

#### Implementação

> "Tabelas: `connect_me_resident_requests` / Sem tabela de respostas (é assimétrico)" — `commercial.md:104-105` (F1)

**Divergência**: a migration 00302 cria uma **tabela única** `connect_me_requests` com coluna `flow` enum (F23:30). Não há tabela `connect_me_resident_requests` separada. Single-table inheritance por `flow`. §18.

---

### Fluxo 3 — Empresa → Empresa Plus/Pro (Req 19.2)

**Flow enum**: `enterprise_to_enterprise` (F9:51, F23:11).

**Restrição**: apenas `plan_tier ∈ {plus, pro}` (F1:123, F4:118-119).

#### Tipos de parceria (5)

| Tipo | Descrição | Fonte |
|---|---|---|
| `subcontratacao` | Empresa A subcontratar empresa B | F1:114, F4:122 |
| `parceria_tecnica` | Colaboração técnica entre empresas | F1:115 |
| `fornecimento_materiais` | Empresa B fornece materiais a A | F1:116 |
| `indicacao_servicos` | Indicação cruzada | F1:117 |
| `outro` | Livre | F1:118 |

> A entidade Go (F10) não tipa `partnership_type` — está embutido em `message` ou futura coluna. §18.

#### Fluxo (similar ao Fluxo 1)

1. Empresa A (Plus/Pro) escolhe empresa B (qualquer).
2. POST /connect-me com `flow=enterprise_to_enterprise` + `sender_company_id=A.id` + `receiver_company_id=B.id`.
3. Dados de contato de A ocultos até B aceitar.
4. B clica "Tenho Interesse" → reveal + LGPD log.
5. Auto-close 48h (F4:130).

#### Quota

Matriz funcional (F3:156-157): "Empresa Plus = (E→E apenas)" (ambíguo, sem número) | "Empresa Pro = ∞".

`commercial.md:126` (F1): "ilimitado para Pro, configurável para Plus".

`personas-and-plans.md` (legado): "não aplicável".

**3 redações divergentes = Q8** (F4:135). Resolução canônica V2: Plus tem quota configurável via plano (default 10/mês), Pro = ilimitado. Pendente confirmação §18.

#### Implementação

> "Tabelas: `connect_me_enterprise_requests`" — `commercial.md:129` (F1)

Na prática, está na tabela `connect_me_requests` com `flow=enterprise_to_enterprise`. Single-table.

---

### Fluxo 4 — Síndico → Parceiro da Vizinhança (Req 19.3)

**Flow enum**: `syndic_to_local_company` (F9:52, F23:12).

**Persona sender**: syndic | **receiver**: local_company (parceiro com cadastro simplificado, pode ter CPF só — F5:182).

#### Tipos de promoção (5)

| Tipo | Descrição | Fonte |
|---|---|---|
| `desconto_exclusivo` | Desconto só para condomínio X | F1:138, F4:144 |
| `promocao_dia` | Promoção por tempo limitado | F1:139 |
| `campanha_sazonal` | Natal, ano novo, Black Friday, etc | F1:140 |
| `parceria_continua` | Relacionamento contínuo | F1:141 |
| `outro` | Livre | F1:142 |

#### Fluxo

1. Síndico explora parceiros em **VS2 — Explorar Parceiros** (filtros categoria/bairro — F5:198).
2. Em **VS3 — Detalhe do Parceiro**, clica "Criar acordo/Campanha".
3. POST /connect-me com `flow=syndic_to_local_company`.
4. Dados do síndico ocultos até parceiro aceitar (F4:151).
5. Parceiro aceita → reveal + LGPD log + pode responder com proposta de promoção exclusiva.
6. Auto-close 48h.

Resultado (quando aceito): o parceiro cadastra uma **promoção exclusiva** (`neighborhood_benefits`, F4:160) com escopo `exclusiva_condominio` que aparece em **VM5 — Benefícios do meu Condomínio** para os moradores.

#### Quotas

Aplicam quotas de Síndico por tier universal (ADR-0032): `plan_tier=base` 2/ano, `plan_tier=plus` 4/ano, `plan_tier=pro` ∞ (F4:157 — a fonte usa terminologia legada N1/N2/N3 banida por D-067/DT-004). Compartilha o `feature_usages.feature_key="connect_me"` com Fluxo 1 — **todas as iniciativas de síndico consomem o mesmo bucket anual**.

#### Implementação

> "Tabelas: `connect_me_neighborhood_requests`, `neighborhood_benefits`" — `commercial.md:153` (F1)

Novamente: implementação real está em `connect_me_requests` (single-table) + `neighborhood_benefits` separada (vive no sub-produto 08-vizinhanca).

---

## 7. Marketing Link (feature separada — Empresa → Agência)

> **CLARIFICAÇÃO**: Marketing Link **não é Connect Me** — `Connect-Me-4-direcoes.md:165` (F4).

### Diferenças estruturais

| Aspecto | Connect Me | Marketing Link |
|---|---|---|
| Personas | 4 direções (§6) | Empresa → Agência |
| Vínculo | Não cria; só revela contato | **NÃO cria vínculo automático** (F4:170) |
| Reveal | Ao "Tenho Interesse" | N/A (intenção registrada, não revela nada de sensitive) |
| Quota | Anual por persona/tier | **Sem quota** (F4:171, não é volume sensitive) |
| Unidirecional? | Sim | "Não é unidirecional puro" (F4:172) — é formulário de intenção |
| Autoexpira 48h? | Sim | Não |
| Anti-assédio? | Botão Denunciar | Não aplicável |
| Tela principal | E2-E4 (empresa), S18-S20 (síndico) | **E16** (empresa preenche) → **MK8** (agência vê lista) |

### Conceito

Marketing Link é um "lead interno" da Agência de Marketing. Vínculo oficial (delegação Zitadel) só acontece via fluxo **E13-E14 — Gestão de Agência / Convidar Agência com token** (F5:126-128), **não** via Marketing Link.

### Tipos de apoio (6) — F4:178

```
producao_videos_institucionais | gestao_conteudos_tecnicos |
estrategia_posicionamento | gestao_redes_sociais |
consultoria_marketing | outro
```

### Prioridade (3) — F4:183

```
imediato | proximos_30_dias | sem_prazo_definido
```

### Status do Marketing Link (6) — F4:188

```
sent | viewed | proposal_sent | atendido | accepted | rejected
```

### Implementação

> "Tabelas: `marketing_link_requests` / Aparece em painel da agência (MK8)" — `Connect-Me-4-direcoes.md:192-193` (F4)

**Status no código**: nenhum arquivo `marketing_link*.go` encontrado no backend atual. Nenhuma migration com `marketing_link`. **Não implementado** — backlog. Gap G-01 §18.

---

## 8. Aggregates

### 8.1 `ConnectMeRequest` (aggregate root do fluxo)

Arquivo: `internal/modules/commercial/domain/entities/connect_me_request.go` (F10).

**Invariantes duros** (F10:53-69):

- `flow` deve ser 1 dos 4 enums válidos (`ErrInvalidConnectFlow`)
- `sender_id ≠ receiver_id` (`ErrSameSenderAndReceiver`)
- `subject`: TRIM 2–300 chars
- `message`: 1–5000 chars (sem TRIM para preservar quebras de linha)
- `consent_version`: TRIM 1–32 chars
- `sender_ip`: 3–45 chars (IPv4/IPv6)
- `decline_reason`: ≤ 2000 chars
- `expires_at > created_at` (DB CHECK F23:60)
- `created_at + 48h = expires_at` (setado no construtor F10:83)

**Transitions** (F10:131-165):

- `Accept()`: `pending → accepted`, seta `decided_at = now`. Idempotente: falha com `ErrConnectionAlreadyDecided` se já terminal.
- `Decline(reason)`: `pending → declined`, seta `decline_reason + decided_at`. Mesma idempotência.
- `MarkExpired()`: `pending → expired` (silencioso se não-pending — chamado por cron).

**Factories**:

- `NewConnectMeRequest` (F10:45-87): valida tudo, seta `status=pending`, `expires_at = now + AutoCloseAfter`.
- `ReconstructConnectMeRequest` (F10:89-109): hidratação do DB sem revalidar (performance).

### 8.2 `Proposal`

Arquivo: `.../domain/entities/proposal.go` (F11).

> "Proposal representa uma proposta comercial de uma empresa para um condomínio (Req 4.4). Estado: draft → submitted → accepted|rejected|withdrawn." — `proposal.go:23-25` (F11)

**Campos** (F11:26-41):

- `id, companyID, condominiumID, connectMeRequestID`
- `title` (TRIM 5–200), `scope` (20–5000), `estimatedCostCents` (int64 > 0)
- `attachments []string` (JSONB — F25:16)
- `status`, `version` (incrementa em Update), `submittedAt *time.Time`
- `createdBy`, `createdAt`, `updatedAt`

**FK opcional** `connect_me_request_id UUID REFERENCES connect_me_requests(id)` (F25:12) — proposta pode nascer sem Connect Me prévio (empresa resolve por outro canal).

**Transitions** (F9:202-205, F11:131-197):

- `Submit()`: draft → submitted (seta `submitted_at`).
- `Accept()`: submitted → accepted.
- `Reject()`: submitted → rejected.
- `Withdraw()`: draft|submitted → withdrawn (empresa retira).
- `Update()`: só em draft. Incrementa `version`.

**Status terminal**: accepted, rejected, withdrawn (F9:198).

### 8.3 `ClosedDeal`

Arquivo: `.../domain/entities/closed_deal.go` (F12).

> "ClosedDeal é o contrato fechado entre empresa e condomínio (Req 4.5). Dupla assinatura: empresa → síndico. Ambos assinados = is_immutable=true + auto-publicação na Timeline." — `closed_deal.go:26-28` (F12)

**Invariantes DB críticos** (F26:57):

```sql
CHECK ((status = 'signed') = is_immutable)
```

Não existe signed sem immutable, não existe immutable sem signed.

**Transitions** (F9:116-122, F12:138-195):

- `Submit()`: draft → awaiting_company_signature
- `CompanySign(userID, ip)`: awaiting_company → awaiting_syndic (registra `company_signed_by/at/ip`)
- `SyndicSign(userID, ip, timelineEntryID)`: awaiting_syndic → signed + `is_immutable=true`. Caller cria TimelineEntry ANTES de chamar (F19:156-177 mostra ordem).
- `Cancel(reason)`: qualquer não-terminal → canceled. Bloqueado se `is_immutable`.

**Auto-publicação Timeline**:

> "Deal confirmado → NATS → Timeline (automático)" — `System Architecture - Commercial.md:150` (F7)

> "Publica TimelineEntry automaticamente (Req 4.5). bodyJSON = {deal_id, company_id, title, total_amount, currency, contract_start, contract_end}; EntryType = 'registro_de_execucao'; Title = 'Contrato assinado: ' + deal.Title()" — `closed_deal_use_cases.go:156-176` (F19)

**Dependência explícita**: `types.ITimelinePublisher` (F19:138) — interface injetada, desacopla commercial de institutional.

### 8.4 `HarassmentReport`

Arquivo: `.../domain/entities/harassment_report.go` (F13).

> "HarassmentReport é a denúncia de assédio submetida por qualquer usuário (Req 19). Ciclo de vida: pending_review → reviewed. Imutável após reviewed." — `harassment_report.go:21-23` (F13)

**Campos** (F13:24-38):

- `id, reporterUserID, reportedUserID, connectMeRequestID` (opcional — pode ser fora de Connect Me)
- `description` (10–5000 chars) — F13:53
- `evidenceNotes` (≤ 2000 chars) — F13:57
- `status`, `adminAction *string` (nil até revisão), `adminNote`, `adminID *string`, `reviewedAt *time.Time`

**Invariante DB** (F24:48-56):

```sql
CHECK (
  status = 'pending_review'
  OR (status = 'reviewed' AND admin_action IS NOT NULL
      AND admin_id IS NOT NULL AND reviewed_at IS NOT NULL)
)
```

**Invariante entidade** (F13:49): `reporter_user_id ≠ reported_user_id` (`ErrHarassmentSameUser`).

**Transition** (F13:118-138): `Review(adminID, action, note)`: pending_review → reviewed. `action ∈ {warning, suspension, ban}` (F9:158-162). Bloqueado se já revisado.

**Side-effect no use case** (F20:132-148): se `action ∈ {suspension, ban}`, chama `IUserModerator.ModerateUser` para alterar status do usuário denunciado. Falha da moderação é logada mas não falha a request (audit-first pattern).

### 8.5 `CompanyEvaluation` (pós-execução, Req 26)

Arquivo: `.../domain/entities/company_evaluation.go` (F14).

**5 critérios obrigatórios 1–10** (F14:48-51):

1. `quality`
2. `pricing`
3. `punctuality`
4. `professionalism`
5. `communication`

Média calculada auto (F14:54). `isAnonymousToCompany = true` por default (F14:67).

> "**Anônima para empresa** (síndico anônimo) / Pública para outros síndicos (ver média de avaliação) / Acionada pós-deal (Sprint 3+)" — `commercial.md:357-359` (F1)

### 8.6 `SupplierVoteSession` + `SupplierVoteCast` (Req 21)

Fora do escopo estrito de Connect Me, mas consome propostas que **vêm de Connect Me**. Refs: F9:207-267, F25:29-62.

3 tipos de quorum: `simple_majority` (50% dos votos emitidos), `absolute_majority` (50% do eleitorado), `qualified_majority` (ex: 2/3 do eleitorado).

Valor do voto: `for / against / abstain`.

UNIQUE `(session_id, voter_user_id)` — um voto por votante por sessão (F25:59).

### 8.7 `MarketingLinkRequest` (NÃO IMPLEMENTADO)

Aggregate previsto (F4:192) mas sem arquivo Go nem migration SQL. Gap G-01.

---

## 9. Enums

### 9.1 `ConnectMeFlow` (F9:46-62, F23:8-13)

```
syndic_to_enterprise
resident_to_syndic_cold
enterprise_to_enterprise
syndic_to_local_company
```

4 valores. `IsValid()` faz switch exaustivo.

### 9.2 `ConnectionStatus` (F9:64-86, F23:20-25)

```
pending | accepted | declined | expired
```

4 valores. `IsTerminal() = (s != pending)`.

### 9.3 `ProposalStatus` (F9:178-205)

```
draft | submitted | accepted | rejected | withdrawn
```

5 valores. Helpers: `CanBeSubmitted`, `CanBeReviewed`, `CanBeWithdrawn`, `CanBeEdited`, `IsTerminal`.

### 9.4 `ClosedDealStatus` (F9:113-137, F26:9-15)

```
draft | awaiting_company_signature | awaiting_syndic_signature | signed | canceled
```

5 valores. `IsTerminal() = signed || canceled`.

### 9.5 `HarassmentReportStatus` + `HarassmentAdminAction` (F9:139-176)

```
harassment_report_status: pending_review | reviewed
harassment_admin_action:  warning | suspension | ban
```

`RequiresUserStatusChange()`: true para suspension/ban (chama IUserModerator).

### 9.6 `DeclinationReason` (6 motivos — NÃO é enum Go)

Produto canônico (F4:55-61):

1. `sem_disponibilidade_operacional`
2. `escopo_incompativel`
3. `capacidade_tecnica_insuficiente`
4. `fora_area_atendimento`
5. `orcamento_incompativel`
6. `outro_motivo` (requer texto aberto ≥ 1 char)

**Implementação atual**: campo `decline_reason TEXT` ≤ 2000 chars (F10:22, F23:50) — sem enum. Frontend força UI com 6 opções + texto livre quando "outro". §18.

### 9.7 `UrgencyLevel` (Morador → Síndico)

```
baixa | media | alta | urgente
```

Não é enum Go; vive em `message` ou campo frontend. F1:94.

### 9.8 `ResidentRequestCategory` (Morador → Síndico)

```
duvida_gestao | solicitacao_manutencao | reclamacao | sugestao | outro
```

Não é enum Go. F1:86-90.

### 9.9 `CompanyStatus` (suporta o ecossistema Connect Me)

```
pending_verification | active | suspended | blocked
```

F9:8-12. `IsOperational() = (s == active)` — empresa suspensa/bloqueada **não recebe novos Connect Me**. F9:25-27.

### 9.10 `CompanyRole` (interno à empresa)

```
admin | operator
```

F9:32-43. Só `admin` gerencia usuários da empresa.

### 9.11 `EmpresaSindicaLinkStatus`

```
pending | active | revoked
```

F9:273-288. Não relacionado direto a Connect Me, mas relevante para permissões de quem responde em nome do síndico.

### 9.12 Enterprise Categories (filtro de Connect Me Síndico→Empresa)

Fonte: `enterprise-catergories-subcategories-enum.schema.ts` (F8).

**31 categorias** (F8:6-70; usuário mencionou 32, contei 31 distintas):

1. `ADMINISTRACAO_GESTAO_GOVERNANCA`
2. `JURIDICO_DIREITO_IMOBILIARIO`
3. `CONTABILIDADE_AUDITORIA_FINANCAS`
4. `RH_PESSOAS_SAUDE_OCUPACIONAL`
5. `CURSOS_EDUCACAO_DESENVOLVIMENTO`
6. `CURSOS_ORATORIA_COMUNICACAO`
7. `SEGUROS`
8. `LAUDOS_AUTOVISTORIA_MANUTENCAO_PREDIAL`
9. `SAUDE_AMBIENTAL`
10. `ELETRICA_ENERGIA_SPDA`
11. `HIDRAULICA_SANEAMENTO`
12. `MEDICAO_INDIVIDUALIZACAO_CONSUMO`
13. `GAS`
14. `ELEVADORES_TRANSPORTE_VERTICAL`
15. `CLIMATIZACAO_VENTILACAO_EXAUSTAO`
16. `PREVENCAO_COMBATE_INCENDIOS`
17. `SEGURANCA_CONTROLE_ACESSO`
18. `LIMPEZA_ESPECIALIZADA`
19. `TERCEIRIZACAO_SERVICOS`
20. `JARDINS_AREAS_VERDES`
21. `LAZER_ESPORTE_BEM_ESTAR`
22. `ARQUITETURA_DESIGN`
23. `ENGENHARIA`
24. `CONSTRUCAO_REFORMAS_MANUTENCAO_ESTRUTURAL`
25. `MANUTENCAO_CONSERVACAO_FACHADAS`
26. `GARAGENS_MOBILIDADE_ACESSOS`
27. `TECNOLOGIA_SISTEMAS_CERTIFICACOES`
28. `SUSTENTABILIDADE_ESG`
29. `MERCADO_CONDOMINIOS`
30. `MATERIAIS_PRODUTOS_SUPRIMENTOS`
31. `CONVENIENCIA_COMERCIO_LOCAL_COMUNIDADE`

**Subcategorias**: 183 (contagem exata via `CATEGORIA_SUB_MAP` F8:586-832). Usuário espera 202; divergência G-06.

Distribuição por categoria (contagem de subcategorias — F8 map):

- ADMINISTRACAO_GESTAO_GOVERNANCA: 6
- JURIDICO_DIREITO_IMOBILIARIO: 6
- CONTABILIDADE_AUDITORIA_FINANCAS: 8
- RH_PESSOAS_SAUDE_OCUPACIONAL: 6
- CURSOS_EDUCACAO_DESENVOLVIMENTO: 4
- CURSOS_ORATORIA_COMUNICACAO: 4
- SEGUROS: 5
- LAUDOS_AUTOVISTORIA_MANUTENCAO_PREDIAL: 6
- SAUDE_AMBIENTAL: 4
- ELETRICA_ENERGIA_SPDA: 9
- HIDRAULICA_SANEAMENTO: 9
- MEDICAO_INDIVIDUALIZACAO_CONSUMO: 5
- GAS: 6
- ELEVADORES_TRANSPORTE_VERTICAL: 5
- CLIMATIZACAO_VENTILACAO_EXAUSTAO: 5
- PREVENCAO_COMBATE_INCENDIOS: 7
- SEGURANCA_CONTROLE_ACESSO: 7
- LIMPEZA_ESPECIALIZADA: 5
- TERCEIRIZACAO_SERVICOS: 10
- JARDINS_AREAS_VERDES: 5
- LAZER_ESPORTE_BEM_ESTAR: 6
- ARQUITETURA_DESIGN: 5
- ENGENHARIA: 12
- CONSTRUCAO_REFORMAS_MANUTENCAO_ESTRUTURAL: 11
- MANUTENCAO_CONSERVACAO_FACHADAS: 8
- GARAGENS_MOBILIDADE_ACESSOS: 5
- TECNOLOGIA_SISTEMAS_CERTIFICACOES: 8
- SUSTENTABILIDADE_ESG: 5
- MERCADO_CONDOMINIOS: 7
- MATERIAIS_PRODUTOS_SUPRIMENTOS: 10
- CONVENIENCIA_COMERCIO_LOCAL_COMUNIDADE: 5

Soma: 183. Divergência 202 registrada em §18.

**Uso no Connect Me**: síndico filtra empresas por `category` ao escolher destinatário de Connect Me Síndico→Empresa. Empresa cadastra `categories JSONB NOT NULL DEFAULT '[]'::jsonb` (F27:18) — multi-select.

---

## 10. Telas tela-por-tela

### S18 — Connect Me (hub) — Síndico

Fonte: `Inventario-Telas.md:69` (F5).

**Módulo**: Connect Me.

**Função**: hub de entrada para síndico. 2 abas:

- **Enviados** (usando `ListConnectMeUseCase` com `role=sent` — F17:182) — mostra requests que o síndico criou.
- **Recebidos** (usando `role=received`) — Connect Me de morador (Fluxo 2).

**Ações**:

- Botão "Novo Connect Me" → S19.
- Card por request: status, flow, data, CTA (detalhe ou responder).

**ABAC**: `commercial.connect_me.read` (F22:186).

### S19 — Criar Connect Me — Síndico

**Flow**: Síndico→Empresa (Req 19) ou Síndico→Parceiro Vizinhança (Req 19.3), selecionado no início.

**Formulário**:

- Escolher categoria (uma das 31 enterprise categories para Fluxo 1, ou categoria de parceiro para Fluxo 4).
- `subject` (2–300), `message` (1–5000, inclui urgência e budget), `consent_version` (auto), aceite de termo obrigatório.
- Seleção de destinatário (empresa ou parceiro).

**Rascunho**: Redis SHOULD (F1:67). Provável endpoint draft futuro.

**Sugestão inteligente** (SHOULD F1:68): recomendação de empresas baseada em categoria.

**Submit**: POST /api/v1/connect-me.

**Indicador quota restante**: UI mostra "2 de 2 restantes" ou "ilimitado" antes de permitir.

**ABAC**: `commercial.connect_me.create`.

### S20 — Connect Me Recebido (Morador→Síndico)

Lista de requests Fluxo 2 recebidos. Opção A (canônica): só marca status. Sem campo reply.

**Ações**: "Marcar como visualizado", "Marcar em análise", "Marcar resolvido" (mapeiam para `Accept/Decline` na entity).

Botão "Responder com Comunicado" → leva para **S24 — Criar Comunicado** (público).

### E2 — Oportunidades (Connect Me) — Empresa

Fonte: `Inventario-Telas.md:116` (F5).

Lista de Connect Me recebidos pela empresa (Fluxo 1 Síndico→Empresa; Fluxo 3 Empresa→Empresa).

**Dados visíveis (ocultos)**:

- `subject` (titulo da demanda)
- Categoria (enterprise_category)
- Urgência
- Prazo/budget_range (se no message)
- Tempo restante (48h – elapsed)
- **NÃO mostra**: nome, telefone, e-mail, condomínio.

Badge status: "Novo" | "Visualizada" | "Tempo esgotando".

### E3 — Recusar Oportunidade

**Formulário**:

- Select de 1 dos 6 motivos (§9.6).
- Se "outro" → textarea obrigatório.
- Preview da mensagem automática enviada ao síndico.

**Submit**: POST /api/v1/connect-me/:id/respond `{accept: false, reason: "..."}`.

### E4 — Detalhe da Oportunidade (pós "Tenho Interesse")

Só acessível após empresa clicar "Tenho Interesse". Revela:

- Nome completo do síndico (F4:46)
- Telefone
- E-mail
- Descrição completa da demanda (message completo)
- Condomínio de origem

**LGPD**: ao abrir esta tela, front ecoa "Dados revelados em <timestamp>, versão do termo <X>. Uso permitido apenas para esta oportunidade."

Submit "Tenho Interesse": POST /api/v1/connect-me/:id/respond `{accept: true}`.

Ações a partir daqui:

- Criar Proposal (Req 20) → fluxo E5 Registrar Execução **NÃO** — Proposal tem tela própria.
- Registro do Connect Me fica em histórico.

### M12 — Avaliar Gestão OU Form Connect Me Morador?

Ambiguidade: inventário cita M12 como "Avaliar Gestão (bimestral, 3 perguntas escala 1-10)" — `Inventario-Telas.md:33`. Enunciado do usuário diz "M12 (Morador votação/Connect Me)". Interpretação:

- **M12 canônico**: Avaliar Gestão (institutional — Req 15).
- **Connect Me do morador**: tela adicional, não numerada no inventário atual; provável M16 futura ou sub-rota de M11 (Comunicados).

Registrado como §18 G-05.

### E16 — Formulário Marketing Link (Empresa → Agência)

Fonte: `Inventario-Telas.md:129` (F5).

**Não é Connect Me**. Formulário de intenção registrando que a empresa quer apoio de agência de marketing.

**Campos** (F4:177-188):

- `tipo_apoio` (6 opções, §7)
- `prioridade` (3 opções)
- `descricao` (texto livre)
- Info institucional da empresa (pré-preenchida do perfil)

**NÃO cria vínculo automático.** Para vínculo oficial, usar E13-E14 (convite com token Zitadel).

**Submit**: POST `/api/v1/marketing-links` (endpoint **não existe** hoje — G-01).

### MK8 — Marketing Link Recebido (Agência)

Fonte: `Inventario-Telas.md:158` (F5).

Lista de Marketing Links enviados por empresas. Cada item mostra empresa, tipo_apoio, prioridade, status (`sent/viewed/proposal_sent/atendido/accepted/rejected`).

Agência pode responder com proposta externa (fora do Master Síndico — e-mail/WhatsApp via dados institucionais da empresa).

### Telas relacionadas fora do Connect Me (contexto)

| Tela | Relação |
|---|---|
| E5 Registrar Execução | Pós-Deal, não Connect Me. |
| E10 Dashboard Empresarial | Métricas Connect Me = CTR e taxa de aceite (Req 44 F2:252-265). |
| S26 Validações Pendentes | Hub síndico — inclui propostas vindas de Connect Me. |
| VS1-VS10 Vizinhança Síndico | VS3 é tela-ponte para criar Connect Me Fluxo 4. |

---

## 11. Regras de negócio

| ID | Regra | Origem |
|---|---|---|
| **RN-CM-01** | Connect Me é **unidirecional** (Regra de Ouro 3). Sem chat, sem reply, sem thread, sem histórico de conversa. | F4:13, F1:96-97 |
| **RN-CM-02** | Dados do emissor são **ocultos** até receptor clicar "Tenho Interesse". | F1:52, F4:43 |
| **RN-CM-03** | Ao revelar, **log LGPD obrigatório** em `lgpd_logs` (append-only) com timestamp + IP + finalidade + versão do termo. | F1:60, F4:199-203, F2:172-190 |
| **RN-CM-04** | **Auto-close em 48h** sem resposta (job cron noturno migra pending → expired). | F1:59, F10:12, F23:54 |
| **RN-CM-05** | **Quotas anuais** por persona/plan_tier, reset em 1º janeiro, storage em `feature_usages` com FOR UPDATE. | F1:63-64, F3:148-157, F28 |
| **RN-CM-06** | **Anti-assédio**: botão "Denunciar" em cada Connect Me. Denúncia abre `HarassmentReport` → admin revisa → warning/suspension/ban. | F1:61, F13, F20 |
| **RN-CM-07** | **Anti-prospecção**: Empresa → Morador e Empresa → Síndico ativo proibidos por design (`ConnectMeFlow` enum não tem esses valores). Validação na entity `IsValid()`. | F4:30-34, F9:55-62 |
| **RN-CM-08** | **Agência Marketing não consome Connect Me** — usa Marketing Link (§7). | F3:122, F4:34 |
| **RN-CM-09** | Empresa→Empresa requer **plan_tier ∈ {plus, pro}** (ABAC enforcement). | F1:123, F3:156-157 |
| **RN-CM-10** | Somente `receiver_id` pode responder (accept/decline) — enforcement no use case, não só ABAC. | F17:126-128 |
| **RN-CM-11** | Quota consumida **antes** do persist do ConnectMeRequest — falha ANTES de criar registro. | F17:66-76 |
| **RN-CM-12** | **Isolamento por company_id + condominium_id** (BeyondCorp). Tenant isolation enforçado via ABAC. | F7:30, F22 (RequirePermission com NoTenantExtractor) |
| **RN-CM-13** | Auditabilidade: todo send, respond, reveal, report, review dispara log estruturado no logger + evento NATS. | F17:91, F20:68, F2:116-142 |
| **RN-CM-14** | **Imutabilidade pós-decisão**: accepted/declined/expired não podem voltar a pending. | F10:131-155 `ErrConnectionAlreadyDecided` |
| **RN-CM-15** | **Suspended/blocked companies não recebem Connect Me**. `IsOperational()` bloqueia. | F9:25-27, F15:152 |
| **RN-CM-16** | Rascunho de Connect Me em Redis (SHOULD, não MUST). TTL não especificado. | F1:67 |
| **RN-CM-17** | **Sugestão de empresas** baseada em categoria (SHOULD, recomendação não obrigatória). | F1:68 |
| **RN-CM-18** | **Consent version**: obrigatório no envio. Pedido deve ser recusado se termo ativo diverge do frontend. | F10:64, F23:46 |
| **RN-CM-19** | **Motivo de recusa**: 6 opções estruturadas no frontend; campo livre aceita até 2000 chars; "outro" exige texto. | F4:55-61, F10:22 |
| **RN-CM-20** | Soft-block de subscription expirada: 403 `QUOTA_EXCEEDED` em quotas e 403 `PLAN_REQUIRED` em features, **sem grace period**. | F4:210-211 |

---

## 12. Invariantes

### 12.1 Entity `ConnectMeRequest`

- `flow.IsValid()` — um dos 4 valores.
- `sender_id != receiver_id`.
- `len(subject) ∈ [2, 300]` (após TRIM).
- `len(message) ∈ [1, 5000]`.
- `len(consent_version) ∈ [1, 32]` (após TRIM).
- `len(sender_ip) ∈ [3, 45]`.
- `len(decline_reason) ≤ 2000`.
- `expires_at > created_at` (CHECK DB F23:60).
- `status == pending` ao criar (F10:75).
- `expires_at = created_at + 48h` (F10:83, constante).
- Transição pending → terminal **uma só vez**.
- Status terminal **não** é revertível (F10:132).

### 12.2 Entity `Proposal`

- `title` 5–200 (após TRIM).
- `scope` 20–5000 (sem TRIM).
- `estimatedCostCents > 0` (F25:15 CHECK).
- `attachments` JSONB array, default `[]`.
- `version >= 1`, incrementa a cada Update em draft.
- Apenas `owner company` pode editar/submeter/retirar (F18:80-82).
- Status terminal (`accepted`, `rejected`, `withdrawn`) **imutável**.
- Update bloqueado em status != draft.

### 12.3 Entity `ClosedDeal`

- `title` 2–300 (TRIM).
- `scope_of_work` 10–20000.
- `total_amount_cents >= 0` (contratos simbólicos = 0 permitidos).
- `currency` length == 3 (ex: `BRL`, `USD`).
- `contract_end >= contract_start` (quando ambos setados).
- **`(status == signed) <=> is_immutable == true`** (DB CHECK F26:57).
- Post-sign: entidade bloqueia qualquer mutator (`ErrDealIsImmutable` F12:149, 168).
- Cancel bloqueado após sign ou se já canceled.

### 12.4 Entity `HarassmentReport`

- `reporter != reported` (entity F13:51 + DB F24:47).
- `description` 10–5000 chars.
- `evidence_notes` ≤ 2000.
- `admin_note` ≤ 2000 (DB F24:42).
- Review consistency: reviewed implica `admin_action + admin_id + reviewed_at` não-nulos (DB F24:48).
- Revisão uma única vez (pending_review → reviewed, terminal).

### 12.5 Invariantes cross-aggregate

- **Quota**: `used >= 0` (DB CHECK F28:28, antipattern A9 evitado).
- **Unicidade quota-janela**: `UNIQUE(user_id, feature_key, period_start)` F28:36.
- **FK Proposal.connect_me_request_id**: nullable mas válida (SET NULL on delete não definido; RESTRICT default).
- **FK HarassmentReport.connect_me_request_id**: nullable (denúncia pode ser fora de Connect Me).
- **Proposta única por (Request, Company)**: **não é enforçada no DB atual**. Apenas `idx_proposals_company` e `idx_proposals_condominium` (F25:25-26). UNIQUE constraint ausente = G-02.

### 12.6 Invariantes LGPD

- `lgpd_logs` é **append-only** (Req 37 F2:132).
- Retenção 5 anos (F2:136).
- Direito ao esquecimento: purga só após vencimento (F4:202).

### 12.7 Invariantes anti-assédio

- Denúncia é imutável após reviewed.
- Ação ban é terminal para o usuário denunciado (via `IUserModerator`).
- Persist antes do moderator call: audit-first (F20:127).

---

## 13. State machines

### 13.1 ConnectMeRequest

```
                    +-----------+
                    |  pending  |  (status inicial ao criar)
                    +-----+-----+
                          |
        +-----------------+-----------------+
        |                 |                 |
     Accept()         Decline(reason)   MarkExpired()
        |                 |                 |
        v                 v                 v
 +-----------+      +------------+     +----------+
 | accepted  |      | declined   |     | expired  |
 +-----------+      +------------+     +----------+
 (terminal)         (terminal)         (terminal)
```

Guards:

- `Accept`: status != terminal. Se terminal → `ErrConnectionAlreadyDecided`.
- `Decline`: status != terminal. `len(reason) ≤ 2000`.
- `MarkExpired`: somente se `status == pending`. Idempotente.

### 13.2 Proposal

```
    +--------+                    +-----------+
    | draft  | --- Update() --+   |           |
    +---+----+                |   |           |
        |                     +-->|  draft    | (mesma, só version++)
        | Submit()                |           |
        v                         +-----------+
    +--------------+
    | submitted    | --- Accept() ---> accepted (terminal)
    +--------------+
        |           \-- Reject() ---> rejected (terminal)
        | Withdraw()
        v
    +-------------+
    | withdrawn   | (terminal)
    +-------------+

Também: draft --- Withdraw() ---> withdrawn.
```

### 13.3 ClosedDeal

```
   draft
     |
   Submit()
     v
 awaiting_company_signature
     |
   CompanySign()
     v
 awaiting_syndic_signature
     |
   SyndicSign()   -----> cria TimelineEntry ANTES
     v
   signed (is_immutable=true, terminal)

Qualquer estado não-terminal --- Cancel() --> canceled (terminal).
```

### 13.4 HarassmentReport

```
 pending_review --- Review(admin, action, note) ---> reviewed (terminal)

Side-effect: se action in {suspension, ban}, chama IUserModerator.
```

---

## 14. Domain events

Derivados das chamadas de logger + events citados em `commercial.md`, `cross-domain.md`, `System Architecture - Commercial.md` (F7) e `Connect-Me-4-direcoes.md` (F4).

| Evento | Trigger | Consumer | Dados | Fonte |
|---|---|---|---|---|
| `commercial.connect_me.request.created` | `SendConnectMeUseCase` após persist | NotificationService (e-mail/push à empresa), Analytics | `id, flow, sender_id, receiver_id, receiver_company_id, consent_version, created_at` | F1:74, F17:91-97 |
| `commercial.connect_me.request.interested` | `RespondConnectMeUseCase` com `accept=true` | LGPDLogger → `lgpd_logs`, NotificationService (reveal data ao síndico) | `id, sender_id (revealed to receiver), receiver_id, ip, terms_version, purpose` | F1:75, F4:71 |
| `commercial.connect_me.request.declined` | `RespondConnectMeUseCase` com `accept=false` | NotificationService (recusa educada ao sender) | `id, reason, decline_reason_category` | implícito F17:150-155 |
| `commercial.connect_me.request.expired` (aka `connect_me.auto-expired`) | Job cron `MarkExpired` | NotificationService, Analytics | `id, sender_id, receiver_id, created_at, expires_at` | F1 "auto-close", F7:100 |
| `commercial.proposal.sent` | Proposal.Submit | Syndic Validations feed (S26) | `id, company_id, condominium_id, connect_me_request_id` | F7:115 |
| `commercial.proposal.validated` / `accepted` | Proposal.Accept | ClosedDeal creator, Empresa UI | `proposal_id, accepted_by, accepted_at` | F7:116 |
| `commercial.proposal.rejected` | Proposal.Reject | Empresa UI | idem | F7:117 |
| `commercial.deal.confirmed` (a.k.a `signed_complete`) | ClosedDeal.SyndicSign | Timeline Publisher, ReputationFeed | `deal_id, company_id, condominium_id, timeline_entry_id` | F1:237, F7:153 |
| `commercial.deal.cancelled` | ClosedDeal.Cancel | Empresa UI, Institutional Timeline | `deal_id, reason` | F7:154 |
| `commercial.execution.validated` | ExecutionRecord.Approve (Req 23) | Master Plan updater, Timeline | — | F1:266 |
| `harassment.report.created` | ReportHarassmentUseCase | Admin dashboard feed, NotificationService (admin) | `id, reporter_id, reported_id, connect_me_request_id` | F20:68 |
| `harassment.report.reviewed` | ReviewHarassmentReport | UserModerator side-effect, Audit trail | `id, admin_id, action, reported_id` | F20:150-155 |
| `audit.log.created` | Centralizador de audit | audit_trail table | `user_id, tenant_id, action, resource_type, resource_id, timestamp, ip, user_agent, before, after` | F2:141 |
| `lgpd.data.revealed` (implícito) | Dentro do fluxo `request.interested` | lgpd_logs append | `source_id, target_id, ip, terms_version, purpose, timestamp` | F2:180-190 |
| `marketing_link.request.created` (NÃO IMPLEMENTADO) | — | MK8 feed | — | G-01 |

**Backbone**: NATS JetStream (F7:217-236).

Versionamento de eventos: não documentado ainda; gap G-07.

---

## 15. Dependências com outros sub-produtos

| Sub-produto | Tipo dependência | Detalhe |
|---|---|---|
| **00-billing** | Quota | `feature_usages.feature_key="connect_me"`. `IQuotaConsumer` injetado em `SendConnectMeUseCase` (F17:47-58). `ErrQuotaExhausted` propaga para 409. Mudança de plan_tier reflete quota em runtime (F28:6-10). |
| **01-governanca-institucional** | Pipeline | `ClosedDeal.SyndicSign` chama `ITimelinePublisher.PublishTimelineEntry` com `EntryType=registro_de_execucao` (F19:166-173). `ExecutionRecord` validado também publica na Timeline. |
| **03-reputacao** | Consumer downstream | `CompanyEvaluation` (5 critérios 1-10, Req 26) é anônima e alimenta feed de reputação agregada pública (F1:346-363). |
| **04-conteudo-videos** | Visibility-revogação cruzada | Vídeos institucionais de empresa só visíveis a morador durante votação de fornecedor (Req 103 F2:463-475). Connect Me → Proposal → SupplierVoteSession → `video_visibility_grants`. |
| **05-assembleia-live** | Votação | `SupplierVoteSession` (Req 21) é **parte de Assembleia** (F1:198). 2+ propostas validadas viram votação de fornecedor com quorum. |
| **06-lms-academia** | Nenhum direto | — |
| **07-banco-de-talentos** | Nenhum direto | — |
| **08-vizinhanca** | Fluxo 4 destino | Síndico → Parceiro cria `neighborhood_benefit` exclusivo (F4:160). |
| **09-compliance** | Audit + LGPD | `audit_trail` append-only registra todos eventos de Connect Me. `lgpd_logs` central (F2:190). Governance Score componente "LGPD Compliance 10pt" (F2:26). |
| **10-marketing-agencias** | Fluxo paralelo | Marketing Link (§7). Delegação Zitadel independente; não consome Connect Me. |
| **11-administradoras-condominiais** | Cross-tenant | Admin de administradora pode responder Connect Me em nome do condomínio (via `empresa_sindica_link` F9:271-288). |
| **Identity/IAM** | ABAC engine | `RequirePermission(abacEngine, "commercial.connect_me.<action>", ...)` em `module.go:181-192` (F22). Permissions: `commercial.connect_me.create/read/respond`. `commercial.harassment.report/list/review`. |
| **Notificações (Resend/FCM/Twilio)** | Email/Push/SMS | Req 62 (Email), 63 (SMS), 67 (Push). Template canônico `noreply@mastersindico.com.br` (F2:346). |

---

## 16. Anti-assédio (botão "Denunciar" + admin escalation)

### 16.1 Fluxo

1. Usuário (qualquer persona autenticada) em qualquer Connect Me vê botão **[Denunciar]**.
2. Clica → form com `description` (10–5000 chars), `evidence_notes` (≤ 2000, opcional).
3. POST `/api/v1/harassment-reports` (F22:364-367).
4. ABAC permission: `commercial.harassment.report`.
5. Use case `ReportHarassmentUseCase.Execute` (F20:47-75) valida, cria entidade, persist, log estruturado.
6. Evento `harassment.report.created` → admin dashboard.

### 16.2 Revisão

1. Admin em painel vê lista pending (`GET /api/v1/harassment-reports` — permission `commercial.harassment.list`).
2. Admin escolhe ação: **warning** (só registra), **suspension** (users.status = suspended), **ban** (users.banned = true).
3. POST `/api/v1/harassment-reports/:report_id/review` `{action, note}`. Permission `commercial.harassment.review`.
4. Use case `ReviewHarassmentReportUseCase`:
   - Review na entity (F13:118-138) — valida action.
   - **Persist PRIMEIRO** (audit-first — F20:127).
   - Se action ∈ {suspension, ban}: chama `IUserModerator.ModerateUser` (F20:134-147). **Falha é logada como erro crítico mas não reverte a revisão**.
5. Evento `harassment.report.reviewed`.

### 16.3 Tabela

```sql
CREATE TABLE harassment_reports (
    id, reporter_user_id, reported_user_id,
    connect_me_request_id UUID REFERENCES connect_me_requests(id), -- opcional
    description TEXT (10-5000),
    evidence_notes TEXT (≤2000),
    status harassment_report_status DEFAULT 'pending_review',
    admin_action harassment_admin_action, admin_note, admin_id, reviewed_at,
    created_at, updated_at,
    CONSTRAINT chk_different_users,
    CONSTRAINT chk_review_consistency
);
```

3 índices: `reported_user_id`, `status WHERE pending_review`, `reporter_user_id`.

### 16.4 Integração com Companies

`Company.Suspend()` (F15:169-176) é disparada diretamente em tipo "empresa reportada" via sistema maior — mas no escopo atual, moderação de `users` (reported_user_id). Moderação de companies é separada (admin manual).

### 16.5 Audit trail

Todo `harassment.report.*` vai para `audit_trail` (Req 37 F2:113-142). Reteçãode 5 anos. Acesso apenas admin + síndico principal (F2:137).

---

## 17. Estado no código (status Sprint)

| Item | Status | Evidência |
|---|---|---|
| `ConnectMeRequest` entity + 4 fluxos | ✅ Implementado | F10, tests em F_test |
| `connect_me_requests` table + enums | ✅ Implementado | F23 |
| `SendConnectMeUseCase` + quota consumption | ✅ Implementado com FOR UPDATE | F17, F28 |
| `RespondConnectMeUseCase` (Accept/Decline) | ✅ Implementado | F17 |
| `ListConnectMeUseCase` (sent/received) | ✅ Implementado | F17 |
| Handler HTTP POST/GET/Respond | ✅ Implementado | F21 |
| ABAC permissions em `module.go` | ✅ Implementado | F22 |
| LGPD log na reveal | ⚠️ Parcial — entity grava `consent_version` e `sender_ip`, mas `lgpd_logs` centralizada é referenciada só em docs (não vi migration `lgpd_logs` nas fontes lidas). | F1:190, F4:203 |
| Notificação email/push | ⚠️ Provider stubs (F2:63: SMS Sprint 2; Push Sprint 2) | F2 |
| Job cron auto-close 48h | ⚠️ Entity tem `MarkExpired()` mas job scheduler não evidenciado | F10:158 |
| `Proposal` + state machine + vote sessions | ✅ Implementado | F11, F18, F25 |
| `ClosedDeal` + dupla assinatura + auto-Timeline | ✅ Implementado | F12, F19, F26 |
| `HarassmentReport` + review + moderator | ✅ Implementado | F13, F20, F24 |
| `CompanyEvaluation` (5 critérios 1-10) | ✅ Implementado | F14 |
| Marketing Link aggregate | ❌ Não implementado | G-01 |
| UNIQUE constraint `(connect_me_request_id, company_id)` em proposals | ❌ Não existe | G-02 |
| State machine estendida (`open/interested/proposal_sent/em_negociacao`) | ❌ Código tem só 4 estados | G-03 |
| Fluxo 2 opção B (ticket system) | ⏳ Decisão 11 pendente | F4:87-91 |
| Reply morador→síndico | ❌ Opção A canônica venceu | RN-CM-01 |
| E2E tests Connect Me | Parciais (F17 stubs test, não vi integration completo) | — |

---

## 18. Gaps detectados

### Divergências entre fontes

| ID | Divergência | Gravidade | Fontes |
|---|---|---|---|
| **Q2 / G-Q2** | ~~Quota Morador Base: matriz funcional diz **0/ano** (F3:148)~~ **RESOLVIDA 2026-04-24 via D-094** — canônico **2/ano**, matriz corrigida na Fase 11. | 🟢 Resolvido | F3, F4, F1 |
| **Q8** | Quota Empresa→Empresa: 3 redações. `personas-and-plans` "não aplicável" / `functional-matrix` "E→E apenas" / `commercial.md` "ilimitado/configurável" / Obsidian "Apenas Plus e Pro". | 🟡 Medium | F3:156-157, F1:126, F6:122 |
| **Q-State-CM** | State machine: código tem `pending/accepted/declined/expired` (4); doc canônico tem `open/interested/proposal_sent/em_negociacao/closed/expired` (6+). | 🟡 Medium | F4:51, F10:68 |
| **Q-Table-multi** | Docs falam `connect_me_resident_requests`, `connect_me_enterprise_requests`, `connect_me_neighborhood_requests` (F1:104,129,153); código tem **tabela única** `connect_me_requests` com `flow` enum (F23:30). | 🟡 Medium | F1, F23 |
| **Q-M12** | Usuário menciona "M12 (Morador votação/Connect Me)"; inventário-telas F5:33 diz M12 = Avaliar Gestão. Onde está a tela de Connect Me do morador? | 🟠 High | F5, enunciado |
| **Q-Decision-11** | Reply morador → síndico: opção A (frio) vs B (ticket). Pendente decisão cliente. Atual implementação = A. | 🟠 High | F4:87-91 |
| **Q-FieldsMissing** | `service_category, urgency, budget_range, partnership_type, promotion_type` existem nos docs mas não são colunas tipadas na `connect_me_requests` (F23). Provavelmente embutido em `message`. | 🟡 Medium | F23 vs F1 |
| **Q-DeclineReasonEnum** | Docs listam 6 motivos estruturados (F4:55-61); entity aceita string livre ≤ 2000 chars (F10:22). Enforcement só no frontend. | 🟢 Low | F4 vs F10 |
| **Q-Categories** | Enunciado diz **32 categorias + 202 subcategorias**. Arquivo real (F8) tem **31 categorias + 183 subcategorias**. | 🟡 Medium | F8, enunciado |
| **Q-Quotas-Monthly-Annual** | Obsidian Comercial F6:146: "Quotas são por /ano (PDF Matriz Funcional) ou por /mês? Algumas seções do requirements mencionam /month." | 🟡 Medium | F6:146 |

### Gaps de implementação

| ID | Gap | Gravidade | Ação sugerida |
|---|---|---|---|
| **G-01** | **Marketing Link não implementado**: sem entity, sem migration, sem handler. Só docs. | 🟠 High | Spec + implement em Sprint M2. |
| **G-02** | **Sem UNIQUE (connect_me_request_id, company_id)** em `proposals`. Enunciado pede essa invariante; DB não enforça. | 🟡 Medium | Adicionar `UNIQUE(connect_me_request_id, company_id) WHERE status != 'withdrawn'` em migration nova. |
| **G-03** | **State machine produto estendida** (`open/interested/proposal_sent/em_negociacao`) não refletida em código — só 4 estados. | 🟡 Medium | Decidir: alinhar código à documentação (adicionar estados) OU reescrever doc para refletir implementação. |
| **G-04** | **Tabela `lgpd_logs` não encontrada** nas migrations lidas. Referenciada em F1:190 + F4:203. Pode estar em migration fora do range commercial. | 🟠 High | Verificar migrations `identity/` ou `compliance/`. |
| **G-05** | **Tela de Connect Me do Morador ausente do inventário** (M1-M15 não cobre). | 🟡 Medium | Criar M16 "Connect Me Morador" ou desambiguar M11/M12. |
| **G-06** | **Contagem divergente de categorias/subcategorias**. | 🟢 Low | Fechar count com cliente. |
| **G-07** | **Versionamento de eventos NATS** não documentado. | 🟡 Medium | Criar ADR de schema versioning. |
| **G-08** | **Admin dashboard para HarassmentReport** não mapeado em inventário de telas. | 🟡 Medium | Adicionar AD1-ADN jornada admin. |
| **G-09** | **Rascunho Redis** para Connect Me (SHOULD) não implementado — sem endpoint/key TTL documentado. | 🟢 Low | Spec se virar MUST. |
| **G-10** | **Sugestão automática de empresas** (SHOULD F1:68) não implementada — recomendação por categoria. | 🟢 Low | Sprint futuro (search/ML). |
| **G-11** | **Flow "Admin intervenção em Connect Me"**: admin pode visualizar request específico? Docs não definem. Anti-LGPD se sim indiscriminadamente. | 🟡 Medium | ADR explícito. |
| **G-12** | **LGPD purge automática após 5 anos** não agendada (cron). | 🟡 Medium | Job LGPD housekeeping. |
| **G-13** | **Quota shared entre 3 fluxos de síndico** (F1 Fluxo 1 + Fluxo 4 consomem `connect_me`) pode confundir UX; doc não aborda. | 🟢 Low | Doc UX. |
| **G-14** | **`sender_ip` como coluna TEXT sem proteção adicional** (LGPD considera IP dado pessoal em contexto). OK por RN-CM-03, mas retention policy precisa estar declarada. | 🟡 Medium | ADR retention. |
| **G-15** | **Sem rate limiting específico** para Connect Me além da quota anual. Empresa maliciosa pode encher mailbox de Connect Me→Empresa de um concorrente em 1 dia. | 🟡 Medium | Rate limit por receiver (ex: max 5 connect_me pending de senders distintos por dia). |

---

## 18.5 Telas & Endpoints

Rastreabilidade consolidada em [[../../03-subprojects/traceability]] §2.2 (Connect Me) + §2.9 (Marketing Link — E14/E16/MK8).

**Escopo UI**:
- Síndico: S18 (hub) · S19 (criar) · S20 (receber M→S) — 3 telas web
- Empresa: E1 · E2 (recebidos) · E3 (recusar) · E4 (interesse + proposta) · E5 (pipeline) · E7 (deals) · E14 (agência) · E16 (marketing link) — 8 telas
- Mobile: 0 (fora M1)

**Endpoints-chave** (`/api/v1/commercial/*`):
- `/connect-me` POST · GET `?sender|receiver`
- `/connect-me/:id/respond` POST (accept/reject 6 motivos)
- `/connect-me/:id/interested` POST (reveal + LGPD log)
- `/connect-me/me/summary` GET (quota + stats — 🟡 inferido)
- `/proposals` · `/:id/draft` · `/:id/submit`
- `/closed-deals/own` · `/:id/company-sign` · `/signature-status`
- `/marketing-link/received` · `/:id/attended` · `/:id/view`

**Invariantes observáveis em UI**: INV-037 (unidirecional, sem reply) · INV-038 (proposta única) · INV-039 (imutável pós send) · INV-040 (deal imutável pós dupla assinatura) · INV-041 (auto-publica timeline) · INV-045 (quota decrementa publish) · INV-046 (reset 1/jan BRT) · INV-047 (auto-close 48h) · INV-048 (LGPD log reveal obrigatório) · INV-049 (agência zero-access deals/CM).

**Fluxo crítico** (traceability §7.3): S19 → POST CM → E2 → E4 interested (reveal) → proposta → síndico aceita → E8 dupla assinatura → INV-041 evento → timeline automática.

**Sprint**: backend Sprint 4 ✅ · UI Sprint 10 🟡 (M1 core S18-S20 bloqueador).

---

## 19. Referências cruzadas

### Documentos Master Síndico v2 (esta pasta)

- [[01-governanca-institucional]] — Timeline consumer do evento `deal.confirmed`.
- [[03-reputacao]] — CompanyEvaluation feed.
- [[04-conteudo-videos]] — visibility-revocation via SupplierVoteSession.
- [[05-assembleia-live]] — supplier voting parte da assembleia.
- [[08-vizinhanca]] — destino do Fluxo 4 (neighborhood_benefits).
- [[09-compliance]] — audit_trail + lgpd_logs + governance score.
- [[10-marketing-agencias]] — Marketing Link (§7) + delegação Zitadel E13-E14.
- [[_moc]] — índice geral dos 11 sub-produtos.

### Documentos Backend (fontes)

- `backend/.kiro/specs/master-sindico/requirements/commercial.md` — canônico Req 18–27.3.
- `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` — Req 33-47, 61-68, 102, 103.
- `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` — quotas × persona × plan.
- `backend/.kiro/specs/master-sindico/requirements/identity.md` — Req 2 (ABAC), Req 10 (quotas).
- `backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md` — mapping persona × plan.
- `backend/internal/modules/commercial/**` — implementação Go (aggregates, use cases, handlers).
- `backend/internal/shared/providers/database/migrations/003*.sql` — schema commercial.

### Content (fontes canônicas produto)

- `Development/Content/03-Modulos/Connect-Me-4-direcoes.md` — fonte canônica 4 fluxos + Marketing Link + quebras.
- `Development/Content/02-Jornadas/Inventario-Telas.md` — 114+ telas mapeadas.
- `Development/enterprise-catergories-subcategories-enum.schema.ts` — 31 categorias + 183 subcategorias.

### Obsidian Vault (legado)

- `Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Comercial.md`
- `Obsidian Vault/Clients/MasterSindico/01 - Arquitetura de Sistema/System Architecture - Commercial.md`
- `Obsidian Vault/Clients/MasterSindico/01 - Arquitetura de Sistema/Fluxo Connect Me.canvas` (não lido, canvas binário)

### Regras de negócio (vault legado, referenciadas)

- `regras-de-negocio/regras-canonicas.md` — Regra de Ouro 3 (Connect Me ≠ Chat), Regra 10 (LGPD), Decision 5 (4 fluxos), Decision 9 (M→S frio), Decision 10 (quotas anuais), Decision 11 (reply pendente).
- `regras-de-negocio/quebras-detectadas.md` — Q2 (quota morador), Q8 (E→E).

### ADRs relevantes

- ADR-0032 — Planos globais Stripe-style (aboliu N1/N2/N3).
- D-056 — Agnosticismo de provedor (IPaymentGateway, IEmailProvider, etc).
- D-134 (revoga D-058/D-072) — Admin = `apps/admin` dedicada no monorepo web.
- ~~D-065 — Morador Base Connect Me = 0/ano~~ (sobrescrito por **D-094** Fase 11 = **2/ano**).

---

## Metadata da destilação

- **Método**: leitura arquivo-por-arquivo, zero grep, citações literais com `path:linha`.
- **Fontes lidas**: 28 arquivos (§1).
- **Volume**: ~5000 linhas de fonte.
- **Dual-check**: PENDENTE — agente separado em contexto limpo deve revisar este arquivo contra §18 gaps.
- **Status**: `destilado` — pronto para revisão QA + Pentester + Arquiteto BeyondCorp.
