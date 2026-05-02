---
title: Content — Vídeos, Busca, Academia LMS e Ebooks
version: 1.0
status: stable
type: requirement
domain: content
tags: [video, mux, search, opensearch, lms, academy, ebook, marketing-agency]
---

# Content — Vídeos, Busca, Academia LMS e Ebooks

Domínio de conteúdo: vídeos institucionais via Mux, busca full-text, academia LMS integrada e ebooks digitais.

---

## Req 29 — Vídeos Institucionais

Upload de vídeos via Mux com quotas, trava trimestral e visibilidade restrita.

### 12 Tipos de vídeo

1. Apresentação da empresa
2. Portfólio de serviços
3. Depoimento de cliente
4. Tutorial/como fazer
5. Notícia/comunicado
6. Evento registrado
7. Demonstração técnica
8. Consultoria
9. Webinar
10. Vídeo explicativo
11. Entrevista
12. Outro

### Quotas mensais por plano

| Plano | Uploads/mês |
|---|---|
| Síndico N1 | 4 |
| Síndico N2 | 8 |
| Síndico N3 | 12 |
| Empresa Plus | 8 |
| Empresa Pro | 12 |

### Requisitos MUST

- Upload via **Mux** (`IVideoProvider`) — codificação automática HLS
- Storage: MinIO (self-hosted) ou Cloudflare R2 (prod)
- **Trava trimestral: 90 dias** entre atualizações do mesmo vídeo (Rule 7)
- Análise: views, avg_watch_time, completion_rate, retention_rate
- Paywall Base: truncar stream aos 25% (preview)

### Visibilidade (Rule 4)

- **Moradores**: SOMENTE durante votação de fornecedor em assembleia (Req 57)
  - Condição: `autorizar_exibicao_votacao = true` (flag na empresa)
  - Acesso revogado automaticamente **após fim da votação**
- Síndicos: sempre acesso completo

### Requisitos SHOULD

- Subtítulos automáticos via Mux (Sprint 4+)
- Recomendações baseadas em comportamento (Req 32)
- Social sharing com controle de visibilidade (Sprint 3+)

### Notas de implementação

- Tabelas: `videos`, `video_analytics`, `video_visibility_grants`
- Webhook Mux: `video.ready` → atualiza status
- Evento NATS: `video.visibility.revoked` → notificação à empresa
- Campo: `updated_at`, `last_updated_trimester` para tracking de trava

---

## Req 30 — Agência de Marketing

Ator delegado (não é tenant) que gerencia conteúdo em nome de empresa.

### Requisitos MUST

- **Não é tenant** — não tem login próprio, não é persona com role
- Convidada por empresa via token (Sprint 5)
- Upload em nome da empresa (vídeo atribuído à empresa, não à agência)
- **Restrições**: SÓ gerencia vídeos e analytics
  - ❌ Não pode acessar dados de negócio (deals, propostas)
  - ❌ Não pode gerenciar usuários da empresa
  - ❌ Não pode acessar Connect Me entrada/saída
- Revogação imediata: token invalidado, sem acesso

### Tokens de acesso

- Gerado por admin da empresa
- TTL configurável (ex: 90 dias)
- Escopo: `videos:upload`, `videos:edit`, `analytics:read`
- Audit log de todas as ações (quem, quando, o quê)

### Notas de implementação

- Tabelas: `marketing_agency_tokens`, `marketing_agency_access_logs`
- Enum: `token_scope`
- Middleware: valida token (tipo delegado) antes de permitir upload

---

## Req 31 — Busca e Descoberta

Full-text search com filtros e autocomplete.

### Requisitos MUST

- **Stack**: OpenSearch (Sprint 5) — ou Meilisearch temporário (Sprint 2-4)
- Indexação de: vídeos, empresas, profissionais (síndicos N2/N3), parceiros
- Filtros: categoria, localização (geolocalização), rating, tipo de vídeo
- Autocomplete: sugestões enquanto digita
- Resultado paginado (20 por página)

### Funcionalidades SHOULD

- Busca facetada (refine por múltiplos critérios)
- Busca com proximidade de location (ex: 5km de distância)
- Histórico de buscas (últimas 10 — storage local)

### Notas de implementação

- Índices: `videos`, `companies`, `syndics`, `local_partners`
- Campo `_score` para relevância
- Sync: NATS listener em `video.created`, `company.updated`, etc → re-indexar

---

## Req 32 — Recomendações

Engine baseado em comportamento de usuário.

### Requisitos MUST

- Click-through tracking: quando morador clica em vídeo, registra
- Recomendações: baseadas em vídeos similares (mesma categoria, tipo)
- Ranking: views + CTR + completion_rate
- Personalizado por morador (não global)

### Requisitos SHOULD

- Collaborative filtering (Sprint 4+): usuários parecidos assistem vídeos parecidos
- A/B testing de algoritmo de recomendação (Sprint 5+)

### Notas de implementação

- Tabelas: `video_interactions` (clicks, views, watch_time)
- Cálculo: job noturno que gera recomendações (materialized view)
- Endpoint: `GET /recommendations` (autenticado, per-user)

---

## Req 97 — Ebooks e Conteúdo Digital

Acesso a PDFs e EPUBs por plan premium.

### Requisitos MUST

- Formatos: PDF, EPUB
- Storage: MinIO
- Acesso: restrito a planos Plus/Pro (validação ABAC)
- Biblioteca organizada por categoria
- Sem DRM (direitos gerenciados via tenant isolation)

### Notas de implementação

- Tabelas: `digital_content` (tipo: book)
- Campo: `access_level` (free, plus, pro)

---

## Req 98.1 — Academia LMS

Academia integrada com 5 áreas, 3 trilhas, cursos certificados.

### 5 Áreas

1. **Cursos** — aprendizado estruturado
2. **Treinamentos** — sessões práticas
3. **Lives** — transmissões ao vivo com Q&A
4. **Fórum** — discussão entre usuários
5. **Biblioteca** — recursos extras (PDFs, links)

### 10 Categorias de curso

1. Governança condominial
2. Legislação (LGPD, trabalhista, etc)
3. Gestão financeira
4. RH & folha de pagamento
5. Manutenção predial
6. Compliance
7. Segurança
8. Relações humanas
9. Empreendedorismo
10. Tecnologia

### 3 Trilhas (caminho recomendado)

1. **Síndico**: módulos de governança + legislação + gestão financeira
2. **Empresa**: compliance + RH + legislação + segurança
3. **Morador**: direitos/deveres + segurança + comunidade

### Acesso por persona

| Persona | Áreas | Cursos | Certificados |
|---------|-------|--------|---|
| Morador Base | Lives, Fórum, Biblioteca | Visualizar | Sem certificado |
| Morador Pagante | Cursos, Treinamentos, Lives, Fórum, Biblioteca | Visualizar + Fazer | Certificado |
| Síndico N1+ | Cursos, Treinamentos, Lives, Fórum, Biblioteca | Fazer | Certificado |
| Empresa Plus+ | Cursos, Treinamentos, Lives, Fórum, Biblioteca | Fazer | Certificado (criar) |
| Parceiro Vizinhança | — | — | — |

**Regra**: Parceiro Vizinhança **NÃO** tem acesso (botão oculto).

### Cursos e Certificados

- **Estrutura**: módulos + lições + quiz
- **Certificado automático**: ao atingir 100% de conclusão
- **Validade**: indefinida (conteúdo evergreen)
- **Emissão**: PDF certificado disponível para download

### Lives

- **4 tipos**: aberta, exclusiva (plan), ao vivo com replay, seminar série
- **Agendamento**: calendário público
- **Replay**: disponível por 30 dias após transmissão
- **Q&A**: chat em tempo real (moderado)

### Fórum

- **7 categorias**: por domínio de conhecimento
- **Tópicos**: criados por qualquer membro
- **Restrição**: empresas podem criar tópicos (síndicos também); moradores apenas comentam
- **Moderação**: admin remove conteúdo impróprio
- **Etiquetas**: para classificação

### Integração cross-domain

- **Recomendação automática**: ao registrar execução (Req 23), sistema recomenda cursos de treinamento relevante
- **Análise de gaps**: dashboard síndico mostra "Você completou 60% da trilha Síndico"

### Requisitos MUST

- Conteúdo: 50+ cursos básicos (Sprints 6-7)
- 5 trilhas com sequenciamento lógico
- Quiz com feedback imediato
- Certificado PDF com assinatura digital (Sprint 3+)
- Busca em fórum (full-text via OpenSearch)
- Notificação de nova resposta no fórum

### Requisitos SHOULD

- Gamificação: badges por conclusão de trilha (Sprint 4+)
- Leaderboard por categoria (top aprendizes — Sprint 5+)
- Integração com Slack para notificações (Sprint 6+)

### Notas de implementação

- Tabelas: `courses`, `course_modules`, `course_lessons`, `quiz_questions`, `quiz_responses`, `certificates`, `forum_topics`, `forum_posts`, `forum_likes`, `forum_subscriptions`
- Enums: `course_status` (draft, published, archived), `certificate_type`, `live_type`
- Evento NATS: `course.completed` → emit `certificate.generated` + send email
- Recomendação: após `execution.validated` → query cursos relevantes por categoria

---

## Invariantes de content

1. Vídeo é imutável após publicação (edição via adendo — Req 25)
2. Trava trimestral: 90 dias mínimo entre atualizações do mesmo vídeo
3. Visibilidade de vídeo: moradores só durante votação de fornecedor
4. Agência de Marketing: acesso zero a dados de negócio
5. Acesso Parceiro Vizinhança: sem Academia LMS
6. Certificado: emitido automaticamente ao 100% de conclusão

---

## Referências cruzadas

- **Req 29** (Content): visibilidade de vídeo durante assembleia (Req 57)
- **Req 23** (Commercial): recomendação automática de treinamentos pós-execução
- **Req 6.2** (Institutional): vídeo institucional de síndico (trava trimestral)
- **Personas e Planos**: `personas-and-plans.md` — quotas de vídeo por plan
- **Functional Matrix**: `functional-matrix.md` — acesso LMS por persona × plano
