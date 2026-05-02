---
title: ADR-0039 — IAM Cloudflare-style (Member + Group + Role + PermissionGroup + Quota + Plan)
type: adr
tags: [adr, security, iam, abac, rbac, master-sindico, v2, fase-14]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
supersedes_partial: [0012-abac-redis-cache]
---

# ADR-0039 — IAM Cloudflare-style aplicado ao Master Síndico

## Status

`accepted (conditional)` — 2026-04-23 (Fase 14, derivado de D-116). Substitui parcialmente o modelo de personas pré-estabelecidas implícito em [[0003-zitadel-oidc-provider]] e na matriz funcional histórica. Reforça e re-arquiteta [[0012-abac-redis-cache]] (cache permanece; ordem de avaliação se mantém; matriz canônica muda).

> **Conditional Acceptance** (ajuste do dual-check 2026-04-23; refinado com R1-R6 pós-análise GPT 2026-04-24): a **decisão arquitetural** está aceita (modelo, entidades, patterns, mapping). A acceptance fica **condicional** à entrega de 4 artefatos pré-M1 — sem eles, o ADR fica em `accepted (conditional)`; com eles, promove para `accepted (active)`:
> 1. **Catálogo PermissionGroup seed file** versionado em git (`backend/internal/modules/identity/iam/seed/permission_groups_v1.yaml`) com todos os PGs do §4 **incluindo categoria `pg.iam.*`** (delegar ≠ executar).
> 2. **Boundary templates por `group_kind`** (`iam/seed/group_boundaries_v1.yaml`) — define `boundary_permission_group_ids` default para cada kind (condo, enterprise, local_company, marketing_agency, condo_administrator, platform).
> 3. **Migration idempotente** persona-legada → Group+Membership+Role (script Go re-runnable, dry-run mode obrigatório, audit log de cada conversão); DB-enforces INV-IAM-001..006.
> 4. **Endpoints platform-side admin** funcionais (tabela §Painel admin) com validação INV-IAM-005 (boundary) + INV-IAM-006 (creator ⊇ target); inclui `POST /admin/policies/dry-run` e `GET /admin/simulate-as/{user_id}`; Quota table com `scope_type` + `soft_threshold` + `on_hard_exceeded`.
>
> Status promove para `accepted (active)` quando os 4 estiverem em `main` e validados em staging.

## Contexto

O modelo herdado tratava persona como **role pré-estabelecida** (`syndic_admin`, `resident_titular`, `enterprise_admin`, `local_company_admin`, `marketing_agency_admin`, `platform_admin`). Cada persona já vinha com capacidades fixas no código. Isso gera os anti-patterns documentados em [[../../../../10-knowledge/security/iam-models-benchmark#7. Anti-patterns]]:

- **Role Explosion** (Anti-pattern 1): cada novo subtipo (auxiliar de síndico, marketing da empresa, contador da administradora) força nova role + edição de matriz × código.
- **Permission baked into code** (Anti-pattern 2): `if role == "syndic_admin" { ... }` espalhado em handlers.
- **God Role** (Anti-pattern 3): `platform_admin` com capacidade transversal sem possibilidade de elevação JIT.

O cliente (João) trouxe insight no dia 2026-04-23 (sessão Fase 14):

> *"cada usuario obrigatoriamente vai ter uma role… e se em vez de termos essas roles pre-estabelecidas, so termos a role de admin… podemos trabalhar com role, group, quota e plan… na hora de criar o usuario enterprise, por exemplo temos ele como group com somente 1 enterprise_admin e subroles dentro do grupo enterprise… o sindico tambem tem role_type, ou seja tudo isso se transforma em (group_role)_*(role_type)… cada grupo a gente podesse criar uma role 'padrao'… podemos ver o que as grandes empresas fazem proximo disso e complementar"*

Benchmark realizado (F14.2 → [[../../../../10-knowledge/security/iam-models-benchmark]]) consolidou 7 patterns transversais de AWS/GCP/Cloudflare/Azure/Okta. Aplicáveis ao MS:

- **Pattern 1** — Permission Groups desacoplados de Papéis Humanos (Cloudflare é o mais explícito).
- **Pattern 2** — Resource Scoping com Hierarquia (`tenant → unit → membership → resource`).
- **Pattern 4** — Default Roles por Group (onboarding sem toque individual).
- **Pattern 6** — Tag-based ABAC simples (tags `tenant`, `plan`, `role_type` sem DSL complexa).

Patterns 3 (CEL), 5 (PIM), 7 (Group Membership Rules dinâmicas) ficam **fora** do M1 (registrados como debt — ver §Alternativas e [[../patterns]]).

## Decisão

**Modelo IAM canônico do Master Síndico = Cloudflare-style adaptado**, com **7 entidades primárias** (a `Membership` foi destacada como entidade explícita para resolver o caso multi-Group por Member — ver §2.5):

```
Member ──┐                                       ┌──▶ Plan      (billing tier, ADR-0032)
         │                                       │
         │            (one Member               │
         │             can have N               │
         │             Memberships)             │
         │                                       │
         ▼                                       │
   Membership ─────▶ Group ── default_role_id ──┴──▶ Quota     (derived from Plan + overrides)
   (member×group×    │
    role×status)     │
         │           ▼
         └──▶ Role ──▶ PermissionGroup ──▶ Permission(action, resource_pattern)
                          (capacidade           (átomo, avaliado em ABAC)
                           técnica)

Resource hierarchy:  ms://platform
                     ms://tenant/{tenant_id}
                     ms://tenant/{tenant_id}/unit/{unit_id}
                     ms://tenant/{tenant_id}/.../{nested}
                     ms://group/{group_id}
                     ms://group/{group_id}/member/{user_id}
```

> **Nota terminológica**: o cliente usou informalmente `(group_role)_*_(role_type)` no insight original. **Formalizamos como `(group_kind)_(role_type)`** — `group_kind` é o **enum-tipo do agrupamento** (condo, enterprise, etc.); `role_type` é a **função interna** (admin, assistant, dependent, etc.). A composição é a **chave canônica de Role.role_name**.

### 0. Princípios fundamentais (guardrails invariantes)

Declarados explicitamente porque orientam todas as decisões de catálogo PermissionGroup, role composition, policy evaluation e admin self-service. Alinhados com [benchmark §1.9 AWS lições](../../../../10-knowledge/security/iam-models-benchmark#19-lições-aws) e são **não-negociáveis** no produto:

1. **Default = zero permissão** (fail-closed). Membership nova sem Role atribuída = deny para tudo (não herda platform). Catálogo PG não tem entrada "catch-all". Este é o implicit deny da AWS ([doc oficial](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)).
2. **Permissões são aditivas**. Múltiplas PermissionGroups na Role somam capacidades; não há subtração entre PGs. Se `pg.assembleia.read` + `pg.assembleia.moderate` = membro tem ambas.
3. **Deny > Allow**. Caso futuros Deny Policies sejam adotados (debt M2+ com CEL/OPA), `deny` **vence** qualquer `allow`. Fail-safe design.
4. **Least Privilege**. Role templates M1 começam **minimalistas** (o mínimo para o role_type ser útil no happy-path); escalar via adição explícita via painel admin. Nada de role "catch-most" herdada.
5. **Permission Boundary (guard-rail de Group)**. Veja §2.a — `Group.boundary_permission_group_ids[]` define o teto absoluto; nenhuma Role dentro do Group pode conter PGs fora do boundary, independente de quem atribui. Espelha [AWS Permissions Boundary](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) (§1.7 do benchmark).
6. **Creator ⊇ target**. Ao criar/editar Role ou atribuir Role a Membership, o **conjunto alvo** de PGs deve ser subconjunto das PGs efetivas do **criador**. Impede privilege escalation indireta (§Anti-escalada).

### 1. Member (identidade)

- 1:1 com user em **Zitadel** (ADR-0003). Não tem permissões intrínsecas.
- Identificado por `Zitadel sub` claim. Chave externa `user_id` (UUIDv7).
- **Pode estar em N Groups simultaneamente** via N Memberships independentes (ver §2.5). Ex: João é titular do condo A, dependente de outra unidade no condo B, e employee de uma empresa Connect Me — 3 Memberships, 3 Roles independentes.
- **Tags do Member** carregadas via OIDC custom claims (são **estado de UI**, **não** fonte de authz — authz sempre passa `tenant_id` + `group_id` explícitos no request):
  - `tags.tenant_active` — condomínio/grupo selecionado **no momento** na UI (apenas filtro de visualização — não autoriza nada)
  - `tags.plan_active` — plano vigente do `tenant_active` (deriva de Stripe — usado para feature gating em UI; authz revalida no backend)
  - `tags.mfa` — boolean (claim atual da sessão)
  - `tags.device_compliant` — boolean (para INV-021/one-device)
  - `tags.memberships_count` — int (UI mostra seletor multi-tenant se > 1)

**Importante** (anti-pattern §7 do benchmark — single-tenant assumptions): authz **nunca** confia em `tags.tenant_active` para decidir Allow/Deny. Cada chamada autenticada carrega `(user_id, tenant_id, action, resource_uri)` como tupla obrigatória; backend resolve Membership ativa correspondente a `(user_id, tenant_id)` e usa **essa** Role/PermissionGroups.

### 2. Group (coleção organizacional de pessoas)

- Coleção de Members com escopo organizacional bem-definido. **Não é capacidade técnica** (Pattern 1).
- Tipos canônicos do MS (`group_kind`):
  - `platform` — Master Síndico operação (Anthropic do produto)
  - `condo` — condomínio (síndico + conselho + moradores titulares + dependentes)
  - `enterprise` — empresa Connect Me (PJ formal aprovada)
  - `local_company` — empresa local Connect Me (PJ menor / autônomo)
  - `marketing_agency` — agência de marketing
  - `condo_administrator` — administradora condominial (gere N condomínios)
- **Atributos do Group**:
  - `group_id` (UUIDv7)
  - `group_kind` (enum acima)
  - `display_name` (livre — "Empresa XYZ Ltda", "Sindicato Edifício Aurora")
  - `default_role_id` (FK para Role — **fonte canônica única** da role auto-atribuída a novos members)
  - `boundary_permission_group_ids[]` — **Permission Boundary** do Group (teto absoluto de capacidades para qualquer Role dentro desse Group). Espelha [AWS Permissions Boundary](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) (§1.7 do benchmark). Default por `group_kind` (ex.: `condo` boundary inclui PGs governanca/assembleia/finance/score/member/compliance; não inclui `pg.platform.*`). Ver §2.a.
  - `plan_id` (FK billing — ADR-0032)
  - `parent_group_id` opcional (suporta hierarquia, ex: administradora ↑ condomínios geridos)
  - `tenant_id` opcional — `condo` materializa em tenant; demais grupos podem ter tenant próprio (enterprise é tenant Connect Me)
  - `tags` (k:v livre) — `region`, `segment`, `vertical`
- Tipos `condo`, `enterprise`, `local_company`, `marketing_agency`, `condo_administrator` mapeiam 1:1 com **tenant SaaS**, mantendo [[0021-multi-tenant-row-based]] e [[0029-tenant-id-typed]].

### 2.a Permission Boundary (guard-rail do Group)

Boundary é **contrato do platform_admin com o Group**: define o universo de PermissionGroups que Roles daquele Group podem conter. **Group admin compõe Roles livremente** dentro desse universo; **não consegue sair dele** sem envolver `platform_admin` (que é o único a editar o boundary).

**Exemplo concreto**:

```
Group "Sindicato Aurora" (group_kind=condo, plan=plus)
  boundary_permission_group_ids = [
    pg.governanca.*,
    pg.assembleia.*,
    pg.member.*,
    pg.finance.read,
    pg.finance.create_invoice,    // não inclui approve_payment (reservado para role custom com aprovação explícita platform)
    pg.compliance.audit_read,
    pg.lms.read, pg.lms.enroll,
    pg.score.read,
    pg.iam.role.create,            // gerir sub-roles dentro do condo
    pg.iam.role.assign_permission_group,
    pg.iam.membership.*,
  ]
  # NÃO inclui: pg.connect_me.approve_vendor (é platform-side),
  #             pg.platform.* (nunca para condo),
  #             pg.finance.approve_payment (requer permissão especial)
```

**Invariante** (DB-enforced):

- **INV-IAM-005** — `∀ role ∈ roles(group): role.permission_group_ids ⊆ group.boundary_permission_group_ids`. Violações são rejeitadas pelo service `RoleService.setPermissionGroups(role_id, pgs)` com erro `IAM_BOUNDARY_VIOLATION`.

**Evolução do boundary**:

- Apenas `platform_admin` com PG `pg.iam.group.edit_boundary` pode alterar `boundary_permission_group_ids[]` (ver §4 categoria `iam`).
- Aumentar boundary = sem efeito retroativo em Roles existentes (não adiciona PGs; apenas permite adições futuras).
- **Reduzir boundary** = efeito imediato via `RoleService.rebalance()`: Roles com PGs agora fora do boundary têm esses PGs **removidos** (audit log `policy_change_log` entry `boundary_reduction_trim`); cache invalidado.
- Cliente (João) ou equipe de produto pode solicitar expansion por `POST /platform-admin/groups/{id}/boundary-request` — workflow moderado por `platform_admin`.

**Boundary ≠ Plan**:

- `Plan` determina **quais PGs são efetivas** via condition `plan_in` (time-of-access gate).
- `Boundary` determina **quais PGs podem estar na Role** (design-time gate).
- Exemplo: `plan=base` pode ter `pg.assembleia.vote` no boundary mas condition `plan_in: ["plus","pro"]` na Permission nega em runtime — usuário com base pré-disposto a upgrade, sem mexer em Role.

### 2.5 Membership (vínculo Member × Group × Role)

Entidade explícita — **fonte canônica única do vínculo** entre pessoa, agrupamento e função. Resolve o caso multi-Group por Member (síndico que é também titular em outro condo, dependente em terceiro, employee em empresa).

- **Atributos**:
  - `membership_id` (UUIDv7)
  - `member_id` (FK Member — Zitadel user)
  - `group_id` (FK Group)
  - `role_id` (FK Role — escopo do `group_id`; constraint DB garante `role.group_id = membership.group_id`)
  - `status` ∈ `pending_invitation` / `active` / `suspended` / `ended` (espelha [[0021-multi-tenant-row-based]] + matriz funcional do sub-produto governança)
  - `joined_at`, `ended_at` (audit time-track)
  - `invited_by_member_id` (audit chain de convites)
  - `tags` (k:v livre — sobrescreve tags do Group/Role para esse vínculo, ex: `region_override`)
- **Invariantes** (DB-enforced):
  - **INV-IAM-001**: `(member_id, group_id, status='active')` é **único** — um member não tem 2 memberships ativos no mesmo group.
  - **INV-IAM-002**: `member_id × Group(group_kind='condo', role_type='titular', status='active')` por unit é único — preserva BR-GI-004 (1 titular ativo por unit, herdado de [[../../00-product/sub-produtos/01-governanca-institucional]]).
  - **INV-IAM-003**: `role.group_id = membership.group_id` (impossível atribuir role de outro group).
  - **INV-IAM-004**: `Group.default_role_id` referencia uma Role com `role.group_id = group.id` (consistência hierárquica).
  - **INV-IAM-005** (boundary): `role.permission_group_ids ⊆ role.group.boundary_permission_group_ids` — ver §2.a.
  - **INV-IAM-006** (creator ⊇ target, anti-escalada): em `POST /admin/groups/{id}/roles` ou `PATCH .../permission-groups`, `new_pgs ⊆ (creator.effective_pgs ∩ group.boundary_pgs)`. Em `POST /admin/.../members/{user_id}/role`, `target_role.pgs ⊆ creator.effective_pgs`. Creator com PG `pg.iam.role.*` não consegue transferir PGs que ele próprio não possui. Ver §Anti-escalada.
- **Authz semantics**:
  - **Toda** request autenticada passa `(user_id, tenant_id, action, resource_uri)`.
  - Backend resolve **a Membership ativa** correspondente a `(user_id, group_id_of_tenant_id)`. Se não houver → deny.
  - PermissionGroups effective do request = união dos PGs da Role da Membership ativa **+** Plan-gates **+** condition-eval (ver §Ordem).
- **Cache key ABAC** revisada: `abac:{user_id}:{group_id}:{action}:{resource_uri_canonical}` (substitui `tenant_id` por `group_id` — mais preciso para grupos não-tenant como administradora).
- **Conflict resolution multi-Group**:
  - Member com 2 memberships em condos diferentes (titular em A, dependente em B): UI seleciona qual; cada request carrega `tenant_id` explícito → resolve a membership certa.
  - Member com membership em `enterprise` E `condo` (ex.: empresa cujo dono é também síndico): Group admin controla via convite; sem promoção implícita; cada UI session opera em **1 Group ativo**.
  - **Sem cross-Group permission spillover**: capacidade técnica de um Group **não vaza** para outro. Auditoria forte.

### 3. Role (composição interna do Group)

- Nomeada via padrão `{group_kind}_{role_type}` — insight do usuário, formaliza ARI (Role Identity).
- **Único role primitivo `admin` no nível `platform`** — é o único hard-coded; tudo o mais é criado/configurado via painel:
  - `platform_admin` → super-power transversal (com elevação JIT futura — debt M2+).
- Roles compostas ao criar Group (ou via template do `group_kind`):
  - **Group `condo`** → `condo_syndic_admin` (default), `condo_syndic_assistant`, `condo_council_member`, `condo_resident_titular`, `condo_resident_dependent`
  - **Group `enterprise`** → `enterprise_admin` (default), `enterprise_employee`, `enterprise_marketing`, `enterprise_finance`
  - **Group `local_company`** → `local_company_admin` (default), `local_company_employee`
  - **Group `marketing_agency`** → `marketing_agency_admin` (default), `marketing_agency_employee` (com ABAC delegação para tenants clientes — D-095/D-102)
  - **Group `condo_administrator`** → `condo_administrator_admin` (default), `condo_administrator_employee` (vincula condo via parent_group)
- **Default Role** (Pattern 4): atributo do Group. Quando member entra no Group sem explicit role, herda o default. Admin do Group pode trocar/sobrescrever per-member.
- **Sub-roles dentro do Group**: Group admin compõe novos sub-roles colando Permission Groups via painel, **sem deploy**. Apenas `platform_admin` precisa de deploy.
- Atributos da Role:
  - `role_id` (UUIDv7)
  - `group_id` (FK Group — escopo da Role)
  - `role_type` (string livre, sugerido enum por `group_kind`: admin / assistant / employee / dependent / etc.)
  - `role_name` canonical = `{group_kind}_{role_type}` (derivado, único por `group_id`)
  - `permission_group_ids[]` (composição N:M)
  - `is_system` (boolean — `true` para `platform_admin`, false para custom)
- **Default role**: NÃO é flag na Role — é FK `Group.default_role_id` (fonte canônica única, ver §2). Removido `is_default` da Role para evitar circularidade. Invariante INV-IAM-004 (§2.5) garante consistência.

### 4. PermissionGroup (capacidade técnica)

- Coleção nomeada de Permissions atômicas. **Independente de Group humano** (Pattern 1).
- Catálogo mínimo M1 (curado por domínio, gerido por owner técnico):
  - `pg.governanca.read`, `pg.governanca.moderate`, `pg.governanca.edit_plano_diretor`
  - `pg.assembleia.read`, `pg.assembleia.create`, `pg.assembleia.moderate`, `pg.assembleia.vote`, `pg.assembleia.close`, `pg.assembleia.ata_sign`
  - `pg.connect_me.list`, `pg.connect_me.contract`, `pg.connect_me.approve_vendor` (platform-side), `pg.connect_me.dispute`
  - `pg.video.read`, `pg.video.upload`, `pg.video.moderate`
  - `pg.lms.read`, `pg.lms.enroll`, `pg.lms.create_course`, `pg.lms.certify`
  - `pg.member.read`, `pg.member.invite`, `pg.member.revoke`, `pg.member.promote`
  - `pg.finance.read`, `pg.finance.create_invoice`, `pg.finance.approve_payment`
  - `pg.compliance.audit_read`, `pg.compliance.fix_noncompliance`, `pg.compliance.lgpd_export`
  - `pg.score.read`, `pg.score.recompute_request`
  - **Categoria `iam` (executar ≠ delegar — espelha [AWS `iam:*` actions](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html))**:
    - `pg.iam.role.create`, `pg.iam.role.delete`, `pg.iam.role.edit` — criar/editar/apagar Roles dentro do Group
    - `pg.iam.role.assign_permission_group` — re-bind PGs na Role (validado contra INV-IAM-005 boundary + INV-IAM-006 creator)
    - `pg.iam.membership.create`, `pg.iam.membership.change_role`, `pg.iam.membership.end` — gerir Memberships
    - `pg.iam.group.edit_default_role` — trocar `Group.default_role_id`
    - `pg.iam.group.edit_boundary` — **apenas `platform_admin`**: editar `Group.boundary_permission_group_ids`
    - `pg.iam.policy.dry_run` — endpoint dry-run (debug, não muta estado)
    - `pg.iam.policy.simulate_as` — endpoint simulate-as-user (debug)
    - `pg.iam.audit.read` — query do `policy_change_log`
  - `pg.platform.tenant_admin`, `pg.platform.global_moderate`, `pg.platform.audit_full` (apenas `platform_admin`)
- Permission atômica = `(action, resource_pattern, conditions)` no formato:
  ```
  action:           "assembly:vote"               // namespace:verb
  resource_pattern: "ms://tenant/{tenant_id}/assembly/{*}"
  conditions:       { plan_in: ["base","plus","pro"], mfa: true_if_step_up }
  ```

### 5. Quota (limites quantitativos)

- **Separada de IAM** (não é Permission). Avaliada em business logic via service `QuotaService`. Espelha [AWS Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html): soft (alerta 80%) vs hard (bloqueia 100%), com override por escopo.
- Atributos:
  - `quota_id` (UUIDv7)
  - `scope_type` ∈ `global` / `plan` / `group` / `membership` — nível da regra (precedence abaixo)
  - `scope_id` (FK correspondente ao scope_type; nullable para `global`)
  - `metric` (enum — `members_max`, `condos_max`, `connect_me_calls_year`, `videos_year`, `dependents_per_titular`, `sms_per_day`, `lgpd_exports_per_month`, etc.)
  - `limit` (int) — teto hard
  - `soft_threshold` (float, default 0.80) — percentual do limit que dispara alerta
  - `period` (enum — `lifetime` / `year` / `month` / `day`)
  - `current` (counter — refresh via webhook ou job idempotente)
  - `on_soft_exceeded` (enum — `alert_only` / `alert_and_throttle`) — default `alert_only`
  - `on_hard_exceeded` (enum — `block` / `queue` / `allow_with_audit`) — default `block`
  - `derived_from_plan` (boolean — `true` quando o limite vem do Plan; admin pode sobrescrever)

**Precedence de resolução** (do mais específico ao mais genérico, primeiro match vence):

```
1. scope_type='membership', scope_id=membership.id         (override per-user)
2. scope_type='group',      scope_id=group.id              (override per-group)
3. scope_type='plan',       scope_id=group.plan_id         (default do plano)
4. scope_type='global',     scope_id=null                  (fallback do produto)
```

Isso espelha AWS Service Quotas que permite quotas em nível account/region/resource ([doc oficial](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html)).

**Métricas operacionais** (em [[0020-opentelemetry-grafana-sentry]]):

- `quota_utilization{metric, scope_type, scope_id} = current / limit`
- Alarmes: 80% → warning no Grafana; 100% → alert crítico + action handler (block/queue/allow_with_audit)
- Dashboard `quota_status` por Group visível no painel admin (copy-from AWS Service Quotas Console)

**Exemplos concretos**:

- Group `enterprise` plano `pro` → quota `scope=plan, plan_id=pro, metric=connect_me_calls_year, limit=30` (D-079)
- Admin override caso-a-caso (D-111) → adiciona `scope=group, group_id=G-XYZ, metric=connect_me_calls_year, limit=unlimited`; regra nível 2 vence
- Membership específica de agência com cap personalizado → `scope=membership, scope_id=M-789, metric=videos_year, limit=100`

**Action handler `on_hard_exceeded`**:

- `block` — retorna `HTTP 429 Quota Exceeded` (padrão para quotas de consumo tipo SMS, videos)
- `queue` — enfileira request, processa quando contador reseta (útil para quotas com período curto: SMS/day)
- `allow_with_audit` — permite overage, registra em `quota_overage_log`, bill à parte no ciclo Stripe seguinte (útil para Connect Me quando cliente autorizou usage-based em contrato)

### 6. Plan (billing tier — separado de IAM)

- **Já existe** via [[0032-global-plans-stripe-style]] (`trial`/`base`/`plus`/`pro`).
- IAM consulta `plan_id` do Group para resolver:
  - Permission Groups habilitadas (algumas só vêm com `plus+`)
  - Quotas default (preenchidas no Quota como `derived_from_plan=true`)
- **Plan ≠ Role**. Mudar plano não muda Role; muda apenas o conjunto resolvido de Permissions ativas (via condition `plan_in`).

---

### Resource hierarchy & escopo

URI-style hierárquico (Pattern 2, inspirado em GCP folders/Azure scopes):

```
ms://platform                                          (root — só platform_admin)
ms://group/{group_id}                                  (escopo do grupo)
ms://group/{group_id}/member/{user_id}                 (member-scoped)
ms://tenant/{tenant_id}                                (condomínio / enterprise / etc.)
ms://tenant/{tenant_id}/unit/{unit_id}
ms://tenant/{tenant_id}/unit/{unit_id}/membership/{m_id}
ms://tenant/{tenant_id}/assembly/{a_id}
ms://tenant/{tenant_id}/assembly/{a_id}/vote/{v_id}
ms://tenant/{tenant_id}/connect_me/vendor/{v_id}
ms://tenant/{tenant_id}/lms/course/{c_id}
ms://tenant/{tenant_id}/timeline/{entry_id}
```

- Match com **prefix glob**: `ms://tenant/T-001/assembly/*` cobre todas as assembleias do tenant T-001.
- **Herança permissiva**: bind em `ms://tenant/T-001` cobre descendentes; bind em `ms://tenant/T-001/assembly/A-9` cobre só A-9.
- **Effective permissions**: UI mostra resolvido (não apenas assignments) — Pattern 2 pitfall.

### Tag-based ABAC simples (Pattern 6)

Tags em principal (Member) e resource. Avaliação em condition:

```yaml
condition:
  resource.tags.tenant == request.tenant_id   # nunca confia em tags.tenant_active (UI-state)
  resource.tags.plan in principal.plan_allowed_for(group_id)
```

Tags definidas no M1 (curadoria fechada):

| Tag           | Em Member?         | Em Resource?       | Origem                              |
|---------------|--------------------|--------------------|-------------------------------------|
| `tenant`      | (UI-state apenas)  | tenant_id          | request.tenant_id é a fonte real    |
| `plan`        | plan_allowed[group_id] | plan_required  | Stripe webhook                      |
| `role_type`   | role_type via Membership ativa | role_required | Role assignment             |
| `region`      | sim                | sim                | Group                               |
| `mfa`         | sim                | step_up_required   | OIDC claim                          |
| `device_id`   | sim                | —                  | INV one-device                      |

**Regra de governança**: recurso novo sem tag `tenant` → **deny** (mitiga tagging drift).

### Vocabulário condicional fechado M1

Apenas as conditions abaixo são suportadas em M1 (anti-Pattern 7 — Overly Complex DSL). Tudo fora dessa tabela é compile-time error no policy linter:

| Condition           | Tipo               | Sintaxe                                         | Quando aplicar |
|---------------------|--------------------|-------------------------------------------------|----------------|
| `plan_in`           | `string[]`         | `{plan_in: ["base","plus","pro"]}`              | Feature por tier de plano |
| `mfa_required`      | `boolean`          | `{mfa_required: true}`                          | Action requer claim mfa=true na sessão |
| `step_up_required`  | `enum:reason`      | `{step_up_required: "policy_change"}`           | Step-up MFA específico para ação crítica |
| `device_compliant`  | `boolean`          | `{device_compliant: true}`                      | INV-021 one-device |
| `time_window`       | `cron-like`        | `{time_window: "Mon-Fri 08:00-20:00 BRT"}`      | Restrição operacional |
| `geo_in`            | `string[]` (ISO)   | `{geo_in: ["BR"]}`                              | LGPD residency |
| `quota_metric`      | `enum`             | `{quota_metric: "connect_me_calls_year"}`       | Gate Quota (avaliado por QuotaService) |

**Crescer requer ADR menor** (cada novo condition entra via ADR-0039.x companion). M2+: avaliar promoção para CEL/Rego se vocabulário passar de 12 conditions.

### Ordem de avaliação (preserva [[0012-abac-redis-cache]])

```
1. admin_bypass         → if principal in platform_admin → allow (com audit + step-up se ação ≥ critical)
2. tenant_match         → request.tenant_id resolve Membership ativa do principal (else deny)
                          [DB-layer fail-safe: [[0034-postgres-rls-default-on-tenant-guard]] reforça via RLS]
3. resource_scope       → effective permissions cobrem resource_uri (resolved via prefix match)
4. permission_match     → action ∈ permission_groups effective do principal (via Role da Membership)
5. plan_gate            → resource.tags.plan_required ∈ plan_allowed_for(group_id) (deriva de Plan)
6. quota_gate           → QuotaService.canConsume(metric, group_id, +1)
7. condition_eval       → mfa_required, step_up_required, device_compliant, time_window, geo_in (vocabulário fechado acima)
fail-closed em qualquer passo
```

### Cache & invalidação revisada

Keys:

- `abac:{user_id}:{group_id}:{action}:{resource_uri_canonical}` — decisão atomica, TTL 5min
- `abac:role:{role_id}:members` — **SET** de `member_id`s afetados por essa Role (atualizado on Membership.create/end)
- `abac:group:{group_id}:roles` — SET de `role_id`s do Group

Invalidação:

- **Stripe `subscription.updated`** → invalida `abac:{user_id}:*` (todos os groups do user, plan mudou)
- **Role.permission_group_ids alterado** → resolve `abac:role:{role_id}:members` → `SUNION` → invalida cada user afetado (cobre Group `enterprise`/`marketing_agency` que não são tenant)
- **Membership.create/end/role_change** → invalida `abac:{user_id}:{group_id}:*` para esse member específico
- **Group.default_role_id swap** → não invalida cache (afeta apenas novos members futuros)
- **Catálogo PermissionGroup mudou** → flush global coordenado (raro, ADR menor required)

TTL 5min cobre janelas de webhook drop (consistência eventual).

### Painel admin self-service (inspirado em Cloudflare)

`platform_admin` administra **catálogo global** (Permission Groups, Roles templates por `group_kind`, boundaries de Group). Group admin (qualquer Role com PGs `pg.iam.*`) administra **dentro do Group**, limitado por (a) boundary do Group (INV-IAM-005) e (b) PGs próprias do criador (INV-IAM-006):

| Endpoint | PG requerido | Regras de validação |
|---|---|---|
| `GET /admin/groups/{id}/roles` | `pg.iam.role.create` OR `pg.iam.audit.read` | — |
| `POST /admin/groups/{id}/roles` | `pg.iam.role.create` | new_pgs ⊆ group.boundary ⊆ creator.effective_pgs (INV-IAM-005 + INV-IAM-006) |
| `PATCH /admin/groups/{id}/roles/{role_id}/permission-groups` | `pg.iam.role.assign_permission_group` | idem acima; audit log obrigatório |
| `PATCH /admin/groups/{id}/default-role` | `pg.iam.group.edit_default_role` | new_default.group_id = id |
| `POST /admin/groups/{id}/members/{user_id}/role` | `pg.iam.membership.change_role` | **target_role.pgs ⊆ creator.effective_pgs** (anti-escalada indireta) |
| `GET /admin/groups/{id}/effective-permissions/{user_id}` | `pg.iam.audit.read` | — |
| `POST /admin/policies/dry-run` | `pg.iam.policy.dry_run` | read-only simulação |
| `GET /admin/simulate-as/{user_id}?action=X&resource=Y` | `pg.iam.policy.simulate_as` | read-only; retorna decisão + passo que disparou allow/deny |
| `GET /admin/audit/policy-changes` | `pg.iam.audit.read` | query imutável |
| `PATCH /platform-admin/groups/{id}/boundary` | `pg.iam.group.edit_boundary` (apenas `platform_admin`) | trigger `RoleService.rebalance()` se reduzir boundary |

**Anti-escalada indireta** (INV-IAM-006): o ataque canônico seria `enterprise_admin` (que tem `pg.iam.role.create` + `pg.iam.role.assign_permission_group`) criar role `shadow` com `pg.finance.approve_payment` **sem possuí-la pessoalmente**. A regra `creator ⊇ target` no service layer impede isso antes de qualquer persistência. Audit `policy_change_log` registra tentativas negadas com reason `IAM_ESCALATION_ATTEMPT`.

Audit log imutável de **toda** mudança de policy ([[0013-timeline-immutable]] + [[0033-ata-timeline-db-immutability]] aplicado a `policy_change_log`). Campo `reason_of_change` obrigatório em toda request de mutação (free text, ≥12 chars, grep-able).

### Trade-offs explícitos

- **Sem PIM (JIT elevation, Pattern 5)** no M1. `platform_admin` é static. **Mitigação**: MFA obrigatório ([[0026-admin-mfa-m1]]) + step-up em ações críticas (timeline imutável, alterar policy, exportar LGPD). Debt registrada para M2+ (ADR futura).
- **Sem CEL (Pattern 3)**. Conditions são vocabulário fechado (tabela acima). **Mitigação**: vocabulário restrito = legibilidade; admin não escreve DSL. Se a complexidade crescer, evoluir para OPA/Cedar (debt M2+ no MOC).
- **Sem Group Membership Rules dinâmicas (Pattern 7)**. Member entra em Group por convite explícito (síndico convida titular; titular convida dependente; admin enterprise convida employee). **Mitigação**: simplicidade de auditoria; LGPD friendly. Para `condo_administrator` que gere N condos, herança por `parent_group_id` cobre 80% do uso. Debt M3+ se HRIS de empresa cliente quiser sync via SCIM.

## Consequências

### Positivas

- **Acaba a Role Explosion**: novo subtipo de empresa não força ADR — admin do Group compõe via painel.
- **Capacidade técnica desacoplada de pessoa**: re-org de Permission Groups (renomear `pg.assembleia.moderate` → split em `moderate_chat` / `moderate_pauta`) não exige re-criar roles no Group.
- **Onboarding limpo**: criar Group → default role já existe → primeiro member entra com permissões corretas; depois Group admin convida demais (Pattern 4).
- **Hierarquia de tenants natural**: condo_administrator vê filhos via `parent_group_id` sem rebind manual de policies (Pattern 2).
- **Audit limpo**: cada mudança de Role/PermissionGroup/Quota é evento imutável.
- **Painel admin é o produto**: vende-se "self-service de permissões granulares" (paridade com Cloudflare/AWS Org/GCP Folders).
- **Reaproveita Zitadel** (ADR-0003): claims OIDC carregam `tags.tenant_active`, `tags.plan_active`, `role_id`, `permission_group_ids` (resolvidos no token issuance ou via introspection cached).
- **Reaproveita Redis ABAC cache** (ADR-0012): nada quebra.

### Negativas

- **Migration de estado existente** (legacy persona pré-estabelecida): mapping 1:1 inicial — **com legacy** (existente em backend Go hoje) e **sem legacy** (Roles novas que existem só no novo modelo, primeiros members atribuídos manualmente pelo Group admin):

  **Com legacy persona (migration automática)**:
  - `syndic_admin` → Group `condo` + Role `condo_syndic_admin` (default_role_id do Group)
  - `syndic_assistant` → Group `condo` + Role `condo_syndic_assistant`
  - `resident_titular` → Group `condo` + Role `condo_resident_titular`
  - `resident_dependent` → Group `condo` + Role `condo_resident_dependent`
  - `enterprise_admin` → Group `enterprise` + Role `enterprise_admin` (default_role_id)
  - `local_company_admin` → Group `local_company` + Role `local_company_admin` (default_role_id)
  - `marketing_agency_admin` → Group `marketing_agency` + Role `marketing_agency_admin` (default_role_id)
  - `platform_admin` → Group `platform` + Role `platform_admin` (system, único primitivo)

  **Sem legacy persona (Role criada vazia; Group admin atribui primeiros members via painel)**:
  - `condo_council_member` (conselheiro) — não existe persona Go hoje; sub-role do Group `condo`
  - `enterprise_employee` — sub-role do `enterprise`; convidado pelo `enterprise_admin`
  - `enterprise_marketing` — sub-role do `enterprise` (D-095/D-102 cobrem delegação cross-tenant)
  - `enterprise_finance` — sub-role do `enterprise`
  - `local_company_employee` — sub-role do `local_company`
  - `marketing_agency_employee` — sub-role do `marketing_agency` (delegação ABAC para tenants clientes)
  - `condo_administrator_admin` — Role default do Group `condo_administrator` (administradora condominial — D-114)
  - `condo_administrator_employee` — sub-role do `condo_administrator`

  Migration **idempotente** via job: para cada user com persona legada, calcular Group + Role; criar Membership ativa; preservar `consent_signed_at`, `joined_at`, `invited_by_member_id` (best-effort de inferência); seed catálogo PermissionGroup M1 (§4); seed Role templates por `group_kind`. Roles "sem legacy" são seeded vazias de Memberships — Group admin atribui via painel pós-migration.
- **Catálogo PermissionGroup vira artefato vivo**: precisa de governance (review em ADR menor a cada nova capability cross-BC).
- **UI de admin é pesada**: precisa caprichar UX no painel (effective permissions, copy-from-template, simulate-as-user) — task de design.
- **Cache invalidation mais granular**: mudanças de Role precisam derrubar cache do Group inteiro (todos members afetados). Mitigação: invalidar por padrão `abac:*:{tenant_id}:*` (já suportado por [[0012-abac-redis-cache]]).
- **Resource URI canonicalização**: dois URIs equivalentes não podem cachear como entradas diferentes. Resolver via builder único `ResourceURIBuilder.canonical(...)`.

### Neutras / observações

- Plan e Quota ficam **explicitamente separados** de IAM. Mudança de plano dispara recálculo de quotas, mas Roles e PermissionGroups permanecem; o que muda é o conjunto **effective** de Permissions (gate `plan_in`). Isso simplifica billing change → não precisa re-bind de roles.
- O catálogo M1 é fechado por agora; abrir para clientes criarem PermissionGroups custom é debt M3+ (provavelmente nunca para SaaS multi-tenant — risco de explosão).

## Alternativas consideradas

1. **Manter persona pré-estabelecida (status quo)** — recusada: anti-pattern Role Explosion confirmado pelos casos D-095 (agência), D-102 (admin agência), D-114 (administradora) onde personas novas exigiriam novas hard-coded roles.
2. **OPA + Rego policies** ([[../../../../10-knowledge/security/opa-rego]]) — recusada para M1 (complexidade DSL, curva de aprendizado para o painel admin self-service). Debt M2+ se condições crescerem (Pattern 3).
3. **Zanzibar/OpenFGA (rebac)** ([[../../../../10-knowledge/security/zanzibar-openfga]]) — recusada: mental model de relations (`viewer`, `editor` por objeto) não casa com o domínio MS, que é forte em hierarquia tenant + plan-gate. ReBAC seria over-engineering para uso atual; revisitar M3+ se Connect Me ganhar relacionamentos N:M complexos (recomendações sociais).
4. **AWS Cedar** — recusada: gerenciado AWS lock-in; CEL/JSON tem mesma expressividade sem o lock.
5. **Casbin** — recusada: força modelo único (RBAC ou ABAC), não combina nativamente com Resource Hierarchy + Tags + Plans + Quotas.
6. **GCP IAM imitado 1:1 (Org/Folder/Project + CEL)** — recusada: Cloudflare é mais próximo do mental model do cliente (painel self-service, Permission Groups explícitos) e mais simples para M1.

## Pontos abertos / debts conscientes

- **D-IAM-PIM**: PIM/JIT elevation para `platform_admin` (M2+). Hoje: MFA + step-up.
- **D-IAM-CEL**: vocabulário de condition cresce → migrar para CEL/Rego (M2+).
- **D-IAM-SCIM**: SCIM provisioning para enterprise/agência sync HRIS (M3+, [[../../../../10-knowledge/security/scim-provisioning]]).
- **D-IAM-DELEGATION**: delegação ABAC marketing_agency → tenant cliente (D-095/D-102) precisa formalizar via novo PermissionGroup `pg.platform.act_on_behalf_of` + audit trail.
- **D-IAM-CUSTOM-PG**: clientes criarem Permission Groups custom (provavelmente nunca; debt aberto para 2027).
- **D-IAM-USER-DIRECT-PG**: assignment de PermissionGroup direto ao user (bypass Role) como caso excepcional — AWS IAM Identity Center e GCP IAM permitem. Útil para ad-hoc ("Alice precisa de `pg.lgpd.export` em 1 ocasião" sem criar role dedicada). **Debt M3+** — até lá, criar Role one-off com escopo limitado é aceitável. Se adotado, tabela `member_direct_permissions(member_id, permission_group_id, granted_by, granted_at, expires_at)` com TTL obrigatório + audit.
- **D-IAM-BOUNDARY-UI**: painel visual do boundary do Group (equivalente ao [AWS Permissions Boundary UI](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)) com diff antes/depois ao editar — M2+ (M1 entrega CRUD funcional sem UI rica).

## Aplicação progressiva

| Marco | Entrega |
|---|---|
| **pré-M1 (Sprint 10)** | Catálogo Permission Groups + boundary templates por `group_kind` + Role templates + painel admin platform-side (CRUD PG/Role/Boundary); migration idempotente persona → Group+Membership+Role; INV-IAM-001..006 DB-enforced; Quota table com `scope_type` + soft/hard |
| **M1 (2026-05-18)** | Painel admin de Group (CRUD sub-roles sob boundary, default role swap, member role assignment com anti-escalada INV-IAM-006, audit log, dry-run, simulate-as); ordem de avaliação com Plan + Quota gates; métrica `quota_utilization` em Grafana |
| **M2+** | PIM/JIT elevation para `platform_admin`; eventual migração para OPA/Cedar se conditions crescerem; UI rica de boundary editor com diff |
| **M3+** | SCIM provisioning enterprise/agência; eventual ReBAC para Connect Me social graph; D-IAM-USER-DIRECT-PG se necessidade emergir |

## Referências

- [[../../../../10-knowledge/security/iam-models-benchmark]] — benchmark consolidado AWS/GCP/Cloudflare/Azure/Okta (1617 linhas, 7 patterns + 8 anti-patterns)
- [[../../../../10-knowledge/security/iam]] — fundamentos IAM
- [[../../../../10-knowledge/security/abac]] — ABAC pattern reference
- [[../../../../10-knowledge/security/least-privilege]]
- [[../../../../10-knowledge/security/delegation-impersonation]] — caso D-095/D-102
- [[../../../../10-knowledge/providers/cloudflare/access]] — Members/Groups/Permission Groups source
- [[../../../../10-knowledge/providers/cloudflare/_moc]] — Cloudflare provider catalog
- [[0003-zitadel-oidc-provider]] — IdP autenticação (mantido)
- [[0012-abac-redis-cache]] — engine + cache (mantido, ordem atualizada)
- [[0021-multi-tenant-row-based]] — multi-tenant base
- [[0026-admin-mfa-m1]] — MFA admin (substitui PIM em M1)
- [[0029-tenant-id-typed]] — tenant ID type-driven
- [[0032-global-plans-stripe-style]] — Plan separado de Role
- [[0034-postgres-rls-default-on-tenant-guard]] — RLS reforça tenant_match no DB
- [[../../STATE]] §D-116
