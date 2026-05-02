---
title: ONB-S6 — Síndico Registros e Responsabilidade (Aceites)
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, sindico]
project: master-sindico
persona: sindico
category: ONB
screen_id: ONB-S6
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-S6 — Síndico · Registros e Responsabilidade (Tela 6)

## Finalidade

**3 aceites obrigatórios** para ativar perfil do síndico: Termo de Registro de Gestão e Linha do Tempo, Termo de Indicadores/Selos, Termo de Links Temporários.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 6 Síndico (literal)

## Persona e ABAC

- `syndic` pré-ativação.
- ABAC: `identity.terms.accept` + `institutional.syndic.activate`.

## Fluxo da tela

1. Mensagem principal (literal): **"A plataforma registra. A responsabilidade é sua."**
2. **3 aceites obrigatórios** (checkboxes + texto expansível):
   - **Termo de Registro de Gestão e Linha do Tempo** — autoriza registro imutável de decisões como histórico público.
   - **Termo de Indicadores, Selos e Certificação Própria** — autodeclaração + disclaimer.
   - **Termo de Links Temporários e Compartilhamento** — regras de compartilhamento de perfil/timeline.
3. Botão **[Finalizar cadastro]** → [[ONB-SFIN]].

## Componentes

- Accordion com texto literal (pendente jurídico cliente).
- Checkbox versão + data.

## Estados (loading/empty/error/success)

- **Error**: aceite faltando.
- **Success**: prossegue para confirmação.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Aceitar 3 termos | `/api/v1/onboarding/terms/accept` | POST | `{versions:{gestao, indicadores, links}}` | `{accepted_at}` |

## Regras de negócio críticas

- Versões de termo versionadas por hash (LGPD consent versioning).
- `lgpd_logs` com entradas por termo + legal basis (consent/legitimate_interest).
- Retenção 5 anos (INV-091).

## Ligações

- Origem: [[ONB-S5]].
- Destino: [[ONB-SFIN]].
- Relacionados: [[../../../../04-requirements/compliance-lgpd]].

## Gaps/ressalvas

- Texto **literal** dos 3 termos pendente — jurídico Master Síndico.

## Links

- [[_moc]]
- [[ONB-S5]]
- [[ONB-SFIN]]
