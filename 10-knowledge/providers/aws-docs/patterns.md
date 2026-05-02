---
title: AWS Docs MCP — Patterns
type: note
tags: [provider, mcp, aws-docs, patterns]
source: https://docs.aws.amazon.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [AWS Docs Patterns]
---

# AWS Docs MCP — Patterns

Padrões atemporais de uso do MCP AWS Docs em decisão arquitetural AWS.

## 1. Fluxo canônico de consulta

```
1. search_documentation("S3 bucket policy examples")
   → top-N URLs relevantes
2. read_documentation(<url específica>)
   → policy JSON + explicação
3. Cross-check pricing (não incluído em docs):
   → https://calculator.aws
4. Cross-check region availability:
   → https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/
5. Cross-check quota real da conta:
   → Service Quotas console
```

**Nunca** parar em passo 2 se decisão envolve custo, disponibilidade regional ou quotas.

## 2. IAM least-privilege a partir de exemplo docs

Exemplos oficiais de IAM policy tendem a ser **permissivos demais** para produção (`"Resource": "*"`, `"Action": "s3:*"`). Protocolo:

1. Começar pelo exemplo da doc.
2. **Restringir `Resource`** para ARN específico (bucket, função, tabela).
3. **Restringir `Action`** ao conjunto mínimo realmente usado.
4. **Adicionar `Condition`** com `aws:SourceIp`, `aws:PrincipalOrgID`, `aws:ResourceTag` conforme modelo de segurança.
5. Validar com IAM Policy Simulator antes de aplicar.

## 3. Cross-region availability check

Docs dizem "Service X is GA". Não significa que está **na sua região**.

Para decisões multi-region:
1. Fetch doc do serviço.
2. Abrir `regional-product-services` e confirmar pelo menos as regiões críticas do projeto.
3. Se ausente em região obrigatória: registrar como `D-###` no STATE do projeto + alternativa (serviço equivalente ou deploy em outra região).

Exemplo concreto: Lambda@Edge só roda em `us-east-1`; impacta decisões de CDN + edge compute.

## 4. Pricing cálculo — docs são **insuficientes**

Doc diz "Lambda charges: $0.20 per 1M requests + $0.0000166667 per GB-second". Real:
- Data transfer out (especialmente cross-region) tem pricing próprio.
- Provisioned Concurrency muda modelo.
- Free tier afeta dev; prod geralmente não.
- Enterprise agreements alteram pricing.

**Sempre** usar calculator.aws + cruzar com fatura real de projeto existente.

## 5. Service Quotas console > doc

Doc diz "Lambda: 1000 concurrent executions (default)". Sua conta pode ter `500` (reduzido por "conta nova") ou `5000` (aumentado por request).

Para decisão de escala: `aws service-quotas list-service-quotas --service-code lambda` ou console. Doc é ponto de partida; conta é realidade.

## 6. Awareness de deprecations + migration paths

APIs AWS são **muito estáveis** mas deprecações acontecem:
- EC2 Classic (já removida)
- Old S3 URL style (path vs virtual-host)
- Redshift → Redshift Serverless

Quando doc menciona "legacy" ou "deprecated": seguir migration guide linked. Research-loop Etapa 2 deve pegar isso.

## 7. re:Invent talks como fonte complementar

Doc cobre "how". re:Invent talks (YouTube) cobrem:
- **Patterns em produção** (customer case studies).
- **Anti-patterns confessos** — quando AWS recomenda serviço X em vez de Y.
- **Roadmap público** — futuro próximo do serviço.

Integrar com research-loop Etapa 3 (big tech practices).

## 8. Prefira managed > self-hosted, salvo exceção

Defaults recomendados AWS (ECS Fargate, RDS, MSK, EKS) têm ergonomia superior ao self-hosted no EC2. Só ir self-hosted quando:
- Economia massiva comprovada (tipicamente > $10k/mês).
- Compliance específico que managed não atende.
- Feature que managed não expõe.

Doc geralmente defaulta pra managed — boa heurística.

## 9. Combinação com `railway/_moc`

Para startups/MVPs: **AWS é tipicamente over-engineering**. [[../railway/_moc]] (PaaS) cobre 80% dos casos com 20% do esforço ops.

Ir para AWS quando:
- Escala > $5k/mês em PaaS.
- Compliance AWS-only (FedRAMP, HIPAA BAA específico).
- Serviços AWS-only (Ground Station, Braket, IoT Greengrass).
- Equipe DevOps madura.

Research-loop deve começar por "Railway serve?" antes de "AWS ECS ou Lambda?".

## 10. Audit trail de decisões AWS

Toda decisão arquitetural envolvendo AWS registrada em `STATE.md` do projeto como D-### com:
- URL específico da doc consultada.
- Data da consulta (`source_date`).
- Region(s) confirmadas.
- Alternativas rejeitadas (GCP/Azure/Railway equivalente).
- Cost estimate do Pricing Calculator.

Sem isso, re-avaliação 6 meses depois exige refazer research do zero.

## Links

- [[_moc]]
- [[overview]]
- [[../_moc]]
- [[../railway/_moc]] — alternativa PaaS
- [[../../../30-agents/playbooks/research-loop]]
- [[../../security/owasp-top-10]] — OWASP mapeia para muitos serviços AWS (WAF, Shield, GuardDuty)
