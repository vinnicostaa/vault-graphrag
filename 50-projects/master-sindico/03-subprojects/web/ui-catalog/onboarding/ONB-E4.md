---
title: ONB-E4 — Empresa Atuação no Mercado
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, empresa]
project: master-sindico
persona: empresa
category: ONB
screen_id: ONB-E4
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-E4 — Empresa · Atuação no Mercado (Tela 4)

## Finalidade

Definição geográfica e categórica da atuação: cidades/estados, categoria principal, subcategorias (até 5). Usado para matchmaking com condomínios/síndicos que buscam fornecedores ([[ASM-PAUTA]] tipo `escolha_fornecedor`).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — TELA 4 Empresa
- [[../../../../04-requirements/registration-data]] §3.1

## Persona e ABAC

- `enterprise` pré-ativação.
- ABAC: `institutional.company.set_categories`.

## Fluxo da tela

1. Campos:
   - **Cidades e Estados de atuação** (multi-select).
   - **Categoria principal** (enum da plataforma — gerenciado por Admin CNT-029).
   - **Subcategorias de atuação** (até 5 itens da mesma taxonomia).
2. Botão **[Continuar]** → [[ONB-E5]].

## Componentes

- Multi-select cidades com busca.
- Tree-select categoria/subcategorias.

## Estados (loading/empty/error/success)

- **Error**: > 5 subcategorias · categoria faltando.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Salvar draft | `/api/v1/onboarding/draft` | PATCH | `{step:4, cities[], categoria, subcategorias[≤5]}` | 204 |
| Listar categorias | `/api/v1/categories` | GET | — | array Category |

## Regras de negócio críticas

- **Máximo 5 subcategorias** (regra produto).
- Taxonomia controlada pelo Admin (novas categorias requerem aprovação — [[../../../admin-privilegios]]).

## Ligações

- Origem: [[ONB-E3]].
- Destino: [[ONB-E5]].

## Gaps/ressalvas

- Categoria fica travada após ativação: mudança por ticket.

## Links

- [[_moc]]
- [[ONB-E3]]
- [[ONB-E5]]
