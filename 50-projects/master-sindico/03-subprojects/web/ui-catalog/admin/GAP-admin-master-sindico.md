---
title: GAP CRÍTICO — Telas Admin Master Síndico ausentes (cliente N/D)
type: note
tags:
  - master-sindico
  - ui-catalog
  - web
  - admin
  - gap
  - bloqueador
project: master-sindico
category: ADM
status: gap
stack: web
created: 2026-04-24T00:00:00.000Z
---

# GAP CRÍTICO — Telas Admin Master Síndico ausentes (cliente N/D)

## Resumo

O **cliente Master Síndico NÃO documentou** telas administrativas do portal interno (ADM-1..ADM-N) em nenhum dos PDFs originais entregues (ONBOARDING DOS PERFIS, MÓDULO DAS TELAS ASSEMBLEIA, ARQUITETURA FUNCIONAL, JORNADAS). A única fonte que menciona funcionalidades admin é o **legado técnico antigo** ([[../../../../../90-archive/from-development-content-2026-04-23/excopo-tecnico-antigo|excopo-tecnico-antigo]] §4.6, janeiro/2026, SOW contratual original), listando 5 **capacidades** — porém sem wireframes, sem campos, sem fluxos detalhados.

## Fonte canônica (legado — 5 capacidades)

Ref literal: [[../../../../../90-archive/from-development-content-2026-04-23/excopo-tecnico-antigo]] §4.6 "Painel Administrativo Global (MasterSíndico - Web Only)":

1. **4.6.1 Dashboard Financeiro** — Visão geral de assinaturas ativas/canceladas; integração com Mercado Pago (ou gateway escolhido — hoje Stripe, [[../../../../02-architecture/adr/0004-stripe-payment-gateway]]); relatórios MRR.
2. **4.6.2 Gestão de Usuários** — Listar usuários (filtros por perfil/plano); bloquear/desbloquear; editar dados em casos excepcionais; histórico de atividade.
3. **4.6.3 Moderação de Conteúdo** — Aprovar/reprovar vídeos denunciados; remover vídeos que violem diretrizes; suspender empresas reincidentes; moderar posts do fórum.
4. **4.6.4 Configurações da Plataforma** — Edição de textos de termos/política; upload de banners; configuração de categorias técnicas; parâmetros globais (janela votação, limites de upload, etc.).
5. **4.6.5 Envio de Comunicados Oficiais (Broadcast Email/SMS)** — Segmentação por perfil/plano; editor texto+CTA; agendamento; log de disparos. **Único canal oficial de massa** (§2.2).

## Decisão de produto (STATE + admin-privilegios)

[[../../../admin-privilegios]] canoniza (D-058, 2026-04-22):

> **Admin Master Síndico NÃO é aplicação separada.** É **role privilegiada (`admin`) transversal**, implementada como **1 privilégio ABAC a mais** (`admin_bypass`) sobre o mesmo backend e as mesmas apps web/mobile. Telas privilegiadas vivem **dentro** dos portais existentes (`sindico-web`, `empresa-web`, `marketing-web`, `morador-web` — hoje consolidados em `web/`), **gated por permissão**.

Sub-papéis operacionais (super-admin, moderador, suporte, financeiro) são **scoped permissions** ABAC, não apps distintas. Ver também:
- **Q28** em [[../../../../../90-archive/from-development-content-2026-04-23/regras-de-negocio/quebras-detectadas|quebras-detectadas]] — admin global.
- **D-062** — admin não auto-registra (provisionado no Zitadel).
- **D-073** — sub-papéis via `admin_profiles` table, não roles Zitadel separadas.

## O que já existe (stubs)

- [[../../../admin-privilegios]] documenta **AD1–AD8** conceitualmente (§4.1), com rotas `/admin/*` e endpoints `/admin/v1/*`.
- ABAC engine + `admin_bypass` funcional em produção (Sprint 1-3 legado).
- `audit_trail` + `lgpd_logs` em produção (parcial).
- MFA TOTP opt-in em Zitadel (movendo para required M1 — [[../../../../02-architecture/adr/0026-admin-mfa-m1]]).

Porém **nenhuma tela web implementada** (M2–M3 conforme [[../../../../10-schedule/_moc]]).

## Gap bloqueador (pós-M1)

Para **M2/M3** (quando começa efetivamente a construir os portais admin), precisamos do cliente:

1. **Wireframes/mockups** de cada tela ADM-1..ADM-N.
2. **Campos exatos** de cada formulário (ex: quais filtros em 4.6.2 Gestão de Usuários? Quais categorias em 4.6.4?).
3. **Fluxo de moderação** com SLA definido (hoje ADR-0024 define 48h manual M1, mas UX final do moderador é genérica).
4. **Segmentação do broadcast** (4.6.5): quais dimensões do segmento são exigidas? Quem assina o comunicado — CEO? DPO?
5. **Dashboard financeiro**: KPIs específicos (além do MRR — LTV, churn, cohort?).

## Perguntas para o cliente (checklist)

- [ ] **Dashboard financeiro**: quais KPIs além de MRR e assinaturas ativas/canceladas? Integração Mercado Pago ainda desejada ou pode ser somente Stripe?
- [ ] **Gestão de usuários**: filtros obrigatórios (além de perfil/plano)? Campos editáveis "em casos excepcionais" — definir quais (nome? e-mail? CPF? unidade?).
- [ ] **Moderação**: fluxo desejado (fila única? pré-moderação vs reativa? SLA por tipo de conteúdo?).
- [ ] **Configurações**: quais parâmetros globais? Lista exata (sabemos "tempo janela votação", "limite vídeos por mês" — o resto?).
- [ ] **Broadcast**: segmentos exigidos (geográfico? plano? última atividade?). Dry-run obrigatório?
- [ ] **4-eyes e MFA**: confirmar aceite do cliente sobre ([[../../../admin-privilegios]] §5 e §6) — rotas críticas exigem step-up + segregação.

## Funcionalidades admin derivadas já identificadas (transversal)

Que existem por consequência da arquitetura transversal, mas sem UI específica cliente:

- Aprovação manual de CNPJ ([[../../../admin-privilegios]] §11) — derivado de [[../onboarding/ONB-E2]].
- Override de inadimplência ([[../assembleia/ASM-INAD]]) — operação delegada, não admin global.
- Adendo administrativo em ata imutável ([[../assembleia/ASM-ENCER]] + ADR-0033).
- Lock 90d bypass via 4-eyes (D-054) — sobre vídeos institucionais ([[../onboarding/ONB-E6]], [[../onboarding/ONB-S5]]).

## Status

**Bloqueador pós-M1**. M1 (Sprint 10, 2026-05-08) entrega as telas de persona (sindico/morador/empresa/parceiro). M2 (Sprints 11-12) começa ADM com ADR-específica + wireframes do cliente.

## Ligações

- [[../../../admin-privilegios]] — canon D-058 + D-054 + D-056.
- [[../../../../../90-archive/from-development-content-2026-04-23/excopo-tecnico-antigo]] §4.6.
- [[../../../../02-architecture/adr/0026-admin-mfa-m1]].
- [[../../../../02-architecture/adr/0024-moderation-cascade]].
- [[../../../../STATE]] — D-054, D-056, D-058.
- [[_moc]]

## Referências rápidas

- Persona Admin: [[../../../../00-product/personas]] §6.
- Sub-papéis: Super Admin · Moderador · Suporte · Financeiro (detalhe em [[../../../admin-privilegios]] §2).
- Ações ABAC canônicas: [[../../../admin-privilegios]] §3.2 matriz exaustiva.
