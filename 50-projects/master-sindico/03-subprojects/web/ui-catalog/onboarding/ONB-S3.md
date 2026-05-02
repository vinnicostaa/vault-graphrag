---
title: ONB-S3 — Síndico Sua Atuação
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, sindico]
project: master-sindico
persona: sindico
category: ONB
screen_id: ONB-S3
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-S3 — Síndico · Sua Atuação (Tela 3)

## Finalidade

Campos de atuação profissional: endereço profissional, cidade/UF de atuação, tipo de síndico, tempo de atuação, quantidade de condomínios sob gestão. Contextualiza experiência.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 3 Síndico
- [[../../../../04-requirements/registration-data]] §2.1

## Persona e ABAC

- `syndic` pré-ativação.
- ABAC: `identity.draft.write`.

## Fluxo da tela

1. Mensagem principal (literal): **"Conte como você atua hoje."**
2. Mensagem de apoio (literal): _"Essas informações ajudam a contextualizar sua experiência e o alcance da sua gestão."_
3. Campos:
   - **Endereço profissional** (CEP + lookup).
   - **Cidade e Estado de atuação** (multi-select).
   - **Tipo de síndico**: `Morador OU Profissional` (ou `Interino`).
   - **Tempo de atuação** (anos 0-60).
   - **Quantidade de condomínios sob gestão** (0-15 — limite IDN-).
4. Botão **[Continuar]** → [[ONB-S4]].

## Componentes

- Inputs com validação numérica.
- Multi-select cidades.
- Radio tipo síndico.

## Estados (loading/empty/error/success)

- **Error**: anos > 60 · condomínios > 15.
- **Success**: draft salvo.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:3, address, cep, cities, sindico_type, years, condominiums_qty}` | 204 |

## Regras de negócio críticas

- Limite **15 condomínios por conta** (IDN-016) — exceções via suporte.
- Tipo síndico pode mudar pós-ativação via ticket (imutável no cadastro inicial para governança).
- Anos de experiência auto-declarado (verificação opcional M3+).

## Ligações

- Origem: [[ONB-S2]].
- Destino: [[ONB-S4]].

## Gaps/ressalvas

- Campo "administradoras vinculadas" fica em [[ONB-S5]] (presença profissional) — diferente do dado "empresa síndica" institucional.

## Links

- [[_moc]]
- [[ONB-S2]]
- [[ONB-S4]]
