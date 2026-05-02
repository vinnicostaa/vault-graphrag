---
title: "Master Síndico — Portal do Arquiteto"
version: "4.0 (Systematic Edition)"
date: 2026-03-23
status: "ativo"
total_requisitos: 104
total_criterios: "770+"
total_enums: 36
total_telas: 114
tags:
  - mastersindico
  - arquitetura
  - ddd
  - portal
---

# 🏗️ Master Síndico — Portal do Arquiteto

> **Status da Stack:** Golang + SolidJS + Flutter + Zitadel + NATS + OpenSearch
> **Deadline 90%:** 20/07/2026 | **Entrega Final:** 08/08/2026

---

## 🧭 Navegação Sistemática

### 1. 🎯 Estratégia & Visão Geral
*Documentos de alto nível e definições de ecossistema.*

- [[Arquitetura do Ecossistema|🏛️ Arquitetura do Ecossistema]]: Domínios, Tenants e Actors.
- [[🗺️ Mapa de Artefatos Visuais|🗺️ Mapa de Artefatos Visuais]]: Índice de todos os 23 Canvas do projeto.
- [[Decisões e Pendências|⚖️ Decisões e Pendências]]: Histórico de ADRs e Gaps ativos.

### 2. 🧱 Arquitetura Técnica (Go Edition)
*O blueprint de engenharia e infraestrutura.*

- [[System Architecture|🏗️ System Architecture]]: Visão técnica do Monolito Modular.
- [[System Architecture - Infrastructure|⚙️ Infrastructure]]: Zitadel, NATS, OpenSearch e AWS.
- [[System Architecture - API Design|📡 API Design]]: Endpoints REST e WebSockets.
- [[Enums e Listas Mestre|📚 Enums e Listas Mestre]]: Dicionário de dados (36+ listas).

### 3. 🛡️ Domínios de Negócio (Vertical Slices)
*Requisitos detalhados e regras de cada contexto.*

- [[Domínio - Identidade|🆔 Identidade]]: Auth, Billing, Onboarding e ABAC.
- [[Domínio - Institucional|🏛️ Institucional]]: Condomínio, Unidades e Governança (LT/PD).
- [[Domínio - Comercial|💼 Comercial]]: Connect Me, Marketplace e Vizinhança.
- [[Domínio - Conteúdo|📚 Conteúdo]]: LMS, Vídeos e Busca.
- [[Domínio - Assembleia|🏟️ Assembleia]]: Motor Real-time e Votações.

### 4. 🚀 Plano de Execução
*Sprints, entregas e definição de sucesso.*

- [[Sprint 1 Identity|Sprint 1]]: Identidade & Fundação.
- [[2026-04-15 Entrega|Milestone 1]]: 15/04/2026.
- [[Definition of Done|✅ Definition of Done]]: Checklist sistemático por slice.

---

## 🏰 Visão Macro do Sistema

![[Arquitetura de Domínios.canvas]]

---

## 🔴 Bloqueadores Críticos (Gaps)

| # | Gap | Impacto | Status |
|---|-----|---------|--------|
| 1 | **Painel Admin MS** | Gestão global de tenants e moderação | 🔴 Pendente |
| 2 | **Paywall Logic** | Hard block vs Soft block no Go Middleware | 🔴 Pendente |
| 3 | **Cotas Connect Me** | Unidade temporal (Ano vs Mês) | 🔴 Pendente |

---

## 🛠️ Stack Tecnológica Confirmada

- **Backend:** Go (Gin/Echo) + GORM/Ent + NATS.
- **Frontend:** SolidJS (Rsbuild) + Flutter (Mobile).
- **Identity:** Zitadel (OIDC) + Casbin (ABAC).
- **Search:** OpenSearch.
- **Infra:** AWS (RDS, Redis, S3, SES).

**Última Atualização:** 23/03/2026
