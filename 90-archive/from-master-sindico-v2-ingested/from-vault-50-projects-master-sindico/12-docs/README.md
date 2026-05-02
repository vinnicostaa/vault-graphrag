---
title: README — Master Síndico (entry point)
type: project
tags: [master-sindico, docs, readme, entry-point]
project: master-sindico
created: 2026-04-22
---

# Master Síndico — Documentação

Entry point pra devs, stakeholders, futuros agentes. Para contrato de agente, veja [[../CLAUDE]].

---

## Quem deveria ler o quê

| Persona | Leia |
|---|---|
| **Novo dev** | [[../overview]] → [[../03-subprojects/backend/README]] → [[../06-execution/STEPS]] → [[how-to/setup-dev-env]] |
| **Agente Claude** | [[../../../CLAUDE]] → [[../CLAUDE]] → [[../STATE]] → [[../11-audit/AUDIT]] |
| **Stakeholder** | [[../00-product/vision]] → [[../ROADMAP]] → [[../10-schedule/cronograma-macro]] |
| **Oncall** | [[runbooks/_moc]] → [[../08-security/incident-response]] |
| **Novo síndico usuário** | (fora do escopo deste vault — doc no app) |

---

## Navegação por assunto

### Visão de produto
- [[../00-product/vision]]
- [[../00-product/personas]]
- [[../00-product/business-model]]
- [[../00-product/glossary]]

### Domínio e regras
- [[../01-domain/bounded-contexts]]
- [[../01-domain/ubiquitous-language]]
- [[../01-domain/business-rules]]
- [[../01-domain/domain-rules]]
- [[../01-domain/invariants]]

### Arquitetura
- [[../02-architecture/system-overview]]
- [[../02-architecture/clean-arch-mapping]]
- [[../02-architecture/openapi/_moc]]
- [[../02-architecture/adr/_moc]]

### Sub-projetos
- [[../03-subprojects/backend/README]]
- [[../03-subprojects/web/README]]
- [[../03-subprojects/mobile/README]]

### Requisitos e Tasks
- [[../04-requirements/_moc]]
- [[../05-tasks/_moc]]

### Execução e Qualidade
- [[../06-execution/STEPS]]
- [[../07-quality/CLEAN_ARCH]]
- [[../07-quality/CLEAN_CODE]]
- [[../07-quality/CODE_CALISTHENICS]]
- [[../07-quality/PATTERNS]]
- [[../09-testing/test-strategy]]

### Segurança
- [[../08-security/BEYOND_CORP]]
- [[../08-security/threat-model]]
- [[../08-security/cve-process]]

### Governança (vivos)
- [[../STATE]] — decisões e dívidas
- [[../11-audit/AUDIT]] — findings
- [[../SESSION_CHARTER]] — sessão corrente
- [[../STEERING]] — não-negociáveis
- [[../ROADMAP]] — cronograma
- [[../DoR-DoD]] — critérios ready/done

---

## How-tos

Ver [[how-to/_moc]].

A criar (priorizado):
- `how-to/setup-dev-env.md`
- `how-to/criar-modulo.md`
- `how-to/criar-use-case.md`
- `how-to/adicionar-webhook.md`
- `how-to/fazer-migration.md`
- `how-to/rodar-testes-local.md`
- `how-to/deploy-staging.md`
- `how-to/rollback-producao.md`

## Runbooks

Ver [[runbooks/_moc]].

A criar (priorizado):
- `runbooks/db-down.md`
- `runbooks/redis-down.md`
- `runbooks/zitadel-down.md`
- `runbooks/stripe-webhook-fail.md`
- `runbooks/mux-webhook-fail.md`
- `runbooks/livekit-incident.md`
- `runbooks/cve-critical.md`

## ADRs

Ver [[../02-architecture/adr/_moc]]. Índice em [[adr-index]].

## Changelog

Ver [[changelog]].

---

## Suporte

Para dúvidas técnicas internas: abrir issue no repo.
Para LGPD / privacidade: DPO (a registrar).

---

## Atualização deste documento

Este README é atualizado:
- Ao fim de cada sprint (se links novos foram criados)
- Ao mudar estrutura da pasta do projeto
- Sempre que uma nova categoria de leitor for identificada

## Links

- [[_moc]]
- [[../CLAUDE]]
- [[../../../CLAUDE]] — raiz do vault
