---
title: "06-lms-academia — LMS / Academia Master Síndico (destilação Fase 8)"
type: spec
tags: [sub-produto, master-sindico, v2, fase-8, lms, academia, cursos, trilhas, certificados, forum, biblioteca, lives, content]
created: 2026-04-23
updated: 2026-04-23
status: destilado
dual_check: pending
bc: content
req: [29, 97, 98.1]
screens: [LMS1..LMS15]
---

# 06 — LMS / Academia Master Síndico

> **Sub-produto Master Síndico** (Fase 8 v2). Academia integrada ao BC `content`, com cursos, trilhas, quizzes, certificados, biblioteca, fórum e lives. **15 telas canônicas** (LMS1–LMS15). **Parceiro Vizinhança NÃO tem acesso** (INV-067). Lives podem ser criadas por **Admin MS + Empresa Pro** (D-097 Fase 11 — sobrescreve D-062/D-063/D-076 que restringiam a admin-only no MVP).
>
> Planos globais Stripe-style (`trial | base | plus | pro` — ADR-0032). Admin = `apps/admin` dedicada no monorepo web (D-134 Fase H revoga D-058/D-072). Agnosticismo de provedor (D-056) — Mux para vídeo-aulas via `IVideoProvider`.

---

## 1. Fontes consultadas

Destilação **exaustiva** arquivo-por-arquivo (não grep), com citação `arquivo (data)`:

### 1.1 Requirements canônicos (Fase 4/7 da v2 + legado)

- `vault/50-projects/master-sindico/specs/requirements/content.md` — **Req 29, 30, 31, 32, 97, 98.1** (2026-03-10, status `stable`). Fonte primária da matriz 5 áreas × 10 categorias × 3 trilhas, cotas de vídeo, 15 telas LMS1–LMS15, acesso por persona.
- `master-sindico-v2/04-requirements/functional/content.md` — **CNT-001..CNT-035** (2026-04-23). EARS specs; CNT-016 a CNT-025 são LMS; CNT-022 lock Lives admin-only.
- `vault/50-projects/master-sindico/specs/requirements/functional-matrix.md` (v1.0 stable 2026-04-22) — matriz acesso × persona × plano. Nota: a matriz original usa slugs N1/N2/N3/Plus/Pro; **ADR-0032 reescreve para tiers universais** — aplico essa conversão aqui.
- `vault/50-projects/master-sindico/specs/requirements/personas-and-plans.md` (v1.0 stable) — trial por persona, quotas, enum `plan_tier`.

### 1.2 Domínio (v2 destilada)

- `master-sindico-v2/01-domain/aggregates/Course.md` — entidade raiz, VOs, invariantes, factory, repo, eventos.
- `master-sindico-v2/01-domain/aggregates/Enrollment.md` — progresso %, quiz scores, `IsComplete`.
- `master-sindico-v2/01-domain/aggregates/Certificate.md` — serial único, verificação pública.
- `master-sindico-v2/01-domain/aggregates/LiveSession.md` (compartilhado com assembly) — referência para infraestrutura de Lives.
- `master-sindico-v2/01-domain/bounded-contexts.md` — contexto `content` descreve LMS como parte do BC.
- `master-sindico-v2/01-domain/domain-events.md` — **E-057..E-061** (CoursePublished, EnrollmentCreated, LessonCompleted, CourseCompleted, CertificateGenerated).
- `master-sindico-v2/01-domain/invariants.md` — **INV-066, INV-067, INV-068 (Certificate), INV-069, INV-070** (LMS/content). *(INV-037 canônico = Connect Me unidirecional; referências anteriores neste arquivo a INV-037 para Certificate eram erradas — corrigido por DT-009.)*

### 1.3 Cliente (Obsidian client-vault — destilado)

- `vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Domínio - Conteúdo.md` (v3.0, 2026-03-10) — Req 98.1 detalhado: 5 áreas, 10 categorias, 3 níveis, 3 trilhas, rotas de API, endpoints.
- `vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Mapa Academia.md` — canvas convertido: nodes/edges do mapa conceitual (Cursos/Treinamentos/Lives/Fórum/Biblioteca + Certificados + Trilhas).
- `vault/50-projects/master-sindico/client-vault/01 - Arquitetura de Sistema/System Architecture - Content.md` — descreve **AcademiaModule** (NOVO) com `Course, Lesson, Training, LiveEvent, Certificate, LearningPath, LibraryContent`.
- `vault/50-projects/master-sindico/client-vault/03 - Modelagem e Dados/Enums e Listas Mestre.md` (v3.0) — enums canônicos (CourseCategory 10, CourseLevel 3, LiveType 4, LiveAccessLevel 2, LibraryMaterialType 6, ForumCategory 7).

### 1.4 Jornada tela-a-tela (legado)

- `vault/50-projects/master-sindico/_archive/content-discovery-2026-03/modulo-academia.md` — **JORNADA LMS TELA-A-TELA** (LMS1–LMS15) originalmente convertida do `contents/MÓDULO ACADEMIA MASTER SÍNDICO.ini`. Mapeamento original difere do briefing desta task (v2 reordena): ver §9.

### 1.5 Arquitetura e decisões

- `master-sindico-v2/02-architecture/adr/0032-global-plans-stripe-style.md` (accepted 2026-04-23) — tiers universais `{trial, base, plus, pro}`.
- `master-sindico-v2/STATE.md` — D-056 (agnosticismo de provedor), D-058 (admin = role privilegiada), ~~D-062 (Lives admin-only no MVP)~~ **sobrescrita por D-097 Fase 11 (Lives = Admin MS + Empresa Pro; primeira live ainda é apresentação oficial pela Admin MS)**, D-066/D-067 (abolir N1/N2/N3).
- `master-sindico-v2/04-requirements/global.md` — **GR-029** (lock 90d em vídeo — aplica-se por extensão conceitual a conteúdo curso publicado → **RC-7** nas invariantes).

### 1.6 Código Go backend

- **Pendência**: não foi encontrado código Go do bounded context `content` em `backend/internal/modules/content/` dentro deste repositório de docs (`automation-agents`). O projeto `Development/backend/` (cliente João, fora deste vault) é a fonte real; aqui só specs e domínio. Módulo LMS marcado como **M3 thin** (infra + 3 trilhas piloto) em `ROADMAP.md` §M3. Dual-check pendente em sub-projeto `03-subprojects/backend/`.

---

## 2. Propósito

Academia digital **contextualizada ao universo condominial brasileiro**, oferecendo capacitação continuada nos 5 pilares temáticos:

1. **Direito condominial** — Código Civil art. 1.331–1.358-A, Convenção, Regimento Interno, LGPD aplicada a condomínios, legislação trabalhista (porteiros/zeladores), normas CLT/terceirização, ABNT NBR 5674 (manutenção predial), CAU/CREA.
2. **Finanças condominiais** — orçamento anual, previsão, fundo de reserva, inadimplência, taxa extra, prestação de contas, regime competência/caixa, relação com administradoras.
3. **Comunicação e gestão de conflitos** — mediação vizinhança (animais, ruído, vagas, pets, obras), comunicação não-violenta com moradores, convocação de assembleias, gestão de crise.
4. **Gestão técnico-operacional** — plano diretor, saúde ambiental (pragas, reservatórios, lixo), segurança predial, brigada de incêndio, manutenção preventiva vs corretiva, elevadores, fachada, gases.
5. **Governança e tecnologia** — boas práticas de síndico, ferramentas digitais, sustentabilidade (energia solar, captação de chuva), contratação consciente de fornecedores, uso do Master Síndico na rotina.

A Academia **integra cross-domain** com Governança Institucional (Req 23): ao registrar uma execução de serviço, a plataforma recomenda treinamentos relacionados (ex.: manutenção de bombas → treinamento hidráulico). Integra também com Reputação — certificados em cursos de "Relacionamento com síndicos" alimentam score de Empresa.

**Missão**: reduzir assimetria de informação que historicamente fez síndicos caírem em armadilhas contratuais, trabalhistas e técnicas — **conhecimento transforma gestão**.

---

## 3. Cinco áreas canônicas

| # | Área | Estrutura | Objetivo pedagógico |
|---|------|-----------|---------------------|
| 1 | **Cursos** | Curso → Módulo → Aula (vídeo) → Material complementar → Quiz por módulo → Certificado ao 100% | Aprendizado estruturado e longo (tipicamente 4–20h de carga) |
| 2 | **Treinamentos** | Peça curta e prática (vídeo único, sem módulos) | Capacitação operacional rápida (15–60 min): porteiros, segurança, primeiros socorros, atendimento morador, prevenção de pragas, emergência |
| 3 | **Lives** | Transmissão agendada + chat + Q&A + replay | Discussão ao vivo com especialistas; **Admin MS + Empresa Pro** podem criar (D-097 Fase 11 sobrescreve D-062/D-063 admin-only MVP); Agência via delegação de empresa Pro |
| 4 | **Fórum** | Tópico → Respostas → Curtidas → Marcar resolvido | Construção coletiva de conhecimento, 7 categorias temáticas |
| 5 | **Biblioteca** | Material avulso (PDF/EPUB/vídeo/link) por tipo e categoria | Consulta de referência (guias, manuais, ebooks, materiais técnicos) |

Fonte: `Domínio - Conteúdo.md` §Requisito 98.1; `content.md` Req 98.1 "5 Áreas".

---

## 4. Três trilhas piloto (M3)

A Academia M3 (Sprint 10+ / 2026-07-20 conforme `ROADMAP.md` §M3) entrega **infraestrutura LMS + 3 trilhas piloto** (uma por persona). M4+ expande para 50+ cursos via parceria **ABRACS**.

| Trilha | Persona-alvo | Cursos recomendados (ordenação sugerida) | Objetivo |
|--------|--------------|------------------------------------------|----------|
| **Trilha Síndico Iniciante → Intermediário → Profissional** | `syndic` (tier `plus`+) | Gestão condominial básica → Legislação condominial → Administração financeira → Mediação de conflitos → Manutenção predial avançada | Formar síndico do zero até nível profissional |
| **Trilha Empresa** | `enterprise` (tier `plus`+) | Boas práticas prestação condomínio → Segurança do trabalho → Saúde ambiental → Relacionamento com síndicos → Captação comercial ética | Padronizar qualidade de atendimento de prestadoras |
| **Trilha Morador** | `resident` pagante | Convivência condominial → Uso responsável de áreas comuns → Sustentabilidade no condomínio → Segurança residencial | Educar morador engajado |

**Regra**: trilha = **conjunto ordenado de cursos** (DAG linear). O avanço é livre (aluno pode pular), mas o sistema recomenda a ordem. Certificado de **trilha** (M4+) requer todos os cursos concluídos.

A pergunta "Síndico Iniciante/Intermediário/Profissional" da Fase 8 é atendida por **3 estágios da Trilha Síndico** (nível `basico`/`intermediario`/`avancado` dos cursos que a compõem), não por 3 trilhas separadas — alinhado com `CourseLevel 3` do enum canônico.

Fontes: `Domínio - Conteúdo.md` §"Trilhas de Aprendizado"; `Course.md` domain `CourseTrilha` enum; `content.md` §"3 Trilhas".

---

## 5. Personas-alvo

Sistema Zitadel roles em uso (`personas-and-plans.md`): `syndic | resident | enterprise | marketing | local_company | admin`.

| Persona | role | Acesso LMS | Observação |
|---------|------|------------|------------|
| **Síndico** (todos tiers) | `syndic` | ✅ Todas as 5 áreas | Certificado exige `tier ≥ plus` (antigo N2). Tier `trial`: visualizar, sem certificado até conversão |
| **Morador Pagante** | `resident` tier `plus`/`pro` | ✅ Todas as 5 áreas | Certificado habilitado |
| **Morador Base** | `resident` tier `base` | Parcial: Lives (pós-D-062: só replay público), Fórum, Biblioteca | **Sem** cursos completos, **sem** certificado |
| **Empresa Plus** | `enterprise` tier `plus` | ✅ Consumir cursos + certificado | Não cria cursos |
| **Empresa Pro** | `enterprise` tier `pro` | ✅ Consumir + **criar cursos certificados** | Produtora de conteúdo; moderação admin pré-publish |
| **Agência de Marketing** | `marketing` | ✅ Consumir (sem certificado) | Sem `membership` em tenant condômino; foco em produção de vídeo |
| **Admin Master Síndico** | `admin` | ✅ Total + **curador** | Cria cursos institucionais, modera fórum, agenda Lives, configura trilhas, aprova conteúdo de empresas |
| **Parceiro da Vizinhança** | `local_company` | ❌ **BLOQUEADO** | INV-067: ABAC nega + UI oculta botão Academia. Comércio de bairro não é foco pedagógico da plataforma |

Fontes: `functional-matrix.md` seção "Academia LMS"; `content.md` Req 98.1 §"Acesso por persona"; `personas-and-plans.md` §Enum user_role; **INV-067** em `invariants.md`.

---

## 6. Tiers × role — matriz de capacidade (reescrita ADR-0032)

Matriz **ortogonal** (role × tier) aplicada por ABAC engine. Colunas são tiers universais; linhas combinam feature + role aplicável.

| Feature LMS | Trial | Base | Plus | Pro | Roles aplicáveis |
|-------------|-------|------|------|-----|------------------|
| Ver catálogo cursos | ✅ | ✅ | ✅ | ✅ | syndic, resident, enterprise, marketing, admin |
| Matricular em curso (enroll) | ⏳ preview | ❌ | ✅ | ✅ | syndic, resident (pagante), enterprise, marketing |
| Assistir aulas completas | ⏳ | ❌ | ✅ | ✅ | syndic, resident, enterprise, marketing |
| Fazer quiz + aprovação | ⏳ | ❌ | ✅ | ✅ | syndic, resident, enterprise |
| **Receber certificado** | ❌ | ❌ | ✅ | ✅ | **syndic** (≥ plus), **resident** (pagante), **enterprise** (≥ plus). Nunca: marketing, local_company |
| **Criar curso certificado** | ❌ | ❌ | ❌ | ✅ | **enterprise** (pro), **admin** (bypass) |
| Ver replay Live | ✅ | ✅ | ✅ | ✅ | todos exceto `local_company` |
| Participar Live ao vivo | ⏳ | ✅ (abertas) | ✅ | ✅ | todos exceto `local_company`; lives `exclusivas` exigem tier |
| **Criar/agendar Live** | ❌ | ❌ | ❌ | ❌ | **somente `admin`** (D-062 MVP) |
| Ler tópicos do Fórum | ✅ | ✅ | ✅ | ✅ | syndic, resident (pagante+), enterprise, marketing |
| Comentar tópico | ⏳ | ✅ (pagante) | ✅ | ✅ | syndic, resident (pagante), enterprise, marketing |
| **Criar tópico** | ❌ | ❌ | ✅ | ✅ | **syndic**, **enterprise**, **admin** — moradores NÃO criam (só comentam) |
| Acessar Biblioteca (consulta livre) | ✅ | ✅ | ✅ | ✅ | todos exceto `local_company` |
| Download ebook/material | ⏳ | parcial | ✅ | ✅ | syndic, resident (pagante), enterprise |
| Acesso Academia (botão menu visível) | ✅ | ✅ | ✅ | ✅ | **todos exceto `local_company`** — ABAC INV-067 |

**Legenda**: ⏳ = trial (bloqueado até conversão); ✅ = permitido; ❌ = 403 `PLAN_REQUIRED` ou feature oculta.

Derivação (exemplo ABAC):

```
(role=local_company, *, resource=academy.*) → DENY (INV-067, hard)
(role=syndic, tier=plus, action=course.certificate.earn) → ALLOW
(role=resident, tier=base, action=course.lesson.watch) → DENY (PLAN_REQUIRED)
(role=enterprise, tier=pro, action=course.create) → ALLOW
(role=enterprise, tier=plus, action=course.create) → DENY (PLAN_REQUIRED)
(role=*, action=live.schedule) → DENY (salvo role=admin; D-062)
```

Fontes: ADR-0032 seção "ABAC engine"; `functional-matrix.md` linhas "Academia LMS"; CNT-020 + CNT-022 em `functional/content.md`.

---

## 7. Aggregates DDD (BC `content`)

Todos em `master-sindico-v2/01-domain/aggregates/`. DDD tático: entidades privadas, factories `NewX`/`ReconstructX`, repo interface no domain.

### 7.1 `Course` (raiz)

```
Course
├─ CourseID, title, description
├─ area: CourseArea            (cursos | treinamentos | lives | forum | biblioteca)
├─ category: CourseCategory    (10 valores — §8)
├─ trilha: *CourseTrilha       (sindico | empresa | morador)
├─ status: CourseStatus        (draft | published | archived)
├─ modules []CourseModule      (ordenadas)
│   └─ lessons []CourseLesson  (order, title, videoID?, textContent?, duration)
│   └─ quiz *Quiz              (questions[], passThreshold default 70)
├─ accessLevel: AccessLevel    (gratuito | pago)
├─ estimatedHours int
└─ publishedAt, createdAt, updatedAt
```

Métodos: `AddModule`, `RemoveModule` (só `draft`), `Publish` (`draft → published`), `Archive` (`published → archived`).

Invariantes do agregado:
- `publishedAt != NULL` ⇒ `status=published`
- `passThreshold ∈ [1,100]`, default 70
- Parceiro Vizinhança nunca instancia (INV-067 validado no caller)

### 7.2 `Enrollment` (raiz)

```
Enrollment
├─ EnrollmentID, userID, courseID
├─ progressPercent int [0..100]
├─ completedLessonIDs []LessonID
├─ quizScores map[ModuleID]int [0..100]
├─ startedAt, *completedAt, lastAccessAt
└─ UNIQUE (user_id, course_id)
```

Métodos: `CompleteLesson(lessonID, totalLessons)`, `SubmitQuizScore(moduleID, score)`, `IsComplete() bool`, `MarkComplete()`.

Invariante **INV-068**: `progressPercent == 100` dispara `Certificate` (idempotente).

### 7.3 `Certificate` (raiz)

```
Certificate
├─ CertificateID, serialNumber (UNIQUE, VO CertificateSerial)
├─ userID, courseID, enrollmentID
├─ pdfURL (MinIO / R2)
├─ issuedAt, *digitalSignedAt (Sprint 3+ MP 2.200-2), *validUntil (default evergreen)
```

URL pública de verificação: `/certificates/verify/{serial}` — sem auth. Imutável pós-issue.

### 7.4 `Quiz` (dentro de `CourseModule`)

Sub-entidade; não é raiz. Perguntas com `correct_index`, `weight`, `options`. Aprovação >= `passThreshold` (default 70%) marca módulo completo.

### 7.5 `ForumTopic` + `ForumPost` (raízes separadas)

Tabelas `forum_topics`, `forum_posts`, `forum_likes`, `forum_subscriptions` (`content.md` Req 98.1 notas). Categoria `ForumCategory` (7 valores). Moderação reativa via botão "Denunciar" (CNT-021).

**Nota**: agregados `ForumTopic` e `ForumPost` não estão formalmente destilados ainda em `01-domain/aggregates/` (gap — ver §16). Referência no contexto `content`.

### 7.6 `LibraryMaterial` (raiz)

PDF/EPUB/vídeo/link avulso. Campos: `type: LibraryMaterialType (6)`, `category`, `access_level`, `pdf_url`/`epub_url`/`video_id`/`external_url`, `author`, `description`. Tabela `digital_content` (CNT-025). Storage MinIO/R2.

**Gap**: aggregate formal `LibraryMaterial.md` ausente (ver §16).

### 7.7 `Live` (LMS) ≠ `LiveSession` (Assembly)

A `LiveSession` destilada em `01-domain/aggregates/LiveSession.md` pertence ao BC `assembly` (Livekit SFU). A **Live LMS** é infraestrutura distinta, via **Mux Live** ou **Cloudflare Stream** (pendência CNT-022: ADR research-loop). Campos: `title`, `scheduled_at`, `speaker`, `type: LiveType(4)`, `access_level: LiveAccessLevel(2)`, `replay_url`, `state: scheduled|live|ended|replay`. Admin é o único criador (D-062 MVP).

**Gap**: aggregate formal `Live` (LMS) ausente (ver §16).

Fontes: `Course.md`, `Enrollment.md`, `Certificate.md` em `master-sindico-v2/01-domain/aggregates/`; `System Architecture - Content.md` §AcademiaModule; CNT-016 a CNT-024 em `functional/content.md`.

---

## 8. Enums canônicos

Todos derivados de `Enums e Listas Mestre.md` v3.0 (2026-03-10).

### 8.1 `CourseCategory` (10 valores)

```
gestao_condominial
saude_ambiental
seguranca_predial
manutencao_predial
legislacao_condominial
mediacao_conflitos
tecnologia_condominios
sustentabilidade
administracao_financeira
outros
```

Obs.: `Enums e Listas Mestre.md` adiciona `convivencia_moradores` e `seguranca` (**Apenas Fórum**) → ver §8.4.

### 8.2 `CourseLevel` (3 valores)

```
basico | intermediario | avancado
```

### 8.3 `CourseAccessLevel` (2 valores)

```
gratuito | pago
```

Cursos `pago` direcionam para módulo de Billing (Stripe via `IPaymentGateway`) — LMS3 tela curso mostra CTA "Comprar".

### 8.4 `ForumCategory` (7 valores)

```
gestao_condominial
manutencao_predial
seguranca
saude_ambiental
tecnologia
convivencia_moradores
legislacao_condominial
```

### 8.5 `LibraryMaterialType` (6 valores)

```
videos | guias_tecnicos | manuais | livros_digitais | ebooks | materiais_tecnicos
```

### 8.6 `LiveType` (4 valores)

```
debates_condominiais
lancamento_solucoes
palestras_tecnicas
entrevistas_especialistas
```

Nota: `content.md` Req 98.1 legado citava também `aberta | exclusiva_plan | ao_vivo_com_replay | seminar_serie` (tipologia de acesso/formato). O **enum canônico** (`Enums e Listas Mestre.md`) é temático — os 4 acima. Acesso/formato vira combinação (`LiveType × LiveAccessLevel × replay_disponivel`).

### 8.7 `LiveAccessLevel` (2 valores)

```
abertas      — todos do LMS (exceto local_company)
exclusivas   — baseado em plano/convite (tier >= plus, ou whitelist admin)
```

### 8.8 `CourseTrilha` (3 valores)

```
trilha_sindico | trilha_empresa | trilha_morador
```

### 8.9 `CourseStatus` (3 valores) — DDD

```
draft | published | archived
```

### 8.10 `EnrollmentStatus` (3 valores) — DDD derivado

```
active | completed | cancelled
```

(Derivado de `Enrollment.completedAt != NULL` + flag `cancelled_at`; não é enum explícito no código mas governa state machine — §12.)

### 8.11 `QuizAttemptStatus` (3 valores)

```
in_progress | passed | failed
```

(`passed` se score >= `passThreshold`; `failed` permite retry — CNT-018.)

Fontes: `Enums e Listas Mestre.md` §"Academia LMS"; `Course.md` VO; `content.md` Req 98.1; `functional/content.md` CNT-016..CNT-024.

---

## 9. Telas LMS1–LMS15 (catálogo tela-a-tela)

> **Divergência documental observada**: o briefing desta task usa um mapeamento "lógico" das 15 telas (LMS1 Home, LMS5 Player, LMS7 Certificado, LMS8 Fórum, LMS10 Lives, LMS11–14 subtipos de Live). O arquivo original convertido do `.ini` (`_archive/content-discovery-2026-03/modulo-academia.md`) usa **outro mapeamento** (LMS1 Home, LMS2 Catálogo, LMS5 Certificado, LMS6–7 Treinamentos, LMS8–10 Lives+Replay, LMS11–13 Fórum, LMS14–15 Biblioteca).
>
> **Fonte de verdade**: adoto o mapeamento do **briefing da task** (ordenação lógica de jornada do usuário), anotando o original em cada tela quando diverge. Uniformizar depois em dual-check com o cliente (ver §16 gap G-05).

### LMS1 — Home Academia

- **Caminho**: Menu principal → "Academia Master Síndico"
- **Visibilidade do item de menu**: oculto para `local_company` (INV-067)
- **Mensagem institucional**: _"O conhecimento transforma gestão. A Academia Master Síndico reúne cursos, treinamentos e experiências para fortalecer o ecossistema condominial."_
- **Seções exibidas**: cards das 5 áreas (Cursos / Treinamentos / Lives / Fórum / Biblioteca) + bloco "Trilha recomendada para você" (depende de `user.role` → sugere Trilha Síndico / Empresa / Morador) + "Seu progresso" (KPIs rápidos)
- **Botões**: Explorar cursos · Explorar treinamentos · Ver agenda de lives · Participar do fórum · Acessar biblioteca
- **CTA de conversão** (se `tier=base` ou `trial`): banner "Desbloqueie certificação — Assine Plus"
- **Fonte**: `modulo-academia.md` TELA LMS1 (original confere)

### LMS2 — Trilhas

- **Caminho**: Home Academia → "Trilhas" (novo na v2; no legado esta tela era "Explorar cursos")
- **Elementos**: 3 cards grandes: `Trilha Síndico Iniciante → Intermediário → Profissional`, `Trilha Empresa`, `Trilha Morador`. Cada card mostra: cursos incluídos, progresso do usuário (%), tempo estimado total (h), nível
- **Filtros**: nenhum (tela conceitual) — clique abre jornada ordenada
- **Regra**: trilha é recomendação ordenada — usuário pode navegar em qualquer ordem, mas progress bar usa a sequência canônica
- **Fonte**: `Domínio - Conteúdo.md` §"Trilhas de Aprendizado"; `Mapa Academia.md` edges TRILHA_* → CURSOS/TREINA/LIVES

### LMS3 — Catálogo de Cursos

- **Caminho**: Home Academia → "Cursos" (ou vindo de trilha)
- **Mensagem**: _"Aprofunde seus conhecimentos com cursos criados para quem vive o universo condominial."_
- **Filtros disponíveis**:
  - `CourseCategory` (10 valores — §8.1)
  - `CourseLevel` (basico / intermediario / avancado)
  - `CourseAccessLevel` (gratuito / pago)
  - Trilha (Síndico / Empresa / Morador / todas)
  - Duração estimada
- **Busca full-text** via `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending dual-check, D-120 revoga D-067 PG tsvector — CNT-005): título, descrição, palavras-chave
- **Card do curso**: título · professor · carga horária · nível · tipo (gratuito/pago) · progresso se já matriculado · botão "Ver curso"
- **Fonte**: `modulo-academia.md` TELA LMS2 original; reposicionado como LMS3

### LMS4 — Detalhe do Curso

- **Caminho**: LMS3 Catálogo → clique no card
- **Informações exibidas**: título, professor (com perfil linkável), descrição longa, carga horária, número de aulas, conteúdo programático (módulos e lições), acesso (free/pago/plano mínimo), pré-requisitos, avaliações de alunos (quando M4+)
- **Botão principal**: `Inscrever-se` (gratuito) ou `Comprar e inscrever-se` (pago → fluxo Stripe)
- **Regra**: se `role=local_company` → 403; se `tier` insuficiente → banner "Plano requerido: Plus"
- **Observação**: cursos pagos entram no módulo Billing; trial mostra preview (descrição + 1ª aula livre)
- **Fonte**: `modulo-academia.md` TELA LMS3

### LMS5 — Player de Aula

- **Caminho**: LMS4 → "Inscrever-se" → LMS5 (primeira aula) OU continuação da próxima aula
- **Elementos**:
  - Player de vídeo (Mux HLS via `IVideoProvider` — CNT-010). Controles: play/pause, seek, fullscreen, qualidade adaptativa, legendas (M2+)
  - Sidebar: lista de módulos/aulas com ícones de progresso (✓ concluída, ● atual, ○ pendente)
  - Abaixo do player: resumo da aula (texto), material complementar (PDFs/links), botão **"Marcar aula como concluída"**, botão "Próxima aula"
- **Tracking**: CNT-011 — pulse frontend a cada 15s com `watch_percentage`; view conta se ≥ 70%
- **Ao marcar concluída**: dispara `LessonCompleted` (E-059). Se todas as aulas do módulo concluídas → libera quiz (LMS6)
- **Fonte**: `modulo-academia.md` TELA LMS4

### LMS6 — Quiz

- **Caminho**: LMS5 → concluir todas as aulas do módulo → `Ir para o Quiz`
- **Elementos**: lista de questões (`QuizQuestion` entity), opções múltipla escolha, contador "Questão X de N"
- **Submissão**: cálculo de score (peso por questão). Feedback imediato (CNT-018)
- **Aprovação**: score >= `passThreshold` (default 70%) → módulo marcado `passed` (`QuizAttemptStatus=passed`). Dispara avanço de `progressPercent` em `Enrollment`
- **Reprovação**: score < 70% → `failed`; botão "Tentar novamente" (retries ilimitados; M4+ pode adicionar cooldown)
- **Ao passar último módulo**: `progressPercent = 100` → LMS7
- **Fonte**: `content.md` Req 98.1 + CNT-018; `Course.md` Quiz sub-entity

### LMS7 — Certificado gerado

- **Caminho**: LMS6 → aprovação do último módulo (100% curso) OU última aula se curso sem quiz
- **Mensagem**: _"Parabéns por concluir este curso da Academia Master Síndico."_
- **Geração**: dispara `CourseCompleted` (E-060) → handler `GenerateCertificate` → cria `Certificate` com `serialNumber` único + PDF async (job) com QR code → emite `CertificateGenerated` (E-061) → email ao aluno com link
- **Conteúdo do PDF**: nome, nome do curso, carga horária, data, `serialNumber`, QR code apontando para `/certificates/verify/{serial}` (público, sem auth)
- **Botões**: `Baixar certificado` (PDF) · `Compartilhar certificado` (LinkedIn, link público) · `Ver outros cursos`
- **Assinatura digital BR**: Sprint 3+ (MP 2.200-2 ICP-Brasil); M4+ pleno — pendência CNT-019 (ADR provedor)
- **Regra elegibilidade**: quem recebe certificado (derivado de matriz §6):
  - Morador Pagante ✓
  - Síndico tier `plus`/`pro` ✓ (antes "N2+")
  - Empresa tier `plus`/`pro` ✓
  - Nunca: `local_company`, `marketing`, `syndic/trial`, `resident/base`
- **Fonte**: `modulo-academia.md` TELA LMS5 (original); `Certificate.md` aggregate; CNT-019

### LMS8 — Fórum

- **Caminho**: Home Academia → "Fórum"
- **Mensagem**: _"Compartilhe experiências, faça perguntas e construa conhecimento coletivo."_
- **Elementos**: abas por `ForumCategory` (7), lista de tópicos com título, autor, última resposta, curtidas, status (aberto/resolvido), busca full-text (OpenSearch)
- **Botão**: `Criar tópico` — só visível para `role ∈ {syndic, enterprise, admin}` (moradores **não** criam; matriz §6)
- **Ações**: `responder_topico`, `curtir_resposta`, `marcar_como_resolvido` (autor do tópico ou admin)
- **Moderação**: reativa via botão "Denunciar" → fila admin (CNT-027)
- **Notificação**: sub em tópico → email/push quando nova resposta (CNT-028 canal oficial limitado)
- **Fonte**: `modulo-academia.md` TELA LMS11 (original); `Domínio - Conteúdo.md` §"Área 4: Fórum"

### LMS9 — Biblioteca

- **Caminho**: Home Academia → "Biblioteca"
- **Mensagem**: _"Materiais de consulta para aprofundar conhecimentos sobre o universo condominial."_
- **Filtros**:
  - Tipo de material (`LibraryMaterialType` — 6 valores)
  - Categoria (reaproveita `CourseCategory` 10)
  - Autor
- **Lista**: cards com título, autor, tipo, data, botão "Ver/Baixar"
- **Página do material (detalhe)**: título, descrição, autor, tipo, tamanho, botões **Baixar material** (PDF/EPUB via signed URL MinIO/R2 TTL 24h) e **Ler online** (viewer embutido)
- **Paywall**: Base parcial (alguns materiais gratuitos); Plus/Pro integral (CNT-025; INV-069)
- **Exemplos** (`Domínio - Conteúdo.md`): Guia Prevenção Pragas, Manual Manutenção Predial, Guia Convivência Condominial, Manual Síndico Iniciante
- **Fonte**: `modulo-academia.md` TELA LMS14+LMS15 (originais combinadas aqui em LMS9)

### LMS10 — Lives (agenda + acesso geral)

- **Caminho**: Home Academia → "Lives"
- **Mensagem**: _"Encontros ao vivo com especialistas do universo condominial."_
- **Lista**: próximas lives (data, tema, palestrante, `LiveType`, `LiveAccessLevel`), passadas (replay disponível por 30d — `content.md` Req 98.1)
- **Ações**:
  - `Participar` (quando `live_state=live` — abre sala LMS11–14 por tipo; ou replay se `ended`)
  - `Lembrar-me` (subscription → notificação push/email pré-evento)
- **Regra D-097 Fase 11** (sobrescreve D-062): criação/agendamento por **`role=admin`** (Admin MS) OU **`(role=enterprise, tier=pro)`**. Agência (`role=marketing`) via delegação de empresa Pro (grant token ADR-0032). Primeira live = apresentação da plataforma Master Síndico (kickoff oficial) pela Admin MS. Após isso, Empresa Pro pode criar regularmente quando feature gate estiver implementado.
- **Acesso**:
  - `local_company`: 403 (INV-067)
  - `trial`: ⏳
  - `base` morador: só lives `abertas` (replay ok)
  - `plus`/`pro` (todos roles): `abertas` + `exclusivas`
- **Fonte**: `modulo-academia.md` TELAS LMS8–LMS10 (originais); CNT-022; D-062

### LMS11 — Sala de Live: Lançamento de Solução

- **Caminho**: LMS10 → clique em live ao vivo cujo `LiveType = lancamento_solucoes`
- **Elementos**: vídeo ao vivo (Mux Live/Cloudflare Stream RTMP), chat (moderado — admin toggle), campo de pergunta ao palestrante, CTA "Saber mais" linkando empresa/produto
- **Contexto**: empresas (Pro) apresentam soluções inovadoras; formato pitch 20–30 min + Q&A
- **Observação**: ao encerrar, grava replay automaticamente (CNT-023; 30d disponível)
- **Fonte**: `Enums e Listas Mestre.md` LiveType; briefing task §9

### LMS12 — Sala de Live: Debate Condominial

- **Caminho**: LMS10 → live ao vivo `LiveType = debates_condominiais`
- **Elementos**: transmissão multi-participante (round-table), chat, enquetes rápidas em tela, Q&A
- **Contexto**: temas polêmicos (ex.: autogestão vs administradora, cotas de estacionamento, animais em áreas comuns), tipicamente admin + 2–4 convidados
- **Fonte**: briefing task §9; `LiveType.debates_condominiais`

### LMS13 — Sala de Live: Palestra Técnica

- **Caminho**: LMS10 → live `LiveType = palestras_tecnicas`
- **Elementos**: palestrante único (slides compartilhados + câmera), chat, Q&A ao final
- **Contexto**: aprofundamento técnico-normativo (NR-10, ABNT NBR 5674, LGPD em condomínio, Código Civil art. 1.335)
- **Fonte**: briefing task §9; `LiveType.palestras_tecnicas`

### LMS14 — Sala de Live: Entrevista com Especialista

- **Caminho**: LMS10 → live `LiveType = entrevistas_especialistas`
- **Elementos**: formato entrevista (entrevistador + entrevistado), chat, perguntas do público filtradas
- **Contexto**: entrevistas com advogados condominiais, engenheiros, síndicos veteranos, autoridades setoriais (ABRACS, parceria M4+)
- **Fonte**: briefing task §9; `LiveType.entrevistas_especialistas`

### LMS15 — Meu Progresso

- **Caminho**: Home Academia → "Meu progresso" (ou avatar usuário → atalho)
- **Elementos**:
  - KPIs: cursos iniciados, cursos concluídos, horas de estudo, certificados emitidos, lives assistidas, tópicos criados/respondidos
  - Lista de certificados (com botão baixar/compartilhar)
  - Progresso das trilhas (barras %) — integra com `Enrollment.progressPercent` agregado
  - Recomendações: cursos sugeridos por comportamento (CNT-007 M3+) ou por registros de execução cross-domain (Req 23)
  - Badges de gamificação (Sprint 4+; SHOULD)
- **Fonte**: `Domínio - Conteúdo.md` §"Integração Cross-Domain" + "Análise de gaps"; `modulo-academia.md` §MÉTRICAS DO LMS (adminside); briefing task §9 (userside)

---

## 10. Regras de negócio do módulo

Consolidado de `content.md` Req 98.1 §"Cursos e Certificados"/§"Lives"/§"Fórum"; `functional/content.md` CNT-016..CNT-025; `business-rules.md`.

1. **Trilha = conjunto ordenado de cursos** (DAG linear). Progressão livre; certificado de trilha requer todos concluídos.
2. **Quiz**: aprovação ≥ 70% (default `passThreshold=70` em `Quiz`). Reprovação permite retry ilimitado.
3. **Certificado requer**:
   - Aprovação em 100% das aulas/quizzes do curso (`Enrollment.IsComplete()`)
   - Plano mínimo: `tier >= plus` para síndico/empresa; `resident_pagante` (tier plus+) para morador
   - Role elegível (§5)
4. **Parceiro Vizinhança sem acesso** — botão oculto + ABAC DENY (INV-067).
5. **Lives**: criação restrita a `admin` no MVP (D-062). Primeira live = apresentação oficial da plataforma.
6. **Lives replay**: disponível por 30 dias pós-transmissão.
7. **Fórum — moderação reativa**: botão "Denunciar" → fila admin. Admin remove conteúdo impróprio; audit trail obrigatório (CNT-027).
8. **Fórum — autoria**: moradores **apenas comentam**; empresas e síndicos criam tópicos (conhecimento técnico valioso — `Domínio - Conteúdo.md`).
9. **Cursos pagos**: integram com Billing (Stripe `IPaymentGateway`). Compra cria `Enrollment` automático.
10. **Cursos criados por empresa**: `tier=pro` exigido. Passam por **moderação admin antes do publish** (workflow: `draft → submitted → published|rejected`).
11. **Empresa Plus consome mas NÃO cria** cursos certificados (matriz §6).
12. **Admin Master Síndico cria cursos institucionais** sem moderação externa (bypass ABAC D-058).
13. **Recomendação cross-domain**: ao registrar `execution.validated` (Req 23 Commercial), engine sugere cursos relacionados na próxima sessão LMS (CNT-007).
14. **Certificado é verificável publicamente** via URL `/certificates/verify/{serialNumber}` (sem auth — serial único cripto-seguro).
15. **Biblioteca — ebooks**: PDF/EPUB; sem DRM; visibilidade por plano (CNT-025); signed URL TTL 24h.
16. **Lives exclusivas**: requerem tier `>= plus` OU whitelist admin (convite); `live_access_level` controla filtro ABAC.
17. **Gamificação** (SHOULD, Sprint 4+): badges por trilha concluída, leaderboard por categoria (não-social — respeita "rede social cega" CNT-012 no que cabe).

---

## 11. Invariantes (content — INV-066..INV-070, INV-068 Certificate)

| ID | Invariante | Ponto de enforcement |
|----|-----------|---------------------|
| **INV-068** | `Enrollment.progressPercent == 100` dispara `Certificate` (idempotente; canônico) | Handler `LessonCompleted` → check → emit `CertificateGenerated` |
| **INV-066** | Agência Marketing: token com scope `videos:*`, `analytics:read` — **sem acesso pedagógico completo** | ABAC policy por token; aplicável a consumo LMS limitado |
| **INV-067** | **Parceiro Vizinhança: sem acesso à Academia** (hard) | ABAC DENY; UI oculta botão em menu principal |
| **INV-068 (reforço op.)** | Certificate gerado **automaticamente** ao `Enrollment.progress = 100%` (não manual) — mesmo INV canônico acima visto pelo ângulo operacional | Domain event handler |
| **INV-069** | Ebook access filtrado por `plan_tier` | ABAC + query filter |
| **INV-070** | Signed playback URL Mux: TTL ≤ 24h — previne hotlinking de aulas | `IVideoProvider.GetPlaybackURL` seta TTL |
| **INV-LMS-A** (derivada) | Certificado imutável pós-issue; `serialNumber` UNIQUE verificável sem auth | Schema UNIQUE + factory `NewCertificate` lock |
| **INV-LMS-B** (derivada) | Curso `published` = imutável para lições core; edições geram **adendo** (análogo a Req 25 timeline) | `Course.RemoveModule` só em `status=draft` |
| **INV-LMS-C** (derivada — **RC-7**, **GR-029 análogo**) | **Lock 90d pós-publish** para conteúdo de curso certificado (evitar gaming: alterar prova após alunos completarem) | Trigger DB + domain check |
| **INV-LMS-D** (derivada) | Moderação obrigatória antes de `publish` para cursos de Empresa Pro | Workflow `submitted` → admin review → `published|rejected` |
| **INV-LMS-E** (derivada) | Fórum: moradores **não criam** tópicos — só comentam | ABAC ação `forum.topic.create` exige role `syndic|enterprise|admin` |
| **INV-LMS-F** (derivada) | Live agendamento: **somente admin** (D-062 MVP) | ABAC ação `live.schedule` só `role=admin` |

Fontes: `invariants.md` INV-066..INV-070 (INV-068 Certificate canônico); derivações da task briefing §11.

---

## 12. State machines

### 12.1 `Course`

```
      NewCourse()                         Publish()                    Archive()
────────────────────> [draft] ──────────────────────> [published] ────────────> [archived]
                        │                                  ▲
                        │ RemoveModule/AddModule (livre)   │
                        └──────────────────────────────────┘
                         apenas em draft
```

- `draft → published`: requer `admin` (curso institucional) ou workflow `submitted → admin approves` (empresa Pro). Seta `publishedAt=NOW()`, pode disparar `CourseContentLockedUntil = publishedAt + 90d` (INV-LMS-C / RC-7).
- `published → archived`: admin-only; remove do catálogo mas preserva certificados já emitidos.
- Nunca volta a `draft` (análogo a timeline imutável).

### 12.2 `Enrollment`

```
                    NewEnrollment()                   progress=100%
user matricula ──────────────────────> [active] ──────────────────────> [completed]
                                          │
                                          │ Cancel() (soft — user aborta)
                                          ▼
                                       [cancelled]
```

- `active`: padrão pós-matrícula. Pode avançar progress via `CompleteLesson` + `SubmitQuizScore`.
- `completed`: `progressPercent=100` + `completedAt=NOW()`. Dispara `CourseCompleted` (E-060) → certificate.
- `cancelled`: soft delete (preserva histórico); raro. Re-enroll cria novo `Enrollment` (UNIQUE quebrado? → política: 1 active por (user, course); cancelled permite re-enroll).

### 12.3 `Quiz` (attempt)

```
user inicia ────> [in_progress]
                       │
            score ≥ 70% │
                       ├────> [passed]  → marca módulo completo
            score < 70% │
                       └────> [failed]  → retry → novo attempt
```

### 12.4 `Certificate`

```
Enrollment 100% ────> NewCertificate ────> [issued]
                                              │
                                              │ Sign() (Sprint 3+)
                                              ▼
                                           [signed]   ← MP 2.200-2 ICP-Brasil
                                              │
                                              │ validUntil (opcional)
                                              ▼
                                           [expired]  (raro — evergreen default)
```

- Imutável: nunca volta a estado anterior. Nunca revogado (salvo fraude — admin-only audit trail).

### 12.5 `Live` LMS

```
admin agenda ────> [scheduled]
                        │
                        │ startedAt  (admin inicia stream)
                        ▼
                     [live]
                        │
                        │ endedAt (admin encerra)
                        ▼
                     [ended]
                        │
                        │ replay processado (Mux/CF Stream)
                        ▼
                     [replay_available]  ← 30 dias disponível
                        │
                        │ >30d pós-ended
                        ▼
                     [replay_expired]
```

Fontes: `Course.md`/`Enrollment.md`/`Certificate.md` métodos; `state-machines.md`; CNT-022/CNT-023.

---

## 13. Domain events

Catálogo em `domain-events.md` (v2).

| ID | Evento | Publisher | Payload | Subscribers |
|----|--------|-----------|---------|-------------|
| **E-057** | `CoursePublished` | content | `{course_id, category, area, trilha?, published_at}` | cross-domain (notificação alunos interessados por categoria/trilha) |
| **E-058** | `EnrollmentCreated` | content | `{enrollment_id, user_id, course_id, created_at}` | cross-domain (analytics produto) |
| **E-059** | `LessonCompleted` | content | `{enrollment_id, lesson_id, completed_at, progress_percent}` | content (handler interno: se `progress=100` emite `CourseCompleted`) |
| **E-060** | `CourseCompleted` | content | `{enrollment_id, course_id, completed_at}` | content (handler `GenerateCertificate`); cross-domain (badge gamification Sprint 4+) |
| **E-061** | `CertificateGenerated` | content | `{certificate_id, serial_number, user_id, course_id, issued_at}` | cross-domain (notification email com PDF + reputação commercial alimenta score de empresa se curso foi dela) |

Eventos implícitos a destilar (gap §16):

- `QuizAttempted` / `QuizPassed` (hoje só registros em tabela; sem evento formal)
- `ForumTopicCreated`, `ForumReplyPosted`
- `LiveScheduled`, `LiveStarted`, `LiveEnded`, `LiveReplayAvailable`
- `LibraryMaterialPublished`

Fontes: `domain-events.md` §content; `functional/content.md` CNT-019 "Evento NATS `course.completed` → emit `certificate.generated`".

---

## 14. Dependências inter-sub-produto

| Sub-produto | Papel na Academia |
|-------------|-------------------|
| **Content (vídeos)** | **Infraestrutura base**: toda aula LMS é um `Video` (Mux). LMS depende de `IVideoProvider`, upload presigned, signed playback URL TTL 24h (INV-070), thumbnail (CNT-032). Vídeos aula podem reaproveitar tabela `videos` com tipo `video_aula` |
| **Identity** | `membership` em tenant condômino determina role; role + tier são entrada do ABAC. Zitadel roles: `syndic|resident|enterprise|marketing|local_company|admin` |
| **Billing** | Tier universal (`trial|base|plus|pro`) governa acesso. Cursos pagos integram Stripe `IPaymentGateway` → enrollment automático pós-pagamento. Soft block sem grace period (D-066/D-067 ADR-0032) |
| **Commercial** | **Cross-recommendation**: ao validar `execution_record` (Req 23), engine sugere cursos relacionados à categoria de serviço. Certificado em cursos de "Relacionamento com síndicos" **alimenta reputação** (Bronze→Diamante) da empresa que emite/certifica |
| **Institutional** | Timeline pode anexar menções a cursos concluídos (ex.: síndico publica adendo: "completei curso de LGPD"). Plano Diretor — treinamentos recomendados podem vincular a atividades técnicas |
| **Assembly** | Compartilha conceito `LiveSession` (Livekit) mas **LMS Live usa Mux Live / Cloudflare Stream** (infra separada — pendência CNT-022) |
| **Reputação** | Certificado de curso oficial pode alimentar compliance/score (M4+ regra) |
| **Governança Institucional** | Recomendação por tipo de registro (Req 23) |
| **Banco de Talentos** | Ortogonal; morador com vídeo-currículo pode ter certificados LMS exibidos no perfil (M4+) |
| **Vizinhança** | **NENHUMA** — parceiro local_company não acessa (INV-067) |
| **Compliance** | Certificados LMS podem contar como indicador de governança em auditoria C1-C11 (M4+) |
| **Marketing Agencies** | Role `marketing` tem consumo limitado; não gera certificado (matriz §6) |
| **Administradoras Condominiais** | M4+: trilha exclusiva "Administradora Profissional" |

Fontes: `bounded-contexts.md` §content/commercial/identity; `Domínio - Conteúdo.md` §"Integração cross-domain"; ADR-0032.

---

## 15. Estado no código

| Fase | Escopo | Status |
|------|--------|--------|
| **M1 (2026-05-08)** | Identity + Billing + Institutional base | ✅ Fora do LMS (LMS não entra no MVP) |
| **M2 (2026-06-20)** | Connect Me + Conteúdo (vídeos institucionais, moderação, lock 90d, busca OpenSearch) | ✅ Infraestrutura de vídeo que LMS reaproveita |
| **M3 (2026-07-20)** | **LMS thin** + Assembleia + polish | 🟡 **LMS aqui**: infraestrutura agregados `Course/Enrollment/Certificate/Quiz` + 3 trilhas piloto (Síndico / Empresa / Morador) + ~5 cursos seeds de cada + fórum MVP + biblioteca seed + Lives **Admin MS + Empresa Pro** (D-097; primeira live = apresentação plataforma pela Admin MS) |
| **M4+** | LMS pleno | ⏸ **Parceria ABRACS** (Associação Brasileira das Administradoras de Condomínios — previsto): 50+ cursos, gamificação (badges, leaderboard), certificação digital plena (MP 2.200-2), assinatura digital ICP-Brasil, recomendação collaborative filtering, integração Slack |

**Código Go atual** (`automation-agents` contém specs — não código): zero módulo Go `content/lms/*` destilado aqui. Aggregates `Course.md`, `Enrollment.md`, `Certificate.md` existem como specs DDD em `01-domain/aggregates/`. Implementação em `backend/internal/modules/content/` (repo cliente João, fora deste vault) é M3.

**Coverage targets** (aplicáveis quando implementado): domain 95%, application 90%, infra 85%, http 75% (`vault/CLAUDE.md` §8).

Fontes: `ROADMAP.md`; `CLAUDE.md` §6 Marcos; `bounded-contexts.md` §5-content.

---

## 16. Gaps e pendências

| ID | Gap | Impacto | Próximo passo |
|----|-----|---------|---------------|
| **G-01** | Aggregate formal `ForumTopic`/`ForumPost` ausente em `01-domain/aggregates/` | Alto (fórum é uma das 5 áreas) | Destilar em Fase 8 dual-check |
| **G-02** | Aggregate formal `LibraryMaterial` ausente | Médio | Destilar |
| **G-03** | Aggregate formal `Live` (LMS — distinto de assembly `LiveSession`) ausente | Alto (infra diferente: Mux Live vs Livekit) | Destilar + ADR provedor (CNT-022) |
| **G-04** | Domain events `QuizAttempted`/`QuizPassed`, `ForumTopicCreated`, `LiveScheduled/Started/Ended`, `LibraryMaterialPublished` não formalizados | Médio | Adicionar em `domain-events.md` |
| **G-05** | Divergência de numeração LMS1–LMS15 entre briefing task (jornada lógica) e `modulo-academia.md` original (ini convertido) | Alto (documentação) | Dual-check com cliente; escolher uma numeração como verdade e atualizar |
| **G-06** | SLA de moderação (48h? pipeline semi-automatizado?) não fixa para cursos empresa | Médio | ADR (análogo CNT-003 vídeos) |
| **G-07** | Provedor de assinatura digital BR para certificado (ICP-Brasil MP 2.200-2) sem escolha (CNT-019 pendência) | Baixo (M4+) | Research-loop + ADR |
| **G-08** | Provedor Live LMS (Mux Live vs Cloudflare Stream) — CNT-022 pendência | Médio (bloqueia M3 Lives) | Research-loop + ADR |
| **G-09** | Certificado de **trilha** (não apenas de curso) não especificado em agregados | Baixo (M4+) | Adicionar novo agregado `TrailCertificate` ou campo em `Certificate` |
| **G-10** | Gamificação (badges, leaderboard) sem aggregate; SHOULD Sprint 4+ | Baixo | Backlog M4+ |
| **G-11** | Código Go `content/lms/*` ainda não existe (spec-only) | Alto mas esperado — M3 | Implementação Sprint 10 |
| **G-12** | Enum `CourseArea` vs `LibraryMaterialType` vs `LiveType` — taxonomia paralela não-hierárquica; confusão potencial entre "Vídeo institucional" (Req 29) × "Vídeo-aula de curso" (Req 98.1) × "Vídeo da Biblioteca" | Médio | Unificar em `ContentType` hierárquico ou documentar separação clara |
| **G-13** | Relação `Trilha ↔ Course` (1:N ordenada) sem agregado dedicado — hoje só campo `trilha *CourseTrilha` em `Course` | Médio | Criar agregado `Trail` (ordenação explícita, progress bar agregado) |
| **G-14** | **RC-7 lock 90d pós-publish em curso certificado**: regra citada na task, análogo a GR-029 (vídeo), mas **não formalizada** em `business-rules.md` nem em `Course.md` | Alto (integridade do certificado) | Adicionar BR- # em `business-rules.md` + método `Course.Lock()` |
| **G-15** | Recomendação cross-domain (Req 23 → cursos) — fluxo descrito mas código e evento não destilados | Médio | Adicionar no serviço `Recommendation` domain cross |

---

## 17. Referências cruzadas

### Canônicas da v2 (`master-sindico-v2/`)

- [[../../01-domain/bounded-contexts#5-content]]
- [[../../01-domain/aggregates/Course]]
- [[../../01-domain/aggregates/Enrollment]]
- [[../../01-domain/aggregates/Certificate]]
- [[../../01-domain/aggregates/LiveSession]] (BC assembly, referência infra)
- [[../../01-domain/domain-events]] §E-057..E-061
- [[../../01-domain/invariants]] INV-066..INV-070 (INV-068 Certificate canônico)
- [[../../01-domain/state-machines]] (Course/Enrollment)
- [[../../01-domain/business-rules]] BR-033 (lock 90d análogo)
- [[../../04-requirements/functional/content]] CNT-016..CNT-025
- [[../../04-requirements/matrix-functional]]
- [[../../04-requirements/global]] GR-029 (lock 90d)
- [[../../02-architecture/adr/0032-global-plans-stripe-style]]
- [[../../02-architecture/adr/0010-mux-video-provider]]
- [[../../STATE]] D-056 (agnostic), D-134 (admin app dedicada — revoga D-058), D-097 (lives admin + Empresa Pro + agência delegada — revoga D-062/D-063/D-076), D-080 (planos Stripe-style)
- [[../../ROADMAP]] §M3
- [[../../00-product/sub-produtos/_moc]]

### Legado / cliente (destilado)

- `vault/50-projects/master-sindico/specs/requirements/content.md` Req 98.1
- `vault/50-projects/master-sindico/specs/requirements/functional-matrix.md` §Academia LMS
- `vault/50-projects/master-sindico/specs/requirements/personas-and-plans.md`
- `vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Domínio - Conteúdo.md` v3.0
- `vault/50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Mapa Academia.md` (canvas)
- `vault/50-projects/master-sindico/client-vault/01 - Arquitetura de Sistema/System Architecture - Content.md`
- `vault/50-projects/master-sindico/client-vault/03 - Modelagem e Dados/Enums e Listas Mestre.md` v3.0
- `vault/50-projects/master-sindico/_archive/content-discovery-2026-03/modulo-academia.md` — **LMS1–LMS15 originais**

### Sub-produtos relacionados

- [[04-conteudo-videos]] — infraestrutura Mux (aula = Video)
- [[05-assembleia-live]] — LiveSession (Livekit) — paralelo ao Live LMS (Mux Live)
- [[01-governanca-institucional]] — recomendação cross-domain Req 23
- [[03-reputacao]] — certificados alimentam score
- [[07-banco-de-talentos]] — certificados podem aparecer no perfil morador (M4+)
- [[08-vizinhanca]] — **EXCLUÍDO** (INV-067)
- [[10-marketing-agencias]] — consumo LMS limitado

---

## Anexo A — Rotas de API (legado `Domínio - Conteúdo.md` Req 98.1)

```
GET    /v1/academy/courses            → Listar cursos (filtros: category, level, access_level, trilha)
GET    /v1/academy/courses/:id        → Detalhe curso (LMS4)
POST   /v1/academy/courses/:id/enroll → Matricular (cria Enrollment, dispara Stripe se pago)
POST   /v1/academy/courses/:id/lessons/:lid/complete → Marca aula concluída (dispara LessonCompleted E-059)
POST   /v1/academy/courses/:id/quiz/:qid/submit       → Submete quiz (retorna score + status)
GET    /v1/academy/trainings          → Listar treinamentos
GET    /v1/academy/lives              → Listar lives (agenda + replays)
POST   /v1/academy/lives/:id/remind   → "Lembrar-me" (subscription)
POST   /v1/academy/lives              → [admin + enterprise:pro — D-097 Fase 11] agendar live
GET    /v1/academy/forum              → Listar tópicos (filtros: category)
POST   /v1/academy/forum              → Criar tópico [role=syndic|enterprise|admin]
POST   /v1/academy/forum/:id/reply    → Responder
POST   /v1/academy/forum/:id/like     → Curtir
POST   /v1/academy/forum/:id/resolve  → Marcar resolvido [autor | admin]
GET    /v1/academy/library            → Listar conteúdos (tipo, categoria)
GET    /v1/academy/library/:id        → Detalhe material (signed URL)
GET    /v1/academy/paths              → Trilhas de aprendizado
GET    /v1/academy/certificates       → Meus certificados
GET    /certificates/verify/:serial   → [público, sem auth] verificar certificado
```

OpenAPI 3.1 contrato obrigatório (`vault/CLAUDE.md` §8 item 11).

---

## Anexo B — Glossário

- **Academia** = Academia Master Síndico, nome comercial do LMS.
- **Trilha** = conjunto ordenado de cursos que forma uma jornada pedagógica por persona.
- **Certificado** = documento PDF com `serialNumber` único, verificável publicamente, emitido ao completar 100% de curso, requer plano mínimo.
- **Quiz** = avaliação por módulo, aprovação ≥ 70% (default), retries ilimitados.
- **Live** = transmissão ao vivo agendada, criada por **Admin MS + Empresa Pro** (D-097 Fase 11 — sobrescreve D-062 admin-only MVP), 4 tipos (`LiveType`), 2 níveis de acesso (`LiveAccessLevel`), replay 30d.
- **Biblioteca** = repositório de materiais avulsos (6 tipos — `LibraryMaterialType`), consulta livre.
- **Fórum** = espaço discursivo em 7 categorias (`ForumCategory`); moradores apenas comentam.
- **ABRACS** = Associação Brasileira das Administradoras de Condomínios (parceria M4+ para currículo pleno).
- **RC-7** = regra de conteúdo nº 7 — lock 90d pós-publish (análogo a GR-029 para vídeo; extensão para curso certificado — gap G-14).
- **MP 2.200-2** = Medida Provisória que estabelece ICP-Brasil; baseia assinatura digital de certificado (Sprint 3+).
