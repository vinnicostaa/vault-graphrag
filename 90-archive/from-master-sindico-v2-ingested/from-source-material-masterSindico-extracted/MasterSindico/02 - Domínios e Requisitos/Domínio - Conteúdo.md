---
title: "Domínio - Conteúdo"
version: "3.0"
date: 2026-03-10
status: "em-revisão"
high_domain: "Conteúdo"
requisitos: "Req 29–32, 97–98.1"
total_requisitos: 8
tags:
  - mastersindico
  - dominio
  - conteudo
  - video
  - lms
  - academia
---

# 📚 Domínio Principal: Conteúdo

> **High-Domain:** Conteúdo | **Requisitos:** 29–32, 97–98.1 | **Telas:** LMS1–LMS15
> Distribuição de conteúdo, streaming, busca, academia, fórum, biblioteca.
> **Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] > [[Arquitetura do Ecossistema|🏛️ Arquitetura]] > 📄 Domínio - Conteúdo

---

## Visão do Domínio

![[Dominio Conteudo.canvas]]

---

## Requisito 29: Biblioteca de Vídeos Institucionais

> **História do Usuário:** Como empresa, eu quero fazer upload de vídeos institucionais, para que eu possa demonstrar meus serviços.

### Critérios de Aceite Principais

| # | Critério |
|---|----------|
| 1 | Upload com: `title`, `description`, `video_type`, `thumbnail`, checkbox `autorizar_exibicao_votacao` |
| 2 | **12 tipos de vídeo** (enum completo em [[Enums e Listas Mestre]]) |
| 3 | Integração com streaming (Mux/Cloudflare) via `Adapter_Layer` |
| 4 | Quotas mensais por plano (ver tabela abaixo) |
| 5 | Status: `processing`, `ready`, `failed`, `deleted`, `hidden` |
| 6 | Ocultar vídeo mantém no banco mas remove da visibilidade pública |
| 7 | Analytics: `views`, `avg_watch_time`, `completion_rate`, `retention_rate` |

### Regra Crucial — Visibilidade de Vídeos para Moradores

> [!CAUTION]
> Moradores **NÃO** acessam vídeos de empresas automaticamente. Acesso existe **SOMENTE** durante votação de fornecedor em assembleia, e **SOMENTE** se `autorizar_exibicao_votacao = true`. Após votação, acesso é **BLOQUEADO** automaticamente.

![[Visibilidade Videos.canvas]]

### Limites de Upload por Plano

| Plano | Vídeos/mês | Acesso a Vídeos de Empresa |
|-------|-----------|---------------------------|
| Síndico N1 | 4 | 25% preview |
| Síndico N2 | 8 | 25% preview |
| Síndico N3 | 12 | Completo |
| Empresa Plus | 8 | — |
| Empresa Pro | 12 | — |

### Rotas de API (Req 29)

```
POST   /v1/videos                    → Upload de vídeo
GET    /v1/videos/:id                → Detalhe do vídeo
PUT    /v1/videos/:id/hide           → Ocultar vídeo
GET    /v1/videos/:id/analytics      → Analytics do vídeo
GET    /v1/videos/search             → Buscar vídeos
```

---

## Requisito 30: Gestão de Agência de Marketing

> Agência é **actor delegado** — não é tenant, opera com permissão da empresa.

| Regra | Detalhe |
|-------|---------|
| Link | 1 agência pode atender múltiplas empresas |
| Upload | Agência faz upload **em nome da empresa** |
| Atribuição | Vídeo é atribuído à empresa, não à agência |
| Dashboard | Empresas gerenciadas, vídeos publicados, total views, vídeo mais acessado |
| Restrições | **SÓ** gerencia vídeos e analytics. NÃO acessa: Connect Me, registros, comunicados, Timeline, dados de síndicos/moradores |

---

## Requisitos 30.1–30.3: Gestão de Usuários e Tracking

### Req 30.1 — Gestão de Usuários da Empresa
Admin adiciona usuários: `administrador`, `operador_tecnico`. Permissões do operador: `registrar_execucao`, `criar_atividade_tecnica`, `enviar_comunicados`, `publicar_videos`. Limite de usuários por plano.

### Req 30.2 — Tracking de Submissões
Centraliza todos os envios da empresa: registros, atividades, comunicados. Status: `rascunho` → `enviado` → `aguardando_validacao` → `solicitado_ajuste` → `aprovado` → `publicado` / `rejeitado`.

### Req 30.3 — Detalhe da Submissão
Mostra feedback do síndico, permite reeditar quando `solicitado_ajuste`, mantém histórico de versões.

---

## Requisitos 31–32: Busca e Recomendações

### Req 31 — Busca e Descoberta
Full-text search com ranking de relevância. Filtros: `service_category`, `location`, `rating`, `video_type`. Autocomplete, trending searches, popular content.

### Req 32 — Recomendações Personalizadas
Engine analisa comportamento do síndico (buscas, visualizações, Connect Me). Recomenda: empresas, vídeos, categorias de serviço. Click-through tracking.

---

## Requisito 97: Ebooks e Conteúdo Digital

Formatos: PDF, EPUB. Metadata: title, author, description, category, cover. Acesso por plano premium. DRM opcional.

---

## Requisito 98.1: Academia Master Síndico (LMS Completo)

> **15 telas** · **5 áreas** · **3 trilhas de aprendizado** · **45 critérios de aceite**
> Acesso: Síndicos ✅ · Empresas ✅ · Moradores ✅ · **Parceiro ❌** (botão escondido)

### Mapa da Academia

![[Mapa Academia.canvas]]

### Área 1: Cursos

| Campo | Detalhe |
|-------|---------|
| Estrutura | Módulos → Aulas → Material complementar → Certificado |
| Tipos | Gratuitos e Pagos |
| **10 categorias** | `gestao_condominial`, `saude_ambiental`, `seguranca_predial`, `manutencao_predial`, `legislacao_condominial`, `mediacao_conflitos`, `tecnologia_condominios`, `sustentabilidade`, `administracao_financeira`, `outros` |
| **3 níveis** | `basico`, `intermediario`, `avancado` |
| Progresso | `aulas_concluidas`, `progresso_percentual` |
| Pagamento | Integração com módulo de billing |

### Área 2: Treinamentos

Treinamentos **curtos e práticos** (diferente de cursos que têm módulos):

`treinamento_porteiros` · `treinamento_seguranca_predial` · `treinamento_prevencao_pragas` · `treinamento_emergencia` · `treinamento_atendimento_morador` · `treinamento_primeiros_socorros`

### Área 3: Lives

| Campo | Detalhe |
|-------|---------|
| Funcionalidades | Agenda, participação ao vivo, chat, envio de perguntas |
| **4 tipos** | `debates_condominiais`, `lancamento_solucoes`, `palestras_tecnicas`, `entrevistas_especialistas` |
| Acesso | `abertas` (todos) ou `exclusivas` (planos específicos/convites) |
| Replay | Lives gravadas ficam disponíveis para visualização posterior |
| "Lembrar-me" | Registro para lives futuras com notificação |

### Área 4: Fórum

**7 categorias:** `gestao_condominial`, `manutencao_predial`, `seguranca`, `saude_ambiental`, `tecnologia`, `convivencia_moradores`, `legislacao_condominial`

Ações: `criar_topico`, `responder_topico`, `curtir_resposta`, `marcar_como_resolvido`

> [!NOTE]
> **Empresas podem criar tópicos** — elas possuem conhecimento técnico valioso para a comunidade.

### Área 5: Biblioteca

**6 tipos de conteúdo:** `videos`, `guias_tecnicos`, `manuais`, `livros_digitais`, `ebooks`, `materiais_tecnicos`

Exemplos: Guia Prevenção Pragas, Manual Manutenção Predial, Guia Convivência Condominial, Manual Síndico Iniciante.

### Trilhas de Aprendizado

| Trilha | Conteúdos Recomendados |
|--------|----------------------|
| **Trilha Síndico** | Gestão condominial, Mediação de conflitos, Legislação, Administração financeira |
| **Trilha Empresa** | Boas práticas, Segurança do trabalho, Saúde ambiental, Relacionamento com síndicos |
| **Trilha Morador** | Convivência condominial, Uso responsável de áreas comuns, Sustentabilidade, Segurança |

### Integração Cross-Domain

A Academia integra com Governança e Timeline: recomenda treinamentos baseados em registros de atividade (ex: registro de manutenção → recomenda treinamento de prevenção de pragas).

### Rotas de API (Req 98.1)

```
GET    /v1/academy/courses            → Listar cursos (filtros)
GET    /v1/academy/courses/:id        → Detalhe do curso
POST   /v1/academy/courses/:id/enroll → Matricular
GET    /v1/academy/trainings          → Listar treinamentos
GET    /v1/academy/lives              → Listar lives
POST   /v1/academy/lives/:id/remind   → "Lembrar-me"
GET    /v1/academy/forum              → Listar tópicos
POST   /v1/academy/forum              → Criar tópico
GET    /v1/academy/library            → Listar conteúdos
GET    /v1/academy/paths              → Trilhas de aprendizado
GET    /v1/academy/certificates       → Meus certificados
```

---

**Breadcrumbs:** [[00 - Índice Master Síndico|🏠 Início]] · **Anterior:** [[Domínio - Comercial]] · **Próximo:** [[Domínio - Assembleia]]
