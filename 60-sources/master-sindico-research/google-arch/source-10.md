---
title: Source 10 — SRE Workbook Error Budget Policy
type: source
tags: [research, google, sre, error-budget, policy, reliability]
source: https://sre.google/workbook/error-budget-policy/
created: 2026-04-23
updated: 2026-04-23
aliases: [sre-error-budget-policy]
---

# Source 10 — SRE Workbook: Error Budget Policy

- **URL primária**: https://sre.google/workbook/error-budget-policy/
- **Livro**: "The Site Reliability Workbook" — O'Reilly, 2018.
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Framework
- Error budget = "1 minus the SLO of the service".
- Ex: 99.9% availability → 0.1% errors → ~1.000 falhas em 1M requests em 4 semanas.

### Roles
- **Service teams**: manage releases + reliability work.
- **CTO**: escalation authority pra disputas.

### Policy template components
- **Trigger**: quando serviço excede error budget em janela de 4 semanas → policy ativa imediato.
- **Primary consequence**: "halt all changes and releases other than P0 issues or security fixes until the service is back within its SLO".
- **Mandatory actions** quando:
  - Internal code bugs ou procedural errors causaram exaustão.
  - Postmortems identificam chance de reduzir dependências externas.
  - Miscategorização permitiu erros fora do budget.
- **Permitted exceptions**: outages por company-wide infra, upstream service failures, out-of-scope traffic, miscategorização.

### Escalation requirements
- Incidente individual consumindo >20% do budget trimestral → postmortem + remediação documentada.
- Categoria de outage recorrente consumindo >20% anualmente → commitment formal de planejamento trimestral.

## Aplicabilidade ao Master Síndico

- **Direta M1**: ADR-00XX "Error Budget Policy do Master Síndico" com:
  - **Trigger**: budget 99.5% exaurido em 4 semanas por BC.
  - **Consequência**: freeze releases exceto P0/security até recuperar. Solo-dev = eu mesmo auto-bloqueio.
  - **Exceções**: Stripe/Zitadel/Livekit/cloud provider (infra externa) → não contabilizar; documentar como A-### + postmortem de categorização.
  - **Escalation**: >20% budget consumido → postmortem obrigatório em `11-audit/postmortems/`.
- **Adaptação solo-dev**: role "CTO" = eu; "Service team" = eu. Política ainda é valiosa: força pausa consciente antes de continuar adicionando features num sistema quebrando.

## Links relacionados

- [[source-04]] SLO chapter.
- [[source-09]] Implementing SLOs.
- [[source-11]] Postmortem Culture (pareia com este).
- Destilado: [[_destilado#1.2 Error budget]].
