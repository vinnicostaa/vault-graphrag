---
title: Pendências Fase H — Patterns a destilar de fontes arquivadas
type: index
tags: [pendencia, fase-h, master-sindico, absorption, 2026-04-25]
created: 2026-04-25
updated: 2026-04-25
status: pending-fase-h
---

# Pendências Fase H — Patterns a destilar

> Fontes em `60-sources/` que contêm padrões/recomendações úteis mas que não foram destilados em canônico durante Fase G. **Fase H** ou implementação futura deve absorver caso-a-caso conforme relevância no momento.

## 1. ARCHITECTURE_ANALYSIS_REPORT.md (2026-04-13, 67KB)

Localização: `60-sources/master-sindico/client-material/internal-docs/2026-04-13-architecture-analysis-report.md`

Doc é análise arquitetural com **score 8.5/10** (auto-rated por agente). Stack proposta: Go + Gin + GORM (canônico Go usa pgx/sqlc, não GORM) + PostgreSQL + Redis + NATS + Zitadel + OpenSearch + K8s. Algumas recomendações são úteis e ainda não foram absorvidas.

### Pendência 1.1 — BeyondCorp 4-fases roadmap

**Conteúdo**: roadmap progressivo de implementação BeyondCorp:
- **Fase 1 — Foundation** (M1): identidade Zitadel, MFA básico, ABAC engine, audit log
- **Fase 2 — Device Trust** (M2-M3): device fingerprint progressivo (D-135 já parcialmente coberto), certificate pinning mobile, OS health checks
- **Fase 3 — Context-Aware** (M3-M4): geo-fencing, behavior anomaly detection, time-based policies
- **Fase 4 — Continuous Auth** (M4+): re-autenticação contextual, step-up MFA dinâmico, session risk scoring

**Destino canônico sugerido**: `08-security/BEYOND_CORP.md` — adicionar seção "Roadmap 4 Fases" referenciando a fonte.

**Esforço**: ~1h destilação + cross-link com D-117 (Cloudflare Access), D-126 (MFA admin M1), D-135 (device fingerprint).

**Prioridade**: 🟡 M2 (após M1 estabilizar)

### Pendência 1.2 — Retry budget + jitter pattern

**Conteúdo**: pattern detalhado de retry com:
- Retry budget global por endpoint (ex: 10% das requests podem falhar e retentar)
- Exponential backoff com jitter (decorrelated jitter — Polly/AWS pattern)
- Circuit breaker thresholds
- Timeout escalation
- Dead letter queue após N falhas

**Destino canônico sugerido**: `02-architecture/patterns.md` (existente) — adicionar seção "Retry & Resilience Patterns" OU `10-knowledge/patterns/retry-budget-jitter.md` (knowledge atemporal).

**Esforço**: ~2h destilação + exemplos Go práticos.

**Prioridade**: 🟡 M2 (otimização performance)

### Pendência 1.3 — PBT shrinkers customizados + PROPERTY_TEST_SEED

**Conteúdo**: 29 properties property-based testing (gopter library Go) com:
- Custom shrinkers para tipos do domínio (FracaoIdeal, CPF, CNPJ, MoneyBRL)
- Variável env `PROPERTY_TEST_SEED` para reprodutibilidade
- Strategy: cada property roda 100 casos default, 1000 em CI nightly
- Failure analysis com seed dump

**Destino canônico sugerido**: `09-testing/pbt.md` (existente Fase C) — adicionar seção "Custom Shrinkers" + reprodutibilidade.

**Esforço**: ~1h destilação.

**Prioridade**: 🟢 M2-M3 (qualidade testes)

## 2. master-sindico-arquitetura-solucoes.md (2026-03-09)

Localização: `60-sources/master-sindico/client-material/internal-docs/2026-03-09-arquitetura-solucoes.md`

Doc do arquiteto de soluções 2026-03-09. **Maior parte já foi destilada** em sessões anteriores (enums, BCs, personas). Restou pendente:

### Pendência 2.1 — Diagrama de fluxo macro 4 high-domains

**Conteúdo**: visualização do ecossistema (Identity → Institutional → Commercial → Content) com setas de dependência + "DNS Architecture pattern" (rejeitado como Erlang OTP em Fase C, mas o **diagrama de dependências** em si é útil).

**Destino canônico sugerido**: `02-architecture/system-overview.md` — adicionar mermaid diagram.

**Esforço**: ~30min.

**Prioridade**: 🟢 nice-to-have (clareza arquitetural)

## 3. requirements(6).md (2026-04-13, 206KB) e mastersindico-requirements.pdf (2026-03-09)

Localização: `60-sources/master-sindico/client-material/internal-requirements/`

**Status**: já destilados na **Fase C** (~301 Reqs canônicos). Pendência só com decisões finais já apontadas no relatório Fase C.

## 4. JORNADAS, MÓDULOS, MATRIZES (PDFs 09-29/03)

**Status**: sendo destilados nas **Fases D, E, F** em paralelo (em curso 2026-04-25).

## 5. Sub-roles em rotas internas (D-134)

Pendência cross-cutting: app `apps/admin` é dedicada apenas pro **owner Master Síndico**. Outros sub-roles ficam em rotas internas dos produtos:
- Administradora condominial (D-114) → rotas internas em `cms` (Assembly module)
- Agência Marketing (D-102) → rotas internas em `cms` (Content module)
- Empresa Síndica (D-073/D-095) → rotas internas em `cms` (Institutional module)
- Auditor Assembly (D-132) → rotas internas em `cms` (Assembly module efêmero)

**Destino canônico**: cada produto deve ganhar seção "sub-roles internos" no respectivo `04-requirements/functional/<bc>.md` ou `03-subprojects/web/ui-catalog/<bc>/`.

**Esforço**: ~4h (4 sub-roles × ~1h cada).

**Prioridade**: 🟡 M2-M3 (após app `apps/admin` estabilizar M1).

## Vizinhos

- [[../../../50-projects/master-sindico/_archive/2026-04-24-chaos-processed/_README]]
- [[../../../50-projects/master-sindico/STATE]]
- [[../../../50-projects/master-sindico/AUDIT]]
- [[_moc]] (a criar com inventário de fontes)
