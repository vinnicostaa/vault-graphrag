---
title: Archive 2026-04-24 — Chaos processado (Fase A)
type: archive-index
tags:
  - archive
  - chaos
  - master-sindico
  - 2026-04-24T00:00:00.000Z
  - fase-a
created: 2026-04-24T00:00:00.000Z
sessions:
  - systematic-review-chaos-2026-04-24
references:
  - STATE.md
  - AUDIT.md
status: archived
---

# Archive 2026-04-24 — `_chaos/` processado (Fase A)

> Archive criado pela sessão de consolidação de 2026-04-24 a pedido do João: *"meta aqui é remover tudo dessa pasta chaos e organizar no projeto se tiver divergência... priorize a informação mais recente antes de cruzar dados com o projeto atual"*. Análise foi feita por 4 subagentes Opus (J/L/K2 + arquitetura-solucoes leitura direta) cruzando 61 arquivos com canônico atual.

> ⚠️ **NÃO REMOVER**. Material aqui é **referência histórica**, registro de evolução do projeto, e proteção contratual. Política do vault: arquivar, nunca destruir (`feedback_vault_reorganization_2026_04_22`).

## Conteúdo (14 arquivos arquivados nesta fase)

### `legacy-oso-ts-proposal/` — 5 arquivos (08/02/2026)
Proposta de refatoração para stack **TypeScript/Fastify/Drizzle/Oso Cloud** — **stack morta desde 2026-02-28** (substituída por Go+sqlc+ABAC engine próprio). Material valioso como referência histórica do que foi considerado antes da migração Go.

- `ANALISE_TECNICA_COMPLETA_OSO.md`
- `DIAGRAMAS_ARQUITETURA_OSO.md`
- `GUIA_MIGRACAO.md`
- `PLANO_REFATORACAO_OSO.md`
- `RESUMO_EXECUTIVO.md`

### `stale-stack-proposals/` — 3 arquivos (15/03 + 13/04 + 29/03)
Análises arquiteturais com stacks superadas:
- `# Master Síndico — Plano de Execução Det.md` (15/03) — propunha **AWS ECS + NestJS/Fastify + React Native + mastersindico-schemas TS**. Marcos contratuais (08/05, 20/06, 20/07) batiam com canônico ROADMAP, mas stack inteira foi substituída.
- `mastersindico-content-analysis.md` (13/04) + `.pdf` (29/03) — **TS/Fastify/Drizzle/Bun/CASL + Mercado Pago + AWS** com 8 sprints até 08/08/2026. Superada por D-067/D-080 (Go), D-117 (Cloudflare), D-119 (SolidJS+Rsbuild+UnoCSS), D-118 (Twilio), D-120 (search OpenSearch/Meili). Conteúdo aproveitável já destilado em paralelismo Main+Background → futuro `30-agents/playbooks/` (Fase C).

### `superseded-scopes/` — 1 arquivo (15/01/2026)
- `novo escopo-7.pdf` — superado por `novo escopo-8.pdf` (29/03/2026). Continha N1/N2/N3 (abolido por D-081/D-096) e Fastify (substituído por Go). **Regras de negócio aproveitáveis** já destiladas pelo Agent K2 (privacidade métricas, vídeo trava 90d, Connect Me direções). Resto é histórico.

### `original-mysindico-contract/` — 1 arquivo (30/12/2025)
- `Escopo de Prestação de serviços para MySindico-2.pdf` — **contrato comercial original** com branding **"My Síndico"** (renomeado para "Master Síndico" entre 30/12/25 e 15/01/26) e stack **Fastify+Node+TanStack Start+Mux CDN**. Conteúdo: unit economics Dec/2025 (217K usuários, 17K produtores, US$19.395/mês operacional, US$1/mês por pagante). **Referência histórica para justificar pricing** ao board.

### `visual-id-historic/` — 1 arquivo (14/01/2026)
- `Versão 2 - ID Visual Master Síndico-1.pdf` — PDF de identidade visual. Conteúdo de texto extraído foi pobre (39 linhas via pdftotext); o material gráfico real (paleta OKLCH + Manrope/Michroma + 0.5rem border-radius) **já está canonizado** em `02-architecture/design-tokens.md` (D-108 — extraído de `Development/web/apps/*/src/main.css`).

### `superseded-overviews/` — 1 arquivo (09/03/2026)
- `Arquitetura da parte de governança.pdf` — overview introdutório com **3 personas** (Síndico/Morador/Empresa). Superado pelo canônico atual com **5+ personas** (`personas.md`) + 11 sub-produtos consolidados em ADR-0032. Os fluxos operacionais úteis (§9 execução de serviço + atividade técnica) serão destilados na Fase C para `04-requirements/functional/institutional.md`.

### `duplicates/` — 2 arquivos (09/03/2026)
- `JORNADA DA EMPRESA-2.pdf` — **idêntico bit-a-bit** a `JORNADA DA EMPRESA-1.pdf` (confirmado por `diff` no Agent K2). Versão duplicada acidentalmente.
- `JORNADA DA EMPRESA.pdf` — versão **incompleta** (cortada em E16, primeiras 363 linhas idênticas a -1). Provavelmente versão anterior ao -1 completo.
- A versão canônica `-1` (606 linhas) será destilada na Fase D.

## Próximas fases (não executadas ainda)

- **Fase B (4h)**: absorver 27 MDs feature-specs (23/02) → `03-subprojects/web/ui-catalog/` + design system + states feedback patterns
- **Fase C (6h)**: cruzar `requirements(6).md` (4256 linhas) com `04-requirements/functional/` existente — adicionar deltas (Reqs 14.1, 15, 25, 27, 40, 41, 42, 47, 52, 94, 96)
- **Fase D (4h)**: destilar 5 jornadas + 2 currículo + onboarding + vizinhança + matrizes (após tradução N1/N2/N3)
- **Fase E (4h)**: módulo Assembleia M3 — **bloqueada** aguardando esclarecimento João sobre escopo (live vs assíncrona)
- **Fase F (1h)**: TELA DO MORADOR.ini + ARCHITECTURE_ANALYSIS_REPORT (BeyondCorp roadmap + retry budget)
- **Fase G (1h)**: STATE.md D-126..D-155 + AUDIT.md A-053..A-085 + cleanup final

## Decisões consolidadas (2026-04-24 — confirmadas pelo João)

Ver `STATE.md` D-126..D-155 (a registrar). Sumário:

- **N1/N2/N3 permanecem abolidos** — D-081/D-096 vencem Matriz Funcional + escopo-7
- **Stack Go vence** TS/Fastify/Drizzle/NestJS/AWS ECS
- **Stripe vence Mercado Pago** (PIX/boletos via Stripe BR)
- **Telemetria OTel+Grafana+Sentry** vence OSS Prometheus+Loki+Jaeger
- **Vídeo-currículo 90s** (não 60s)
- **15 condomínios/síndico** (não 12)
- **Morador Pagante NÃO existe** — só planos `trial/base/plus/pro` (item #6 João: *"Morador pagante não existe mais"*)
- **Reputação Bronze→Diamante expandida para Síndicos** (não só Empresa)
- **Banco de Talentos M3** + vídeo-currículo 90s
- **Podcast = feature interna de Conteúdo** (não sub-produto #12)
- **Voice Input pt-BR aprovado** (M3+ acessibilidade)
- **Procuração 100% digital** (Lei 14.063, sem upload PDF físico)
- **Admin painel dedicada** (REVOGA D-058 que dizia role transversal embutida)
- **Bloqueio múltiplas contas por IP rejeitado** — pesquisar alternativa via hash de dispositivo
- **Pendentes**: Onboarding URL, tenant kinds 2 vs 4, persona Administradora, persona Auditor, aggregate boundaries

## Vizinhos

- [[../../STATE]]
- [[../../AUDIT]]
- [[../../11-audit/audit-log/2026-04-24-multitenant-consolidation-map]]
- [[../../_chaos]] (em processo de esvaziamento)


---

## Fase B concluída — 2026-04-25 (27 arquivos)

> Sessão de absorção dos **27 MDs feature-specs / design-system** datados de 2026-02-23 que estavam em `_chaos/`. Conteúdo absorvido para overlay canônico `03-subprojects/web/ui-catalog/`, `03-subprojects/web/patterns/ui-states.md` e `02-architecture/design-tokens.md` (delta).

### Arquivos arquivados em `feature-specs-23-fev/`

| # | Arquivo original | Destino canônico |
|---|---|---|
| 00 | `00-DESIGN-TOKENS.md` | `02-architecture/design-tokens.md` §18 (delta merged) |
| 01 | `01-AUTH-ONBOARDING.md` | `ui-catalog/auth/onboarding.md` |
| 02 | `02-APP-SHELL.md` | `ui-catalog/shell/app-shell.md` |
| 03 | `03-HOME-DASHBOARD.md` | `ui-catalog/cms/home-dashboard.md` |
| 04 | `04-PROFILE-MANAGEMENT.md` | `ui-catalog/cms/profile-management.md` |
| 05 | `05-VIDEO-PLAYER.md` | `ui-catalog/content/video-player.md` |
| 06 | `06-VIDEO-UPLOAD.md` | `ui-catalog/content/video-upload.md` |
| 07 | `07-SEARCH-ENGINE.md` | `ui-catalog/cross/search-engine.md` |
| 08 | `08-CONNECT-ME.md` | `ui-catalog/commercial/connect-me.md` |
| 09 | `09-ASSEMBLY-MANAGEMENT.md` | `ui-catalog/assembly/management.md` |
| 10 | `10-VOTING-SYSTEM.md` | `ui-catalog/assembly/voting.md` |
| 11 | `11-TRANSPARENCY-DASHBOARD.md` | `ui-catalog/institutional/transparency.md` |
| 12 | `12-EVALUATION-RATING.md` | `ui-catalog/commercial/evaluation.md` |
| 13 | `13-COUPON-ENGINE.md` | `ui-catalog/commercial/coupon-engine.md` |
| 14 | `14-BENEFITS-CLUB.md` | `ui-catalog/commercial/benefits-club.md` |
| 15 | `15-LMS-COURSES.md` | `ui-catalog/lms/courses.md` |
| 16 | `16-PRIVATE-FEEDBACK.md` | `ui-catalog/cross/private-feedback.md` |
| 17 | `17-FORUM.md` | `ui-catalog/forum/main.md` |
| 18 | `18-LIVE-STREAMING.md` | `ui-catalog/content/live-streaming.md` (D-133 — Live broadcast ≠ Assembleia) |
| 19 | `19-PODCAST.md` | `ui-catalog/content/podcast.md` |
| 20 | `20-ADMIN-PANEL.md` | `ui-catalog/admin/main.md` (D-134 — `apps/admin` separada, revoga D-058) |
| 21 | `21-TALENT-BANK.md` | `ui-catalog/commercial/talent-bank.md` (D-099 — 5 vínculos, M3) |
| 22 | `22-SETTINGS.md` | `ui-catalog/cross/settings.md` |
| 23 | `23-NOTIFICATIONS.md` | `ui-catalog/cross/notifications.md` |
| 24 | `24-STATES-FEEDBACK.md` | `03-subprojects/web/patterns/ui-states.md` (PATTERN cross, NÃO ui-catalog) |
| 25 | `MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md` (398 KB, 7814 linhas) | dividido em 4: §1-2 → `02-architecture/design-tokens.md` §18 (delta); §3 → `ui-catalog/shell/topbar-sidebar-bottomnav.md`; §4-5 + §11 → `ui-catalog/_components-and-screens.md` (índice); §6 → `patterns/ui-states.md` |
| 26 | `MASTER_SINDICO_DESIGN_SYSTEM_ADDENDUM_v1.1.md` | `60-sources/master-sindico-research/ui-references/2026-02-addendum-voicelabs-genaigurus.md` |

### Tradução aplicada (vocabulário canônico)

- **`N1/N2/N3`** → `plan_tier ∈ {trial, base, plus, pro}` — ADR-0032 / D-081 / D-096.
- **"Morador Pagante"** removido — D-126 (confirmado João 2026-04-25): morador é gratuito por padrão; addon **Banco de Talentos** (D-099, M3, 5 vínculos profissionais visíveis) é opt-in.
- **"My Síndico"** → "Master Síndico".
- **`Síndico Pro/Plus/Base`** ainda permitido como label UI; código / spec usa `plan_tier`.

### Decisões revisadas / consolidadas (2026-04-25)

- **D-133**: Lives institucionais broadcast (Mux Live ou Cloudflare Stream) ≠ Assembleia (Livekit conferência tipo Meet). Provider de Live broadcast ≠ provider de assembly.
- **D-134** (revoga D-058): Admin agora é app dedicada `apps/admin/` no monorepo web, não embutida em `cms/`. Catálogo macro `web/ui-catalog#b7-admin-ms-embutido-ad1-ad8` precisa atualização para refletir D-134.
- **D-099** (substitui D-064): Banco de Talentos morador exibe **5 vínculos profissionais** (não 3), M3.
- **D-126**: "Morador Pagante" descontinuado.

### Próximas fases (do plano original)

- **Fase C (6 h)**: cruzar `requirements(6).md` com `04-requirements/functional/` — adicionar deltas (Reqs 14.1, 15, 25, 27, 40, 41, 42, 47, 52, 94, 96).
- **Fase D (4 h)**: destilar 5 jornadas + 2 currículo + onboarding + vizinhança + matrizes (após tradução N1/N2/N3).
- **Fase E (4 h)**: Assembleia M3 (bloqueada — esclarecimento João sobre escopo live vs assíncrona).
- **Fase F (1 h)**: TELA DO MORADOR.ini + ARCHITECTURE_ANALYSIS_REPORT.
- **Fase G (1 h)**: STATE.md D-126..D-155 + AUDIT.md + cleanup final.

### Observação — `_chaos/` após Fase B

Restam **20 arquivos** em `_chaos/` (PDFs de jornadas / matrizes / arquitetura de assembleia + `requirements(6).md` + `master-sindico-arquitetura-solucoes.md` + `ARCHITECTURE_ANALYSIS_REPORT.md` + `novo escopo-8.pdf` + `TELA DO MORADOR.ini`) — alvos das Fases C, D, E, F.


---

## Fase F concluída — 2026-04-25 (2 PDFs Matrizes Funcionais)

> Sessão de absorção dos **2 PDFs de Matrizes Funcionais** datados de 2026-03-09 que estavam em `_chaos/`. Conteúdo absorvido para overlay canônico `04-requirements/matrix-functional.md` + `00-product/business-model.md` via tradução **N1/N2/N3 → plan_tier** (D-081/D-096) e remoção do label "Morador Pagante" (D-126 sobre Morador Pagante / espírito D-067/D-096).

### Arquivos arquivados em `60-sources/master-sindico-research/client-material/pdfs/`

| # | Arquivo original | Destino canônico | Conteúdo absorvido |
|---|---|---|---|
| F1 | `Matriz Funcional GERAL.pdf` (4.0 MB) | `60-sources/master-sindico-research/client-material/pdfs/2026-03-09-matriz-funcional-geral.pdf` | 8 seções (Consumo de Conteúdos / Comunidade / Publicação / Perfil-Visibilidade / Connect Me / Gestão Condominial / Currículos / Clube Benefícios) — colunas legadas `Morador Pg \| Base \| Síndico N1 \| Síndico N2 \| Síndico N3 \| Emp. Plus \| Emp. Pro \| Marketing \| Vizinhança` |
| F2 | `MATRIZ FUNCIONAL MÍDULO TRIAL.pdf` (3.9 MB) | `60-sources/master-sindico-research/client-material/pdfs/2026-03-09-matriz-funcional-trial.pdf` | Trial Síndico 15d / Empresa 7d / Parceiro 30d — features liberadas/bloqueadas + mensagens de upgrade contextuais |

### Tradução aplicada (vocabulário canônico)

| Legado (PDF) | Canônico (Fase F) |
|---|---|
| Síndico N1 / Síndico Gestor | `(role=syndic, tier=base)` |
| Síndico N2 / Síndico Plus (label PDF) | `(role=syndic, tier=plus)` |
| Síndico N3 / Síndico Pro (label PDF) | `(role=syndic, tier=pro)` |
| Empresa Plus / Emp. Plus | `(role=enterprise, tier=plus)` |
| Empresa Pro / Emp. Pro | `(role=enterprise, tier=pro)` |
| Morador Pg / Morador Pagante | `(role=resident, tier=plus)` — **rótulo abolido como label vivo (D-126); UI mostra "Morador Plus" / "Morador Pro"** |
| Morador Base | `(role=resident, tier=base)` |
| Marketing | `(role=marketing)` — sub-role rotas internas D-102 |
| Vizinhança / Comércio Local | `(role=local_company)` |
| "My Síndico" | "Master Síndico" |
| Lives (criação "My Síndico") | Admin + Empresa Pro (D-097/D-133) |

**Contagem**:
- ~22 menções a "N1/N2/N3" / "Síndico Gestor/Plus/Pro" → traduzidas para `plan_tier` em §"Tradução histórica" expandida
- ~8 menções "Morador Pg / Morador Pagante" → removidas como label vivo de `business-model.md` §3.2 / §5.1 / §7.5 / §8 (substituídas por "Morador Plus/Pro" ou neutralizadas)
- 0 menções "My Síndico" no canônico (já era "Master Síndico" antes da Fase F)
- Connect Me Morador Base atualizado de `0/ano` (D-058 revogada) → **2/ano** (D-094 Fase 11)

### Patches aplicados

**`04-requirements/matrix-functional.md`**:
- Banner topo Fase F com cross-link aos PDFs movidos
- §"Tradução histórica" expandida: 14 entradas (vs 11 anteriores) — adicionados Síndico Gestor/Plus/Pro (labels PDF), Morador Pg, Comércio Local, "My Síndico", Lives criação
- §"Matriz de Trial detalhada" **NOVA** — 3 sub-seções (Síndico/Empresa/Parceiro) com features × estado trial × pós-trial canonizadas + regras gerais + mensagens contextuais de upgrade (PDF MÓDULO TRIAL)
- §"Deltas Fase F" **NOVA** — 23 deltas analisados (14 alinhados, 4 canônico vence, 5 novas pendências PEND-MATRIX-04 atualizada + PEND-MATRIX-06/07/08 novas)
- §"Pendências" expandida com PEND-MATRIX-06 (limite perfil Empresa Plus), PEND-MATRIX-07 (escopo link compartilhamento timeline), PEND-MATRIX-08 (Plano de Eleição/Reeleição como feature distinta)
- §"Referências" adiciona links aos PDFs absorvidos
- Frontmatter `status: consolidado-fase-8` → `consolidado-fase-f`; tag `fase-f-pdfs-absorvidos`; `updated: 2026-04-25`

**`00-product/business-model.md`**:
- Banner topo Fase F com cross-link à matriz de trial detalhada
- §3.2 Morador: tabela atualizada — `0/ano` → `2/ano` (D-094 sobrescreve D-058); label "Morador Pagante" → "Morador Plus" / "Morador Pro"
- §5.1 Trial table: "Morador Pagante" → "Morador Plus/Pro" + nota explicativa
- §5.2: confirmação trial Síndico/Empresa/Parceiro reforçada com referência ao PDF
- §7.2 trigger morador: "0/ano em base" → "2/ano em base — D-094"
- §8 Pricing: "Morador Pagante" → "Morador Plus" na tabela
- §13 Decisões vivas: D-058 marcada como revogada; D-094..D-103 + D-115/D-126 adicionadas
- Frontmatter `consolidado-fase-8` → `consolidado-fase-f`; tag `fase-f-pdfs-absorvidos`

### Deltas/conflitos detectados (canônico vence)

| Δ | PDF (legado) | Canônico (vence) | Justificativa |
|---|---|---|---|
| Lives criação | "My Síndico" único criador | Admin + Empresa Pro + agência delegada (D-097/D-133) | PDF é pré-D-097/2026-04-24 |
| Score Governança | 1 score implícito | 2 scores (Governança 1-10 + Compliance 0-100) (D-103) | PDF não cobre detalhe |
| Banco Talentos vínculos | implícito 3 (ESTRUTURA CADASTRO) | 5 (D-099) | D-099 sobrescreve D-064 |
| Sub-produtos count | features avulsas | 11 (D-115) | Consolidação Fase 11 |
| "Morador Pagante" label | usado como coluna ("Morador Pg") | abolido como label vivo (D-126/espírito D-067) | UI/backend canonizados pós-Fase 11 |

### Pendências novas (PEND-MATRIX-06/07/08)

PDFs trazem 3 features que não estavam formalizadas no canônico — registradas como pendências:
- **PEND-MATRIX-06**: limite perfil Empresa Plus ("2 fotos + 1 vídeo 60s")
- **PEND-MATRIX-07**: escopo do "Link de compartilhamento de atividades/manutenções" (público / restrito-condomínio / OAuth?)
- **PEND-MATRIX-08**: "Plano de Eleição/Reeleição" — feature distinta ou dimensão do Plano Diretor?

### Observação — `_chaos/` após Fase F

Restam **5 arquivos** em `_chaos/` (PDFs Assembleia: `ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf`, `DIAGRAMA VISUAL DA ASSEMBLEIA.pdf`, `MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO.pdf` — Fase E em paralelo; `ARCHITECTURE_ANALYSIS_REPORT.md` — futura Fase F2; `####################TELA DO MORADOR#####.ini` — futura Fase F2).



---

## Fase E concluída — 2026-04-25 (3 PDFs Assembleia)

> Sessão de absorção dos **3 PDFs do Módulo Assembleia** datados de 2026-03-09 (Master Síndico) que estavam em `_chaos/`. Conteúdo absorvido para overlay canônico `03-subprojects/web/ui-catalog/assembly/` (7 telas novas) + `04-requirements/functional/assembly.md` (11 novos Reqs ASM-034..044) + patches em aggregates `Assembly` / `AgendaItem` / `Minutes`.

### Arquivos arquivados em `60-sources/master-sindico-research/client-material/pdfs/`

| # | Arquivo original | Destino canônico |
|---|---|---|
| E1 | `ARQUITETURA FUNCIONAL DO MÓDULO ASSEMBLEIA.pdf` (1281 linhas) | `60-sources/.../pdfs/2026-03-09-arquitetura-funcional-assembleia.pdf` |
| E2 | `DIAGRAMA VISUAL DA ASSEMBLEIA.pdf` (289 linhas) | `60-sources/.../pdfs/2026-03-09-diagrama-visual-assembleia.pdf` |
| E3 | `MÓDULO DAS TELAS ASSEMBLEIA MASTER SÍNDICO.pdf` (661 linhas) | `60-sources/.../pdfs/2026-03-09-modulo-telas-assembleia.pdf` |

### Telas frontend criadas (7 novos arquivos em `ui-catalog/assembly/`)

| Arquivo | Conteúdo |
|---|---|
| `pre-assembly.md` | Wizard 4 steps + procuração 100% digital (ASM-010) + análise fornecedores + simulador quórum + painel monitoramento + cadastro inadimplência + importar fração ideal + validar procurações |
| `live-session.md` | Painel ao vivo síndico + cadastro mesa diretora + vídeo institucional pre-roll + telão 2 áreas + fila fala cronometrada + voto presencial assistido + status final do item (4 estados) |
| `check-in.md` | 3 modalidades (app, QR code, terminal) + estados de presença (presente/ausente_momentaneo/desconectado) |
| `post-assembly.md` | Encerramento + registro final + homologação + ata homologada (hash + carimbo Lei 14.063) + recibo individual de voto + 8 relatórios + linha do tempo |
| `administrator-panel.md` | Aceite convite + dashboard admin + 4 capabilities canônicas (D-127) + audit trail das ações + termo responsabilidade |
| `transparency-score.md` | Score 10 componentes em 4 subíndices (Convocação/Participação/Votação/Documentação) + widget público no perfil condomínio + faixas (Excelência/Bom/Regular/Baixo/Crítico) |
| `contingency-mode.md` | Ativação modo offline + auto-save Redis + tela morador degraded + histórico contingências + desativação |

### Patches em arquivos existentes

- `ui-catalog/assembly/management.md` — adicionado bloco "Telas detalhadas Fase E" no §Links com 7 cross-links
- `ui-catalog/assembly/voting.md` — adicionada seção "Atualizações Fase E" + cross-links para telas detalhadas
- `ui-catalog/_moc.md` §Assembly (BC) — 7 entradas novas listadas
- `04-requirements/functional/assembly.md`:
  - Banner Fase E adicionado no topo
  - **ASM-028 reescrito** com 4 subíndices granulares (Convocação 30pt + Participação 30pt + Votação 20pt + Documentação 20pt) — substitui lista anterior
  - **11 novos Reqs**: ASM-034 (Painel admin 4 capabilities) · ASM-035 (Mesa diretora pré-iniciar) · ASM-036 (Vídeo institucional pre-roll) · ASM-037 (Telão 2 áreas spec técnico) · ASM-038 (Status final item — 4 estados) · ASM-039 (Microfone alternado + transcrição) · ASM-040 (Apresentação dentro plataforma) · ASM-041 (Audit trail 22+ eventos) · ASM-042 (Modo contingência ratificado + UX) · ASM-043 (Cláusulas legais Master Síndico) · ASM-044 (Tipos de voto canônicos SIM/NAO/ABS + ESCALA1-5)
  - §Pendências atualizada (resolução parcial + 3 novas)
- `01-domain/aggregates/Assembly.md` — declarado entities filhas (Vote D-130, AssemblyAdministratorLink D-127, Auditor D-132, AssemblyDirectorship futuro) + métodos novos (`GrantAdministrator`, `ActivateContingency`, `CascadeInvalidateOnEnd`)
- `01-domain/aggregates/AgendaItem.md` — Vote como entity (D-130) + 4 estados terminais (ASM-038)
- `01-domain/aggregates/Minutes.md` — campos `contentHashSHA256` + `timestampCarimbo` (Lei 14.063 — Fase E) + nova invariante INV-### (integridade verificável)

### Reqs novos vs já cobertos

**Já cobertos por Fase C (R48/R52/R57)** — confirmados Fase E sem duplicação:
- Procuração 100% digital (ASM-010 / R52) — **rejeitado upload PDF físico** (PDFs §4.13 mencionam "apresentar fisicamente" — REJEITADO em D-_chaos absorção Fase C; Fase E ratifica)
- Lista de inadimplentes T-1h (R48 §5)
- Status enum 5 estados (R48 §8)
- Tempo fala 2/3min + prorroga (R48 §11)
- Vídeo institucional pre-roll (R48 §12) — agora tem spec técnico próprio (ASM-036)
- Voto presencial assistido (R57 §6 / ASM-016)
- Termo de voto canônico (R57 §8)
- Reabertura votação pré-homologação (R57 §13-§14)

**Novos Fase E (não cobertos pela Fase C)**:
- Painel admin 4 capabilities canônicas (ASM-034) — D-127 já mencionou link entity, mas capabilities não estavam formalizadas
- Mesa diretora obrigatória pré-iniciar (ASM-035)
- Telão 2 áreas spec técnico (ASM-037)
- Status final item 4 estados (ASM-038)
- Microfone alternado + transcrição (ASM-039)
- Apresentação dentro da plataforma (ASM-040)
- Audit trail 22+ eventos canônicos (ASM-041)
- Modo contingência UX completo (ASM-042) — ASM-022 já existia, agora enriquecido
- Cláusulas legais Master Síndico (ASM-043)
- Tipos de voto canônicos SIM/NAO/ABS + ESCALA1-5 (ASM-044) — antes só "Aprovar/Reprovar/Múltipla/Fração ideal/Eleição"

### Conflitos detectados (entre PDFs ou com decisões vigentes)

| Δ | PDF (legado) | Canônico vigente | Resolução |
|---|---|---|---|
| Procuração: \"apresentar fisicamente\" (§4.13) | upload PDF físico | 100% digital Lei 14.063 (ASM-010 + Fase C) | **PDF rejeitado** — canônico vence |
| Score Transparência: lista 10 itens (§8) | lista canônica diferente em ASM-028 anterior | substituída por canônica do PDF | **PDF vence** — mais granular e PT-BR consistente, agrupada em 4 subíndices |
| Eleição: \"hating imediato\" (§4.4) | ASM-ELE-AVAL (avaliação escondida obrigatória) | mecanismo já existente | **alinhado** — assume-se ASM-ELE-AVAL é o "hating" |
| \"Morador Pagante\" implícito em \"morador apto a votar\" | abolido D-126 | \"morador apto = morador apto\" | **canônico vence** — rótulo morador-pagante não usado |
| Síndico vota? (PDFs sugerem morador-síndico vota) | INV-081: presidente não vota | imparcialidade do presidente | **canônico vence** (síndico-presidente não vota; síndico fora-mesa pode votar quando proprietário) |

### Pendências (continuam abertas)

- **Provider broadcast Live (ADR-0044)**: Cloudflare Stream vs Mux Live — pendente. **Não afeta Assembly** (D-133 — Assembly = Livekit conferência); afeta apenas content/live-streaming.
- **Provider Livekit Cloud vs self-hosted (ASM-020)**: ADR M4+ pendente.
- **Provider de assinatura digital ICP-Brasil (ASM-010, ASM-024)**: Bry, ITI, AC SERPRO — ADR M4+; M3 stub HMAC.
- **Provider de transcrição (ASM-039)**: Whisper local vs Speechmatics cloud — ADR pendente.
- **Conversor PowerPoint→PDF (ASM-040)**: cliente-side via libreoffice headless OU servidor — ADR pendente.
- **Detecção automática problemas (ASM-022/042)**: heartbeat WebSocket threshold + sync conflict resolution policy (LWW vs reject).
- **Voto presencial assistido — UX assinatura biométrica do morador no terminal (ASM-016)**: Touch ID nativo iPad / Windows Hello — spec técnica adicional necessária.
- **Algoritmo "captação ótima" microfone (ASM-039)**: VAD + RMS sliding window — spec.
- **Capability `assembly.contingency.activate`**: confirmar inclusão no `pg.assembly.administrator_delegation` (D-116 IAM).

### Observação — `_chaos/` após Fase E

`_chaos/` está **vazio** após Fase E (todos os arquivos absorvidos pelas Fases A-F). Restavam apenas `ARCHITECTURE_ANALYSIS_REPORT.md` e `master-sindico-arquitetura-solucoes.md` que aparentemente foram movidos por outra Fase paralela.


---

## Fase D concluída — 2026-04-25 (9 PDFs Jornadas + Currículo + Onboarding + Vizinhança)

> Sessão de absorção dos **9 PDFs de jornadas e módulos** datados de 2026-03-09/10 (Master Síndico) que estavam em `_chaos/`. Conteúdo destilado em **8 specs de jornada frontend** em `03-subprojects/web/ui-catalog/jornadas/` (~140 telas catalogadas com fluxos UX completos), preservando os PDFs originais em `60-sources/master-sindico-research/client-material/pdfs/`.
>
> **Princípio fundamental do João (2026-04-25)**: *"os documentos tambem diziam sobre fluxos de tela, isso tem que ir para os fronts do projeto, é de suma importancia que esses fluxos sejam do front e nao do back, ja que isso é relevante para o front, o backend cuida de prover as features para essas telas"*.

### PDFs movidos para `60-sources/master-sindico-research/client-material/pdfs/`

| # | Arquivo original (`_chaos/`) | Destino canônico (`60-sources/.../pdfs/`) | Linhas | Tela frontend (jornadas/) |
|---|---|---|---|---|
| D1 | `JORNADA DO SÍNDICO completa.pdf` | `2026-03-09-jornada-sindico-completa.pdf` | 1634 | `sindico.md` (S1-S31, 31 telas) |
| D2 | `JORNADA DO MORADOR.pdf` | `2026-03-09-jornada-morador.pdf` | 619 | `morador.md` (M1-M15, 15 telas) |
| D3 | `JORNADA DA EMPRESA-1.pdf` | `2026-03-09-jornada-empresa.pdf` | 606 | `empresa.md` (E1-E16, 16 telas) |
| D4 | `JORNADA DA EMPRESA DE MARKETING.pdf` | `2026-03-09-jornada-agencia-marketing.pdf` | 291 | `agencia-marketing.md` (MK1-MK8, 8 telas) |
| D5 | `Jornada do Parceiro da Vizinhança.pdf` | `2026-03-10-jornada-parceiro-vizinhanca.pdf` | 444 | `parceiro-vizinhanca.md` (VZ1-VZ6) |
| D6 | `MÓDULO VIZINHANÇA - MORADOR.pdf` | `2026-03-10-modulo-vizinhanca-morador.pdf` | 394 | `parceiro-vizinhanca.md` (VM1-VM7 — combinado) |
| D7 | `ESTRUTURA DE CADASTRO DE CURRÍCULO.pdf` | `2026-03-09-estrutura-cadastro-curriculo.pdf` | 304 | `curriculo-cadastro.md` (M-CV-1 a M-CV-11, 11 telas) |
| D8 | `COMO A EMPRESA VÊ O CURRÍCULO DO MORADOR (1).pdf` | `2026-03-09-empresa-ve-curriculo-morador.pdf` | 217 | `curriculo-empresa-view.md` (E-CV-1 a E-CV-7, 7 telas) |
| D9 | `ONBOARDING DOS PERFIS.pdf` | `2026-03-09-onboarding-perfis.pdf` | 446 | `onboarding.md` (4 perfis × 3-6 telas, 17 telas) |

### 8 specs de jornada frontend criadas em `03-subprojects/web/ui-catalog/jornadas/`

```
ui-catalog/jornadas/
├── _moc.md                    ← Índice
├── _pendencias-fase-h.md      ← Divergências/gaps detectados (consolidado)
├── sindico.md                 ← S1-S31 (31 telas) · cms · D1
├── morador.md                 ← M1-M15 (15 telas) · cms · D2
├── empresa.md                 ← E1-E16 (16 telas) · cms · D3
├── agencia-marketing.md       ← MK1-MK8 (8 telas) · cms · D4
├── parceiro-vizinhanca.md     ← VZ1-VZ6 + VM1-VM7 (13 telas) · cms+forum · D5+D6 (combinados)
├── onboarding.md              ← 4 perfis × telas (~17 telas) · auth · D9
├── curriculo-cadastro.md      ← M-CV-1 a M-CV-11 (11 telas) · cms · D7
└── curriculo-empresa-view.md  ← E-CV-1 a E-CV-7 (7 telas) · cms · D8
```

**Total**: ~118 telas catalogadas com fluxos UX completos (estados, ações, campos, regras canônicas referenciadas), além das ~17 do onboarding (telas comuns + por perfil) — totalizando ~135 telas com spec UX detalhada.

### Tradução aplicada (vocabulário canônico)

| Legado (PDF) | Canônico (Fase D) |
|---|---|
| `N1/N2/N3` | `plan_tier ∈ {trial, base, plus, pro}` (D-081, D-096, ADR-0032) |
| "Morador Pagante" | REMOVIDO — morador é `base` gratuito + addon Banco de Talentos opt-in (D-126, D-099) |
| "My Síndico" | "Master Síndico" |
| Mandato como aggregate | Flags em `Membership(role=sindico)` (D-129) |
| Empresa Síndica | Membership especial com 5 toggles parametrizáveis (D-073) |
| Admin role transversal | App dedicada `apps/admin` (D-134) |
| Lives broadcast = Assembleia | Lives (Mux/Cloudflare) ≠ Assembleia (Livekit) (D-133) |
| 4 tenant kinds | 2 tenant kinds (D-126); user/partner são EntityID |

### Cross-links principais criados

Cada tela frontend referencia (sem duplicar regras):
- **Aggregates**: Condominium, Membership, MasterPlanAction, TimelineEntry, ExecutionRecord, Communicado, Event, ConnectMe, EmpresaProfile, Partner, Promotion, PromotionActivation, Curriculum, AuditLog, ResponsibilityTerm, ManagementEvaluation, MoradorQuestion, AgenciaLink, MarketingLink, EmpresaUser, EmpresaVideo, Account, Profile, LegalAcceptance, GovernanceMarkers
- **Reqs canônicos**: REQ-IDN-* (identity), REQ-INS-* (institutional), REQ-GOV-* (governance), REQ-COM-* (commercial), REQ-CPL-* (compliance) — referência por âncora, sem duplicação
- **Invariantes**: INV-TIMELINE-INSERT-ONLY, INV-AUDIT-IMMUTABLE, INV-LGPD-DATA-REVEAL, INV-DEPENDENT-MAX-2, INV-VIDEO-VISIBILITY-VOTING, INV-PROMOTION-SEGMENTATION, INV-MARKETING-LINK-NO-AUTO-VINCULO, INV-AVALIACAO-ANONIMA, INV-CURRICULUM-NO-DIRECT-CONTACT, etc.
- **ADRs**: ADR-0028 (LGPD), ADR-0029 (mandato flags), ADR-0030 (tenant kinds 2), ADR-0032 (plan_tier), ADR-0033 (vídeo lock 90d), ADR-0034 (Banco de Talentos addon)
- **Patterns**: dashboard-cards-pivot, feed-infinite-scroll, multi-step-with-autosave, voice-input-textarea, legal-terms-checkbox, destructive-double-confirm, version-history, expirable-content, dashboard-kpi-cards, etc.
- **Enums**: parceiro-vizinhanca-categorias (38), curriculum-areas (18), curriculum-modalidades (9), promotion-types (7), areas-impactadas (26), timeline-activity-types (26), risk-levels (9), event-types (13), communicado-types (8), video-types (12), governance-markers — alguns ainda a criar (Fase H)

### Pendências consolidadas em `_pendencias-fase-h.md`

**A. Divergências numéricas** (PDF vs canônico):
- A.1 — Limite condomínios síndico: PDF=12, macro=15 (média)
- A.2 — Vídeo Banco de Talentos: PDF=60s, macro=90s (baixa — adotar 60s)
- A.3 — Vínculos PDF export: PDF=3, D-099=5 (baixa — export pode truncar)
- A.4 — Indicadores Dashboard Empresa: PDF=8, macro=13 (baixa — macro super-set)

**B. Ambiguidades / valores não especificados**: idade mínima dependente; cooldown contato Banco de Talentos; subtema vídeo agência; resposta única a perguntas; preservação de termos jurídicos íntegros (S29/S30)

**C. Aggregates novos a confirmar**: ContactRequest, PromotionActivation, ResponsibilityTerm, ManagementEvaluation, MoradorQuestion, CompanyTag

**D. Cross-checks Reqs Fase F**: 9 enums a criar/confirmar (categorias-vizinhanca, curriculum-areas, modalidades, áreas-impactadas, timeline-activity-types, risk-levels, event-types, communicado-types, video-types, governance-markers)

**E. Invariantes a confirmar/criar**: ~14 invariantes nomeados explicitamente

### Patches em arquivos existentes

- `03-subprojects/web/ui-catalog/_moc.md` — adicionada seção "Jornadas (UI specs por persona — Fase D 2026-04-25)" listando as 8 jornadas
- `03-subprojects/web/ui-catalog.md` (catálogo macro) — adicionada seção cross-link à pasta `jornadas/`

### Observação — `_chaos/` após Fase D

`_chaos/` está **vazio** após Fase D (todos os 9 PDFs movidos para `60-sources/master-sindico-research/client-material/pdfs/`). Restavam apenas arquivos das Fases E/F que aparentemente já foram processados em paralelo.

### Conformidade com princípio backend/frontend

- **Telas, fluxos, wireframes, ações UX** → este Fase D em `ui-catalog/jornadas/` ✅
- **Regras de negócio** → NÃO duplicadas; apenas REFERENCIADAS via wikilinks aos canônicos `04-requirements/functional/<bc>.md` ✅
- **Invariantes novas / aggregates novos** → registrados em `_pendencias-fase-h.md` para Fase H avaliar (não tocados aqui) ✅
- **Reqs novos** → idem (Fase C já cobriu o grosso) ✅


---

## ✅ FECHAMENTO 2026-04-25 — Fases C-H concluídas

`_chaos/` está **VAZIO** ✅. Total de **61 arquivos processados** em **8 fases** (A-H) entre 2026-04-24 e 2026-04-25.

### Distribuição final

| Fase | Conteúdo | Destino | Arquivos |
|---|---|---|---|
| A | Legacy/superseded/duplicates | `_archive/2026-04-24-chaos-processed/{7 sub-pastas}/` | 14 |
| B | 27 MDs feature-specs (23/02) | `03-subprojects/web/ui-catalog/{shell,auth,cms,content,commercial,assembly,institutional,lms,forum,admin,cross}/` + `patterns/ui-states.md` + `design-system.md` + `60-sources/.../ui-references/` | 27 |
| C | requirements(6).md + mastersindico-requirements.pdf | `60-sources/.../client-material/{internal-requirements,pdfs}/` + patches em `04-requirements/functional/*` | 2 |
| D | 5 jornadas + onboarding + 2 currículo + vizinhança morador | `03-subprojects/web/ui-catalog/jornadas/` (frontend!) + `60-sources/.../pdfs/` | 9 |
| E | 3 PDFs Assembleia | `03-subprojects/web/ui-catalog/assembly/` (7 telas novas) + patches functional/assembly + 3 aggregates patched + `60-sources/.../pdfs/` | 3 |
| F | 2 Matrizes funcionais | patches `matrix-functional.md` + `business-model.md` + `60-sources/.../pdfs/` | 2 |
| G | escopo-8 + ARCH_ANALYSIS + arquitetura-solucoes + tela-morador.ini | `60-sources/.../pdfs/` (escopo-8) + `60-sources/.../internal-docs/` (3 outros) + `_pendencias-fase-h.md` | 4 |
| H | Cleanup final (D-136 Morador Pagante + D-137 sumário + grep N1/N2/N3 + AUDIT update + DT-012 vocabulary debt) | STATE.md + AUDIT.md + 9 patches críticos N2+ → plan_tier | 0 (operacional) |
| **Total** | | | **61** |

### Decisões D-### resultantes

12 decisões: **D-126..D-137**
- D-126 Tenant kinds 2
- D-127 AssemblyAdministratorLink entity
- D-128 Block aggregate opcional
- D-129 Mandate flags em Membership
- D-130 Vote entity de AgendaItem
- D-131 Adjustment cross-domain (já existia, movido)
- D-132 Auditor role temporária ABAC
- D-133 Live ≠ Assembleia (correção conceitual)
- D-134 App admin dedicada (revoga D-072)
- D-135 Bloqueio IP rejeitado, device fingerprint progressivo
- D-136 Morador Pagante abolido como label vivo (regulariza decisão João item #6 sessão 2026-04-25)
- D-137 Sumário consolidação (este fechamento)

### Findings resolvidos

A-053..A-082 (~30 findings) — ver [[../../AUDIT]] sessão 2026-04-25.

### Débitos pendentes

- **DT-012** — Vocabulary cleanup N2+/N3+/Morador Pagante em ~15 arquivos não-críticos (rolling M2)
- **A-076** — Dual-check retroativo D-094..D-118 (Sprint 10)
- **A-077** — Reescrita completa admin-privilegios.md (banner suficiente por ora)
- **A-078..A-082** — 5 patterns + sub-roles + ADR-0044 broadcast pendentes (M2-M3)

### Patterns pendentes a destilar

Ver [[../../60-sources/master-sindico-research/client-material/_pendencias-fase-h]]:
1. BeyondCorp 4-fases roadmap → `08-security/BEYOND_CORP.md` (M2)
2. Retry budget + jitter → `02-architecture/patterns.md` (M2)
3. PBT custom shrinkers + PROPERTY_TEST_SEED → `09-testing/pbt.md` (M2-M3)
4. Diagrama 4 high-domains → `02-architecture/system-overview.md` (nice-to-have)
5. Sub-roles internos D-114/D-102/D-073/D-132 → respectivos functional/* (M2-M3)

### Vizinhos

- [[../../STATE#D-137]] — sumário oficial
- [[../../AUDIT]] — sessão 2026-04-25
- [[../../60-sources/master-sindico-research/client-material/_pendencias-fase-h]] — patterns
