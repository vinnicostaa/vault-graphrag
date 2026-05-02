---
title: System Architecture — Content Domain
parent: System Architecture
domain: CONTENT
status: Not Started
tags:
  - mastersindico
  - architecture
  - content
  - lms
  - talent
  - ddd
---

# HIGH-DOMAIN: Content

> **Status:** Schema de vídeos existente, módulos novos — Sprint 4+
> **Tenant Isolation:** Híbrido (`owner_id` + `tenant_type`)
> **Módulos:** 4 (1 existente + 3 novos)
> **Source of Truth:** Requirements 29, 93, 95-98.1, Decision 6

---

## Visão Geral

O domínio Content gerencia toda a produção e consumo de conteúdo na plataforma: vídeos institucionais, cursos educacionais, fórum de discussão, e o Banco de Talentos (currículos de moradores).

```
CONTENT (Tenant: owner_id + tenant_type)
├── VideoModule              — Vídeos institucionais + Mux HLS ⚠️ (expandir)
├── ContentInteractionModule — Likes, ratings, fórum (NOVO)
├── AcademiaModule           — LMS: cursos, trainings, lives, certificados (NOVO)
└── TalentBankModule         — Banco de Talentos com 11 telas (NOVO)
```

---

## VideoModule (EXISTENTE — expandir)
**Req:** 29, 93

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | Video, VideoView, VideoComment, VideoLike |
| **Provider** | Mux API (HLS adaptive streaming) |
| **Quotas** | Plan-based: número de uploads e storage |
| **Visibilidade** | Plan-based (Base→apenas vídeos do plano, Pro→acesso completo) |
| **Lives** | Decidir: feature única ou separada (plataforma vs Academia) → **Decision 9** |

**Events:**
- `content.video.uploaded`
- `content.video.processed`
- `content.video.hidden`

---

## ContentInteractionModule (NOVO)
**Req:** 95

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | Like, Rating, ForumTopic, ForumReply |
| **Fórum** | Topics categorizados, busca via Meilisearch |
| **Moderação** | Reativa (reports) + manual (admin) |

**Events:**
- `content.interaction.liked`
- `content.interaction.rated`
- `content.interaction.forum-topic-created`

---

## AcademiaModule (NOVO)
**Req:** 96, 97, 98, 98.1

LMS completo para educação continuada de síndicos, moradores e empresas.

| Aspecto | Detalhe |
|---------|---------|
| **Entities** | Course, Lesson, Training, LiveEvent, Certificate, LearningPath, LibraryContent |
| **Tipos Curso** | Gratuito, pago, exclusivo por plano |
| **Certificados** | Gerados automaticamente ao completar curso |
| **Lives Educacionais** | Streaming ao vivo para cursos (verificar se compartilha infra com Lives gerais) |
| **Biblioteca** | Conteúdo textual complementar |
| **Target Audience** | Síndicos, moradores, empresas |

**Events:**
- `content.academy.course-created`
- `content.academy.course-completed`
- `content.academy.certificate-issued`
- `content.academy.live-started`

---

## TalentBankModule (NOVO — Decision 6)
**Req:** Decision 6 (11 telas + 7 funcionalidades empresa)

> [!WARNING]
> **Divergência corrigida:** O blueprint mencionou como footnote. É um módulo completo com LGPD pesado.
> **Pendente confirmação:** Entra no escopo atual? → Decision pendente do client.

### Lado Morador — 11 Telas de Cadastro

| Tela | Conteúdo |
|------|----------|
| 01 | Welcome + explicação |
| 02 | Vídeo currículo (90s, Mux, lock de 90 dias) |
| 03 | Perfil profissional (6 perguntas abertas) |
| 04 | Áreas de interesse (18 áreas + função desejada) |
| 05 | Dados pessoais (nome, telefone, CEP, CNH, NRs) |
| 06 | Disponibilidade (tipos contrato, horários, início) |
| 07 | Pretensão salarial (mínimo + ideal) |
| 08 | Experiência profissional (últimos 3-5 vínculos) → **Decision pendente** |
| 09 | Formação educacional + auto-avaliação |
| 10 | Consentimento LGPD (obrigatório) |
| 11 | Revisão final + submissão |

### Lado Empresa — 7 Funcionalidades

1. **Busca avançada** com filtros (Meilisearch)
2. **Sistema de favoritos** (salvar perfis)
3. **Exportação PDF** de currículos
4. **Histórico de buscas** com relatórios
5. **Analytics de funil** (buscas → views → favoritos → contatos)
6. **Auto-tags** (7 categorias calculadas automaticamente)
7. **Tags manuais** (privadas por empresa)

### Schema Principal

```
curricula:
  morador_id (unique — 1 currículo por morador)
  status: draft | active | suspended | deleted
  
  -- Dados pessoais, disponibilidade, salário
  full_name, phone, email, cep, age, marital_status
  cnh_type, nr_courses[], contract_types[], available_schedules[]
  start_availability, commute_time, min_salary, ideal_salary
  
  -- LGPD
  lgpd_consented_at, lgpd_consent_ip, lgpd_consent_version
  
  -- Tabelas relacionadas:
  curriculum_videos (Mux, 90s, lock 90 dias)
  curriculum_professional_profiles (6 perguntas)
  curriculum_areas_of_interest (18 áreas)
  curriculum_work_experience (3-5 últimos)
  curriculum_education
```

### Meilisearch Index

```
Index: curricula
  Searchable: areas_of_interest, availability, contract_types, nr_courses
  Filterable: areas_of_interest, cnh_type, nr_courses, availability, 
              contract_type, commute_time, salary_range
  Visibility: Companies only (Plus/Pro)
```

**Events:**
- `content.talent.curriculum-created`
- `content.talent.curriculum-updated`
- `content.talent.curriculum-viewed`
- `content.talent.curriculum-favorited`
- `content.talent.curriculum-exported`

---

## Navegação

← [[System Architecture - Commercial]] | [[System Architecture]] | [[System Architecture - Assembly]] →
