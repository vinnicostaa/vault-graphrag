---
title: ONB-E2 — Empresa Identidade Institucional
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, empresa]
project: master-sindico
persona: empresa
category: ONB
screen_id: ONB-E2
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-E2 — Empresa · Identidade Institucional (Tela 2)

## Finalidade

Identificação institucional da empresa: logo, razão social, CNPJ, data aniversário, representante legal. Insumo para validação automática de CNPJ (via adapter Receita M2+; validador de dígito M1).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 2 Empresa
- [[../../../../04-requirements/registration-data]] §3.1

## Persona e ABAC

- **Primária**: `enterprise` pré-ativação.
- ABAC: `identity.draft.write`.

## Fluxo da tela

1. Campos:
   - **Logo** (upload, 5MB, ≥200×200).
   - **Razão social** (non-empty).
   - **Nome fantasia** (opcional).
   - **CNPJ** (mask + dígito verificador).
   - **Data de aniversário da empresa** (data passada).
   - **Nome do representante legal**.
   - **E-mail do representante legal**.
2. Botão **[Continuar]** → [[ONB-E3]].

## Componentes

- Uploader logo.
- Input CNPJ mascarado.
- Validação online (ReceitaWS fallback M2+).

## Estados (loading/empty/error/success)

- **Error**: CNPJ inválido · CNPJ já cadastrado · e-mail inválido.
- **Success**: draft salvo.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:2, logo, razao, fantasia, cnpj, aniversario, repr:{nome,email}}` | 204 |
| Validar CNPJ | `/api/v1/identity/validate-cnpj` | POST | `{cnpj}` | `{valid, duplicate, status_rf?}` |

## Regras de negócio críticas

- CNPJ imutável após ativação (IDN-029).
- Validação dígito + Receita Federal stub (M1) / adapter real (M2+).
- Razão social altera apenas via suporte (lei).
- Se CNPJ falha validação automática → fila manual admin ([[../../../admin-privilegios]] §11).

## Ligações

- Origem: [[ONB-01-identificacao-inicial]].
- Destino: [[ONB-E3]].
- Relacionados: [[../../../admin-privilegios]] §11 (aprovação manual CNPJ).

## Gaps/ressalvas

- Adapter Receita Federal oficial: M2+.

## Links

- [[_moc]]
- [[ONB-E3]]
