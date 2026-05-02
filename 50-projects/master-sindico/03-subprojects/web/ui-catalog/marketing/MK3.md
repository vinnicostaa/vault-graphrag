---
title: MK3 — Gerenciar Empresa
type: ui-screen
tags: [master-sindico, ui-catalog, web, marketing, marketing-agencias, conteudo-videos]
project: master-sindico
persona: marketing
category: MK
screen_id: MK3
sub_produto: marketing-agencias
plan_requirement: any
status: specification
stack: web
created: 2026-04-23
---

# MK3 — Gerenciar Empresa

## Finalidade
Tela de **switch de contexto** para a empresa selecionada. Funciona como sub-home da agência dentro de uma empresa específica. Apresenta 3 cards: Publicar vídeo, Biblioteca, Dashboard da empresa. Toda ação dali em diante opera no escopo daquela empresa (ABAC scoped).

## Fonte canônica
- `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DA EMPRESA DE MARKETING.md` (TELA MK3 — Gerenciar Empresa)

## Persona e ABAC
- **Persona primária**: Marketing
- **Plan tier mínimo**: any
- **ABAC action**: `marketing.empresa.context_switch`
- **Restrições**: agência precisa ter vínculo ativo com a empresa (status = Ativa em [[MK2]]).

## Fluxo da tela
1. Agência em [[MK2]] → clica [Gerenciar empresa] em uma linha.
2. Sistema valida grants ABAC para aquela empresa.
3. Sistema renderiza header de contexto: "Gerenciando: {Empresa X}" + breadcrumb.
4. Sistema mostra 3 cards: Publicar vídeo, Biblioteca, Dashboard da empresa.
5. Agência clica em um card → navega para a tela respectiva (com `empresa_id` no contexto).
6. Botão [Voltar a empresas atendidas] retorna a [[MK2]].

## Componentes
- `ContextHeader` (logo empresa + nome fantasia + breadcrumb + botão sair)
- `CardGrid3` (Publicar vídeo / Biblioteca / Dashboard)
- `RecentActivityFeed` (últimos vídeos publicados — opcional)
- `EmptyState`

## Cards (3 — fonte PDF MK3)
1. **Publicar vídeo** → [[MK4]] no contexto da empresa
2. **Biblioteca de vídeos** → [[MK5]]
3. **Dashboard da empresa** → [[MK6]]

## Estados
- **Loading**: skeleton 3 cards
- **Empty**: empresa nova sem vídeos — cards funcionam normalmente
- **Error**: toast retry
- **Permission-denied**: agência tenta acessar empresa cujo vínculo foi revogado → redireciona para [[MK2]] com toast "Acesso revogado pela empresa"

## Integração com backend
| Ação UI | Endpoint | Método | Retorno |
|---|---|---|---|
| Validar contexto | `/api/v1/marketing/empresas/:id/context` | GET | { granted: bool, empresa: {...} } |
| Atividade recente | `/api/v1/content/videos?empresa_id=:id&limit=5` | GET | Video[] |

Use case `marketing.SwitchContext` + ABAC validation com `agency_id` + `empresa_id`.

## Regras de negócio críticas
- **Switch de contexto** é operação ABAC: cada chamada subsequente carrega `empresa_id` no header
- **Revogação imediata**: se empresa revogar acesso enquanto agência está nesta tela, próxima ação bloqueia com 403 → redireciona para [[MK2]]
- **Auditoria**: switch é registrado (timestamp, agency_user, empresa_id, action_attempted)
- **Limitação ABAC** mantida: dentro do contexto da empresa, agência ainda NÃO acessa Connect Me, Timeline, dados síndico/morador

## Ligações
- **Tela origem**: [[MK2]]
- **Tela destino**: [[MK4]], [[MK5]], [[MK6]], [[MK2]] (voltar)
- **Sub-produto**: [[../../../../00-product/sub-produtos/10-marketing-agencias]]
- **Backend**: [[../../requirements]]
- **Inventário**: [[../../../../../../90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas]]

## Gaps/ressalvas
- "Atividade recente" não está no PDF — sugerido para UX (mostra últimos 5 vídeos com thumbnail)
- Comportamento se vínculo está "Convite enviado" (não aceito): tela bloqueada — definir mensagem
- Múltiplos usuários da agência atuando simultaneamente na mesma empresa: sem regra de coordenação (locking de edição) — definir

## Links
- [[../../../../_moc]]
- [[../../../../CLAUDE]]
- [[../../../../client-canvas/Modelo ABAC]]
