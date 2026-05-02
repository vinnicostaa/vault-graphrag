---
title: ONB-S2 — Síndico Identidade de Gestão
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, sindico]
project: master-sindico
persona: sindico
category: ONB
screen_id: ONB-S2
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-S2 — Síndico · Identidade de Gestão (Tela 2)

## Finalidade

Captura dados de identificação profissional do síndico: foto profissional, nome, nascimento, e-mail profissional, telefone, CPF. É a base da persona pública do síndico na plataforma e da linha do tempo de gestão.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 2 Síndico
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]]
- [[../../../../04-requirements/registration-data]] §2.1

## Persona e ABAC

- **Primária**: `syndic` pré-ativação.
- ABAC: `identity.draft.write`.

## Fluxo da tela

1. Mensagem principal (literal): **"Ser síndico é assumir responsabilidade."**
2. Mensagem de apoio (literal): _"Aqui, sua atuação não fica restrita ao momento. Ela é registrada, organizada e se transforma em histórico de gestão."_
3. Campos:
   - **Foto profissional** (upload 5MB, ≥400×400)
   - **Nome completo**
   - **Data de nascimento**
   - **E-mail profissional**
   - **Telefone** E.164
   - **CPF**
4. Botão **[Continuar]** → [[ONB-S3]].

## Componentes

- Uploader avatar profissional.
- Inputs com masks.
- Hint "Email profissional (preferível ao pessoal)".

## Estados (loading/empty/error/success)

- **Error**: CPF inválido/duplicado, idade < 18, e-mail inválido.
- **Success**: draft salvo.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:2, photo, name, birth, prof_email, phone, cpf}` | 204 |

## Regras de negócio críticas

- CPF imutável (IDN-029) + único.
- E-mail profissional pode diferir do e-mail da [[ONB-01-identificacao-inicial]]; Zitadel valida o da tela 1 mas guarda o profissional como dado de perfil.
- Mensagens canônicas em [[../../../../04-requirements/registration-data]] §2.5.

## Ligações

- Origem: [[ONB-01-identificacao-inicial]].
- Destino: [[ONB-S3]].
- Relacionados: [[../../../../04-requirements/registration-data]] §2.

## Gaps/ressalvas

- Nível (N1/N2/N3) não é escolhido aqui — derivado do plano Stripe (ou parametrizado na URL §2.5).

## Links

- [[_moc]]
- [[ONB-S3]]
