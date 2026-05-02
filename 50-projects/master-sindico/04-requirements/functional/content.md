---
title: Functional Requirements — Content BC
type: requirement
tags:
  - requirements
  - functional
  - content
  - video
  - mux
  - lms
  - forum
  - live
  - master-sindico
  - v2
  - ears
source: >-
  legacy specs/requirements/content + novo escopo-7 + cross-domain +
  _chaos/requirements(6).md (2026-04-13) + _chaos/mastersindico-requirements.pdf
  (2026-03-09)
created: 2026-04-23T00:00:00.000Z
updated: '2026-04-25'
fase_c_absorvido: true
---

# Functional Requirements — Content BC

> **Banner Fase C (2026-04-25)**: enriquecido com R96 (QR Code) + R29 (12 tipos vídeo institucional canônicos) + LMS enums (categorias/níveis/tipos live) absorvidos do `_chaos/`. D-133 `BroadcastLive` mapeado como aggregate futuro M2 (provider via ADR-0044 dual-check).

Requisitos funcionais do **bounded context `content`**. Responsabilidade: vídeos institucionais (Mux + lock 90d + moderação), busca via `ISearchProvider` real (OpenSearch ou Meilisearch — ADR-0042 pending; D-120 revoga D-067 PG tsvector), privacidade de métricas cross-tenant, LMS (cursos, módulos, quizzes, certificados), fórum, lives institucionais (D-097: admin + Empresa Pro + agência delegada — revoga D-063/D-076 admin-only), ebooks.

Formato **EARS**. Prioridade: M (M1), S (M2), C (M3+), W (backlog).

> Destilado de `specs/requirements/content.md` + `novo escopo-7.md` §1.4, §1.7, §4.1-4.7 + matriz funcional.

---

## CNT-001 — Upload de vídeo via Mux presigned URL

**Prioridade**: M  
**EARS**: *Quando* user autorizado (GR-002) emite `POST /videos` com metadados, *o sistema deve* (1) validar quota (BIL-017/BIL-019), (2) validar lock 90d (GR-029), (3) chamar `IVideoProvider.CreateDirectUpload()` → Mux retorna `upload_url + mux_asset_id`, (4) persistir `videos {id, user_id, company_id, type, status='pending_upload', mux_asset_id}`, (5) retornar `upload_url` pro frontend.

**Dependências**: GR-019, BIL-017, BIL-019, GR-029.

**Critério de aceite**:
- Presigned URL TTL 15min.
- Client PUT direto pro Mux (não passa pelo backend).
- Webhook `video.asset.ready` triggera `status=ready`.

**Fonte**: legado `content.md` Req 29, `novo escopo-7.md` §1.4 + §1.7.

---

## CNT-002 — 12 tipos de vídeo institucional canônicos

**Prioridade**: M  
**EARS**: *Quando* vídeo é criado por empresa/síndico/admin, *o sistema deve* aceitar `video_type` em enum 12 itens canônicos (R-_chaos absorção Fase C). Subtipos especiais ortogonais: `institucional` (default), `instrucional` (LMS), `video_curriculum` (Banco de Talentos).

**Tipos canônicos (12 itens — R-_chaos Tipos de Vídeo Institucional Fase C)**:

```
apresentacao_institucional, apresentacao_equipe_tecnica, demonstracao_servico,
explicacao_tecnica, boas_praticas_condominiais, orientacao_preventiva,
caso_real_atendimento, treinamento_tecnico, explicacao_legislacao_normas,
dicas_manutencao, conteudo_educativo_sindicos, conteudo_educativo_moradores
```

> **Nota Fase C**: lista anterior `{apresentacao_empresa, portfolio_servicos, depoimento_cliente, tutorial, noticia_comunicado, evento_registrado, demonstracao_tecnica, consultoria, webinar, explicativo, entrevista, outro}` substituída pela canônica do `_chaos/requirements(6).md`. Migrar enum se DB já populado (expand-contract — CLAUDE.md §12).

**Critério de aceite**:
- Enum Postgres `video_institutional_type` (12 itens).
- Subtipo (institucional/instrucional/curriculum) é coluna separada `video_subtype`.
- Tipo determina regras de visibilidade (CNT-008) e lock (GR-029).
- Tipo `caso_real_atendimento` exige aprovação reforçada na moderação (CNT-003) — pode envolver dados de clientes.

**Fonte**: legado `content.md` Req 29; **R-_chaos enums absorção Fase C** (12 tipos canônicos pt-BR alinhados a contexto condominial).

---

## CNT-003 — Moderação de vídeo (pipeline editorial)

**Prioridade**: M  
**EARS**: *Quando* webhook Mux `video.asset.ready`, *o sistema deve* (1) atualizar `status=pending_moderation`, (2) adicionar à fila admin. Admin review → `approved` (publish + lock 90d) ou `rejected` (com motivo).

**Critério de aceite**:
- Tabela `video_moderations {video_id, status, moderator_id, reason, moderated_at}`.
- SLA: 48h em M1 (manual).
- Vídeo rejeitado: empresa notificada, pode recorrer.

**Fonte**: legado `content.md` Req 29, [[00-product/portfolio-de-produtos]] Produto 4.

---

## CNT-004 — Publish + lock 90d

**Prioridade**: M  
**EARS**: *Quando* moderação aprova, *o sistema deve* setar `published_at=now()`, `locked_until = published_at + 90 days`. *Enquanto* `locked_until > now()` e user tenta substituir vídeo do mesmo tipo → `409 video_locked`.

**Dependências**: GR-029.

**Critério de aceite**:
- Campo `locked_until` consultado em CNT-001.
- UI mostra "Você pode atualizar este vídeo em X dias".

**Fonte**: legado `content.md` Req 29 rule 7, `novo escopo-7.md` §2.4.

---

## CNT-005 — Busca full-text (OpenSearch)

**Prioridade**: S  
**EARS**: *Quando* usuário busca, *o sistema deve* consultar índices OpenSearch: `videos, companies, syndics, local_partners`. Retornar paginado (20/página) com filtros: categoria, localização (geo), rating, tipo de vídeo.

**Critério de aceite**:
- `ISearchProvider` real obrigatório M1 (D-120 Fase 14 revoga D-067 que dizia tsvector); ADR-0042 `proposed pending dual-check` decide OpenSearch vs Meilisearch. PG tsvector vigente é stub temporário.
- Autocomplete enquanto digita.
- Evento `video.published` → re-index.

**Fonte**: legado `content.md` Req 31, `novo escopo-7.md` §1.5.

---

## CNT-006 — Facetas e filtros geo

**Prioridade**: S  
**EARS**: *Quando* busca tem `?cep` ou `?geo=<lat,lng>`, *o sistema deve* filtrar por proximidade (ex: 5km). Retornar resultados ordenados por `_score` + distância.

**Critério de aceite**:
- Índice geo OpenSearch ou PostGIS.
- Raio configurável (default 5km).

**Fonte**: legado `content.md` Req 31 "SHOULD".

---

## CNT-007 — Recomendações baseadas em comportamento

**Prioridade**: C (M3+)  
**EARS**: *Quando* morador clica em vídeo, *o sistema deve* registrar click em `video_interactions {user_id, video_id, action, timestamp}`. Engine noturno gera recomendações (materialized view) por usuário.

**Critério de aceite**:
- Ranking: views + CTR + completion_rate.
- Collaborative filtering M4+.

**Fonte**: legado `content.md` Req 32.

---

## CNT-008 — Paywall Base: preview 25% em vídeos empresa

**Prioridade**: M  
**EARS**: *Quando* Morador Base acessa vídeo de empresa, *o sistema deve* servir stream truncado aos 25% (via Mux signed playback URL com duration limit). Síndicos Plus/Pro e Moradores Pagantes → integral.

**Dependências**: [[../matrix-functional]].

**Critério de aceite**:
- Signed playback URL com `policy.duration_limit`.
- Matriz: Base/Pagante/N1/N2 → preview; N3 → integral (matriz funcional).
- Mensagem: "Assine um plano superior pra ver o vídeo completo".

**Fonte**: legado `content.md` Req 29 Rule 4, `novo escopo-7.md` §1.7, matriz funcional "Vídeos empresa (preview 25%)".

---

## CNT-009 — Visibilidade de vídeo empresa aos moradores (Rule 4)

**Prioridade**: M  
**EARS**: *Enquanto* não há votação de fornecedor ativa em assembleia, *o sistema deve* bloquear acesso de morador a vídeo de empresa (exceto preview 25% conforme CNT-008). *Quando* votação de fornecedor abre e `autorizar_exibicao_votacao=true` na empresa, *o sistema deve* emitir `video_visibility_grants` temporário. *Quando* votação encerra → revoga grant.

**Dependências**: ASM-021, GR-002.

**Critério de aceite**:
- Tabela `video_visibility_grants {video_id, assembly_id, granted_at, revoked_at}`.
- Evento NATS `assembly.voting.closed` → `video.visibility.revoked`.
- Middleware ABAC valida grants ativos.

**Fonte**: legado `content.md` Req 29 Rule 4, `cross-domain.md` Req 103.

---

## CNT-010 — Player de vídeo customizado

**Prioridade**: M  
**EARS**: *Quando* user acessa vídeo, *o sistema deve* renderizar player Mux (HLS) com: play/pause, seek, fullscreen, qualidade adaptativa, legendas (M2+).

**Critério de aceite**:
- Web: Mux Player SolidJS.
- Mobile: player nativo iOS/Android (Flutter).

**Fonte**: legado `novo escopo-7.md` §1.7.

---

## CNT-011 — Contador de visualização ≥ 70%

**Prioridade**: S  
**EARS**: *Quando* user assiste vídeo ≥ 70%, *o sistema deve* incrementar counter `video_analytics` (view only). < 70% não conta.

**Critério de aceite**:
- Frontend envia pulse a cada 15s com `watch_percentage`.
- Backend grava em `video_views`.

**Fonte**: legado `novo escopo-7.md` §1.7, regra de pontuação síndico.

---

## CNT-012 — Privacidade de métricas ("rede social cega")

**Prioridade**: M  
**EARS**: *Quando* user acessa conteúdo próprio, *o sistema deve* exibir contadores de likes, lista de comentários, total views. *Quando* user acessa conteúdo de OUTRO → UI cega (sem contadores, sem comentários visíveis).

**Dependências**: GR-003 (tenant isolation aplicado a métricas).

**Critério de aceite**:
- ABAC regra: `content.metrics.read` só pro dono.
- PBT: cross-tenant API retorna 403 em `/videos/{id}/metrics`.

**Fonte**: legado `novo escopo-7.md` §2.3, §4.2.

---

## CNT-013 — Likes e comentários (registrados, só visíveis ao dono)

**Prioridade**: S  
**EARS**: *Quando* user dá like/comenta, *o sistema deve* persistir em `content_likes`, `content_comments`. *Quando* não-dono consulta → array vazio.

**Critério de aceite**:
- Likes por `(user_id, content_id)` UNIQUE.
- Comentários com audit trail.
- Reativa moderação (botão denunciar).

**Fonte**: legado `novo escopo-7.md` §4.2.

---

## CNT-014 — Agência de Marketing: upload em nome de empresa

**Prioridade**: S (M3)  
**EARS**: *Quando* agência (via token IDN-025 — **login Zitadel com role `marketing`** per [[../../STATE#d-095-—-agência-de-marketing-tem-login-zitadel-painel-admin-próprio-fecha-q4|D-095 Fase 11]]) emite `POST /videos` com `company_id`, *o sistema deve* validar scope (`videos:upload`), criar vídeo atribuído à empresa (não à agência).

**Critério de aceite**:
- Audit trail: marker `marketing_agency_user_id`.
- Agência NÃO vê deals, Connect Me, usuários.
- **Agência TEM login próprio Zitadel com role `marketing`** + painel admin próprio (8 telas MK1-MK8 em `03-subprojects/web/ui-catalog/marketing/`). Opera sempre delegada a uma empresa via token de escopo restrito (ABAC grant).

**Fonte**: ~~legado `content.md` Req 30~~ (versão "sem login" era STALE — **fechada via D-095 Fase 11**, 2026-04-24). Canônico atual alinha com T7 identity + ADR-0032 + `personas-and-plans.md`.

---

## CNT-015 — Subtítulos automáticos Mux

**Prioridade**: C (M3+)  
**EARS**: *Quando* vídeo aprovado, *o sistema deve* acionar geração de subtítulos automáticos via Mux API.

**Fonte**: legado `content.md` Req 29 "SHOULD".

---

## CNT-016 — LMS: estrutura Curso → Módulo → Aula

**Prioridade**: C (M3 thin)  
**EARS**: *Quando* admin/empresa Pro cria curso, *o sistema deve* aceitar hierarquia: `Course → Module → Lesson {video_id, descricao, anexos[]}`.

**Critério de aceite**:
- Tabelas `courses`, `course_modules`, `course_lessons`.
- Admin (Master Síndico) e Empresa Pro podem criar cursos certificados.
- Empresa Plus NÃO pode criar (matriz funcional).

**Fonte**: legado `content.md` Req 98.1, `novo escopo-7.md` §4.1.

---

## CNT-017 — LMS: 5 áreas × 10 categorias × 3 trilhas

**Prioridade**: C  
**EARS**: *Quando* catálogo é publicado, *o sistema deve* organizar em:
- **5 áreas**: Cursos, Treinamentos, Lives, Fórum, Biblioteca.
- **10 categorias**: Governança, Legislação, Gestão Financeira, RH, Manutenção, Compliance, Segurança, Relações Humanas, Empreendedorismo, Tecnologia.
- **3 trilhas**: Síndico, Empresa, Morador.

**Fonte**: legado `content.md` Req 98.1.

---

## CNT-018 — LMS: quiz + aprovação ≥ 70%

**Prioridade**: C  
**EARS**: *Quando* aluno responde quiz, *o sistema deve* calcular score. *Quando* score ≥ 70% → marca aula/módulo como `completed`; *quando* < 70% → permite retry.

**Critério de aceite**:
- Tabelas `quiz_questions`, `quiz_responses`.
- Feedback imediato.

**Fonte**: legado `content.md` Req 98.1, [[00-product/portfolio-de-produtos]] Produto 6a.

---

## CNT-019 — LMS: certificado automático ao 100%

**Prioridade**: C  
**EARS**: *Quando* aluno completa 100% do curso, *o sistema deve* gerar certificado PDF (nome, curso, data, QR code de validação) via job async.

**Critério de aceite**:
- Certificado armazenado em S3.
- URL pública de validação (`/certificates/verify/{qr_code}`).
- Assinatura digital BR (M4+ pleno).

**Fonte**: legado `content.md` Req 98.1, `novo escopo-7.md` §4.1.

---

## CNT-020 — LMS: acesso por persona

**Prioridade**: C  
**EARS**: *Quando* user acessa LMS, *o sistema deve* aplicar matriz:
- Morador Base: Lives, Fórum, Biblioteca (sem certificado).
- Morador Pagante: Cursos, Treinamentos, Lives, Fórum, Biblioteca + certificado.
- Síndico N1: sem cursos certificados.
- Síndico N2/N3: cursos + certificado.
- Empresa Plus: consumir cursos + certificado.
- Empresa Pro: **criar** cursos certificados.
- Parceiro Vizinhança: **NÃO acessa** (botão oculto).

**Dependências**: [[../matrix-functional]].

**Critério de aceite**:
- ABAC regra por feature.

**Fonte**: legado `content.md` Req 98.1, `novo escopo-7.md` §4.1, matriz funcional.

---

## CNT-021 — LMS: fórum (7 categorias, moderação reativa)

**Prioridade**: C  
**EARS**: *Quando* user autorizado cria tópico, *o sistema deve* aceitar categoria + texto. Moradores apenas comentam; empresas e síndicos criam. Moderação reativa via botão "Denunciar".

**Dependências**: matriz funcional.

**Critério de aceite**:
- Tabelas `forum_topics`, `forum_posts`, `forum_likes`.
- 7 categorias configuráveis.
- Full-text search.

**Fonte**: legado `content.md` Req 98.1, `novo escopo-7.md` §4.3.

---

## CNT-022 — Lives: restritas ao Admin Master Síndico (Rule 5.7)

**Prioridade**: C (M3+)  
**EARS**: *Quando* admin inicia live, *o sistema deve* expor RTMP ingest via Mux Live ou Cloudflare Stream. *Quando* síndico/empresa tenta iniciar → 403.

**Critério de aceite**:
- Só admin role permite criação.
- Live fica disponível para replay 30d.

**Fonte**: legado `content.md` Req 98.1 (Lives), `novo escopo-7.md` §4.4 + §5.7.

---

## CNT-023 — Lives: 4 tipos

**Prioridade**: C  
**EARS**: *Quando* admin agenda live, *o sistema deve* aceitar `live_type ∈ {aberta, exclusiva_plan, ao_vivo_com_replay, seminar_serie}`.

**Critério de aceite**:
- Calendário público.
- Chat tempo real (moderado, admin toggle).

**Fonte**: legado `content.md` Req 98.1.

---

## CNT-024 — Lives: acesso por persona (matriz)

**Prioridade**: C  
**EARS**: *Quando* user tenta acessar live, *o sistema deve* aplicar matriz:
- Base: só replay.
- Pagante/Síndico/Empresa/Marketing: live + replay.
- Parceiro Vizinhança: sem acesso.

**Fonte**: legado `novo escopo-7.md` §4.4, matriz funcional.

---

## CNT-025 — Ebooks e conteúdo digital

**Prioridade**: C  
**EARS**: *Quando* admin/empresa Pro publica ebook, *o sistema deve* armazenar em S3 (PDF, EPUB). Acesso: Plus/Pro (matriz). Sem DRM; visibilidade controlada via tenant + plan.

**Critério de aceite**:
- Tabela `digital_content`.
- Signed URL 24h.

**Fonte**: legado `content.md` Req 97, `novo escopo-7.md` §4.1.

---

## CNT-026 — Podcast (integração YouTube)

**Prioridade**: C  
**EARS**: *Quando* aba Podcast é acessada, *o sistema deve* listar episódios via YouTube Data API (canal oficial). Cache 1h.

**Critério de aceite**:
- iframe ou player nativo.
- Filtros por plano (pagantes+).

**Fonte**: legado `novo escopo-7.md` §4.5.

---

## CNT-027 — Admin: moderação global

**Prioridade**: M (básica)  
**EARS**: *Quando* admin acessa painel, *o sistema deve* expor: aprovar/rejeitar vídeo, remover vídeo violador, suspender funcionalidades empresa reincidente, moderar post do fórum.

**Critério de aceite**:
- Audit trail completo.
- Motivos padronizados.

**Fonte**: legado `novo escopo-7.md` §4.6.3.

---

## CNT-028 — Admin: broadcast (comunicado oficial plataforma)

**Prioridade**: M  
**EARS**: *Quando* admin emite broadcast, *o sistema deve* enviar email/SMS segmentado por persona/plano. **Único canal oficial de comunicação em massa**.

**Critério de aceite**:
- Editor (texto + CTA).
- Agendamento.
- Log de disparos.

**Fonte**: legado `novo escopo-7.md` §4.6.5 + §2.2.

---

## CNT-029 — Admin: configurações globais

**Prioridade**: M  
**EARS**: *Quando* admin configura, *o sistema deve* permitir edição de: textos de termos, política de privacidade, banners home, categorias técnicas, parâmetros globais (tempo janela votação, limite vídeos/mês, etc.).

**Critério de aceite**:
- Versionamento de textos (GR-023).

**Fonte**: legado `novo escopo-7.md` §4.6.4.

---

## CNT-030 — Vídeo-currículo: visualizar (empresas/síndicos via matriz)

**Prioridade**: S (M3 principal)  
**EARS**: *Quando* user autorizado (Plus/Pro/Marketing/Vizinhança) acessa vídeo-currículo de morador pagante, *o sistema deve* servir full (sem paywall — matriz funcional "Visualizar currículos").

**Dependências**: COM-029, COM-028.

**Critério de aceite**:
- Player padrão.
- Botão "Demonstrar Interesse" (CNT-031).

**Fonte**: legado `novo escopo-7.md` §4.7.

---

## CNT-031 — Vídeo-currículo: "Demonstrar Interesse"

**Prioridade**: S  
**EARS**: *Quando* empresa clica "Demonstrar Interesse", *o sistema deve* notificar morador via push/email (sujeito a restrições GR-002 §1.6 "sem notificações sociais"). Operacionalmente: segue regras Connect Me.

**Critério de aceite**:
- Rate limit por empresa.
- Morador pode ignorar.

**Fonte**: legado `novo escopo-7.md` §4.7.

---

## CNT-032 — Thumbnail de vídeo

**Prioridade**: M  
**EARS**: *Quando* vídeo é processado pelo Mux, *o sistema deve* usar thumbnail gerado automaticamente (frame arbitrário ou definido por user).

**Critério de aceite**:
- Storyboard Mux disponível.
- User pode fazer override.

**Fonte**: legado `content.md`.

---

## CNT-033 — Paywall: vídeos institucionais de Síndico N3 (morador integral)

**Prioridade**: M  
**EARS**: *Quando* morador acessa vídeo de síndico com `plan_tier=pro` (D-067/D-081/D-096), *o sistema deve* servir integral (matriz funcional).

**Fonte**: legado matriz funcional "Vídeos empresa (integral)".

---

## CNT-034 — Video analytics (privacidade por dono)

**Prioridade**: S  
**EARS**: *Quando* dono do vídeo emite `GET /videos/{id}/analytics`, *o sistema deve* retornar: views, avg_watch_time, completion_rate, retention_rate. *Quando* não-dono → 403.

**Dependências**: GR-011, CNT-012.

**Critério de aceite**:
- ABAC regra.
- Cálculos via job noturno.

**Fonte**: legado `content.md` Req 29, `novo escopo-7.md` §4.2.

---

## CNT-035 — Anexos a pauta (vídeo/áudio por item)

**Prioridade**: S  
**EARS**: *Quando* síndico anexa vídeo/áudio a item de pauta de assembleia, *o sistema deve* permitir upload via Mux + vinculação `assembly_agenda_items.attachment_url`.

**Dependências**: ASM-008.

**Fonte**: legado `novo escopo-7.md` §2.1.

---

## Fase C — Deltas absorvidos 2026-04-25

### CNT-036 — QR Code Generator (multi-purpose)

**Prioridade**: M (M1 condominium QR; M2 partner_links e event_check_in)  
**EARS**: *Quando* admin/sistema gera QR Code, *o sistema deve* (R96 absorção Fase C):
- Aceitar 3 tipos de finalidade: `condominium_access` (M1 — onboarding morador via scan), `partner_link` (M2 — Vizinhança redirect), `event_check_in` (M3 — assembleia ASM-012).
- Suportar formatos de export: PNG, SVG, PDF.
- Aceitar customização: `size, color (hex), logo_embedding (URL)`.
- Conter payload assinado HMAC-keyed (ADR-0028) com `{type, target_id, issued_at, expires_at?}`.
- Tracking: `qr_scans {id, qr_code_id, scanner_user_id?, scanned_at, ip_address, scan_location?}` append-only.
- Regeneração: invalidar QR antigo (`revoked_at`), emitir novo `qr_code_id` distinto.
- Suporte a expiração TTL pra acesso temporário.
- Emit events: `QRCodeGenerated`, `QRCodeScanned` (anonimizado se scanner não autenticado).

**Casos de uso M1**:
- Onboarding morador: síndico imprime QR do `condominium_id` em quadro de avisos → morador escaneia → app abre `/onboarding?condominium_id=X` (M1 obrigatório — INS-001 já gera QR no create condominium).

**Casos de uso M2+**:
- Vizinhança: parceiro recebe QR do seu perfil pra imprimir em estabelecimento (link curto `/p/{partner_id}`).
- Assembleia: QR de check-in com TTL = `assembly.scheduled_date + 1 hour` (ASM-012 modalidade `qr_code`).

**Critério de aceite**:
- Tabela `qr_codes {id, type, target_id, payload_hmac, expires_at, revoked_at, created_by_user_id, created_at}`.
- Payload HMAC: `HMAC(qr_code_id, {type, target_id, issued_at})` chave em ADR-0028.
- Endpoint `GET /qr/scan?token=...` valida HMAC + TTL antes de redirect.
- LGPD: `qr_scans` retém 90d (analytics), depois purge ou agrega.

**Dependências**: ADR-0028 (HMAC keyed), INS-001 (condominium QR), COM-030 (partner profile), ASM-012 (event check-in).

**Fonte**: **R96 absorção Fase C** (`_chaos/requirements(6).md`).

---

### CNT-037 — LMS taxonomies canônicas (categorias, níveis, lives, fórum, biblioteca)

**Prioridade**: C (M3+ — alinhado com CNT-016..CNT-021)  
**EARS**: *Quando* LMS publica curso/live/post/biblioteca, *o sistema deve* aceitar enums canônicos (R-_chaos LMS enums absorção Fase C):

**LMS Course Categories (10 itens)**:
```
gestao_condominial, saude_ambiental, seguranca_predial, manutencao_predial,
legislacao_condominial, mediacao_conflitos, tecnologia_condominios,
sustentabilidade, administracao_financeira, outros
```

**LMS Course Levels (3 itens)**: `basico, intermediario, avancado`.

**LMS Live Access Types (2 itens)**: `abertas` (todos), `exclusivas` (planos específicos ou convite).

**LMS Live Event Types (4 itens — R-_chaos)**:
```
debates_condominiais, lancamento_solucoes, palestras_tecnicas, entrevistas_especialistas
```

**LMS Forum Categories (7 itens)**:
```
gestao_condominial, manutencao_predial, seguranca, saude_ambiental,
tecnologia, convivencia_moradores, legislacao_condominial
```

**LMS Library Content Types (6 itens)**:
```
videos, guias_tecnicos, manuais, livros_digitais, ebooks, materiais_tecnicos
```

> **Nota Fase C**: CNT-017 menciona "5 áreas × 10 categorias × 3 trilhas" — consolidar com este enum canônico em fase de implementação. A "10 categorias" do CNT-017 alinha com `LMS Course Categories` (10 itens). As "5 áreas" (Cursos, Treinamentos, Lives, Fórum, Biblioteca) são módulos do LMS, não enums.

**Critério de aceite**:
- 5 enums Postgres ou catalog tables: `lms_course_category` (10), `lms_course_level` (3), `lms_live_event_type` (4), `lms_forum_category` (7), `lms_library_content_type` (6).
- Seed migration alinhado com R-_chaos.

**Fonte**: legado `content.md` Req 98.1; **R-_chaos LMS enums absorção Fase C**.

---

### CNT-038 — Voice Input (speech-to-text pt-BR) — backlog acessibilidade

**Prioridade**: C (M3+ acessibilidade)  
**EARS**: *Quando* user clica botão "🎤 Falar" em campo de descrição (timeline, comunicado, master_plan), *o sistema deve* (R95 absorção Fase C — backlog M3+):
- Integrar com Browser Speech Recognition API (Web) ou native iOS/Android speech-to-text (Mobile).
- Display recording indicator (UI feedback visual).
- Suportar pause/resume durante gravação.
- Permitir editing transcribed text antes de submission (não auto-submit).
- Suportar pt-BR language como default.
- Fallback gracioso para text input se speech recognition não disponível ou falha.
- Emit `VoiceInputUsed` event (analytics — NÃO armazena audio raw).

**Critério de aceite**:
- M1-M2: feature DESLIGADA (não implementada).
- M3+: avaliar via feature flag (BIL-022 rollout gradual).
- Sem dependência de provider externo (zero custo backend; usa Web Speech API ou native).
- Acessibilidade WCAG (R91): voice input atende usuários com mobilidade reduzida.

**Dependências**: R91 acessibilidade (non-functional.md), BIL-022 (feature flags).

**Fonte**: **R95 absorção Fase C** — backlog acessibilidade.

---

## Invariantes de content

1. Vídeo imutável após publicação (edição via adendo).
2. Trava trimestral 90d entre atualizações do mesmo tipo.
3. Vídeo-currículo: único por morador (não galeria).
4. Visibilidade de vídeo empresa a morador: apenas via preview 25% OU durante votação de fornecedor.
5. Métricas só visíveis ao dono ("cega" para terceiros).
6. Agência: zero acesso a dados de negócio.
7. Parceiro Vizinhança: sem LMS.
8. Certificado emitido só em 100% (quiz ≥ 70%).
9. Lives restritas ao Admin.
10. Upload direto via Mux presigned (backend nunca proxy).

---

## Pendências

- ⚠️ **PENDÊNCIA CNT-003**: SLA exata de moderação (48h M1; pipeline semi-automatizado M3+) — definir.
- ⚠️ **PENDÊNCIA CNT-005**: Meilisearch temporário vs OpenSearch direto — ADR.
- ⚠️ **PENDÊNCIA CNT-019**: provedor de assinatura digital BR de certificado — ADR M4+.
- ⚠️ **PENDÊNCIA CNT-028**: template de segmentação (por role, plan, condomínio?) — dual-check.
- ⚠️ **PENDÊNCIA CNT-022**: Mux Live vs Cloudflare Stream — ADR research-loop.

---

## Referências

- Legado: `specs/requirements/content.md`, `novo escopo-7.md` §1.4, §1.7, §4.1-4.7, §5.6-5.7, `cross-domain.md` Req 103, `contextos/MUX_VIDEO_CONTEXT.md` (101K).
- [[../global]] GR-018, GR-019, GR-029.
- [[../../01-domain/bounded-contexts#content]]
- [[../../00-product/portfolio-de-produtos]] Produto 4, 6a.
- [[../matrix-functional]] — Consumo Conteúdo, Publicação, LMS.
- [[_moc]]
