---
title: ASM-DASH — Dashboard do Módulo Assembleia
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-DASH
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-DASH — Dashboard do Módulo Assembleia

## Finalidade

Tela de entrada do síndico (e administradora vinculada) no módulo Assembleia. Lista todas as assembleias do condomínio segregadas por etapa (em rascunho, publicadas, em andamento ao vivo, encerradas, arquivadas) e serve de launchpad para criar uma nova assembleia ou retomar fluxos em aberto.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §2 (5 blocos) e §12 (diagrama — DASHBOARD → CRIAR ASSEMBLEIA)
- [[../../../../../../90-archive/master-sindico-pdfs-cliente-originais/MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO]] — bloco 1 (Configuração Estrutural)
- [[../../../05-assembleia-live]] — sub-produto canônico
- [[../../../../04-requirements/functional/assembly]] Req 48 (entrada do módulo)

## Persona e ABAC

- **Primária**: `syndic` (role principal do condomínio).
- **Secundária**: `administradora` vinculada ao condomínio via token (§3.3 funcional) — visualização de assembleias em construção e CTA para ações delegadas (validar procurações, subir frações).
- ABAC: `assembly.read`, `assembly.create`, `assembly.list` com `tenant_id = condominium_id`. Morador **não** acessa esta tela (seu ponto de entrada é a timeline / notificação de convocação).

## Fluxo da tela

1. Header com nome do condomínio ativo (dropdown se síndico multicondomínio — máx 15, Req IDN-016).
2. 4 abas contadoras: **Rascunho (n)** · **Publicadas (n)** · **Ao Vivo (n, badge vermelho)** · **Encerradas (n, últimas 90d)**.
3. Cada card mostra: título · tipo (ordinária/extraordinária/implantação) · data 1ª convocação · status edital · % ciência · CTA contextual.
4. CTA global: **[+ Nova assembleia]** → navega para [[ASM-CFG]].
5. Linha do tempo (últimos 12 meses) com eventos críticos (publicações, encerramentos) — link para [[ASM-REL]] relatórios.

## Componentes

- Tabs com contadores dinâmicos (SSE ou polling 30s).
- Cards clicáveis (SolidJS feature-slice `assembly-panel`).
- Botão primário CTA alto-contraste "+ Nova assembleia".
- Empty-state ilustrado se nenhuma assembleia ainda.
- Banner de alerta quando há ciência < 70% a 3 dias da 1ª convocação.

## Estados (loading/empty/error/success)

- **Loading**: skeleton de 3 cards + tabs sem contador.
- **Empty**: "Nenhuma assembleia registrada ainda. Crie a primeira para iniciar a linha do tempo de governança."
- **Error**: banner de erro com retry; fallback de cache local.
- **Success**: lista populada + badge "ao vivo" pulsante se houver assembleia ativa.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Listar assembleias | `/api/v1/condominiums/:id/assemblies` | GET | `?status=draft|published|live|closed` | array Assembly |
| Criar nova | `/api/v1/condominiums/:id/assemblies` | POST | `{type, date}` seed | `{assembly_id}` |
| KPIs do dashboard | `/api/v1/condominiums/:id/assemblies/kpis` | GET | — | `{count_by_status, avg_cience, next_date}` |

## Regras de negócio críticas

- Síndico só vê assembleias do condomínio ao qual tem `membership` ativo (INV tenant isolation).
- Administradora só entra via token externo (§3.3) — se não tiver `assembly.write` permission, CTAs de criação ficam desabilitados.
- Assembleia em `live` **bloqueia** criação de nova assembleia no mesmo condomínio (conflito de estado ao vivo).
- Cache ABAC Redis 5 min, invalidado por webhook ao mudar role/membership.

## Ligações

- Destino: [[ASM-CFG]] (criar), [[ASM-PAUTA]], [[ASM-EDITAL]], [[ASM-PAINEL-SIND]] (quando ao vivo).
- Relacionados: [[../../../05-assembleia-live]], [[../../../../client-canvas/Arquitetura Assembleia]], [[../../../../client-canvas/Fluxo Assembleia Ao Vivo]].

## Gaps/ressalvas

- Filtros avançados (por ano, tipo, status homologação) ficam para M2.
- Export CSV da lista fica M2+ (M1 entrega visualização apenas).

## Links

- [[_moc]]
- [[../../../admin-privilegios]]
- [[../../../../STATE]]
