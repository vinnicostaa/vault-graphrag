---
title: AWS Docs MCP — Overview
type: note
tags: [provider, mcp, aws-docs, overview, documentation]
source: https://docs.aws.amazon.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [AWS Docs Overview, awslabs-mcp]
---

# AWS Docs MCP (awslabs) — Overview

Servidor MCP mantido pelo **AWS Labs** (`awslabs/mcp`, monorepo Python) que expõe busca e leitura da documentação oficial AWS. Sub-server especializado `aws-docs` dentro do monorepo.

## Por que um MCP dedicado

AWS tem ~200+ serviços com doc espalhada em múltiplos produtos, regiões, consoles. Busca no site oficial (docs.aws.amazon.com) funciona mas é otimizada para humano (SEO-heavy, anúncios, navegação clicável). Agente LLM precisa:
- Retorno estruturado.
- Query por tópico específico.
- Sem HTML ruidoso.
- Com fonte verificada (URL).

## Arquitetura

```
Claude host
   │ (MCP call)
   ▼
awslabs/mcp (Python monorepo)
   │
   ├─→ aws-docs sub-server      ← este
   ├─→ aws-cdk sub-server
   ├─→ aws-ecs sub-server
   └─→ ... outros sub-servers
```

Cada sub-server pode ser habilitado/desabilitado individualmente no host.

## Funções principais do sub-server aws-docs

| Função | Uso |
|---|---|
| `search_documentation(query)` | Full-text search nas docs (retorna URLs ranqueadas) |
| `read_documentation(url)` | Lê uma página renderizada (HTML → texto limpo) |
| `recommend(url)` | Sugere docs relacionadas/similares |

## Casos de uso — quando usar

1. **Decisão arquitetural em stack AWS** — ECS vs Lambda, RDS vs Aurora, SQS vs SNS vs EventBridge.
2. **Validar features/limits** de um serviço antes de decidir.
3. **Exemplos de IAM policy** — doc tem snippets boilerplate.
4. **Research-loop Etapa 2** quando stack inclui AWS.
5. **CDK/Terraform**: cross-check de recursos disponíveis no provider AWS atual.

## Quando **NÃO** substitui outras fontes

| Gap | Alternativa |
|---|---|
| **Pricing real** | [AWS Pricing Calculator](https://calculator.aws) — docs cobrem em prosa, calculator dá número |
| **Region availability** | [Services by Region](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) — nem todo serviço em toda região |
| **Quota real da conta** | Service Quotas console — defaults na doc ≠ sua conta |
| **Patterns production** | Engineering blog AWS + AWS Architecture Blog — docs cobrem "how to use", não "when/why" |
| **Incident history** | AWS Health Dashboard + re:Invent talks (postmortems) |

## Comparação rápida com concorrentes cloud

| Cloud | Doc MCP equivalente |
|---|---|
| GCP | Não há MCP oficial (2026); `WebFetch` em `cloud.google.com/docs` |
| Azure | `microsoft/mcp` tem `azure-docs` sub-server |
| Cloudflare | Não há MCP oficial; `developers.cloudflare.com` via WebFetch |

AWS Labs lidera em volume de MCP servers oficiais.

## Limitações

- **Docs vs reality**: defaults documentados podem diferir de conta real (Lambda concurrency limit, Service Quotas). **Sempre** cruzar.
- **Feature releases**: doc pode marcar "GA" mas rollout geográfico incompleto.
- **API versions**: APIs legadas ainda no doc; preferir v2/v3 quando existem.
- **Security guidance genérico**: IAM condition examples são permissivos demais para prod — ajustar.

## Links

- [[_moc]]
- [[patterns]]
- [[../_moc]]
- [[../../../30-agents/mcp-integration]]
- [[../../../30-agents/playbooks/research-loop]]
- [[../railway/_moc]] — PaaS alternativo quando AWS é over-engineering
