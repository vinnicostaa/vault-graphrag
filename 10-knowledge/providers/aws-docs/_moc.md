---
title: MOC — AWS Docs MCP (documentação oficial AWS)
type: moc
tags: [moc, provider, mcp, aws, documentation]
category: documentation
mcp: awslabs/mcp (sub-module aws-docs)
sdk-official: "awslabs/mcp (Python)"
doc-oficial: https://docs.aws.amazon.com
doc-consulted: 2026-04-24T00:00:00.000Z
created: 2026-04-24
updated: 2026-04-24
aliases: [AWS Docs MCP, aws-mcp]
---

# AWS Docs MCP (awslabs)

**Categoria**: documentation retrieval (AWS services).

**MCP disponível**: ✅ via `awslabs/mcp` (monorepo oficial AWS, Python). Sub-server `aws-docs` especializado em search/read da doc oficial.

**Doc oficial**: https://docs.aws.amazon.com + https://github.com/awslabs/mcp. Consultada 2026-04-24.

## Quando usar

1. Decisão arquitetural envolvendo serviço AWS (ECS vs Lambda, RDS vs Aurora, SQS vs SNS vs EventBridge).
2. Validar feature/limits de um serviço antes de commit a decisão.
3. Buscar boilerplate IAM policy (doc tem examples).
4. Research-loop Etapa 2 quando stack inclui AWS.

## Quando **NÃO** substitui

- **Pricing**: docs cobrem em prosa; use [AWS Pricing Calculator](https://calculator.aws) para número realista.
- **Region availability**: nem todo serviço/feature vale em todas regiões — checar [Services by Region](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/).
- **Quotas reais**: docs listam default; conta real pode ter quotas diferentes (Service Quotas console).

## Funções principais

| Função | Uso |
|---|---|
| `search_documentation(query)` | Full-text search nas docs AWS |
| `read_documentation(url)` | Lê uma página (renderizada) |
| `recommend(url)` | Doc relacionada/similar |

## Padrão de uso

```
1. search_documentation("S3 bucket policy examples")
   → top-N URLs relevantes
2. read_documentation(<url específica>)
   → policy JSON + explicação
3. Validar service availability na região alvo (docs ≠ reality):
   → https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/
4. Cross-check pricing no Pricing Calculator
```

## Quando docs AWS divergem de reality

- **Defaults documentados** podem ser diferentes da conta real (ex.: Lambda default concurrency limit pode estar reduzido por conta).
- **Feature releases** podem ter rollout geográfico — doc diz "GA" mas region X ainda não tem.
- **API versions** — algumas APIs legadas ainda no doc; preferir v2/v3 quando disponível.

## Antipatterns

- ❌ Citar cost número da doc como "real cost" — pricing muda; use calculator.
- ❌ Assumir feature disponível em us-east-1 tem paridade em sa-east-1 (São Paulo).
- ❌ Ignorar IAM conditions — exemplo da doc pode ser permissivo demais para prod.

## Glossário rápido (AWS-specific)

- **ARN**: Amazon Resource Name — identificador global único.
- **IAM Condition Key**: filtro granular em policy (ex.: `aws:SourceIp`, `aws:PrincipalOrgID`).
- **VPC Endpoint**: conexão privada para serviço AWS sem passar pela internet pública.
- **Control Plane vs Data Plane**: APIs de gestão (CreateBucket) vs APIs de uso (PutObject).

## Sub-docs

- [[overview]] — arquitetura do awslabs/mcp monorepo + sub-servers (ECS, CDK, etc.)
- [[patterns]] — research-loop em decisão AWS, cross-region checks, IAM least-privilege workflow

## Vizinhos

- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../railway/_moc]] — alternativa PaaS quando AWS é over-engineering para o caso
- [[../../../30-agents/playbooks/research-loop]]
