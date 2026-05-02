---
title: Pattern — Legal Terms Checkbox
type: pattern
tags:
  - pattern
  - ui
  - web
  - master-sindico
  - lgpd
  - legal
created: '2026-04-27'
updated: '2026-04-27'
---

# Pattern — Legal Terms Checkbox

Aceite legal explícito via checkbox **antes** de submit em fluxos com consequência regulatória (LGPD, Termo de uso, Aceite de mandato, declaração de responsabilidade técnica).

## Estrutura

- **Checkbox unchecked por default** — nunca pré-marcado (LGPD art. 7º — consentimento livre).
- **Texto do termo**: parágrafo curto + link "Ler termo completo" abrindo modal/nova aba (PDF).
- **Versionamento**: checkbox carrega `terms_version` do termo aceito; persiste no banco em `LegalAcceptance` aggregate.
- **CTA bloqueado** até checkbox marcado (com explicação do bloqueio em hover/aria).
- **Audit log**: timestamp, IP, user-agent, terms_version no `audit_trail`.

## Quando usar

- LGPD termo de uso (todo signup).
- Aceite de mandato (síndico).
- Aceite de Connect Me (validação).
- Declaração de responsabilidade (NR-1, técnica).

## Quando evitar

- Para "newsletter" ou opt-in marketing — usa toggle separado, fora do submit principal.

## Stack

SolidJS + Kobalte (Checkbox + Dialog para modal). Persistência via `LegalAcceptance` aggregate ([[../../../01-domain/aggregates/_moc]]).

## Cross-refs

- [[forms]] — geralmente vive dentro de form
- [[self-declaration-disclaimer]] — quando termo é auto-declaração
- [[../../../08-security/lgpd]]
- [[../../../04-requirements/functional/compliance]]

## Links

- [[_moc]]
