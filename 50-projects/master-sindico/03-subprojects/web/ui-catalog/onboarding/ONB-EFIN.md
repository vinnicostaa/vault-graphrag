---
title: ONB-EFIN — Empresa Confirmação Final
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, empresa]
project: master-sindico
persona: empresa
category: ONB
screen_id: ONB-EFIN
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-EFIN — Empresa · Confirmação Final

## Finalidade

Celebra conclusão do cadastro da empresa. Avisa que conteúdo (vídeos) entra em moderação e que validação CNPJ pode demorar (fila admin).

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — Tela Final

## Persona e ABAC

- `enterprise` ativação.
- ABAC: `institutional.company.activate`.

## Fluxo da tela

1. Mensagem principal (literal): **"Cadastro concluído com sucesso!"**
2. Nota (literal): _"Algumas informações podem requerer revisão antes de se tornarem visíveis."_
3. Resumo: razão social, CNPJ mascarado, plano, status moderação.
4. Botão **[Acessar plataforma]** → dashboard empresa.

## Componentes

- Ilustração + card resumo + banner moderação.

## Estados (loading/empty/error/success)

- **Loading**: migração Redis → `commercial.EnterpriseProfile`.
- **Success**: redirect.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Finalizar | `/api/v1/onboarding/complete` | POST | `{draft_id}` | `{user_id, role:"enterprise", tier, cnpj_status}` |

## Regras de negócio críticas

- Se CNPJ em fila admin → status `pending_manual_review`; usuário pode navegar mas ações limitadas.
- Vídeo institucional em `moderation_state=pending` (SLA 48h manual).
- Evento `enterprise.registered` via NATS para billing/content.

## Ligações

- Origem: [[ONB-E7]].
- Destino: dashboard empresa.
- Relacionados: [[../../../admin-privilegios]] §11.

## Gaps/ressalvas

- Dashboard empresa (tela pós-cadastro) ainda não especificada.

## Links

- [[_moc]]
- [[ONB-E7]]
