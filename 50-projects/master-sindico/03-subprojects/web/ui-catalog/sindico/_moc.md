---
title: MOC — UI Catalog Síndico (web) S1-S31
type: moc
tags: [master-sindico, ui-catalog, web, sindico, moc]
project: master-sindico
persona: sindico
stack: web
created: 2026-04-24
---

# MOC — UI Catalog Síndico (web)

Catálogo das 31 telas canônicas da persona Síndico no front web. IDs estáveis S1–S31 conforme `vault/90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas.md` (§2). Cada tela tem spec própria com finalidade, fontes canônicas, persona/ABAC, fluxo, componentes, estados, integração backend, regras críticas, ligações e gaps.

## Tabela completa (S1–S31)

| ID | Título | Sub-produto | Plan tier mínimo | Categoria/Módulo |
|---|---|---|---|---|
| [[S1]] | Entrada da Governança | governanca-institucional | trial | Onboarding |
| [[S2]] | Seletor de Condomínios | governanca-institucional | trial | Navegação |
| [[S3]] | Cadastrar Condomínio (Endereço) | governanca-institucional | trial | Registry |
| [[S4]] | Cadastrar Condomínio (Mandato) | governanca-institucional | trial | Registry |
| [[S5]] | Identidade do Condomínio (ID/QR) | governanca-institucional | trial | Registry |
| [[S6]] | Central de Governança (home 13 cards) | governanca-institucional | trial | Home |
| [[S7]] | Plano Diretor (lista) | governanca-institucional | trial | PD |
| [[S8]] | Nova Ação PD | governanca-institucional | trial | PD |
| [[S9]] | Lista Ações PD (filtros avançados) | governanca-institucional | trial | PD |
| [[S10]] | Detalhe Ação PD (campos + histórico) | governanca-institucional | trial | PD |
| [[S11]] | Linha do Tempo (lista) | governanca-institucional | trial | LT |
| [[S12]] | Criar Atividade LT | governanca-institucional | base | LT |
| [[S13]] | Editar Atividade LT (nova versão) | governanca-institucional | base | LT |
| [[S14]] | Histórico LT | governanca-institucional | base | LT |
| [[S15]] | Eventos (lista + indicadores) | governanca-institucional | base | Eventos |
| [[S16]] | Criar Evento (18+ tipos) | governanca-institucional | base | Eventos |
| [[S17]] | Lista Eventos (submenu filtros) | governanca-institucional | base | Eventos |
| [[S18]] | Connect Me (hub) | connect-me | base | Connect Me |
| [[S19]] | Criar Connect Me (LGPD) | connect-me | base | Connect Me |
| [[S20]] | Connect Me Recebido (M→S) | connect-me | base | Connect Me |
| [[S21]] | Registros de Execução (hub) | governanca-institucional | base | Validações |
| [[S22]] | Detalhe Registro Execução (5 critérios) | governanca-institucional | base | Validações/Reputação |
| [[S23]] | Comunicados (hub) | governanca-institucional | base | Comunicados |
| [[S24]] | Criar Comunicado | governanca-institucional | base | Comunicados |
| [[S25]] | Lista Comunicados | governanca-institucional | base | Comunicados |
| [[S26]] | Validações Pendentes (hub) | governanca-institucional | base | Validações |
| [[S27]] | Dashboard (indicadores agregados) | governanca-institucional | plus | Dashboard |
| [[S28]] | Compliance (entrada — C1) | compliance | trial | Compliance |
| [[S29]] | Termo de Responsabilidade (C2) | compliance | trial | Compliance |
| [[S30]] | Declaração Anual (C2) | compliance | trial | Compliance |
| [[S31]] | Auditoria (C5 — trilha imutável) | compliance | trial | Compliance |

## Mapa por sub-produto

- **governanca-institucional** (24 telas): S1–S17, S21–S27
- **connect-me** (3 telas): S18, S19, S20
- **compliance** (4 telas): S28, S29, S30, S31

## Mapa por plan tier mínimo

- **trial** (criação durante onboarding): S1–S11, S28–S31 — 15 telas
- **base** (operação normal): S12–S26 — 15 telas
- **plus** (dashboards completos): S27 — 1 tela

## Telas-pivô (críticas para arquitetura)

- [[S6]] — única dashboard cross-módulo do síndico (13 cards + indicadores)
- [[S26]] — hub centralizado de inbox de validação (cross-módulo)
- [[S28]] — entrada do módulo Compliance (gateway das C1–C11)

## Sobreposições com módulo Compliance C1-C11

| Tela S | Tela C correspondente | Observação |
|---|---|---|
| [[S28]] | C1 | Painel/entrada |
| [[S29]] | C2 (variante) | Termo Responsabilidade — overlap com S30 |
| [[S30]] | C2 | Declaração Anual canônica |
| [[S31]] | C5 | Trilha de auditoria |

C3, C4, C6, C7, C8, C9, C10, C11 não têm IDs S correspondentes em S1-S31; vivem como sub-páginas dentro do hub S28 ou em rotas dedicadas (`/app/compliance/{signatures|conflicts|amendments|risk|contracts|lgpd|score|dossier}`).

## Ligações cruzadas

- Sub-produtos: [[../../../../00-product/sub-produtos/_moc]]
- Backend requirements: [[../../requirements]]
- Design system: [[../../design-system]]
- Mobile catalog (espelho): [[../../../mobile/ui-catalog]]
- Canvas do cliente: [[../../../../client-canvas/Fluxo Governanca]] · [[../../../../client-canvas/Fluxo Connect Me]] · [[../../../../client-canvas/Fluxo Ciclo Servico]]
- Fontes canônicas:
  - `vault/90-archive/from-development-content-2026-04-23/contents/####################TELA DO MORADOR#####(1).md`
  - `vault/90-archive/from-development-content-2026-04-23/02-Jornadas/Inventario-Telas.md`
  - `vault/90-archive/master-sindico-pdfs-cliente-originais/JORNADA DO SÍNDICO completa.md`
  - `vault/90-archive/from-development-content-2026-04-23/03-Modulos/Compliance-Detalhado.md`

## Gaps agregados (top 5)

1. **Endpoints REST inferidos** em ~80% das telas — backend usa nomenclatura própria (validation_items vs service-records, dashboard agregador vs múltiplas chamadas, etc.). Necessário cruzar com `requirements/` ao final.
2. **Sobreposição S29/S30** (termo de responsabilidade vs declaração anual) — cliente trata como telas separadas mas Compliance-Detalhado.md fala apenas de C2. Decisão produto pendente.
3. **Inconsistência Score Governança** (Q25): 1-10 vs 0-100 — adotar 0-100 (D-061) mas várias fontes legadas ainda usam 1-10.
4. **Quotas Connect Me** divergentes entre fontes (D-067 pendente): matriz funcional vs Decision 10 vs sub-produto destilação. Implementar conforme matriz canônica + flag de feature toggle.
5. **Listas mestre sobrepostas** (Áreas Impactadas .ini vs PDF S8 — 16 itens semânticos vs 26 áreas físicas). Backend precisa unificar enum ou suportar duas dimensões (categoria semântica + área física).

## Links

- [[../../ui-catalog]] — MOC pai do catálogo web
