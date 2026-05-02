---
title: MOC — 01-domain
type: moc
tags: [moc, master-sindico, ddd, domain]
project: master-sindico
created: 2026-04-22
---

# 01-domain — Domínio, DDD, regras

**Modelo do negócio** em linguagem ubíqua. Separa claramente três tipos de regra: **de negócio** (o quê), **de domínio** (invariantes), **de implementação** (como).

## Notas canônicas

### Modelo
- [[bounded-contexts]] — `identity` · `billing` · `institutional` · `commercial` · `content` · `assembly` + cross-domain
- [[context-map]] — relações entre contextos (shared kernel, customer-supplier, ACL)
- [[ubiquitous-language]] — glossário com termos pt-BR vs inglês técnico
- [[aggregates]] — agregados por contexto (Condomínio-raiz, Assembleia-raiz, Subscription-raiz, etc.)
- [[value-objects]] — CPF, CNPJ, Email, Password, Money, Phone, FracaoIdeal, ReputacaoTier

### Regras
- [[business-rules]] — regras de **negócio** (o quê o negócio exige): trial por persona, reputação, ciclo Connect Me, quórum por fração ideal
- [[domain-rules]] — **invariantes** e pré-/pós-condições das entidades (soma de fração = 100%, 1 sessão ativa por user, reputação derivada de métricas)
- [[implementation-rules]] — regras de **implementação** do projeto (CQRS, UoW, Saga, sem SDK no domain, mappers obrigatórios)
- [[invariants]] — lista consolidada de invariantes testados com PBT

### Eventos de domínio
- [[domain-events]] — `CondominiumCreated`, `UnitRegistered`, `MembershipAssigned`, `SubscriptionActivated`, `VideoApproved`, `AssemblyClosed`, etc.

## Fontes (destilar)

- [[../client-vault/02 - Domínios e Requisitos/Domínio - Identidade]]
- [[../client-vault/02 - Domínios e Requisitos/Domínio - Comercial]]
- [[../client-vault/02 - Domínios e Requisitos/Domínio - Conteúdo]]
- [[../client-vault/02 - Domínios e Requisitos/Domínio - Assembleia]]
- [[../client-vault/02 - Domínios e Requisitos/Domínio - Cross-Domain e Integrações]]
- [[../client-vault/02 - Domínios e Requisitos/Modelo ABAC]]
- [[regras-canonicas]]
- [[../11-audit/quebras-detectadas-2026-04-22]]
- [[../specs/requirements/identity]], [[../specs/requirements/commercial]], [[../specs/requirements/content]], [[../specs/requirements/assembly]], [[../specs/requirements/institutional]], [[../specs/requirements/billing]]

## Vizinhos

- [[../_moc]]
- [[../00-product/_moc]]
- [[../02-architecture/_moc]] — traduz domínio em arquitetura
- [[../04-requirements/_moc]] — requirements funcionais por módulo
- [[../../../10-knowledge/patterns/_moc]] — padrões DDD tático
