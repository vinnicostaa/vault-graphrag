---
title: Runbook — CI/CD GitHub Actions (Web CF Pages + Backend AWS ECS)
type: guide
tags:
  - runbook
  - ci-cd
  - github-actions
  - cloudflare-pages
  - aws-ecs
  - wrangler
  - master-sindico
  - v2
source: >-
  Vault deploy-strategy + cloudflare/wrangler-action + aws-actions/* docs
  oficiais 2026-04-27
created: '2026-04-27'
updated: '2026-04-27'
---

# Runbook — CI/CD GitHub Actions

Workflows GHA para deploy automatizado de **7 apps web (CF Pages)** + **backend Go (AWS ECS Fargate)** + **Worker BFF (CF Workers)** + **migrations (goose expand-contract)** + **smoke tests (Playwright)**.

**Princípios**:
- OIDC GitHub→AWS (não long-lived access keys)
- Wrangler API token com scope mínimo (não "All Resources")
- Migrations expand-only no mesmo deploy (contract sempre em deploy seguinte)
- Smoke test obrigatório pós-deploy; rollback automático se falhar

## 1. Repos e estrutura

Vault [[../../03-subprojects/_moc]]: 3 repos independentes (`web`, `backend`, `app`). Workflows GHA por repo.

```
web/
├── .github/workflows/web-deploy.yml         # 7 Pages
└── apps/{auth,cms,admin,lms,forum,assembly,lp}/

backend/
├── .github/workflows/backend-deploy.yml     # ECS Fargate + migrations
├── .github/workflows/bff-deploy.yml         # Worker BFF (api.mastersindico.com.br)
├── workers/bff/                             # Worker TS (Hono)
└── internal/                                # Go

infra/   (opcional repo separado OU pasta em backend)
├── terraform/
└── .github/workflows/infra-plan.yml         # terraform plan em PR; apply manual
```

## 2. Secrets GitHub Actions (configurar antes)

**Repo `web`**:
- `CF_API_TOKEN` — Cloudflare API token (scope: `Workers Scripts:Edit`, `Pages:Edit`, `Account:Read`, `Zone:Read` para `mastersindico.com.br`)
- `CF_ACCOUNT_ID` — ID da conta CF (`55b12830f282ead1051a0fa72e82088d` — vault memória já registrou)

**Repo `backend`**:
- `CF_API_TOKEN` — mesmo (para deploy do BFF)
- `CF_ACCOUNT_ID` — mesmo
- `AWS_ACCOUNT_ID` — repo var (não secret)
- `AWS_DEPLOY_ROLE` — repo var: `arn:aws:iam::<ACCOUNT_ID>:role/github-actions-deploy` (OIDC)

**OIDC GitHub→AWS** (configurar 1 vez na conta AWS):
```bash
# Criar OIDC provider
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1

# Criar role com trust policy para GHA
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com" },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": { "token.actions.githubusercontent.com:aud": "sts.amazonaws.com" },
      "StringLike": { "token.actions.githubusercontent.com:sub": "repo:mastersindico/backend:*" }
    }
  }]
}
EOF

aws iam create-role --role-name github-actions-deploy \
  --assume-role-policy-document file://trust-policy.json

# Attach policies necessárias (least privilege)
aws iam put-role-policy --role-name github-actions-deploy \
  --policy-name DeployPolicy \
  --policy-document file://deploy-policy.json
# deploy-policy.json: ECR push + ECS update-service + Secrets read + CloudWatch logs
```

## 3. Workflow `web-deploy.yml` (7 Pages em paralelo)

```yaml
# web/.github/workflows/web-deploy.yml
name: web-deploy

on:
  push:
    branches: [main]
    paths: ['apps/**', 'packages/**', 'bun.lock', '.github/workflows/web-deploy.yml']
  pull_request:
    branches: [main]
    paths: ['apps/**', 'packages/**']

concurrency:
  group: web-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with: { bun-version: latest }
      - run: bun install --frozen-lockfile
      - run: bun run check        # biome
      - run: bun run test         # vitest

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    strategy:
      matrix:
        app: [admin, assembly, auth, cms, forum, lms, lp]
      max-parallel: 3
      fail-fast: false
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
        with: { bun-version: latest }
      - run: bun install --frozen-lockfile

      - name: Build ${{ matrix.app }}
        working-directory: apps/${{ matrix.app }}
        run: bun run build
        env:
          PUBLIC_API_URL: https://api.mastersindico.com.br
          PUBLIC_AUTH_URL: https://auth.mastersindico.com.br
          PUBLIC_ENV: prod

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
          accountId: ${{ secrets.CF_ACCOUNT_ID }}
          command: pages deploy apps/${{ matrix.app }}/dist --project-name=mastersindico-${{ matrix.app }} --branch=main

      - name: Health check
        run: |
          DOMAIN=$(case "${{ matrix.app }}" in
            auth) echo "auth.mastersindico.com.br" ;;
            cms) echo "app.mastersindico.com.br" ;;
            admin) echo "admin.mastersindico.com.br" ;;
            lms) echo "academia.mastersindico.com.br" ;;
            forum) echo "vizinhanca.mastersindico.com.br" ;;
            assembly) echo "assembleia.mastersindico.com.br" ;;
            lp) echo "mastersindico.com.br" ;;
          esac)
          # Aguarda 30s para CF cache invalidate
          sleep 30
          curl -f -o /dev/null -s -w "Status: %{http_code}\n" https://$DOMAIN/
```

## 4. Workflow `backend-deploy.yml` (ECS Fargate + migrations)

```yaml
# backend/.github/workflows/backend-deploy.yml
name: backend-deploy

on:
  push:
    branches: [main]
    paths: ['internal/**', 'cmd/**', 'go.*', 'Dockerfile', '.github/workflows/backend-deploy.yml']

permissions:
  id-token: write       # OIDC
  contents: read

concurrency:
  group: backend-${{ github.ref }}
  cancel-in-progress: false

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:18-alpine
        env: { POSTGRES_PASSWORD: testpw, POSTGRES_DB: msindico_test }
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
      redis:
        image: redis:7-alpine
        ports: ['6379:6379']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.26' }

      - run: go test -race -count=1 ./...
      - run: go vet ./...
      - name: gosec
        uses: securego/gosec@master
        with: { args: '-severity high ./...' }
      - name: govulncheck
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_DEPLOY_ROLE }}
          aws-region: sa-east-1

      - name: Login to ECR
        id: ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, push image
        env:
          ECR_REGISTRY: ${{ steps.ecr.outputs.registry }}
          ECR_REPO: mastersindico/backend
          TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPO:$TAG -t $ECR_REGISTRY/$ECR_REPO:latest -f Dockerfile .
          docker push $ECR_REGISTRY/$ECR_REPO:$TAG
          docker push $ECR_REGISTRY/$ECR_REPO:latest

      - name: Run goose migrations (expand only)
        env:
          DB_URL: ${{ secrets.RDS_PROD_URL }}   # alternativa: pegar via aws secretsmanager
        run: |
          curl -L https://github.com/pressly/goose/releases/latest/download/goose_linux_amd64 -o goose
          chmod +x goose
          ./goose -dir internal/shared/providers/database/migrations postgres "$DB_URL" up

      - name: Render task definition
        id: taskdef
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: infra/ecs/backend-task.json
          container-name: backend
          image: ${{ steps.ecr.outputs.registry }}/mastersindico/backend:${{ github.sha }}

      - name: Deploy ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.taskdef.outputs.task-definition }}
          service: backend
          cluster: ms-prod
          wait-for-service-stability: true

      - name: Smoke test
        run: |
          sleep 30
          curl -f https://api.mastersindico.com.br/health/ready
```

**Nota crítica migrations**: o step `goose ... up` executa **apenas migrations expand**. Migrations contract (DROP COLUMN/TABLE) precisam estar em PR separado, com aviso explícito + DB snapshot pré-deploy. CI **bloqueia** se detectar `DROP` em primeiro deploy de uma mudança (lint custom).

## 5. Workflow `bff-deploy.yml` (Worker BFF)

```yaml
# backend/.github/workflows/bff-deploy.yml
name: bff-deploy

on:
  push:
    branches: [main]
    paths: ['workers/bff/**', '.github/workflows/bff-deploy.yml']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - working-directory: workers/bff
        run: bun install --frozen-lockfile

      - working-directory: workers/bff
        run: bun run test

      - name: Deploy Worker BFF
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
          accountId: ${{ secrets.CF_ACCOUNT_ID }}
          workingDirectory: workers/bff
          command: deploy --env production

      - name: Smoke test
        run: |
          sleep 10
          curl -f https://api.mastersindico.com.br/health
```

## 6. Smoke test Playwright (5 fluxos críticos vault [[../../06-execution/deploy]])

`.github/workflows/smoke-tests.yml` (rodado pós-deploy backend):

```yaml
name: smoke-tests
on:
  workflow_run:
    workflows: ['backend-deploy']
    types: [completed]
    branches: [main]

jobs:
  smoke:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install --frozen-lockfile
      - run: bunx playwright install chromium

      - name: Smoke tests prod
        run: bun playwright test smoke/ --reporter=line
        env:
          BASE_URL: https://api.mastersindico.com.br
          AUTH_URL: https://auth.mastersindico.com.br
          APP_URL: https://app.mastersindico.com.br

      - name: Notify on failure
        if: failure()
        run: |
          curl -X POST $SLACK_WEBHOOK -d '{"text":":rotating_light: Smoke test FALHOU em prod - rollback recomendado"}'
```

## 7. Auto-rollback se smoke falhar

```yaml
# Adicionar ao backend-deploy.yml após smoke step
- name: Auto-rollback on smoke failure
  if: failure() && steps.smoke.outcome == 'failure'
  run: |
    PREV_TASK=$(aws ecs list-task-definitions --family-prefix ms-backend --sort DESC --query 'taskDefinitionArns[1]' --output text)
    aws ecs update-service --cluster ms-prod --service backend --task-definition $PREV_TASK
    echo "::error::Auto-rollback to $PREV_TASK"
```

## 8. Concurrency + branch protection

GitHub Settings → Branches → main → Branch protection:
- Require pull request reviews (1+)
- Require status checks: `test` (web) + `test` (backend) + `bff-deploy/test` antes de merge
- Require linear history
- Disable force push

## 9. Anti-padrões

- ❌ Long-lived AWS access keys em GHA secrets — use OIDC
- ❌ `--skip-tests` em `wrangler deploy` — roda check sempre
- ❌ Migrations DROP no mesmo deploy do código que usa nova schema
- ❌ Sem `wait-for-service-stability` em ECS deploy — pipeline retorna sucesso mas task pode estar crashing
- ❌ Sem smoke test pós-deploy — descobre regressão pelo cliente
- ❌ Sem `concurrency` group — 2 deploys paralelos de backend = race condition em migrations

## 10. Troubleshooting

| Sintoma | Causa | Fix |
|---|---|---|
| `wrangler-action` 401 | CF_API_TOKEN sem scope correto | Recriar token com `Workers:Edit` + `Pages:Edit` + `Account:Read` |
| ECS deploy timeout | task crashing em loop (image quebrada / secret missing / DB unreachable) | `aws ecs describe-services` + `aws logs tail /ecs/ms-backend` |
| Migration goose falha "permission denied" | DB user sem privilegio CREATE/ALTER | Usar usuário admin para migrations; user app só CRUD |
| Smoke test 502 | BFF não enxerga backend (Workers VPC binding errado) | Verificar `wrangler vpc service list` + tunnel HEALTHY |
| Pages deploy "project not found" | nome `mastersindico-<app>` não criado | `wrangler pages project create mastersindico-<app> --production-branch=main` |

## Links

- [[_moc]]
- [[../../06-execution/deploy-strategy]] §5
- [[deploy-cloudflare-pages]] (este runbook)
- [[deploy-aws-ecs-fargate]] (a criar — atualmente coberto em deploy-strategy §3.4)
- [[rollback-deploy]]
- [[secret-rotation]]
- cloudflare/wrangler-action: https://github.com/cloudflare/wrangler-action
- aws-actions/amazon-ecs-deploy-task-definition: https://github.com/aws-actions/amazon-ecs-deploy-task-definition
- aws-actions/configure-aws-credentials (OIDC): https://github.com/aws-actions/configure-aws-credentials
- goose migrations: https://github.com/pressly/goose
