---
title: Destilado Instagram/Meta — aplicabilidade ao Master Síndico
type: note
tags: [research, instagram, meta, destilado, master-sindico, v2, feed-ranking, tao, moderation, cache]
source: 13-research/instagram/source-01..10
created: 2026-04-23
updated: 2026-04-23
aliases: [instagram-destilado, meta-destilado]
---

# Destilado Instagram/Meta — aplicabilidade ao Master Síndico

> Síntese PT-BR das 10 fontes primárias (Meta Engineering, Instagram Engineering, USENIX, about.fb.com). Foco: o que importa pra arquitetura do Master Síndico (governança condominial BR, multi-tenant, 7 sub-produtos) — **não** é resumo enciclopédico do Instagram.

Fontes originais em `source-01.md` .. `source-10.md`. Nomes técnicos (TAO, Haystack, Akkio, FXL, Haxl, Sigma, ig2vec, MTML) ficam em EN; discussão em PT-BR.

---

## 1. Feed ranking (Explore/Reels) — aplicável sim, com ressalvas

### O que Meta/Instagram faz (resumido de `source-01`, `source-02`, `source-07`)

Funil de ranking multi-estágio do Instagram Explore (agosto/2023):

1. **Retrieval** — múltiplas fontes de candidatos (heurística tipo "trending" + ML tipo embeddings `ig2vec` + two-tower NN) selecionam centenas de itens de um pool de bilhões.
2. **First-stage ranker** — modelo leve, cacheável, reduz para milhares de candidatos.
3. **Second-stage ranker** — modelo pesado Multi-Task Multi-Label (MTML) opera nos ~100 finalistas.
4. **Final reranking** — aplica filtros de integridade e regras de diversidade.

Valor estimado = `W_click·P(click) + W_like·P(like) − W_see_less·P(see_less) + …`. Em 2025, a infraestrutura suporta 1000+ modelos em produção com um Model Registry central no Configerator (`source-02`).

### Aplicabilidade ao Master Síndico

Temos **dois surfaces** onde ranking faz sentido:

#### A) Feed de vídeos do condomínio

- **Escala**: centenas/milhares de vídeos por condomínio, não bilhões. O funil Meta de 4 estágios é **overkill** pro M1/M2.
- **O que cabe**: ranking simples por sinais (recência + engajamento + relevância pro perfil do síndico/morador), sem ML treinado.
- **O que NÃO cabe**: two-tower NN, embeddings pré-computadas, ANN service. Custo de infra não justifica em 1 condomínio.
- **Quando reconsiderar**: se M3+ trouxer >100 condomínios e >100k vídeos, avaliar retrieval + first-stage ranker light (sem second-stage).
- **Pattern a herdar agora**: **separar retrieval de ranking** (mesmo que ambos sejam SQL hoje), deixando o contrato pronto pra trocar a query retrieval por vector search no futuro.

#### B) Connect Me — ranking de empresas/prestadores

- **Aqui o paralelo é mais forte** porque o problema é marketplace (ranking ≈ Explore), não feed.
- **Sinais candidatos**: reputação média, volume de contratações, SLA de resposta, distância geográfica, match categoria × necessidade, taxa de conversão histórica.
- **Pattern aplicável**: value model somatório ponderado — `score = Σ Wi · Si`, com pesos configuráveis por ADR. Começar determinístico (não-ML) no M1; coletar labels implícitos (clique, contato iniciado, contratação fechada, avaliação posterior) que serão o ground truth se migrarmos pra ML em M3+.
- **Anti-pattern a evitar**: ranking 100% engajamento (like/click). Meta conta que precisou de sinais de **aversão** (`see less`, `not interested`) como contrapeso (`source-01`). Pro Connect Me: computar sinais negativos (empresa reportada, contrato cancelado, avaliação ruim) desde o D1.

**Decisão sugerida** (ver STATE): ranking Connect Me M1 = somatório ponderado determinístico; retrieval e ranking separados em camadas (interface `ConnectMeRanker` na domain).

---

## 2. Stories ephemeral (24h) — **NÃO aplicar**

### O que Meta faz (de `source-03`)

Stories: conteúdo desaparece após 24h, CDN agressivo, pré-fetch preditivo, tray ordenado por frequência de interação. "Story Highlights" é pipeline separado de retenção indefinida.

### Decisão Master Síndico

**Rejeitar explicitamente o padrão ephemeral.** Master Síndico valoriza **memória institucional permanente** via timeline imutável (uma das invariantes do produto). Comunicações e eventos do condomínio precisam ser auditáveis por anos (LGPD, disputas, assembleias, prestação de contas). Transitoriedade vai **contra** o valor central.

**O que pode ser inspirado sem copiar**:
- **Pré-fetch preditivo na tray** — aplicável ao feed de vídeos mobile (baixar os N primeiros do feed do condomínio do usuário antes dele clicar). Isso é UX/perf, não modelo ephemeral.
- **Lifecycle management separado** — quando tivermos notificações push/banners de assembleia próxima, isso sim é ephemeral (some após o evento). Tratar em bounded context distinto do timeline.

---

## 3. TAO (The Associations and Objects) — patterns aplicáveis no cache ABAC e feed

### O que Meta faz (de `source-04`)

TAO é o datastore de grafo social do Facebook (USENIX ATC 2013). Modelo: **objects** (nós tipados) + **associations** (arestas tipadas). Arquitetura:

- **Two-tier caching** por região: **follower tier** (clientes batem aqui) → **leader tier** (fala com MySQL e mantém consistência da região) → **MySQL** (persistente).
- **Write-through cache** — toda escrita passa pelo cache antes de cair no DB; cache invalida/atualiza em tempo de write.
- **Eventual consistency** entre regiões, lag tipicamente <1s. Read-after-write garantido dentro de um tier.
- Escala: 1B+ reads/s, millions writes/s.

### Aplicabilidade ao Master Síndico

#### A) Cache ABAC (autorização)

O ABAC cache (Redis) do Master Síndico armazena decisões de `pode_X?` baseadas em atributos (tenant, papel, recurso, contexto). Pattern TAO que aplica:

- **Write-through em mudança de permissão** — quando um síndico promove/demove um morador, a escrita vai simultaneamente ao DB **e** invalida/reescreve o cache ABAC. **Não usar write-behind aqui** — autorização não tolera janela de inconsistência; aprovar quem já foi demovido é vulnerabilidade de segurança.
- **Leader-follower regional** — overkill pra BR only (M1). Guardar a ideia caso o M3+ expanda pra nuvens regionais SP/RJ/NE (ver seção 5).
- **Chave de invalidação granular** — TAO usa shard-by-object-id. Pro ABAC: chave `tenant:{id}:user:{id}:perm_fingerprint:{hash}` permite invalidar apenas o subgrafo que mudou, não o cache inteiro.

#### B) Feed cache

Aqui **write-behind é aceitável** porque:
- Janela de inconsistência de segundos é tolerável (um vídeo novo aparecer com 2s de atraso não quebra o produto).
- Volume de writes alto (upload, like, comment) se beneficia de conflation (Meta menciona em `source-10`).

**Decisão sugerida**:

| Cache | Pattern | Justificativa |
|---|---|---|
| ABAC / autorização | **write-through** | segurança não tolera inconsistência |
| Feed de vídeos | **write-behind + conflation** | throughput, inconsistência de segundos tolerável |
| Ranking Connect Me (score pré-computado) | **write-behind** ou **refresh periódico** | score é aproximação mesmo, 5-15 min de lag OK |
| Sessão de usuário (JWT claims enriquecidos) | **write-through** | segurança |

---

## 4. Moderação de conteúdo — processo manual + ML, cronograma M3+

### O que Meta/Instagram faz (de `source-08`, complementado por `source-10`)

Pipeline híbrido:

1. **Classificadores ML** — CNN (imagens explícitas), transformer NLP (texto em dezenas de línguas), Rosetta OCR (texto sobre imagem, CNN+LSTM), detecção de objeto (YOLO/Faster R-CNN).
2. **Confidence threshold** — alto = auto-remove, médio = fila de revisão humana, baixo = deixa passar.
3. **Feedback loop** — decisões humanas retreinam o modelo.
4. **Sistema Sigma** anti-abuse (`source-10`) — regras em Haskell/Haxl, 1M+ req/s, combate spam/phishing/malware. Migrado de FXL em 2015, throughput +20-30%.

Taxa de detecção proativa em 2020: ~95% de hate speech removido foi encontrado pelo próprio Meta antes de report. Em 2025 Meta reduziu escopo proativo (decisão de política, não técnica).

### Aplicabilidade ao Master Síndico

**Cronograma sugerido**:

| Fase | Moderação |
|---|---|
| **M1 (2026-05)** | 100% manual. Síndico modera vídeos do próprio condomínio. Admin Master modera denúncias inter-tenant. Fila de denúncia + SLA. |
| **M2** | Filtros estáticos (lista de palavrões, regex de CPF/RG/telefone vazados, hashes de imagens bloqueadas conhecidas). Nenhum ML. |
| **M3+** | Avaliar API externa pronta (Perspective API, Azure Content Moderator, AWS Rekognition) **antes** de treinar modelo próprio. Treinar só se volume + labels justificarem custo. |
| **M4+** | Ponderar ML próprio para PT-BR + contexto condominial (siglas, jargão). Requer pipeline de labeling, retreino, avaliação. |

**Não replicar**:
- Rosetta OCR, CNN customizada, transformer próprio — overkill no horizonte visível.
- Haskell/Haxl — decisão de stack não negociável (Go no backend).

**Replicar conceitualmente**:
- **Confidence threshold com fila humana** — mesmo que o "classificador" seja lista de palavrões, o fluxo auto/humano deve existir desde o M1.
- **Sinais negativos explícitos** — todo item moderável tem botão "reportar" com motivo tipado (spam, ofensa, PII vazada, off-topic).
- **Feedback loop** — decisões de moderador são registradas como labels; vira dataset treinável no futuro sem re-trabalho.

**Decisão ABAC**: permissão `moderate_content` é atributo por escopo (síndico → só próprio condomínio; admin master → cross-tenant). Ver `08-security/abac-matrix.md` quando for escrito.

---

## 5. Multi-master / multi-region — BR only hoje, arquitetura permite regionalização

### O que Meta faz (de `source-04`, `source-05`)

- **TAO multi-region**: master-slave por shard; leader-follower dentro da região; eventual consistency entre regiões.
- **Akkio** (2018): data placement service que coloca μshards (tipicamente 1 usuário = 1 μshard) em 2-3 regiões próximas ao padrão de acesso, não em todas. Resultados: 40% menos storage, 50% menos WAN, ~50% menos latência percebida.

### Aplicabilidade ao Master Síndico

**M1-M2 (BR only)**: uma região AWS sa-east-1 (São Paulo). **Não implementar multi-master.** Seria engenharia prematura (YAGNI §14 do CLAUDE.md).

**O que preparar na arquitetura** pra não bloquear regionalização futura:

- **Sharding por tenant desde o D1** — `tenant_id` em toda tabela, toda query, toda chave de cache. Isso é requisito multi-tenant já, e é exatamente o que permite migrar um tenant de região sem mexer nos outros (μshard pattern Akkio).
- **Evitar cross-tenant joins** — se a query precisa cruzar tenants, algo está errado. Exceção: Admin Master (cross-tenant por design, mas em leitura read-only com BI).
- **ID estratégia** — usar ULIDs/UUIDs prefixados por tenant em vez de sequences globais, evitando colisão se regiões divergirem.
- **Outbox pattern + eventos idempotentes** — preparar pra replicação assíncrona SP↔RJ↔NE quando vier.

**Cenário M3+ realista**: 1 região principal + 1 DR (disaster recovery) cross-AZ inicialmente. Multi-region ativo-ativo só se M4+ tiver condomínios em regiões distintas com latência inaceitável na região única.

---

## 6. Write-through vs write-behind — consolidado

Já discutido em §3. Tabela consolidada para referência rápida (ver ADR quando escrita):

| Critério | Write-through | Write-behind |
|---|---|---|
| Consistência | forte (cache+DB sempre iguais) | eventual (janela de ms a segundos) |
| Latência write | maior (espera DB) | menor (só cache) |
| Risco perda de dado | baixo | alto se cache cair antes do flush |
| Conflation | não | sim (consolidar N writes em 1) |
| Caso Master Síndico | ABAC, sessão, auditoria, billing | feed, métricas de engajamento, scores de ranking |

**Regra prática**: se o dado perdido causa violação de segurança, LGPD, ou disputa financeira → write-through. Se é métrica aproximável ou cache de UI → write-behind ok.

---

## 7. Direct/DM infra — útil pelo **contra**-exemplo (Connect Me NÃO é chat)

### O que Meta faz (de `source-09`)

Messenger/Instagram Direct em 2023 rolou E2EE default usando Signal Protocol + **Labyrinth Protocol** próprio pra permitir histórico server-side cifrado (diferente do WhatsApp que é device-centric). Multi-device via username/password, OHAI + Anonymous Credentials pra features compatíveis com E2EE. Reescreveu "quase todo" o codebase de messaging do zero.

### Aplicabilidade ao Master Síndico

**Connect Me é unidirecional** (síndico procura empresa, contata, fecha — não há thread contínuo). **Não é chat.** Reforço dessa decisão:

1. E2EE messaging é **muito caro** pra construir corretamente. Meta gastou anos. Não é commodity, não existe drop-in.
2. Armazenar histórico de chat vira problema de LGPD (retenção, direito de esquecimento, export, subpoena).
3. Multi-device sync com E2EE é pesadelo operacional.
4. Moderação em E2EE é quase impossível (não dá pra ver o conteúdo), o que conflita com a necessidade de moderação do produto.

**O que o Connect Me precisa**:
- **Canal de contato estruturado** (formulário tipado: serviço, data desejada, descrição, orçamento esperado). Não free-text chat.
- **Notificações assíncronas** (e-mail/push) quando empresa responde — também estruturadas.
- **Evolução do contato virando contrato** (upload de proposta, aceite, pagamento via integração futura). Isso é state machine, não chat.
- **Observação importante**: se no futuro surgir demanda de "chat" (usuários pedirão), tratar como **nova feature** com ADR explícita de trade-offs, não como evolução natural do Connect Me. Considerar delegar a solução pronta (Intercom, Twilio Conversations) antes de construir.

---

## 8. GraphQL — não adotar no M1/M2

### O que Meta faz (de `source-06`)

GraphQL nasceu no Facebook em 2012 (Lee Byron, Dan Schafer, Nick Schrock) pra resolver over/under-fetching em mobile News Feed. Open-sourced em 2015. Instagram em si usa **Django + REST**, não GraphQL (as apps Meta pais usam GraphQL, Instagram é stack separada).

### Aplicabilidade ao Master Síndico

**Decisão sugerida**: **REST + OpenAPI 3.1** no M1/M2 (já canonizado em §3 do CLAUDE.md raiz). Motivos:

- Contrato OpenAPI é pré-requisito inegociável no projeto (§11 dos padrões herdados).
- Volume de surfaces/clientes no M1 é pequeno — web admin + mobile síndico. Não há N clientes heterogêneos pedindo shapes diferentes (que é o problema que GraphQL resolve).
- Cache HTTP (CDN, browser) funciona bem com REST GET. Com GraphQL (POST único) não.
- Curva de aprendizado + tooling + segurança (N+1, complexidade de query) custa mais do que resolve no M1.

**Quando reconsiderar**: M3+ se surgirem 3+ clientes com shapes muito diferentes (web admin, síndico mobile, morador mobile, API parceiros, embed em sites de condomínio). Nesse momento avaliar GraphQL BFF pontual, não troca total.

---

## 9. Django monolito → lições de scaling (de `source-05` e contexto)

Instagram server ainda é monolito Django, milhões de LOC, centenas de engenheiros, deploys a cada ~7 min, ~100 deploys/dia em produção. Mantém usando **static analysis + codemods (LibCST open-source)** ao invés de quebrar em microsserviços.

### Lições aplicáveis ao Master Síndico

Mesmo sendo Go e não Python, os princípios valem:

1. **Monolito bem-modularizado escala muito** — Master Síndico começa como modular monolith Go (multi-produto com bounded contexts internos). Microsserviços só quando dor real justificar.
2. **Automatizar refactoring em larga escala** — codemods/linters custom. Em Go: `gofmt`, `golangci-lint`, `gopls`, e scripts `ast`-based pra mudanças massivas. Isso protege de drift quando o codebase crescer.
3. **Deploy contínuo com confiança** — precisa de CI rápido + testes determinísticos + rollback instantâneo. Preparar pipeline desde M1 mesmo com 1 deploy/dia.
4. **Não quebrar em microsserviços sem necessidade** — §14 (YAGNI) do CLAUDE.md ecoa isso.

---

## Referências cruzadas

- ADRs a criar (sugestão):
  - `ADR-0001-ranking-connect-me-deterministico-m1.md`
  - `ADR-0002-write-through-abac-write-behind-feed.md`
  - `ADR-0003-connect-me-nao-e-chat.md`
  - `ADR-0004-moderacao-manual-m1-ml-m3plus.md`
  - `ADR-0005-rest-openapi-vs-graphql.md`
  - `ADR-0006-multi-tenant-sharding-prep-regional.md`

- Vizinhos:
  - [[../linkedin/_destilado]] — marketplace reputacional (Connect Me tem mais paralelo com LinkedIn do que Instagram)
  - [[../tiktok/_destilado]] — feed de vídeos curtos (complementar pro feed do condomínio)
  - [[../beyond-corp/_destilado]] — identity-centric, ABAC (reforça §3A)
  - [[../google-arch/_destilado]] — Spanner/TrueTime (alternativa a TAO se precisarmos de strong consistency cross-region)

## Fontes (detalhe em source-NN.md)

- `source-01.md` — Scaling Instagram Explore Recommendations (Meta Eng, 2023-08-09)
- `source-02.md` — Journey to 1000 models (Meta Eng, 2025-05-21)
- `source-03.md` — Instagram Stories ephemeral architecture (sistematizado)
- `source-04.md` — TAO: Power of the Graph (Meta Eng, 2013-06-25)
- `source-05.md` — Akkio data placement (Meta Eng, 2018-10-08)
- `source-06.md` — GraphQL: A Data Query Language (Meta Eng, 2015-09-14)
- `source-07.md` — Powered by AI: Instagram's Explore (Instagram Eng, 2019 aprox)
- `source-08.md` — Instagram AI content moderation (Meta/Instagram help + análise)
- `source-09.md` — Messenger E2EE default rollout (Meta Eng, 2023-12-06)
- `source-10.md` — Fighting Spam with Haskell / Sigma (Meta Eng, 2015-06-26)
