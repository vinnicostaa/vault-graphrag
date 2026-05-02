---
title: Business Rules — Master Síndico (destiladas, testáveis)
type: spec
tags: [domain, business-rules, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/01-domain/{business-rules,regras-canonicas}.md + specs/requirements/* + contextos/ANALISE_COMPLETA_MODELAGEM.md
created: 2026-04-23
updated: 2026-04-23
---

# Business Rules — Master Síndico v2

Regras de **negócio** (o que o negócio exige). Distintas de [[invariants]] (o que o domínio nunca viola em runtime) e das regras de implementação (`90-ingested/.../implementation-rules.md` → migrado pra patterns em `02-architecture/`).

**Formato**: `BR-###` | regra | BC | invariant relacionado (se houver) | teste previsto.

---

## Onboarding & Identidade

### BR-001 — Trial persona-aware

- **Regra**: Duração e features bloqueadas variam por persona.
  - Síndico: **15 dias**
  - Empresa: **7 dias**
  - Parceiro Vizinhança: **30 dias**
  - Morador: **—** (sem trial; Base gratuito ou Pagante pago)
- **BC**: billing (Subscription.TrialDuration)
- **Invariante**: INV-017 (TrialDuration persona-aware)
- **Teste**: Unit — `Plan.TrialDurationFor(persona)` retorna duração correta.

### BR-002 — Identidade única por CPF/CNPJ

- **Regra**: 1 CPF = 1 User (PF); 1 CNPJ = 1 User (PJ). Mesmo User pode ter múltiplos **vínculos** (Membership) em tenants diferentes.
- **BC**: identity
- **Invariante**: INV-002, INV-003
- **Teste**: Integration — unicidade DB via UNIQUE parcial.

### BR-003 — Onboarding por persona com auto-save em Redis

- **Regra**:
  - Morador: 4 telas (buscar condo → unidade → termo LGPD → foto)
  - Síndico: 6 telas (dados pessoais → tipo → markers → criar condo → mandato → empresa síndica)
  - Empresa: 7 telas (dados → CNPJ → categorias → compliance markers → logo/fotos → termos → dashboard)
  - Parceiro: 4 telas (dados estabelecimento → endereço/CEP → logo/descrição → dashboard)
- **Auto-save**: progresso em Redis `onboarding:{user_id}:{step}` (TTL 30d)
- **BC**: identity/institutional/commercial
- **Teste**: Integration (persistência Redis).

### BR-004 — 1 dispositivo ativo por conta

- **Regra**: Default 1 dispositivo; admin pode multi. Login em novo device invalida sessão anterior.
- **BC**: identity
- **Invariante**: INV-005
- **Teste**: Integration (concorrência) + E2E.

### BR-005 — Guard clause `NO_ROLE_ASSIGNED`

- **Regra**: Tokens Zitadel sem role retornam `403 NO_ROLE_ASSIGNED` **antes** do UPSERT em `users` (T15).
- **BC**: identity
- **Invariante**: INV-004, INV-010
- **Teste**: Integration.

---

## Governança & Timeline

### BR-006 — Timeline imutável

- **Regra**: `TimelineEntry` nunca é editado ou deletado. Correção = nova entry com `corrects_entry_id` + motivo.
- **BC**: institutional
- **Invariante**: INV-024, INV-032
- **Teste**: CI grep (bloqueia `UPDATE timeline_entries`) + Schema test.

### BR-007 — 15 condomínios por síndico (hard limit)

- **Regra**: 1 síndico gerencia no máximo **15 condomínios** (Decision 11). Acima: nova conta ou solicitação especial admin.
- **BC**: institutional
- **Invariante**: INV-027
- **Teste**: Integration.

### BR-008 — Σ Fração Ideal ≤ 100%

- **Regra**: Soma de fração ideal das unidades de um condomínio não pode exceder 100%.
- **BC**: institutional
- **Invariante**: INV-021, INV-022
- **Teste**: PBT (100 casos) + CHECK constraint.

### BR-009 — Announcement imutável pós-publish

- **Regra**: Comunicado oficial não pode ser editado após transição `draft → published`. Correção via adendo (Amendment).
- **BC**: institutional
- **Invariante**: INV-025
- **Teste**: Unit + Integration.

### BR-010 — Plano Diretor atualizado **apenas** via evento da Timeline

- **Regra**: Status de atividade em `MasterPlan` muda apenas via evento `ExecutionValidated` (Req 23) ou `TimelineEntryAdded`. Nunca via edição direta.
- **BC**: institutional
- **Invariante**: INV-031
- **Teste**: Integration.

### BR-011 — Mandato: datas válidas + histórico preservado

- **Regra**: `mandate.end_date > start_date`. Encerramento de mandato preserva histórico (Timeline; nunca apagado). Transferência: 7 dias de transição gradual.
- **BC**: institutional
- **Invariante**: INV-028
- **Teste**: Unit + Integration.

### BR-012 — Subsíndico e Conselho: permissões específicas

- **Regra**:
  - **Subsíndico**: maioria das ações do síndico, exceto encerrar mandato próprio e designar novo síndico.
  - **Conselho**: aprovar contratos acima de threshold (SupplierVote), vigiar atos.
- **BC**: institutional/cross-domain (ABAC)
- **Invariante**: INV-086 (ABAC ordem)
- **Teste**: Integration (matriz ABAC).

---

## Billing & Planos

### BR-013 — Soft-block sem grace period

- **Regra**: Expiração de subscription → features premium bloqueadas imediatamente. Dados permanecem intocados. UI mostra banner "Reativar" + link Stripe Customer Portal.
- **BC**: billing
- **Invariante**: INV-019
- **Teste**: Integration.

### BR-014 — Planos têm quotas; exceder bloqueia

- **Regra**: Ex: ConnectMe Básico 2/ano. Ao exceder: `403 QUOTA_EXCEEDED`. UI sugere upgrade. Reset: anual (ConnectMe) ou mensal (uploads vídeo).
- **BC**: billing
- **Invariante**: INV-016, INV-046
- **Teste**: PBT (300 casos) + Integration.

### BR-015 — Quota reset anual em 1º de janeiro (ConnectMe)

- **Regra**: Reset global 1º jan 00:00 (timezone BRT — **⚠️ dual-check P-INV-002**). Tracking `ms:quota:{userID}:connect_me:{year}`.
- **BC**: billing/commercial
- **Invariante**: INV-046
- **Teste**: Integration (simulate date).

### BR-016 — Quota reset mensal em 1º do mês (Vídeos)

- **Regra**: Vídeos institucionais têm quota mensal. Reset 1º do mês 00:00 BRT.
- **BC**: billing/content
- **Teste**: Integration.

### BR-017 — Stripe Customer Portal integrado (self-service)

- **Regra**: Upgrade/downgrade via Customer Portal (Stripe gerencia pro-rata). Backend reflete via webhook `subscription.updated`.
- **BC**: billing
- **Teste**: Integration.

### BR-018 — Trial nunca volta após conversão

- **Regra**: Após primeira conversão `trial → active` (pagamento ok), não há trial novamente pra mesmo User no mesmo plano.
- **BC**: billing
- **Invariante**: INV-013
- **Teste**: Unit + Integration.

---

## Connect Me

### BR-019 — Connect Me é unidirecional (não chat)

- **Regra**: Formulário estruturado, sem reply, sem thread, sem histórico de conversa. Input → email disparado.
- **BC**: commercial
- **Invariante**: INV-022, INV-037
- **Teste**: API contract (sem endpoint de reply).

### BR-020 — Connect Me direção controlada

- **Regra**: Quatro fluxos canônicos (Decision 5):
  1. **Síndico → Empresa** (quotas N1: 2/ano, N2: 4/ano, N3: ilimitado)
  2. **Morador → Síndico** (Base 2/ano, Pagante 4/ano)
  3. **Empresa → Empresa** (Plus/Pro apenas)
  4. **Síndico → Parceiro Vizinhança**
- Marketing Agência: **sem** Connect Me (anti-spam).
- **BC**: commercial
- **Invariante**: INV-045
- **Teste**: Integration (ABAC por fluxo).

### BR-021 — Auto-close Connect Me 48h sem interesse

- **Regra**: RFP sem `interest_expressed` em 48h → auto-close.
- **BC**: commercial
- **Invariante**: INV-047
- **Teste**: Integration (job + simulate time).

### BR-022 — Connect Me LGPD log ao revelar dados

- **Regra**: Quando empresa clica "Tenho Interesse" → revela dados síndico → **OBRIGATÓRIO** log LGPD com timestamp, IP, propósito, versão termos (retenção 5 anos).
- **BC**: commercial/cross-domain
- **Invariante**: INV-048
- **Teste**: Integration.

### BR-023 — Anti-assédio em Connect Me

- **Regra**: Botão "Denunciar" em cada Connect Me. Ações admin: warning/suspension/ban. Harassment confirmado → empresa suspensa.
- **BC**: commercial
- **Invariante**: INV-055
- **Teste**: Integration.

### BR-024 — Proposta imutável após `sent`

- **Regra**: Uma vez enviada, proposta não pode ser editada. Correção via Amendment (Req 25).
- **BC**: commercial
- **Invariante**: INV-039
- **Teste**: Unit + Integration.

### BR-025 — ClosedDeal imutável após dupla assinatura

- **Regra**: Empresa assina primeiro; síndico depois. Após ambas: imutável; flag `is_immutable=true`; auto-publica TimelineEntry.
- **BC**: commercial
- **Invariante**: INV-040, INV-041
- **Teste**: Integration.

### BR-026 — Validação obrigatória de execução pelo síndico

- **Regra**: Empresa registra execução → síndico **deve** validar (aceitar/rejeitar com motivo). Bloqueia pagamento até validação (Sprint 3+).
- **BC**: commercial
- **Invariante**: (captura de Req 23)
- **Teste**: Integration.

### BR-027 — Avaliação pós-contrato anônima

- **Regra**: Avaliação de empresa é **anônima ao avaliado**. Pública entre síndicos (média). Aciona após `contract.state=completed`.
- **BC**: commercial
- **Invariante**: INV-044
- **Teste**: Integration (API response filter).

---

## Reputação

### BR-028 — Reputação Bronze→Diamante determinística

- **Regra**: Função pura de `(closed_deals, avg_rating, condos_count, harassment_reports, videos_approved)`. Sem input humano direto. Recalcula em cada `CompanyEvaluationSubmitted`.
- **BC**: commercial
- **Invariante**: INV-036
- **Teste**: Unit (determinismo: mesmo input → mesmo output).

### BR-029 — Parâmetros Bronze→Diamante (destilado do legado)

| Tier | Requisito mínimo acumulado |
|---|---|
| Bronze | Default ao criar empresa (cadastro completo + CNPJ verificado) |
| Prata | ≥ 3 contratos + avg ≥ 3/5 |
| Ouro | ≥ 10 contratos + avg ≥ 3.5/5 + ≥ 3 condomínios distintos |
| Platina | ≥ 30 contratos + ≥ 4/5 + ≥ 10 condomínios + ≥ 1 vídeo institucional aprovado + ≥ 1 case em vídeo |
| Diamante | ≥ 100 contratos + ≥ 4.5/5 + ≥ 30 condomínios + ≥ 3 vídeos + múltiplos cases + tempo plataforma ≥ 12m |

- **⚠️ PENDÊNCIA**: parâmetros exatos a fechar na Fase 4 via dual-check (D-010 STATE).
- **BC**: commercial
- **Teste**: Unit (cada threshold).

### BR-030 — Degradação de tier permitida

- **Regra**: Tier pode cair se avaliação média despenca ou contratos cancelados — mesma função, mesmos inputs.
- **BC**: commercial
- **Invariante**: INV-036
- **Teste**: Unit.

### BR-031 — Suspensão temporária ≠ despromoção

- **Regra**: `HarassmentReported` confirmado → flag `suspended=true` **sobre** o tier atual (não rebaixa tier). Duração: 72h cooldown moderação.
- **BC**: commercial
- **Invariante**: INV-055
- **Teste**: Integration.

### BR-032 — Reputação global, visibilidade por tenant

- **Regra**: Reputação calculada sobre todos contratos (global). Mas moradores/síndicos só veem **avaliações em detalhe** do próprio tenant. Tier global visível a todos.
- **BC**: commercial/cross-domain (ABAC)
- **Teste**: Integration (cross-tenant).

---

## Conteúdo & Vídeos

### BR-033 — Vídeo publicado tem trava de 90 dias

- **Regra**: Após `VideoPublished`, `locked_until = published_at + 90d`. Motivo: churn prevention + evita spam publish/remove. Override: role `admin` em caso legal + audit log.
- **BC**: content
- **Invariante**: INV-056, INV-057
- **Teste**: Unit + Integration.

### BR-034 — Moderação manual antes de publish (MVP)

- **Regra**: Upload → `moderation` pending → admin aprova em 48h → `approved` → owner publica → `published + locked 90d`. ML moderation: M4+.
- **BC**: content
- **Invariante**: INV-058
- **Teste**: Integration.

### BR-035 — Trava trimestral por vídeo específico

- **Regra**: Substituir o **mesmo** vídeo (via `replaces_video_id`) requer ≥ 90 dias desde último upload. Não confundir com quota mensal de uploads **novos**.
- **BC**: content
- **Invariante**: INV-059
- **Teste**: Unit.

### BR-036 — Moradores veem vídeos empresa **apenas** durante votação fornecedor

- **Regra**: Default: morador não vê vídeo empresa. Unlock temporário: `votacao_fornecedor` ativa + empresa `autorizar_exibicao_votacao=true` → `VideoVisibilityGrant` criado. Revogado automaticamente ao fechar votação (NATS event `VotingClosed`).
- **BC**: content/assembly
- **Invariante**: INV-061, INV-062, INV-063
- **Teste**: Integration (cross-BC event).

### BR-037 — Privacidade de métricas entre tenants

- **Regra**: Views/likes/comments de vídeos de tenant X **nunca** vazam pra tenant Y. Empresas veem agregados globais (sem identificação de condomínio específico).
- **BC**: content/cross-domain
- **Invariante**: INV-060
- **Teste**: Integration (100+ cross-tenant tests).

### BR-038 — Agência de Marketing: acesso restrito

- **Regra**: Actor delegado (não tenant). Escopo token: `videos:upload|edit`, `analytics:read`. **Zero** acesso a deals/propostas/Connect Me/gestão usuários.
- **BC**: content/identity
- **Invariante**: INV-049, INV-066
- **Teste**: Integration (ABAC).

### BR-039 — Parceiro Vizinhança: sem LMS

- **Regra**: Parceiro **não** tem acesso a Academia (cursos, trilhas, certificados). Botão oculto na UI.
- **BC**: content
- **Invariante**: INV-067
- **Teste**: Integration + E2E.

### BR-040 — Certificate emitido automaticamente ao 100%

- **Regra**: `Enrollment.progress = 100%` → handler `GenerateCertificate` → PDF + serial verificável. Assinatura digital BR plena em Sprint 3+.
- **BC**: content
- **Invariante**: INV-037, INV-068
- **Teste**: Integration.

---

## Assembleia & Votação

### BR-041 — Assembleia não pode começar sem edital publicado

- **Regra**: `PublishEdital` é pré-requisito de `StartAssembly` (A-013 legado fix).
- **BC**: assembly
- **Invariante**: INV-074
- **Teste**: Unit + Integration.

### BR-042 — Quórum calculado por fração ideal

- **Regra**: Peso de voto = fração ideal da unidade do votante. Quórum = Σ frações presentes (ou representadas via proxy). Não é "1 pessoa = 1 voto".
- **BC**: assembly
- **Invariante**: INV-073
- **Teste**: PBT + Unit.

### BR-043 — Tipos de quórum

- **Regra**:
  - `simple_majority` — > 50% das frações presentes
  - `qualified_majority` — ≥ 2/3 (ex: alteração de convenção)
  - `unanimous` — 100%
  - Destituição síndico: 1/2 + 1 das frações
- **BC**: assembly
- **Teste**: Unit (cada tipo).

### BR-044 — Ciência obrigatória de pauta

- **Regra**: Morador precisa clicar "Li e entendi a pauta" pra destravar app após `AgendaPublished`. Sem ciência: app travado (middleware valida). Auto-unlock após assembleia fechada.
- **BC**: assembly
- **Invariante**: (Req 50.2)
- **Teste**: Integration + E2E.

### BR-045 — Vote único + TOCTOU protegido

- **Regra**: UNIQUE(agenda_item_id, voter_id). Se race: ErrConflict → 409 (A-025 legado fix). Sem substituição de voto.
- **BC**: assembly
- **Invariante**: INV-071
- **Teste**: PBT TOCTOU + Integration.

### BR-046 — Ausência momentânea: 15min janela

- **Regra**: Morador check-in mas saiu — tem 15min pra voltar e votar. Após: `Absent` automático. Morador pode reabrir app e votar dentro da janela.
- **BC**: assembly
- **Invariante**: INV-076
- **Teste**: Integration.

### BR-047 — Procuração: proxy replica voto titular

- **Regra**: Proxy não pode votar diferente do titular. Se titular votou, proxy é impedido. Proxy revogável até 1h pré-assembleia. Proxy válido até 24h pós-encerramento (voto atrasado).
- **BC**: assembly
- **Invariante**: INV-077, INV-078, INV-079
- **Teste**: Unit + Integration.

### BR-048 — Presidente síndico não vota

- **Regra**: Síndico presidente da assembleia é impedido de votar (imparcialidade).
- **BC**: assembly
- **Invariante**: INV-081
- **Teste**: Unit + Integration.

### BR-049 — Ata imutável + homologação obrigatória

- **Regra**: Ata gerada auto → publicada imutável → votação de homologação (maioria simples) → `homologated_at` → imutável absoluto. Correção: nova entry na Timeline (nunca edita ata).
- **BC**: assembly
- **Invariante**: INV-072, INV-085
- **Teste**: Unit + Integration.

### BR-050 — WebSocket latência < 200ms (NFR crítico)

- **Regra**: Check-in + votação real-time exigem latência P95 < 200ms via WebSocket.
- **BC**: assembly/cross-domain (infra)
- **Teste**: Performance (load test).

### BR-051 — Score de transparência 10 componentes

- **Regra**: Cálculo automático pós-homologação (Req 60.7):
  1. Pauta publicada ≥ 5 dias antes (10pt)
  2. Edital distribuído (10pt)
  3. Ciência pauta registrada (10pt)
  4. Quórum atingido (10pt)
  5. Votação online disponível (10pt)
  6. Ata publicada Timeline (10pt)
  7. Relatórios públicos acessíveis (10pt)
  8. Score fala equilibrada (10pt)
  9. Homologação registrada (10pt)
  10. Sem atrasos encerramento (10pt)
- **BC**: assembly/cross-domain
- **Teste**: Unit.

### BR-052 — Modo contingência (offline fallback)

- **Regra**: WebSocket cai → app salva votos em Redis local → auto-sync quando volta → last-write-wins em reenvio.
- **BC**: assembly
- **Teste**: Integration.

---

## LGPD & Compliance

### BR-053 — LGPD direito de exclusão SLA 15 dias

- **Regra**: User solicita exclusão. Hard delete: User, Sessions, DeviceFingerprints, dados pessoais em Company. Soft archive + pseudonimização: votes, avaliações, minutes (imutáveis). SLA 15 dias.
- **BC**: cross-domain
- **Invariante**: INV-091
- **Teste**: Integration (E2E).

### BR-054 — LGPD log de revelação de dados

- **Regra**: Toda revelação de dados pessoais (Connect Me, `/me/export`, etc) gera `LGPDLog` append-only. Retenção 5 anos. Direito ao esquecimento após vencimento → hard delete.
- **BC**: cross-domain
- **Invariante**: INV-091
- **Teste**: Integration.

### BR-055 — Termos versionados + aceite rastreado

- **Regra**: Termos têm versão. Se mudam, usuários devem aceitar nova versão antes de usar app. Aceite rastreado com timestamp + IP + versão.
- **BC**: cross-domain
- **Teste**: Integration.

### BR-056 — Declaração anual obrigatória

- **Regra**: Síndico declara conformidade 1x/ano. Atrasada: bloqueia exportação de dossiê (Req 46). Histórico 5 anos.
- **BC**: institutional/cross-domain
- **Invariante**: INV-034
- **Teste**: Integration.

### BR-057 — Audit trail append-only 5 anos

- **Regra**: Ações críticas (login, votação, publicação comunicado, validação execução, mudança permissão) vão pra `audit_trail`. Append-only. Retenção 5 anos. Admin + síndico principal acessam.
- **BC**: cross-domain
- **Invariante**: INV-090
- **Teste**: Schema + Integration.

### BR-058 — PII nunca em logs

- **Regra**: CPF, CNPJ, email, telefone, token, password **nunca** em logs. Usar `Masked()`. CI bloqueia via grep regex.
- **BC**: cross-domain
- **Invariante**: INV-092
- **Teste**: CI static check.

---

## Segurança & ABAC

### BR-059 — ABAC default-deny

- **Regra**: Ação não configurada no catálogo → nega. Admin bypass é explícito + audit log do bypass.
- **BC**: cross-domain
- **Invariante**: INV-087, INV-088
- **Teste**: PBT (400 casos).

### BR-060 — Tenant isolation estrito

- **Regra**: Toda query escopada por condomínio tem `WHERE condominium_id = $tenant_id` (ou IN). Cross-tenant apenas via grants explícitos.
- **BC**: cross-domain
- **Invariante**: INV-089
- **Teste**: Integration (100+ cross-tenant).

### BR-061 — Webhook HMAC verify antes de parse

- **Regra**: Stripe, Mux, Livekit — HMAC signature **antes** de parse body. Falha → 400 sem parse.
- **BC**: cross-domain
- **Invariante**: INV-093, INV-064, INV-018
- **Teste**: Pattern enforce + Integration.

### BR-062 — Rate limit estruturado

- **Regra**: `/auth/*` 20rpm burst 5; `/api/v1/*` 60rpm burst 10. Token bucket com cleanup (A-032 fix).
- **BC**: cross-domain
- **Invariante**: INV-097
- **Teste**: Integration.

### BR-063 — Antifraude camadas (atualizada D-135)

- **Regra** (Req 102, ratificado por [[../STATE#D-135|D-135]] 2026-04-25):
  1. 1 device por conta (Rule 8) — `device_fingerprint = SHA-256(ip || user_agent || canvas_hash)`
  2. CAPTCHA após 3 falhas login
  3. **Device fingerprint progressivo** em signup (não bloqueio absoluto):
     - 1-2 contas mesmo device em 24h: silent log
     - 3 contas: CAPTCHA Turnstile (D-117)
     - 5+ contas: revisão manual + alerta admin
  4. Detecção anomalia IP novo → SMS confirmação (Sprint 3+)
- **BC**: cross-domain/identity
- **Teste**: Integration.

> **Histórico — bloqueio por IP rejeitado**: Req 102 §1 do `_chaos/requirements(6).md` propunha "Bloqueio múltiplas contas mesmo IP < 1h". REJEITADO por [[../STATE#D-135|D-135]] (famílias compartilham wifi do condomínio → falsos positivos seriam regra). Adotado device fingerprint progressivo. Spec implementação: [[../04-requirements/functional/identity#IDN-026]].

---

## Vizinhança

### BR-064 — Cupom lock 4h por usuário-loja

- **Regra**: Usuário gera cupom em loja → 4h cooldown pra mesma loja (anti-spam).
- **BC**: commercial
- **Invariante**: INV-051
- **Teste**: Integration (concurrent).

### BR-065 — Formato CouponCode

- **Regra**: `ID_LOJA(5) + SEQ(3) + PALAVRA(5)` — ex: `00123055PAO`. Palavra do dia: máximo 5 letras maiúsculas.
- **BC**: commercial
- **Invariante**: INV-052, INV-053
- **Teste**: Unit.

### BR-066 — Promoção exclusiva vs aberta

- **Regra**:
  - **Aberta (bairro)**: válida para moradores de condomínios no bairro/CEP (raio configurável; default 2km).
  - **Exclusiva (condomínio)**: síndico negocia; válida **só** para aquele condomínio.
- **BC**: commercial
- **Teste**: Integration.

### BR-067 — Seal "Síndico-aprovado"

- **Regra**: Síndicos com `plan_tier ∈ {plus, pro}` (D-067/D-081/D-096) podem conceder seal a parceiro. Não é determinístico (decisão do síndico). Rastreável.
- **BC**: commercial/institutional
- **Teste**: Integration.

### BR-068 — Parceiro NÃO presta serviço ao condomínio

- **Regra**: Parceiro Vizinhança vende ao **morador como consumidor final** (pizzaria, petshop, academia). Distingue de **Empresa** que presta serviço ao condomínio (portaria, limpeza, reforma).
- **BC**: commercial
- **Teste**: regra de cadastro (diferentes fluxos).

---

## Marketplace & Banco de Talentos

### BR-069 — Vídeo-currículo 90s + lock 90d

- **Regra**: Morador (Pagante) publica vídeo-currículo 90s via Mux. Trava 90d pós-upload (igual vídeo institucional).
- **BC**: content/commercial
- **Invariante**: INV-059
- **Teste**: Unit.

### BR-070 — Empresas Plus/Pro buscam moradores

- **Regra**: Apenas empresas Plus/Pro podem visualizar currículos do Banco de Talentos. Convite via Connect Me fluxo especial (Empresa → Morador/Síndico).
- **BC**: commercial
- **Teste**: Integration (ABAC).

---

## Multi-produto cross-cutting

### BR-071 — 7 sub-produtos mapeados a BCs

- **Regra**: Todo sub-produto tem BC principal + cross. Nenhum sub-produto "flutuante" sem aggregate proprietário.
  1. Governança Institucional → `institutional`
  2. Connect Me → `commercial`
  3. Reputação → `commercial`
  4. Conteúdo & Vídeos → `content`
  5. Assembleias Live → `assembly`
  6. LMS & Banco Talentos → `content` + `commercial`
  7. Vizinhança → `commercial`
- **BC**: todos
- **Teste**: code review.

### BR-072 — Frontend nunca calcula regra crítica

- **Regra**: Frontend é UI. Backend é verdade única. Quotas, money, prazos, reputação, quórum: **sempre** backend.
- **BC**: cross-domain
- **Teste**: API contract test.

### BR-073 — SDK externo isolado em provider

- **Regra**: Stripe/Mux/Livekit/Zitadel SDK vive apenas em `infrastructure/providers/<provider>/`. Domínio não conhece provider.
- **BC**: todos
- **Teste**: code review + static import check.

### BR-074 — Cache Redis com invalidação por webhook

- **Regra**: Cache ABAC TTL 5min. Invalidação por webhook Stripe `customer.subscription.*`. Versioning pra evitar cache poisoning (AP-005 legado).
- **BC**: cross-domain
- **Teste**: Integration.

---

## Total: **74 regras** (30-50 esperadas — expandidas pra minúcia)

Distribuição:

| BC | Count |
|---|---|
| identity | 5 |
| billing | 6 |
| institutional | 7 |
| commercial (Connect Me + Reputação + Vizinhança + Talentos) | 25 |
| content | 8 |
| assembly | 12 |
| cross-domain (LGPD + ABAC + infra) | 11 |
| **total** | **74** |

---

## ⚠️ Pendências de dual-check (Fase 5)

- **P-BR-001**: BR-029 parâmetros exatos Bronze→Diamante (D-010 STATE).
- **P-BR-002**: BR-030 degradação reversível — **em quanto tempo**? Legado não explicita recálculo retroativo.
- **P-BR-003**: BR-032 visibilidade agregada global vs detalhada por tenant — confirmar cálculo em SQL (NOT IN) ou via ABAC policy.
- **P-BR-004**: BR-036 quem autoriza `autorizar_exibicao_votacao=true`: empresa unilateralmente ou precisa aprovação síndico? Legado: empresa.
- **P-BR-005**: BR-046 ausência momentânea 15min — confirmar timer começa no `CheckIn` ou no `CheckOut` implícito (saída do app).
- **P-BR-006**: BR-051 componente 8 "fala equilibrada" — algoritmo? Legado não especifica. Sugestão: Gini coefficient sobre duração de falas.
- **P-BR-007**: BR-056 declaração anual atrasada **bloqueia** dossiê — confirmar se é bloqueio total ou só warning.
- **P-BR-008**: BR-067 seal "Síndico-aprovado" — confirmar se precisa votação (colegiada) ou um síndico sozinho decide (legado: unilateral).
- **P-BR-009**: BR-020 Marketing Agência sem Connect Me — isso impede a agência de convidar empresas? Confirmar fluxo alternativo.

---

## Links

- [[_moc]]
- [[bounded-contexts]]
- [[invariants]] (enforcement técnico das BRs)
- [[context-map]]
- [[domain-events]]
- [[state-machines]]
- [[aggregates/_moc]]
- [[../04-requirements/_moc]] (a destilar — Req 1-103)
- [[../00-product/portfolio-de-produtos]]
