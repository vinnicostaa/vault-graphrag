---
title: ONB-E3 — Empresa Contatos e Organização
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, empresa]
project: master-sindico
persona: empresa
category: ONB
screen_id: ONB-E3
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-E3 — Empresa · Contatos e Organização (Tela 3)

## Finalidade

Captura canais de comunicação comerciais + endereço sede + dados financeiros. Insumos para billing (Stripe), contratos e fluxos oficiais.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 3 Empresa
- [[../../../../04-requirements/registration-data]] §3.1

## Persona e ABAC

- `enterprise` pré-ativação.
- ABAC: `identity.draft.write`.

## Fluxo da tela

1. Campos:
   - **E-mail comercial**
   - **Telefone comercial** (E.164)
   - **Endereço comercial com CEP** (ViaCEP)
   - **Financeiro**: nome · telefone · e-mail
2. Botão **[Continuar]** → [[ONB-E4]].

## Componentes

- Inputs com masks BR.
- Agrupamento visual "Contato comercial" vs "Financeiro".

## Estados (loading/empty/error/success)

- **Error**: CEP inválido · e-mail duplicado.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:3, email_com, tel_com, endereco, cep, fin:{nome,tel,email}}` | 204 |

## Regras de negócio críticas

- Dados financeiros usados em emissão de NF e conciliação Stripe.
- CEP obrigatório (ViaCEP agnóstico XD-018).

## Ligações

- Origem: [[ONB-E2]].
- Destino: [[ONB-E4]].

## Gaps/ressalvas

- Múltiplos endereços (matriz + filiais): fora do escopo M1.

## Links

- [[_moc]]
- [[ONB-E2]]
- [[ONB-E4]]
