---
title: Destilado — LinkedIn Engineering → Master Síndico
type: note
tags: [research, linkedin, destilado, master-sindico, v2, kafka, search, trust-safety, reputation]
source: engineering.linkedin.com + arXiv + ByteByteGo
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-destilado, research-linkedin-sintese]
---

# Destilado — LinkedIn Engineering aplicado ao Master Síndico

Síntese das lições de arquitetura LinkedIn (feed, graph, Espresso, Kafka, Trust & Safety, skills/endorsements, PYMK, search, frontend) com recorte estrito **do que se aplica ao Master Síndico** e do que **NÃO se aplica** (para evitar cargo-cult).

Contexto do master-sindico:
- SaaS de governança condominial BR (primeiro cliente M1 2026-05-08)
- Sub-produtos: Banco de Talentos (vídeo-currículo morador), Connect Me (marketplace empresa↔síndico), Reputação Bronze→Diamante
- Stack: Go backend, NATS JetStream (event bus "quando necessário"), OpenSearch (busca), Postgres (fonte de verdade), Flutter app, Next.js web
- Princípio forte: determinístico > ML-heavy enquanto houver ambiguidade jurídica/escala modesta

Fontes mapeadas em `[[source-01]]` … `[[source-10]]`.

---

## 1. Kafka / log unificado → NATS JetStream "quando necessário"

### O que LinkedIn fez ([[source-01]])
- Problema original: M produtores × N consumidores = O(N²) de pipelines custom entre Search, Voldemort, Espresso, Hadoop.
- Solução: log central append-only particionado, com totalmente-ordenado por partição (sem ordem global).
- Kafka virou "a espinha dorsal de integração" e só depois virou produto.
- Jay Kreps formaliza em "The Log" (2013) três usos do log: integração de dados, stream processing, state machine replication.

### Aplicável ao Master Síndico
- **NÃO adotar Kafka** em M1/M2. Escala e complexidade não justificam.
- NATS JetStream atende perfeitamente **quando** surgir a condição de M×N do LinkedIn — ou seja, quando tivermos **≥ 3 módulos emitindo eventos consumidos por ≥ 3 outros módulos** (threshold deliberado).
- Em M1 (governança + billing) o único event-bus real é: `AssemblyEvent → NotificationModule`. Isso é 1×1 — basta **outbox pattern sobre Postgres** + worker que dispara HTTP interno, sem broker.
- Critério objetivo para ligar NATS JetStream: (i) ≥ 3 módulos produtores, ou (ii) consumidor precisa replay histórico (auditoria LGPD), ou (iii) latência de outbox-polling vira gargalo (> p95 5s entre produção e consumo).
- **Defender explicitamente** em ADR quando escrever NATS: referenciar "The Log" + número de produtores/consumidores reais.

### Anti-padrão a evitar
- Adotar Kafka/NATS "porque é moderno" sem ≥ 3 produtores × ≥ 3 consumidores reais. LinkedIn construiu Kafka **em 2010 com 100M+ membros**, não em dia 1.
- Virar "event-sourcing everywhere" — LinkedIn usa log principalmente para **integração**, não para modelar agregados de domínio. Agregados master-sindico (Condominio, Assembleia, Contrato) continuam CRUD clássico em Postgres com UoW; eventos são **efeitos colaterais publicados via outbox**, não source of truth.

---

## 2. Social graph / Espresso → NÃO aplicável em M1/M2

### O que LinkedIn fez ([[source-02]], [[source-04]])
- Espresso: NoSQL documento distribuído, Avro + MySQL/InnoDB pluggable, change capture via Databus, substituiu Oracle quando o grafo passou de centenas de milhões.
- Grafo real (conexões member↔member) construído **em cima** do Espresso via materialized views + um serviço de grafo dedicado (LIquid, mais recente).
- Escala: milhões de registros/s, centenas de TB.

### Aplicável ao Master Síndico
- **NÃO em M1/M2**. Grafo síndico↔morador↔empresa↔condomínio é raso (< 1M edges mesmo no cenário otimista de 2030) e modelável em Postgres com tabelas de junção + índices parciais.
- **Considerar em M3+** se aparecer caso de uso tipo "síndicos que gerenciam condomínios próximos ao seu" (≥ 3 hops) ou "empresas cujos técnicos atenderam condomínios de síndicos que você conhece" (recomendação cross-tenant).
- Mesmo em M3, começar com queries SQL recursivas (`WITH RECURSIVE` no Postgres) antes de pensar em graph DB dedicado. LinkedIn só migrou quando RDBMS monolítico **não conseguiu** escalar — é o critério honesto.

### Anti-padrão a evitar
- Adotar Neo4j/Dgraph "porque tem relacionamento" sem medir que a query recursiva em Postgres virou gargalo.
- Modelar Condomínio↔Unidade↔Morador como "grafo" quando é árvore com 3-4 níveis fixos — é só FK bem indexada.

---

## 3. Busca Voyager/Galene → OpenSearch já é stack; lições aplicam

### O que LinkedIn fez ([[source-03]])
- Galene: camada Lucene + federação + query understanding + ML ranking separado do document fetching.
- Arquitetura em camadas: frontend → federator (scatter-gather) → mid-tier (ML ranking) → backend indexes.
- Parity check framework (comparar legado vs novo antes de A/B test) e rollout incremental (5% → ramp) são lições universais.
- Galene ingesta signals via Kafka (ex: skill assessments passadas alimentam ranking de recruiter search).

### Aplicável ao Master Síndico
- OpenSearch é **nosso equivalente ao Lucene/Galene core** — mesma família, decisão já tomada (vide stack do projeto).
- **Lições diretas**:
  1. **Separar document fetching de ranking**: primeira página do OpenSearch retorna candidatos, re-ranking determinístico aplica boosting (reputação, proximidade geográfica, match de skill). Evita acoplar relevance logic ao índice.
  2. **Query understanding antes do search**: normalizar "encanador" ↔ "hidráulico" ↔ "bombeiro hidráulico" via sinônimos curados (não ML) no M1. Termo técnico BR.
  3. **Boosting por perfil do buscador**: síndico em SP buscando "elétrica" prioriza empresas SP > GO. Regra simples no query-time, sem ML.
  4. **Federator pattern faz sentido** quando tivermos ≥ 2 índices distintos (ex: empresas + moradores-talentos + assembleias-passadas). M1 só tem empresas+talentos, então pode ser um único multi-index search; se virar 3+ verticais, vale um federator gateway.
- **Skill assessment feed** (LinkedIn alimenta índice via Kafka): para master-sindico, mesmo padrão — quando morador ganha badge de skill via Banco de Talentos, outbox dispara update no OpenSearch (não reindex full).

### Anti-padrão a evitar
- ML ranking em M1. LinkedIn tem 1B+ usuários; nós teremos 1-10k no primeiro ano. Sinais escassos → modelo vira ruído. Ranking determinístico com boost weights auditáveis é superior até ter **muito mais dado**.
- Índice único gigante. Separar por vertical desde M1: `idx-empresas`, `idx-talentos`, `idx-posts` (quando houver conteúdo).

---

## 4. Trust & Safety → master-sindico tem "harassment report" Sprint 4; lições anti-fake parciais

### O que LinkedIn fez ([[source-05]], [[source-06]])
- Escala: 4M transações/s de abuse detection, 5B counters transientes, 200K QPS writes / 3M QPS reads só em counters.
- Arquitetura:
  - **Edge layer** (proteções básicas, body-based)
  - **Unified content filtering library** reutilizável por vários serviços
  - **Custom models** para endpoints de alto risco (registro, login, pagamento, criação de conteúdo)
  - **Throttling self-serve** (velocity/frequency counters)
- Para perfis inapropriados: **CNN text classifier** treinado em labels binárias (removido vs ativo). Mitigação de viés (blocklist antigo) via rotulagem manual de falsos-positivos.
- Graceful degradation: se scoring falha, (i) falha transação, (ii) re-score async, ou (iii) valor default — decidido por endpoint.

### Aplicável ao Master Síndico
- **Harassment report (Sprint 4) já previsto** — ampliar o escopo pra incluir:
  1. **Counter-based velocity detection**: morador X postou 20 reclamações em 1h → rate limit suave (silent throttling) + flag pra moderação. Postgres + Redis contadores bastam; não precisa Pinot/Samza.
  2. **Unified content filtering**: single library interna (`internal/safety/content`) usada por AssemblyComment, HarassmentReport, PostBody, ConnectMeMessage. Mesma interface, mesmo log de auditoria.
  3. **High-risk endpoints** específicos: registro de empresa (Connect Me), criação de conta síndico, aprovação de contrato. Cada um merece regra dedicada + rate limit mais apertado.
- **Anti-fake para empresa (Connect Me)**:
  - CNPJ + validação na Receita via integração (já previsto) = parcial. LinkedIn aprende que **validação documental sozinha não basta** — bots cadastram empresas reais mortas.
  - Faltam mecanismos adicionais inspirados em LinkedIn:
    - (a) **Device fingerprint + IP velocity**: N cadastros do mesmo device/IP em janela curta = captcha forte + revisão manual.
    - (b) **Behavioral signals pós-cadastro**: empresa recém-criada que manda 50 propostas em 1 dia = shadow-ban até revisão.
    - (c) **Proof-of-work leve** no cadastro (HMAC time-puzzle) pra tornar automação cara sem friccionar humano.
  - Registrar como **A-### em AUDIT** se não estiver no escopo atual.
- **Convolutional classifier** do LinkedIn para perfil inapropriado: **NÃO replicar em M1**. Palavras-chave + denúncia humana + moderador on-call é mais honesto em escala pequena. Migrar pra ML só quando o volume de denúncias **sobrecarregar moderação manual** (threshold concreto: > 100 denúncias/semana não resolvidas em 24h).

### Anti-padrão a evitar
- "Fraud detection ML" antes de ter denúncias reais suficientes pra treinar. Em master-sindico o universo é tão menor que manual+regras supera ML durante anos.
- Confundir "moderação" com "censura". Toda ação automática precisa log + recurso humano, tanto pra compliance LGPD quanto pra direito de ampla defesa do síndico/morador.

---

## 5. Reputação Bronze→Diamante → determinística, LinkedIn é ML-heavy

### O que LinkedIn fez ([[source-07]], [[source-08]])
- Feed e PYMK ranking usam LLM + Generative Recommender + transformer com 1000+ interações históricas, GPU serving com custom CUDA kernels.
- Escala justifica: 1.3B usuários, milhões de candidatos por request, dezenas de features correlacionadas.
- Mesmo assim, LinkedIn **mantém sinais explícitos e auditáveis** no feed: profile identity (skills, industry, geography), activity recentes, hashtags seguidas. Ranking é ML, features são interpretáveis.

### Aplicável ao Master Síndico
- **NÃO migrar pra ML em M1/M2 por princípio**. Reputação Bronze→Diamante deve ser:
  - **Determinística** (fórmula publicada)
  - **Auditável** (toda subida/descida tem transaction log)
  - **Reprodutível** (mesmo input → mesmo output)
  - **Explicável** ao síndico/empresa ("você perdeu 5 pontos por X contratos atrasados")
- Fórmula vive em `internal/reputation/score.go` com pesos versionados (ADR sempre que mudar). Contraste direto com LinkedIn que **não pode** explicar "por que você apareceu no topo do feed" — nossa vantagem competitiva jurídica em BR é justamente **poder explicar**.
- Lição aproveitável do feed LinkedIn: **features interpretáveis por trás do score** — mesmo quando LinkedIn vira ML, identity/activity/connections continuam sendo as dimensões. Nossa reputação usa mesmas famílias: (i) histórico contratual, (ii) avaliações recebidas, (iii) atividade na plataforma, (iv) selos/verificações. Só a função de combinação muda (determinística vs ML).
- **M3+**: se houver pressão pra "ranking recomendado" no Connect Me, estudar two-tower neural do LinkedIn PYMK ([[source-09]]) — mas ainda assim **reputação Bronze→Diamante permanece determinística**; o ranking de busca seria uma camada separada, re-rankeando resultados já filtrados pela reputação determinística.

### Anti-padrão a evitar
- "Reputação ML" com modelo caixa-preta — vira buraco negro regulatório (LGPD art. 20 exige explicabilidade de decisões automatizadas).
- Múltiplas fórmulas concorrentes (A/B test de reputação) — só gera desigualdade entre empresas sob mesma plataforma. A/B test é adequado em feed, não em reputação contratual.

---

## 6. Skills taxonomy → aplicável ao Banco de Talentos morador

### O que LinkedIn fez ([[source-08]])
- **Skills Graph**: ~39k skills, 374k aliases, 200k+ conexões entre skills (desde 2021, +35% em 2 anos).
- Taxonomy mantida por **taxonomistas humanos + ML** — dual approach.
- **Skill Assessments**:
  - Questões num Espresso + REST via rest.li
  - Rasch Model pra calibrar dificuldade
  - 3 estágios: Draft → Limited Ramp (sem pontuar) → Full Ramp
  - Adaptive testing (acertou → próxima mais difícil)
  - Anti-cheat: retakes limitados, copy disabled, timer por questão
  - Badge aparece se percentil ≥ 70
  - Feed no Galene via Kafka → recruiter search ranking

### Aplicável ao Master Síndico (Banco de Talentos morador)
- **Taxonomy curada** faz sentido mesmo em M1. Universo de habilidades domésticas BR é muito menor que LinkedIn (estimativa: 200-500 skills: "elétrica residencial", "jardinagem", "cuidador idoso", "confeitaria"). Curar manualmente + permitir sugestão com revisão > tentar auto-extract.
- **Aliases**: "eletricista" / "eletricidade residencial" / "instalação elétrica" → mesmo skill_id. Evita fragmentação.
- **Skill Assessments**: provavelmente **fora de escopo pra M1** (sub-produto Banco de Talentos ainda está em roadmap).
  - Quando entrar: **não replicar** Rasch Model completo. Começar com banco estático de questões + time-limit + percentil simples.
  - **Anti-cheat barato**: timer + disable-copy + limite de tentativas cobre 80%. Adaptive testing é M3+.
- **Integração com busca** (quando morador ganha badge, OpenSearch reflete): usar **outbox → OpenSearch writer**, como LinkedIn usa Kafka → Galene. Padrão direto.
- **Verificação > Assessment**: no contexto BR, badge via assessment técnico tem valor limitado. Complementar (ou priorizar) **verificações documentais**: CREA para eletricista profissional, certificação Senac para cuidador, etc. LinkedIn não tem esse universo; nós temos.

### Anti-padrão a evitar
- Auto-generated skills via NLP em CV. LinkedIn usa pra scale; master-sindico tem universo limitado e **confiança importa mais que cobertura**.
- Assessment que não mapeia pra contratação real. Se síndico não pode filtrar por badge no search, badge não vale nada.

---

## 7. Endorsements → análogo às avaliações pós-contrato; fraude é o problema

### O que LinkedIn fez ([[source-10]])
- Endorsements originalmente em SQL monolítico (10B rows), indexação `recipient_id → skill → endorser_id`. Buscar "tudo que Alice endossou" exigia full table scan.
- Migração: GraphDB como serving + SQL como source of truth. Emitem eventos via Kafka, Samza processa.
- **12 features top** (dentre 80 candidatas) distinguem endorsements valiosos. Definição de qualidade: "endosso feito por conexão que conhece a pessoa **e** o skill".
- ML suggestions separadas em dois pipelines: Endorsable Skills + Suggested Endorsements, ambos Hadoop batch.

### Aplicável ao Master Síndico (avaliações pós-contrato)
- **Análogo direto**: síndico avalia empresa pós-serviço (estrelas + tags + texto). Empresa **deveria** poder avaliar síndico também (pagamento em dia, comunicação, escopo claro) — **bilateral**.
- **Fraude é o risco crítico**:
  - **Self-endorsement** via contas falsas (síndico cria conta fake pra se avaliar 5 estrelas) → exige proof-of-work: avaliação só liberada se **houve contrato registrado + pagamento processado + prazo de execução cumprido**. Ou seja, avaliação é evento derivado de contrato, não input livre.
  - **Conluio** (empresa A avalia empresa B que avalia A): detectar via **grafo de quem-avalia-quem**. Se o grafo mostra ciclos densos entre poucas empresas, suspeito. LinkedIn resolve isso em parte pelo "recipient 1st degree connection" — nossa versão: só avalia quem **contratou** (relação unidirecional empresa↔síndico).
  - **Retaliação** (empresa avalia mal síndico porque síndico avaliou mal ela): LinkedIn não tem esse caso. Nosso mitigador: avaliações **duplamente cegas até ambas submeterem** ou **até prazo de 14 dias** — igual Airbnb. Registrar ADR.
- **Feature ranking de avaliações** (LinkedIn filtra endorsements "de conexão que conhece a pessoa e o skill"): adaptação direta — priorizar **avaliações de contratos maiores** (valor, duração) + **avaliações textuais** sobre estrelas-sem-texto + **avaliadores com reputação consolidada**. Determinístico, não ML.

### Anti-padrão a evitar
- Permitir avaliação sem contrato registrado. Abre festival de fake reviews.
- Mostrar avaliador antes de ambos terem avaliado — causa retaliação.
- Deletar avaliações negativas por pedido do avaliado. Destrói confiança e atrai ação MP/PROCON.

---

## 8. Frontend: Ember (Pemberly) → modern (React/Next)

### O que LinkedIn fez ([[source-09]])
- **Pemberly**: Ember.js + Play mid-tier + BigPipe streaming + Glimmer2 renderer + lazy engines.
- Hoje migrando pra React (via embroider + github.com/linkedin/react-in-ember bridges). Migração **lenta** por tamanho da base.
- Lições de migração de framework: codemods pesados, coexistência prolongada (código velho + novo lado-a-lado), confusão de padrões pra devs novos.

### Aplicável ao Master Síndico
- **Next.js 15 + Flutter** já é a decisão. Não há migração de framework pendente.
- **Lição transferível**: quando mudar padrão (ex: Server Components → Server Actions; Riverpod → hooks customizados), **codemods antes de mandato humano**. Automatizar o refactor mecânico, deixar revisão humana pro que tem semântica.
- **BigPipe-like pattern** (streaming HTML progressivo): Next.js RSC + Suspense cobre nativamente. Referência histórica boa pra entender por que SSR streaming importa em páginas densas (dashboard de síndico com 8+ widgets).
- **Ember lazy engines** → análogo moderno: Next.js route groups + dynamic imports. Feed de posts do condomínio carrega, analytics sidebar carrega depois.

### Anti-padrão a evitar
- Migração big-bang de framework. LinkedIn aprendeu que coexistência é o caminho, mas **só começou depois que a dor era insuportável**. No master-sindico, **não fazer migração framework em M1/M2**.

---

## 9. People You May Know → fora de escopo direto; ideia lateral

### O que LinkedIn fez ([[source-09]])
- Candidate generation em 3 famílias: graph walks (N-hop, Personalized PageRank), two-tower neural embeddings, heurísticas.
- Treinamento: **impressions como positivas** (não invites), 3 tipos de negative sampling.

### Aplicável ao Master Síndico
- **Direto em M1/M2: nada**. Não temos rede social interna onde "síndicos que você talvez conheça" faça sentido.
- **Ideia lateral pra Connect Me M3+**: "empresas que atenderam condomínios similares ao seu" — PPR em grafo condomínio↔empresa faz sentido se tivermos ≥ 10k empresas e ≥ 1k condomínios. Ranking: features interpretáveis (cidade, tipo de serviço, reputação atual).
- **Ainda assim começar com heurísticas** (LinkedIn usa como 3ª família), não com neural embeddings. "Empresas bem avaliadas no seu bairro" resolve 80% do caso.

---

## 10. Resumo decisões (D-### candidatos)

Decisões a registrar em `STATE.md` após revisão dual-check:

- **D-LKD-001**: NATS JetStream só quando ≥ 3 produtores × ≥ 3 consumidores reais, com outbox sobre Postgres como default em M1.
- **D-LKD-002**: Reputação Bronze→Diamante permanece **determinística e auditável** por toda vida do produto; ML é para ranking de busca, nunca para reputação contratual.
- **D-LKD-003**: Unified content filtering library (`internal/safety/content`) adotada pra harassment, assembly comment, connect me message, post body.
- **D-LKD-004**: Avaliações pós-contrato são **dupla-cegas até ambas submeterem ou 14 dias** (padrão Airbnb), bilaterais síndico↔empresa.
- **D-LKD-005**: Separar document fetching de ranking no OpenSearch — re-rank determinístico em camada aplicação.
- **D-LKD-006**: Anti-fake de empresa (Connect Me) além de CNPJ Receita: device fingerprint + IP velocity + behavioral post-signup. Abrir A-### se fora do escopo M1.
- **D-LKD-007**: Skills taxonomy curada manualmente com aliases, universo ~200-500 skills BR, sem NLP auto-extract em M1/M2.
- **D-LKD-008**: Grafo relacional fica em Postgres (`WITH RECURSIVE`) até provar gargalo. Graph DB dedicado é M3+.

---

## Vizinhos

- [[_moc]] — MOC da pasta linkedin
- [[../_moc]] — MOC geral de 13-research
- [[../../STATE]] — onde vão as D-LKD-###
- [[../../AUDIT]] — onde vão A-### de lacunas (ex: anti-fake além de CNPJ)
- [[../../02-architecture/patterns]] — onde entram padrões (outbox, unified content filter, document fetching vs ranking)
- [[source-01]]..[[source-10]] — fontes individuais
