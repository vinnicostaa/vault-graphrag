---
title: Pendências Fase D — Telas e Jornadas (catálogo UI)
type: pendencias
tags: [master-sindico, fase-d, ui-catalog, screens, pendencias]
project: master-sindico
created: 2026-04-25
updated: 2026-04-25
phase: D-pending
---

# Pendências Fase D — Telas e Jornadas

Lista de Reqs e itens de UX/jornada **delegados pra Fase D** (catálogo UI em `03-subprojects/web/ui-catalog/`), **NÃO** processados em Fase C (2026-04-25 — absorção backend/spec).

> **Princípio Fase C**: regras de negócio + invariantes + EARS formais → `04-requirements/functional/*` (backend); telas/wireframes/UX → `03-subprojects/web/ui-catalog/` (frontend, Fase D).

## Origem das pendências

Identificadas durante Fase C ao processar `_chaos/requirements(6).md` (3936 linhas) + `_chaos/mastersindico-requirements.pdf` (4256 linhas texto).

---

## 1. Síndico Journey (S1–S31, 31 telas)

Origem: `_chaos/requirements(6).md` linhas 3387-3422 + `JORNADA DO SÍNDICO completa.pdf`.

Inclui:
- S1: Login + seleção contexto (multi-condo)
- S2: Dashboard principal (governance score, próximos marcos, validações pendentes)
- S3-S5: Onboarding síndico (6 telas — already specified em IDN-019)
- S6-S10: Cadastro de condomínio + unidades
- S11-S15: Gestão Master Plan (criar atividade, atualizar status, exportar PDF)
- S16-S20: Comunicados, Eventos, Enquetes
- S21-S25: Connect Me, Propostas, Deals
- S26-S31: Assembleia (config, edital, painel presidente, ata, homologação)

## 2. Morador Journey (M1–M15, 15 telas)

Origem: `_chaos/requirements(6).md` linhas 3367-3386 + `JORNADA DO MORADOR.pdf`.

Inclui:
- M1-M4: Onboarding morador (4 telas — IDN-019)
- M5: Feed Timeline read-only
- M6: Comunicados (lista, modal de leitura)
- M7: Eventos (calendário, confirmação)
- M8: Vizinhança (Clube de Benefícios, busca por bairro)
- M9: Cupons (gerar, histórico)
- M10: Banco de Talentos (vídeo-currículo, lista)
- M11-M13: Connect Me Morador → Síndico (one-shot, sem reply)
- M14: Avaliação de gestão (3 perguntas bimestrais — INS-031)
- M15: Assembleia (ciência, check-in, votação app)

## 3. Empresa Journey (E1–E16, 16 telas)

Origem: `_chaos/requirements(6).md` linhas 3423-3443 + `JORNADA DA EMPRESA.pdf`.

Detalhes E1-E14 já em `_chaos/requirements(6).md` linhas 3184-3322 (Company Panel Details). Aplicar em Fase D:
- E1: Home painel empresarial (7 indicadores)
- E2: Connect Me Opportunities (lista + "Tenho Interesse" + "Recusar com motivo")
- E4: Send Proposal (form estruturado)
- E5: My Proposals (lista)
- E7: Closed Deals (lista + estados)
- E8: Termo de Execução (texto canônico já em COM-019)
- E9: Service Execution Registry (form estruturado COM-048)
- E11: Company Evaluations Received (5 critérios — COM-050)
- E12: Dashboard Indicators (E1 expandido)
- E14: Company Compliance (dec anual, LGPD, garantias)
- E15-E16: Banco de Talentos (busca currículos, "Demonstrar Interesse")

## 4. Marketing Agency Journey (MK1–MK8, 8 telas)

Origem: `_chaos/requirements(6).md` linhas 3444-3456 + `JORNADA DA EMPRESA DE MARKETING.pdf`.

D-095 confirma: agência tem login Zitadel próprio + painel admin (8 telas).
- MK1: Login agência
- MK2: Dashboard (empresas atendidas, vídeos publicados, top performers)
- MK3: Empresas (lista de empresas que delegam acesso à agência)
- MK4: Upload de vídeo (em nome de empresa — CNT-014 com escopo limitado)
- MK5: Editor de marketing link (CNT-041 — Marketing Link sponsored)
- MK6: Analytics agregado (D-102 painel admin agência)
- MK7: Convites pendentes
- MK8: Histórico de ações (audit trail visível à agência)

## 5. Assembly Module Screens (AS1–AS8 — Painel Administradora, ~8 telas)

Origem: `_chaos/requirements(6).md` linhas 3457-3462 + `MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO.pdf`.

- AS1: Dashboard administradora (assembleias ativas)
- AS2: Import lista_inadimplentes
- AS3: Validação de procurações (R52)
- AS4: Painel quorum projection
- AS5: Painel presidente (síndico — separa pra UI)
- AS6: Telão presentation
- AS7: Telão operational
- AS8: Pós-assembleia (ata, homologação, scoring)

## 6. Admin Master Síndico (AD1–AD8 — D-134 app dedicada)

D-134 ratifica: app dedicada `apps/admin` (subdomínio `admin.mastersindico.com.br`).

5 módulos do admin (D-134):
1. Dashboard Financeiro (revenue, MRR, churn)
2. Gestão de Usuários (suspender, banir, MFA reset, LGPD review)
3. Moderação (vídeos pendentes, denúncias, content flagged)
4. Configuração de Plataforma (planos, preços, feature flags, ADRs)
5. Broadcast (notificações cross-tenant, comunicados sistema)

Telas estimadas: ~8-15 conforme módulo (M1: dashboard básico; M2-M3: módulos adicionais).

## 7. LMS Screens (~15 telas)

Origem: `_chaos/requirements(6).md` Req 98.1.

- LMS1-LMS3: Catalog (cursos, treinamentos, lives)
- LMS4-LMS6: Player de curso (módulo, aula, quiz)
- LMS7: Certificado emitido (PDF QR validation — CNT-019)
- LMS8-LMS10: Lives (calendário, watch, replay)
- LMS11-LMS13: Fórum (lista tópicos, criar post, thread)
- LMS14-LMS15: Biblioteca (lista, viewer ebook/PDF)

## 8. Vizinhança Screens (VS, VM, VE, VZ — 35 telas total)

Origem: `_chaos/requirements(6).md` Reqs 27.1, 27.2, 27.3.

- **VS1-VS10** (Síndico): dashboard parceria, indicar à comunidade, criar benefício exclusivo, histórico, métricas.
- **VM1-VM7** (Morador): feed por bairro, gerar cupom, histórico, busca por categoria, perfil parceiro.
- **VE1-VE6** (Empresa — relacionamento B2B com Vizinhança).
- **VZ1-VZ6** (Parceiro): dashboard, criar promoção, palavra do dia, métricas, gerenciar exclusivas.

---

## Estimativa total Fase D

| Persona | Telas | Marco principal |
|---|---|---|
| Síndico | 31 | M1-M3 |
| Morador | 15 | M1-M3 |
| Empresa | 16 | M1-M2 |
| Marketing Agency | 8 | M3 |
| Assembly Admin | ~8 | M3 |
| Admin Master Síndico | ~8-15 | M1-M3 |
| LMS | ~15 | M3 |
| Vizinhança (VS/VM/VE/VZ) | ~35 | M2-M3 |

**Total estimado**: ~136-143 telas em catalog UI.

## Procedimento sugerido Fase D

1. Para cada persona: criar pasta em `03-subprojects/web/ui-catalog/{persona}/`.
2. Para cada tela: criar `Sxx-nome-tela.md` com layout, componentes, estados, navegação, links a Reqs canônicos (ex: "implementa COM-019, COM-024").
3. Wireframes ASCII ou Excalidraw (sem alta fidelidade — esse é spec não design final).
4. Linkar para `D-Set Tokens UI` (D-108 paleta OKLCH canônica).
5. Cross-link com componentes em `packages/ui` quando existir.

## Cross-references

- [[internal-requirements/_moc]] — fontes originais arquivadas.
- [[../../../../50-projects/master-sindico/04-requirements/functional/_moc]] — Reqs canônicos backend (Fase C).
- [[../../../../50-projects/master-sindico/STATE]] — D-108 (paleta OKLCH), D-119 (stack web), D-134 (app admin).
- [[../../../../50-projects/master-sindico/03-subprojects/web/ui-catalog]] — destino canônico Fase D (a popular).
