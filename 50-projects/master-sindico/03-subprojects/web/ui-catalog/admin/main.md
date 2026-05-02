---
title: Admin Panel — apps/admin (UI Spec)
type: ui-spec
tags: [ui-catalog, master-sindico, admin, dashboard, moderation]
bc: cross
source: _chaos/20-ADMIN-PANEL.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 20 — Painel Administrativo Global (Master Síndico)

> **Origem**: `_chaos/20-ADMIN-PANEL.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).
> **⚠️ D-134 (2026-04-25 — REVOGA D-058)**: Admin agora é **app dedicada `apps/admin`** no monorepo web (porta a definir, ex: 3006). NÃO mais embutida em `cms` sob `/admin/*`. Esta spec descreve a app dedicada. As rotas `/admin/*` aqui referenciadas mapeiam para a raiz da app `apps/admin/` (ex: `apps/admin/dashboard` → `admin.mastersindico.com.br/dashboard`).

> **Sprint 4** · **Exclusivo Web** (não existe no mobile) · Acesso: Apenas Admin Master Síndico (Dono da Plataforma).
> Dependências: [[../../../../02-architecture/design-tokens|design-tokens]] · [[../shell/app-shell|app-shell]] · [[../cross/private-feedback|private-feedback]].

---

## 1. Contexto e regras de negócio

### Restrição crítica de acesso
- **EXCLUSIVO WEB**: este painel NÃO existe no aplicativo mobile.
- **Apenas o role `admin`** (Master Síndico Dono da Plataforma) tem acesso.
- Qualquer outro role tentando acessar app `apps/admin` → **HTTP 403** + redirect para home.
- Middleware ABAC do TanStack Router bloqueia renderização da rota — **nenhum componente do admin carrega** para perfis não autorizados.
- MFA obrigatório (M2+ — ADR-0026).
- 4-eyes em ações destrutivas (M3+).

### Escopo funcional (5 módulos)

1. **Dashboard Financeiro** (4.6.1)
2. **Gestão de Usuários** (4.6.2)
3. **Moderação de Conteúdo** (4.6.3)
4. **Configurações da Plataforma** (4.6.4)
5. **Comunicados Oficiais — Broadcast** (4.6.5)

### Princípio de design
Mesma identidade visual da plataforma (Navy + Gold), porém com **densidade de informação maior**. O layout prioriza eficiência operacional sobre estética emocional — é uma ferramenta de trabalho.

---

## 2. Layout geral

### Estrutura

```
┌─────────────────────────────────────────────────────────────┐
│ TOPBAR ADMIN (Navy --surface, 56px)                          │
│ [☰]  Master Síndico · Admin    [🔔] [Avatar ▾]              │
├────────────┬────────────────────────────────────────────────┤
│  SIDEBAR   │  CONTENT AREA                                   │
│  ADMIN     │  Breadcrumb: Admin > Dashboard                 │
│  (240px)   │  ───────────────────────────────                │
│            │  (Conteúdo do módulo ativo)                     │
│  Dashboard │                                                  │
│  Usuários  │                                                  │
│  Moderação │                                                  │
│  Config.   │                                                  │
│  Comunic.  │                                                  │
└────────────┴────────────────────────────────────────────────┘
```

### Sidebar admin (separada do shell padrão)

```
PAINEL ADMINISTRATIVO

📊 Dashboard
   └─ Financeiro
   └─ Visão Geral

👥 Usuários
   └─ Listar Todos
   └─ Por Perfil/Plano
   └─ Histórico

🛡️ Moderação
   └─ Vídeos Denunciados
   └─ Posts do Fórum
   └─ Conteúdo Removido

⚙️ Configurações
   └─ Termos e Políticas
   └─ Banners da Home
   └─ Categorias Técnicas
   └─ Parâmetros Globais

📢 Comunicados
   └─ Novo Comunicado
   └─ Histórico de Envios
   └─ Agendados

← Voltar à Plataforma (link → cms)
v2.0.0
```

### Responsividade
| Breakpoint | Comportamento |
|---|---|
| ≥ 1280 px | Sidebar 240 px sempre visível |
| 1024–1279 px | Colapsável |
| < 1024 px | **NÃO SUPORTADO** — `MinScreenGuard` exibe "O painel administrativo requer tela ≥ 1024 px" |

---

## 3. Módulo 1 — Dashboard Financeiro

Rota: `/dashboard` (raiz da app `apps/admin`).

### KPI Cards (4 colunas grid)

- **MRR** (Michroma 28 px)
- **Assinaturas Ativas**
- **Cancelamentos (mês)**
- **Churn Rate**

Variação positiva: `--success` (verde, ↑). Variação negativa: `--destructive` (vermelho, ↓). Subtexto "vs mês anterior" em `--muted-foreground`.

### Gráfico MRR (Line Chart)
- Cor linha: `--primary` (Gold), 2 px.
- Área preenchida: `--primary` opacity 0.1 → transparent.
- Grid: `--border` opacity 0.5 dashed.
- Lib sugerida: `chart.js` ou `unovis` (leve, tree-shakeable, integração SolidJS).

### Distribuição por Plano

Donut chart com cores por `plan_tier` (paleta derivada do design system, sem sair do espectro Navy-Gold):

| Tier | Cor OKLCH |
|---|---|
| Morador `base` | `--muted` (cinza) |
| Síndico `trial` | `oklch(0.5 0.06 256)` (navy médio) |
| Síndico `base` | `oklch(0.6 0.07 256)` (navy claro) |
| Síndico `plus` | `oklch(0.55 0.09 84)` (gold médio) |
| Síndico `pro` | `--primary` (Gold pleno) |
| Empresa `trial` | `oklch(0.65 0.05 256)` |
| Empresa `plus` | `oklch(0.55 0.09 84)` |
| Empresa `pro` | `oklch(0.72 0.12 84)` (gold brilhante) |
| Marketing | `oklch(0.65 0.10 200)` |
| `local_company` | `oklch(0.7 0.05 84)` (gold pálido) |

### Integração com gateway de pagamento
Dashboard consome dados via API backend (cacheados); frontend **não** se comunica direto com Stripe / Pagar.me. Refresh: polling 5 min ou botão manual.

---

## 4. Módulo 2 — Gestão de Usuários

Rotas: `/usuarios`, `/usuarios/:userId`.

### Listagem
- Tabela HTML semântica (`<table>`).
- Filtros: por `role`, `plan_tier`, status (Ativo / Bloqueado / Pendente), data de cadastro.
- Busca por nome / email / CPF / CNPJ.
- Paginação: 25 itens / página, server-side cursor-based.

### Detalhe do usuário (tabs Kobalte)
1. **Dados** — read-only + botão "Editar" (caso excepcional).
2. **Condomínios** (apenas Síndicos) — lista de vínculos.
3. **Atividade** — timeline vertical das últimas 50 ações.
4. **Conteúdo** — grid de vídeos / cursos publicados.
5. **Ações** — Bloquear / Desbloquear / Resetar Senha / Alterar Plano.

### Ação "Bloquear Usuário"
Modal com:
- Motivo obrigatório (textarea 100–500 chars).
- Checkbox "Enviar email notificando bloqueio".
- Confirmação dupla.

---

## 5. Módulo 3 — Moderação de Conteúdo

Rotas: `/moderacao`, `/moderacao/videos`, `/moderacao/forum`, `/moderacao/historico`.

### Fila de moderação
- 3 cards no topo: Vídeos pendentes / Fórum pendente / Histórico total.
- Lista de itens denunciados com ações inline `[Assistir/Ler]` `[Aprovar]` `[Reprovar]`.

### Modal de reprovação
- Select de motivo (obrigatório):
  - Violação de diretrizes
  - Conteúdo comercial / propaganda
  - Dados de contato expostos
  - Conteúdo ofensivo
  - Informação perigosa / falsa
- Textarea opcional (max 500).
- Checkbox "Suspender funcionalidades do autor" (visível se reincidente).
- Checkbox "Notificar autor por email" (default checked).

### Histórico de moderação
- Tabela com Data, Tipo, Conteúdo, Autor, Ação, Moderador.
- Exportável em CSV (gerado backend).

---

## 6. Módulo 4 — Configurações da Plataforma

Rotas: `/configuracoes/{termos|banners|categorias|parametros}`.

### Termos e Políticas
- Editor rich text (toolbar: Bold / Italic / Headings / Link / Listas).
- Versionamento simples (data + snapshot).
- Aviso `--warning`: "Ao publicar, todos os usuários serão notificados".
- Lib sugerida: `tiptap` headless ou Markdown + preview.

### Banners da Home
- Drag & drop para reordenação (`@thisbeyond/solid-dnd`).
- Upload PNG / JPG / WebP, max 2 MB, ratio recomendado 16:5.
- Limite: 5 banners ativos simultâneos.

### Categorias Técnicas (31 do Cap. 6)
- Accordion Kobalte.
- Editar nome / Adicionar subcategoria / Desativar (soft delete).
- Aviso ao desativar: "X empresas estão vinculadas a esta categoria".

### Parâmetros Globais
Inputs numéricos para parâmetros críticos:

```
- Tempo padrão de janela de votação (dias)
- Limite de vídeos/mês (Síndico plus, plus, pro / Empresa plus, pro)
- Trava trimestral institucional (90 dias) ⚠️ Alteração afeta hard-coded
- Limite de Connect Me (Morador base, plus / Síndico base, plus, pro)
- Trava de cupom por estabelecimento (4 horas)
- Mínimo de visualização para contagem (% — 70 default)
```

> Validação Zod: todos números inteiros positivos. Avisos `--warning` em parâmetros que afetam regras hard-coded.

---

## 7. Módulo 5 — Comunicados Broadcast

Rotas: `/comunicados/{novo|historico|agendados}`.

### Criação de comunicado
- **Segmentação**: Todos | Por `role` (multi-select) | Por `plan_tier` (multi-select).
- **Assunto** (5–200 chars).
- **Mensagem** (rich text, 20–5000 chars).
- **CTA opcional**: texto botão (max 30) + URL destino.
- **Agendamento**: Enviar agora | Agendar (date + time picker, GMT-3).
- **Preview live** do email renderizado.
- **Estimativa de destinatários** atualiza em tempo real.

### Confirmação de envio (modal)
"Você está prestes a enviar um comunicado para ~X destinatários. Esta ação não pode ser desfeita."

---

## 8. Validações Zod (Admin)

```ts
const GlobalParamsSchema = z.object({
  votingWindowDays: z.number().int().min(1).max(90),
  videoLimitSindicoPlus: z.number().int().min(1).max(50),
  videoLimitSindicoPro: z.number().int().min(1).max(50),
  videoLimitEmpPlus: z.number().int().min(1).max(50),
  videoLimitEmpPro: z.number().int().min(1).max(50),
  quarterlyLockDays: z.number().int().min(30).max(365),
  connectMeLimitBase: z.number().int().min(0).max(20),
  connectMeLimitPlus: z.number().int().min(0).max(20),
  couponCooldownHours: z.number().int().min(1).max(48),
  minViewPercentage: z.number().int().min(50).max(100),
});

const BroadcastSchema = z.object({
  targeting: z.discriminatedUnion("type", [
    z.object({ type: z.literal("all") }),
    z.object({ type: z.literal("byRole"), roles: z.array(z.string()).min(1) }),
    z.object({ type: z.literal("byPlanTier"), tiers: z.array(z.enum(["trial", "base", "plus", "pro"])).min(1) }),
  ]),
  subject: z.string().min(5).max(200),
  body: z.string().min(20).max(5000),
  cta: z.object({ text: z.string().max(30), url: z.string().url() }).optional(),
  schedule: z.discriminatedUnion("type", [
    z.object({ type: z.literal("now") }),
    z.object({ type: z.literal("scheduled"), dateTime: z.string().datetime() }),
  ]),
});

const BlockUserSchema = z.object({
  reason: z.string().min(100).max(500),
  notifyByEmail: z.boolean(),
});

const RemoveContentSchema = z.object({
  reason: z.enum([
    "content_violation",
    "commercial_content",
    "exposed_contact_data",
    "offensive_content",
    "dangerous_information",
  ]),
  notes: z.string().max(500).optional(),
  suspendAuthor: z.boolean(),
  notifyAuthor: z.boolean(),
});
```

---

## 9. Componentes Admin

| Componente | Descrição |
|---|---|
| `AdminShell` | Layout wrapper sidebar + topbar admin (separado do `AppShell` padrão) |
| `AdminSidebar` | Navegação 240 px |
| `AdminTopbar` | Barra superior 56 px |
| `KpiCard` | Card de métrica com variação |
| `ChartLine` | Gráfico de linha MRR |
| `ChartDonut` | Distribuição por plano |
| `UserTable` | Tabela paginada |
| `UserDetail` | Detalhe com tabs |
| `ModerationQueue` | Fila de itens |
| `ModerationAction` | Modal de ação |
| `RichTextEditor` | Editor (tiptap) |
| `BannerManager` | Drag & drop banners |
| `CategoryTree` | Accordion 31 categorias |
| `ParamInput` | Input numérico com label + unidade |
| `BroadcastForm` | Formulário comunicado |
| `EmailPreview` | Preview renderizado |
| `MinScreenGuard` | Bloqueio < 1024 px |

## 10. Estados e feedback

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

Toasts (solid-sonner): success em salvar parâmetros / enviar comunicado, warning em bloquear usuário, destructive em remover conteúdo.

## 11. Notas de implementação

1. **App separada `apps/admin`** (D-134 — revoga D-058). Layout completamente independente do `AppShell` padrão.
2. **Backend rejeita requests admin de user-agents mobile** como camada extra de segurança (além de `MinScreenGuard` no frontend).
3. **Drag & drop**: `@thisbeyond/solid-dnd` para banners e reordenação de categorias.
4. **Paginação server-side cursor-based** via TanStack Query com `keepPreviousData: true`.
5. **Exports CSV** gerados backend (não processa frontend).
6. **MFA obrigatório** (TOTP) M2+ — ADR-0026.
7. **4-eyes** em destrutivas (grant temporário ABAC, suspender tenant) M3+.

## Links

- [[../../../../STATE]] — D-134 admin app dedicada (revoga D-058)
- [[../../../web/ui-catalog#b7-admin-ms--app-dedicada-d-134]] — telas AD1-AD8 (catálogo macro, atualizado para D-134 — app dedicada `apps/admin`)
- [[../../../../00-product/business-model]] — `plan_tier`, planos, parâmetros
- [[../../../../08-security/BEYOND_CORP|08-security/BEYOND_CORP]]
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
