---
title: gsd-security-auditor
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-security-auditor.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [security-auditor]
---

# gsd-security-auditor

**Família**: [[family-audit]]. **Pipeline (nosso)**: Reviewer (dimensão de segurança).

## Intenção

Não faz scan cego de vulnerabilidades — verifica que as mitigações **declaradas** em `<threat_model>` do PLAN.md realmente existem no código. Cada threat tem disposition explícita (`mitigate`/`accept`/`transfer`) e precisa de evidência no arquivo certo. Implementação é read-only; gaps são reportados em SECURITY.md, nunca corrigidos aqui.

## Entrada

- PLAN.md com bloco `<threat_model>`: registro completo (IDs, categorias STRIDE, dispositions, mitigation plans).
- SUMMARY.md seção `## Threat Flags`: nova superfície de ataque detectada durante execução.
- `<config>`: `asvs_level` (1/2/3), `block_on` (open / unregistered / none).
- Arquivos de implementação (grep dos padrões de mitigação).

## Saída

- SECURITY.md com:
  - Tabela de verificação (Threat ID × disposition × evidência `file:line`).
  - `unregistered_flag` para atack surface nova sem mapeamento.
  - Accepted risks log quando disposition é `accept`.
- Retorno: `## SECURED`, `## OPEN_THREATS` (blocker: threat unverified) ou `## ESCALATE`.

## Ferramentas declaradas

Read, Write, Edit, Bash, Glob, Grep.

## Padrão-chave replicável

- **Mitigate precisa de grep em TODOS os entry points**: uma única ocorrência da mitigação não basta se o threat afeta múltiplos pontos. Aceitar match único é o modo soft clássico.
- **Transfer e accept têm ônus de documentação**: `accept` exige entrada no accepted risks log; `transfer` exige referência à documentação (vendor SLA, insurance). Ausência de doc = threat OPEN, independente da disposition declarada.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-security-auditor]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
