---
title: ONB-P4 — Parceiro Aceites
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, parceiro]
project: master-sindico
persona: parceiro
category: ONB
screen_id: ONB-P4
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-P4 — Parceiro da Vizinhança · Aceites (Tela 4)

## Finalidade

**2 aceites obrigatórios** para ativar o parceiro: Termo de Promoções + Código de Conduta de Uso Indevido.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 4 Parceiro (literal)

## Persona e ABAC

- `local_company` pré-ativação.
- ABAC: `identity.terms.accept`.

## Fluxo da tela

1. **2 aceites obrigatórios**:
   - **Termo de Promoções** — regras para anúncio de promoções (sem enganar consumidor, respeito a preços/validade).
   - **Código de Conduta de Uso Indevido** — o que caracteriza abuso/má-fé + consequências (suspensão/ban — IDN-030).
2. Botão **[Finalizar cadastro]** → [[ONB-PFIN]].

## Componentes

- Accordion com texto literal (pendente jurídico cliente).
- Checkboxes versionados.

## Estados (loading/empty/error/success)

- **Error**: aceite faltando.
- **Success**: prossegue.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Aceitar | `/api/v1/onboarding/terms/accept` | POST | `{versions:{promocoes, conduta}}` | `{accepted_at}` |

## Regras de negócio críticas

- Consent versionado.
- Violação do código de conduta dispara workflow de moderação ([[../../../admin-privilegios]] §9.2 harassment reports).

## Ligações

- Origem: [[ONB-P3]].
- Destino: [[ONB-PFIN]].

## Gaps/ressalvas

- Texto literal pendente jurídico cliente.

## Links

- [[_moc]]
- [[ONB-P3]]
- [[ONB-PFIN]]
