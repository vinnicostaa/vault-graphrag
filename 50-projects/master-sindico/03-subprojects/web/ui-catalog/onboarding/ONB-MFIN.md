---
title: ONB-MFIN — Morador Confirmação Final
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, morador]
project: master-sindico
persona: morador
category: ONB
screen_id: ONB-MFIN
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-MFIN — Morador · Confirmação Final

## Finalidade

Tela de sucesso do onboarding do morador. Celebra conclusão, avisa sobre validações pendentes (ex: aprovação de membership pelo síndico) e direciona para home da plataforma.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — Tela Final (literal)

## Persona e ABAC

- **Primária**: `resident` ativação pronta.
- ABAC: `identity.activate` (migra draft Redis → persistência definitiva).

## Fluxo da tela

1. Mensagem principal (literal): **"Cadastro concluído com sucesso!"**
2. Nota (literal): _"Algumas informações podem requerer revisão antes de se tornarem visíveis."_
3. Resumo do cadastro (nome, condomínio, status de membership).
4. Botão **[Acessar plataforma]** → redireciona para home autenticada.

## Componentes

- Ilustração/ícone de sucesso.
- Card resumo.
- CTA primário.

## Estados (loading/empty/error/success)

- **Loading**: "Estamos finalizando seu cadastro…" enquanto migra de Redis para Postgres.
- **Success**: redireciona.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Finalizar | `/api/v1/onboarding/complete` | POST | `{draft_id}` | `{user_id, role, status}` |

## Regras de negócio críticas

- **Migração Redis → Postgres**: dados migram para tabelas do bounded context `identity`, `institutional` (ResidentProfile).
- E-mail de boas-vindas disparado via `IEmailProvider`.
- Status final pode ser `active` ou `incomplete` (se membership pendente).
- Evento NATS `user.registered` publicado para outros BCs reagirem (billing trial, content default feeds).

## Ligações

- Origem: [[ONB-M4]].
- Destino: home morador.
- Relacionados: [[../../../../client-canvas/Fluxo Registro]].

## Gaps/ressalvas

- Se e-mail ainda não validado (Zitadel), UX pede validação antes de acessar; hoje tolerante 24h.

## Links

- [[_moc]]
- [[ONB-M4]]
- [[../../../../client-canvas/Fluxo Registro]]
