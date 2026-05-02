---
title: ONB-SFIN — Síndico Confirmação Final
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, sindico]
project: master-sindico
persona: sindico
category: ONB
screen_id: ONB-SFIN
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-SFIN — Síndico · Confirmação Final

## Finalidade

Tela de sucesso do onboarding do síndico. Celebra conclusão, avisa sobre revisões que podem ser necessárias (vídeo institucional N2/N3 entra em fila de moderação), e direciona para dashboard.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — Tela Final (literal)

## Persona e ABAC

- `syndic` ativação pronta.
- ABAC: `identity.activate` + `institutional.syndic.activate`.

## Fluxo da tela

1. Mensagem principal (literal): **"Cadastro concluído com sucesso!"**
2. Nota (literal): _"Algumas informações podem requerer revisão antes de se tornarem visíveis."_
3. Resumo: nome, tipo, condomínios vinculados (se houver), badge plano (N1/N2/N3).
4. Botão **[Acessar plataforma]** → dashboard síndico.

## Componentes

- Ilustração + card resumo.
- Banner "Vídeo institucional em moderação (48h SLA)" se N2/N3.

## Estados (loading/empty/error/success)

- **Loading**: migração Redis → Postgres (`SindicoProfile`).
- **Success**: redirect.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Finalizar | `/api/v1/onboarding/complete` | POST | `{draft_id}` | `{user_id, role:"syndic", tier, status}` |

## Regras de negócio críticas

- Migração para `institutional.SindicoProfile` (bounded context institutional).
- Vídeo institucional (N2/N3) entra em `moderation_state=pending`; SLA 48h manual M1 (ADR-0024).
- Evento `syndic.registered` via NATS para billing/content reagirem.
- Plano inicial (N1/N2/N3) derivado do parâmetro URL ou do checkout Stripe.

## Ligações

- Origem: [[ONB-S6]].
- Destino: dashboard.
- Relacionados: [[../../../admin-privilegios]] §9 (moderação vídeo).

## Gaps/ressalvas

- Se pagamento (Stripe) ainda pendente, status `pending_payment` em vez de `active`.

## Links

- [[_moc]]
- [[ONB-S6]]
