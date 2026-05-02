---
title: ONB-P2 — Parceiro Identidade do Estabelecimento
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, parceiro]
project: master-sindico
persona: parceiro
category: ONB
screen_id: ONB-P2
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-P2 — Parceiro da Vizinhança · Identidade do Estabelecimento (Tela 2)

## Finalidade

Identificação do parceiro local: logo, nome do estabelecimento, **CNPJ OU CPF** (Decision 12 / D-064 aceita CPF para MEI/autônomo), data aniversário, representante legal.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 2 Parceiro
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/Jornada do Parceiro da Vizinhança]]

## Persona e ABAC

- **Primária**: `local_company` pré-ativação.
- ABAC: `identity.draft.write`.

## Fluxo da tela

1. Mensagem principal (literal): **"Seu negócio faz parte do cotidiano do condomínio."**
2. Campos:
   - **Logo** (upload 5MB).
   - **Nome do estabelecimento**.
   - **CNPJ OU CPF** (toggle; Decision 12 aceita CPF).
   - **Data de aniversário**.
   - **Nome do representante legal**.
   - **E-mail do representante legal**.
3. Botão **[Continuar]** → [[ONB-P3]].

## Componentes

- Toggle PF/PJ.
- Inputs mascarados conforme seleção.

## Estados (loading/empty/error/success)

- **Error**: CPF/CNPJ inválido, duplicado.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:2, logo, name, document:{type:"cpf"|"cnpj", value}, aniversario, repr}` | 204 |

## Regras de negócio críticas

- **Decision 12 / D-064**: CPF aceito (MEI, autônomo, comércio familiar). CNPJ também aceito.
- Documento imutável (IDN-029).

## Ligações

- Origem: [[ONB-01-identificacao-inicial]].
- Destino: [[ONB-P3]].

## Gaps/ressalvas

- Verificação de CPF ativo (SPC/Serasa): fora do escopo M1.

## Links

- [[_moc]]
- [[ONB-P3]]
