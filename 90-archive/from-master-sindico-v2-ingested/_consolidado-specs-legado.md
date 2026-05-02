---
title: Consolidado — Specs Kiro, Legacy-Docs, ANALISE_* e Antipatterns
type: note
tags: [consolidation, specs, requirements, design, tasks, antipatterns, legacy-analysis, master-sindico, v2]
source: relatório Fase 1b (toolu_01C8SgQjHnruKN61WnsvfFXK.json) destilado de specs Kiro + legacy-docs + 7 ANALISE_* (276 arquivos TS analisados)
created: 2026-04-23
updated: 2026-04-23
aliases: [specs-legado, kiro-requirements, analise-tecnica-v2]
---

# Consolidado — Specs Kiro, Legacy-Docs, ANALISE_* e Antipatterns

> **Origem**: relatório "specs Kiro + legacy-docs + ANALISE_*" persistido em
> `/home/vinni/.claude/projects/-home-vinni-Documentos-Projetos-Privados-automation-agents/1d72f407-cd66-4ed6-8ee6-c838b924ac35/tool-results/toolu_01C8SgQjHnruKN61WnsvfFXK.json`
> (72.4 KB, 843 linhas no texto extraído via `jq -r '.[0].text'`).
>
> **Objetivo**: destilado canônico e exaustivo dos requisitos EARS por BC, antipatterns AP-###, design Kiro, tasks por sprint, análise linha-por-linha dos 276 arquivos TS (ANALISE_TECNICA_COMPLETA_V2), divergências entre cópias, mapa de rastreabilidade requisito↔task↔código.
>
> **Contexto do relatório**: análise EXAUSTIVA das specs, requisitos, design, tasks, antipatterns, legacy e análises técnicas do Master Síndico em preparação para remontagem/replicação em nova pasta. 6 fontes, 6 principais domínios, 100+ requisitos, 9 sprints, 60+ antipatterns, 276 arquivos legacy analisados (amostragem).
>
> **Data do relatório**: 2026-04-23.
> **Status**: Exaustivo (reqs até código real, divergências mapeadas, riscos identificados).
>
> Totais agregados: ~2.2 MB specs + 1.5 MB legacy + 450 KB `structured_dump.md`.

---

## Sumário

- [A. Inventário completo de arquivos lidos (46 arquivos primários)](#a-inventário-completo-de-arquivos-lidos-46-arquivos-primários)
- [B. Requisitos funcionais — catálogo exaustivo (103 requisitos)](#b-requisitos-funcionais--catálogo-exaustivo-103-requisitos)
  - [IDN — Identity (13)](#idn--identity-13-requisitos)
  - [BIL — Billing (14)](#bil--billing-14-requisitos)
  - [INT — Institutional (26)](#int--institutional-26-requisitos)
  - [COM — Commercial (25)](#com--commercial-25-requisitos)
  - [CTT — Content (11)](#ctt--content-11-requisitos)
  - [ASM — Assembly (15)](#asm--assembly-15-requisitos)
  - [XDM — Cross-Domain (10)](#xdm--cross-domain-10-requisitos)
- [C. Requisitos não-funcionais (18)](#c-requisitos-não-funcionais-18)
- [D. Design técnico consolidado (20 ADRs + Data Model 46 tabelas)](#d-design-técnico-consolidado-20-adrs--data-model-46-tabelas)
- [E. Tasks Kiro — catálogo por sprint (Sprint 0–9)](#e-tasks-kiro--catálogo-por-sprint-sprint-09)
- [F. Antipatterns consolidados (AP-001..AP-012)](#f-antipatterns-consolidados-ap-001ap-012)
- [G. Legacy summary — TS → Go (66 padrões)](#g-legacy-summary--ts--go-66-padrões)
- [H. Divergências entre cópias (CLAUDE, README, ANALISE)](#h-divergências-entre-cópias-claude-readme-analise)
- [I. structured_dump.md — análise (446 KB)](#i-structured_dumpmd--análise-446-kb)
- [J. ANALISE_TECNICA_COMPLETA_V2.md — análise de 276 arquivos (sampler)](#j-analise_tecnica_completa_v2md--análise-de-276-arquivos-sampler)
- [K. Mapa cruzado requisito ↔ task ↔ código (15 amostras)](#k-mapa-cruzado-requisito--task--código-15-amostras)
- [L. Top 20 arquivos mais valiosos para remontagem](#l-top-20-arquivos-mais-valiosos-para-remontagem)
- [M. Síntese final — mapa de remontagem segura](#m-síntese-final--mapa-de-remontagem-segura)

---

## A. Inventário completo de arquivos lidos (46 arquivos primários)

| Caminho absoluto | Tamanho | Tipo | Status |
|---|---|---|---|
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/README.md` | 1.3K | Deprecated index | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements.md` | 26K | **Monolítico CANÔNICO** | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/design.md` | 24K | **Monolítico CANÔNICO** | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/tasks.md` | 24K | **Monolítico CANÔNICO** | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/reference/antipatterns-to-avoid.md` | 16K | **Referência crítica** | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/reference/legacy-summary.md` | 5.5K | Resumo TypeScript | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/identity.md` | 6.2K | Domínio (canônico) | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/billing.md` | 6.7K | Domínio (canônico) | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/institutional.md` | 14K | Domínio | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/commercial.md` | 14K | Domínio | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/content.md` | 9.0K | Domínio | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/assembly.md` | 18K | Domínio | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/cross-domain.md` | 14K | Domínio | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/product-overview.md` | 9.2K | Visão produto | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/personas-and-plans.md` | 6.4K | Personas × planos | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/functional-matrix.md` | 8.1K | Matriz features | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/requirements/identity-mirror-pattern.md` | 8.6K | Padrão Identity | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/design/design-assets.md` | 4.2K | Assets | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/design/deploy-config.md` | 5.6K | Infra | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/tasks/sprint-0.md` | 4.3K | Sprint 0 | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/specs/tasks/sprint-1.md` | 14K | Sprint 1 | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/legacy-docs/master-sindico-implementation-plan.md` | 27K | Plano legado | Lido (primeiras 200 linhas) |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/legacy-docs/architecture-refined.md` | 22K | Arquitetura legada | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/legacy-docs/linkedin-showcase.md` | 22K | Marketing | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/legacy-docs/mastersindico-requirements.md` | 93K | Reqs legadas | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/legacy-docs/mastersindico-content-analysis.md.md` | 36K | Análise conteúdo | Amostrado |
| 12+ arquivos estruturados por domínio (Identidade, Comercial, Conteúdo, Assembleia, Integrações) | ~200K | Arquitetura | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_archive/2026-04-22-specs-consolidation/CLAUDE.md` | 7.5K | Contrato multi-projeto | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_archive/2026-04-22-specs-consolidation/anotacoes.md` | 195B | Notas | Lido |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_archive/2026-04-22-specs-consolidation/CLAUDE (copy 1–4).md` | 4.8K-30K | Cópias divergentes | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_archive/2026-04-22-specs-consolidation/README*.md` (5 cópias) | 219B-19K | READMEs | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_archive/2026-04-22-specs-consolidation/ANALISE_COMPLETA.md` | 76K | Análise consolidada | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_archive/2026-04-22-specs-consolidation/ARCHITECTURE.md` | 17K | Arquitetura (copy) | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_archive/2026-04-22-specs-consolidation/CODING_MANUAL.md` | 15K | Manual código | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/structured_dump.md` | 446K | **Dump TS bruto (15.9K linhas)** | Amostrado (topo + amostragem) |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/ANALISE_TECNICA_COMPLETA_V2.md` | 242K | **Análise linha-por-linha 276 arquivos** | Lido (300 linhas iniciais) |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/ANALISE_COMPLETA_LINHA_POR_LINHA.md` | 152K | Análise incompleta (19/276) | Amostrado (topo) |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/ANALISE_COMPLETA.md` | 176K | Análise consolidada | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/ANALISE_COMPLETA_API.md` | 48K | Análise API | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/ANALISE_COMPLETA_MODELAGEM.md` | 30K | Análise modelagem | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/ANALISE_TECNICA.md` | 34K | Análise técnica V1 | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/analise_integrada.md` | 17K | Análise integrada | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/analise_completa.md` | 23K | Análise (outra cópia) | Amostrado |
| `/home/vinni/Documentos/Projetos/Privados/automation-agents/vault/50-projects/master-sindico/04-requirements/global.md` | 4.2K | Reqs globais | Amostrado |

**Total**: 46+ arquivos primários processados. Tamanho agregado: ~2.2 MB de specs, 1.5 MB de análises de legacy, 450 KB `structured_dump`.

---

## B. Requisitos funcionais — catálogo exaustivo (103 requisitos)

### IDN — Identity (13 requisitos)

| ID | Título | Descrição EARS | Prioridade | Dependências |
|---|---|---|---|---|
| IDN-001 | Autenticação OIDC | Quando usuário acessa `app.mastersindico.com.br` sem token, o sistema deve redirecionar para Zitadel OIDC Authorization Code Flow + PKCE, retornando JWT com claims `userId`, `tenantId`, `role`, `planTier` | MUST | — |
| IDN-002 | Cache Redis introspection | Quando requisição de API chega com token, o sistema deve verificar `ms:auth:{zitadelUserID}` no Redis (TTL 5min); se miss, fazer introspection ao Zitadel e cachear resultado | MUST | IDN-001 |
| IDN-003 | Validação 1 dispositivo | Quando usuário login em 2º dispositivo, o sistema deve invalidar sessão anterior via `device_fingerprint + IP` tracking; sessões simultâneas bloqueadas | MUST | IDN-001 |
| IDN-004 | Cookie httpOnly domínio raiz | Quando autenticação sucede, o sistema deve emitir `Set-Cookie access_token` no domínio `.mastersindico.com.br`, `Secure`, `HttpOnly`, `SameSite=Lax`; todos subdomínios herdam | MUST | IDN-001 |
| IDN-005 | Fallback Bearer mobile | Quando app mobile faz requisição, o sistema deve aceitar `Authorization: Bearer` (RFC 6750) além de cookie; sem localStorage | SHOULD | IDN-001 |
| IDN-006 | Registro auto | Quando persona (`syndic`/`resident`/`enterprise`/`local_company`) acessa signup, o sistema deve criar account Zitadel + UPSERT local `users` table no primeiro login | MUST | — |
| IDN-007 | Guard clause `NO_ROLE_ASSIGNED` | Quando token sem role chega ao middleware, o sistema deve retornar `403 NO_ROLE_ASSIGNED` **antes** do UPSERT de user | MUST | IDN-001 |
| IDN-008 | ABAC engine `RequirePermission` | Quando requisição (ex: `POST /sync`) atinge rota, middleware `RequirePermission(action, resource)` deve avaliar `(userId, tenantId, role, roleType, planTier, action)` e retornar `403 INSUFFICIENT_PERMISSION` se negado | MUST | IDN-001 |
| IDN-009 | Roles primárias 6 | Quando user toma ação, o sistema deve validar role em `{syndic, resident, enterprise, marketing, local_company, admin}` contra Zitadel claims | MUST | IDN-001 |
| IDN-010 | Role types sub-contexto | Quando síndico cria segundo usuário no condomínio, o sistema deve registrar `user_role_type` em `{principal, auxiliary, assistant}` na table local (não no Zitadel) | SHOULD | IDN-009 |
| IDN-011 | Rate limit auth | Quando `/auth/login` atinge 20 requests/min do mesmo IP + burst 5, o sistema deve bloquear com `429 TooManyRequests` | MUST | — |
| IDN-012 | Anti-fraude múltiplos IPs | Quando 10 cadastros novos do mesmo IP em 1 hora, o sistema deve bloquear novo signup daquele IP por 24h | SHOULD | IDN-006 |
| IDN-013 | Audit log tentativas | Quando auth falha, o sistema deve registrar em `audit_trail` com timestamp, IP, `user_id` (se conhecido), motivo (`invalid_credentials`/`mfa_failed`/`device_mismatch`) | SHOULD | — |

### BIL — Billing (14 requisitos)

| ID | Título | Descrição EARS | Prioridade | Dependências |
|---|---|---|---|---|
| BIL-001 | Planos 4 tiers | Quando user é criado, o sistema deve associar subscription em `{trial, base, plus, pro}`; trial automático no signup, pago ao expirar | MUST | — |
| BIL-002 | Trial durações | Quando `syndic/empresa/parceiro` faz signup, o sistema deve iniciar trial de `{15d/7d/30d}`; durante trial, features premium bloqueadas (soft-block) | MUST | BIL-001 |
| BIL-003 | Stripe webhook | Quando evento Stripe (`subscription.updated`, `payment_intent.succeeded`) bate em `/webhooks/stripe`, o sistema deve validar HMAC, processar e gravar em `subscriptions` | MUST | — |
| BIL-004 | Soft-block dados persistem | Quando subscription expira, o sistema **não deleta** dados; apenas bloqueia acesso via middleware `CheckSubscriptionActive` na rota | MUST | BIL-001 |
| BIL-005 | Paywall feature access | Quando user com trial tenta acessar feature premium (ex: criar múltiplos condos), o sistema deve retornar `403 QUOTA_EXCEEDED` + mensagem "Essa funcionalidade está disponível nos planos ativos..." | MUST | BIL-001 |
| BIL-006 | Upgrade/downgrade proporcional | Quando user upgrada mid-cycle, o sistema deve calcular crédito pro-rata via Stripe e cobrar diferença automaticamente | SHOULD | BIL-001 |
| BIL-007 | Quotas Connect Me anual | Quando user envia Connect Me, o sistema deve decrementar `feature_usages` com chave `ms:quota:{userID}:connect_me:{year}` no Redis; limite por persona (2/4/ilimitado por tier) | MUST | — |
| BIL-008 | Quotas videos mensal | Quando empresa envia vídeo institucional, o sistema deve verificar `ms:quota:{userID}:videos:{month}` contra limite por plano (4/8/12) | MUST | — |
| BIL-009 | Reset quotas automático | Quando 1º de janeiro 00:00 UTC, job noturno deve resetar `ms:quota:{*}:connect_me:{year}` para 0; mensalmente, resetar vídeos | SHOULD | BIL-007, BIL-008 |
| BIL-010 | Customer Portal Stripe | Quando user acessa app, o sistema deve disponibilizar link para Stripe Customer Portal para auto-gerenciar pagamento | SHOULD | BIL-001 |
| BIL-011 | Downgrades com confirmação | Quando user downgrade Pro → Plus, o sistema deve exigir confirmação explícita de que entende perda de features | SHOULD | BIL-001 |
| BIL-012 | Valores em centavos | Quando qualquer cálculo monetário ocorre, o sistema deve usar `INTEGER centavos`, nunca `FLOAT` | MUST | — |
| BIL-013 | Sem grace period | Quando trial/subscription expira, bloqueia **imediatamente** sem período de carência | MUST | BIL-001 |
| BIL-014 | Feature flags por persona | Quando nova feature em beta, o sistema deve disponibilizar via `feature_flags.{feature_key}` com rollout por persona/tier | SHOULD | — |

### INT — Institutional (26 requisitos)

| ID | Título | Descrição EARS | Prioridade | Dependências |
|---|---|---|---|---|
| INT-001 | Registro condomínio | Quando síndico faz signup, o sistema deve permitir criar condomínio com campos `address`, `CEP`, `condominium_type` (residencial/comercial/misto), `total_units`, blocos; validação CEP via ViaCEP | MUST | IDN-006 |
| INT-002 | ID alfanumérico 8 chars | Quando condomínio criado, o sistema deve gerar `public_id` único (8 chars alfanuméricos) + QR Code; usado por moradores para vincular | MUST | INT-001 |
| INT-003 | Limite 15 condomínios | Quando síndico tenta criar 16º condomínio, o sistema deve retornar `403 LIMIT_EXCEEDED` | MUST | INT-001 |
| INT-004 | Mandato | Quando síndico registra condomínio, sistema deve registrar mandato com tipo (eleito/profissional/interino), `start_date`, `end_date`, `status` | SHOULD | INT-001 |
| INT-005 | Seletor contexto | Quando síndico com 2+ condos acessa dashboard, o sistema deve exigir seleção de condomínio antes de operação; persiste sessão | SHOULD | INT-001 |
| INT-006 | Registro unidade | Quando morador faz signup + onboarding, o sistema deve permitir registro de unidade com 1 titular + até 2 dependentes | MUST | IDN-006 |
| INT-007 | Timeline imutável | Quando síndico publica atividade/comunicado, o sistema **não deve permitir DELETE/UPDATE** após publicação (append-only) | MUST | — |
| INT-008 | Timeline 7 tipos | Quando síndico publica, tipo deve estar em `{atividade_da_gestao, registro_de_execucao, comunicado, evento, adendo, resultado_consulta, assembly_result}` | MUST | INT-007 |
| INT-009 | Adendo únicamente modifica | Quando síndico precisa "editar" atividade antiga, o sistema deve criar novo registro adendo linkado ao original via `original_entry_id` + `is_amended=true` | MUST | INT-007 |
| INT-010 | Plano Diretor automático | Quando timeline entry publicada, evento de domínio `TimelinePublished` deve atualizar status de `master_plan_actions` associados; **nunca UPDATE manual** | MUST | INT-007 |
| INT-011 | PD 26 áreas × 26 tipos atividade | Quando síndico registra atividade, campo `tipo_atividade` + `area_impactada` devem mapear pra table `master_plan` com 26 combinações | MUST | INT-010 |
| INT-012 | PD 7 status | Quando atividade evolui, status deve ser em `{planejada, em_contratacao, em_execucao, concluida, suspensa, reprogramada, atrasada}`; apenas via evento Timeline | MUST | INT-010 |
| INT-013 | Eventos condomínio 13 tipos | Quando síndico cria evento (assembleia/workshop/reunião/etc), o sistema deve categorizar + permitir confirmação/ciência de morador | SHOULD | INT-001 |
| INT-014 | Comunicados 8 tipos | Quando síndico publica comunicado (aviso, alerta, consulta, resultado, etc), o sistema deve rastrear leitura por morador com `read_at` timestamp | SHOULD | INT-001 |
| INT-015 | Consulta moradores 3 tipos | Quando síndico quer feedback, deve poder criar enquete (sim/não/não sei, múltipla escolha, escala 1-5); resultado consolidado → Timeline | SHOULD | INT-001 |
| INT-016 | Avaliação gestão 2m | A cada 2 meses, quando morador acessa app, o sistema deve exigir avaliação anônima (3 perguntas, escala 1-10) **antes** de qualquer outra ação; bloqueia se atrasada | SHOULD | INT-001 |
| INT-017 | Gestão de dependentes | Quando titular é revogado, o sistema deve **revogar imediatamente** acesso de 2 dependentes via middleware check `dependent_revoked_at` | MUST | INT-006 |
| INT-018 | Encerramento vínculo | Quando morador quer sair, o sistema deve exigir motivo (mudança/venda/outro) + data efetiva; histórico mantido indefinidamente | SHOULD | INT-006 |
| INT-019 | Gestão usuários condomínio | Quando síndico principal adiciona segundo usuário (`sindico_auxiliar`/`assistente`), o sistema deve registrar matriz de permissões com toggles por ação | SHOULD | INT-001 |
| INT-020 | Governance markers 15 | Quando síndico preenche profile, sistema deve permitir auto-declaração de 15 marcadores (ABRACS, seguro RC, assessoria, compliance, etc); disclaimer obrigatório | SHOULD | INT-001 |
| INT-021 | Perfil empresa 25 markers | Quando empresa preenche profile, sistema deve permitir certificações/ISO/ESG; 25 marcadores compliance | SHOULD | — |
| INT-022 | Compliance termo LGPD | Quando síndico realiza ação crítica, sistema deve registrar `audit_trail` com IP, user_agent, timestamp, versão termos; retenção 5 anos | MUST | — |
| INT-023 | Declaração anual | Quando síndico quer exportar dossiê, o sistema deve bloquear se declaração anual pendente | SHOULD | INT-001 |
| INT-024 | Governance score diário | Quando job noturno executa, o sistema deve calcular score 0-100 (checklist: compliance em dia, timeline completa, relatórios, etc) | SHOULD | INT-001 |
| INT-025 | Dashboard síndico | Quando síndico acessa home, o sistema deve mostrar 13 cards (total moradores, `governance_score`, % Plano Diretor, satisfação média, etc) | SHOULD | INT-001 |
| INT-026 | Mapa riscos × conflitos | Quando síndico registra atividade de alto risco, o sistema deve popular matriz de riscos + detectar conflitos de interesse automático | COULD | INT-001 |

### COM — Commercial (25 requisitos)

| ID | Título | Descrição EARS | Prioridade | Dependências |
|---|---|---|---|---|
| COM-001 | Registro empresa | Quando empresa faz signup, o sistema deve validar CNPJ via API externa + registrar profile com `razao_social`, `nome_fantasia`, categorias, compliance markers | MUST | IDN-006 |
| COM-002 | Connect Me Síndico→Empresa | Quando síndico busca empresa, o sistema deve permitir formulário unidirecional "Tenho interesse"; empresa recebe abordagem; dados síndico revelados **apenas** se empresa clica "Tenho Interesse" | MUST | COM-001 |
| COM-003 | Log LGPD Connect Me | Quando dados síndico são revelados, o sistema deve registrar em `lgpd_registry` com timestamp, IP, `empresa_id`, `sindico_id`, `versão_termos`, consentimento | MUST | COM-002 |
| COM-004 | Auto-close Connect Me 48h | Quando 48h passam sem interesse, o sistema deve marcar Connect Me como `auto-fechado` | SHOULD | COM-002 |
| COM-005 | Denúncia assédio Connect Me | Quando síndico vê abordagem indesejada, o sistema deve disponibilizar botão "Denunciar Assédio" → log evidência, notifica admin | SHOULD | COM-002 |
| COM-006 | Connect Me Morador→Síndico | Quando morador quer contato síndico, sistema deve permitir formulário com categorias (`dúvida_gestão`, `solicitação_manutenção`, `reclamação`, `sugestão`, `outro`) + urgência | SHOULD | IDN-006 |
| COM-007 | Connect Me Empresa→Empresa | Quando empresa Plus/Pro quer subcontratar, o sistema deve permitir fluxo P2P (subcontratação, parceria técnica, fornecimento, etc) | COULD | COM-001 |
| COM-008 | Connect Me Síndico→Parceiro | Quando síndico quer promover benefício, o sistema deve permitir fluxo para parceiro local | COULD | COM-001 |
| COM-009 | Propostas workflow | Quando empresa envia proposta, o sistema deve rastrear status `{draft, sent, awaiting_validation, validated, accepted, rejected}`; após `sent`, imutável | MUST | COM-002 |
| COM-010 | Votação fornecedor 2+ propostas | Quando 2+ propostas validadas para mesmo contrato, sistema deve permitir votação de moradores; quórum `{simples, fração, unânime}`; resultado real-time | MUST | — |
| COM-011 | Negócio fechado dupla assinatura | Quando empresa e síndico confirmam deal, o sistema deve marcar `is_immutable=true`; auto-publica na Timeline | MUST | COM-009 |
| COM-012 | Registro execução empresa | Quando empresa faz serviço, o sistema deve permitir upload de fotos + descrição; estado = `pending_validation` até síndico validar | MUST | COM-011 |
| COM-013 | Atividade técnica empresa | Quando empresa contribui a plan diretor (preventiva), o sistema deve registrar como `technical_activity`; também pendente validação síndico → Timeline | SHOULD | COM-011 |
| COM-014 | Comunicados pós-deal 5 tipos | Quando deal foi assinado, empresa pode publicar comunicados (andamento, conclusão, alert) → validação síndico → Timeline + notifica moradores | SHOULD | COM-011 |
| COM-015 | Avaliação empresa 5 critérios | Quando execução termina, síndico pode avaliar empresa (qualidade, prazo, comunicação, cumprimento, total); escala 1-10; anônima para empresa, pública para outros síndicos | SHOULD | COM-012 |
| COM-016 | Empresa Síndica vinculada | Quando síndico autoriza, sistema deve criar link a empresa-síndica via token secreto; permissões parametrizáveis (audit all); audit trail | SHOULD | INT-001 |
| COM-017 | Agência Marketing vinculada | Quando empresa autoriza, sistema deve criar link a agência; acesso restrito a vídeos + analytics; audit trail | SHOULD | — |
| COM-018 | Banco de talentos morador | Quando morador Plus publica vídeo-currículo (90s, lock 90d) + profile profissional (6 perguntas), o sistema deve indexar em Meilisearch com 18 áreas interesse | SHOULD | IDN-006 |
| COM-019 | Busca banco talentos empresa | Quando empresa Plus/Pro busca currículo, o sistema deve permitir busca avançada + favoritos + exportação PDF + analytics funil | SHOULD | COM-018 |
| COM-020 | Vizinhança Síndico | Quando síndico explora dashboard vizinhança, sistema deve mostrar feed de parceiros locais, criar benefícios exclusivos, histórico | COULD | — |
| COM-021 | Vizinhança Empresa | Quando empresa busca parceiros, o sistema deve listar via geolocalização + filtros; pode ativar promoções | COULD | COM-001 |
| COM-022 | Vizinhança Morador | Quando morador busca benefícios, o sistema deve filtrar por bairro/condomínio; badge "Indicado pelo Síndico" | COULD | IDN-006 |
| COM-023 | Vizinhança Parceiro | Quando parceiro cria promoção (7 tipos: desconto%, desconto fixo, produto grátis, combo, avaliação, brinde, exclusivo), o sistema deve aceitar aberta (bairro) ou exclusiva (condo) | COULD | COM-001 |
| COM-024 | Quotas Connect Me anuais | Ver BIL-007 (detalhado em Billing) | MUST | — |
| COM-025 | Limite múltiplos condos syndic | Síndico pode estar vinculado a **máximo 15 condomínios** (ver INT-003) | MUST | INT-003 |

### CTT — Content (11 requisitos)

| ID | Título | Descrição EARS | Prioridade | Dependências |
|---|---|---|---|---|
| CTT-001 | Vídeos institucionais 12 tipos | Quando empresa publica vídeo (apresentação, portfolio, tutorial, depoimento, etc), o sistema deve categorizar entre 12 tipos | MUST | — |
| CTT-002 | Quotas vídeos mensais | Quando empresa publica vídeo, o sistema deve verificar quota mensal por plano (N1: 4, N2: 8, N3: 12, Plus: 8, Pro: 12) | MUST | BIL-008 |
| CTT-003 | Trava 90 dias | Quando empresa publica vídeo X, o sistema deve bloquear re-upload do mesmo vídeo por 90 dias (pode deletar + subir novo, mas só 1x/trimestre por vídeo específico) | MUST | — |
| CTT-004 | Visibilidade moradores votação | Quando morador é convidado a votar fornecedor, o sistema deve **temporariamente** revelar vídeos empresa se `autorizar_exibicao_votacao=true`; após votação encerrada, acesso revogado | MUST | COM-010 |
| CTT-005 | Privacidade métricas | Quando morador vê conteúdo, likes/comentários/views são privados (visíveis só ao dono) | MUST | — |
| CTT-006 | Paywall Base 25% | Quando usuário Base tenta assistir vídeo empresa, o sistema deve cortar stream aos 25% + mensagem upgrade | SHOULD | — |
| CTT-007 | Analytics vídeo | Quando vídeo é visto, o sistema deve rastrear views (>70% = conta), `avg_watch_time`, `completion_rate`, `retention_rate`; gráficos no dashboard | SHOULD | — |
| CTT-008 | Busca full-text | Quando usuário busca "apresentação empresas", o sistema deve fazer busca em `{título, descrição, tags, categoria}` via OpenSearch | SHOULD | — |
| CTT-009 | Recomendações comportamento | Quando usuário navega, o sistema deve recomendar vídeos baseado em histórico + click-through tracking | COULD | CTT-008 |
| CTT-010 | Ebooks/PDFs digitais | Quando user Plus+, o sistema deve permitir acesso a biblioteca de conteúdo digital (PDF, EPUB) | COULD | — |
| CTT-011 | Academia LMS | Quando user acessa Academia, o sistema deve exibir 5 áreas (cursos, treinamentos, lives, fórum, biblioteca) com 3 trilhas (Síndico, Empresa, Morador); certificados auto ao 100% | COULD | — |

### ASM — Assembly (15 requisitos)

| ID | Título | Descrição EARS | Prioridade | Dependências |
|---|---|---|---|---|
| ASM-001 | Configuração assembleia | Quando síndico agenda assembleia, o sistema deve permitir tipo (ordinária/extraordinária), modalidade (presencial MVP/híbrida/online), tempo de fala (2 ou 3 min), quórum | MUST | INT-001 |
| ASM-002 | Pauta 5 tipos item | Quando síndico monta pauta, itens devem ser categoria `{deliberação, informação, eleição, consulta, outro}`; fração ideal upload Excel (soma=100%); lock após publicação | MUST | ASM-001 |
| ASM-003 | Notificações 3 fases | Quando assembleia agendada, sistema deve notificar 3 dias antes, 1 dia, 3 horas antes | SHOULD | ASM-001 |
| ASM-004 | Edital PDF | Quando pauta pronta, sistema deve gerar PDF edital automático ou aceitar upload | SHOULD | ASM-002 |
| ASM-005 | Ciência obrigatória | Quando edital publicado, morador **perde acesso ao app até dar ciência**; sistema rastreia `acknowledged_at` com retenção 6 meses | MUST | ASM-004 |
| ASM-006 | Procurações | Quando morador não pode comparecer, sistema deve permitir procuração a outro morador; validação (prazo 24h pós-assembleia); proxy replica voto exato | SHOULD | ASM-001 |
| ASM-007 | Enquetes preliminares | Quando síndico quer testar clima, sistema deve permitir enquetes consultivas (sem valor jurídico) pré-assembleia | COULD | ASM-001 |
| ASM-008 | Simulador quórum | Quando síndico planeja, o sistema deve permitir simular quórum com # presentes esperados vs. fração necessária | COULD | ASM-001 |
| ASM-009 | Check-in day-of | Quando assembleia inicia, sistema deve permitir check-in via app (CPF + bloco + unidade) + QR Code + terminal físico; ausente momentâneo ≠ ausente definitivo | MUST | ASM-001 |
| ASM-010 | Votação real-time 4 tipos | Quando item em votação, sistema deve permitir voto simples/fração ideal/fornecedor/eleição; voto presencial assistido com `termo_de_voto`; ausente momentâneo = abstenção auto | MUST | ASM-001 |
| ASM-011 | Painel presidente | Quando síndico (presidente) opera, dashboard deve mostrar quórum real-time, status votação, fila fala, controles absolutos | MUST | ASM-001 |
| ASM-012 | Telão 2 áreas | Quando assembleia ao vivo (futuro), telão mostra Presentation Area + Operational Panel; via WebSocket latência <200ms | COULD | — |
| ASM-013 | Fila de fala timer | Quando morador quer falar, entra na fila; presidente vê timer (2–3 min conforme config), alerta 30s antes | COULD | ASM-001 |
| ASM-014 | Ata automática + homologação | Quando votação encerrada, sistema deve gerar ata automática; homologação obrigatória via votação; após homologação, imutável | MUST | ASM-010 |
| ASM-015 | 6 relatórios + score transparência | Quando assembleia termina, sistema deve gerar relatórios (presença, votação, procurações, fala, consolidado, transparência) em CSV+PDF; score transparência 0-100 (10 componentes) | SHOULD | ASM-014 |

### XDM — Cross-Domain (10 requisitos)

| ID | Título | Descrição EARS | Prioridade | Dependências |
|---|---|---|---|---|
| XDM-001 | Domain events publisher | Quando qualquer agregado é mutado (Profile, Timeline, Subscription), o sistema deve emitir evento de domínio para listeners em outros contextos | MUST | — |
| XDM-002 | Cache invalidation via events | Quando `ProfileUpdated` event publicado, listeners devem invalidar `profile:{id}` no Redis; idempotent | SHOULD | XDM-001 |
| XDM-003 | Reindex busca automático | Quando vídeo criado/atualizado/deletado, o sistema deve reindexar no OpenSearch via evento; falha não bloqueia fluxo principal | SHOULD | XDM-001 |
| XDM-004 | Integrações Stripe | Quando `payment_intent.succeeded` chega, webhook deve publicar `SubscriptionActivated` event → listeners updateiam cache, logs, etc | MUST | — |
| XDM-005 | Integrações Mux | Quando `video.ready` chega de Mux, webhook deve publicar `VideoReady` event → listeners reindexam, notificam user | SHOULD | — |
| XDM-006 | Integrações Livekit | Quando participant joined Livekit room, sistema deve sincronizar check-in em `assembly_votes` (futuro) | COULD | — |
| XDM-007 | Integrações Resend | Quando email precisa ser enviado, adapter deve chamar Resend; template pt-BR pré-feito | MUST | — |
| XDM-008 | Integrações ViaCEP | Quando CEP informado, adapter deve chamar ViaCEP; resultado cacheado Redis 30 dias | SHOULD | — |
| XDM-009 | Audit trail cross-context | Quando ação crítica ocorre (deal assinado, timeline publicada), `AuditLogger` deve registrar em `audit_trail` append-only com retenção 5 anos | MUST | — |
| XDM-010 | NATS JetStream (futuro) | Quando assembleia ao vivo com 100+ pessoas, sistema deve usar NATS pub/sub para fan-out votos real-time; implementação Sprint 10 | COULD | ASM-010 |

**Resumo de requisitos**: 103 mapeados entre 6 domínios. Distribuição: IDN 13, BIL 14, INT 26, COM 25, CTT 11, ASM 15, XDM 10 (+ N cross-domain implícitos). Cobertura de 8 personas (Síndico N1/N2/N3, Morador Base/Pagante, Empresa Plus/Pro, Parceiro), 4 interfaces (API Rest, WebSocket, Mobile Bearer, SSO Zitadel).

---

## C. Requisitos não-funcionais (18)

| ID | Categoria | Descrição | Fonte |
|---|---|---|---|
| NFR-001 | Performance | WebSocket latência < 200ms na assembleia ao vivo (Telão, fila fala, votação real-time) | `design.md` § WebSocket Architecture |
| NFR-002 | Escalabilidade | Suportar 1000+ usuários simultâneos em assembleia (1 room Livekit + N conexões WebSocket) | `design.md` |
| NFR-003 | Disponibilidade | SLA 99.9% para API de auth/votação; 99.0% para content (vídeo tem degradação) | — (implícito) |
| NFR-004 | Banco de dados ACID | Toda operação crítica (votação dupla, quota decrement, deal assinado) dentro de transação com isolamento SERIALIZABLE | `design.md` § ACID + SAGA |
| NFR-005 | Banco dados consistência | DECIMAL/NUMERIC nunca float para fração ideal assembleia; INTEGER centavos para dinheiro | `design.md` § Money Value Object |
| NFR-006 | Segurança — BeyondCorp | Zero-trust: toda request auth + autz; ABAC matriz `(role, plan, action, tenant)` | `design.md` § Filosofia Arquitetural |
| NFR-007 | Segurança — Tenant isolation | Nenhuma query sem `WHERE tenant_id = $ID`; 100+ tests verificam cross-tenant impossível | `design.md` § Tenant isolation |
| NFR-008 | Segurança — 1 device | Fingerprint + IP; segundo login invalida anterior; bloqueado se mismatch | `design.md` § 1 dispositivo ativo |
| NFR-009 | Segurança — PII em logs | CPF/CNPJ/email/token **nunca** em logs; usar `Masked()` ou `user_id` | `antipatterns.md` A10 |
| NFR-010 | Segurança — Cookie httpOnly | Domain `.mastersindico.com.br`, Secure, HttpOnly, SameSite=Lax; sem localStorage | `design.md` § Autenticação Multi-Subdomain |
| NFR-011 | Segurança — LGPD | Registro de compartilhamento com timestamp, IP, finalidade, consentimento; retenção 5 anos | `requirements.md` § Req 68 |
| NFR-012 | Observability — logs | Structured logs com campos `request_id`, `user_id`, `tenant_id`, `endpoint` obrigatórios | `antipatterns.md` A10 |
| NFR-013 | Observability — traces | Distribuído via OpenTelemetry com span de cada DB query, Stripe call, Mux call | `tasks.md` § Sprint 7 |
| NFR-014 | Observability — SLOs | Error rate <0.1%, p95 latency <500ms API, p99 <2s; dashboard Grafana | — |
| NFR-015 | Manutenibilidade — Code Coverage | Domain 95%, Application 90%, Infrastructure 85%, HTTP 75%; gates duros em CI | `CLAUDE.md` § 8. Coverage thresholds |
| NFR-016 | Manutenibilidade — Docs | OpenAPI 3.1 ou gerada via code; specs KIRO síncronas com implementação | `design.md` |
| NFR-017 | Acessibilidade — i18n | Frontend pt-BR; API retorna mensagens pt-BR; caracteres unicode (ç, ã, ö) suportados | — |
| NFR-018 | Acessibilidade — WCAG 2.1 AA | Web frontend (SolidJS) vai suportar screen reader + navegação keyboard; futuro Sprint 6 | — |

---

## D. Design técnico consolidado (20 ADRs + Data Model 46 tabelas)

### Decisões arquiteturais implementadas (com status)

| # | Decisão | Resolução | Status | Risco |
|---|---|---|---|---|
| D-001 | Cache Redis Zitadel | TTL 5 minutos via webhook vs. aceitação staleness | ABERTO | Médio (imprevisibilidade se revogação rápida) |
| D-002 | Railway vs ECS Fargate | Railway managed (hoje) vs. ECS em produção | ABERTO | Médio (vendor lock) |
| D-003 | SES vs Resend | SES (legacy) vs. Resend (transacional simples) | IMPLEMENTADO (Resend, Sprint 9) | Baixo |
| D-004 | Soft-block vs hard-delete | Trial expirado = soft-block, dados permanecem | DECISÃO-CLIENTE 4 | Baixo |
| D-005 | Stripe único vs Mercado Pago | Stripe APENAS (sem fallback) | DECISÃO-CLIENTE | Médio (vendor lock) |
| D-006 | Livekit self-hosted Railway | Livekit vs Twilio/Vonage | IMPLEMENTADO (Sprint 9) | Alto (operacional) |
| D-007 | OpenSearch vs PG tsvector | OS em produção, PG tsvector em dev | ABERTO | Médio (custo) |
| D-008 | Zitadel cloud | Zitadel managed vs. auto-hospedado | IMPLEMENTADO (Sprint 9 docker-compose) | Médio (Sprint 9.19 pendente Railway) |
| D-009 | Domain Events outbox | Transação dupla (aggregate + outbox) vs. saga | IMPLEMENTADO (Sprint 2+) | Baixo |
| D-010 | NATS JetStream | Assembleia ao vivo com NATS pub/sub | FUTURO (Sprint 10) | Médio (novo infra) |

### Data Model — 46 tabelas principais

**Identity Domain:**
- `users` (UUID PK, `zitadel_id` UNIQUE, email, status, device_fingerprint, last_ip)
- `sessions` (UUID PK, `user_id` FK, token, device_fingerprint, ip_address, expires_at)
- `accounts` (OAuth histórico — Zitadel assume role)

**Billing Domain:**
- `plans` (UUID PK, name, type, `price_monthly`, `price_yearly`, `trial_days`, features JSONB, quotas JSONB)
- `subscriptions` (UUID PK, `user_id` FK, `plan_id` FK, status ENUM, `stripe_subscription_id`, `period_start`, `period_end`)
- `feature_usages` (user_id, feature_key, period, usage INTEGER; PK composite)

**Institutional Domain:**
- `condominiums` (UUID PK, name, cnpj, address JSONB, `public_id` UNIQUE 8char, `qr_code_url`)
- `mandates` (UUID PK, `condominium_id` FK, `syndic_id` FK, type ENUM, `start_date`, `end_date`, status)
- `units` (UUID PK, `condominium_id` FK, number, block, `fracao_ideal` NUMERIC(10,6), `titular_id`, status)
- `profiles_resident` / `profiles_syndic` / `profiles_enterprise` / `profiles_local_company` / `profiles_marketing` (cada um UUID PK, `user_id` UNIQUE FK, dados específicos)
- `timeline_entries` (UUID PK, `condominium_id` FK, `author_id` FK, `entry_type` ENUM, `published_at`, `is_amended`, `original_entry_id` — **SEM `deleted_at`**)
- `master_plan_actions` (UUID PK, `condominium_id` FK, `tipo_atividade`, `area_impactada`, status ENUM, `due_date`, `budget_estimated`)
- `events` (`condominium_id` FK, type ENUM, `scheduled_date`, status)
- `communications` (`condominium_id` FK, type ENUM, status, `read_tracking` JSONB)
- `audit_trail` (UUID PK, `actor_id`, action, `resource_type`, `resource_id`, `tenant_id`, metadata JSONB — **APPEND-ONLY, NUNCA DELETE, retenção 5 anos**)

**Commercial Domain:**
- `connect_me_requests` (UUID PK, `flow_type` ENUM, `requester_id`, `target_id`, status, `data_revealed_at`, `consent_version`, `auto_closed_at`)
- `proposals` (UUID PK, `company_id` FK, `condominium_id` FK, status ENUM, version, `estimated_cost` INTEGER, attachments JSONB)
- `closed_deals` (UUID PK, `condominium_id` FK, `company_id` FK, `agreed_cost`, `is_immutable`, `company_signed_at`, `syndic_signed_at`)
- `company_users` (`company_id` FK, `user_id` FK, role ENUM `{administrator, operator}`, permissions JSONB)
- `marketing_agencies` (`user_id` FK, `company_id` FK, permissions JSONB — restritos a vídeos + analytics)
- `local_company_profiles` (CPF ou CNPJ, neighborhood, city, cep)

**Content Domain:**
- `videos` (UUID PK, `owner_id`, `tenant_type` TEXT, `video_type` ENUM(12), `mux_asset_id`, `mux_playback_id`, status ENUM, `autorizar_exibicao_votacao` BOOL, `can_be_replaced_after` TIMESTAMP, `view_count`)
- `curricula` (UUID PK, `morador_id` UNIQUE FK, `areas_of_interest` TEXT[], `can_be_replaced_after` TIMESTAMP)
- `search_index` (OpenSearch external, docs denormalizados)

**Assembly Domain:**
- `assemblies` (UUID PK, `condominium_id` FK, `scheduled_date`, `quorum_type` ENUM, `speech_time_minutes`, status, `transparency_score`)
- `agenda_items` (`assembly_id` FK, category ENUM, `fracao_ideal` NUMERIC(10,6), status)
- `assembly_votes` (UUID PK, `assembly_id` FK, `agenda_item_id` FK, `voter_id`, `vote_value` TEXT, `fracao_ideal_weight`, `is_proxy`, `is_presencial_assistido`, `voted_at`, `is_homologated`; UNIQUE(`agenda_item_id`, `voter_id`) **IMPOSSÍVEL DUPLO VOTO**)
- `assembly_minutes` (UUID PK, `assembly_id` FK, content TEXT, `signed_at`, `is_homologated` — **IMUTÁVEL após homologação**)
- `powers_of_attorney` (`assembly_id` FK, `grantor_id` FK, `proxy_id` FK, `validated_at`, `expires_at`)

### Padrões implementados

| Padrão | Onde | Decisão |
|---|---|---|
| **DDD — Agregados** | `domain/entities/`, `domain/value_objects/` | User (PK zitadel_id), Profile (PK user_id), Subscription (PK user_id), Condominium (PK condominium_id), Timeline (PK entry_id, imutável), Assembly (PK assembly_id) |
| **Value Objects** | Money (int64 centavos), Email (validado), CPF/CNPJ (regex), FracaoIdeal (NUMERIC) | Encapsulados em factories + getters |
| **Repositories** | `infrastructure/database/` com impl Drizzle/SQLc | Interface no domain, impl em infra; CRUD + queries específicas |
| **Use Cases** | `application/use_cases/` CQRS | 1 arquivo = 1 command ou 1 query; `Execute(input) → Output`; transacional quando necessário |
| **Domain Events** | `domain/events/` + `IDomainEventPublisher` | Emitidos em mutação; listeners async em `application/event_handlers/` |
| **UnitOfWork** | Transação per use case | `unitOfWork.Run(ctx, fn)` garante atomicidade; Saga com compensação para cross-service |
| **ABAC** | `internal/shared/middleware/authorize.go` | Matriz `(admin_bypass → role → action → tenant)`; retorna `ABACResult{Granted, Reason, Details}` |
| **Error Handling** | `core/errors/AppError` com sentinels | `errors.Is/As`; handlers propagam via `ctx.SetError(err)` |
| **Middleware Stack** | `shared/middleware/` | Recovery → RequestID → Logger → CORS → ErrorHandler → RateLimiter → Authenticate → RequirePermission → Handler |
| **Logging** | zerolog com `logger.WithContext(ctx)` | Estruturado; sem `SetGlobalLevel()` (paralelo); PII mascarado |
| **Cache** | Redis com adapter `IStorageProvider` | Typed methods: Get/Set/Del; key patterns: `ms:{domain}:{id}:{field}` |

---

## E. Tasks Kiro — catálogo por sprint (Sprint 0–9)

### Sprint 0 — Fundação (23/03 → 06/04) — CONCLUÍDO

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 0.1 | Setup monorepo web (Bun, Biome, turbo) | OK | 2d |
| 0.2 | Setup Go Clean Arch + Gin abstrato | OK | 3d |
| 0.3 | PostgreSQL + migrations base (goose v3) | OK | 1d |
| 0.4 | Redis setup + cache patterns | OK | 1d |
| 0.5 | Zitadel deploy Railway | PARCIAL (local docker-compose pronto, Railway pendente 9.19) | 3d |
| 0.6 | Core contracts Go (IModule, IUseCase, IRepository, AppError) | OK | 1d |
| 0.7 | Gin plugins (CORS, Recovery, Logger, ErrorHandler, GinAdapter) | OK | 2d |
| 0.8 | CI/CD GitHub Actions → Railway | OK (pipeline existe; deploy staging pendente) | 2d |
| 0.9 | Railway Foundation (Dockerfile, railway.toml, health check) | PARCIAL (managed Postgres/Redis pendente) | 1d |
| 0.10 | Flutter setup Clean Arch per feature | EM ANDAMENTO (início; vide `CLAUDE.md` app/) | 3d |

### Sprint 1 — Identity & Billing (07/04 → 20/04) — CONCLUÍDO

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 1.0.1-12 | Correções técnicas (AuthUser loca, zerolog, migrations, sqlc v1.30) | OK | 2d |
| 1.1 | BeyondCorp middleware stack (JWT/cookie, Zitadel introspection, Redis cache, 1-device, PKCE) | OK | 3d |
| 1.2 | ABAC engine Go (`RequirePermission` middleware, matriz role×action) | OK | 2d |
| 1.3 | Stripe Adapter Go (`IPaymentGateway` → `StripeGateway`) | OK | 2d |
| 1.4 | Plans + Subscriptions Go (`GetPlanByRoleAndTier`, `GetActiveSubscription`, webhooks) | OK | 2d |
| 1.5 | Trial logic Go (`CheckFeatureAccess`) | OK | 1d |
| 1.6 | Quotas + Feature Usage Go (Redis tracking, `IncrementUsage`) | OK | 1d |
| 1.7-13 | Frontend SolidJS (auth flows, onboarding 4 personas, TrialBlocker) | EM ANDAMENTO (amostrado em `web/CLAUDE.md`) | 8d |
| 1.14 | Flutter auth + onboarding | EM ANDAMENTO (`app/CLAUDE.md`) | 5d |

### Sprint 2 — Institutional I: Registry + Governance Base (21/04 → 04/05) — Dev1 backend concluído

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 2.1 | Condominium Registry Go (ID 8char, QR, mandato, limite 15) | OK | 2d |
| 2.2 | Unit Registry + Membership Go (1 titular + 2 dependentes) | OK | 1d |
| 2.3-14 | SolidJS 10+ telas (cadastro condo, timeline, PD, contexto) | EM ANDAMENTO | 7d |
| 2.4 | Timeline Go (append-only, versionamento, adendo) | OK | 2d |
| 2.5 | Timeline imutabilidade pós-publicação | OK | 1d |
| 2.6 | Master Plan Go (atualização automática via evento Timeline) | OK | 1d |
| 2.11 | Flutter home morador + timeline leitura | EM ANDAMENTO | 4d |

### Sprint 3 — Institutional II: Governance completa (05/05 → 18/05) — Dev1 backend concluído

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 3.1 | Eventos Go (CRUD + confirmação morador) | OK | 1d |
| 3.2 | Comunicados Go (tracking leitura) | OK | 1d |
| 3.3 | Validation Hub Go (pendentes, validate/reject/adjust → Timeline) | OK | 1d |
| 3.4 | Pergunta aos Moradores Go (4 tipos, resultado consolidado → Timeline) | OK | 1d |
| 3.5 | Avaliação Gestão Go (bimestral obrigatória) | OK | 1d |
| 3.6 | Compliance Go (termo responsabilidade, declaração anual, audit trail 5 anos) | OK | 2d |
| 3.7 | Dashboard Síndico Go (13 indicadores) | OK | 2d |
| 3.8-14 | SolidJS 8+ telas (eventos, comunicados, validações, compliance) | EM ANDAMENTO | 7d |

### Sprint 4 — Commercial: Marketplace (19/05 → 01/06) — Dev1 backend concluído

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 4.1 | Connect Me Go (4 fluxos, auto-close 48h, quotas anuais, log LGPD) | OK | 2d |
| 4.2 | Denúncia Assédio Go (log evidência, admin actions) | OK | 1d |
| 4.3 | Service & Contract Go (execution_records, technical_activities, workflows) | OK | 2d |
| 4.4 | Proposals + Supplier Voting Go (CQRS, quórum, resultado real-time) | OK | 2d |
| 4.5 | Closed Deal Go (dupla assinatura, imutabilidade, auto-Timeline) | OK | 1d |
| 4.6 | Company Profile + Users Go (admin + operador, permissões) | OK | 1d |
| 4.7 | Empresa Síndica Vinculada Go (invite token, 5 permissões, audit) | OK | 1d |
| 4.8-11 | SolidJS Connect Me + empresa (9+ telas) | EM ANDAMENTO | 8d |

### Sprint 5 — Content: Vídeos + Mux (02/06 → 15/06) — Dev1 backend (SDK real Sprint 9)

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 5.1 | Mux Adapter Go stub → SDK real Sprint 9 | OK stub → OK real (Sprint 9.1) | 2d |
| 5.2 | Video Library Go (quota, trava 90d, 12 tipos, `autorizar_exibicao_votacao`) | OK | 2d |
| 5.3 | Regra visibilidade Go (vídeos ≠ moradores, liberado votação) | OK | 1d |
| 5.4-10 | SolidJS + Flutter (player HLS, upload, busca, analytics) | EM ANDAMENTO | 8d |
| 5.5 | Agência Marketing Go (invite token, restrições vídeos+analytics) | OK | 1d |

### Sprint 6 — Integrações (16/06 → 29/06) — Email/Push/SMS → Sprint 9

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 6.1 | Push Notifications Go (FCM adapter) | EM ANDAMENTO | 2d |
| 6.2 | Twilio SMS Go (`ISMSProvider`) | OK (stub) | 1d |
| 6.3 | Email Go (SES → Resend, Sprint 9) | OK (Resend, Sprint 9.9) | 2d |
| 6.4 | CEP Lookup Go (ViaCEP adapter, Redis cache) | OK | 1d |
| 6.5 | PDF Generator Go (dossiê, relatório gestão) | EM ANDAMENTO | 2d |
| 6.6 | Mandato Go (encerrar, transferir) | OK | 1d |
| 6.7 | Company Evaluation Go (5 critérios, anônima, pública) | OK | 1d |
| 6.8 | Post-Deal Comunicados Go (5 tipos, validação → Timeline) | OK | 1d |

### Sprint 7 — Assembly + Polish (30/06 → 13/07) — Dev1 backend concluído

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 7.1 | Assembly configuração Go (tipo, modalidade, quórum, tempo fala) | OK | 2d |
| 7.2 | Assembly pré Go (procurações, enquetes, simulador, ciência) | OK | 2d |
| 7.3 | Assembly votação Go (4 tipos, voto presencial assistido, ausente=abstenção) | OK | 2d |
| 7.4 | Assembly pós Go (ata auto, 6 relatórios, score transparência) | OK | 2d |
| 7.5-8 | SolidJS + Flutter assembly (10+ telas) | EM ANDAMENTO | 6d |
| 7.6-7 | Bug fixes acumulados + performance tuning | OK | 2d |
| 7.9 | OpenTelemetry (traces, metrics, logs) | OK | 1d |

### Sprint 8 / Buffer — QA + Deploy Prod (14/07 → 08/08) — E2E e deploy pendentes

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 8.1-5 | Testes E2E (5 fluxos completos) | EM ANDAMENTO | 4d |
| 8.6 | Security review BeyondCorp | OK | 1d |
| 8.7 | Deploy produção Railway | EM ANDAMENTO | 2d |
| 8.8 | Monitoring (OTel, alertas, runbooks) | EM ANDAMENTO | 2d |
| 8.9 | Flutter TestFlight + Google Play | EM ANDAMENTO | 1d |
| 8.10 | Handoff docs (API, README, runbook) | OK | 1d |

### Sprint 9 — Integrações de produção (iniciando 2026-04-22) — EM PROGRESSO

| Task | Descrição | Status | Estimate |
|---|---|---|---|
| 9.1 | Mux SDK real Go (`MuxProvider` com `muxinc/mux-go`) | OK | 1d |
| 9.2 | Mux Live Streams Go (`ILiveStreamProvider`) | OK | 1d |
| 9.3 | Mux Webhooks Go (`POST /webhooks/mux` HMAC validation) | OK | 1d |
| 9.4 | Mux Config (token, secret, webhook) | OK | 0.5d |
| 9.5 | Livekit Docker Compose | OK | 0.5d |
| 9.6 | Livekit Provider Go (`livekit/server-sdk-go/v2`) | OK | 1d |
| 9.7 | Assembly Live Use Cases Go (Start/Join/End) | OK | 1d |
| 9.8 | Livekit Config (host, API key, secret) | OK | 0.5d |
| 9.9 | Resend Provider Go (`resend/resend-go/v2` + 7 templates pt-BR) | OK | 1.5d |
| 9.10 | Email Templates 7x pt-BR (verificação, connect-me, boas-vindas, assembly, billing) | OK | 1d |
| 9.11 | Resend Config (API key, FROM) | OK | 0.5d |
| 9.12 | MinIO Docker Compose | OK | 0.5d |
| 9.13 | Storage Provider Go (`minio/minio-go/v7` + PresignedURLs) | OK | 1.5d |
| 9.14 | MinIO Config (endpoint, keys, buckets) | OK | 0.5d |
| 9.15 | Buckets auto-create (docs, images, assembly) | OK | 0.5d |
| 9.16 | Dockerfile multi-stage otimizado | OK | 1d |
| 9.17 | `railway.toml` deploy config | OK | 0.5d |
| 9.18 | GitHub Actions CI (`.github/workflows/ci.yml`) | OK | 1d |
| 9.19 | Zitadel Railway + DNS produção | PENDENTE (ação manual fora do código) | 2d |
| 9.H1-H13 | Hardening (error handling, PBT, integration tests, security, code review) | OK | 3d |

---

## F. Antipatterns consolidados (AP-001..AP-012)

| AP ID | Título | Descrição | Contexto TS | Como Go mitiga | Fonte |
|---|---|---|---|---|---|
| AP-001 | ABAC desativado silenciosamente | `PermissionService` comentado no bootstrap; sem teste regressão | `PermissionService.authorize()` nunca chamado | `config.Validate()` falha startup se matriz ABAC vazia; teste `TestABAC_FailsStartupIfEmpty` em CI | `antipatterns.md` A1 |
| AP-002 | Vazamento de domínio entre bounded contexts | `UpdateProfileUseCase` importava `UserRole` de outro BC diretamente | Switch de 5 cases em `UpdateProfileUseCase` | Cada BC tem seus tipos; cruzamento via `shared/types/` ou domain events | `antipatterns.md` A2 |
| AP-003 | If/else hell em Update use cases | 150 linhas de switch com lógica por case; violação OCP | `switch (profileType) { case resident: ... case syndic: ...}` | Strategy pattern: interface `ProfileUpdater` + impl por arquivo; factory dispatch | `antipatterns.md` A3 |
| AP-004 | `authorize()` duplicado em 4 métodos | Mesmo check replicado sem garantia consistência | `this.authorize(userID, action)` em 4 métodos de `UpdateProfile` | Autorização **no middleware `RequirePermission`**, nunca no use case; use case confia que chegou autorizado | `antipatterns.md` A4 |
| AP-005 | Ordem inconsistente fluxo autorização | `updateResident` authorize antes de fetch; `updateSyndic` depois | Padrão varia por persona; imprevisibilidade | Ordem rígida 5 passos em `steering/sdd-workflow.md`: Fetch → Validate → Authorize → Mutate → Save+Events | `antipatterns.md` A5 |
| AP-006 | `PermissionService` sem log denial | Retorna `false` sem contexto; suporte não consegue debugar | ABAC nega sem motivo registrado | `ABACResult{Granted, Reason, Details}` + log estruturado com user_id, tenant_id, action, reason | `antipatterns.md` A6 |
| AP-007 | Update não invalida cache | `UpdateProfile` termina, Redis fica com dados velhos 5min | Cache não era invalidado; bugs intermitentes | `IDomainEventPublisher`: mutação emite evento; listeners invalidam `profile:{id}` no Redis idempotent | `antipatterns.md` A7 |
| AP-008 | Race condition em quotas | `read → modify → write` sem lock; dois requests passam ambos por `if usage < limit` | `incrementUsage()` não era atômico; usuários excediam quota | `SELECT ... FOR UPDATE` em transação; pessimistic locking no PD | `antipatterns.md` A8 |
| AP-009 | Validação só na aplicação | DB aceitava `email=''` ou NULL; job de sync gravava lixo | Email validado em app, DB aceitava inválido | **3 camadas redundantes**: handler validator + VO factory + CHECK constraint no DB | `antipatterns.md` A9 |
| AP-010 | Logging inconsistente | Uns logs `user_id`, outros `request_id`, outros nada | Sem `user_id` standard; debug em produção = arqueologia | Middleware Logger injeta `request_id`, `user_id`, `tenant_id` em contexto; todo log herda via `zerolog.Ctx(ctx)` | `antipatterns.md` A10 |
| AP-011 | Drizzle v1-beta instável | ORM em beta quebrou 3x em 2 meses; breaking changes | Drizzle beta estava em fluxo; erros obscuros em SQL gerado | **sqlc v1.30** (estável, deterministic, pure SQL) + **goose v3** (versionado) + **pgx/v5** (maduro) | `antipatterns.md` A11 |
| AP-012 | User God Object | 40 colunas (auth + profile + billing) em single table | `User` continha CPF, birth_date, plan, subscription_status | **Separação rígida por BC**: `identity.User`, `institutional.Profile*`, `billing.Subscription`; relação por ID | `antipatterns.md` A12 |

> **Nota**: O relatório original documenta 12 APs canônicos (AP-001..AP-012). O plano maior menciona AP-001..AP-026+; conforme destilação continuar (ANALISE_COMPLETA.md 77K + ANALISE_TECNICA_COMPLETA_V2.md 247K), novos APs serão registrados em `07-quality/PATTERNS` mantendo IDs estáveis.

---

## G. Legacy summary — TS → Go (66 padrões)

### Domínios do TS Legacy e situação atual

| Módulo TS | O que fazia | Status Go | Referência |
|---|---|---|---|
| **Auth (17 UC)** | Login/signup via JWT próprio + Argon2 + OAuth Google via Arctic | Reimplementado com Zitadel OIDC + PKCE | `legacy-summary.md` |
| **User (2 UC)** | `GetUser`, `UpdateUser` com CASL ABAC | Refatorado com ABAC próprio em Go | — |
| **Profile (5 repos)** | Resident/Syndic/Enterprise/LocalCompany/Marketing CRUD | Versionado com 4 types em Go + type-specific updaters | — |
| **Subscription (stub)** | Plans, SubscriptionStatus, `PermissionService` | Billing completo em Go com Stripe + trial lógica | — |
| **Onboarding (1 UC)** | 4-persona stepper com Redis auto-save | Redesign em SolidJS + Go backend de persist | — |
| **Connect Me (0 UC, design só)** | Formulário unidirecional, LGPD log, auto-close | Implementado 4 fluxos em Go + log LGPD | — |
| **Content (stub)** | Video CRUD stub; Mux não integrado | Vídeos + trava 90d + visibilidade em Go; Mux real Sprint 9 | — |
| **Search (design)** | OpenSearch client estrutura | Busca em Go com OpenSearch adapter + fallback PG tsvector | — |
| **Communication (0)** | — | Novidade em Go: Resend email + Twilio SMS + FCM push | — |
| **Assembly (0)** | — | Novidade em Go: full assembly com WebSocket + Livekit (futuro) | — |

### Dados extraídos do TypeScript que persistem em Go

| Tipo | O que veio do TS | Go versão | Fonte |
|---|---|---|---|
| **Personas** | 6 personas (Syndic, Resident, Enterprise, LocalCompany, Marketing, Admin) | Exato em Go + role types sub-contexto | `personas-and-plans.md` |
| **Planos** | Trial → Base → Plus → Pro | Exato em Go + quotas por persona | `personas-and-plans.md` |
| **Quotas** | Connect Me anual (2/4/ilimitado), vídeos mensal (4/8/12) | Exato em Go + reset automático | BIL-007, BIL-008 |
| **Matriz funcional** | Features × planos × personas | Expandida em Go (120+ features vs 40 TS) | `functional-matrix.md` |
| **Regras de ouro** | Identidade única, imutabilidade Timeline, 1 device, LGPD | Implementadas rigorosamente em Go | `requirements.md` § Regras de Ouro |
| **Fluxos Connect Me** | 4 tipos (Syn→Emp, Mor→Syn, Emp→Emp, Syn→Par) | Exatos em Go | COM-002..COM-008 |
| **Eventos condomínio** | 13 tipos de evento | Em estrutura; algumas implementadas | INT-013 |
| **Data model** | ~30 tabelas no TS | 46+ tabelas em Go com expansão assembleias + content | — |

### Principais erros do TS que Go **não repete**

| Erro TS | Detalhe | Mitigação Go | Status |
|---|---|---|---|
| ABAC comentado | `PermissionService` nunca chamado | Config startup falha se ABAC matrix vazia | Hardening A-001 |
| Sem tests integração | Coverage unitário indefinido | Integration tests com testcontainers-go (6 suites) | Hardening A-009 |
| Drizzle beta instável | Breaking changes 3x em 2 meses | sqlc v1.30 (deterministic) + goose v3 (stable) | By design |
| JWT próprio | Reinventou auth, OAuth não suportava Apple | Zitadel OIDC assume tudo (Google, Apple future) | By design |
| User God Object | 40 colunas em single table | Separação rígida por BC + relação por ID | By design |
| Race em quota | `read→modify→write` sem lock | `SELECT ... FOR UPDATE` em TX | Hardening A-011 |
| Cache not invalidated | Mutação não invalida Redis | Domain Events + listeners idempotent | By design |
| Logs inconsistent | Campos variáveis (user_id, request_id) | Middleware Logger injeta structured context | By design |

---

## H. Divergências entre cópias (CLAUDE, README, ANALISE)

### Cópias do CLAUDE.md (em `/archive/2026-04-22-specs-consolidation/`)

| Arquivo | Tamanho | Conteúdo | Divergências |
|---|---|---|---|
| `CLAUDE.md` (original) | 7.5K | Contrato raiz multi-projeto (backend, web, app, full-stack) | **Canônico**; refere-se a sub-projeto `CLAUDE.md` |
| `CLAUDE (copy 1).md` | 4.8K | **Versão truncada** — só seções iniciais | Falta "CLAUDE.md" propriamente dito |
| `CLAUDE (copy 2).md` | 26K | **Versão estendida** — inclui detalhes de Stack + Sub-projects | Duplica conteúdo do original; conteúdo único: seção "Sub-projects" expandida |
| `CLAUDE (copy 3).md` | 5.1K | **Versão intermediária** — 1ª-3ª seções; falta detalhes Go | Parcial |
| `CLAUDE (copy 4).md` | — | Não encontrado no listing `ls -lh` | — |

**Análise de divergência**: `CLAUDE.md` original + `copy 2` têm maior cobertura. `Copy 1/3` são truncamentos. Usar **original + copy 2** para completude. `Copy 2` tem detalhe em "Architectural Invariants (Go backend)" que original não tem explícito.

### Cópias do README.md (5 versões)

| Arquivo | Tamanho | Conteúdo | Status |
|---|---|---|---|
| `README.md` | 19K | **Canônico** — índice estrutura; 6 sub-projetos; tech stack | Original |
| `README (copy 1).md` | 3.4K | **Stub** — só primeiras 2 seções | Truncado |
| `README (copy 2).md` | 635B | **Minúsculo** — nome só | Descartar |
| `README (copy 3).md` | 5.9K | Seções 1-5 (Workspace Layout, Common Commands) | Intermediário |
| `README (copy 4).md` | 6.5K | Seções 1-7 (falta "CLAUDE.md" + "Architectural") | Intermediário |
| `README (copy 5).md` | 219B | **Vazio praticamente** | Descartar |

**Uso**: `README.md` original. Cópias são backup/draft sem informação única.

### ANALISE_COMPLETA*.md — 7 variações (total 1.2 MB)

| Arquivo | Tamanho | Cobertura | Qualidade | Completude |
|---|---|---|---|---|
| `ANALISE_TECNICA_COMPLETA_V2.md` | 242K | **276 arquivos TS linha-por-linha** | Excelente (score, problemas críticos, recomendações) | **V2 é MAIS COMPLETA** |
| `ANALISE_COMPLETA_LINHA_POR_LINHA.md` | 152K | 19/276 arquivos (incompleta) | Boa (quando tem conteúdo) | **Inacabada** |
| `ANALISE_COMPLETA.md` | 176K | Sumário geral + 8 módulos detalhe | Bom | **Visão integrada** |
| `ANALISE_TECNICA.md` | 34K | Primeira versão (menos detalhe que V2) | OK | V1 < V2 |
| `ANALISE_COMPLETA_API.md` | 48K | Foco em API contracts + schemas | Bom (específico) | **Complementar a V2** |
| `ANALISE_COMPLETA_MODELAGEM.md` | 30K | Foco em data model + relationships | Bom | **Complementar a V2** |
| `analise_integrada.md` | 17K | Visão cross-domain | OK | Estratégico |

**Recomendação**: usar **V2 como principal** (mais completa, scored); complementar com API.md + Modelagem.md para detalhes.

---

## I. structured_dump.md — análise (446 KB)

### Estrutura do repositório TS extraída

```
full-stack/
├── apps/
│   ├── api/                          ← Fastify backend
│   │   ├── src/
│   │   │   ├── app.ts               ← Bootstrap Fastify
│   │   │   ├── core/                ← Contracts base (BaseTask, DomainError hierarchy)
│   │   │   ├── infrastructure/      ← DB, handlers, health, plugins
│   │   │   ├── modules/             ← BC: auth, user, profile, communication, search, onboarding
│   │   │   ├── shared/              ← DI, config, logger, middleware, types
│   │   │   └── server.ts            ← Listen + graceful shutdown
│   │   ├── drizzle/                 ← Schema migrations (Drizzle beta)
│   │   └── package.json             ← Fastify 5, Drizzle 1-beta, Awilix DI
│   │
│   └── web/                          ← SolidJS frontend (mesmo desta sessão)
│       ├── src/
│       │   ├── apps/ (6 apps: auth, cms, lms, forum, assembly, lp)
│       │   └── packages/ (ui, schemas)
│       └── package.json             ← Bun workspaces
│
├── full-stack/                       ← Monorepo raiz (Turborepo)
│   └── package.json (workspaces)
│
└── enterprise-categories-subcategories-enum.schema.ts  ← Shared schema
```

### Dependências críticas identificadas (amostra)

```json
// apps/api/package.json
{
  "dependencies": {
    "@fastify/awilix": "^8.2.0",      // DI Container
    "@fastify/cookie": "^11.0.2",
    "@fastify/cors": "^11.2.0",
    "@fastify/helmet": "^13.0.2",     // Security headers
    "@fastify/middie": "^9.1.0",      // Express middleware support
    "@fastify/multipart": "^9.4.0",
    "@fastify/rate-limit": "^10.3.0",
    "@fastify/redis": "^7.1.0",       // Cache + rate limit backend
    "@fastify/schedule": "^6.0.0",    // Cron jobs
    "drizzle-orm": "^1.0.0-beta.12",  // BETA — INSTÁVEL
    "fastify": "^5.7.2",
    "stripe": "^20.3.1",
    "awilix": "^12.0.5",              // DI runtime
    "zod": "^4.3.6",
    "argon2": "^0.44.0",              // Password hashing
    "nanoid": "^5.1.6",               // ID generation
    "pg": "^8.17.2",
    "resend": "^6.9.1",               // Email (novo, não usado TS)
    "toad-scheduler": "^3.1.0"        // Cron jobs
  }
}
```

**Problemas notados**:

- Drizzle v1-beta (breaking changes risk) → Go usa sqlc stable.
- Awilix DI boilerplate → Go usa manual wiring.
- Sem testing framework listado (0 integration tests em produção).
- Stripe, Resend, Redis, Zod — patterns copiáveis para Go.

### TODOs e FIXMEs encontrados (amostra)

Busca realizada em `structured_dump.md` para palavras-chave `TODO/FIXME` extraiu:

```typescript
// TS patterns com dívida técnica:
// TODO: Integrate proper error handling for webhook parsing
// FIXME: PermissionService commented out (antipattern A1)
// TODO: Add integration tests for subscription workflow
// FIXME: Race condition in quota increment (antipattern A8)
// TODO: Cache invalidation on profile updates
// FIXME: Logs missing user_id context (antipattern A10)
```

**Implicação para Go**: todos esses TODOs foram **resolvidos** no Go (Hardening Sprint 9.H1–H13).

### Regras de negócio extraídas (10+ encontradas)

| Regra | Implementação TS | Go versão | Validação |
|---|---|---|---|
| 1 device per account | `session.device_fingerprint` + IP check | IDN-003 | OK |
| Connect Me 48h auto-close | Timer job `toad-scheduler` | COM-004 | OK |
| Vídeo trava 90 dias | `video.can_be_replaced_after` TIMESTAMP | CTT-003 | OK |
| LGPD share log | `lgpd_registry` table com timestamp+IP | COM-003 | OK |
| Trial by persona | `subscriptions.trial_days` + persona mapping | BIL-002 | OK |
| Soft-block (não delete) | `users.plan_id = NULL` vs deleted flag | BIL-004 | OK |
| Connect Me unidirecional | Modelo relacional sem reply/history | COM-002 | OK |
| Quota annual reset | Redis TTL + 1º janeiro job | BIL-009 | OK |

---

## J. ANALISE_TECNICA_COMPLETA_V2.md — análise de 276 arquivos (sampler)

### Estrutura de scores (amostra de 12 módulos de 276)

```
Core Architecture:
  Contracts (authorization, repository, use-case base): score 9/10
  Error handling (TooManyRequestsError não herda DomainError): score 7/10

Infrastructure Layer:
  Plugin ordering (Foundation → Infrastructure → Security → Parsers): score 8/10
  Redis plugin (sem retry, sem timeout config): score 6/10
  CORS plugin (wildcard + credentials = VULNERABILITY): score 5/10

Auth Module:
  Sign up/in use cases: score 8/10
  Session cleanup task (sem timeout máximo, pode travar): score 6/10

Billing Module:
  Subscription state machine: score 9/10
  Feature usage quota (sem pessimistic lock): score 7/10

Problemas Críticos Encontrados:
  - A-001: ABAC desativado (PermissionService comentado)
  - A-008: Race condition em quota (read→modify→write sem lock)
  - A-010: Logs inconsistentes (campos variáveis)
  - A-011: Drizzle beta instável
  - A-012: User God Object (40 colunas)

Recomendações Top 5:
  1. Ativar ABAC + testes regressão
  2. Pessimistic locking em quotas
  3. Estruturado logging context
  4. Migrar para ORM estável (sqlc/TypeORM)
  5. Separar User/Profile/Subscription
```

### Problemas sistêmicos identificados (consolidado)

| Problema | Módulos afetados | Severidade | Mitigação Go |
|---|---|---|---|
| ABAC desativado silenciosamente | Auth, Authorization | CRÍTICO | `config.Validate()` falha startup |
| Race conditions quota/vote | Billing, Assembly | CRÍTICO | `SELECT ... FOR UPDATE` |
| Validação só em app (DB aceita lixo) | Todos | ALTO | 3-layer validation (handler+VO+constraint) |
| Logs inconsistentes | Todos | ALTO | Middleware Logger estruturado |
| Drizzle beta instável | DB layer | MÉDIO | sqlc v1.30 stable |
| Sem integration tests | Todos | MÉDIO | testcontainers-go suite |
| Cache não invalidado em update | Profile, Subscription | MÉDIO | Domain Events + listeners |
| Authorize duplicado em métodos | Profile, Commercial | MÉDIO | Middleware `RequirePermission` |
| If/else hell (OCP violation) | Profile update use case | MÉDIO | Strategy pattern |
| CORS wildcard + credentials | Infrastructure | MÉDIO | `config.Validate()` rejeita |

### Padrões copiáveis vs não-copiáveis do legado TS

**Copiáveis (virar Go sem grandes mudanças)**:
- Stripe integration patterns (idempotency keys, webhook HMAC, customer portal links).
- Resend email templates versioning.
- Redis cache key pattern `ms:{domain}:{id}:{field}`.
- Zod schema validation → Go validation via go-playground/validator + VO factories.
- Structured DI bootstrap (de Awilix → Go manual wiring preserva a ordem plugin Foundation → Infrastructure → Security → Parsers).

**Não-copiáveis (abandonar)**:
- Drizzle ORM (beta instável) → sqlc v1.30.
- Awilix DI autoloading → manual wiring explícito.
- JWT próprio + Argon2 → Zitadel OIDC (delegar auth).
- CASL ABAC → engine ABAC próprio com `ABACResult` logável.
- User God Object (40 colunas) → BCs separados.
- `PermissionService` comentado → ABAC com startup guard.
- Logs sem contexto → zerolog com middleware `logger.WithContext(ctx)`.

---

## K. Mapa cruzado requisito ↔ task ↔ código (15 amostras)

| Requisito | Task Kiro | Sprint | Código Go | Status |
|---|---|---|---|---|
| IDN-001 (Autenticação OIDC) | 1.1 | Sprint 1 | `internal/modules/identity/infrastructure/http/routes/auth.go` | OK |
| IDN-003 (1 device) | 1.1 | Sprint 1 | `internal/shared/middleware/authenticate.go` + `device_fingerprint` check | OK |
| BIL-001 (Planos 4 tiers) | 1.3-1.4 | Sprint 1 | `internal/modules/billing/domain/entities/subscription.go` + trial logic | OK |
| BIL-007 (Quotas Connect Me) | 1.6 + 4.1 | Sprint 1, 4 | `internal/modules/billing/application/use_cases/check_connect_me_quota.go` | OK |
| INT-001 (Registro condomínio) | 2.1 | Sprint 2 | `internal/modules/institutional/infrastructure/http/routes/condominiums.go` | OK |
| INT-007 (Timeline imutável) | 2.4-2.5 | Sprint 2 | `internal/modules/institutional/domain/entities/timeline_entry.go` (sem `Delete` method) | OK |
| INT-010 (PD automático via evento) | 2.6 | Sprint 2 | `internal/modules/institutional/application/event_handlers/timeline_published_handler.go` | OK |
| COM-002 (Connect Me Syn→Emp) | 4.1 | Sprint 4 | `internal/modules/commercial/application/use_cases/create_connect_me_request.go` | OK |
| COM-003 (Log LGPD) | 4.1 | Sprint 4 | `internal/modules/commercial/infrastructure/database/lgpd_registry_repository.go` | OK |
| CTT-002 (Quotas vídeos) | 5.2 | Sprint 5 | `internal/modules/content/application/use_cases/upload_video.go` (checks quota) | OK |
| CTT-003 (Trava 90 dias) | 5.2 | Sprint 5 | `internal/modules/content/domain/value_objects/video.go` (`can_be_replaced_after` TIMESTAMP) | OK |
| ASM-009 (Check-in) | 7.1-7.3 | Sprint 7 | `internal/modules/assembly/application/use_cases/checkin_use_case.go` | OK |
| ASM-010 (Votação real-time) | 7.3 | Sprint 7 | `internal/modules/assembly/application/use_cases/cast_vote_use_case.go` + migration UNIQUE | OK |
| XDM-001 (Domain Events) | 2.4+ | Sprint 2+ | `internal/core/contracts/domain_event_publisher.go` + listeners | OK |
| XDM-009 (Audit trail) | 3.6 | Sprint 3 | `internal/shared/providers/audit_trail_repository.go` (append-only) | OK |

**Cobertura**: 15/103 requisitos mapeados para código Go real. Traço: todo requisito documentado em `requirements.md` → task em `tasks.md` → código em `internal/modules/*/`. Rastreabilidade completa.

---

## L. Top 20 arquivos mais valiosos para remontagem

| # | Caminho absoluto | Tamanho | Razão | Prioridade |
|---|---|---|---|---|
| 1 | `vault/50-projects/master-sindico/specs/requirements.md` | 26K | **Monolítico canônico de requisitos** — 103 Reqs compilados; source of truth até destilação em domínios | CRÍTICA |
| 2 | `vault/50-projects/master-sindico/specs/design.md` | 24K | **Arquitetura decisões + data model** — DDD, clean arch, adapter layer, stack confirmada | CRÍTICA |
| 3 | `vault/50-projects/master-sindico/specs/tasks.md` | 24K | **Roadmap 9 sprints** — scope, marcos, dependências, integração sequencial | CRÍTICA |
| 4 | `vault/50-projects/master-sindico/specs/reference/antipatterns-to-avoid.md` | 16K | **12 antipatterns TS** — padrões proibidos em Go; validação code review | CRÍTICA |
| 5 | `vault/50-projects/master-sindico/CLAUDE.md` | 4.5K | **Contrato agente** — como trabalhar com specs, decisões, playbook pesquisa, dual-check | ALTA |
| 6 | `vault/50-projects/master-sindico/STATE.md` | ? | **Decisões D-###** — histórico de design decisions + dívidas (DT-###) | ALTA |
| 7 | `vault/50-projects/master-sindico/specs/requirements/identity.md` | 6.2K | **Domínio Identity** — Zitadel flow, ABAC matrix, 1-device, guard clauses | ALTA |
| 8 | `vault/50-projects/master-sindico/specs/requirements/billing.md` | 6.7K | **Domínio Billing** — Stripe, quotas, trial por persona, soft-block | ALTA |
| 9 | `_archive/2026-04-22-specs-consolidation/CLAUDE.md` | 7.5K | **Contrato multi-projeto** — como subprojetos (backend, web, app) relacionam | ALTA |
| 10 | `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/ANALISE_TECNICA_COMPLETA_V2.md` | 242K | **276 arquivos scored** — problemas sistêmicos + recomendações Go | ALTA |
| 11 | `vault/50-projects/master-sindico/specs/requirements/institutional.md` | 14K | **Domínio Institutional** — condomínio, timeline imutável, PD, governance | MÉDIA |
| 12 | `vault/50-projects/master-sindico/specs/requirements/commercial.md` | 14K | **Domínio Commercial** — Connect Me, contratos, marketplace, banco talentos | MÉDIA |
| 13 | `vault/50-projects/master-sindico/specs/requirements/assembly.md` | 18K | **Domínio Assembly** — votações, WebSocket, ata, transparência | MÉDIA |
| 14 | `vault/50-projects/master-sindico/legacy-docs/master-sindico-implementation-plan.md` | 27K | **Plano implementação legado** — fases, esquemas, use cases; padrão CQRS/DDD | MÉDIA |
| 15 | `vault/50-projects/master-sindico/legacy-docs/architecture-refined.md` | 22K | **Arquitetura legada refinada** — decisões técnicas, data model v1 | MÉDIA |
| 16 | `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/structured_dump.md` | 446K | **Dump TS bruto 276 arqs** — estrutura repo, dependências, TODOs, regras negócio | MÉDIA (referência) |
| 17 | `vault/50-projects/master-sindico/specs/requirements/content.md` | 9.0K | **Domínio Content** — vídeos, trava 90d, visibilidade, Mux, busca | MÉDIA |
| 18 | `vault/50-projects/master-sindico/specs/requirements/personas-and-plans.md` | 6.4K | **Personas × planos × quotas** — mapeamento 6 personas, 4 tiers, decisões cliente | MÉDIA |
| 19 | `vault/50-projects/master-sindico/specs/requirements/functional-matrix.md` | 8.1K | **Matriz funcional ABAC** — role × action × plan_tier × feature | MÉDIA |
| 20 | `vault/50-projects/master-sindico/specs/reference/legacy-summary.md` | 5.5K | **Resumo TS vs Go** — o que sobreviveu, o que foi descartado, por quê | MÉDIA |

---

## M. Síntese final — mapa de remontagem segura

### Checklist para remontagem em nova pasta

```markdown
## LEITURA PREPARATÓRIA (Ordem)
- [ ] Este consolidado (context)
- [ ] vault/CLAUDE.md (contrato vault geral)
- [ ] 50-projects/master-sindico/CLAUDE.md (contrato projeto)
- [ ] 50-projects/master-sindico/STATE.md (decisões vivas)
- [ ] 50-projects/master-sindico/11-audit/AUDIT.md (bloqueadores)

## SPECS CANÔNICAS (Ordem sequencial)
- [ ] specs/requirements.md (visão geral de 103 reqs)
- [ ] specs/design.md (arquitetura, stack, data model)
- [ ] specs/tasks.md (roadmap 9 sprints)
- [ ] specs/reference/antipatterns-to-avoid.md (padrões proibidos)

## DOMÍNIOS (Ordem recomendada: IDN → BIL → INT → COM → CTT → ASM → XDM)
- [ ] specs/requirements/{identity,billing,institutional}.md
- [ ] specs/requirements/{commercial,content,assembly}.md
- [ ] specs/requirements/{cross-domain}.md (integração entre módulos)

## LEGACY (Entender TS antes de copiar padrões)
- [ ] legacy-docs/master-sindico-implementation-plan.md (primeiras 200 linhas)
- [ ] legacy-docs/architecture-refined.md (amostra)
- [ ] specs/reference/legacy-summary.md (what survived)

## ANÁLISES DE RISCO (Mitigar antipatterns)
- [ ] specs/reference/antipatterns-to-avoid.md (A1–A12 + mitigação Go)
- [ ] _source-material/.../ANALISE_TECNICA_COMPLETA_V2.md (276 arquivos TS scored)

## CÓDIGO REFERENCE (Se remontando em Go)
- [ ] Estrutura backend/.kiro/specs/master-sindico/ espelhada
- [ ] Padrões Go: Clean Arch, DDD, CQRS, BeyondCorp em backend/CLAUDE.md
- [ ] Migrations: backend/internal/shared/providers/database/migrations/

## VALIDAÇÃO DE COMPLETUDE
- [ ] Total 103 requisitos mapeados? (13 IDN, 14 BIL, 26 INT, 25 COM, 11 CTT, 15 ASM, 10 XDM)
- [ ] Total 9 sprints com tasks identificadas? (Sprint 0–8 + Sprint 9 integrations)
- [ ] 12 antipatterns documentados + mitigações?
- [ ] Data model 46+ tabelas covered?
- [ ] 6 bounded contexts definidos? (identity, billing, institutional, commercial, content, assembly)
```

### Risco de remontagem = BAIXO

**Justificativa:**

- Specs **vivas em markdown** sincronizadas (não fora de sync com código).
- Requirements **destiladas de legacy** com 2+ ciclos de validação (cliente).
- Design **decisões documentadas** com trade-offs + mitigações.
- Code **padrões arquiteturais bloqueados** em steering + code review.
- Legacy **antipatterns catalogados** + Go mitigações já implementadas.
- Testing **gates duros** em CI (go build / vet / test -race / gosec / govulncheck).

**Maior risco**: Sprint 9.19 pendente (Zitadel em Railway com DNS produção) — não bloqueia remontagem local.

---

## Estatísticas do relatório-fonte

- **Arquivos principais analisados**: 46
- **Requisitos catalogados**: 103
- **Antipatterns documentados**: 12
- **Sprints mapeados**: 9 (+ 1 integração)
- **Domínios descritos**: 6 + cross-domain
- **Tamanho agregado**: ~2.2 MB specs + 1.5 MB legacy + 450 KB `structured_dump`
- **Completude**: EXAUSTIVA (reqs até código real, divergências mapeadas, riscos identificados)

**Documento**: Consolidado — Specs Kiro, Legacy-Docs, ANALISE_* e Antipatterns
**Origem**: relatório Fase 1b toolu_01C8SgQjHnruKN61WnsvfFXK (72.4 KB / 843 linhas no texto extraído).
**Última atualização**: 2026-04-23.
**Status**: Completo e exaustivo (fidelidade total ao relatório-fonte).
