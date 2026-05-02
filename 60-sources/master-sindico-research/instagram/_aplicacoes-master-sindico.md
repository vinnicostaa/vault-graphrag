---
title: Aplicações Instagram/Meta aos feeds sociais do Master Síndico
type: note
tags: [research, instagram, meta, aplicacoes, master-sindico, v2, feed-ranking, tao, stories, moderation, fanout, consistency]
source: 13-research/instagram/_destilado.md (268 linhas) + cruzamento 7 sub-produtos + invariants + patterns + STATE
created: 2026-04-23
updated: 2026-04-23
aliases: [instagram-aplicacoes, meta-aplicacoes-master-sindico]
---

# Aplicações Instagram/Meta aos feeds sociais do Master Síndico

> Cruzamento nominativo do research [[_destilado]] com os **cinco feeds sociais** internos do Master Síndico v2: **Timeline condomínio** (BC `institutional`), **Connect Me** (marketplace unidirecional `commercial`), **Banco de Talentos** (`content`+`commercial`), **Vizinhança** (`commercial` hyperlocal com cupons), **Conteúdo/Vídeos** (`content` institucional). Cada feed tem natureza própria (governança imutável vs marketplace efêmero vs hyperlocal com lock 4h); os padrões Meta **não se aplicam uniformemente**.
>
> Escopo temporal: decisões tomáveis **agora** (M1 2026-05-08) vs reserváveis pra M2/M3+. Referência Meta sempre em EN (TAO, Akkio, MTML, ig2vec, Sigma, Haxl); discussão em PT-BR.

---

## 1. Feed ranking (Explore/Reels) × feeds do Master Síndico

Meta opera funil 4-stage (retrieval → first-stage → MTML second-stage → final rerank) com 1000+ modelos em produção ([[_destilado#1-feed-ranking-explore-reels-aplicável-sim-com-ressalvas]]). O `_destilado` já consolidou aplicabilidade pra **Conteúdo/Vídeos** e **Connect Me**. Aqui complemento os três feeds restantes:

### A) Timeline condomínio — **NÃO adotar ranking ML; manter cronológico**

**Decisão M1**: ordenação estritamente cronológica (append-only, INV-024 imutável — ver [[../../01-domain/invariants#inv-024]]).

Rationale:

- A **invariante timeline imutável** ([[../../01-domain/invariants#inv-024]]) é o valor central do produto (memória institucional auditável por anos — LGPD, disputas, prestação de contas). Reordenar por "relevância" quebra o contrato com o síndico: "aparece primeiro" passa a significar "mais engajado", não "mais recente". Síndico perde rastreabilidade temporal.
- **Escala**: 1 condomínio ~500 moradores, ~10-50 entries/mês. Não há problema de over-fetching que ranking ML resolveria.
- **Governança vs descoberta**: Timeline é registro, não feed social. Usuário abre pra "ver o que foi publicado esta semana", não pra "descobrir conteúdo relevante".
- **Custo infra**: MTML + two-tower NN + ig2vec embeddings pra 50 entries/mês é 4 ordens de magnitude de overkill.

**O que pode entrar em M2 (sem violar a invariante)**:

- **Filtro/facet (não ranking)**: tabs "tudo | meus comentários | minha unidade | urgente" (INV-025 Announcement priority). Ordem dentro de cada tab continua cronológica.
- **Pin de prioridade** (Announcement `priority=urgente`) empurrado pro topo **sem** esconder demais: conceito de "sticky" do Reddit, não de ranking do Instagram. Decisão do síndico (publicador), não do algoritmo.
- **Pré-fetch preditivo mobile** da §2 do destilado — UX/perf, não ranking.

**Não fazer em M1/M2**: personalização por "últimos comentários do user + vínculo moradia + papel". Esse tipo de sinal é ranking implícito; quebra previsibilidade do registro institucional.

**Decisão sugerida** (candidata a D-### / ADR): "Timeline condomínio ordenada estritamente por `created_at DESC` + filtros por faceta. Qualquer reordenação por relevância exige ADR explícita revisitando INV-024."

### B) Vizinhança — **ranking leve determinístico já especificado**

Vizinhança já define ordenação composta de 4 níveis em [[../../00-product/sub-produtos/08-vizinhanca#12-3-priorização-do-feed-vm1]]:

1. `is_sindico_approved=true` + condomínio do morador → topo.
2. `has_active_promotion=true` + dentro de 2km.
3. Dentro de 2km sem promoção.
4. Mesmo bairro fora do raio estrito.

**Paralelo Meta**: equivale ao **retrieval** (ST_DWithin + filtros) + **rerank determinístico** por sinais (seal + active promo). Sem ML.

**M1 é isso.** Correto. Evolução M3+ (só se escala justificar):

- Adicionar sinais negativos (seguindo §4 do destilado — aversão): morador "escondeu parceiro X" → penaliza no score daquele condomínio. Simétrico ao `see less` do Instagram.
- Coletar CTR (card-view → clique-promoção → ativação) desde D1 pra ter labels implícitos; reusa em ML M3+ se migrar.

### C) Banco de Talentos — **ranking de busca, não feed**

Banco de Talentos não é feed — é **busca assimétrica** ([[../../00-product/sub-produtos/07-banco-de-talentos#9-empresa-view-busca-favoritos-tags-raio-geográfico]]). Empresa Plus/Pro filtra por `TalentArea[] + CEP + raio + modalidade`; resultado é lista paginada.

Paralelo correto não é Explore, é **search engine**:

- **M1**: Postgres `tsvector` + filtros estruturados (D-067). Sem ranking além de `rank(tsvector_query)`.
- **M2 (deterministic ranking)**: score composto `0.4*match_area + 0.3*distance_score + 0.2*nr_match + 0.1*recency`. Valores configuráveis via ADR (igual Connect Me §1B do destilado).
- **M3+**: reavaliar se volume justificar two-tower (empresa-embedding × candidate-embedding). **Não** prioritário.

**Auto-tags Pro (7 categorias calculadas)** são features derivadas determinísticas ("alta-disponibilidade", "NR-completo"). **Não** são ML — cálculo SQL sobre os campos do currículo. Correto mantê-lo assim em M1-M2.

### D) Connect Me — **já coberto no destilado §1B** (value model somatório, sem ML M1)

Sem complemento adicional. Manter o que está.

### E) Conteúdo/Vídeos — **já coberto no destilado §1A** (separar retrieval de ranking, sem ML M1)

Sem complemento adicional.

---

## 2. TAO social graph × modelagem relacional do Master Síndico

TAO modela **objects tipados + associations tipadas** com two-tier cache por região, write-through, 1B+ reads/s ([[_destilado#3-tao-the-associations-and-objects-patterns-aplicáveis-no-cache-abac-e-feed]]).

### Mapping Master Síndico (já presente, sem nome próprio)

Nosso grafo social implícito existe, mas **não** como grafo genérico — são **aggregates DDD com associações tipadas no schema**:

| Meta (TAO) | Master Síndico | Aggregate / tabela |
|---|---|---|
| follow (user→user) | morador ↔ condomínio | `Membership(user_id, unit_id, status)` — [[../../01-domain/invariants#inv-026]] |
| like (user→post) | morador → announcement `read_receipt` | `announcement_reads(user_id, announcement_id, read_at)` |
| like (user→photo) | morador → vídeo empresa `view` | `video_views(user_id, video_id, ts)` |
| comment (user→post) | morador → timeline_entry (NÃO existe — timeline é imutável) | — |
| endorsement (LinkedIn) | síndico → parceiro seal | `Seal(partner_id, condominium_id, granted_by_syndic_id)` — [[../../00-product/sub-produtos/08-vizinhanca#8-5-seal-aggregate--síndico-aprovado]] |
| relationship | síndico → condomínio (mandate) | `SyndicMandate(condominium_id, syndic_id, end_date)` — [[../../01-domain/invariants#inv-027]] |
| relationship | empresa → empresa (B2B) | `ConnectMeRequest(sender_company_id, receiver_company_id, flow=enterprise_to_enterprise)` |
| relationship | empresa → administradora | `EmpresaSindicaLink` (código vivo `commercial/entities/`) |
| multi-identity | síndico multi-cond (até 15) | `SyndicMandate[]` por `syndic_user_id` — [[../../01-domain/invariants#inv-027]] |

**Conclusão**: **o data model atual cobre**. Não precisa introduzir abstração "social graph genérico". Cada relação é um aggregate com invariantes específicos (1 titular + ≤2 dependentes por unit, ≤15 condomínios/síndico, seal 1-por-par, etc.). Meta precisou de TAO porque tinha bilhões de nós heterogêneos com queries arbitrárias; nós temos dezenas de tipos de relação bem conhecidos.

### Patterns TAO aplicáveis

**Já no destilado §3A** (ABAC cache write-through) **e §3B** (feed cache write-behind) — aplicar como está.

**Adicional nominativo** (reforço de patterns já latentes em [[../../02-architecture/patterns]]):

- **Event outbox como "leader tier"**: o outbox pattern (§14 do patterns.md) faz semanticamente o que TAO faz entre leader e follower — garante que a mudança no aggregate propaga pra consumidores (NATS JetStream: search index, notificações, analytics) **dentro da mesma tx**. É TAO light.
- **Shard-by-tenant**: INV-089 (tenant isolation) já garante que `tenant_id` está em toda query. Quando/se multi-region vier (§5 destilado), migração μshard-Akkio é mecânica: um tenant = uma μshard.

---

## 3. Consistency model × timeline imutável (append-only)

Meta usa eventual consistency cross-region (lag <1s) pra feeds; read-after-write dentro de um tier ([[_destilado#3-tao-the-associations-and-objects-patterns-aplicáveis-no-cache-abac-e-feed]]).

### Nossos feeds são mais fortes (append-only + imutável) — **trade-off explícito**

| Feed | Modelo de consistência | Fundamento |
|---|---|---|
| **Timeline condomínio** | **Strong + imutável** (INV-024, INV-025, INV-032, INV-050) | Auditoria LGPD/civil; correção via nova entry com `corrects_entry_id`, não UPDATE |
| **Vizinhança cupons** | **Strong no emit** (índice parcial UNIQUE `WHERE status=active` + Redis lock 4h) | Anti-spam duro; lock 4h é invariante ([[../../00-product/sub-produtos/08-vizinhanca#10-2-trava-temporal-lock-4h]]) |
| **Vizinhança benefício escopo** | **Strong** (INV-093 equivalente — `scope` imutável após publicação) | Morador não pode ver "promoção mudou de exclusiva pra aberta" retroativamente |
| **Coupon redemptions** | **Append-only** (sem UPDATE/DELETE) | LGPD audit trail |
| **Connect Me request** | **State machine determinística** (pending→accepted/declined/expired) | Auditoria de revelação de dados ([[../../01-domain/invariants#inv-048]]) |
| **Banco Talentos search logs** | **Append-only** (INV-BT-07) | LGPD — morador pede relatório de quem viu |
| **Vídeos (likes, views)** | **Eventual** (write-behind tolerável) | Métrica aproximada; igual §3B do destilado |
| **Reputation scores** | **Eventual** (recálculo noturno batch) | Igual §3B |

### Trade-off a documentar (candidato a ADR)

- **Custo**: writes mais caros (write-through, UoW com FOR UPDATE em quotas, UNIQUE constraints impedindo conflation). Impossibilidade de "undo silencioso" — correção é explícita (adendo/amendment, INV-050).
- **Benefício**: confiabilidade jurídica (LGPD, Código Civil, disputas em assembleia), ausência de race condition em invariantes críticas (vote-único INV-071, evaluation-única INV-043), rastreabilidade forense (pentest §2 do `00-product/sub-produtos`).
- **Escala esperada**: não somos Instagram. 500 moradores × 1000 condomínios × 50 entries/mês = ~25M rows/ano na timeline. Postgres single-region aguenta com folga.

**Decisão sugerida** (candidata a ADR): "Feeds de governança/auditoria/anti-fraude (Timeline, Announcement, Cupom, Connect Me request, CompanyEvaluation, audit_trail, lgpd_logs, curriculum_search_logs) usam **strong consistency + append-only**. Feeds de UX/métricas (video views, likes, reputation score) usam **eventual consistency write-behind**. Divergência do modelo Meta é **intencional** e documentada."

---

## 4. Stories efêmero × "Palavra-chave do dia" Vizinhança

Stories Meta: 24h TTL, CDN agressivo, pré-fetch preditivo ([[_destilado#2-stories-ephemeral-24h-não-aplicar]]). O destilado já decidiu **rejeitar o padrão ephemeral** pra governança.

### Palavra-chave do dia (Req 100.2) — paralelo legítimo

A feature `palavra do dia` da Vizinhança ([[../../00-product/sub-produtos/08-vizinhanca#10-3-palavra-chave-do-dia-gamification-tier-pro]]) tem dinâmica parecida com Stories:

- **Rotação diária obrigatória** (1 palavra por parceiro por dia).
- **TTL natural** (à meia-noite vira outra).
- **Acesso gated** (morador digita palavra → desbloqueia promoção).

**O que herdar do Stories sem copiar**:

- **Lifecycle management separado** (bounded context diferente do Coupon padrão): tabela `daily_keywords(partner_id, day, palavra)` com PK `(partner_id, day)` — similar ao lifecycle separado do `story_highlights` no Meta.
- **Rate-limit por morador** (5 tentativas/dia em qualquer parceiro) — previne brute-force, equivalente aos anti-abuse signals do Sigma (§8 destilado).
- **Rotação forçada** — parceiro não pode "fixar" a palavra (diferente do Highlights). UPSERT by `(partner_id, day)` naturalmente invalida o anterior.
- **Audit trail das tentativas** (`coupon.daily_keyword.guessed` event) — mantém rastreabilidade (vs Stories que apaga).

**O que NÃO copiar**:

- CDN agressivo + pré-fetch preditivo da tray — não há "tray" na Vizinhança; feed VM1 já resolve descoberta.
- "Efêmero" no sentido LGPD — evento do guess fica em log permanente (apagar violaria trilha de auditoria).

---

## 5. Django monolith lessons × Go modular monolith + NATS

Instagram é Django milhões LOC, centenas devs, ~100 deploys/dia, mantido via codemods (LibCST) em vez de quebrar em microservices ([[_destilado#9-django-monolito-lições-de-scaling-de-source-05-e-contexto]]).

### Master Síndico: já melhor posicionado

Stack atual (via [[../../02-architecture/patterns]] + código vivo):

- **Go modular monolith** com bounded contexts explícitos (`identity | billing | institutional | commercial | content | assembly | cross-domain`).
- **NATS JetStream** para eventos entre BCs (Outbox pattern §14 patterns.md).
- **Clean Architecture estrita** (INV-086 ABAC sequencial; domain zero import de framework — §3 CLAUDE).
- **sqlc** para type-safe SQL (não ORM mágico).

**Vantagens vs Django monolito**:

- Tipos estáticos Go evitam classe inteira de bugs que Instagram corrige com LibCST codemods.
- Modularidade DDD (§1 patterns.md) torna split futuro em serviços trivial — fronteiras já existem como packages.
- NATS outbox já faz "async side-effects decoupling" que Django+Celery faz pior.

### O que herdar do caso Instagram

1. **Não quebrar em microservices prematuramente** (§14 YAGNI CLAUDE.md + §9 destilado). Monolito modular Go escala pra 100k+ condomínios antes de virar problema. Já é o plano (ADR-0001 Clean Arch + DDD).
2. **CI rápido + rollback instantâneo desde M1** (mesmo com 1 deploy/dia planejado). Pipeline canônico precisa: testes determinísticos, blue/green ou canary via Railway/ECS, DB migrations reversíveis. Registrar como requisito de `06-execution/ci-cd.md` (não criar agora, só bookmark).
3. **Automação de refactoring em escala**: `gofmt`, `golangci-lint`, `gopls`, `go/ast`-based scripts. Equivalente Go do LibCST. Em M1 basta `golangci-lint` strict; scripts AST quando > 100k LOC.
4. **Model Registry central** (§1 destilado, Meta Configerator): **não aplicável em M1-M2** (não temos ML). Reavaliar em M3+ quando Connect Me/Talentos ranking virar ML — antes disso, "registry" é um arquivo de config com pesos por BC.

---

## 6. Pull vs Push fanout × Timeline condomínio (500 moradores)

Meta combina estratégias (§3 destilado menciona write-behind + conflation pra feed). Pergunta concreta: timeline com 500 moradores — **pull-on-read** ou **push-on-write**?

### Análise Master Síndico

**Variáveis de dimensionamento**:

- 500 moradores/condo × 1000 condos M2 = 500k usuários ativos teóricos.
- 10-50 entries/mês/condo = write rate baixíssimo (~0.02 entries/s em pico agregado).
- Read rate: morador abre app 1-5x/dia (~30k-150k reads/dia; trivial).

**Decisão M1**: **pull-on-read**.

Rationale:

- Write rate é **3 ordens de magnitude** menor que read rate — push-on-write pré-computa feed de 500 moradores pra entrada que talvez 50 leiam. Desperdício.
- Escopo do feed é **determinístico por tenant** (um morador = seu condomínio). Query `SELECT FROM timeline_entries WHERE condominium_id = :id ORDER BY created_at DESC LIMIT 50` é trivial.
- Read cache Redis (TTL 60s) no resultado paginado já absorve 90% do tráfego.

**Quando reconsiderar push (M3+)**:

- Se algum usuário for membership em >10 condomínios (administradora via `EmpresaSindicaLink`) **e** quiser "feed agregado cross-condomínio". Aí pull-on-read faz N queries — push pré-computa o merge. Instagram tem isso pra feed de 500 follows.
- Se timeline virar feed personalizado (reordenação por relevância) — mas isso é rejeitado em §1A acima.

**Híbrido pragmático** (se surgir caso multi-cond): **fan-out on write** para usuários com >10 vínculos; **fan-out on read** pro resto. Paralelo exato com estratégia "celebrities vs normal users" do Twitter/Meta.

---

## 7. Content moderation em escala × Cupons e conteúdo Vizinhança

Meta: pipeline híbrido ML (CNN, transformer NLP, Rosetta OCR, detecção objeto) + threshold + fila humana + Sigma anti-abuse Haskell ([[_destilado#4-moderação-de-conteúdo-processo-manual--ml-cronograma-m3]]). Destilado cronograma: M1 manual → M2 filtros estáticos → M3+ API externa → M4+ ML próprio.

### Foco específico: cupons e promoções Vizinhança (moderar promoção enganosa)

Risco real em produção: parceiro publica "50% desconto em qualquer produto" mas na prática aplica só em um item.

**Moderação M1** (manual + estrutural — não precisa ML):

1. **Termos estruturados obrigatórios** no campo `NeighborhoodBenefit.termos`: limite "válido de/até", quantidade mínima, exclusões. Template forçado em UI, não free-text livre.
2. **Botão "Reportar cupom enganoso"** em VM3 (detalhe promoção) — motivo tipado (`promocao_enganosa | vencido | estabelecimento_recusou | outro`). Fila em `coupon_reports` (análoga a `harassment_reports` vivo).
3. **Admin modera** (D-058 role transversal) — suspende parceiro ou força-expirar cupom (INV equivalente ao §10.4 do 08-vizinhanca).
4. **Sinal negativo persistente**: parceiro com N reports confirmados em janela 30d → bloqueia novas promoções até revisão. Equivalente `see_less` weight do Meta (§1 destilado).

**M2** (filtros estáticos, sem ML):

- **Blacklist de termos**: "grátis", "somente hoje com qualquer compra", "desconto em tudo" — flag para revisão admin antes de publicar. Heurística dumb, funciona.
- **Inconsistência de valor**: se `tipo=desconto_percentual` e `descricao` contém número > 100 → flag.
- **Regex de CPF/CNPJ/telefone vazados** (igual M2 do destilado) aplicável a `descricao` tanto em cupom quanto em curriculum do Banco Talentos quanto em comentários Connect Me.

**M3+ (API externa antes de modelo próprio)**:

- Perspective API ou Azure Content Moderator pra detecção de linguagem ofensiva / promessas irreais em PT-BR. Custo previsível.
- **Não** treinar modelo próprio até volume + labels humanos justificarem.

### Para os outros feeds

- **Timeline condomínio**: síndico autopublica — moderação cruzada pelo admin MS (D-058) apenas em denúncia. Não há volume que justifique ML em M2.
- **Banco Talentos**: vídeo-currículo 90s + texto. M1 manual (admin review em BT11 approval). M3+ avaliar Rekognition pra detectar imagens impróprias no vídeo.
- **Connect Me**: já tem `HarassmentReport` vivo com status `pending | warning | suspension | ban` — pattern correto, não precisa de ML em M1.
- **Conteúdo/Vídeos**: pre-publish moderation mandatory (INV-058 state machine `moderation → approved → published`). Pattern já correto.

---

## Top 5 aplicações práticas — M1 2026-05-08 (≤ 400 palavras)

**1. Manter Timeline cronológica estrita (não adotar ranking Meta).** A invariante [[../../01-domain/invariants#inv-024]] (imutabilidade) é valor central. Reordenação por "relevância" quebra o contrato com o síndico — vira feed social, deixa de ser registro institucional auditável. M2 pode adicionar filtros por faceta e pin de `priority=urgente`, **sem** ranking. Decisão pro STATE: "Timeline ordenada estritamente por `created_at DESC` + filtros; qualquer reordenação exige ADR revisitando INV-024".

**2. Documentar explicitamente o trade-off strong-vs-eventual consistency.** Nossos feeds divergem de Meta por design: Timeline/Cupom/Connect Me/search logs são **append-only com strong consistency** (LGPD, disputa civil, anti-fraude); views/likes/reputation-scores são eventual write-behind. Isso custa writes mais caros mas ganha rastreabilidade jurídica. Candidato a ADR nominativa "Consistency model por feed" em `02-architecture/adr/`.

**3. Confirmar data model cobre o "social graph" sem abstração TAO genérica.** Mapping §2 acima mostra que `Membership`, `SyndicMandate`, `Seal`, `EmpresaSindicaLink`, `ConnectMeRequest` já modelam todas as relações relevantes com invariantes específicos. **Não** criar camada `SocialGraph` genérica — seria abstração prematura (§14 YAGNI). Outbox pattern (§14 patterns.md) já resolve propagação de mudanças (equivalente light ao "leader tier" do TAO).

**4. Pull-on-read para Timeline (500 moradores).** Write rate 0.02 entries/s vs read rate 30k-150k/dia. Push-on-write desperdiça em pré-computação. `SELECT WHERE condominium_id ORDER BY created_at DESC LIMIT 50` + Redis read-cache 60s resolve M1-M2. Híbrido push-for-multi-cond só se administradora com >10 vínculos vier em M3+.

**5. Moderação M1 pragmática (manual + estrutural), não ML.** Cupons Vizinhança viram risco concreto de promessa enganosa: obrigar `termos` estruturado em UI (não free-text), botão "reportar" com motivo tipado, fila `coupon_reports`, admin MS decide (D-058). Mesmo padrão pra Banco de Talentos (admin aprova em BT11), Connect Me (`HarassmentReport` já vivo). ML entra em M3+ via API externa (Perspective/Rekognition) antes de modelo próprio.

---

## Referências

- Destilado original: [[_destilado]] (268 linhas, 10 fontes Meta Engineering)
- Fontes primárias: `source-01.md` .. `source-11.md`
- Sub-produtos cruzados: [[../../00-product/sub-produtos/01-governanca-institucional]], [[../../00-product/sub-produtos/02-connect-me]], [[../../00-product/sub-produtos/04-conteudo-videos]], [[../../00-product/sub-produtos/07-banco-de-talentos]], [[../../00-product/sub-produtos/08-vizinhanca]]
- Invariantes: [[../../01-domain/invariants]] (INV-024, INV-025, INV-026, INV-027, INV-032, INV-043, INV-048, INV-050, INV-058, INV-071, INV-086, INV-089, INV-093)
- Patterns: [[../../02-architecture/patterns]] (§1 DDD, §14 Outbox, §15 Observer)
- Decisões vivas: [[../../STATE]] (D-056 agnosticismo, D-058 admin transversal, D-064 cupons ativos, D-066/067 planos globais)
- Research vizinhos: [[../linkedin/_destilado]] (reputação Connect Me), [[../tiktok/_destilado]] (feed vídeos + cascade moderation), [[../beyond-corp/_destilado]] (ABAC reforço §3A), [[../google-arch/_destilado]] (Spanner alternativa a TAO)
