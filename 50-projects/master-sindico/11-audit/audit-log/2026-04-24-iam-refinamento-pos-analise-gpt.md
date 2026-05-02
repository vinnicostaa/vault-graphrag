---
title: 2026-04-24 — Refinamento ADR-0039 pós-análise do material GPT (R1-R6)
type: audit
tags:
  - audit
  - iam
  - cross-check
  - master-sindico
  - v2
  - fase-14-1
status: completed
date: 2026-04-24T00:00:00.000Z
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
---

# Refinamento ADR-0039 pós-análise GPT

## Contexto

O cliente João compartilhou em 2026-04-24 o material que um ChatGPT externo havia elaborado com ele em 4 rodadas sobre IAM (hierarquia vs controle; AWS/GCP/CF; Permission Boundary; Quota separada). Pediu análise minuciosa cruzada com docs e implementações reais.

Análise feita cruzando cada claim do GPT contra:
- benchmark F14.2 [[../../../../10-knowledge/security/iam-models-benchmark]] (1617 linhas, AWS/GCP/CF/Azure/Okta com citações oficiais)
- [[../../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]] atual

## Veredito

**Material GPT decente com 6 insights úteis + 3 simplificações enganosas + 1 armadilha clara**. A afirmação central do GPT ("você mistura hierarquia organizacional com controle de acesso") **já estava resolvida** no ADR-0039 via Pattern 1 (Group ≠ PermissionGroup). O GPT chegou praticamente no mesmo desenho com nomes diferentes.

Do total, **6 refinamentos (R1-R6) aplicados** ao ADR-0039. **2 sugestões rejeitadas** por contrariar anti-patterns já identificados.

## Refinamentos aplicados

| # | Refinamento | Origem GPT | Fonte real |
|---|---|---|---|
| R1 | Permission Boundary formal no Group (`boundary_permission_group_ids[]`) + INV-IAM-005 + §2.a dedicada | resposta 3 | [AWS Permissions Boundary](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) + benchmark §1.7 |
| R2 | §0 Princípios fundamentais (default=zero, aditiva, deny>allow, least privilege, boundary, creator⊇target) | resposta 2 | AWS Policy Evaluation Logic + benchmark §1.9 |
| R3 | Regra da interseção em `POST /roles` + INV-IAM-006 | resposta 3 | AWS `iam:PassRole` + Permission Boundary combinados |
| R4 | Categoria `pg.iam.*` (executar ≠ delegar): `pg.iam.role.*`, `pg.iam.membership.*`, `pg.iam.group.edit_boundary`, `pg.iam.policy.dry_run/simulate_as` | resposta 3 (conceito correto, sintaxe errada como boolean) | [AWS `iam:*` actions](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html) |
| R5 | Quota refinada: `scope_type` (global/plan/group/membership), `soft_threshold`, `on_hard_exceeded`, precedence de resolução, métrica `quota_utilization` em Grafana | resposta 4 | [AWS Service Quotas](https://docs.aws.amazon.com/servicequotas/latest/userguide/intro.html) |
| R6 | Anti-escalada indireta explicitada em §Painel admin (tabela de endpoints) + reason `IAM_ESCALATION_ATTEMPT` em audit log | resposta 3 | AWS escalation mitigation + INV-IAM-006 |

## Sugestões GPT rejeitadas

| # | Sugestão GPT | Por que rejeitado |
|---|---|---|
| E2 | Flag boolean `Role.can_grant_permissions` ou `Permission.can_be_delegated` | AWS/GCP/CF não fazem assim — usam permissões específicas (`iam:AttachRolePolicy`). Booleans não compõem e perdem granularidade. Resolvido por R4 (categoria `pg.iam.*`). |
| E3 | Adotar Policies JSON AWS-style (Effect/Action/Resource/Condition/150+ condition keys) em M1 | **Anti-pattern 7** do nosso benchmark (Overly Complex DSL). AWS Policy Language tem 6 camadas de avaliação e 150+ condition keys — apropriado para AWS (200+ serviços), over-engineering para MS. Mantido vocabulário fechado de 7 conditions; evolução para CEL/OPA fica como D-IAM-CEL M2+. |

## Sugestões GPT questionáveis (ignoradas ou como debt futuro)

| # | Observação | Tratamento |
|---|---|---|
| E1 | Contradição interna: GPT oscila entre "naming estruturado `group_type:role_type` é bom" e "role_type não é necessário" | Nenhuma ação — nosso `(group_kind)_(role_type)` está correto; é naming convention, não hierarquia |
| E4 | Stack sugerida (Prisma/NestJS/NATS microserviços) | Ignorado — stack real é Go + sqlc + goose + monolito modular (ADR-0001) |
| — | `UserPermission direct` (bypass Role) | Aceito como debt **D-IAM-USER-DIRECT-PG M3+** (AWS IAM Identity Center e GCP IAM fazem). Até lá, Role one-off é aceitável. |
| — | UI rica de boundary editor com diff | Aceito como debt **D-IAM-BOUNDARY-UI M2+**. M1 entrega CRUD funcional. |

## Impacto nos artefatos pré-M1

Conditional Acceptance de ADR-0039 passou de **3 para 4 artefatos** pré-requisito:

1. ~~Catálogo PG seed file~~ (original) → **expandido** incluindo categoria `pg.iam.*`
2. **NOVO**: Boundary templates por `group_kind` (`iam/seed/group_boundaries_v1.yaml`)
3. Migration idempotente persona → Group+Membership+Role — **agora DB-enforces INV-IAM-001..006** (incluindo boundary INV-IAM-005 e anti-escalada INV-IAM-006)
4. Endpoints platform-side admin **com validação** boundary + creator⊇target; Quota table com `scope_type` + `soft_threshold` + `on_hard_exceeded`

## Entidades do modelo (final)

Passou de 7 para **7 + 1 refinada** entidades primárias:

1. Member (Zitadel)
2. Group (+ `boundary_permission_group_ids[]`)
3. Membership (+ INV-IAM-005/006)
4. Role (+ INV-IAM-005 boundary subset)
5. PermissionGroup (+ categoria `pg.iam.*`)
6. Quota (**refatorada** com scope_type + soft/hard)
7. Plan

## Métricas operacionais adicionadas

- `quota_utilization{metric, scope_type, scope_id}` → Grafana dashboard
- Alarmes 80% (warning) + 100% (crítico com action handler)
- `iam_policy_changes_total{kind, group_id}` → taxa de mudanças de policy
- `iam_escalation_attempts_total{creator_group, target_group}` → tentativas negadas pela INV-IAM-006

## Invariantes DB-enforced (final)

- **INV-IAM-001** — membership único ativo por (member, group)
- **INV-IAM-002** — 1 titular ativo por unit (preserva BR-GI-004)
- **INV-IAM-003** — `role.group_id = membership.group_id`
- **INV-IAM-004** — `Group.default_role_id` ∈ Roles do group
- **INV-IAM-005** — `role.pgs ⊆ group.boundary_pgs` (**novo**, R1)
- **INV-IAM-006** — `new_pgs ⊆ creator.effective_pgs ∩ group.boundary_pgs` (**novo**, R3+R6)

## Próximos passos

- [x] Aplicar R1-R6 ao ADR-0039 (done — 2026-04-24)
- [x] Atualizar §Conditional Acceptance (3 → 4 artefatos) (done)
- [x] Registrar refinamento em STATE §D-116 (done — nota "refinado pós-análise GPT R1-R6")
- [ ] (opcional) Dual-check das mudanças R1-R6 — decisão arquitetural não mudou, só polimento operacional. Opcional.
- [ ] (Sprint 10) Incluir `group_boundaries_v1.yaml` no seed file — novo artefato
- [ ] (Sprint 10) DB migrations adicionar `groups.boundary_permission_group_ids` + invariantes INV-IAM-005/006
- [ ] (Sprint 10) Quota table refactored: `scope_type`, `soft_threshold`, `on_hard_exceeded`, precedence resolver
- [ ] (Sprint 10) Painel platform-side `PATCH /platform-admin/groups/{id}/boundary` + trigger `RoleService.rebalance()` em boundary reduction

## Vizinhos

- [[_moc]]
- [[2026-04-23-fase14-iam-cloudflare-sms]] — ADR-0039 original + dual-check + D-116..D-118
- [[../../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]]
- [[../../STATE]] §D-116
