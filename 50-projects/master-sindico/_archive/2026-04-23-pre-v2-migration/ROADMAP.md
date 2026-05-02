---
title: ROADMAP — Master Síndico
type: project
tags: [master-sindico, roadmap, milestones]
project: master-sindico
created: 2026-04-22
updated: 2026-04-22
---

# ROADMAP — Master Síndico

Cronograma macro do produto. Detalhamento por sprint em [[05-tasks/by-sprint/_moc]] e por calendário em [[10-schedule/cronograma-macro]].

## Marcos

### 🟢 M1 — MVP contratual · **2026-05-08**

Entregar plataforma **contratualmente usável**: síndico assina, condomínio é cadastrado, morador é vinculado, cobrança recorrente roda.

**Escopo**:
- ✅ Identity completo (auth Zitadel, 1-device, sessão)
- ✅ Billing base (trial por persona, planos, Stripe, quotas)
- ✅ Institutional base (condomínio, unidade, membership, timeline)
- ✅ Backend hardening + gates (coverage, gosec, govulncheck)
- ⏳ Frontend web: onboarding síndico + dashboard básico (Sprint 10)
- ⏳ Mobile: auth + home shell (Sprint 10)

**Critérios de aceite**:
- [ ] Síndico cria conta, escolhe plano, passa no trial
- [ ] Condomínio cadastrado com pelo menos 1 unidade, 1 membership ativo
- [ ] Timeline recebe primeira entry (onboarding + plano diretor)
- [ ] Stripe recorrência validada com evento real (não stub)
- [ ] Zitadel produção + DNS `.mastersindico.com.br`
- [ ] Observability OTel ativa, Sentry monitorando
- [ ] Docs OpenAPI publicados + README web/app atualizados

**Gates**:
- Backend 100% verdes (build, vet, test, integration, gosec, govulncheck)
- Coverage global ≥ 85%
- AUDIT 🔴🟡 zeradas

### 🟡 M2 — Connect Me + Conteúdo · **2026-06-20**

Plataforma **ativa como marketplace** + hub de conteúdo.

**Escopo**:
- Commercial: 4 fluxos Connect Me (vizinhança, academia, etc.), marketplace, banco de talentos, empresas-sindicas
- Content: upload Mux, trava 90d, busca, privacidade de métricas
- Cross-domain: integrações entre Commercial ↔ Content ↔ Institutional
- Frontend: telas Connect Me + perfis + player Mux
- Mobile: paridade das features M2

**Critérios de aceite**:
- [ ] Síndico contrata empresa via Connect Me com trilha de auditoria
- [ ] Empresa faz upload de vídeo aprovado por moderação
- [ ] Reputação Bronze→Diamante calculada e visível
- [ ] Saga do upload Mux (A-027) com compensação validada em staging
- [ ] Reputação refletida em ranking de busca

### 🔵 M3 — Assembleia + LMS + Polish · **2026-07-20**

**Governança digital** completa + ecossistema educacional.

**Escopo**:
- Assembly: live via Livekit, fila de fala, votação em tempo real, ata assinada
- LMS: cursos pra síndicos, certificação, trilhas
- Polish: performance, a11y, i18n prep, UX refino
- Observability madura: SLOs definidos, alertas, runbooks

**Critérios de aceite**:
- [ ] Assembleia ao vivo 50+ concorrentes sem degradar
- [ ] Ata publicada imutável, PDF assinado digitalmente
- [ ] Saga live session (A-028) validada em staging com caos
- [ ] LMS com 3+ cursos publicados, certificação de 10+ síndicos
- [ ] p99 API < 400ms; disponibilidade mensal ≥ 99.5%
- [ ] Runbooks pra 5 incidentes mais prováveis

## Cadência de sprints

Sprints de **1 semana** (decisão registrada em STATE). Começam segunda, terminam domingo. Gate `sprint-auditor` roda domingo.

- Sprint 0-9: entregues (ver [[05-tasks/by-sprint/_moc]])
- Sprint 10 (em curso 2026-04-22 → 2026-04-28): finalizar M1 — frontend/mobile MVP + Saga Mux/Livekit + IAuditLogger + polish backend

## Dependências externas (risco)

| Item | Risco | Mitigação |
|---|---|---|
| Zitadel cloud vs self-host | Médio — cloud depende de latência BR | Self-host via Railway + DNS, [[../02-architecture/adr/|ADR a criar]] |
| Stripe live webhooks | Baixo | Stripe CLI validado; replay em staging |
| Mux API rate-limit | Baixo | Retry + backoff [[11-audit/AUDIT]] A-033 |
| Livekit disponibilidade | Médio (assembleias críticas) | Saga com reconciliação (A-028) + failover region |
| Railway vs ECS | Médio — D-024 em aberto | Migração planejada pra M3 se escala exigir |

## Não está no roadmap (fora de escopo)

- ERP condominial (cobrança, contas a pagar, NF-e)
- Chat interno tempo real (reuso de WhatsApp pelos clientes)
- Financeiro do condomínio (prestação de contas detalhada por fornecedor)
- Internacional (foco Brasil first; i18n prep em M3, lançamento export só pós-M3)

## Revisão do roadmap

- Revisão semanal (fim de sprint) — ajuste tático
- Revisão de marco — aferir se data bate com realidade; slip > 1 semana = escalado
- Atualização deste doc obrigatória a cada `sprint-auditor`

## Links

- [[CLAUDE]]
- [[STATE]]
- [[SESSION_CHARTER]]
- [[11-audit/AUDIT]]
- [[10-schedule/cronograma-macro]]
- [[05-tasks/by-sprint/_moc]]
- [[DoR-DoD]]
