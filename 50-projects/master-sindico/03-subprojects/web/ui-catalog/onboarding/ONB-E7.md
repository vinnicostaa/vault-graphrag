---
title: ONB-E7 — Empresa Aceites (Tela 7)
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, empresa]
project: master-sindico
persona: empresa
category: ONB
screen_id: ONB-E7
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-E7 — Empresa · Uso da Plataforma (Tela 7)

## Finalidade

**2 aceites obrigatórios**: Termo de Conteúdo Técnico + Termo de Uso do Connect Me. Sela a política de conteúdo e a forma oficial de conectar com moradores/síndicos.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 7 Empresa

## Persona e ABAC

- `enterprise` pré-ativação.
- ABAC: `identity.terms.accept`.

## Fluxo da tela

1. Mensagem principal (literal): **"A plataforma facilita conexões, não garante contratos."**
2. **2 aceites**:
   - **Termo de Conteúdo Técnico** (regras de publicação sem chamadas comerciais).
   - **Termo de Uso do Connect Me** (canal único oficial para leads).
3. Botão **[Finalizar cadastro]** → [[ONB-EFIN]].

## Componentes

- Accordion com texto literal (pendente jurídico).
- Checkboxes.

## Estados (loading/empty/error/success)

- **Error**: aceite faltando.
- **Success**: prossegue.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Aceitar | `/api/v1/onboarding/terms/accept` | POST | `{versions:{tecnico, connectme}}` | `{accepted_at}` |

## Regras de negócio críticas

- Consent versionado (hash + data).
- Connect Me: canal LGPD-compliant (opt-in, double opt-in futuro).

## Ligações

- Origem: [[ONB-E6]].
- Destino: [[ONB-EFIN]].

## Gaps/ressalvas

- Texto literal pendente jurídico.

## Links

- [[_moc]]
- [[ONB-E6]]
- [[ONB-EFIN]]
