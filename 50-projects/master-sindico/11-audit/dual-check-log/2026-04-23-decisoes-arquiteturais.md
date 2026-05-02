---
title: Dual-Check — Decisões arquiteturais consolidadas (D-020..D-043, ADRs 0001-0025, 10 P-### de domínio)
type: note
tags: [dual-check, architecture, master-sindico, v2]
created: 2026-04-23
updated: 2026-04-23
aliases: [dual-check-arquitetural-2026-04-23]
---

# Dual-Check — Revisão independente (contexto limpo)

Revisor em **sessão nova**, sem ter participado da Fase 3/4 de redação dos artefatos. Insumos consultados: [[../../STATE]], [[../../02-architecture/adr/_moc]] + ADRs 0001-0025, [[../../STEERING]], [[../../CLAUDE]], [[../../13-research/linkedin/_destilado]], [[../../13-research/google-arch/_destilado]], arquivos com pendências explícitas `P-CM-###`, `P-INV-###`, `P-EVT-###`, `P-BR-###`, `P-UL-###`, `P-SM-###`, `P-BC-###`. Re-pesquisa via WebSearch cobriu: ULID/UUIDv7, SES/Resend, Railway/ECS, Vite/Bun, Bloc/Riverpod, Hive/Drift, freezed, jailbreak detection OWASP MASVS, lucide/phosphor, Inter/Manrope, Web Push iOS, admin subdomain, saga orch/choreo, outbox, assembleias BR.

Sem acesso ao agente que escreveu os artefatos; avaliação baseada **apenas** no registrado + pesquisa independente.

---

## 0. Tabela-resumo

### 0.1 Decisões STATE + ADR (herdadas ou tomadas na v2)

| Ref | Tema | Proposta registrada | Veredito | Ajuste sugerido |
|---|---|---|---|---|
| ADR-0001 | Clean Arch + DDD + CQRS | accepted | ✅ | — |
| ADR-0002 | Gin abstraído via HTTPRouter | accepted | ✅ | — |
| ADR-0003 | Zitadel OIDC | accepted | ✅ | — |
| ADR-0004 | Stripe IPaymentGateway | accepted | ⚠️ | Avaliar Pagar.me/Pix M2 (mercado BR) |
| ADR-0005 | sqlc queries tipadas | accepted | ✅ | — |
| ADR-0006 | pgx/v5 | accepted | ✅ | — |
| ADR-0007 | Goose particionado | accepted | ✅ | — |
| ADR-0008 | zerolog | accepted | ✅ | — |
| ADR-0009 / D-028 | ULID/UUIDv7 PK | accepted ULID | ⚠️ | Trocar default para UUIDv7 (PG 18 nativo) |
| ADR-0010 | Mux video provider | accepted | ✅ | — |
| ADR-0011 | LiveKit SFU | accepted | ✅ | — |
| ADR-0012 | ABAC + Redis 5min | accepted | ✅ | — |
| ADR-0013 | Timeline imutável | accepted | ✅ | — |
| ADR-0014 | 1-device policy | accepted | ✅ | — |
| ADR-0015 | UoW intra / Saga inter | accepted | ⚠️ | Explicitar "orchestration" vs "choreography" por saga |
| ADR-0016 | BeyondCorp posture | accepted | ✅ | — |
| ADR-0017 / D-024 | Railway M1, AWS M4+ | accepted | ✅ | — |
| ADR-0018 | tsvector dev / OpenSearch prod | accepted | ✅ | — |
| ADR-0019 | NATS JetStream threshold | accepted | ✅ | — |
| ADR-0020 | OTel + Grafana + Sentry | accepted | ✅ | — |
| ADR-0021 | Multi-tenant row-based + PK composto | accepted | ✅ | — |
| ADR-0022 | Circuit breaker gobreaker | accepted | ✅ | — |
| ADR-0023 | Recsys determinístico M1 | accepted | ✅ | — |
| ADR-0024 | Moderação cascade | accepted | ✅ | — |
| ADR-0025 | SLO 99.5% M1 + error budget | accepted | ✅ | — |
| D-020 | Stack web Vite+Tailwind (vs Bun+Rsbuild+UnoCSS legado) | Vite+Tailwind | ✅ | — |
| D-024 | Railway vs ECS | Railway→ECS | ✅ (ver ADR-0017) | — |
| D-026 | SES vs Resend | em aberto | ⚠️→❌ | Começar Resend M1; SES M3+ |
| D-029 | Fonte corpo (Inter vs Manrope) | em aberto | ⚠️ | Inter (maior ecossistema/métricas) |
| D-030 | Icon set (Lucide vs Phosphor) | em aberto | ✅ Lucide | — |
| D-031 | Dark mode | em aberto (proposto sim) | ⚠️ | Sim no app autenticado, tokens prontos; LP sempre light |
| D-032 | Push no web | em aberto (M2+) | ⚠️ | OK M2; atenção restrição PWA-only iOS |
| D-033 | Admin subdomain vs path | em aberto (A recomendado) | ✅ | Opção A com `__Host-` cookie |
| D-040 | Bloc vs Riverpod | em aberto | ⚠️ | Manter Bloc (paridade legado + event audit) |
| D-041 | Hive vs sqflite | Hive proposto | ⚠️ | OK Hive M1; avaliar Drift M2 se cresce |
| D-042 | freezed | em aberto | ✅ | Adotar desde M1 |
| D-043 | Jailbreak handling | em aberto | ⚠️ | Seguir MASVS-R: bloquear features críticas + log |

### 0.2 P-### de domínio (10 pendências pra dual-check)

| Ref | Tema | Proposta atual | Veredito | Ajuste |
|---|---|---|---|---|
| P-CM-001 | Event bus topology | Outbox PG M1; NATS M2+ threshold | ✅ | — |
| P-CM-003 / P-EVT-001 | TenantID tipado (`CondominiumID` vs `CompanyID`) | criar VOs distintos | ✅ | Tipos distintos em Go + JSON tagged |
| P-BR-001 / D-010 | Parâmetros Reputação Bronze→Diamante | determinístico (ADR-0023) | ⚠️ | Congelar pesos em ADR próprio; publicar fórmula |
| P-INV-002 | Timezone BRT vs UTC resets | BRT | ⚠️ | Armazenar UTC, apresentar BRT, rodar jobs em BRT `America/Sao_Paulo` |
| P-CM-004 | Saga orchestrator M2+ | síncrono hoje, orch futuro | ⚠️ | Já usar orchestrator de saga desde M1 (ADR-0015) |
| P-EVT-005 / P-CM-005 | SupplierVoteSession ownership | `assembly` (com resultado → commercial via evento) | ✅ | — |
| P-INV-006 | Proposal UNIQUE draft | UNIQUE só em `status != draft` | ✅ | Partial unique index |
| P-INV-007 | Homologação obrigatória | toda assembleia | ❌→⚠️ | Obrigatória só em assembleias deliberativas (AGO/AGE); não em permanente |
| P-INV-004 | Lock 90d bypass admin | com motivo + audit | ✅ | Motivo obrigatório + dual-admin (4-eyes) para alto valor |
| P-EVT-003 | MasterPlan update exclusivo via evento | evento + handler em institutional | ✅ | — |

### 0.3 Resumo quantitativo

- ✅ Concordo: **27**
- ⚠️ Ressalva: **14**
- ❌ Discordo: **2** (D-026 SES default; P-INV-007 homologação em permanente)
- A-DC-### abertos: **6**
- D-### sugeridos pra STATE: **10**

---

## 1. Decisões STATE + ADR (detalhado)

### 1.1 ADR-0001 — Clean Arch + DDD + CQRS — ✅

**Revisão independente**: Clean Architecture + DDD tático é consenso para SaaS multi-tenant com invariantes complexas (fração ideal, vote único, quota persona-aware). CQRS leve é adequado M1-M2 sem event sourcing completo. Linter `go-arch-lint` como enforcement é sólido — já adotado por projetos Go sérios (Mattermost, Gitea). Overhead cognitivo citado nas consequências é real mas mitigável com onboarding estruturado.

**Fontes**: [Uncle Bob - Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html); [Vernon - IDDD]; consenso Go community (projectlayout `internal/modules/<bc>/`).

### 1.2 ADR-0002 — Gin abstraído — ✅

**Revisão**: abstrair Gin via `contracts.HTTPRouter` é decisão defensiva sensata (trocar por Fiber/Echo/chi sem tocar domínio). Não adiciona complexidade relevante.

### 1.3 ADR-0003 — Zitadel OIDC — ✅

**Revisão**: Zitadel self-host em M1 é boa escolha custo/feature. MFA/WebAuthn nativo + Organizations 1:1 tenants bate com multi-tenant row-based (ADR-0021). Preocupação com ops (upgrade, backup, HA) é real — registrar em DT-### pra não esquecer em M1.

**Fonte adicional**: [Zitadel self-hosting docs](https://zitadel.com/docs/self-hosting/overview) confirma production-readiness com PostgreSQL backend (ok — master-sindico já tem PG).

### 1.4 ADR-0004 — Stripe — ⚠️

**Revisão**: Stripe BR funciona e tem SAQ A, mas:
- 3.99% + R$0,39/transação é **caro** pra margem de mercado condominial BR (tickets baixos).
- Pix ainda é beta no Stripe BR (confirmei em doc oficial — Pix só saiu de beta para checkout hospedado em 2025; integração full ainda limitada).
- Pagar.me (Stone) + Pagseguro dominam BR com Pix nativo + comissão menor (~1-2%).

**Sugestão**: Manter Stripe M1 (legado + velocidade), mas criar **A-DC-001** pra reavaliar pagamento BR em M2 quando volume justificar.

**Fonte**: [Stripe BR docs](https://stripe.com/br/payments/payment-methods).

### 1.5 ADR-0005 (sqlc), ADR-0006 (pgx/v5), ADR-0007 (Goose particionado), ADR-0008 (zerolog) — ✅ cada

**Revisão em bloco**: stack Go-Postgres canônica moderna. sqlc é o padrão de fato 2024-2026 (tipado + gen em CI). pgx/v5 supera lib/pq em performance e concurrency. Goose com migrations particionadas por módulo alinha com DDD. zerolog (ou zap) é consenso de alto throughput. Nenhuma divergência.

### 1.6 ADR-0009 / D-028 — ULID vs UUIDv7 PK — ⚠️

**Proposta**: ULID como padrão; UUIDv7 aceito como equivalente.

**Revisão**: concordo **no princípio** (time-ordered PK em BYTEA composto `(tenant_id, id)` é correto), mas **o default deveria inverter**:

- **Postgres 18 lançou `uuidv7()` nativo** (RFC 9562, 2024) — 33% mais rápido que UUIDv4 + 16% throughput maior ([thenile.dev/blog/uuidv7](https://www.thenile.dev/blog/uuidv7), [saybackend.com/blog/uuidv7-postgres-comparison](https://saybackend.com/blog/uuidv7-postgres-comparison/)).
- UUIDv7 cabe em `UUID` nativo do PG (16 bytes, tooling universal) — zero fricção com clientes que esperam UUID canônico hyphen-based.
- ULID precisa encoding Crockford base32 de ida e volta em cada API boundary — fricção sem ganho funcional (ambos são 128 bits, ambos time-ordered).
- Como ADR-0017 usa Railway+Postgres, o suporte PG 18 nativo **chega grátis**.

**Ressalva — não é ❌**: a decisão "time-ordered PK" é ✅; só o escolher ULID em vez de UUIDv7 parece herança histórica.

**Sugestão — D-### novo**: trocar default ULID→UUIDv7 em novas tabelas pós PG18 upgrade; tabelas antigas ficam ULID (não migrar — custo alto).

**Fontes**: [Dave Allie - ULID vs UUIDv7](https://blog.daveallie.com/ulid-primary-keys/); [iXam comparativo](https://www.ixam.net/en/blog/2025/08/uuidv4v7ulid/); [RFC 9562](https://datatracker.ietf.org/doc/rfc9562/).

### 1.7 ADR-0010 (Mux) — ✅

**Revisão**: Mux terceirizar 100% pipeline é decisão economicamente correta pra M1-M2 em governança condominial (volume baixo). Lock 90d como VO é bonito arquiteturalmente. Preocupação com custo em M3+ é já documentada (Bunny.net/CF Stream como saída). ADR completo.

### 1.8 ADR-0011 (LiveKit SFU) — ✅

**Revisão**: LiveKit Cloud M1 → self-host sob gatilho é padrão correto. Trade-off E2EE-off vs gravação para ata (Código Civil art. 1.354) bem justificado — ata legal é requisito duro, privacidade é via tenant isolation + DPA + SRTP/DTLS. Referência cruzada Google Meet destilado apropriada.

### 1.9 ADR-0012 (ABAC + Redis 5min) — ✅

**Revisão**: ABAC matrix ordenada (admin_bypass → role → action → tenant → scope) é design limpo e debuggable. Cache TTL 5min com invalidação via webhook é trade-off aceitável (staleness max 5min ≠ trading algorítmico). Fail-closed bem especificado. Ordem matriz alinha com BeyondCorp §2.2/§3.2.

### 1.10 ADR-0013 (Timeline imutável) — ✅

**Revisão**: INSERT-only com hash-chain (SHA-256 rolling) é ouro para prova documental. RULE `DO INSTEAD NOTHING` força no nível DB. Resolução LGPD direito ao esquecimento via pseudonimização em JSONB (preserva hash original em `audit_lgpd_logs`) é a abordagem certa — alternativa (delete real) quebraria a propriedade central.

Recomendação extra (cosmética): adicionar alerta operacional em Grafana pra detectar quebra do hash-chain (forense tampering).

### 1.11 ADR-0014 (1-device policy) — ✅

**Revisão**: já incorporado em BeyondCorp posture. Device fingerprint baseado em IP+UA+canvas hash é OK mas frágil (navegador apaga canvas API) — NÃO substitui, apenas complementa auth. Está bem documentado. Sem ressalvas.

### 1.12 ADR-0015 (UoW intra / Saga inter) — ⚠️

**Revisão**: a decisão macro (UoW para 1 BC, Saga para cross-BC/provider externo) é ✅ correta. A ressalva é que o ADR **não explicita orchestration vs choreography** em cada saga canônica.

- Research 2026 ([ByteByteGo saga orch/choreo](https://blog.bytebytego.com/p/saga-pattern-demystified-orchestration); [Temporal saga patterns](https://temporal.io/blog/to-choreograph-or-orchestrate-your-saga-that-is-the-question)) converge: **híbrido** (orch para fluxos complexos 3+ steps + compensação; choreo para fluxos 1-2 steps simples).
- Das 4 sagas canônicas listadas (`ContractSaga`, `VideoPublishSaga`, `AssemblySaga`, `TrialExpirySaga`), todas têm 3+ steps + compensation semantics → **orchestration** é a escolha certa.
- `AssemblySaga` em particular cruza LiveKit (externo) + institutional (timeline imutável) + content (atas). Um orchestrator central reduz cognitive load e dá debuggability.

**Sugestão — D-### novo**: declarar explicitamente "orchestration por padrão em sagas multi-step; choreography só se saga tem 2 steps + 1 compensation e não atravessa provider externo".

**Fontes**: [Microservices.io Saga Pattern](https://microservices.io/patterns/data/saga.html); [Temporal saga mastering](https://temporal.io/blog/mastering-saga-patterns-for-distributed-transactions-in-microservices).

### 1.13 ADR-0016 (BeyondCorp) — ✅

**Revisão**: posture canônica herdada do legado, alinhada com research destilado. Zero-trust sem rede confiável é a premissa correta. Sem ressalvas.

### 1.14 ADR-0017 / D-024 (Railway M1, AWS ECS M4+) — ✅

**Revisão**: gatilhos de migração (≥500 condomínios OR SLO 99.9% OR SOC 2 OR data residency OR custo 3× AWS) são objetivos e defensáveis. Railway 40% mais barato que AWS em early-stage é confirmado ([getdeploying.com/aws-vs-railway](https://getdeploying.com/aws-vs-railway); [sealos.io railway-vs-aws](https://sealos.io/comparison/railway-vs-aws/)). Falta de multi-AZ nativo Railway é desvantagem real, mas compensada pelo SLO 99.5% inicial (ADR-0025). Fly.io reavaliação em M3 já documentada.

### 1.15 ADR-0018 (tsvector dev / OpenSearch prod) — ✅

**Revisão**: PostgreSQL `tsvector` cobre M1 sem operar cluster OpenSearch (evita cargo-cult LinkedIn §3 destilado). Migração pra OpenSearch só quando volume + necessidade de federator justificar. Bom gradualismo.

### 1.16 ADR-0019 (NATS JetStream threshold) — ✅

**Revisão**: excelente aplicação de research-backed decisions. Critério objetivo "3 produtores × 3 consumidores OR replay OR latência > 5s p95 OR coordenação fina" **evita** cargo-cult Kafka. Outbox pattern em PG cobre M1. Migração sem downtime descrita (dual dispatcher → switchover) é padrão industry. Schema de streams e ordering keys bem escolhidos (ex: `events.institutional` ordered by `condominium_id` preserva causalidade por tenant).

**Fonte adicional que reforça**: [Decodable - Revisiting the Outbox Pattern](https://www.decodable.co/blog/revisiting-the-outbox-pattern) e [microservices.io transactional-outbox](https://microservices.io/patterns/data/transactional-outbox.html) — outbox CDC (Debezium) entra como alternativa em M2-M3 se dispatcher PG virar gargalo.

### 1.17 ADR-0020 (OTel + Grafana + Sentry) — ✅

**Revisão**: stack open-source em M1 (free tiers Grafana Cloud + Sentry) economiza ~$300-500/mês vs Datadog. OTel é vendor-neutral (portável). Zero-PII em spans + lista scrub concreta (CPF, CNPJ, email, token, password) é LGPD-compliant. Head-based 1% sampling + 100% 5xx é correto — tail-based deixar M2 (precisa infra extra).

### 1.18 ADR-0021 (Multi-tenant row-based) — ✅

**Revisão**: shared DB + `tenant_id` row-based + PK composto `(tenant_id, id)` + RLS opcional é alinhado com [Google Spanner multi-tenancy patterns](https://cloud.google.com/spanner/docs/multi-tenant-patterns). Regras de enforcement (linter sqlc, RLS, cross-tenant test harness, middleware `SET LOCAL app.tenant_id`) cobrem defense-in-depth. Preocupação com noisy neighbor documentada (M3+ connection pool limits por tier) está correta.

### 1.19 ADR-0022 (Circuit breaker gobreaker) — ✅

**Revisão**: composição `Retry(CB(Bulkhead(TimeLimiter(call))))` é canônica. Configuração por provider (timeout, retry, abertura, concurrent) é tabela concreta e auditável. Fail-closed em auth/billing, fail-open em features cosméticas é a regra-ouro correta.

### 1.20 ADR-0023 (Recsys determinístico M1) — ✅

**Revisão**: 2-stage retrieval (OpenSearch) + re-rank (Go determinístico com pesos `w1..w6` versionados) é aplicação cirúrgica de lições LinkedIn/YouTube/TikTok. Proxy-métrica "contato iniciado + serviço concluído" (não CTR) herda YouTube-§5 e evita clickbait. Endpoint `/ranking-explain` resolve LGPD Art. 20 (explicabilidade de decisões automatizadas). Gatilhos de ML (M3+ sob ≥10k interações + ≥1k empresas) são objetivos.

### 1.21 ADR-0024 (Moderação cascade) — ✅

**Revisão**: trilha por volume (M1 manual → M2 cascade → M3 custom) é gradual e custo-controlado. Lock 90d vídeos força moderação pré-publish (correto para irreversibilidade). Appeals (LGPD Art. 20) dentro de 7d com revisão humana está certo. `internal/safety/content` unified library herda corretamente LinkedIn §4 destilado.

### 1.22 ADR-0025 (SLO 99.5% + error budget) — ✅

**Revisão**: 3 SLIs por BC (latência p95 + availability + correção negócio) cobre as 3 dimensões Google SRE. Budget 99.5% = 3.6h/mês é conservador-certo pra M1 solo-dev. Multi-window burn rate (1h / 6h / 24h / 72h) com thresholds objetivos (14.4× / 6× / 1× / 0.5×) é canônico Google. Regra "alert sem runbook é banido" é pro-tool. Excelente.

---

## 2. Decisões pendentes D-020..D-043

### 2.1 D-020 — Stack web: Vite + Solid Router + Tailwind (vs legado Bun + Rsbuild + UnoCSS + Kobalte) — ✅

**Revisão**: justificativa da v2 (simplificação 2026-04) é defensável:

- **Vite vs Rsbuild**: vite-plugin-solid é o caminho oficialmente documentado ([solidjs/templates](https://github.com/solidjs/templates)). Rsbuild + Solid depende de babel transform custom ([discussion #1694 solid-start](https://github.com/solidjs/solid-start/discussions/1694) confirma que "SolidJS relies on babel transform through vite-plugin-solid" — Rsbuild sem plugin oficial = fricção). Rsbuild tem pequena vantagem em build time, mas bundler não é gargalo real.
- **Tailwind vs UnoCSS**: UnoCSS é "10x mais rápido em dev" e menor em prod ([pkgpulse.com/tailwind-vs-unocss-2026](https://www.pkgpulse.com/blog/tailwind-vs-unocss-2026)) — **mas** Tailwind tem ecossistema bem maior (docs, recrutamento, plugins, AI copilots com conhecimento profundo). Para SaaS BR com equipe pequena, recrutamento > otimização de 100ms HMR.
- **Bun**: ainda imaturo como bundler ("suitable for very basic use cases" segundo HN e PkgPulse). Para dev ambiente é ok como runtime rápido, mas build prod em Vite. Decisão v2 alinha.
- **Kobalte → headless próprio**: não-mencionado explicitamente na doc, mas Solid headless UI é escasso. Observar se projeto não reinventa acessibilidade; se reinventar, voltar a Kobalte.

**Veredito ✅** com uma **A-DC-002**: validar que design system web não reinventa acessibilidade de dialog/popover/combobox (headless UI Solid é mais escasso que React — se Kobalte sair, Ark UI Solid é alternativa).

**Fontes**: [solidjs/templates](https://github.com/solidjs/templates); [solidjs rsbuild discussion](https://github.com/solidjs/solid-start/discussions/1694); [pkgpulse tailwind vs unocss](https://www.pkgpulse.com/blog/tailwind-vs-unocss-2026); [pkgpulse bun vs vite](https://www.pkgpulse.com/blog/bun-vs-vite-2026).

### 2.2 D-024 — Railway vs AWS ECS — ✅ (já consolidado em ADR-0017)

**Revisão**: ver 1.14. Sem ressalvas.

### 2.3 D-026 — SES vs Resend — ❌

**Proposta registrada**: em aberto, com tendência SES pra M1.

**Revisão independente**: **discordo do default SES em M1**. Dados 2026:

- **Resend**: $20/mês até 50k emails. Setup em horas. SDK React Email + webhooks nativos (bounces/complaints sem configurar SNS).
- **AWS SES**: $0.10/1k emails ($5/mês para 50k) — 4x mais barato. Mas exige: domain verify, DKIM, warm-up IP dedicado ($24.95/mês se quiser), configurar SNS pra bounces, lidar com sandbox-production promotion, acompanhar reputation score.

Dado:
- M1 volume baixo (primeiro cliente 2026-05-08; talvez 500-5000 emails/mês).
- Time pequeno/solo.
- **Savings SES ≈ $15/mês** em M1 não justificam 4-8h de dev + risco de bounces-loop + overhead SNS wiring.
- Mailflow Authority corrobora ([mailflowauthority.com resend-vs-aws-ses](https://mailflowauthority.com/email-comparisons/resend-vs-aws-ses)): *"for growth-stage companies at 200K+/month, SES saves $200+/month"* — mas 50K/month é Resend-territory.

**Proposta alternativa**:

| Fase | Volume estimado | Provider | Razão |
|---|---|---|---|
| M1 (2026-05) | < 10k email/mês | **Resend** | Setup fast, webhooks prontos, React Email (templates bonitos) |
| M2 (2026-Q3) | 10-50k | **Resend** | Ainda barato ($20/mês fixed), sem reconfigurar |
| M3 (2027+) | > 100k | **Migrar SES** | $0.10/1k compensa setup, dedicated IP + VDM justificam |

Abstrair via `IEmailProvider` já está em STEERING §10 → troca é trivial. `provider.IEmailProvider` com 2 impls (`resend`, `ses`) selecionáveis por env var.

**Sugestão — D-### novo em STATE**: **D-046 — Email provider: Resend M1-M2, SES M3+**. Criar ADR-0026.

**Fontes**: [Mailflow Authority Resend vs SES](https://mailflowauthority.com/email-comparisons/resend-vs-aws-ses); [AWS SES pricing](https://aws.amazon.com/ses/pricing/); [Resend pricing](https://resend.com/pricing).

### 2.4 D-028 — ULIDv7 vs UUIDv7 — ⚠️

**Revisão**: já tratado em 1.6. Sugestão = trocar default para UUIDv7 (PG18 nativo + menos encoding overhead).

### 2.5 D-029 — Fonte corpo: Inter vs Manrope — ⚠️ Inter

**Proposta**: pendente; recomendação Inter ou Manrope.

**Revisão**: **Inter** é preferível pros seguintes motivos (dados 2026):

- Inter é a fonte UI open-source mais usada em dashboards em 2026 ([figma.com best-fonts-for-websites](https://www.figma.com/resource-library/best-fonts-for-websites/); [diversekit 26-best-free-fonts](https://diversekit.com/blog/26-best-free-fonts-for-ui-and-web-design-in-2026)).
- Variable font completo (9 weights + italic).
- Métricas de x-height + aperture otimizadas para legibilidade em 14-16px (default corpo do design-system).
- Ecossistema: Figma nativo, Google Fonts, Tailwind default, npm @fontsource/inter.
- Recrutamento/familiaridade: qualquer dev BR chegou em projeto Inter.

Manrope:
- Variable desde 2019. Terminações mais suaves — ligeiramente menos formal.
- Legado web usa Manrope. Se importa paridade, manter.

**Recomendação**: **Inter** (M1 novo design). Manrope como alternativa **apenas se legado Manrope ficar em zonas compartilhadas** (landing page existente).

**Caveat**: escolher font pairing com **Michroma (wordmark)** — Inter neutro cria contraste melhor que Manrope (ambos geométricos → conflitam sutilmente).

**Fonte**: [BonFX - What Fonts Go With Inter](https://bonfx.com/what-fonts-go-with-inter/); [Untitled UI 28 Best Free Fonts](https://www.untitledui.com/blog/best-free-fonts).

### 2.6 D-030 — Icon set: Lucide vs Phosphor — ✅ Lucide

**Proposta**: pendente; recomendação Lucide.

**Revisão**: **Lucide** é a escolha certa:

- 1500+ ícones curados em grid 24×24, stroke 1.5 consistente.
- **16× mais popular** que Phosphor em download volume ([wmtips Lucide vs Phosphor](https://www.wmtips.com/technologies/compare/lucide-vs-phosphor-icons/)).
- Tree-shaking nativo em Vite/Turbopack com footprint próximo do source gzip ([codetodeploy Hidden Bundle Cost](https://medium.com/codetodeploy/the-hidden-bundle-cost-of-react-icons-why-lucide-wins-in-2026-1ddb74c1a86c)).
- `lucide-solid` existe como package first-party.

Phosphor tem mais ícones (9000+) com 6 weights — bom pra design-heavy, mas overhead de bundle 16-18× vs source gzip. Pra UI condominial, não justifica.

**Veredito ✅ Lucide**.

**Fontes**: [Lucide guide](https://lucide.dev/guide/); [frontend-hero best-icon-libraries-react](https://frontend-hero.com/best-icon-libraries-react).

### 2.7 D-031 — Dark mode — ⚠️

**Proposta**: habilitar em app autenticado via `[data-theme="dark"]` + `prefers-color-scheme` + toggle em account menu.

**Revisão**: **✅ no app autenticado + LP sempre light**. Observação:

- Tokens CSS já estruturados (`src/styles/tokens.css` com variáveis) — trivial adicionar seção `[data-theme="dark"]`.
- Toggle `system/light/dark` no account menu é padrão bom.
- Cuidado com contraste dark: matriz WCAG AA em §12 do design-system só cobre light — **faltam pares dark**. Antes de shipping dark, regenerar a matriz.

**Sugestão**: **A-DC-003** — regenerar matriz contraste WCAG AA para dark mode **antes** de habilitar feature. Caso contrário feature lança com falha AA no botão primário.

**Compliance (Rule 4 LGPD docs)**: `ui-catalog` §P-UI-005 menciona que compliance pode ficar sempre light — concordo (docs legais em dark ficam cansativos + screenshots pra MP/PROCON ficam ambíguos).

### 2.8 D-032 — Push notifications web — ⚠️

**Proposta**: backlog M2+.

**Revisão**: timing ok, mas **restrições 2026 precisam registro explícito**:

- Web Push em iOS Safari só funciona se app for **adicionado à Home Screen** como PWA ([magicbell pwa-ios-limitations](https://www.magicbell.com/blog/pwa-ios-limitations-safari-support-complete-guide)). User comum não adiciona → adoption ≈ 5-15%.
- EU: PWAs em Safari nunca recebem push (regulatório). master-sindico é BR-only → ok.
- Safari 18.4 (2025) lançou **Declarative Web Push** sem service worker — simplifica, mas ainda requer Home Screen.
- Android Chrome funciona normalmente (VAPID standard).

**Conclusão**: push web é **upgrade marginal** em iOS (only PWA-installed); push **primário continua FCM via app mobile**. Não vender push-web como feature-equivalent.

**Sugestão — A-DC-004**: documentar esta restrição em BR-WEB-PWA-003 e em [[../../04-requirements/_moc]].

**Fontes**: [magicbell using-push-notifications-in-pwas](https://www.magicbell.com/blog/using-push-notifications-in-pwas); [PushAlert Safari 16+](https://pushalert.co/blog/safari-web-push-api-support-browser-notifications/).

### 2.9 D-033 — Admin subdomain vs path — ✅ Opção A (`admin.mastersindico.com.br`)

**Proposta**: Opção A (subdomain separado) recomendado.

**Revisão**: **concordo ✅**, com reforços específicos:

- **Cookies separados**: cookie admin com `__Host-` prefix (`__Host-ms_admin_session`) — sem `Domain=`, sempre Secure, Path=/, domain-locked. Previne subdomain takeover do cookie admin ([MDN HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies); [canitakeyoursubdomain.name](https://canitakeyoursubdomain.name/)).
- **CSP mais estrita**: `admin.` com CSP distinta de `app.`/`www.` (admin sem third-party scripts, nonce-based).
- **IP allowlist** (sugestão adicional): admin acessível só via VPN interna ou IP corp allowlist — BeyondCorp já contempla device posture; admin merece camada extra.
- **Impersonation audit** (§17.2 security.md web) já resolve trilha obrigatória.

**Sugestão — D-### novo**: D-047 — Admin isolation via subdomain `admin.mastersindico.com.br` + `__Host-` cookie + CSP stricter + impersonation banner. Criar ADR-0027.

**Fontes**: [Invicti subdomain separation](https://www.invicti.com/blog/web-security/separating-subdomains-from-third-party-hosted-www-domain); [GF.dev cookie security](https://gf.dev/learn/cookie-security-secure-httponly-samesite).

### 2.10 D-040 — Bloc vs Riverpod (Flutter state) — ⚠️ Manter Bloc

**Proposta**: pendente; v2 avalia Riverpod; legado usa flutter_bloc.

**Revisão**: **manter Bloc**. Rationale:

- Consensus 2026 ([flutterstudio state-management-comparison-2026](https://flutterstudio.dev/blog/state-management-comparison-2026.html); [samioda flutter state management 2026](https://samioda.com/en/blog/flutter-state-management-2026)): Riverpod é mais moderno/menos boilerplate, **mas** Bloc é preferido para **"enterprise and regulated industries, providing event-driven audit trails, strict separation of concerns, battle-tested at scale"**.
- Master Síndico é **governança condominial regulada (LGPD + Código Civil)**: event-driven audit trail do Bloc é natural — cada `Event` pode ir para trilha de auditoria local (timeline mobile-side).
- Legado **já** usa flutter_bloc. Mudar pra Riverpod = refactor N features + testes novos = semanas perdidas sem ganho LGPD-compliant relevante.
- Integração com `go_router` + `get_it + injectable` é madura em Bloc.

**Ressalva**: considerar `bloc_concurrency` + `hydrated_bloc` para:
- `transformer: droppable()` em buttons (evitar double-tap).
- `HydratedBloc` para estado persistido offline (memberships, timeline cached).

**Sugestão — D-### novo**: D-048 — **Manter flutter_bloc** como state manager; adotar `bloc_concurrency` + `hydrated_bloc` como extensions obrigatórias.

**Fontes**: [flutterstudio.dev state-management-2026](https://flutterstudio.dev/blog/state-management-comparison-2026.html); [foresightmobile best-flutter-state-management](https://foresightmobile.com/blog/best-flutter-state-management).

### 2.11 D-041 — Hive vs sqflite vs Drift — ⚠️ Hive OK M1; avaliar Drift M2

**Proposta**: Hive.

**Revisão**: **Hive é ok para M1** dado o use-case declarado (`user`, `memberships`, `timeline:{condId}` recente, `events:{condId}`, `plano-diretor`). Tudo é key-value simples + TTL — Hive brilha nisso.

**Porém** preocupações médio-prazo:

- Hive v2 é **não-mantido** desde 2023 (repositório original `hivedb/hive` parado). Há `hive_ce` fork mantido ([community edition](https://pub.dev/packages/hive_ce)) — se adotar Hive, **usar `hive_ce` não `hive`**.
- Sem queries SQL / joins / relations — quando offline crescer (M2 offline-rich: notificações, reclamações offline, assemblies com deliberations cached), Hive vira cama-de-gato.
- Drift é ORM moderno sobre sqlite com reactive streams + tipagem forte + migrations — preferred para apps que crescem.

**Recomendação**:

| Fase | Stack cache | Razão |
|---|---|---|
| M1 (OFF-001..005) | **hive_ce** (simples KV) | Use-case é trivial; setup rápido |
| M2 (se offline-rich) | **Drift** (migrar dados Hive → Drift uma vez) | Queries + relations + reactive |
| M3 (offline sync) | Drift + Powersync ou próprio | Sync bidirecional |

**Sugestão — A-DC-005**: se adotar Hive, usar `hive_ce` (community edition mantido). Registrar em DT-### que upgrade Drift em M2 é esperado.

**Fontes**: [pub.dev hive_ce](https://pub.dev/packages/hive_ce); [vibe-studio offline-first caching hive drift](https://vibe-studio.ai/insights/offline-first-apps-caching-strategies-with-hive-and-drift-in-flutter); [objectbox flutter-databases](https://objectbox.io/flutter-databases-sqflite-hive-objectbox-and-moor/).

### 2.12 D-042 — freezed — ✅ Adotar

**Proposta**: pendente em Fase 5.

**Revisão**: **adotar desde M1**. Rationale:

- Consensus 2026 ([thelinuxcode freezed 2026](https://thelinuxcode.com/flutter-create-a-dart-model-using-the-freezed-package-practical-scalable-2026-ready/); [pub.dev/packages/freezed](https://pub.dev/packages/freezed)): *"If you're building a Flutter app in 2026 and still hand-writing data models, you're leaving quality and speed on the table."*
- Freezed 3.0 + Dart 3 unions nativas = sweet spot (some do boilerplate anterior).
- Eliminates ~30-50% do boilerplate de Entity/State/Event (copyWith, toString, hashCode, equality).
- **Bloc + freezed + sealed classes** é combo canônico moderno: `sealed class TimelineState` + subclasses `TimelineInitial / Loading / Loaded / Error` com freezed.

**Caveat operacional**: build_runner demora em projetos médios-grandes (ex: "> 30s em projetos médios em 2026"). Mitigar:
- `generate_for` targeted.
- `build_runner watch` em dev.
- CI caching.

**Sugestão — D-### novo**: D-049 — **Adotar freezed** em todas entidades, states, events, value objects a partir de M1.

**Fontes**: [freezed package pub.dev](https://pub.dev/packages/freezed); [thelinuxcode practical 2026 guide](https://thelinuxcode.com/flutter-create-a-dart-model-using-the-freezed-package-practical-scalable-2026-ready/).

### 2.13 D-043 — Jailbreak handling — ⚠️ seguir MASVS-R

**Proposta**: report em header `X-Device-Integrity` sem bloquear.

**Revisão**: **atualizar política** alinhando com OWASP MASVS-R + MASTG (2024-2026):

- OWASP MASWE-0097 ([mas.owasp.org MASWE-0097](https://mas.owasp.org/MASWE/MASVS-RESILIENCE/MASWE-0097/)) define root/jailbreak detection como **mandatory control** quando app lida com dados sensíveis/decisões críticas (votação, assembleia live, pagamento).
- `flutter_jailbreak_detection` é trivially bypassed com Frida — MASVS-R explicitamente diz: "any system-level API could be easily bypassed using reverse engineering tools".
- Solução moderna (2026): usar **freeRASP** ([RASP Flutter](https://docs.talsec.app/appsec-articles/articles/how-to-detect-jailbreak-on-flutter)) — Runtime Application Self-Protection que combina: root/jailbreak + Frida/hook detection + debugger detection + app signature + device binding.

**Política proposta**:

| Estado | Ação backend | Ação app |
|---|---|---|
| `integrity=ok` | normal | normal |
| `integrity=dev-mode` | log + audit | aviso amarelo "modo dev detectado" |
| `integrity=jailbroken` (prod) | log + audit + **bloquear** features críticas (votação, assembleia live, assinatura contrato) | modal "Este dispositivo não é seguro; use outro para votação" |
| `integrity=hooked` (Frida) | log + audit + **bloquear** sessão + forçar re-login | logout + alerta security |

Rationale:
- Não bloquear user comum em dev-mode (síndico técnico).
- **Bloquear** só as features que têm consequência jurídica (voto, assinatura, contrato).
- LGPD audit log obrigatório (operador hostil pode estar tentando fraude).

**Sugestão — D-### novo**: D-050 — **Jailbreak handling escalonado por feature criticality**. Criar ADR-0028.

**Fontes**: [OWASP MASVS](https://mas.owasp.org/); [MASWE-0097](https://mas.owasp.org/MASWE/MASVS-RESILIENCE/MASWE-0097/); [Appdome MASVS explained](https://www.appdome.com/dev-sec-blog/owasp-masvs-explained/); [Talsec Flutter RASP](https://docs.talsec.app/appsec-articles/articles/how-to-detect-jailbreak-on-flutter).

---

## 3. Pendências de domínio (10 P-###)

### 3.1 P-CM-001 — Event bus topology — ✅

**Proposta**: outbox PG M1; NATS M2+ threshold (ADR-0019).

**Revisão**: ver 1.16. Sem ressalvas adicionais. ADR-0019 já é a resposta canônica.

### 3.2 P-CM-003 / P-EVT-001 — TenantID tipado (`CondominiumID` vs `CompanyID`) — ✅

**Proposta**: criar VOs distintos `CondominiumID`, `CompanyID` em vez de `TenantID` ambíguo.

**Revisão**: **concordo ✅**. DDD + type-driven design diz:

```go
// domain/shared/ids.go
type CondominiumID ULID
type CompanyID     ULID
type UserID        ULID
```

- Previne mismatch runtime (compilador reclama se passar `CompanyID` onde se espera `CondominiumID`).
- Eventos payload carregam `tenant_type: "condominium" | "company"` + `tenant_id: <ULID>` para consumidores externos (OpenSearch, NATS) que não têm tipo Go.
- sqlc gera tipo opaco por tabela — funciona natural.
- Em ABAC matrix (ADR-0012), `tenant` step checa `user.active_tenant_id` considerando tipo.

**Caveat**: migração do legado (que usa `TenantID` genérico) é trabalho extra. Marcar como **DT-### novo** para não perder.

### 3.3 P-BR-001 / D-010 — Parâmetros Reputação Bronze→Diamante — ⚠️

**Proposta**: determinístico (ADR-0023 ranking), parâmetros exatos pendentes.

**Revisão**: determinismo ✅. **Mas faltam especificar os tiers**. Proposta:

```
Score reputação ∈ [0, 100]:
  score = 40 * rating_avg_weighted      // média de avaliações ponderadas por valor do contrato
        + 25 * success_rate             // contratos concluídos / (contratos concluídos + atrasados + cancelados por fault)
        + 15 * tenure_years_capped      // ln(1 + anos_ativo) * 15 / ln(1 + 10)
        + 10 * volume_cases_capped      // contratos_fechados capped em 50
        + 10 * content_engagement       // views vídeo-institucional + seal síndicos recebidos

Tiers:
  Bronze    [0, 40)
  Prata     [40, 60)
  Ouro      [60, 75)
  Platina   [75, 88)
  Diamante  [88, 100]
```

Rationale:
- Rating com **maior peso** (40%) — é o sinal mais forte de qualidade.
- Success rate (25%) — objetividade contratual.
- Tenure com log-cap (plafond 10 anos) — experiência sim, mas não perpetuar dinossauro.
- Volume capped — não premiar só empresa grande.
- Engagement — empresas que investem em conteúdo educativo.

**Cold start**: empresa nova sem reviews recebe "não classificado" (não Bronze-zero) por 30 dias ou 3 contratos, o que vier primeiro.

**Degradação**: reversível. Recálculo noturno (job).

**Sugestão — D-### novo**: D-051 — **Fórmula Reputação Bronze→Diamante** (valores acima). Criar ADR-0029. Primeira revisão pós-3 meses com dados reais.

**Fonte**: herança conceitual [[../../13-research/linkedin/_destilado]] §5 (sinais explícitos e interpretáveis); LGPD Art. 20 (explicabilidade).

### 3.4 P-INV-002 — Timezone BRT vs UTC — ⚠️

**Proposta**: BRT.

**Revisão**: **convenção híbrida correta**:

- **Storage**: UTC sempre (`TIMESTAMPTZ` Postgres, que normaliza).
- **Apresentação**: BRT (`America/Sao_Paulo`).
- **Jobs agendados** (reset quota anual, recálculo reputação, expiração trial): **executar em timezone BRT**, porque negócio pensa em "01/jan/00:00 horário de Brasília".

Invariante técnica:
- Go: `loc, _ := time.LoadLocation("America/Sao_Paulo")` + `time.Now().In(loc)`.
- Cron expression do job usar `TZ=America/Sao_Paulo 0 0 1 1 *` (reset anual Connect Me).
- Flutter: `timezone` package + `initializeTimeZones()`.
- Postgres: `SET TIME ZONE 'America/Sao_Paulo';` em session dos jobs; `now() AT TIME ZONE 'America/Sao_Paulo'` em queries.

**Horário de verão**: Brasil não tem desde 2019 (Decreto 9.772/2019), mas `America/Sao_Paulo` tzdata lida — **usar nome IANA** e não offset `-03:00` fixo. Se algum dia voltar DST, libc pega grátis.

**Sugestão — D-### novo**: D-052 — **Timezone canônico**: storage UTC; apresentação e jobs em `America/Sao_Paulo`; usar IANA name (nunca offset).

**Fonte**: [IANA TZ Database](https://www.iana.org/time-zones); [Decreto 9.772/2019](https://www.planalto.gov.br/ccivil_03/_ato2019-2022/2019/decreto/d9772.htm) (fim do DST BR).

### 3.5 P-CM-004 — Saga orchestrator — ⚠️

**Proposta**: saga síncrona hoje; M2+ avaliar orchestrator.

**Revisão**: **adotar orchestrator desde M1** (alinhado com ressalva 1.12 em ADR-0015).

- ADR-0015 já define `saga_instances` table com state persistido — **isso já é orchestration**, só falta explicitar.
- Sagas M1 (`ContractSaga`, `TrialExpirySaga`) têm 3+ steps → orchestrator ganha debuggability desde dia-1.
- NÃO adotar Temporal.io em M1 (overkill custo) — implementar `saga.Orchestrator` in-house em Go com `saga_instances` + worker + state machine.
- M3+ reavaliar Temporal se volume justificar.

**Arquitetura in-house sugerida**:

```go
type SagaOrchestrator interface {
    Start(ctx, sagaType, aggregateID, payload) error
    Resume(ctx, sagaInstanceID) error   // worker pega pending
    Compensate(ctx, sagaInstanceID, reason) error
}

type SagaDefinition struct {
    Steps []SagaStep         // forward
    Compensations []SagaStep // reverse order
}
```

Worker polls `saga_instances WHERE state='pending' LIMIT N FOR UPDATE SKIP LOCKED`.

**Sugestão**: explicitar em ADR-0015 (minor update) que **orchestration** é padrão. Fechar P-CM-004.

**Fontes**: [ByteByteGo saga-pattern](https://blog.bytebytego.com/p/saga-pattern-demystified-orchestration); [microservices.io saga](https://microservices.io/patterns/data/saga.html); [Temporal saga patterns](https://temporal.io/blog/mastering-saga-patterns-for-distributed-transactions-in-microservices).

### 3.6 P-EVT-005 / P-CM-005 — SupplierVoteSession ownership — ✅

**Proposta**: `assembly` é dono; `commercial` consome via evento `SupplierVoteClosed`.

**Revisão**: **concordo ✅**. Rationale:

- `SupplierVoteSession` é **materialmente uma sessão de voto em assembleia** — cabe no BC `assembly` (ubiquitous language).
- `commercial` não deve acoplar-se ao processo de voto; deve reagir a `SupplierVoteClosed(winner_company_id, contract_intent)` e avançar pipeline.
- Assim o context-map fica: `assembly --SupplierVoteClosed--> commercial` (one-way event).
- Evita shared DB schema cross-BC (regra do context-map.md v2).

Existe ambiguidade histórica no legado (tabelas em ambos BCs). Consolidar: mover tabela para `assembly.supplier_vote_sessions`; manter em `commercial` só a view materializada `supplier_vote_results_cached` atualizada via handler do evento.

### 3.7 P-INV-006 — Proposal UNIQUE draft — ✅

**Proposta**: UNIQUE só em `status != draft`.

**Revisão**: **concordo ✅**. Implementar como:

```sql
CREATE UNIQUE INDEX unique_proposal_per_request_company
    ON commercial.proposals (tenant_id, request_id, company_id)
    WHERE status <> 'draft';
```

Postgres suporta **partial unique index** nativo — solução limpa, sem trigger. Empresa pode ter N drafts concorrentes; só 1 submissão formal.

**Sugestão**: adicionar check via Specification pattern no domain ("no submit if there's existing non-draft") pra falhar cedo antes de IK violation.

### 3.8 P-INV-007 — Homologação obrigatória em toda assembleia — ❌

**Proposta**: toda assembleia (legado).

**Revisão**: **discordo**. Distinção jurídica BR:

- **Assembleia Geral Ordinária (AGO)** e **Assembleia Geral Extraordinária (AGE)** — deliberativas, precisam ata + homologação pra executar decisões (orçamento, obras, alteração convenção).
- **Assembleia em sessão permanente** (Lei 14.309/2022, [Planalto](https://www.planalto.gov.br/ccivil_03/_Ato2019-2022/2022/Lei/L14309.htm)): sessão contínua pra deliberações progressivas — homologação **por deliberação**, não por sessão.
- **Assembleias consultivas / informativas** (sem deliberação): **não** precisam homologação.

**Proposta alternativa**:

```
INV-085 (corrigida): Homologação obrigatória em assembleias DELIBERATIVAS (AGO + AGE),
antes de publicar ata + timeline. Assembleias puramente consultivas/informativas
NÃO requerem homologação.

Em sessão permanente (Lei 14.309), homologação aplica-se por DELIBERAÇÃO individual,
não por sessão.
```

**Sugestão — D-### novo**: D-053 — **Homologação escalonada por tipo de assembleia**. Criar ADR-0030 + atualizar INV-085 em [[../../01-domain/invariants]].

**Fontes**: [Lei 14.309/2022 Planalto](https://www.planalto.gov.br/ccivil_03/_Ato2019-2022/2022/Lei/L14309.htm); [Código Civil arts. 1.350-1.355](https://www.planalto.gov.br/ccivil_03/leis/2002/l10406compilada.htm); [sindiconet assembleias art 1350-1355](https://www.sindiconet.com.br/informese/assembleias-o-que-diz-a-lei-art1350-1355-administracao-assembleias-de-condominio).

### 3.9 P-INV-004 — Lock 90d bypass admin — ✅ com reforço

**Proposta**: admin pode bypass com motivo + audit log; legado menciona "só em caso legal".

**Revisão**: **concordo ✅ com reforço "4-eyes"**. Rationale:

- Lock 90d é diferencial competitivo — bypass fácil erode o valor.
- Bypass exige:
  1. Motivo textual obrigatório (≥ 50 chars).
  2. Categoria (legal_order / LGPD_takedown / content_violation / user_request).
  3. Audit log append-only (`video_lock_bypass_log`).
  4. **4-eyes policy**: 2 admins distintos aprovam (um cria request, outro aprova). Mitiga admin rogue.
  5. Notification ao dono do vídeo + sindicância opcional.

**Sugestão — D-### novo**: D-054 — **Lock 90d bypass: 4-eyes policy + motivo obrigatório + categoria + audit**. ADR-0031.

**Fonte**: princípio herdado [BeyondCorp §3.4 audit imutável](https://beyondcorp.com) + separation-of-duties security standard ([NIST SP 800-53 AC-5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)).

### 3.10 P-EVT-003 — MasterPlan update exclusivo via evento — ✅

**Proposta**: `ExecutionValidated` evento em `commercial` → handler em `institutional` atualiza MasterPlan.

**Revisão**: **concordo ✅**. Rationale:

- Mantém `commercial` sem importar `institutional/domain` (regra STEERING §14).
- Event-driven communication alinha com ADR-0019 e CLAUDE §4 (§5 BC isolation).
- Handler em `institutional/application/event_handlers/master_plan_execution_handler.go`.
- Idempotência via `event_id` dedup (padrão ADR-0019).
- **Single writer** do MasterPlan é `institutional` — só ele persiste. `commercial` nunca toca tabela `master_plans`.

**Caveat**: handler precisa UoW intra-BC (ADR-0015) — atualização MasterPlan + emissão de evento `MasterPlanAmended` (pra content/notifications) atômico.

---

## 4. Findings novos A-DC-### (dual-check)

### A-DC-001 — Stripe em BR tem alto custo + Pix limitado

**Severidade**: média.

**Descrição**: Stripe cobra 3.99% + R$0,39/trx em BR. Pix integrado ainda é beta. Pagar.me (Stone) + PagSeguro custam 1-2% + Pix nativo. Em M2+ com volume, pode justificar avaliar gateway BR-native ou multi-provider.

**Ação**: reavaliar em M2 dashboard de custo de payment-gateway vs competidores; registrar em [[../../03-subprojects/backend/requirements]].

**Link**: §1.4 desta revisão.

### A-DC-002 — Solid headless UI é escasso — validar acessibilidade dialog/popover

**Severidade**: baixa.

**Descrição**: migração de Kobalte pra headless próprio é risco de reinventar acessibilidade. Ecossistema Solid é menor que React; se Kobalte não for adotado, considerar `@ark-ui/solid` como fallback.

**Ação**: revisar [[../../03-subprojects/web/design-system]] §8 (componentes complexos) pra garantir que dialog, popover, combobox, menu, tabs usam headless lib auditada ou seguem [WAI-ARIA authoring practices](https://www.w3.org/WAI/ARIA/apg/).

**Link**: §2.1 desta revisão.

### A-DC-003 — Dark mode sem matriz contraste AA regenerada

**Severidade**: média.

**Descrição**: design-system §12 tem matriz WCAG AA só para light. Habilitar dark mode sem recalcular matriz = feature lança com violações AA (ex: botão primário ouro em fundo dark pode cair abaixo de 4.5:1).

**Ação**: regenerar matriz contraste para modo escuro antes de shipping D-031. Adicionar test CI usando [axe-core](https://github.com/dequelabs/axe-core) em todas páginas.

**Link**: §2.7 desta revisão.

### A-DC-004 — Restrição Web Push iOS Safari (só PWA home screen)

**Severidade**: baixa.

**Descrição**: push web em iOS Safari só funciona se app for adicionado à home screen como PWA. Adoption prática ≈ 5-15%. Não vender como substituto de push FCM nativo.

**Ação**: atualizar BR-WEB-PWA-003 (push notifications) com disclaimer explícito; alinhar expectativa de stakeholder.

**Link**: §2.8 desta revisão. Fonte: [magicbell PWA iOS limitations](https://www.magicbell.com/blog/pwa-ios-limitations-safari-support-complete-guide).

### A-DC-005 — Usar `hive_ce` (community edition) — Hive original não-mantido

**Severidade**: média.

**Descrição**: `hive` (hivedb) parado desde 2023. `hive_ce` fork mantido ativo. Se projeto decidir Hive, **obrigatoriamente** `hive_ce`; senão dependency rot em 2-3 anos.

**Ação**: se adotar D-041 (Hive), trocar pub package `hive` → `hive_ce`. Documentar em mobile/requirements.md. Também marcar DT-### de migração Drift M2.

**Link**: §2.11. Fonte: [pub.dev hive_ce](https://pub.dev/packages/hive_ce).

### A-DC-006 — Bloc precisa `bloc_concurrency` + `hydrated_bloc` como complementos

**Severidade**: baixa.

**Descrição**: Bloc "puro" tem dois gotchas mobile-UX:
1. Double-tap em botão dispara 2 eventos → 2 requests. `transformer: droppable()` de `bloc_concurrency` resolve.
2. Estado perdido ao matar app. `hydrated_bloc` persiste auto em storage local.

Sem esses plugins, Bloc fica aquém do que Riverpod já tem "out of the box".

**Ação**: adicionar ao pubspec.yaml + documentar em [[../../03-subprojects/mobile/patterns]].

**Link**: §2.10.

---

## 5. Recomendações consolidadas

### 5.1 D-### sugeridos para STATE

Novas decisões a registrar pós-dual-check:

| ID sugerido | Tema | Origem |
|---|---|---|
| D-046 | Email provider: Resend M1-M2, SES M3+ | §2.3 |
| D-047 | Admin isolation via subdomain `admin.` + `__Host-` cookie + CSP strict | §2.9 |
| D-048 | Manter flutter_bloc + adotar `bloc_concurrency` + `hydrated_bloc` | §2.10 |
| D-049 | Adotar `freezed` desde M1 (todas entities/states/events) | §2.12 |
| D-050 | Jailbreak handling escalonado por feature criticality (MASVS-R) | §2.13 |
| D-051 | Fórmula Reputação Bronze→Diamante (40/25/15/10/10 + tiers [0-40/40-60/60-75/75-88/88-100]) | §3.3 |
| D-052 | Timezone canônico (storage UTC, apresentação/jobs `America/Sao_Paulo`) | §3.4 |
| D-053 | Homologação escalonada (obrigatória em AGO+AGE, não em informativa/consultiva; por-deliberação em permanente) | §3.8 |
| D-054 | Lock 90d bypass: 4-eyes policy + motivo + categoria + audit | §3.9 |
| D-055 | UUIDv7 default em novas tabelas pós-PG18 (ULID legacy mantido) | §1.6 + §2.4 |

### 5.2 ADRs a criar

- ADR-0026 — Email provider cascade Resend→SES (cobre D-046).
- ADR-0027 — Admin subdomain isolation (D-047).
- ADR-0028 — Jailbreak / RASP strategy (D-050).
- ADR-0029 — Reputation formula + tiers (D-051).
- ADR-0030 — Assembly homologation scope (D-053).
- ADR-0031 — Video lock-90d bypass governance (D-054).

### 5.3 Atualizações em STATE / STEERING / ADR existentes

- **ADR-0015** — minor update: explicitar "orchestration default; choreography só se ≤ 2 steps e sem provider externo".
- **ADR-0009** — minor update: trocar default ULID → UUIDv7 em novas tabelas pós-PG18; ULID existing mantido (D-055).
- **STATE.md D-015** — fechar (observabilidade decidida em ADR-0020).
- **STATE.md D-010** — fechar com D-051 (fórmula).
- **STATE.md D-011, D-012, D-013, D-014** — permanecem abertos (não foram alvo deste dual-check).
- **P-CM-001** — fechar (ADR-0019).
- **P-EVT-005, P-CM-005** — fechar com §3.6.
- **P-INV-006** — fechar com §3.7.
- **P-EVT-003** — fechar com §3.10.

### 5.4 DT-### novos

- DT-novo-A: migrar tipos `TenantID` ambíguo → `CondominiumID`/`CompanyID` no legado (esforço medium).
- DT-novo-B: regenerar matriz contraste WCAG AA para dark mode (esforço small).
- DT-novo-C: migrar Hive → Drift em M2 se offline-rich (esforço medium-large).

---

## 6. Resumo para o usuário

### Contagem de veredito

| Categoria | ✅ Concordo | ⚠️ Ressalva | ❌ Discordo | Total |
|---|---|---|---|---|
| ADRs 0001-0025 | 22 | 3 | 0 | 25 |
| D-020..D-043 (pendentes web/mobile/cloud) | 3 | 8 | 1 | 12 |
| P-### domínio (10 pendências) | 5 | 4 | 1 | 10 |
| **Total geral** | **30** | **15** | **2** | **47** |

### A-DC-### abertos

6 findings novos:
- A-DC-001: Stripe custo BR + Pix limitado (média).
- A-DC-002: Solid headless UI escasso (baixa).
- A-DC-003: Dark mode sem matriz AA regenerada (média).
- A-DC-004: Web Push iOS Safari restrição (baixa).
- A-DC-005: Hive não-mantido → usar `hive_ce` (média).
- A-DC-006: Bloc precisa `bloc_concurrency` + `hydrated_bloc` (baixa).

### D-### sugeridos

10 novas decisões a incorporar em STATE (D-046..D-055) + 6 ADRs novas (0026-0031) + 3 DT-### novos + 2 minor updates em ADR existentes (0009, 0015).

Nenhum achado foi 🔴-bloqueante. As 2 discordâncias (❌) têm alternativa concreta e com fontes oficiais.

---

## Links

- [[_moc]]
- [[template]]
- [[../AUDIT]]
- [[../../STATE]]
- [[../../02-architecture/adr/_moc]]
- [[../../STEERING]]
- [[../../CLAUDE]]
- [[../../13-research/linkedin/_destilado]]
- [[../../13-research/google-arch/_destilado]]
- [[../../06-execution/playbooks/dual-check]]

## Fontes externas consolidadas

### ULID / UUIDv7 (D-028)
- [Dave Allie — ULIDs and Primary Keys](https://blog.daveallie.com/ulid-primary-keys/)
- [iXam — UUID v4 / v7 / ULID comparação](https://www.ixam.net/en/blog/2025/08/uuidv4v7ulid/)
- [Nile — UUIDv7 Comes to PostgreSQL 18](https://www.thenile.dev/blog/uuidv7)
- [SayBackend — PostgreSQL UUIDv7 Benchmark](https://saybackend.com/blog/uuidv7-postgres-comparison/)
- [RFC 9562](https://datatracker.ietf.org/doc/rfc9562/)

### Email providers (D-026)
- [Mailflow Authority — Resend vs AWS SES 2026](https://mailflowauthority.com/email-comparisons/resend-vs-aws-ses)
- [AWS SES pricing](https://aws.amazon.com/ses/pricing/)
- [Forwardemail — Resend vs SES comparison](https://forwardemail.net/en/blog/resend-vs-amazon-simple-email-service-ses-email-service-comparison)

### Cloud infra (D-024)
- [Sealos — Railway vs AWS](https://sealos.io/comparison/railway-vs-aws/)
- [GetDeploying — AWS vs Railway](https://getdeploying.com/aws-vs-railway)
- [AWS Fargate pricing](https://aws.amazon.com/fargate/pricing/)

### Stack web (D-020)
- [SolidJS templates (official)](https://github.com/solidjs/templates)
- [SolidJS + Rsbuild discussion](https://github.com/solidjs/solid-start/discussions/1694)
- [PkgPulse — Tailwind vs UnoCSS 2026](https://www.pkgpulse.com/blog/tailwind-vs-unocss-2026)
- [PkgPulse — Bun vs Vite 2026](https://www.pkgpulse.com/blog/bun-vs-vite-2026)

### Flutter state (D-040)
- [FlutterStudio — State management comparison 2026](https://flutterstudio.dev/blog/state-management-comparison-2026.html)
- [Samioda — Flutter state 2026](https://samioda.com/en/blog/flutter-state-management-2026)
- [Foresight — Best Flutter state management](https://foresightmobile.com/blog/best-flutter-state-management)

### Flutter storage (D-041)
- [ObjectBox — Flutter databases overview](https://objectbox.io/flutter-databases-sqflite-hive-objectbox-and-moor/)
- [Vibe Studio — Hive vs Drift offline-first](https://vibe-studio.ai/insights/offline-first-apps-caching-strategies-with-hive-and-drift-in-flutter)
- [pub.dev — hive_ce](https://pub.dev/packages/hive_ce)

### Freezed (D-042)
- [pub.dev — freezed](https://pub.dev/packages/freezed)
- [TheLinuxCode — freezed 2026 guide](https://thelinuxcode.com/flutter-create-a-dart-model-using-the-freezed-package-practical-scalable-2026-ready/)

### Jailbreak / MASVS-R (D-043)
- [OWASP MASVS](https://mas.owasp.org/)
- [OWASP MASWE-0097](https://mas.owasp.org/MASWE/MASVS-RESILIENCE/MASWE-0097/)
- [Appdome — MASVS explained 2026](https://www.appdome.com/dev-sec-blog/owasp-masvs-explained/)
- [Talsec — How to detect jailbreak Flutter](https://docs.talsec.app/appsec-articles/articles/how-to-detect-jailbreak-on-flutter)

### Icons (D-030)
- [Lucide guide](https://lucide.dev/guide/)
- [wmtips — Lucide vs Phosphor](https://www.wmtips.com/technologies/compare/lucide-vs-phosphor-icons/)
- [Frontend Hero — Best icon libraries React 2026](https://frontend-hero.com/best-icon-libraries-react)

### Fontes tipográficas (D-029)
- [Figma — Best fonts for websites 2026](https://www.figma.com/resource-library/best-fonts-for-websites/)
- [Untitled UI — 28 best free fonts 2026](https://www.untitledui.com/blog/best-free-fonts)
- [BonFX — What fonts go with Inter](https://bonfx.com/what-fonts-go-with-inter/)

### Web Push PWA (D-032)
- [MagicBell — PWA iOS limitations](https://www.magicbell.com/blog/pwa-ios-limitations-safari-support-complete-guide)
- [MagicBell — Push notifications in PWAs guide](https://www.magicbell.com/blog/using-push-notifications-in-pwas)
- [PushAlert — Safari 16+ Web Push](https://pushalert.co/blog/safari-web-push-api-support-browser-notifications/)

### Admin subdomain (D-033)
- [Invicti — Separating subdomains](https://www.invicti.com/blog/web-security/separating-subdomains-from-third-party-hosted-www-domain)
- [GF.dev — Cookie security](https://gf.dev/learn/cookie-security-secure-httponly-samesite)
- [MDN — HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [canitakeyoursubdomain.name](https://canitakeyoursubdomain.name/)

### Saga (P-CM-004 / ADR-0015)
- [ByteByteGo — Saga pattern demystified](https://blog.bytebytego.com/p/saga-pattern-demystified-orchestration)
- [microservices.io — Saga pattern](https://microservices.io/patterns/data/saga.html)
- [Temporal — Saga orchestration vs choreography](https://temporal.io/blog/to-choreograph-or-orchestrate-your-saga-that-is-the-question)

### Outbox (ADR-0019)
- [microservices.io — Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html)
- [Decodable — Revisiting the Outbox Pattern](https://www.decodable.co/blog/revisiting-the-outbox-pattern)
- [event-driven.io — Outbox + Postgres logical replication](https://event-driven.io/en/push_based_outbox_pattern_with_postgres_logical_replication/)

### Assembleias condomínio BR (P-INV-007)
- [Lei 14.309/2022 — Planalto](https://www.planalto.gov.br/ccivil_03/_Ato2019-2022/2022/Lei/L14309.htm)
- [Código Civil arts. 1.350-1.355 — Planalto](https://www.planalto.gov.br/ccivil_03/leis/2002/l10406compilada.htm)
- [Sindiconet — assembleias lei art 1350-1355](https://www.sindiconet.com.br/informese/assembleias-o-que-diz-a-lei-art1350-1355-administracao-assembleias-de-condominio)

### Timezone BR (P-INV-002)
- [IANA TZ Database](https://www.iana.org/time-zones)
- [Decreto 9.772/2019 — fim do DST BR](https://www.planalto.gov.br/ccivil_03/_ato2019-2022/2019/decreto/d9772.htm)
