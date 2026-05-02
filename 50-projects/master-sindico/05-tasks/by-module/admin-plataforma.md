---
title: Tasks Admin Plataforma Master Síndico (D-113)
type: note
tags:
  - task-list
  - master-sindico
  - by-module
project: master-sindico
milestone_target: m2-m3
status: backlog
created: 2026-04-24T00:00:00.000Z
---

# Tasks Admin Plataforma Master Síndico

Backlog de tasks para construir painel admin da plataforma Master Síndico. Ordenado por prioridade. Canonizado via [[../../STATE#d-113|D-113]] (2026-04-24).

**Objetivo**: Admin MS (role `admin` Zitadel, 2-5 pessoas) tem painel próprio para operar a plataforma. NÃO é o admin-agência (MK1-MK8) nem o admin-administradora ([[./administradoras-condominiais]]).

**Princípio**: em vez de construir tudo de uma vez, absorver padrões de grandes empresas e entregar por prioridade em tasks.

## Benchmark de grandes empresas

| Empresa | Padrão aproveitado | Aplicação Master Síndico |
|---|---|---|
| **Google Cloud Console** | Navegação hierárquica projeto→recurso; filtros persistentes; bulk ops | Navegação Condomínio→Usuários→Ban em lote |
| **Stripe Dashboard** | Cards métricas em grid; filtros de data global; export CSV universal | Dashboard financeiro MRR+churn; export CSV de todas tabelas |
| **Cloudflare Dashboard** | Tabs secundárias por recurso; logs em tempo real; actions em modal | Moderação conteúdo com logs live |
| **Linear Admin** | ABAC granular por member; audit log inline em cada ação; keyboard shortcuts | Permissões granulares; audit ao lado de cada ação destrutiva |
| **Vercel Dashboard** | Deployments em timeline; rollback 1-clique; preview branches | Config de feature flags com rollback 1-clique |
| **GitHub Enterprise Admin** | User list com badges (Owner, Member, Outside); ban com motivo | Gestão de usuários com roles + motivo em cada ação |

## Tasks priorizadas

### M2 — Início (2026-06)

#### T-ADM-01 — Dashboard Financeiro (read-only MRR + churn + cohorts)

**Prioridade**: P0 (bloqueador para operar negócio)
**Esforço**: 3-4 dias (web) + 1-2 dias (backend endpoint agregador)
**Benchmark primário**: Stripe Dashboard

**Escopo**:
- Cards métricas grid responsive:
  - MRR atual + Δ mês anterior (↑↓)
  - Novos assinantes / churn do mês
  - ARPU (Average Revenue Per User)
  - Distribuição por plano (trial/base/plus/pro) stacked bar
  - Cohort mensal retention (12 meses)
- Filtros globais: período (últimos 7d/30d/90d/12m/custom) + plano
- Export CSV universal em todas tabelas
- Read-only (sem ações destrutivas)

**Telas**:
- [[../../03-subprojects/web/ui-catalog/admin/_moc|admin/ADM-DASH-01 (a criar)]] (criar)
- backend endpoint: `/api/v1/admin/metrics/financial?period=30d`

**Dependências**:
- Stripe webhook já em prod (✅ Sprint 9)
- Agregador Go para queries caras (cache Redis 1h)

#### T-ADM-02 — Gestão de Usuários (listar + ban/reativar + editar role)

**Prioridade**: P0 (compliance LGPD exige)
**Esforço**: 4-5 dias (web) + 1 dia (backend endpoints + audit)
**Benchmark primário**: GitHub Enterprise Admin + Linear

**Escopo**:
- Tabela usuários com colunas: email, nome, CPF mascarado, role, plan_tier, status (active/banned/soft-deleted), last_login, created_at
- Filtros: role, plano, status, search por email/CPF/nome
- Actions per-row:
  - **Ban** (modal com motivo obrigatório + confirmação dupla + audit log) — implementa soft-delete universal D-101
  - **Reativar** (se soft-deleted)
  - **Editar role** (dropdown + confirmação — audit log com `before/after`)
  - **Ver detalhes** (perfil, mandatos, assinaturas, Connect Me, LGPD exports)
- Audit log inline — painel lateral mostra últimas ações sobre esse usuário

**Telas**:
- [[../../03-subprojects/web/ui-catalog/admin/_moc|admin/ADM-USERS-01 (a criar)]] listagem
- [[../../03-subprojects/web/ui-catalog/admin/_moc|admin/ADM-USERS-02 (a criar)]] detalhe

**Backend endpoints**:
- `GET /api/v1/admin/users?filter=...&page=N`
- `POST /api/v1/admin/users/:id/ban`
- `POST /api/v1/admin/users/:id/reactivate`
- `PATCH /api/v1/admin/users/:id/role`
- `GET /api/v1/admin/users/:id/audit-log`

### M2 — Fim (2026-06 late)

#### T-ADM-03 — Moderação de Conteúdo

**Prioridade**: P1 (LGPD + abuse prevention)
**Esforço**: 5-6 dias
**Benchmark primário**: YouTube Studio + Twitter Admin

**Escopo**:
- 3 filas separadas: **Vídeos**, **Comunicados**, **Posts Fórum**
- Cada fila tem filtros + bulk actions (approve/reject multiple)
- Actions: aprovar, rejeitar com motivo, ocultar temporariamente, banir autor (redirecionar para T-ADM-02)
- Report queue (denúncias de usuários)

**Telas**:
- [[../../03-subprojects/web/ui-catalog/admin/_moc|admin/ADM-MOD-01 (a criar)]] overview 3 filas
- ADM-MOD-02/03/04 por tipo

### M3 — Início (2026-07)

#### T-ADM-04 — Configurações da Plataforma (feature flags + plan caps)

**Prioridade**: P1
**Esforço**: 3 dias
**Benchmark primário**: Vercel + Cloudflare

**Escopo**:
- Feature flags globais (habilitar/desabilitar LMS live por condomínio, BT público, etc)
- Plan caps (editar limites de quota por plano — ex: Empresa Plus cap E→E configurável via D-111)
- Config de providers (Mux tier, Livekit max rooms, Stripe test vs live)
- **Destrutivo**: modal com confirmação + audit log (mudança de plan cap afeta todos os usuários desse tier)

**Telas**:
- [[../../03-subprojects/web/ui-catalog/admin/_moc|admin/ADM-CONFIG-01 (a criar)]]

### M3 — Fim (2026-07 late)

#### T-ADM-05 — Broadcast Email/SMS

**Prioridade**: P2
**Esforço**: 4-5 dias
**Benchmark primário**: Mailchimp + SendGrid

**Escopo**:
- Editor de template (variáveis dinâmicas: `{{user_name}}`, `{{condominium_name}}`)
- Segmentação (por role, plano, condomínio, status)
- Preview antes de enviar
- Dry-run (ver estimativa de destinatários sem enviar)
- Scheduling (agora / data-hora futura)
- Audit log completo: quem enviou, quando, para quantos, taxa abertura (se Resend webhook implementado)

## Tarefas paralelas transversais

### T-ADM-00a — Layout base admin (shell)

**Prioridade**: P0 (bloqueador de todas as ADM-*)
**Esforço**: 2 dias
**Benchmark**: Linear layout

**Escopo**:
- Sidebar navegação (Dashboard, Usuários, Moderação, Config, Broadcast)
- Topbar com admin avatar + logout + ambiente (dev/staging/prod)
- Layout 2-column responsivo
- Keyboard shortcuts básicos (⌘K busca global)

### T-ADM-00b — Audit log universal

**Prioridade**: P0 (LGPD + compliance)
**Esforço**: 2 dias
**Benchmark**: Linear audit log

**Escopo**:
- Toda action destrutiva (ban, editar role, mudar plan cap, mudar feature flag) registra em `admin_audit_log` table
- Formato: `(admin_user_id, action_type, target_type, target_id, before_state, after_state, reason, ip, user_agent, timestamp)`
- Append-only (não pode editar/deletar)
- Retenção LGPD: 5 anos
- Export CSV para compliance

## Observações

- **Todas as tasks ADM-*** consumem design-tokens canônicos ([[../../02-architecture/design-tokens]]).
- **Componentes reutilizáveis** já planejados (feature-plan-2026-04-24): DataTable, FilterBar, ExportButton, ConfirmContextModal.
- **Dual review obrigatório**: toda ação destrutiva requer confirmação dupla (modal com campo texto "digite BANIR para confirmar").
- **A11y**: WCAG 2.1 AA obrigatório; alguns fluxos destrutivos AAA.

## Links

- [[../_moc]]
- [[../../STATE#d-113|D-113]]
- [[../../03-subprojects/admin-privilegios]]
- [[../../03-subprojects/web/ui-catalog/admin/GAP-admin-master-sindico]]
- [[administradoras-condominiais]] — análogo para admin-administradora
