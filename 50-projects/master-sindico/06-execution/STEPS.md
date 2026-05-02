---
title: STEPS — Zero → Produção (Master Síndico v2)
type: guide
tags: [execution, steps, zero-to-prod, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/06-execution/STEPS.md (herdado) + ROADMAP v2 + cronograma-macro v2
created: 2026-04-23
updated: 2026-04-23
---

# STEPS — Zero → Produção

Sequência macro do projeto Master Síndico. Herdada do legado; adaptada pra remontagem (só docs; implementação segue no backend/web/mobile existentes).

6 fases formais, cada uma com **entrada → saída → responsável → gates**. Passar pro próximo só com gate verde.

---

## 0. Discovery

**Entrada**: visão + personas + business model não consolidados.

**Atividades**:
- Visão de produto escrita e aprovada
- Personas mapeadas (síndico N1/N2/N3, morador base/pagante, empresa Plus/Pro, parceiro vizinhança, agência de marketing)
- Business model (trial persona-aware + reputação Bronze→Diamante + 7 sub-produtos)
- Concorrência avaliada (ERPs vs proposta de governança)
- Ubiquitous language validada com dev cliente

**Saída**: [[../00-product/vision]], [[../00-product/personas]], [[../00-product/business-model]], [[../00-product/portfolio-de-produtos]], [[../00-product/glossary]].

**Responsável**: Product Owner + Arquiteto.

**Gate**: visão escrita, em `00-product/`, com 7 sub-produtos mapeados (D-003 em STATE).

**Status v2**: ✅ Concluído (Fase 0 + Fase 1 do plano de remontagem).

---

## 1. Onboarding + Setup

**Entrada**: scaffold canônico vazio.

**Atividades**:
- Dev responde questionário ([[../../vault/30-agents/playbooks/onboarding-novo-projeto]])
- Scaffold instanciado (Johnny.Decimal 00-12 + 13-research + 90-ingested)
- Stack escolhida e documentada (D-001..D-010)
- Não-negociáveis em [[../STEERING]]
- Constituição técnica aplicada ([[../CLAUDE]] §3)
- CI/CD GitHub Actions + Railway configurados
- `.claude/settings.json` por sub-projeto (backend/web/mobile)
- Branch strategy: `main` deployable; feature branches curtas
- Primeira rota `/health` em staging

**Saída**: CLAUDE.md do projeto preenchido; STATE com kick-off D-001..D-005; CI verde; `/health` em staging.

**Responsável**: Arquiteto + Dev.

**Gate**: CI verde em PR de exemplo; `/health` responde 200 em staging; ABAC engine config valida (falha startup se matriz vazia, pattern A-001 mitigado).

**Status v2**: ✅ Concluído (sprints 0-1 legado).

---

## 2. Sprints 0-N — Implementação por módulo

**Entrada**: backlog priorizado (MoSCoW) em [[../05-tasks/backlog]].

**Atividades** (loop por sprint):

### Loop SDD 5-fase (por task)

1. **Discuss** — ler requirement + design + AUDIT + STATE
2. **Plan** — escrever `<plan>` com `<verify>` (contrato de acceptance)
3. **Execute** — ordem canônica (migration → sqlc → domain → use case → mapper → repo → handler → tests)
4. **Verify** — gates verdes (build + vet + test -race + integration + gosec -sev high + govulncheck)
5. **Ship** — commit imperativo, PR, review, merge, deploy staging

### Gate 0 toda sessão

Abrir [[../11-audit/AUDIT]]. Itens 🔴🟡 **bloqueiam** task nova.

### Ordem canônica de implementação

```
1. Migration SQL + CHECK constraints (numeração particionada por módulo)
2. sqlc query (sem SELECT *)
3. Domain entity / VO (factories NewX + ReconstructX)
4. Repo interface (domain) + Domain event (se cross-context)
5. Use case CQRS (command ≠ query, arquivos separados)
6. Mapper + Repo impl (infrastructure/database)
7. Provider externo (se SDK: infrastructure/providers/<provider>/)
8. Handler + rota OpenAPI-first
9. Wire-up no module + main
10. Testes unidade + integração (+ PBT em invariants críticos)
```

**Saída**: código em `main`, deploy staging; task fechada com AC verificado.

**Responsável**: Dev + agente (SDD fluxo).

**Gate entre sprints**: `AUDIT.md` sem 🔴; coverage global ≥ 85% (domain 95%, app 90%, infra 85%, http 75%); gates CI verdes.

**Status v2**: ⏳ em curso (Sprint 10 finalizando M1).

---

## 3. QA de Sprint (weekly)

**Entrada**: sprint encerrando (domingo 23h59).

**Atividades** (domingo):
- Rodar `sprint-auditor` (skill local) — varre commits da semana, levanta findings
- Materializar findings em [[../11-audit/AUDIT]] com A-### novos
- Propor fixes pra próxima sprint (criar T-### no backlog)
- Dual-check em decisões arquiteturais tomadas na semana ([[playbooks/dual-check]])
- Revisão de deps + CVE ([[playbooks/dep-audit]] + [[playbooks/cve-scan]])
- Performance: `go tool pprof` em endpoints críticos; k6 em rotas alto-tráfego (a partir Sprint 11+)
- Retro blameless (5-10min autorreflexão): o que travou? o que ficou bom?

**Saída**: AUDIT atualizado; sprint fechado; tasks próxima sprint populadas.

**Responsável**: agente (sprint-auditor) + dev (review).

**Gate**: coverage global ≥ 85%; gosec -severity=high = 0; govulncheck = 0 reachable.

---

## 4. Staging Deploy (contínuo)

**Entrada**: PR mergeada em `main`.

**Atividades**:
- GitHub Actions → Railway staging (auto)
- Smoke test Playwright em staging (5 fluxos críticos mínimos)
- Monitor OTel + Sentry por **24h** (error rate, p95, throughput)
- Se error rate > 0.5% OU p95 > 500ms por 15min consecutivos → **rollback automático** (promote versão anterior)

**Saída**: staging estável pronto pra promoção.

**Responsável**: CI/CD + oncall.

**Gate pro prod**: 48h em staging sem regressão detectada; smoke test passando.

---

## 5. Produção Deploy (por marco)

**Entrada**: staging estável 48h; marco em janela (M1 2026-05-08, M2 2026-06-20, M3 2026-07-20).

**Pré-deploy** (checklist 48h antes):
- [ ] AUDIT.md sem 🔴 (🟡 com plano)
- [ ] STATE sem bloqueador (B-###)
- [ ] CHANGELOG atualizado
- [ ] Release Notes na PR body
- [ ] DB snapshot pré-migration (se aplicável)
- [ ] Feature flags criadas (rollout progressivo se necessário)
- [ ] Runbook revisado pra mudança operacional
- [ ] Oncall notificado (canal + PagerDuty)
- [ ] Code freeze 48h pré-deploy

**Deploy**:
- Railway `promote staging → prod` (ou canary via feature flag)
- `/health/ready` verifica PG + Redis + Zitadel + Stripe + Mux + Livekit reachable
- Smoke test em prod (endpoint crítico por sub-projeto)

**Pós-deploy (48h monitoring window)**:
- Monitorar error rate, p95, p99, throughput
- Monitorar Sentry (alert se > 10 events novos em 5min)
- Webhooks Stripe/Mux/Livekit: recebidos sem erro; DLQ vazia
- Se rollback: ver [[rollback]]

**Saída**: marco entregue ao cliente; release tag + changelog publicados.

**Responsável**: oncall + ops.

**Gates por marco**:
- **M1 (2026-05-08)**: Identity + Billing + Institutional base; frontend web + mobile; LGPD base; AUDIT 🔴 = 0
- **M2 (2026-06-20)**: Connect Me 4 fluxos + Reputação Bronze→Diamante + Vizinhança + moderação manual; chaos passing
- **M3 (2026-07-20)**: Assembly Live + LMS thin + Banco Talentos + a11y WCAG AA + SLO 99.5% mantido 7d

---

## 6. Operação Contínua (pós-M3)

**Entrada**: M3 deployado; sistema em produção estável.

**Atividades**:
- **Oncall rotation** 24/7 a partir Sprint 11 (assembleias live exigem)
- **SLO review mensal**: uptime, p95, error budget
- **Post-mortem blameless** de qualquer incidente > 15min ([[../11-audit/postmortems/template]])
- **Dep-audit recorrente** (semanal ou por release) ([[playbooks/dep-audit]])
- **CVE response** conforme SLA ([[playbooks/cve-scan]])
- **LGPD cumprimento** — requests de exclusão em ≤ 15 dias; breach notification ANPD 72h
- **Backup/restore test** trimestral
- **Rotação de secrets** trimestral (Stripe keys, Zitadel keys, Twilio tokens)

**Saída**: produto operando em regime; retro mensal alimenta próximos marcos.

**Responsável**: oncall + DPO + dev senior.

**Gate contínuo**: SLO 99.5% mantido; error budget não esgotado; AUDIT sem 🔴 persistente.

---

## Marco de transição entre fases

| De → Para | Gate |
|---|---|
| Fase 0 → Fase 1 | Visão escrita + 7 sub-produtos mapeados |
| Fase 1 → Fase 2 | Scaffold + CI verde + `/health` staging + ABAC matrix populada |
| Sprint N → Sprint N+1 | AUDIT 🔴 = 0 + coverage OK + gates verdes |
| Pré-M1 → M1 deploy | Sprint 10 fim + staging 48h limpo + runbooks prontos + LGPD DPO ciente |
| Pré-M2 → M2 deploy | Sprint 13 fim + chaos tests passing + load test 1000 users OK |
| Pré-M3 → M3 deploy | Sprint 16 fim + a11y WCAG AA + SLO 99.5% 7d |
| M3 → Escala | SLO 99.5% mantido 2 semanas + load test 10k OK + plano migração infra (D-024) |

---

## Links

- [[_moc]]
- [[dev]] — fluxo dev diário
- [[qa]] — QA sprint weekly
- [[deploy]] — staging + prod
- [[rollback]] — gatilhos e procedimento
- [[incident]] — incident response
- [[playbooks/_moc]]
- [[../ROADMAP]]
- [[../10-schedule/cronograma-macro]]
- [[../11-audit/AUDIT]]
- [[../../vault/10-knowledge/methodology/sdd-workflow]]
- [[../../vault/30-agents/playbooks/plan-and-execute]]
