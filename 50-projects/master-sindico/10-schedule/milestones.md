---
title: Milestones — Master Síndico v2
type: guide
tags: [schedule, milestones, m1, m2, m3, master-sindico, v2]
source: ROADMAP.md + 00-product/portfolio-de-produtos.md + cronograma-macro.md
created: 2026-04-23
updated: 2026-04-23
---

# Milestones — Master Síndico v2

Detalhamento de cada marco. Pra cada: data firme, escopo contratual, critério de aceite, gates bloqueadores, post-deploy monitoring window.

---

## 🟢 M1 — MVP Contratual

**Data firme**: **2026-05-08** (sexta)

### Escopo contratual

Plataforma operando em produção com 3 bounded contexts core:

- **Identity** (BC `identity`): auth Zitadel OIDC+PKCE, sessões, 1-device policy, device fingerprint, ABAC claims
- **Billing** (BC `billing`): planos (Trial/Base/Plus/Pro), trial persona-aware (síndico 15d, empresa 7d, parceiro 30d), Stripe recorrência, quotas (Connect Me anual, vídeos mensal), soft-block sem grace period
- **Institutional** (BC `institutional`): condomínio, unidades com fração ideal, memberships, timeline imutável, plano diretor base, comunicados, eventos, enquetes, compliance básica

Frontend:
- **Web** (SolidJS): onboarding síndico + dashboard + timeline + plano diretor + gerenciar unidades/moradores
- **Mobile** (Flutter): auth + home shell síndico/morador + timeline leitura

LGPD base:
- Consent registry versionado
- `GET /me/export` (portabilidade)
- `DELETE /me` (direito de exclusão, SLA 15d)
- Audit trail append-only 5 anos
- DPO notificado + runbook 72h breach notification

### Critério de aceite

✅ **E2E contratual**: Síndico cria conta → escolhe plano → trial passa → upgrade pago → Stripe recorrência real cobra → Zitadel produção + DNS `mastersindico.com.br` + app `.mastersindico.com.br` wildcard cookie funcional.

### Gates bloqueadores

- [ ] [[../11-audit/AUDIT]] 🔴 = 0 (A-020, A-021, A-023, A-024 fixados ou 🟡 com plano)
- [ ] Coverage global ≥ 85% (domain 95%, app 90%, infra 85%, http 75%)
- [ ] Gates CI todos verdes (build, vet, test -race, integration, gosec -sev high, govulncheck)
- [ ] Staging 48h estável (error rate < 0.5%, p95 < 500ms)
- [ ] Smoke test Playwright passing
- [ ] Sentry prod captura errors
- [ ] Zitadel Railway + DNS `auth.mastersindico.com.br` ok
- [ ] Stripe live mode webhook recebendo events
- [ ] LGPD DPO ciente + SLA 15d operacionalizado
- [ ] Runbooks (rollback, incident-lgpd, webhook-reprocess) escritos
- [ ] CHANGELOG M1 publicado

### Post-deploy monitoring window

**48h** pós-deploy (2026-05-08 → 2026-05-10):
- Error rate < 0.5%
- p95 API < 500ms
- p99 API < 2s
- Webhook delivery rate ≥ 99% (Stripe)
- Sentry < 10 errors novos/5min
- Oncall ativo 24/7 nesse período

Se breach em qualquer métrica: decisão rollback (ver [[../06-execution/rollback]]).

---

## 🟡 M2 — Connect Me + Conteúdo

**Data firme**: **2026-06-20** (sábado)

### Escopo contratual

Adiciona bounded contexts **commercial** + **content**:

- **Connect Me 4 fluxos**:
  - Síndico → Empresa (RFP → propostas → escolha)
  - Morador → Síndico (form estruturado)
  - Empresa → Empresa Plus/Pro (subcontratação)
  - Síndico → Parceiro Vizinhança
- **Reputação Bronze→Diamante** (determinística) com dashboard empresa
- **Marketplace de empresas** (busca + filtro por tier)
- **Banco de Talentos** thin (vídeo-currículo 90s + profile + busca empresa)
- **Vídeos institucionais** (Mux): upload direto presigned, HLS player, lock 90d, moderação manual 48h
- **Privacidade métricas cross-tenant** (ABAC)
- **Busca OpenSearch** prod
- **Vizinhança**: mapa + cupons lock 4h + seal "Síndico-aprovado"

### Critério de aceite

✅ **Connect Me E2E**: síndico publica RFP → 3 empresas recebem notificação (FCM + email) → 2 enviam proposta → síndico escolhe → contrato fechado dupla assinatura → auto-publish timeline → avaliação pós-execução alimenta reputação.

✅ **Vídeo E2E**: empresa Plus sobe vídeo institucional → moderação manual 48h → aprovado → publish → lock 90d enforce → views de síndicos contabilizadas; métricas privadas a outros.

✅ **Vizinhança E2E**: parceiro publica cupom com lock 4h → morador em raio vê notificação → redime antes da expiração → log.

### Gates bloqueadores

- [ ] AUDIT 🔴 = 0 (A-029, A-030, A-031, A-032 fixados)
- [ ] Chaos tests passing (Mux/Livekit/Stripe webhook failure)
- [ ] Load test 1000 concurrent users: API p95 < 500ms
- [ ] Moderação manual UI demo-ready
- [ ] OpenSearch prod respondendo ≤ 200ms
- [ ] Reputação determinística (PBT degradação + promoção)
- [ ] CHANGELOG M2

### Post-deploy monitoring window

**48h** pós-deploy (2026-06-20 → 2026-06-22). Oncall 24/7.

---

## 🔵 M3 — Assembleia Live + LMS + Polish

**Data firme**: **2026-07-20** (segunda)

### Escopo contratual

Adiciona **assembly** + LMS thin + polish operacional:

- **Assembleias Live** (Livekit):
  - Edital PDF auto/upload
  - Pauta 5 tipos de item (deliberação, informação, eleição, consulta, outro)
  - Ciência obrigatória morador (bloqueia app até ack)
  - Check-in (app QR + terminal físico)
  - Votação real-time WebSocket < 200ms
  - Quórum fracional (peso = fração ideal)
  - Procuração (proxy) registrada pré-sessão
  - Fila de fala cronometrada + alertas 30s
  - Ata imutável auto-gerada pós-sessão
  - Homologação (votação pós-ata)
  - Score de transparência (0-100, 10 componentes)
  - 6 relatórios (presença, votação, procurações, fala, consolidado, transparência) CSV+PDF
  - Telão 2 áreas (Presentation + Operational) thin
- **LMS thin**:
  - 5 áreas (Gestão, Direito, Finanças, Comunicação, Técnica)
  - 3 trilhas piloto (Síndico Iniciante, Intermediário, Profissional)
  - Certificados auto ao 100% + URL verificável
- **Ebooks biblioteca base** (PDF S3 signed URL)
- **a11y** WCAG 2.1 AA em fluxos críticos
- **SLO 99.5%** em rotas críticas (monitor últimos 7d)
- **Observability maturity**: OTel + Grafana + Sentry em prod

### Critério de aceite

✅ **Assembleia Live E2E**: síndico agenda ordinária → publica edital → moradores dão ciência → check-in dia-D → votação item por item (alguns com fração ideal) → procuração respeitada → ata auto-gerada → homologação → transparência score calculada → 6 relatórios gerados.

✅ **LMS E2E**: síndico faz trilha "Iniciante" → videoaulas (Mux) + ebook (S3) + quiz (70% aprovação) → 100% → certificado emitido → URL pública verificável.

### Gates bloqueadores

- [ ] AUDIT 🔴 = 0 (A-033, A-034 fixados)
- [ ] a11y audit pass (axe-core 0 violations em fluxos críticos)
- [ ] SLO 99.5% em rotas críticas últimos 7d
- [ ] 6 relatórios gerados corretamente em teste com dados reais
- [ ] Livekit em prod — 10 participantes simultâneos sem drop
- [ ] Ata imutável pós-publish (UPDATE bloqueado DB + app)
- [ ] Certificado URL pública acessível sem auth

### Post-deploy monitoring window

**48h** pós-deploy (2026-07-20 → 2026-07-22). Oncall 24/7.

---

## Marcos pós-M3 (backlog)

### M4 — Escala + Compliance Avançado (Q3 2026)

- Validação Receita Federal (CNPJ/CPF)
- Compliance NR-1, ABRACS integration
- ML moderação (OpenAI Moderation ou AWS Rekognition) — fechar D-014
- Breach notification 72h automática

### M5 — Expansão + Multi-região (Q4 2026)

- Empresa-síndica (multi-condomínio) — fechar D-012
- Agência de Marketing multi-empresa — fechar D-011
- Vizinhança geográfica avançada (raio + cupons 4h)
- Assembleia híbrida online/presencial

### M6 — Mobile Premium + AI (2027)

- AI copiloto síndico (sumários timeline, alertas conformidade)
- Notificações inteligentes (FCM + priorização ML)
- Mobile feature parity + offline-first

---

## Comms plan por marco

### Pré-deploy (T-7 dias)

- Email síndicos beta: "Próxima entrega em <data>"
- Status page atualizada com release window
- DPO comms se LGPD afetado

### Deploy day

- Status page: "Maintenance window <time>"
- Oncall stand-by
- Canal #announcement fires quando prod estável

### Pós-deploy (T+24h)

- Changelog público (`/changelog` no web + email)
- Release notes com screenshots pra features novas
- Onboarding sessão ao vivo (opcional) pra clientes beta

---

## Links

- [[_moc]]
- [[cronograma-macro]]
- [[calendar]]
- [[../ROADMAP]]
- [[../00-product/portfolio-de-produtos]]
- [[../05-tasks/backlog]]
- [[../06-execution/deploy]]
