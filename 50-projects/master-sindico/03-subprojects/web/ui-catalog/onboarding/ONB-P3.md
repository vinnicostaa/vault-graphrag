---
title: ONB-P3 — Parceiro Contato e Visual
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, parceiro]
project: master-sindico
persona: parceiro
category: ONB
screen_id: ONB-P3
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-P3 — Parceiro · Contato e Visual (Tela 3)

## Finalidade

Contatos operacionais do parceiro + dados financeiros + até 3 fotos do estabelecimento para vitrine visual na página "Vizinhança" do morador.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 3 Parceiro

## Persona e ABAC

- `local_company` pré-ativação.
- ABAC: `identity.draft.write`.

## Fluxo da tela

1. Campos:
   - **E-mail comercial**
   - **Telefone comercial** (E.164)
   - **Endereço com CEP** (ViaCEP)
   - **Financeiro**: nome · telefone · e-mail
   - **Até 3 fotos** (galeria) — 5MB cada, JPG/PNG.
2. Botão **[Continuar]** → [[ONB-P4]].

## Componentes

- Uploader galeria com preview.
- Inputs mascarados.

## Estados (loading/empty/error/success)

- **Error**: CEP inválido · > 3 fotos.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:3, email_com, tel, endereco, cep, financeiro:{...}, photos[≤3]}` | 204 |
| Upload foto | `/api/v1/portfolios/upload` | POST | multipart | `{photo_url}` |

## Regras de negócio críticas

- Até 3 fotos (regra produto).
- Fotos entram em moderação leve (nudez/violência — M2 automático; M1 manual reativo).

## Ligações

- Origem: [[ONB-P2]].
- Destino: [[ONB-P4]].

## Gaps/ressalvas

- Galeria > 3 fotos em plano pago futuro: backlog.

## Links

- [[_moc]]
- [[ONB-P2]]
- [[ONB-P4]]
