---
title: MK1 — Painel da Agência
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias]
project: master-sindico
persona: marketing
category: MK
screen_id: MK1
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK1 — Painel da Agência

## Finalidade
Home da agência de marketing logada. Centro de operação dedicado à gestão de conteúdos institucionais das empresas que delegaram acesso à agência. Apresenta cards de acesso aos módulos e notificações de convites/marketing links recebidos.

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK1 — Painel da Agência)
- `vault/50-projects/master-sindico/00-product/sub-produtos/10-marketing-agencias.md` (D-059/D-060 — agência é sub-produto interno)

## Persona e ABAC
- **Persona primária**: Marketing (role `marketing` — Zitadel próprio, conta dedicada)
- **Plan tier mínimo**: any (agência não tem tier; é convidada por empresa Plus/Pro)
- **ABAC action**: `marketing.dashboard.read`
- **Restrições**: agência só acessa empresas que aceitaram convite via [[../empresa/E14]]; sem acesso a Connect Me, Timeline, dados síndico/morador.

## Fluxo da tela
1. Agência loga (Zitadel próprio).
2. Sistema carrega notificações pendentes em paralelo (convites empresa, Marketing Links recebidos).
3. Sistema renderiza header com nome da agência + avatar.
4. Cards são exibidos em grid responsivo (5 cards principais).
5. Notificações aparecem como toast persistente / badge nos cards.
6. Agência clica em qualquer card → navega.

## Componentes
- `HeaderAgencia` (logo + nome + responsável)
- `NotificationBar` (convites pendentes empresa + Marketing Links novos)
- `CardGrid5` (Empresas atendidas / Publicar vídeos / Biblioteca / Dashboard / Marketing Link recebido)
- `EmptyState` (agência nova sem empresa)

## Cards (5 — fonte JORNADA DA EMPRESA DE MARKETING.md)
1. **Empresas atendidas** → [[MK2]]
2. **Publicar vídeos** → atalho a [[MK4]] (mas exige seleção empresa via [[MK3]])
3. **Biblioteca de vídeos** → atalho a [[MK5]]
4. **Dashboard de desempenho** → [[MK7]]
5. **Marketing Link recebido** → [[MK8]]

## Notificações (fonte PDF)
- Empresa enviou convite (aparece em [[MK2]] após aceite)
- Empresa enviou Marketing Link (aparece em [[MK8]])

## Estados
- **Loading**: skeleton dos 5 cards
- **Empty**: agência sem empresas atendidas — placeholder "Aguarde convite de empresa"
- **Error**: toast retry
- **Pending invites**: badge vermelho com contador no card "Empresas atendidas"

## Integração com backend
| Ação UI | Endpoint | Método | Retorno |
|---|---|---|---|
| Carregar painel | `/api/v1/marketing/dashboard` | GET | { pending_invites, marketing_links_new, empresas_count } |
| Listar notificações | `/api/v1/marketing/notifications` | GET | Notification[] |

Use case `marketing.GetDashboard`. Inferido — módulo `marketing/` não existe explicitamente no backend (gap §10-marketing-agencias.md).

## Regras de negócio críticas
- **D-059/D-060**: Agência tem login próprio Zitadel (role `marketing`); NÃO é actor delegado via impersonation
- **Sem Connect Me**: agência não envia nem recebe Connect Me (apenas Marketing Link)
- **Limitações ABAC** (gates frontend + backend):
  - ❌ Connect Me / Registros execução / Comunicados técnicos / Timeline / Dados síndico / Dados morador / Dashboard governança
- **Multi-empresa**: agência atende N empresas (todas que delegaram acesso)
- **Status agência por empresa**: Convite enviado / Ativa / Encerrada (controlado em [[../empresa/E14]])

## Ligações
- **Tela origem**: Login (auth)
- **Tela destino**: [[MK2]], [[MK7]], [[MK8]]
- **Sub-produto**: [[../../../../00-product/sub-produtos/10-marketing-agencias]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- Backend Go ainda não tem módulo `marketing/` dedicado — feature lives em `content/` (videos) + `commercial/` (marketing_link_requests) + `identity/` (role marketing). Gap documentado em [[../../../../00-product/sub-produtos/10-marketing-agencias]] §18.
- "Cards de empresa" no painel pode mostrar prévia das últimas atividades (vídeos publicados recentemente) — não especificado, sugerir
- Onboarding agência: primeiro acesso após aceitar convite — fluxo não está nesta entrega

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Dominio Conteudo]]
