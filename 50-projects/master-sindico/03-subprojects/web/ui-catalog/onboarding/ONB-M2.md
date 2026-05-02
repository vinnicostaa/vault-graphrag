---
title: ONB-M2 — Morador Identidade
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, morador]
project: master-sindico
persona: morador
category: ONB
screen_id: ONB-M2
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-M2 — Morador · Identidade (Tela 2)

## Finalidade

Captura dados pessoais do morador: foto, nome completo, data de nascimento, telefone, CPF. É o conjunto mínimo que leva o morador a `visibility_ready` quando combinado com vínculo de condomínio ([[ONB-M3]]).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 2 Morador
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/ONBOARDING DOS PERFIS]]
- [[../../../../04-requirements/registration-data]] §1.1

## Persona e ABAC

- **Primária**: `resident` (pré-ativação).
- ABAC: `identity.draft.write`.

## Fluxo da tela

1. Mensagem principal (literal): **"Você faz parte da vida do condomínio."**
2. Mensagem de apoio (literal): _"Na Master Síndico, o morador acompanha a gestão, entende o que está sendo feito e participa de forma mais consciente."_
3. Campos:
   - **Foto de perfil** (upload, 5MB max, JPG/PNG/WebP, ≥200×200)
   - **Nome completo** (pré-preenchido de [[ONB-01-identificacao-inicial]])
   - **Data de nascimento** (≥ 18 anos)
   - **Telefone** (+55 DDD formato E.164)
   - **CPF** (mask + dígito verificador)
4. Botão **[Continuar]** → [[ONB-M3]].

## Componentes

- Uploader de avatar com crop.
- Input date (calendário).
- Input tel mascarado.
- Input CPF mascarado.

## Estados (loading/empty/error/success)

- **Error**: CPF inválido (dígito) · CPF já cadastrado · idade < 18.
- **Success**: draft salvo automaticamente.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:2, photo_url, full_name, birth, phone, cpf}` | 204 |
| Upload foto | `/api/v1/media/avatars/upload` | POST | multipart | `{photo_url}` |
| Validar CPF | `/api/v1/identity/validate-cpf` | POST | `{cpf}` | `{valid, duplicate}` |

## Regras de negócio críticas

- **CPF imutável** após ativação (IDN-029).
- CPF **único** — duplicado bloqueia.
- Idade mínima 18 (IDN-).
- Validação de dígito no front + back (IDN-017).
- LGPD: CPF, data nascimento, telefone entram em `lgpd_logs` com `legal_basis = execution_of_contract`.
- Mensagens pt-BR canônicas em [[../../../../04-requirements/registration-data]] §1.4.

## Ligações

- Origem: [[ONB-01-identificacao-inicial]].
- Destino: [[ONB-M3]].
- Relacionados: [[../../../../04-requirements/registration-data]] §1.

## Gaps/ressalvas

- Upload de documento (RG/CNH): **fora do escopo M1** — backlog para anti-fraude M2+.
- Morador dependente tem fluxo ligeiramente diferente (adicionado por titular) — não passa por este onboarding; será convidado via link interno.

## Links

- [[_moc]]
- [[ONB-M3]]
- [[../../../../04-requirements/registration-data]]
