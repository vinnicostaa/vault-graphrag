---
title: MK8 — Marketing Link Recebido
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias]
project: master-sindico
persona: marketing
category: MK
screen_id: MK8
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK8 — Marketing Link Recebido

## Finalidade
Lista de Marketing Links recebidos pela agência (formulários enviados por empresas via [[../empresa/E16]] manifestando intenção de contratação de serviços de marketing). Agência pode entrar em contato e marcar como atendido, mas vínculo formal só ocorre se a empresa convidar via [[../empresa/E14]].

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK8 — Marketing Link Recebido + REGRAS)
- `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Connect-Me-4-direcoes.md` (Marketing Link — clarificação)

## Persona e ABAC
- **Persona primária**: Marketing (Administrador da agência)
- **Plan tier mínimo**: any
- **ABAC action**: `commercial.marketing_link.list_received`
- **Restrições**: lista filtrada por `agency_id`.

## Fluxo da tela
1. Agência em [[MK1]] → clica card "Marketing Link recebido".
2. Sistema lista solicitações pendentes (mais recentes primeiro).
3. Cada linha: Empresa, Responsável, Telefone, Email, Tipo de apoio solicitado, Prioridade, Data da solicitação, Descrição.
4. Agência clica [Entrar em contato] → atalho mailto/tel com dados pré-preenchidos.
5. Agência clica [Marcar como atendido] → status muda para `atendido`.
6. Filtros: prioridade, tipo de apoio, status, período.

## Componentes
- `MarketingLinkTable` (linhas com badge prioridade)
- `BadgePriority` (Imediato → vermelho / 30d → laranja / Sem prazo → cinza)
- `BadgeStatus` (sent / viewed / atendido / rejected)
- `ContactActionGroup` (mailto, tel, marcar atendido)
- `FilterBar` (prioridade, tipo, status)
- `EmptyState`

## Campos exibidos por linha (fonte PDF MK8)
- Empresa (nome fantasia)
- Responsável
- Telefone
- Email
- Tipo de apoio solicitado (1 das 6 opções)
- Prioridade (Imediato / 30 dias / Sem prazo)
- Data da solicitação
- Descrição da necessidade (resumida — expandir no clique)
- Status atual

## Estados
- **Loading**: skeleton tabela
- **Empty**: "Nenhum Marketing Link recebido ainda. Quando empresas preencherem o formulário [[../empresa/E16]], as solicitações aparecerão aqui."
- **Error**: toast retry
- **Atendido**: linha cinza com badge "Atendido em DD/MM/AAAA"
- **Filtered empty**: "Nenhuma solicitação para os filtros aplicados"

## Integração com backend
| Ação UI | Endpoint | Método | Payload |
|---|---|---|---|
| Listar Marketing Links | `/api/v1/commercial/marketing-link/received` | GET | MarketingLinkRequest[] |
| Marcar como atendido | `/api/v1/commercial/marketing-link/:id/attended` | POST | - |
| Marcar como visualizado | `/api/v1/commercial/marketing-link/:id/view` | POST | - (auto ao abrir) |

Use cases `commercial.ListReceivedMarketingLinks` + `commercial.MarkAttended` + tabela `marketing_link_requests`.

## Regras de negócio críticas
- **NÃO cria vínculo automático** entre empresa e agência (regra dura — fonte F4 + PDF MK8)
- **Vínculo formal** só por convite via [[../empresa/E14]]
- **Sem revelação cruzada**: dados de contato da empresa estão visíveis desde o início (não há reveal LGPD-logado como Connect Me)
- **Prioridade "Imediato"** dispara notificação push à agência
- **Não tem quota** (não é volume sensitive)
- **Anti-spam**: empresa não pode enviar > N Marketing Links para a mesma agência em janela de tempo (a definir, sugerido 1/30d)
- **Auditoria**: visualização + atendimento registrados (timestamp, agency_user)

## Ligações
- **Tela origem**: [[MK1]]
- **Tela destino**: detalhe da solicitação (modal expansão), [[../empresa/E14]] (se agência for convidada formalmente — fluxo paralelo iniciado pela empresa)
- **Sub-produto**: [[../../../../00-product/sub-produtos/10-marketing-agencias]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- "Status" do Marketing Link: 6 estados definidos em F4 (`sent / viewed / proposal_sent / atendido / accepted / rejected`) — UI cobre principais, transições completas não modeladas
- "Resposta da agência" (proposta enviada via plataforma): PDF não menciona — fluxo é externo (telefone/email); plataforma só registra atendimento
- Anti-spam por (empresa, agência): regra concreta pendente — sugerir 1 solicitação por 30 dias

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Dominio Comercial]]
