---
title: ASM-SIM — Simulador de Quórum
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-SIM
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-SIM — Simulador de Quórum

## Finalidade

Painel analítico que projeta o quórum por item da pauta cruzando: total de unidades, total de frações, ciência registrada, confirmações de participação, procurações aprovadas/pendentes, inadimplência. Orienta o síndico se há viabilidade de deliberação antes da 1ª convocação.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §4.16 (Simulador de quórum)
- [[../../../../04-requirements/functional/assembly]] Req 57.0

## Persona e ABAC

- **Primária**: `syndic`.
- **Secundária**: `administradora`.
- ABAC: `assembly.quorum.read`.

## Fluxo da tela

1. Cards resumo no topo:
   - Total de unidades / Total de frações
   - % ciência ([[ASM-CIEN]])
   - % confirmação de presença
   - Nº procurações pendentes / aprovadas
   - Nº unidades inaptas (inadimplentes)
2. Tabela por item da pauta: **item · tipo · quórum necessário (%) · projetado (%)** · status (viável / incerto / inviável).
3. Gráfico de barras comparativo.
4. Botão **[Recalcular]** (manual refresh; auto-refresh a cada 30s).

## Componentes

- KPI cards.
- TanStack Table com coloração semafórica por status.
- Chart (recharts/visx).

## Estados (loading/empty/error/success)

- **Empty**: "Dados insuficientes — aguarde planilha de fração" ([[ASM-FRAC]]).
- **Success**: cards verdes quando viável, amarelo marginal, vermelho inviável.

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Simular | `/api/v1/assemblies/:id/quorum-simulation` | GET | — | `{totals, per_item:[{id, required, projected, status}]}` |

## Regras de negócio críticas

- Cálculo depende de [[ASM-FRAC]] importada e de [[ASM-INAD]] aplicada.
- Procurações pendentes **não** contam no projetado; aprovadas sim.
- Projeção é **estimativa** — não substitui quórum real aferido em tempo real.
- Unidades inaptas não somam no projetado.

## Ligações

- Origem: [[ASM-FRAC]], [[ASM-INAD]], [[ASM-PROC-VAL]].
- Destino: [[ASM-PAINEL-SIND]] (transição para live).
- Relacionados: [[../../../../client-canvas/Arquitetura Assembleia]].

## Gaps/ressalvas

- Exibir projeção considerando piores/melhores cenários: M2.
- Simular cenários hipotéticos (ex: "se mais 10 unidades confirmarem"): backlog.

## Links

- [[_moc]]
- [[ASM-FRAC]]
- [[ASM-PAINEL-SIND]]
