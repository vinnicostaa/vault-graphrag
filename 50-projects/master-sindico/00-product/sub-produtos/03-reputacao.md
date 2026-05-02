---
title: "03-reputacao — Bronze→Diamante determinística (Sub-produto Reputação)"
type: note
tags: [sub-produto, master-sindico, v2, fase-8, reputacao, bronze-diamante, determinista]
source: destilacao-fase-8
created: 2026-04-23
updated: 2026-04-23
status: stable
---

# 03 — Reputação Bronze → Diamante (determinística)

> **Sub-produto interno** do Master Síndico. Não é módulo de UI próprio: é um **serviço de domínio** (Domain Service `ReputationCalculator`) que vive dentro do bounded context `commercial`, alimenta o aggregate `Company` (campo `reputation_tier`) e aparece nas superfícies de consumo (perfil público, feed Connect Me, busca, badge institucional). Regra inegociável desde Fase 3: **determinístico, função pura, zero input humano direto, zero ML em M1/M2** (ADR-0023).

---

## 1. Fontes consultadas (inventário por arquivo + data)

| Fonte | Papel | Trechos relevantes |
|---|---|---|
| `Development/backend/.kiro/specs/master-sindico/requirements/commercial.md` (2026-04-22) | Spec viva Abril/2026 | Req 18 (cadastro empresa, status `pending_verification → active → suspended → blocked`); Req 26 (Avaliação de Empresa — 5 critérios escala 1-10, anônima pra empresa); Req 19 anti-assédio (denúncia → ações admin `warning/suspension/ban`) |
| `Development/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` (2026-04-22) | Spec viva — compliance, trilha append-only, LGPD 5 anos | Req 37 audit append-only; Req 102 Antifraude (1-device, CAPTCHA, IP block, anomalia); Req 103 visibilidade estrita de vídeo empresa |
| `Development/backend/internal/modules/commercial/domain/entities/company.go` (2026-04-21) | Código Go — aggregate `Company` | Campos implementados hoje: `id, legalName, tradeName, cnpj, status, description, categories, email, phone, website, cep, city, state, createdBy`. **Não contém** `reputation_tier` nem `suspended_until` ainda. Métodos `Activate() / Suspend() / Block()` |
| `Development/backend/internal/modules/commercial/domain/entities/company_evaluation.go` (2026-04-21) | Código Go — aggregate `CompanyEvaluation` | 5 critérios `int 1-10`: `criterionQuality, criterionPricing, criterionPunctuality, criterionProfessionalism, criterionCommunication`. `averageScore` calculada no construtor (`sum/5`). `isAnonymousToCompany = true` default. **Sem referência a Reputation tier** — é input para o cálculo, não o cálculo em si |
| `Development/backend/internal/modules/commercial/domain/entities/closed_deal.go` (2026-04-21) | Código Go — aggregate `ClosedDeal` | Dupla assinatura (`companySign → syndicSign`); `isImmutable=true` após `SyndicSign`; auto-publica Timeline. Input para fórmula: contador de `closed_deals_signed` por empresa |
| `Development/backend/internal/modules/commercial/domain/enums/enums.go` (2026-04-21) | Código Go — enums | `CompanyStatus` (pending_verification/active/suspended/blocked); `HarassmentReportStatus` + `HarassmentAdminAction` (warning/suspension/ban) — nenhum enum `ReputationTier` **ainda não escrito** |
| `Development/backend/internal/modules/commercial/application/use_cases/company_evaluation_use_cases.go` (2026-04-21) | Use case — `SubmitEvaluationUseCase` | Valida duplicata (`HasEvaluated`), cria aggregate, persiste. **Não chama `RecalculateReputation`** ainda — integração prevista |
| `Development/backend/internal/shared/providers/database/migrations/00301_create_companies.sql` | Schema — `companies` | Colunas atuais: `id, legal_name, trade_name, cnpj, status, description, categories, email, phone, website, cep, city, state, created_by, created_at, updated_at, deleted_at`. **Faltam**: `reputation_tier`, `suspended_until`, `score_components` JSONB |
| `Development/backend/internal/shared/providers/database/migrations/00308_create_company_evaluations.sql` | Schema — `company_evaluations` | UNIQUE `(company_id, evaluator_user_id, condominium_id)`; `average_score NUMERIC(4,2)`; `is_anonymous_to_company BOOLEAN DEFAULT TRUE`. Index em `company_id` |
| `Development/backend/docs/openapi.yaml` (2026-04-22) linhas 116 e 234 | OpenAPI | Enum **antigo** em `Plan.tier`: `[bronze, prata, ouro, diamante]` — **4 tiers**, sem Platina (diverge do v2 canônico). Campo `reputation_score: number` em `CondominiumDashboard` |
| `Development/backend/.kiro/specs/master-sindico/requirements/product-overview.md` linha 21 | Visão de produto | "Reputação → sistema de status baseado em evidência (Bronze → Diamante)" |
| `Development/backend/.kiro/steering/project-overview.md` linha 9 | Steering | "governança aplicada: critério na contratação, memória institucional auditável, reputação baseada em evidência (Bronze → Diamante)" |
| `Development/Content/regras-de-negocio/regras-canonicas.md` linha 22 (2026-04-22) | Canonização consolidada | Reputação listada como 3º dos 4 pilares de governança |
| `Development/Content/regras-de-negocio/quebras-detectadas.md` Q15 (2026-04-22) | Auditoria de drift | **"Recortes (`requirements/*.md`) não trazem Bronze→Diamante"** — spec canônica de reputação não existia no backend até esta Fase 8. Fix previsto: criar `requirements/reputation.md` quando tema entrar em sprint (post-launch — Sprint 10+) |
| `Development/Content/_handoffs/2026-04-22-relatorio-agente3-analista.md` linha 157 | Handoff analista | "Quando reputação Bronze→Diamante entrar em sprint, criar `requirements/reputation.md`" — reputação **não está em sprint ativa (0-9)**, é post-launch |
| `Development/Content/excopo-tecnico-antigo.txt` linhas 108, 125 + `full-stack/contextos/extracts/escopo.txt` linhas 130, 147 | Escopo técnico original (Mar/2026) | Referências primárias do Manual: **"Status Bronze/Prata/Ouro" do Síndico**, exibido no perfil público — **contexto histórico diferente do v2** (ver §3 abaixo) |
| `Development/full-stack/contextos/extracts/manual.txt` Capítulo 8, linhas 2685-2910 | **Manual original do cliente** (Mar/2026) | Capítulo 8 completo: "SISTEMA DE RECONHECIMENTO E STATUS DO SÍNDICO" — **4 tiers (Bronze/Prata/Ouro/Diamante)** do **síndico** (não da empresa). Critérios: vídeos assistidos ≥70%, condomínios vinculados, módulos capacitação concluídos. §8.5 "NÃO é ranking / NÃO gera competição / NÃO é gamificação" |
| `Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Comercial.md` (2026-03-10) | Vault Obsidian | Req 26 Avaliação de Empresa — 5 critérios: `qualidade_servico, cumprimento_prazo, atendimento, custo_beneficio, recomendacao` (nomes ligeiramente divergentes do código Go — ver §11) |
| `master-sindico-v2/STATE.md` D-051 (2026-04-23) | Decisão canônica v2 | **D-051 Fórmula Reputação Bronze→Diamante**: pesos 40/25/15/10/10 + 5 tiers [0-40/40-60/60-75/75-88/88-100] + cold start + degradação reversível |
| `master-sindico-v2/11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais.md` §3.3 (2026-04-23) | Dual-check raíz da fórmula | Derivação dos 5 componentes, rationale de cada peso, cold start, LGPD Art. 20 explicabilidade |
| `master-sindico-v2/AUDIT.md` linha 169, 220 | AUDIT fechamento | D-051 ✅ fechado; P-BR-001 / D-010 🟢 resolvido → ADR-0020 + adendo ADR-0023 planejados |
| `master-sindico-v2/01-domain/business-rules.md` BR-028..BR-032 | Regras de negócio | BR-028 determinística · BR-029 thresholds 3/10/30/100 · BR-030 degradação · BR-031 suspensão ≠ despromoção · BR-032 global/tenant visibility |
| `master-sindico-v2/01-domain/invariants.md` INV-036, INV-054, INV-055 | Invariantes | Função determinística · evento dispara recálculo idempotente · harassment confirmado flag (não rebaixa tier) |
| `master-sindico-v2/01-domain/state-machines.md` §6 Company Reputation Tier | Máquina de estado | Diagrama de 5 tiers + transições reversíveis + flag `suspended` paralela |
| `master-sindico-v2/01-domain/domain-events.md` E-040, E-041 | Eventos de domínio | `CompanyEvaluationSubmitted` (payload completo) · `ReputationTierChanged` (payload: previous_tier, new_tier, inputs) |
| `master-sindico-v2/01-domain/aggregates/Company.md` + `CompanyEvaluation.md` | DDD destilado v2 | Campos, factories, métodos, repos, eventos — versão destilada que estende o código atual (reputation_tier, suspendedFlag, suspendedUntil, ApplyHarassmentSuspension) |
| `master-sindico-v2/04-requirements/functional/commercial.md` COM-025..COM-027 | Requisitos funcionais (EARS) | COM-025 tabela de requisitos por tier · COM-026 suspensão temporária · COM-027 breakdown `/companies/{id}/reputation-breakdown` |
| `master-sindico-v2/04-requirements/global.md` GR-028 | Requisitos globais | GR-028 "Reputação determinística" · PATCH direto em `companies.reputation_tier` → 403 |
| `master-sindico-v2/04-requirements/registration-data.md` §3.4 | Dados de cadastro | `reputation_tier: bronze \| prata \| ouro \| platina \| diamante` (derivado, GR-028); `suspended_until: timestamptz NULL` |
| `master-sindico-v2/02-architecture/adr/0023-recsys-deterministic-m1.md` | ADR-0023 | Ranking Connect Me usa `reputation_tier_bonus` (w6 em score composto): Bronze=0, Prata=0.05, …, Diamante=0.20 |
| `master-sindico-v2/08-security/threat-model.md` §15 | Threat model | **Gap-SEC-008** decay temporal ausente → M3 decay mensal · **Gap-SEC-010** reputation ring/farm detection → M4 ML-based |
| `master-sindico-v2/05-tasks/backlog.md` T-151..T-155 | Backlog Sprint 11 | T-151 Recálculo determinístico pós-evento · T-152 Degradação PBT · T-155 Fechar D-010 → ADR-00XX (thresholds congelados em MVP) |
| `master-sindico-v2/03-subprojects/backend/tasks.md` BE-101, BE-106, BE-110, BE-160 | Tasks de backend | Aggregate Company + CNPJ + `ReputacaoTier` VO · `ReputationCalculator` domain service · `SubmitEvaluation → event → recalc saga` · Subscriber `ReputationRecalc` |
| `master-sindico-v2/analise-crua.md` linhas 340-380 | Síntese Fase 7 | Consolidação do sub-produto com todas as regras vivas |

**Marcador canônico crítico**: a fórmula determinística de Reputação **é sub-produto v2 (M2-M3)** — **não está implementada no código backend** até Sprint 9 (2026-04-23). O que existe hoje no código é apenas a coleta de inputs: `ClosedDeal` (imutável após dupla assinatura), `CompanyEvaluation` (média dos 5 critérios), `HarassmentReport` (denúncia → ação admin). O `ReputationCalculator` + campo `reputation_tier` + evento `ReputationTierChanged` são **tasks agendadas** (backlog T-151..T-155, BE-101, BE-106, BE-110, BE-160).

---

## 2. Propósito — a dor #1 resolvida

Critério **na contratação**. Síndicos contratam empresas para condomínio (portaria, limpeza, reforma, manutenção preventiva, reformas estruturais) hoje por **indicação informal** ("o fulano indicou"), **urgência** ("a bomba estourou, preciso agora") ou **preço mais barato** (sem qualificar a empresa). Resultado: contratações reativas, contratos mal cumpridos, obras mal feitas, re-trabalho, litígio e perda financeira ao condomínio inteiro.

O Master Síndico inverte a lógica: oferece um **score composto por evidência auditável** que o síndico vê **antes de abrir um Connect Me**, **antes de aceitar uma proposta**, **antes de assinar um Negócio Fechado**. A empresa não se auto-declara "5 estrelas" — o sistema calcula o tier a partir de métricas reais da plataforma, e o síndico pode clicar e ver **exatamente** os números que geraram aquele tier (auditabilidade total).

Citação canônica — `master-sindico-v2/analise-crua.md`:

> _"Síndico clica no badge Platina de uma empresa e vê as métricas que geram o tier (45 contratos, 4,2 de média em 38 avaliações, 18 condomínios diferentes, 2 vídeos institucionais, 1 case). Compare com 'estrelas' genéricas de marketplaces onde você nunca sabe se foi review pago. Anti-fraude estrutural: impossível comprar tier porque os inputs são observáveis e auditáveis."_

Os **4 pilares de governança** do Master Síndico (`regras-canonicas.md` §1):
1. **Critério** — contratações por método, não urgência
2. **Registro** — memória institucional auditável (Timeline)
3. **Reputação** — sistema de status baseado em evidência (Bronze → Diamante) ← este sub-produto
4. **Integração** — ecossistema unificado

---

## 3. O que NÃO é (anti-definição taxativa)

**NÃO é estrelas** — não há input de usuário "dê sua nota geral de 1 a 5". Há 5 critérios específicos 1-10 cada (detalhe §11); a média **por avaliação** é derivada, a reputação **da empresa** é um score composto de múltiplas avaliações + outras métricas.

**NÃO é ML / não é IA / não é black-box**. ADR-0023 é explícita: ranking/reputação com modelo estatístico não-explicável é **vetoado em M1/M2** (LGPD Art. 20 exige explicabilidade de decisões automatizadas). Reavaliação ML só a partir de M3+ **e mesmo assim condicionada a ≥ 10k interações/mês + ≥ 1k empresas + validação stakeholder**. Primeira iteração (se entrar): LambdaMART/XGBoost batch diário + resultado cached Redis. **Nunca deep learning.**

**NÃO é ranking editorial** / não há "editor Master Síndico escolhendo empresa destaque". Não há boost promocional. Não há "featured" pago. Não há admin que "aumenta tier manualmente". GR-028: operador Master Síndico que tenta `PATCH /companies/{id}/reputation_tier` recebe **403**.

**NÃO é ranking público no sentido Netflix/Yelp**. Herança do Manual original (§8.5, extracts/manual.txt linhas 2827-2834 do cliente): _"Não é ranking / Não compara síndicos entre si / Não gera competição pública / Não é gamificação agressiva / Não substitui avaliação de gestão"_. Transplantado para empresa no v2: reputação alimenta **ordenação** (usada como critério de ordenação dentro de busca Connect Me com outros fatores — ADR-0023) e **filtro** ("mostrar apenas Ouro+"), mas não há "top 10 empresas da semana" como feed público.

**NÃO é o sistema original do Manual (síndico)**. O Manual Cap. 8 original (Mar/2026) descrevia **reputação do SÍNDICO** (Bronze/Prata/Ouro/Diamante) a partir de vídeos assistidos + condomínios + módulos concluídos. A destilação v2 (Abril/2026) pivotou para **reputação da EMPRESA** porque (a) é o alvo comercial-chave do produto (empresas pagam Plus/Pro), (b) métricas observáveis contratuais (Deals+Evaluations) são muito mais robustas que "assistiu vídeo" (fraude: autoplay, bots), (c) o Manual já determinou anti-gamificação em §8.5. Reputação de síndico pode retornar M3+ como feature complementar (fora de escopo nesta destilação).

**NÃO é campo editável pelo dono**. Empresa não edita próprio tier. Empresa **vê** seu tier e o breakdown de componentes. Pode **recorrer** via suporte apenas em caso de denúncia infundada ou dado observável incorreto (ex: avaliação atribuída a outra empresa) — nunca em caso de "não concordo com a nota".

**NÃO é score financeiro / de crédito** (tipo Serasa/SPC). Nada sobre pagamento, faturamento, default. É exclusivamente reputação **operacional** no escopo do ecossistema Master Síndico.

**NÃO é síncrono com a avaliação de gestão do síndico** (Req 15 institucional) — reputação da empresa é `commercial`; avaliação de gestão é `institutional` (morador → síndico). Contextos separados.

---

## 4. Tiers completos (5) — thresholds exatos

Autoridade canônica: `master-sindico-v2/04-requirements/functional/commercial.md` COM-025 + `business-rules.md` BR-029 + `state-machines.md` §6 + `portfolio-de-produtos.md` Produto 3 + `STATE.md` D-051 + `analise-crua.md`.

| Tier | Requisito acumulado (AND lógico — todas as condições) | Componente dominante |
|---|---|---|
| **Bronze** | Default ao criar empresa — cadastro completo + CNPJ verificado via Receita Federal. Qualquer empresa `active` começa aqui | Entrada no ecossistema |
| **Prata** | **≥ 3** contratos fechados (`ClosedDeal.status=signed` imutável) **E** avaliação média **≥ 3/5** (2 dígitos; normalização 1-10 → 0-5: dividir por 2) | Base operacional mínima |
| **Ouro** | **≥ 10** contratos fechados **E** avaliação média **≥ 3,5/5** **E** **≥ 3** condomínios distintos atendidos (deals com `condominium_id` diferente) | Presença comprovada no mercado |
| **Platina** | **≥ 30** contratos fechados **E** avaliação média **≥ 4,0/5** **E** **≥ 10** condomínios distintos **E** **≥ 1** vídeo institucional aprovado (moderação `approved`) **E** **≥ 1** case registrado em vídeo | Profissionalização + conteúdo |
| **Diamante** | **≥ 100** contratos fechados **E** avaliação média **≥ 4,5/5** **E** **≥ 30** condomínios distintos **E** **≥ 3** vídeos institucionais aprovados **E** **múltiplos cases** em vídeo **E** tempo de plataforma **≥ 12 meses** | Referência de mercado |

**Semântica dos thresholds simbólicos**: `3 / 10 / 30 / 100` contratos são **proxies de maturidade** (D-051) — também funcionam como eixo do componente `volume_cases_capped` (cap em 50 na fórmula pura §5).

**Normalização de escala 1-10 → 0-5**: Req 26 usa escala 1-10 em cada critério (código Go `CompanyEvaluation`), mas os thresholds de tier acima usam escala 0-5 (legado institucional.md Req 6.2). A normalização canônica é `rating_0_5 = rating_1_10 / 2` (aplicada na função pura antes de comparar com thresholds).

**Cold start** (D-051): empresa nova sem reviews **não recebe Bronze-zero** — recebe rótulo visual **"Não classificada"** por 30 dias **OU** até completar 3 contratos, o que vier primeiro. Razão: empresa entrar e já ser "Bronze (o mais baixo)" cria estigma injusto; melhor sinal honesto "sem histórico".

**Transições proibidas** (state-machines §6, fechamento A-DC-QA-061):
- ❌ Pular tier (ex: Bronze → Ouro sem Prata intermediário) mesmo que inputs bateriam Ouro — recálculo é sequencial tier a tier (avaliação crescente)
- ❌ Rebaixamento para estado fora da enum (não há "canceled" / "deleted" — se empresa `status=blocked` o tier continua mas não é exibido publicamente)

---

## 5. Fórmula determinística — pesos canonizados (D-051)

Score ∈ [0, 100]:

```
score = 40 * rating_avg_weighted       -- peso 40%: média de avaliações ponderada por valor do contrato (Money em centavos)
      + 25 * success_rate              -- peso 25%: contratos_concluidos / (concluidos + atrasados + cancelados_por_fault)
      + 15 * tenure_years_capped       -- peso 15%: ln(1 + anos_ativo_na_plataforma) * 15 / ln(1 + 10)  (plafond 10 anos)
      + 10 * volume_cases_capped       -- peso 10%: min(contratos_fechados_assinados, 50) / 50
      + 10 * content_engagement        -- peso 10%: views_video_institucional_normalizados + seal_sindicos_recebidos
```

**Mapa score → tier** (D-051):

| Faixa de score | Tier |
|---|---|
| [0, 40) | Bronze |
| [40, 60) | Prata |
| [60, 75) | Ouro |
| [75, 88) | Platina |
| [88, 100] | Diamante |

**Consistência com §4**: os thresholds narrativos (3/10/30/100 contratos, 3/3,5/4/4,5 avg) atuam como **gates mínimos explícitos** que empresa precisa cumprir AND-lógico; a fórmula numérica produz o score contínuo. Em caso de score > faixa de tier mas gate mínimo não cumprido (ex: score=65 mas só 8 contratos): **tier fica em Prata**, não Ouro. A função pura aplica gates **antes** do corte por faixa.

**Rationale por peso** (dual-check §3.3 + analise-crua):
- `rating_avg_weighted` 40%: é o sinal mais forte de qualidade (avaliação 1-10 em 5 critérios, Req 26). Ponderar por valor do contrato (`ClosedDeal.totalAmountCents`) evita que 10 pequenas avaliações 10/10 valham o mesmo que 1 grande contrato crítico.
- `success_rate` 25%: objetividade contratual — numerador "concluído" vem de `ExecutionRecord.status=validated` pelo síndico (Req 23). Denominador inclui `canceled_by_fault` (empresa causou), excluindo cancelamentos por motivo neutro.
- `tenure_years_capped` 15%: experiência conta, mas log-cap em 10 anos impede "dinossauro" que congelou há 8 anos dominando ranking. Fórmula `ln(1+x) * 15/ln(11)` retorna 0 em x=0 e 15 em x=10.
- `volume_cases_capped` 10%: volume capped em 50 contratos — não premiar exclusivamente empresas grandes sobre pequenas boas.
- `content_engagement` 10%: incentiva empresas que investem em conteúdo educativo (vídeos institucionais aprovados + seal "Síndico-aprovado" recebidos). Decisão: 10% é teto — conteúdo é **complementar**, não substitui entrega contratual.

**Propriedade anti-manipulação** (threat model §15): dos 5 componentes, **4 exigem dupla-assinatura ou validação de síndico real** (contrato só conta pós-`SyndicSign`, avaliação só de síndico com membership válido, case em vídeo passa por moderação). Somente `content_engagement` tem componente autopublicável (views), mitigado por peso baixo + moderação obrigatória + Gap-SEC-011 antivírus + Rule 4 visibilidade estrita.

---

## 6. Inputs auditáveis (5 fontes + sua origem exata)

Todos os inputs são **observáveis na base de dados** e **registrados em audit trail append-only** (Req 37). Nenhum input é "declarado" pela empresa.

| Input | Fonte primária | Tabela/agregado | Contagem |
|---|---|---|---|
| **Contratos fechados** | `commercial/entities/closed_deal.go` `status='signed'` AND `isImmutable=true` | `closed_deals` | `COUNT(*) WHERE company_id=? AND status='signed'` |
| **Avaliação média ponderada** | `commercial/entities/company_evaluation.go` (`averageScore` normalizada 0-5) × `ClosedDeal.totalAmountCents` | `company_evaluations` JOIN `closed_deals` | `SUM(avg_score * deal_amount) / SUM(deal_amount)` |
| **Condomínios distintos atendidos** | `closed_deal.condominium_id` | `closed_deals` | `COUNT(DISTINCT condominium_id) WHERE company_id=? AND status='signed'` |
| **Vídeos institucionais aprovados** | BC `content` — `Video.type='institucional' AND moderation_status='approved'` | `videos` | `COUNT(*) WHERE tenant_id=? AND type='institucional' AND moderation_status='approved'` |
| **Cases em vídeo** | BC `content` — `Video.subtype='case' AND moderation_status='approved'` | `videos` | idem com filtro subtype |
| **Tempo de plataforma** | `companies.created_at` — primeira ativação `status=active` | `companies` | `NOW() - activated_at` |
| **Success rate** | `execution_records.status='validated'` vs `canceled_by_fault` + `closed_deals.status='canceled'` com `cancelReason` atribuído à empresa | `execution_records`, `closed_deals` | ratio |
| **Content engagement** | Views agregadas + selos `Seal` concedidos por síndicos com `plan_tier >= plus` | `video_views`, `seals` | normalizado 0-1 |

**Atenção `company_evaluations` x 5 critérios** (detalhe §11): cada avaliação tem 5 notas 1-10, e o código já calcula `averageScore = (sum 5 notas) / 5` no construtor (`company_evaluation.go:54`). A fórmula de Reputação consome esse `averageScore` dividido por 2 (normaliza para escala 0-5).

**LGPD + Retenção**: inputs são dados **agregados observáveis** da operação (não são dados pessoais sensíveis). `evaluator_user_id` nunca vaza para empresa (INV-044); no payload de `ReputationTierChanged` é **hashed** (`domain-events.md` E-040 payload mostra `evaluator_user_id_hashed`).

---

## 7. Recálculo em eventos (síncrono UoW)

**Regra de ouro**: cada evento que altera um input dispara `RecalculateReputation(companyID)` **dentro da Unit of Work** da mesma transação. Idempotente — chamar N vezes com mesmos inputs retorna mesmo tier.

### Eventos gatilho (fonte canônica: `domain-events.md` E-040..E-043 + `business-rules.md` BR-028)

| Evento | Publisher | Momento | Ação |
|---|---|---|---|
| `CompanyEvaluationSubmitted` (E-040) | commercial (handler de `SubmitEvaluationUseCase`) | Síndico submete avaliação pós-contrato (Req 26) | Handler interno: `ReputationCalculator.Recompute(companyID)` dentro UoW |
| `ClosedDealSigned` / `deal.signed_complete` (E-039 variante) | commercial (handler de `SyndicSign`) | Dupla assinatura completa (imutável) | `RecalculateReputation(companyID)` |
| `ExecutionRecordValidated` | commercial | Síndico valida execução (Req 23) | Recalcula — afeta `success_rate` |
| `VideoApproved` (E-033 content) | content | Moderador aprova vídeo institucional/case | Subscriber em `commercial` chama `RecalculateReputation` — afeta `content_engagement` |
| `HarassmentReportConfirmed` | commercial (handler admin) | Admin confirma denúncia (ação `suspension` ou `ban`) | Seta `suspended_until = now() + 72h` (§9) — **não** recalcula tier |
| `SealGranted` | commercial | Síndico com `plan_tier >= plus` concede "Síndico-aprovado" | Recalcula — afeta `content_engagement` |

### Evento emitido

`ReputationTierChanged` (E-041):
- **Payload**: `{ company_id, previous_tier, new_tier, calculated_at, inputs: { contracts, avg_rating, condos_count, videos, tenure_months, engagement } }`
- **Subscribers**: `cross-domain` (audit trail append-only + notificação push empresa); frontend (atualiza badge em tempo real via websocket ou poll curto)
- **Idempotência**: handler `RecalculateReputation` é idempotente (INV-054) — se `new_tier == previous_tier`, evento **não** é emitido.

### Decisão síncrono vs assíncrono

Fonte: `state-machines.md` P-SM-006 + `domain-events.md` P-EVT-006 — sinal pendente em destilação indicava "Legado sugere reactive dentro da UoW de `CompanyEvaluationSubmitted`. Confirmar custo." D-051 congelou **síncrono**: cálculo é O(N) em N = avaliações + contratos da empresa (indexável, tipicamente < 1000 em M2); cache futuro em Redis se necessário (backlog T-153).

Job noturno adicional (`02-architecture/c4-components.md` `reputation_recompute_job.go`): varredura diária de todas empresas `active` para recalcular casos onde input mudou por efeito colateral (ex: condomínio foi removido → `condos_count` caiu sem evento específico). Executado em TZ `America/Sao_Paulo` (D-052).

---

## 8. Degradação permitida

**Regra**: tier **pode descer**. Mesma função, mesmos inputs, resultado diferente conforme inputs mudam.

Cenários (BR-030):
- Avaliação média despenca após N avaliações ruins consecutivas
- Contratos cancelados por fault da empresa entram em `success_rate` como denominador
- Vídeos institucionais são removidos ou invalidados → `content_engagement` cai
- Condomínios deixam de ser atendidos (tempo passa sem novo deal em determinado condo) → `condos_count` ativo cai (definição de "ativo": decay temporal ainda não implementado — Gap-SEC-008)

**Pontos críticos**:
- Não há "floor" — empresa pode ir de Diamante a Prata em uma janela curta se inputs justificarem (cenário raro, mas possível). Evento `ReputationTierChanged` com `previous_tier=Diamante, new_tier=Prata` é emitido; notificação específica (email + in-app) alerta a empresa com breakdown do que mudou.
- Não há "período de carência" de rebaixamento — se cruzou o threshold, recálculo no próximo evento reflete. Exceção: **harassment não rebaixa** (§9).
- Test: `T-COM-sm-rep-degradation-reversible` (state-machines.md §6) — property-based testing (PBT): input series que piora progressivamente resulta em tier monotonicamente não-crescente.

---

## 9. Suspensão temporária por harassment confirmado

Fonte canônica: `business-rules.md` BR-031 + `invariants.md` INV-055 + COM-026 + `commercial.md` Req 19 anti-assédio + `aggregates/Company.md` `ApplyHarassmentSuspension()`.

**Regra**: quando `HarassmentReportConfirmed` (admin escolheu ação `suspension` ou `ban` em `HarassmentAdminAction`), o aggregate `Company` ativa:

```go
c.suspendedFlag  = true
c.suspendedUntil = time.Now().Add(72 * time.Hour)
// reputationTier PERMANECE inalterado
```

**Semântica**:
- Não é despromoção. O tier permanece (ex: empresa era Ouro → continua Ouro internamente)
- Flag `suspended` é **paralela** ao tier, não o substitui
- UI renderiza badge: "Ouro — Suspensa temporariamente" (visual cinza + alerta)
- Acesso a fluxos comerciais (Connect Me, propostas, fechamento de deal) bloqueado durante janela 72h
- Após expiração `suspended_until`, handler automático reseta flag e empresa retoma operação

**Por que 72h?** Janela de moderação manual: operador Master Síndico tem até 72h para investigar denúncia, coletar evidência, decidir escalar para `ban` permanente ou encerrar com `warning`. Se em 72h nada mudou, flag expira.

**Acão `ban` permanente** (enum `HarassmentAdminActionBan` em `enums.go:161`): `company.status = 'blocked'` (terminal no enum `CompanyStatus`). Neste caso tier **não é exibido** (empresa blocked é removida de buscas públicas). Tier no banco permanece como registro histórico.

**Test**: `T-COM-sm-rep-suspended-flag` (state-machines.md §6).

---

## 10. Aggregates (DDD tático)

Fonte: `01-domain/aggregates/Company.md`, `CompanyEvaluation.md`, `bounded-contexts.md` §4 commercial.

### 10.1 `Company` (Aggregate Root)

BC: `commercial` · Tenant próprio (`tenant_type=company`)

Campos relevantes à Reputação:
```go
type Company struct {
    id                CompanyID
    cnpj              CNPJ
    razaoSocial       string
    status            CompanyStatus        // pending_verification | active | suspended | blocked
    reputationTier    ReputacaoTier        // Bronze | Prata | Ouro | Platina | Diamante (enum pt-BR)
    suspendedFlag     bool                 // flag 72h harassment (paralela ao tier)
    suspendedUntil    *time.Time
    createdAt, updatedAt time.Time
}
```

Métodos:
- `UpdateReputationTier(newTier ReputacaoTier, inputs ReputationInputs) error` — chamado exclusivamente por `ReputationCalculator.Recompute(...)`
- `ApplyHarassmentSuspension(until time.Time)` — seta flag, NÃO mexe no tier (BR-031)
- `LiftHarassmentSuspension()` — reset flag ao expirar 72h

Invariantes:
- INV-003: CNPJ UNIQUE
- INV-036: Reputação é função determinística
- INV-054: `ReputationTierChanged` dispara recálculo idempotente
- INV-055: Harassment suspende (flag) sem rebaixar tier

Eventos emitidos (relevantes):
- `ReputationTierChanged` (E-041)

### 10.2 `CompanyEvaluation` (Aggregate Root — INPUT da Reputação)

BC: `commercial`

```go
type CompanyEvaluation struct {
    id                 EvaluationID
    companyID          CompanyID
    evaluatorUserID    UserID            // síndico — NUNCA exposto para empresa (INV-044)
    condominiumID      CondominiumID
    dealID             DealID
    criteria           EvaluationCriteria  // 5 notas 1-10
    averageScore       float64             // (sum 5 notas) / 5 — calculada no construtor
    isAnonymousToCompany bool               // default true
    flags              []EvaluationFlag    // enum qualitativos opcionais
    comment            string
    submittedAt, createdAt time.Time
}
```

Invariantes:
- INV-043: UNIQUE `(company_id, evaluator_user_id, condominium_id)` — 1 avaliação por trio (síndico só avalia 1x a empresa X no condomínio Y; voltará se deal novo for criado)
- INV-044: anônima — API response mapper filtra `evaluator_user_id` pra response da empresa
- Imutável pós-criação (sem setters)

Eventos emitidos:
- `CompanyEvaluationSubmitted` (E-040)

### 10.3 `ReputationEvent` (histórico append-only — **a criar na Sprint 11**)

**Nota de estado**: entidade **não existe ainda no código** — é spec do v2 (backlog T-151..T-155). Nome tentativo: `reputation_events` (tabela append-only) ou `reputation_snapshots` (uma linha por recálculo que mudou tier).

Proposta de schema canônico:
```sql
CREATE TABLE reputation_snapshots (
    id                UUID PRIMARY KEY,
    company_id        UUID NOT NULL REFERENCES companies(id),
    previous_tier     reputacao_tier NOT NULL,
    new_tier          reputacao_tier NOT NULL,
    score             NUMERIC(5,2) NOT NULL CHECK (score >= 0 AND score <= 100),
    components        JSONB NOT NULL,  -- { rating_avg_weighted, success_rate, tenure_years_capped, volume_cases_capped, content_engagement }
    inputs_snapshot   JSONB NOT NULL,  -- { contracts_count, avg_rating, condos_distinct, videos_approved, tenure_months }
    triggered_by_event TEXT NOT NULL,  -- "CompanyEvaluationSubmitted" | "ClosedDealSigned" | ...
    calculated_at     TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_reputation_snapshots_company_time ON reputation_snapshots(company_id, calculated_at DESC);
-- Sem updated_at: append-only (INV-033 retention 5 anos)
```

Rationale do snapshot: permite `GET /companies/{id}/reputation-breakdown` (§7 COM-027) retornar **última snapshot** + histórico de transições; também cumpre LGPD Art. 20 explicabilidade (quais dados/cálculos geraram tal decisão).

---

## 11. Enums

### 11.1 `ReputacaoTier` (VO Enum pt-BR)

Fonte: `ubiquitous-language.md` + `bounded-contexts.md` §4 + `registration-data.md` §3.4.

```go
type ReputacaoTier string

const (
    ReputacaoTierBronze   ReputacaoTier = "bronze"
    ReputacaoTierPrata    ReputacaoTier = "prata"
    ReputacaoTierOuro     ReputacaoTier = "ouro"
    ReputacaoTierPlatina  ReputacaoTier = "platina"
    ReputacaoTierDiamante ReputacaoTier = "diamante"
)
```

**Observação crítica de drift**: o OpenAPI atual (`backend/docs/openapi.yaml` linha 116) define `enum: [bronze, prata, ouro, diamante]` — **4 tiers** — para `Plan.tier` (stale — deve ser `trial/base/plus/pro` — ADR-0032) e também deixa subentendido 4 tiers de reputação. A canonização v2 **fixa 5 tiers** (inclui Platina) — a correção do OpenAPI está no backlog de M2.

**Decisão pt-BR**: `bounded-contexts.md` §4 nota explícita: "enum pt-BR" (diferente de `CompanyStatus` EN). Razão: tiers Bronze/Prata/Ouro/Platina/Diamante são **termos de produto/marketing** em pt-BR. Se um dia internacionalizar, criar mapping i18n na camada de apresentação, não renomear no domínio.

### 11.2 `EvaluationCriteria` — 5 critérios (Req 26) — síndico → empresa

Fonte canônica do código Go: `commercial/domain/entities/company_evaluation.go:17-32` (campos) + `:54` (média).

| ID no código | Label canônico (Req 26 / commercial.md) | Escala | Alias no Obsidian (Domínio - Comercial.md linhas 186-194) |
|---|---|---|---|
| `criterionQuality` | Qualidade do trabalho | 1-10 | `qualidade_servico` |
| `criterionPricing` | Adequação de custos | 1-10 | `custo_beneficio` |
| `criterionPunctuality` | Cumprimento de prazos | 1-10 | `cumprimento_prazo` |
| `criterionProfessionalism` | Profissionalismo | 1-10 | `atendimento` (aproximação semântica) |
| `criterionCommunication` | Comunicação | 1-10 | `recomendacao` (divergência — ver nota) |

**Nota de divergência documental**: a lista do Obsidian Vault (Mar/2026) usa 5 nomes ligeiramente diferentes dos campos Go (Abr/2026). A **fonte canônica vencedora** é o código Go + commercial.md Req 26 (hierarquia de fontes CLAUDE.md §7: "decisões vivas / spec viva vence"). Reconciliação:
- `qualidade_servico` → `criterionQuality` ✅
- `cumprimento_prazo` → `criterionPunctuality` ✅
- `custo_beneficio` → `criterionPricing` ✅
- `atendimento` e `recomendacao` do Obsidian foram **consolidados** em `criterionProfessionalism` + `criterionCommunication` — o campo "recomendacao" foi removido (NPS-like era anti-padrão, não era multi-critério puro). Essa reconciliação é finding **A-### novo** a registrar em AUDIT (será levantado no dual-check desta destilação).

`averageScore` é calculado como `(quality + pricing + punctuality + professionalism + communication) / 5.0` (float 1-10). A fórmula de Reputação (§5) consome essa média dividida por 2 para escala 0-5.

### 11.3 `EvaluationFlag` (opcional, qualitativo)

Sugerido em `aggregates/CompanyEvaluation.md` — enum não canonizado ainda (pendência):
- `pontualidade` (positiva)
- `preco_justo` (positiva)
- `comunicacao_excelente` (positiva)

Uso: `content_engagement` auxiliar (não é componente da fórmula principal) — empresa com flags positivas recebe pequeno bonus. **Congelado em M1**.

### 11.4 `HarassmentAdminAction` (já no código: `enums.go:158-161`)

```go
HarassmentAdminActionWarning    = "warning"
HarassmentAdminActionSuspension = "suspension"
HarassmentAdminActionBan        = "ban"
```

Somente `suspension` e `ban` impactam reputação (§9). `warning` gera log sem mudança de estado.

---

## 12. Regras de negócio citadas com fonte

Hierarquia de autoridade: código Go/migrations (se divergir da spec) > spec `.kiro/requirements/*.md` > v2 destilado (`business-rules.md`, `invariants.md`) > Obsidian Vault > Manual original.

| ID | Regra | Fonte |
|---|---|---|
| **BR-003** | Reputação é função pura / determinística | `master-sindico-v2/01-domain/business-rules.md` · STEERING §39 |
| **BR-028** | Função pura de (closed_deals, avg_rating, condos_count, harassment, videos_approved); recalcula em cada `CompanyEvaluationSubmitted` | `business-rules.md` BR-028 |
| **BR-029** | Thresholds 3/10/30/100 contratos + avg 3/3,5/4/4,5 por tier | `business-rules.md` BR-029 · `04-requirements/functional/commercial.md` COM-025 |
| **BR-030** | Degradação permitida — tier pode descer | `business-rules.md` BR-030 |
| **BR-031** | Harassment confirmado: flag `suspended=true` 72h, NÃO rebaixa tier | `business-rules.md` BR-031 · INV-055 |
| **BR-032** | Reputação global (calculada sobre todos os contratos); avaliações em detalhe visíveis só ao próprio tenant; tier global público | `business-rules.md` BR-032 |
| **GR-028** | Operador Master Síndico que tenta PATCH direto em `reputation_tier` → 403 | `04-requirements/global.md` GR-028 |
| **COM-025** | Tabela de requisitos por tier (EARS) | `04-requirements/functional/commercial.md` |
| **COM-026** | Suspensão temporária por harassment (72h) | `04-requirements/functional/commercial.md` |
| **COM-027** | Breakdown visível: `GET /companies/{id}/reputation-breakdown` — zero black box | `04-requirements/functional/commercial.md` |
| **INV-036** | Função determinística (sem randomness; mesmo input → mesmo output) | `invariants.md` · DR-016 |
| **INV-043** | CompanyEvaluation UNIQUE (company, evaluator, condominium) — 1 avaliação única | `invariants.md` · DR-006 · A-029 legado fix |
| **INV-044** | CompanyAssessment anônima — response mapper filtra `evaluator_user_id` | `invariants.md` · Req 26 |
| **INV-054** | `ReputationTierChanged` dispara recálculo idempotente | `invariants.md` |
| **INV-055** | Harassment confirmado → `suspended` flag, não rebaixa tier | `invariants.md` |
| **D-051** | Fórmula pesos 40/25/15/10/10 + 5 tiers + thresholds | `STATE.md` |
| **D-010** | Parametrização aberta (histórico — resolvido por D-051) | `STATE.md` · AUDIT.md linha 220 |
| **ADR-0023** | Ranking Connect Me determinístico; `reputation_tier_bonus` como w6 | `02-architecture/adr/0023-recsys-deterministic-m1.md` |
| **ADR-0029** (planejada) | "reputacao-formula-tiers" — congela fórmula; qualquer mudança exige dual-check + pentester pass | `STATE.md` D-051 Impacto |

---

## 13. Invariantes

| ID | Invariante | Enforcement |
|---|---|---|
| INV-036 | Reputação é função determinística — sem randomness, sem input humano direto | Domain service `ReputationCalculator.Compute(company)` puro (property-based test: mesmo input → mesmo output 100% das iterações) |
| INV-054 | `ReputationTierChanged` dispara recálculo e notificação — handler é idempotente | Teste de integração: 2x chamada = 1x efeito |
| INV-055 | `HarassmentReportConfirmed` suspende empresa via flag, não rebaixa tier | Teste: simular harassment confirmado — assert(`tier == previous_tier AND suspended_flag == true`) |
| **Zero input humano direto** | GR-028: operador MS que tenta `PATCH /companies/{id}/reputation_tier` → 403 | ABAC policy nega; route handler não aceita campo `reputation_tier` em body |
| **Transparência total** | COM-027: endpoint `/companies/{id}/reputation-breakdown` retorna componentes + inputs snapshot | Integration test — breakdown sempre disponível, sempre atualizado |
| **Explicabilidade (LGPD Art. 20)** | Endpoint `/companies/{id}/ranking-explain` (quando aparece em ranking Connect Me) retorna top-5 fatores + contribuições | ADR-0023 — contrato OpenAPI |
| **Imutabilidade da snapshot** | `reputation_snapshots` (tabela) é append-only — sem UPDATE/DELETE | Schema sem `updated_at`; políticas SQL RLS + trigger deny UPDATE/DELETE |

---

## 14. Dependências cross-sub-produto

Reputação **não é isolada** — é consumidora e produtora de eventos em outros sub-produtos:

| Dependência | Direção | Uso |
|---|---|---|
| **02 Connect Me** | ⬅ + ⮕ | Produtor: evento `ClosedDealSigned` dispara recálculo. Consumidor: feed de empresas em Connect Me é ordenado com `reputation_tier_bonus` como w6 no score composto (ADR-0023) |
| **04 Conteúdo & Vídeos** | ⬅ | Evento `VideoApproved` (moderação) incrementa `content_engagement`. `VideoRemoved/Rejected` decrementa. Case em vídeo é subtype `case` |
| **01 Governança Institucional** | ⬅ + ⮕ | Input: `condos_distinct_count` vem de `closed_deals.condominium_id` distintos (fonte: `institutional` via `Condominium` aggregate). Consumidor: perfil público do síndico pode exibir tier das empresas contratadas no último mandato (futuro) |
| **05 Assembleia Live** | ⬅ | Votação de fornecedor (Req 21 — 2+ propostas validadas): UI ordena por reputation_tier (desc) para facilitar decisão do conselho |
| **07 Banco de Talentos** | — | Independente (talentos é morador↔empresa, reputação é síndico↔empresa) |
| **08 Vizinhança** | — | Independente (Vizinhança tem sistema próprio de Selo + Cupom — não usa Bronze→Diamante) |
| **09 Compliance** | ⮕ | Dashboard Governance Score (Req 33) pode exibir "% empresas contratadas Ouro+" como indicador de critério de contratação |
| **10 Marketing Agências** | ⮕ | Agência vê tier de cada empresa que gerencia; contribui indiretamente para `content_engagement` (publica vídeo aprovado → tier sobe) |
| **identity** | ⬅ | ABAC — usa `reputation_tier` em policies (ex: "somente Ouro+ pode emitir proposta em RFP Premium" — futuro) |
| **billing** | ⬅ | Não afeta subscription (tier de plano é separado — `trial/base/plus/pro`); mas analytics pode correlacionar (empresa Pro com Diamante vs Plus com Bronze) |

**Não-depende-de**:
- Não depende do `plan_tier` da empresa (`plus` vs `pro`) para cálculo do tier reputacional. Empresa com plano `plus` pode ser Diamante; Empresa `pro` pode ser Bronze. Tiers comerciais (plano) e tiers reputacionais são **eixos independentes**.
- Não depende de tier de síndico (reputação de síndico não está em escopo v2 — remanente do Manual original §8 não foi portado).

---

## 15. Anti-fraude estrutural (inputs observáveis)

A fórmula foi desenhada para que **não exista caminho curto para Diamante**. Cada componente requer ação de terceiro legítimo:

| Componente | Por que difícil fraudar |
|---|---|
| `contratos_fechados` | Cada contrato exige dupla assinatura (`CompanySign` + `SyndicSign`); `ClosedDeal.isImmutable=true` após assinatura. Criar contrato fantasma exige síndico cúmplice com condomínio real + membership válido + IP + audit trail (Req 37) |
| `avaliação_media` | Avaliação só é submetida por síndico com `ClosedDeal.syndicSignedBy == evaluatorUserID` (use case valida) e UNIQUE (company, evaluator, condominium) — não pode avaliar a mesma empresa 10x no mesmo condomínio |
| `condomínios_distintos` | `condominium_id` distintos exige contratos em **condomínios verificados** (CNPJ validado — Req 18) e síndicos com membership válido no condomínio (`institutional` BC) |
| `vídeos_aprovados` | Moderação manual (M1-M3); ML moderação a partir de M4+. Vídeo precisa passar `moderation_status='approved'`. Quota mensal por plano (Plus 8, Pro 12) — não é "upar 100 vídeos em 1 dia" |
| `cases_em_vídeo` | Subtype `case` exige conteúdo específico (estudo de caso real com condomínio identificável) — moderação manual flagra genéricos |
| `tempo_plataforma` | Ativação oficial (`companies.status = active`) exige CNPJ verificado via Receita Federal (M2+) — não é "criar conta hoje e aparecer com 5 anos de tenure" |

**Gaps conhecidos** (threat model §15):
- **Gap-SEC-007** (CNPJ Receita Federal ausente em M1): mitigado por CNPJ formato válido + unicidade + admin aprovação manual da transição `pending_verification → active`
- **Gap-SEC-008** (decay temporal ausente): empresa com 100 contratos em 2022 e 0 em 2023-2024 mantém Diamante — resolvido em M3 com decay mensal (detalhe §16)
- **Gap-SEC-009** (dual-approval em moderação ausente): single moderador pode aprovar vídeo fraudulento — resolvido em M3
- **Gap-SEC-010** (farm detection / ring detection ausente): anel de condomínios-síndicos fantasmas poderia inflar avaliações — resolvido em M4 com ML-based (detalhe §16)

---

## 16. Gap-SEC-008 decay temporal + Gap-SEC-010 farm detection (herança Fase 5)

### Gap-SEC-008 — Decay temporal

**Problema**: empresa com histórico forte em 2022 (100 contratos, avg 4,8) mantém **Diamante indefinidamente** em 2024 mesmo sem novos contratos. Inputs são "stock", não "flow" — qualquer janela temporal adequada resolveria.

**Mitigação prevista (M3)**: decay mensal aplicado a componentes "stock":
- `rating_avg_weighted`: janela rolling 24 meses (média ponderada com half-life 12m). Avaliações mais antigas contam menos.
- `volume_cases_capped`: contratos > 36 meses deixam de contar (hard cutoff).
- `condos_distinct`: condomínio é "ativo" só se houve deal nos últimos 18 meses.

Implementação: job noturno `reputation_decay_job.go` percorre empresas ativas, recalcula com janela. Impacto: tier pode descer por **passagem do tempo** sem evento novo — requer comunicação explícita ao mercado ("Reputação reflete momento atual" — herança Manual §8.3: _"O status reflete momento atual de engajamento, não histórico acumulado eterno"_).

**Registro**: `threat-model.md` §15 Gap-SEC-008 · `STEPS M3` a agendar.

### Gap-SEC-010 — Farm / ring detection

**Problema**: operação coordenada em que empresa X cria 5 condomínios fantasmas com 5 síndicos cúmplices, cada um fecha 20 deals pequenos com avg 5/5 → empresa sobe a Diamante em 6 meses.

**Mitigação prevista (M4)**: detecção ML-based + heurísticas:
- Density-based clustering: se grafo (empresa → síndicos → condomínios) é suspeito (ex: alta concentração, baixa entropia de IP/ASN), flag para revisão
- Correlação temporal: 20 deals criados/assinados em janela < 1 semana é anomalia
- Cross-check com dados de Receita Federal (CNPJ do condomínio ativo, síndico CPF válido, etc)

Implementação: jobs ML + human-in-the-loop (flag + admin revisa + opcional `HarassmentReportConfirmed` com `ban`).

**Registro**: `threat-model.md` §15 Gap-SEC-010 · `STEPS M4` a agendar.

---

## 17. Estado no código (2026-04-23, Sprint 9)

| Item | Status | Evidência |
|---|---|---|
| Aggregate `Company` implementado | ✅ parcial | `company.go` tem `id, cnpj, status, descrição, categorias, contato, endereço`; **faltam** `reputation_tier`, `suspended_flag`, `suspended_until` |
| Aggregate `CompanyEvaluation` implementado | ✅ completo | `company_evaluation.go` — 5 critérios 1-10, média automática, anônima por default, UNIQUE em migration |
| Aggregate `ClosedDeal` implementado | ✅ completo | `closed_deal.go` — dupla assinatura, imutabilidade pós-`SyndicSign` |
| Enum `ReputacaoTier` | ❌ não existe | Enum não está em `commercial/domain/enums/enums.go`. OpenAPI legado cita 4 tiers (sem Platina) |
| Migration `00301_create_companies.sql` | ⚠️ incompleta | Sem colunas `reputation_tier`, `suspended_until`. Migration adicional será necessária |
| Migration `company_evaluations` | ✅ completa | `00308` — UNIQUE, CHECK 1-10, average NUMERIC(4,2) |
| `ReputationCalculator` domain service | ❌ não existe | Pasta `commercial/domain/services/` existe mas vazia. Task BE-106 |
| Use case `SubmitEvaluationUseCase` | ✅ parcial | `company_evaluation_use_cases.go` cria eval + persiste. **Não chama** `RecalculateReputation` ainda |
| Use case `CloseDealUseCase` | ✅ | `closed_deal_use_cases.go` — mas sem trigger de recálculo |
| Evento `CompanyEvaluationSubmitted` | ❌ não publicado | Handler existe em use case, mas sem `IEventPublisher.Publish(...)` integrado |
| Evento `ReputationTierChanged` | ❌ não existe | Task T-151 |
| Subscriber `ReputationRecalc` | ❌ não existe | Task BE-160 |
| Endpoint `/companies/{id}/reputation-breakdown` | ❌ não existe | Task — M2 |
| Endpoint `/companies/{id}/ranking-explain` | ❌ não existe | Task — M2 ADR-0023 compliance |
| Job noturno `reputation_recompute_job.go` | ❌ não existe | Task M2-M3 |
| Job noturno `reputation_decay_job.go` | ❌ não existe | M3 — Gap-SEC-008 |

**Interpretação**: os **inputs** da fórmula estão todos em coleta. A **função pura** e o **campo `reputation_tier` na `Company`** + **evento + subscriber + endpoints** são o corpo de trabalho de Sprint 11+. A spec (v2 destilado) está **100% fechada** (D-051). Implementação **0%** além de inputs.

---

## 18. Telas

**Reputação não tem telas dedicadas** — é metadado que aparece em superfícies de outros sub-produtos:

| Superfície | Tela onde aparece | Uso do tier |
|---|---|---|
| **Perfil público da Empresa** (sub-produto 2 Connect Me + 11 Administradoras Condominiais) | `E?` (a numerar em inventário de telas v2) | Badge grande de tier (ícone medalha + cor) + métricas resumo (N contratos, N avaliações, avg) + botão "Ver breakdown" |
| **Breakdown de reputação** | nova tela — espec. COM-027 | Componentes da fórmula + inputs agregados + histórico de transições (últimas 10 snapshots) |
| **Feed Connect Me** (busca de empresa) | E2–E6 (sub-produto 02) | Tier aparece como **filtro** ("Mostrar apenas Ouro+") + **ordenação** (default: relevância composto; alternativa: tier DESC) |
| **Votação de fornecedor** (sub-produto 05 Assembleia Live — Req 21) | telas de assembleia | Propostas exibem tier da empresa para informar conselho |
| **Perfil público do Síndico** (sub-produto 01) | tela de perfil | **Não** exibe Reputação da empresa — espaço é para reputação do síndico (M3+ remanescente do Manual §8) |
| **Dashboard Empresa** (sub-produto 11 internos — `Req 44` cross-domain.md) | dashboard own | Tier atual + tendência (subiu/desceu) + componentes com gaps ("1 condomínio para Ouro") |
| **Admin Master Síndico** (painel interno) | dashboard admin | Tier é **read-only** (GR-028 — 403 em PATCH direto); admin vê mas não altera. Pode aprovar/rejeitar moderação que afeta tier (vídeo, harassment) |

**Nota UI** (sub-produto 03-subprojects/web/design-system.md linha 225): tokens CSS para `reputation tiers` foram reservados (classes `.tier-bronze`, `.tier-prata`, `.tier-ouro`, `.tier-platina`, `.tier-diamante` com paleta metálica + ícone medalha Material Symbols).

---

## 19. Gaps / pendências

| ID | Tema | Status | Resolução prevista |
|---|---|---|---|
| **D-010 / P-BR-001** | Parametrização da fórmula: congelada em código vs configurável via banco | ✅ resolvido por D-051 — **congelada em M1** via ADR-0029 (a escrever). Mudança exige dual-check + pentester pass | M2 ADR-0029 publicada |
| **A-### (novo — a registrar)** | Nomes dos 5 critérios Req 26: divergência Obsidian Vault (Mar/2026) × código Go (Abr/2026) | ⚠️ identificado nesta destilação (§11.2) — código Go vence por hierarquia de fontes | Registrar como A-### em AUDIT + atualizar `Domínio - Comercial.md` no Obsidian Vault |
| **Drift OpenAPI** | `docs/openapi.yaml:116` lista 4 tiers `[bronze, prata, ouro, diamante]` (sem Platina) — stale | ⚠️ identificado | Corrigir em M2 quando expor endpoints públicos de reputação |
| **Implementação completa** | `ReputationCalculator`, enum `ReputacaoTier`, migração para adicionar `reputation_tier` + `suspended_until` em `companies`, evento `ReputationTierChanged`, subscribers, endpoints | ⏳ backlog Sprint 11+ (T-151..T-155, BE-101, BE-106, BE-110, BE-160) | M2 (entregável completo) |
| **P-EVT-006** | Recálculo: síncrono dentro UoW vs job async | ✅ resolvido — síncrono dentro UoW + job noturno complementar (D-051 não explicita, mas D-052 fixa TZ SP para jobs) | — |
| **P-SM-006** | mesmo tema: reactive vs batch | ✅ resolvido — ambos (reactive é prioritário; batch é salvaguarda) | — |
| **Reputação de síndico** (Manual Cap. 8 original) | Portado para empresa no v2; síndico não foi mapeado | ⏳ fora de escopo M1-M3. Avaliar M4+ | Decisão stakeholder |
| **Reputação de agência de marketing** | Não mapeado | ⏳ portfolio Produto 3 menciona "futuro síndicos profissionais e agências" | M4+ |
| **Decay temporal** (Gap-SEC-008) | Empresa "congelada" mantém tier | ⏳ M3 — `reputation_decay_job` + janela rolling | M3 |
| **Farm/ring detection** (Gap-SEC-010) | Operações coordenadas fraudulentas | ⏳ M4 — ML-based + human-in-the-loop | M4 |
| **`EvaluationFlag` enum** | Flags qualitativas opcionais em `CompanyEvaluation` | ⏳ congelado em M1 — não impacta fórmula principal | M3+ |
| **Endpoint explicabilidade** (LGPD Art. 20) | `/companies/{id}/ranking-explain` | ⏳ M2 junto com Connect Me ranking | M2 — ADR-0023 compliance |
| **Relação com Avaliação de Gestão** (Req 15) | Avaliação morador→síndico (institutional) é separada — confirmar que não interfere em reputação de empresa | ✅ confirmado: BCs separados (BR-032 escopo global vs tenant) | — |

---

## 20. Referências cruzadas

### Sub-produtos dependentes/consumidores

- [[01-governanca-institucional]] — fonte de `condos_distinct_count` (via `Condominium` aggregate); consumidor futuro em dashboards
- [[02-connect-me]] — consumidor: ordenação + filtro por tier; produtor: evento `ClosedDealSigned`
- [[04-conteudo-videos]] — produtor: evento `VideoApproved/Rejected/Removed`
- [[05-assembleia-live]] — consumidor: votação de fornecedor exibe tier das empresas em propostas
- [[09-compliance]] — consumidor: indicadores derivados no Governance Score
- [[10-marketing-agencias]] — produtor indireto (publica vídeos pela empresa — content_engagement)
- [[11-administradoras-condominiais]] — consumidor: tier de empresas contratadas

### Documentos técnicos canônicos v2

- [[../../STATE]] D-051, D-010 (resolvido), D-052 (TZ dos jobs), D-056 (agnosticismo provedor), D-058 (admin role privilegiada)
- [[../../AUDIT]] linha 169 (D-051 ✅), linha 220 (P-BR-001 🟢 resolvido)
- [[../../01-domain/aggregates/Company]] — campos, métodos, invariantes
- [[../../01-domain/aggregates/CompanyEvaluation]] — 5 critérios, anonimato
- [[../../01-domain/aggregates/ClosedDeal]] — dupla assinatura (input: contracts count)
- [[../../01-domain/business-rules]] BR-028..BR-032
- [[../../01-domain/invariants]] INV-036, INV-043, INV-044, INV-054, INV-055
- [[../../01-domain/state-machines]] §6 Company Reputation Tier + flag `suspended` paralela
- [[../../01-domain/domain-events]] E-040 `CompanyEvaluationSubmitted`, E-041 `ReputationTierChanged`
- [[../../01-domain/ubiquitous-language]] — `ReputacaoTier`, `ReputationCalculator`
- [[../../01-domain/bounded-contexts]] §4 commercial
- [[../../04-requirements/functional/commercial]] COM-025, COM-026, COM-027
- [[../../04-requirements/global]] GR-028
- [[../../04-requirements/registration-data]] §3.4 (campo `reputation_tier` derivado)
- [[../../02-architecture/adr/0023-recsys-deterministic-m1]] — determinístico, explicável, sem ML até M3+
- [[../../STATE#D-051|STATE D-051 (reputação fórmula determinística)]] (planejada) — congelamento da fórmula
- [[../../02-architecture/c4-components]] §3.4 commercial — `ReputationCalculator` como Domain Service
- [[../../02-architecture/patterns]] §9 Specification (`TopReputationSpec{MinTier: Ouro}`)
- [[../../08-security/threat-model]] §15 Gap-SEC-007, 008, 009, 010
- [[../../05-tasks/backlog]] T-151..T-155
- [[../../03-subprojects/backend/tasks]] BE-101, BE-106, BE-110, BE-160
- [[../../11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] §3.3 (derivação da fórmula)
- [[../../09-testing/test-traceability]] (testes a mapear: `T-COM-sm-rep-*`)
- [[../../07-quality/CLEAN_CODE]] (booleans `RequiresReputationTier`, etc)

### Fontes de material-fonte (hierarquia 7/8/9 do CLAUDE.md)

- `Development/backend/.kiro/specs/master-sindico/requirements/commercial.md` Req 18, 19, 22, 26
- `Development/backend/.kiro/specs/master-sindico/requirements/cross-domain.md` Req 37, 102
- `Development/backend/.kiro/specs/master-sindico/requirements/product-overview.md` linha 21 (Visão)
- `Development/backend/internal/modules/commercial/domain/entities/{company,company_evaluation,closed_deal}.go`
- `Development/backend/internal/modules/commercial/domain/enums/enums.go`
- `Development/backend/internal/modules/commercial/application/use_cases/{company_evaluation,closed_deal}_use_cases.go`
- `Development/backend/internal/shared/providers/database/migrations/00301_create_companies.sql`
- `Development/backend/internal/shared/providers/database/migrations/00308_create_company_evaluations.sql`
- `Development/backend/docs/openapi.yaml` linhas 116, 234 (drift a corrigir)
- `Development/Content/regras-de-negocio/regras-canonicas.md` §1 (4 pilares)
- `Development/Content/regras-de-negocio/quebras-detectadas.md` Q15 (gap histórico de spec)
- `Development/Content/_handoffs/2026-04-22-relatorio-agente3-analista.md` linha 157 (Sprint 10+ agendamento)
- `Development/full-stack/contextos/extracts/manual.txt` Capítulo 8 linhas 2685-2910 (Manual original — reputação do SÍNDICO, 4 tiers, história)
- `Development/full-stack/contextos/extracts/escopo.txt` linhas 128-147 (Escopo Técnico original — Status Bronze/Prata/Ouro/Diamante do Síndico)
- `Obsidian Vault/Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Comercial.md` (2026-03-10) — Req 26 com 5 critérios (divergência de nomes registrada §11.2)

### Regras arquiteturais aplicáveis v2 (D-056..D-068)

- **Planos globais Stripe-style** (ADR-0032 / D-067): eixo `trial/base/plus/pro` é **ortogonal** ao eixo reputação Bronze→Diamante. Empresa Pro pode ser Bronze; Empresa Plus pode ser Diamante.
- **Sem slugs**: nome do tier na UI é label visual; no storage é enum canônico `bronze/prata/ouro/platina/diamante` (lowercase, sem acento — consistente com Postgres enum).
- **Admin = `apps/admin` dedicada** (D-134 Fase H revoga D-058/D-072): admin **não** pode alterar tier direto (GR-028 — 403). Admin pode: aprovar/rejeitar vídeos, confirmar/rejeitar harassment reports, bloquear/desbloquear empresa — todas essas ações afetam tier **indiretamente via inputs**. Acesso via `admin.mastersindico.com.br` com Cloudflare Access SSO+MFA, NÃO mais embutido em `cms/`.
- **Agência de marketing é sub-produto interno próprio** (sub-produto 10): agência gerencia conteúdo pela empresa mas não afeta diretamente tier — apenas publica vídeos que passam por moderação e entram em `content_engagement`.
- **Agnosticismo de provedor** (D-056): não há provedor externo na Reputação (nada é terceirizado) — é lógica de domínio pura 100% em Go. NATS é transporte de evento, mas a função é pura.

---

## Assinatura

**Destilador**: agent dedicado Fase 8 · sub-produto Reputação  
**Método**: leitura arquivo-por-arquivo, catálogo fiel com citação de fonte+linha+data  
**Dual-check obrigatório**: agente B em contexto limpo antes de marcar estável  
**Próximo passo**: levantar A-### em `AUDIT.md` para drift de nomes dos 5 critérios (§11.2) + registrar no backlog correção do OpenAPI drift (§19)
