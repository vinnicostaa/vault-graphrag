---
title: ROADMAP — Master Síndico v2
type: roadmap
tags: [roadmap, milestones, master-sindico, v2]
created: 2026-04-23
updated: 2026-04-23
---

# ROADMAP — Master Síndico v2

Marcos de entrega do produto (herdado do legado; será revisado à luz do research web na Fase 3). Este arquivo descreve o roadmap **do produto**, não da remontagem em si (esse está no plano em `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`).

---

## Marcos do produto (herdados; sujeitos a revisão)

### 🟢 M1 — MVP Contratual

- **Data**: **2026-05-18** (firme após D-106)
- **Escopo**: Identity + Billing + Institutional base + busca real
  - ✅ Identity: auth Zitadel OIDC+PKCE, sessões, 1-device policy, device fingerprint
  - ✅ Billing: trial por plano (15d trial → base/plus/pro — D-081/D-096), Stripe recorrência, quotas
  - ✅ Institutional: condomínio, unidades, `Unit.unit_type` + `fracao_ideal NUMERIC(10,4)` (BE-001), timeline imutável, membership, plano diretor base
  - ⏳ Frontend web (SolidJS + UnoCSS + Rsbuild + TanStack + Bun — D-119): onboarding síndico + dashboard básico
  - ⏳ Mobile (Flutter + flutter_bloc — D-048/049): auth + home shell
  - ⏳ **ISearchProvider** real (OpenSearch ou Meilisearch — D-120 / ADR-0042) — busca é vital, não mock
  - ⏳ Outbox `domain_events` table + worker poller (ADR-0009/0015 áreas)
  - ⏳ `webhook_events` UNIQUE table + anti-replay HMAC 2min + cookie sem ponto inicial (ADR-0027)
  - ⏳ `TenantBoundaryGuard` middleware (ADR-0034)
  - ⏳ MFA admin obrigatório + step-up MFA (ADR-0026)
- **Critério de aceite**: Síndico cria conta → escolhe plano → trial → Stripe recorrência real → Zitadel produção + DNS `mastersindico.com.br` + busca funcionando em institutional/commercial
- **Gates bloqueadores**: AUDIT 🔴🟡 = 0, coverage global ≥ 85%, DR drill executado pré-M1, todos os gates verdes, dual-check fechado para D-094..D-118 e ADRs proposed

### 🟡 M2 — Connect Me + Conteúdo

- **Data**: 2026-06-20
- **Escopo**: Commercial + Content
  - Connect Me (4 fluxos: Síndico→Empresa, Morador→Síndico, Empresa→Empresa Plus/Pro, Síndico→Parceiro)
  - Reputação Bronze→Diamante (determinística)
  - Marketplace de empresas + banco de talentos (vídeo-currículo 90s)
  - Vídeos institucionais (Mux, trava 90d)
  - Moderação manual
  - Busca expandida (relevance scoring, faceted search, busca em transcrições de vídeo) sobre o `ISearchProvider` já implantado em M1

### 🔵 M3 — Assembleia + LMS + Polish

- **Data**: 2026-07-20
- **Escopo**: Assembly + LMS + polish operacional
  - Assembleias live (Livekit), votação quórum fracional, ata imutável, edital PDF, proxy/procuração, fila de fala
  - LMS thin (5 áreas, 3 trilhas iniciais, certificados)
  - A11y (WCAG 2.1 AA), SLO 99.5% em rotas críticas
  - Observability maturity (OTel + Grafana + Sentry em prod)

---

## Marcos de operação (pós-M3, backlog)

### M4 — Escala + Compliance Avançado (Q3 2026)

- Validação Receita Federal (CNPJ/CPF compliance)
- Compliance NR-1, ABRACS integration
- ML moderação (OpenAI Moderation ou AWS Rekognition)
- Breach notification 72h automática

### M5 — Expansão + Multi-região (Q4 2026)

- Empresa-síndica (multi-condomínio gestão delegada)
- Agência de Marketing multi-empresa
- Vizinhança geográfica avançada (raio + cupons 4h)
- Assembleia híbrida online/presencial (se demanda validar)

### M6 — Mobile Premium + AI (2027)

- AI copiloto síndico (sumários timeline, alertas conformidade)
- Notificações inteligentes (FCM + priorização ML)
- Mobile feature parity + offline-first

---

## Revisões previstas na Fase 3 da remontagem

- **M1**: sem mudança provável — data firme, escopo protegido.
- **M2**: revisar ordem Connect Me vs Content a partir de research Netflix/YouTube/Instagram (marketplace + content discovery).
- **M3**: revisar estratégia Livekit vs WebRTC custom a partir de research Google Meet. Definir se LMS entra thin ou só backlog (M4).
- **M4-M6**: preencher detalhes após research.

---

## Links

- Plano da remontagem: `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`
- Cronograma detalhado: [[10-schedule/cronograma-macro]] (após Fase 4)
- Milestones por sub-produto: [[10-schedule/milestones]] (após Fase 4)
- Roadmap legado vivo: `../vault/50-projects/master-sindico/ROADMAP.md`
