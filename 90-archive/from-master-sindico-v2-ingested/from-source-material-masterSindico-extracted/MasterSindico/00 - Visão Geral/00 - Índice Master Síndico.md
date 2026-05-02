---
title: "Master Síndico — Índice de Requisitos"
version: "3.0"
date: 2026-03-10
status: "em-revisão"
total_requisitos: 104
total_criterios: "770+"
total_enums: 36
total_telas: 114
documentos_analisados: "23/23"
decisoes_resolvidas: 6
decisoes_pendentes: 6
tags:
  - mastersindico
  - requisitos
  - arquitetura
  - ddd
  - indice
---

# 🏗️ Master Síndico — Índice de Requisitos

> **Breadcrumbs:** 🏠 Início > 📄 00 - Índice Master Síndico
> **104 requisitos** · **770+ critérios de aceite** · **36+ enums** · **114 telas mapeadas**

---

## 📋 Visão Geral do Ecossistema

O **Master Síndico** é um ecossistema digital de governança condominial aplicada. Não é um app único — é uma **rede de domínios de negócio integrados** que atendem síndicos, moradores, empresas, agências de marketing e parceiros de vizinhança.

### 🎯 Problema Central

> Condomínios operam sem rastreabilidade institucional, sem blindagem profissional para síndicos, e sem governança aplicada. Empresas prestadoras não têm canal controlado para se conectar a condomínios. Moradores não têm visibilidade sobre a gestão.

### 💡 Solução

Uma plataforma **multi-tenant** com governança baseada em:

- **Linha do Tempo imutável** — append-only, memória institucional, após publicação é imutável (apenas adendo)
- **Plano Diretor reativo** — status só muda quando atividade vinculada é publicada na Timeline (event-driven)
- **Validações centralizadas** — empresa → síndico → publicação (síndico é gatekeeper do tenant)
- **Compliance robusto** — Termo de Responsabilidade, Declaração Anual, Auditoria, LGPD
- **Marketplace unidirecional** — Connect Me com LGPD by design (dados revelados após aceite)
- **Arquitetura event-driven** — comunicação cross-tenant via barramento de eventos

---

## 🗺️ Mapa de Navegação

### Domínios de Requisitos

| Domínio | Requisitos | Telas | Status |
|---------|-----------|-------|--------|
| [[Domínio - Identidade]] | Req 1–17 | M1–M15, S1–S6 | ✅ Completo |
| [[Domínio - Comercial]] | Req 18–27.3 | E1–E16, S7–S31, VS1–VS10, VE1–VE6, VZ1–VZ6, VM1–VM7 | ✅ Completo |
| [[Domínio - Conteúdo]] | Req 29–32, 97–98.1 | LMS1–LMS15 | ✅ Completo |
| [[Domínio - Assembleia]] | Req 48–60.12 | 15+ telas assembleia | ✅ Completo |
| [[Domínio - Cross-Domain e Integrações]] | Req 33–47, 61–67, 102–103 | — | ✅ Completo |

### Referência

| Documento | Conteúdo |
|-----------|----------|
| [[Arquitetura do Ecossistema]] | High-domains, tenants, actors, fluxos |
| [[Enums e Listas Mestre]] | 36+ listas de referência normalizadas |
| [[Decisões Pendentes]] | 11 decisões (6 resolvidas, 5 pendentes) |
| [[Status do Documento]] | Métricas, cobertura, gaps explícitos |

---

## 🏰 Arquitetura de Domínios

![[Arquitetura de Domínios.canvas]]

---

## 👥 Tipos de Tenant e Actors

### Modelo de Tenants

![[Modelo de Tenants.canvas]]

### Regras de Tenant

| Tipo | Quem opera | Isolamento | Multi-user |
|------|-----------|------------|------------|
| **Condomínio** | Síndico (owner), Morador (participante), Empresa Síndica (delegado) | Completo — dados isolados por condomínio | Sim — 1 síndico + N moradores |
| **Empresa** | Admin (owner), Operador Técnico (restrito) | Completo — dados da empresa isolados | Sim — admin + operadores |
| **Agência** | Actor delegado da empresa | Sem tenant próprio — acessa via token da empresa | Não — 1 agência por empresa |
| **Parceiro** | Login independente | Sem tenant — entidade independente | Não — 1 parceiro |

---

## 📊 Requisitos Não-Funcionais (Visão Geral)

| Categoria | Requisito | Detalhes | Req # |
|-----------|-----------|----------|-------|
| **Segurança** | 1 dispositivo por conta | Fingerprint + IP tracking | Req 102 |
| **Segurança** | LGPD by design | Consentimento, log de acesso, direito ao esquecimento | Req 1 |
| **Segurança** | Rate limiting | Brute force + CAPTCHA | Req 102 |
| **Performance** | Tempo de resposta | < 200ms para APIs críticas | — |
| **Performance** | Assembleia ao vivo | WebSocket, real-time updates | Req 48 |
| **Acessibilidade** | WCAG 2.1 AA | Contraste, navegação por teclado | — |
| **Integração** | 7 serviços externos | Mux, Mercado Pago, SES, Twilio, S3, ViaCEP, OAuth | — |
| **Compliance** | LGPD | Consentimento explícito, anonimização, direito ao esquecimento | Req 1, 19 |
| **Visibilidade** | Regras por plano | 25% preview para tiers baixos, acesso completo para Pro | Req 103 |

---

## 🔴 Gaps Explícitos (Sem Documentação Suficiente)

> [!CAUTION]
> Os itens abaixo **NÃO têm documentação suficiente** e precisam ser resolvidos antes da implementação. Nenhuma suposição foi feita — cada item requer input do cliente.

| # | Gap | Impacto | Bloqueia | Decision |
|---|-----|---------|----------|----------|
| 1 | **Painel Admin MasterSíndico** | Define domínio operacional acima de tudo | Hierarquia de domínios | Decision 8 |
| 2 | **Regras de Paywall** | Hard block vs soft block (warning + grace period) | Lógica de frontend | Decision 4 |
| 3 | **Banco de Talentos** | Entra ou não no escopo atual? | Planejamento de sprints | Decision 6 |
| 4 | **Mobile App** | Quais features no mobile? Push, camera, QR? | Arquitetura responsiva | Decision 10 |
| 5 | **Connect Me quotas** | /ano ou /mês? Matriz diz /ano, requirements /mês | Regra de billing | — |
| 6 | **Limite de condomínios** | 12 (regra 5 do PDF) ou 15 (tela S2 do PDF)? | Validação de backend | — |
| 7 | **Currículo: vínculos** | Últimos 3 (visualização) ou 5 (cadastro)? | Schema do banco | Decision 6 |
| 8 | **Status do Plano Diretor** | 7 (PDF) ou 11 (.ini)? | Enum e validação | — |
| 9 | **Morador-Síndico** | Formulário unidirecional ou ticket system? | Arquitetura Connect Me | Decision 11 |

---

## 🛠️ Stack Tecnológica

![[Stack Tecnologica.canvas]]

---

## 📅 Informações do Projeto

| Campo | Valor |
|-------|-------|
| **Cliente** | João — Master Síndico |
| **Início do Projeto** | 2026 |
| **Documentos Analisados** | 23/23 (100%) |
| **Documento Faltante** | Painel Admin |
| **Arquivo descartado** | `TELA DO MORADOR.ini` (PDFs são fonte de verdade) |
| **Última Revisão** | 10/03/2026 |
| **Próximo Passo** | Obter decisões pendentes do cliente |
