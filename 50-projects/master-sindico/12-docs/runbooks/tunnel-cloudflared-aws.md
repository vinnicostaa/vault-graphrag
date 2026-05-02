---
title: Runbook — Cloudflare Tunnel (cloudflared) na VPC AWS
type: guide
tags:
  - runbook
  - cloudflared
  - tunnel
  - aws
  - ecs
  - workers-vpc
  - master-sindico
  - v2
source: >-
  Cloudflare Tunnel docs + Workers VPC + AWS deployment guide oficiais
  consultados 2026-04-27
created: '2026-04-27'
updated: '2026-04-27'
---

# Runbook — Cloudflare Tunnel (cloudflared) na VPC AWS

Conecta Workers (CF edge) ao backend Go (AWS ECS sa-east-1) via Cloudflare Tunnel + Workers VPC binding (ou public hostname fallback).

**Critério crítico** (CF docs literal): *"We do NOT recommend using cloudflared in autoscaling setups because downscaling will break existing user connections"*. Usar **desired_count fixo = 2** com AZ-spread.

## 1. Pré-requisitos

- Conta AWS com VPC criada em sa-east-1 (vault [[../../06-execution/deploy-strategy]] §3.2)
- ECS cluster `ms-prod` provisionado (§3.4)
- AWS Secrets Manager com secret `ms/prod/cloudflared/tunnel-token` (vazio inicialmente; preencher após criar tunnel CF)
- Conta Cloudflare com role `Connectivity Directory Admin` (cliente habilita para João)
- DNS zone `mastersindico.com.br` ativa em CF (vault [[dns-domain-setup]])

## 2. Criar Tunnel no Cloudflare (remotely-managed)

**Via dashboard** (preferido para M1):
1. https://dash.cloudflare.com → Zero Trust → Networks → Tunnels
2. Create Tunnel → Cloudflared
3. Name: `ms-aws-saeast1-prod`
4. Save tunnel
5. **Copy install token** (string `eyJhIj...`) — NÃO commitar no git

**Via API** (alternativa CI):
```bash
curl -X POST https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/cfd_tunnel \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"name":"ms-aws-saeast1-prod","tunnel_secret":"<base64 random 32 bytes>","config_src":"cloudflare"}'
# Retorna tunnel_id e token
```

## 3. Salvar token no AWS Secrets Manager

```bash
TUNNEL_TOKEN="eyJhIj..."   # token copiado do dashboard

aws secretsmanager put-secret-value \
  --secret-id ms/prod/cloudflared/tunnel-token \
  --secret-string "$TUNNEL_TOKEN" \
  --region sa-east-1
```

## 4. ECS Task Definition cloudflared

Arquivo `infra/ecs/cloudflared-task.json`:

```jsonc
{
  "family": "ms-cloudflared",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/ms-ecs-execution",
  "taskRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/ms-ecs-task",
  "containerDefinitions": [{
    "name": "cloudflared",
    "image": "cloudflare/cloudflared:2026.4-latest",
    "essential": true,
    "command": ["tunnel", "--no-autoupdate", "--metrics", "0.0.0.0:2000", "run"],
    "secrets": [{
      "name": "TUNNEL_TOKEN",
      "valueFrom": "arn:aws:secretsmanager:sa-east-1:<ACCOUNT_ID>:secret:ms/prod/cloudflared/tunnel-token"
    }],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/ms-cloudflared",
        "awslogs-region": "sa-east-1",
        "awslogs-stream-prefix": "cloudflared",
        "awslogs-create-group": "true"
      }
    },
    "healthCheck": {
      "command": ["CMD-SHELL", "wget -q http://localhost:2000/ready -O /dev/null || exit 1"],
      "interval": 30, "timeout": 5, "retries": 3, "startPeriod": 10
    }
  }]
}
```

## 5. ECS Service cloudflared (HA — 2 réplicas fixo, AZ-spread)

```bash
aws ecs register-task-definition \
  --cli-input-json file://infra/ecs/cloudflared-task.json \
  --region sa-east-1

aws ecs create-service \
  --cluster ms-prod \
  --service-name cloudflared \
  --task-definition ms-cloudflared \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[<PRIVATE_SUBNET_A>,<PRIVATE_SUBNET_B>],securityGroups=[sg-cloudflared],assignPublicIp=DISABLED}" \
  --placement-strategy "type=spread,field=attribute:ecs.availability-zone" \
  --health-check-grace-period-seconds 30 \
  --enable-execute-command \
  --region sa-east-1
```

**NÃO criar Application Auto Scaling** para este serviço. Operar manualmente em scale-out raro.

## 6. CloudWatch alarme se tunnel cai

```bash
# Alarm: < 2 healthy tasks por 5min
aws cloudwatch put-metric-alarm \
  --alarm-name ms-cloudflared-low-healthy-tasks \
  --metric-name HealthyTaskCount \
  --namespace AWS/ECS \
  --dimensions Name=ClusterName,Value=ms-prod Name=ServiceName,Value=cloudflared \
  --statistic Average --period 60 --evaluation-periods 5 \
  --threshold 2 --comparison-operator LessThanThreshold \
  --alarm-actions arn:aws:sns:sa-east-1:<ACCOUNT_ID>:ms-prod-alerts \
  --region sa-east-1
```

## 7. Configurar Workers VPC Service (Opção A — recomendada)

Pré-requisito: roles CF `Connectivity Directory Admin` para João.

```bash
# Recuperar tunnel UUID
CF_TUNNEL_ID=$(curl -s https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/cfd_tunnel \
  -H "Authorization: Bearer $CF_API_TOKEN" | jq -r '.result[] | select(.name=="ms-aws-saeast1-prod") | .id')

# Criar VPC Service apontando para backend Go via tunnel
npx wrangler vpc service create ms-go-backend \
  --type http \
  --tunnel-id $CF_TUNNEL_ID \
  --hostname backend.internal.mastersindico.local \
  --http-port 8080
# → retorna SERVICE_ID
```

**Adicionar binding** em wrangler.jsonc do BFF:
```jsonc
"vpc_services": [
  {
    "binding": "GO_BACKEND",
    "service_id": "<SERVICE_ID>",
    "remote": true
  }
]
```

**Worker code**:
```ts
const response = await env.GO_BACKEND.fetch(
  `http://backend.internal:8080${url.pathname}`
);
```

## 8. Plano B (fallback) — Public Hostname + Access Service Token

Se Workers VPC quebrar (BETA), usar Tunnel Public Hostname:

**Dashboard CF** → Zero Trust → Networks → Tunnels → `ms-aws-saeast1-prod` → Public Hostname:
- Hostname: `backend-internal.mastersindico.com.br`
- Service: `http://backend.internal:8080`
- Path: `*` (catch-all)

**Cloudflare Access** protege o hostname:
- Application → Self-hosted → `backend-internal.mastersindico.com.br`
- Policy: require Service Token (não interativo)
- Generate token: Settings → Service Auth → Create token "ms-bff-to-backend"

**Worker code (Plano B)**:
```ts
const response = await fetch(
  `https://backend-internal.mastersindico.com.br${url.pathname}`,
  {
    headers: {
      'CF-Access-Client-Id': env.CF_ACCESS_CLIENT_ID,
      'CF-Access-Client-Secret': env.CF_ACCESS_CLIENT_SECRET,
    },
    method: request.method,
    body: request.body,
  }
);
```

Latência +20-50ms, mas 100% GA.

## 9. Verificação fim-a-fim

Após cloudflared rodando + VPC service (ou public hostname) configurado:

```bash
# 1. Tunnel saudável (CF dashboard)
# Zero Trust → Networks → Tunnels → ms-aws-saeast1-prod
# Status deve ser "HEALTHY" com 2 connectors

# 2. Worker BFF deploy + test
cd workers/bff
npx wrangler deploy --env production
curl https://api.mastersindico.com.br/health
# Esperado: 200 OK { "ok": true } (passando pelo backend Go via tunnel)

# 3. ECS task cloudflared logs
aws logs tail /ecs/ms-cloudflared --region sa-east-1 --follow
# Esperado: "Connection registered" + "Updated to new configuration"
```

## 10. Troubleshooting comum

| Sintoma | Causa provável | Fix |
|---|---|---|
| Tunnel `STATUS: DOWN` no dashboard CF | cloudflared task não conseguiu egress 443 | Validar SG egress allow 443 + outbound NAT GW |
| Worker BFF 502 ao chamar `env.GO_BACKEND.fetch()` | service_id errado em wrangler.jsonc OU tunnel offline | `wrangler vpc service list` + verifica CF dashboard |
| `cloudflared` task entra em loop CrashBackoff | TUNNEL_TOKEN inválido em Secrets Manager | Re-criar tunnel + atualizar secret |
| Latência alta backend (>200ms) | Worker placement não está em sa-east-1 | Adicionar `placement.region = "aws:sa-east-1"` no wrangler.jsonc do BFF |
| 2 conectores listados mas só 1 healthy | AZ down OU task killed | Verificar ECS service events; alarm CloudWatch dispara |

## 11. Operação contínua

- **Monitor**: CF dashboard → Zero Trust → Networks → Tunnels → metrics; CW logs `/ecs/ms-cloudflared`
- **Update cloudflared**: bump image tag em task definition (`cloudflare/cloudflared:2026.X-latest`); re-deploy ECS service
- **Rotate token**: gerar novo token no CF dashboard → atualizar Secrets Manager → ECS service force redeploy
- **Logs**: `cloudflared` log level `info` por default; mudar para `debug` temporariamente em troubleshoot

## 12. Anti-padrões

- ❌ **Application Auto Scaling no service cloudflared** — quebra connections em downscale
- ❌ **1 task cloudflared single-AZ** — sem HA
- ❌ **TUNNEL_TOKEN em env var não-secret** — leaked em logs/dashboards
- ❌ **Backend Go com porta exposta na internet** — defeats the purpose; SG `sg-go-backend` aceita só de `sg-cloudflared`
- ❌ **Múltiplos tunnels para mesmo backend** — confunde routing; 1 tunnel + N replicas
- ❌ **Esquecer Origin CA cert no backend** se quiser mTLS interno (Workers VPC suporta desde Fev/2026)

## Links

- [[_moc]]
- [[../../06-execution/deploy-strategy]] §2.4, §4
- [[../../02-architecture/adr/0045-cloudflare-only-stack-deprecate-railway]]
- [[dns-domain-setup]]
- [[rollback-deploy]]
- Cloudflare Tunnel: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/
- Tunnel AWS deployment: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/deployment-guides/aws/
- Workers VPC: https://developers.cloudflare.com/workers-vpc/
- Tunnel availability: https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/configure-tunnels/tunnel-availability/
- Workers VPC troubleshooting: https://developers.cloudflare.com/workers-vpc/reference/troubleshooting/
