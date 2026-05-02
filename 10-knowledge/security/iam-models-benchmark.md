---
title: 'IAM Models Benchmark — AWS, GCP, Cloudflare, Azure AD, Okta'
type: concept
tags:
  - security
  - iam
  - idp
  - rbac
  - abac
  - benchmark
  - knowledge
doc-consulted: '2026-04-23T00:00:00.000Z'
created: '2026-04-23T00:00:00.000Z'
updated: '2026-04-23T00:00:00.000Z'
aliases:
  - IAM Benchmark
  - IAM Models Comparison
  - Cloud IAM Benchmark
---

# IAM Models Benchmark — AWS, GCP, Cloudflare, Azure AD, Okta

Análise comparativa dos **5 modelos canônicos de IAM enterprise** (2024-2026). Foco em **patterns reutilizáveis** para sistemas multi-tenant que precisam de permissões granulares expostas via painel admin self-service.

> **Escopo**: este doc é conhecimento atemporal e agnóstico de produto. Para aplicação concreta ao Master Síndico ver [[../../50-projects/master-sindico/_index]]. Para o guarda-chuva conceitual de IAM ver [[iam]]. Para modelos de autorização puros ver [[authorization-models]], [[rbac]], [[abac]], [[rebac]].

---

## TL;DR

Cinco abordagens, três eixos comuns (**identidade → agrupamento → capacidade**), uma convergência:

| # | Empresa | Primitiva de "pessoa" | Agrupamento | Capacidade técnica | Alvo/escopo | Condicional |
|---|---|---|---|---|---|---|
| 1 | **AWS IAM** | User / Role | Group | Policy (JSON IAM) | ARN | `Condition` block |
| 2 | **GCP IAM** | User / Service Account | Google Group | Role (predefined/custom) | Resource hierarchy | CEL expressions |
| 3 | **Cloudflare** | Member | Group | Permission Group | Resource Group (zones/accounts) | `effect: allow\|deny` + resources |
| 4 | **Azure AD / Entra** | User / Service Principal | Security Group | Role definition (actions/notActions) | Scope (MG → Sub → RG → Resource) | Conditional Access Policy |
| 5 | **Okta** | User | Group (static/dynamic) | Admin Role / App assignment | Org / App / Group | Network Zones, Behavior, Risk |

**Insight central**: todos separam **três coisas distintas** que software legado costuma fundir:

1. **Quem** (user, service account, workload) — identidade com credenciais.
2. **Agrupamento lógico** (group/collection) — coleção gerenciável de pessoas ou identidades.
3. **Capacidade técnica** (permission/policy) — string que descreve **o que pode ser feito** sobre **o quê**, **em qual escopo**, **sob qual condição**.

Chamar **roles** um catálogo de "pessoas" (ex.: `syndic`, `resident`, `enterprise`) é confundir a camada 2 com a camada 3. Os cinco provedores ensinam o oposto: `admin` é a única primitiva de bootstrap; todo o resto é composição **Group × Permission × Scope × Condition** gerida via painel.

### 7 Patterns transversais (detalhados em §6)

1. **Permission Groups desacoplados de Papéis Humanos** — capacidade técnica é N:M com grupo de pessoas (Cloudflare, AWS).
2. **Resource Scoping com Hierarquia** — herança org → projeto/conta → recurso, com override (GCP, Azure).
3. **Conditional Access via expressão declarativa** — CEL/JSON `Condition`/Access Policy (GCP, AWS, Azure).
4. **Default Roles por Group** — atribuição automática de capacidades quando membro entra no group (Azure, Okta).
5. **Just-in-Time Elevation (PIM)** — privilégio elevado temporário com aprovação + audit (Azure PIM, AWS STS + MFA, Okta Privileged Access).
6. **Tag-based ABAC** — atributos em recursos + condition policies em vez de explodir roles (AWS recomendação atual).
7. **Group Membership Rules dinâmicas** — regras sobre atributos do usuário alimentam group automaticamente (Okta, Azure Dynamic Groups).

### 5 Anti-patterns principais (§7)

- **Role explosion** (role nova para cada combinação persona × capacidade × escopo).
- **Permission baked into code** (permissions no código-fonte, não em policies externas).
- **God role** (admin único sem segregation of duties).
- **Static-only assignment** (sem PIM, elevação temporal ou aprovação).
- **No audit trail em policy change** (mutação sem log imutável).

---

## 1. AWS IAM

### 1.1 Primitivos

| Primitivo | Descrição | Exemplo |
|---|---|---|
| **Principal** | Entidade que faz requisições (User, Role, Federated User, AWS Service) | `arn:aws:iam::123:user/alice` |
| **User** | Identidade humana ou service com credenciais long-lived (chave de acesso ou senha) | `alice`, `ci-bot` |
| **Group** | Coleção de Users; policies atribuídas ao Group se propagam | `Developers`, `Admins` |
| **Role** | Identidade assumível (short-lived via STS); sem credenciais permanentes | `LambdaExecutionRole` |
| **Policy** | Documento JSON que **permite** ou **nega** ações em recursos | ver §1.2 |
| **Resource** | Entidade AWS (bucket S3, função Lambda, tabela DynamoDB) | `arn:aws:s3:::my-bucket/*` |
| **Session** | Instância temporária de credenciais (STS) | Token de até 12h |

### 1.2 Estrutura de Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadOnlyOnBucket",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::123456789012:user/alice" },
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::company-data",
        "arn:aws:s3:::company-data/*"
      ],
      "Condition": {
        "IpAddress": { "aws:SourceIp": "203.0.113.0/24" },
        "Bool":      { "aws:MultiFactorAuthPresent": "true" }
      }
    }
  ]
}
```

Os **5 elementos** de um Statement:

| Elemento | Função | Exemplo |
|---|---|---|
| `Effect` | `Allow` ou `Deny` (deny vence) | `"Effect": "Allow"` |
| `Principal` | Quem se aplica (só em resource-based policies) | User, Role, Account, Federated |
| `Action` | Verbo do serviço (`service:Verb`), wildcards permitidos | `s3:GetObject`, `ec2:*` |
| `Resource` | ARN(s) alvo | `arn:aws:s3:::bucket/*` |
| `Condition` | Expressão declarativa que refina | IP, MFA, tag, horário, TLS |

### 1.3 Tipos de Policy

AWS tem **6 tipos distintos** de policies, avaliados em ordem hierárquica:

1. **Identity-based policy** — anexada a User, Group, Role. "O que **esta identidade** pode fazer?"
2. **Resource-based policy** — anexada a recurso (bucket policy, Lambda resource policy). "Quem pode acessar **este recurso**?"
3. **Permission Boundary** — limite superior para uma identidade: mesmo que policy identity-based permita X, se boundary não permitir, **nega**.
4. **Service Control Policy (SCP)** — anexada a OU/Account no AWS Organizations. Guard-rail top-down para toda uma conta/OU.
5. **Session Policy** — passada inline ao `AssumeRole`; restringe ainda mais a sessão.
6. **Access Control List (ACL)** — legado S3/VPC; pré-IAM. Evitar.

**Lógica de avaliação**:

```
IF qualquer Deny explícito      → DENY
ELSE IF fora do SCP             → DENY
ELSE IF fora da Permission Boundary → DENY
ELSE IF identity allow OU resource allow → ALLOW
ELSE                            → DENY (default)
```

### 1.4 Trust Policy vs Permission Policy

Toda Role tem **duas policies ortogonais**:

| Policy | Responde | Exemplo de cenário |
|---|---|---|
| **Trust Policy** | "**Quem** pode assumir esta role?" | Lambda service, conta parceira, user federado |
| **Permission Policy** | "O que **esta role**, depois de assumida, pode fazer?" | `s3:GetObject`, `dynamodb:PutItem` |

```json
// Trust Policy (quem assume)
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "lambda.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

Essa separação permite delegation clean: time de segurança controla Trust Policy; time de app controla Permission Policy.

### 1.5 STS AssumeRole flow

```
┌─────────┐  AssumeRole(roleArn, externalId)  ┌─────┐
│  User   │ ─────────────────────────────────▶│ STS │
│ (alice) │                                   │     │
└─────────┘                                   └─────┘
      │                                          │
      │         { AccessKey, Secret,             │
      │           SessionToken, Expires }        │
      │◀─────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│ Sessão (ex.: 1h) com credenciais    │
│ temporárias + permission policy da  │
│ role (interseção com session policy │
│ se fornecida no AssumeRole).        │
└─────────────────────────────────────┘
```

**Vantagens**:
- Sem credenciais long-lived distribuídas.
- Auditável: `sts:AssumeRole` aparece no CloudTrail com `sourceIdentity`.
- Delegation cross-account com `ExternalId` para evitar confused deputy.

### 1.6 ABAC com tags (recomendação AWS 2020+)

Em vez de criar **N roles** para **N combinações**, AWS recomenda **tag-based access control**:

```json
// Identity-based policy reutilizável para qualquer projeto
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:PutObject"],
    "Resource": "arn:aws:s3:::*/*",
    "Condition": {
      "StringEquals": {
        "s3:ExistingObjectTag/Project": "${aws:PrincipalTag/Project}"
      }
    }
  }]
}
```

Princípio:
- **Tag no principal**: `alice` tem tag `Project=Phoenix`.
- **Tag no recurso**: objetos S3 têm tag `Project=Phoenix`.
- **Policy única**: "permite se tag do principal == tag do recurso".

Quando novo projeto nasce, basta **taggear** — sem mudar policy.

### 1.7 Permissions Boundary (guard-rail)

Boundary é o **teto absoluto**: nem que uma identity policy permita `iam:*`, se boundary só permite `s3:*` e `dynamodb:*`, efetivamente só pode essas.

Uso típico:
- Developers podem criar suas próprias roles/policies, **mas** dentro de um boundary imposto pelo time de segurança.
- Evita privilege escalation (dev não consegue criar role com `iam:*` para si).

Exemplo de boundary que limita dev a criar recursos só em dev-env:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "BoundaryLimitS3andDynamo",
      "Effect": "Allow",
      "Action": ["s3:*", "dynamodb:*", "iam:PassRole"],
      "Resource": "*",
      "Condition": {
        "StringEquals": { "aws:RequestTag/env": "dev" }
      }
    },
    {
      "Sid": "DenyIAMWrite",
      "Effect": "Deny",
      "Action": [
        "iam:CreatePolicy",
        "iam:DeletePolicy",
        "iam:CreateUser",
        "iam:CreateAccessKey"
      ],
      "Resource": "*"
    }
  ]
}
```

### 1.8 Session Policies (fine-grained AssumeRole)

Ao chamar `AssumeRole`, pode-se passar **Session Policy** inline que **restringe ainda mais** a Role assumida:

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123:role/DataTeam \
  --role-session-name "alice-analysis-2026-04-23" \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::analytics-2026-04/*"
    }]
  }' \
  --duration-seconds 3600
```

Mesmo que a Role `DataTeam` tenha `s3:*` em todos buckets, **esta sessão** só tem `s3:GetObject` em um prefixo específico. Ideal para broker patterns: serviço central troca credenciais plenas por credenciais minimizadas para cada request.

### 1.9 Lições AWS

- **ARN como identificador universal**: `arn:partition:service:region:account-id:resource-type/resource-id`. Toda policy referencia ARN ou padrão com wildcard.
- **Deny sempre vence Allow**: design fail-safe.
- **Policy é dado, não código**: stored como JSON em IAM; versionável, difável, testável offline.
- **Identity-based vs Resource-based**: dois ângulos do mesmo controle (cruzamento dá flexibilidade).
- **Condition é o canivete suíço**: `aws:SourceIp`, `aws:MultiFactorAuthPresent`, `aws:CurrentTime`, `aws:PrincipalTag/*`, `aws:RequestTag/*`, `aws:ResourceTag/*`.
- **Menos roles + mais tags** é a trajetória oficial.

### 1.10 Referências oficiais AWS

- AWS IAM User Guide: https://docs.aws.amazon.com/IAM/latest/UserGuide/
- Policy Evaluation Logic: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html
- ABAC Tutorial: https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_attribute-based-access-control.html
- Permissions Boundaries: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html

---

## 2. Google Cloud IAM

### 2.1 Primitivos

| Primitivo | Descrição | Exemplo |
|---|---|---|
| **Member (Principal)** | Identidade que recebe acesso | `user:`, `group:`, `serviceAccount:`, `domain:`, `allUsers`, `allAuthenticatedUsers` |
| **Role** | Coleção nomeada de permissions | `roles/storage.objectViewer` |
| **Permission** | Unidade atômica (`service.resource.verb`) | `storage.objects.get` |
| **Resource** | Qualquer entidade GCP organizada em hierarquia | Organization, Folder, Project, Bucket, GCE instance |
| **IAM Policy** | Coleção de **bindings** `{role, members, condition}` anexada a resource | ver §2.3 |

### 2.2 Resource Hierarchy

```
Organization (company.com)
   └── Folder (engineering)
          └── Folder (backend)
                 └── Project (prod-api)
                        └── Resource (bucket-logs)
```

Policies **herdam top-down**: binding em Organization vale para todos os Projects abaixo. **Policy de recurso = união** de todas as policies herdadas.

**Não há deny por herança** (até 2023; hoje existem Deny Policies opcionais). Composição é aditiva por padrão.

### 2.3 IAM Binding com Condition

```yaml
bindings:
- role: roles/storage.objectViewer
  members:
  - user:alice@company.com
  - group:analysts@company.com
  - serviceAccount:etl@prod-api.iam.gserviceaccount.com
  condition:
    title: "Apenas buckets de prod, em horário comercial BR"
    expression: |
      resource.name.startsWith("projects/_/buckets/prod-") &&
      request.time.getHours("America/Sao_Paulo") >= 8 &&
      request.time.getHours("America/Sao_Paulo") <= 18
```

### 2.4 Tipos de Roles

| Tipo | Descrição | Exemplo | Quando usar |
|---|---|---|---|
| **Basic** | Owner/Editor/Viewer — coarse, legado | `roles/editor` | Quase nunca. Evitar. |
| **Predefined** | Curadas pelo GCP por serviço | `roles/storage.objectAdmin` | Preferência padrão. |
| **Custom** | Criadas pelo cliente com subset de permissions | `projects/x/roles/customDataViewer` | Quando predefined não bate exato. |

GCP tem **>10.000 permissions atômicas**; custom roles permitem granularidade máxima sem escrever code.

### 2.5 CEL — Common Expression Language

CEL é linguagem declarativa embutida em policies GCP (e também Firestore rules, Cloud Armor, etc.):

```cel
// Exemplo 1: resource name
resource.name.startsWith("projects/prod-")

// Exemplo 2: tempo
request.time < timestamp("2026-06-01T00:00:00Z")

// Exemplo 3: claim do usuário (OIDC)
"admin" in request.auth.claims.groups

// Exemplo 4: IP origem
inIpRange(origin.ip, "10.0.0.0/8")

// Exemplo 5: tag do resource
has(resource.labels.env) && resource.labels.env == "staging"

// Exemplo 6: combinado
resource.type == "storage.googleapis.com/Bucket" &&
resource.name.startsWith("projects/_/buckets/public-") &&
request.time.getDayOfWeek() >= 1 && request.time.getDayOfWeek() <= 5
```

**Atributos disponíveis**:
- `request.*`: time, auth (claims), path, method
- `resource.*`: name, type, service, labels (tags)
- `origin.*`: ip, region

### 2.6 Policy Intersection

Quando há Org Policies + IAM Policies + VPC Service Controls, o acesso efetivo é **interseção**:

```
Acesso concedido ⇔
   IAM Allow (via binding, aplicando Condition)
   ∧ NÃO bloqueado por Deny Policy
   ∧ NÃO bloqueado por Org Policy constraint
   ∧ NÃO bloqueado por VPC Service Controls perimeter
```

Isto permite **layered defense**: security team define perímetro top-down sem mexer em projects.

### 2.7 Workload Identity Federation

Alternativa a "service account key.json" (credencial estática arriscada):

```
┌─────────────────┐
│ GitHub Actions  │  OIDC token com claims
│ (external IdP)  │  (repo, ref, workflow)
└────────┬────────┘
         │ token
         ▼
┌─────────────────┐
│ GCP STS         │  valida token contra Workload
│ (token broker)  │  Identity Pool + Provider config
└────────┬────────┘
         │ access token curto (1h)
         ▼
┌─────────────────┐
│ GCP APIs        │
└─────────────────┘
```

**Sem key.json armazenada**. Federação com GitHub, Okta, AWS, qualquer OIDC-compatible.

### 2.8 Deny Policies (adicional recente)

Antes de 2022, IAM GCP era **só aditivo**. Em 2023+, surgiram **Deny Policies** para expressar exceções:

```yaml
# Deny Policy anexada a folder "finance"
name: deny-external-reads-on-payroll
rules:
  - deniedPrincipals:
      - "principalSet://iam.googleapis.com/projects/-/locations/global/workloadIdentityPools/external-*"
    deniedPermissions:
      - "storage.objects.get"
      - "storage.objects.list"
    resource:
      startsWith: "projects/_/buckets/payroll-"
    condition:
      expression: |
        !("trustedPartner" in request.auth.claims.groups)
```

Deny vence Allow; permite padrões tipo "todos podem ler exceto X".

### 2.9 Troubleshooting: Policy Troubleshooter

GCP tem ferramenta oficial que responde "por que este request foi negado / aprovado?":

```bash
gcloud policy-troubleshoot iam \
  alice@company.com \
  --permission=storage.objects.get \
  --full-resource-name=//storage.googleapis.com/projects/_/buckets/my-bucket/objects/report.pdf
```

Retorna bindings que contribuíram para decisão, conditions avaliadas, herança de hierarchy. Equivalente AWS: IAM Access Analyzer + Policy Simulator.

### 2.10 Lições GCP

- **Hierarquia nativa**: Organization → Folder → Project → Resource com herança clara.
- **CEL** é a ferramenta de condition mais expressiva do mercado entre clouds majors.
- **Permissions atômicas finas** (`storage.objects.get`) compostas em Roles reutilizáveis.
- **Custom Roles** evitam role explosion: cliente compõe o subset exato.
- **Workload Identity Federation** abolindo long-lived keys é padrão 2024+.
- **Deny Policies** (recentes) permitem override negativo em hierarquia — antes só aditiva.

### 2.11 Referências oficiais GCP

- IAM Overview: https://cloud.google.com/iam/docs/overview
- IAM Conditions: https://cloud.google.com/iam/docs/conditions-overview
- CEL Spec: https://github.com/google/cel-spec
- Resource Hierarchy: https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy
- Workload Identity Federation: https://cloud.google.com/iam/docs/workload-identity-federation

---

## 3. Cloudflare Account/Zone IAM

### 3.1 Primitivos

Cloudflare tem a separação **mais didática** dos 5 — ideal para inspiração:

| Primitivo | Descrição | Exemplo |
|---|---|---|
| **Member** | Usuário físico dentro de uma conta | `alice@company.com` |
| **Group** | Coleção nomeada de Members (humanos) | `SRE-Brasil` |
| **Permission Group** | **Capacidade técnica** canned; agrupa permissions atômicas | `Administrator`, `DNS Read`, `Workers Edit` |
| **Custom Role** | Composição de Permission Groups + Resources (nome próprio) | `role-dns-editor-br` |
| **Policy** | Binding `(groups|members) × (permission_groups) × resources × effect` | ver §3.3 |
| **Resource Group** | Coleção nomeada de resources alvo (zones, workers, etc.) | `RG-br-zones` |

### 3.2 Member vs Group vs Permission Group

**A distinção-chave que o painel Cloudflare ensina**:

- `Group` (também chamado "User Group") = **grupo de pessoas**. Ex.: `Equipe DNS Brasil`.
- `Permission Group` = **capacidade técnica canned**. Ex.: `DNS Administrator` (agrupa ~30 permissions atômicas DNS).

Um Member pode ser atribuído a múltiplos Groups; um Group pode ter múltiplos Permission Groups via Custom Roles ou Policies.

**N:M em tudo**: Member N:M Group, Group N:M Permission Group, Permission Group N:M Resource.

### 3.3 Policy shape

```json
{
  "id": "policy-abc-123",
  "permission_groups": [
    { "id": "pg-dns-read",  "name": "DNS Read" },
    { "id": "pg-dns-write", "name": "DNS Write" }
  ],
  "resources": {
    "com.cloudflare.api.account.zone.0123abc...": "*"
  },
  "access": "allow"
}
```

Elementos:
- `permission_groups`: lista de capacidades técnicas.
- `resources`: mapa ARN-like → ação (`*` = todas). Pode ser **scoped** a um zone, um account inteiro, ou `*` global.
- `access`: `allow` ou `deny` (deny vence).

### 3.4 Custom Roles

Quando canned Permission Groups não bastam, cliente combina em **Custom Role**:

```
Custom Role "DNS Ops BR"
  = [Permission Group DNS Read]
  + [Permission Group DNS Write]
  + [Permission Group Analytics Read]
  scoped to Resource Group "BR Zones"
```

Reusável, nomeável, auditável. Troca "papel humano" (`syndico`) por "capacidade composta" (`dns-ops-br`).

### 3.5 API endpoints

```
GET  /accounts/{id}/iam/permission_groups
GET  /accounts/{id}/iam/roles
POST /accounts/{id}/iam/policies
GET  /accounts/{id}/iam/resource_groups
POST /accounts/{id}/members
POST /accounts/{id}/members/{member_id}/roles
```

API pública e versionada — painel admin é **thin layer** sobre essa API.

### 3.6 Cloudflare Access (Zero Trust adicional)

Camada **ortogonal** ao IAM da conta: Cloudflare Access gate aplicações (SaaS, self-hosted, SSH, RDP) com:

```yaml
# Cloudflare Access Application
name: internal-admin
domain: admin.company.com
session_duration: 8h
policies:
  - name: allow-br-mfa
    decision: allow
    include:
      - group: "sre-brasil"
    require:
      - geo: ["BR"]
      - mfa: { provider: "any" }
  - name: deny-high-risk
    decision: deny
    include:
      - { everyone: true }
    require:
      - ip_list: "tor-exit-nodes"
```

Elementos:
- **Include rules** (OR): condições para qualificar.
- **Exclude rules** (NOT): bloqueios.
- **Require rules** (AND): obrigatórias em cima do include.
- **Identity providers**: OIDC, SAML, GitHub, Google, Okta, etc.

**Service tokens** para M2M; **purpose justification** para audit; **session policies** para timeout e re-auth.

### 3.7 Playbook: painel admin self-service (inspirado em Cloudflare)

O painel Cloudflare expõe uma sequência linear que é referência de UX:

1. **Members** tab: lista pessoas + invite flow (email → accept → MFA enforcement).
2. **Groups** tab: create group → add members (com search + bulk).
3. **Roles** tab:
   - **Built-in** (read-only, canned por Cloudflare).
   - **Custom**: nome + descrição + seleção de Permission Groups (checkbox com busca).
4. **Policies** tab: `(Group | Member) × Role × Resources × Effect`.
5. **Audit Logs** tab: todo change listado com actor, timestamp, diff.

Cada aba é uma CRUD API — admin experimenta sem engenharia envolvida. O painel é **thin UI**; a verdade está na API.

**Lições de UX desse flow**:
- Search-first em seleção de permission groups (100+ items).
- Preview do "effective permissions" antes de salvar.
- Dry-run: "se eu aplicar esta policy, quem afetaria?" listando members impactados.
- Undo imediato via audit log.

### 3.8 Lições Cloudflare

- **Clarifica semântica** que AWS/GCP confundem: **capacidade técnica ≠ papel humano**.
- **Painel admin** é apenas UI sobre API REST — self-service first-class.
- **Canned Permission Groups** (~50-100) cobrem 90% dos casos; Custom Roles para os 10%.
- **Resource scoping** por zone/account/wildcard.
- **Policies com effect** (allow/deny) seguindo AWS-style.
- **Zero Trust (Access)** é camada separada mas complementar (não confundir).

### 3.9 Referências oficiais Cloudflare

- IAM API: https://developers.cloudflare.com/fundamentals/api/reference/permissions/
- Members & Roles: https://developers.cloudflare.com/fundamentals/setup/manage-members/
- Permission Policies: https://developers.cloudflare.com/fundamentals/setup/manage-members/policies/
- Cloudflare Access: https://developers.cloudflare.com/cloudflare-one/policies/access/

---

## 4. Azure AD / Microsoft Entra ID

### 4.1 Primitivos

| Primitivo | Descrição | Exemplo |
|---|---|---|
| **User** | Identidade humana (member ou guest) | `alice@tenant.onmicrosoft.com` |
| **Group** | Coleção — Security, Microsoft 365, Mail-enabled, Distribution | `g-sre-br` |
| **Service Principal** | Identidade para aplicação/workload | App registration |
| **Managed Identity** | Service Principal gerenciado pelo Azure (sem secret manual) | System/User-assigned MI |
| **Directory Role** | Roles built-in do tenant Entra | `Global Administrator`, `User Administrator` |
| **Azure RBAC Role** | Roles sobre resources Azure (plano de controle) | `Contributor`, `Reader`, Custom |
| **Role Assignment** | Binding `(principal) × (role) × (scope)` | — |
| **Scope** | Nível hierárquico onde assignment aplica | MG → Sub → RG → Resource |

### 4.2 Scope hierarchy

```
Management Group
   └── Subscription
          └── Resource Group
                 └── Resource
```

Assignment em nível superior **herda** para todos abaixo. Role definition tem `assignableScopes` limitando onde pode ser atribuída.

### 4.3 Custom Role definition

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Pode iniciar/parar VMs, mas não criar nem deletar.",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [
    "Microsoft.Compute/virtualMachines/delete"
  ],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/11111111-1111-1111-1111-111111111111",
    "/subscriptions/22222222-2222-2222-2222-222222222222/resourceGroups/prod"
  ]
}
```

Quatro listas de actions:
- `Actions`: control plane (operações sobre resource).
- `NotActions`: subtraído de Actions (exclusão fina).
- `DataActions`: data plane (ler/escrever dados dentro do resource — ex.: blob em storage).
- `NotDataActions`: subtraído de DataActions.

### 4.4 PIM — Privileged Identity Management

**Just-in-Time elevation** é o diferencial Azure:

```
┌──────────┐   1. alice é "eligible" para       ┌──────────┐
│  alice   │      role X (não assignment ativa) │  Entra   │
└────┬─────┘                                    │  PIM     │
     │                                          └────┬─────┘
     │ 2. request activation (reason, duration)      │
     │───────────────────────────────────────────────▶│
     │                                                │
     │ 3. approval required? → sends to approver     │
     │                                                │
     │ 4. MFA challenge                              │
     │                                                │
     │ 5. role ativa por N horas (auditado)          │
     │◀───────────────────────────────────────────────│
     │
     │ 6. role desativa automaticamente expirado
     ▼
```

Benefícios:
- Acesso privilegiado **zero em repouso** (ninguém é Global Admin permanente).
- Audit log completo: quem ativou, quando, motivo, aprovador.
- Approval workflow parametrizável.
- Time-bound (1h, 4h, 8h max).
- MFA obrigatória na elevação.

### 4.5 Conditional Access

Policies declarativas que avaliam **sinais** no momento do sign-in:

```
SE (user está em group X) E
   (app = "SAP")         E
   (location ∉ trusted)  E
   (device ∉ compliant)  E
   (sign-in risk ≥ medium)
ENTÃO require MFA + compliant device
      OU block
```

Sinais disponíveis:
- **Identity**: user, group, role, guest vs member.
- **Cloud app**: qual aplicação está sendo acessada.
- **Location**: IP ranges, países, trusted networks.
- **Device state**: compliant, hybrid-joined, enrolled.
- **Sign-in risk**: Microsoft Defender AI score (low/medium/high).
- **User risk**: comportamento histórico.
- **Client app**: browser, modern auth client, legacy.

Controles aplicáveis:
- **Grant**: require MFA, require compliant device, require terms, require app protection.
- **Session**: limit app session, use persistent browser, use token protection.

### 4.6 Dynamic Groups

Regras em sintaxe declarativa atribuem membros automaticamente:

```
(user.department -eq "Engineering") -and
(user.country -eq "Brazil") -and
(user.accountEnabled -eq true)
```

Novo user com esses atributos entra no group sem intervenção manual; se atributo muda, entra/sai automaticamente.

### 4.7 Conditional Access — exemplos completos

**Policy 1: MFA obrigatória para admins**

```yaml
name: "Admins-require-MFA"
state: enabled
conditions:
  users:
    include_roles:
      - Global Administrator
      - Privileged Role Administrator
      - Security Administrator
  applications:
    include: ["All cloud apps"]
  locations:
    include: ["Any"]
grant_controls:
  operator: OR
  built_in_controls:
    - require_mfa
```

**Policy 2: block legacy auth**

```yaml
name: "Block-legacy-auth"
state: enabled
conditions:
  users:
    include: ["All users"]
    exclude_groups: ["service-accounts-exempt"]
  client_app_types:
    - exchange_active_sync
    - other_clients
grant_controls:
  operator: OR
  built_in_controls:
    - block
```

**Policy 3: compliant device para acesso a SharePoint sensível**

```yaml
name: "SharePoint-require-compliant-device"
state: enabled
conditions:
  users:
    include_groups: ["finance", "legal"]
  applications:
    include_apps: ["Office 365 SharePoint Online"]
  client_app_types: ["browser", "mobileAppsAndDesktopClients"]
grant_controls:
  operator: AND
  built_in_controls:
    - require_mfa
    - require_compliant_device
session_controls:
  sign_in_frequency: { value: 1, type: hours }
  persistent_browser: never
```

### 4.8 Lições Azure / Entra

- **Scope hierarchy** é força principal (igual GCP): MG → Sub → RG → Resource.
- **Actions / NotActions / DataActions / NotDataActions** separam control plane de data plane (AWS não tem essa distinção explícita).
- **PIM** é gold standard de JIT elevation — mais maduro que equivalentes AWS/GCP.
- **Conditional Access** é o motor ABAC da Microsoft; baseado em **sinais** não só em atributos.
- **Dynamic Groups** atribuem membership por regra, evitando intervenção manual em joiner/mover.

### 4.9 Referências oficiais Azure

- Azure RBAC: https://learn.microsoft.com/en-us/azure/role-based-access-control/overview
- Custom Roles: https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles
- Entra PIM: https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure
- Conditional Access: https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview
- Dynamic Groups: https://learn.microsoft.com/en-us/entra/identity/users/groups-dynamic-membership

---

## 5. Okta

### 5.1 Primitivos

| Primitivo | Descrição | Exemplo |
|---|---|---|
| **User** | Identidade humana no Universal Directory | `alice@company.com` |
| **Group** | Coleção de Users (static ou dynamic via rule) | `sre-brasil` |
| **App** | Aplicação integrada (SAML, OIDC, SWA) | `Slack`, `GitHub`, `Salesforce` |
| **App Assignment** | `(user|group) → app [+ profile attrs]` | Alice → Slack com role `admin` |
| **Authorization Server** | OIDC/OAuth server customizável com scopes/claims | `default` ou custom |
| **Policy (Authn/Sign-on/Password)** | Regras para login, MFA, sessão | ver §5.4 |
| **Network Zone** | Definição de IP ranges para allow/deny | `Office-BR`, `BlockList` |

### 5.2 Universal Directory

Schema do user é **extensível**:

```json
{
  "profile": {
    "firstName": "Alice",
    "lastName": "Silva",
    "email": "alice@company.com",
    "countryCode": "BR",
    "department": "Engineering",
    "employeeNumber": "E1234",
    "customAttributes": {
      "costCenter": "CC-BR-001",
      "clearanceLevel": "confidential",
      "ssh_public_key": "ssh-ed25519 AAAA..."
    }
  }
}
```

Atributos customizados alimentam **Group Rules**, **claims em tokens**, **App profiles**, etc.

### 5.3 Group Membership Rules (Dynamic Groups)

Regras em Okta Expression Language:

```
// Group "Engenharia Brasil"
user.department == "Engineering" AND user.countryCode == "BR"

// Group "Tier-1 Support"
Arrays.contains({"support", "customer-success"}, user.department) AND
user.employeeType != "contractor"

// Group "Elevated Access Eligible"
user.clearanceLevel == "confidential" AND
user.mfaEnrolled == true
```

Quando user é criado ou atributo muda, Okta reavalia e ajusta membership.

### 5.4 Policies — 3 tipos

Okta separa em **3 domínios de policy**:

| Policy | Quando avalia | Efeito |
|---|---|---|
| **Password Policy** | Enrollment / change password | Complexidade, histórico, expiração |
| **Sign-on Policy** | Autenticação (Okta global) | Require MFA, session timeout, IdP selection |
| **Authentication Policy** (per app) | Acesso a uma app específica | Fatores específicos, frequência de re-auth |

Exemplo Sign-on Policy:

```yaml
name: "Require MFA for externals"
if:
  groups: ["external-contractors"]
  network_zone: NOT "trusted-office"
then:
  primary_factor: password
  additional_factor:
    required: true
    providers: [okta_verify, webauthn]
  session:
    max_duration: 4h
    idle_timeout: 15m
```

### 5.5 Authorization Servers + Custom Claims

Okta permite criar **OIDC/OAuth Authorization Servers** customizados com:

- **Scopes** próprios: `read:invoices`, `write:settings`.
- **Claims** em tokens ID/Access:

```javascript
// Claim "roles" no access_token
user.groups.filter(
  g => g.name.startsWith("role-")
).map(g => g.name.replace("role-", ""))

// Claim "tenant_id"
user.tenantId

// Claim "permissions" baseado em grupos
user.groups.contains("syndics-group")
  ? ["condo:read", "condo:write", "membership:manage"]
  : ["condo:read"]
```

Isto permite que **resource servers** (APIs downstream) façam authz **só olhando o token**, sem callback extra ao IdP.

### 5.6 Network Zones

```yaml
network_zones:
  - name: "Office-SP"
    type: IP
    gateways:
      - "203.0.113.0/24"
      - "198.51.100.0/24"
    proxies: []

  - name: "BlockList-HighRisk"
    type: IP
    gateways:
      - "0.0.0.0/0"
    exclude:
      - "trusted-partners"
    asn:
      blocklist: [12345, 67890]

  - name: "Dynamic-Tor"
    type: DYNAMIC
    locations: []
    proxy_type: TorAnonymizer
```

Zones alimentam Sign-on Policies (allow/deny/require MFA se zona X).

### 5.7 Behavior Detection + Risk-based auth

Okta ThreatInsight + Behavior Detection:

- **New Device**: primeira vez naquele browser/OS.
- **New Geolocation**: país/cidade inédita.
- **New IP**: fora do histórico conhecido.
- **Velocity**: dois logins fisicamente impossíveis (SP → Tokyo em 10min).
- **Risk score** combinado → alimenta policy.

Policy example: "Se behavior `new_device AND new_geolocation` → require step-up MFA".

### 5.8 Factors MFA suportados

Okta centraliza factors em um único registro por user; policies referenciam qual factor atende:

| Factor | Força | Phishing-resistant | Uso |
|---|---|---|---|
| Password | Fraca | Não | Legacy; sempre step-up |
| SMS OTP | Média | Não (SIM-swap) | Desencorajado em enterprise |
| Voice OTP | Média | Não | Fallback |
| Email OTP | Média | Não | Recovery |
| Okta Verify Push | Alta | Parcial (MFA-fatigue) | Default workforce |
| Okta Verify TOTP | Alta | Não | Offline |
| WebAuthn / FIDO2 | Muito alta | **Sim** | Recomendado para privileged |
| Smart Card (PIV/CAC) | Muito alta | Sim | Governo / regulado |
| Biometria | Alta | Parcial | Via device |
| Number Challenge | Alta | Sim (anti-fatigue) | Modernização 2023+ |

### 5.9 Exemplo completo de Authorization Server customizado

```javascript
// Authorization Server: "condo-api"
//
// Scopes
{
  scopes: [
    { name: "condo:read",       description: "Ler dados do condomínio" },
    { name: "condo:write",      description: "Editar dados do condomínio" },
    { name: "membership:read",  description: "Listar memberships" },
    { name: "membership:write", description: "Criar/editar memberships" },
    { name: "finance:read",     description: "Ler relatórios financeiros" },
    { name: "finance:write",    description: "Editar dados financeiros" },
  ]
}

// Claim "permissions" no access_token (via expression)
user.groups
  .filter(g => g.profile.app === "condo-api")
  .flatMap(g => g.profile.scopes)
  .distinct()

// Claim "tenant_ids" (multi-tenant)
user.memberships.map(m => m.tenantId)

// Claim "primary_tenant" (tenant ativo na sessão)
session.attributes.activeTenantId

// Access Policy exigindo MFA para scope sensível
{
  name: "finance-requires-mfa",
  conditions: {
    scopes: ["finance:write"],
    clients: ["condo-admin-panel"]
  },
  rules: [{
    people: { groups: { include: ["finance-officers"] } },
    actions: {
      access_token_lifetime: "1h",
      require_mfa_on_issue: true
    }
  }]
}
```

### 5.10 Lições Okta

- **Schema extensível no Universal Directory** vira substrato para **dynamic groups**, **claims**, **policies**.
- **Dynamic Groups com regras declarativas** eliminam trabalho manual joiner/mover.
- **Authorization Servers customizáveis** desacoplam app de IAM core — claims viajam no token.
- **Policies em 3 níveis** (password/sign-on/app) separam responsabilidades de cada domínio.
- **Network Zones + Behavior + Risk** = motor ABAC contextual forte.
- **API-first**: toda feature via REST → automação e painel self-service viáveis.

### 5.11 Referências oficiais Okta

- Universal Directory: https://developer.okta.com/docs/concepts/universal-directory/
- Group Rules: https://help.okta.com/en-us/content/topics/users-groups-profiles/usgp-about-group-rules.htm
- Sign-on Policies: https://help.okta.com/en-us/content/topics/security/policies/about-policies.htm
- Authorization Servers: https://developer.okta.com/docs/concepts/auth-servers/
- Network Zones: https://help.okta.com/en-us/content/topics/security/network/network-zones.htm
- Behavior Detection: https://help.okta.com/en-us/content/topics/security/behavior-detection.htm

---

## 6. Patterns transversais aplicáveis

Sete patterns emergem da intersecção dos cinco modelos.

### Pattern 1 — Permission Groups desacoplados de Papéis Humanos

**Observado em**: Cloudflare (explícito), AWS (via IAM Policy vs IAM Group), Azure (Role vs Security Group), Okta (App Assignment vs Group).

**Princípio**:

```
┌─────────────┐     N:M      ┌──────────────────┐
│   Group     │◀────────────▶│ Permission Group │
│ (pessoas)   │              │ (capacidades)    │
└─────────────┘              └──────────────────┘
      ▲                               ▲
      │ N                             │ N
      │                               │
┌─────────────┐              ┌──────────────────┐
│   User      │              │   Permission     │
└─────────────┘              │   (atômica)      │
                             └──────────────────┘
```

- **Group**: coleção de pessoas gerida por owner de domínio (ex.: gerente RH).
- **Permission Group**: coleção de permissions atômicas gerida por owner técnico.
- **Binding**: painel admin compõe `Group × Permission Group × Scope`.

**Benefício**: reorganização humana não mexe em capacidades técnicas; reorg técnica não mexe em composição humana.

**Pitfall**: confundir os dois (nomear um group `admin` quando quer dizer "permission group admin") volta ao problema.

### Pattern 2 — Resource Scoping com Hierarquia

**Observado em**: GCP (Org/Folder/Project/Resource), Azure (MG/Sub/RG/Resource), AWS Organizations (Root/OU/Account), Cloudflare (Account/Zone/Resource Group).

**Princípio**: binding em nível alto **herda** para descendentes; binding em nível baixo **override** ou soma.

**Multi-tenant SaaS mapping típico**:

```
Tenant Root
   └── Tenant (condomínio)
          └── Sub-resource (unidade, membership, invoice)
```

Binding em `Tenant` alcança todas as unidades; binding em unidade específica **apenas aquela** unidade.

**Benefício**:
- Permissão N:M sem explosão (atribua no nível alto, recursos novos herdam).
- Reflete mental model de usuário (org → sub-unidades).

**Pitfall**: herança silenciosa pode surpreender — UI deve mostrar **effective permissions** não apenas assignments.

### Pattern 3 — Conditional Access via expressão declarativa

**Observado em**: GCP (CEL), AWS (`Condition` JSON), Azure (Conditional Access), Okta (Expression Language).

**Atributos comuns em condition**:

| Dimensão | Exemplo |
|---|---|
| Tempo | `request.time.getHours() >= 8 && <= 18` |
| IP / Rede | `inIpRange(origin.ip, "10.0.0.0/8")` |
| MFA | `request.auth.claims.mfa == true` |
| Device | `device.compliant == true` |
| Resource tag | `resource.tags.env == principal.tags.env` |
| Risk score | `signin.risk < "medium"` |
| Geolocation | `origin.country == "BR"` |

**Princípio**: em vez de roles estáticas ("admin-office-hours"), uma role + condition dinâmica expressa em DSL.

**Benefício**: combinatória enxuta; regras versionáveis em policy, não espalhadas em código.

**Pitfall**: DSLs complexas viram ilegíveis; manter vocabulário restrito e validável (CEL tem type system — boa referência).

### Pattern 4 — Default Roles por Group

**Observado em**: Azure AD, Okta (via app assignment no Group), AWS IAM Groups, Cloudflare Groups.

**Princípio**: ao criar um Group, admin associa **um ou mais Permission Groups padrão**. Novos members herdam automaticamente.

```
Create Group "SRE-Brasil"
  Default Permission Groups:
    - Read (all resources)
    - Edit (DNS + Workers)
    - Admin (no)

Assign Alice → SRE-Brasil
  → Alice inherits Read + Edit(DNS, Workers)
```

**Benefício**: onboarding sem toque individual; consistência entre membros do mesmo group.

**Pitfall**: member precisa ocasionalmente de role extra específica (exception). Modelar: `group defaults + per-member overrides` (Cloudflare faz bem).

### Pattern 5 — Just-in-Time Elevation (PIM)

**Observado em**: Azure PIM (maduro), AWS (via STS AssumeRole + MFA), Okta Privileged Access, Cloudflare (service tokens com TTL).

**Princípio**: capacidade privilegiada (ex.: `admin`) é **eligible** em repouso; user ativa sob demanda, com:

1. **Justificativa** (texto livre, ticket JIRA).
2. **MFA challenge** no momento da elevação.
3. **Approval workflow** opcional (aprovação por par ou manager).
4. **Time-bound** (1h, 4h, 8h max).
5. **Audit log** imutável da ativação.

**Arquitetura típica**:

```
eligible_roles(user) = [adminX, adminY]
active_roles(user)   = []  (em repouso)

POST /pim/activate
  body: { role: adminX, reason: "JIRA-1234", duration: 4h }
  → MFA challenge
  → (opcional) approval notification
  → inject claim "pim:adminX" no token, exp=+4h
  → log "user=alice activated=adminX reason=... approver=..."

após 4h, claim expira; token rotation volta a state eligible.
```

**Benefício**: dramática redução do blast radius de credencial vazada; surface de ataque reduzida.

**Pitfall**: UX ruim faz user odiar — precisa ser **rápido** (5-10s elevação) senão contorna.

### Pattern 6 — Tag-based ABAC

**Observado em**: AWS (recomendação oficial 2020+), GCP (via labels), Azure (tags).

**Princípio**: em vez de criar role para cada combinação `(projeto × ambiente × time)`, **tagueia** principal e resource; policy única compara tags.

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::*/*",
  "Condition": {
    "StringEquals": {
      "aws:ResourceTag/project": "${aws:PrincipalTag/project}",
      "aws:ResourceTag/env":     "${aws:PrincipalTag/env}"
    }
  }
}
```

**Quando vale**:
- Alto número de projetos/tenants.
- Principals e resources bem tagueados no onboarding.
- Esquema de tags estável (governance).

**Quando NÃO vale**:
- Poucos projetos / baixa variação.
- Tagging inconsistente ou tagging drift.
- Lógica de permissão complexa além de match de tags.

**Pitfall**: tagging drift (recurso novo sem tag) → policy falha silenciosamente. Necessário policy de governança: "todo recurso sem tag `project` → deny".

### Pattern 7 — Group Membership Rules dinâmicas

**Observado em**: Okta (Group Rules), Azure AD Dynamic Groups, GCP (via IdP attributes).

**Princípio**: group membership é **derivada** de atributos do user, não atribuída manualmente.

```
// Regra Okta
user.department == "Engineering" AND
user.countryCode == "BR" AND
user.employmentStatus == "active"
→ add to group "eng-br-active"
```

Quando atributo muda (user vira contractor → `employmentStatus = "contractor"`), group remove automaticamente.

**Benefício**:
- Joiner: novo funcionário atribuído corretamente na hora que HRIS sync roda.
- Mover: promoção/transferência ajusta membership sem manual.
- Leaver: desativação dispara remoção de groups sensíveis.

**Pitfall**:
- Atributo errado na fonte (HRIS) propaga para permissions.
- Latência de sync (HRIS → IdP → groups → downstream apps) pode ser minutos a horas.
- Necessário process para **emergência**: override manual em caso de incident.

---

## 7. Anti-patterns

Catálogo observado em sistemas legados + literatura AWS/GCP/Microsoft.

### Anti-pattern 1 — Role Explosion

**Sintoma**: catálogo de roles cresce linearmente com combinações `(persona × escopo × capacidade × ambiente)`. Chega-se a centenas de roles nomeadas tipo `admin-sp-dns-prod`, `editor-rj-workers-staging`.

**Causa raiz**: tratar role como "papel humano concreto em contexto específico" em vez de "composição de capacidades técnicas".

**Remediação**:
- Separar Group (pessoa/agrupamento) de Permission Group (capacidade técnica) — Pattern 1.
- Usar Resource Scoping em vez de embutir escopo no nome — Pattern 2.
- Usar Tag-based ABAC para ambiente/projeto — Pattern 6.

### Anti-pattern 2 — Permission baked into code

**Sintoma**:

```typescript
if (user.role === "syndic" || user.role === "admin") {
  allowAction();
}
```

Regras de autorização **espalhadas em if/else** pelo código.

**Causa raiz**: ausência de camada de policy externalizada.

**Problemas**:
- Mudança de regra exige deploy.
- Sem audit: impossível rastrear quem pode fazer o quê.
- Inconsistência: a mesma regra escrita em 3 places diferentes com lógicas sutilmente distintas.
- Testabilidade ruim (mocks espalhados).

**Remediação**:
- Policy externa (OPA/Rego, Cedar, casbin, ou tabela em DB).
- Middleware/decorator único que avalia policy.
- Policies versionadas e testáveis offline.

### Anti-pattern 3 — God Role

**Sintoma**: existe role `super-admin` / `root` / `master` que **tudo** pode, atribuída a várias pessoas sem segregação.

**Causa raiz**: facilidade de bootstrap ("cria uma role que faz tudo e segue") sem refino posterior.

**Problemas**:
- Uma credencial comprometida = game over.
- Impossível rastrear responsável por mudança em audit (todo mundo é root).
- Separation of Duties violada (mesma pessoa aprova e executa).

**Remediação**:
- Segregação em múltiplas roles (security admin ≠ billing admin ≠ user admin).
- PIM para elevação temporária — Pattern 5.
- Aprovação multi-pessoa para ações destrutivas.
- Break-glass accounts (acesso emergencial com audit especial).

### Anti-pattern 4 — Static-only Assignment

**Sintoma**: todos os privilégios são **permanentes**. Ninguém tem role temporária; todo mundo tem todas as roles que já precisou.

**Causa raiz**: falta de primitiva de elevação temporal.

**Problemas**:
- Access creep (mover sem limpeza).
- Blast radius máximo permanente.
- Compliance aponta em access reviews.

**Remediação**:
- PIM / JIT elevation — Pattern 5.
- Access reviews periódicas (trimestral) com revogação por default se não justificado.
- Role-eligible vs role-active distinção.

### Anti-pattern 5 — No Audit Trail em Policy Change

**Sintoma**: admin altera permissions no painel; mudança aplicada sem log imutável de quem fez, quando, diff do before/after.

**Causa raiz**: audit tratado como afterthought.

**Problemas**:
- Incidente: impossível entender quando permission mudou.
- Compliance: SOC2 exige evidência.
- Insider threat: admin mal-intencionado apaga rastros.

**Remediação**:
- Policy changes → event imutável em log append-only (ex.: CloudTrail, Cloud Audit Logs, Okta System Log).
- Diff do before/after (JSON patch).
- Separação de audit log do DB operacional (segurança diferente).
- Alerts em mudanças sensíveis (ex.: adicionar member a `admin-global`).

### Anti-pattern 6 — Fusion de Authn e Authz

**Sintoma**: identidade e autorização misturadas em uma única tabela `users` com coluna `role`.

**Causa raiz**: evolução orgânica sem refatoração.

**Problemas**:
- Trocar IdP (próprio → Zitadel/Auth0) exige migrar authz junto.
- Authz não escala para multi-tenant (qual role em qual tenant?).
- Claims em JWT ficam acoplados à forma de autenticação.

**Remediação**:
- IdP externo cuida de authn (user identity + MFA + sessions).
- Authz em tabela própria `memberships` ou serviço PDP (Policy Decision Point).
- Token IdP só carrega identidade; authz é consultado separadamente ou via claims opcionais.

### Anti-pattern 7 — Overly Complex DSL

**Sintoma**: policy DSL (CEL, Rego, JSON custom) vira ilegível; ninguém além do autor entende.

**Causa raiz**: tentativa de cobrir edge case em policy em vez de design.

**Problemas**:
- Manutenção difícil.
- Review review (segurança) ignora porque não entende.
- Bugs sutis em condições compostas.

**Remediação**:
- Vocabulário restrito (menos atributos, mais semântica).
- Tests unitários para policies.
- Preferir múltiplas policies pequenas a uma mega-policy.
- Linter/visualizador do policy tree.

### Anti-pattern 8 — Missing Deny Mechanism

**Sintoma**: modelo só-aditivo (GCP legado era assim). Impossível expressar "todos os engineers exceto João podem X".

**Causa raiz**: simplificação arquitetural precoce.

**Problemas**:
- Incidente exige revogação individual — implementação vira gambiarra (tirar de group, recriar com exceção).
- Not-suspended user com acesso vasto difícil de limitar granularmente.

**Remediação**:
- Suportar Deny Policies (AWS sempre teve; GCP agora tem; Okta via exclude).
- Deny vence Allow (fail-safe).
- UI mostra effective permissions considerando denies.

### Anti-pattern 9 — Single-tenant assumptions in multi-tenant code

**Sintoma**: código assume user tem **uma** role globalmente. Em multi-tenant, user pode ser admin no tenant A e viewer no tenant B.

**Causa raiz**: modelo mental single-tenant (SaaS small → enterprise).

**Problemas**:
- Tokens JWT com claim `role: "admin"` sem tenant context → vazamento cross-tenant.
- Queries de autorização sem filtro por tenant.

**Remediação**:
- Authz sempre na forma `(user, tenant, action, resource)` — 4-tuple.
- Token carrega `tenant_id` ativo na sessão.
- Middleware injeta tenant context em todo request.
- Membership como primitiva explícita: `user ∈ tenant com role X`.

### Anti-pattern 10 — No Service Account Hygiene

**Sintoma**: workloads (scripts, CI, serviços) rodam com credencial long-lived compartilhada ("a chave da API"); às vezes a mesma credencial humana.

**Causa raiz**: simplicidade no bootstrap.

**Problemas**:
- Credencial vaza em log/git → permanente.
- Audit não distingue human vs machine.
- Revogação atinge todos os users simultaneamente.

**Remediação**:
- Service accounts distintas por workload (Principle of Least Privilege).
- Short-lived tokens (STS, Workload Identity Federation).
- mTLS / SPIFFE para workload identity (ver [[spiffe-spire]]).
- Rotation automática.

---

## 8. Tabela comparativa consolidada

### 8.1 Primitivos

| Conceito | AWS | GCP | Cloudflare | Azure AD | Okta |
|---|---|---|---|---|---|
| Identidade humana | User | Member (user:) | Member | User | User |
| Workload | Role + STS | Service Account | API Token | Service Principal / MI | API Service App |
| Agrupamento | Group | Google Group | Group | Security Group | Group |
| Capacidade canned | Managed Policy | Predefined Role | Permission Group | Built-in Role | Admin Role |
| Capacidade custom | Customer Managed Policy | Custom Role | Custom Role | Custom Role | — |
| Unidade atômica | Action (`s3:GetObject`) | Permission (`storage.objects.get`) | Permission atômica | Action / DataAction | App assignment |
| Alvo (resource) | ARN | Resource name (hierarchy) | Resource (zone/account) | Scope (MG/Sub/RG/Res) | App / Org |
| Binding | Attach policy | Binding (member+role+cond) | Policy | Role Assignment | Assignment |
| Condition | JSON `Condition` | CEL expression | Resource map + effect | Conditional Access | Sign-on / App Policy |
| Deny | `Effect: "Deny"` | Deny Policy (recent) | `access: deny` | `NotActions` / CA block | Exclude rules |
| Guard-rail top | SCP | Org Policy | Account Policy | MG-scoped RBAC | Global admin role |
| JIT Elevation | AssumeRole + MFA | (via external) | Service Token TTL | PIM | Privileged Access |
| Attribute-based | Tag Condition | CEL + labels | Resource Group | Conditional Access | Behavior + Zones |
| Dynamic Groups | — | — (IdP sync) | — | Dynamic Groups | Group Rules |

### 8.2 Força / Fraqueza por modelo

| Modelo | Força principal | Fraqueza principal |
|---|---|---|
| **AWS IAM** | Granularidade extrema + Trust/Permission split + Conditions ricas | Complexidade; JSON verbose; curva de aprendizado alta |
| **GCP IAM** | Hierarquia limpa + CEL expressivo + Workload Identity Federation | Legacy basic roles (Owner/Editor/Viewer) confundem; permissions count explode |
| **Cloudflare** | Semântica clara (Group × Permission Group); API-first | Escopo limitado a produtos Cloudflare; não é IAM para apps terceiros |
| **Azure AD** | PIM maduro + Conditional Access robusto + Dynamic Groups | Dualidade Directory Role × Azure RBAC confunde; docs labirínticas |
| **Okta** | Dynamic groups + schema extensível + Authorization Servers | Authz em tokens (claims) bate limite; precisa combinar com PDP externo para fine-grained |

---

## 9. Diagrama conceitual proposto (síntese)

```
┌─────────────────────────────────────────────────────────────────┐
│                        IAM moderno                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐       N:M       ┌─────────┐                        │
│  │  User   │◀───────────────▶│  Group  │  (pessoas)             │
│  └─────────┘                 └─────────┘                        │
│       │                            │                            │
│       │ attrs                      │ default roles              │
│       ▼                            ▼                            │
│  ┌─────────┐               ┌─────────────────┐                  │
│  │ Profile │──Rule─────▶   │ Permission      │                  │
│  │ attrs   │  (dynamic)    │ Group (canned/  │                  │
│  └─────────┘               │  custom)        │                  │
│                            └─────────────────┘                  │
│                                    │                            │
│                                    │ contains                   │
│                                    ▼                            │
│                            ┌─────────────────┐                  │
│                            │   Permission    │                  │
│                            │   (atômica)     │                  │
│                            │  verb + resource│                  │
│                            └─────────────────┘                  │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Policy / Binding layer                     │    │
│  │                                                         │    │
│  │   (Group | User)  ×  (Permission Group | Custom Role)   │    │
│  │               ×  Resource Scope                         │    │
│  │               ×  Condition (CEL / tags / CA)            │    │
│  │               ×  Effect (allow | deny)                  │    │
│  │               ×  Time-bound? (PIM)                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌───────────────────┐        ┌─────────────────────────────┐   │
│  │ Resource          │        │ Audit Log (imutável)        │   │
│  │   hierarchy       │        │  - policy changes           │   │
│  │   Org → Tenant    │        │  - elevations (PIM)         │   │
│  │        → SubRes   │        │  - access grants/denies     │   │
│  └───────────────────┘        └─────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 9.1 Fluxo de autorização canônico

```
   request(user, action, resource)
          │
          ▼
   ┌────────────────┐
   │ Authn          │  (IdP: OIDC/SAML)  →  token
   │  user verified │
   └────────────────┘
          │
          ▼
   ┌────────────────┐
   │ Resolve        │  user → groups (static + dynamic rules)
   │  memberships   │        → effective permission groups
   │                │        → effective scopes
   └────────────────┘
          │
          ▼
   ┌────────────────┐
   │ Evaluate       │  policy(user, groups, perm_groups, scope,
   │  policy        │         condition, effect)
   │                │  ↳ apply hierarchy inheritance
   │                │  ↳ evaluate conditions (CEL/tags/CA signals)
   │                │  ↳ Deny overrides Allow
   └────────────────┘
          │
          ▼
   ┌────────────────┐
   │ Decision       │  allow | deny | challenge (MFA/approval)
   └────────────────┘
          │
          ▼
   ┌────────────────┐
   │ Audit          │  log (decision + reason + context)
   └────────────────┘
```

---

## 10. Recomendações gerais de design (síntese)

1. **Apenas `admin` primitivo**. O resto: composição `Group × Permission Group × Scope × Condition`.
2. **Permission Groups canned** no bootstrap (~20-50) cobrem 80-90% dos casos; Custom Roles para o resto.
3. **Default Roles por Group** automatiza onboarding; exceções via per-user override.
4. **Resource hierarchy natural** ao domínio (ex.: `Tenant → Sub-resource`).
5. **Tag-based ABAC** quando combinatória de projeto/ambiente explode.
6. **Dynamic Group Rules** reduzem joiner/mover manual.
7. **PIM/JIT** para capacidades destrutivas (backlog se não for M1).
8. **Deny mechanism** desde o dia 1 (mesmo simples).
9. **Audit log imutável** de policy changes + access decisions, separado do DB operacional.
10. **Policy externa em dado**, não em código — permite hot-reload, testes, review.
11. **API-first**: painel admin é cliente de uma API IAM pública; self-service é consequência.
12. **Condition DSL restrita**: CEL-like mas com vocabulário curado (não liberar Turing-complete).

---

## 11. Relações

- **Conceitual (guarda-chuva)**: [[iam]]
- **Modelos de autorização**: [[authorization-models]], [[rbac]], [[abac]], [[rebac]]
- **Autenticação**: [[identity-provider]], [[oidc]], [[oauth-2]], [[saml-2]]
- **Provisionamento**: [[scim-provisioning]]
- **Policy engines**: [[opa-rego]]
- **Delegation**: [[delegation-impersonation]]
- **Workload identity**: [[spiffe-spire]]
- **Zero Trust**: [[beyond-corp]]
- **Princípios**: [[least-privilege]], [[security-principles]]
- **Providers concretos**: [[../providers/cloudflare/access]], [[../providers/zitadel/_moc]]

---

## 12. Fontes

### AWS
- AWS IAM User Guide — https://docs.aws.amazon.com/IAM/latest/UserGuide/ (consultada 2026-04-23)
- Policy Evaluation Logic — https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html (consultada 2026-04-23)
- ABAC Tutorial — https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_attribute-based-access-control.html (consultada 2026-04-23)
- Permissions Boundaries — https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html (consultada 2026-04-23)
- STS AssumeRole — https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html (consultada 2026-04-23)

### GCP
- Cloud IAM Overview — https://cloud.google.com/iam/docs/overview (consultada 2026-04-23)
- IAM Conditions — https://cloud.google.com/iam/docs/conditions-overview (consultada 2026-04-23)
- CEL Spec — https://github.com/google/cel-spec (consultada 2026-04-23)
- Resource Hierarchy — https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy (consultada 2026-04-23)
- Workload Identity Federation — https://cloud.google.com/iam/docs/workload-identity-federation (consultada 2026-04-23)
- Deny Policies — https://cloud.google.com/iam/docs/deny-overview (consultada 2026-04-23)

### Cloudflare
- IAM API Reference — https://developers.cloudflare.com/fundamentals/api/reference/permissions/ (consultada 2026-04-23)
- Members & Roles — https://developers.cloudflare.com/fundamentals/setup/manage-members/ (consultada 2026-04-23)
- Permission Policies — https://developers.cloudflare.com/fundamentals/setup/manage-members/policies/ (consultada 2026-04-23)
- Cloudflare Access — https://developers.cloudflare.com/cloudflare-one/policies/access/ (consultada 2026-04-23)

### Azure AD / Entra ID
- Azure RBAC Overview — https://learn.microsoft.com/en-us/azure/role-based-access-control/overview (consultada 2026-04-23)
- Custom Roles — https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles (consultada 2026-04-23)
- PIM Configure — https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure (consultada 2026-04-23)
- Conditional Access — https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview (consultada 2026-04-23)
- Dynamic Groups — https://learn.microsoft.com/en-us/entra/identity/users/groups-dynamic-membership (consultada 2026-04-23)

### Okta
- Universal Directory — https://developer.okta.com/docs/concepts/universal-directory/ (consultada 2026-04-23)
- Group Rules — https://help.okta.com/en-us/content/topics/users-groups-profiles/usgp-about-group-rules.htm (consultada 2026-04-23)
- Policies — https://help.okta.com/en-us/content/topics/security/policies/about-policies.htm (consultada 2026-04-23)
- Authorization Servers — https://developer.okta.com/docs/concepts/auth-servers/ (consultada 2026-04-23)
- Network Zones — https://help.okta.com/en-us/content/topics/security/network/network-zones.htm (consultada 2026-04-23)
- Behavior Detection — https://help.okta.com/en-us/content/topics/security/behavior-detection.htm (consultada 2026-04-23)

---

## Links

- [[_moc]]
- [[iam]]
- [[authorization-models]]
- [[rbac]]
- [[abac]]
- [[rebac]]
- [[least-privilege]]
- [[security-principles]]
- [[identity-provider]]
- [[scim-provisioning]]
- [[opa-rego]]
- [[beyond-corp]]
