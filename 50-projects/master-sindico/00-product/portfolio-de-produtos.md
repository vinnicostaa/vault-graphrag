---
title: Portfolio de Produtos — Master Síndico (consolidado Fase 8)
type: note
tags: [product, portfolio, sub-products, master-sindico, v2, fase-8, consolidado]
source: ADR-0032 + STATE.md D-056..D-069 Fase 8 + 00-product/sub-produtos/01..11 (fonte autoritativa) + backend/.kiro/specs/master-sindico/requirements/*
created: 2026-04-23
updated: 2026-04-23
status: consolidado-fase-8
dual_check: pending
aliases: [portfolio-fase-8, sub-produtos-master-sindico]
---

# Portfolio de Produtos — Master Síndico

Catálogo dos **11 sub-produtos integrados** que compõem a plataforma Master Síndico. Este documento substitui integralmente a versão Fase 3 (que mencionava 7 sub-produtos com slugs compostos `syndic_n1/n2/n3`, `enterprise_plus/pro`, `resident_base/paid`, `local_company_standard`, `marketing_standard`). Todos esses slugs foram **abolidos** pela ADR-0032 / D-057 Fase 8 / D-066 Fase 7 e pelo D-067 Fase 7.

**Princípio estruturante**: cada sub-produto tem **bounded context(s) atribuído(s)** (D-003 legado); multi-produto explícito é invariante. Planos globais Stripe-style (ADR-0032) — personas carregam matriz de permissão.

**Fonte autoritativa de coerência**: [[../00-product/sub-produtos/_moc]] + os 11 arquivos em `00-product/sub-produtos/01..11`. Este documento **não reescreve** conteúdo dos sub-produtos — referencia e consolida a visão de portfolio.

---

## 0. Regras arquiteturais transversais (herdadas — não repetir em cada sub-produto)

- **Planos globais**: `plan_tier ∈ {trial, base, plus, pro}` universal para TODAS as personas (ADR-0032).
- **Roles Zitadel**: `syndic | resident | enterprise | marketing | local_company | admin`.
- **Admin = `apps/admin` dedicada** no monorepo web com Cloudflare Access SSO+MFA (D-134 Fase H revoga D-058/D-072 que diziam admin transversal embutido em CMS).
- **Agência Marketing = sub-produto interno próprio** com login Zitadel + delegação ABAC (D-059 Fase 7 / D-060 Fase 8).
- **Agnosticismo de provedor** (D-056/D-071 — INVARIANTE): Railway + Cloudflare edge (D-117) + Zitadel + Resend + Mux + Livekit + MinIO + Twilio (prod) ou Noop (dev — D-118) + PostgreSQL 18 + Redis + NATS, todos atrás de interfaces canônicas.
- **Morador Base Connect Me = 2/ano** (D-094 Fase 11 revoga D-058/D-079 que ratificavam 0/ano).
- **Sem N1/N2/N3** (D-067 Fase 7) — quando citação legada surgir, traduzir explicitamente.
- **Invariantes globais (GR-###)**: GR-028 (reputação imutável via PATCH), GR-029 (lock 90d vídeo), GR-019 (presigned URL Mux — backend nunca faz proxy de bytes).

---

## Produto 1 — Governança Institucional

### Descrição

Núcleo do MVP. Coração da plataforma onde **tudo vira histórico**. Cadastro do **Condomínio**, **Unidades** com fração ideal (DECIMAL 4 casas, Σ ≤ 100%), **Memberships** (User↔Condomínio↔Unit+Role), **Timeline imutável** (INSERT-only, sem UPDATE/DELETE), **Plano Diretor** plurianual 3-5 anos **reativo** (atualiza exclusivamente por reação a entradas da Timeline — Req 12 AC8), **Comunicados oficiais** imutáveis pós-publish (8 tipos × 4 prioridades), **Eventos** (13 tipos), **Enquetes internas** (3 tipos), **Validações centralizadas** (hub), **Avaliação de gestão** bimestral 1-10.

Todo o resto do ecossistema pluga aqui: Connect Me publica via validação → Timeline; Assembleia persiste ata como TimelineEntry; Compliance bloqueia encerramento de mandato; Billing controla criação de condomínio; Identity gera Memberships; Content hospeda vídeos que o síndico cola no perfil institucional.

### Personas-alvo

- **Síndico** (criador, editor, decisor principal) — role `syndic`, sub-tipos `principal | auxiliary | assistant`.
- **Subsíndico / Conselho** (editores delegados em escopo limitado) — via scoped permissions ABAC.
- **Morador** (consumidor: Timeline, ata, comunicados; responde enquete, avalia gestão) — sub-tipos `titular | dependent`.
- **Administradora Condominial** (ator delegado via `EmpresaSindicaLink` — ver Produto 11).
- **Admin Master Síndico** (audit trail completo, moderação, broadcasts).

### Tiers aplicáveis

Síndico `trial / base / plus / pro` — hard limit **15 condomínios/síndico**. Morador `base / plus` (pagante). Empresa/Parceiro não operam neste sub-produto diretamente.

### Bounded Context

- **institutional** (principal)
- Cross: **identity** (user/session/ABAC), **commercial** (deals confirmed → TimelineEntry), **assembly** (assembly_result → TimelineEntry), **billing** (quota condomínios).

### Invariantes

- Σ fração ideal das unidades ≤ 100% (CHECK DB + PBT).
- Timeline: INSERT-only, sem UPDATE/DELETE/`deleted_at`.
- Membership ativa única por `(user, condominium)` em dado momento.
- Announcement imutável pós-publish (correção → adendo via Req 25).
- `uq_syndic_mandates_active_principal` — 1 principal ativo por condomínio (migration 00201:72-74).
- Plano Diretor atualiza exclusivamente por reação a TimelineEntry (Req 12 AC8).
- Dependente não vota em assembleia (Req 9) — revogação de titular cascateia bloqueio dos dependentes (Req 17).

### Marco

**M1 (2026-05-08)** — base contratual; Poll + Announcement completo em M2; Event + compliance em M3.

### Diferenciais

- **Timeline como prova documental** — ERPs concorrentes (Superlógica, Igora) armazenam registros em silos financeiros; Master Síndico unifica decisão-acontecimento-documento numa sequência imutável com valor jurídico.
- **Plano Diretor reativo como produto** — template + acompanhamento de execução (% completo), não "um PDF qualquer".
- **Score de Governança único** (D-066 Fase 8) — 0-100, 10 componentes, job noturno, histórico 12 meses, alerta < 60.

### Dependências técnicas

- PostgreSQL 18 (timeline append-only, frações ideais DECIMAL(4,2), particionamento futuro 100 condomínios+).
- **Busca full-text via `ISearchProvider` real** (OpenSearch ou Meilisearch — D-120 Fase 14 revoga parcialmente D-067; ADR-0042 `proposed pending dual-check`). PG tsvector descartado como solução M1.
- NATS JetStream (events cross-domain).

### Detalhamento completo

[[../00-product/sub-produtos/01-governanca-institucional]]

---

## Produto 2 — Connect Me (marketplace formal unidirecional + Marketing Link)

### Descrição

Marketplace **unidirecional de contratação formal** (não é chat, não salva histórico de conversas, não tem reply thread — Regra de Ouro 3). Formulário estruturado que registra interesse entre personas, **preservando privacidade do emissor até o receptor demonstrar interesse explícito** (LGPD by design).

**4 direções canônicas** (`ConnectMeFlow` enum Go):

1. **Síndico → Empresa** (`ConnectMeFlowSyndicToEnterprise`) — RFP (Request for Proposal) estruturado: categoria, descrição problema, orçamento teto, prazo, entregáveis, anexos. Empresas qualificadas recebem; enviam proposta estruturada; síndico escolhe; vira contrato com milestones + dupla assinatura.
2. **Morador → Síndico** (`ConnectMeFlowResidentToSyndicCold`) — morador solicita algo formal ao síndico (dúvida de gestão, solicitação de manutenção, reclamação, sugestão, outro). Frio (sem histórico). **Morador Base = 2/ano** (D-094 Fase 11 revoga D-058/D-079 que diziam 0/ano). Morador Pagante = 4/ano.
3. **Empresa → Empresa** (`ConnectMeFlowEnterpriseToEnterprise`) — networking B2B entre empresas condominiais (subcontratação, consórcio, parceria técnica). Só `plus+` pode iniciar.
4. **Síndico → Parceiro Vizinhança** (`ConnectMeFlowSyndicToLocalCompany`) — síndico convida comércio local para parceria/benefício. Acordo leve (não contrato).

**Direções proibidas** (anti-prospecção): Empresa→Morador (anti-spam), Empresa→Síndico ativo (empresa só RECEBE de síndico), Agência→Qualquer, Parceiro→Qualquer (parceiro só recebe).

**Marketing Link** (feature DISTINTA de Connect Me — `E16→MK8`): formulário Empresa→Agência de Marketing registrando intenção a partir de tags em vídeos institucionais. Sem cota. Não revela contato de síndico. Tabela separada `marketing_link_requests`.

**Anti-assédio**: botão "Denunciar" em todo Connect Me → `HarassmentReport` → admin revisa → ações `warning / suspension / ban` (enum `HarassmentAdminAction`).

### Personas-alvo

- **Síndico** (abre 3 direções: →Empresa, →Parceiro; escolhe propostas).
- **Empresa `plus/pro`** (recebe/responde; `plus+` inicia E→E).
- **Morador pagante** (abre Morador→Síndico).
- **Parceiro Vizinhança** (recebe Síndico→Parceiro).
- **Admin** (revisa denúncias de assédio — não envia nem recebe Connect Me).
- **Agência de Marketing** NÃO envia nem recebe Connect Me; recebe Marketing Link.

### Tiers aplicáveis

| Persona | trial | base | plus | pro |
|---|---|---|---|---|
| syndic | 0 | 2/ano | 4/ano | ilimitado |
| resident | n/a | **0/ano** | 4/ano | 4/ano |
| enterprise | 0 | n/a | ✅ E→E | ilimitado |
| local_company | 0 | — (recebe) | — (recebe) | — (recebe) |

### Bounded Context

- **commercial** (principal)
- Cross: publica eventos que **institutional** consome (TimelineEntry), **identity** (ABAC quotas), **content** (Marketing Link vincula a vídeos).

### Invariantes

- Proposta única por `(ConnectMeRequest, empresa)` — UNIQUE DB.
- RFP não editável pós-"publicado" (edição cria nova versão).
- Quota decrementa no ato de publicar RFP, não no rascunho (antipattern A8 evitado — `SELECT ... FOR UPDATE`).
- `SupplierVoteCast` único por `(session, voter)` — A-026 legado.
- Dupla assinatura de ClosedDeal: `companySign → syndicSign`; `isImmutable=true` após assinatura do síndico.
- Auto-publica TimelineEntry após deal fechado.

### Marco

**M2 (2026-06-20)** — 4 fluxos + anti-assédio + LGPD logs. M3: marketplace ranking via Reputação (ADR-0023).

### Diferenciais

- **Unidirecional evita "ruído de WhatsApp"** — decisão tem trilha clara auditável.
- **Auditabilidade total** na Timeline — prova contra contestações (ERPs registram contrato; Master Síndico registra processo completo de escolha).
- **LGPD by design** — PII do emissor escondida até "Tenho Interesse" explícito do receptor; toda revelação gravada em `lgpd_logs`.
- **Reputação alimentada automaticamente** — avaliação pós-contrato entra na fórmula Bronze→Diamante.

### Dependências técnicas

- PostgreSQL 18 (`connect_me_requests`, `proposals`, `closed_deals`, `harassment_reports`, `supplier_vote_sessions`, `feature_usages`).
- Redis (cache quotas `ms:quota:{userID}:connect_me:{year}`).
- Resend (notificações empresa quando RFP chega — `IEmailProvider`).
- Twilio (SMS fallback — `ISMSProvider`).
- NATS JetStream (events cross-domain).

### Detalhamento completo

[[../00-product/sub-produtos/02-connect-me]]

---

## Produto 3 — Reputação Bronze → Diamante determinística

### Descrição

Sistema de tiers automático que qualifica empresas (e futuramente síndicos profissionais, agências). Calculado **deterministicamente** via fórmula pública a partir de métricas auditáveis — **zero ML**, **zero black-box**, **zero input humano direto** no tier. Alimenta ranking Connect Me (ADR-0023), badge público, filtros de busca.

**5 tiers** (fórmula determinística canonizada em D-051):

| Tier | Requisito acumulado |
|---|---|
| **Bronze** | Default — cadastro completo + CNPJ verificado (Receita Federal) |
| **Prata** | ≥ 3 contratos fechados + avaliação média ≥ 3/5 |
| **Ouro** | ≥ 10 contratos + ≥ 3.5/5 + ≥ 3 condomínios distintos |
| **Platina** | ≥ 30 contratos + ≥ 4.0/5 + ≥ 10 condomínios + ≥ 1 vídeo institucional aprovado + ≥ 1 case em vídeo |
| **Diamante** | ≥ 100 contratos + ≥ 4.5/5 + ≥ 30 condomínios + ≥ 3 vídeos + múltiplos cases + tempo de plataforma ≥ 12 meses |

**Fórmula score [0,100]**: `40 * rating_avg_weighted + 25 * success_rate + 15 * tenure_years_capped + 10 * volume_cases_capped + 10 * content_engagement`.

**Cold start**: empresa nova recebe rótulo "Não classificada" por 30 dias OU até 3 contratos (evita estigma).

**Sub-produto interno** (não é módulo de UI próprio — é `ReputationCalculator` Domain Service em `commercial` BC).

### Personas-alvo

- **Empresa `plus/pro`** (alvo principal do score).
- **Síndico** (consumidor: filtro Connect Me por tier, decisão de contratação).
- **Morador** (consumidor: visualiza tier em perfil público).

### Tiers aplicáveis

Não é feature vendida por tier — é score que a empresa acumula. Empresa precisa ter tier `plus+` para ter perfil com reputação visível.

### Bounded Context

- **commercial** (aggregate `Company.reputation_tier`, `CompanyEvaluation`, `ReputationCalculator` domain service).
- Cross: **content** (video institucional published → componente `content_engagement`).

### Invariantes

- **Determinístico**: função pura (ADR-0023). Mesmo input, mesmo output.
- **Recálculo em eventos**: novo contrato fechado, nova avaliação, novo vídeo aprovado → `RecalculateReputation(companyId)` idempotente.
- **Degradação permitida** (se avaliação cai + contratos cancelados).
- **Suspensão temporária** por harassment confirmado = flag `suspended=true` sobre tier atual; NÃO é despromoção.
- **GR-028**: `PATCH /companies/{id}/reputation_tier` direto → 403 **mesmo para admin** — reputação é inalterável manualmente.
- **Sem pular tier**: recálculo sequencial (Bronze→Prata→Ouro...).
- **LGPD Art. 20 explicabilidade**: síndico clica tier → vê breakdown completo das métricas.

### Marco

**M2 (2026-06-20)** — Bronze→Diamante + recálculo idempotente. M3: dashboard público empresa + Gap-SEC-008 (decay temporal). M4+: ML ring/farm detection (Gap-SEC-010).

### Diferenciais

- **Auditável**: síndico clica tier → vê as métricas que o geram (contratos, avaliações, condomínios, vídeos). Zero black box.
- **Anti-fraude estrutural**: impossível comprar tier — inputs observáveis (contratos reais, avaliações de síndicos reais, vídeos moderados).
- **Self-reinforcing**: empresa sobe → fecha mais contratos → atende bem → sobe. Alinhamento virtuoso.
- **Não é estrelas genéricas** — 5 critérios 1-10 específicos (qualidade, preço, pontualidade, profissionalismo, comunicação).
- **Não é score financeiro** (Serasa/SPC) — é reputação operacional no ecossistema.

### Dependências técnicas

- PostgreSQL (`companies`, `company_evaluations`, `closed_deals`, futuros campos `reputation_tier`, `suspended_until`, `score_components JSONB`).
- NATS JetStream (events `CompanyEvaluationSubmitted`, `ReputationTierChanged` — recálculo assíncrono).

### Detalhamento completo

[[../00-product/sub-produtos/03-reputacao]]

---

## Produto 4 — Conteúdo & Vídeos

### Descrição

Infraestrutura de upload, moderação, publicação, busca e distribuição de **vídeos institucionais** de empresas (12 tipos canônicos — enum `VideoType`: `apresentacao_empresa`, `case_sucesso`, `tour_virtual`, `tutorial_tecnico`, etc.) e vídeo do **perfil institucional do síndico** (trava trimestral Req 29). Backed por **Mux** via interface `IVideoProvider` (D-056 agnosticismo): upload direto presigned (GR-019 — backend nunca faz proxy de bytes), HLS transcoding, signed playback URL, CDN global.

**Trava de 90 dias pós-publish** (GR-029 / INV-RC-7): vídeo publicado não pode ser substituído por 90 dias. Motivador: churn prevention + compromisso com conteúdo.

**Moderação antes de visibilidade pública**: upload → `VideoModeration` aggregate em "pending" → admin aprova/rejeita em 48h → publish → lock 90d. Moderação manual em MVP; ML pipeline futuro M4+.

**Visibilidade estrita** (Rule 4 Req 103): vídeos de empresa **blindados por default** para moradores — só aparecem em contexto de votação de fornecedor ativa em assembleia (`autorizar_exibicao_votacao=true` + votação ativa). Auto-revogação via NATS `assembly.voting.closed` → `video.visibility.revoked`.

**Shorts canonicamente NÃO existe** (registrar): o formato vertical curto presente em redes sociais (Instagram Reels, TikTok, YouTube Shorts) **não é feature** do Master Síndico no MVP nem no roadmap atual. Vídeos institucionais são de formato longo controlado (pitch 1-2min, tutorial técnico 5-15min, tour virtual 2-5min). Vídeo-currículo 90s (Banco de Talentos — Produto 7) usa infra Mux compartilhada mas é outro sub-produto.

### Personas-alvo

- **Empresa `plus/pro`** (publisher: vídeos institucionais + cases).
- **Síndico `plus/pro`** (publisher: vídeo do perfil institucional + vídeo instrucional no LMS).
- **Morador** (consumer: vídeos síndico ilimitado; vídeos empresa preview 25% base, integral apenas em contexto votação).
- **Agência de Marketing** (publisher em nome da empresa via DelegationGrant — grant `ManageVideos`).
- **Admin** (moderação cross-tenant + aprovação manual).

### Tiers aplicáveis

| Persona | Uploads/mês |
|---|---|
| syndic `trial/base` | ⏳ / 0 |
| syndic `plus` | 4/mês |
| syndic `pro` | 12/mês |
| enterprise `trial` | ⏳ |
| enterprise `plus` | 8/mês |
| enterprise `pro` | 12/mês |

### Bounded Context

- **content** (principal — aggregate `Video`, `MarketingAgencyLink`).
- Cross: **commercial** (vídeo institucional empresa alimenta reputação tier Platina+), **assembly** (Rule 4 visibilidade).

### Invariantes

- **Lock 90d pós-publish** (GR-029).
- **Moderação antes de público** — upload → pending → approved → published.
- **Privacidade cross-tenant**: views/likes de empresa X não vazam pra empresa Y (ABAC strict).
- **CDN signed playback** — URL Mux assinada, expira, anti-hotlinking.
- **GR-019**: upload presigned direto pro Mux; backend **nunca** proxya bytes.
- **Rule 4**: vídeo empresa integral só durante votação de fornecedor.
- Title length 2-200 chars; description ≤ 2000; 12 tipos canônicos.

### Marco

**M2 (2026-06-20)** — básico com moderação manual + lock 90d + HLS via Mux + visibilidade estrita.

### Diferenciais

- **Lock 90d** — diferencial do produto; outros lugares não travam.
- **Moderação editorial** — não é free-for-all; Master Síndico cura qualidade institucional.
- **Categorização estruturada** — 12 tipos alinhados com Connect Me categories.
- **Visibilidade Rule 4** — privacy by design cirúrgica.

### Dependências técnicas

- **Mux** (via `IVideoProvider`) — upload direto presigned, HLS, signed playback, webhooks `asset.ready/errored`.
- **MinIO dev / R2 ou S3 prod** via `IStorageProvider` (thumbnails, storyboard — D-067 reconciliado).
- **`ISearchProvider` real** (OpenSearch ou Meilisearch — ADR-0042 pending dual-check; D-120 revoga D-067 PG tsvector).
- Resend (notificação moderação concluída).
- NATS (events `video.created/updated/visibility.*`).

### Detalhamento completo

[[../00-product/sub-produtos/04-conteudo-videos]]

---

## Produto 5 — Assembleias Live

### Descrição

Sub-produto mais ambicioso tecnicamente e de maior valor jurídico da plataforma. Automatiza o ciclo condominial brasileiro válido: **configuração → edital → ciência → quórum → votação com fração ideal → ata imutável com score de transparência**. Aderência a Art. 1.349 CC + Lei 4.591/1964 + Lei 14.063/2020 (assinatura digital).

**Aditivo contratual Fase 8** (D-062): escopo expandido para **9-10 sprints** com WebSocket + Telão + fila de fala + votação em tempo real. **Livekit** é provedor abstrato (`ILiveProvider` — D-056).

**5 blocos canônicos** (`Assembleia-Live-Funcional.md:17-24`):

1. **Bloco 1** — Configuração estrutural do condomínio (convenção, quórum, fração ideal, elegibilidade).
2. **Bloco 2** — Convocação + edital + rastreabilidade de ciência.
3. **Bloco 3** — Preparação de quórum, procurações 24h antes, fração ideal, elegibilidade de voto.
4. **Bloco 4** — Condução presencial/híbrida/online (no MVP: presencial completo; híbrida/online preparadas mas inativas).
5. **Bloco 5** — Apoio à fala (fila cronometrada), votação, deliberação, transparência operacional em tempo real, ata imutável + trilha de auditoria.

**Eleição gera avaliação escondida ativa** (D-065 Fase 8) — pode desabilitar depois; registra sinal de reputação do síndico.

**Lives = admin + Empresa Pro + agência delegada** (D-097 Fase 11 revoga D-062/D-063/D-076 que diziam admin-only MVP) — primeira live do sistema = apresentação da plataforma pelo próprio Master Síndico.

### Personas-alvo

- **Síndico principal** (cria, publica edital, inicia, preside, encerra, homologa).
- **Síndico auxiliar** (valida execução, sem presidir).
- **Administradora** (recebe token; read+observações; sobe fração ideal + inadimplência; valida procurações; homologa — Produto 11).
- **Conselho/Auditor** (read + observações, opcional cadastrar antes).
- **Morador titular pagante** (ciência, presença, procuração, check-in, votar com fração ideal).
- **Dependente** (leitura apenas; SEM voto — Req 9).
- **Admin MS** (read global + audit trail; não vota).

### Tiers aplicáveis

| Ação | Morador base | Morador pagante | Síndico trial+ |
|---|---|---|---|
| Ver pauta + dar ciência | ✅ | ✅ | ✅ |
| **Votar** | **❌ hard paywall** | ✅ | n/a |
| Procuração | ❌ | ✅ | n/a |
| Criar assembleia / publicar pauta / painel presidente | n/a | n/a | ✅ (trial+) |
| Relatório export | n/a | n/a | ⏳ trial / ✅ base+ |

### Bounded Context

- **assembly** (principal)
- Cross: **institutional** (assembly_result → TimelineEntry imutável), **commercial** (SupplierVoteSession sob assembleia para contratos acima de threshold), **identity** (ABAC + fração ideal via Membership).

### Invariantes

- **Quórum fracional**: peso de voto = fração ideal da unidade (Lei do Condomínio BR), não 1 pessoa = 1 voto.
- **Vote único por (agenda_item, voter)** — UNIQUE DB + `ErrConflict` (A-025 resolvido legado).
- **Vote APPEND-ONLY** — sem setters; código Go `vote.go` sem UPDATE/DELETE.
- **Minutes imutável pós-publish** — correção vira nova entry na Timeline institucional.
- **Assembly não inicia sem edital publicado** (A-013).
- **Proxy token**: hex 32B, `validUntil = scheduledDate + 24h`; validado 24h antes.
- **AgendaItem lock após publish** — fraction string.
- **ScienceRecord imutável** — `editalVersion` default "v1".
- **CheckIn só em `em_andamento`** — state machine rígida.
- **Retry backoff Livekit 100/300/900ms** (A-033/A-034) — saga compensação EndRoom.

### Marco

**M3 (2026-07-20)** — sessão live completa + votação + ata (MVP presencial). Aditivo contratual Fase 8: 9-10 sprints. Híbrida/online inativas em MVP.

### Diferenciais

- **Ata gerada automaticamente** — mapping pauta→resultado→texto padronizado.
- **Assinatura digital BR** — aderência a MP 2.200-2 e Lei 14.063 (valor jurídico).
- **Score de transparência** — `TransparencyComponents` (10 campos em `minutes.go`).
- **Auditabilidade total**: cada voto, proxy, fala com timestamps + frações.
- **Procuração + fração ideal** nativos — não é Google Forms adaptado.

### Dependências técnicas

- **Livekit** (via `ILiveProvider`) — SFU WebRTC, gravação, chat metadata channel, TURN/STUN gerenciados.
- **WebSocket** (nova camada aditiva) — fila de fala + telão + votação RT.
- **PostgreSQL** (`assembly`, `agenda_items`, `votes`, `minutes`, `proxies`, `attendance_records`, `science_records`).
- **PDF gen lib** (edital + ata).
- **NATS JetStream** (events `assembly.voting.closed` → dispara `video.visibility.revoked`).

### Detalhamento completo

[[../00-product/sub-produtos/05-assembleia-live]]

---

## Produto 6 — LMS / Academia Master Síndico

### Descrição

Academia digital **contextualizada ao universo condominial brasileiro**, integrada ao BC `content`. **5 áreas canônicas** (Direito condominial, Finanças condominiais, Comunicação/gestão de conflitos, Gestão técnico-operacional, Governança e tecnologia), **3 níveis** (básico/intermediário/avançado), **3 trilhas piloto** M3 (Síndico Iniciante→Intermediário→Profissional, Empresa, Morador). **15 telas canônicas** (LMS1–LMS15).

**Componentes** (`Domínio - Conteúdo.md §Requisito 98.1`):

1. **Cursos** — Curso → Módulo → Aula (vídeo Mux) → Material complementar → Quiz por módulo → Certificado ao 100%.
2. **Treinamentos** — peça curta e prática (vídeo único 15-60 min): porteiros, primeiros socorros, atendimento morador, prevenção de pragas, emergência.
3. **Lives** — transmissão agendada + chat + Q&A + replay; **admin-only no MVP** (D-062/D-063 Fase 8).
4. **Fórum** — tópico → respostas → curtidas → marcar resolvido; 7 categorias temáticas. Criar tópico restrito a sub-roles específicos.
5. **Biblioteca** — material avulso (PDF/EPUB via `IStorageProvider` — MinIO/R2 — não Mux; vídeo/link) por tipo e categoria.

**Parceiro Vizinhança NÃO tem acesso** (INV-067 / Decision 7) — botão oculto na navegação.

**Integração cross-domain** (Req 23): registrar execução de serviço recomenda treinamentos relacionados (ex: manutenção de bombas → treinamento hidráulico). Certificados em "Relacionamento com síndicos" alimentam score de Empresa (Reputação).

### Personas-alvo

- **Síndico** (tiers `trial/base/plus/pro` — certificado exige `plus+`; trial vê mas não emite certificado).
- **Morador pagante** (tier `plus` — certificado emitido).
- **Empresa** (tier `plus` consome; tier `pro` **cria cursos** para certificação profissional).
- **Parceiro Vizinhança** — SEM acesso.
- **Agência de Marketing** — não consome/cria.
- **Admin** (cria lives MVP; moderação de conteúdo).

### Tiers aplicáveis

Matriz `functional-matrix.md:47-49` consolidada:

| Ação | `resident.base` | `resident.plus` | `syndic.trial/base/plus/pro` | `enterprise.trial/plus/pro` |
|---|---|---|---|---|
| Ver Lives | ❌ | ✅ | ✅ | ✅ |
| Fazer cursos | ❌ | ✅ | ✅ | ✅ |
| Certificado | ❌ | ✅ | ❌ / ❌ / ✅ / ✅ | ❌ / ✅ / ✅ |
| Criar curso | ❌ | ❌ | ❌ | ❌ / ❌ / ✅ (pro only) |

### Bounded Context

- **content** (principal — aggregate `Course`, `Enrollment`, `Certificate`, `LiveSession`).
- Cross: **commercial** (certificado alimenta reputação empresa), **institutional** (recomendação cross-execução Req 23).

### Invariantes

- **Certificate serial único** (INV-068) — verificável via URL pública.
- **Aprovação ≥ 70%** em quiz para avançar módulo.
- **Lives admin-only MVP** (D-062/D-063).
- **Lock 90d pós-publish** (aplicado ao conteúdo curso publicado — RC-7 via extensão GR-029).
- **Fórum — criar tópico restrito** (não público geral).

### Marco

**M3 thin (2026-07-20)** — 5 áreas + 3 trilhas + certificados sem assinatura BR pleno. **M4+ pleno** + parceria ABRACS (50+ cursos).

### Diferenciais

- **Capacitação contextualizada BR** — direito condominial real, CLT/terceirização porteiros, ABNT NBR 5674 manutenção predial.
- **Certificados verificáveis** — síndico linka no Connect Me para aumentar confiança.
- **5 áreas específicas** vs genérico de marketplaces de curso.
- **Integração cross-produto** — execução de serviço recomenda curso relacionado.

### Dependências técnicas

- **Mux** (vídeo-aulas via `IVideoProvider`).
- **MinIO dev / R2 prod** (ebooks PDF/EPUB via `IStorageProvider`).
- **PostgreSQL** (`courses`, `enrollments`, `quizzes`, `certificates`, `forum_topics`).
- **Livekit** (Lives admin-only MVP — `ILiveStreamProvider`).

### Detalhamento completo

[[../00-product/sub-produtos/06-lms-academia]]

---

## Produto 7 — Banco de Talentos

### Descrição

Sub-produto que **conecta morador-candidato a empresa condominial contratante** dentro do Master Síndico, via **vídeo-currículo curto (90s)** + ficha estruturada + busca estruturada por área/região/CEP/raio/modalidade/horário. Mercado sub-atendido (doméstica, porteiro, zeladoria, manutenção técnica, jardinagem, limpeza de reservatório, brigada de incêndio NR-23/35, controle de pragas).

**11 telas canônicas** (BT01–BT11) — v2 cobertura 0% hoje; task agendada.

**Ficha estruturada** (destilada de `System Architecture - Content.md § TalentBankModule`):

- **18 áreas de atuação condominial** (enum `TalentArea`).
- **9 modalidades de contratação** (`ContractModality`: clt/pj/estagio/temporario/folguista/diarista/freelancer/intermitente/outros).
- **4 valores CNH** (`nao/A/B/AB`).
- **4 NR certifications**.
- **4 schedules de disponibilidade** (`AvailabilitySchedule`).
- **3 start availability** (`StartAvailability`).
- **3 travel time** (`TravelTime`).
- **5 últimos vínculos profissionais** (D-099 Fase 11 — sobrescreve D-060/D-064; alterável via matriz funcional; Q39 fechada 2026-04-24 na direção do PDF original).

### Personas-alvo

- **Morador pagante** (publisher do vídeo-currículo).
- **Empresa `plus/pro`** (busca + favoritos + tags + convite via Connect Me especial).
- **Agência de Marketing** (via grant ABAC — buscar currículos em nome da empresa cliente).
- **Parceiro Vizinhança** — SEM acesso.
- **Síndico** — não consome diretamente.

### Tiers aplicáveis

| Ação | base | plus | pro |
|---|---|---|---|
| Upload vídeo-currículo 90s | ❌ | ✅ (resident) | ✅ (resident) |
| Ver/buscar currículos | ❌ | ✅ (enterprise, marketing) | ✅ (enterprise, marketing) |

### Bounded Context

- **commercial** (busca + favoritos + tags — lado empresa).
- Cross: **content** (vídeo-currículo usa infra Mux via `IVideoProvider`; lock 90d; mesma moderação).

### Invariantes

- **Vídeo-currículo 90 segundos exatos** (não 60 — decisão consolidada).
- **Lock 90d pós-upload** (GR-029 extensão).
- **Morador pagante exclusivo**; Base bloqueado.
- **Empresa Plus/Pro exclusivo**; Parceiro bloqueado.
- **Assimétrico** (morador publica ↔ empresa busca) — sem feed público.
- **LGPD by design**: morador controla visibilidade; logs de quem viu currículo em `curriculum_search_logs`.

### Marco

**M3 (2026-07-20)** — vídeo-currículo + search básico + 11 telas. M4+: ML match ranking.

### Diferenciais

- **Vídeo como mídia primária** (alto-impacto, baixo esforço) — não CV texto estilo Europass.
- **Nicho condominial** — porteiros, zeladores, diaristas, técnicos prediais sub-atendidos por LinkedIn/Catho.
- **B2B2C** — empresa contrata; morador trabalha em condomínios da carteira.
- **Assimétrico** — sem rede social, sem feed, sem endorsements.

### Dependências técnicas

- **Mux** (via `IVideoProvider` — infra compartilhada com Produto 4).
- **PostgreSQL** (`curricula`, `curriculum_videos`, `curriculum_professional_profiles`, `curriculum_areas_of_interest`, `curriculum_work_experience`, `curriculum_education`, `company_curriculum_favorites`, `company_curriculum_tags`, `curriculum_search_logs`).
- **`ISearchProvider` real** (busca estruturada — OpenSearch ou Meilisearch via ADR-0042; D-120 revoga D-067 tsvector).

### Detalhamento completo

[[../00-product/sub-produtos/07-banco-de-talentos]]

---

## Produto 8 — Vizinhança

### Descrição

Rede hyperlocal que conecta **comércio local (Parceiro)** a **moradores pagantes** dos condomínios da região, com **curadoria do síndico** (seal "Síndico-aprovado" via votação informal de síndicos `plus+`) e **empresas condominiais como participantes secundários**.

**4 jornadas integradas, 29 telas**:

- **VZ1–VZ6** — Jornada Parceiro (cadastro, raio, promoções, cupons, métricas).
- **VS1–VS10** — Jornada Síndico (indicar parceiros, conceder seal, curadoria do bairro).
- **VE1–VE6** — Jornada Empresa (ver parceiros do bairro, benefícios corporativos).
- **VM1–VM7** — Jornada Morador (consumir benefícios, resgatar cupons).

**Sistema de Cupons ATIVO** (D-059 Fase 8 / D-064 Fase 7 / Q23 fechada):
- **Formato canônico**: `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` = 13 chars (ex: `00123055PAO`).
- **Lock 4h por estabelecimento** — um cupom ativo a cada 4h (escassez genuína, anti-spam).
- **Escopo duplo**: `aberta_bairro` (todo ecossistema local) vs `exclusiva_condominio` (só moradores do condo).
- **Palavra-chave do dia** (Req 100 — gamificação): parceiro `pro` define palavra diária; morador redime cupom digitando palavra.

**Parceiro vende ao morador como consumidor final** — NÃO presta serviço ao condomínio. Serviço-ao-condomínio é jornada B2B formal (Connect Me → Proposta → Deal → Execução, Produto 2).

### Personas-alvo

- **Parceiro Vizinhança** (`local_company` — publisher, tier `plus+` operacional MVP).
- **Morador pagante** (`resident.plus+` — consumer).
- **Síndico `plus+`** (emissor do seal; curadoria).
- **Empresa `plus/pro`** (ver parceiros do bairro; benefícios corporativos secundários).

### Tiers aplicáveis

| Ação | `local_company.trial` | `local_company.plus` | `local_company.pro` |
|---|---|---|---|
| Criar cupom (lock 4h) | ⏳ | ✅ | ✅ |
| Palavra-chave do dia | ❌ | ⚠️ | ✅ |
| Receber seal síndico | ✅ | ✅ | ✅ |
| Enviar Connect Me | ❌ | ❌ | ❌ (parceiro só RECEBE) |

### Bounded Context

- **commercial** (principal — aggregate `Partner`, `NeighborhoodBenefit`, `Coupon` — pendente de implementação Go).
- Cross: **institutional** (seal do síndico afeta visibilidade).

### Invariantes

- **Parceiro não envia Connect Me** — só recebe (anti-spam a síndicos).
- **Cupom lock 4h por estabelecimento** — escassez genuína.
- **Formato cupom fixo** `ID_LOJA(5)+SEQ(3)+PALAVRA(5)` — validação rígida.
- **Parceiro NÃO tem acesso à Academia LMS** (Decision 7).
- **Raio default 2km**, ajustável por admin.
- **Escopo `aberta_bairro` vs `exclusiva_condominio`** — ABAC strict.

### Marco

**M2 (2026-06-20)** — mapa + cupons + seal + 4 jornadas. M3: polish geo + campanhas complexas.

### Diferenciais

- **Hyperlocal com raio configurável** — não Groupon genérico.
- **Curadoria do síndico** — seal afeta visibilidade; não é mercado aberto.
- **Sistema Cupons com lock temporal** — anti-spam + escassez.
- **Ativação presencial** ("apresente esta tela no estabelecimento") — não processa pagamento; não gerencia entrega.
- **Gamification** via palavra-chave do dia (tier `pro`).

### Dependências técnicas

- **PostgreSQL** (lat/long + raio — PostGIS opcional M5+).
- **`ISearchProvider` real** (busca hyperlocal — OpenSearch ou Meilisearch via ADR-0042; D-120 revoga D-067 tsvector).
- Resend (notificação morador cupom novo próximo).
- **Redis** (lock 4h cupom + palavra-chave cache).

### Detalhamento completo

[[../00-product/sub-produtos/08-vizinhanca]]

---

## Produto 9 — Compliance

### Descrição

Camada transversal de **blindagem documental + LGPD + memória probatória** que operacionaliza:

- **Cap. VII do Código Civil (arts. 1.347-1.356)** — deveres do síndico.
- **Lei 13.709/2018 (LGPD)** — rastreabilidade de revelação de dados pessoais (Connect Me).
- **Deveres fiduciários** — declarações, conflitos de interesse, trilha imutável.
- **Transparência condominial** — dossiê exportável para assembleia/seguro/processo judicial.

**11 telas canônicas** (C1–C11) na Central de Governança do síndico.

**Princípio operacional**: módulo NÃO bloqueia o cadastro inicial. Bloqueia **4 gatilhos críticos**:

| Gatilho | Trigger | Bloqueia se |
|---|---|---|
| **G1: encerrar_mandato** | `EndMandateUseCase.Execute` | Declaração anual do ano corrente não submetida |
| **G2: gerar_relatorio_final** | `GenerateFinalReport` (pendente) | G1 + assinaturas incompletas + dossiê não gerado |
| **G3: marcar_negocio_fechado** | `CloseDeal` commercial | Termos pendentes (C8) — opcional |
| **G4: exportar_dossie** | `ExportDossier` (pendente C11) | Declaração anual + avaliação gestão em dia + compliance markers |

**1 Score Governança único** (D-066 Fase 8 — revisável) — 0-100, 10 componentes, job noturno, histórico 12 meses, alerta < 60. A decisão abole o modelo anterior "2 scores (Governança + Compliance)".

### Personas-alvo

- **Síndico principal** (assina, declara, visualiza auditoria).
- **Síndico auxiliar / Conselho / Administradora** (assinatura obrigatória de termo).
- **Admin MS** (auditoria total + revisão de conflitos).

### Tiers aplicáveis

Transversal — todas as ações compliance do síndico ficam disponíveis a partir de `trial` (com bloqueios em criação). Admin bypass total.

### Bounded Context

- **institutional** (principal — aggregates `ComplianceTerm`, `AnnualDeclaration`, `AuditLogEntry`, `ManagementAssessmentResponse`).
- Cross: **commercial** (bloqueio de close deal via G3), **assembly** (C6 adendos), **identity** (LGPD logs vinculados a membership).

### Invariantes

- **Append-only enforcement**: DB-level via query pattern — nunca UPDATE/DELETE em `audit_log_entries`, `lgpd_logs`, `annual_declarations`, `timeline_entries`.
- **Hash SHA-256 do termo assinado** — integridade jurídica; nova versão de termo exige nova assinatura.
- **Adendos vs edit** (Rule 2): correção sem apagar histórico; nova entry com `previous_entry_id`.
- **Compliance só "Em dia" quando TODOS usuários obrigatórios assinarem** — `Compliance-Detalhado.md:70`.
- **Retenção LGPD 5 anos** em `lgpd_logs`, `audit_log_entries`, `annual_declarations`.
- **Score é determinístico** — não editável manualmente.
- **Regra de continuidade entre mandatos**: novo síndico assume → nova declaração; histórico read-only; Score reseta.

### Marco

**M1 (2026-05-08)** — base compliance (declaração anual + assinaturas + auditoria). M2: Score + LGPD logs completos. M3: Dossiê exportável + Risco + Contratual.

### Diferenciais

- **Não é checklist estático** — arquitetura viva integrada aos módulos.
- **Operacionaliza Cap. VII CC** — declarações com IP/timestamp/hash.
- **LGPD by design** — `lgpd_logs` integrado a Connect Me.
- **Dossiê exportável** (C11) — artefato para assembleia/seguro/processo.
- **Score único transparente** com histórico de 12 meses.

### Dependências técnicas

- PostgreSQL (`compliance_term_signatures`, `annual_declarations`, `audit_log_entries`, `lgpd_logs`, `governance_score_history`).
- **SHA-256 hashing** (Go stdlib).
- Job noturno (Go cron) — recálculo Score.
- Resend (lembretes declaração anual).

### Detalhamento completo

[[../00-product/sub-produtos/09-compliance]]

---

## Produto 10 — Marketing / Agências

### Descrição

**Sub-produto interno próprio** (D-059 Fase 7 / D-060 Fase 8) — NÃO é produto externo, NÃO é impersonation/actor delegado (modelo antigo legado). Área dedicada da plataforma onde **agências de marketing** administram o marketing institucional de **múltiplas empresas condominiais clientes**.

**Login próprio via Zitadel** (role `marketing`); **delegação ABAC explícita** (aggregate `DelegationGrant`) por empresa cliente — ela opera **no tenant da empresa** via grants `ManageVideos`, `ViewAnalytics`. NÃO tem tenant próprio condominial.

**8 telas canônicas** (MK1–MK8) — família MK v2 cobertura ~100%.

Agência **nunca vê dados sensíveis**: Connect Me, propostas, registros técnicos, plano diretor, dados pessoais de síndicos/moradores — ABAC cirúrgico (Req 30 restrições).

**Marketing Link** (feature distinta de Connect Me): formulário Empresa→Agência a partir de tags em vídeos. Sem cota. Tabela separada `marketing_link_requests`. E16→MK8.

Agência é convidada por **Empresa Pagante** (tier `plus+`) via E14 "Convidar Agência" — token por email. Não se auto-cadastra do zero comprando plano.

### Personas-alvo

- **Agência de Marketing** (`marketing`) — atua em nome de Empresa cliente.
- **Empresa `plus/pro`** (convida agência; revoga grant).
- **Admin MS** (moderação cross-tenant + aprovação de cadastro de agência).

### Tiers aplicáveis

Agência **não consome plano próprio no MVP** — opera sob tier da empresa cliente. Capacidade "Delegar Agência" é da Empresa e exige `plus+` (`functional-matrix.md:87`).

### Bounded Context

- **content** (agência edita/publica vídeos via `MarketingAgencyLink`).
- **commercial** (recebe Marketing Link em nome de empresa — tabela `marketing_link_requests`).
- **identity** (role `marketing`, autenticação Zitadel + PKCE).

### Invariantes

- **DelegationGrant** aggregate com permissões granulares (`ManageVideos`, `ViewAnalytics`).
- **1 agência ↔ N empresas** (`Domínio - Conteúdo.md:77-88 Req 30`).
- **Vídeo atribuído à empresa**, não à agência (identidade preservada — síndico/morador vê conteúdo da Empresa X).
- **Token de convite** com TTL; revogação automática após TTL.
- **Revogação a qualquer momento pela empresa** — agência perde acesso imediato (cache Redis invalidado).
- **Proibido**: enviar/receber Connect Me, ver deals, ver propostas, gerenciar usuários da empresa.
- **1 dispositivo ativo por conta** (Rule 8).
- **Cache Redis `ms:auth:{zitadelUserID}` TTL 5min**.

### Marco

**M3 (2026-07-20)** — login Zitadel + DelegationGrant + MK1-MK8 + Marketing Link completo.

### Diferenciais

- **Lock-in operacional desejado** — conteúdo feito lá dentro, métricas centralizadas, templates preservados.
- **Segurança por desenho** — agência NUNCA vê dados sensíveis (Connect Me, propostas, plano diretor).
- **Identidade preservada** — vídeo é da empresa; se trocar de agência, nada muda para síndico/morador.
- **Grants cirúrgicos** — ABAC explícito por empresa, por permissão.
- **Marketing Link** como canal dedicado de captação de leads não-intrusiva.

### Dependências técnicas

- **Zitadel** (via `IIdentityProvider` — auth code + PKCE).
- **PostgreSQL** (`marketing_agency_links`, `marketing_link_requests`).
- **Mux** (via `IVideoProvider` — upload vídeos em nome da empresa).
- **Redis** (cache auth + tokens de convite).
- Resend (notificação convite + revogação).

### Detalhamento completo

[[../00-product/sub-produtos/10-marketing-agencias]]

---

## Produto 11 — Administradoras Condominiais

### Descrição

Empresa especializada em **administração condominial** (contábil, jurídica, prestação de contas, suporte operacional ao síndico). No MVP, **não é produto separado**: é um **tipo de Empresa** (role `enterprise`, subcategoria `ADMINISTRADORAS_CONDOMINIOS` dentro da categoria `ADMINISTRACAO_GESTAO_GOVERNANCA` — `enterprise-catergories-subcategories-enum.schema.ts`) que pode ser **vinculada** a um condomínio via aggregate `EmpresaSindicaLink` no BC `commercial`, com **permissões delegadas configuráveis**.

**3 termos overlapping**:

- **Administradora** — termo de mercado (empresa de administração condominial) + termo usado em Assembleia.
- **Empresa Síndica** — termo do aggregate/fluxo de delegação (`EmpresaSindicaLink`, `empresa_sindica_links`).
- **Administradoras Condominiais** — rotulagem deste sub-produto + subcategoria no enum.

No MVP, todos se referem à **mesma entidade**. Em **M5+**, evolui para `empresa-síndica` = tenant multi-condomínio próprio com dashboard consolidado.

**Atribuições operacionais**: contabilidade condominial, prestação de contas, assessoria jurídica, emissão de lista de inadimplentes (obrigatória até 1h antes da 1ª convocação de assembleia), gestão documental, suporte em Assembleia (mesa + validar procurações 24h antes + homologar votação + emitir edital), validação delegada de registros de execução e comunicados técnicos (se síndico autorizar — permissões 4 e 5).

### Personas-alvo

- **Empresa com subcategoria `ADMINISTRADORAS_CONDOMINIOS`** (role `enterprise`, sub-tipos `administrator | operator`).
- **Síndico** (emissor do convite via tela S5; configura permissões).
- **Admin MS** (moderação + auditoria do vínculo).

### Tiers aplicáveis

Administradora opera com os mesmos tiers da Empresa (`plus / pro`; trial 7 dias). Tier `pro` geralmente necessário para operações de gestão (validar execuções, gerenciar múltiplos condomínios na carteira).

### Bounded Context

- **commercial** (principal — aggregate `EmpresaSindicaLink`, state machine `pending → active → revoked`).
- Cross: **institutional** (modelagem `EmpresaSindica` no CondominiumModule; TimelineEntry ao vincular/revogar), **assembly** (mesa da administradora, homologação).

### Invariantes

- **Chave de vínculo**: `(condominium_id, company_id)` quando ativo; `(condominium_id, token)` na fase pending.
- **Status enum** (`EmpresaSindicaLinkStatus`): `pending → active → revoked`.
- **Token UUIDv7 único** no convite; email para administradora; `Accept(company_id)` carimba `linked_at`.
- **Permissões configuráveis**: configurar por síndico (ex: "pode validar execução de outras empresas", "pode publicar comunicado técnico").
- **CNPJ normalizado** 14 dígitos (`migration 00310_normalize_empresa_sindica_cnpj.sql`).
- **Audit log obrigatório** — toda ação da administradora registrada.
- **Voto em assembleia**: administradora pode ocupar mesa + homologar + apresentar documentos no telão, mas voto ponderado por fração ideal pertence aos moradores; voto de desempate = presidente da mesa (síndico).
- **Administradora ≠ síndico eleito** — Código Civil art. 1.347 (síndico é figura legal).
- **Administradora ≠ Admin MS** — role `admin` é equipe interna Master Síndico.

### Marco

**M1 (2026-05-08)** — cadastro como Empresa + `EmpresaSindicaLink` básico + S5 tela de convite. M2: permissões granulares configuráveis. M5+: tenant multi-condomínio próprio + dashboard consolidado.

### Diferenciais

- **Delegatária de poderes de gestão** — diferente de Empresa prestadora comum (que executa serviços pontuais).
- **Permissões configuráveis** — síndico decide o que a administradora pode fazer.
- **Integração nativa com Assembleia** — mesa da administradora + procurações + homologação.
- **Evolução roadmap clara** — M5+ tenant próprio com dashboard consolidado.
- **Token + audit** — Req 7 AC9-12.

### Dependências técnicas

- PostgreSQL (`empresa_sindica_links`, `companies`, `admin_permissions`).
- Resend (envio do token de convite por email).
- Zitadel (role `enterprise` + `role_type=administrator`).

### Detalhamento completo

[[../00-product/sub-produtos/11-administradoras-condominiais]]

---

## Mapa sub-produto × bounded context × marco × status

| # | Sub-produto | BC principal | BCs cross | Marco | Status legado |
|---|---|---|---|---|---|
| 1 | Governança Institucional | institutional | identity, commercial, assembly, billing | **M1** | ✅ Produção (Sprint 2/3) |
| 2 | Connect Me (4 direções + Marketing Link) | commercial | institutional, identity, content | **M2** | ✅ Produção (Sprint 4) |
| 3 | Reputação Bronze→Diamante | commercial | content | **M2** | ⏳ Calculator pendente (backlog T-151..T-155) |
| 4 | Conteúdo & Vídeos (lock 90d; Shorts NÃO existe) | content | commercial, assembly | **M2** | ✅ Produção (Sprint 5) |
| 5 | Assembleias Live (5 blocos WebSocket) | assembly | institutional, identity, commercial | **M3** (9-10 sprints aditivo D-062) | ✅ Backend produção (Sprint 7); frontend incompleto |
| 6 | LMS/Academia (15 telas, 5 áreas, 3 trilhas) | content | identity, billing, commercial | **M3 thin** / **M4+ pleno** | ⏳ Backlog |
| 7 | Banco de Talentos (5 vínculos — D-099 Fase 11, 11 telas) | commercial | content, identity | **M3** | ⏳ Backlog (0% cobertura v2 hoje) |
| 8 | Vizinhança (4 jornadas VZ/VS/VE/VM + Sistema Cupons ATIVO) | commercial | institutional | **M2** | ⏳ Parcial (Connect Me Síndico→Parceiro pronto) |
| 9 | Compliance (C1-C11 + 4 gatilhos + Score único) | institutional | commercial, assembly, identity | **M1** | ✅ Base produção; Score + Dossiê pendentes |
| 10 | Marketing / Agências (sub-produto interno próprio) | content | commercial, identity | **M3** | ⏳ MK1-MK8 mapeado; DelegationGrant pendente |
| 11 | Administradoras Condominiais (EmpresaSindicaLink) | commercial | institutional, assembly, identity | **M1** | ✅ Produção backend; UI S5 parcial |

---

## Dependências inter-sub-produto

- **1 Governança Institucional** é pré-requisito de todos os outros (condomínio criado + memberships ativos).
- **Billing** (transversal, não listado como sub-produto — é o "habilitador" de quotas/features) é pré-requisito de 2-11.
- **2 Connect Me + 3 Reputação acoplados**: toda avaliação pós-contrato em Connect Me alimenta Reputação.
- **4 Conteúdo & Vídeos** é transversal: perfil empresa precisa para subir Platina+ (Reputação); Connect Me mostra vídeo institucional; LMS usa infra Mux; Banco de Talentos usa infra Mux.
- **5 Assembleias** gera TimelineEntry em Governança + pode disparar supplier vote que alimenta Connect Me (contrato formalizado) + dispara `video.visibility.revoked` via NATS.
- **6 LMS + 7 Banco de Talentos** compartilham infra de conteúdo (Mux + moderação + lock 90d).
- **8 Vizinhança** semi-isolado; integra via seal do síndico + Connect Me Síndico→Parceiro.
- **9 Compliance** transversal — bloqueia 4 gatilhos em outros sub-produtos.
- **10 Marketing / Agências** opera sobre Produto 4 (vídeos) + recebe Marketing Link do Produto 2 (Connect Me BC mas feature separada).
- **11 Administradoras** opera sobre Produto 1 (condomínio) + Produto 5 (assembleia) com permissões delegadas.

---

## Roadmap de entrega por sub-produto

| # | Sub-produto | M1 (2026-05-08) | M2 (2026-06-20) | M3 (2026-07-20) | Pós-M3 |
|---|---|---|---|---|---|
| 1 | Governança | ✅ Base | 🔼 Poll + Announcement completo | 🔼 Event + compliance | ML signals |
| 2 | Connect Me | — | ✅ 4 fluxos + Marketing Link | 🔼 Marketplace ranking via Reputação | Match ML |
| 3 | Reputação | — | ✅ Bronze→Diamante | 🔼 Dashboard público empresa | ML + decay |
| 4 | Conteúdo & Vídeos | — | ✅ Mux + lock 90d + mod manual + Rule 4 | 🔼 Privacidade métricas | ML moderation |
| 5 | Assembleia | — | Backend ready | ✅ Live + votação + ata + WebSocket | Online/híbrido M4+ |
| 6 | LMS | — | — | ✅ Thin (5 áreas, 3 trilhas, 15 telas) | M4 pleno + ABRACS |
| 7 | Banco Talentos | — | — | ✅ Vídeo-curr + search + 11 telas | ML match |
| 8 | Vizinhança | — | ✅ Mapa + cupons + seal + 4 jornadas | 🔼 Polish geo + palavra dia pleno | — |
| 9 | Compliance | ✅ Base (declaração + assinatura + auditoria) | 🔼 Score único + LGPD logs | 🔼 Dossiê C11 + Risco C7 | — |
| 10 | Marketing/Agências | — | — | ✅ Zitadel + DelegationGrant + MK1-MK8 | Plano próprio M5+ |
| 11 | Administradoras | ✅ EmpresaSindicaLink + S5 básico | 🔼 Permissões granulares | — | Tenant próprio M5+ |

---

## Anti-catálogo (o que ninguém promete — protege o foco)

Ver [[vision#fora-de-escopo-guardrails]]:

- **Assembleia online/híbrida completa** — M4+ (MVP é presencial; online/híbrida inativas).
- **Empresa-síndica tenant próprio multi-condomínio** — M5+.
- **Agência Marketing plano próprio** — M5+ sob demanda validada.
- **Moderação ML** — M4+ (MVP manual 48h).
- **Multi-região/país** — M6+.
- **Marketplace financeiro** (split pagamento Connect Me) — M5+ com compliance BR.
- **Integração ERP** (Superlógica, Igora) — M4+ parceria comercial.
- **Shorts / vídeo vertical curto estilo TikTok/Reels** — NÃO existe no roadmap; vídeos institucionais são longos controlados.
- **Reputação de síndico** (Bronze/Prata/Ouro/Diamante do síndico, herança do Manual Cap. 8) — M3+ se retornar; MVP foca reputação de Empresa.
- **Lives abertas ao LMS** — D-097 Fase 11 (revoga D-062/D-063/D-076 admin-only): admin + Empresa Pro + agência delegada criam lives. Primeira live = apresentação Master Síndico.
- **PATCH manual de reputation_tier** — sempre 403 (GR-028).
- **PostGIS para hyperlocal** — M5+; MVP usa lat/long + raio PostgreSQL.
- **OpenSearch ou Meilisearch** — `ISearchProvider` real obrigatório M1 (D-120 Fase 14 revoga D-067 que dizia PG tsvector); ADR-0042 `proposed pending dual-check`.
- **AWS SES** — abolido (D-067 Fase 7 reconciliação); usa Resend (M1-M2; SES M3+ via D-100 quando volume justificar).
- **AWS ECS** — abolido em M1-M3 (D-067 Fase 7); usa Railway (M4+ AWS via ADR-0017; reversível via D-056 agnosticismo). Edge primário = Cloudflare (D-117 Fase 14 / ADR-0040).

---

## Terminologia legada (tradução explícita)

Quando consultar fontes legadas, traduzir sempre:

- *"fonte usa rótulo 'N2' banido por D-067 Fase 7 — canônico aqui é `plan_tier=plus`"*.
- *"fonte usa slug `enterprise_pro` banido por ADR-0032 — canônico é `(role=enterprise, plan_tier=pro)`"*.
- *"fonte chama de 7 sub-produtos — canônico Fase 8 = 11 sub-produtos (`_moc.md` §Catálogo)"*.
- *"fonte descreve Agência como 'actor delegado via impersonation' — canônico D-060 é 'sub-produto interno com login Zitadel + DelegationGrant ABAC'"*.
- *"fonte descreve Admin como role privilegiada transversal embutida em CMS — canônico atual D-134 (Fase H, revoga D-058/D-061/D-072) é '`apps/admin` dedicada no monorepo web com Cloudflare Access SSO+MFA'"*.
- *"fonte lista PG tsvector como busca M1 — canônico atual D-120 (Fase 14, revoga D-067) é '`ISearchProvider` real (OpenSearch ou Meilisearch via ADR-0042 pending dual-check)'"*.
- *"fonte lista AWS SES — canônico D-067 é 'Resend M1-M2; SES M3+ via D-100 quando volume justificar'"*.

---

## Referências

- **Fonte autoritativa de coerência**: [[../00-product/sub-produtos/_moc]] + os 11 arquivos em `00-product/sub-produtos/01..11`.
- **ADR canônica**: [[../02-architecture/adr/0032-global-plans-stripe-style]].
- **Decisões vivas**: `STATE.md` Fase 8 D-056..D-069 + Fase 7 D-056..D-070 (ver DT-008 sobre duplicação).
- **Matriz funcional (backend)**: `Development/backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` (pendente DT-004 reescrita).
- **Requirements canônicos (backend)**: `Development/backend/.kiro/specs/master-sindico/requirements/` (institutional, commercial, content, assembly, identity, cross-domain, functional-matrix, personas-and-plans).
- **Código Go vivo**: `Development/backend/internal/modules/{institutional,commercial,content,assembly,identity}/`.
- **Obsidian Vault do cliente** (D-068): `~/Documentos/Obsidian Vault/Clients/MasterSindico/`.
- **Links**: [[vision]], [[personas]], [[business-model]], [[glossary]], [[../01-domain/bounded-contexts]], [[../ROADMAP]], [[../03-subprojects/admin-privileges]].
