---
title: Deploy Strategy — Cloudflare edge + AWS sa-east-1 origin (D-138 + ADR-0045)
type: guide
tags:
  - execution
  - deploy
  - cloudflare
  - aws
  - workers-vpc
  - ecs-fargate
  - rds
  - master-sindico
  - v2
source: >-
  vault CLAUDE.md raiz §4 Research-Loop + §5 Dual-check (Agente A+B) +
  Cloudflare/AWS docs oficiais consultadas 2026-04-27
created: '2026-04-27T22:00:00.000Z'
updated: '2026-04-27T22:00:00.000Z'
supersedes_partial:
  - 06-execution/deploy.md (Railway-era)
aliases:
  - deploy-strategy-cloudflare-aws
---

# Deploy Strategy — Master Síndico

Estratégia consolidada de deploy aplicando D-138 (Cloudflare-first) + ADR-0045 (Cloudflare-only stack; Railway descontinuado) + D-138-E (AWS sa-east-1 como provider de origin).

**Substitui parcialmente** [[deploy]] (Railway-era; mantido para histórico até consolidação total).

**Construída com Research-Loop §4 + Dual-check §5** do vault [[../../CLAUDE]] raiz. Agente A propôs; Agente B (Plan, contexto limpo) revisou. Discrepâncias resolvidas; pendências marcadas explicitamente.

---

## 0. Modelo mental

```
[Browser/Mobile]
       ↓
[Cloudflare Edge — 330+ PoPs]
   ├─ Pages × 7 (auth, cms, admin, lms, forum, assembly, lp) → mastersindico-<app>
   ├─ Workers BFF (api.mastersindico.com.br)
   │     ↓ env.GO_BACKEND.fetch() via Workers VPC binding (BETA)
   │     ↓ Cloudflare Tunnel QUIC outbound
   ├─ R2 (uploads, exports LGPD, vídeos institucionais)
   ├─ KV (sessions snapshot, config, feature flags)
   ├─ Durable Objects (ABAC determinístico, contadores quota)
   ├─ Queues (jobs assíncronos edge)
   ├─ Cron Triggers (scheduled leves)
   ├─ Access (admin SSO+MFA — admin.mastersindico.com.br)
   ├─ Email Routing (inbound suporte@, billing@, bounces+resend@)
   ├─ Turnstile (CAPTCHA signup/reset/Connect Me)
   └─ DNS + WAF + DDoS + CDN + Universal SSL (free tier ilimitado)

                         (cloudflared sidecar/service)
                                  ↓
[AWS sa-east-1 — VPC privada]
   ├─ ECS Fargate × 2+ tasks: backend Go (Gin + sqlc + worker)
   ├─ RDS PostgreSQL Multi-AZ (RLS + partitioning + atas imutáveis)
   ├─ ElastiCache Redis 7 (ABAC TTL determinístico)
   ├─ OpenSearch Service (search canônico — ADR-0042 Opção A)
   ├─ ECR (registry containers)
   ├─ CloudWatch Logs (origin)
   ├─ Secrets Manager + KMS (criptografia at-rest)
   ├─ IAM (roles ECS task, RDS, OpenSearch — least privilege)
   └─ VPC + 6 subnets (3 AZ × {private, public}) + NAT GW + IGW + SGs

[SaaS externos] — Stripe, Mux, LiveKit, Zitadel (mantidos via interfaces D-139..141), Resend (email outbound), Twilio (SMS), Sentry (postergado M2 — D-142)
```

---

## 1. Bloqueadores legais (PRÉ-PROD obrigatório)

| Item | Quem | SLA | Status |
|---|---|---|---|
| **DPA Cloudflare** assinada | João (controlador) via dashboard Account → Compliance → DPA | imediato (autoassinatura) | ⏳ pendente |
| **DPA AWS GDPR** assinada | João via AWS Artifact → Agreements → AWS GDPR DPA | imediato | ⏳ pendente |
| **DPO formalmente nomeado** | João (vault `08-security/lgpd.md` §7.3) | 1 dia | ⏳ pendente |

**Sem DPAs, NENHUM dado pessoal real entra em prod.** Bloqueador P0.

---

## 2. Cloudflare Layer

### 2.1 DNS + Zone (FAZER PRIMEIRO — bloqueia todo resto)

**Order**: cliente delega NS para Cloudflare ANTES de qualquer outro setup.

```bash
# 1. Dashboard CF: Add Site → mastersindico.com.br → Free plan
# 2. Cliente atualiza NS em registrar BR (Registro.br):
#    NS antigos → ns1.cloudflare.com / ns2.cloudflare.com (varia por conta)
# 3. Universal SSL emitido automaticamente (cert wildcard *.mastersindico.com.br)
# 4. Propagação 1-24h (.com.br via Registro.br)

# Verificar:
dig NS mastersindico.com.br +short
```

**Subdomínios canônicos** (alinha com [[deploy]] §5 + ADR-0045):

| Subdomain | Destino | Configurado em |
|---|---|---|
| `mastersindico.com.br` (apex) | Pages (lp) | wrangler `routes` |
| `auth.mastersindico.com.br` | Pages (auth) | idem |
| `app.mastersindico.com.br` | Pages (cms) | idem |
| `admin.mastersindico.com.br` | Pages (admin) atrás de Cloudflare Access | dashboard Access |
| `academia.mastersindico.com.br` | Pages (lms) | wrangler |
| `vizinhanca.mastersindico.com.br` | Pages (forum) | wrangler |
| `assembleia.mastersindico.com.br` | Pages (assembly) | wrangler |
| `api.mastersindico.com.br` | Workers BFF | wrangler `routes` |
| `webhooks.mastersindico.com.br` | Workers idempotente (Stripe/Mux/LiveKit/Resend/Twilio) | wrangler |
| `cdn.mastersindico.com.br` | R2 público | dashboard R2 → Custom Domains |
| `mail.mastersindico.com.br` | MX = CF Email Routing inbound + SPF/DKIM Resend | dashboard Email Routing + DNS records |

**Risco DNS**: TLD `.com.br` exige conformidade Registro.br; CAA records validar suporte; DNSSEC opcional M2.

### 2.2 Pages × 7 apps SolidJS

> **DECISÃO 2026-04-27 (research-loop)**: usar **Pages clássico** com `wrangler.toml` para M1 (já implementado em `Development/web/apps/<app>/wrangler.toml`). Migração para **Workers + Static Assets** (`wrangler deploy` em vez de `wrangler pages deploy`) entra como **D-### futuro M2** — guia oficial CF [migrate-pages-to-workers] confirma é o caminho moderno (Wrangler v4+) mas Pages clássico continua suportado.

**Comandos por app**:
```bash
cd apps/<APP>   # auth, cms, admin, lms, forum, assembly, lp
bun run build   # → dist/
npx wrangler pages deploy dist --project-name=mastersindico-<APP> --branch=main
npx wrangler pages project create mastersindico-<APP> --production-branch=main
npx wrangler pages secret put PUBLIC_API_URL --project-name=mastersindico-<APP>
```

**Custom domain** por app via dashboard Pages → Custom Domains. CF gera CNAME automático.

**Custo**: Pages free tier — ilimitado dentro de 500 builds/mês free, 100k req/dia free.

### 2.3 Workers BFF (`api.mastersindico.com.br`)

```bash
npx wrangler init mastersindico-bff --type hono
# wrangler.jsonc:
```

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "mastersindico-bff",
  "main": "src/index.ts",
  "compatibility_date": "2026-04-27",
  "compatibility_flags": ["nodejs_compat"],
  "placement": { "region": "aws:sa-east-1" },
  "observability": { "enabled": true, "head_sampling_rate": 1.0 },
  "routes": [
    { "pattern": "api.mastersindico.com.br/*", "custom_domain": true }
  ],
  "vpc_services": [
    {
      "binding": "GO_BACKEND",
      "service_id": "<SERVICE_ID_workers_vpc_create>",
      "remote": true
    }
  ],
  "r2_buckets": [
    { "binding": "UPLOADS", "bucket_name": "mastersindico-uploads" }
  ],
  "kv_namespaces": [
    { "binding": "SESSIONS", "id": "<KV_ID>" }
  ],
  "queues": {
    "producers": [
      { "binding": "WEBHOOK_DLQ", "queue": "webhooks-dlq" }
    ]
  }
}
```

**Workers Placement Hints** (Jan/2026): `placement.region = "aws:sa-east-1"` posiciona Worker no PoP CF mais próximo de sa-east-1, reduzindo latência ao backend AWS.

**Comandos**:
```bash
npx wrangler secret put STRIPE_WEBHOOK_SECRET --env production
npx wrangler secret put ZITADEL_INTROSPECT_TOKEN --env production
npx wrangler deploy --env production
```

**Custo**: Workers Paid $5/mês mínimo (necessário para Workers VPC binding + 30s CPU + 10MB script + observability 7d).

### 2.4 Workers VPC binding (CF ↔ AWS)

> **STATUS BETA confirmado** (CF docs: "Features and APIs may change before general availability. While in beta, Workers VPC is available for free to all Workers plans"). Risco residual de breaking change. **Plano B**: Tunnel + Public Hostname + Cloudflare Access Service Token (latência +20-50ms).

**Pré-requisito**: Tunnel CF criado + cloudflared rodando na VPC AWS (§4.3).

```bash
# Roles necessários na conta CF: Connectivity Directory Admin (criar) + Connectivity Directory Bind (vincular)
npx wrangler vpc service create ms-go-backend \
  --type http \
  --tunnel-id <TUNNEL_UUID> \
  --hostname backend.internal.mastersindico.local \
  --http-port 8080
# → retorna SERVICE_ID
# Colar SERVICE_ID em vpc_services do wrangler.jsonc do BFF
```

**Worker BFF acessa backend via binding**:
```ts
const response = await env.GO_BACKEND.fetch(
  `http://backend.internal:8080${new URL(request.url).pathname}`
);
```

**Origin CA certs suportadas** (Fev/2026): pode usar Cloudflare Origin CA cert no backend para HTTPS interno; simplifica vs Let's Encrypt.

### 2.5 R2 + KV + Durable Objects + Queues

```bash
npx wrangler r2 bucket create mastersindico-uploads
npx wrangler r2 bucket create mastersindico-exports-lgpd
npx wrangler r2 bucket create mastersindico-logs-archive

npx wrangler kv namespace create SESSIONS
npx wrangler kv namespace create FEATURE_FLAGS
npx wrangler kv namespace create CMS_CONTENT_CACHE

npx wrangler queues create webhooks-dlq
npx wrangler queues create video-moderation-jobs
```

**Durable Objects** (declarado em wrangler.jsonc + classe TS):
```jsonc
"durable_objects": {
  "bindings": [
    { "name": "ABAC_RESOLVER", "class_name": "AbacResolver" },
    { "name": "QUOTA_COUNTER", "class_name": "QuotaCounter" }
  ]
},
"migrations": [
  { "tag": "v1", "new_classes": ["AbacResolver", "QuotaCounter"] }
]
```

**Custo M1**: R2 free <10GB; KV ~$0.50/mês; Queues ~$0.40/mês.

### 2.6 Cron Triggers

```jsonc
"triggers": {
  "crons": [
    "0 3 * * *",       // lgpd_retention_cleanup (R2 archive)
    "*/15 * * * *"     // health check rolling
  ]
}
```

Limites: 3 crons free / 5 paid / 1000 com Crons API.

### 2.7 Access (admin SSO + MFA)

Dashboard Cloudflare One → Access → Applications:
- Type: Self-hosted
- Application domain: `admin.mastersindico.com.br`
- Identity providers: Google + GitHub (MFA enforce)
- Policy: require `mfa = true` + email domain do time
- Service tokens para CI healthcheck (não interativo)

**Custo**: Free tier até 50 users.

### 2.8 Email Routing + Turnstile

**Email Routing** (dashboard → Email):
- Custom address `lgpd@mastersindico.com.br` → forward DPO email
- `bounces+resend@`, `bounces+twilio@` → Worker handler para mark email/phone inválido em DB
- MX records auto-configurados

**Turnstile** (dashboard → Turnstile):
- Widget para `auth.mastersindico.com.br` (signup, password reset)
- Widget para `app.mastersindico.com.br/connect-me` (Connect Me form)
- Site key + secret key → wrangler secret put

### 2.9 Tunnel CF (configuração CF side; instalação AWS side em §4.3)

Dashboard Zero Trust → Networks → Tunnels → Create:
- Name: `ms-aws-saeast1-prod`
- Connectors: Cloudflared
- Token gerado → salvar em **AWS Secrets Manager** (`ms/prod/cloudflared/tunnel-token`)
- Public hostnames (fallback Plan B se Workers VPC quebrar):
  - `backend-internal.mastersindico.com.br` → `http://backend.internal:8080` (atrás de Access Service Token)

**HA**: 2+ réplicas cloudflared, AZ-spread. **NÃO autoscaling** (CF docs explicitamente: "downscaling will break existing user connections").

---

## 3. AWS sa-east-1 Layer

> **PENDÊNCIAS P0** antes de codificar:
> - Validar **PostgreSQL 18 GA em RDS sa-east-1**: `aws rds describe-db-engine-versions --engine postgres --region sa-east-1`. Se PG18 não GA, fallback PG17 + plano migration M2.
> - Validar **service quotas sa-east-1** (ECS tasks, OpenSearch nodes, RDS instances).
> - DPA AWS assinada via AWS Artifact.

### 3.1 Provisionamento via Terraform/CDK (não cliques manuais)

**Recomendação**: AWS CDK (TypeScript) ou Terraform `terraform-aws-modules/*` (community-validated, vault Context7 confirmou via `/terraform-aws-modules/terraform-aws-ecs`).

Pasta sugerida (separar repo se preferir):
```
infra/
├── terraform/
│   ├── 01-network/        # VPC + 6 subnets + NAT + IGW + SGs
│   ├── 02-data/           # RDS + ElastiCache + OpenSearch
│   ├── 03-compute/        # ECS cluster + ECR + IAM roles
│   ├── 04-secrets/        # KMS + Secrets Manager
│   └── 05-observability/  # CloudWatch log groups + alarms
└── README.md
```

### 3.2 VPC + Networking

CIDR `10.20.0.0/16`. 3 AZs (sa-east-1a/b/c). 6 subnets:
- `10.20.10.0/24`, `10.20.11.0/24`, `10.20.12.0/24` — private (3 AZs)
- `10.20.20.0/24`, `10.20.21.0/24`, `10.20.22.0/24` — public (3 AZs)

NAT Gateway Multi-AZ ($65/mês) ou single-AZ ($32/mês — trade-off HA × custo, M1 OK single).

### 3.3 Security Groups (least privilege)

| SG | Inbound | Outbound | Purpose |
|---|---|---|---|
| `sg-cloudflared` | none | TCP/UDP 443 (CF edge) | Tunnel egress |
| `sg-go-backend` | TCP 8080 from `sg-cloudflared` | all | Backend Go |
| `sg-rds-pg` | TCP 5432 from `sg-go-backend` | none | RDS |
| `sg-redis` | TCP 6379 from `sg-go-backend` | none | ElastiCache |
| `sg-opensearch` | TCP 443 from `sg-go-backend` | none | OpenSearch |

### 3.4 ECS Fargate (Backend Go + cloudflared)

**ECR**:
```bash
aws ecr create-repository --repository-name mastersindico/backend --region sa-east-1
aws ecr create-repository --repository-name mastersindico/cloudflared --region sa-east-1
```

**Task definition** (sidecar pattern OU service separado — Agente B recomenda **service separado** para HA cloudflared, evita acoplamento backend↔cloudflared replicas):

```jsonc
{
  "family": "ms-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [{
    "name": "backend",
    "image": "<ECR>/mastersindico/backend:<TAG>",
    "portMappings": [{ "containerPort": 8080, "protocol": "tcp" }],
    "secrets": [
      { "name": "DATABASE_URL", "valueFrom": "arn:aws:secretsmanager:sa-east-1:...:secret:ms/prod/db/url" },
      { "name": "REDIS_URL", "valueFrom": "arn:aws:secretsmanager:...:ms/prod/redis/url" },
      { "name": "OPENSEARCH_URL", "valueFrom": "arn:aws:secretsmanager:...:ms/prod/opensearch/url" }
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/ms-backend",
        "awslogs-region": "sa-east-1",
        "awslogs-stream-prefix": "go"
      }
    },
    "healthCheck": {
      "command": ["CMD-SHELL", "wget -q http://localhost:8080/health/ready || exit 1"],
      "interval": 30, "timeout": 5, "retries": 3, "startPeriod": 60
    }
  }]
}
```

**Cloudflared service separado** (AGENTE B recomenda):
```jsonc
{
  "family": "ms-cloudflared",
  "containerDefinitions": [{
    "name": "cloudflared",
    "image": "cloudflare/cloudflared:2026.4-latest",
    "command": ["tunnel", "--no-autoupdate", "run"],
    "secrets": [{
      "name": "TUNNEL_TOKEN",
      "valueFrom": "arn:aws:secretsmanager:...:ms/prod/cloudflared/tunnel-token"
    }],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": { "awslogs-group": "/ecs/ms-cloudflared", "awslogs-region": "sa-east-1" }
    }
  }]
}
```

**ECS service**:
```bash
aws ecs create-service \
  --cluster ms-prod \
  --service-name backend \
  --task-definition ms-backend:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[<PRIVATE_A>,<PRIVATE_B>],securityGroups=[sg-go-backend],assignPublicIp=DISABLED}" \
  --health-check-grace-period-seconds 60

aws ecs create-service \
  --cluster ms-prod \
  --service-name cloudflared \
  --task-definition ms-cloudflared:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[<PRIVATE_A>,<PRIVATE_B>],securityGroups=[sg-cloudflared],assignPublicIp=DISABLED}"
# desired-count fixo = 2; NÃO autoscaling
```

**Custo**: 2 backend (0.5vCPU/1GB) + 2 cloudflared (0.25vCPU/0.5GB) ≈ $30-40/mês.

### 3.5 RDS PostgreSQL Multi-AZ

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name ms-prod-pg \
  --db-subnet-group-description "Master Sindico PG private" \
  --subnet-ids <PRIVATE_A> <PRIVATE_B>

aws rds create-db-instance \
  --db-instance-identifier ms-prod-pg \
  --engine postgres --engine-version 18.1  `# fallback 17.x se 18 não GA em sa-east-1` \
  --db-instance-class db.t4g.medium \
  --allocated-storage 50 --storage-type gp3 \
  --multi-az \
  --master-username ms_admin \
  --master-user-password "$(aws secretsmanager get-random-password --password-length 32 --query RandomPassword --output text)" \
  --vpc-security-group-ids sg-rds-pg \
  --backup-retention-period 14 \
  --enable-iam-database-authentication \
  --storage-encrypted --kms-key-id <CMK_ARN> \
  --auto-minor-version-upgrade \
  --deletion-protection \
  --enable-cloudwatch-logs-exports postgresql \
  --region sa-east-1
```

**Custo**: db.t4g.medium Multi-AZ ≈ $130/mês + 50GB gp3 ≈ $7/mês = ~$140/mês.

**Backup PITR + cross-region snapshot mensal** (ADR-0035): habilitar `BackupRetentionPeriod=14` + Lambda mensal copia snapshot para us-east-1 com Object Lock R2.

### 3.6 ElastiCache Redis 7

```bash
aws elasticache create-cache-subnet-group \
  --cache-subnet-group-name ms-prod-redis \
  --cache-subnet-group-description "Master Sindico Redis private" \
  --subnet-ids <PRIVATE_A> <PRIVATE_B>

aws elasticache create-cache-cluster \
  --cache-cluster-id ms-prod-redis \
  --engine redis --engine-version 7.1 \
  --cache-node-type cache.t4g.micro \
  --num-cache-nodes 1 \
  --cache-subnet-group-name ms-prod-redis \
  --security-group-ids sg-redis \
  --region sa-east-1
# M1: single-node ($15/mês). M2+: Multi-AZ replication group.
```

### 3.7 OpenSearch Service (search canônico — fecha ADR-0042 Opção A)

```bash
aws opensearch create-domain \
  --domain-name ms-prod-search \
  --engine-version OpenSearch_2.13 \
  --cluster-config "InstanceType=t3.small.search,InstanceCount=1" \
  --ebs-options "EBSEnabled=true,VolumeType=gp3,VolumeSize=50" \
  --vpc-options "SubnetIds=<PRIVATE_A>,SecurityGroupIds=sg-opensearch" \
  --node-to-node-encryption-options "Enabled=true" \
  --encryption-at-rest-options "Enabled=true,KmsKeyId=<CMK_ARN>" \
  --access-policies file://opensearch-access-policy.json \
  --advanced-security-options "Enabled=true,InternalUserDatabaseEnabled=true,MasterUserOptions={MasterUserName=admin,MasterUserPassword=$(aws secretsmanager get-random-password ...)}" \
  --region sa-east-1
```

**Brazilian analyzer**: criar índices com `"analyzer": "brazilian"` (suporte nativo OpenSearch).

**Custo**: t3.small.search 1 node ~$30-50/mês M1; multi-AZ M2.

### 3.8 Secrets Manager + KMS + IAM

```bash
aws kms create-key \
  --description "Master Sindico CMK prod" \
  --multi-region \
  --policy file://kms-policy.json

aws kms create-alias \
  --alias-name alias/mastersindico-prod \
  --target-key-id <CMK_ID>

# Secrets
aws secretsmanager create-secret \
  --name ms/prod/db/master-password \
  --kms-key-id <CMK_ARN> \
  --secret-string "$(openssl rand -base64 32)"

aws secretsmanager create-secret --name ms/prod/db/url
aws secretsmanager create-secret --name ms/prod/redis/url
aws secretsmanager create-secret --name ms/prod/opensearch/url
aws secretsmanager create-secret --name ms/prod/cloudflared/tunnel-token
aws secretsmanager create-secret --name ms/prod/stripe/webhook-secret
aws secretsmanager create-secret --name ms/prod/zitadel/introspect-token
aws secretsmanager create-secret --name ms/prod/resend/api-key
aws secretsmanager create-secret --name ms/prod/twilio/auth-token
aws secretsmanager create-secret --name ms/prod/mux/access-token-secret
aws secretsmanager create-secret --name ms/prod/livekit/api-secret

# IAM role para ECS task
aws iam create-role --role-name ms-ecs-task --assume-role-policy-document file://ecs-trust.json
aws iam attach-role-policy --role-name ms-ecs-task --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
aws iam put-role-policy --role-name ms-ecs-task --policy-name SecretsRead --policy-document file://secrets-read.json
```

**Rotação trimestral** (vault `06-execution/deploy.md` §4): habilitar `rotation_lambda` para PG password (RDS native).

### 3.9 NÃO usar AWS para (cobertos por CF/SaaS — ver D-138-E)

S3 (R2 vence) · Route53 (CF DNS) · CloudFront (CF CDN) · WAF AWS (CF WAF) · API Gateway (Workers) · Lambda (Workers) · Cognito (Zitadel) · SES (Resend) · SNS (Workers Queues + Twilio).

---

## 4. Conexão CF ↔ AWS (Tunnel + Workers VPC)

### 4.1 Plano A (recomendado): Workers VPC binding

Pré-requisitos:
- Tunnel CF criado (§2.9)
- cloudflared rodando 2+ réplicas no ECS (§3.4)
- Roles CF: `Connectivity Directory Admin` (criar VPC service) + `Connectivity Directory Bind` (vincular Worker)

```bash
npx wrangler vpc service create ms-go-backend \
  --type http \
  --tunnel-id <TUNNEL_UUID> \
  --hostname backend.internal.mastersindico.local \
  --http-port 8080
# → SERVICE_ID em wrangler.jsonc do BFF (§2.3)
```

**Limitação**: Workers VPC é **BETA**; Plano B abaixo é fallback validado.

### 4.2 Plano B (fallback se Workers VPC quebrar pré-M1)

Tunnel + Public Hostname `backend-internal.mastersindico.com.br` → atrás de Cloudflare Access Service Token. Worker BFF usa `fetch()` HTTPS normal:

```ts
const response = await fetch(`https://backend-internal.mastersindico.com.br${path}`, {
  headers: {
    'CF-Access-Client-Id': env.CF_ACCESS_CLIENT_ID,
    'CF-Access-Client-Secret': env.CF_ACCESS_CLIENT_SECRET,
  }
});
```

Latência +20-50ms vs binding privado, mas 100% GA.

### 4.3 Cloudflared HA (CRÍTICO — CF docs explícito)

> CF documentação afirma literal: *"We do not recommend using cloudflared in autoscaling setups because downscaling will break existing user connections"*.

**Logo**:
- ECS service cloudflared `desiredCount=2` **fixo** (não autoscaling)
- AZ-spread (subnets em 2+ AZs)
- CloudWatch alarme se `< 2 healthy tasks` por 5min → SNS → email oncall

---

## 5. CI/CD GitHub Actions

### 5.1 Workflow web (`.github/workflows/web.yml`)

```yaml
name: web-deploy
on:
  push:
    branches: [main]
    paths: ['apps/**', 'packages/**', 'web/**']

jobs:
  deploy:
    strategy:
      matrix:
        app: [admin, assembly, auth, cms, forum, lms, lp]
      max-parallel: 3
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install --frozen-lockfile
      - run: cd apps/${{ matrix.app }} && bun run build
      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
          accountId: ${{ secrets.CF_ACCOUNT_ID }}
          command: pages deploy apps/${{ matrix.app }}/dist --project-name=mastersindico-${{ matrix.app }} --branch=main
```

### 5.2 Workflow backend (`.github/workflows/backend.yml`)

```yaml
name: backend-deploy
on:
  push:
    branches: [main]
    paths: ['backend/**']

permissions:
  id-token: write   # OIDC GitHub→AWS
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.26' }

      # Tests
      - run: cd backend && go test -race ./...
      - run: cd backend && go vet ./...
      - run: cd backend && gosec -severity high ./...
      - run: cd backend && govulncheck ./...

      # AWS OIDC
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/github-actions-deploy
          aws-region: sa-east-1

      # Build + Push
      - uses: aws-actions/amazon-ecr-login@v2
      - run: |
          docker build -t $ECR_URI:$GITHUB_SHA -t $ECR_URI:latest -f backend/Dockerfile backend/
          docker push $ECR_URI:$GITHUB_SHA
          docker push $ECR_URI:latest
        env:
          ECR_URI: ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.sa-east-1.amazonaws.com/mastersindico/backend

      # Migrations expand (NÃO contract no mesmo deploy)
      - run: |
          curl -L https://github.com/pressly/goose/releases/latest/download/goose_linux_amd64 -o goose
          chmod +x goose
          ./goose -dir backend/internal/shared/providers/database/migrations postgres "$(aws secretsmanager get-secret-value --secret-id ms/prod/db/url --query SecretString --output text)" up

      # Deploy ECS
      - uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: infra/ecs/backend-task.json
          service: backend
          cluster: ms-prod
          wait-for-service-stability: true

      # Smoke test
      - run: npx playwright test smoke-test.spec.ts --reporter=line
        env:
          BASE_URL: https://api.mastersindico.com.br
```

**Migrations expand-contract**: NUNCA `DROP TABLE` / `DROP COLUMN` no mesmo deploy. Linter CI bloqueia.

### 5.3 Smoke tests Playwright (5 fluxos críticos)

Vault [[deploy]] §1 já lista. Reutilizar.

---

## 6. Secrets Management

| Secret | Lugar primário | Como injetar | Rotação |
|---|---|---|---|
| TUNNEL_TOKEN | AWS Secrets Manager | ECS task secrets[] | Manual quando rotacionar tunnel |
| DATABASE_URL | AWS Secrets Manager (rotation Lambda) | ECS task secrets[] | 90d auto |
| REDIS_URL | AWS Secrets Manager | ECS task secrets[] | 180d |
| OPENSEARCH_URL | AWS Secrets Manager | ECS task secrets[] | 180d |
| STRIPE_WEBHOOK_SECRET | CF Workers secret + AWS Secrets Manager | wrangler secret put + ECS env | 90d (vault [[../12-docs/runbooks/secret-rotation]]) |
| ZITADEL_INTROSPECT_TOKEN | CF Workers secret + AWS Secrets Manager | idem | 180d |
| RESEND_API_KEY | CF Workers secret | wrangler secret put | 180d |
| TWILIO_AUTH_TOKEN | AWS Secrets Manager | ECS task secrets[] | 180d |
| MUX_TOKEN_SECRET | AWS Secrets Manager | ECS | 180d |
| LIVEKIT_API_SECRET | AWS Secrets Manager | ECS | 180d |
| CF_API_TOKEN | GitHub Actions secret | secrets.CF_API_TOKEN | manual |
| AWS_ACCOUNT_ID | GitHub Actions repo var (não secret) | OIDC role assumption | N/A |

**Cron Worker validador**: a criar — verifica idade dos secrets via APIs Stripe/Resend/etc e alerta se > 90d (vault exige rotação trimestral).

---

## 7. Observability M1 (sem Sentry — D-142 postergado M2)

### 7.1 Backend Go logs → CloudWatch

Driver `awslogs` na ECS task (config §3.4). zerolog JSON. Queries via CloudWatch Logs Insights.

**CloudWatch Alarms**:
- Error rate `ms.error_rate{bc=identity}` > 0.5% → SNS topic → email oncall
- p95 API > 500ms por 15min → SNS
- Webhook fail rate > 5% por 1h → SNS

### 7.2 Edge logs → Workers Logs + Logpush para R2

Workers Logs built-in (`observability.enabled = true` em wrangler.jsonc).

Logpush job para R2 long-term archive (5y para audit_log + lgpd_logs):
```bash
curl https://api.cloudflare.com/client/v4/accounts/$ACCT/logpush/jobs \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  --json '{
    "name": "workers-archive",
    "destination_conf": "r2://mastersindico-logs-archive/{DATE}?account-id=...",
    "dataset": "workers_trace_events",
    "enabled": true
  }'
```

### 7.3 Sem PagerDuty M1

[[deploy]] menciona PagerDuty mas D-138-C orçamento M1 não confirma. **CloudWatch SNS → email** suficiente M1; PagerDuty M2 se cliente decidir contratar.

### 7.4 PENDÊNCIA OTel/Loki/Tempo/Mimir

Vault [[../02-architecture/observability]] menciona stack OTel (ADR-0020). Decisão M1: **não wirar** (alinha D-142 Sentry postergado). M2 reabrir.

---

## 8. Smoke tests + Rollback

### 8.1 Smoke tests (Playwright)

5 fluxos críticos do vault [[deploy]] §1. Roda no GHA pós-deploy contra `staging.mastersindico.com.br` e `mastersindico.com.br` (prod).

### 8.2 Rollback por camada (TTR alvo)

| Camada | Comando | TTR |
|---|---|---|
| **CF Pages** | dashboard "Rollback to" OU `wrangler pages deployment list` | < 5min |
| **CF Workers** | `wrangler rollback --message "incident X"` | < 2min |
| **AWS ECS backend** | `aws ecs update-service --task-definition ms-backend:N-1` | < 10min |
| **AWS ECS cloudflared** | idem | < 5min |
| **RDS migration (expand)** | rollback código sem ação DB (forward-compat) | < 10min |
| **RDS migration (contract destrutivo)** | PITR restore (proibido sem aviso prévio + DB snapshot) | 2-30min |

### 8.3 Procedure rollback

Ver runbook atualizado [[../12-docs/runbooks/rollback-deploy]] (precisa atualização da versão Railway-era).

---

## 9. DPA LGPD (PRÉ-PROD)

### Cloudflare DPA

- URL: https://www.cloudflare.com/cloudflare-customer-dpa/
- Self-service via dashboard → Account → Compliance → Data Processing Agreement
- Cliente assina; arquivar PDF no Drive corporativo + entry em [[../STATE]] D-### "DPA Cloudflare assinada"

### AWS DPA (GDPR)

- Console → AWS Artifact → Agreements → AWS GDPR Data Processing Addendum
- Cliente assina; arquivar PDF + entry [[../STATE]]

### Bloqueador

**Sem ambos DPAs, prod NÃO recebe dados pessoais**. Aplica-se a todas as 6 personas (síndico, morador, empresa, parceiro, agência, admin).

---

## 10. Cronograma sugerido (21 dias úteis até M1 2026-05-18)

| Dias | Marcos |
|---|---|
| **D1** (28/04) | DPAs assinadas; NS delegado para CF; PG18 GA validado; roles CF para João |
| **D2-3** | Terraform/CDK: VPC + subnets + NAT + SGs + KMS + IAM |
| **D4-5** | RDS Multi-AZ + ElastiCache + OpenSearch provisionados |
| **D6** | Tunnel CF criado; cloudflared task pronta para deploy |
| **D7** | Worker BFF Hono `/health` + Workers VPC binding testado |
| **D8** | Backend Go Dockerfile + ECR + ECS task definition + service |
| **D9** | goose `up` contra RDS via bastion temporário |
| **D10-12** | 7 Pages deploy + custom domains + secrets |
| **D13-14** | GHA workflows web + backend + Playwright smoke |
| **D15** | Rollback drill end-to-end |
| **D16-17** | Coverage gates + dual-check final + DR drill |
| **D18-21** | Buffer + go/no-go M1 |

---

## 11. Custo estimado mensal M1 (USD, conservador)

| Item | Preço |
|---|---|
| CF Workers Paid | $5 + 10M req grátis |
| CF Access (10 admins) | $30 |
| CF R2 (<50GB) | $1-5 |
| CF Workers VPC | **GRÁTIS durante beta** |
| AWS ECS Fargate (4 tasks) | $30-40 |
| AWS RDS db.t4g.medium Multi-AZ + 50GB | $140 |
| AWS ElastiCache cache.t4g.micro | $15-20 |
| AWS OpenSearch t3.small.search | $30-50 |
| AWS NAT GW (single-AZ M1) | $32 |
| AWS data transfer + CloudWatch | $20 |
| AWS Secrets Manager + KMS | $10 |
| Twilio SMS volume baixo | $5-30 |
| Resend (3k emails free) | $0 |
| Stripe / Mux / LiveKit / Zitadel | já contratados / pay-per-use |
| **Total estimado M1** | **$320-380/mês** |

Comparação Railway anterior (~$100-150 sem multi-AZ + sem busca real) — investimento adicional justifica compliance + escalabilidade.

---

## 12. Top 5 Riscos priorizados (consolidado Agente A+B)

| # | Risco | Sev | Mitigação |
|---|---|---|---|
| 1 | **Workers VPC ainda BETA** ("may change before GA") | 🔴 Alto | Plano B Tunnel + Public Hostname + Access Service Token validado em paralelo Sprint 10 |
| 2 | **PG18 RDS sa-east-1 GA?** (não validado) | 🔴 Alto | Validar D1; fallback PG17; planejar migration M2 |
| 3 | **DPAs LGPD não assinadas** | 🔴 Alto | Cliente assina D1 ambas via auto-portal |
| 4 | **cloudflared HA frágil** (sem autoscaling, AZ-spread manual) | 🟠 Médio | 2 tasks fixo + AZ-spread + CW alarme |
| 5 | **21 dias para M1 com infra zero** | 🟠 Médio | Pareamento intensivo + Terraform pré-pronto + freeze features novas |

---

## 13. Pendências para validação adicional

1. PG18 GA em RDS sa-east-1 (validar D1 via CLI)
2. Workers VPC quebra-de-API durante beta (subscrever changelog CF)
3. Pages clássico vs Workers Static Assets (decidir M1 ou M2?)
4. PagerDuty M1 sim/não (orçamento)
5. DPO formalmente nomeado
6. Backup PG cross-region snapshot mensal (ADR-0035)
7. Stack OTel Loki/Tempo/Mimir vs CloudWatch+CF Logs M1
8. Custo Workers VPC pós-GA (beta grátis; preço futuro desconhecido)
9. NAT Multi-AZ ($65) vs single ($32) trade-off

---

## Links

- [[../STATE]] — D-138, D-138-E, D-139..D-142
- [[../02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]]
- [[../02-architecture/adr/0017-railway-initial-aws-m4]] (parte ressuscitada via D-138-E)
- [[../02-architecture/adr/0042-search-provider]] — Opção A (OpenSearch sa-east-1) fechada
- [[deploy]] — versão Railway-era (parcialmente substituída por esta)
- [[rollback]] — strategy
- [[../12-docs/runbooks/rollback-deploy]] — operacional (precisa update Railway→CF+AWS)
- [[../12-docs/runbooks/secret-rotation]] — operacional (precisa update)
- [[../12-docs/runbooks/dr]] — disaster recovery
- [[../12-docs/runbooks/incident-lgpd-breach]]
- [[../12-docs/runbooks/webhook-reprocessing]]
- [[../../../10-knowledge/providers/cloudflare/_moc]]
- [[../../../10-knowledge/providers/cloudflare/integration-guide]]
- [[../../../10-knowledge/providers/cloudflare/workers]]
- [[../../../10-knowledge/providers/cloudflare/tunnel]]
- [[../../../10-knowledge/providers/cloudflare/access]]
- Cloudflare Workers VPC docs: https://developers.cloudflare.com/workers-vpc/
- Cloudflare Tunnel deployment AWS: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/deployment-guides/aws/
- Workers Placement Hints (Jan/2026): https://developers.cloudflare.com/workers/configuration/placement/
- Migrate Pages to Workers guide: consultado via MCP `mcp__plugin_cloudflare_cloudflare-docs__migrate_pages_to_workers_guide`
- Terraform AWS ECS module: https://registry.terraform.io/modules/terraform-aws-modules/ecs/aws
- AWS GDPR DPA: https://aws.amazon.com/compliance/gdpr-center/
- Cloudflare DPA: https://www.cloudflare.com/cloudflare-customer-dpa/
