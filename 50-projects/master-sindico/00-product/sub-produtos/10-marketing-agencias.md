---
title: "10-marketing-agencias — Sub-produto Agências de Marketing (Fase 8 destilado)"
type: note
tags: [sub-produto, master-sindico, v2, fase-8, marketing, agencia, delegation, abac, zitadel]
source: agregado multi-fonte (ver §1)
created: 2026-04-23
updated: 2026-04-23
status: destilado
depends_on: [D-059, D-060, D-057 (ADR-0032), D-058, D-061, D-056]
---

# 10-marketing-agencias — Agências de Marketing (sub-produto interno de primeira classe)

> **D-059/D-060 + D-095/D-102 (Fase 11 CONFIRMA CANÔNICO)**: Agência de Marketing **NÃO é produto externo**, **NÃO é actor delegado via impersonation token**. É **sub-produto interno próprio** da plataforma Master Síndico. Agência tem **login próprio via Zitadel** (role `marketing`), tem seu **painel admin próprio** (MK1-MK8) e administra marketing digital de **empresas condominiais clientes** via **grants ABAC explícitos** (BeyondCorp). O modelo antigo "agência = actor delegado, não é tenant, sem login" (legado `Domínio - Conteúdo.md` Req 30, `legacy-docs/Matriz Funcional.md`) **foi substituído** pelo modelo de conta própria + delegação scoped — ver D-059, D-060, e **D-095 Fase 11 (2026-04-24) que fecha definitivamente Q4** + **D-102 Fase 11 que desambigua painel admin **da agência** (este sub-produto MK1-MK8) vs painel admin **da plataforma** (GAP `03-subprojects/web/ui-catalog/admin/GAP-admin-master-sindico.md`, backlog M2/M3)**.

---

## 1. Fontes consultadas

Leitura exaustiva arquivo-por-arquivo (método Fase 8 — D-069). Cada trecho citado traz `arquivo:linha (data)`.

### Backend `.kiro/specs/master-sindico/requirements/`

- `commercial.md:479-481` (2026-04-22) — Commercial é domínio cross-domain com Content; concentra Connect Me, propostas, deals, registros, avaliação. Marketing Link **não está explicitamente destilado em commercial.md** — menção indireta em Req 19 (Connect Me) e referência de tabela `marketing_link_requests` nos arquivos sucessores. Marketing Link é **feature distinta** de Connect Me (agência), conforme descrito em `04-telas-jornadas.md:144-149` e Inventário MK8.
- `identity.md:52` (2026-04-22): "Roles primárias (1:1 com Zitadel, em inglês): `syndic`, `resident`, `enterprise`, `marketing`, `local_company`, `admin`"
- `identity.md:91`: "Personas que NÃO se registram: `marketing` (Agência) — é convidada por uma empresa via token (Sprint 5); `admin` — provisionada manualmente no Zitadel (sem fluxo de auto-registro)"
- `personas-and-plans.md:21` (2026-04-22): "Agência de Marketing | Gerencia conteúdo em nome de empresa | Actor delegado (não é tenant) | Sem perfil próprio, vinculado a Empresa" — **lido sob D-060**: nomenclatura antiga; hoje a agência **tem sim login próprio** Zitadel; "não é tenant" deve ser interpretado como "não opera tenant condominial próprio" (ela não vende condomínio; administra no tenant da empresa cliente).
- `personas-and-plans.md:51-53`: tier antigo `marketing_standard` — **ABOLIDO por ADR-0032/D-057**; hoje o catálogo de planos é global `{trial, base, plus, pro}` universal e ortogonal à role.
- `functional-matrix.md:114-125` (2026-04-22) — seção "Agência de Marketing (Delegated Actor)": tabela permissões reproduzida em §5 abaixo (tier único para agência; hoje só atua em empresas com plano `plus` ou `pro` — ver `functional-matrix.md:87` "Agência Marketing | Delegar | ❌ | ✅ | ✅" para Trial/Plus/Pro Empresa).

### Código Go (`backend/internal/modules/…`)

- **Não acessível neste repo** (apenas documentação destilada). Estado do código confirmado via notas `CLAUDE.md:5 Bounded contexts`: `content` e `commercial` existem em produção; nenhum módulo dedicado `marketing/` — agência vive encaixada em `content` (videos) + `commercial` (marketing_link_requests) + `identity` (role `marketing`). `90-ingested/from-vault-50-projects-master-sindico/CLAUDE.md:95` lista `commercial` como responsável por "empresas-sindicas"; não cita módulo agência. Gap: ver §18.

### Content (Inventário de Telas + Jornada)

- `_archive/content-discovery-2026-03/jornada-empresa-marketing.md:28-271` (2026-04-22, convertido do PDF canônico do cliente "JORNADA DA EMPRESA DE MARKETING"): **documento mais fiel**. Reproduz literalmente MK1-MK8, limitações, integração com outras jornadas, regras de Marketing Link não-vinculante. Este arquivo é a **fonte primária de UX** para o sub-produto.
- `90-ingested/_reconciliacao-dev/04-telas-jornadas.md:23 (row MK)` (2026-04-23): família MK = 8 telas, v2 já ~100%.
- `90-ingested/_reconciliacao-dev/04-telas-jornadas.md:153-167`: inventário resumido MK1-MK8 sob a premissa antiga (impersonation scoped-token); **reclassificado por D-060** como login próprio + grant ABAC.
- `90-ingested/_reconciliacao-dev/04-telas-jornadas.md:144-149` (E16 Marketing Link): "E16 Marketing Link ≠ Connect Me: formulário de interesse; vínculo só via E13 convite formal. Tabela backend: `marketing_link_requests` (separada de ConnectMe)". **Confirma** Marketing Link como **feature distinta**.

### Obsidian Vault do cliente (D-068)

- `Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Identidade.md:97,115,127,219` (cliente, Abril/2026): roles incluem `agencia`; role `agencia` = "Delegado da Empresa. Apenas conteúdo institucional"; registro suportado para `agencia`; onboarding agência = "perfil da agência, tipos de serviço, portfólio" — o próprio onboarding desmente "agência não se registra". Interpretação reconciliada (D-060): agência **se registra sim** quando convidada, mas o **primeiro ato** é aceitar token de convite da empresa; ela **não se cadastra sozinha "do zero"** comprando plano próprio.
- `Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Identidade.md:202` (tabela Trial): Empresa Trial = bloqueia "convidar agência" — confirma que convidar agência **é feature Pagante da Empresa** (Plus/Pro, não Trial).
- `Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Conteúdo.md:77-88` (Req 30): regras de delegação — 1 agência ↔ N empresas; upload em nome da empresa; vídeo atribuído à empresa; restrições ≡ §6.
- `Clients/MasterSindico/03 - Modelagem e Dados/Enums e Listas Mestre.md:84-89` (cliente, Abril/2026): **fonte canônica de enums** — `Tipos de Suporte Marketing`, `Marketing Link Priority`, `Marketing Link Status`, `Status da Agência`. Todos reproduzidos em §8.

### STATE.md (decisões vivas v2)

- `master-sindico-v2/STATE.md:354-364` (D-060): Agência = sub-produto interno próprio (nova leitura canônica)
- `master-sindico-v2/STATE.md:497-502` (D-059, Fase 7 redundante com D-060): "substitui D-011 (agência tenant vs actor)"
- `master-sindico-v2/STATE.md:313-337` (D-057 + ADR-0032): abolição de `marketing_standard`
- `master-sindico-v2/02-architecture/adr/0032-global-plans-stripe-style.md:26,72,176` — slug `marketing_standard` proibido; role ortogonal ao tier

### Reputação + Escopo-7

- `vault/50-projects/master-sindico/contextos/novo escopo-7.md:603-604` (cliente PDF): "Empresa de Marketing: Criação de cursos, vídeos institucionais, Marketing Link (tag por vídeo para gerar leads), participação fórum/lives" — **sugere capabilities mais amplas** que MK1-MK8 (criação de cursos, fórum, lives). Esse escopo amplo **não aparece** no PDF canônico MK1-MK8 nem na matriz funcional — **registrar como gap (§18)** para confirmação do product owner.

### Reconciliação

- `master-sindico-v2/11-audit/audit-log/2026-04-23-reconciliacao-v2-vs-dev.md` — confirma MK1-MK8 100% mapeado em v2; pendência: reclassificar o modelo de operação (impersonation → grant ABAC) e criar sub-produto próprio.

---

## 2. Propósito

Área **interna e dedicada** da plataforma Master Síndico onde **agências de marketing** — tipicamente agências terceirizadas que prestam serviço de marketing digital — **administram o marketing institucional de múltiplas empresas condominiais clientes** dentro do ecossistema.

A agência **não substitui** a Empresa cliente, **não opera no lugar dela** em decisões comerciais, contratos, Connect Me ou finanças. Ela é uma **mão especializada em conteúdo institucional**: produz vídeos, edita metadados, mede performance, recebe intenções de contato comercial (Marketing Link) e reporta analytics cross-empresa à agência-mãe.

A existência dela como sub-produto interno (não como serviço externo) tem 3 razões:

1. **Lock-in operacional desejado** da plataforma: o conteúdo é feito lá dentro, métricas ficam lá, templates ficam lá; a empresa cliente enxerga tudo auditável.
2. **Segurança por desenho**: agência **nunca vê dados sensíveis da empresa** (Connect Me, propostas, registros, plano diretor, dados pessoais de síndicos/moradores). Grant ABAC é cirúrgico.
3. **Identidade preservada**: o vídeo é da **empresa**, não da agência. Se a empresa trocar de agência, nada muda pro síndico/morador — continua vendo conteúdo da Empresa X.

Posicionamento do cliente (`jornada-empresa-marketing.md:17-24`): _"A jornada da agência de marketing tem como objetivo apoiar empresas prestadoras de serviço na construção da sua presença institucional dentro do ecossistema da plataforma. A agência atua exclusivamente na gestão de conteúdo institucional das empresas. Ela não possui acesso a dados de condomínios, síndicos ou registros técnicos."_

---

## 3. Como é integrado (identidade + delegação)

### 3.1 Login próprio via Zitadel (D-059, D-060)

- Agência cria **User** na plataforma via convite de empresa cliente (token) — ver §10 Marketing Link **não** gera conta; quem gera conta é **E14 "Convidar Agência"** (jornada E — Empresa).
- User carrega `role=marketing` no Zitadel (enum `user_role` — `identity.md:52,130`).
- Autenticação **idêntica** ao resto da plataforma: Authorization Code Flow + PKCE via Zitadel (`identity.md` Req 1), cookie httpOnly no domínio `.mastersindico.com.br`, fallback Bearer mobile, 1-device policy (Rule 8).
- Cache Redis `ms:auth:{zitadelUserID}` TTL 5min (`identity.md:20-22`).
- 1 dispositivo ativo por conta (Rule 8, `identity.md:25`).

### 3.2 Delegação ABAC por empresa (BeyondCorp — D-060)

A agência **não tem tenant próprio** (não vende condomínio; não é empresa condominial). Ela opera **no tenant da empresa cliente**, mas com **grants explícitos** que limitam **quais ações** pode executar e **em qual empresa**.

Cada vínculo agência↔empresa é representado por um **DelegationGrant** (aggregate novo — ver §7):

```
DelegationGrant
  id:                 UUID
  agency_user_id:     FK User (role=marketing)
  company_tenant_id:  FK Tenant (role=enterprise, tenant_type='company')
  granted_by:         FK User (empresa_administrador que aceitou/emitiu)
  scopes:             enum_set {videos:upload, videos:edit, analytics:read, marketing_link:receive, marketing_link:manage}
  status:             enum {requested, accepted, active, revoked}
  invited_at:         timestamp
  accepted_at:        timestamp?
  revoked_at:         timestamp?
  last_access_at:     timestamp
  token_ttl_days:     int (ex: 90; rotação por policy)
  audit_trail_id:     FK AuditEntry
```

O middleware ABAC estende a matriz padrão (D-057, ADR-0032 §ABAC engine) com um passo extra **antes** do check (role × tier × action):

```go
// pseudo — extensão do Engine.Allow para role=marketing
if user.Role == RoleMarketing {
    grant, err := grantRepo.FindActive(user.ID, ctx.TargetTenantID())
    if err != nil { return ErrForbidden("NO_DELEGATION") }
    if !grant.HasScope(action) { return ErrForbidden("SCOPE_NOT_GRANTED") }
    // ABAC continua: avalia matriz (role=marketing, plan.tier=N/A, action, resource)
}
```

O **plano ativo** que rege as permissões não é da agência — é o **plano da empresa cliente**. Se a empresa está em `plus`, agência herda quotas de vídeo `plus` da empresa; se empresa vai pra `pro`, quotas sobem automaticamente. Agência não paga plano próprio (no MVP — ver §18 gaps).

### 3.3 Fluxo de vinculação (resumo)

Leitura exaustiva em §10 e §13. Alto nível:

```
1. Empresa (administrador) acessa E13 "Gestão de Agência" → E14 "Convidar Agência"
2. Empresa informa email + escopos desejados → sistema gera token de convite
3. Email sai via IEmailProvider (Resend — D-067)
4. Agência clica link → fluxo Zitadel (registra se new user + login se existente)
5. Agência aceita termos + confirma → DelegationGrant.status = 'accepted' → 'active'
6. Evento DelegationAccepted publicado (NATS JetStream)
7. Agência entra em MK1 — vê empresa vinculada em MK2
8. A qualquer momento, Empresa (E15 "Gerenciar Acessos Agência") pode revogar → status='revoked' → todos os tokens invalidados imediatamente; evento DelegationRevoked
```

---

## 4. Personas envolvidas

| Persona | Role Zitadel | Papel no sub-produto |
|---|---|---|
| **Agência de Marketing** | `marketing` | Opera MK1-MK8; produz vídeo; lê analytics; gere Marketing Link recebido; opera apenas escopos concedidos. |
| **Empresa cliente (administrador)** | `enterprise` + `role_type=administrator` | Gere vínculo: convida (E14), concede/revoga escopos (E15), aceita/rejeita propostas em Marketing Link. **Único** que pode alterar plano da empresa, adicionar representante legal, aceitar contratos. |
| **Empresa cliente (operador)** | `enterprise` + `role_type=operator` | **Não** gerencia agência (só administrador faz). Pode ver conteúdo publicado pela agência como se fosse interno. |
| **Empresa representante legal** (pessoa natural vinculada à empresa como sócia/diretora via contrato social) | `enterprise` + `role_type=administrator` com flag `is_legal_representative=true` | Aprova delegação em compliance terms (Req 33-42 LGPD). **Assina** termo de delegação. Pode ser a mesma pessoa que o administrador comum ou outra. |
| **Admin MS** | `admin` | **Nunca** opera em nome de agência (D-061). Acessa tudo em moderação e suporte, via bypass ABAC auditado (ver `03-subprojects/admin-privileges.md`). |
| **Síndico / Morador / Parceiro da Vizinhança** | — | **Não interagem com agência**. Moradores só veem vídeos institucionais publicados pela agência em uma situação específica: durante **votação de fornecedor em assembleia** com `autorizar_exibicao_votacao=true` (Req 29 `requirements.md:314`; `jornada-empresa-marketing.md:158-164`). Após votação, acesso é revogado (Req 103). Síndico **nunca** interage com agência (`jornada-empresa-marketing.md:291-292`: "síndico nunca interage com agência"). |

---

## 5. Tiers × role (planos globais Stripe-style — ADR-0032)

Conforme D-057/ADR-0032, tiers `{trial, base, plus, pro}` são **universais**. Role `marketing` **não compra plano próprio no MVP**. As quotas/features disponíveis à agência derivam integralmente do plano da **empresa cliente** sobre a qual ela tem DelegationGrant.

| Capability (escopo da agência) | Empresa Trial | Empresa Base | Empresa Plus | Empresa Pro |
|---|---|---|---|---|
| **Empresa pode convidar agência** | ❌ (Trial bloqueia "convidar agência" — `Domínio - Identidade.md:202`) | ❌ (não existe tier "base" pra empresa na matriz — empresa passa de Trial → Plus ou Pro conforme `functional-matrix.md:81-96`) | ✅ | ✅ |
| **Agência publica vídeo em nome da empresa** | ❌ | ❌ | 8/mês (quota da empresa `functional-matrix.md:85`) | 12/mês |
| **Agência edita metadados do vídeo** | ❌ | ❌ | ✅ | ✅ |
| **Agência lê analytics da empresa** | ❌ | ❌ | ✅ | ✅ |
| **Agência recebe Marketing Link da empresa** | ❌ (só Pagante — `functional-matrix.md:79-82`; Marketing Link é capability da Empresa enviar) | ❌ | ✅ | ✅ |
| **Agência gere Marketing Link recebido** | — | — | ✅ | ✅ |
| **Agência consulta cross-empresa analytics** (MK7) | — | — | Requer grant ativo em ≥ 1 empresa | Idem |

Referências cruzadas: `functional-matrix.md:114-125` reproduzido aqui:

| Funcionalidade | Ação | Permitido? |
|---|---|---|
| Vídeos | Upload em nome da empresa | ✅ |
| Vídeos | Editar metadados | ✅ |
| Vídeos | Ver analytics | ✅ |
| Deals | Visualizar | ❌ |
| Connect Me | Enviar/receber | ❌ |
| Usuários | Gerenciar | ❌ |
| Propostas | Acessar | ❌ |
| Token | Revogação automática após TTL | ✅ |

Slugs compostos (`marketing_standard`, `enterprise_plus`) **proibidos** (ADR-0032 §"Regra dura").

---

## 6. Modelo de delegação ABAC (detalhado)

### 6.1 Conta

- Agência tem conta `User` com `role=marketing` (Zitadel + PG mirror, `identity.md:52,96,120`).
- `users.zitadel_id` UNIQUE (invariante 1 do Identity).
- `users.role ≠ ''` — middleware rejeita 403 `NO_ROLE_ASSIGNED` antes do UPSERT (Req 1 T15).
- Password / refresh token gerenciados **exclusivamente** pelo Zitadel (Req 3).

### 6.2 Grant por empresa (scoped)

Cada DelegationGrant é emitido por **uma** empresa cliente, cobrindo **exatamente uma** empresa. Agência atendendo 5 empresas = 5 grants.

**Escopos canônicos** (ver §8 enum `TokenScope`):

- `videos:upload on:company:Y` — criar vídeo no tenant da empresa Y
- `videos:edit on:company:Y` — editar título, descrição, tags, ocultar/reexibir
- `analytics:read on:company:Y` — ler métricas de vídeos da empresa Y
- `marketing_link:receive on:company:Y` — receber solicitações de contato dirigidas à agência (MK8)
- `marketing_link:manage on:company:Y` — marcar como atendido, registrar proposta enviada

O middleware constrói a **sessão de trabalho** (MK2 "Seletor Empresa") — ao selecionar a empresa X, os claims de tenant viram `company_tenant_id=X`. Requests subsequentes carregam esse escopo até troca ou logout.

### 6.3 Proibições duras (NUNCA pode)

Conforme `jornada-empresa-marketing.md:275-286`, `functional-matrix.md:121-124`, `Domínio - Conteúdo.md:87`:

- ❌ Alterar plano da empresa (billing)
- ❌ Adicionar/remover representante legal
- ❌ Aceitar/assinar contratos (propostas, deals, adendos)
- ❌ Ver **Connect Me** (qualquer direção — S→E, E→E, E→M, etc.)
- ❌ Ver **registros de execução** ou **atividades técnicas**
- ❌ Ver **comunicados técnicos** ou **comunicados pós-deal**
- ❌ Ver **linha do tempo do condomínio**
- ❌ Ver **dados de síndicos** (nome, email, telefone, CPF) — mesmo em caso de "Tenho Interesse"
- ❌ Ver **dados de moradores** (CPF, unidade, fração ideal)
- ❌ Ver **dashboard de governança** do síndico
- ❌ Gerenciar usuários da empresa
- ❌ Acessar propostas, deals, execuções, avaliações
- ❌ Acessar perfil público da empresa para **editar** (só pode publicar vídeo que entra como conteúdo institucional)

### 6.4 Revogação imediata

`Domínio - Conteúdo.md:87` + `jornada-empresa-marketing.md:275-286`: qualquer alteração no DelegationGrant (status=`revoked` ou escopo retirado) invalida:

- Token Redis `ms:auth:{zitadelUserID}` — purgado (ou stale até 5min, conforme D-001 pendente)
- Sessão `sessions` — força re-login da agência
- Requests em voo com `company_tenant_id` afetado — retornam 403 `SCOPE_REVOKED`
- NATS event `DelegationRevoked` publicado — consumers de cache/auditoria reagem

### 6.5 Audit log obrigatório

Toda ação delegada gera entry append-only (imutável) em `audit_entries` (aggregate `AuditEntry` em `01-domain/aggregates/AuditEntry.md`):

```
AuditEntry {
  id, actor_user_id=<agency>, on_behalf_of_tenant_id=<company>,
  action=<"videos.upload"|"videos.edit"|...>,
  resource_id, scope_used, occurred_at, ip, device_fingerprint, request_id
}
```

MK8 da agência (audit log delegation) expõe **leitura do histórico das próprias ações** da agência. A empresa vê o **mesmo log filtrado pelas ações que ocorreram no tenant dela**. Admin MS vê tudo via privilégios (D-058/D-061).

### 6.6 Rotação e TTL de token (plano)

`functional-matrix.md:125`: "Token | Revogação automática após TTL | ✅". Política prevista:

- TTL do DelegationGrant: **90 dias default**; empresa pode estender.
- Ao expirar: `status='expired'`, ação de re-confirmação pela empresa (clique em "renovar").
- Sessão OIDC Zitadel tem seu próprio TTL (5min access, refresh do Zitadel).

---

## 7. Aggregates (DDD)

Nomes sugeridos e campos mínimos. Nenhum modelado formalmente em `01-domain/aggregates/` ainda (gap §18).

### 7.1 MarketingAgencyLink (vínculo operacional)

Representa o vínculo ativo agência↔empresa após aceite. **Alias** conceitual de DelegationGrant quando focado no ângulo "relação agência-empresa". Pode ser modelado como **o próprio DelegationGrant** ou como projeção read-only.

```
MarketingAgencyLink (read projection)
  agency_user_id, agency_display_name, agency_logo_url,
  company_tenant_id, company_display_name,
  grant_id, status, granted_scopes,
  started_at, last_activity_at, videos_published_count, analytics_views_total
```

Fonte: `Domínio - Conteúdo.md:82-88` Req 30.

### 7.2 MarketingLinkRequest (Marketing Link — intenção de contato)

```
MarketingLinkRequest
  id:                 UUID
  from_company_tenant_id: FK Tenant (empresa interessada)
  to_agency_user_id:  FK User (agência candidata)
  support_type:       enum MarketingSupportType (6)
  priority:           enum SupportPriority (3)
  responsavel_name:   string (pessoa de contato da empresa)
  responsavel_phone:  string
  responsavel_email:  string
  message:            text
  status:             enum MarketingLinkStatus (6)
  sent_at:            timestamp
  viewed_at:          timestamp?
  proposal_sent_at:   timestamp?
  answered_at:        timestamp?  (atendido/accepted/rejected)
  terms_version:      string  (LGPD log)
```

Tabela backend sugerida: `marketing_link_requests` (confirmada em `04-telas-jornadas.md:149`).

**Diferença crítica vs Connect Me**: Marketing Link **não cria vínculo automático**. Cria **intenção de conversa**. O vínculo formal (DelegationGrant/MarketingAgencyLink) só nasce via **E14 convite** (`jornada-empresa-marketing.md:265-271`).

### 7.3 DelegationGrant (contrato de delegação)

Campos já listados em §3.2. Este é o **aggregate root** do sub-produto. Invariantes:

- `scopes ≠ ∅` (grant sem escopo = inútil)
- `status=active` ⇒ `accepted_at IS NOT NULL`
- `revoked_at IS NOT NULL` ⇒ `status='revoked'`
- 1 grant ativo por (agency_user_id, company_tenant_id) — `UNIQUE WHERE status IN ('requested','accepted','active')`

---

## 8. Enums canônicos

Fonte: `Clients/MasterSindico/03 - Modelagem e Dados/Enums e Listas Mestre.md:84-89` (Abril/2026) + síntese ABAC.

### 8.1 `TokenScope` (scopes de delegação — deriva D-060)

```
videos:upload | videos:edit | analytics:read | marketing_link:receive | marketing_link:manage
```

### 8.2 `MarketingSupportType` (6 valores — tipos de suporte solicitados via Marketing Link)

```
producao_videos_institucionais
gestao_conteudos_tecnicos
estrategia_posicionamento
gestao_redes_sociais
consultoria_marketing
outro
```

### 8.3 `SupportPriority` (3 valores — prazo desejado pela empresa)

```
imediato
proximos_30_dias
sem_prazo_definido
```

### 8.4 `MarketingLinkStatus` (6 valores — ciclo de vida da intenção)

```
sent          — formulário enviado pela empresa
viewed        — agência abriu o registro em MK8
proposal_sent — agência mandou proposta via canal externo (email/telefone) e marcou aqui
atendido      — agência marcou como "já falei com a empresa"
accepted      — empresa confirmou que virou contrato (fora da plataforma) ou enviou E14 convite
rejected      — empresa desistiu / agência recusou
```

### 8.5 `Status da Agência` (fonte do cliente — vínculo)

```
convite_enviado | ativa | encerrada
```

Mapeia para DelegationGrant.status (`requested` → `convite_enviado`; `active` → `ativa`; `revoked` → `encerrada`).

### 8.6 Role Zitadel

```
marketing   (role=`marketing`)
```

### 8.7 Tipos de vídeo que a agência publica (12 valores — ver `04-conteudo-videos.md` quando destilado)

```
apresentacao_institucional | apresentacao_equipe_tecnica | demonstracao_servico
explicacao_tecnica | boas_praticas_condominiais | orientacao_preventiva
caso_real_atendimento | treinamento_tecnico | explicacao_legislacao_normas
dicas_manutencao | conteudo_educativo_sindicos | conteudo_educativo_moradores
```

Fonte: `Enums e Listas Mestre.md:108` + `jornada-empresa-marketing.md:127-140` (MK4 "Tipo de vídeo").

---

## 9. Telas MK1–MK8 (tela-por-tela, fluxo completo)

Reproduzido fielmente de `_archive/content-discovery-2026-03/jornada-empresa-marketing.md:28-272` (fonte canônica PDF cliente). Campos entre parênteses foram deduzidos de coerência com a família E (E13-E16).

### 9.1 MK1 — Painel da Agência (dashboard multi-empresa)

**Rota (sugerida)**: `/agencia/painel`
**Caminho**: App → Painel Empresarial → Perfil Agência de Marketing (fonte: `jornada-empresa-marketing.md:32-33`)

**Mensagem institucional (literal)**: "Este é o espaço onde sua agência administra os conteúdos institucionais das empresas que confiaram à sua equipe a gestão da comunicação dentro da plataforma Master Síndico. Aqui você pode publicar vídeos institucionais e acompanhar o desempenho desses conteúdos."

**Cards** (5):
- Empresas atendidas → MK2
- Publicar vídeos (atalho — escolhe empresa primeiro) → MK2 → MK4
- Biblioteca de vídeos (agregada) → MK3 (com filtro cross-empresa)
- Dashboard de desempenho → MK7
- Marketing Link recebido → MK8 (badge com contagem de `status=sent`)

**Notificações** (fonte `:53-57`): badge/alert quando:
- Empresa envia convite (novo DelegationGrant pendente)
- Empresa envia Marketing Link (novo MarketingLinkRequest com `status=sent`)

### 9.2 MK2 — Empresas atendidas (seletor de empresa, ativação de scoped session)

**Rota**: `/agencia/empresas`
**Caminho**: MK1 → Empresas atendidas

**Mensagem institucional (literal)**: "Aqui estão as empresas que autorizaram sua agência a administrar seus conteúdos institucionais dentro da plataforma."

**Lista exibida** (campos por linha):
- Empresa (razão social / logo)
- Segmento da empresa (categoria de serviço)
- Cidade
- Data de início da parceria (DelegationGrant.accepted_at)

**Botão**: "Gerenciar empresa" → MK3

**Regra (`:84-86`)**: "Empresas aparecem apenas após aceitar convite enviado pela empresa prestadora." → filtro `WHERE status='active'`.

**Ação de seleção**: dispara **ativação de sessão scoped** (claim `company_tenant_id` na ABAC context). A partir daqui, todas as ações MK3+ são **escopadas** nessa empresa.

### 9.3 MK3 — Gerenciar Empresa (biblioteca + operações no tenant)

**Rota**: `/agencia/empresas/:companyId`
**Caminho**: MK2 → Gerenciar empresa

**Mensagem institucional (literal)**: "Nesta área sua agência administra os conteúdos institucionais da empresa dentro da plataforma."

**Cards** (3):
- Publicar vídeo → MK4
- Biblioteca de vídeos → MK5
- Dashboard da empresa → MK6

### 9.4 MK4 — Publicar Vídeo (upload + metadados)

**Rota**: `/agencia/empresas/:companyId/videos/novo`
**Caminho**: MK3 → Publicar vídeo

**Mensagem institucional (literal)**: "Os vídeos institucionais ajudam empresas a apresentar seus serviços e compartilhar conhecimento com o ecossistema condominial."

**Campos**:
- Empresa vinculada (readonly — pré-preenchido pela sessão scoped)
- Título do vídeo (string)
- **Tipo de vídeo** (12 valores — ver §8.7)
- Subtema do vídeo (auto-preenchido com base no segmento da empresa)
- Descrição do vídeo (text)
- Link do vídeo (URL Mux ou upload direto)

**Checkbox crítico**: "Autorizar exibição em processos de escolha de fornecedor" (`jornada-empresa-marketing.md:149-150`). Mapeia para `videos.autorizar_exibicao_votacao` (Req 29 `requirements.md:314`). Se marcado, vídeo fica visível a moradores **apenas** durante votação de fornecedor em assembleia. Req 103 revoga acesso pós-votação.

**Botões**:
- Salvar rascunho (`videos.status='draft'`)
- Publicar vídeo (`videos.status='published'` + trigger lock 90d — Req 29)

**Regras (`:157-164`, literal)**: "Moradores não acessam vídeos institucionais normalmente. Eles apenas visualizam vídeos quando a empresa participa de processo de escolha de fornecedor no módulo assembleia. Após a votação o acesso é bloqueado."

**Quota**: conforme plano da empresa (`plus=8/mês`, `pro=12/mês` — `functional-matrix.md:85`). Incrementa `feature_usages` sob `user_id=company_admin` (não a agência) — porque o vídeo é da empresa.

**ABAC check**: `TokenScope=videos:upload on:company:Y` obrigatório.

### 9.5 MK5 — Biblioteca de Vídeos

**Rota**: `/agencia/empresas/:companyId/videos`
**Caminho**: MK3 → Biblioteca de vídeos

**Mensagem institucional (literal)**: "Aqui estão todos os vídeos publicados para esta empresa."

**Informações por vídeo**:
- Título
- Tipo de vídeo (12 valores)
- Data de publicação
- Visualizações (total agregado)

**Botões por vídeo**:
- Editar vídeo → edição de metadados (respeitando lock 90d para vídeo substituído — Req 29)
- Ocultar vídeo → `status='hidden'`; **não remove** histórico (literal `:191-192`: "Ocultar vídeo não remove o histórico").

**ABAC check**: `TokenScope=videos:edit on:company:Y`.

### 9.6 MK6 — Dashboard da Empresa (analytics delegada por empresa)

**Rota**: `/agencia/empresas/:companyId/dashboard`
**Caminho**: MK3 → Dashboard da empresa

**Mensagem institucional (literal)**: "Acompanhe o desempenho dos conteúdos publicados para esta empresa."

**Indicadores (`:207-213`)**:
- Total de vídeos publicados
- Total de visualizações
- Vídeo mais visualizado
- Tempo médio de visualização
- Taxa de retenção

**ABAC check**: `TokenScope=analytics:read on:company:Y`.

**Restrição**: não expõe métricas relativas a **identidade** de quem assistiu (ex: quais síndicos viram) — anonimizado (Req 29 "Analytics: views, avg_watch_time, completion_rate, retention_rate" — `requirements.md:315`).

### 9.7 MK7 — Dashboard Geral da Agência (cross-empresa)

**Rota**: `/agencia/painel/dashboard`
**Caminho**: MK1 → Dashboard

**Mensagem institucional (literal)**: "Este painel apresenta uma visão geral do desempenho dos conteúdos produzidos pela sua agência dentro da plataforma."

**Indicadores (`:229-235`)**:
- Número de empresas atendidas
- Total de vídeos publicados (somatório cross-empresa)
- Total de visualizações (somatório)
- Vídeo mais acessado (cross-empresa)
- Empresa com maior engajamento

**Export** (sugerido; ver gap §18): PDF/CSV cross-empresa.

**ABAC check**: requer ≥ 1 DelegationGrant `status=active` com `analytics:read`. Agregação é feita sob cada tenant independentemente (join cross-tenant controlado; testes de isolamento cross-tenant obrigatórios — `08-security/threat-model`).

### 9.8 MK8 — Marketing Link Recebido

**Rota**: `/agencia/marketing-link`
**Caminho**: MK1 → Marketing Link recebido

**Mensagem institucional (literal)**: "Empresas interessadas em contratar serviços de marketing podem registrar aqui suas solicitações de contato com sua agência."

**Lista exibida (`:252-259`)**:
- Empresa (razão social)
- Responsável (pessoa de contato)
- Telefone
- Email
- Tipo de apoio solicitado (`MarketingSupportType` — §8.2)
- Data da solicitação

**Botões (`:261-263`)**:
- Entrar em contato (externaliza: abre cliente de email / telefone — não há thread interna; estilo Connect Me frio)
- Marcar como atendido → `MarketingLinkStatus.atendido`

**Regras (literal `:265-271`)**:
- "Marketing Link não cria vínculo automático entre empresa e agência."
- "Ele apenas registra a intenção de contato."
- "O vínculo oficial ocorre quando a empresa envia convite em Gestão de Agência de Marketing." (E14)

**ABAC check**: `TokenScope=marketing_link:receive` — genérico (não é por empresa; é geral ao usuário agência). Recebimento vem via NATS `marketing_link.request.created`.

**Tela adicional (gap)**: "Audit Log Delegation" (MK8 alternativo no inventário v2 `_reconciliacao-dev/04-telas-jornadas.md:166`) — exibe histórico de ações da agência. O PDF canônico do cliente **não cita** essa tela; é do inventário v2. Registrar como gap (§18) — decidir se é MK9 ou se substitui MK8 do inventário v2.

---

## 10. Marketing Link (feature distinta — E16 + MK8)

Marketing Link é **feature distinta** de Connect Me e distinta de DelegationGrant. Vive entre **Empresa (emissora)** e **Agência (receptora)**.

### 10.1 Fluxo

1. **Empresa (E16)** preenche formulário dirigido a **uma agência específica** já cadastrada (busca por perfil/portfólio — gap: busca pública de agências ainda não destilada):
   - Tipo de apoio (MarketingSupportType)
   - Prioridade (SupportPriority)
   - Responsável da empresa (nome/telefone/email)
   - Mensagem livre
2. Sistema grava `marketing_link_requests` com `status='sent'`.
3. NATS `marketing_link.request.created` → notificação push/email para agência.
4. Agência abre **MK8** → `status='viewed'`.
5. Agência conversa externamente (telefone/email) → clica "Marcar como atendido" → `status='atendido'` ou registra proposta enviada (`proposal_sent`).
6. **Se virar negócio**: empresa inicia **E14 "Convidar Agência"** — fluxo independente que cria DelegationGrant. Marketing Link fica com `status='accepted'`.
7. **Se desistir**: `status='rejected'` (por qualquer das partes).

### 10.2 Invariantes Marketing Link

- `status='sent'` imutável após envio (frio, estilo Connect Me — `commercial.md:79-106` Req 19.1 para Connect Me M→S; mesma filosofia)
- Dados de contato da empresa são **visíveis à agência desde o envio** (diferente de Connect Me S→E que oculta dados do remetente até interesse — aqui a empresa quer ser contatada, então revela de cara)
- Não há thread / reply na plataforma — conversa acontece fora
- LGPD log obrigatório: `terms_version`, timestamp, IP (Rule 10 Req 19)

### 10.3 Não-vínculo (crítico)

**Marketing Link NÃO cria DelegationGrant**. Repetido em 3 fontes:
- `jornada-empresa-marketing.md:266-271`
- `_reconciliacao-dev/04-telas-jornadas.md:147-149`
- STATE §D-060 (implícito: vínculo = grant explícito)

Vínculo só se materializa via **E14 convite formal + aceite do token pela agência**.

---

## 11. Regras de negócio (invariantes operacionais)

Consolidação das regras exaustivas, com fonte:

| # | Regra | Fonte |
|---|---|---|
| 1 | Agência tem login próprio via Zitadel (role `marketing`); não é impersonation | D-059/D-060 (STATE:497-502, 354-364) |
| 2 | Agência **não** opera tenant condominial próprio; opera no tenant de empresas clientes via DelegationGrant | D-060 |
| 3 | Delegação é **scoped** por empresa e por escopo (lista fechada §8.1) | `functional-matrix.md:114-125` + D-060 |
| 4 | Revogação de delegação é **imediata**: purge cache + sessão force-logout + event `DelegationRevoked` | `Domínio - Conteúdo.md:87` + §6.4 |
| 5 | **Audit obrigatório**: toda ação delegada (videos.upload/edit, analytics.read, marketing_link.manage) grava AuditEntry imutável | §6.5 |
| 6 | Agência **nunca** acessa dados financeiros, comerciais, Connect Me, timeline, registros, comunicados (lista fechada §6.3) | `jornada-empresa-marketing.md:277-286` + `functional-matrix.md:121-124` |
| 7 | Vídeo publicado pela agência é **atribuído à empresa**, não à agência (autoria institucional pertence à empresa) | `Domínio - Conteúdo.md:85` + `jornada-empresa-marketing.md:294-296` |
| 8 | Morador **não vê** vídeos institucionais normalmente; só durante votação de fornecedor se `autorizar_exibicao_votacao=true` | Req 29 + Req 103 (`requirements.md:314,421-424`) |
| 9 | Após votação, acesso do morador ao vídeo é **revogado** | Req 103 |
| 10 | Síndico **nunca** interage com agência diretamente | `jornada-empresa-marketing.md:291-292` |
| 11 | Marketing Link **não cria vínculo automático** | `jornada-empresa-marketing.md:266-271` |
| 12 | Empresa Trial **não pode** convidar agência (feature gated Pagante) | `Domínio - Identidade.md:202` |
| 13 | 1 agência pode atender **múltiplas empresas** (1:N grants) | `Domínio - Conteúdo.md:83` |
| 14 | Quota de vídeos é da **empresa** (não da agência); agência consome quota da empresa ao publicar | Req 29 `requirements.md:312` + §5 |
| 15 | Lock 90d: vídeo substituído só pode ser re-editado após 90 dias (Req 29); aplicável também quando agência é quem edita | Req 29 `requirements.md:313` |
| 16 | Admin MS pode moderar conteúdo produzido pela agência; bypass ABAC auditado (D-058/D-061) | `03-subprojects/admin-privileges.md` |
| 17 | Onboarding agência: perfil da agência + tipos de serviço + portfólio (guiado) | `Domínio - Identidade.md:219` |
| 18 | Agência não auto-cadastra "do zero" — primeiro contato é via token de convite de empresa (ou inscrição em catálogo público de agências — backlog) | `identity.md:91` + D-060 |
| 19 | Agência não paga plano próprio no MVP; quotas derivam do plano da empresa cliente | §5 (interpretação ADR-0032 + D-060) — **gap registrar** |
| 20 | Telas MK1-MK8 são dedicadas a role=marketing; síndico/morador/empresa não veem essas rotas | `jornada-empresa-marketing.md:28-272` |

---

## 12. Invariantes (domínio)

- **INV-DELEG-001** — `DelegationGrant.scopes ≠ ∅` sempre que status ∈ {requested, accepted, active}
- **INV-DELEG-002** — `DelegationGrant.status='active'` ⇒ `accepted_at IS NOT NULL`
- **INV-DELEG-003** — 1 grant ativo por (agency_user_id, company_tenant_id); constraint UNIQUE PARTIAL `WHERE status ∈ ('requested','accepted','active')`
- **INV-DELEG-004** — `revoked_at IS NOT NULL` ⇒ `status='revoked'` (imutável após revogação; nova vinculação exige novo grant)
- **INV-DELEG-005** — Token Zitadel da agência não carrega direitos cross-tenant per se; direito vem do grant. Se grant não existe, ABAC rejeita mesmo com role válida.
- **INV-MKL-001** — MarketingLinkRequest imutável após `status='sent'` (fluxo só avança; empresa não pode editar depois)
- **INV-MKL-002** — `MarketingLinkRequest` não pode ser criada por Empresa Trial (guard middleware + matriz)
- **INV-MKL-003** — `MarketingLinkRequest` **nunca** cria DelegationGrant como side-effect. Vínculo formal **só** via E14.
- **INV-AUDIT-001** — Toda ação da agência em tenant de empresa gera AuditEntry (append-only, imutável). Empresa pode ler entries do próprio tenant; agência pode ler entries do próprio user_id.
- **INV-COMPLIANCE-001** — 1 representante legal por empresa (único que pode emitir grant delegation de primeira classe; pode delegar "convite" a outro admin, mas assinatura de termos é do representante legal)
- **INV-VIDEO-001** — Vídeo publicado pela agência: `video.company_tenant_id` é da **empresa**; `video.published_by_user_id` é da **agência**; dono (owner) do aggregate vídeo = empresa.
- **INV-QUOTA-001** — `feature_usages` de "video_upload_month" atribuído ao `company_tenant_id` (não ao agency_user_id).

---

## 13. State machine

### 13.1 DelegationGrant

```
[start]
   │
   │ empresa cria (E14 convite → email com token)
   ▼
(requested) ──── agência decline ──→ (revoked)
   │
   │ agência aceita via token + Zitadel login
   ▼
(accepted) ──── empresa revoga ──→ (revoked)
   │
   │ primeira ação bem-sucedida OR idle timeout (depende de policy)
   ▼
(active) ─────┬──── empresa revoga ──→ (revoked)
              │
              ├──── escopo modificado ──→ (active) [status inalterado; escopos mudam]
              │
              ├──── TTL expira ──────→ (expired)
              │                           │
              │                           └── empresa renova ──→ (active)
              │
              └──── agência auto-desvincula ──→ (revoked)
```

### 13.2 MarketingLinkRequest

```
[empresa preenche E16]
   │
   ▼
(sent) ───── agência abre MK8 ─────→ (viewed)
   │                                     │
   │                                     ├─ agência envia proposta ──→ (proposal_sent)
   │                                     │                               │
   │                                     │                               └─ aceito ──→ (accepted) [fluxo E14 inicia se quiser virar grant]
   │                                     │                               └─ rejeitado ──→ (rejected)
   │                                     │
   │                                     └─ agência marca "atendido" ─→ (atendido)
   │                                                                       │
   │                                                                       ├─ aceito ──→ (accepted)
   │                                                                       └─ rejeitado ──→ (rejected)
   │
   └── timeout sem abertura (policy; sugestão: 30d) ──→ (rejected) [auto]
```

---

## 14. Domain events

Publicados via `IMessaging` (NATS JetStream — D-019 legado/D-067), consumidos por auditoria, notificação, reporting.

| Evento | Payload mínimo | Publicador | Consumidores |
|---|---|---|---|
| `DelegationRequested` | `{grant_id, company_tenant_id, invited_email, scopes, token_hash}` | Commercial (E14) | Notification (email agência), Audit |
| `DelegationAccepted` | `{grant_id, agency_user_id, company_tenant_id, scopes, accepted_at}` | Identity + Commercial | Notification (empresa), Audit, Projection `marketing_agency_links` |
| `DelegationScopesUpdated` | `{grant_id, old_scopes, new_scopes, changed_by}` | Commercial (E15) | Audit, Cache invalidation |
| `DelegationRevoked` | `{grant_id, reason, revoked_by, revoked_at}` | Commercial (E15) | Identity (purge session+cache), Audit, Notification (agência), Projection |
| `DelegationExpired` | `{grant_id, expired_at}` | Commercial (job noturno TTL) | Notification (ambos), Audit |
| `MarketingLinkSent` | `{request_id, from_company_tenant_id, to_agency_user_id, support_type, priority}` | Commercial (E16) | Notification (agência push+email), Inbox MK8 |
| `MarketingLinkViewed` | `{request_id, viewed_at}` | Commercial (MK8 open) | Analytics |
| `MarketingLinkStatusChanged` | `{request_id, old_status, new_status}` | Commercial (MK8) | Analytics, Notification (empresa) |
| `VideoPublishedByAgency` | `{video_id, company_tenant_id, agency_user_id, video_type}` | Content (MK4) | Search index, Moderation, Audit, Timeline (se aplicável — não-public por default) |
| `VideoHiddenByAgency` | `{video_id, agency_user_id, hidden_at}` | Content (MK5) | Search index, Audit |

---

## 15. Dependências inter-sub-produto e providers

| Dependência | Motivo | Sub-produto / provider |
|---|---|---|
| **Identity (Zitadel)** | Login da agência (role `marketing`); 1-device; ABAC engine; grants | `01-governanca-institucional` indireto; módulo `identity` |
| **Commercial (Empresa cliente)** | Emissão de DelegationGrant via E14; revogação via E15; formulário E16 Marketing Link; tenant de operação | `02-connect-me` (Marketing Link vive próximo ao Connect Me em arquitetura de tabelas); módulo `commercial` |
| **Content (Vídeos Mux)** | Upload/edit/analytics de vídeos em nome da empresa; Mux como `IVideoProvider`; lock 90d; moderação | `04-conteudo-videos`; módulo `content` |
| **Billing (Empresa cliente)** | Quotas de vídeo derivadas do plano `plus`/`pro` da empresa; Stripe `IPaymentGateway` | módulo `billing` (sem impacto direto — agência não paga) |
| **Cross-domain (Audit Log)** | IAuditLogger imutável (DT-010 legado em andamento); cada ação delegada grava entry | `09-compliance` (auditoria integrada) |
| **Messaging (NATS)** | Eventos de domínio `DelegationRequested/Accepted/Revoked`, `MarketingLinkSent`, etc. | `IMessaging` |
| **Email (Resend)** | Convite de delegação; notificações Marketing Link | `IEmailProvider` (D-067 Resend canônico Abril/2026) |
| **Push (FCM)** | Push para agência ao receber Marketing Link | `IPushProvider` |
| **Storage (MinIO/R2)** | Logo/portfólio da agência (onboarding `Domínio - Identidade.md:219`) | `IStorageProvider` |
| **Assembly (Live)** | **Cross-link importante**: vídeos com `autorizar_exibicao_votacao=true` são exibidos aos moradores durante votação de fornecedor | `05-assembleia-live`; módulo `assembly` (Req 29 + Req 103) |
| **Admin MS (role privilegiada)** | Moderação de vídeos publicados pela agência; suspensão de delegação em caso de abuso (D-058, D-061) | `03-subprojects/admin-privileges` |

---

## 16. Anti-abuso e threat-model específico

Base: `08-security/BEYOND_CORP` + `08-security/threat-model`. Ameaças específicas do sub-produto:

| Threat | Vetor | Mitigação |
|---|---|---|
| **Escalação cross-tenant** — agência manipula `company_tenant_id` no request pra acessar empresa não-delegada | Tampering no claim | ABAC engine valida `DelegationGrant` em cada request; token JWT Zitadel não carrega tenant (carrega só role); tenant vem de sessão selecionada + verificação DB |
| **Persistência de acesso após revogação** | Token cacheado Redis | Purge síncrono (event-driven via `DelegationRevoked`); TTL cache 5min (D-001 legado aceita staleness, monitorar); webhook Zitadel de invalidação (D-001 pendente) |
| **Agência maliciosa: publicar conteúdo difamatório em nome da empresa** | Abuso de grant ativo | Moderação cross-tenant (D-024 ADR-0024) + revisão pré-publicação de vídeos institucionais; feature flag "requer aprovação do administrador da empresa antes de publicar" (backlog — gap §18) |
| **Exfiltração de analytics sensíveis** | Analytics expondo PII indireta (ex: tempo exato em que síndico X assistiu) | Anonimização (Req 29) + sem PII em métricas (só agregados) |
| **Brute-force em token de convite** | Enumeração de tokens | Token de 1-uso com TTL curto (ex: 48h), rate-limit no endpoint de aceite, fingerprint de device |
| **Phishing** — email falso de "convite de agência" | Spoofing de email | Link absoluto sob `mastersindico.com.br` domain; SPF/DKIM/DMARC (Resend); aviso visual no Zitadel login que carrega contexto da empresa convidante |
| **Revenge revoke** — empresa revoga grant e agência deleta vídeos por retaliação antes | Janela entre "vou revogar" e "revoguei" | Vídeos **não** são deletáveis pela agência; só `hidden` (Req 29 + MK5 literal). Deleção é admin-only / empresa-only. |
| **Vídeo persiste conteúdo inapropriado após revogação** | Vídeo publicado vive no tenant da empresa; empresa revoga, vídeo continua visível | Correto por desenho (vídeo é da empresa); cabe à empresa moderar conteúdo herdado via Content module. |
| **LGPD — dados de contato vazados em Marketing Link** | Agência é terceira parte recebendo nome/email/telefone de funcionário da empresa | LGPD log com `terms_version` (Req 19 Rule 10); consentimento na E16 (empresa aceita que dados vão à agência externa) |
| **Audit log tampering** | Agência apaga rastro | AuditEntry é append-only sem `deleted_at` (`01-domain/aggregates/AuditEntry.md`); WORM (write-once read-many) via partitioning + read-only role no DB |
| **Cross-tenant data leak em MK7 analytics consolidado** | JOIN mal isolado cruzando métricas de empresas diferentes para mesma agência | Queries específicas por tenant com `WHERE company_tenant_id IN (...)` da lista de grants ativos; testes sistemáticos cross-tenant (100+ ABAC tests no legado) |

---

## 17. Estado no código (Sprint atual — 2026-04-23)

Baseado em `CLAUDE.md:5 Bounded contexts` (`vault/50-projects/master-sindico/CLAUDE.md`):

| Componente | Status backend | Observação |
|---|---|---|
| Role Zitadel `marketing` | ✅ Produção (module `identity`) | Enum `user_role` inclui `marketing` (`identity.md:52`; `personas-and-plans.md:130`) |
| Login OIDC agência | ✅ Produção | Mesmo fluxo de qualquer role (Req 1); nenhuma diferença no backend |
| 1-device policy agência | ✅ Produção | Mesmo middleware `Authenticate` (Req 1) |
| ABAC engine | ✅ Produção | Usa `(user.role × plan.tier × action)`; **não valida ainda `DelegationGrant`** — gap §18 |
| DelegationGrant aggregate | ❌ **Não implementado** | Nenhum módulo `marketing/` ou `delegation/` em `backend/internal/modules/` segundo `CLAUDE.md` legado. Provavelmente vive encaixado em `commercial` (empresa gerencia agência) — confirmar no backend sub-projeto Fase 8. |
| Tabela `marketing_link_requests` | 🟡 **Referenciada, não confirmada** | `_reconciliacao-dev/04-telas-jornadas.md:149` fala "Tabela backend: `marketing_link_requests`"; não confirmado em código. |
| Tabela `marketing_agency_links` ou `delegation_grants` | ❌ **Não modelado** | Sem evidência em contextos/ analysis de DB. |
| Telas MK1-MK8 (web) | 🟡 Parcial (~100% inventário; estado real ?) | `_reconciliacao-dev/04-telas-jornadas.md:23` diz "v2 atual ~100%". **Atenção**: esse "100%" é medida de **inventário catalogado**, não de **implementação funcional**. Estado real a confirmar em subprojeto `web` Fase 8. |
| Telas E13/E14/E15/E16 (empresa) | 🟡 Parcial (~90%) | Idem. |
| Audit log cross-module | 🟡 DT-010 em aberto | Legado já lista DT-010 "audit trail LGPD cross-module" como dívida aberta. Crítico para o sub-produto. |
| NATS events `Delegation*` | ❌ Não implementados | Sem spec formal ainda. |
| Moderação de conteúdo publicado pela agência | 🟡 ADR-0024 (cascata) | Cascade moderation existe; integração explícita com agência precisa ser validada. |
| Integração Mux (vídeo da agência) | ✅ Produção (module `content`) | Upload via `IVideoProvider`. Lock 90d implementado. |

---

## 18. Gaps (questões abertas / pendências / ambiguidades)

Priorizadas: 🔴 crítico para M2 (Commercial+Content), 🟡 ajuste antes de ship, 🟢 backlog.

| ID | Nível | Descrição | Fonte |
|---|---|---|---|
| G-MKT-01 | 🔴 | **DelegationGrant não modelado** como aggregate em `01-domain/aggregates/`. Sem isso, ABAC da agência não tem contrato. | §7.3 + §17 |
| G-MKT-02 | 🔴 | **ABAC engine não valida DelegationGrant** atualmente (ADR-0032 §ABAC não tem o passo "se role=marketing"). Necessário estender. | §6.2 |
| G-MKT-03 | 🔴 | **Tabela `delegation_grants`** não existe em contextos/DB. Migration + sqlc + repo obrigatórios. | §17 |
| G-MKT-04 | 🔴 | **Marketing Link** ainda não tem spec em `commercial.md` (ausente de Req 18–27.3). Criar Req X.Y em `04-requirements/functional/commercial.md`. | `commercial.md` inteiro |
| G-MKT-05 | 🟡 | **Tela MK8 diverge** entre o PDF canônico do cliente (Marketing Link Recebido) e o inventário v2 (Audit Log Delegation). Decidir: 8 telas ou 9 telas? | `jornada-empresa-marketing.md:239-272` vs `_reconciliacao-dev/04-telas-jornadas.md:166` |
| G-MKT-06 | 🟡 | **Plano próprio de agência**: `personas-and-plans.md` tinha `marketing_standard` (abolido por ADR-0032). O novo modelo: agência não paga nada? Paga plano global `plus/pro`? Agência-Freemium? Sem decisão formal. | §5 + STATE D-057 |
| G-MKT-07 | 🟡 | **Onboarding agência**: `Domínio - Identidade.md:219` fala "perfil, tipos de serviço, portfólio" mas não há spec de tela de cadastro da agência (MK0? registrar como tela faltante?). | `Domínio - Identidade.md:219` |
| G-MKT-08 | 🟡 | **Busca pública de agências**: E16 Marketing Link precisa que a empresa **encontre** uma agência. Existe catálogo público? Filtro por especialidade? Rating? Sem spec. | `jornada-empresa-marketing.md` silêncio |
| G-MKT-09 | 🟡 | **Escopo de capabilities ampliado** — `novo escopo-7.md:603-604` menciona "Empresa de Marketing: Criação de cursos, vídeos institucionais, Marketing Link (tag por vídeo para gerar leads), participação fórum/lives". **Criação de cursos, tag por vídeo para leads, fórum, lives** **não** aparecem nas MK1-MK8 nem na matriz funcional. Confirmar com PO: escopo expandido no futuro ou erro no PDF? | `novo escopo-7.md:603-604` vs `jornada-empresa-marketing.md` |
| G-MKT-10 | 🟡 | **Aprovação prévia do vídeo pelo administrador da empresa**: não existe fluxo de aprovação; agência publica direto. Risco de conteúdo inadequado. Considerar feature "exige aprovação" opt-in pela empresa. | §16 linha "agência maliciosa" |
| G-MKT-11 | 🟡 | **Audit Log Delegation MK8 (v2)**: mesmo que se opte pelo MK8 canônico do PDF, precisa existir **alguma** tela de audit; pode virar MK9 ou sub-aba de MK1. | §9.8 + §6.5 |
| G-MKT-12 | 🟡 | **TTL de DelegationGrant não especificado formalmente**. Sugestão 90d sem confirmação. | §6.6 |
| G-MKT-13 | 🟡 | **Quota de vídeos: quem consome quando agência publica?** Matriz funcional diz agência publica em nome da empresa mas não explicita se consume quota do `feature_usages` da empresa ou da agência. Regra §11 linha 14 assume quota da empresa; confirmar. | §5 + §11.14 |
| G-MKT-14 | 🟡 | **Representante legal**: mencionado em §4 como papel distinto mas não há campo `is_legal_representative` no schema do cliente. Modelagem pendente. | §4 |
| G-MKT-15 | 🟢 | **Fórmula de rating/reputação de agências**: D-051 definiu fórmula de reputação empresa (Bronze→Diamante). Agência vai ter reputação análoga? `portfolio-de-produtos.md:129` prevê "futuro" para síndicos profissionais e agências. Backlog M5+. | `portfolio-de-produtos.md:129` |
| G-MKT-16 | 🟢 | **Self-service de agência** (cadastro sem convite, listada em catálogo público) vs modelo atual (só via convite). Pesquisa de mercado pendente. | `identity.md:91` |
| G-MKT-17 | 🟢 | **Plural de delegação**: pode uma agência ter **sub-usuários** (ex: um "diretor de arte" + um "editor" dentro da mesma agência)? Hoje agência = 1 user. Matriz futura. | §4 |
| G-MKT-18 | 🟢 | **Tag de vídeo para geração de lead** (`novo escopo-7.md:604`): feature distinta de Marketing Link — gera lead pela **visualização** do vídeo, não pelo formulário. Verificar se entra no escopo M2+. | `novo escopo-7.md:604` |
| G-MKT-19 | 🟡 | **Desvincular unilateral por agência**: a agência pode se desvincular? Está implícito no state machine (auto-desvincular→revoked). Formalizar. | §13.1 |
| G-MKT-20 | 🟡 | **Encerramento de agência** (empresa fecha portas): DelegationGrants ativos precisam ser encerrados em lote. Fluxo admin ou self-service? | §13.1 |

---

## 18.5 Telas & Endpoints

Rastreabilidade consolidada em [[../../03-subprojects/traceability]] §2.9 (Marketing & Agências) + §3.2 (matriz inversa commercial).

**Escopo UI**:
- Agência: MK1-MK8 (8 telas) — painel, empresas, context switch, CRUD vídeos, analytics, dashboard consolidado, marketing link inbox
- Empresa ↔ agência: E14 (gestão) · E16 (marketing link intenção)
- Total: **10 telas web**, 0 mobile

**Endpoints-chave**:
- Agência: `/api/v1/marketing/dashboard` · `/notifications` · `/empresas` · `/empresas/:id/context` · `/dashboard/consolidated` · `/dashboard/export`
- Delegação (via empresa): `/api/v1/enterprise/agencies` · `/invite` · `/:id/revoke` · `/:id/permissions`
- Vídeos delegados: `/api/v1/content/videos?empresa_id=:id` · `/:id` (PUT) · `/:id/hide` · `/:id/restore`
- Marketing Link: `/api/v1/commercial/marketing-link/received` · `/:id/attended` · `/:id/view` · `/api/v1/commercial/marketing-link` (POST intenção)

**Invariantes críticos**: INV-049 (agência zero-access a deals/CM/propostas/timeline) · INV-066 (token escopo restrito `videos:*` + `analytics:read`) · INV-057 (admin bypass audit em remove pre-90d) · INV-060 (sem leak métricas cross-tenant) · IDN-025 (invite token one-shot, sem auto-registro).

**ABAC policy**: `delegated_marketing` (ver [[../../03-subprojects/traceability]] §8 matriz ABAC). 100+ testes de integração garantem DENY para `deal:*`, `proposal:*`, `connect_me:*`, `timeline:create/amend`.

**Fluxo crítico** (traceability §7.5): E14 convida → IDN-025 invite token → Zitadel cria user `marketing_agency` → ABAC policy delegada → MK3 context switch → MK4 upload → Mux → INV-064 HMAC.

**Sprints**: Sprint 5 ✅ backend (marketing agencies delegation + Mux integration) · UI Sprint 10 🟡.

**Gaps identificados**: Perfil público de agência (vitrine que origina Marketing Link) fora de escopo · rate-limit por (empresa, agência) sem regra concreta · websocket vs polling para indicadores em tempo real.

---

## 19. Referências cruzadas

### Decisões vivas

- **D-059** — Agência é sub-produto interno próprio — `master-sindico-v2/STATE.md:497-502`
- **D-060** — Login Zitadel próprio + delegação ABAC — `master-sindico-v2/STATE.md:354-364`
- **D-057 / ADR-0032** — Planos globais Stripe-style, sem slugs compostos — `master-sindico-v2/STATE.md:313-337`, `02-architecture/adr/0032-global-plans-stripe-style.md`
- **D-058 / D-061** — Admin MS é role privilegiada, não app separada — `master-sindico-v2/STATE.md:366-376`
- **D-056** — Agnosticismo de provedor — `master-sindico-v2/STATE.md:297-311`
- **D-067** — Stack canônica Abril/2026 (Resend, PG tsvector, Railway) — `master-sindico-v2/STATE.md:418-429`
- **D-068** — Obsidian Vault do cliente como fonte canônica — `master-sindico-v2/STATE.md:431-437`
- **DT-010** — Audit trail LGPD cross-module (crítico pra este sub-produto) — legado

### Fontes originais (pesos de verdade)

1. **PDF cliente canônico**: `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/_archive/content-discovery-2026-03/jornada-empresa-marketing.md` (texto integral MK1-MK8)
2. **Obsidian Vault cliente — Abril/2026** (D-068):
   - `/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Identidade.md`
   - `/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Conteúdo.md`
   - `/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Comercial.md`
   - `/home/vinni/Documentos/Obsidian Vault/Clients/MasterSindico/03 - Modelagem e Dados/Enums e Listas Mestre.md`
3. **Requirements destilados v1.0 stable Abril/2026**:
   - `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/specs/requirements/commercial.md`
   - `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/specs/requirements/identity.md`
   - `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/specs/requirements/personas-and-plans.md`
   - `master-sindico-v2/90-ingested/from-vault-50-projects-master-sindico/specs/requirements/functional-matrix.md`
4. **Inventário reconciliação**: `master-sindico-v2/90-ingested/_reconciliacao-dev/04-telas-jornadas.md`
5. **Escopo-7 (PDF contratual cliente)**: `vault/50-projects/master-sindico/contextos/novo escopo-7.md:603-604`

### Sub-produtos vizinhos

- [[01-governanca-institucional]] — tenant condomínio (agência **não** acessa)
- [[02-connect-me]] — Marketing Link é feature paralela; Connect Me não é acessível à agência
- [[04-conteudo-videos]] — **sub-produto espelho**: vídeo é da empresa, publicado pela agência com grant; integrações Mux, lock 90d, moderação, visibilidade-controlada durante assembleia
- [[05-assembleia-live]] — Req 29+103: vídeos da empresa (publicados pela agência) visíveis a moradores só durante votação de fornecedor
- [[09-compliance]] — audit trail imutável (DT-010); LGPD log de delegação

### Arquitetura

- `master-sindico-v2/02-architecture/adr/0032-global-plans-stripe-style.md`
- `master-sindico-v2/02-architecture/adr/0012-abac-redis-cache.md`
- `master-sindico-v2/02-architecture/adr/0024-moderation-cascade.md`
- `master-sindico-v2/02-architecture/patterns.md`
- `master-sindico-v2/08-security/BEYOND_CORP` (quando preenchido)

### Pasta canônica no backend

- `backend/internal/modules/identity/` (role marketing)
- `backend/internal/modules/commercial/` (convite E14, Marketing Link E16)
- `backend/internal/modules/content/` (vídeos, upload)
- **Novo** (pendente): `backend/internal/modules/commercial/delegation/` ou módulo dedicado — decidir em Fase 8 backend

### Telas (catálogo web)

- Web: família MK (MK1-MK8) + E13/E14/E15/E16 (empresa) — ver `03-subprojects/web/ui-catalog.md`

---

## 20. Notas do destilador (meta)

- Este arquivo substitui o stub Fase 8 datado de 2026-04-23.
- **Dual-check pendente** (agent B em contexto limpo) — conforme método Fase 8 D-069.
- Conflito entre fontes resolvido hierarquicamente por STATE D-059/D-060 (nova leitura canônica) sobre `personas-and-plans.md:21` e `functional-matrix.md:114` (nomenclatura antiga "Delegated Actor, não é tenant"). Ambas **não foram descartadas** — foram reinterpretadas à luz das decisões.
- Nomenclatura **`marketing_standard` / N1/N2/N3 banida** (ADR-0032 §"Regra dura").
- Todos os trechos literais citados preservam português original do cliente.
- Gaps G-MKT-01..20 abrem trilha de tasks para M2+.
