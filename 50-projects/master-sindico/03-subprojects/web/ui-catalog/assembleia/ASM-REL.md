---
title: ASM-REL — 6 Relatórios Pós-Assembleia
type: ui-screen
tags: [master-sindico, ui-catalog, web, assembleia]
project: master-sindico
persona: sindico
secondary_persona: administradora
category: ASM
screen_id: ASM-REL
sub_produto: assembleia-live
status: specification
stack: web
created: 2026-04-24
---

# ASM-REL — Relatórios Pós-Assembleia (6 tipos)

## Finalidade

Hub de relatórios consolidados após encerramento da assembleia, com 6 tipos canônicos: Presença · Votos · Procurações · Trilha de auditoria · Decisões · Consolidado (§8). Todos com export PDF/CSV. Linha do tempo integrada.

## Fonte canônica

- [[../../../../../../90-archive/from-development-content-2026-04-23/03-Modulos/Assembleia-Live-Funcional]] §8 (Pós-Assembleia e Histórico)
- [[../../../../04-requirements/functional/assembly]] Req 60

## Persona e ABAC

- **Primária**: `syndic` · `administradora`.
- **Secundária morador**: pode ver relatórios de sua assembleia em versão resumida (não a trilha de auditoria completa).
- ABAC: `assembly.report.read` (operadores) / `assembly.report.read_self` (morador).

## Fluxo da tela

1. Dashboard com 6 cards, um por relatório:
   - **Presença** — lista unidades/presença/horário/modalidade.
   - **Votos** — voto por item + unidade + peso.
   - **Procurações** — outorgadas e validadas.
   - **Trilha de auditoria** — eventos append-only (INV-088; admin vê extra — [[../../../admin-privilegios]] §4.2).
   - **Decisões** — status final de cada item + reason.
   - **Consolidado** — sumário executivo + Score de Transparência.
2. Cada card: view inline + export PDF + export CSV (exceto auditoria = JSON).
3. Linha do tempo embutida (§8): assembleia · edital · pauta · resultados · relatórios.

## Componentes

- Cards clicáveis com preview.
- Geradores de PDF lazy-loaded.
- Timeline component.

## Estados (loading/empty/error/success)

- **Loading**: skeletons por card.
- **Success**: badge "exportado em X formato".

## Integração com backend

| Ação | Endpoint | Método | Payload | Retorno |
|---|---|---|---|---|
| Baixar presença | `/api/v1/assemblies/:id/reports/attendance` | GET | `?format=pdf|csv` | binary |
| Baixar votos | `/api/v1/assemblies/:id/reports/votes` | GET | `?format=pdf|csv` | binary |
| Baixar procurações | `/api/v1/assemblies/:id/reports/proxies` | GET | — | binary |
| Baixar auditoria | `/api/v1/assemblies/:id/reports/audit` | GET | `?format=json|csv` | binary |
| Baixar decisões | `/api/v1/assemblies/:id/reports/decisions` | GET | — | binary |
| Consolidado + Score | `/api/v1/assemblies/:id/reports/summary` | GET | — | `{pdf, score_components[10]}` |
| Timeline | `/api/v1/assemblies/:id/timeline` | GET | — | array Event |

## Regras de negócio críticas

- **Score de Transparência** (§9 — Req 60.7): 10 componentes (% ciência, % presença, % votantes, % leitura pauta, % docs, completude auditoria, regularidade homologação, cumprimento pauta, tempo médio item, % itens c/ documentação prévia).
- Auditoria: admin vê colunas extras (actor_role, IP, trace_id) — [[../../../admin-privilegios]] §4.2.
- PII mascarada em exports (INV-092).
- Retenção: relatórios disponíveis por 5 anos (ligado à retenção audit_trail).
- Export em PDF usa template com hash SHA-256 para integridade.

## Ligações

- Origem: [[ASM-ENCER]].
- Destino: histórico do condomínio / governance score.
- Relacionados: [[../../../admin-privilegios]] §7 (audit trail), [[../../../../client-canvas/Arquitetura Assembleia]].

## Gaps/ressalvas

- Relatório personalizado (BI) **fora do escopo M1** (§4.6 excopo-tecnico-antigo confirma).
- Comparativo entre assembleias do mesmo condomínio: backlog.

## Links

- [[_moc]]
- [[ASM-ENCER]]
- [[../../../admin-privilegios]]
