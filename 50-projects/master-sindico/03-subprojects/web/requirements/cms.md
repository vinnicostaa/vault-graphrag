---
title: cms — Web Requirements (SolidJS)
type: requirement
tags: [master-sindico, web, solidjs, cms, feature-req, core]
project: master-sindico
stack: web
app: cms
port: 3001
milestone: m1
status: scaffold
created: 2026-04-24
---

# cms — Web Requirements

## Escopo e marco

**Marco**: M1 (Sprint 10, 2026-05-08). Entregável principal: portal principal multi-persona com Morador (M1-M15), Síndico base (S3-S6, S11-S14), Admin D-058 (AD1-AD2), Conta/Sistema (SYS1-SYS3). **O maior app** — comporta quase todo o produto exceto assembly live e lms.

M2 expande: Síndico completo (S1-S31), Compliance (C1-C11), Empresa (E1-E16), Agência (MK1-MK8), Vizinhança subset (VZ/VS/VE/VM), Admin completo (AD3-AD6).

M3 adiciona: Banco Talentos (TEL1-TEL11), Admin 4-eyes (AD7-AD8), integração profunda com lms/assembly embeds.

**Bloqueador M1** — pior caso porque cms depende de auth + ui atoms + schemas simultaneamente.

## Personas servidas

- `sindico` (N1/N2/N3) — 31 telas principais + compliance + admin views restritas
- `morador` — 15 telas + Vizinhança subset
- `empresa` (base/plus/pro) — 16 telas painel
- `agencia` — 8 telas marketing
- `admin_ms` (D-058 transversal) — 8 AD telas embutidas sob `/admin/*`, ABAC gated, MFA M2+, 4-eyes M3+
- `morador` → subset Vizinhança (VM1-VM7)

Admin MS **não tem app separada** (D-058): views admin ficam em cms sob `/admin/*`, protegidas por route guard + backend ABAC + banner "Modo Admin".

## Telas cobertas

Resumo (141+ telas catalogadas; ver [[../ui-catalog]]):

| Cluster | Range ID | Total | UI catalog link |
|---|---|---|---|
| Morador | M1-M15 | 15 | [[../ui-catalog/morador/_moc]] |
| Síndico | S1-S31 | 31 | [[../ui-catalog/sindico/_moc]] |
| Compliance | C1-C11 | 11 | [[../ui-catalog/compliance/_moc]] |
| Empresa | E1-E16 | 16 | [[../ui-catalog/empresa/_moc]] |
| Agência | MK1-MK8 | 8 | [[../ui-catalog/marketing/_moc]] |
| Vizinhança | VZ/VS/VE/VM | 29 | [[../ui-catalog/vizinhanca/_moc]] |
| Banco Talentos | TEL1-TEL11 | 11 | [[../ui-catalog/banco-talentos/_moc]] |
| Admin MS (D-058) | AD1-AD8 | 8 | [[../ui-catalog/admin/_moc]] |
| Conta/Sistema | SYS1-SYS8 | 8 | — |

## Funcionais (FR-WEB-CMS-NNN)

### Shell / Navegação

- **FR-WEB-CMS-001** — Layout `_app`: sidebar colapsável + header com user-menu + breadcrumbs + content area.
- **FR-WEB-CMS-002** — Sidebar navega por persona ativa; se usuário tem múltiplas personas (ex: sindico + morador em dois condomínios), dropdown "Trocar contexto".
- **FR-WEB-CMS-003** — Role switcher (Zitadel ABAC) via `GET /auth/me` + `POST /auth/switch-context`.
- **FR-WEB-CMS-004** — Breadcrumbs auto-gerados por TanStack Router `useMatches`.
- **FR-WEB-CMS-005** — Command palette Kbd `⌘K` / `Ctrl+K` (backlog M2).

### Morador (M1-M15) — M1 target

- **FR-WEB-CMS-010** — M1 Dashboard morador: próximos eventos, comunicados recentes, timeline resumo.
- **FR-WEB-CMS-011** — M2 Timeline condomínio: lista infinita, filtros (categoria, data), reações.
- **FR-WEB-CMS-012** — M3 Comunicados: lista + detalhe + anexos (download assinado).
- **FR-WEB-CMS-013** — M4 Eventos: lista + detalhe + RSVP + calendário iCal export.
- **FR-WEB-CMS-014** — M5 Minha Unidade: dados fração ideal + moradores + contatos síndico.
- **FR-WEB-CMS-015** — M6 Boletos: lista + download + linha digitável copy.
- **FR-WEB-CMS-016** — M7 Documentos: biblioteca (convenção, regulamento, atas) — read-only morador.
- **FR-WEB-CMS-017** — M8 Reservas: áreas comuns + calendário + regras + confirmação.
- **FR-WEB-CMS-018** — M9 Ocorrências: abrir + acompanhar status + anexar fotos.
- **FR-WEB-CMS-019** — M10 Visitantes: pré-autorizar + QR code.
- **FR-WEB-CMS-020** — M11 Encomendas: portaria → notifica morador → retirada.
- **FR-WEB-CMS-021** — M12 Enquetes: votar + ver resultados (após fechamento).
- **FR-WEB-CMS-022** — M13 Chat síndico: 1-1 threaded.
- **FR-WEB-CMS-023** — M14 Perfil morador: editar foto + telefone + notificação prefs.
- **FR-WEB-CMS-024** — M15 Vizinhança portal morador: acesso VM1-VM7.

### Síndico (S1-S31) — M1 parcial (S3-S6, S11-S14), M2 completa

- **FR-WEB-CMS-030** — S3 Dashboard síndico: KPIs (inadimplência %, ocorrências abertas, próxima assembleia).
- **FR-WEB-CMS-031** — S4 Timeline admin (pode criar publicações oficiais com voice input R2).
- **FR-WEB-CMS-032** — S5 Comunicados — criar/editar/publicar + agendamento + destinatários por bloco/unidade.
- **FR-WEB-CMS-033** — S6 Eventos — criar/editar + RSVP tracking + export CSV.
- **FR-WEB-CMS-034** — S11 Moradores — CRUD + busca + filtros + import CSV.
- **FR-WEB-CMS-035** — S12 Plano Diretor — timeline estratégica com voice input R2.
- **FR-WEB-CMS-036** — S13 Finanças — visão geral (link contabilidade externa, MVP read-only).
- **FR-WEB-CMS-037** — S14 Relatórios — templates + export PDF.
- **FR-WEB-CMS-038** — S15-S20 Connect Me (M2): 4 fluxos de intermediação (morador↔empresa, sindico↔empresa, empresa↔morador, morador↔morador-vizinhanca).
- **FR-WEB-CMS-039** — S21-S25 Ocorrências / Ordens de Serviço (M2).
- **FR-WEB-CMS-040** — S26 Validações pendentes (M2) — aprovar registros de execução.
- **FR-WEB-CMS-041** — S27-S31 Reservas admin, documentos admin, compliance export, mandato management.

### Empresa (E1-E16) — M2

- **FR-WEB-CMS-050** — E1 Dashboard empresa (contratos ativos, faturas, leads Connect Me).
- **FR-WEB-CMS-051** — E2-E4 Connect Me respostas (empresa recebe lead → responde → revela contato após consent).
- **FR-WEB-CMS-052** — E5 Registro de execução (com voice input R2) → vira validação pendente síndico.
- **FR-WEB-CMS-053** — E6-E8 Portfólio empresa + orçamentos + propostas.
- **FR-WEB-CMS-054** — E9-E12 Contratos + faturamento + comprovantes.
- **FR-WEB-CMS-055** — E13-E16 Equipe + presença + avaliações recebidas.

### Agência (MK1-MK8) — M2

- **FR-WEB-CMS-060** — MK1-MK3 Campanhas Connect Me + analytics.
- **FR-WEB-CMS-061** — MK4-MK6 Landing pages criadas via builder simples (M2 stretch).
- **FR-WEB-CMS-062** — MK7-MK8 Relatórios ROI.

### Compliance (C1-C11) — M2

- **FR-WEB-CMS-070** — C1 Visão geral compliance (doc expirados, assembleias pendentes, atas atrasadas).
- **FR-WEB-CMS-071** — C2-C5 Documentos obrigatórios por tipo condomínio.
- **FR-WEB-CMS-072** — C6-C9 Calendário regulatório + alertas.
- **FR-WEB-CMS-073** — C10-C11 Export LGPD (DSAR usuário) + auditoria.

### Admin MS D-058 (AD1-AD8)

- **FR-WEB-CMS-080** — Route guard `/admin/*`: backend ABAC + MFA M2+ + banner "Modo Admin".
- **FR-WEB-CMS-081** — AD1 Dashboard admin (tenants ativos, MRR, churn).
- **FR-WEB-CMS-082** — AD2 Tenants — lista + suspender (double-confirm + 4-eyes M3).
- **FR-WEB-CMS-083** — AD3-AD6 Usuários + planos + incidentes + feature flags.
- **FR-WEB-CMS-084** — AD7-AD8 Auditoria + broadcast global (4-eyes approval).

### Conta/Sistema (SYS1-SYS8)

- **FR-WEB-CMS-090** — SYS1 Perfil usuário + alterar senha + sessões ativas.
- **FR-WEB-CMS-091** — SYS2 Notificações prefs.
- **FR-WEB-CMS-092** — SYS3 LGPD: consent prefs + export dados + pedido de exclusão.
- **FR-WEB-CMS-093** — SYS4-SYS8 Plano atual + billing + API keys (empresa/agência) + webhooks.

## Não-funcionais (NFR-WEB-CMS)

- **Performance**: LCP < 2.5s mobile / 1.8s desktop; INP < 200ms; CLS < 0.1. Bundle initial < **300 KB gzip**. Cada rota lazy < 150 KB.
- **Acessibilidade**: WCAG 2.1 AA obrigatório; AAA em Connect Me + votação + pagamento. Kobalte cobre 80%. Contraste OKLCH matriz validada.
- **SEO**: cms **não indexável** — `noindex,nofollow`. Exceto páginas públicas de unidade (se ativadas por flag).
- **i18n**: pt-BR; termos preservados (síndico, condomínio, fração ideal, Connect Me, outorgante/outorgado, ata, quórum).
- **Multi-tenancy**: todas queries carregam `X-Tenant-Id` header derivado do contexto ativo; frontend NUNCA decide permissão — backend ABAC é source-of-truth.
- **Coverage**: 95% domain schemas, 90% hooks/queries, 85% components críticos, 75% overall.

## Rotas TanStack Router

```
src/routes/
├── __root.tsx
├── _app.tsx                            # layout autenticado (sidebar + header)
├── _app.index.tsx                      # redirect persona-aware
├── _app.morador.$.tsx                  # M1-M15 (lazy per sub-route)
├── _app.sindico.$.tsx                  # S1-S31
├── _app.empresa.$.tsx                  # E1-E16
├── _app.agencia.$.tsx                  # MK1-MK8
├── _app.vizinhanca.$.tsx               # VZ/VS/VE/VM
├── _app.compliance.$.tsx               # C1-C11
├── _app.banco-talentos.$.tsx           # TEL1-TEL11
├── _app.admin.$.tsx                    # AD1-AD8 (guard: beforeLoad MFA)
├── _app.conta.$.tsx                    # SYS1-SYS8
└── _app.sistema.$.tsx                  # SYS admin
```

## Estado

- **Signals + createStore** para estado local de formulário e UI.
- **TanStack Solid Query** para server state: `staleTime` curto (30s) para listas, longo (5min) para referência (personas, planos).
- **TanStack Solid Form + Zod adapter** (`packages/schemas`) em TODOS formulários (20+ formulários no M2).
- **Store global**: `createStore` singleton `tenantContext` (tenant ativo, persona, roles) + `notifications` queue; resto é TanStack Query cache.
- **Optimistic updates** em: reservar área (M8), abrir ocorrência (M9), criar comunicado (S5), RSVP evento (M4).

## Integração backend

- **Base URL**: `http://localhost:8000/api/v1/*` / `https://api.mastersindico.com.br/api/v1/*`.
- **Auth**: Cookie httpOnly `.mastersindico.com.br` (setado pelo auth app).
- **Headers**: `X-Tenant-Id`, `X-CSRF-Token` (double-submit), `Accept-Language: pt-BR`.

### Endpoints consumidos (amostra — total ~80)

| Método | Path | Payload | Retorno |
|---|---|---|---|
| GET | `/cms/morador/dashboard` | — | `{events,comunicados,timeline}` |
| GET | `/cms/morador/units/me` | — | `{unit,residents,sindico}` |
| POST | `/cms/sindico/comunicados` | `{titulo,corpo,destinatarios,agendamento?}` | 201 `{id}` |
| POST | `/cms/sindico/eventos` | `{titulo,data,local,rsvp}` | 201 `{id}` |
| GET | `/cms/sindico/moradores?page&q` | — | `{items,total}` |
| POST | `/cms/reservas` | `{areaId,data,slot}` | 201 |
| POST | `/cms/ocorrencias` | `{titulo,descricao,anexos[]}` | 201 |
| POST | `/cms/connect-me/requests` | `{tipo,destino,mensagem}` | 201 |
| POST | `/cms/connect-me/:id/reveal` | `{consent}` | 200 `{contato}` |
| GET | `/cms/admin/tenants?page` | — | `{items,total}` |
| POST | `/cms/admin/tenants/:id/suspend` | `{motivo,fourEyes?}` | 204 |
| POST | `/cms/uploads/ticket` | `{kind,size}` | `{uploadUrl,uploadId}` |
| GET | `/cms/lgpd/consent` | — | `{items}` |
| POST | `/cms/lgpd/export` | — | 202 `{jobId}` |

## Design system

- `packages/ui` atoms: **20+ componentes necessários** — Button ✅ + faltam: TextField, Select, Dialog, Dropdown, Sheet (mobile sidebar), Tabs, DataTable, Avatar, Badge, Toast (Kobalte), Skeleton, Empty, ErrorState, PermissionDenied, Spinner, Calendar, DatePicker, FileUpload, Dropzone, RichText (tiptap?), VoiceInput.
- CVA variants por app ex: `card-sindico` (navy) vs `card-morador` (gold accent).
- Dark mode via Kobalte ColorModeProvider com toggle em header.

## Segurança

- **CSP**: script-src com nonce; connect-src allowlist + Mux; img-src + data:.
- **XSS**: SolidJS auto-escape + DOMPurify em rich-text (comunicados/timeline).
- **CSRF**: double-submit `X-CSRF-Token` + SameSite=Lax.
- **IDOR**: backend valida tenant-id server-side; frontend nunca decide permissão.
- **Admin isolation D-058**: banner amarelo fixo "Modo Admin", fundo sutilmente diferente, MFA recheck antes de ações destrutivas (suspender/broadcast).
- **LGPD**: Connect Me modal de consent versionado; export/delete de dados via SYS3.

## Testes

- **Vitest (unit)** thresholds: 95% schemas, 90% hooks/queries, 85% components críticos, 75% overall.
- **@solidjs/testing-library** — cobrir shell (sidebar, role switcher), Morador M1-M5, Síndico S3-S6, Admin AD1-AD2 M1.
- **MSW** — handlers por cluster (morador, sindico, admin).
- **Playwright (E2E M3)** — fluxo crítico: login → criar comunicado → morador vê → reage.
- **jest-axe** em todas telas de M1.

## Status atual (real) — AUDIT A-FASE10-DEV-005

🔴 **Bloqueador M1**. `apps/cms/src/index.tsx` = stub `<div>Hello cms!</div>`. `AuthProvider`/`QueryClientProvider`/`Toaster` comentados. Zero das 141+ telas implementadas. Zero testes. 19 atoms faltantes em `packages/ui` (só Button existe). `@tanstack/solid-query`, `@tanstack/solid-form`, `msw`, `vitest` NÃO em `bun.lock`. `packages/schemas` vazio.

## Gaps/decisões abertas

- **G1**: rich-text (comunicados/timeline): tiptap Solid vs Lexical vs editor custom — decidir Sprint 10.
- **G2**: DataTable — TanStack Solid Table ainda é alpha; avaliar vs custom.
- **G3**: voice input R2 Web Speech API fallback — iOS Safari limitações.
- **G4**: Admin sub-role matrix M2 — quantos sub-roles ABAC precisos (super_admin, support, billing)?
- **G5**: command palette ⌘K — priorizar M2 ou backlog?
- **G6**: SSE vs polling para notificações in-app — SSE tem mais reconnect issues.

## Links

- [[_moc]]
- [[../../../CLAUDE]]
- [[../architecture]]
- [[../design-system]]
- [[../security]]
- [[../requirements]]
- [[../ui-catalog/sindico/_moc]]
- [[../ui-catalog/morador/_moc]]
- [[../ui-catalog/empresa/_moc]]
- [[../ui-catalog/admin/_moc]]
- [[../ui-catalog/compliance/_moc]]
- [[../../../00-product/personas]]
- [[../../admin-privilegios]]
