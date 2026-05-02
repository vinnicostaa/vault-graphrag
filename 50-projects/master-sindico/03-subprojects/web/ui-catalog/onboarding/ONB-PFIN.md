---
title: ONB-PFIN — Parceiro Confirmação Final
type: ui-screen
tags: [master-sindico, ui-catalog, web, onboarding, parceiro]
project: master-sindico
persona: parceiro
category: ONB
screen_id: ONB-PFIN
sub_produto: onboarding
status: specification
stack: web
created: 2026-04-24
---

# ONB-PFIN — Parceiro · Confirmação Final

## Finalidade

Celebra conclusão do cadastro do parceiro da vizinhança. Pronto para publicar primeira promoção e aparecer na aba "Vizinhança" dos moradores do bairro.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Onboarding-por-Persona]] — Tela Final

## Persona e ABAC

- `local_company` ativação.
- ABAC: `institutional.local_company.activate`.

## Fluxo da tela

1. Mensagem principal (literal): **"Cadastro concluído com sucesso!"**
2. Nota (literal): _"Algumas informações podem requerer revisão antes de se tornarem visíveis."_
3. Resumo: nome estabelecimento, documento mascarado, bairro/cidade.
4. Botão **[Acessar plataforma]** → dashboard parceiro.

## Componentes

- Ilustração + card + CTA.

## Estados (loading/empty/error/success)

- **Loading**: migração Redis → `institutional.LocalCompanyProfile`.
- **Success**: redirect.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Finalizar | `/api/v1/onboarding/complete` | POST | `{draft_id}` | `{user_id, role:"local_company", status}` |

## Regras de negócio críticas

- Fotos entram em fila de moderação leve (48h SLA manual).
- Evento `local_company.registered` via NATS.

## Ligações

- Origem: [[ONB-P4]].
- Destino: dashboard parceiro.

## Gaps/ressalvas

- Dashboard parceiro (pós-cadastro) não especificada ainda.

## Links

- [[_moc]]
- [[ONB-P4]]
